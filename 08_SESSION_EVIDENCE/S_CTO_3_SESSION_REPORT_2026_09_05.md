# S-CTO-3 — END-OF-SESSION REPORT (for the Brain)

- **Session:** S-CTO-3 — CLOSE-U3 BUILD (money-event identity fix)
- **Date:** 2026-09-05
- **Mode:** MODE B build unit (founder-authorized), api-only writable, STAGING only. Prod untouched.
- **Repo written:** `tawadoo_api_js` only. web / bo / mcp: read-only, untouched.
- **Proposed status:** `FINISHED — COMPLETE` (code fix + deploy + full authenticated E2E live proof). **Independent QA to accept.**
- **Commit:** `ab37f4b5da05e41e7a5d08d5f3257631079cca2c` on `Ramzi_V2` (pushed, in sync with origin, integrated on the single-source line — §28 satisfied).
- **Rollback point:** `bceef760…` (pre-session HEAD).
- **Evidence files:** `CLOSE_U3_IDENTITY_FIX_EVIDENCE_2026_09_05.md` (full ladder + §15 E2E proof). Diagnosis input: `CLOSE_U3_DIAGNOSIS_EVIDENCE_2026_09_05.md` (S-CTO-1).

---

## 1. WHAT THE SESSION WAS

Fix the S-CTO-1 root cause (b): money/commerce server events were emitted with the **entity id** (`ta_entity.id`) in the Amplitude `user_id` slot, making the money spine unusable for user-level funnels/cohorts. Every money emitter now stamps the **acting user id** (buyer, or seller *owner* user) while the entity id is preserved in `properties` under its **existing** key. No pipeline shape change. No new names invented.

---

## 2. WHAT WAS BUILT (BACK-END — tawadoo_api_js) — DONE + VERIFIED

8 files changed + 1 new fail-first spec. Each money emitter's 2nd `trackServerEvent` arg (= the Amplitude `user_id`) changed from an entity id to the acting user id; entity id moved to an existing property key.

| Area | Events fixed | user_id now | Entity id kept under |
|---|---|---|---|
| orders.service.ts | order_placed | `dto.buyerUserId` | buyer_entity_id, seller_entity_id |
| | order_cancelled (buyer) | `userId` | buyer_entity_id |
| | order_cancelled (seller) | seller `entity.owner.id` (lookup) | seller_entity_id |
| | order_confirmed_by_seller | `sellerEntity.owner.id` | seller_entity_id |
| | delivery_quote_requested/received/failed (×9 sites) | `ctx.buyerUserId` | seller_entity_id + buyer_user_id (already in props) |
| | trackOrderTransition (6 events: order_request_submitted, seller_order_confirmed, seller_order_declined, tawssil_shipment_creation_requested, buyer_final_order_confirmed, buyer_final_order_declined) | resolved actor→owner (buyer via order.buyer_user_id; seller via entity.owner) | seller_entity_id, buyer_entity_id |
| | order_shipped / in_transit / returned / delivered (Tawssil webhook) | `order.buyer_user_id` | seller_entity_id, buyer_entity_id |
| offer.service.ts | offer_received / offer_viewed_by_seller / offer_accepted | seller `entity.owner.id` | seller_entity_id |
| bid-transaction.service.ts | bid_outbid | `previousBidder.owner.id` | previous_bidder_id (already) |
| bid-entity.service.ts | bid_room_joined | `entityModel.owner.id` | (no prop existed; NOT invented) |
| store-boost.service.ts | boost_purchased (store) | `entity.owner.id` | seller_entity_id |
| publication-boost.service.ts | boost_purchased (listing) | `publication.entity.owner.id` | seller_entity_id |
| syndication-subscription.service.ts | subscription_purchased | `userId` | — |
| guest-orders.controller.ts | guest_order_placed | `linkedBuyerUserId` (owner if account-matched, else null — never faked) | guest_id (already) |

Already-correct (verified, unchanged): coin_earned/coin_spent, subscription_renewed/upgraded/downgraded/expired.

**Verification level reached (back-end): 5/5 (exists → wired → reachable → runtime-verified → accepted-pending-QA).**
- Build/typecheck GREEN; 0 new lint errors; fail-first spec RED→GREEN; 9 touched-area suites / 100 tests PASS.
- CI (real test jobs) GREEN; deployed digest `sha256:bde10f1f…` = commit on BOTH running ECS tasks.
- **Live E2E:** authenticated test user bought a store boost (spent 1000 coins) → `boost_purchased` landed in the lake with `user_id` = real user (resolves in ta_user, NOT ta_entity) + `seller_entity_id` property, delivered 0 DLQ, and **confirmed in Amplitude** under that user (amplitudeId 1702624709476). Plus 2 anonymous delivery_quote_* types post-deploy showed 0 entity-keyed.

