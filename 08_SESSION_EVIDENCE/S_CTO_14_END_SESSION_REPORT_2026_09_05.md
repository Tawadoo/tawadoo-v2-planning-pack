# S-CTO-14 — END-SESSION REPORT

**Session:** S-CTO-14 · EVENTS-vs-UI 360 (READ-ONLY MAP)
**Date:** 2026-09-05 · **Mode:** MODE A discovery, strictly read-only
**Companion evidence:** `EVENTS_VS_UI_360_EVIDENCE_2026_09_05.md` (full 492-row master table lives there)
**Parallel sessions live at the same time:** S-CTO-11 (browser/web, draft-autosave finisher), S-CTO-5 (web QA). This report explains where their work overlapped my read-only view.

---

## 0. ONE-PARAGRAPH SUMMARY FOR THE FOUNDER

I mapped all 492 declared analytics events against (a) the actual code that emits them and (b) what has really landed in the Amplitude data lake. Result: **157 are truly alive, 128 are built-in-code-but-not-yet-proven-in-the-lake, 207 are paper-only.** Along the way I found four things worth the Brain queuing as follow-ups, one of which is a **stale-documentation contradiction** and one is a **historical data-loss incident that is already fixed but should be remembered**. The map itself is complete and reconciles exactly (157+128+207=492). Nothing was built or changed. No browser was run (S-CTO-11 owns it this round).

---

## 1. STATUS: FINISHED — COMPLETE (for a read-only mapping session)

