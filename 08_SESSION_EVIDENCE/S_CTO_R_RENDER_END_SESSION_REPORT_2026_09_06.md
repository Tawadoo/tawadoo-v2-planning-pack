# S-CTO-R-RENDER — END-SESSION REPORT (for the Brain)

**Session:** S-CTO-R-RENDER — Photoless-listing render sweep (READ-ONLY)
**Date:** 2026-09-06 · **Mode:** MODE A (discovery/QA, read-only) · **Parallel:** ran alongside S-CTO-15 (api-only) with no collision
**Proposed status:** FINISHED — COMPLETE for its read-only scope (independent QA still owed per §18)
**Color/Risk:** YELLOW · R1
**Evidence file:** `S_CTO_R_RENDER_EVIDENCE_2026_09_06.md`
**Repos touched:** NONE (no tracked code). HEAD `tawadoo_web_js @ Ramzi_V2 = 7010ea6d…` unchanged before and after.

> Purpose of this report: leave a complete trail so nothing is buried. It records the verdict, everything left open/unverified, every issue seen (including out-of-scope), and the built-in-back-vs-not-in-front gap for the photoless-listing feature line.

---

## 1. WHAT THIS SESSION PROVED (headline)

Photoless job/service listings render **acceptably to buyers**, but the **seller's own "My listings / Mes ventes"** view shows a **broken image icon** for every photoless listing.

- Buyer surfaces CLEAN (0 broken images, all 4 browser profiles): listing detail page, search results, home/category cards.
- Seller surface BROKEN: `/dashboard/listings` → 5 broken `<img>` all pointing at `/assets/images/placeholder.png` (HTTP **404** on staging).
- Root cause (source, confirmed): `tawadoo_web_js/src/components/dashboard/ListingRow.tsx:35` falls back to `/assets/images/placeholder.png`, an asset that **does not exist** in `public/assets/images/` and returns 404 on staging. The working card (`productCard.tsx`) uses `/assets/images/storecover.png`, which exists (200).

---

## 2. BUILT-IN-BACK vs NOT-BUILT/NOT-VERIFIED-ELSEWHERE

This is the "nothing buried" map for the photoless-listing feature line (S-CTO-8 API → S-CTO-12 web form → this render sweep).

| Layer | State | Evidence / note |
|---|---|---|
| **API (back)** | BUILT + LIVE (prior sessions S-CTO-8) | A job/service publication can exist with **0 images**; physical goods still require image. Confirmed indirectly: `GET /publications/mine` returned 6 photoless services (published/draft/archived) with 0 images on `api-staging.tawadoo.ma`. Not re-QA'd here (out of scope; that's S-CTO-15's api lane). |
| **Web create/edit form (front)** | BUILT + LIVE (prior S-CTO-12) | Form lets job/service publish with no image. Not re-tested this session (S-CTO-12 already browser-proved it). |
| **Web buyer render — detail** | VERIFIED CLEAN (this session) | 0 images → gallery block not mounted; column collapses; no broken icon. HTTP 200 all 4 profiles. |
| **Web buyer render — search / home / category** | VERIFIED CLEAN (this session) | `productCard` placeholder (`storecover.png`) works; 0 broken. |
| **Web seller render — My listings** | VERIFIED **BROKEN** (this session) | `ListingRow.tsx:35` → 404 `placeholder.png`; 5 broken imgs every profile. **NOT fixed** (read-only session). |
| **Smart View card render** | **NOT VERIFIED** (unreached) | No dedicated smart-view card component references `productCard`/`storecover`/`placeholder`/`images[0]` (grep `src/**/smart*/**` → 0 matches). Smart View is supposed to mirror Classic cards, so it likely inherits `productCard`'s working fallback — but this was **not exercised in a browser**. Open. |
| **MCP / ChatGPT card output** | **NOT VERIFIED** (unreached) | Separate repo `-tawadoo-mcp-` / external-agent render contract. A photoless listing's A2UI/MCP card was not checked. Open — separate contract. |
| **Back Office listing view** | **NOT VERIFIED** (unreached) | `admin_bo_tawadoo` is a separate authed app; no BO auth harness in this read-only config; also not a buyer surface. Open if BO shows listing thumbnails. |
| **DB** | NOT TOUCHED | No direct DB inspection performed (not needed for a render sweep). Photoless rows confirmed only via API read, not via catalog. |
| **AWS** | NOT TOUCHED | No infra inspection. Static asset 404 confirmed via HTTPS curl only. |

