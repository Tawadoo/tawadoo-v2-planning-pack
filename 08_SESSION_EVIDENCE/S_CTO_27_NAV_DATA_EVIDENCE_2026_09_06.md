# S-CTO-27-NAV-DATA — END-OF-SESSION REPORT (for Brain B16+ QA & queueing)

**Session:** S-CTO-27-NAV-DATA · READ-ONLY DATA INVESTIGATION → widened into a moderation/verified genealogy audit at founder request.
**Env:** STAGING (read) + PROD (read-only SELECT only) · **Date:** 2026-09-06
**Repos on:** `Ramzi_V2` (api HEAD `8d55a20`, clean).
**Proposed status:** FINISHED — COMPLETE for the original count question; the audit surfaced OPEN items below that the Brain must queue. Independent Brain QA owes acceptance.
**Skills:** tawadoo-source-truth + tawadoo-data-sovereignty + tawadoo-runtime-provider-qa — discovered, activated, applied.

---

## 0. FOUNDER RULING CAPTURED THIS SESSION (binding, must propagate)

> **`is_verified = true` ⇒ the listing MUST be live (publicly searchable/visible). Verified is the authority. `is_moderated = true` does NOT make a listing live by itself, and must never override a human verify. Both columns are kept — they are two different things: verified = human trust decision; moderated = system content-safety signal.**

This is a NEW founder invariant. It is NOT how the current V2 code behaves (see §4 CONTRADICTION). The Brain must add it to the terminology/architecture law and scope a cutover fix.

---

## 1. EXECUTIVE SUMMARY

1. **Original question (isModerated=null blast radius):** the class `status=published AND isVerified=true AND isModerated IS NULL` = **0** on staging. The whole `is_moderated` column has **0 nulls** anywhere (staging AND prod). S-CTO-26 row-repair is a **no-op — nothing to repair.**
2. **`is_moderated` is NOT a Kiro naming mess.** It is a real content-moderation subsystem, born **2026-05-14** (commit `856a458`, human dev **aladin-sridi**, "Add content moderation"), 14 months *after* `is_verified` (born **2025-03-23**, commit `6f042c9`, embendev24). Two flags, two purposes, two authors, built 14 months apart.
3. **verified vs moderated (the real relationship):** `is_verified` = human trust; `is_moderated` = automated content-safety (AWS Rekognition images/video + Bedrock text-in-image + OpenSearch kNN duplicate detection).
4. **Discrepancy prod vs staging is expected:** V2 (Ramzi_V2 on staging) is the rebuild of the live web/app. Moderation is exercised on the V2 web path only; prod's real traffic (mobile + legacy web) does not send `x-api-version: v2`, so it bypasses the moderation gate — dormant in prod-reality, live-wired on staging.
5. **CONTRADICTION to fix before cutover:** current V2 code gates public SSR render + search index + feeds on **`is_moderated`**, NOT `is_verified` — the exact opposite of the founder ruling in §0. See §4.

---

## 2. GENEALOGY — DATES, AUTHORS, PURPOSE (git + live verified)

| Flag | Born | Commit / Author | Purpose | Set by |
|---|---|---|---|---|
| `is_verified` | 2025-03-23 | `6f042c9` embendev24 "verification phone and email…publication" | Human trust: seller/listing approved | Admin BO "verify" action |
| `is_moderated` | 2026-05-14 | `856a458` aladin-sridi "Add content moderation" | System content-safety: media + text + dedup passed | Automated pipeline at create/publish; + WS-A3 state machine on approve |

- Repo initial commit 2024-08-23. `is_verified` predates moderation by 14 months.
- The 2026-05-14 commit shipped a full subsystem: Rekognition image/video moderation, Bedrock text moderation, kNN dedup, publication-image upload moderation, publish moderation gate, revalidate endpoint — gated behind `ENABLE_CONTENT_MODERATION` + `x-api-version: v2`. It touched `is_verified` only as references (4), did not rename/replace it.
- **WS-A3 unified state machine** (commit `206d579`, 2026-07-25, ramzi/Kiro program) repurposed publication-level `is_moderated` to move in **lockstep** with `is_verified` (approved ⇒ both true). This is the source of the "they look like the same thing" confusion.
- **Documented historic split-brain bug** (MCP repo `ROAD_TO_95_EXECUTION_SUMMARY.md`): "BO verify flipped `is_verified`; API/FE gated on `is_moderated` — two columns, admin approval never made listing live." Fixed by making BO verify set BOTH. This fix is what coupled them.

---

## 3. WHERE MODERATION COMES FROM → WHERE IT GOES (source→sink map)

