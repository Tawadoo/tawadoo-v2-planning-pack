# SESSION ID S-CTO-R-RENDER — PHOTOLESS-LISTING RENDER SWEEP (READ-ONLY)

**Date:** 2026-09-06 · **Status proposed:** FINISHED — COMPLETE (read-only; verdict per surface delivered)
**Verdict color/risk:** YELLOW · R1 (one buyer-facing surface renders broken; low blast radius, trivial fix)
**Skills:** tawadoo-source-truth (DISCOVERED + ACTIVATED via disclose_context), tawadoo-runtime-provider-qa (DISCOVERED + ACTIVATED via disclose_context) — SKILL COMPLETION recorded below.
**No code changed. HEADs unchanged.**

---

## FIVE-LINE OPENER (corrected after preflight)

```
SESSION: S-CTO-R-RENDER — PHOTOLESS-LISTING RENDER SWEEP (READ-ONLY)
BRANCH: tawadoo_web_js @ Ramzi_V2 (upstream origin/Ramzi_V2), HEAD 7010ea6d — VERIFIED, unchanged at end
MISSION: verify a photoless job/service renders cleanly across all buyer surfaces; report broken spots + whether an image fallback exists. NO fix.
ORDER: source-truth on card components + fallback assets → read-only staging API probe to find a PUBLISHED photoless service → real-browser sweep (Chromium+WebKit+mobile+AR-RTL) of each buyer surface → per-surface verdict → evidence
TREE STATE: only yarn.lock (M, pre-existing) + untracked probe/spec/report artifacts; NO tracked code touched; HEAD unchanged
```

---

## SUBJECT (real, published, photoless)

- Published photoless **SERVICE**: id `2c387987-eb7e-4f91-a6e7-0fec8d8a7f74`, slug `test_de_moderation_e2e_1784992927031`, category `auto_services`, **0 images** — confirmed via read-only `GET /publications/mine` on `api-staging.tawadoo.ma`.
- The S-CTO-12 seeded draft (`67084812-…`) returned **404** (cleaned up) — not needed; 5 real published photoless services already exist for the seller entity, so no create/publish was performed (no coins spent, zero mutation).

---

## SOURCE TRUTH — where the fallbacks are (and aren't)

| Surface component | file:line | Fallback logic | Asset exists? |
|---|---|---|---|
| Search / Home / Category card | `tawadoo_web_js/src/components/product/productCard.tsx:159,169,178` | `DEFAULT_PLACEHOLDER = "/assets/images/storecover.png"`; chain thumb → first publication image → placeholder; `onError` re-falls to placeholder | **YES** — `public/assets/images/storecover.png` (36 KB); staging `storecover.png` → HTTP **200** |
| Listing detail — main gallery | `tawadoo_web_js/src/components/product/productDetails.tsx:263,888–926` | `images = (itemAnnounce.images||[]).filter(publication/video)`; renders `images.length===1 ? single : Swiper`. **No `length===0` branch** → empty Swiper `.map()` → the `bg-sand-100` media box is not mounted at all when there are 0 images | N/A (column collapses; no broken `<img>`) |
| Listing detail — share/modal preview | `tawadoo_web_js/src/components/product/productDetails.tsx:1551` | guarded: `images.length>0 && images[0].url ? … : "/assets/images/storecover.png"` | YES (storecover) |
| **Seller "My listings" / "Mes ventes" row** | `tawadoo_web_js/src/components/dashboard/ListingRow.tsx:35` | `const thumb = product.images?.[0]?.url \|\| product.defaultImage \|\| "/assets/images/placeholder.png"` | **NO** — `public/assets/images/placeholder.png` does **not exist**; staging `placeholder.png` → HTTP **404** ← **THE GAP** |
| Dashboard order/offer/bid/boost cards | `BuyerPurchases.tsx:166`, `BuyerOffers.tsx:194`, `BuyerAuctions.tsx:194`, `MyBoostsCard.tsx:331` | conditional `thumbnail ? <img> : <no-image block>` (renders a clean no-image box) | N/A (clean, no broken img) |

---

## PER-SURFACE RENDER TABLE (real browser, staging.tawadoo.ma)

Browsers: Chromium desktop FR, WebKit desktop FR, Chromium mobile (Pixel 7) FR, Chromium desktop AR-RTL. Broken = `<img>` loaded with `naturalWidth===0`.

