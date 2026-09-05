# TAWADOO V2 — COMPLETE MASTER PLAN (for ChatGPT co-planning)
**Author:** Kiro Brain B23 (CTO). **Date:** 2026-09-06. **Purpose:** give ChatGPT the FULL, source-true state of the Tawadoo V2 program from Brain B16 → now, so it can help plan the rest: what to build, how, when, and how to refactor — the moment the in-flight NAV-integrity audit lands. Nothing summarized away. Every claim is source/live-verified by prior sessions or tagged UNVERIFIED/UNKNOWN.

---
## 0. HOW TO READ THIS + HOW WE OPERATE
- **Founder = Ramzi:** non-technical, final authority, does NO technical work. Kiro (me) executes end-to-end and drives the sequence; Ramzi decides business/policy and authorizes each unit.
- **North Star (never changes unless Ramzi says so):** transform the EXISTING Tawadoo V2 codebase into clean, coherent, maintainable, **human-engineered** code — preserving working behavior, no rewrites-for-elegance, no regressions, no AI fingerprints. Customer-facing fixes/features are **subordinate leverage** inside this mission.
- **Order (the roadmap):** all-functional (staging) → **security gate** → **pre-refactor crash-proof architecture RE-REVIEW (the gate)** → **structural refactor (5 layers)** → **sync/rebuild app + MCP** → **prod cutover**. Prod is protected until Ramzi opens go-live. Staging = future prod (days away).
- **`Ramzi_V2` is the only truth branch on every repo.** Staging is built from it.
- **Machine limit:** max 2 concurrent Kiro sessions (3 crashes the machine). Prefer 1 writer + 1 reader; never 2 browser sweeps.
- **THE 4 GOLDEN TRUTHS (govern all planning):** NOTICE ≠ ROADMAP · INVENTORY ≠ INSTRUCTION · BUILT ≠ LIVE · DEFERRED ≠ REJECTED. (+ INVESTIGATION ≠ IMPLEMENTATION · HISTORY ≠ CURRENT STATE.)
- **"Built" has 5+ levels:** exists → wired → reachable → runtime-verified → journey-accepted. A checkbox/report proves none alone. Source truth ≠ runtime truth.
- **Repos:** `tawadoo_web_js` (Next.js web) · `tawadoo_api_js` (NestJS API) · `admin_bo_tawadoo` (AdminJS BO) · `-tawadoo-mcp-` (ChatGPT/MCP, live at mcp.tawadoo.ma) · `tawadoo_mobile_app` + `tawadoo_app_mobile_ui_only` (mobile, ~7 weeks stale).
- **Infra:** AWS eu-west-1, acct 438465169079, ECS `tw-staging-cluster`, Aurora PostgreSQL (in-VPC, IP-locked — direct psql usually BLOCKED; ECS-Exec agents down), OpenSearch (`publications-v2` keyword + `publication-embeddings-v2` kNN). Prod = LIVE legacy stack (~57,324 users / ~21,434 published), the new analytics/intelligence platform lives ONLY on staging and activates at cutover.

---
## 1. LIVE STATE RIGHT NOW (2026-09-06, git-verified)
- HEADs, all `Ramzi_V2`, synced: web `8b0a815f` · api `8d55a20` · bo `07b3a84` · mcp `d8efb4a`.
- **IN FLIGHT (1):** **S-CTO-25-NAV-MAP** — the systemic entity-addressing & navigation-integrity audit (read-only, Stage 1 of the NAV refactor). See §6. Everything else is parked behind it.
- Progress estimate (honest): codebase-refactored ~20% · **structural refactor ~5–10% (barely started — only the S141 schema foundation exists)** · whole-program-to-prod ~45–50%. Commerce spine is solid; the real remaining distance is the intelligence/moat + the structural refactor.

