# SESSION ID S-CTO-20 — MODERATION-360 (READ-ONLY) — EVIDENCE

**Date:** 2026-09-06 · **Status:** FINISHED — COMPLETE (read-only forensics; no code, no HEAD change) · **Author:** Kiro (Principal Engineer diagnostician) · **Founder:** Ramzi
**Skills active:** `tawadoo-source-truth` (activated), `tawadoo-data-sovereignty` (activated), `tawadoo-runtime-provider-qa` (activated).
**Boundary honored:** Writable = NONE (this evidence file only). No commit, no deploy, no moderation action, no prod, no browser write. All findings traced from SOURCE (file:line). Live/DB = best-effort read + sanitized block.

---

## FIVE-LINE OPENER (corrected after preflight)

```
SESSION: S-CTO-20 — MODERATION-360 (READ-ONLY)
BRANCH: tawadoo_api_js @ Ramzi_V2 (50cfa12) · admin_bo_tawadoo @ Ramzi_V2 (07b3a84) — both clean, verified
MISSION: map publish→moderate→index→feed; who sets isModerated vs isVerified; why fresh photoless stays hidden; is staging moderation ON; confirm/refute C1 feed inner-join. NO change.
ORDER: architecture-map (source) → moderation-owner trace → flag set/read table → feed-formatter check → live/DB (best-effort) → verdicts → evidence
TREE STATE: clean on both repos; nothing touched but this evidence file.
```

Preflight git (verified this session):
- `tawadoo_api_js` HEAD `50cfa12814a42438849498011e8c8ee65f28065b` on `Ramzi_V2`, `git status` clean.
- `admin_bo_tawadoo` HEAD `07b3a845bd296675dfb940666bb1343c03fe0a21` on `Ramzi_V2`, `git status` clean.

---

## 1. ARCHITECTURE CHAIN MAP — publish → moderate → index → feed (owners, file:line)

This is the deliverable. Every step, its owner, and the exact source.

