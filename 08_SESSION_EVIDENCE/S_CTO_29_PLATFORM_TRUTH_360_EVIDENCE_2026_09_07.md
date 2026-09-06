# S-CTO-29 — PLATFORM TRUTH 360 + FEED-INCIDENT END-SESSION REPORT

**Session:** S-CTO-29-PLATFORM-TRUTH-360
**Date:** 2026-09-06 (evidence dated 2026-09-07 per prompt)
**Type:** READ-ONLY 360 investigation → interrupted by a founder EMERGENCY (feed recall + cron pause), then resumed as end-session report.
**Env:** STAGING only. Prod NOT mutated except the single founder-authorized Meta-feed recall (see §INCIDENT).
**Own-session status proposal:** `FINISHED — INCOMPLETE`. Truth map A–F complete (source+runtime). G (naming table) and H (live/dormant ledger) are delivered here in condensed form. The formal write of A–H section-by-section was cut short by the emergency; this report carries all findings so nothing is lost. **Brain QA owes acceptance + must queue the flagged items below.**

**Skills:** tawadoo-source-truth, tawadoo-data-sovereignty, tawadoo-runtime-provider-qa — discovered + activated at session start.

---

## 0. EXECUTIVE TRUTH (≤30 lines)

1. **Entities:** 60,001 total = **57,356 individual** + **2,645 store**. There is **no `isPremium`/`seller_type` column** — "premium" is DERIVED from an active `ta_syndication_user_subscription`. Only **3 premium subs active** (3 users). Individual-vs-store = `ta_entity.is_store`.
2. **Publications:** 24,975 total (21,451 published / 3,157 draft / 267 archived / 100 deleted-status).
3. **Public visibility** = `isVerified && status='published' && deletedAt IS NULL` → **19,875 live**. **Index/feed eligibility** = `isModerated && published && not-deleted` → **20,033**. The gap = exactly **158** (see I5).
4. **`is_live` (I1): DEAD** — false on all 24,975 rows; no setter, no reader. Live auctions use a separate `ta_publication_live` table.
5. **`active` (I2): near-vestigial** — only set false on hard-delete (100 rows = the 100 deleted-status). Read only by orders + content-engine, NOT by public search/detail.
6. **158 moderated-not-verified published rows (I5):** confirmed live (not soft-deleted), origin = **legacy `BackfillIsModerated` migration (1753372800000)**, NOT any runtime approve/publish (0 have an approve/publish audit event; audit log is sparse — 195 rows total). They sit IN the search index + feed-eligible but are 404 on the page and excluded from search results (not verified) = **"in index/feed but not publicly renderable."**
7. **Moderation vs verification are DECOUPLED by path:** self-publish sets `status=published` and leaves both flags false (pending); the human BO **Verify** (`approvePublication`) sets `isModerated=true` AND `isVerified=true` together. No current setter produces moderated-true/verified-false.
8. **SLOT = an active `ta_syndication_campaign_product` row** under a subscription's `max_live_items`, backed by a live subscription (`source=subscription`) OR a live boost (`source=boost` + `pub.boosted`). **Distribution is GATED behind a purchased package/boost — NOT automatic for every eligible listing** (contradicts the "silently distribute every eligible listing" assumption).
9. **The whole distribution system is called "syndication" in code** (no module named distribution/catalogue/feed). **Only 1 live slot platform-wide.**
10. **Feeds: only Meta was live** (1 feed_config row, category "shoes", 1 product, regenerated every 6h). **Google / TikTok / ChatGPT feeds = 0 rows = DORMANT.** ← reconciles S-CTO-22api.
11. **🔴 INCIDENT:** the feed cron was firing from staging **into the PRODUCTION media bucket** (`tw-prod-media-storage-prod`, tagged env=prod), publicly readable by Meta. This is the "test product cost us a GMC suspension" path. Founder ordered recall + pause. **DONE this session** (see §INCIDENT).
12. **Storefronts:** Classic + Smart both call the same `publications/search` API; Smart is a SEPARATE additive layer (no Classic imports it). MCP live at prod, resolves to the same `/{locale}/p/{slug}` entity as web.
13. **🟠 Sovereignty gaps:** (a) MCP logs interactions to a LOCAL sqlite/JSONL sink — **no write to `ta_analytics_event`** found; (b) `for-you` API has **no server-side lake write** (client impression only). Both may violate Commandment 2 — flagged, not confirmed.
14. **Monetization:** `ta_monetization_config` table **does not exist** (constants hardcoded). Trial service LIVE but **0 trials claimed**. Store-video built + cron-scheduled but **social autopost dry-run gated OFF**. `for-you` + immersive `Feed` = the two main-screen surfaces, both built. Coin orders: **129 all PENDING, 0 PAID** on staging.

