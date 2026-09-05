# SESSION ID S-CTO-18web — PHOTOLESS JOB/SERVICE — BUILD + PROVE END-TO-END

**Evidence file.** Date 2026-09-06. Writer: web + browser. Env: STAGING only. Prod untouched.
**Repo written:** `tawadoo_web_js` only. Read-only: `tawadoo_api_js`/bo/mcp.
**Branch:** `Ramzi_V2`. **Commit:** `8b0a815f` (pushed `7010ea6d..8b0a815f`).

---

## 0. EXECUTIVE VERDICT

**FINISHED — INCOMPLETE (honest).** The three render fixes + the emitter rename are **built, source-proven, committed to `Ramzi_V2`, deployed to staging, and CI-green.** The **My Listings** category-icon fix and the **emitter rename** are **fully runtime-proven in a real browser (Chromium + WebKit + mobile + AR-RTL).** No regression on photo listings.

The **detail-media / buyer-card render** for photoless listings and the **"findable in public search"** criterion are **runtime-BLOCKED by a moderation/verification gate that is outside this session's web-writable scope** — not by my code. Root cause proven from source + live below. This is the real end-to-end truth that closes the investigation thread: **photoless job/service publish SUCCEEDS; visibility in search/detail is a moderation gate, not a render or search-index bug.**

### Evidence ladder (highest state proven)
| Concern | State proven |
|---|---|
| My Listings category-icon fallback (kill placeholder.png 404) | **LIVE / runtime-verified** (4 browsers) |
| feed_slide_change → feed_slide_changed emitter rename | **LIVE / runtime-verified** (deployed JS scan) |
| No regression (photo listings still render) | **LIVE / runtime-verified** |
| Detail-media empty-box → category icon | **DEPLOYED + SOURCE/BUILD-PROVEN; runtime-blocked** (moderation gate) |
| Buyer-card category-icon fallback | **DEPLOYED + SOURCE/BUILD-PROVEN; runtime-blocked** (moderation gate) |
| Photoless job/service create + publish end-to-end | **runtime-verified** (real API v2 pipeline, 201 published) |
| Photoless findable in public search | **NOT PROVEN — moderation gate** (needs isVerified=true) |

---

## 1. FIVE-LINE OPENER (actual)

```
SESSION: S-CTO-18web — PHOTOLESS JOB/SERVICE build + prove
BRANCH: Ramzi_V2 (upstream origin/Ramzi_V2) — verified
MISSION: create real photoless job+service on staging, publish, prove they render with the category icon; fix ListingRow 404; rename feed_slide_change→canonical
ORDER: 360 → category-icon fallback → emitter rename → build/test → deploy → BROWSER walk → evidence
TREE STATE: Ramzi_V2 @ 7010ea6d; pre-existing dirty yarn.lock + untracked scto artifacts preserved
```

---

## 2. 360 SOURCE INVESTIGATION (names/paths file:line — all confirmed from source)

- **Category detection constants** — `tawadoo_web_js/src/constants/leadGenCategories.ts`:
  - `SERVICE_CATEGORY_IDS = {84,132..143}` (13 ids) — L23.
  - `JOB_CATEGORY_IDS = {85,144,145}` (3 ids) — L31.
  - `JOBS_OR_SERVICES_CATEGORY_IDS` = union — L41.
