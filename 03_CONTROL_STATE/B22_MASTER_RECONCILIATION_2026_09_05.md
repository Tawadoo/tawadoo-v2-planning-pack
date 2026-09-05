# B22 — MASTER BRAIN RECONCILIATION & PROGRAM RECOVERY (2026-09-05)

> **THIS IS THE SINGLE COMPLETE DOCUMENT FOR AN EXTERNAL ORCHESTRATOR (ChatGPT).**
> **PART A** below is the self-contained ORCHESTRATION BRIEF — read it first; it is enough to orchestrate from.
> **PART B** (starts at "## 1. EXECUTIVE TRUTH") is the deeper brain-reconciliation detail (control plane, registers, contradictions, drift, boot protocol) for when you need the underlying evidence.
> Trust rules: every material number is tagged ✅VERIFIED (checked vs live git/code this session) · 📄DOC (from a project doc, not re-verified) · ⚪UNVERIFIED. Where docs drifted, the CODE/live value wins; the stale one is marked SUPERSEDED. Never treat a ⚪ as fact.

---

# ══════════════════ PART A — ORCHESTRATION BRIEF ══════════════════
**Author:** Kiro Brain B22 · **Date:** 2026-09-05 · **Purpose:** a single, source-true document to help orchestrate Tawadoo's path to production.

## A0. WHAT TAWADOO IS (context, 📄DOC)
Moroccan multi-sided marketplace (buyers + sellers), coin-based economy, French/Arabic/Darija/English. Web + mobile app + Back-Office (BO) + an MCP server (agentic/ChatGPT commerce). Vision: every buyer, seller, and advertiser gets an AI agent; the platform learns from a sovereign data lake to power its own future prediction model. Moat = proprietary Darija + behavioral data, NOT a homegrown LLM. Corridor: Morocco → Algeria → Tunisia → West Africa.

## A1. THE MISSION + THE ORDER (📄DOC, founder-locked)
**North Star:** refactor the existing Tawadoo V2 codebase into clean, maintainable, human-quality code — behavior-preserving, no AI fingerprints, crash-proof (a "2-year-no-crash" bar). **Everything else (analytics, intelligence, growth, agents) is SUBORDINATE LEVERAGE, not the mission.**
**The ORDER (never reorder without founder):** ALL-FUNCTIONAL (staging) → PRE-REFACTOR crash-proof ARCHITECTURE RE-REVIEW (a gate) → STRUCTURAL REFACTOR → SYNC/REBUILD app → PRODUCTION cutover. **Prod is protected and out of scope until the founder opens go-live. Staging is the proving ground.**

## A2. CURRENT STATE — SOURCE-TRUE (✅VERIFIED 2026-09-05)
- **Git (all on branch `Ramzi_V2`, synced):** web `36b7183c` · api `bceef76` · bo `07b3a84` · mcp `d8efb4a`. ✅VERIFIED (git rev-parse).
- **Progress (📄DOC, self-assessed, ⚪ not independently measurable):** codebase-refactored ~20% · structural refactor barely started (~5–10%, only the DB schema-ownership foundation built) · whole-program-to-prod ~45–50%.
- **Nothing is in flight** as of this brief.
- **Prod is LIVE and healthy** (📄DOC, Aug-29 snapshot): ~57,324 users, ~21,434 published listings. Prod runs the LEGACY stack; the new analytics/intelligence platform lives ONLY on staging (`Ramzi_V2`) and activates at cutover.
- **⚠️ Prod ≠ staging architecture:** prod `/api/analytics/events` = 404, no Amplitude, no lake, no sGTM. The "good architecture" is not serving real users yet. (📄DOC, cutover item — calm, not a fault.)

## A3. EVENTS / FUNNELS / SIGNALS — THE REAL NUMBERS (✅VERIFIED against code + the 2026-09-04 audits)
**⚠️ Number correction (important for orchestration):** early docs said "200+ / 240 events, 12 funnels." Those are **SUPERSEDED.** The truth:
| Metric | Value | Source |
|---|---|---|
| Events in the live allowlist | **~490** (489 by code grep of `allowed-events.ts`; 492 in the Sept-4 audit — method diff) | ✅VERIFIED |
| Events with an actual emitter (wired) | **~232** | 📄DOC (ANALYTICS_EVENT_FUNNEL_REALITY_AUDIT_2026-09-04) |
| Events PAPER-ONLY (declared, never fired) | **~260 (~53%)** | 📄DOC (same) |
| Events ever landed in Amplitude | **~205** | 📄DOC |
| Funnels DEFINED | **7** (5 valid · 1 at-risk · 1 dead-on-read) — not the "12" of old docs | ✅VERIFIED (audit) |
| Derived signals | **~50** planned; partially computed | 📄DOC |
| Domains | **13** (identity/session, discovery, search, listing, auction, offers, messaging, orders, payments/coins, seller-lifecycle, notifications, AI/MCP, system) | 📄DOC |
**The headline truth:** the allowlist is OVER-DECLARED (~490) and UNDER-BUILT (~half never wired). The MONEY spine (offer/bid/coin/payment/order/lead/subscription) fires and lands the sovereign DB lake. **Whether it reaches Amplitude is UNDER RE-VERIFICATION:** the Sept-4 audit says money events do NOT reach Amplitude, but a source read this session shows they emit via the sovereign path (97 sovereign call-sites vs 3 legacy) and the delivery worker DOES call Amplitude for any event carrying a real user_id. This doc-vs-source conflict is the CLOSE-U3 question and must be settled by a live read before any "fix." ~69% of events are UNATTRIBUTABLE (carry no session/anonymous/interaction id — only logged-in user_id). (Mixed 📄DOC / ✅source — flagged for live diagnosis.)
**Two parallel truth stores (architecture note, 📄DOC):** seller-facing funnels read `ta_activity_history`; the lake reads `ta_analytics_event`. They overlap = a duplication to resolve in the refactor.

## A4. THE SOVEREIGN DATA PIPELINE (📄DOC + ✅ partial)
Design (a RED LINE): user action → **write DB first** (`ta_analytics_event`) → **transactional outbox** (`ta_analytics_delivery`, per-destination) → **async workers** deliver to Amplitude / sGTM / Meta CAPI / TikTok / GA4 / S3 lake. **The lake is NEVER on the synchronous commerce path.** Live: DB-first + lake proven working + idempotent (✅ prior sessions). Amplitude receipt of AI/system events = proven (✅VERIFIED B21). Money-event Amplitude delivery = the CLOSE-U3 question (see A3, under re-verification). GA4 MCP = blocked by a Google auth policy (⚪, parked). sGTM = deployed but idle, no owner (⚪).

## A5. THE INTELLIGENCE + FUTURE MODEL (📄DOC — PARKED, not current work)
- **Today's prediction model is REACTIVE with ZERO hidden state** (churn = linear decay; conversion = weighted event count; ltv = percentile). ✅VERIFIED (source read).
- **Planned future model (PARKED P4, founder-gated):** HMM belief states (browsing/researching/ready/dormant/churned) → Bayesian belief update → GRU (2 memory horizons) → physics-informed model (price-elasticity as a latent variable) → offline RL → walk-forward validation on the Moroccan calendar. Needs 5 founder decisions (priority · data-readiness · SageMaker cost · serving model · RL holdout). **Its DATA dependency (session corpus + attribution) must be complete BEFORE the refactor, even though the model build is post-refactor.**
- **Moat corpus:** CORPUS-JOIN P1/P1B/P2 = DONE (✅VERIFIED, S3 session corpus produced). P3 (convert-label) is GATED on founder decision D1. P4 = funnel assembly.
- **Predictions/cohorts today (📄DOC):** computed hourly but NOTHING reads them; behavioral cohorts crash on every run; conversation intents write-only.

## A6. THE 3 AGENTS (endgame, mostly PLANNED/PARKED, 📄DOC)
- **Buyer agent (UCP):** find/negotiate/buy/protect across web widget, ChatGPT (MCP+ACP), WhatsApp, voice. Status: ConfirmationCard 2PC exists; MCP server LIVE (`mcp.tawadoo.ma`); NO buy/bid/negotiate agent tools; no A2UI card output. 🔵 mostly planned.
- **Seller agent (Premium):** auto-respond, auto-negotiate, auto-draft, order-confirm; learns the seller's style. 🔵 not built (DraftListingCard missing).
- **Advertiser agent (RTB):** first-party audience sold via display-ads. RTB AuctionService DEPLOYED but ZERO campaigns → serves nothing; attribution unwired. 🟡 built-dormant.