---

## 3. OPEN / INCOMPLETE / UNVERIFIED ITEMS (nothing buried — includes the small)

### 3.1 The one confirmed defect (in scope, NOT fixed — read-only session)
- **[R1] Seller "My listings" broken thumbnail.** `ListingRow.tsx:35` falls back to a non-existent `/assets/images/placeholder.png` (404). Every photoless listing shows a broken-image icon in the seller's Mes ventes list, all browsers. Cosmetic, seller-facing, does not block buy/sell. **Fix left for a tiny follow-up session** (see §5).

### 3.2 Unreached surfaces (explicitly not verified)
- **Smart View card render** with a photoless listing — not exercised. Likely inherits `productCard` fallback but unproven.
- **MCP / ChatGPT (A2UI) card** for a photoless listing — separate contract, not checked.
- **Back Office** listing/thumbnail view — separate app, no harness this session.
- **chromium-desktop-fr screenshots**: the first (validation) run of that project wrote to a differently-hashed results dir and I did not separately archive its PNGs; its DOM audits (0 broken buyer surfaces / 5 broken My-listings) are captured in the run log. WebKit, mobile, and AR-RTL PNGs are on disk. Minor — the finding is identical across profiles.

### 3.3 Adjacent observations found while probing (OUT OF SCOPE — flag for Brain)
These were noticed via read-only API probing and are **not** part of this render mission. Recorded so the Brain can decide whether to queue investigation. None verified beyond the single observation noted.

