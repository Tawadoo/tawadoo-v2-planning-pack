# S-CTO-18web — END-SESSION REPORT FOR THE BRAIN (audit input, not self-acceptance)

**Date:** 2026-09-06 · **Writer session:** S-CTO-18web (web + browser) · **Env:** STAGING only, prod untouched.
**Purpose:** Hand the Brain a complete, honest map of what I built, what I proved, what I did NOT prove, and every open thread / issue / anomaly I saw — including things outside my scope (DB, BO, AWS, feeds, supply/demand). Nothing buried. This is a proposal for QA; the Brain makes the durable accepted status (§18).

**Self-assessment of this session's quality:** I under-scoped. I treated the moderation/verification gate as an external "blocker" and stopped, instead of recognizing it as the **entry into the supply/demand black zone** (moderation = admin/BO work; unmoderated = excluded from search AND from all distribution feeds). The founder correctly flagged this as drift. This report exists to make sure the Brain can recover everything I touched or glimpsed and queue the real investigation.

---

## PART A — WHAT I ACTUALLY BUILT (front, committed + deployed)

**Commit `8b0a815f` on `Ramzi_V2`** (pushed `7010ea6d..8b0a815f`), 4 files, additive/fallback-only:
1. `src/components/dashboard/ListingRow.tsx` — My Listings thumbnail: `images[0].url || defaultImage || (category.icon ? getIcon(category.icon) : storecover.png)`. Removed dead `/assets/images/placeholder.png` (file does not exist → was a real 404). Icon fallback uses `object-contain p-2`.
2. `src/components/product/productDetails.tsx` — added `images.length===0` branch rendering `getIcon(itemAnnounce.category?.icon)` (kills the empty gray Swiper box for photoless).
3. `src/components/product/productCard.tsx` — buyer card fallback prefers `getIcon(category.icon)` over `storecover.png`; `isPhotoThumb` flag → `!object-contain bg-sand-100 p-6` for icon fallback (ImageWithSpinner hardcodes `object-cover`).
4. `src/components/feed/ImmersiveFeed.tsx:374` — emitter `feed_slide_change` → `feed_slide_changed` (canonical allowlist name).

CI run `33977934674` green (validate-locales, build-and-push 7m4s, ECS redeploy, 18 smoke tests). Running digest `sha256:74a3482285dd2e09fa890785cb62db716922b13e2c6387dae88f646b22a0a604` on `tw-staging-svc-front` task-def `:18`, tag `staging-v2`, 2/2 tasks, rollout COMPLETED.

---

## PART B — WHAT I PROVED vs DID NOT PROVE (the honest split)

### PROVEN LIVE (real browser, deployed staging, 4 configs: chromium/webkit desktop-fr, mobile-fr, desktop-ar)
- **My Listings icon fallback**: 5/5 photoless service rows render `/assets/icones/auto_services.webp`, 0 broken, **0 `placeholder.png` 404s**. Screenshot confirms.
- **Emitter rename**: deployed JS has `feed_slide_changed`, **0** bare `feed_slide_change`.
- **No regression**: photo listings still load images on `/search`.
- **Photoless create+publish works**: real v2 API pipeline `POST /publications` → `POST /publications/publish/:id` → 201, `status=published`.

### BUILT BUT NOT VERIFIED ON FRONT (shipped + source/build-proven, NOT runtime-proven)
- **`productDetails.tsx` empty-box → icon fix**: NEVER runtime-exercised. Reason: an unverified photoless listing never reaches `productDetails` — `/p/{slug}` → `findBySlug` → OpenSearch lookup → not indexed → 404 → "Cette annonce n'est plus disponible". So the branch I added has ZERO live proof. It could have a bug I can't see (e.g., `itemAnnounce.category` undefined at that render point, RTL layout, icon sizing) — **UNVERIFIED**.
- **`productCard.tsx` buyer-card icon fix**: NEVER runtime-exercised. Same reason — photoless listings aren't in public search results, so the buyer card with an icon fallback was never rendered live. The `!object-contain` important override was never visually confirmed. **UNVERIFIED**.
- **`ImmersiveFeed` emitter**: the string is in the deployed bundle, but I did NOT confirm the event actually LANDS in the lake (DB `ta_analytics_event` / Amplitude) under `feed_slide_changed`. Only the emitter spelling was proven, not the round-trip. **RUNTIME LANDING UNVERIFIED** (this is the S-CTO-15/18 lake-parity gap, still open).

