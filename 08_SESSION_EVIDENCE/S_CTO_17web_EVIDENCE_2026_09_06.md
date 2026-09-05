# SESSION ID S-CTO-17web — CATEGORY-ICON FALLBACK + EMITTER RENAMES (web) — END-SESSION REPORT

**Date:** 2026-09-06 · **Repo (writable):** `tawadoo_web_js` only · **Branch:** `Ramzi_V2`
**Skills applied:** tawadoo-source-truth · tawadoo-runtime-provider-qa
**Mode:** MODE A (discovery/design + founder-decision surfacing). **No code written, no build, no deploy, no browser verification this session.**
**Proposed status:** **FINISHED — INCOMPLETE (BLOCKED on founder decision + cross-repo dependency).**
Independent Brain QA to re-verify per §18. This report is a hypothesis to verify from source, not authority.

---

## FIVE-LINE OPENER (as run)

```
SESSION: S-CTO-17web — CATEGORY-ICON FALLBACK + EMITTER RENAMES (web)
BRANCH: Ramzi_V2 (verified; upstream origin/Ramzi_V2; HEAD == Ramzi_V2 @ 7010ea6d)
MISSION: photoless job/service → show category icon (from carousel) instead of broken placeholder; rename web emitters to canonical S-CTO-15 names.
ORDER: reality check → wire fallback → rename emitters → build/test → deploy → browser-verify → evidence
TREE STATE: clean tracked tree except ` M yarn.lock`; many untracked prior-session artifacts (playwright-report-*, tests/e2e-staging/*, prior evidence .md). Nothing of mine committed.
```

---

## 0. EXECUTIVE VERDICT

This session completed the **reality check and authoritative research** for both jobs, then **STOPPED before implementation** because 2 of the 3 emitter renames are blocked by a cross-repo dependency and a payload-contract ambiguity that are founder/allowlist-owner decisions (§48 name-collision hard gate, §52.1 founder-decision STOP, §11 repo isolation). The founder asked for deep research (done, §6/§52.2), then asked for this end-session report instead of authorizing the build. **Therefore nothing was implemented, built, deployed, or browser-verified.** Job 1 is fully scoped and safe to build; Job 2a is safe to do; Job 2b and Job 2c are blocked.

Evidence ladder reached this session: **documented → source** (source truth established for every touchpoint). NOT reached: local / CI / deployed / live. No runtime or browser proof exists — do not treat any part as runtime-true.

---

## 1. WHAT WAS ACTUALLY DONE (source-verified findings only)

### 1.1 Git / branch preflight (§28)
- `tawadoo_web_js` on `Ramzi_V2`, upstream `origin/Ramzi_V2`, HEAD == `Ramzi_V2` == `7010ea6d96814eb8c80c6c5b73b363de16da4f06`.
- Working tree: `M yarn.lock` (pre-existing, not mine) + a large set of **untracked** prior-session artifacts (see §6). No commits made this session.
- **NOT DONE:** deployed-commit → `Ramzi_V2` ancestry check (§28.1 step 3). Deferred because no build/deploy occurred; must be done by the implementing session before it finishes.

### 1.2 Category-icon source (the reuse target) — CONFIRMED
- Carousel maps category → icon via `getIcon(category.icon)` at `src/components/home/CategoriesCarouselClient.tsx:107`, importing from `src/utils/icones.ts`.
- `getIcon(iconName)` (`src/utils/icones.ts` ~line 215) resolves `/assets/icones/images/<name>.png` (13 top-level slugs) or `/assets/icones/<name>.(webp|png)`, else falls back to `DEFAULT_ICON = '/assets/icones/default-icon.png'`. **It never 404s.** This is the exact existing source to reuse — no new asset/icon set needed (satisfies the NAME-COLLISION hard gate).
- `getOnlyIcon` is a sibling that skips the `images/` variant.

### 1.3 The broken-placeholder spot(s) — CONFIRMED via source + sub-agent trace
- **Seller "Mes ventes" (the confirmed 404):** `src/components/dashboard/ListingRow.tsx:35` — `const thumb = product.images?.[0]?.url || product.defaultImage || "/assets/images/placeholder.png";` rendered by a raw `<img src={thumb}>` (~line 168). This is the **only** use of `/assets/images/placeholder.png` in `src/`. `placeholder.png` reportedly 404s (per prompt; NOT re-verified in a browser this session — see OPEN-1).
- **Buyer search card:** `src/components/product/productCard.tsx` — `resolveInitialThumbnail()` (~160-176) falls back to `DEFAULT_PLACEHOLDER = "/assets/images/storecover.png"` (a valid image, **not** the 404). `getIcon` is already imported here but unused for fallback. `product.category.icon` is typed and available (`interface ItemAnnounce`, ~79-84).
- **Detail media column:** `src/components/product/productDetails.tsx:263` builds `images`; the media block (~888) branches `images.length === 1 ? single : <Swiper>` with **no `images.length === 0` guard** → a photoless listing renders an **empty Swiper / empty `bg-sand-100` box** (a visual gap, not a 404). `itemAnnounce.category.icon` is typed/available (~129-135). A separate share/buy-now modal thumbnail at ~1551 already falls back to `storecover.png`.

