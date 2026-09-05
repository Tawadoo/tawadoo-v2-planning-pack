# S-CTO-22api — END-OF-SESSION REPORT FOR THE BRAIN

**Session:** S-CTO-22api — Feed photoless-drop fix (F1)
**Date:** 2026-09-06
**Author:** execution session (Kiro), for independent Brain QA + queue planning
**Repo touched:** `tawadoo_api_js` only · **Branch:** `Ramzi_V2` · **Env:** STAGING only
**Companion evidence:** `S_CTO_22api_FEED_PHOTOLESS_FIX_EVIDENCE_2026_09_06.md` (⚠️ contains one wrong claim — corrected in §B1 below)

> Purpose of this document: surface EVERYTHING — including small, out-of-scope, and newly-discovered issues — so nothing is buried. Proposed status is a hypothesis for the Brain to verify from source/live, not an accepted truth (§18, §29.3).

---

## A. WHAT WAS ACTUALLY DONE (proposed status: FINISHED — COMPLETE on the fix; INCOMPLETE on one live check)

### A1. The fix (backend, shipped + deployed)
- **File:** `tawadoo_api_js/src/modules/syndication/services/feed-generator.service.ts`, method `queryActiveCampaignProducts()`.
- **Change:** `.innerJoinAndSelect('pub.images','images')` → `.leftJoinAndSelect('pub.images','images')` (+ explanatory comment). One line of behavior change.
- **Effect:** a publication with **zero image rows** is no longer eliminated by SQL before the existing placeholder logic runs. It now survives, gets the branded placeholder, and reaches the per-channel formatters. Google's real-image + buy_now gates still exclude it; Meta/TikTok/ChatGPT accept it per their rules.
- **New test:** `src/modules/syndication/services/feed-photoless-eligibility.spec.ts` (4 tests, fail-first proven red→green).
- **Commit:** `8d55a207c88a6b125b47316cb361c69d463a3b54` on `Ramzi_V2` (2 files, +180/-1). Pushed. Working tree clean.
- **CI:** run `33982341954` SUCCESS (quality-gate + build-and-push + ECS deploy + wait services-stable).
- **Deployed:** ECS `tw-staging-svc-back`, task-def `:44`, 2/2 running, PRIMARY/COMPLETED, running image digest `sha256:6bf1af70be3c3a06da335e83d4cb0a654dea044a4004ec3ec4f7bbb0f05f52fc`.

### A2. What is PROVEN vs NOT
- ✅ **Backend code:** built, wired, reachable, deployed, running (5-levels: exists→wired→reachable→runtime-verified via spec→…).
- ✅ **Behavioral correctness + guards intact:** runtime fail-first spec against the real function + real `FeedQualityGateService`.
- ✅ **Full syndication module:** 211 tests / 16 suites green (local).
- 🟡 **Live feed-file artifact containing a truly-photoless job/service:** NOT captured (see §B for why, and the correction).
- ⬜ **Frontend (web):** NOT APPLICABLE — this fix is server-side feed generation only; no UI. Nothing was built or needed in `tawadoo_web_js`.
- ⬜ **BO (admin):** NOT APPLICABLE — no BO change. (But see §C4 — BO feed-report endpoint exists and was NOT exercised.)
- ⬜ **DB:** no schema/migration change. DB was NOT directly readable (see §B2).
- ⬜ **AWS:** no infra change. Read-only inspection only.

---

## B. CORRECTIONS TO MY OWN EARLIER EVIDENCE (do not let this be buried)

### B1. ❌ I stated "S3 feeds/ prefix is EMPTY / generation dormant." THIS WAS WRONG.
- I checked `s3://tw-media-storage-preprod/feeds/` — the **code default** bucket. But the **staging task-def `:44` sets `FEED_S3_BUCKET=tw-prod-media-storage-prod`** (a PROD-named bucket). The real staging feeds live there.
- **Truth:** staging feed generation is **ACTIVELY RUNNING on the 6-hour cron**, producing files up to **2026-09-05 19:00** (one minute before my deploy). 39+ objects, all under `feeds/meta/shoes/`.
- Consequence: my companion evidence file's §8 "empty/dormant feeds" boundary reasoning is partly wrong. The correct statement is in §B3.

### B2. Staging DB genuinely unreachable from this machine (this part stands).
- `tw-staging-cluster.cluster-…rds.amazonaws.com` → private VPC `10.0.4.105`, TCP:5432 times out. PROVEN not directly reachable.
- ECS exec IS enabled but `ExecuteCommandAgent` = `TargetNotConnected` on both running tasks. So no live-DB probe without a fresh deploy (not done).