Every acceptance criterion in the S-CTO-14 brief was met:
- [x] Real allowlist count confirmed from source: **492** (not "~490").
- [x] Every event has a wired status (source emitter y/n, with representative file:line for the ambiguous families).
- [x] Every event has a landing status (Amplitude project 795817 catalog, live `lastSeen`).
- [x] Every event has a verdict: **LIVE / WIRED-NOT-LANDING / PAPER** (my three-state refinement of the brief's build/prune/ambiguous).
- [x] Grouped by domain (29 domains), per-domain counts.
- [x] User-facing BUILD shortlist (ranked, 10 groups).
- [x] PRUNE candidate list + AMBIGUOUS list for founder.
- [x] Sacred money/create/video events cross-checked.
- [x] Next-session recommendations with one-line reasons.
- [x] No code changed; all four canonical HEADs verified.

---

## 2. WHAT IS "BUILT IN THE BACK" BUT NOT BUILT / NOT VERIFIED ELSEWHERE

This is the core of what the founder asked. Breakdown by where the gap sits:

### 2A. Built in API (emitter exists) but NOT proven to land — the money funnel (highest priority)
Real server- or client-side emitters exist and route to the sovereign `/api/analytics/events` endpoint, but **zero landings** in the lake — because no real buyer completed these flows in staging (staging = Ramzi's private test env, low real traffic):

| Event | Where it's wired | Missing proof |
|---|---|---|
| `offer_submitted` (15 refs), `offer_accepted` (11), `offer_received`, `offer_responded`, `offer_viewed_by_seller` | web `makeOffer.tsx`, api `offer.service.ts` | Never landed → run a real offer in staging |
| `bid_placed` (16), `bid_outbid`, `bid_room_joined`, `auction_won`, `auction_lost`, `bid_deposit_refunded`, +8 auction | web `auction-view.tsx`/`productDetails.tsx`, api `bid-*.service.ts` | Auction domain has **0 landings** → no auction completed in staging |
| `payment_completed`, `payment_failed` | web coin-payment view, api payments | Never landed |
| `coin_purchased`, `coin_spent`, `coin_earned` | api `payments.service.ts`, `wallet.service.ts` | Never landed |
| Order state machine: `order_request_submitted`, `seller_order_confirmed`, `buyer_final_order_confirmed`, `order_shipped/in_transit/delivered/returned`, `tawssil_shipment_creation_requested`, `guest_order_placed` | api `orders.service.ts` via `trackOrderTransition()` | Never landed → run one end-to-end order |
| `rewarded_video_*` (7), `subscription_*` (5) | web `RewardedVideoPlayer.tsx`, api `syndication-subscription.service.ts` | Never landed |
| `overlay_bid/buy_now/contact/offer` | web `TransactionOverlay.tsx` | Never landed |

**Verdict: NOT a rebuild. Needs ONE real end-to-end staging transaction to confirm the pipeline (S-CTO-11's browser turn).** `order_placed` did land historically but last on **2026-07-16** (stale, pre-30-day window) — same root cause.

### 2B. Built in API (server emitters) — AI/prediction/conversation — landing partially
- **AI-pipeline (`intelligence-enrichment/services/ai-pipeline-tracking.service.ts`)**: `ai_generation_requested/failed`, `ai_suggestion_shown/accepted/edited/rejected`, `ai_moderation_rejected`, `ai_search_enriched`, `ai_category_overridden`, `behavioral_anomaly_scored` — all real `trackServerEvent`, currently WIRED-NOT-LANDING (fire on rare AI paths). Sibling events `ai_generation_completed`, `ai_moderation_passed/triggered`, `ai_embedding_generated`, `ai_classification_computed` DO land.
- **Conversation-intelligence (`conversation-intelligence.service.ts`)**: `voice_note_sent`, `purchase_blocker_identified`, `seller_response_measured`, `negotiation_started`, `photo_shared_in_chat`, `dernier_prix_asked/livraison_asked/echange_asked` — real emitters; some land (`disponible_asked`, `ghosting_detected`, `message_intent_classified`), some don't.

### 2C. Built in BO — almost NO sovereign analytics emission (finding)
The BO repo (`admin_bo_tawadoo`) has **essentially no analytics event emitters**. Grepping all 335 never-landed events across BO returned 14 hits, and 13 were **false positives** (DB column names like `coin_spent`, `purchased_at`, `coin_purchases` in `MoneyDashboard.tsx`/entities). The only real BO emitter is `ask_ramzi_interaction` (`routes/ask-ramzi.ts:1484`). **Implication:** admin actions in the BO (22 pages) are not feeding the lake. If admin behavior is meant to be training/audit data (Commandment 2), that is a gap. **Flagged for future investigation — outside my scope to decide.**

### 2D. Front-end surfaces that EXIST but are not emitting (the real BUILD shortlist)
PAPER events that map to a surface already shipped — these are the low-risk "mirror Classic + add emit" wins:
- **Listing card / detail:** `listing_card_clicked/shared/wishlisted`, `listing_phone_revealed`, `listing_seller_clicked`, `listing_reported`, `listing_gallery_opened`, `listing_video_progress/completed` (the surface + `listing_video_played` already exist).
- **Chat:** `message_received`, `message_read`, `chat_media_shared`, `chat_location_shared`, `chat_blocked`, `chat_reported` (only `message_sent` lands today — half of every conversation is dark).
- **Offer modal:** `offer_form_opened`, `offer_amount_entered`, `offer_message_added`, `offer_countered`, `offer_cancelled`.
- **Search:** `search_typed`, `search_focused`, `search_filter_removed`, `search_pagination`, `subcategory_selected`.
- **WhatsApp callbacks (API):** `whatsapp_delivered`, `whatsapp_read`, `whatsapp_link_clicked` (only `whatsapp_sent` lands).
- **Push consent:** `push_permission_requested/granted`.

### 2E. DB / AWS
- **DB:** the sovereign fact table is `ta_analytics_event` (user_id nullable UUID, source enum). I did not query the DB directly this session (read-only, no direct DB access attempted — landing truth came from Amplitude, which is downstream of the DB→delivery worker→Amplitude pipeline). **Not verified at DB level:** whether every "landed in Amplitude" event also has rows in `ta_analytics_event` and an S3 lake JSONL pair. Deferred — needs a DB read session.
- **AWS:** not touched. Amplitude is the only external system I read (read-only catalog). No S3 lake inspection, no ECS/task/log inspection.

---

## 3. ISSUES / ERRORS FOUND (including outside my scope) — FOR BRAIN TO QUEUE

### ISSUE-1 (documentation contradiction — should fix): allowlist header comment is STALE and wrong
`tawadoo_api_js/.../constants/allowed-events.ts` header says: *"Only events in this set are accepted… Unknown event types are rejected with HTTP 400."*
**The actual code does the opposite.** `analytics-ingestion.service.ts:52-67` (B13-S85) **logs unknown events and PERSISTS them with `_is_canonical=false`** — it does not reject. Confirmed by test `analytics-ingestion.service.spec.ts` F8 ("persist unknown event types with _is_canonical=false instead of rejecting"). 
**Why it matters:** any future session reading that comment will reason wrongly about the ingestion contract. It also explains ISSUE-2. **Fix: correct the comment.** (Small, but it's a source-of-truth lie.)

### ISSUE-2 (governance — founder decision): 40 event names land that are NOT in the allowlist
Because ingestion accepts unknown events (ISSUE-1), 40 non-canonical names are in the lake, flagged `_is_canonical=false`. This is naming drift + duplication:
- **17 undeclared `smart_view_*` events** landing but never added to the allowlist (allowlist declares only 7; code emits ~24). Biggest single gap.
- **Duplicate spellings both landing:** `session_start`/`session_end` vs canonical `session_started`/`session_ended`; `login` vs `login_completed`; `feed_enter`/`feed_exit` vs `feed_entered`/`feed_exited`; `feed_slide_change` vs `feed_slide_changed`; `promo_dismiss` vs `promo_banner_dismissed`; `video_play` vs `listing_video_played`; `ai_generated` vs `ai_generation_completed`; `price_suggestion_shown` vs `seller_pricing_suggestion_seen`.
- **Legacy auth-gate family:** `auth_prompt_*`, `gate_*`, `image_search_gate_*`.
- **Undeclared store events:** `store_search`, `store_categories_updated`, `store_delivery_configured`, `context_switch`.
**Founder decision needed:** for each, rename-in-code-to-canonical / add-to-allowlist / prune. Recommend this as the FIRST follow-up (S-CTO-15) because every later build inherits the ambiguity, and because you cannot safely prune a "paper" allowlist name that has a landing twin under a different spelling.

### ISSUE-3 (historical incident — ALREADY FIXED, keep in Brain memory)
A spec file (`sovereignty-system-identity.analytics-ingestion.spec.ts`) documents a real past data-loss: AI-pipeline/prediction/cron callers passed the string `'system'` into the UUID `user_id` column → Postgres `22P02` → `trackServerEvent` swallowed the error → **~1,590 events/7d silently dropped.** 
**Status: FIXED** in current Ramzi_V2 — `constants/analytics-identity.ts` now uses `user_id = null` + `source` enum for system/cron/ai events, guarded by F6 tests + a system-event Amplitude-delivery guard. Consistent with `ai_embedding_generated`/`prediction_*`/`cron_job_completed` now landing. **No action needed; recorded so the Brain remembers the incident and does not reintroduce the pattern.** It also means: some "WIRED-NOT-LANDING" server events pre-fix may have been lost; post-fix they should land once triggered.

### ISSUE-4 (second source of drift — investigate): MCP has its own separate event list
`-tawadoo-mcp-/src/client_mcp/server.py:935+` contains its own Python array of event names (e.g. `seller_contact_started`, `favorite_added`, `user_feedback_positive/negative`) that do NOT match the canonical TS allowlist. The MCP is a second, independent taxonomy. **Not reconciled anywhere.** Should be folded into the S-CTO-15 naming reconciliation. (Outside my grep scope for the 492 — surfaced incidentally.)

### ISSUE-5 (open, not verified): DB↔Amplitude reconciliation not checked
"Landed in Amplitude" was my landing proof. I did not confirm each landed event also has (a) a `ta_analytics_event` row and (b) an S3 lake JSONL training pair. The delivery worker (DB→Amplitude) could theoretically drop or transform. Low risk (Amplitude is downstream), but unverified. Deferred to a DB/lake read session.

### ISSUE-6 (method limitation, disclosed): landing is binary here, not volume
The Amplitude catalog gives `firstSeen`/`lastSeen`, not per-event volume. So I can say "landed / never landed" and "within 30d / stale," but not "landed once vs 10,000 times." A low-volume LIVE event and a high-volume one look identical in my map. Not needed for build/prune decisions, but noted.

---

## 4. THINGS LEFT OPEN / SMALL LOOSE ENDS

1. **`purchase` (72 source refs)** — almost all are the English word in UI copy, not an emitter; likely superseded by `order_placed`/`payment_completed`. I classified it WIRED-NOT-LANDING conservatively, but it is probably PRUNE or a GA4-compat alias. Founder call.
2. **`ask_ramzi_tool_call`** is PAPER while `ask_ramzi_interaction` is wired — is the sub-event intended? BO Ask-Ramzi surface.
3. **`dispute_opened/resolved`, blog + `programmatic_page_*`** — PAPER with no emitter; unknown whether the user/BO surface exists un-instrumented (→ build) or does not exist (→ defer). Needs a quick surface-existence check.
4. **`agents` domain (23 events, all paper)** and **advertiser self-serve (17 paper)** and **predictions (5 paper)** — future roadmap; should be marked "roadmap, not platform" so they stop inflating "declared coverage" numbers. Anti-over-architecture (§ steering): no premature ML/agent infra.
5. **`test_event`** — pure dev artifact in the allowlist; safe prune.

---

## 5. PARALLEL-SESSION OVERLAP (honest note per §11 repo-isolation)

- At session start, `tawadoo_web_js` HEAD was `36b7183`; at session end it was `780bdb58` with 18 dirty tracked files. **None are mine.** The dirty files are S-CTO-11 / S-CTO-5 artifacts: `S_CTO_11_DRAFT_AUTOSAVE_FINISHER_EVIDENCE_*.md`, `S_CTO_5_END_SESSION_REPORT_*.md`, `AI_LISTING_JOURNEY_INTEGRITY_EVIDENCE_*.md`, `CREATE_FORM_CONSOLIDATION_EVIDENCE_*.md`, multiple `playwright-report-scto*/` dirs and `playwright.scto*.config.ts`. The web HEAD advanced because S-CTO-11 committed on its turn.
- `-tawadoo-mcp-` has 31 dirty files — pre-existing spec/audit docs, not mine.
- **I performed zero repo writes/edits/commits.** My only file output is at the workspace root (`/Users/ramzihannachi/Code`), which is not a git repo, so it touches no branch.
- **Read-only guarantee holds:** api HEAD `673eac0` clean, bo HEAD `07b3a84` clean; the file I read (`allowed-events.ts`) is unchanged.

---

## 6. RECOMMENDED NEXT SESSIONS (ranked)

1. **S-CTO-15 · Event naming-drift reconciliation (MODE A → founder decision).** Resolve the 40 landing-but-undeclared names (ISSUE-2) + the MCP separate list (ISSUE-4) + fix the stale allowlist comment (ISSUE-1). Founder decides rename/add/prune per name. **Do before any wiring** — every later build inherits this.
2. **S-CTO-16 · Money-funnel landing proof (needs S-CTO-11 browser).** One real end-to-end staging transaction (offer → accept → order → pay → deliver) to confirm the §2A WIRED-NOT-LANDING money events actually land. Proves the revenue pipeline without rebuilding.
3. **S-CTO-17 · Listing-view + messaging + offer-funnel emitter wiring (§2D shortlist #1–#4).** Highest user-facing value, lowest risk. Mirror Classic action, add sovereign emit.
4. **S-CTO-18 (smaller) · BO analytics-emission gap (§2C) + DB/lake reconciliation (ISSUE-5).** Decide whether admin actions should feed the lake; confirm DB↔Amplitude↔S3 parity.

---

## 7. TOOLS / PERMISSIONS / COST

- Tools used: source grep (4 Ramzi_V2 repos), grep_search, Amplitude read-only MCP (project 795817 catalog). 
- No novel paid tool activated. No IAM change. No AWS mutation. No browser. No commit/deploy. No prod. **COST: none incurred.**

---

_S-CTO-14 end-session report. Read-only. 492 events mapped and reconciled (157 LIVE + 128 WIRED-NOT-LANDING + 207 PAPER). 6 issues flagged (1 stale-doc, 1 governance/drift, 1 fixed-historical-incident, 1 second-taxonomy, 2 unverified/limitations). All four canonical HEADs unchanged._