### 1.4 Data shape — CONFIRMED (`category.icon` available on all three surfaces)
- `publications/mine` (seller list) returns the OpenSearch `_source`; the indexed doc includes `category: { id, code, icon, coinBidNeed }` (built in API `publication-index-body.ts` ~85-92; mapping in `elasticsearch-init.service.ts` ~58-64). So `product.category.icon` is present on the seller row (currently unused).
- Buyer card + detail both type `category.icon: string`. **No API change required** for Job 1.

### 1.5 Job/service gating constant — CONFIRMED
- `src/constants/leadGenCategories.ts` exports `JOBS_OR_SERVICES_CATEGORY_IDS` (union of `SERVICE_CATEGORY_IDS` {84,132–143} and `JOB_CATEGORY_IDS` {85,144,145}). This is the existing, verified way to gate the fallback exactly to job/service listings (matches the founder framing and avoids showing a category icon in place of a missing *physical-good* photo). `LEAD_GEN_CATEGORY_IDS` is already imported in `productCard.tsx`.

### 1.6 The three emitter call-sites — CONFIRMED from source + cross-checked against S-CTO-15 + live API allowlist
Read the **authoritative** canonical source: `tawadoo_api_js/src/modules/analytics-ingestion/constants/allowed-events.ts` (read-only) and `S_CTO_15_ALLOWLIST_FIX_EVIDENCE_2026_09_05.md` §1D "RENAME family".

| # | Old (web emitter) | Site | Canonical target | Allowlist has target? | Verdict |
|---|---|---|---|---|---|
| 2a | `feed_slide_change` | `src/components/feed/ImmersiveFeed.tsx:374` (only occurrence in repo) | `feed_slide_changed` | **YES** (line 277) | **SAFE — do now** |
| 2b | `app_banner_dismissed` | `src/components/OpenInAppBanner.tsx:231` | `app_install_banner_dismissed` (S-CTO-15 O10) | **NO — absent from allowlist** | **BLOCKED** |
| 2c | `ai_generated` | `src/sections/dashboard/createProduct/product-form-v2.tsx:678` | `ai_generation_completed` | canonical exists (389) but… | **BLOCKED / likely no-op** |

Key source facts:
- **2a:** grep for `feed_slide_change\b` returns exactly one hit (`ImmersiveFeed.tsx:374`). Note `productCarousel.tsx:47` already emits the correct `feed_slide_changed`. Clean single-site rename, canonical already declared. No dashboard/test depends on the old string (grep across `**/*.{ts,tsx,json}` = single hit).
- **2b:** `app_install_banner_dismissed` is **not** in the live API allowlist (grep confirmed). S-CTO-15 §1D explicitly said this needs the canonical **added to the allowlist first** ("ADD target canonical + rename emitter → S-CTO-16"). This session is `tawadoo_web_js`-only (API read-only, §11). Renaming the web emitter now → the event lands `_is_canonical=false` (the exact drift S-CTO-15 built surfacing to catch). **Cross-repo ordering dependency.**
- **2c:** In web, `ai_generated` at `product-form-v2.tsx:678` is a **property key** inside the `listing_creation_completed` event payload — **not** a standalone `track('ai_generated', …)` event. Grep `track\w*\(\s*['"]ai_generated['"]` = **zero** hits. S-CTO-15 §1D independently concluded `ai_generated` is "only a DB column, not an event." Renaming a property to an event name (`ai_generation_completed`) would corrupt the `listing_creation_completed` payload contract, not fix a name. **There is no web event to rename.**