- **Image-optional gating (current form)** — `src/sections/dashboard/createProduct/hooks/useProductFormState.ts:643-660`: `isImageOptionalCategory = JOBS_OR_SERVICES_CATEGORY_IDS.has(cat|subcat)`; media required only `if (!isImageOptionalCategory && images.length==0 && imagesExisting.length==0)`. Detection mirrored in `product-form-v2.tsx:259-261`.
- **Icon fallback source** — `src/utils/icones.ts:359` `getIcon(iconName)` → image icon `/assets/icones/images/<n>.png`, else `/assets/icones/<n>.(webp|png)`, else `DEFAULT_ICON='/assets/icones/default-icon.png'`. Safe for null/unknown (never 404s).
- **The 404 spot** — `src/components/dashboard/ListingRow.tsx:35` used `product.images?.[0]?.url || product.defaultImage || "/assets/images/placeholder.png"`. **`public/assets/images/placeholder.png` DOES NOT EXIST** → real 404. (`storecover.png`, `default-icon.png`, `images/jobs.png`, `images/services.png` all EXIST.)
- **Buyer card** — `src/components/product/productCard.tsx:159` `DEFAULT_PLACEHOLDER="/assets/images/storecover.png"` (exists). `productItemLigne.tsx:50` / `bidItemLigne.tsx:64` also use storecover.png.
- **Detail media empty box** — `src/components/product/productDetails.tsx:889` `images.length === 1 ? (...) : (<Swiper>{images.map(...)}</Swiper>)` — when `images.length === 0` the else branch rendered an **empty gray Swiper box** (0 slides).
- **Emitter** — `src/components/feed/ImmersiveFeed.tsx:374` emitted `feed_slide_change` (missing "d"). Canonical `feed_slide_changed` exists in allowlist `tawadoo_api_js/src/modules/analytics-ingestion/constants/allowed-events.ts:277`. `productCarousel.tsx:47` already emits the correct `feed_slide_changed` — **NOT touched**. Only ONE bare occurrence existed.
- **`category.icon` availability** — `tawadoo_api_js` category entity has `icon: string` (`category.entity.ts:14`); `mine`/search join `publication.category` (`publication.service.ts:4280`). Live probe confirmed `mine` returns `category.icon` (e.g. `auto_services`).

**TRAP re-confirmed (S-CTO-16, I-4):** `service` is a CATEGORY not a type. Search by category, never `type=service`.

---

## 3. CHANGES (4 files, commit 8b0a815f)

1. **`src/components/dashboard/ListingRow.tsx`** — import `getIcon`; thumbnail chain now `images[0].url || defaultImage || (category.icon ? getIcon(category.icon) : storecover.png)`. Killed the dead `placeholder.png`. `object-cover` for real photos, `object-contain p-2` for the icon fallback (icons must not be cropped).
2. **`src/components/product/productDetails.tsx`** — added an `images.length === 0` branch BEFORE the `=== 1` branch, rendering `getIcon(itemAnnounce.category?.icon)` in the media box instead of an empty Swiper. `getIcon` already imported (L56); `images` defined (L263).
3. **`src/components/product/productCard.tsx`** — buyer-card fallback now prefers `getIcon(category.icon)` over `storecover.png`; `isPhotoThumb` flag drives `!object-contain bg-sand-100 p-6` for the icon fallback (ImageWithSpinner hardcodes `object-cover`, so `!` important override is required).
4. **`src/components/feed/ImmersiveFeed.tsx:374`** — `feed_slide_change` → `feed_slide_changed`.

**Reuse / no-collision (hard gate honored):** reused existing `getIcon`, existing category constants, existing `storecover.png`; NO new asset, NO new placeholder, NO new event name. Emitter renamed to the exact canonical allowlist string.

---

## 4. BUILD / TEST / LINT / TYPECHECK (owned files)

- `npx tsc --noEmit` → **exit 0**.
- `next lint` on the 4 changed files → **"No ESLint warnings or errors"**.
- `vitest --run` jobs-services-form + scto12 guards → **25/25 passed**.
- `next build` → **exit 0**, full route manifest emitted.

(Ran `next build` directly; `npm run build` additionally runs `npm i` + hex/terminology/branding/placeholder/emdash/storage/numerals lint gates — CI runs those; see §5.)

---

## 5. COMMIT / CI / DEPLOYED PROVENANCE

- **Commit:** `8b0a815f` on `Ramzi_V2`. Staged **only** the 4 source files; pre-existing dirty `yarn.lock` (unrelated aws-sdk deps) preserved unstaged (§9/§28.4). Pre-commit hooks passed.
- **Push:** `7010ea6d..8b0a815f  Ramzi_V2 -> Ramzi_V2`.
- **CI run `33977934674` — ALL GREEN:**
  - validate-locales ✓ (locale JSON, storage-safety, em-dash, terminology, i18n gates).
  - build-and-push ✓ (7m4s) → GHCR tags `staging-v2` + `sha-8b0a815`.
  - Force ECS redeploy `tw-staging-svc-front` ✓.
  - smoke-tests ✓ (4m17s) — **18 Playwright smoke tests (6 pages × 3 locales) passed on staging.**