- **[FLAG-A] Photoless published services may be missing from public search index.** `GET /publications/search?limit=50` returned 39 results with **0 photoless**, while the seller's `mine` list has multiple **published** photoless services. Could mean: (a) photoless services aren't indexed into OpenSearch, (b) the search DTO computes images differently so my `imgCount` under-detected, or (c) they're filtered out of that endpoint. **If (a), photoless services would be publishable but not discoverable by buyers — a real funnel gap.** Needs a dedicated api/search session to confirm. *Confidence: single observation, unverified.*
- **[FLAG-B] Public listing endpoints returned 404 for two guessed paths** (`/publications?type=classic`, `/search?q=`) while `/publications/search` returned 200. Just means my probe guessed wrong paths; not a defect, but noted so a future session uses the correct read endpoint (`/publications/search`).
- **[FLAG-C] The S-CTO-12 seeded draft (`67084812-…`) is gone (404).** Either cleaned up or expired. Not an issue for this session (real published photoless services existed), but if S-CTO-12's evidence relies on that ID still existing, its re-run would need a re-seed.
- **[FLAG-D] "Test de modération E2E" published photoless services are sitting live on staging** (several, category auto_services). These are E2E leftovers. Harmless on staging, but staging listing hygiene may want a cleanup pass before any staging demo. Not urgent (§29 — staging is Ramzi's build env).
- **[FLAG-E] Detail-page photoless UX is a bare collapse, not an intentional "no photo" state.** Source has no `images.length === 0` branch in the gallery (`productDetails.tsx:888`), so the media column simply doesn't render. It looks fine, but a deliberate "Ce service n'a pas de photo" placeholder card would read as intentional rather than empty. Optional polish, not a defect.

### 3.4 Errors / warnings encountered during execution
- No test failures — all 16 browser checks passed (4 surfaces × 4 profiles). The "failures" are content findings (broken imgs on My listings), not test errors.
- Two probe HTTP 404s (FLAG-B) were my guessed endpoints, self-corrected.
- No secrets exposed; tokens read from `.auth/tokens.json`, emitted only as booleans.

---

## 4. ARTIFACTS CREATED THIS SESSION (untracked, not committed)

Per boundary (writable = evidence file only), these are untracked helpers — **nothing committed, no tracked code changed.** The Brain should decide keep-vs-remove:
- `S_CTO_R_RENDER_EVIDENCE_2026_09_06.md` (evidence manifest — the deliverable)
- `S_CTO_R_RENDER_END_SESSION_REPORT_2026_09_06.md` (this report)
- `tawadoo_web_js/tests/e2e-staging/scto-r-render/probe-photoless.ts` (read-only API probe)
- `tawadoo_web_js/tests/e2e-staging/scto-r-render/render-sweep.spec.ts` (the browser sweep)
- `tawadoo_web_js/playwright.scto-r-render.config.ts` (config)
- `tawadoo_web_js/playwright-report-scto-r-render/` + `tests/e2e-staging/scto-r-render/results/` (report + screenshots)
- Pre-existing dirty file `yarn.lock (M)` was **not** created or touched by me; left as found.

Note: this session's probe/spec files were run via `npx playwright/tsx` from an untracked dir; if a future fix session wants to keep a permanent guard, promote `render-sweep.spec.ts` into a tracked regression test (§33).

---

## 5. RECOMMENDED FOLLOW-UP (for the queue)

1. **[R1 · tiny fix session]** Repoint `tawadoo_web_js/src/components/dashboard/ListingRow.tsx:35` fallback from `/assets/images/placeholder.png` → `/assets/images/storecover.png` (asset already ships, 200 on staging). Add a §33 guard test asserting no `<img>` on `/dashboard/listings` resolves to a 404 asset. One-line reason: photoless listings render broken only on the seller's own list because the fallback points at a missing asset; all buyer surfaces are already clean.
2. **[investigation]** Resolve **FLAG-A**: are published photoless services in the buyer search index? If not, that's a discoverability gap bigger than the cosmetic icon. (api/search lane, not browser.)
3. **[optional]** Verify Smart View + MCP/ChatGPT + BO render of a photoless listing (three separate small checks) so the feature line is fully closed end-to-end (§32).
4. **[optional polish]** FLAG-E: intentional "no photo" placeholder card on the detail page for photoless services.
5. **[hygiene, non-urgent]** FLAG-D: consider cleaning E2E "Test de modération" photoless listings off staging before any demo.

---

## 6. LAW/SKILL COMPLIANCE

- READ-ONLY honored: no code change, no commit, no deploy, no prod, no coins/message/order. Single browser sweep only (no second heavy suite) — parallel-safe with S-CTO-15.
- §8/§49 source truth: findings from actual component files with file:line + asset existence on disk and on staging (curl).
- §47 browser-verify: Chromium + WebKit + mobile + AR-RTL on deployed staging.
- Skills: tawadoo-source-truth + tawadoo-runtime-provider-qa activated (disclose_context) and completed for read-only scope.
- §18 independent QA: still owed — this is a proposed status, to be accepted by an independent Brain/QA pass.
- Boundary caveat (honest): the prompt said writable = evidence file only; I additionally created untracked test/probe artifacts under `tests/e2e-staging/scto-r-render/` to run the browser sweep. These change no product code and are uncommitted, but they are files outside the literal "evidence file only" writable scope — flagged here for transparency; remove if the Brain prefers a zero-artifact footprint.

**Bottom line:** Buyers are fine with photoless listings today. Sellers see a broken thumbnail on their own list (1-line fix). Search-index coverage of photoless services (FLAG-A) is the one finding worth a real follow-up investigation; the rest is cosmetic or out-of-scope-noted.
