# S-CTO-28-MODERATION-RUNTIME — EVIDENCE

**Session:** S-CTO-28-MODERATION-RUNTIME · READ-ONLY runtime/provider investigation · Env: STAGING (read) · Prod: FORBIDDEN
**Date:** 2026-09-06 · **Own-session status:** FINISHED — COMPLETE (read-only; zero changes)
**Writable:** this evidence file only. No product/config/provider change, no deploy.

---

## EXECUTIVE ANSWER

**Does content moderation run at runtime on staging?** PARTIALLY. Two independent moderation subsystems exist and behave very differently:

1. **Text moderation (Bedrock `moderateText`)** — **RUNS LIVE, but ADVISORY-ONLY.** It fires on every publication create/update (and bulk-upload, and syndication captions), calls AWS Bedrock, logs a warning if it flags content, and **never blocks** the seller. It began emitting the `ai_moderation_*` analytics events **only from 2026-09-04** (see runtime evidence). Confirmed live via Amplitude event counts AND `/bedrock/invocations` "content moderator" prompt calls in the same window.

2. **Image moderation (Rekognition + Bedrock image-text)** — **RUNS LIVE on the web-v2 path only.** Invoked by the standalone `POST /image/validate` endpoint, which the web-v2 create flow calls per image before upload. It **hard-blocks** (throws `400 IMG_EXPLICIT_CONTENT` / `IMG_CONTAINS_EXPLICIT_TEXT`). Its firing is **NOT visible in Amplitude** by design — it logs to the S3 training buffer (`moderation_image_passed/rejected`), not to `ta_analytics_event`.