---

## 1. FIVE-LINE OPENERS

**Initial:**
```
BRANCH: verifying per repo in preflight
MISSION: read-only truth map of the entire platform; FIX NOTHING
ORDER: reality check → source map → runtime-prove → answer questions → naming table → live/dormant ledger → truth report
BLOCKERS: prod DB UNKNOWN; GHCR read denied; ta_publication_image subqueries time out (bound)
TREE STATE: verifying; writes only evidence file
```
**Corrected (post-preflight):**
```
BRANCH: api Ramzi_V2@8d55a20 · web Ramzi_V2@8b0a815f · bo Ramzi_V2@07b3a84 · mcp Ramzi_V2@d8efb4a (all match prompt) · mobile Ramzi_V2@2814824 · mobile_ui new_design@9d37f3c
MISSION: read-only truth map; FIX NOTHING (later amended by founder emergency: recall Meta + pause feeds, staging)
ORDER: reality check ✓ → source map ✓ → runtime-prove ✓ → questions ✓ → naming → ledger → report
BLOCKERS: prod DB not directly read (used staging read-only RunTask); ta_publication_image bounded OK
TREE STATE: pre-existing dirty work in api/web/bo/mcp (parked S-CTO-26 SSR + evidence docs) — PRESERVED, untouched
```

## 2. REPO / BRANCH PROVENANCE
All HEADs match the prompt exactly: **api** Ramzi_V2 `8d55a20`, **web** Ramzi_V2 `8b0a815f`, **bo** Ramzi_V2 `07b3a84`, **mcp** Ramzi_V2 `d8efb4a`. mobile Ramzi_V2 `2814824`; mobile_ui `new_design 9d37f3c`.
**Dirty work (PRESERVED, untouched):** api has the parked S-CTO-26 SSR gate (`index-eligibility.ts`, `publication.service.ts`, `ssr-public-visibility-gate.spec.ts`); web/mcp have evidence docs + playwright reports; bo has `test-results/`. No repo code was modified this session.

---

## 3. RUNTIME EVIDENCE (staging DB, read-only RunTask, `tw_runtime_b11`, SET TRANSACTION READ ONLY)

Four probe tasks ran on `tw-staging-cluster` / `tw-staging-task-back:44`; sanitized JSON to CloudWatch `/ecs/tw-staging-back`. Temp scripts deleted.

