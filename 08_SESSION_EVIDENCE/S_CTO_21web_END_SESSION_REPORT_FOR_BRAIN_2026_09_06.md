# S-CTO-21web — END-SESSION REPORT FOR THE BRAIN (audit input, not self-acceptance)

**Date:** 2026-09-06 · **Writer session:** S-CTO-21web (web + browser) · **Env:** STAGING only, prod untouched.
**Purpose:** Hand the Brain a complete, honest map of what this session did, what it proved, what it did NOT prove, and every open thread / issue / anomaly seen — including things outside its scope (api, moderation, DB, BO, AWS, feeds). Nothing buried. This is a proposal for QA; the Brain makes the durable accepted status (§18).
**Companion evidence file:** `S_CTO_21web_EVIDENCE_2026_09_06.md` (finding matrix, digests, browser logs, RED/YELLOW/BLUE).

**Self-assessment of this session's quality:** Correct and honest, but note the meta-issue in Part C0: the prompt asked me to "finish + build" render code that **was already fully built and deployed** by S-CTO-18web. There was nothing to build. The real deliverable was *verification*, and the verifiable part was already the same spec S-CTO-18web wrote. So this session largely **re-confirmed** S-CTO-18web rather than adding new ground. The Brain should decide whether re-proving an already-proven fix warranted a full session, or whether S-CTO-21web should have been folded into S-CTO-20.

---

## PART A — WHAT THIS SESSION ACTUALLY DID (front)

**ZERO source code changed.** No commit, no push, no deploy. The render code the prompt asked to "finish" was already committed in **S-CTO-18web commit `8b0a815f`** and already deployed. Confirmed from source, present at HEAD:

1. `src/components/product/productDetails.tsx:888-901` — `images.length===0` → `getIcon((itemAnnounce.category as any)?.icon)` (kills empty gray Swiper for photoless). **Present, unchanged.**
2. `src/components/product/productCard.tsx:161-172` — buyer-card `categoryIconUrl` fallback via `getIcon`, wired into `resolveInitialThumbnail()` + `handleThumbnailError()`. **Present, unchanged.**
3. `src/components/dashboard/ListingRow.tsx:43` — My Listings icon fallback via `getIcon`. **Present, unchanged.**
4. `src/utils/icones.ts:359` — `getIcon()` returns a **local asset path**, `DEFAULT_ICON` on unknown → **never 404s, never null**. **Confirmed by source read.**

**What I DID (all read-only / verification):**
- Confirmed the render code from source + traced all 20 `getIcon` call-sites (no collision, no new name/asset created).
- Verified deployed digest via read-only AWS: `tw-staging-cluster / tw-staging-svc-front`, task-def `:18`, 2/2 running, rollout COMPLETED, image `ghcr.io/embendev24/tawadoo-web-js:staging-v2` digest `sha256:74a3482285dd2e09fa890785cb62db716922b13e2c6387dae88f646b22a0a604` — **identical to the S-CTO-18web deploy of 8b0a815f** (no drift).
- Re-ran the S-CTO-18 render journey (`playwright.scto18.config.ts`) against deployed staging in Chromium desktop-fr, WebKit desktop-fr, Chromium mobile-fr, Chromium desktop-ar (RTL). **4/4 passed.** Auth via seller storage-state (read-only login/refresh through global-setup — no writes).

---

## PART B — WHAT I PROVED vs DID NOT PROVE (the honest split)

### PROVEN LIVE (real browser, deployed staging, 4 configs)
- **My Listings photoless→category-icon**: 5/5 photoless rows render `/assets/icones/auto_services.webp`, 0 broken, **0 `placeholder.png` 404s** — across Chromium/WebKit/mobile/RTL. RTL screenshot confirms the icon renders correctly on the right with no empty box. (Re-confirms S-CTO-18web; no regression.)
- **No regression (photo listings)**: `/search` photo cards still load images (40/33/40/40 loaded in sample), 0 placeholder 404s.
- **Deployed digest == commit 8b0a815f** (read-only AWS).
- **`getIcon` never-404 guarantee**: confirmed from source (`icones.ts:359-368`).

