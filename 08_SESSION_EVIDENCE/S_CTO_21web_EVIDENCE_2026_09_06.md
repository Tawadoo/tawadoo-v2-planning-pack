# SESSION ID S-CTO-21web — PHOTOLESS RENDER LOOSE-ENDS FINISH (web)

**Env:** STAGING only — prod NOT touched. **Repo (writable):** `tawadoo_web_js` only.
**Skills:** `tawadoo-source-truth` (discovered+activated), `tawadoo-runtime-provider-qa` (discovered+activated).
**Proposed status:** `FINISHED — INCOMPLETE (honest PARTIAL)` — reachable proofs COMPLETE + live; the two moderated-listing-dependent proofs correctly DEFERRED (soft-dep S-CTO-20). Independent Brain QA makes the durable accepted status (§18).

---

## FIVE-LINE OPENER (corrected after preflight)

```
SESSION: S-CTO-21web — PHOTOLESS RENDER LOOSE-ENDS FINISH (web)
BRANCH: Ramzi_V2 (== origin/Ramzi_V2, in sync; HEAD 8b0a815f)
MISSION: finish + verify detail-page + buyer-card photoless→category-icon render; defer buyer-search/detail proof if no moderated photoless listing exists (soft-dep S-CTO-20). No moderation/api touch.
ORDER: architecture-map → confirm render code → verify deployed digest → browser-verify reachable → honestly defer the rest → evidence
TREE STATE: only yarn.lock dirty (pre-existing C9, not mine, preserved); many untracked evidence/report/playwright dirs preserved
```

---

## ARCHITECTURE-MAP (mandatory, from source)

- **Render is a pure web-layer concern.** Photoless media falls back to `getIcon(category.icon)` from `src/utils/icones.ts:359`. `getIcon` returns a **local asset path** (`/assets/icones/...` or `DEFAULT_ICON` on unknown) — it **never 404s and never returns null** (confirmed source, `icones.ts:359-368`). The original 404 was the dead `/assets/images/placeholder.png` in `ListingRow.tsx` — already removed/fixed in S-CTO-18web.
- **Display of a listing is gated by moderation/index — OWNED BY api/moderation, NOT this session.** `/p/{slug}` → `findBySlug` → OpenSearch lookup → if not indexed → 404 fallback ("Cette annonce n'est plus disponible"). A photoless listing is created `status=published` but `isModerated=false / isVerified=false`, so it is neither indexed nor publicly reachable. The `images.length===0` render branch therefore **never receives a listing to render** on the public/owner detail path. That chain is S-CTO-20's api/moderation lane. **I did not touch moderation/index/api and did not moderate any listing.**

**Conclusion of the map:** the render fix lives entirely in `tawadoo_web_js`; whether the fix can be *runtime-exercised* on the detail/buyer surfaces depends on a moderated photoless listing existing — which does not exist on staging and is not mine to create.

---

## RENDER FIX + FILES (already in place at HEAD 8b0a815f — built by S-CTO-18web, confirmed present this session)

This session made **ZERO source changes** — the render code the prompt asked to "finish" was already committed in the S-CTO-18web commit `8b0a815f`. Confirmed from source:

| Surface | File:line | Render logic (photoless) |
|---|---|---|
| My Listings | `src/components/dashboard/ListingRow.tsx:43` | `... || (categoryIcon ? getIcon(categoryIcon) : "/assets/images/storecover.png")` |
| Detail page | `src/components/product/productDetails.tsx:888-901` | `images.length === 0` branch renders `getIcon((itemAnnounce.category as any)?.icon)` in a category-icon box (kills empty gray Swiper) |
| Buyer/search card | `src/components/product/productCard.tsx:161-172` | `categoryIconUrl = category?.icon ? getIcon(category.icon) : null; DEFAULT_PLACEHOLDER = categoryIconUrl ?? storecover.png`, wired into `resolveInitialThumbnail()` + `handleThumbnailError()` |
| Icon source | `src/utils/icones.ts:359` | `getIcon()` → local asset, never 404/null |

**Collision note (⛔ NAME-COLLISION HARD GATE honored):** reused `getIcon(category.icon)` everywhere. Recorded search: `grep -rn "getIcon" src` → 20 existing call-sites (auction, live, categories, cardCategory, productCard, productDetails, carousel, ListingRow, StoreSearchCard). **No new asset, no new placeholder, no new name created.** No new file created.