| Metric | Value |
|---|---|
| Entities total / individual / store | 60,001 / 57,356 / 2,645 |
| Entity active=false / deleted / paused | 141 / 0 / 0 |
| Store boosted live | 1 |
| Active premium subscriptions (distinct users) | 3 (3) |
| Subscriptions active / inactive | 3 / 13 |
| Publications total | 24,975 |
| — published / draft / archived / deleted-status | 21,451 / 3,157 / 267 / 100 |
| — type classic / offer / bid | 16,857 / 5,686 / 2,432 |
| is_verified true / false | 22,140 / 2,835 |
| is_moderated true / false | 20,043 / 4,932 |
| **is_live true** | **0 (of 24,975)** → DEAD |
| **active false** | **100** (= the 100 deleted-status; active_false_published=0) |
| Public-visible (verified+published+not-deleted) | **19,875** |
| Index-eligible (moderated+published+not-deleted) | **20,033** |
| **I5 moderated-not-verified — all / published / of-which-soft-deleted** | 161 / **158** / 1 |
| I7 verified-not-moderated — all (published subset all soft-deleted) | 2,258 (1,337 "published" but ALL soft-deleted; live gap = **0**) |
| boosted publications live / all-time | 3 / 3,666 |
| Campaigns (subscription / boost) | 7 (6 / 1) |
| Campaign-products total / active | 10 / **1** |
| **Live slots platform-wide** | **1** |
| feed_config rows (only channel) | 1 (**meta**, category shoes, 1 product, last_gen today 00:00) |
| distribution_asset rows | 7 |
| social_post / google_substore / trial rows | **0 / 0 / 0** |
| store_video rows | 4 |
| wallet / wallet_history(30d) / packages | 59,678 / 48 / 14 |
| coin orders — PENDING / PAID | 129 / **0** |
| Audit log rows (created/updated/published/approved) | 195 (100/51/32/12) |
| ta_publication_image | large (bounded count hit 200k cap; no timeout) |

**Runtime storefront probes:** MCP prod `/sse` = 200 (root 404 expected); web staging `/fr`, `/fr/search`, `/fr/smart` = 200. All LIVE.

---

## 4. SOURCE TRUTH (file:line) — sections A–F

### A. Entity & listing core
- `ta_entity` (`tawadoo_api_js/src/modules/entity/entity.entity.ts`): seller type = **`is_store`** (L34-35) only. Also `active` (L92), `is_paused` (L102), `store_boosted`/`store_boost_expire_at` (L108-112), `deleted_at` (L105). **No isPremium/plan/badge/verified column.**
- Premium derived via `DeliveryConfigService.isPremiumSeller()` (`entity/delivery-config.service.ts:230-241`), set by `SyndicationSubscriptionService` (`syndication/services/syndication-subscription.service.ts:116,158-168`).
- `ta_publication` (`publication/publication.entity.ts`): `status` enum L108 (draft/published/archived/deleted), `isVerified` L131, `isModerated` L143, `isLive` L128, `active` L146, `boosted`/`boostExpireAt` L122-126, `deletedAt` L178. Partial index `idx_publication_eligible` on (entity,isModerated,status) WHERE moderated+published (L39-42).
- Setters/readers: `publication.service.ts` approve `isModerated=isVerified=true` (~L3731-3732), reject both false (~L3416-3419), edit-reset both false (~L540-541); public search filters `status='published'` + `isVerified=true` (multiple: ~L1304-1305, 1487-1488, 2570-2571, 4540). State machine `moderation-state-machine.ts` (states draft/pending/approved/rejected).
- **I4 "active" meanings (≥9):** publication.active (delete tombstone), entity.active (store enabled), store_boost.active, syndication_user_subscription.active (current premium), syndication_campaign_product.active (**the slot flag**), google_substore.active, wallet.active, entity_user.active, images.active.

### B. Moderation → verification → visibility
- Self-publish `PublishPipelineService.publish()` (`publication/publish-pipeline.service.ts`): sets only `status=published`, leaves both flags false; audit `published`/`system`.
- Human verify `verifyPublication → approvePublication` (`publication.service.ts:~3297, 3707`, guard `verify/:id` SecretKeyGuard): both flags true; audit `approved`/`admin`.
- Audit entity `publication-audit-log.entity.ts` (`ta_publication_audit_log`): id, publication_id, action(created/updated/approved/rejected/published/publish_rolled_back), actor_id, actor_type, metadata(jsonb), created_at. **No old/new flag snapshot.**
- **158-row origin:** migration `1753372800000-BackfillIsModerated.ts` set is_moderated=true on published rows without touching is_verified. Runtime confirms 0 of the 158 have an approve/publish audit event.
- **DEV1 owner-preview:** `findBySlug` (`publication.service.ts:755, 828`): non-public listing 404s UNLESS `entityID === publication.entity.id` (JWT-validated) → owner-only preview, **no leak** to guests/others. SSR path `findBySlugForSSR` has **no** owner preview (verified-only). Web mirror `productDetails.tsx:314-317` `isOwner`.

