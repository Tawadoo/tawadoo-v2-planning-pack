# TAWADOO V2 — HANDOFF: BRAIN B22 → BRAIN B23 (CTO mode) — 2026-09-06
**YOU ARE BRAIN B23.** You continue as permanent CTO/Release Commander in Tawadoo V2 CTO mode — same laws, same mission, no re-planning. B22 hit context limit; this is the complete, source-true handoff so you resume with ZERO context loss.

**BOOT (do this first, in order):**
1. Read THIS file top-to-bottom.
2. Read `TAWADOO_V2_CTO_OPERATING_SYSTEM.md` (all binding laws + the SESSION LEDGER + the FRONTEND REGRESSION REGISTER + parked registers).
3. Read `TAWADOO_BUSINESS_TRUTH_2026_09_06.md` (core business — moderation/human-in-the-loop, jobs/services, distribution/feeds/guards, lead-gen model) — you must KNOW this, never ask the founder what's in it.
4. Read `TAWADOO_V2_MASTER_QUEUE.md` (the full phase queue: functional → security → refactor R0-R6 → Bayesian M0-M5 → cutover).
5. Verify live git HEADs before touching anything (§1). Then fire the next moves in §8.
**Continue the S-CTO-<n> session numbering (last = S-CTO-24). Do NOT restart it.**

**For:** Brain B23 (this Kiro lineage) AND the ChatGPT orchestrator. Every claim below is source/live-verified or tagged ⚪UNVERIFIED. Nothing buried.

## 0. WHO / MISSION
Founder = Ramzi (non-technical, final authority, does NO technical work — Kiro executes end-to-end). Kiro = permanent CTO/Release Commander. **Mission:** get Tawadoo V2 coherent + safe + verified → replace live V2 across web + app + ChatGPT/MCP. Order: **all-functional (staging) → security gate → refactor (5-layer modular monolith) → sync app+MCP → prod cutover.** Prod protected until founder opens go-live. `Ramzi_V2` = truth on every repo; staging = future prod in days.