### BUILT + DEPLOYED BUT NOT RUNTIME-VERIFIED (shipped in 8b0a815f, live-proof blocked)
- **`productDetails.tsx` empty-box→icon (detail page)**: still **UNVERIFIED at runtime**. All 3 known photoless slugs (`...927031`, `...090477`, `...014733`) on `/p/{slug}` render **"Cette annonce n'est plus disponible" + "Voir les annonces similaires"** with `mediaImgs=0` (screenshot captured). The `images.length===0` branch never receives a listing, so it cannot execute. **Reason = moderation/index gate (unmoderated → not in OpenSearch → 404 fallback), owned by api/moderation (S-CTO-20), NOT a render bug and NOT my lane.**
- **`productCard.tsx` buyer-card icon (search card)**: still **UNVERIFIED at runtime**. Photoless listings aren't in public search (unverified), so the buyer card with icon fallback never rendered live. Same moderation-gate reason. The `!object-contain` icon-fallback styling was never visually confirmed on a buyer card.

### NOT PROVEN ANYWHERE (unchanged since S-CTO-18web — the supply/demand black zone)
Everything in S-CTO-18web Parts B/C/D that is out of the web-render lane remains open and was **not** advanced by this session (I did not touch api/moderation/index/DB/BO/feeds). Carried forward verbatim below in Part C so nothing is buried.

---

## PART C — OPEN THREADS / ISSUES / ANOMALIES (flag everything, incl. out-of-scope)

### C0. 🟡 META: prompt asked to "build/finish" code that was already built + deployed
S-CTO-21web's prompt said "finish + deploy the render fix." But `8b0a815f` (S-CTO-18web) already shipped all three render fixes to staging. There was **no code to write and no deploy to do**. The only novel work possible was runtime verification, and the reachable part was identical to the spec S-CTO-18web already ran. **The Brain should reconcile the queue:** S-CTO-21web's real value = "confirmed no drift + re-proved reachable + formally deferred the rest to S-CTO-20." Consider whether this should have been a note on S-CTO-20 rather than a standalone session. **Prompt-authoring signal, not an execution defect.**

### C1. 🔴 Detail + buyer-card render CANNOT be proven until a photoless listing is moderated (hard dependency on S-CTO-20)
This is the one real open item in-lane-adjacent. The deployed render code is correct on inspection but per §5 ("built ≠ runtime-verified") must NOT be claimed done. **Blocker:** staging has zero moderated/verified photoless listings. **Unblock:** S-CTO-20 moderates one photoless job + one service → indexed → reachable → re-run `playwright.scto18.config.ts` step (B) + add a buyer-card-in-search assertion. No web code change needed.

### C2. 🔴 Photoless publish leaves listing UNMODERATED → invisible in search AND feeds (carryover from S-CTO-18web C2 — UNRESOLVED)
Fresh photoless publishes return `status=published` but `isModerated=false`/`isVerified=false` and don't land in OpenSearch. **Who sets `isModerated=true`?** The moderation branch (`enableModeration` → `updatePublicationIsModerated(id,true)`) only runs when `ENABLE_CONTENT_MODERATION=true` AND `x-api-version:v2`. **Nobody has confirmed whether staging has `ENABLE_CONTENT_MODERATION=true`, nor whether the branch runs/fails silently.** This is the core unknown of the whole photoless thread. **Out of my scope (api). S-CTO-20.**

### C3. 🟡 `isModerated` vs `isVerified` are two different flags with unclear relationship (carryover S-CTO-18web C3 — UNRESOLVED)
The 6 older My-Listings photoless services are `isModerated=true` (indexed → show in My Listings) yet `isVerified=false` (NOT in public search / detail 404). Index eligibility uses `isModerated`; public search + detail use `isVerified`. **Who sets which, and why do these have one but not the other? The exact state machine is unmapped.** **S-CTO-20 should map `create → publish → isModerated → isVerified → index → search → feeds`.**

### C4. 🔴 Google Merchant feed may SILENTLY DROP photoless listings (carryover S-CTO-18web C1 — UNVERIFIED, potential supply leak)
`syndication-feed.service.ts:43` uses `.innerJoinAndSelect('pub.images','images')` → a publication with zero images is excluded from the feed entirely, even if `is_moderated=true AND status=published`. Yet `FeedEligibilityQuery.baseConditions` has NO image requirement. **Contradiction** between central eligibility (photoless OK) and the Google feed builder (inner-join drops photoless). If photoless job/service are meant to distribute, this is a real supply leak. **NOT runtime-verified. api lane.**