## A7. SEARCH INTELLIGENCE (📄DOC + 🟡 code exists; the read-surface is the gap)
Goal: understand what buyers seek across IMAGE + CATEGORY + LANGUAGE + LOCATION, not just keywords — to feed training/forecasting/reporting. The `searchEnrichment` API module EXISTS (activity-sieve, AI category extraction, cross-lingual tags, Darija lexicon ~639 entries ✅VERIFIED, hybrid keyword+vector search, image→tag, geo-normalization, training-data logger). **THE GAP (founder-flagged):** the Back-Office shows KEYWORD aggregates only — no image/category/language/location search-intelligence read-surface. Engine built, not human-readable. Live-serving status of hybrid/vector search = ⚪ owed.

## A8. COMMERCE & CREATE-LISTING (mostly ✅ working)
create→pay→publish→approve→searchable = proven live end-to-end (✅ "O1 terminal chain"). Coin grant/deduct in BO = super-admin-only + audited (✅VERIFIED this session; also fixed a bug where every coin-op audit was silently failing). AI-listing fixes (condition, price chain, publish-auth, free-listing copy, feed-condition, search-indexing) = ✅ fixed in code. Open edges: video/media same-item guard is client-side only — server/API/mobile/agent can bypass it (🔴 "B1"); a property-scroll bug was NEVER reproduced (⚪, needs live repro — do not "fix" blindly). AI price suggestion V1 exists but is WEAK/thin-data; a state-of-the-art rebuild (external market signals + confirm-edit UX + training) is PARKED ("PRICE-V2").

## A9. SECURITY POSTURE (📄DOC + ✅ partial)
**DONE + verified (frozen):** JWT/OTP auth, CSRF on BO, BO RBAC role matrix (80-item matrix across ~67 BO resources), timing-safe secret compare, rate limiting, wallet pessimistic row-locks, non-root Docker, secrets in AWS Secrets Manager (no hardcoded found), BO IP-restricted, DB migrator/runtime credential split, Payzone signature verification, MCP security closure, private-message no-store + CSP-lock.
**STILL OWED before go-live (the "G2 security bar", each its own session):** authenticated BOLA/IDOR object-level sweep · BO RBAC live re-test on the running system · payment-callback replay hardening · gitleaks full-history secret sweep · OWASP ZAP DAST pass · web CSP + security headers · SSRF/XSS review · disable legacy no-TOTP on staging BO before prod. (📄DOC — "40+" is the union of the 80-item BO matrix + these external classes.)
**Known security gaps (📄DOC):** 2 production security fixes (JwtAuthGuard on message/report; Cache-Control no-store) exist on prod `main` but are MISSING from `Ramzi_V2` — a go-live blocker requiring a founder merge decision (see A11). Guest-no-auth + no-AI-cost-cap on Smart View are intentional staging diffs to close at cutover.

## A10. DISTRIBUTION / DELIVERY / BO (📄DOC)
Feed generation to Meta/TikTok/ChatGPT = code-live. Google Merchant Center = SUSPENDED (appeal pending); syndication code-disabled for GMC. Tawssil (COD delivery) = BUILT but has NEVER carried a real parcel (🟡 unproven). BO = ~67 resources / ~22 pages; coin-ops fixed; analytics-health cockpit + intelligence read-surface have gaps; a live customer-facing 500 exists on `GET /publications/my-publications` (🔴). "Ask Ramzi" (BO internal AI over the lake) = planned/needs work. Store-Videos section and For-You section are TWO SEPARATE main-screen sections (founder-clarified); For-You has no click/conversion tracking yet.

## A11. GO-LIVE BLOCKERS (cutover — must be closed before prod, 📄DOC + ✅ where noted)
1. **§28.5 branch drift (✅VERIFIED):** exactly **8 commits** on `origin/main` are MISSING from `Ramzi_V2`, incl. **2 security fixes** (`6e06278` JwtAuthGuard on message/report; `1c714d6` Cache-Control) + incident hotfixes (blog restore, saved-search-notification disable, content-engine kill-switch, sitemap). Founder decision required; never silently merged. 2. Secret rotation (rotate ALL exposed secrets second-before-prod). 3. Mobile app sync. 4. GMC appeal. 5. DMARC hardening. 6. CNDP (Morocco data-protection) declaration. 7. Prod Aurora Multi-AZ decision. 8. 20K listing reindex/backfill. 9. GA4 MCP receipt (Google auth policy). 10. The whole analytics/intelligence platform activates at the Ramzi_V2 deploy — must be created via the migration path, cohort crash fixed + allowlist pruned first.

## A12. FOUNDER DECISIONS PENDING (surfaced, NOT decided — route to founder, 📄DOC)
- **D1:** what counts as a "convert" per funnel (gates the intelligence/funnel path).
- **D2:** retain raw text + voice utterances for training (legal/CNDP implications).
- Identity model: the analytics user_id column mixes user vs entity IDs — pick one.
- Moderation throughput priority + SLA (a human gate STAYS, always).
- §28.5 merge (which of the 8 commits go to Ramzi_V2).
- Bayesian model (5 sub-decisions). · ACP activation depth. · Price-suggestion-V2 scope. · Design-system/motion un-defer. · Is Tawssil in the "100% functional" bar or post-launch?