### 1.7 Authoritative research (done per §6 / §52.2) — cited
- **Photoless-image UX:** never leave a broken image ([Cloudinary](https://cloudinary.com/guides/ecosystems/shopify-broken-images-fix)); use an **icon** (not a photographic placeholder) so it clearly reads as a placeholder, and prefer a **per-category** icon ([UX StackExchange 45324](https://ux.stackexchange.com/questions/45324/placeholders-for-missing-images), [UX SE 42262](https://ux.stackexchange.com/questions/42262/what-to-display-when-there-is-no-admin-submitted-image-for-categories-and-produc)); for physical goods avoid anything mistakable for the real product ([UX SE 114158](https://ux.stackexchange.com/questions/114158/archetype-photos-vs-placeholder)) — mitigated by gating to job/service. Baymard 2026 confirms resolvable image UX defects drive abandonment ([Baymard](https://baymard.com/blog/current-state-ecommerce-product-page-ux)). → **Founder's category-icon decision is best-practice-aligned.**
- **Event renames:** renaming without lifecycle is an incomplete fix — historical data keeps the old name, destination mappings may split old/new, dashboards break ([Trackingplan](https://www.trackingplan.com/blog/event-taxonomy), [OneUptime](https://oneuptime.com/blog/post/2026-01-30-how-to-build-event-naming-conventions/view)); the canonical name should exist in the controlled vocabulary **before** the emitter points at it ([Brainforge](https://brainforge.ai/resources/event-taxonomy-best-practices/)). → **Confirms defer 2b until the allowlist owns the name; confirms 2a is the safe kind.**
_Content rephrased for compliance with licensing restrictions._

---

## 2. WHAT WAS NOT DONE (nothing buried)

**Job 1 — category-icon fallback: DESIGNED, NOT BUILT.**
- No edit made to `ListingRow.tsx`, `productCard.tsx`, `productDetails.tsx`, or any shared helper.
- Not built on the FRONT (seller row / buyer card / detail media). Not built anywhere.
- Not build-verified, not lint/typecheck-verified, not deployed, **not browser-verified** (Chromium/WebKit/mobile/RTL) — none of it happened.

**Job 2 — emitter renames: 1 designed-not-built, 2 blocked.**
- 2a (`feed_slide_change`→`feed_slide_changed`): safe but **NOT applied** (no code written this session).
- 2b (`app_banner_dismissed`): **BLOCKED** — canonical `app_install_banner_dismissed` absent from API allowlist; needs an API-writable session to add it first (cross-repo).
- 2c (`ai_generated`): **BLOCKED / no-op** — not a web event; renaming would corrupt a payload contract; needs founder clarification of intent.

**Verification chain — NONE performed this session:** no `yarn build`, no `yarn test`, no lint, no tsc, no CI run, no deploy, no digest check, no live/lake/Amplitude emitter-landing proof, no screenshots. Do not infer any runtime truth from this report.

---

## 3. OPEN / INCOMPLETE / FLAGGED ITEMS (for Brain to queue)

### BLOCKERS (must resolve before the emitter work can complete)
- **BLK-1 (founder + cross-repo):** `app_install_banner_dismissed` is not in the API allowlist. Correct order (per S-CTO-15 + research): an **API-writable** session adds the canonical name, *then* a web session renames `OpenInAppBanner.tsx:231`. Founder to confirm the canonical string (S-CTO-15 proposed `app_install_banner_dismissed`) and authorize the API allowlist add. Owner: API allowlist session (S-CTO-16-api or equivalent) → then web.
- **BLK-2 (founder clarification):** `ai_generated` is a property, not a web event. Decide: (a) skip (recommended — nothing to rename); or (b) the real event `ai_generated_multilingual` (`basic-info-section.tsx:240`) should be retired/renamed toward `ai_generation_completed` — a payload/event-contract change whose canonical handling belongs with the allowlist owner. Founder/Brain call.

### OPEN ITEMS — in scope, not yet proven
- **OPEN-1:** The `placeholder.png` 404 is asserted by the prompt but **not browser-verified** this session. Confirm the asset truly 404s (it's referenced only at `ListingRow.tsx:35`; check whether `public/assets/images/placeholder.png` exists on disk / in the deployed build). If the file simply doesn't exist, that's the root cause.
- **OPEN-2:** Job 1 build + 3-surface wiring + build/test/lint/tsc + deploy + browser proof (Chromium/WebKit/mobile/AR-RTL) + no-regression on photo listings — all still to do.
- **OPEN-3:** 2a rename + confirm canonical landing (lake/Amplitude or network payload) — still to do.
- **OPEN-4:** §28.1 deployed-commit ancestry check for `tawadoo_web_js` before the implementing session claims COMPLETE.

### OUT-OF-SCOPE ISSUES NOTICED (do not fix here — Brain to triage/queue)
- **OOS-1 (detail media, all listings):** `productDetails.tsx` has **no `images.length === 0` guard** (~888). Any photoless listing (not just job/service) renders an **empty Swiper / empty box**. Broader than the seller-thumbnail 404; worth a dedicated fix + regression guard (§33). Classify B (regression/UX gap).
- **OOS-2 (inconsistent placeholders):** three different fallbacks exist for "no image" — `placeholder.png` (ListingRow, 404), `storecover.png` (productCard + productDetails modal), and empty-Swiper (detail media). No shared "thumbnail-with-category-icon-fallback" helper. Candidate for one consolidated resolver reusing `getIcon` (refactor-hygiene, only after behavior is proven — §D/refactor rules). Classify A/D.
- **OOS-3 (S-CTO-15 residue):** S-CTO-15 §1D listed `session_start`/`session_end`/`feed_enter`/`feed_exit`/`login` as "no literal string emitter in current web src — likely var-emitted, note for S-CTO-16." Not investigated this session. Also `smart_view_no_results_recovery` (declared-not-wired) still needs a wire-or-prune decision. Queue for the S-CTO-16 web/api pass.
- **OOS-4 (founder B-list from S-CTO-15 still pending):** B1 `store_search` / B2 `store_categories_updated` (DISCUSS) and B3–B5 smart_view_* (REJECT-as-redundant) await the founder's APPROVE/REJECT. Not this session's scope but unresolved and easy to bury.
- **OOS-5 (test-fixture debt from S-CTO-15):** `test_event` prune is deferred because `analytics-ingestion.service.spec.ts:235` uses it as a fixture; migrate the fixture then prune. API-side.

### ERRORS / TOOL NOTES ENCOUNTERED
- **ERR-1 (self, corrected):** my first batch of `read_file` calls used paths relative to the workspace root (`/Users/ramzihannachi/Code/src/...`) instead of the web repo (`/Users/ramzihannachi/Code/tawadoo_web_js/src/...`) → all failed "Path does not exist." Recovered via a context-gatherer sub-agent + corrected absolute paths. No impact on findings, but noting for hygiene: the writable repo is a **subfolder**, not the workspace root.
- **ERR-2 (env):** `web_fetch` returned HTTP 403 (ux.stackexchange) and an empty body (trackingplan). Mitigated: the search-result snippets are verbatim page chunks and were sufficient to cite; no conclusion rests on a blocked fetch.
- **No AWS/DB/BO/MCP access was attempted this session** (pure source investigation) — so no AccessDenied/blocker evidence exists for those surfaces. The implementing session must attempt the staging deploy + lake/Amplitude read for emitter-landing proof.

---

## 4. STATE BY SURFACE (built vs not, verified vs not) — nothing buried

| Surface | This session | Built? | Verified? |
|---|---|---|---|
| **Web front** (ListingRow / productCard / productDetails / ImmersiveFeed / OpenInAppBanner / product-form-v2) | source read only | **No code changed** | Not built → nothing to verify |
| **API back** (`tawadoo_api_js` allowlist) | read-only reference | N/A (read-only this session) | Confirmed `feed_slide_changed`, `ai_generation_completed` present; `app_install_banner_dismissed` **absent** |
| **DB** | not touched | N/A | `category.icon` confirmed indexed in OpenSearch `_source` (from source, not a live query) |
| **BO** (`admin_bo_tawadoo`) | not touched | N/A | Not in scope; not inspected |
| **AWS** (ECS/staging) | not touched | N/A | No deploy, no digest check, no health probe this session |
| **MCP** (`-tawadoo-mcp-`) | not touched | N/A | Not in scope |
| **Lake / Amplitude** | not touched | N/A | No emitter-landing proof captured |

---

## 5. ROLLBACK CONTRACT
- **N/A this session** — no commit, no deploy, no data change. Nothing to roll back.
- For the implementing session: Job 1 is additive (image-fallback only) → source-revert commit; no migration; no data. 2a is a single-string emitter rename → source-revert commit; verify canonical landing before/after.

---

## 6. TREE STATE / UNTRACKED ARTIFACTS (preserve — §9/§28.4)
Untracked at session end (none mine, all preserved, nothing deleted):
- `playwright-report-b13-qa/`, `playwright-report-scto-r-render/`, `playwright-report-scto11/`, `-scto12/`, `-scto5-full/`, `-scto5/`
- `playwright.o1.config.ts`, `playwright.scto-r-render.config.ts`, `playwright.scto11.config.ts`, `playwright.scto12.config.ts`, `playwright.scto5-full.config.ts`, `playwright.scto5.config.ts`
- `tests/e2e-staging/{o1,scto-r-render,scto11,scto12,scto5-full,scto5}/`
- `semantic-review/`, and prior evidence `.md` files (`AI_LISTING_JOURNEY_INTEGRITY_EVIDENCE_2026_09_05.md`, `CREATE_FORM_CONSOLIDATION_EVIDENCE_2026_09_05.md`, `S_CTO_11_DRAFT_AUTOSAVE_FINISHER_EVIDENCE_2026_09_05.md`, `S_CTO_5_END_SESSION_REPORT_2026_09_05.md`)
- `M yarn.lock` (pre-existing modification, not mine)
- **The prompt mentioned a `scto-r-render` probe dir "leave or clean per note":** I **left it untouched** (did not confirm it safe to delete; a parallel session may own it). Flag: Brain to confirm ownership before any cleanup.

---

## 7. TRAFFIC-LIGHT + R-STATE
- **RED:** none introduced (no code changed). BLK-1 (cross-repo allowlist gap) and BLK-2 (property-vs-event) block the emitter work.
- **YELLOW:** OPEN-1 (404 not browser-confirmed), OOS-1 (empty-Swiper for all photoless listings), OOS-2 (three inconsistent placeholders), OOS-3/4/5 (S-CTO-15 residue + founder B-list + fixture debt).
- **BLUE (done):** source truth for both jobs; carousel icon source identified + reuse path defined; the real 404 site isolated to ListingRow; `category.icon` availability proven on all 3 surfaces; job/service gating constant found; all 3 emitter sites located + cross-checked vs allowlist + S-CTO-15; authoritative research done + cited; both founder doubts resolved with a recommendation.
- **R0/R1/R2/R3:** R0 source-truth ✓ · R1 local+CI ✗ (not run) · R2 deployed+digest ✗ · R3 live/browser ✗.

---

## 8. RECOMMENDATION (for founder/Brain)
1. **Job 1** (category-icon fallback, gated to `JOBS_OR_SERVICES_CATEGORY_IDS`, reusing `getIcon`, on all 3 surfaces) — best-practice-confirmed; authorize a web MODE-B build unit.
2. **Job 2a** (`feed_slide_change`→`feed_slide_changed`) — bundle into the same web unit; safe.
3. **Job 2b** — split to an **API-writable** unit that adds `app_install_banner_dismissed` to the allowlist, then a web rename; sequence, don't merge into web-only.
4. **Job 2c** — skip unless founder clarifies BLK-2.

---

## 9. NEXT SESSION IDs (one-line reasons)
- **S-CTO-17web-BUILD — category-icon fallback + 2a rename (web-writable):** implement Job 1 on ListingRow/productCard/productDetails reusing `getIcon` gated to job/service, plus the safe `feed_slide_changed` rename; build/test/lint/tsc, deploy staging, browser-verify (Chromium+WebKit+mobile+AR-RTL) incl. photoless AND photo listings, prove 2a canonical landing, add a regression guard (§33), do the §28.1 ancestry check. Also decide OOS-1 (empty-Swiper guard) in the same unit or split.
- **S-CTO-16-api — allowlist add for banner canonical (API-writable):** add `app_install_banner_dismissed` (BLK-1) + carry OOS-3/OOS-5 (var-emitted session events, `smart_view_no_results_recovery` wire/prune, `test_event` fixture migration). Then a web rename of `OpenInAppBanner.tsx:231`.
- **Founder micro-decision:** BLK-2 (ai_generated skip vs. rename `ai_generated_multilingual`) + the S-CTO-15 B-list (OOS-4: approve B1/B2 later, reject B3–B5).

---

## 10. TOOLS / PERMISSIONS / COST
- Tools used: source grep/read (web writable-repo + api/S-CTO-15 evidence read-only), `git` (read-only status/log/rev-parse — no mutation), one `context-gatherer` sub-agent (read-only), web search + web_fetch (2 fetches blocked, snippets sufficient).
- No code written, no commit, no push, no build, no deploy. No AWS/DB/BO/MCP access. No novel paid tool, no IAM change, no secret read/printed, no prod touched. **COST: none incurred.**

_S-CTO-17web end. Investigation + research complete; implementation intentionally not started (founder-decision + cross-repo blockers surfaced, then founder requested this report). Nothing built on front/back/DB/BO/AWS this session. Proposed status: FINISHED — INCOMPLETE (BLOCKED). All open, out-of-scope, and error items recorded above for Brain triage — nothing buried._