---

## 3. WHAT WAS **NOT** BUILT / **NOT** VERIFIED — by layer

### 3.1 FRONT-END (tawadoo_web_js) — NOT TOUCHED, NOT VERIFIED
- **No front-end change was needed or made.** These are server-emitted sovereign events; the fix is entirely at the server emit boundary. The front-end does not set the `user_id` for these events.
- **NOT verified via browser (Chromium/WebKit):** I did NOT drive the actual web UI (signup → add coins → boost) in a browser. Reason: staging `/auth/register` hangs (see 4.1) and the money action was exercised via the real authenticated **API** instead. The user-facing web flow for store-boost was NOT visually confirmed this session. → **Open item OPEN-U3-1.**
- Front-end money-event emission (if any events are fired client-side with the SDK, e.g. `begin_checkout`, `coin_pack_viewed`, `boost_popup_opened`) was OUT OF SCOPE and unchanged; those already carry browser/user identity via the web SDK.

### 3.2 DATABASE
- **No schema/DDL change** (code-only fix). No migration. Forward-safe.
- `deriveEventId` now hashes the user id instead of the entity id → **dedup keys differ going forward** (no historical collision; expected).
- **Historical rows NOT backfilled:** the ~19 pre-deploy money events keyed by entity id remain mis-keyed in `ta_analytics_event` and in Amplitude history. Forward-fix only. → **Parked: OPEN-U3-2 (historical backfill).**
- Test data created on staging (harmless, tagged): user `520d75eb-…`, entity `fbfa1da7-…`, wallet `59689` (1100 coins), store boost `id=3`, wallet_history `scto3_test_grant`. **Left in place** (staging is founder's build/test env). → **Optional cleanup: OPEN-U3-3.**

### 3.3 BACK-OFFICE (admin_bo_tawadoo) — NOT TOUCHED
- No BO change. The money-event identity is a runtime emit concern, not a BO surface. No BO verification performed or needed.

### 3.4 AWS / INFRA
- ECS service `tw-staging-svc-back` redeployed via CI (task-def `tw-staging-task-back:44`, mutable `staging-v2` tag — §42 respected). Rollout COMPLETED, 2/2 tasks healthy, digest verified.
- **ECS Exec data channel is DOWN** on `tw-staging-svc-back` (both tasks return `TargetNotConnectedException` — agent RUNNING but ssmmessages data channel not connected). Pre-existing (S-CTO-1 flagged it). I worked around it with bounded `ecs run-task` reads/seeds. → **Open item OPEN-U3-4 (ops hygiene).**
- No IAM change, no new paid resource, no standing resource. Bounded Fargate tasks (2 reads + 1 seed + 1 verify) auto-stopped. No secrets printed.

### 3.5 AMPLITUDE
- Confirmed `boost_purchased` for the test user is present and correctly keyed. `order_id` may still be registered as a **user-scoped** property in the taxonomy (S-CTO-1 BLUE/R3) — a downstream symptom of the old keying; NOT re-checked/cleaned this session. → **Parked: OPEN-U3-5.**

---

## 4. ISSUES / ERRORS ENCOUNTERED (incl. out-of-scope) — for the Brain to queue

### 4.1 `POST /auth/register` hangs on staging (ENVIRONMENTAL) — **NEW, notable**
- **Symptom:** register never returns (>90s), while `/auth/login` and `/orders/delivery-cities` respond in <0.3s.
- **Root cause (source + behavior):** `email-hygiene.service.ts` does a DNS **MX-record lookup (5s timeout)** on the email domain; the staging tasks' VPC appears to **block outbound DNS MX resolution**, so every external domain times out → `mx_timeout` → register rejects (or stalls on repeated attempts). This blocks scripted AND real-UI signup on staging with any real email domain.
- **Impact:** onboarding cannot be exercised on staging via the normal path; likely also affects any human trying to sign up on staging. Prod may differ (different egress) — **unverified on prod.**
- **Classification:** YELLOW / R2. → **OPEN-U3-6.** Investigate VPC DNS egress / whether hygiene should fail-open on `mx_timeout` in non-prod.

### 4.2 Pre-existing failing test suites (NOT caused by this session) — **flag for stabilization**
Full `yarn test` = 116 suites, 6 failed. **All 6 fail identically on clean HEAD `bceef76`** (proven by stash). This session added ZERO new failures. The failing suites:
- `src/modules/entity/entity-boot.spec.ts`
- `src/modules/searchEnrichment/services/hybrid-search.service.spec.ts`
- `src/modules/searchEnrichment/services/hybrid-search-sort.spec.ts`
- `src/modules/whatsapp/whatsapp-consent-gate.property.spec.ts`
- `src/modules/whatsapp/whatsapp-bridge.service.spec.ts` (16 tests fail on clean HEAD)
- `src/modules/whatsapp/whatsapp-dispatch-sovereignty.guard.spec.ts` (this one PASSES on clean HEAD; it only failed transiently under my reverted whatsapp change — now GREEN again)
- **Classification:** YELLOW / R2. → **OPEN-U3-7 (pre-existing test-suite stabilization).** These represent an eroding CI signal; the money-modules CI job only runs a subset, so these don't block deploys today.

### 4.3 Large pre-existing lint baseline (NOT caused by this session)
- Touched files carry pre-existing `@typescript-eslint/no-unused-vars` errors (`catch (_)` pattern, unused imports). I introduced none. Not blocking, but the `eslint` baseline is dirty. → **BLUE / R3, OPEN-U3-8 (lint hygiene).**

### 4.4 Very noisy DB query logging on staging back service
- CloudWatch `/ecs/tw-staging-back` logs echo full SQL for routine reads (e.g. ConsentRegister SELECTs on every request) — high log volume/cost, and makes log filtering hard. → **BLUE / R3, OPEN-U3-9 (disable SQL echo / lower TypeORM logging on staging).**

### 4.5 whatsapp_sent identity (deliberately NOT changed — scope discipline)
- `whatsapp_sent` still emits with the entity id in the user slot. It is a **messaging/dispatch** event, not money-spine, and has an existing sovereignty-guard test asserting the current contract. I reclassified it mid-session, it broke that test, and I **reverted** it (correct — out of scope). → **Parked OPEN-7 adjacent: OPEN-U3-10 (decide whatsapp_sent identity contract with founder).**

### 4.6 OPEN-7 (`prediction_conversion_scored` "system" uuid reject)
- Confirmed by S-CTO-1, still open, system-source path, out of money-spine scope. → **YELLOW / R2, OPEN-U3-11.**

---

## 5. NON-REGRESSION / SAFETY CONFIRMATIONS
- Classic View, order state machine (incl. privacy contract test), analytics ingestion/worker: all touched-area suites GREEN. No behavior change beyond which id is stamped.
- Delivery pipeline shape unchanged; worker 0 DLQ before, during, after.
- No prod mutation. No Dockerfile/workflow change. No `git add .`. No force-push. No secret exposed. One concern per commit.

---

## 6. OPEN ITEMS SUMMARY (for the queue)

| ID | Item | Layer | Class | Owner/next |
|---|---|---|---|---|
| OPEN-U3-1 | Browser-verify (Chromium/WebKit) the store-boost web UI flow end-to-end | Front | YELLOW/R2 | web session, once register works or via seeded+login cookie |
| OPEN-U3-2 | Historical backfill of pre-fix entity-keyed money events (lake + Amplitude) | DB/Amplitude | BLUE/R3 | separate data session (founder decision on scope) |
| OPEN-U3-3 | Delete staging test data (user 520d75eb…, entity fbfa1da7…, boost id=3) | DB | BLUE/R3 | optional cleanup task |
| OPEN-U3-4 | ECS Exec data channel down on tw-staging-svc-back (ssmmessages) | AWS | YELLOW/R2 | ops/cutover checklist |
| OPEN-U3-5 | `order_id` registered as USER-scoped property in Amplitude (should be event-scoped) | Amplitude | BLUE/R3 | verify post-fix |
| OPEN-U3-6 | `/auth/register` hangs on staging (VPC DNS MX egress / hygiene fail-open) | AWS/Back | YELLOW/R2 | investigate; check prod parity |
| OPEN-U3-7 | 6 pre-existing failing test suites (entity-boot, hybrid-search×2, whatsapp×3) | Back | YELLOW/R2 | stabilization session |
| OPEN-U3-8 | Pre-existing lint baseline (unused vars/imports) | Back | BLUE/R3 | hygiene session |
| OPEN-U3-9 | Noisy SQL query logging on staging back (cost/observability) | AWS/Back | BLUE/R3 | config change |
| OPEN-U3-10 | whatsapp_sent identity contract (messaging, not money) | Back | YELLOW | founder decision |
| OPEN-U3-11 | OPEN-7 prediction_conversion_scored "system" uuid reject | Back | YELLOW/R2 | separate session |

---

## 7. NEXT SESSION
**ID:** `S-CTO-4 — independent QA-accept CLOSE-U3 + (optional) browser E2E of store-boost UI`.
**One-line reason:** verify S-CTO-3 from source/live independently, accept the durable status in the Brain/queue, and optionally close OPEN-U3-1 with a real browser run once the register/DNS block (OPEN-U3-6) is understood.