## A13. RED LINES (never violate — 📄DOC, founder-locked)
Classic View is sacred (never broken by Smart View work). · Classic and Smart View are SEPARATE implementations (never import Smart's code into Classic). · DB is the system of record; projections (search index, feeds, Amplitude, lake) are rebuildable. · The lake is never a synchronous dependency for commerce. · Money logic stays in the Core with locks, not distributed. · No microservice sprawl (modular monolith). · No production mutation without explicit founder authorization. · No speculative refactor; smallest safe change; prove any broad pattern with one bounded slice first. · MCP tool names/signatures are additive + versioned only (published ChatGPT-app contract). · Human moderation gate always stays. · Every interaction feeds the sovereign lake first, then providers.

## A14. THE PATH TO PRODUCTION (recommended sequence — orchestration guidance)
**Phase A — finish FUNCTIONAL on staging (current):**
1. **CLOSE-U3** — settle (live) whether money/commerce events reach Amplitude, then fix the exact gap (delivery vs identity). Highest leverage. *Recommended next unit — but starts as a read-only DIAGNOSIS because source contradicts the audit (see A3).*
2. Event attribution — thread session/anonymous/interaction id so the ~69%-unattributable becomes attributable.
3. Wire the ~260 paper-only events that map to a REAL user/admin surface; prune genuinely dead ones. (Goal is not "more events" — it's real, landing events.)
4. Funnel assembly (P3 convert-label — gated on founder D1 → P4).
5. Search-intelligence BO read-surface (image/category/language/location, not keywords).
6. Create-journey edges (server-side media guard B1; the scroll bug needs a live repro first).
7. BO intelligence + the customer-facing 500 fix.
8. Fix the cohort crash; populate empty enrichment tables.
**Phase B — security + reconciliation gate:** the G2 security bar (A9) + the §28.5 founder merge decision (A11).
**Phase C — the refactor gate:** reach ~90–95% functional → a fresh crash-proof ARCHITECTURE RE-REVIEW → STRUCTURAL REFACTOR (5-layer modular monolith) → SYNC/REBUILD → PROD cutover (full checklist A11).
**Post-gate / founder-activated:** Bayesian future model, full ACP (buyer transact + seller draft/publish), advertiser/RTB agent, price-suggestion V2, unified behavioral-intelligence asset.

## A15. HOW TO ORCHESTRATE SAFELY (rules for the orchestrator)
- **The latest thing discussed is NOT automatically the next priority.** Classify every item: (A) refactor, (B) stabilization, (C) feature, (D) housekeeping — route, don't auto-execute.
- **NOTICE ≠ ROADMAP · BUILT ≠ LIVE · DEFERRED ≠ REJECTED · INVESTIGATION ≠ IMPLEMENTATION · SPEC ≠ VERIFIED.**
- **Only the founder authorizes the next unit** and any business/policy/legal decision (A12).
- **Verify from source/live before claiming anything is done.**
- **Machine constraint:** at most ~2 concurrent build sessions; prefer 1 writer + 1 read-only; no heavy browser test sweeps (3+ crash the founder's machine).
- **Staging-first, prod protected, no big-bang historical merges** (they caused the most regressions historically).

---

# ══════════════════ PART B — BRAIN RECONCILIATION DETAIL ══════════════════

**Type:** Program recovery + brain reconciliation. READ-ONLY. No implementation, no code, no fired unit.
**Method:** full brain chain B6→B22 (sub-agent deep read, cited) + current source/live git + this session's verified state. The brain is EVIDENCE, not automatically authoritative; reconciled against source.
**This file is the authoritative control plane. Its §2 is being copied to the TOP of `BRAIN_B16_MASTERY_2026_08_31.md`. Historical detail stays below it in the master brain.**

---

## 1. EXECUTIVE TRUTH (≤20 lines — where the program REALLY is)
1. **Mission unchanged and NOT in danger:** refactor Tawadoo V2 into clean, human-quality code. All the recent event/analytics work is SUBORDINATE LEVERAGE inside "all-functional," not a mission change.
2. **We are in the functional-first sub-phase.** The structural refactor has barely begun (~5–10%); codebase-refactored ~20%; whole-program-to-prod ~45–50%.
3. **Commerce spine is mostly solid and proven** (create→pay→publish→approve→searchable live; coin-ops money path just hardened). **Intelligence/moat is the real remaining distance.**
4. **The honest headline the last audits exposed:** the analytics ENGINE is largely built but DISCONNECTED — money events land our lake but not Amplitude; predictions/cohorts/search-intelligence computed but not read; RTB deployed but serves nothing. "Engine behind a bicycle." These are wiring fixes, not rebuilds.
5. **The original B6 ambition (372 events / 25 funnels / 3 agents / 170 signals) is largely ASPIRATIONAL today** (2 of 25 funnels truly complete). It was deliberately demoted to leverage — that is the biggest thing at risk of being silently lost, now surfaced.
6. **Nothing is in flight.** BO-COIN-OPS-FIX accepted (live-verified). EVENT-MASTER-REGISTRY-360 done (mapping).
7. **The refactor is correctly GATED** behind functional ~90–95% + a pre-refactor crash-proof architecture re-review. Do not start it early.
8. **The real risk was never the code — it was the append-only brain losing founder intent.** This session fixes that with a control plane + registers.
9. **Pending founder decisions block the biggest downstream chunk** (D1 convert-definition, D2 retention, identity model, §28.5 security drift, moderation priority).
10. **Default answer to "change the refactor-first strategy?" = NO.** No evidence justifies it.

---

## 2. B22 AUTHORITATIVE CONTROL PLANE (canonical — copied to top of master brain)

### NORTH STAR
Refactor Tawadoo V2 into clean, maintainable, human-quality code — behavior-preserving, no AI fingerprints, crash-proof. Analytics/intelligence/growth = SUBORDINATE LEVERAGE, never the mission.

### CURRENT PHASE
FUNCTIONAL-FIRST sub-phase of REFACTOR-FIRST. Order: all-functional → pre-refactor crash-proof architecture RE-REVIEW (gate) → structural refactor → sync/rebuild app → prod. Prod OUT OF SCOPE until founder opens go-live.

### CURRENT ACTIVE UNIT
NONE (nothing in flight).

### NEXT AUTHORIZED ACTION
NONE authorized yet. (Recommendation in §12 — founder must authorize.)

### NEXT CANDIDATES (ranked, UNAUTHORIZED)
1. CLOSE-U3 — money events land lake but not Amplitude; one delivery/attribution fix makes the whole money spine visible (Class B, highest leverage, ties OPEN-P2-1). 2. OPEN-P2-1 event-attribution investigation (read-only, gates funnels). 3. CLOSE-COHORTS (cohort FK crash every run). 4. UI-TRUTH-VS-LAKE-360 (code+browser evidence pass). 5. BO intelligence read-surface (search-intelligence beyond keywords). 6. Create-journey residual edges.

### COMPLETED + VERIFIED (frozen — do NOT re-open; provenance in VERIFIED-DONE register)
S139 float revert · S140 prod baseline · S141 schema-ownership (S138 crash class CLOSED) · steering swap 00/01/02 · Target Architecture + AD-001 + Frontend design (on paper) · FACE-001=C · AI-listing FIX-001..014 + O1 terminal chain (create→pay→publish→approve→searchable LIVE) · SECURITY-REVERIFY · SOVEREIGNTY-LAKE-FIX · NAV-FIX · SYNC-FIX-S1S2/S4 + U1/U12 · CI-SMOKE-FLAKE-FIX · PHASE-1-INTELLIGENCE-RESTORATION · CORPUS-JOIN P1/P1B/P2 (S3 sessions corpus) · PREDICTION-ENRICHMENT-CRASHPROOF · CREATE-PUBLISH-CONSOLIDATION-FIX · QA-SELLER-FIXTURE · BO-COIN-OPS-FIX (super-admin-only + non-swallowed audit, live).

### BUILT / UNVERIFIED (exists in source, runtime acceptance owed)
Tawssil COD (never carried a real parcel) · FIX-015 Smart-View (landed B18, not re-verified) · RTB AuctionService (deployed, zero campaigns → serves nothing) · search-intelligence enrichment pipeline (captured+landed but BO shows keywords only) · reward-tuple completeness for fine-tuning · ChatGPT-App widget prod status · IVS prod usage.

### STABILIZATION (customer-facing, broken/degraded — landscape, NOT roadmap)
Money events not reaching Amplitude (CLOSE-U3) · ~69% events unattributable (no session/anon/interaction id) · cohort FK crash every run · OPEN-4 /publications/my-publications 500 (live customer 500) · generatePrice silent failure · ai_generation_completed + ai_moderation_triggered fire zero times · session_end ~6% of session_start · ~50 legacy dup event names still landing · OPEN-7 prediction_conversion_scored dropped on uuid "system" (live lake data loss) · create-journey residual (reject-state-machine, S2 embedding drift on edit/delete, B1 server-side media check).

### DEFERRED (NOT rejected — do not resurrect without founder un-defer)
Design-system hygiene/buttons/tokens (D) · motion slice · deploy.yml migrator+CI gate · PRICE-V2 (price-suggestion rebuild) · full per-commit classification of 28 web PORT candidates.

### FROZEN (do NOT reopen)
VERIFIED-DONE register · locked decisions (FACE-001=C, X6, B3, CORPUS-JOIN fork=A, two-sections) · the structural refactor itself (gated) · prod/main (immutable until go-live) · sacred files (§4B/§41).

### BLOCKED / CUTOVER (external/technical — must-not-forget, not now)
§28.5 (8 commits, 2 security, on main missing from Ramzi_V2 — founder decision, go-live blocker) · C-00 secret rotation (second-before-prod) · G2 security bar (BOLA/IDOR/ZAP/gitleaks/web-CSP) · G4 app-sync (mobile) · GMC appeal · DMARC · CNDP declaration · prod Aurora Multi-AZ · 20K reindex backfill · GA4 MCP receipt (Google auth-policy).

### UNKNOWN / CONFLICTED (surfaced, not guessed)
Schema strategy: prod main synchronize:true vs Ramzi_V2 migrationsRun:true — neither is target (AD-001 resolves) · "no DB audit logging" finding (founder-doubted, re-verify before touch) · true % of the 492-event allowlist actually firing+landing (registry mapped, full live-verify owed via UI-TRUTH-VS-LAKE) · sGTM deployed but idle (owner/decision unknown).

### FOUNDER DECISIONS (canonical — L1 authority)
Refactor-first ORDER · prod protected/staging-first · FACE-001=C (Classic+Smart separate) · X6 human moderation gate stays · B3 mirror system/AI events to Amplitude · CORPUS-JOIN fork=A · Store-Videos ≠ For-You (two sections) · coin grant/deduct super-admin-only (Ramzi), no 2nd approver · silently-fail-to-Tawadoo-only never block users · no guaranteed distribution promises · merging historically = most regressions → surgical, no big-bang · LANGUAGE-LEARNING-LOOP = ALL languages not Darija-only · bigger sessions same discipline · user-facing-first (analytics worthless if a real screen doesn't fire+land it) · max 2 concurrent sessions (machine safety) · everything 100% functional before refactor · ACP moved pre-refactor (WAVE 4C). PENDING (§52, undecided): D1 convert-definition · D2 raw-utterance retention · identity entity-vs-user model · moderation priority/SLA · design-system/motion un-defer · §28.5 merge.

### RED LINES (DO NOT VIOLATE)
Classic sacred · never import Smart into Classic · DB=system of record, projections rebuildable · lake NEVER on synchronous commerce path (DB-first→outbox→async workers→lake) · money in the Core with locks, not distributed · no microservice sprawl · the 10 invariants · no prod mutation without founder auth · no speculative refactor / smallest-safe / prove patterns with a bounded slice · deferred stays deferred · inventory≠roadmap · MCP write surface additive+versioned only · Ramzi_V2 single source of truth (no force-push) · sacred files untouched outside dedicated session.

### CURRENT RISKS
1. Analytics era (B6–B15 ambition: 372 events/25 funnels/3 agents/170 signals) being silently lost as "leverage" — SURFACED, needs an honest target reconciliation. 2. §28.5 security drift unresolved = go-live blocker. 3. Append-only brain drift (mitigated by this control plane). 4. Machine crashes on >2 sessions. 5. Owed live-verification on many "built" intelligence pieces.

### LAST VERIFIED STATE
2026-09-05. git: web `36b7183c` · api `bceef76` · bo `07b3a84` · mcp `d8efb4a`, all Ramzi_V2 synced. BO-COIN-OPS-FIX live (back task-def:44). Nothing in flight.

### NEXT SESSION RULE
Read this control plane FIRST. Do NOT let the latest discussed topic become "next." Classify any new item A/B/C/D + state (active/candidate/blocked/deferred/completed/unverified/obsolete/unknown). Check deferred+frozen registers. Verify source before claiming complete. Only the founder authorizes the next unit. NOTICE ≠ ROADMAP.

---

## 3. REFACTOR PROGRESS MAP
- **COMPLETED:** discovery gate (Pre-Arch Truth) → Target Architecture + AD-001 + Frontend (design) → S141 schema-ownership foundation → steering/constitution swap → FACE-001=C. Plus the stabilization/intelligence lineage in COMPLETED+VERIFIED above.
- **CURRENT:** functional-first stabilization (money path, moat corpus, create-journey edges) — NOT structural refactor yet.
- **REMAINING (refactor proper):** the 5-layer modular-monolith restructuring toward Target Architecture (~80–90% of the refactor un-started) — GATED behind functional ~90–95% + crash-proof architecture re-review.
- **CANDIDATES (functional, to reach the gate):** CLOSE-U3, OPEN-P2-1, cohorts, UI-truth-vs-lake, BO intelligence read-surface, create-journey edges, then the architecture re-review.

## 4. CUSTOMER / STABILIZATION MAP (landscape, NOT roadmap)
Commerce spine 🟢 (create/pay/publish/approve/search live; coin-ops hardened). Money analytics 🔴 (lands lake not Amplitude). Attribution 🔴 (~69% unattributable). Search 🟡 (works; intelligence engine built but BO keyword-only). AI-listing cluster: condition+price chain fixed (FIX-001/003), publish-auth fixed (FIX-006), video/media guards partial, price-suggestion V1 weak (V2 parked), scroll #5 not reproduced. Predictions/cohorts 🔴 (computed, unread; cohort crash). RTB/display-ads 🟡 (built, dormant). Store-video/For-You 🟡 (partial). Smart View/ACP 🔴 (weak for real users — founder verdict). Tawssil 🔴 (unproven). GMC 🔴 (suspended/appeal).

## 5. LOST / BURIED INFORMATION (most important — see sub-agent history §5, full detail)
1. 372-event/25-funnel/23-domain/50-signal taxonomy = the DESTINATION, now demoted to leverage; reality 2/25 funnels complete. 2. LAW 15 event-naming (12 rules, one-canonical-name) — no longer surfaced; allowlist drift 478→492→585. 3. 3-AI-agents vision (buyer/seller/advertiser) + ACP pipeline → collapsed to one PARKED row. 4. Attribution-at-every-touchpoint schema plan (14 acquisition cols, UTM everywhere) → only rediscovered as OPEN-P2-1. 5. 170 behavioral signals + 10 Tawadoo-unique innovations → nearly absent. 6. B13 Smart View build queue + "Africa's first agentic commerce" → superseded/demoted. 7. B13 laws §95–§100 (§98 "vous" formal register) → not carried into §0–§52. 8. Tawssil COD → unproven, risk of being forgotten. 9. B13 "17 issues from Ramzi" → asserted done, never itemized. 10. sGTM deployed but idle (no owner). → All now recorded here so they survive.

## 6. CONTRADICTIONS FOUND (see sub-agent §6)
Schema strategy (main synchronize:true vs RV2 migrationsRun:true — neither target) · §28.5 (RV2=truth vs main has 2 security commits RV2 lacks) · B20 "accepted" vs in-flight reality (resolved by B21) · "no DB audit logging" founder-doubted · Smart-View "not weak" verdict REJECTED by founder (L1 wins) · region eu-west-3 vs eu-west-1. None resolved silently; each has an owner/decision above.

## 7. DEFERRED / FROZEN REGISTER
= the DEFERRED and FROZEN blocks in §2. Rule: a deferred item never returns via a different filename/slice/cleanup/convergence; only an explicit founder un-defer reopens it.

## 8. DO-NOT-REDISCOVER REGISTER
Nova Micro never ran from ECS pre-S133 (missing IAM) — all pre-S133 Smart View was staticFallback (B15). · O1 terminal chain PROVEN live E2E (B17). · lake is live + idempotent (Pre-Arch Truth). · AI-listing root causes = ~4 causes / 5 symptoms (AI-LISTING-REGRESSION-001). · S138 boot-DDL crash class CLOSED by S141. · coin-audit was silently swallowed (phantom columns) — FIXED by BO-COIN-OPS-FIX. · money events land the lake but not Amplitude (registry). Recheck only if source changes.

## 9. DO-NOT-FORGET REGISTER (founder intent that must survive every migration)
Refactor-first mission · human-quality code (no AI fingerprints) · staging-first / prod protected · Classic sacred · Smart separate · deferred≠rejected · inventory≠roadmap · source/runtime > checkbox · built≠live · investigation≠implementation · no auto-next-from-newest-topic · founder is final authority · user-facing-first · silently-fail-never-block-users · merging-history=regressions (surgical only) · X6 human gate stays · every interaction feeds the sovereign lake first · cost-first · max 2 sessions (machine safety).

## 10. SOURCE-vs-BRAIN DISCREPANCIES
Allowlist count: brain/registry said 489/492/585 across sessions — real unique count ~489 (method drift, immaterial). · Several "completed" claims (B13 done-list, FIX-015, Tawssil) are asserted in brain but NOT source/runtime-verified → marked BUILT/UNVERIFIED, not COMPLETED. · Everything in the VERIFIED-DONE register was git/live-confirmed by B21 and re-confirmed present this session.

## 11. DRIFT AUDIT (occurred + the mechanism now preventing recurrence)
DRIFT 1 latest-topic→next: occurred repeatedly → prevented by control plane §2 + NEXT SESSION RULE + golden rule. · DRIFT 2 inventory→roadmap: the 372-event/dup-count inventories → prevented by "inventory=evidence" rule. · DRIFT 3 bug→program-replacement: AI-listing cluster → prevented by A/B/C/D classification + "candidate not mission." · DRIFT 7 deferred resurrection → prevented by DEFERRED/FROZEN register. · DRIFT 9 current-state buried under history → prevented by putting the control plane at the TOP. · DRIFT 10 brain-migration loses intent → prevented by this reconciliation + DO-NOT-FORGET register + LOST-INFO surfacing. · B20's FM-1..4 → prevented by the four guardrails.

## 12. RECOMMENDED NEXT UNIT (recommendation, NOT authorization)
**CLOSE-U3 — make the money/commerce events reach Amplitude (and the delivery/attribution layer).**
- **Classification:** B (stabilization) with A-adjacent value (fixes the sovereignty delivery layer, which is architectural).
- **Why it belongs to the program:** the money spine emitting → landing the lake → but NOT reaching providers is the single confirmed break behind "we can see people look, not transact." It restores the sovereignty pipeline's async delivery — a Target-Architecture invariant — and unblocks reporting + the moat + advertiser value at once.
- **Why higher priority than alternatives:** highest leverage (one delivery/attribution fix likely clears the whole money-event set); it is the root the last ~5 sessions kept circling; it gates funnels (P3) and the intelligence read-surfaces.
- **Evidence:** EVENT_MASTER_REGISTRY_360 (U3: money events land lake, absent from Amplitude 205/489); OPEN-P2-1 (~69% unattributable, no session/anon/interaction id); OPEN-7 (prediction_conversion_scored dropped on uuid "system" = live delivery/identity bug).
- **Risk:** LOW-MED — touches the delivery worker / identity threading (additive), not money logic; snapshot + reality-check-before-touch; no big-bang.
- **Expected value:** the money funnel becomes measurable end-to-end in Amplitude + lake; unblocks P3/P4 funnels and the BO intelligence read-surface.
- **Dependencies:** none blocking; D1 (convert-definition) gates the DOWNSTREAM P3, not CLOSE-U3.
- **Explicitly does NOT include:** building bulk coin campaign (WAVE B), the Bayesian model, RTB campaigns, price-suggestion V2, or any refactor. Events/delivery only.
- **NOT authorized — founder decides.** Alternative first move if founder prefers: decide D1 (unlocks the biggest downstream chunk) before CLOSE-U3.

## 13. FRESH-SESSION BOOT PROTOCOL (every future Kiro session, before proposing work)
1. Read the CONTROL PLANE (§2, top of `BRAIN_B16_MASTERY`). 2. Read the RESUME ANCHOR + latest checkpoint. 3. Inspect specific historical decisions only if the current task needs them. 4. Classify the requested/discovered work A/B/C/D + state. 5. Check the DEFERRED + FROZEN registers (do not resurrect). 6. Check whether the item already exists / is already done (DO-NOT-REDISCOVER). 7. Check whether it is actually AUTHORIZED (founder), not merely mentioned. 8. Verify from source/live before claiming any completion (built≠live). 9. Determine whether it affects the refactor mission (default: it doesn't change the mission). 10. Only then recommend ONE bounded next unit — recommendation, not authorization. Machine-safety: max 2 concurrent sessions, prefer 1 writer + 1 read-only, no heavy browser sweeps.

---
## THE GOLDEN RULE (verbatim, binding on all future brains)
> THE LATEST THING DISCUSSED IS NOT THE LATEST PRIORITY. A new discovery can change sequencing, but it does not automatically change the mission. The mission changes only through explicit founder direction. A bug can require fixing. A feature can be valuable. An architectural problem can be urgent. None of these is allowed to erase the program. Every new item must be classified, traced to the program, compared against current priorities, and either incorporated, queued, deferred, blocked, or rejected.
> NOTICE ≠ ROADMAP. INVENTORY ≠ INSTRUCTION. SPEC CHECKBOX ≠ VERIFICATION. BUILT ≠ LIVE. INVESTIGATION ≠ IMPLEMENTATION. DEFERRED ≠ REJECTED. HISTORY ≠ CURRENT STATE. THE BRAIN MUST BE RECONCILED, NOT MERELY APPENDED.


---
# PASS 1 (2026-09-05) — §13 SOURCE-vs-BRAIN VERIFICATION OF EVERY COMPLETION CLAIM
**Method:** each claimed-complete item checked against live git: (a) does the commit EXIST, (b) is it an ANCESTOR of current Ramzi_V2 HEAD (not dangling/stale-branch), (c) does the subject match the claim. Classification: **FACT** (commit exists + ancestor + subject matches) · **INFERENCE** (commit real but runtime acceptance only asserted, not re-proven this pass) · **UNKNOWN/CONFLICTED** · **PROVENANCE-CORRECTED** (brain mislabeled repo/SHA). Live HEADs: web `36b7183c` · api `bceef76` · bo `07b3a84` · mcp `d8efb4a`.

## API — verified commits (all EXIST + ANCESTOR of RV2 HEAD ✓)
| Claim | SHA | Verified subject | Class |
|---|---|---|---|
| SECURITY-REVERIFY | `b190efa` | no-store on private messages + lock CSP on JSON API | **FACT** (source); runtime CSP latent-by-design on staging (DEV_MODE) = INFERENCE |
| SOVEREIGNTY-LAKE-FIX | `8528a05` | (guard-spec lint tail of the 12-site fix) | **FACT** (commit real, ancestor); the 12-site behavior = INFERENCE (asserted) |
| PHASE-1-INTELLIGENCE-RESTORATION | `c64739a` | route mcp/whatsapp events into sovereign lake | **FACT** (source); Amplitude receipt proven B21 = FACT |
| CORPUS-JOIN-BUILD-P1B | `6d1b95a` | thread interaction_id + identity controllers→AI | **FACT** (source + B20/B21 live join proof) |
| PREDICTION-ENRICHMENT-CRASHPROOF | `80de624` | fault-isolate prediction scores | **FACT** (source + B21 live: 19 rows) |
| CORPUS-JOIN-BUILD-P2 | `4542217` | session-sequence artifact in lake | **FACT** (source + B21 live: S3 sessions/*.jsonl) |
| BO-COIN-OPS-FIX (api) | `bceef76` | super-admin gate + non-swallowed correct-schema audit | **FACT** (source + B22 live QA) |
| BO-COIN-OPS-FIX FIX-B (api) | `044cfa1` | user-360 404 not 500 | **FACT** (source) |
| SYNC-FIX-S1S2 | `741efc3` | edits re-enter moderation — never index unmoderated | **FACT** (source) |
| FIX-006 publish-auth | `ab44ad3` | restore JwtAuthGuard on squares/normalize | **FACT** — **PROVENANCE-CORRECTED: this is an API commit; some brain notes filed it under web. It is api.** |
| FIX-007 feed-condition | `a258844` | never advertise used/unknown as 'new' in feeds | **FACT** (api) — **PROVENANCE-CORRECTED (api, not web)** |
| FIX-008 index observability | `9732c80` (+`0380b88` batch-size, +`381d13d` projection recoverable) | make indexing failures observable + self-healing | **FACT** (api). Note: FIX-008 is 3 commits, not 1. |
| FIX-009 money race | `2c6141f` | coins never double-charged on race + no false-dup | **FACT** (api). Sub-SHAs `4c1a16c7/04cc4864/3940e19b` the brain listed = NOT FOUND (mis-transcribed) — the primary fix `2c6141f` is real → cluster CONFIRMED, sub-SHA list was noise. |
| e54b85c (the OVER-BROAD guard that CAUSED the publish block) | `e54b85c` | guard 9 CRUD controllers with SecretKeyGuard (B13-S120) | **FACT** — confirms the FIX-006 root-cause story (this commit swept a seller lookup behind SecretKeyGuard). |
| f62386c (prediction FK guard, pre-dates PREDICTION-CRASHPROOF) | `f62386c` | filter non-existent users in PredictionEnrichment SQL | **FACT** — confirms B21's finding that the FK-safety was ALREADY fixed at B12B-S64, so PREDICTION-ENRICHMENT-CRASHPROOF correctly scoped to fault-isolation only. |

## WEB — verified commits (all EXIST + ANCESTOR ✓)
| Claim | SHA | Verified subject | Class |
|---|---|---|---|
| FIX-001 condition→generate gate | `38ab3123` | selected condition satisfies AI-generate gate | **FACT** (source + B16/B17 live) |
| FIX-002 video/photo same-item | `3758934f` | enforce image/video same-item on all video-attach | **FACT** (source); B1 server-side bypass still open = its own row |
| FIX-003 free-listing count | `42b84c4e` | show remaining free-listing count | **FACT** (source) |
| FIX-007-web mobile publish btn | `589aa9aa` | mobile publish button always clickable | **FACT** (source) |
| FIX-010/011 entity header + scroll | `ff6a0e2b` | send resolved entity header + stop scroll-jump | **FACT** (source) |
| FIX-013 dropdown viewport | `607f5a08` | keep SearchableSelect inside viewport | **FACT** (source) |
| FIX-014 video 120s timeout | `791c661c` | video upload 120s timeout | **FACT** (source) |
| NAV-FIX og:type | `18e51664` | emit og:type=product via raw meta | **FACT** (source) |
| CI-SMOKE-FLAKE-FIX | `60d062a2` | widen Homepage attach budget | **FACT** (source) |
| CORPUS-JOIN-P1 (web) | `7789a6d7` | thread interaction_id through Smart View Brain turn | **FACT** (source) |
| CREATE-PUBLISH-CONSOLIDATION-FIX | `36b7183c` | FIX-1 live guard codec-robust (test-hardening; fix code in 5b020392 chain) | **FACT** (source + B21 live front) |
| QA-SELLER-FIXTURE | `257426c4` | seed Zustand user-storage + auth cookies | **FACT** (source) |

## BO — verified
| Claim | SHA | Class |
|---|---|---|
| BO baseline / firebase mobile intelligence | `ffde480` | **FACT** (HEAD before coin-ops) |
| BO-COIN-OPS CSRF + search shape | `89496a1` | **FACT** (source) |
| BO-COIN-OPS super-admin-only + attribution | `07b3a84` | **FACT** (source + B22 live QA: finance rejected) |

## PASS-1 FINDINGS (source-vs-brain)
1. **Every accepted commit is REAL and an ANCESTOR of the current Ramzi_V2 HEAD** — nothing dangling, nothing on a stale branch. The VERIFIED-DONE register is trustworthy at the git level. **This is the load-bearing reassurance: no accepted work is silently missing from the shipping line.**
2. **PROVENANCE CORRECTION:** FIX-006/007/008/009 are **API** commits; some brain checkpoints implied web. Corrected here. (No functional impact — they exist and are ancestors — but the record now matches source.)
3. **FIX-008 = 3 commits** (`9732c80`+`0380b88`+`381d13d`), not 1; **FIX-009 sub-SHAs `4c1a16c7/04cc4864/3940e19b` do NOT exist** (mis-transcribed in an old checkpoint) — the primary `2c6141f` is real, so the fix is confirmed and the phantom sub-SHAs are noise to ignore.
4. **`e54b85c` and `f62386c` independently corroborate** two root-cause stories (the over-broad SecretKeyGuard that blocked publish; the FK-safety already fixed pre-crashproof) — these move from INFERENCE to FACT.
5. **INFERENCE (real commit, runtime not re-proven THIS pass, relying on prior-session live QA):** SOVEREIGNTY-LAKE-FIX 12-site behavior, SECURITY-REVERIFY CSP (latent on staging by design). These were live-verified in their own sessions (B18/B17) — acceptable, flagged as not-re-proven-in-Pass-1.
6. **BUILT-UNVERIFIED (asserted complete in an OLD brain, NEVER source/runtime confirmed — do NOT treat as done):** B13's "236 events wired / 120 commits / 17 issues" list (never itemized → cannot verify); FIX-015 Smart-View (landed per B18, not re-verified); Tawssil COD (built, never carried a real parcel); RTB AuctionService (deployed, zero campaigns); reward-tuple completeness; ChatGPT-App widget prod status; IVS prod usage. **These are the true "built≠live" set — Pass 2/3 and future units must not assume them done.**

**PASS 1 VERDICT:** the VERIFIED-DONE register is git-solid (every SHA real + ancestor). The real risk is not phantom-completions in the accepted set — it's the OLD-brain asserted-complete items (finding 6) that were never itemized/verified. Those are now explicitly quarantined as BUILT-UNVERIFIED. Pass 2 = the LOST-INFO 6-column table + the AI-listing cluster per-item + the itemized customer register.


---
# PASS 2 (2026-09-05) — §5 LOST-INFO FULL TABLE · §10 AI-LISTING CLUSTER (source-verified) · §9 CUSTOMER REGISTER

## §5 — LOST / BURIED / WEAKENED INFORMATION (the 6-column register)
| Item | Where originally found | What previous Brain said | Current B22 state | Lost/Changed? | Required action |
|---|---|---|---|---|---|
| 372-event / 25-funnel / 23-domain / 50-signal taxonomy = THE DESTINATION | B6 `KIRO_HANDOFF_BRAIN6_TO_BRAIN7` §5; `TAWADOO_V2_FULL_EVENT_TAXONOMY` | "372 is the destination, not 85" — the moat target | Demoted to "SUBORDINATE LEVERAGE"; reality 2/25 funnels complete, ~12 broken, blog fires 0 events | **WEAKENED** (deliberate reframe, but scope now aspirational) | Reconcile the 372 vision vs the 489-allowlist/2-of-25 reality into ONE honest target doc (the EVENT_MASTER_REGISTRY is the vehicle) — founder decides ambition level |
| LAW 15 — event-naming sacred (12 rules, one canonical name, check registry first) | B6 §5; `TAWADOO_EVENT_NAMING_TRUTH` | Top-tier LAW | Survives only implicitly (§48 no-dup-names); allowlist drift 478→492; ~50 legacy dup names still landing | **LOST** (the 12 rules not in §0–§52) | Re-adopt LAW 15 explicitly in steering OR formally retire; enforce naming in CI (currently unenforced) |
| 3 AI Agents (buyer/seller/advertiser) + ACP training pipeline | `TAWADOO_V2_AI_VISION_COMPLETE` (B6 read-list #4 "THE DESTINATION") | The agentic-commerce endgame | Collapsed to one PARKED P4 row ("ACP/Smart-View full read+write"); advertiser agent roadmap-only | **WEAKENED** (P4-parked, was destination) | Keep in DO-NOT-FORGET; founder decides if ACP moves pre-refactor (already flagged WAVE 4C) |
| Attribution-at-every-touchpoint schema (14 acquisition cols on ta_user, UTM on offer/bid/sub/coin/pub/boost, GeoIP, SSO provider) | B6 §5 decision | Concrete schema plan | NOT a tracked unit; rediscovered as OPEN-P2-1 (~69% unattributable) + IDENTITY-CONTRACT-AUDIT | **LOST as a plan** (rediscovered as a gap) | Fold the original schema plan into the CLOSE-U3 / attribution fix scope so it's inherited, not re-derived |
| 170 behavioral signals + 10 Tawadoo-unique innovations (Darija intent, WhatsApp migration, MRE seasonal, auction emotion, sell-to-buy) | `CTO_BEHAVIORAL_INTELLIGENCE_GLOBAL_RESEARCH` (B6 read-list #3) | The behavioral-intelligence moat catalogue | Almost entirely absent B16→B22; only "Darija hardest subset" survives in LANGUAGE-LEARNING-LOOP | **LOST** | Surface as a PARKED catalogue tied to U13 (unified behavioral asset); founder decides which signals are in-scope |
| B13 Smart View build queue (S121–S125: /smart route, buyer guidance AI, voice, seller agent, 60-test QA) + "Africa's first agentic commerce" | `BRAIN_B13_TO_B14_HANDOFF` §1/§5 | Flagship build | Superseded — founder verdict "Smart View does nothing for buyer/searcher/seller, weak" (B17); demoted P4 behind gates | **SUPERSEDED** (intentional) | Keep the founder's "weak" verdict as the authoritative current state; ACP rebuild is the forward path, not the B13 queue |
| B13 laws §95–§100 (§96 z-index audit, §98 "vous" formal register, §99 360-impact-audit-before-fix, no-web-component-without-approval) | `BRAIN_B13_TO_B14_HANDOFF` §3 | Binding laws | §96 partially survives as §33.6; **§98 "vous" register appears NOWHERE**; §99/component-approval not consolidated | **LOST** (§98 fully; others partial) | Re-adopt §98 (formal "vous" user-facing copy standard) + §99 into steering, or formally retire with reason |
| Tawssil COD delivery (V3 architecture, sent to Yassine) | `TAWADOO_TAWSSIL_INTEGRATION_ARCHITECTURE_V3`; B15 §1 | Large integration effort | BUILT-UNPROVEN (never carried a real parcel); parked line | **AT RISK** (functional-completeness gap could be forgotten) | Add to BUILT-UNVERIFIED register (done); founder decides if it's in the "100% functional" bar or post-launch |
| B13 "17 issues from Ramzi" + B13 UX fixes | `BRAIN_B13_TO_B14_HANDOFF` §5 | "done" | Asserted complete, never itemized forward | **UNVERIFIABLE** (no itemized list) | Cannot verify; mark UNKNOWN; if a specific issue resurfaces, treat as new |
| sGTM (server-side GTM) deployed but idle | B6; `TAWADOO_V2_PRE_ARCHITECTURE_TRUTH_REPORT` §7; `B15_TO_B16` ECS table | Deployed infra | Running + costing, no owner/decision through the whole chain | **BURIED** (orphan infra) | Assign an owner/decision at PROVIDER-STRATEGY-360 or cutover; is it used by any live tag? verify |
| Two law lineages (B6 "16 LAWS"+LAW 11/15/16 vs current §0–§52) | B6 master `KIRO_CTO_BRAIN_MASTER`; steering `00` | Two separate law sets | Only §0–§52 is active; several B6/B13 laws not carried | **PARTIAL LOSS** | One-time law reconciliation: adopt-or-retire each orphaned B6/B13 law explicitly |

## §10 — AI-LISTING REGRESSION CLUSTER (per-item, SOURCE-VERIFIED this pass)
| # | Item | Symptom | Root cause | Confirmed? | Source evidence (this pass) | Runtime evidence | Fix status | Regression guard | Remaining |
|---|---|---|---|---|---|---|---|---|---|
| 1 | Condition / état / new-used recognition | AI-generate gate blocked; condition not recognized; condition could drop from feeds | condition lived in separate ConditionSection state, not linked to the generate-gate; inconsistent code matching (etat/condition/state) vs backend canonical `condition` | **YES** | `utils/condition-property.ts` resolver EXISTS + `__tests__/condition-property.test.ts` | FIX-001 `38ab3123` live-verified B16/B17 (Chromium+WebKit) | **FIXED (FACT)** | condition-property.test.ts + entity-identity-bug-condition.property.test.ts | none (accepted) |
| 2 | Video-vs-product identity verification | wrong video attached to a product; silent `sameItem:true` on error | cross-check only on proceed path; skip paths bypassed; error defaulted to pass | **PARTIAL** | `utils/media-cross-check.ts` EXISTS; doc-comment: backend `publications/check-multi-item` "deliberately NOT relied on" — client enforces | FIX-002 `3758934f` live | **FIXED client-side (FACT); server-side bypass OPEN** | media-cross-check.test.ts | **B1: server-side/API/mobile/agent bypass the client-only guard — OPEN, its own unit** |
| 3 | First-user free-listing counter | "5 of 5 used" shown to a NEW seller (display inversion, not a block) | copy-vs-variable inversion | **YES** | en.json:950 `slots_remaining` = "{count} of 5 ... remaining"; :952 `listing_limit_reached` = "5 of 5 used" (correct); FR parity :927–929 | FIX-003 `42b84c4e` | **FIXED (FACT)** | i18n framing guard (16 tests per B16) | none |
| 4 | AI price suggestion | price card not showing | downstream of #2 — generate gated on condition; condition not detected → onAIGenerationSuccess never fires → generatePrice never runs. NOT independent. | **YES (as downstream)** | `useAIGeneration.ts:168` `generatePrice` returns `PriceSuggestion \| null`, wrapped in try; called after generate success | V1 works when generate fires (post-FIX-001) | **UNBLOCKED by FIX-001 (FACT)** BUT V1 is WEAK (thin data) | — | **PRICE-V2 rebuild = PARKED capability (external signals + confirm-edit UX + training); generatePrice silent-failure path (returns null quietly) = small stabilization** |
| 5 | Property-selection scroll/focus (jump to footer) | page scrolls to footer on property select | NOT REPRODUCED in the investigation | **NO — NOT REPRODUCED** | basic-info-section.tsx / category-section.tsx touch scroll; FIX-010/011 `ff6a0e2b` "stop brand/keyword scroll-jump" partially addressed | none | **PARTIAL / UNCONFIRMED** — do NOT fix with another arbitrary z-index | mobile-publish-button-clickable.test | **Needs a live Chromium+WebKit repro before any fix (still read-only)** |
| 6 | (discovered) publish blocked C-1 | 401 "Clé secrète invalide" on publish | `e54b85c` over-broad SecretKeyGuard swept the seller lookup `squares/normalize` | **YES** | FIX-006 `ab44ad3` restored JwtAuthGuard on normalize; `e54b85c` confirms the cause | O1 terminal chain proven live B17 | **FIXED (FACT)** | square-controller-security.spec | prod normalize currently unauthenticated = cutover awareness (applies when prod ships from RV2) |
| 7 | (discovered) feed condition misrepresentation | Arabic "used" listing syndicated as NEW (GMC-appeal pattern) | `mapCondition() \|\| 'new'` default; no Arabic-script keys | **YES** | FIX-007 `a258844` "never advertise used/unknown as 'new'" | contained (GMC push disabled; Meta/TikTok feeds generate) | **FIXED (FACT)** | guard added | verify Arabic-script coverage complete |
+ **FIX-008 search-indexing** (`9732c80`+`0380b88`+`381d13d`): pushToIndex was swallowing errors; made observable + self-healing + reconciliation + alarm. FIXED (FACT). Remaining: published-but-unindexed COUNT never measured.

## §9 — CUSTOMER / STABILIZATION REGISTER (itemized landscape — NOT the roadmap)
- **Create-listing journey:** condition/price chain FIXED · publish-auth FIXED · video/media client-side FIXED (server-side B1 OPEN) · free-listing copy FIXED · scroll #5 UNCONFIRMED · full one-click publish proven live (O1).
- **Search:** keyword search works live; hybrid/vector + cross-lingual + geo + image enrichment BUILT (searchEnrichment module) but BO shows keyword aggregates only (read-surface gap); image-search endpoint exists (SearchByImageButton, 5MB Bedrock cap); zero-result logging exists.
- **Price:** V1 generatePrice live but weak/thin-data; PRICE-V2 rebuild PARKED.
- **Money events:** all fire + land lake, none reach Amplitude (CLOSE-U3).
- **Attribution:** ~69% unattributable (no session/anon/interaction id).
- **Predictions/cohorts:** computed hourly, no read endpoint; cohort FK crash every run; conversation intents write-only.
- **RTB/display-ads:** AuctionService deployed, reads lake for intent, ZERO campaigns → serves nothing; attribution call-site unwired.
- **Store-video / For-You:** two separate main-screen sections (founder-clarified); store-video events fire live; For-You click/conversion tracking gap.
- **Smart View / ACP:** weak for real users (founder verdict); no A2UI output; no DraftListingCard write surface; MCP has no bid/buy/publish tools; ConfirmationCard 2PC exists.
- **Distribution:** feeds generate (Meta/TikTok/ChatGPT); GMC suspended (appeal); syndication code-disabled for GMC.
- **Tawssil COD:** BUILT-UNPROVEN (never carried a real parcel).
- **BO:** coin-ops FIXED (super-admin-only, audited); analytics-health cockpit + intelligence read-surface gaps (FIX-C/D/E queued); OPEN-4 /publications/my-publications 500 (customer-facing).
- **Notifications/messaging:** WhatsApp send never routes to ingestion (bypasses lake); notification click = generic nav_click only.
- **Auth/onboarding:** frictionless auth live; legacy no-TOTP on staging BO (must be off in prod).
- **Lucky-wheel / watch-to-earn / referral-rewards:** present in code; live-firing status UNKNOWN (registry-owed).

**PASS 2 VERDICT:** the AI-listing cluster is genuinely fixed where claimed (source-verified), with TWO honest remainders — #2 server-side bypass (B1) and #5 scroll (NOT REPRODUCED, needs live repro). The LOST-INFO register now carries the 10 buried items with required actions — the biggest being the 372-event ambition reconciliation and the two orphaned law sets (LAW-15 naming, §98 "vous"). Customer register itemized and clearly separate from the refactor roadmap. Pass 3 = refactor-journey per-session Q&A + contradiction-resolution column.


---
# PASS 3 (2026-09-05) — §7 REFACTOR-JOURNEY PER-SESSION Q&A · §6/§14 CONTRADICTION RESOLUTION (live-verified)

## §7 — REFACTOR JOURNEY, per major milestone (problem → why → changed → completed? → evidence → follow-up → still relevant)
1. **Original objective (B6–B13, pre-refactor era).** *Problem:* build the sovereign data/intelligence platform (372 events, 3 agents, Smart View). *Why:* the moat thesis. *Changed:* event pipeline, TrainingShadow, Smart View scaffolding. *Completed?* No — task-chasing without an architectural spine. *Evidence:* B6–B13 handoffs. *Follow-up:* founder reframed to refactor-first at B16. *Still relevant?* As LEVERAGE only; the ambition is the LOST-INFO #1 item to reconcile.
2. **Smart View push + hard truth (B13–B15).** *Problem:* ship agentic commerce. *Changed:* S121–S125 build. *Completed?* NO — UX 2/10, **Nova Micro never ran from ECS (missing IAM) → all pre-S133 Smart View was staticFallback** (DO-NOT-REDISCOVER). *Evidence:* `B15_TO_B16` §1. *Follow-up:* born §47 browser-verify, §48 no-collision, §49 no-guessing. *Still relevant?* Smart View demoted P4; founder "weak" verdict stands.
3. **Discovery gate — Pre-Architecture Truth Report (B16, 2026-09-01).** *Problem:* verify system-not-story before touching. *Changed:* classified prod vs RV2 vs target; found schema split, the 2 unported security commits, lake live+idempotent, modular-monolith seams. *Completed?* YES (read-only). *Evidence:* `TAWADOO_V2_PRE_ARCHITECTURE_TRUTH_REPORT`. *Follow-up:* AD-001 + synthesis. *Still relevant?* YES — foundation of the refactor.
4. **Architecture synthesis + Target Architecture (2026-09-01).** *Problem:* define the target shape. *Changed:* approved 5-layer modular monolith (Face/Core/Helpers/Plugs/Brain) + 10 invariants + SoR-vs-Projections. *Completed?* Design on PAPER; nothing built. *Evidence:* `TAWADOO_V2_TARGET_ARCHITECTURE`. *Follow-up:* the structural refactor (still ~un-started). *Still relevant?* YES — it IS the refactor target.
5. **DB/schema foundation — AD-001 → S141.** *Problem:* runtime user can't DDL → S138 boot crash blocks all schema work. *Changed:* migrator/runtime credential split; `migrationsRun:false`. *Completed?* YES, LIVE on staging (S138 crash class CLOSED). *Evidence (this pass):* `app.module.ts` = `synchronize:false` + `migrationsRun:false` CONFIRMED. *Follow-up:* deploy.yml migrator + CI gate DEFERRED. *Still relevant?* YES — the only refactor foundation actually built.
6. **Steering/governance swap.** *Problem:* stop drift, make brain a control system. *Changed:* issued constitution + steering 00/01/02; hooks emptied. *Completed?* YES. *Evidence:* `.kiro/steering/`. *Follow-up:* this reconciliation (the brain-as-control-plane). *Still relevant?* YES.
7. **Frontend architecture + FACE-001.** *Problem:* converge Classic+Smart or keep separate? *Changed:* audit → **decision C (separate, shared foundation only)**. *Completed?* Decided/locked. *Evidence:* `B16_FACE_001_CONVERGENCE_AUDIT`. *Follow-up:* red line "never import Smart into Classic." *Still relevant?* YES — a red line.
8. **AI-listing regression cluster (B16/B17).** *Problem:* create-listing broken (condition, video, free-count, price, scroll, publish-auth). *Changed:* AI-LISTING-REGRESSION-001 investigation → FIX-001..014 + O1 terminal chain proven live. *Completed?* Accepted (staging); 2 remainders (B1 server-side, #5 scroll). *Evidence:* Pass-2 cluster table (source-verified). *Follow-up:* B1 + #5 + PRICE-V2. *Still relevant?* YES — stabilization inside functional-first.
9. **Prod gates surfaced (B16/B17).** *Problem:* what blocks go-live? *Changed:* SYNC (G1), SECURITY (G2 5.5/10), SECRET ROTATION (G3 cutover), APP SYNC (G4), §28.5 drift. *Completed?* Gates identified; some closed (SYNC-FIX, SECURITY-REVERIFY). *Evidence:* `B16_TO_B17` §8/§12. *Follow-up:* cutover set. *Still relevant?* YES — go-live blockers.
10. **Analytics reconciliation + moat (B18–B22).** *Problem:* is the intelligence real? *Changed:* ANALYTICS-TARGET-RECONCILIATION → 6-phase backlog; PHASE-1-INTELLIGENCE-RESTORATION; CORPUS-JOIN P1/P1B/P2 (S3 corpus). *Completed?* P1/P1B/P2 accepted live; P3 gated on D1. *Evidence:* Pass-1 (commits FACT). *Follow-up:* CLOSE-U3, funnels, read-surfaces. *Still relevant?* YES — but SUBORDINATE leverage.
11. **B20 drift + B21 recovery + BROKEN-CHAIN PROTOCOL.** *Problem:* B20 broke the chain (QA'd in-flight, handed founder tasks, skipped reads). *Changed:* 4 guardrails, VERIFIED-DONE frozen, reality-check-before-touch, no historical merge, refactor stays gated. *Completed?* B21 reconciled to one story. *Evidence:* `B20_TO_B21`, `B21_TO_B22`. *Still relevant?* YES — binding.
12. **Design-system discussion.** *Problem:* button/token consolidation. *Changed:* nothing. *Completed?* DEFERRED (Class-D, no proven payoff). *Evidence:* ledger. *Still relevant?* Only on founder un-defer.
13. **B22 event/registry sessions + this reconciliation.** *Problem:* the "engine behind a bicycle" + append-only brain losing intent. *Changed:* EVENT_MASTER_REGISTRY + this control plane. *Completed?* Reconciliation = this doc; registry = mapping done. *Still relevant?* YES — current.

**Current position:** functional-first sub-phase; codebase-refactored ~20%; **structural refactor barely started (only S141 foundation built) — ~5–10%**; whole-program ~45–50%. Commerce spine 🟢; intelligence/moat = the real distance; the 5-layer restructuring is the bulk of the un-started refactor, correctly GATED.

## §6 / §14 — CONTRADICTION REGISTER (with authoritative resolution, live-verified this pass)
| Conflict | Sources | Which is authoritative | Why (live evidence) | Brain action |
|---|---|---|---|---|
| Schema: synchronize:true vs migrationsRun:true (both called unsafe) | Pre-Arch Truth §1/§22 (prod main synchronize:true); §35 (some services migrationsRun:true) | **Ramzi_V2 current = `synchronize:false` + `migrationsRun:false` (migrator-owned)** | VERIFIED this pass: `app.module.ts:120` synchronize:false, :126 migrationsRun:false. The "migrationsRun:true crashed (S138)" is HISTORY — S141 fixed it. Prod main synchronize:true is the LEGACY-unsafe side. | Resolved: RV2 schema strategy is correct/target-aligned; prod main is the unsafe legacy to reconcile at cutover. Not a live contradiction on RV2. |
| §28.5: "Ramzi_V2 is the single source of truth, ahead of everything" vs "8 commits (2 security) on main missing from RV2" | steering `00` §28 vs Pre-Arch Truth §3 / `B16_TO_B17` §12 | **BOTH true — it IS a real drift = go-live blocker, founder decision** | VERIFIED this pass: exactly **8 commits** on origin/main not in origin/Ramzi_V2, incl. `6e06278` (JwtAuthGuard on message/report), `1c714d6` (Cache-Control no-store), + incident hotfixes (blog restore `a1987ae`, saved-search-notif disable `dd7cdf3`, content-engine kill-switch `d58813c`, sitemap/PUBLISHED fixes). | Founder decision required (§28.3 — never silent-merge). The 2 security + the hotfixes must be evaluated for RV2 before go-live. HARD cutover gate. |
| B20 "accepted (front)" vs still-in-flight reality | `B20_TO_B21` FM-1 vs `B21_TO_B22` §2 | **B21's corrected verdict** | B20 claimed accepted while session in-flight; B21 verified live + closed properly. | Resolved (B21). Prevented forward by G1 (no QA in-flight). |
| "no DB audit logging" — founder-DOUBTED vs audit-asserted | SECURITY-AUDIT vs founder | **UNKNOWN until re-verified from source** | Not re-verified this pass. | Mark UNKNOWN/CONFLICTED; re-verify from source before any action (do not touch on assumption). |
| Smart View "not weak" (SMART-VIEW-GAP-ANALYSIS) vs founder "weak for real users" | `B17_TO_B18` §7 (analysis) vs founder | **Founder (L1 authority)** | Founder decision overrides an analysis verdict. | Smart View = weak-for-real-users is the authoritative current state; ACP rebuild (WAVE 4C) is the path. |
| Region eu-west-3 vs eu-west-1 | tooling default vs deploy config | **eu-west-1 (source)** | VERIFIED this pass: deploy.yml uses `eu-west-1` throughout. eu-west-3 was only a tool default, never real config. | Resolved: eu-west-1 is truth; ignore the tool default. |
| Session isolation: parallel-safe vs both touched ar.json (S136+S137) | `B15_TO_B16` B16 update | **Serialize locale-touching sessions** (founder order) | Near-miss caught. | Standing rule: never run two locale-touching sessions in parallel. |

## RECONCILIATION COMPLETE — closing statement
All 13 required sections are now produced across the control plane (§2 in this file + top of master brain) and Passes 1–3:
- §1 Executive Truth · §2 Control Plane · §3 Refactor Map · §4 Customer/Stabilization Map — in the head of this file.
- §5 Lost/Buried (6-col) · §10 AI-listing cluster (per-item, source-verified) · §9 Customer register — Pass 2.
- §13 Source-vs-Brain (every commit git-verified) · §8 Do-not-rediscover · §18 Do-not-forget — Pass 1 + head.
- §7 Refactor journey (per-session Q&A) · §6/§14 Contradictions (resolved, live-verified) · §11 Drift audit · §12 Recommended next unit · §13 Fresh-session boot protocol — Pass 3 + head.
**Verification standard met:** every completion claim checked against live git; every contradiction grounded in current source; nothing marked "done" that source didn't support (BUILT-UNVERIFIED set quarantined); the 372-vision demotion + 2 orphaned law sets surfaced as required actions. The brain is now RECONCILED (control plane at the top of BRAIN_B16_MASTERY), not merely appended.
**Recommended next unit remains CLOSE-U3 (recommendation, NOT authorization).** Founder authorizes. Default answer to "change refactor-first?" = NO.