**Producers (writers of `is_moderated`), file:line in tawadoo_api_js:**
- `publication-image.controller.ts:~122` — on `publication-images/upload`, if `shouldModerate` (=`ENABLE_CONTENT_MODERATION==='true' && x-api-version==='v2'`): runs checksum + limits, stamps media-row `isModerated=true`. Heavy checks in `image.service.validateImage` (Rekognition `detectModerationLabels` + Bedrock `moderateImageText`) and `video.service.validateVideo` (frame-sampled Rekognition). Explicit content ⇒ hard reject (`IMG_EXPLICIT_CONTENT` / `IMG_CONTAINS_EXPLICIT_TEXT` / `VID_EXPLICIT_CONTENT`).
- `publication.service.ts:3547 validateAllMediaModerated` — publish gate: blocks publish if any media row still `isModerated=false`.
- `publication.service.ts:3730 approvePublication` — sets publication `isModerated=true` + `isVerified=true` (atomic).
- `publication.service.ts:3415 rejectPublication` — both false + status=draft.
- `publication.service.ts:542 update()` — both false (edit ⇒ re-moderation).
- `moderation-state-machine.ts:139-146 getColumnUpdates` — canonical pairing: approved={both true}, others={both false}.
- BO `admin_bo_tawadoo/src/resources/publication.resource.ts:197` verify action — API-first, then `record.update({is_verified:true, is_moderated:true})`.

**Consumers (readers — where it GOES):**
- `index-eligibility.ts:27 isIndexEligible` — requires `isModerated===true` (+ published + not deleted). Single choke point for the OpenSearch index projection.
- `findBySlugForSSR` (`publication.service.ts:851`) — SSR public render gate uses `isIndexEligible` ⇒ **requires isModerated=true**.
- `feed-eligibility.query.ts:27 baseConditions` — `isModerated:true` ⇒ feeds, syndication, distribution groups, MCP search eligibility.
- `syndication-eligibility.service.ts`, `store-boost.service.ts:346`, `cockpit.service.ts` (moderation counts).
- **Note:** public search + Classic `findBySlug` gate on `is_verified`, NOT is_moderated.

**Repo spread:** api = 46 files, mcp = 8, bo = 6, web = 3 (web reads the flag **indirectly** — it only sends `x-api-version:v2`; it does not read is_moderated), mobile = 0.

---

## 4. ⚠️ CONTRADICTION AGAINST THE FOUNDER RULING (§0) — HIGH PRIORITY FOR BRAIN

Founder rule: **verified=true ⇒ MUST be live; moderated does not gate visibility.**
Current V2 code does the OPPOSITE on key surfaces:

| Surface | Current gate | Founder-required gate | Verdict |
|---|---|---|---|
| SSR page render (`findBySlugForSSR`) | `is_moderated===true` (ignores is_verified) | `is_verified===true` | **VIOLATES** — a verified+unmoderated listing 404s on SSR |
| Search index eligibility (`isIndexEligible`) | `is_moderated===true` | must not hide a verified listing | **VIOLATES** |
| Feed / syndication / distribution / MCP | `is_moderated===true` | verified live; moderated may be an *extra* quality filter for external channels only | **PARTIAL** — acceptable for external feeds, NOT for on-platform visibility |
| Classic search + `findBySlug` | `is_verified===true` | `is_verified===true` | **CORRECT** |

Today this is masked because WS-A3 couples the flags (approve sets both). The violation becomes live the instant they diverge (verified=true, moderated=false/null). **Cutover blocker:** re-key all on-platform visibility gates to `is_verified`; demote `is_moderated` to a non-blocking signal (external-distribution quality + safety flag) that never overrides verify. Decouple the state machine so human verify makes a listing live regardless of moderation.

---

## 5. BUILT-vs-VERIFIED MATRIX (nothing buried)

**Backend (tawadoo_api_js, Ramzi_V2) — BUILT + reachable:**
- Content-moderation pipeline (Rekognition image/video, Bedrock text-in-image, kNN dedup) — EXISTS, wired, reachable. `ENABLE_CONTENT_MODERATION=true` on staging AND prod task defs.
- Moderation state machine (WS-A3), backfill migration, index-eligibility predicate, feed-eligibility query — all present.

**NOT VERIFIED AT RUNTIME (the honest gaps):**
- **Moderation actually executing on live traffic = UNPROVEN.** Zero moderation-rejection log hits in staging (30d) and prod (30d). Staging has no organic v2 traffic; prod traffic doesn't send v2. No captured Rekognition/Bedrock invocation tied to a real listing. State = "reachable + config-on", NOT "runtime-verified in the wild." A bounded synthetic v2 upload on staging would close this (offered, not yet run).
- **Publication-image moderation freshness query (prod) = NOT COMPLETED** — the read task was aborted by the founder before results. The intended read (are recent prod images being stamped isModerated=true, i.e. is the pipeline running) is still open.

**Frontend (tawadoo_web_js):**
- New V2 create flow (`product-form-v2.tsx`, `productDetails.tsx`) SENDS `x-api-version: v2` on create/update/upload/publish — BUILT + verified in source.
- Web does NOT read `is_moderated` directly — visibility is server-driven. No web change needed for the flag itself.
- **The current live web/app (prod) is NOT the V2 flow** — expected; V2 not cut over.

**Mobile app (tawadoo_mobile_app + tawadoo_app_mobile_ui_only):**
- Sends **NO** `x-api-version` header ⇒ all mobile-created listings take the **legacy path, skipping moderation entirely.** ~7 weeks stale vs api/web. **OPEN:** mobile must adopt v2 (or the API must default appropriately) or moderation will never protect mobile-origin listings post-cutover.