### B3. What the LIVE feed actually shows (inspected this session, read-only)
The single current Meta/shoes feed item (`v1788631205929.json`, 2026-09-05):
```
id: c2e92829-…  title: "Adidas Baskets Homme Pointure 39 Noir Neuf Paris"
image_link: …/assets/placeholder-listing.png   ← PLACEHOLDER (this item is photo-less!)
price: "300000 MAD"   condition: (absent)   custom_label_0: buy_now
```
- This proves the **placeholder path already fires in production-of-feeds** for a photo-less item — BUT this item survived the **pre-fix INNER join**, which means it is NOT a true zero-image-row listing. See §C1 (important nuance).

---

## C. OPEN ITEMS / ISSUES FOUND (for the Brain to queue) — nothing buried

### C1. 🔴 IMPORTANT NUANCE — the F1 bug has TWO shapes; my fix covers one; verify the other is understood
- My fix covers the **truly photo-less** case: a publication with **zero `ta_publication_image` rows** (INNER join drops it). Proven fixed.
- The live shoes item above is photo-less in the FEED (placeholder) yet **was NOT dropped pre-fix** — so it must have **≥1 image row of a BANNED type** (`video`/`map`/`cover`/`thumb`). Path: INNER join succeeds (a row exists) → `qualifyingImages.filter(type==='image')` is empty → placeholder pushed. This is a **different, already-working path**, not the one I fixed.
- **Why the Brain must note this:** the original F1 framing ("photoless job/service dropped") is accurate ONLY for zero-image-row listings. Listings with a non-`image` media row were already passing. A future QA that seeds a test listing must create a listing with **literally no image rows** to exercise my fix — seeding one with a video row would NOT reproduce the pre-fix drop and could produce a false "already worked" conclusion.
- **Not verified from live:** whether real eligible job/service listings on staging/prod tend to have zero image rows vs a video/cover row. Needs a DB read (blocked this session).

### C2. 🟠 Staging writes feeds to a PROD-NAMED bucket (`tw-prod-media-storage-prod`)
- `FEED_S3_BUCKET=tw-prod-media-storage-prod` and `FEED_PUBLIC_BASE_URL=https://tw-prod-media-storage-prod.s3…` on the **staging** task-def `:44`, while `ENV_MODE=STAGING`.
- The feed generator's `link` still points to `https://staging.tawadoo.ma/...` (correct), so items are staging items — but the generated feed FILES are written into a bucket named like prod.
- **Risk to flag:** (a) staging/prod bucket-naming confusion (real ban risk if a prod feed URL is ever registered with a provider pointing at staging-generated content, or vice-versa); (b) my own earlier confusion came directly from this. **Owner:** infra/config. **Action:** confirm intended bucket per env; document it; ensure no provider is ever pointed at a mixed bucket. NOT changed this session (out of scope, config/infra).