### C. Findability / search / indexing
- Keyword index writer `publication-search.service.ts` (`INDEX_PUBLICATIONS` default `publications_2`; v2 target `publications-v2`). **Population gate = `isIndexEligible` (isModerated)** at `publication-search.service.ts:71` (choke point `reindexOne`).
- Image/vector index `publication-embeddings`(-v2): writer `publication.service.ts` `saveEmbeddingsToIndex` (~L3624), remove `removeEmbeddingFromIndex` (~L4045); population gate ALSO `isIndexEligible` (~L3781 add, ~L1139 remove).
- **Both indexes share the SAME population gate (isModerated)** — no eligibility drift by rule. `isIndexEligible` = moderated + published + not-deleted (`index-eligibility.ts:26-32`). Read-time both filter `isVerified`.
- **The real drift is moderated (in index) vs verified (renderable)** = the 158 rows.
- search-by-image: SEPARATE pipeline (`publication.service.ts:4063`), **login-gated** (`publication.controller.ts:1219` JwtAuthGuard + rate-limit) vs text search open (OptionalJwtAuthGuard). Not wired into HybridSearchService.
- **C3:** `ta_publication_image` bounded count OK (no subquery → no timeout); index health not further probed (see OPEN items).

### D. Slots / boost / distribution / feeds / catalogue
- Entities present: `ta_store_boost`, `ta_publication_boost`, `ta_syndication_campaign`(+_product), `ta_syndication_user_subscription`, `ta_syndication_google_substore`, `ta_syndication_social_post`, `ta_feed_config`, `ta_distribution_asset`, `ta_channel_performance`, `ta_campaign_performance`, `ta_plan_boost_category`, `ta_sponsor_campaign`/`ta_sponsor_event` (display-ads, different subsystem). **`ta_package_boost_usage` = raw-SQL only, no entity.**
- Feed selection `feed-generator.service.ts` `queryActiveCampaignProducts()`: requires active campaign-product + (live subscription OR live boost). Cron `scheduleFeedGeneration()` L156-161, `FEED_REGENERATION_CRON` default `0 */6 * * *`.
- **Slot capture:** `SyndicationCampaignService.createCampaign/activateProduct` (max_live_items quota) + `attachBoostDistribution`. **No auto rotation/swap.**
- Per-channel: Google/Meta/TikTok/ChatGPT formatters exist; runtime shows **only Meta ever generated**.

### E. Three storefronts
- Classic: `productCard.tsx` → `/{locale}/p/{slug}`; search `app/[locale]/search/page.tsx` → `publications/search`; canonical params enumerated there.
- Smart: `components/smart-view/` (~20 files), route `app/[locale]/smart/page.tsx`; `executeSearch` calls same `publications/search`; brain `app/api/ai/guidance/route.ts` → MCP `smart_search` over SSE (`MCP_BASE` default `mcp-staging.tawadoo.ma`). **SEPARATE**: only the smart route + LayoutWrapper bubble + a SiteHeader link reference smart-view; no Classic component imports it.
- MCP `-tawadoo-mcp-/src/client_mcp/server.py`: tools smart_search, get_buyer/seller_guidance, search_vehicles/real_estate/vacation/high_tech, display_recommendations, compare_products, browse_categories, count_listings, get_featured_products. `_build_product_url` (`client.py:445-453`) = `/{locale}/p/{slug}` (same as web). Widget v20 + legacy v2–v19 shims.