### C5. 🟡 Owner cannot preview own unverified photoless detail (carryover S-CTO-18web C6 — product decision needed)
`product-details-view.tsx` fetches `publications/slug/{slug}` client-side; the owner-bypass in `findBySlug` only helps if the listing is INDEXED. **I re-confirmed this live this session:** as the authenticated seller/owner, all 3 photoless slugs still returned "Cette annonce n'est plus disponible." So a seller who publishes a photoless job/service **currently cannot see their own live detail page** until a moderator approves. **Is that intended? Founder/product decision.**

### C6. 🟡 `feed_slide_changed` emitter lake-landing still NOT verified (carryover S-CTO-18web C7)
S-CTO-18web fixed the emitter spelling in the bundle but never confirmed the event LANDS in `ta_analytics_event` / Amplitude / S3. **Untouched this session.** Folds into the S-CTO-15/18 Amplitude parity gap (tracking-plan registration not done; only 1/28 events runtime-exercised; DEFAULT-partition events dormant).

### C7. ⚪ Pre-existing dirty `yarn.lock` on Ramzi_V2 — owner still unidentified (carryover S-CTO-18web C9)
`yarn.lock` was dirty at my session start (299 insertions, aws-sdk deps — likely `@aws-sdk/client-bedrock-runtime`, an AI/Bedrock feature branch). **Not mine. Preserved unstaged, untouched.** Someone's uncommitted dependency change is still sitting on the Ramzi_V2 working tree. **Brain should identify the owner and decide whether to commit or discard.** Per §28.4, don't reset/clean it.

### C8. ⚪ Stale test-data on staging (carryover S-CTO-18web C10)
The 2 archived test listings from S-CTO-18web (JOB `04611700-...`, SERVICE `413eb21e-...`, both `status=archived`) plus 6 older E2E photoless services persist on staging DB. This session created **no new listings** (verified: `seeded.json` unchanged, re-run reused existing IDs + read-only login). But the clutter remains. **Staging test-data hygiene is drifting — Brain should schedule a cleanup pass.**

### C9. 🟡 Untracked evidence/report/playwright clutter accumulating on Ramzi_V2 working tree
`git status` shows a large untracked set (NOT created by me, pre-existing from S-CTO-5/11/12/18): `playwright-report-scto5/11/12/18/`, `playwright.scto*.config.ts`, `tests/e2e-staging/scto*/`, `semantic-review/`, several `*_EVIDENCE_*.md` and `*_END_SESSION_REPORT_*.md`. These are gitignored-by-nature evidence artifacts but are piling up untracked. **`.auth/*.json` is correctly gitignored** (JWT tokens never committed — verified). **Brain should decide a retention/cleanup policy for stale playwright reports and configs** so the tree stays reviewable (§ refactor-hygiene, only after truth is stable).

### C10. ⚪ My re-run refreshed `.auth/*.json` and `.auth/tokens.json` via global-setup (read-only, expected)
global-setup re-logs-in the free/premium/seller roles and rewrites their storage-state files. These are gitignored, contain JWTs, and were NOT printed anywhere. No secret exposure. Noted only for completeness so the Brain knows the auth files' mtimes changed today.

---

## PART D — WHAT IS TRUE IN BACKEND BUT NOT (RE)BUILT / VERIFIED ELSEWHERE

(Carried from S-CTO-18web Part D — this session did NOT touch api/DB/BO/AWS-write/feeds, so all remain as last known. Re-stated so nothing is buried.)

- **Backend allows photoless job/service** (S-CTO-8, api `bd5cf2a`) — TRUE in source; create returns 201. ✅ Not re-tested this session.
- **Front image-optional form gating** (`useProductFormState.ts:643`) — TRUE in source; unit-tested. Full UI publish of a photoless listing **through the FORM** (not the API) still NOT browser-verified (S-CTO-18web used the API path; S-CTO-12 proved "not image-blocked" only). **Still open.**
- **Search doesn't require a photo** (S-CTO-16) — TRUE for the query predicate, BUT search requires `isVerified=true` + indexed. So "photoless is searchable" is only true AFTER moderation. **Unchanged.**
- **Feeds gate on `isModerated=true`** (source) + likely inner-join image drop (C4) — **NOT runtime-verified.**
- **BO moderation** (`bo_moderation_audit_log`, verify/reject, moderation-reconciliation in `admin_bo_tawadoo`) — **NOT opened/verified this session or S-CTO-18web.** Nobody has confirmed whether the BO moderation queue shows pending photoless listings or whether staging has a working moderator flow.
- **DB truth** (`isModerated`/`isVerified` actual table values) — still **inferred from API responses, never psql-confirmed** (staging DB IP-locked per S-CTO-16). This session did NOT query the DB.
- **AWS** — this session only did **read-only** ECS describe (service/task/digest/health). Did NOT touch feeds infra, OpenSearch (IP-locked), Amplitude, or any write path. Prod cluster `tw-prod-ecs-cluster` NOT touched.