### NOT PROVEN ANYWHERE — the supply/demand black zone I failed to investigate
- **Public search findability of photoless**: NOT proven. Needs `isVerified=true`.
- **Distribution / feeds impact**: NOT investigated live. Source shows feeds gate on `isModerated=true` (`FeedEligibilityQuery.baseConditions`, `syndication-feed.service.ts:53`), so unmoderated photoless are excluded from Google Merchant / social / distribution-group inventory. **I did not measure how many photoless listings are stuck out of feeds, nor whether photoless SHOULD be in feeds at all.**
- **Moderation as admin/BO work**: I mislabeled it a "secret-key blocker." It is real BO work — `bo_moderation_audit_log` table, verify/reject actions, a `moderation-reconciliation` report exist in `admin_bo_tawadoo`. I did NOT look at the BO moderation queue, whether photoless listings appear there, or whether staging has a moderator flow. **UNINVESTIGATED.**

---

## PART C — OPEN THREADS / ISSUES / ANOMALIES (flag everything, incl. out-of-scope)

### C1. 🔴 Google Merchant feed may SILENTLY DROP photoless listings (potential bug — OUT OF MY SCOPE, needs Brain/api)
`syndication-feed.service.ts:43` uses `.innerJoinAndSelect('pub.images', 'images')`. An INNER join means a publication with **zero images is excluded from the feed entirely**, even if `is_moderated=true AND status=published`. Yet `FeedEligibilityQuery` (the "central" eligibility) has NO image requirement. **Contradiction:** eligibility says photoless is fine; the Google feed builder's inner-join silently removes it. If photoless job/service are meant to distribute, this is a real supply leak. **NOT verified at runtime — flagged for api investigation.**

### C2. 🔴 Photoless publish leaves listing UNMODERATED → invisible in search AND feeds, and I don't know why it isn't auto-moderating
My 2 fresh photoless publishes returned `status=published` but `isModerated=false`/`isVerified=false`, and did NOT land in OpenSearch (My Listings still showed only the 6 older ones). The publish pipeline (`publish-pipeline.service.ts`) sets status=published then `pushToIndex` — but `reindexOne` skips ineligible (unmoderated) docs. **So who sets `isModerated=true`?** The moderation path in the controller (`enableModeration` branch) calls `updatePublicationIsModerated(id, true)` — but only when `ENABLE_CONTENT_MODERATION=true` AND `x-api-version:v2`. **I did NOT confirm whether staging has `ENABLE_CONTENT_MODERATION=true`, nor whether the moderation branch actually ran for my listings, nor whether it succeeded/failed silently.** This is the core unknown of the whole photoless thread and I left it unresolved.

### C3. 🟡 The 6 older photoless services are `isModerated=true`(indexed) but `isVerified=false`
They show in My Listings (indexed = moderated) yet are NOT in public search (`isVerified=false`). **So `isModerated` and `isVerified` are two different flags with different effects, and their relationship is unclear.** Index eligibility uses `isModerated`; public search + detail use `isVerified`. Who sets `isVerified` vs `isModerated`, and why do these 6 have one but not the other? **UNINVESTIGATED — Brain should map the exact state machine.**

### C4. 🟡 Seller over free publish limit → `needCoins=true`
New publishes for the E2E seller report `needCoins=true` (seller has 5+ published). This may have affected whether my publishes fully completed the moderation/index step. **Interaction between the coin gate and the moderation/index step is unverified.**