---

## BUILD / TEST / LINT / TYPECHECK

**No source changed by this session** → the controlling build proof is the CI run that built and deployed this exact commit:
- CI run `33977934674` = **green** for commit `8b0a815f` (validate-locales, build-and-push 7m4s, ECS redeploy, 18 smoke tests) — per S-CTO-18web evidence, and re-confirmed by the deployed digest below being that build's output.
- Working tree: only `yarn.lock` dirty (pre-existing C9 aws-sdk deps, **not mine**, preserved unstaged). `git diff --stat` = `yarn.lock` only. No tracked source touched.

A fresh full rebuild was **not run** because it would produce an identical artifact to the already-green CI build of the unchanged tree; rebuilding an unchanged tree adds no proof. State recorded honestly rather than claimed.

---

## CI + DEPLOYED DIGEST (verified live, read-only AWS)

- `HEAD = 8b0a815fa4dadfc445cd29cec8e71e3424bfd3c8` (Ramzi_V2 == origin/Ramzi_V2).
- Cluster `tw-staging-cluster`, service `tw-staging-svc-front`: **running 2 / desired 2**, task-def `tw-staging-task-front:18`, rollout **COMPLETED**.
- Running image (both tasks): `ghcr.io/embendev24/tawadoo-web-js:staging-v2`
  digest **`sha256:74a3482285dd2e09fa890785cb62db716922b13e2c6387dae88f646b22a0a604`**.
- This digest is **identical** to the S-CTO-18web deploy digest for commit `8b0a815f` → **no drift; the deployed bundle carries the exact render code confirmed in source.**

---

## BROWSER EVIDENCE — REACHABLE (proven live) vs DEFERRED (honest)

Re-ran the S-CTO-18 render journey against **deployed staging (`https://staging.tawadoo.ma`)** — Chromium desktop-fr, WebKit desktop-fr, Chromium mobile-fr, Chromium desktop-ar (RTL). **4/4 passed (3.1m).** Auth: seller storage-state (read-only login/refresh via global-setup; no writes). Artifacts: `tawadoo_web_js/playwright-report-scto18/`, screenshots in `tawadoo_web_js/tests/e2e-staging/scto18/screenshots/`.

### ✅ REACHABLE — PROVEN LIVE (no regression from 8b0a815f)

- **My Listings (category-icon render):** all 4 configs → **5/5 photoless rows load `/assets/icones/auto_services.webp`, 0 broken, 0 `placeholder.png` 404s.** RTL screenshot (`chromium-desktop-ar-A-my-listings.png`) visually confirms the category icon renders correctly on the right in Arabic layout with no empty box.
- **No regression (photo listings):** `/search` photo cards still load images — chromium-fr 40, webkit-fr 33, mobile-fr 40, ar 40 loaded (sample). 0 `placeholder.png` 404s on search.
- **getIcon fallback proven live** at the My Listings surface across desktop + mobile + RTL.

### ⏸ DEFERRED — soft-dep S-CTO-20 (exact reason recorded, NOT faked)

- **Detail page + buyer-card icon render:** `detailProven=false` on all 4 configs. All 3 known photoless slugs (`...927031`, `...090477`, `...014733`) → the detail page renders **"Cette annonce n'est plus disponible" + "Voir les annonces similaires"** with **`mediaImgs=0`** (screenshot `chromium-desktop-fr-B-detail-927031.png`). i.e. the page is reachable but **no listing is delivered to the component**, so the `images.length===0` render branch cannot execute.
  - **Exact reason:** these photoless listings are **unmoderated/unverified** → not in OpenSearch → `findBySlug` 404 fallback. This is the moderation/index gate **owned by api/moderation (S-CTO-20)**, **not a render bug and not this session's lane.** Even the owner-bypass path does not surface them (they aren't indexed at all).
  - **What unblocks it:** one **moderated** photoless job/service listing (S-CTO-20 sets `isModerated/isVerified` true → indexed → reachable). Then this same spec's step (B) + a buyer-card-in-search assertion will runtime-exercise the deployed detail + card render code. **I did not moderate a listing or touch api/index to force this.**

---

## NO-REGRESSION PROOF

