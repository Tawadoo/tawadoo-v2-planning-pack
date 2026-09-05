# TAWADOO V2 — MASTER QUEUE (single clear view)
**Updated:** 2026-09-05 · **Owner:** Kiro CTO (drives sequence) · **Founder:** Ramzi (authorizes each unit + business decisions)
**North Star:** all-functional (staging) → security gate → refactor → sync app + MCP → prod cutover. Prod protected. Machine limit: max 2 sessions at once.

**Status keys:** ✅ DONE · 🔵 IN FLIGHT · 📋 QUEUED (ready/near-ready) · ⏸ PARKED · ⛔ NEEDS FOUNDER DECISION · ✖ RETIRED · 🔒 GATED (later phase)

**NOW IN FLIGHT (2, machine max):** S-CTO-8 (job/service image, api) · S-CTO-9 (auth security, api). Everything else awaits a free slot + founder go.
**FOUNDER OPERATING MODE:** each step discussed, founder decides as we go. Kiro drives sequence + 2-phrase briefs. Nothing fires without founder go.

---

## A. SESSIONS (numbered, canonical)
| ID | What it does (plain) | Repo | Status |
|---|---|---|---|
| S-CTO-1 | Diagnose money-events → Amplitude identity | api (RO) | ✅ DONE |
| S-CTO-2 | Create-form draft + autosave (built) | web | ⚠️ built, uncommitted (P1) |
| S-CTO-3 | Money events now tagged to real user | api | ✅ DONE (live) |
| S-CTO-4 | Commit + verify the S-CTO-2 draft work (no deletion) | web | ⏸ PARKED |
| S-CTO-5 | Video upload+record in create-listing, no dup | web | ✅ DONE (live) |
| S-CTO-6 | (narrow MX fix) | — | ✖ RETIRED → S-CTO-7 |
| S-CTO-7 | Diagnose all 3 signup methods + auth gates | api (RO) | ✅ DONE |
| S-CTO-8 | Job/service publishable w/o image (server) | api | 🔵 IN FLIGHT |
| S-CTO-9 | Auth security: rate-limit spoof + prod error-leak | api | 🔵 IN FLIGHT |
| S-CTO-10 | Auth flow: login=password / forgot=OTP / signup=OTP + web-research + end-to-end | api+web | 📋 QUEUED (needs research; after P1) |
| S-CTO-11 | Draft finisher (= do S-CTO-4) → resolve P1 in product-form-v2 | web | 📋 QUEUED (prereq for more create-listing web work) |
| S-CTO-12 | Job/service WEB side (form image-optional + in-form record button O4) | web | 📋 QUEUED (after S-CTO-11 clears P1) |
| S-CTO-13 | Fix customer-facing 500 on GET /publications/my-publications (R-2) | api (+web verify) | 📋 QUEUED |
| S-CTO-14 | EVENTS-vs-UI 360 (read-only): which of ~260 paper events need a real UI vs prune | api+web (RO) | 📋 QUEUED (answers "events missing UI") |
| S-CTO-15 | SMART VIEW COMPLETION 360 (read-only): what SV is vs target ACP; real gap list | web (RO) | 📋 QUEUED |
| S-CTO-16 | Cohort cron crash fix (R-5) | api | 📋 QUEUED |
| S-CTO-17 | APP audit (read-only): is mobile synced to current API? (R-6) | app (RO) | 📋 QUEUED |
| S-CTO-18 | ChatGPT/MCP V1→V2 contract inventory (read-only) (R-7) | mcp (RO) | 📋 QUEUED |
| S-CTO-19 | §28.5 merge — port 2 prod security fixes into Ramzi_V2 (R-3) | api | ⛔ NEEDS FOUNDER DECISION (which commits) |
| S-CTO-20 | Server-side media same-item guard (R-4, B1) | api | 📋 QUEUED |

*(build sessions that follow a read-only diagnosis get their own ID when the diagnosis lands — read→build law.)*

---