---
## 2. THE REFACTOR JOURNEY B16 → NOW (what actually happened, in order)
This is the spine ChatGPT must understand — the refactor was DESIGNED, its foundation laid, then the program went into a "finish functional first" sub-phase.
1. **S139** — reverted a float-precision change (money-safety).
2. **S140** — prod baseline audit (`B16_S140_PROD_BASELINE_AUDIT`) — captured what legacy prod actually is.
3. **Architecture discovery** — `TAWADOO_V2_PRE_ARCHITECTURE_TRUTH_REPORT.md` (closed the discovery).
4. **Architecture synthesis + FOUNDER-APPROVED DESIGN (2026-09-01):** `TAWADOO_V2_TARGET_ARCHITECTURE.md` = **5-layer modular monolith** (NOT microservices), money stays in ONE tight Core (locks + shared transactions — verified from code, do NOT distribute), sovereign DB→lake, providers behind adapters. Plus `AD-001_SCHEMA_OWNERSHIP_DETAILED_PLAN.md` + the synthesis mandate + `TAWADOO_V2_FRONTEND_ARCHITECTURE.md`.
5. **S141 — schema-ownership foundation (DONE on staging):** split the runtime DB user from the migrator; CLOSED the S138 boot-DDL crash class (runtime user can't run DDL — §35). This is the ONLY structural-refactor piece actually built so far.
6. **Steering swap** — installed the current laws: `00-EXECUTION-PROMPT-NON-REGRESSION-LAW` (§0–§52), `01-B14-CTO-STANDARD` (3 Commandments), `02-REFACTOR-PROGRAM`.
7. **FACE-001 convergence audit → LOCKED DECISION C:** Classic View and Smart View remain **SEPARATE implementations, shared design foundation only.** RED LINE: never import Smart's `smart-view.css`/`SmartProductCard` into Classic.
8. **AI-listing cluster FIX-001..015** (create-listing journey) — code-level fixes.
9. **B16→B21 lineage** ran investigations + functional fixes (analytics, corpus, create-publish, prediction crash-proofing). B20 broke the chain (retired). B21→B22 handoff. **B22 did a full reconciliation** (`B22_MASTER_RECONCILIATION_2026_09_05.md`) to recover lost founder intent. **B22→B23 handoff** (`HANDOFF_B22_CTO_2026_09_06.md`) = current lineage.
10. **B23 (now):** running the CTO operating system (`TAWADOO_V2_CTO_OPERATING_SYSTEM.md`), the master queue, business truth; fired the NAV-integrity audit as the elevated priority unit.

**KEY REFRAME (founder was right):** the huge event/funnel backlog is mostly **WIRE + PROVE existing flows**, not build-from-scratch. Much of "not landing" = flows never exercised in staging + buttons missing a fire-event.

---
## 3. WHAT IS DONE + VERIFIED (frozen — do NOT reopen or re-plan)
**Foundation:** S139 float revert · S140 prod baseline · architecture design on paper (TARGET_ARCHITECTURE + AD-001 + FRONTEND) · **S141 schema-ownership** · steering swap · FACE-001 (decision C) · QA seller fixtures.
**Commerce + create-listing:** AI-listing FIX-001..015 · O1 create→pay→publish→approve→searchable happy path proven live · **S-CTO-3 money-event identity fix** (`ab37f4b` — all ~18 money emitters stamp the acting user_id, proven live in DB + Amplitude) · **S-CTO-5 video upload+record** (no dup, proven Chromium+WebKit+AR-RTL) · **S-CTO-8 job/service image-optional server** (`ff5c66c`+`673eac0`) · **S-CTO-9 auth security** (`a400393` — rate-limit header-spoof CLOSED) · **S-CTO-11 draft/autosave** (`780bdb58` — product-form-v2 now clean, P1 resolved) · **S-CTO-12 job/service web** (`7010ea6d` — seller side, image-optional form+edit).
**Analytics/moat data foundation:** cohort crash fault-isolation + Amplitude system-delivery + mcp/whatsapp→lake (`c64739a`) · **CORPUS-JOIN P1/P1B/P2** (session-sequence corpus in S3, interaction_id join key threaded + live-proven) · prediction-enrichment crash-proofing (`80de624`).
**Events:** **S-CTO-14 event map** (492 events = 157 LIVE + 128 WIRED-NOT-LANDING + 207 PAPER) · **S-CTO-13a naming reconciliation** · **S-CTO-15 allowlist ADD + reject-unknown build** (`bd5cf2a`) · **S-CTO-19api allowlist add** (`50cfa12`).
**Distribution/moderation:** **S-CTO-22api feed photoless-drop fix** (`8d55a20` — leftJoin, shape-1) · **S-CTO-18web category-icon fallback** (`8b0a815f`) · **S-CTO-20 moderation-360 map** · **S-CTO-23qa moderation→visibility runtime QA ✅ ACCEPTED** (human-in-the-loop chain proven live: publish→unverified=invisible → founder BO Verify → visible).
**Security foundations (earlier):** Payzone signature verify · MCP security closure · private-message no-store+CSP · BO RBAC 80-item matrix (source).
**FR diagnosis:** **S-CTO-24** root-caused all 11 frontend regressions from source (read-only).

---
## 4. THE CUSTOMER-FACING / STABILIZATION LANDSCAPE (the product truth — NOT the roadmap, informs it)

### 4A. THE 11 FRONTEND REGRESSIONS (founder-reported LIVE w/ screenshots; DESKTOP broken / MOBILE fine — never break mobile; prod-main = the GOOD reference)
Root-caused by S-CTO-24. **NOTE: FR-9, FR-11, and the "item unavailable" class are now folded into the NAV-integrity refactor (§6) — they are symptoms of the addressing defect, not standalone fixes.**
- **FR-1 PINK RECTANGLE → REMOVE ENTIRELY** (founder doesn't recognize it; strong candidate `LiveAuctionStrip.tsx` rose/purple gradient — confirm before deleting). Addressing-independent.
- **FR-2 AUCTION CARD:** a not-started auction must be HIDDEN unless the user can SUBSCRIBE to notify-on-start (email + in-app). **Unbuilt end-to-end** — no endpoint, no DB entity, no start trigger, no UI (only FCM/email/in-app channels exist). = feature build.
- **FR-3 PAGINATION dead** (page 1→2 across the board). `search-results-view.tsx` `handlePageChange` + `Pagination` gated on `totalPages>=2`. Addressing-independent.
- **FR-4 CHAT/MESSAGES dead.** Known-fragile (rebuilt +5 fixes) — needs a live E2E guard; verify socket connect + ALB WebSocket upgrade (UNVERIFIED).
- **FR-5 DESKTOP DETAIL LAYOUT:** huge empty space + HORIZONTAL "grow your business" strip; prod-main = VERTICAL dual side AD banners. Desktop only.
- **FR-6 PRODUCT CARDS ugly vs prod-main;** restore prod-main card style + tags (negotiable/auction) + vertical side banners.
- **FR-7 SAVE-SEARCH renders everywhere** (even bare category). `search-results-view.tsx:371` `SAVEABLE_SEARCH_PARAMS` includes `cat/scat/type/store` → gate opens without a real search. Require real search (location city/lat/lng OR search_text OR real filter). Surgical. Do NOT disturb the S4 save-search email-consent.
- **FR-8 NOTIF BADGE STUCK** after viewing. `SiteHeader.tsx:39` reads store; **mark-all-read endpoint does NOT exist in api (per-item only) → must be BUILT.** Addressing-adjacent.
- **FR-9 NOTIF DEEPLINK dead-ends `/notification-unavailable`** — api has a deeplink helper but omits service/objectId + web handler has no lead branch. **→ folded into NAV-integrity (it's the poster child of the addressing defect).**
- **FR-10 LEAD-UNLOCK TRANSPARENCY:** api COMPUTES how paid (free allowance vs 10 coins) + balance, but the CONTROLLER DROPS it → front can't show truth. Return it + build the confirm UI (cost X / free-from-package / N left / balance). Lead model: free/bonus quota FIRST else 10 coins (`leads.service.ts:402-409`, `LEAD_UNLOCK_COST`).
- **FR-11 MARK-LEAD-STATUS FAILS** ("Échec de la mise à jour"). `PATCH /leads/:id/status` (`leads.controller.ts:527`). Root-cause candidates (source-verified): entity-id scope miss (`seller_entity_id`), `markAsContacted` requires unlock first (throws Forbidden), `markAsViewed` only transitions from NEW. **→ overlaps NAV (entity resolution/authz) — re-scope after NAV-FIX.**

**Fix order (post-NAV):** functional (FR-3 pagination, FR-8 badge, FR-11, FR-10) → remove/surgical (FR-1, FR-7) → chat (FR-4, own careful session) → visual (FR-5 layout + FR-6 cards) → FR-2 auction subscribe-notify (feature).

### 4B. AUTH FLOW (RED, cost-sensitive) — A-FLOW1
Founder rule: on identifier entry, `check-identifier` BRANCHES → LOGIN (existing+password, **NO OTP** = the cost saving) · FORGOT (**OTP required** — only ownership proof for WhatsApp-only users) · SIGNUP (OTP). NO SMS service (WhatsApp OTP or email OTP or SSO only). Mandatory web research (identifier-first auth, enumeration-safe, reset-token vs OTP, Morocco). End-to-end: events + training + DB + BO auth funnels. Diagnosis done (S-CTO-7); build queued (S-CTO-10).
Security escalations from S-CTO-7/9: A-ESC2 rate-limit spoof = CLOSED (`a400393`). A-ESC1 prod error-leak (old build / DEV_MODE) = founder HOLD, rides cutover. BO+MCP may share the spoof pattern (read-only audit owed).

### 4C. OTHER PRODUCT SURFACES (founder-flagged, mapped)
- **Store-Videos section** (store owners speak about their store, Instagram-stories style, grid+feed view, click→store page) AND **For-You section** (returning-user personalization from history/saved-search/bookmarks/location/contacted/viewed) = TWO DIFFERENT main-screen sections. Store-video AI→lake, For-You tracking, social-posting are backlog (STORE-SOCIAL B1–B9).
- **Coin ops (BO):** grant/deduct super-admin-only (=Ramzi), individual OR bulk, + notifications + reporting + campaign events (e.g. "Happy Eid, 100 coins to new/filtered users"). Basic fix DONE (super-admin gate + CSRF + audit); the bulk/campaign "Eid gift" tool (WAVE B) is a separate unbuilt feature.
- **AI listing (supply/demand core):** upload 1+ videos → AI scan vs prohibited + same-item match against images → prompt add video (upload/record) → immediate re-scan → final form (add/remove photos+videos) → categories/subcategory detected → user details → price suggestion (to improve) → human-in-the-loop → location → type (auction/negotiable) → delivery → AI generates listing from everything → publish clean → feeds to distribution. Jobs/services = image-optional (can pick car if relevant). Happy path proven; residual edges queued.
- **Search-intelligence (founder priority, not user-blocking):** the `searchEnrichment` engine EXISTS in api (activity-sieve, AI category extraction, cross-lingual tags, Darija lexicon ~639 terms, hybrid keyword+vector, image→tag, geo) but the BO shows only KEYWORD aggregates. Founder wants proper intelligence — what/where/how users search across image/category/language/location, feeding training + forecasting + reporting + advertiser value. Plan: api (surface search-intelligence events + enrichment) → BO (read UI with those dimensions). History: `CTO_BEHAVIORAL_INTELLIGENCE_GLOBAL_RESEARCH_2026_08_20.md` (~170-signal plan) + `TAWADOO_V2_AI_VISION_COMPLETE_2026_08_19.md` (image_search events, Titan 1024-dim on ~210K images).

---
## 5. THE EVENTS / ANALYTICS / SMART-VIEW / ACP PICTURE (detailed — "finish all events" means this)
**The bicycle-behind-a-Ferrari problem the founder keeps hitting:** we can see people BROWSE, we largely can't see people TRANSACT — the highest-value events are wired but not landing.

- **THE MAP (S-CTO-14, source-verified):** 492 events = **157 LIVE** + **128 WIRED-NOT-LANDING** + **207 PAPER**. Planned ambition was ~372–489 events / 20+ domains / 40+ funnels / ~170 signals; reality ≈ 2 of 25 funnels complete. This ambition is DEMOTED to "leverage" but must NOT be lost.
- **GAP 1 — money/commerce spine wired, never landed live:** offer_submitted, offer_accepted/rejected/countered (seller side has NO emit), bid_placed, coin_purchased, payment_completed, subscription_purchased, seller_order_confirmed, lead_unlocked, order_placed — emit code exists, screens exist, but never landed (no real buyer completed the flow in staging). Fix = WIRE + PROVE with real staging transactions, not rebuild.
- **GAP 2 — whole surfaces emit nothing:** report/abuse, offer accept/reject/counter, dashboard views (my-listings/bids/boosts/wallet), blog/content, search sort/paginate/autocomplete, listing photo/video upload, notification click/open, MoneyBand. Real screens, no emitter.
- **GAP 3 — duplicates + legacy drift:** several actions double-fire (offer_submitted+callback_requested; bid_placed+auction_joined); ~50 legacy names still land despite the kill order; 40 names land that aren't in the allowlist (17 undeclared smart_view_*).
- **DELIVERY/IDENTITY (the deeper cause):** money events reach Amplitude but were tagged with entity id not user id (S-CTO-3 fixed the emit; historical mis-keyed data NOT backfilled). ~69% of events are unattributable (no session_id/anon_id/interaction_id — only logged-in user_id). This is the biggest single limiter of funnels + the moat.
- **PROVIDERS:** Amplitude LIVE (EU vs US server-URL is a silent-drop risk to verify) · sGTM staging-only, GA4-only · MCP writes local SQLite (NOT the sovereign lake — Commandment 2 violation) · WhatsApp send doesn't route to ingestion · GA4/Amplitude MCP receipt needs a 30-sec founder re-auth (auth, not absent).
- **SMART VIEW / ACP:** Smart View chat + voice capture is live (to analytics only); founder says Smart View runtime quality is WEAK (high-priority product signal). NOT built: full ACP (agentic commerce) buyer-transact + seller-draft; A2UI declarative cards for external agents; the seller-agent (15 events), buyer-agent (8), ACP (3) blocks are declared-not-wired roadmap placeholders.
- **BO analytics:** 22 admin pages, only ask_ramzi_interaction fires — big Commandment-2 gap if admin actions must feed the lake.
- **UNKNOWN-EVENT LAW (founder):** an unknown event = BUILD it (wire its surface) or, if it has no UI, SURFACE it to the founder to approve/reject — never silently ignore.

---
## 6. THE ACTIVE PRIORITY UNIT — SYSTEMIC ENTITY & NAVIGATION INTEGRITY REFACTOR (in flight)
**Why it jumped the queue:** the founder identified that "item no longer available", broken sharing, dead deep-links, and notifications-that-can't-find-their-target are NOT isolated frontend bugs — they're one **cross-system architectural class**: entity addressing/resolution is not coherent across surfaces. FR-9/FR-11 are living proof. This IS the refactor (classification A). PATTERN OVER INSTANCE.
- **Stage 1 = S-CTO-25-NAV-MAP (IN FLIGHT, read-only):** trace 10 representative entities (listing from search / image-search / category / feed / seller's own / lead / notification target / share target / saved / auction) end-to-end across web + API + search/index + BO + DB. App + MCP = contract inventory only (Scope A). Founder constraint: **derive canonical identity from the SYSTEM OF RECORD (DB→Core→API→projection→URL→notif→share→app→MCP), NOT from the web impl.** Returns 7 artifacts: Entity Addressing Matrix · Failure Matrix (every "unavailable" → 1 of 12 true states) · Root-Cause Patterns · Canonical Target Model · Migration Surface · App/MCP Compat Notes · Stage-2 Architectural Decision.
- **Stage 2 = NAV-FIX (build, after founder approves the map):** implement the smallest correct canonical mechanism (route builders / entity references / resolver contract / notification-target structure / share-link generation — simple, NO "UniversalEntityRouter" framework) + migrate the affected callers + a navigation regression matrix (search→detail, share→fresh-browser, notif→target, category/feed→detail).
- **Then:** App + MCP conform to the established contract in their own units.
- **The 12 states to distinguish per failure:** 1 genuinely absent · 2 intentionally unavailable · 3 visibility wrong · 4 projection/index stale · 5 wrong id · 6 wrong entity type · 7 wrong slug · 8 wrong route · 9 wrong environment · 10 authz mismatch · 11 cache/client-state · 12 unknown. The system must never collapse these into one generic message.
- **Known anchors it will fold in:** FR-9 notif deadlink; the moderation two-gate (`isModerated`=index-eligibility via `index-eligibility.ts`, `isVerified`=search-visibility); vector-leg has no isVerified filter (D2); "view listing" 404 on pending moderation.

---
## 7. THE FULL QUEUE (every unit, classified, with status)
**Classification:** A refactor/architecture · B stabilization/regression · C feature · D housekeeping. **Status:** ✅DONE · 🔵IN FLIGHT · 📋QUEUED · ⏸PARKED · ⛔FOUNDER-DECISION · 🔒GATED(later phase).

### PHASE 1 — FINISH FUNCTIONAL (current phase)
| Unit | Class | Status | Notes |
|---|---|---|---|
| **NAV-integrity Stage 1 (S-CTO-25-NAV-MAP)** | A | 🔵 IN FLIGHT | the elevated priority unit (§6) |
| NAV-integrity Stage 2 (NAV-FIX) | A | 📋 QUEUED (after map approved) | canonical mechanism + migrate callers |
| FR wave — functional (FR-3 pagination, FR-8 badge/mark-all-read, FR-10 lead-unlock, FR-11 mark-lead) | B | ⏸ PARKED behind NAV | FR-11 re-scope after NAV; others addressing-independent |
| FR wave — remove/surgical (FR-1 pink, FR-7 save-search) | B | ⏸ PARKED behind NAV | web-only, prompts written |
| FR-4 chat/messages | B | 📋 QUEUED | own careful session + live WebSocket E2E guard |
| FR-5 desktop layout + FR-6 cards→prod-main | B | 📋 QUEUED | visual, never break mobile |
| FR-2 auction subscribe-notify | C | 📋 QUEUED | feature build (endpoint+entity+trigger+UI) |
| Auth flow redesign (S-CTO-10) | B | 📋 QUEUED | login=password/forgot=OTP/signup=OTP + web research + end-to-end |
| Search-intelligence 360 + build (api → BO) | A/C | 📋 QUEUED | engine exists; BO shows keywords only; founder priority |
| Events: WIRE + PROVE money funnel (GAP-1) + dark surfaces (GAP-2) | A/B | 📋 QUEUED | real staging transactions; unknown-event law |
| Event attribution threading (session/anon/interaction id on all events) | A | 📋 QUEUED | unblocks funnels + moat; ~69% unattributable today |
| BO analytics emitter wiring (22 pages) | B | 📋 QUEUED | Commandment-2 gap |
| Cohort cron crash fix | B(R-5) | 📋 QUEUED | must fix before platform activates at cutover |
| Customer-facing 500 (publications/mine) | B(R-2) | 📋 QUEUED | route name was hallucinated earlier; verify `publications/mine` live before calling it RED |
| Media same-item guard server-side (R-4) | A/B | 📋 QUEUED | client-only today, bypassable |
| App audit (mobile synced to current API?) | B(R-6) | 📋 QUEUED | ~7 weeks stale, read-only |
| MCP V1→V2 contract inventory (R-7) | B | 📋 QUEUED | read-only; don't break public contracts |
| STORE-SOCIAL backlog B1–B9 (store-video AI→lake, For-You, social-posting) | B/C | 📋 QUEUED | main-screen sections |
| Coin campaign/bulk "Eid gift" tool (WAVE B) | C | 📋 QUEUED | super-admin, filters, notifications, reporting |
| BO-FIX backlog (analytics-health banner, audit integrity, wallet RBAC, orphan-user FK) | B | 📋 QUEUED | financial-grade |
| §35 runtime-DDL cleanup (leads/entity onModuleInit ALTER TABLE) | A | 📋 QUEUED | same class as S138 crash |
| Distribution follow-ups (D-3 prod-named bucket, D-4 condition:new stripped, D-5 TikTok/ChatGPT feeds, D-6 CI syndication tests) | B | 📋 QUEUED | GMC-ban-history sensitive |

### PHASE 2 — SECURITY GATE (before refactor)
40+ bar: BOLA/IDOR sweep · BO RBAC live re-test · payment-replay · gitleaks · OWASP ZAP · CSP/headers · SSRF/XSS · disable no-TOTP on BO · close A-ESC1/A-ESC2 residue · **§28.5 merge (⛔ founder decision — which of the 2 prod security commits `6e06278` JwtAuthGuard + `1c714d6` Cache-Control merge to Ramzi_V2)**.

### PHASE 3 — THE STRUCTURAL REFACTOR (the mission; GATED behind functional ~90-95% + the RE-REVIEW)
5-layer modular monolith: **FACE** (web+app, Classic+Smart, messages, notifications) · **BUSINESS CORE** (accounts, listings, buy-now/offers/auctions, orders, wallet+coins, payments, delivery — one tight core, instant + never wrong) · **HELPERS/workers** (search indexing, video, distribution feeds, store-video, content, notifications/WhatsApp/email, lake export) · **PLUGS/adapters** (Amplitude/GA4/Meta/Google/TikTok/GMC/Bing/WhatsApp/SendGrid/Firebase/Payzone/Tawssil/Bedrock/OpenAI/MCP — any one down ≠ shop down) · **BRAIN/MOAT** (sovereign DB→S3 lake→future AI).
**10 invariants (never break):** Classic works · login always · money never wrong (locks) · payments verified · orders follow state machine · DB-first then copied · search keyword-fallback · provider-down ≠ shop-down · prod protected · every change reversible + staging-tested.
**Refactor sessions (GATED, fire only after the gate):** R0 pre-refactor architecture RE-REVIEW (the gate) → R1 platform foundation (schema-ownership completion beyond S141 + deploy discipline + IaC capture — none exists today) → R2 draw layer boundaries in code (one bounded slice first) → R3 PLUGS consolidation → R4 resolve the TWO parallel truth stores (`ta_activity_history` vs `ta_analytics_event`) into one System-of-Record + rebuildable projections → R5 HELPERS/workers isolation off the request path → R6 AI-fingerprint + dead-code hygiene (LAST, after behavior stable).

### PHASE 4 — SYNC + CUTOVER
Rebuild/sync app + MCP to the refactored core → cutover checklist: §28.5 merge · secret rotation (incl. Maps key) · app sync · GMC appeal · DMARC · CNDP declaration · Aurora Multi-AZ · 20K listing reindex · GA4 receipt · analytics platform migration-path + cohort-fix + allowlist-prune · full security bar · prod go-live (founder opens).

---
## 8. THE BAYESIAN / HIDDEN-STATE PREDICTION MODEL — THE MOAT (BLUE/post-refactor, founder-activated; its DATA foundation is built NOW)
**This is a major planned asset, not a one-liner. It lives in the BRAIN/MOAT layer. POST-refactor / founder-activated, but the functional work IS building its data foundation — so keep it visible.**
- **The gap it fills:** today's predictions are STATELESS — `PredictionEnrichmentService` (hourly@15) + `CohortEnrichmentService` (hourly@30) run SQL aggregations, write a scalar, forget. No belief carried between cycles; can't tell "silently deliberating" from "churned."
- **The design (5 stages, founder-approved direction):** (1) **HMM belief states** — browsing/researching/ready/dormant/churned + a Bayesian belief-update loop (the cheap first step) · (2) **GRU** (chosen over LSTM — fewer params, faster on ~60K users) with TWO memory horizons (session-intent + behavioral-personality) · (3) **Physics-informed model** — attention conservation, momentum/friction, seasonal hazard, PRICE-ELASTICITY as a latent variable; loss `L = L_prediction + λ·L_physics` · (4) **Offline RL (CQL)** from logged action-reward pairs · (5) **Walk-forward validation** aligned to the Moroccan calendar (Ramadan, salary cycles, seasons) + p-value rigor.
- **Where/how used:** BRAIN/MOAT layer fed by the sovereign lake. Powers buyer/seller/advertiser prediction, churn/conversion/LTV done RIGHT (belief-based not decayed scalar), RTB intent scoring, BO reporting/forecasting, price-suggestion V2. GRU inference = Tier 0 (CPU, cheaper than current SQL). Training = offline batch on SageMaker.
- **How it completes the training data (the link to everything we do now):** consumes (a) SESSION CORPUS (CORPUS-JOIN P1/P1B/P2 = ✅DONE) · (b) ATTRIBUTION (money-identity S-CTO-3 ✅ + event-attribution threading + paper events landing = in progress) · (c) REWARD SIGNAL ("did the user convert after the AI suggestion" = the D1 convert-definition decision, pending → closes the training tuple prompt+response+action). **So events/funnels/identity work IS the moat's data foundation; the model can't be built until (b)+(c) complete.**
- **GATED on 5 founder decisions (undecided — do NOT pre-lean):** 1 priority · 2 data-readiness (measure live volume first) · 3 SageMaker cost (§23 approval) · 4 serving model (GRU replaces PredictionEnrichmentService or runs alongside?) · 5 RL holdout groups acceptable at current scale?
- **NOT proven (must verify before build):** live training-data volume never measured; SageMaker availability not confirmed; GRU-cheaper-than-SQL is a hypothesis.
- **Gated sessions:** M0 data-readiness+cost verify → M1 HMM+belief-update → M2 GRU pipeline (SageMaker) → M3 physics/price-elasticity → M4 offline RL → M5 walk-forward + serving decision.

---
## 9. PENDING FOUNDER DECISIONS (block specific downstream work — do NOT pre-decide)
- **D1 — what counts as a "convert" per funnel** → gates CORPUS-JOIN P3 (convert-label) + the moat reward signal.
- **§28.5 — which of the 2 prod security commits merge to Ramzi_V2** (`6e06278` JwtAuthGuard, `1c714d6` Cache-Control) → go-live blocker.
- **D2 — raw-utterance retention** (text+voice for training, CNDP) → cutover/preprod (founder said not now).
- **The 5 Bayesian decisions** (§8) → at M-phase.
- **Identity contract — entity-vs-user model** (ta_analytics_event mixes user & entity ids) → gates IDENTITY-CONTRACT-AUDIT.
- **Moderation throughput priority + SLA** (human gate STAYS; AI only pre-screens).
- **Tawssil** — in the "100% functional" bar or post-launch? (never carried a real parcel).
- **Coin "grant" semantics + money-op second-approver policy.**
- **Design-system / motion un-defer** (currently deferred).

---
## 10. RED LINES / DO-NOT-VIOLATE (architectural + business)
- **Classic View is SACRED** — never broken by Smart work; mirror it, don't touch it. Classic + Smart = SEPARATE implementations, shared design foundation only (FACE-001 C). Never import Smart into Classic.
- **Human-in-the-loop moderation STAYS ALWAYS** (X6 sacred) — a human approves every listing for public visibility; AI only pre-screens; never auto-approve/remove. Distribution is silent+automatic for ELIGIBLE listings (routing, not a 2nd approval).
- **Distribution silently distributes EVERY eligible listing** from the start (free/paid/boosted-or-not) to maximize traffic before paid kicks in — eligibility + correct-feed-routing is the only gate; never communicate distribution status. GMC-ban history = feed guards are sacred; never weaken one.
- **Money never wrong** (locks + shared transactions in one tight Core — do NOT distribute). Payments verified. Orders follow the state machine.
- **DB is the System of Record**; projections (OpenSearch/lake/cache) are rebuildable and NEVER the authority for whether a commerce entity exists. DB-first → canonical event → outbox → async workers → lake. Lake never on the synchronous commerce path.
- **Provider-down ≠ shop-down** (adapters/Plugs). **Prod protected.** **Every change reversible + staging-tested.** Staging-first, fail-first, browser-verify UI. **No premature microservices.**
- **Every interaction feeds the lake** (Commandment 2 — sovereign DB first, then providers). **Cost-first** (cheapest model that works; vector before LLM; cache; guest quotas). **No user-facing copy without founder-approved FR/AR/EN.** **Grep names before creating (no collisions/hallucination). §35 credential split (no runtime DDL).**

---
## 11. WHAT CHATGPT SHOULD HELP PLAN (once the NAV-MAP audit lands)
When S-CTO-25-NAV-MAP returns its 7 artifacts, help decide:
1. **The canonical entity-addressing contract** — is the proposed model (identity→destination→resolution→visibility→authorization) minimal and correct? Where should it live (which layer/repo)? Does it avoid over-abstraction?
2. **Stage-2 sequencing** — build canonical mechanism first, then which callers to migrate in what order, with what regression matrix, without breaking Classic or mobile.
3. **The functional-completion path after NAV** — sequence the FR wave + auth flow + events-wire-and-prove + search-intelligence so we reach ~90-95% functional with fewest sessions and zero regression (max 2 concurrent, prefer 1 writer + 1 reader).
4. **The security gate** — the 40+ bar contents + §28.5 merge.
5. **The refactor R0→R6 slicing** — how to draw the 5-layer boundaries as bounded, reversible slices that preserve behavior.
6. **The moat activation** — when the data foundation (attribution + reward signal) is complete enough to start M0, and how to validate data-readiness + cost before building.
7. **App + MCP conformance** — how they adopt the canonical contract without breaking live public contracts, and the mobile ~7-week-drift remediation.

**Ground rules for ChatGPT's planning:** respect the North Star + phase order; treat inventory/notice as evidence not roadmap; keep prod protected; every recommendation must be a bounded unit with verification + rollback; surface founder decisions rather than pre-deciding them; PATTERN OVER INSTANCE; smallest correct change; human-quality code, no AI fingerprints.

---
## 12. SOURCE DOCS (all on disk, workspace root `/Users/ramzihannachi/Code/`)
Control: `TAWADOO_V2_CTO_OPERATING_SYSTEM.md` (laws + ledger + FR register + M-ZONE + parked registers) · `TAWADOO_V2_MASTER_QUEUE.md` · `TAWADOO_BUSINESS_TRUTH_2026_09_06.md` · `HANDOFF_B22_CTO_2026_09_06.md` · `B22_MASTER_RECONCILIATION_2026_09_05.md` · `BRAIN_B16_MASTERY_2026_08_31.md`.
Architecture: `TAWADOO_V2_TARGET_ARCHITECTURE.md` · `TAWADOO_V2_PRE_ARCHITECTURE_TRUTH_REPORT.md` · `AD-001_SCHEMA_OWNERSHIP_DETAILED_PLAN.md` · `TAWADOO_V2_FRONTEND_ARCHITECTURE.md` · the SYSTEMIC ENTITY & NAVIGATION INTEGRITY steering (the NAV directive).
Intelligence/moat: `SESSION_REPORT_INTELLIGENCE_LAKE_HIDDEN_STATE_2026_08_31.md` · `CTO_BEHAVIORAL_INTELLIGENCE_GLOBAL_RESEARCH_2026_08_20.md` · `TAWADOO_V2_AI_VISION_COMPLETE_2026_08_19.md`.
Distribution/feeds: `GOOGLE_MERCHANT_APPEAL_2026_08_23.md` + the S-CTO-22api report.
Session evidence: every `S_CTO_<n>_*` report/evidence file. NAV audit will write `S_CTO_25_NAV_MAP_EVIDENCE_2026_09_06.md`.
Laws (steering): `.kiro/steering/00-EXECUTION-PROMPT-NON-REGRESSION-LAW.md` (§0–§52) · `01-B14-CTO-STANDARD.md` (3 Commandments) · `02-REFACTOR-PROGRAM.md`.