```
┌─ SELLER PUBLISHES ──────────────────────────────────────────────────────────────┐
│ OWNER: seller (web/app) → API publication.controller                              │
│ ROUTE: POST publications/publish/:id                                              │
│   publication.controller.ts:487  async publish()                                 │
│   → handlePublicationProcess()  publication.controller.ts:562                     │
│     → publishPipelineService.publish()  publication.controller.ts:650             │
│   DB WRITE (system of record = ta_publication):                                   │
│     publish-pipeline.service.ts:147  status = PublicationStatus.PUBLISHED         │
│     State machine: draft → PENDING (published + isModerated=false + isVerified=false)│
│     moderation-state-machine.ts:141  pending = {published, isModerated:false, isVerified:false}│
│   AUDIT: publish-pipeline.service.ts:155  PublicationAuditLog action='published', actorType='system'│
│   INDEX PUSH (after commit): publish-pipeline.service.ts:212  pushToIndex()       │
│     → but doc is PENDING; whether it is written/removed depends on eligibility     │
│       (publication-search.service.ts pushToIndex gates on index-eligibility)      │
│                                                                                    │
│   ⚠ FRESH PUBLISH ENDS AS "pending": isModerated=false, isVerified=false.          │
└────────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─ MODERATION (the gate) ─────────────────────────────────────────────────────────┐
│ OWNER: BO ADMIN, MANUAL. (there is NO auto-approver / no worker that flips the flag)│
│ TRIGGER PATH:                                                                     │
│   admin_bo_tawadoo → PublicationModerationShow.tsx:142  handleVerify('verifyPublication')│
│   → publication.resource.ts:149  verifyPublication action handler                 │
│   → POST {API_BE_URL}publications/verify/:id  (header x-secret-key)  resource.ts:172│
│   → API publication.controller verify route → publicationService.verifyPublication()│
│   → publication.service.ts:3296 verifyPublication() → :3297 approvePublication()  │
│   → publication.service.ts:3705 approvePublication():                             │
│       :3717 deriveFlag → validateTransition(current,'approved')                   │
│       :3729 publication.isModerated = true                                        │
│       :3731 publication.isVerified  = true   (BOTH set together, in one txn)      │
│       :3745 pushToIndex(publication)  (now eligible → written to OpenSearch)      │
│       :3757 refreshEmbeddingForApprovedPublication (fire-and-forget, k-NN)        │
│   AUDIT: admin_bo publication.resource.ts:~200 INSERT bo_moderation_audit_log 'verify'│
│                                                                                    │
│   SECOND (partial) PATH that can set isModerated=true WITHOUT a BO admin:          │
│   publication.controller.ts:626  updatePublicationIsModerated(id, true)           │
│     — ONLY runs inside `if (enableModeration)` in handlePublicationProcess         │
│     — enableModeration = process.env.ENABLE_CONTENT_MODERATION==='true'            │
│                          && header 'x-api-version'==='v2'   (controller.ts:531)   │
│     — NOTE: this path sets ONLY isModerated=true, NOT isVerified. See §3 caveat.  │
└────────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─ SEARCH INDEX (OpenSearch projection) ──────────────────────────────────────────┐
│ OWNER: PublicationSearchService (API). Projection of ta_publication.              │
│ WRITE ELIGIBILITY (DB→index): index-eligibility.ts:17 INDEX_ELIGIBILITY_WHERE     │
│     = { isModerated:true, status:'published', deletedAt:IsNull() }                │
│   pushToIndex removes an ineligible doc rather than writing it                    │
│   (publication-search.service.ts header comment L61-65).                          │
│ READ FILTER (query time — what public search returns):                            │
│     hybrid-search.service.ts:313  filters = [{status:'published'},{isVerified:true}]│
│     slug-resolver.service.ts:180   must:[{status:'published'},{isVerified:true}]  │
│   ⚠ INDEX WRITE gate = isModerated; SEARCH READ gate = isVerified. Two flags,     │
│     two layers. (Coupled today because approve sets both — see §3.)               │
└────────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─ DISTRIBUTION FEEDS (Google/Meta/TikTok/ChatGPT) ────────────────────────────────┐
│ OWNER: syndication module. Two eligibility surfaces:                              │
│ (a) SELLER-FACING eligibility read (BO/preview):                                  │
│     syndication-eligibility.service.ts:55  leftJoinAndSelect('pub.images')        │
│       :59 isModerated=true :60 status='published' :61 deletedAt IS NULL           │
│       → photoless NOT dropped here (LEFT join). thumbnail='' if no image.          │
│ (b) FEED GENERATION (the actual feed that ships) — feed-generator.service.ts:      │
│     queryActiveCampaignProducts()  ~L560:                                         │
│       .innerJoinAndSelect('cp.publication','pub')                                 │
│       .innerJoinAndSelect('pub.images','images')   ⚠⚠ INNER JOIN ON IMAGES        │
│       .andWhere('pub.is_moderated = true') .andWhere("status='published'")        │
│       .andWhere('pub.deletedAt IS NULL')                                          │
│     → per-channel formatters consume the result:                                  │
│         google-xml.ts / meta-json.ts / tiktok-*.ts / chatgpt-json.ts              │
│     → quality gate: feed-quality-gate.ts applyGates/applyGoogleGate               │
│ NOTE: a listing only reaches feeds if it is BOOSTED or in a SUBSCRIPTION campaign  │
│   (feed-generator query Brackets: campaign.source='subscription'+active sub, OR    │
│    'boost'+boosted+boostExpireAt>NOW). A plain moderated listing with no campaign  │
│   is NOT in any feed by design.                                                    │
└────────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─ PUBLIC READ / UI ───────────────────────────────────────────────────────────────┐
│ Public search results: OpenSearch read filter isVerified:true (above).            │
│ Detail page (public SSR): findBySlugForSSR → isIndexEligible (isModerated+published+notDeleted)│
│     publication.service.ts:851 / :860                                             │
│ Detail page (authenticated / by-id): findBySlug → status===published && isVerified │
│     publication.service.ts:756 / :827                                             │
│   ⚠ Two divergent detail predicates (isModerated-based SSR vs isVerified-based    │
│     authenticated). Equivalent today; drift-risk = A (see §5 X7).                 │
└────────────────────────────────────────────────────────────────────────────────┘
```