## B. RED — RELEASE BLOCKERS (must all clear before refactor)
| # | Blocker | Session |
|---|---|---|
| R-2 | Customer-facing 500 on my-publications | S-CTO-13 |
| R-3 | §28.5: 2 prod security fixes missing from Ramzi_V2 | S-CTO-19 ⛔ your call |
| R-4 | Media same-item guard only client-side (bypassable) | S-CTO-20 |
| R-5 | Cohort cron crashes every run | S-CTO-16 |
| R-6 | Mobile app ~7 weeks stale vs API — unverified | S-CTO-17 |
| R-7 | MCP V1→V2 public contracts not inventoried | S-CTO-18 |
| A-ESC1 | Prod error-leak / possible DEV_MODE | S-CTO-9 🔵 |
| A-ESC2 | Rate-limiter header-spoof bypass | S-CTO-9 🔵 |
| A-FLOW1 | Auth flow: login=password, forgot=OTP (OTP cost) | S-CTO-10 |
| U3-11 | prediction_conversion_scored rejected on "system" uuid → lake loss | (identity-contract fix, queued) |

## C. YELLOW (high-leverage, fix if small)
Event attribution threading · ~260 paper events → wire real ones / prune (S-CTO-14) · search-intelligence BO read-surface (image/category/language/location) · Smart View runtime quality (S-CTO-15) · moderation pipeline map (O3) · coin-debit verify (O5) · OpenSearch video-carry verify (O6) · analytics/lake create+video events land (O7) · disposable blocklist dead · Node-20 CI · API admin tests not in CI · noisy SQL logging.

## D. BLUE (post-launch — do NOT hijack release)
Bayesian/HMM model · full ACP buyer-transact + seller-draft · RTB/advertiser + campaigns · price-suggestion V2 · design-system/motion · broad §28 drift · GA4 MCP · sGTM ownership · SMS→WhatsApp label rename (code hygiene) · WebKit console errors (O8) · digit-run policy (O9) · unified behavioral-intelligence asset.

## E. FOUNDER DECISIONS
| Decision | Status |
|---|---|
| D1 job/service image-optional | ✅ YES (option A) |
| D2 Google Maps key | ✅ DEFER to cutover |
| A-DEC1 prod test accounts | ✅ LEAVE (may reuse) |
| A-DEC2 SMS wording | ✅ CORRECTED (not user-facing; no action) |
| R-3 §28.5 which commits merge to Ramzi_V2 | ⛔ PENDING |

## F. PARKED / PREREQUISITES
- **P1 (blocking):** uncommitted draft in `product-form-v2.tsx` → resolve via S-CTO-11 before any more create-listing web edits.
- Historical money-event backfill (U3-2) · staging test data cleanup (U3-3) · ECS-Exec channel down (U3-4, blocks in-VPC DB reads) · Amplitude order_id re-scope (U3-5) · pre-existing failing test suites (U3-7) · Tawssil never carried a real parcel.

## G. CUTOVER CHECKLIST (pre-prod, not now)
§28.5 merge · secret rotation (incl. Maps key P5) · app sync · GMC appeal · DMARC · CNDP declaration · Aurora Multi-AZ · 20K listing reindex · GA4 receipt · analytics platform migration-path + cohort-fix + allowlist-prune · full security bar (BOLA/IDOR, ZAP, gitleaks, CSP, TOTP-on).

---

## H. THE REFACTOR PHASE (THE ACTUAL MISSION — design in Brain B16 + `TAWADOO_V2_TARGET_ARCHITECTURE.md` + `AD-001` + synthesis mandate)
**GATED:** does NOT start until functional ~90–95% + a fresh crash-proof architecture RE-REVIEW passes. Today: codebase-refactored ~20%, structural refactor ~5–10% (barely started — only the S141 schema-ownership foundation is built). This is the umbrella the functional fixes serve, NOT an afterthought.
**What the refactor IS (from source):** ORGANIZE the code into the approved shape + add the missing platform foundation. It does NOT rebuild what works (Core/money/search/sovereignty stay). Behavior-preserving, no AI fingerprints, one bounded slice at a time, every step reversible + staging-tested.