### C3. 🟠 `condition` silently dropped from the live Meta feed after ~2026-09-02
- OLD shoes feed (≤2026-09-01) had `condition: "new"` + `custom_label_4`. NEW (≥2026-09-02) has **no `condition` field** and no `custom_label_4`, despite the title saying "Neuf" and description "état: neuf".
- This is consistent with the S-CTO condition-integrity change (`resolveFeedCondition` returns null when the condition PROPERTY isn't mapped, and formatters omit it rather than assert `new`). It is **safe-by-design** (avoids used-as-new misrepresentation).
- **But** a genuinely-new item is now shipping to Meta with **no condition at all** because its `condition` property value isn't being resolved from the listing. That is a **data-completeness gap** (lost a valid attribute), not a safety bug. **For the Brain:** investigate why this shoes listing's "neuf" isn't mapped by `resolveFeedCondition` (property code/value mismatch?) — it may be suppressing `condition` across many legitimately-new listings, weakening feed quality. Owner: syndication/property-mapper. Out of S-CTO-22api scope.

### C4. 🟡 Live feed-artifact for the FIXED path never captured (the one INCOMPLETE gate)
- Only `feeds/meta/shoes` exists on staging — a buy_now retail category. No job/service category feed is being produced (likely no active job/service campaign). So there is currently **no live artifact that exercises my zero-image fix**.
- To close: seed ONE eligible **zero-image** service listing in an active staging campaign, trigger `POST /internal/cron/feed-generation` (guarded by `FEED_TRIGGER_KEY`, Secrets Manager `tw-staging/feed-trigger-key`), then read the resulting `feeds/meta|tiktok|chatgpt/<cat>` files (INCLUDE) and confirm absence from `feeds/google/<cat>` (EXCLUDE). Read `GET /internal/feeds/report` for per-channel counts. No provider push needed.
- I did NOT do this because it requires seeding test data (mutation) + a secret read + confirming the zero-image shape (§C1). Proposed as **next session S-CTO-23api**.

### C5. 🟡 Only Meta feeds present — Google/TikTok/ChatGPT feed files absent on staging
- The bucket has only `feeds/meta/shoes/`. No `feeds/google/…`, `feeds/tiktok/…`, `feeds/chatgpt/…`.
- Feeds only upload when `productCount > 0`. So this means: for the current active set (one shoes item), Google excluded it (placeholder-only → `no_real_image`, correct guard behavior!), and TikTok/ChatGPT produced… unclear (should ChatGPT have included it?). **For the Brain:** verify whether TikTok/ChatGPT SHOULD have a shoes file and don't (possible silent formatter/upload gap), or whether it's expected. Not investigated (would need the run report / DB). This is a **possible pre-existing distribution gap unrelated to F1.**

### C6. 🟢 Pre-existing lint baseline in `feed-generator.service.ts` (not mine)
- 6 eslint errors (unused imports/vars: lines 20,21,30,412,647 — `SyndicationCampaign`, `UtmSource`, `UtmMedium`, `FeedRunReport`, `metaExcluded`, `lastGenerated`). Present in HEAD before my change. The repo `lint` script runs with `--fix` (auto-clears), so CI/local `npm run lint` masks them; a no-`--fix` run surfaces them. Minor hygiene; a `tawadoo-refactor-hygiene` pass could clear. Not mine to touch mid-session (§9).

### C7. 🟢 CI does NOT test the syndication module
- CI quality-gate test pattern is `analytics-ingestion|amplitude|migration` only. Syndication tests (including my new spec) are **locally-certified, not CI-certified**. CI does compile them (`tsc`+`nest build`). **For the Brain:** consider widening the CI test pattern to include `syndication` so future feed regressions are caught in CI. Structural CI gap, not a blocker for this fix.

### C8. 🟢 Legacy public feed path still uses INNER join (out of F1 scope, noted)
- `syndication-feed.service.ts:43` (`GET /public/syndication/feed`, `SyndicationFeedService`) still `innerJoinAndSelect('pub.images')`. It does NOT build the channel feed files (separate legacy read path), so it's out of F1 scope — but it would still drop photo-less listings from that public JSON read. Parity fix candidate.

### C9. 🟢 GHCR registry private — commit→digest not locally re-resolvable
- Could not `docker manifest inspect` GHCR (403, no local creds; did NOT expose `GHCR_PAT`). Provenance still derivable via single-build CI + running-task digest. Noted for completeness.

---

## D. GUARDS / SAFETY CONFIRMATION (ban-prevention)
- Google buy_now-only + real-image guards INTACT (the live shoes placeholder item is correctly ABSENT from any google feed — real evidence, §C5).
- Auction/negotiable, campaign, URL-HEAD, moderation gates untouched.
- No real provider push triggered. Prod untouched. No secret exposed. No new cost/permission/tool/infra. `git add .` not used; main not pushed.

---

## E. RECOMMENDED QUEUE (Brain to authorize; discovery ≠ authorization)
1. **S-CTO-23api (verify F1 live):** seed a zero-image service listing on staging, generate feed, capture INCLUDE (meta/tiktok/chatgpt) + EXCLUDE (google) + no-leak. Closes C4. (Confirm the zero-image shape per C1.)
2. **Investigate C1 shape reality:** do real job/service listings have zero image rows or a video/cover row? DB read. Determines true blast radius of F1.
3. **Investigate C3:** why genuinely-"neuf" listings ship with no `condition` — possible feed-quality regression across many listings.
4. **Config review C2:** staging writing to `tw-prod-media-storage-prod` — confirm intended, document, de-risk provider mis-pointing.
5. **Investigate C5:** missing TikTok/ChatGPT feed files for the active shoes item — expected or silent gap?
6. **CI C7:** add `syndication` to CI test pattern.
7. **Hygiene C6 + parity C8** — low priority, bundle into a refactor-hygiene pass.

## F. PROPOSED STATUS
- **F1 fix:** `FINISHED — COMPLETE` on backend build+deploy+behavioral proof; **the live-artifact QA is `FINISHED — INCOMPLETE`** (C4) — one open verification, owner + follow-up named. Independent Brain QA to accept/downgrade from source+live.
- **Next session ID:** S-CTO-23api — one-line reason: capture the live staging feed artifact proving a zero-image eligible service is included (meta/tiktok/chatgpt) and excluded from google, closing the only open F1 gate.
