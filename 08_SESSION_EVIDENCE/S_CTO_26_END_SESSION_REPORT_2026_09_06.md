# S-CTO-26 — END SESSION REPORT (for the Brain)

**Session:** S-CTO-26-SUPPLY-FINDABILITY-360
**Date:** 2026-09-06
**Type:** INVESTIGATION-HEAVY + one safe write (parked) + build-spec + a founder-requested naming 360
**Proposed status:** `FINISHED — INCOMPLETE` (execution paused by founder before deploy; independent Brain QA owes acceptance)
**Environment:** staging only. Prod untouched and not read.
**Author:** execution session (Kiro). This is a proposal to verify from source/live, not durable truth.

> Founder paused the session before deploy to demand a naming/flag 360 first (delivered), then requested this report. Nothing was deployed. The SSR fix is written + locally validated but **parked uncommitted**.

---

## 0. EXECUTIVE VERDICT

- The prompt's central hypothesis is **confirmed with one correction**: the SSR page gate diverges from every other public surface. But the divergence in live staging data runs the **opposite direction** from what the prompt assumed.
  - Prompt assumed: `verified=true, moderated=false` → SSR 404 while detail 200. **Live staging rows for this = 0.**
  - Actual live divergence: `moderated=true, verified=false` → **158 rows** that are SSR-visible but excluded from search AND 404 on the authenticated detail endpoint = "visible but not findable" (Law #1 regression), opposite direction.
- The SSR safe fix (SSR gate → `is_verified`) is **written, unit-proven (fail-first + green), typechecked, linted, built** — but **NOT committed, NOT deployed, NOT browser-verified**. Parked at founder request.
- A founder-requested **flag/naming 360** was completed across DB + code + BO and is summarized in §5. It surfaced multiple dormant/redundant flags and a misleading DB name.
- **Deferred (spec only, not built):** image↔unified search convergence. **Not started** in code beyond source mapping — see §6.

---

## 1. WHAT WAS ACTUALLY DONE (built / verified matrix)

| Item | Built? | Verified? | Where |
|---|---|---|---|
| Skills discovery + activation (all 5) | n/a | Yes (activated this session) | source-truth, runtime-provider-qa, data-sovereignty, secure-release, refactor-hygiene |
| Preflight (branches/HEADs/tree) | n/a | Yes | api Ramzi_V2@8d55a207 (=expected 8d55a20), web Ramzi_V2@8b0a815f (=expected) |
| §28.1 deployed-ancestor gate | n/a | Yes | deployed api commit = 8d55a207 = Ramzi_V2 HEAD (ancestor-or-equal). ECS COMPLETED, 2/2 |
| 360 two-pipeline map (hybrid vs image) | n/a | Yes (source, file:line) | §4 |
| Findability truth from real DB rows | n/a | Yes (staging read-only) | §3 counts + representative rows |
| SSR safe fix code | **Yes (local)** | **NOT committed / NOT deployed / NOT browser-verified** | api: index-eligibility.ts + publication.service.ts + new spec |
| Fail-first + regression guard | Yes | Yes (behavioral red→green, 68 tests) | ssr-public-visibility-gate.spec.ts |
| api build / lint / typecheck | n/a | Yes (all pass) | `nest build` ok, `tsc --noEmit` ok, eslint owned files ok |
| Convergence build spec | **No** | No | Deferred — only source mapping done (§6). SPEC WRITE-UP NOT PRODUCED (see §7 gap G1) |
| Deploy safe fix | **No** | No | Parked at founder request |
| Live browser + findability re-proof | **No** | No | Depends on deploy |
| Feed-eligibility before/after parity guard | **No** | No | Depends on deploy |
| Rollback contract exercise | **No** | No | Depends on deploy |
| Evidence manifest file | **No** | No | Superseded by founder's naming-360 request + this report (§7 gap G2) |
| Ramzi_V2 integration of fix | **No** | No | Fix uncommitted |

**Net:** back-end change is coded + locally validated only. **Nothing** on front-end, DB, BO, AWS, feeds, or embeddings was changed. Front-end was never touched (this session's fix is API-only; SSR render is served by the API's `findBySlugForSSR`).

---

## 2. EXACT REPO / TREE STATE AT SESSION END (nothing buried)

**tawadoo_api_js** — branch `Ramzi_V2`, HEAD `8d55a207`, **uncommitted working changes:**
- `M src/modules/publication/index-eligibility.ts` (+30 lines: new `isPubliclyVisibleForSSR` predicate; `isIndexEligible` UNCHANGED)
- `M src/modules/publication/publication.service.ts` (SSR method wired to new predicate; import updated; one stale comment word fixed "unmoderated"→"unverified")
- `?? src/modules/publication/ssr-public-visibility-gate.spec.ts` (new 7-test regression guard)
- Diff: 2 files changed, 41 insertions(+), 9 deletions(-)

**tawadoo_web_js** — branch `Ramzi_V2`, HEAD `8b0a815f`, **UNTOUCHED by me.** The 27 pre-existing dirty artifacts (scto5/11/12/18/r-render playwright configs/reports, tests/e2e-staging/*, semantic-review/, several *_EVIDENCE_*.md, `M yarn.lock`) are NOT mine and were preserved per §9/§28.4.

**Cleanup done:** removed one stray empty `manifest` file I accidentally created during GHCR provenance probing. No other artifacts left.

**AWS:** all one-off read-only probe tasks self-stopped or were stop-requested; verified 0 leftover — only the 7 legitimate service tasks (back×2, front×2, bo, mcp, sgtm) are running. No cost leak.

---

## 3. DB TRUTH CAPTURED (staging, read-only) — DO NOT LOSE

Read via ECS RunTask `tw-staging-task-back:44`, user `tw_runtime_b11` (read-only), `SET TRANSACTION READ ONLY`.

**Concept counts (published + not-deleted = 20,059):**
- verified = 19,875 · moderated = 20,033 · verified_and_moderated = 19,875
- **verified_not_moderated = 0** · **moderated_not_verified = 158** ← the live divergence
- `is_live = true` anywhere (any status) = **0**
- `active = true` + published = 20,059; `active=false` only on the 100 `deleted` rows
- boosted = 3 · store_boosted = 1 of 2,645 stores

**Flag cross-tab (published + not-deleted):**
| is_verified | is_moderated | active | boosted | count |
|---|---|---|---|---|
| ✅ | ✅ | ✅ | – | 19,875 |
| ❌ | ✅ | ✅ | – | 158 |
| ❌ | ❌ | ✅ | – | 23 |
| ❌ | ❌ | ✅ | ✅ | 3 |

**Representative real rows (slugs) for later browser/matrix use:** classic/offer `lisseurs_de_cheveux` (id 7c725392, 3 imgs), photoless classic `s23qa-synth-...offre_emploi_test` (id bfa8710e), bid `auctiontlsxsatr_listing` (id 7ab126d5), store/pro `chauffage_soufflant` (id 55d0d41e, Casablanca, 9 imgs), with-city `test_de_moderation_e2e_1788519383252` (Temara, id 8d955174).

**Staging is mostly synthetic QA data** (`s23qa-synth-`, `test_de_moderation_e2e_`) — consistent with founder's "~50 truly live in staging." Prod today = 18,944 live (founder-stated; NOT independently read).

---

## 4. 360 TWO-PIPELINE MAP (source-verified, file:line)

| Dimension | A — Unified Hybrid (text) | B — Image search |
|---|---|---|
| Entry | `HybridSearchService.search()` hybrid-search.service.ts:112 | `POST /search-by-image` publication.controller.ts:1220 → `searchPublicationsByImage` publication.service.ts:4063 |
| Input model | `HybridSearchQuery` (text-only, NO image field) hybrid-search-result.interface.ts:4-26 | multipart image + filter body |
| Index | keyword `publications_2`; vector `publication-embeddings`; tags search-metadata | `publication-embeddings-v2` (via alias) kNN k=100 min 0.75 |
| Visibility gate | keyword filter `status=published`+`isVerified=true` (~:313 / :1677 etc.) | DB re-fetch `status=PUBLISHED`+`isVerified=true` (:4289) |
| Filters | category, city, square, min/maxPrice, isStore, isNewProduct, announceType, hasVideo, properties, geo, sort | category, min/maxPrice, store, newProduct, announceType, squareslug, cityslug — **NOT** properties/geo/hasVideo |
| Ranking | hybrid weighted merge/boosts | kNN + signalPriorityMerger (different engine) |
| Auth | OPEN (logged-out OK) | `JwtAuthGuard` + `ImageSearchRateLimitGuard` (login + rate-limit) — CORRECT, keep |

Convergence gap = **index + filter-coverage + ranking divergence** (two engines). Both correctly gate reads on `isVerified`. Auth boundary is correct.

---

## 5. THE FLAG / NAMING MESS — 360 (DB · CODE · BO) — founder-requested, complete

**Three independent axes, wrongly all called "live" colloquially:**

| Axis | DB col | Set by | Read by (gate) | Controls |
|---|---|---|---|---|
| A **Publicly visible/findable** | `is_verified` | approve (human) true; edit/reject false | detail, SSR, ALL search (keyword/category/autocomplete/popular/ongoing/image), sitemap | findable on Tawadoo + ChatGPT |
| B **Index/feed eligible** | `is_moderated` | auto-moderation + human approve true; edit/reject false | OpenSearch index, embeddings, duplicate-detection, distribution-group, syndication feeds | eligible to leave to external channels |
| — **Removed/exists** | `status` + `deletedAt` | publish/reject/delete | every predicate | on/off spine |

**Canonical predicates (index-eligibility.ts):** `isPubliclyVisibleForSSR` (verified) NEW this session vs `isIndexEligible`/`INDEX_ELIGIBILITY_WHERE` (moderated) EXISTING. Code has an explicit "do not collapse them" contract. Feed/syndication uses a **parallel** predicate `FeedEligibilityQuery.baseConditions()` = `{isModerated:true, status:published}` (feed-eligibility.query.ts) — and syndication `feed-generator.service.ts:503` gates on `is_moderated`, NOT `is_verified`. So public-visibility ≠ syndication-eligibility is real in code (respects §9).

**How they relate:** `is_verified` and `is_moderated` are independent; they only move together through approve (both true), edit (both false), reject (both false + status=draft). Between events they diverge → the 158-row case.

**BO surface (admin_bo_tawadoo):** `Publication.entity.js` mirrors all columns (same names, same mess). BO already has a **drift detector** in `routes/report-builder.ts` that flags `audit_says_verified_but_state_differs` / `audit_says_unverified_but_state_verified` / `audit_says_rejected_but_status_not_draft` — i.e. the BO team already sees verify/moderate drift. BO also has `PublicationBoost`/`StoreBoost` entities each with their OWN `active` flag (a third meaning of "active").

---

## 6. DEFERRED (SPEC ONLY, NOT BUILT) — image↔unified convergence

- **Not started in code.** Only source mapping done (§4). The target (add image/vector input to `HybridSearchQuery`; route search-by-image through the hybrid vector leg with the same filter pass + verified gate; one result surface; image↔filter both ways; keep image auth + rate-limit; image-index re-key embeddings→verified as a guarded sub-wave WITH feed-parity guard) is understood but **the written spec deliverable was not produced** (see gap G1).
- Multimodal assets exist (Bedrock/Titan 1024-dim on ~210K images) → convergence is mostly ROUTING existing embeddings, not a new ML stack. No new ML component required (so no §23 cost STOP triggered).

---

## 7. OPEN / INCOMPLETE / GAPS — nothing buried

**Prompt deliverables not completed (because founder paused before deploy):**
- **G1 — Convergence build spec write-up NOT produced.** Source mapping done; the ranked-waves + per-wave blast-radius/risk + feed-parity-guard + image-auth-preserved document was not written. OWNER: next session.
- **G2 — Evidence manifest file NOT written** (`S_CTO_26_SUPPLY_FINDABILITY_360_EVIDENCE_...md`). Founder redirected to the naming 360 + this report before I created it. All its content exists in this report + task context; it should be consolidated into the formal evidence file next session.
- **G3 — Deploy + live browser + findability re-proof (desktop/mobile/logged-out) NOT done.** Parked.
- **G4 — Feed-eligibility before/after parity guard NOT exercised.** Depends on deploy.
- **G5 — Rollback contract defined-but-not-exercised.** Depends on deploy. (Contract is trivial: revert the 2 files / 1 spec; no migration, no infra, no data change — pure code predicate swap.)
- **G6 — Fix NOT integrated into Ramzi_V2** (§28.2). Uncommitted.

**Findings surfaced that need FOUNDER DECISIONS (§52.1):**
- **D1 — Deploy the SSR fix?** APPROVE / CHANGE / HOLD. (Aligns SSR to is_verified; feeds/moderation untouched; closes the 158-row visible-but-not-findable case.) Currently HOLD per founder.

**Flag/data issues found (in and out of scope) for the Brain to queue:**
- **I1 — `is_live` is a dormant gate.** Founder's intended meaning ("starts at verify, ends at delete/unpublish/auction-end") is NOT wired. In code `isLive` is written only by the video **live-session** lifecycle (publication-live.service.ts:72/94), indexed into OpenSearch, but **no query/gate reads it**. DB: `is_live=true` on 0 rows. → Either implement the intended lifecycle or formally retire/rename the flag. Separate session. **Naming collision root cause.**
- **I2 — `Publication.active` is redundant/dead.** Only writer is `deactivateAllByEntityIds` (always paired with `status=DELETED`); no independent reader. DB confirms active=false only on deleted rows. → Cleanup candidate; do not wire new logic to it.
- **I3 — Misleading DB name.** The **staging** cluster's database is named `twdbprod`. This is a real confusion trap (a future session could think it's reading prod). → Document loudly; consider rename at a safe window (risky — many env refs).
- **I4 — Three different "active" meanings** (`Publication.active`, `PublicationBoost.active`, `StoreBoost.active`, plus subscription/campaign `active`). Naming hygiene debt.
- **I5 — 158 legacy `moderated_not_verified` rows** exist in staging (feed-eligible + SSR-visible but unfindable). After the SSR fix they stop being SSR-visible; they remain feed-eligible. Founder should confirm whether these 158 SHOULD be feed-eligible at all, or whether they are stale/half-approved rows needing a data check. (Data question, not code.)
- **I6 — `boosted` write path not fully located.** Boost activation writer lives in a PlanBoostCategory/boost module not fully traced this session (only the training-data-log hook + readers confirmed). → Trace before any boost refactor.
- **I7 — `active`-vs-`status` redundancy** may hide a latent bug if any future code sets `status=DELETED` WITHOUT `active=false` (or vice-versa) since they're maintained in lockstep by only one method. → Consider a DB constraint or consolidation.

**Capability/tooling limitations encountered (proven, not guessed):**
- **C1 — GHCR read = HTTP403.** The `gh` CLI PAT (gho_) lacks `read:packages` scope, so ghcr.io manifest reads (digest→sha-commit mapping) fail. Provenance was proven instead via CI-run headSha + ancestor check + running task count. → If precise digest provenance is required in future, a PAT with `read:packages` is needed (founder to provision).
- **C2 — Prod DB not readable.** I only have staging RunTask access. Prod flag distributions were NOT read; prod=18,944 is founder-stated. → A separate authorized prod read is needed for a prod-vs-staging flag parity check.
- **C3 — Correlated image-count subqueries over ~20K rows time out (>45s).** `ta_publication_image.fk_publication_id` join is heavy; possible missing/uneven index. Use bounded id-list joins. → Possible index-health investigation for `ta_publication_image`.
- **C4 — ECS one-off cold start ~60–90s** (PROVISIONING→PENDING→RUNNING); `aws ecs wait tasks-stopped` can exceed a 120s command window. Poll `lastStatus` instead. (Operational note for future DB-probe sessions.)

**Prompt-vs-source deviations (§52 surfaced):**
- **DEV1 — Owner-preview in SSR does NOT exist.** Prompt said "keep owner-preview from source" in `findBySlugForSSR`; source shows it is intentionally absent (owner previews via authenticated `findBySlug`). Fix was simpler than prompt assumed. Resolved from source, not a blocker.
- **DEV2 — Fail-first direction inverted.** Prompt's "verified-but-not-moderated → SSR 404 vs detail 200" has 0 live rows; the real divergence is the opposite 158 rows. Fail-first was proven behaviorally at the predicate level instead of via a live row of the prompt's stated shape.

---

## 8. SAFE FIX DETAIL (for QA to re-verify from source)

- **New predicate** `isPubliclyVisibleForSSR(pub)` in `index-eligibility.ts` = `isVerified===true && status===PUBLISHED && deletedAt==null`. Placed beside `isIndexEligible` (moderated) which is UNCHANGED and still used at publication.service.ts:1139 & :3781 (embeddings) — verified untouched.
- **Wiring:** `findBySlugForSSR` (publication.service.ts:851) `isPubliclyVisible` now delegates to `isPubliclyVisibleForSSR`. Import line updated to `{ isIndexEligible, isPubliclyVisibleForSSR }`.
- **NOT touched:** `isIndexEligible` logic, embeddings (:1139/:3781), keyword ES filters, feeds/syndication, moderation paths, image auth. No forbidden files. api repo only.
- **Fail-first (behavioral):** temporarily pointing the new predicate at moderated logic → 3 divergence tests FAIL (moderated-not-verified wrongly visible; verified-not-moderated wrongly hidden); restored to verified → 7/7 green.
- **Validation:** 5 suites / 68 tests pass (ssr-gate + index-eligibility-gate + moderation-state-machine + approve-publication + moderation-pipeline-e2e); `tsc --noEmit` exit 0; eslint owned files exit 0; `nest build` exit 0.
- **NOT yet:** committed, CI-run, deployed, browser-verified, feed-parity-checked, rollback-exercised, Ramzi_V2-integrated.

---

## 9. RECOMMENDED NEXT SESSIONS (for the Brain to queue — candidates, not authorized)

1. **S-CTO-26 finisher** (if founder APPROVES D1): commit the parked fix → deploy → CI job check → live browser (desktop+mobile+logged-out) on the representative slugs → feed-eligibility before/after parity (prove syndication set unchanged) → rollback contract → integrate Ramzi_V2 → write the formal evidence manifest (G1/G2/G3/G4/G5/G6).
2. **Convergence spec session** (Mode A design): produce the image↔unified convergence build spec + ranked waves (G1) with feed-parity guard + image-auth-preserved. Feed-critical → guarded.
3. **Flag-hygiene investigation** (Mode A): I1 (`is_live` intended vs dormant), I2 (`Publication.active` dead), I3 (`twdbprod` staging name), I4 (three "active" meanings), I6 (`boosted` writer trace), I7 (active/status lockstep). Founder decisions needed on retire-vs-implement per flag.
4. **158-row data check** (I5): are the moderated-not-verified rows legitimate feed-eligible or stale? Data + business decision.
5. **Prod-vs-staging flag parity** (C2): authorized prod read to compare flag distributions (prod 18,944 vs staging ~20K synthetic).

---

## 10. CANONICAL STATUS

- **Proposed:** `FINISHED — INCOMPLETE` — investigation + naming 360 complete; safe fix coded + locally validated but parked uncommitted at founder request; convergence spec + evidence manifest + deploy/live-proof not done.
- **Missing gates:** G1–G6 (§7). **Blocker:** founder decision D1 (deploy?) + founder redirect to naming 360.
- **Owner of next action:** founder (D1 decision) → then a finisher session.
- **Integration state:** fix NOT in Ramzi_V2 (uncommitted). No production impact. No infra/DB/feed/moderation change made anywhere.
- Independent Brain QA owes acceptance: re-verify from source (the 2 diffs + spec), re-run the 68 tests, and confirm the DB counts before trusting this report.