| Surface | Verdict | Runtime evidence (all 4 profiles unless noted) | Screenshot | Fallback-gap file:line |
|---|---|---|---|---|
| **Listing detail page** `/{loc}/p/{slug}` | **CLEAN** | HTTP 200; title renders; **0 broken images**; media box `no-box` (photoless media column collapses — no grey hole, no broken icon) | `…/results/…-broken-media-*/detail-*.png` (webkit, mobile, ar) + chromium-fr run green | — (collapses cleanly; see note) |
| **Search results** `/{loc}/search?query=moderation` | **CLEAN** | HTTP 200; 58–59 imgs; **0 broken** | `…/search-*.png` | — (`storecover.png` works) |
| **Home / category cards** `/{loc}` | **CLEAN** | 27–88 imgs; **0 broken** | `…/home-*.png` | — |
| **Seller "My listings"** `/{loc}/dashboard/listings` | **BROKEN** | HTTP 200; **5 broken images**, all `…/assets/images/placeholder.png` (404), on **every** profile | `…/mylist-*.png` | `tawadoo_web_js/src/components/dashboard/ListingRow.tsx:35` |
| **Smart View card** | **UNREACHED** | No dedicated Smart-View card component references `productCard`/`storecover`/`placeholder`/`images[0]` (grep `src/**/smart*/**` → no matches). Smart View reuses the same listing endpoints/cards (Classic-mirror), so it inherits `productCard`'s working fallback, but a dedicated Smart-View card render was not exercised this sweep. | — | (separate contract) |
| **MCP / ChatGPT card output** | **UNREACHED** | Separate repo `-tawadoo-mcp-` / external agent contract; out of this browser sweep's scope. Note for a later MCP card-render check. | — | (separate contract) |
| **Back Office listing view** | **UNREACHED** | BO (`admin_bo_tawadoo`) is a separate authed app; not part of the buyer-facing sweep and no BO auth harness in this config. | — | (separate contract) |

**Detail-page note:** source worst case was an empty grey `bg-sand-100` box; runtime proved *better* — with 0 images the gallery block is not mounted at all (`firstMedia=no-box`), so the two-column layout simply shows the info column. Acceptable, though a deliberate "no photo for this service" placeholder would be a nicer polish (optional, not required).

---

## OVERALL VERDICT

**Does a photoless job/service render acceptably to buyers? → YES for buyers; NO on the seller's own "My listings" view.**

- All **buyer-facing** surfaces (detail, search, home/category) render **clean** with a working placeholder or a clean collapse. Buyers never see a broken image.
- The **one broken surface is seller-facing**: "Mes ventes / My listings" shows a broken-image icon for every photoless listing because `ListingRow` falls back to `/assets/images/placeholder.png`, which does not exist (404). This does not block selling or buying — it is cosmetic — but it looks broken to the seller.

## EXACT MINIMAL FIX (if a tiny fix session is authorized)

One-line change, one file — point the seller row at the placeholder that already exists and ships:
- `tawadoo_web_js/src/components/dashboard/ListingRow.tsx:35`
  - from: `… || "/assets/images/placeholder.png";`
  - to:   `… || "/assets/images/storecover.png";`  (asset present, already 200 on staging)
- Alternative (equally valid): add the missing asset `public/assets/images/placeholder.png`. Prefer reusing `storecover.png` to avoid shipping a new asset.
- Add a guard test / known-regression entry (§33) asserting no `<img>` on `/dashboard/listings` resolves to a 404 asset.

No other surface needs a change.

---

## UNREACHED SURFACES + WHY

- **Smart View card** — no dedicated smart-view card component found; Smart View mirrors Classic endpoints/cards so it inherits the working `productCard` fallback, but was not directly rendered this sweep. Follow-up optional.
- **MCP/ChatGPT card** — separate repo/contract (`-tawadoo-mcp-`, external agent render). Note for a later MCP card-render QA.
- **Back Office listing view** — separate admin app (`admin_bo_tawadoo`); no BO harness in this read-only config; not a buyer surface.

---

## CAPABILITY / SAFETY LEDGER

- CAPABILITY AVAILABLE AND USED: Playwright 1.61.1, Chromium + WebKit installed; seller/premium/free storage states via existing global-setup; read-only `GET` API probes on `api-staging.tawadoo.ma`.
- Mutation: **NONE.** No create/publish/edit/delete. No coins spent. No customer message/order/payment. Only navigation + GETs.
- Secrets: none printed; tokens read from `.auth/tokens.json`, emitted only as booleans.
- Boundary: prod NOT touched. HEAD `7010ea6d` unchanged; only `yarn.lock` (pre-existing M) + untracked helper artifacts (`tests/e2e-staging/scto-r-render/*`, `playwright.scto-r-render.config.ts`) — no tracked code modified, nothing committed.
- Parallel-safe with S-CTO-15 (api-only): this was the single authorized browser sweep; no second heavy suite run.

## SKILL COMPLETION

- **tawadoo-source-truth:** source finding matrix built from actual component files (productCard, productDetails, ListingRow, dashboard cards) with file:line; asset existence verified on disk + on staging (curl 404/200); HEAD/branch/tree provenance recorded. COMPLETE.
- **tawadoo-runtime-provider-qa:** real Chromium + WebKit + mobile + AR-RTL runtime proof on deployed staging; positive (buyer surfaces clean) + the broken-surface control (My Listings 5×404 placeholder); no secrets; no mutation; screenshots captured. COMPLETE for this read-only render scope.

## RECOMMENDED FOLLOW-UP

Fire a **tiny fix session (R1, ~1-line + guard test)** to repoint `ListingRow.tsx:35` to `storecover.png` (or add the missing `placeholder.png`). One-line reason: photoless listings render broken only on the seller's own "My listings" view because the fallback points at a 404 asset — everything buyer-facing is already clean.
