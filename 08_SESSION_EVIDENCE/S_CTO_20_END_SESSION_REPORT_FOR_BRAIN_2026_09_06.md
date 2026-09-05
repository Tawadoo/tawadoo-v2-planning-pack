# S-CTO-20 — END-OF-SESSION REPORT FOR THE BRAIN

**Session:** S-CTO-20 · MODERATION-360 (READ-ONLY) · **Date:** 2026-09-06
**Type:** MODE A discovery / forensic mapping. **NOTHING WAS BUILT.** No code, no migration, no deploy, no moderation action.
**Repos touched:** none (both `Ramzi_V2`, clean, HEADs unchanged: `tawadoo_api_js` `50cfa12`, `admin_bo_tawadoo` `07b3a84`).
**Deliverables written:** `S_CTO_20_MODERATION_360_EVIDENCE_2026_09_06.md` (the architecture map + verdicts) and this report.
**Proposed status:** FINISHED — COMPLETE for the read-only mission. **Independent Brain QA still required** (§18) — this is a proposal, not accepted truth.
**Skills:** `tawadoo-source-truth`, `tawadoo-data-sovereignty`, `tawadoo-runtime-provider-qa` all activated this session.

> Purpose of this file: hand the Brain everything, buried nothing — including small/ancillary/out-of-scope observations — so it can queue future investigations. Read alongside the evidence file.

---

## A. WHAT THE MISSION ANSWERED (headline truth, one screen)

1. **Moderation is 100% MANUAL.** A BO admin clicks "Verify" → API `approvePublication()` sets `isModerated=true` AND `isVerified=true` in one transaction + pushes to index. **No auto-approver, no worker, no threshold** anywhere in `tawadoo_api_js/src`.
2. **Fresh listings (photo or photoless) are publicly invisible until a human moderates them.** Public search filters on `isVerified:true`; index write gates on `isModerated:true`; both are false at publish.
3. **C1 CONFIRMED — but Brain checked the wrong file.** The safe LEFT join Brain verified is in the *seller-facing eligibility read*. The **feed that actually ships** (`feed-generator.service.ts` `queryActiveCampaignProducts`) uses **`innerJoinAndSelect('pub.images')`** → photoless job/service listings are silently dropped from ALL feeds before the in-memory placeholder can rescue them. Real supply leak; violates §50.
4. **`isModerated` and `isVerified` are TWO columns gating TWO layers** (index-write vs search-read/detail). Coupled today only because manual approve sets both. The auto-moderation publish branch sets ONLY `isModerated` → latent split-flag hazard.

---

## B. BUILT vs NOT-BUILT / NOT-VERIFIED — by layer (nothing buried)

This session built nothing. The table below is **what the codebase currently contains and its highest proven evidence state**, per layer, so the Brain can see exactly where truth is source-only vs runtime-verified.

| Layer | What EXISTS in source (verified this session, file:line) | NOT built / NOT verified / gap |
|---|---|---|
| **API (back)** | Full moderation state machine (`moderation-state-machine.ts`); manual approve path (`publication.service.ts:3705`); publish→pending pipeline (`publish-pipeline.service.ts:147`); index-eligibility predicate (`index-eligibility.ts`); feed generator + quality gates + 5 channel formatters (`syndication/services/*`). | **Feed images inner-join drops photoless (F1)** — a real bug, NOT fixed (read-only). **No auto-moderation path** (F2) — by absence. **Auto-mod branch sets only isModerated, not isVerified** (F3) — latent, not fixed. |
| **Search (OpenSearch)** | Index write gate = `isModerated` (`index-eligibility.ts:17`); read filter = `isVerified` (`hybrid-search.service.ts:313`, `slug-resolver.service.ts:180`); index body maps `isVerified` (`publication-index-body.ts:47`). | **Not queried live** this session (traced from source only). Whether the running index actually excludes pending docs = source-verified, NOT runtime-verified. |
| **DB (Postgres, system of record)** | `ta_publication.is_moderated` default false (`publication.entity.ts:142`), `is_verified` default false (`:133`); partial index `idx_publication_eligible WHERE is_moderated AND published` (`:39`). Photoless image-optional listings insert ZERO `ta_publication_image` rows (only media upload creates rows, `publication.service.ts:3964`). | **Live row of a known fresh photoless listing NOT read** — staging DB in-VPC/IP-locked, ECS-Exec required. BLOCKED. Exact one-step SELECT provided in evidence §6. Prior sessions read `isModerated=false` for fresh listings (hypothesis, not re-verified this session). |
| **BO (admin_bo_tawadoo)** | `verifyPublication`/`unverifyPublication` actions (`publication.resource.ts:149/:220`) → `POST publications/verify|unverify/:id` with `x-secret-key`; `PublicationModerationShow.tsx` UI; moderation audit log INSERT into `bo_moderation_audit_log`. | **BO moderation QUEUE not opened/observed** (read-only browser was allowed but I performed no BO browsing — S-CTO-21web owns the browser sweep; I avoided collision). So the actual backlog size / whether admins are keeping up = UNKNOWN. `RejectPublicationComponent`/`VerifyPublicationComponent` registered — prior session (B12B-S74) flagged them as "registered but never used"; NOT re-checked here. |
| **AWS / infra** | — | `ENABLE_CONTENT_MODERATION` live value on staging **NOT re-read** this session (prior: =true on td:44). Not the root cause, so not chased. |
| **Front (tawadoo_web_js)** | — | **Entirely out of scope this session; not read.** Whether the web surfaces any "pending moderation" state to the seller, or silently shows nothing, is UNKNOWN and worth a future look (sellers may not know their listing is waiting on a human). |
| **Feeds (S3 projections)** | google-xml / meta-json / tiktok-json / tiktok-csv / chatgpt-json formatters + quality gates all present. | **No live feed inspection** (S3 feed contents not read). The photoless-drop (F1) is proven at the query/source level, NOT by reading a live feed. |

