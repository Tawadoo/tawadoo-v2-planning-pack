# TAWADOO V2 — TARGET ARCHITECTURE (approved design direction)

**Date:** 2026-09-01
**Status:** DESIGN APPROVED (founder, plain-language review). DESIGN ONLY — nothing built yet. Staging clean.
**Grounded in:** verified source code + 2026 best-practice web research (cited in Brain). Not opinion, not assumption.
**Governing:** `TAWADOO_V2_ARCHITECTURAL_SYNTHESIS_MANDATE_2026_09_01.md` + laws §0–§51 + Three Commandments.

---

## 1. THE SHAPE (5 layers)
A well-organized single system ("modular monolith"), NOT 20+ microservices. 2026 research: modular monolith is the right call for teams under ~50 engineers; 42% of orgs that went microservices reversed it. Tawadoo team = founder + Kiro → simple wins.

1. **FACE** — web + app, Classic View + Smart View, messages, notifications.
2. **BUSINESS CORE** — accounts, listings, buy-now/offers/auctions, orders, wallet+coins, payments, delivery. Stays ONE tight core. Instant + never wrong.
3. **HELPERS (workers)** — search indexing, video processing, distribution feeds, store-video reels, content/blog, notifications/WhatsApp/email, lake export. Background; slow/down ≠ shop down.
4. **PLUGS (adapters)** — Amplitude, GA4, Meta, Google, TikTok, GMC, Bing, WhatsApp, SendGrid, Firebase, Payzone, Tawssil, Bedrock/OpenAI, MCP/ChatGPT app. Any one breaks → Tawadoo keeps running.
5. **BRAIN/MOAT** — your DB (sovereign record) → S3 lake → future AI. Underneath everything.

**The tie-together rule:** money/deals in the Core (fast, safe); everything else happens AFTER, in the background, never in the customer's way.

## 2. FEATURE SORTING (verified against code)
- **CORE (instant, never wrong):** accounts, listings, buy-now, orders, offers, auctions, wallet+coins, payments. Smart View sits in the Face and talks to this same Core (same shop, nicer window).
- **HELPERS (background):** search indexing, video processing, distribution feeds, store videos→reels, blog/content, notifications+WhatsApp+email, lake export, delivery messages.
- **PLUGS (outside, behind adapters):** all external providers above.

**Two verified nuances (from code, not assumed):**
- Search: reads pre-built OpenSearch index (fast) + a LIVE AI call to vectorize the query. Code already falls back to keyword-only if AI slow/down. Best practice confirms: keyword = reliable base, vector = enhancement/fallback. INVARIANT: search never fully breaks.
- Money: NOT one giant transaction — wallet uses a pessimistic row-lock (SELECT FOR UPDATE); payments run a transaction the wallet can join so payment+coin-credit commit together; orders rely on wallet's own locking. This is the correct modular-monolith money pattern. INVARIANT: keep money in the Core, protected by locks + shared transactions; do NOT re-architect or distribute it.

## 3. THE 10 RULES WE NEVER BREAK (invariants — from code + laws)
1. Classic View keeps working. 2. Login/accounts always work. 3. Money never wrong (locks stay). 4. Payments verified (Payzone callback signatures). 5. Orders follow their steps (state machine). 6. Every action → DB first, then copied out (sovereignty; Amplitude/lake get copies only). 7. Search never fully breaks (keyword fallback). 8. Outside company down ≠ shop down. 9. Production protected. 10. Every change reversible + staging-tested first.
Any refactor step that would break one of these is blocked by rule.

## 4. DATA OWNERSHIP (Systems of Record vs Projections)
- **System of Record (the truth, cannot be casually rebuilt):** PostgreSQL (accounts, listings, wallet, orders, offers, auctions, payments) + the canonical event store (`ta_analytics_event`).
- **Projections (rebuildable copies):** OpenSearch index, distribution feeds, Amplitude, GA4, caches, the S3 lake.
- **Rule:** projections can be rebuilt from the record; the record cannot. Protect the record; treat everything else as replaceable.

## 5. SOVEREIGNTY = ASYNC, NOT REQUEST-PATH (verified + corrected)
Order: something happens → write your DB first (sovereign) → transactional outbox (`ta_analytics_delivery`) → background workers deliver copies to lake/Amplitude/AI. The lake is NEVER a synchronous dependency for commerce. Verified live: this already works (DB-first, outbox, daily lake export at 01:00 UTC, idempotent). 2026 best practice = exactly the transactional-outbox pattern.

## 6. THE FIRST SAFE STEP — "who is allowed to change database tables"
- **What it is (plain):** today the app tries to change the database structure itself at startup, but lacks permission → crashes. This already caused the S138 incident. It blocks EVERY future improvement that needs a DB change.
- **Verified current state:** `app.module.ts` = `synchronize:false` + `migrationsRun:true` (the crash). AND B11 already STARTED the fix locally (`B11-SEC-DB-CREDENTIAL-C1`, migration `1787900000000-ExtractRuntimeSchemaDDL.ts`) but NEVER finished it (no production run, no CI, no staging rollout, no acceptance). So this is COMPLETING a half-done piece, not greenfield.
- **The fix (direction, approved):** separate the "change-the-database key" (migrator, tightly controlled, table-owner) from the "run-the-app key" (runtime, no DDL). App boots WITHOUT trying to change tables (`migrationsRun:false`); changes run on purpose via the migrator key through a bounded/controlled step. 2026 best practice confirms: separate owner role from runtime role; migrator credential must be tightly scoped, not a standing all-powerful password.
- **Why first:** it's the foundation — nothing else that touches the DB can safely happen until this is fixed. Small, reversible, unblocks everything (seoTitle, image backfill, feed-safety re-land, all schema work).
- **Prod reference:** prod avoids the crash by using `synchronize:true` (auto-sync) — but that's UNSAFE as a target (can silently drop columns). The proper fix is the migrator separation, not copying prod.

## 7. WHAT STAYS EXACTLY AS-IS (do not touch)
The whole business Core (accounts/listings/money/orders/auctions/offers) is well-built and stays. The sovereignty pipeline works and stays. Search works and stays. The refactor ORGANIZES and adds the missing platform foundation (the DB-key fix, IaC, deploy discipline) — it does NOT rebuild what works.

## 8. HARD STOP
This is the approved design. Detailed step-1 plan (AD-001) is next — DESIGN ONLY, still nothing built. Then bounded implementation, one careful step at a time, staging-first, always reversible, awaiting founder go at each consequential gate.