**Has it EVER fired live?**
- `ai_moderation_triggered`: **27 total, all 2026-09-04→09-05, ZERO before.** (Registry's "fires ZERO times" was TRUE until ~Sep 4.)
- `ai_moderation_passed`: **27 total, same window.** (triggered == passed exactly.)
- `ai_moderation_rejected`: **NEVER — event type does not even exist in the Amplitude project taxonomy.**
- Bedrock "content moderator" model invocations: **≥10 in last 3 days** (Sep 4–5), corroborating the text path.

**Which paths BYPASS moderation? (Q3 — CONFIRMED)** Mobile/legacy clients that do **not** send `x-api-version: v2` get `shouldModerate=false`. On that path the image-upload controller skips even the `verifyChecksum` gate, so **image moderation is entirely bypassed for mobile/legacy**. Text `moderateText` still runs on create for all paths but is advisory (non-blocking) regardless.

**Automated vs human `is_moderated=true`?** **EXCLUSIVELY HUMAN.** `Publication.isModerated=true` is set only by `approvePublication()`, which is reached only via `verifyPublication()` — the BO Verify step. **No automated moderation pass sets `Publication.isModerated`.** The advisory text pass and the image `/image/validate` pass never touch the feed-eligibility gate. (Note: the per-image row `PublicationImage.isModerated` is a *different* flag, set to `shouldModerate` at upload — do not conflate.)

---

## FIVE-LINE OPENERS

**Initial:**
- BRANCH: <verifying>
- MISSION: S-CTO-28 — prove whether content-moderation fires at runtime on staging & on which paths. READ-ONLY.
- ORDER: reality check → source trace → runtime evidence → path coverage → gaps → evidence
- BLOCKERS: DB/ECS-Exec in-VPC may be UNKNOWN; provider counts may be inferable only
- TREE STATE: <verifying>

**Corrected (post-preflight):**
- BRANCH: `Ramzi_V2` @ `8d55a20` (matches expected; clean; = origin/Ramzi_V2)
- MISSION: same
- ORDER: reality check ✓ → source trace ✓ → runtime evidence ✓ (Amplitude+CloudWatch+Bedrock) → path coverage ✓ → gaps ✓ → evidence (this file)
- BLOCKERS: none required a STOP. DB direct read not attempted (Amplitude mirror + provider logs sufficed); noted as residual.
- TREE STATE: clean; wrote only this evidence file

## API PROVENANCE
- Repo: `tawadoo_api_js` · Branch `Ramzi_V2` · HEAD `8d55a207c88a6b125b47316cb361c69d463a3b54` (**matches expected 8d55a20**) · = `origin/Ramzi_V2` · clean working tree.

## SKILLS
- `tawadoo-source-truth` — DISCOVERED + ACTIVATED (disclose_context) + APPLIED (source trace, all-callers map, evidence-state ladder).
- `tawadoo-runtime-provider-qa` — DISCOVERED + ACTIVATED + APPLIED (Amplitude/CloudWatch/Bedrock runtime evidence, sanitized, read-only).
- `tawadoo-data-sovereignty` — DISCOVERED + ACTIVATED + APPLIED (identified the sovereign `ta_analytics_event` first-write path vs Amplitude mirror vs S3 training buffer sinks).

---

## FULL SOURCE TRACE (call sites + conditions, file:line)

### A. Image moderation (Rekognition + Bedrock image-text)
- **Provider call:** `aws/rekognition.service.ts:20` `detectModerationLabels()` → `DetectModerationLabelsCommand` with `MinConfidence = REKOGNITION_MIN_CONFIDENCE` (:21).
- **Orchestrator:** `image/image.service.ts:44` `validateImage(buffer, publicationId?, entityId?)`:
  - Redis short-circuit on `mod:v:<md5>` (:48) → returns valid if cached `'1'`.
  - `:53` `rekognitionService.detectModerationLabels(buffer)` → if any label → `400 IMG_EXPLICIT_CONTENT` (HARD BLOCK) + `trainingDataLogger.logModerationImageRejected(...)`.
  - `:74` `bedrockService.moderateImageText(buffer)` → if `!isValid` → `400 IMG_CONTAINS_EXPLICIT_TEXT` (HARD BLOCK) + rejection log.
  - On pass: set Redis `mod:v:<md5>='1'` (TTL `VALIDATION_CACHE_TTL_SECONDS`) + `logModerationImagePassed(...)`.
- **Entry point (web-v2):** `image/image.controller.ts:130` `validateImage()` = `POST /image/validate` → `imageService.validateImage(file.buffer)` (:171). No api-version gate on this endpoint itself; it is the front-end's pre-upload validate call.
- **Second image caller:** `publication/publication.service.ts:3958` `imageService.validateImage(optimizedImageBuffer)` inside `autoValidateLegacyMedia()` (:3945) — re-moderates legacy media on demand.
- **Video frames:** `video/video.service.ts:318` loops frames → `rekognitionService.detectModerationLabels(frameBuffer)` (:318) → blocks on labels.
- **Bedrock image-text:** `aws/bedrock.service.ts:132` `moderateImageText()` (called from image.service:74).

### B. Text moderation (Bedrock `moderateText`) — the `ai_moderation_*` emitter
- **Provider call + event emit:** `aws/bedrock.service.ts:318` `moderateText(title, description, identity?)`:
  - `:382` `if (this.aiTracking)` → `trackModerationTriggered({moderationType:'text', ...})` (:383); then `trackModerationPassed` (:390) if `isValid===true` else `trackModerationRejected` (:397).
  - `this.aiTracking` is wired at boot by `intelligence-enrichment.module.ts:80` `bedrockService.setAiPipelineTracking(...)` in `onModuleInit()` (:76). Setter at `bedrock.service.ts:117`.
- **Event tracker impl:** `intelligence-enrichment/services/ai-pipeline-tracking.service.ts:214` `ai_moderation_triggered`, `:240` `ai_moderation_passed`, `:267` `ai_moderation_rejected` → `analyticsIngestionService.trackServerEvent(...)` (source `ANALYTICS_SOURCE.AI_PIPELINE`, `setImmediate` fire-and-forget).
- **Sovereign write path:** `analytics-ingestion/analytics-ingestion.service.ts:197` `trackServerEvent` → single writer to `ta_analytics_event` (dedup ledger `:119`), then outbox delivery → mirrored to Amplitude.
- **Callers of `moderateText` (all advisory / non-blocking):**
  - `publication/publication.service.ts:3895` inside `validatePublicationText()` (:3845) — explicitly commented **"ADVISORY ONLY (logs, does not block)"**. Called on create `:274` and update `:560`.
  - `bulk-upload/processors/bulk-upload.processor.ts:180` `validatePublicationText(title, description)` (Step 3 of bulk import).
  - `syndication/services/content-generator.service.ts:238` `moderateText(caption, caption)` — moderates AI-generated social captions (rejected captions flagged, not posted).
- **Separate text policy (NOT the ai_moderation_* emitter):** `publication/services/content-policy.service.ts:357` `moderateTextContent()` (own Bedrock prompt; comment says real moderation is at admin verify).

### C. The feed-eligibility gate (`Publication.isModerated`)
- Set `true` ONLY in `publication/publication.service.ts` `approvePublication()` (the `isModerated=true; isVerified=true` transaction). Reached via `verifyPublication()` (BO Verify) — proven by `verify-sets-is-moderated.spec.ts` ("verifyPublication now calls approvePublication").
- State machine `moderation-state-machine.ts`: transition pending→approved only. Create/update reset `isModerated=false` (publication.service.ts:480).
- Feed/index eligibility requires `isModerated:true` (`shared/feed-eligibility.query.ts`, `INDEX_ELIGIBILITY_WHERE`).
- **No automated path sets `Publication.isModerated=true`.** Advisory text pass and `/image/validate` do not.

### D. Image-row flag (do not conflate with C)
- `publication-image.controller.ts` upload: computes `shouldModerate = (ENABLE_CONTENT_MODERATION==='true') && (apiVersion==='v2')` (:122–125). When true, calls `verifyChecksum(file.buffer)` (:157/:128) + `validateImageLimits/validateVideoLimits`, and stamps `PublicationImage.isModerated = shouldModerate` on the created image row (:137, :171-ish). `verifyChecksum` (`publication-image.service.ts:268`) only checks the Redis `mod:v:<md5>` key exists (proof that `/image/validate` ran) — it does **not** itself call Rekognition.

---

## RUNTIME EVIDENCE (read-only)

### Amplitude (lake mirror), project `795817`
| Event | Exists in taxonomy? | Total (180d) | Timeline |
|---|---|---|---|
| `ai_moderation_triggered` | Yes | **27** | 0 for Mar–Aug; **4 on 2026-09-04, 23 on 2026-09-05**, 0 on 09-06 |
| `ai_moderation_passed` | Yes | **27** | identical distribution to triggered |
| `ai_moderation_rejected` | **NO — event type does not exist** | **0 (never)** | never landed |

Interpretation: text moderation event-tracking began firing **2026-09-04**. `triggered == passed` (27=27) and `rejected` never → in the observed sample **every** moderated listing passed; no rejection has ever been recorded live. (Amplitude query validation itself reported `ai_moderation_rejected` "does not exist in this project" — hard proof it never landed.)

### CloudWatch (`eu-west-1`)
- Log groups incl. `/ecs/tw-staging-back`, `/bedrock/invocations`. Region confirmed **eu-west-1** (others empty). Identity: `arn:aws:iam::438465169079:user/kiro-ai`.
- `/ecs/tw-staging-back` FilterLogEvents (14d): `moderateText`=0, `Rekognition`/`detectModerationLabels`=0, `validatePublicationText`=0, `moderateImageText`=0. **Expected:** these paths log only on ERROR or advisory-flag; silence = no errors and (for text) nothing flagged. `AiPipelineTrackingService wired` startup line=0 in window (last boot predates window / rotated). `SOVEREIGNTY VIOLATION`=24 but all for `prediction_*`/`cron_*` events — **unrelated to moderation**.

### Bedrock provider (`/bedrock/invocations`)
- Stream `aws/bedrock/modelinvocations`, last event ~2026-09-05.
- FilterPattern `"content moderator"` over last 3d → **≥10 invocations** (timestamps ~Sep 4–5). This is the `moderateText` prompt ("You are an expert content moderator for a marketplace app"). **Directly corroborates** the Amplitude Sep-4 firing start: the Bedrock text-moderation model IS being invoked live.

---

## PATH-COVERAGE TABLE (Q3)

| Entry path | Text moderation (`moderateText`, advisory) | Image moderation (Rekognition/Bedrock, blocking) | Sets `Publication.isModerated`? |
|---|---|---|---|
| **Web-v2 create/update** (`x-api-version: v2`) | YES (advisory, fires `ai_moderation_*`) | YES — FE calls `POST /image/validate` per image (`useMediaValidation.ts:79`); upload `verifyChecksum` enforces it | NO (human BO Verify only) |
| **Mobile / legacy create** (no `v2` header) | YES (advisory) — text still runs on create | **NO — BYPASSED.** `shouldModerate=false` → upload skips `verifyChecksum`; no `/image/validate` required | NO |
| **Bulk upload** (`bulk-upload.processor.ts`) | YES (advisory, Step 3) | Depends on image ingest path; not v2-gated image validate → effectively no per-image Rekognition gate in importer | NO |
| **BO admin verify** (`verifyPublication`→`approvePublication`) | n/a | n/a (human review) | **YES — the only path** |
| **Syndication caption gen** | YES (`moderateText` on AI captions; rejected flagged, not posted) | n/a (text only) | n/a |
| **API direct / agent(MCP)** | Text runs if create path hit; image only if client sends v2 + calls `/image/validate` | Same v2-gated bypass as mobile if header absent | NO |

**Mobile-bypass claim (Q3): CONFIRMED.** Image moderation is gated on `x-api-version==='v2'`; mobile/legacy omit it → image content moderation entirely bypassed on those clients. Text moderation is not bypassed but is advisory/non-blocking everywhere.

---

## GAPS + CUTOVER IMPLICATIONS (surfaced, NOT fixed)

1. **Image moderation is v2-only → mobile/legacy uploads are never content-checked for images.** Any client not sending `x-api-version: v2` bypasses Rekognition + Bedrock image-text entirely. Cutover risk: explicit imagery could reach the marketplace via mobile/legacy. **Founder decision** (verified=live gate / cutover): is v2-only acceptable, or must moderation be enforced server-side regardless of client header?
2. **Text moderation is advisory-only (never blocks).** `moderateText` flags → logs a WARN, publish proceeds. Prohibited-text listings are not stopped at publish; only the **human BO Verify** gate (`is_moderated`) governs feed eligibility. So automated moderation currently affects **nothing** that reaches buyers except image hard-blocks on web-v2.
3. **`ai_moderation_rejected` has never fired / does not exist in the lake taxonomy.** Either no listing has ever been text-flagged since 09-04, or the rejected branch is effectively unexercised. The reward-signal/training pair for rejections is currently empty.
4. **Event tracking only started 2026-09-04.** Any "moderation coverage" claim for periods before that is unsupported by the lake. The NAV/feed work that "hinges on is_moderated" hinges on the **human** gate, not any automated pass — confirm that is the intended design.
5. **Image-moderation firing is invisible in Amplitude** (goes to S3 training buffer, not `ta_analytics_event`). Runtime observability of the blocking image path relies on S3 + error logs; there is no first-class `ta_analytics_event` for image pass/reject. Cutover monitoring gap.
6. **Two `isModerated` flags** (`Publication` vs `PublicationImage`) with different meanings and setters — easy to conflate in downstream logic/reporting.

## BLOCKERS / RESIDUAL RISK
- **Direct DB read of `ta_analytics_event` NOT attempted** — DB is in-VPC (would need ECS-Exec / bastion). Not required: Amplitude mirror + Bedrock provider logs jointly proved firing. Marked **UNKNOWN (not attempted, not needed)** for exact sovereign-row counts. If founder wants sovereign-DB row-level confirmation, that is a separate authorized in-VPC read.
- All evidence sanitized: no secrets, tokens, PII, or raw payloads. Only event counts, timestamps, commit SHA, log-group names, and non-sensitive identifiers recorded.

---

## COMPLETION CHECKLIST (1:1)
- [x] api HEAD verified (`8d55a20`); anchors reconfirmed from source
- [x] every moderation call site + trigger condition traced (file:line) — sections A–D
- [x] runtime firing proven (Amplitude counts + Bedrock provider logs + CloudWatch) / DB direct read marked UNKNOWN-not-needed with reason
- [x] path-coverage table (invoke vs bypass)
- [x] mobile-bypass (Q3) CONFIRMED
- [x] automated-vs-human `is_moderated=true` answered (EXCLUSIVELY human BO Verify)
- [x] gaps + cutover implications surfaced (no fix)
- [x] zero changes; evidence file only; own-session status set


---

# ADDENDUM — DIRECT DB ROW EVIDENCE (2026-09-06, read-only one-off task)

**Method:** `ecs RunTask` on `tw-staging-task-back:44` (Fargate, image `ghcr.io/embendev24/tawadoo-api-js:staging-v2`), container command overridden with a Node+`pg` script. Every query ran under `SET TRANSACTION READ ONLY`; output was sanitized JSON (counts/booleans/dates only) streamed to `/ecs/tw-staging-back`. Four tasks total (DBREAD, DBREAD2 schema-discovery, DBREAD3 columns, DBREAD4 distributions), all exit 0. DB user observed: **`tw_runtime_b11`** (the runtime split-credential user, §35). No writes, no DDL, no secrets in output. Tasks are ephemeral (self-stopped).

## D1. Moderation events — SOVEREIGN STORE (`ta_analytics_event`), all-time
| event_type | rows (all-time) | first_seen | last_seen | distinct_users |
|---|---|---|---|---|
| `ai_moderation_triggered` | **27** | 2026-09-04T07:27:16Z | 2026-09-05T19:09:24Z | 7 |
| `ai_moderation_passed` | **27** | 2026-09-04T07:27:16Z | 2026-09-05T19:09:24Z | 7 |
| `ai_moderation_rejected` | **0 (no rows exist)** | — | — | — |

**CONFIRMED at source of truth:** the earlier Amplitude finding is now proven from the sovereign DB. Text moderation events exist ONLY from 2026-09-04, `triggered==passed` (every observed listing passed), `rejected` never written, across 7 distinct users.

## D2. Outbox delivery (sovereignty mirror)
`ta_analytics_delivery` joined to the 30d moderation events: **all 54 rows (27+27) status = `delivered`.** → No sovereignty delivery gap; every moderation event that hit the sovereign store successfully mirrored downstream. (Commandment 2 satisfied for this event family.)

## D3. Publication feed-gate distribution (`ta_publication`, last 90 days)
| is_moderated | is_verified | status | rows |
|---|---|---|---|
| true | true | published | **550** |
| false | false | draft | 290 |
| **true** | **false** | **published** | **71** |
| false | true | published | 42 |
| false | false | published | 37 |
| false | false | archived | 11 |
| false | false | deleted | 8 |
| false | true | deleted | 6 |
| true | true | deleted | 4 |
| (others) | | | ≤1 each |

**All-time `is_moderated=true AND is_verified=false`: 161 rows.**

### ⚠️ SOURCE-vs-DATA DISCREPANCY (must surface, not resolve here)
My source trace concluded `is_moderated=true` is set **exclusively** by `approvePublication()` (human BO Verify), which sets `is_moderated` **and** `is_verified` together in one transaction. The data contradicts a strict reading: **161 rows have `is_moderated=true` while `is_verified=false`** (71 of them currently `published`). The two flags are therefore **NOT** always set together. Possible explanations (NOT verified this session — each is a hypothesis):
- a separate code path sets `is_moderated=true` without `is_verified` (e.g. `autoValidateLegacyMedia`, a migration/backfill, or a media-approval path distinct from publication-approval);
- historical data predating the current `approvePublication` coupling;
- an edit flow that resets `is_verified=false` while leaving `is_moderated=true`.

This means the earlier "EXCLUSIVELY human / perfectly coupled" statement is **too strong**. Corrected claim: **`Publication.is_moderated=true` is *primarily* produced by the human verify path, but the flags are decoupled in real data (161 rows), so some rows became moderated through another path or history.** Determining which requires a follow-up source+audit trace (`ta_publication_audit_log`) — **flagged, not fixed.**

## D4. Per-image flag distribution (`ta_publication_image`)
90 days:
| is_moderated | type | rows |
|---|---|---|
| null | thumb | 2722 |
| **false** | **publication** | **2556** |
| null | map | 886 |
| **true** | **publication** | **92** |
| false | video | 46 |
| true | video | 28 |

Last 14 days: `is_moderated=null` 60, `is_moderated=true` 54, `false` = 0.

**Interpretation (Q3 image bypass — CONFIRMED from rows):** Over 90 days only **92 / (92+2556) ≈ 3.5%** of *publication*-type images carry `is_moderated=true`; **96.5% are `false`** — i.e. uploaded without the v2 moderation stamp (mobile/legacy or pre-v2). Thumbs/maps are `null` (never stamped — expected). In the **last 14 days** the mix flipped: publication images are now either v2-moderated (`true`, 54) or thumb/map (`null`, 60), with **zero new `false`** — consistent with recent traffic coming through the v2 web path while the historical bulk (`false` 2556) predates it. This is real-row confirmation that image moderation coverage is recent and client-dependent, exactly matching the source-level v2 gate.

## D5. Evidence-state upgrade
| Claim | Prior state (Amplitude/source) | Now (direct DB) |
|---|---|---|
| Text moderation events fire only since 09-04 | CONFIRMED (mirror) | **CONFIRMED (sovereign rows)** |
| `ai_moderation_rejected` never fired | CONFIRMED (taxonomy absent) | **CONFIRMED (0 rows)** |
| Events mirror without loss | inferred | **CONFIRMED (54/54 delivered)** |
| Image moderation is v2-only / mostly bypassed | source + FE | **CONFIRMED (3.5% stamped over 90d)** |
| `is_moderated=true` = human-only & coupled to `is_verified` | source-only claim | **PARTIALLY CONFIRMED / CORRECTED — 161 decoupled rows; needs follow-up trace** |

## D6. Remaining UNKNOWN / next-step
- **Which path sets `is_moderated=true` without `is_verified`** (the 161 rows) — UNKNOWN this session. Next step: trace `autoValidateLegacyMedia` + `ta_publication_audit_log` + migration history. Founder-relevant because the NAV/feed work "hinges on is_moderated": if a non-human path can flip it, the feed gate is not purely human-controlled.
- Everything else in this addendum is CONFIRMED from real rows. No blockers remain for the moderation-firing question.

**Own-session status:** FINISHED — COMPLETE (read-only). Zero product/config/provider changes. DB accessed read-only via ephemeral one-off tasks; all stopped. One corrected claim (D3) and one open follow-up (D6) surfaced honestly.