**Systems-of-Record vs Projections:**
- System of record: `ta_publication` (Postgres) — owns `status`, `isModerated`, `isVerified`.
- Projections (must not diverge): OpenSearch text index, OpenSearch k-NN embeddings, S3 distribution feeds (google/meta/tiktok/chatgpt XML/JSON), sitemaps.

---

## 2. THE MODERATION-OWNER ANSWER

- **WHO sets `isModerated=true`?** A **BO admin, MANUALLY**, via the AdminJS `verifyPublication` action → `POST publications/verify/:id` (secured by `x-secret-key`) → API `approvePublication()` which sets `isModerated=true` **and** `isVerified=true` in one transaction and pushes to the index. `publication.resource.ts:149`, `publication.service.ts:3705`.
- **Is there an auto-approver / worker / threshold?** **NO.** There is no cron, worker, or scheduled job anywhere that flips `isModerated` to true. Grep across `tawadoo_api_js/src` for every non-test set of `isModerated` returns only: `approvePublication` (BO-triggered), `updatePublicationIsModerated` (called ONLY inside the `enableModeration` publish branch), image/video `isModerated=true` (media-level, different column on `ta_publication_image`), and the reject/edit paths that set it back to `false`.
- **The `enableModeration` publish branch (`controller.ts:626`) IS a form of auto-set** — but it is content-validation (image/duplicate/embedding), and it sets **only `isModerated=true`, NOT `isVerified`**. It runs only when `ENABLE_CONTENT_MODERATION==='true'` AND the request carries header `x-api-version: v2`. See §3 caveat — this is the crux of why "moderation on" doesn't make listings visible.
- **Is staging moderation ENABLED?** Prior sessions read staging ECS task-def and recorded `ENABLE_CONTENT_MODERATION=true` on `tw-staging-svc-back` (SESSION_REPORT_CREATE_PUBLISH_...ROOTCAUSE_360 X6; td:44). I treat that as a hypothesis; I did **not** re-read the live task-def this session (read-only, and the value is not the root cause — see §4). **BLOCKED (best-effort):** exact live env re-read requires ECS describe; recorded as unblock step in §6.

---

## 3. `isModerated` vs `isVerified` — SET / READ TABLE (the crux)

They are **two different DB columns** (`ta_publication.is_moderated` default false `publication.entity.ts:142`; `ta_publication.is_verified` default false `:133`). They gate **different layers**. Today they are *coupled in the primary write path* (approve sets both) but a secondary path can set them apart.

| Flag | WHERE SET (file:line) | WHERE READ (file:line) | Layer it gates |
|---|---|---|---|
| `isModerated` | approve: `publication.service.ts:3729` (→true, +isVerified) · publish auto-mod branch: `controller.ts:626` `updatePublicationIsModerated(id,true)` (→true, **isVerified untouched**) · edit: `publication.service.ts:541` (→false) · reject/unverify: `:3415` (→false) | **INDEX WRITE eligibility** `index-eligibility.ts:17,28` · **FEED gen** `feed-generator.service.ts` inner query `is_moderated=true` · seller feed-eligibility `syndication-eligibility.service.ts:59` · feed-eligibility base `feed-eligibility.query.ts:27` · store-boost `store-boost.service.ts:346` · **public SSR detail** `findBySlugForSSR`→`isIndexEligible` `publication.service.ts:860` | DB→OpenSearch index write; ALL distribution feeds; public SSR detail |
| `isVerified` | approve: `publication.service.ts:3731` (→true) · edit: `:540` (→false) · reject/unverify: `:3416` (→false) · **NOT set by the auto-mod publish branch** | **PUBLIC SEARCH read filter** `hybrid-search.service.ts:313` `{isVerified:true}` · slug-resolver similar-listings `slug-resolver.service.ts:180` · **authenticated detail** `findBySlug` `publication.service.ts:756,827` `status===published && isVerified` · index body maps it `publication-index-body.ts:47` | OpenSearch query-time visibility; authenticated detail page; similar-listings |