**Target shape — 5-layer modular monolith (NOT microservices):**
1. FACE — web + app, Classic + Smart View, messages, notifications.
2. BUSINESS CORE — accounts, listings, buy-now/offers/auctions, orders, wallet+coins, payments, delivery. One tight core, instant + never wrong.
3. HELPERS (workers) — search indexing, video, distribution feeds, store-video, content, notifications/WhatsApp/email, lake export. Background.
4. PLUGS (adapters) — Amplitude/GA4/Meta/Google/TikTok/GMC/Bing/WhatsApp/SendGrid/Firebase/Payzone/Tawssil/Bedrock/OpenAI/MCP. Any one breaks → Tawadoo keeps running.
5. BRAIN/MOAT — sovereign DB → S3 lake → future AI.
**10 invariants (never break):** Classic works · login always · money never wrong (locks) · payments verified · orders follow state machine · DB-first then copied · search keyword-fallback · provider-down ≠ shop-down · prod protected · every change reversible + staging-tested.

**REFACTOR SESSIONS (queued, GATED — fire only after the gate):**
| ID | Refactor unit | Status |
|---|---|---|
| S-CTO-R0 | PRE-REFACTOR ARCHITECTURE RE-REVIEW (the gate) — re-verify the 5-layer design vs current live source; confirm crash-proof plan; produce the bounded slice list | 🔒 GATED (fires when functional ~90–95%) |
| S-CTO-R1 | Platform foundation: the missing DB-key/schema-ownership completion beyond S141 + deploy discipline + IaC capture (no IaC exists today) | 🔒 GATED |
| S-CTO-R2 | Draw the LAYER BOUNDARIES in code: separate FACE / CORE / HELPERS / PLUGS / BRAIN by module ownership + dependency direction (one bounded slice first, prove, then extend) | 🔒 GATED |
| S-CTO-R3 | PLUGS consolidation: every external provider behind a clean adapter (Amplitude/Meta/Google/TikTok/WhatsApp/Payzone/Tawssil/Bedrock…) so provider-down ≠ shop-down | 🔒 GATED |
| S-CTO-R4 | Resolve the TWO parallel truth stores (`ta_activity_history` vs `ta_analytics_event`) → one System-of-Record + rebuildable projections | 🔒 GATED |
| S-CTO-R5 | HELPERS/workers isolation: search-index, video, feeds, lake-export as background workers, never on the customer request path | 🔒 GATED |
| S-CTO-R6 | AI-fingerprint + dead-code hygiene pass across refactored modules (human-quality code) — LAST, after behavior is stable | 🔒 GATED |
*(R-sessions get real numbered IDs when the gate opens; each is one bounded slice with verification + rollback. No mass migration without a proven slice.)*

---

## I. RECOMMENDED ORDER
**PHASE 1 — FINISH FUNCTIONAL (now):**
1. **S-CTO-11** (resolve P1 draft) — unblocks create-listing web work.
2. **S-CTO-12** (job/service web) + **S-CTO-13** (the 500) — parallel-safe.
3. **S-CTO-10** (auth flow + research).
4. **S-CTO-14** (events-vs-UI map) + **S-CTO-15** (Smart View map) — read-only → build lists.
5. **S-CTO-16** (cohort) · **S-CTO-20** (media guard) · **S-CTO-19** (§28.5, your decision).
6. **S-CTO-17** (app audit) · **S-CTO-18** (MCP inventory).

**PHASE 2 — SECURITY GATE:** the 40+ security bar (BOLA/IDOR sweep, BO RBAC live re-test, payment-replay, gitleaks, OWASP ZAP, CSP/headers, SSRF/XSS, disable no-TOTP) + close A-ESC1/A-ESC2.

**PHASE 3 — THE REFACTOR (the mission):** S-CTO-R0 gate → R1 foundation → R2 boundaries → R3 plugs → R4 truth-store → R5 workers → R6 hygiene. Bounded slices, behavior-preserving.

**PHASE 4 — SYNC + CUTOVER:** rebuild/sync app + MCP to the refactored core → full cutover checklist (§G) → prod go-live (founder opens).

---