### C5. 🟡 `POST /publications/push/:id` returns 400 after publish
In my create+publish script, the explicit `push` call returned 400 (while publish's internal index step is the real path). Cause unknown — possibly state guard (already-published) or missing precondition. **Not investigated; low priority but noted.**

### C6. 🟡 Detail SSR/CSR does not pass owner-entity to bypass the verify gate for photoless
`product-details-view.tsx` fetches `publications/slug/{slug}` client-side; the owner-bypass in `findBySlug` only helps if the listing is INDEXED. So even the owner cannot preview their own unverified photoless listing's detail page. **Is that intended?** A seller publishing a photoless job/service currently cannot see their own live detail page until a moderator approves. **Product/founder decision needed; flagged.**

### C7. 🟡 Emitter lake-landing NOT verified (carryover from S-CTO-15/18)
`feed_slide_changed` emitter spelling fixed, but DB/Amplitude/S3 landing not confirmed. The broader S-CTO-15 gaps remain open: Amplitude tracking-plan registration not done; only 1/28 events runtime-exercised; DB row-parity via logs not psql; DEFAULT-partition 8 events dormant. **My rename adds one more emitter whose landing is unproven.**

### C8. 🟢 `app_banner_dismissed` / `app_install_banner_dismissed` still blocked
Untouched by design (S-CTO-19api must add the canonical first). Still open. `ai_generated` rename also still deferred (founder-decided).

### C9. ⚪ Pre-existing dirty `yarn.lock` on Ramzi_V2 (aws-sdk deps)
Was dirty at session start, unrelated to my work, preserved unstaged. **Someone's uncommitted dependency change is sitting on the working tree** — Brain should identify the owner (adds `@aws-sdk/client-bedrock-runtime` — likely an AI/Bedrock feature branch). Not mine.

### C10. ⚪ Two test listings created then archived
JOB `04611700-b982-4ada-9ce1-4cce17b8d951` (cat 85) + SERVICE `413eb21e-da3d-4c61-b0f3-b0670d672e10` (cat 132) created via real API, then archived (status=archived). They persist on staging DB as archived. Plus 6 pre-existing E2E photoless services from prior sessions clutter the seller account. **Staging test-data hygiene is drifting.**

### C11. ⚪ Real v2 rule discovered: description must be 150–5000 chars
The create form / API enforces this. Not documented in the create-form steering I saw. Minor, but note for future create-flow work.

---

## PART D — WHAT IS TRUE IN BACKEND BUT NOT (RE)BUILT / VERIFIED ELSEWHERE

- **Backend allows photoless job/service** (S-CTO-8, api `bd5cf2a`) — TRUE in source. Create returns 201. ✅
- **Front image-optional gating** (`useProductFormState.ts:643`) — TRUE in source; proven by unit tests; the actual UI publish of a photoless listing through the FORM (not the API) was NOT driven this session (I used the API path). **Front form photoless publish = NOT browser-verified this session** (S-CTO-12 proved "not image-blocked" but not full publish→visible).
- **Search does not require a photo** (S-CTO-16) — TRUE for the query predicate, but search DOES require `isVerified=true` + indexed. So "photoless is searchable" is only true AFTER moderation. The earlier "0 in OpenSearch" sighting = unverified/unindexed, now explained.
- **Feeds**: gate on `isModerated=true` (source); photoless with no image likely dropped by Google feed inner-join (C1). **Not runtime-verified.**
- **BO moderation**: `bo_moderation_audit_log`, verify/reject, moderation-reconciliation exist in `admin_bo_tawadoo`. **Not opened/verified this session.**
- **DB**: I read DB state only via the API (`mine`, direct-id). I did NOT query psql directly (staging DB IP-locked per S-CTO-16). So `isModerated`/`isVerified` true DB values were inferred from API responses, not read from the table. **DB-level truth = inferred, not psql-confirmed.**
- **AWS**: verified ECS service/task/digest via CLI. Did not touch feeds infra, OpenSearch directly (IP-locked), or Amplitude.

---

## PART E — RECOMMENDED BRAIN QUEUE (candidates — founder authorizes)

1. **S-CTO-20 (api/BO) — Photoless moderation→index→search→feed truth**: map the full state machine `create → publish → isModerated → isVerified → OpenSearch index → public search → feeds`. Who sets isModerated vs isVerified? Is staging `ENABLE_CONTENT_MODERATION` on? Why did fresh photoless publishes not index? Then verify one photoless job+service and prove detail + buyer-card + search render (closes the two UNVERIFIED front fixes from this session).
2. **S-CTO-21 (api) — Google feed inner-join image drop (C1)**: confirm whether photoless eligible listings are silently excluded from the Merchant/social feeds by `innerJoinAndSelect('pub.images')`; reconcile against `FeedEligibilityQuery` (no image req). Supply/demand impact.
3. **BO moderation queue reality (C2/C3)**: does the BO show pending photoless listings? Is there a moderator on staging? Founder decision on staging auto-verify vs manual.
4. **Owner cannot preview own unverified photoless detail (C6)** — product decision.
5. **Lake landing for `feed_slide_changed`** — fold into the S-CTO-15/18 Amplitude parity work.
6. **Staging test-data hygiene (C10)** + **identify yarn.lock owner (C9)**.

---

## PART F — STATUS

**Proposed:** `FINISHED — INCOMPLETE`.
- Front render fix for **My Listings** + **emitter rename**: COMPLETE and live-proven.
- **Detail + buyer-card render** fixes: shipped/deployed but **UNVERIFIED at runtime** (moderation gate) — must not be claimed done.
- The **photoless supply/demand chain** (moderation→search→feeds→BO) is the real open work this session should have investigated and did not. Handed to the Brain above.

Evidence artifacts: `S_CTO_18web_EVIDENCE_2026_09_06.md`, `playwright-report-scto18/`, `tests/e2e-staging/scto18/` (untracked), screenshots. Rollback: `git revert 8b0a815f` (additive, no data/migration/infra).