## 1. LIVE GIT (✅ verified 2026-09-06) — all Ramzi_V2, synced
web `8b0a815f` · api `8d55a20` · bo `07b3a84` · mcp `d8efb4a`. **Nothing in flight** (S-CTO-24 was the last; read-only, no commits).
Infra: eu-west-1, acct 438465169079, ECS `tw-staging-cluster`. DB in-VPC/IP-locked (ECS-Exec agents down = can't run live psql; the recurring "BLOCKED" on DB reads).

## 2. BINDING LAWS (locked this session — in the operating system, every session inherits)
- **BUSINESS-TRUTH-FIRST:** read `TAWADOO_BUSINESS_TRUTH_2026_09_06.md`; never ask founder what's answered in source.
- **BUILD-NOT-READ:** default to investigate-then-FIX in ONE session; a read-only audit is never a stopping point — its WRITE/fix session must follow. No read-read loops.
- **360-INVESTIGATION-FIRST + ARCHITECTURE-MAP-FIRST:** every session traces the full cross-repo path (web/api/bo/mcp + DB + live), verifies every NAME from source (file:line) — a report/memory name is a hypothesis until grep-confirmed (earned: hallucinated `my-publications` → real `publications/mine`). Map which LAYER owns each step; if the real fix is in another layer, STOP + report (don't build in the wrong layer — earned: S-CTO-18web didn't know indexing lives in OpenSearch).
- **NAME-COLLISION HARD GATE:** grep existing names before creating any; reuse. Earned by real DB/BO duplicate mess.
- **SESSION-ID LAW:** every session = `S-CTO-<n>`, in prompt + opener + evidence filename. Kiro owns numbering (sessions self-suggest colliding numbers — ignore, renumber).
- **DISTRIBUTION/FEED LAW (SERIOUS — GMC ban history):** read `GOOGLE_MERCHANT_APPEAL_2026_08_23.md` + guards before touching feeds; NEVER weaken a ban-prevention guard.
- **HUMAN-IN-THE-LOOP STAYS ALWAYS (X6 sacred):** a human moderates every listing for public visibility; never auto-approved/removed. Distribution is silent+automatic for ELIGIBLE listings (routing, not a 2nd approval).
- Machine: MAX 2 concurrent sessions, prefer 1 writer + 1 reader, never 2 browser sweeps. Prod never mutated without explicit founder approval.
- Read→build pairing; §30 no user-facing copy without founder-approved FR/AR/EN.

## 3. DONE + VERIFIED THIS PROGRAM (frozen — do not reopen)
S-CTO-1 money-events diagnosis · **S-CTO-3 money-event identity fix** (`ab37f4b`, all ~18 emitters stamp acting user_id, proven live) · **S-CTO-5 video create-listing** (upload+record, no dup) · S-CTO-7 auth 3-methods diagnosis · **S-CTO-8 job/service image-optional (server)** (`ff5c66c`+`673eac0`) · **S-CTO-9 auth security** (`a400393`, rate-limit spoof closed) · **S-CTO-11 draft/autosave** (`780bdb58`) · **S-CTO-12 job/service web** (`7010ea6d`, seller-side) · S-CTO-13a naming reconciliation · S-CTO-14 event map (492: 157 landing/128 wired-not-fired/207 paper) · **S-CTO-15 event allowlist + reject-unknown** (`bd5cf2a`) · S-CTO-16 search-FLAG-A diagnosis · **S-CTO-18web category-icon fallback (My Listings)** (`8b0a815f`) · **S-CTO-19api allowlist add** (`50cfa12`) · S-CTO-20 MODERATION-360 map · S-CTO-21web (partial) · **S-CTO-22api feed photoless-drop fix** (`8d55a20`, leftJoin) · S-CTO-24 frontend-regression-360 diagnosis. Earlier frozen: coin-ops fix, S141 schema-ownership, AI-listing FIX-001..015, CORPUS-JOIN P1/P1B/P2, BO RBAC 80-item matrix.
- ⚪ **S-CTO-23qa (moderation runtime QA): NO report, NO commits** — never completed; human-in-the-loop chain still source-mapped only, NOT runtime-proven. Re-fire when useful.

## 4. 🔴 FRONTEND REGRESSION REGISTER (founder-reported LIVE w/ screenshots — staging BROKEN, prod-main = the GOOD reference; DESKTOP broken, MOBILE fine — never break mobile). Root-caused by S-CTO-24.
Built-in-back-but-not-in-front is the pattern. Fix order: FUNCTIONAL first, then remove/visual.
- **FR-1 PINK RECTANGLE → REMOVE ENTIRELY** (founder: doesn't know what it is). Confirm component from source first, then delete its render; STOP if it's a real feature.
- **FR-2 AUCTION CARD:** unbuilt end-to-end — a not-started auction must be HIDDEN unless user can SUBSCRIBE to notify-on-start (email+in-app). **No endpoint, no DB entity, no start trigger, no UI exist** (only FCM/email/in-app channels). = build feature, not a tweak.
- **FR-3 PAGINATION** dead (page 1→2 fails, across the board). Core browse. Confirm `publications/search` page param.
- **FR-4 CHAT/MESSAGES** dead. Known-fragile surface (rebuilt +5 fixes) — needs a live E2E guard (O-10). Verify socket connect + ALB WebSocket upgrade (⚪ not verified).
- **FR-5 DESKTOP DETAIL LAYOUT:** huge empty space + HORIZONTAL "Grow your business" strip; prod-main = VERTICAL dual side AD banners. Fix desktop, don't touch mobile.
- **FR-6 PRODUCT CARDS:** ugly vs prod-main; restore prod-main card style + tags (negotiable/auction) + vertical side banners.
- **FR-7 SAVE-SEARCH MIN-CRITERIA → surgical (cat gate):** `search-results-view.tsx:371` `SAVEABLE_SEARCH_PARAMS` includes `'cat'` → button renders on bare category page. Require min real search (location city/lat/lng OR search_text OR real filter), NOT category/type/store alone. Reuse existing gate; do NOT disturb S4 save-search email-consent.
- **FR-8 NOTIF BADGE STUCK:** count stays after viewing. `SiteHeader.tsx:39` reads store; **mark-all-read endpoint does NOT exist in api (only per-item) → must be BUILT.**
- **FR-9 NOTIF DEEPLINK + COPY:** callback notif has a working deeplink helper in api BUT api omits service/objectId + web handler has no lead branch → dead-ends at `/notification-unavailable`. Fix api payload + web handler. §30 copy.
- **FR-10 LEAD-UNLOCK TRANSPARENCY:** api COMPUTES how paid (allowance vs 10 coins) + balance, but **the controller DROPS it** → front can't show truth. Return it + build the UI prompt (cost X coins / free from package, N left, show balance, confirm). Lead-gen model: free/bonus quota FIRST else 10 coins (`leads.service.ts:402-409`, `LEAD_UNLOCK_COST`).
- **FR-11 MARK-LEAD-STATUS FAILS** ("Échec de la mise à jour"): `PATCH /leads/:id/status` (`leads.controller.ts:528`, viewed/contacted/converted). Diagnose why it fails (auth/entity-scope/payload) + fix. Core lead-gen workflow broken.

## 5. OTHER OPEN THREADS (logged, source-verified)
- **SEARCH-INTELLIGENCE 360+BUILD (founder-flagged):** BO shows KEYWORD aggregates only; founder wants proper intelligence — what/where/how users search across image/category/language/location, feeding training+forecasting+reporting. Engine EXISTS (`searchEnrichment` api: activity-sieve, AI category extraction, cross-lingual tags, Darija lexicon ~639, hybrid keyword+vector, image→tag, geo) but NOT surfaced in BO. Plan: api (surface search-intelligence events + enrichment) → BO (read UI with those dimensions). History: `CTO_BEHAVIORAL_INTELLIGENCE_GLOBAL_RESEARCH_2026_08_20.md` (~170-signal plan) + `TAWADOO_V2_AI_VISION_COMPLETE_2026_08_19.md` (image_search events #336/#337, Titan 1024-dim on ~210K images). MEDIUM-LARGE, plan as its own execution. High value but NOT user-blocking → after frontend regressions.
- **DISTRIBUTION follow-ups (from S-CTO-22api):** D-1 verify a real photoless job/service actually flows to feeds (DB-locked, seed+observe owed) · D-3 🟡 staging writes feeds to PROD-named bucket `tw-prod-media-storage-prod` (config review) · D-4 🟡 `condition: new` stripped since early Sept even for genuinely-new (may hurt reach) · D-5 no TikTok/ChatGPT feeds on staging (only Meta) · D-6 CI doesn't run syndication tests.
- **MODERATION (from S-CTO-20):** manual-only (intended, F2 decided). F1 fix shipped (shape-1). F3 split-flag hazard (auto-mod branch sets only isModerated) + F4 divergent detail predicates — reconcile in refactor. Runtime proof of the chain still owed (S-CTO-23qa never ran).
- **§28 DRIFT (O-12):** origin/main is NOT an ancestor of Ramzi_V2 (main +1335) — confirm no prod-only lead/notif/search fix is missing from the future line. §28.5: 2 prod security commits (`6e06278` JwtAuthGuard, `1c714d6` Cache-Control) still owed to Ramzi_V2 (founder merge decision).
- **§35 runtime DDL (O-7):** `leads.service.ts` + `entity.service.ts` run `ALTER TABLE ADD COLUMN` at startup (onModuleInit) — may warn-and-skip on prod-like credential split → §35 audit owed.
- **Hygiene:** O-6 web hard-codes `coins_spent:10` on lead_unlocked event even when allowance used (wrong lake data) · O-8 stale `required-server-files.json` localhost:3010 · O-9 `listing_sidebar` ad slot defined never rendered.

## 6. PENDING FOUNDER DECISIONS
- §28.5 which prod commits merge to Ramzi_V2 (2 security fixes).
- D1 "what counts as a convert" per funnel (gates funnels + Bayesian reward-signal).
- The 5 Bayesian decisions (priority, data-readiness, SageMaker cost, serving model, RL holdout) — at M-phase.
- D2 raw-utterance retention (CNDP). · Tawssil in the "100% functional" bar or post-launch?

## 7. THE BIG PICTURE QUEUE (full detail in TAWADOO_V2_MASTER_QUEUE.md)
PHASE 1 FUNCTIONAL (now): frontend regressions FR-1..11 → search-intelligence → remaining RED (customer 500 unverified, cohort crash, media-guard server-side, app audit, MCP contract inventory) → auth flow (login=password/forgot=OTP). 
PHASE 2 SECURITY GATE: 40+ bar (BOLA/IDOR, RBAC live re-test, ZAP, gitleaks, CSP, TOTP-off) + §28.5. 
PHASE 3 REFACTOR (gated): R0 arch re-review → R1 platform foundation → R2 layer boundaries → R3 plugs → R4 one-truth-store → R5 workers → R6 hygiene (5-layer modular monolith: FACE/CORE/HELPERS/PLUGS/BRAIN). 
PHASE 4 MOAT (Bayesian, founder-activated): M0 data-readiness → M1 HMM → M2 GRU → M3 physics/price-elasticity → M4 RL → M5 walk-forward. 
PHASE 5 SYNC app+MCP → CUTOVER (checklist: §28.5, secret rotation incl. Maps key, app sync, GMC appeal, DMARC, CNDP, Aurora Multi-AZ, 20K reindex, GA4, analytics platform migration + cohort-fix + allowlist-prune).

## 8. IMMEDIATE NEXT MOVES (nothing in flight — safe to fire 2)
Per BUILD-NOT-READ, go straight to build (S-CTO-24 already diagnosed all 11). Recommended pair (both build, investigate-then-fix, safe = different concerns):
1. **S-CTO-25 — FUNCTIONAL FRONTEND FIX (web+api):** FR-3 pagination + FR-11 mark-lead + FR-8 badge (build mark-all-read api) + FR-9 notif deeplink (api payload + web handler). The user-blocking set. Browser-verify desktop+mobile, regression guards, never break mobile/Classic.
2. **S-CTO-26 — REMOVE + SURGICAL WEB (web):** FR-1 remove pink rectangle + FR-7 tighten save-search cat gate. Small, web-only, safe beside 25.
Then: S-CTO-27 chat (FR-4, its own careful session — fragile, live E2E guard) · S-CTO-28 visual set (FR-5 desktop layout + FR-6 cards toward prod-main) · FR-2 auction subscribe-notify (feature build) · search-intelligence 360+build.

## 9. EVIDENCE FILES (all on disk, workspace root)
Every S-CTO session wrote `S_CTO_<n>_*EVIDENCE*.md` / `*END_SESSION_REPORT*.md`. Control docs: `TAWADOO_V2_CTO_OPERATING_SYSTEM.md` (laws+ledger+registers), `TAWADOO_V2_MASTER_QUEUE.md`, `TAWADOO_BUSINESS_TRUTH_2026_09_06.md`, `B22_MASTER_RECONCILIATION_2026_09_05.md` (+ its PART A orchestration brief), `BRAIN_B16_MASTERY_2026_08_31.md` (history/refactor design), `GOOGLE_MERCHANT_APPEAL_2026_08_23.md` (feed ban history).

**Re-entry:** read §1-2 (git+laws), §3 (done), §4 (the live regressions), §8 (next moves). Fire S-CTO-25 + S-CTO-26. Verify every name from source before touching. Nothing about a user behavior/feature is dropped without founder seeing it.