### F. Seller types & monetization
- Capability gates: `FREE_TIER_MAX_ITEMS=5` (`monetization/monetization-config.service.ts:41`); premium via active subscription unlocks bulk-upload (`bulk-upload.service.ts:67-72`), own-courier delivery (`delivery-config.service.ts:82-100`), higher `max_live_items`, store-video eligibility.
- `ta_monetization_config` — **no entity** (planned; constants hardcoded).
- Wallet spend `wallet.service.ts:333-393` `debitWithLock` (publish, boost, bid). Coins bought via `ta_coin_package` → `ta_coin_package_order` (Payzone).
- Store-video `syndication/services/store-video-posting.service.ts`: daily cron 05:00; **`SOCIAL_AUTOPOST_ENABLED` gate → DRY-RUN unless 'true'** (env value UNKNOWN).
- for-you: API `publication.controller.ts:1748` `getForYou` (**no server-side lake write**); web `ForYouSection.tsx` (client impression `for_you` only). Second surface = immersive `Feed` (`components/feed/ImmersiveFeed.tsx`).
- Trial `trial/trial.service.ts`: 14-day, TRIAL_PACKAGE_ID=41, synthetic subscription; LIVE but 0 claimed.

---

## 5. G — NAMING RECONCILIATION (condensed)