**Why this matters (the divergence the map exposes):**
- `isModerated` decides whether a doc is **written into** the index and feeds.
- `isVerified` decides whether the index **returns** the doc to public search, and whether the authenticated detail page shows it.
- The manual BO approve sets **both**, so today they move together and the split is invisible.
- **The auto-moderation publish branch (`ENABLE_CONTENT_MODERATION=true` + v2 header) sets ONLY `isModerated=true`.** A listing moderated that way would be **index-eligible and feed-eligible, but the OpenSearch query filter `isVerified:true` would still hide it from public search results**, and the authenticated detail (`findBySlug`) would 404 it. This is a latent split-flag hazard, not currently the live cause (see §4).

---

## 4. WHY A FRESH PHOTOLESS LISTING STAYS HIDDEN — VERDICT

The root cause is **not** photoless-ness at the moderation gate. It is the manual-moderation gate itself, plus a separate feed inner-join. Two independent facts:

**(A) Visibility in public search / detail — root cause = MANUAL moderation backlog, NOT photos.**
- A fresh publish always lands as `pending` (`isModerated=false, isVerified=false`), regardless of photos (`publish-pipeline.service.ts:147` + state machine `pending`).
- Public search read filter requires `isVerified:true` (`hybrid-search.service.ts:313`); index write requires `isModerated:true` (`index-eligibility.ts:17`). Both are false until a **human BO admin** clicks Verify.
- There is **no auto-approver**. So EVERY fresh listing (photo or photoless) is publicly invisible until an admin moderates it. This is the supply-visibility gate. The observed "fresh photoless is isModerated=false" (S-CTO-18web) is the **normal pending state of any un-moderated listing**, not a photoless-specific bug.
- The `enableModeration` branch does **not** rescue this: even if it set `isModerated=true` at publish, the public search filter is `isVerified` (which that branch does not set), and it does not call `approvePublication`. So auto-mod alone would still leave the listing out of public search.

**(B) Photoless-specific supply LEAK in the FEED path — confirmed at the query, see §5 C1.** Independent of visibility, a truly photoless (image-optional job/service) listing is dropped from ALL distribution feeds by an inner join before it can even receive the in-memory placeholder.

**Bottom line:** fresh photoless stays hidden because (1) nothing auto-moderates it and a human hasn't verified it (applies to all listings), and (2) even after moderation, if it has zero image rows it is inner-joined out of feeds (photoless-specific).

---

## 5. C1 — CONFIRM / REFUTE at the FORMATTER LEVEL

**Brain's prior C1 note:** "eligibility uses `leftJoinAndSelect('pub.images')` (NOT inner) — photoless NOT dropped." **That is TRUE for the seller-facing eligibility read** (`syndication-eligibility.service.ts:55` — LEFT join). **But it is the WRONG surface.** The feed that actually ships is built by a DIFFERENT method.

**C1 — CONFIRMED (supply leak is real), at:**
- `feed-generator.service.ts` → `queryActiveCampaignProducts()` (the method that builds every channel feed):
  - `.innerJoinAndSelect('cp.publication', 'pub')`
  - **`.innerJoinAndSelect('pub.images', 'images')`  ← INNER JOIN ON IMAGES**