---

## C. FLAGGED FINDINGS — everything, including small & out-of-scope

RED=critical · YELLOW=latent/risk · BLUE=info/by-design. Rung R0 verify / R1 small fix / R2 bounded build / R3 founder decision.

### In-scope (moderation → feed chain)
- **F1 🔴 R2 — Photoless feed drop (CONFIRMED).** `feed-generator.service.ts` `queryActiveCampaignProducts` `.innerJoinAndSelect('pub.images')` eliminates zero-image listings before the placeholder. Non-Google channels (Meta/TikTok/ChatGPT) would have accepted a placeholder. Silent off-platform supply death for jobs/services. **Fix = bounded API slice** (LEFT join for image-optional categories, or build the product object then apply placeholder pre-join). Needs fail-first test + rollback.
- **F2 🟡 R3 — No auto-moderation (founder decision, §52).** Every listing waits on a human. Is this intended policy or a throughput bug? **Must not be self-decided.** Everything in F3 depends on the answer.
- **F3 🟡 R2 — Split-flag hazard.** `enableModeration` publish branch (`controller.ts:626`) sets ONLY `isModerated=true`, never `isVerified`. If auto-mod ever becomes the live path, listings would be index/feed-eligible yet still hidden from public search (`isVerified:true` filter) and 404 on authenticated detail (`findBySlug`). Reconcile into one visibility contract.
- **F4 🟡 R1/R2 — Divergent detail predicates.** Public SSR `findBySlugForSSR`→`isIndexEligible` (isModerated); authenticated `findBySlug` (isVerified) (`publication.service.ts:860` vs `:756/:827`). Equivalent today; drift risk. Converge.

### Small / ancillary / out-of-scope (do NOT bury — for future queues)
- **X1 🔵 R0 — Feeds require an active campaign.** A plain moderated listing is NOT in any feed unless boosted or in a subscription campaign (`feed-generator` query Brackets). By design; documenting so it isn't mis-read as a bug.
- **X2 🟡 R2 — Index has a reconciliation/recovery path; FEEDS may not.** The index has `IndexRecoveryService` + `isIndexEligible` reconciliation (referenced `publication.service.ts:19`, AI-LISTING-FIX-008). I did **not** find an equivalent reconciliation that heals the feed/S3 projection against DB truth. If a feed job silently drops items (e.g. F1, or the URL HEAD-check exclusion below), there may be no self-healing. **Worth a dedicated check.**
- **X3 🟡 R2 — Feed URL validation can silently drop live listings.** `feed-generator.service.ts` `validateProductUrls` does a HEAD request per product with a 5s timeout and **excludes any non-200** (and the code's own TODO admits this is blocking + un-cached). A transient origin hiccup or slow SSR → listing silently absent from that feed run. No metric/alarm on the exclusion count observed. Fragile at scale.
- **X4 🔵 R0 — `getFreshness()` in google-xml always returns `'established'`** (`custom_label_3`), because `FeedGenerationProduct` has no `createdAt`. Cosmetic/reporting only; noted so Google custom-label segmentation isn't trusted as real freshness.
- **X5 🟡 R0/R1 — `duplicate-detection.service.ts` re-validates candidates against Postgres using the SAME `isModerated=true AND published AND notDeleted` predicate** (`:129-134` comment). Good (prevents a pending listing from blocking a new publish) — but it means an UN-moderated duplicate does NOT block; verify that's intended and that the predicate truly matches `index-eligibility` (it's a hand-written SQL predicate, not the shared constant → drift risk).
- **X6 🟡 R2 — pushToIndex failure handling** is a known prior concern (AI-LISTING-FIX-004/008 in the repo docs): index-push failures were historically swallowed; a fix added `recordIndexFailure` + reconciliation. I did NOT re-verify the CURRENT runtime behavior of `pushToIndex` on failure this session (source only). If the Brain wants certainty that a moderated listing always reaches the index, this needs a runtime probe.
- **X7 🔵 — Media-level `isModerated` on `ta_publication_image`** is a SEPARATE column (`publication-image.entity.ts:35`, nullable) from the publication flag. Video/image rows get their own `isModerated`. Not a bug; flagging because the name collision (`isModerated` on two entities) is a readability/drift trap for future sessions.
- **X8 🔵 — `x-secret-key` guards the verify/unverify endpoints** (`publication.resource.ts:172`). I did not verify the secret is set/rotated on staging, nor that the endpoint rejects a missing/invalid key at runtime. Out of scope; note for a security pass.