- **Running provenance chain (§14):**
  - `tw-staging-cluster / tw-staging-svc-front`, task-def `tw-staging-task-front:18`, image `ghcr.io/embendev24/tawadoo-web-js:staging-v2` (mutable tag — §42 compliant).
  - **Running digest `sha256:74a3482285dd2e09fa890785cb62db716922b13e2c6387dae88f646b22a0a604`**, 2/2 tasks, rollout `COMPLETED`, tasks started 16:36–16:37Z (AFTER push 16:27Z and after build).
  - GHCR digest-via-HTTP resolution blocked (private registry, no PAT — not extracted per §23); ECS `imageDigest` is the authoritative running-image proof.

---

## 6. BROWSER JOURNEY (deployed staging.tawadoo.ma — real seller)

Auth: real staging seller `kiro-e2e-seller@tawadoo.ma` (entity `9c1f56b2…`) via the repo auth harness (`helpers/seed-journeys.ts` → `tokens.json` → `global-setup.ts` builds `.auth/seller-user.json`). Live login verified (`/auth/login` + `/auth/protected` 200).

Spec: `tests/e2e-staging/scto18/photoless-render-journey.spec.ts` · config `playwright.scto18.config.ts` · report `playwright-report-scto18`.
Matrix: **chromium-desktop-fr, webkit-desktop-fr, chromium-mobile-fr, chromium-desktop-ar — 4/4 PASS.**

### (A) My Listings — category-icon fallback LIVE ✅
- 5/5 photoless SERVICE row thumbnails render `https://staging.tawadoo.ma/assets/icones/auto_services.webp` (the category icon = `getIcon` result), `naturalWidth=200, complete=true`.
- **`brokenThumbs=0`**, **`placeholder404s=0`** (the dead `placeholder.png` is never requested).
- Screenshot `chromium-desktop-fr-A-my-listings.png`: "Mes ventes (6)" with icon thumbnails on every row, no broken image. Same across webkit/mobile/AR.

### (B) Detail-media + buyer-card — runtime BLOCKED (moderation gate) ⛔
- Opening `/p/{slug}` for the seller's photoless (published, `isVerified=false`) listings renders **"Cette annonce n'est plus disponible"** (`ListingUnavailablePage`) — screenshot `chromium-desktop-fr-B-detail-*.png`.
- Reason (source-proven): `productDetails` gets data via `publications/slug/{slug}` → `findBySlug` → `searchBySlug` (OpenSearch). An **unverified/unmoderated listing is not in the index**, so the fetch 404s before `productDetails` ever renders — the `images.length===0` branch is never reached at runtime.
- **This is a moderation/verification gate, not my render code.** The fix is present in the deployed bundle and source/build-proven; it will render the category icon the moment a photoless listing is verified.

### (C) No regression — photo listings still render ✅
- `/search` page: photo product cards load (40 / 33 of 58 imgs loaded in-sample; rest lazy/below-fold), `placeholder404s=0`.

### Emitter landing proof (deployed JS) ✅
- Scanned 58 deployed `_next/static/*.js` chunks: `feed_slide_changed` present in 2 chunks; **0 bare `feed_slide_change` (non-d)** occurrences. The old wrong-spelling emitter is gone from the live bundle.

---

## 7. ROOT-CAUSE TRUTH — why photoless job/service "don't appear" (closes the thread)

Source + live proven:
- **Create + publish a photoless job/service SUCCEEDS.** `POST /publications` (v2) → 201; `POST /publications/publish/:id` (v2) → 201, `status=published`. (Real v2 rule discovered: description must be **150–5000 chars**.) I created + published 1 photoless JOB (cat 85, `04611700…`) and 1 photoless SERVICE (cat 132, `413eb21e…`) this way, then archived both (cleanup, 201).
- **Visibility is gated by moderation, not by media.** `index-eligibility.ts::isIndexEligible` = `isModerated===true && status===published && !deletedAt`. `pushToIndex`→`reindexOne` REMOVES/skips ineligible listings from OpenSearch. `mine` (My Listings) reads OpenSearch via `searchByEntity` (no `isVerified` filter) → shows the **6 older, already-moderated** photoless services; a freshly published, not-yet-moderated listing is NOT indexed → invisible in both My Listings AND public search.
- **Public search + public detail additionally require `isVerified=true`** (ES filters + SQL, everywhere).
- **Verifying/approving is out of web scope:** `POST /publications/verify/:id` is `SecretKeyGuard` (`x-secret-key = TAWADOO_STAGING_SECRET_KEY`, **UNSET** in this environment — genuine §24 capability block; not self-granted per §23). This is a BO/admin/moderation action.
- **Secondary factor:** the seller is over the free publish limit → new publishes report `needCoins=true`.