- A publication with **zero `ta_publication_image` rows** (a photoless image-optional job/service listing — confirmed such listings insert no image row; only media uploads create rows, `publication.service.ts:3964`) is **eliminated by this inner join** before any product object is built.
- The in-memory placeholder rescue (`feed-generator.service.ts` `PLACEHOLDER_IMAGE_URL`, `if (feedImages.length === 0) feedImages.push(placeholder)`) is therefore **unreachable for a truly photoless row** — the row never survives the join to reach that code. The placeholder only helps a row that HAS at least one image record but no `type==='image'` one (e.g. only a video).
- **Formatter-level corroboration:** even if the row survived, `google-xml.ts:validateProduct` requires `images.length===0 → error 'image_link required'` and `feed-quality-gate.ts:checkProduct` excludes `no_images`; Google gate `applyGoogleGate` Gate 4 excludes `no_real_image` (placeholder doesn't count). So Google would reject a placeholder-only item anyway — but Meta/TikTok/ChatGPT would have accepted the placeholder had the row survived the join. The inner join denies them that chance.

**Net:** C1 is CONFIRMED as a supply leak for photoless image-optional listings, located at the **feed-generator inner join on images**, NOT at the seller eligibility left-join Brain checked. This is a real, silent, photoless-specific feed exclusion (violates Commandment mirror intent + Law §50 "silently fail to Tawadoo-only, never block").

---

## 6. LIVE / DB STATE OF A FRESH PHOTOLESS LISTING — SANITIZED BLOCK + UNBLOCK STEP

**Status: UNKNOWN/BLOCKED (staging DB in-VPC / IP-locked; ECS-Exec required).** Per `tawadoo-runtime-provider-qa` + `tawadoo-data-sovereignty`, I did NOT provision access or open a network path. `aws` and `psql` clients are present locally, but the staging DB is not reachable from this host without the VPC path, and this is a read-only session with no cost/permission authority to change that.

- Prior direct-read evidence (hypothesis, from SESSION_REPORT_CREATE_PUBLISH_...ROOTCAUSE_360 & S-CTO-18web): a seller's fresh published listings read `isModerated=false`, `lightssr`/SSR 404 — consistent with the "pending, un-moderated" state this map predicts. I did not re-verify the live row this session.

**Exact one-step to read it (do NOT auto-run — founder/authorized session only):**
```
# Sanitized: values redacted. Run via a bounded ECS-Exec task on tw-staging-svc-back
# (runtime DB user is read-capable for SELECT), against the staging Postgres:
SELECT id, status, is_moderated, is_verified, deleted_at,
       (SELECT count(*) FROM ta_publication_image i WHERE i.publication_id = p.id) AS image_rows
FROM ta_publication p
WHERE p.id = '<KNOWN_FRESH_PHOTOLESS_PUB_ID>';
# Expected from this map: status='published', is_moderated=false, is_verified=false, image_rows=0.
```
No secrets, connection strings, or PII to be printed; return only the sanitized row above.

---

## 7. THE SUPPLY-VISIBILITY PICTURE (the both-sides funnel — how a listing becomes discoverable)

```
PUBLISH (pending)         →  isModerated=false, isVerified=false
   │
   ├─(gate 1: HUMAN)  BO admin Verify  →  isModerated=true + isVerified=true + pushToIndex
   │        └─ WITHOUT this, listing is invisible in public search (isVerified filter)
   │           and absent from index (isModerated write gate). No auto-approver exists.
   │
   ├─ ON-PLATFORM DISCOVERABILITY (once verified):
   │     public search  ← OpenSearch, filter isVerified:true
   │     public detail  ← SSR isIndexEligible (isModerated) ; auth detail (isVerified)
   │
   └─ OFF-PLATFORM DISCOVERABILITY (feeds) requires ALL of:
         verified (isModerated=true)   AND
         in an active campaign (subscription active  OR  boosted+not expired)  AND
         has ≥1 image ROW  ⚠ (inner join) — photoless job/service dropped here (C1)
         then per-channel quality gates (Google: buy_now+price+real image+known condition; etc.)
```
So a listing is discoverable **on-platform** only after a human moderates it, and **off-platform** only if additionally it is in a paid/boost campaign, has an image row, and passes channel gates. The two most consequential gates are both silent to the seller: (1) the manual moderation wait, (2) the photoless feed inner-join drop.

---

## 8. FINDINGS — RED/YELLOW/BLUE + R0/R1/R2/R3

Legend: RED=critical supply/visibility harm · YELLOW=risk/latent · BLUE=informational/by-design. R0=verify only · R1=small safe fix · R2=bounded build unit · R3=architecture decision (founder).

| # | Sev | Finding | Source (file:line) | Rung | One-line reason |
|---|---|---|---|---|---|
| F1 | 🔴 RED | Photoless (image-optional job/service) listings are dropped from ALL distribution feeds by `innerJoinAndSelect('pub.images')` before the placeholder rescue can run — silent off-platform supply death. | `feed-generator.service.ts` `queryActiveCampaignProducts()` inner join | R2 | Change images join to LEFT for image-optional categories (or gate placeholder pre-join); one bounded feed slice + fail-first test. Violates §50. |
| F2 | 🟡 YELLOW | No auto-moderation of listings: every fresh listing is publicly invisible until a human BO admin clicks Verify. Whole seller base can sit in a moderation backlog, invisible. | no auto-approver in `tawadoo_api_js/src`; `approvePublication` is BO-only | R3 | Founder BUSINESS decision (§52): is manual moderation intended, or should there be an auto-approve/threshold path? Do not build without ruling. |
| F3 | 🟡 YELLOW | Split-flag hazard: the `enableModeration` publish branch sets ONLY `isModerated=true` (not `isVerified`); public search read filter is `isVerified:true`. Auto-moderated listings would be index/feed-eligible yet still hidden from public search + 404 on auth detail. | `controller.ts:626` sets isModerated only; `hybrid-search.service.ts:313` reads isVerified | R2 | Latent: if auto-mod is ever the live path, listings silently miss public search. Reconcile the two flags into one visibility contract. |
| F4 | 🟡 YELLOW | Two divergent detail-page visibility predicates: SSR uses `isIndexEligible` (isModerated), authenticated `findBySlug` uses `isVerified`. Equivalent today (approve sets both) but can drift. | `publication.service.ts:860` (SSR) vs `:756,:827` (auth) | R1/R2 | Consistency audit; converge on one predicate to prevent future drift. |
| F5 | 🔵 BLUE | Feeds require an active campaign (subscription or boost) — a plain moderated listing is intentionally NOT in any feed. | `feed-generator.service.ts` query Brackets (subscription/boost) | R0 | By design; documented here so it is not mis-read as a bug. |
| F6 | 🔵 BLUE | `ENABLE_CONTENT_MODERATION` live value on staging not re-read this session (not the root cause). | prior sessions recorded =true (td:44); not re-verified | R0 | Confirm via ECS describe only if F2/F3 decision needs it; not blocking this map. |
| F7 | 🔵 BLUE | Index WRITE eligibility (isModerated) vs READ filter (isVerified) is a deliberate two-layer projection; correct today only because approve sets both. | `index-eligibility.ts:17` vs `hybrid-search.service.ts:313` | R0 | Informational; the coupling is what F3 could break. |

---

## 9. RECOMMENDED FOLLOW-UP BUILD SESSION(S)

1. **S-CTO-2x BUILD — feed photoless-join fix (F1, R2, 🔴):** convert the feed-generator images inner-join to a left-join for image-optional categories so photoless job/service listings reach the placeholder + non-Google channels. One bounded API slice, fail-first test (photoless job listing → present in ChatGPT/Meta feed with placeholder), rollback = revert one query. *Reason: real, silent off-platform supply death; smallest safe change.*
2. **FOUNDER DECISION (F2, R3, §52) — moderation policy:** Ramzi must rule APPROVE/CHANGE/REJECT on: should Tawadoo keep manual-only moderation, or add an auto-approve/trust path? Everything else (F3) depends on this. *Reason: business/policy, not a technical fact — must not be self-decided.*
3. **S-CTO-2x BUILD — flag-contract convergence (F3+F4, R2, 🟡):** after F2 is decided, reconcile `isModerated` vs `isVerified` into one visibility contract so no path sets one without the other, and unify the two detail predicates. *Reason: latent split-flag hides moderated listings from public search; drift risk.*

---

## ACCEPTANCE CHECK (one-to-one with prompt)

- [x] Full publish→moderate→index→feed chain mapped from source with owners (§1).
- [x] `isModerated` vs `isVerified` set/read points listed (§3).
- [x] Moderation trigger identified: manual BO admin `verifyPublication`→`approvePublication` (§2).
- [x] Staging-moderation on/off: recorded prior evidence + BLOCKED on live re-read (not root cause) (§2, §6/F6).
- [x] C1 confirmed at the formatter/query level with file:line (§5).
- [x] Fresh-listing live state: sanitized block + exact unblock step (§6).
- [x] Supply-visibility both-sides funnel (§7).
- [x] RED/YELLOW/BLUE + R0–R3 per finding (§8); follow-up sessions + one-line reasons (§9).
- [x] No code changed; HEADs unchanged (both repos clean, verified).