---

## D. ISSUES / ERRORS I MET (including tooling & environment, outside scope)

- **Live DB unreachable (BLOCKED, expected):** staging Postgres is in-VPC / IP-locked; `psql` and `aws` CLIs are present locally but no network path without VPC. Did not provision (read-only, no cost/permission authority). Unblock step recorded in evidence §6. This is the one acceptance item that stayed at UNKNOWN/BLOCKED.
- **No runtime/browser verification performed:** by design this session traced from source; the OpenSearch index, the live feeds (S3), and the BO moderation queue were NOT observed live. So all §B "Search/Feeds/BO/DB-live" truths are **source-verified, not runtime-verified**. A follow-up runtime-QA session is warranted for the F1 fix and to confirm the pending→invisible behavior end-to-end.
- **Prior-report reliance:** where I cite prior sessions (S-CTO-18web, ROOTCAUSE_360, AI-LISTING-FIX docs) I treated them as hypotheses and re-derived from source; but the `ENABLE_CONTENT_MODERATION=true` staging value and the "fresh listing reads isModerated=false" live facts are inherited, NOT re-verified this session. Flagged so the Brain doesn't over-trust them.
- **No errors in my own tooling** (grep/read/git all succeeded). No code was run that could fail.

---

## E. RECOMMENDED FUTURE SESSIONS (for the queue)

| Priority | Session | Type | One-line reason |
|---|---|---|---|
| 1 | **Founder decision on moderation policy (F2)** | R3 / §52 STOP | Manual-only vs auto-approve — blocks F1/F3 design; business decision, not technical. |
| 2 | **BUILD: feed photoless-join fix (F1)** | R2 API | Real silent off-platform supply death for jobs/services; smallest safe LEFT-join change + fail-first test. |
| 3 | **BUILD: flag-contract convergence (F3+F4)** | R2 API | After F2 decided — one visibility contract so no path sets isModerated without isVerified; unify detail predicates. |
| 4 | **QA: runtime moderation→visibility E2E** | R0/R1 runtime | Confirm live: pending listing absent from OpenSearch + feeds; approve → appears. Not runtime-verified yet. |
| 5 | **INVESTIGATE: feed reconciliation gap (X2/X3)** | R1/R2 | Index self-heals; feeds may not, and URL-HEAD exclusion can silently drop live listings with no alarm. |
| 6 | **INVESTIGATE: front seller-side "pending moderation" UX** | R0 web | Does the seller know their listing is waiting on a human? Front not read this session. |
| 7 | **QA: BO moderation queue backlog + unused components (B12B-S74 carryover)** | R0 BO | How big is the human backlog; are Verify/Reject components actually wired in the running BO. |

---

## F. INTEGRATION / CONTINUITY (§28, §20)

- No commits produced → nothing to integrate into `Ramzi_V2`. HEADs unchanged on both repos, verified twice this session.
- Evidence file `S_CTO_20_MODERATION_360_EVIDENCE_2026_09_06.md` is the durable architecture map; this file is the Brain handoff. Both under `/Users/ramzihannachi/Code`.
- Re-entry for the next session: start from F2 (founder decision) — do not build F1/F3 before it. F1 is independently buildable if the founder wants the supply leak stopped regardless of the moderation-policy decision.

**Nothing above is buried. Every gap, small note, and out-of-scope observation is in §B, §C, and §D.**