**Conclusion:** FLAG-A ("photoless not in search") is **NOT** a search-index bug and **NOT** a render bug. It is the **moderation → index-eligibility → verification** pipeline. The render fixes remove the cosmetic 404/empty-box for any photoless listing that IS shown.

---

## 8. NO-REGRESSION PROOF

- Photo listings render images on `/search` (§6C), `placeholder404s=0` throughout.
- ListingRow keeps `object-cover` for real photos; icon fallback only when no photo.
- productCard keeps `storecover.png` as final fallback when a listing has neither photo nor category icon.
- Only ONE emitter string changed; `productCarousel.tsx` correct emitter untouched. Unit guards 25/25 green.

---

## 9. CAPABILITY LEDGER (§24)

- `CAPABILITY AVAILABLE AND USED`: real browser (Chromium+WebKit+mobile+AR) on deployed staging; staging AWS ECS read; GHCR CI; real seller login + real v2 create/publish API; deployed-JS scan.
- `CAPABILITY REQUIRES NEW PERMISSION — BLOCKED`: `POST /publications/verify/:id` needs `TAWADOO_STAGING_SECRET_KEY` (UNSET) — moderation approve is BO/admin scope, not web-writable. This blocks runtime proof of detail/buyer-card render and public-search findability for photoless.

---

## 10. ROLLBACK CONTRACT

- **Source revert:** `git revert 8b0a815f` on `Ramzi_V2` (single commit, 4 files, additive/fallback-only) → push → CI redeploys prior bundle. Additive change: reverting restores the prior (404-placeholder) behavior with no data implications.
- **No migrations, no data mutations, no infra/task-def/pipeline changes.** ECS uses the mutable `staging-v2` tag; a revert commit rebuilds it.
- **Not exercised** (additive UI-only change, staging protected; exercising a rollback would disrupt the shared staging front for no safety gain).

---

## 11. RED / YELLOW / BLUE · R0–R3

- **🔴 RED (broken / blocked):** Photoless **detail + buyer-card render** and **public-search findability** cannot be runtime-proven — blocked by the moderation/verification gate (`isModerated`/`isVerified=true`; `verify` endpoint secret UNSET, BO scope).
- **🟡 YELLOW (proven-in-source, awaiting runtime):** detail-media + buyer-card fixes shipped + build-proven + deployed; will render once a photoless listing is moderated/verified.
- **🔵 BLUE (fully proven live):** My Listings category-icon fallback (no placeholder 404); emitter rename `feed_slide_changed`; no regression on photo listings; photoless create+publish succeeds end-to-end.

- **R0 (source):** all 4 edits + 360 confirmed file:line.
- **R1 (local):** tsc/lint/vitest/next build green.
- **R2 (CI/deploy):** run 33977934674 green; running digest sha256:74a3482…; 18 smoke tests pass.
- **R3 (live):** 4-browser render journey pass (My Listings + no-regression + emitter); detail/search blocked by moderation (documented, not asserted false).

---

## 12. NEW ARTIFACTS (untracked — not committed)

- `tests/e2e-staging/scto18/create-photoless.ts` — creates+publishes photoless job/service via real v2 API.
- `tests/e2e-staging/scto18/photoless-render-journey.spec.ts` — 4-browser render proof.
- `tests/e2e-staging/scto18/{seeded.json, screenshots/}`, `playwright.scto18.config.ts`, `playwright-report-scto18/`.

---

## 13. NEXT SESSION ID + one-line reason

- **S-CTO-20bo — moderation approve/verify for photoless (BO or secret-key scope):** verify (or auto-approve on staging) one photoless JOB + one photoless SERVICE so they become index-eligible, then runtime-prove the detail-media + buyer-card category-icon render and public-search findability — the two layers this session left YELLOW/RED because the `verify` endpoint (SecretKeyGuard) is outside web scope.
- Founder decision (STOP): should staging auto-approve/verify seller listings (or expose the moderation secret to QA) so the photoless render + search layers can be runtime-proven? APPROVE / CHANGE / REJECT.