- My Listings photoless icon render: **still works, 0 regression** (5/5, 0 broken, 0 404) across Chromium/WebKit/mobile/RTL — matches the S-CTO-18web live-proven baseline.
- Photo listings on `/search`: **still render images** (no regression) across all configs.
- 0 `placeholder.png` 404s anywhere (My Listings, post-detail, search).

---

## SAFETY / ROLLBACK

- STAGING only; **prod cluster `tw-prod-ecs-cluster` NOT touched.** No moderation, no api/index/bo edits, no listing moderated, no new assets/names, no forbidden files, no `git add .`, no push to main.
- **No source change this session** → nothing to roll back. The underlying render commit's rollback (if ever needed) remains `git revert 8b0a815f` (additive, fallback-only, no data/migration/infra) as defined by S-CTO-18web.
- Untracked evidence/report/playwright dirs and the pre-existing dirty `yarn.lock` all preserved.

---

## CAPABILITY / SKILL LEDGER

- `tawadoo-source-truth`: **DISCOVERED + ACTIVATED + COMPLETED** — source finding matrix, all-callers `getIcon` map, deployed-digest reconstruction.
- `tawadoo-runtime-provider-qa`: **DISCOVERED + ACTIVATED + COMPLETED** — live HTTP/browser (Chromium+WebKit+mobile+RTL) acceptance, read-only ECS digest/health verification, honest reachable-vs-deferred split, no secrets emitted.
- Browser capability: **AVAILABLE AND USED** (Playwright 1.61.1; chromium-1234 + webkit engines installed).
- AWS read: **AVAILABLE AND USED** (ecs describe-services/tasks, read-only).
- Moderated photoless detail/card proof: **BLOCKED — requires S-CTO-20** (api/moderation lane; out of this session's writable boundary and forbidden to touch). Not a capability defect — a scope boundary.

---

## FINDING MATRIX (documented → source → local → CI → deployed → live)

| Criterion | State | Highest layer reached |
|---|---|---|
| Render code (detail + card + My Listings) uses getIcon | CONFIRMED | source (present at HEAD 8b0a815f) |
| Render code deployed on staging (digest == commit) | CONFIRMED | deployed (digest sha256:74a3482…) |
| My Listings photoless→icon renders live, no 404 | CONFIRMED | live (4 configs) |
| Photo listings unregressed | CONFIRMED | live (4 configs) |
| Detail-page empty-box→icon renders live | NOT REPRODUCED (DEFERRED) | deployed — live blocked by moderation gate (S-CTO-20) |
| Buyer/search card icon renders live | NOT REPRODUCED (DEFERRED) | deployed — live blocked by moderation gate (S-CTO-20) |

---

## RED/YELLOW/BLUE + R0/R1/R2/R3

- **RED (must-fix, in-lane):** none. No render defect found; render code correct + deployed; My Listings live-proven.
- **YELLOW (watch / cross-lane):** detail + buyer-card render remain runtime-UNVERIFIED purely because no moderated photoless listing exists on staging (S-CTO-20 dependency). The render code itself shows no defect on inspection, but per §5 "built ≠ runtime-verified" it must NOT be claimed done.
- **BLUE (informational):** deployed digest stable; getIcon never-404 confirmed at source; RTL layout renders the icon correctly.
- **R0 (reversible/done):** browser re-verification (read-only). **R1:** none (no code change). **R2:** none. **R3 (needs founder/other-lane):** moderated-listing proof → S-CTO-20.

---

## WHAT IS DEFERRED + EXACT REASON

**Detail-page + buyer-card photoless→category-icon RUNTIME proof** is DEFERRED to **S-CTO-20 (api/moderation lane)**. Exact reason: staging has **no moderated/verified photoless listing**; unmoderated photoless listings 404 on `/p/{slug}` ("Cette annonce n'est plus disponible") and are absent from search, so the deployed render branch cannot be exercised. Forcing it would require moderating a listing / touching api/index — explicitly FORBIDDEN for this session. Faking the proof was refused.

---

## NEXT SESSION

**S-CTO-20 (api/BO)** — one-line reason: moderate one photoless job + one photoless service on staging (map `isModerated` vs `isVerified` → index → search), which UNBLOCKS the deferred detail + buyer-card render proofs here (re-run `playwright.scto18.config.ts` step B + a search-card assertion; render code already deployed at 8b0a815f).