| Concept | Name suggests | What it TRULY is (source+rows) | LIVE/DORMANT/DEAD | Founder-assumption vs reality |
|---|---|---|---|---|
| `active` (publication) | on/enabled/live | hard-delete tombstone (false only on delete) | near-DEAD | NOT a feed/boost slot (founder's own example) |
| `is_live` | listing is live | unused column; auctions use ta_publication_live | **DEAD** | not the visibility flag |
| `status='published'` | publicly live | "submitted"; needs isVerified too to be visible | LIVE | published ≠ visible |
| `isModerated` | passed moderation | INDEX/FEED eligibility gate | LIVE | ≠ verified; drives findability-population not page render |
| `isVerified` | human-verified | PUBLIC-VISIBILITY gate (page + search) | LIVE | the real "is it live to buyers" flag |
| slot | ad/feed placement | active ta_syndication_campaign_product under max_live_items | LIVE (1 only) | not `active`; is a campaign-product row |
| boost | paid bump | store_boost (entity) OR publication_boost (listing) + pub.boosted flags | LIVE (1 store, 3 pub) | two separate boost systems |
| distribution/syndication | auto push all eligible | GATED behind purchased package/boost | LIVE-but-gated | NOT "silently distribute every eligible listing" |
| campaign | ad campaign | container grouping products per entity+source | LIVE (7) | subscription- or boost-sourced |
| catalogue | product catalogue mgmt | no such module; = "syndication" | n/a | naming gap |
| feed | provider feed | S3 JSON file per (category,channel); only Meta/shoes live | Meta LIVE, others DORMANT | |
| premium (store) | tier/flag | derived from active ta_syndication_user_subscription | LIVE (3) | no isPremium column |
| store vs individual | seller type | ta_entity.is_store boolean | LIVE | no seller_type enum |
| Smart View | separate AI storefront | additive layer over same publications/search API | LIVE | correctly separate from Classic |
| MCP | agent commerce | FastMCP tools resolving to same web entity URL | LIVE (prod) | lake write = LOCAL only (gap) |

## 6. H — LIVE-vs-DORMANT LEDGER (condensed)

**LIVE (rows/runtime prove):** Classic marketplace, Smart View, MCP (prod /sse), keyword+image search, moderation/verify flow, publish pipeline + coin debit, boost (store+publication), **Meta feed only** (6h), wallet, packages, for-you + immersive Feed, trial service (service live), store-video analysis/selection.
**DORMANT (code exists, ~0 real usage):** Google/TikTok/ChatGPT feeds (0 rows), store-video social autopost (dry-run gated), social_post (0), google_substore (0), trials claimed (0), paid coin orders (0 PAID), distribution slots at scale (only 1 active).
**DEAD (nothing reads / redundant):** `ta_publication.is_live` (0 rows, no reader), `ta_publication.active` as a visibility signal (delete-tombstone only).
**UNKNOWN/BLOCKED:** prod DB (not directly read); `SOCIAL_AUTOPOST_ENABLED` env value; whether MCP local business_events are forwarded to the sovereign lake; index alias currently pointing at publications_2 vs -v2; GHCR image read (denied).

---

## 7. 🔴 FEED INCIDENT — what happened + what I did (founder-authorized)

**Discovery (during this session):** the staging feed system was actively firing.
- **How the cron got enabled (CloudTrail):** on **2026-07-31 17:11 UTC**, `kiro-ai` created EventBridge rules `tw-cron-feed-generation` (rate 6h) and `tw-cron-store-video` (05:00) **already in state ENABLED** (`PutRule state=ENABLED`). They were **born live** — no later flip. A **second, independent** scheduler also exists: an **in-app Bull repeatable cron** `generate-all-feeds` registered on every container boot (`feed-generator.service.ts:156-161`, via `bull-cron-helper.ts`). This is why disabling EventBridge alone does NOT fully stop feeds.
- **Where it wrote:** `ta_feed_config` public_url = **`tw-prod-media-storage-prod`** (bucket tags: `env=prod`, `projet=tawadoo-prod`) → **staging wrote its Meta feed into the PRODUCTION media bucket**, publicly readable by Meta. ~40 six-hourly versions since 2026-08-23. This is consistent with "the test product cost us a GMC suspension."

**Actions taken (founder gave explicit, repeated authorization):**
1. **Disabled** EventBridge `tw-cron-feed-generation` and `tw-cron-store-video` (reversible: `aws events enable-rule --name <rule>`). `tawadoo-content-engine-daily` was already disabled. Left `tw-cron-boost-expiry` and `tw-cron-subscription-expiry` ENABLED (internal state, no provider firing).
2. **Recalled the Meta product:** backed up the live file to `s3://tw-prod-media-storage-prod/feeds/_recall_backup_S29/v1788652807639.json` (1,146 bytes, ETag `744236…`), then overwrote `feeds/meta/shoes/v1788652807639.json` with `[]` (empty array, correct feed format, `cache-control: no-cache`). Verified: public URL now returns `[]`. Meta drops the products on next refresh. **Engine untouched — the app can regenerate any time.**
3. Confirmed **no other feed files exist** (google/tiktok/chatgpt/meta_home/meta_vehicles = 0 objects). Nothing to recall there. **Prod MCP/ChatGPT NOT touched.**

**One-command reverse (for prod go-live):**
`aws s3 cp s3://tw-prod-media-storage-prod/feeds/_recall_backup_S29/v1788652807639.json s3://tw-prod-media-storage-prod/feeds/meta/shoes/v1788652807639.json` and re-enable the two rules.

---

## 8. 🚩 OPEN / INCOMPLETE / ERRORS — for Brain to queue (nothing buried)

**P0 — feed durable pause (Brain owns; founder said "one-click reverse for prod"):**
- The EventBridge disable is only a stopgap. The **in-app Bull cron `generate-all-feeds` still exists in Redis and re-registers on every ECS boot** (`feed-generator.service.ts:156`). It was NOT touched (founder said no deletes; make it a proper Brain task). Until a durable env/gate is added, a container restart could resume feed writes. **Recommend a `FEEDS_ENABLED`/no-op-cron env gate wired to a single toggle.**
- **ARCHITECTURE DEFECT:** staging writes to the **prod** media bucket `tw-prod-media-storage-prod`. Staging should use a staging bucket. This crosses the env boundary and is how staging test data reached Google/Meta. Needs a dedicated remediation session.
- `channel-metrics-sync` `@Cron` 03:00 and Meta Commerce daily sync (piggybacks on feed run) — verify they can't fire independently once feeds are gated.

**P1 — sovereignty (Commandment 2) gaps, flagged not confirmed:**
- **MCP** logs interactions to a LOCAL sqlite/JSONL sink (`observability.py`); **no write to `ta_analytics_event`** found in the MCP repo. Either a forwarder exists elsewhere (unverified) or MCP interactions bypass the sovereign lake. Investigate.
- **`for-you` API** (`getForYou`) has **no server-side lake write** — only a client-side `for_you` impression. Server-side sovereignty event may be missing.

**P2 — data / naming truth:**
- **158 moderated-not-verified live rows** (I5): decision-ready. They're feed/index-eligible but not publicly renderable. Brain/founder to decide: verify them, or reset is_moderated. (No fix taken — read-only.)
- **`is_live` column is DEAD** and **`active` is a near-vestigial delete-tombstone** — candidates for documentation/cleanup (NOT this session).
- The reverse set (verified-not-moderated) is entirely soft-deleted → no live findability gap, but the "published" status is not reset on soft-delete (status='published' persists on deleted rows) — a data-hygiene note.

**P3 — coverage NOT completed this session (interrupted by emergency):**
- `ta_publication_image` index HEALTH (beyond existence/bounded count) not fully probed — C3 partially open.
- Index alias runtime target (`publications_2` vs `publications-v2`) UNKNOWN from source.
- Store-video `SOCIAL_AUTOPOST_ENABLED` env value UNVERIFIED (source says dry-run default).
- Prod DB never directly read (used staging) — prod parity of these findings UNKNOWN.
- The formal per-section A–H evidence write was cut short; this report is the consolidated substitute. A follow-up may re-issue the clean A–H doc if Brain wants it.

**Errors/anomalies encountered (logged so they're not lost):**
- Several probe queries hit schema mismatches (`status` column absent on ta_distribution_asset/ta_store_video; coin enum is UPPERCASE `PAID`; `deleted_at` is camelCase `"deletedAt"`) — all resolved by follow-up probes; noted so future probes use correct column names.
- One recall command was interrupted mid-run (timeout, exit -1) and landed nothing; re-verified state before retrying (no partial write). Also corrected the empty-feed format from `{"products":[]}` to `[]` after reading the real file shape.
- GHCR image read denied earlier (out of scope; noted).

---

## 9. WHAT WAS BUILT IN BACKEND BUT NOT (VERIFIED) ELSEWHERE

| Capability | Backend | Frontend / BO / other | Verified live? |
|---|---|---|---|
| Distribution/syndication (feeds, campaigns, slots) | BUILT (syndication module) | Web shows DistributionChannelsSection to seller; BO? unverified | Only Meta feed live; 1 slot |
| Trial (14-day) | BUILT + cron + notifications | Web CTA? unverified | 0 claimed |
| Store-video (analysis + social posting) | BUILT + cron | Upload UI? unverified; autopost dry-run | 4 videos, autopost OFF |
| for-you personalization | BUILT (API) | Web ForYouSection + immersive Feed BUILT | live; **no server lake write** |
| MCP agent commerce | BUILT (FastMCP, prod live) | ChatGPT widget v20 | /sse 200; lake write local-only |
| Monetization config (BO-editable) | NOT built (`ta_monetization_config` absent; hardcoded) | BO editor not possible yet | n/a |
| google_substore / social_post | tables + code BUILT | — | 0 rows (dormant) |
| Coin purchase (Payzone) | BUILT | Web pricing/coins UI present | 0 PAID orders on staging |

---

## 10. CHANGES MADE THIS SESSION (full disclosure)
- **AWS (staging control-plane):** disabled EventBridge rules `tw-cron-feed-generation`, `tw-cron-store-video`.
- **S3 (prod media bucket, founder-authorized recall):** created backup object `feeds/_recall_backup_S29/v1788652807639.json`; overwrote `feeds/meta/shoes/v1788652807639.json` with `[]`.
- **No repo code, no DB rows, no migrations, no deploys.** Temp probe scripts under /tmp deleted.
- Evidence file: this document.

**COMPLETION CHECKLIST:** Preflight+skills ✓ · A ✓ · B ✓ · C ✓(C3 partial) · D ✓ · E ✓ · F ✓ · G ✓(condensed) · H ✓(condensed) · I1 DEAD ✓ · I2 tombstone ✓ · I5 158 origin ✓ · I7 no-live-gap ✓ · DEV1 no-leak ✓ · slots/catalogue-live ✓ · Smart-separate ✓ · MCP-consistency ✓ · premium ✓ · Feed incident recall+pause ✓ · Zero repo/DB changes ✓ · Own-session status: **FINISHED — INCOMPLETE** (P0–P3 open for Brain).