## J. THE BAYESIAN / HIDDEN-STATE PREDICTION MODEL (the MOAT — design in `SESSION_REPORT_INTELLIGENCE_LAKE_HIDDEN_STATE_2026_08_31.md`)
**This is a major planned asset, NOT a BLUE one-liner. It lives in the BRAIN/MOAT layer. It is POST-refactor / founder-activated, but its DATA foundation is being built NOW by the functional work — so it must stay visible.**

**WHY (the gap it fills):** today's predictions are STATELESS — `PredictionEnrichmentService` (hourly@15) + `CohortEnrichmentService` (hourly@30) run SQL aggregations, write a scalar, forget. No memory, no belief carried between cycles. Cannot distinguish "silently deliberating" from "churned." (✅ source-verified.)

**THE DESIGN (5 stages, founder-approved direction):**
1. HMM belief states — browsing / researching / ready / dormant / churned + a Bayesian belief-update loop (the cheap first step).
2. GRU (chosen over LSTM — fewer params, faster on ~60K users) with TWO memory horizons: session-intent + behavioral-personality.
3. Physics-informed model — attention conservation, momentum/friction, seasonal hazard, PRICE-ELASTICITY as a latent variable. Loss `L = L_prediction + λ·L_physics`.
4. Offline RL (CQL) from logged action-reward pairs.
5. Walk-forward validation aligned to the Moroccan calendar (Ramadan, salary cycles, seasons) + p-value rigor.

**WHERE / HOW IT'S USED:** BRAIN/MOAT layer, fed by the sovereign lake. Powers: buyer/seller/advertiser prediction, churn/conversion/LTV done RIGHT (belief-based not decayed scalar), RTB intent scoring, reporting/forecasting in the BO, price-suggestion V2. GRU inference = Tier 0 (CPU, cheaper than current SQL). Training = offline batch on SageMaker.

**HOW IT COMPLETES OUR TRAINING DATA (the link to everything we do now):** the model consumes three things the functional phase is producing —
  (a) SESSION CORPUS — CORPUS-JOIN P1/P1B/P2 = ✅ DONE (S3 session sequences).
  (b) ATTRIBUTION — money-identity fix (S-CTO-3 ✅) + event-attribution threading + the ~260 paper events landing (S-CTO-14) = in progress.
  (c) REWARD SIGNAL — "did the user convert after the AI suggestion" = the D1 convert-definition decision (pending) → this closes the training tuple (prompt + response + user_action).
  **So the events/funnels/identity work IS the data foundation for this model. The model can't be built until (b)+(c) are complete — that's why it's post-functional.**

**GATED ON 5 FOUNDER DECISIONS (undecided — do NOT pre-lean):** 1. priority · 2. data-readiness confirmed (measure live data volume first) · 3. SageMaker cost (§23 approval) · 4. serving model (GRU replaces `PredictionEnrichmentService` or runs alongside?) · 5. RL holdout groups (acceptable at current scale?).

**NOT PROVEN (source caveat, must verify before build):** live training-data volume never measured; SageMaker availability not confirmed; GRU-cheaper-than-SQL is a hypothesis. A future session must verify data volume + cost BEFORE building.

**QUEUED SESSIONS (gated — after functional + refactor, founder-activated):**
| ID | Unit | Status |
|---|---|---|
| S-CTO-M0 | DATA-READINESS + COST VERIFY (read-only): measure live training-data volume, confirm reward-tuple completeness, SageMaker availability + cost, GRU-vs-SQL cost check | 🔒 GATED (needs D1 + data foundation) |
| S-CTO-M1 | HMM belief-state layer + Bayesian belief-update loop (cheap first step) — replaces stateless scalar with carried belief | 🔒 GATED |
| S-CTO-M2 | GRU (2 horizons) training pipeline (offline batch, SageMaker — §23 approval) | 🔒 GATED |
| S-CTO-M3 | Physics-informed loss + price-elasticity latent → feeds price-suggestion V2 | 🔒 GATED |
| S-CTO-M4 | Offline RL (CQL) from logged action-reward pairs + holdout groups | 🔒 GATED |
| S-CTO-M5 | Walk-forward calendar-aligned validation + serving decision (replace/alongside) | 🔒 GATED |