**Database (Aurora PostgreSQL):**
- `ta_publication.is_moderated` — boolean, **NOT NULL, default false** in BOTH prod and staging (0 nulls). `is_verified` same shape.
- Prod: 21,044 published (20,884 verified). Staging: 20,059 published (19,875 verified=true+moderated=true; 158 verified=false+moderated=true; 26 verified=false+moderated=false pending).
- **DISCREPANCY NOTED (open, small):** my FIRST staging read reported `is_nullable: NO` while the prompt's premise assumed nulls could exist; a later read confirmed NOT NULL in both envs. Either the column was already hardened, or the S-CTO-26 premise was stale. Brain should confirm which and close/adjust S-CTO-26.

**BO (admin_bo_tawadoo):**
- verify/unverify/reject actions are API-first and write both flags. `unverifyPublication` sets `is_verified=false` + status=draft but does NOT touch `is_moderated` (asymmetry — low risk today, worth noting for the decoupling work).
- `report-builder.ts` has an audit-vs-state drift detector for verify — good, reusable.

**AWS / infra:**
- ECS Exec on `tw-staging-svc-back`: `enableExecuteCommand=true`, ExecuteCommandAgent self-reports RUNNING, but `execute-command` returns **`TargetNotConnectedException`** on both running tasks. **OPEN INFRA ISSUE** — ECS Exec is effectively broken on staging back; blocks interactive debugging. Brain should queue a fix (SSM agent/task-role/network check).
- RDS Data API disabled on `tw-staging-cluster` (expected).
- DB is in-VPC private (10.0.4.105); no direct psql from workstation (expected).

---

## 6. ERRORS / ANOMALIES MET (even outside scope — nothing buried)

1. **AWS MCP `call_boto3` validation quirks:** operation names must be PascalCase (`GetCallerIdentity`), responses are NOT wrapped in `return_value`, and the static validator cannot see into nested response keys (e.g. `awsvpcConfiguration.subnets`) — had to JSON round-trip to read them. Not a product issue; noted so future sessions don't lose time.
2. **ECS Exec broken on staging** (see §5 AWS) — genuine infra defect, outside this session's scope.
3. **158 staging rows verified=false + moderated=true** — content passed automation but seller not human-verified. Under the founder rule these should NOT be live (verified=false). Confirm they aren't leaking into any surface. Low risk, worth a check.
4. **26 staging rows published + moderated=false + verified=false + active=true** — normal pending-moderation queue, not an anomaly.
5. **Mobile bypass of moderation** (see §5) — the biggest silent gap: most real listing volume never hits the moderation system.
6. **WS-A3 coupling masks the visibility-gate contradiction** — will surface as a real bug the moment flags diverge post-cutover.

---

## 7. OPEN ITEMS FOR THE BRAIN TO QUEUE (future sessions)

- **Q1 (policy → then build):** Re-key all on-platform visibility gates (SSR render, search index) from `is_moderated` to `is_verified` per founder ruling §0; demote `is_moderated` to non-blocking external-distribution/safety signal; decouple the WS-A3 state machine. Cutover blocker.
- **Q2 (verify runtime):** Bounded synthetic v2 upload on staging to prove Rekognition/Bedrock/dedup actually fire and stamp is_moderated — close the "reachable vs runtime-verified" gap.
- **Q3 (mobile):** Decide how mobile-origin listings get moderated (mobile sends v2, or API changes default). Mobile is ~7 weeks stale.
- **Q4 (S-CTO-26):** Close as no-op (0 rows) OR authorize the small preventative NOT-NULL hardening (already NOT NULL per live read — confirm and close). Reconcile the stale null premise.
- **Q5 (infra):** Fix ECS Exec `TargetNotConnected` on tw-staging-svc-back.
- **Q6 (BO asymmetry):** `unverifyPublication` doesn't reset `is_moderated` — align during decoupling.
- **Q7 (complete the aborted read):** prod publication_image moderation-freshness query (was aborted) — confirm whether prod pipeline is stamping any images.

---

## 8. CAPABILITIES / COST / CLEANUP

- **New capability used:** AWS MCP `call_boto3` for bounded READ-ONLY ECS RunTask (measurement path when psql/Exec/DataAPI blocked). Command-override only, no schema/row mutation, tasks self-terminated (exitCode 0).
- **Tasks run:** 4 staging read tasks + 1 prod read task (all STOPPED cleanly) + 1 prod read task aborted by founder before completion (also self-terminates). No orphaned/recurring resources, no new IAM, no new secrets, cost = a few sub-minute Fargate task-seconds on existing infra.
- **Prod:** only pure `SELECT` reads executed. No prod mutation.
- **Writable output:** this evidence file only. No code, no deploy, no migration.
- **Secrets:** none printed; only sanitized aggregates and resource names.

---

## 9. STATUS

**FINISHED — COMPLETE** for the count/genealogy question. **OPEN items Q1–Q7** handed to the Brain (not failures of this read-only session — they are discoveries to queue). Independent Brain QA owes: re-verify count via bounded read, confirm genealogy dates from git, and record the founder ruling §0 + contradiction §4 into the governing law and cutover plan.