---

## PART E — ERRORS / TOOL ISSUES ENCOUNTERED (even minor)

- **E1 (minor, self-corrected):** First AWS call used wrong cluster name `tw-staging` → `ClusterNotFoundException`. Corrected via `aws ecs list-clusters` → real name `tw-staging-cluster`. No impact.
- **E2 (minor, self-corrected):** `describe-tasks` with full task ARNs failed (`Invalid identifier`); re-ran with bare task IDs → success. No impact.
- **E3 (cosmetic):** macOS `ls --time-style` flag unsupported; irrelevant to result.
- **E4 (observation, not an error):** the S-CTO-18 spec asserts `detailProven` as a **log field, not a hard assertion** — so the detail-page DEFER passes the test suite green while honestly reporting `proven=false`. Correct design (doesn't fake a pass), but the Brain should note that a "4/4 passed" headline does NOT mean the detail render was proven — it means the reachable parts passed and the detail part honestly deferred. **Do not let "4/4 passed" be read as "detail render verified."**

---

## PART F — RECOMMENDED BRAIN QUEUE (candidates — founder authorizes)

1. **S-CTO-20 (api/BO) — Photoless moderation→index→search→feed truth** (THE unblock for C1/C2/C3): map the full state machine; answer who sets `isModerated` vs `isVerified`; confirm staging `ENABLE_CONTENT_MODERATION`; moderate one photoless job + service; then the deployed detail + buyer-card render (this session's DEFERRED proofs) can be verified by re-running `playwright.scto18.config.ts` step B + a search-card assertion — **no web code change needed.**
2. **C4 — Google feed inner-join image drop** (api): confirm photoless eligible listings aren't silently excluded from Merchant/social feeds; reconcile `innerJoinAndSelect('pub.images')` vs `FeedEligibilityQuery` (no image req). Supply/demand impact.
3. **C5 — owner cannot preview own unverified photoless detail** — founder/product decision (should a seller see their own pending listing?).
4. **C6 — `feed_slide_changed` (+ broader Amplitude) lake landing** — fold into S-CTO-15/18 parity work.
5. **C7 — identify `yarn.lock` owner** on Ramzi_V2 (likely AI/Bedrock branch); commit or discard per §28.4.
6. **C8 — staging test-data hygiene**: clean archived/stale E2E photoless listings.
7. **C9 — evidence/playwright-report clutter retention policy** (refactor-hygiene, after truth stable).
8. **Front-FORM photoless publish** (Part D) — browser-verify a photoless listing published through the actual create form (not the API), end-to-end to visible. Still never done.

---

## PART G — STATUS

**Proposed:** `FINISHED — INCOMPLETE (honest PARTIAL)`.
- **My Listings render + no-regression + deployed-digest**: COMPLETE and live-proven (4 configs). ✅
- **Detail + buyer-card render**: shipped/deployed at 8b0a815f but **UNVERIFIED at runtime** (moderation gate) — must NOT be claimed done. Hard soft-dep on **S-CTO-20**.
- **Zero source changed, zero commit, prod untouched, no moderation/api/index/BO touch, no new assets/names, no forbidden files.**
- The photoless supply/demand chain (moderation→search→feeds→BO) remains the real open work — unchanged since S-CTO-18web, correctly NOT entered by this render-lane session.

**Rollback:** nothing to roll back (no change). Underlying commit rollback (if ever) = `git revert 8b0a815f` (additive, fallback-only, no data/migration/infra).

**Evidence artifacts:** `S_CTO_21web_EVIDENCE_2026_09_06.md`, `tawadoo_web_js/playwright-report-scto18/`, `tawadoo_web_js/tests/e2e-staging/scto18/screenshots/` (4 configs × A/B/C), all untracked.

**Next session:** **S-CTO-20 (api/BO)** — moderate a photoless listing to unblock the deferred detail + buyer-card render proofs and map the isModerated/isVerified state machine.
