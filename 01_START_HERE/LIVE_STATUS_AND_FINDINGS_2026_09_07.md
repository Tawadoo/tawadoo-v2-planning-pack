# TAWADOO V2 — LIVE STATUS, FINDINGS & PLAN (up to the second) — 2026-09-07
**For ChatGPT: read this FIRST (with the README). This is the current truth as of now. Advise on approach + progress.**
**Author:** Kiro Brain B23 (CTO). Everything below is source/live-verified or tagged UNVERIFIED/UNKNOWN. Nothing buried.

---
## 0. WHERE WE ARE IN ONE PARAGRAPH
We stopped patching the "item unavailable / broken links / findability" symptoms and reframed them as ONE systemic architecture problem: **entity addressing, visibility, and findability are not coherent across surfaces.** We are now in a disciplined **UNDERSTAND-BEFORE-FIX** loop: diagnose from source AND live runtime → prove the truth → only then fix the smallest correct thing → ship a regression guard. A major realization drove this: **a lot of the historical mess/duplication came from earlier Kiro sessions working blind (no DB access, guessing names).** We just fixed that root cause — sessions now have read-only DB access — and we're running a full-platform truth 360 before building further.

---
## 1. LIVE GIT (verified) + IN-FLIGHT
- HEADs, all `Ramzi_V2`, synced: web `8b0a815f` · api `8d55a20` · bo `07b3a84` · mcp `d8efb4a`. Nothing committed in the recent NAV work (SSR fix is parked uncommitted).
- **IN FLIGHT (1):** `S-CTO-29-PLATFORM-TRUTH-360` — read-only full-platform truth map (runs alone).
- Infra: eu-west-1, ECS `tw-staging-cluster`, Aurora `tw-staging-cluster` (in-VPC). Prod = separate cluster, protected, untouched.

---
## 2. THE NAV / VISIBILITY / FINDABILITY UNIT — arc + progress (~55-60%)
This is the current priority (classification A — refactor/architecture). It absorbs the FR "item unavailable"/notif-deeplink bugs (they're symptoms, not separate fixes).

**Stage 0 — DIAGNOSE (S-CTO-25-NAV-MAP): ✅ DONE.** Confirmed ONE systemic defect = 3 root patterns:
- P1 divergent visibility predicate across layers.
- P2 no canonical identity→URL contract (e.g. `/p/null` fallbacks, 3 base-URL sources).
- P3 overloaded/incomplete notification target (objectId is often a literal string; lead notif has no target).
- Proven live: SAME published+verified listing returns 200 from `publications/slug/:slug` but 404 from `publications/lightssr/:slug`.

**Stage 1 — GROUND THE TRUTH: ✅ DONE + QA'd.**
- `S-CTO-27` proved the NAV-MAP's `isModerated=null` premise was WRONG (0 rows — column is NOT NULL). Saved us from building a phantom fix.
- `S-CTO-28` proved the real mechanism: `is_moderated` is set by BOTH the human BO-Verify AND an automated v2-publish path (`publication.controller.ts:626/:816`) — the two flags are DECOUPLED (161 all-time rows moderated-but-not-verified). This is exactly why the SSR gate keying on `is_moderated` is inconsistent.
- Also proved: text moderation runs but is ADVISORY-only (never blocks); image moderation blocks but ONLY on web-v2 → mobile/legacy BYPASS it (~3.5% of images ever moderation-stamped over 90d).
- Founder ruling captured as LAW: **verified=true ⇒ listing MUST be live; moderation does NOT gate public visibility; both flags kept; moderation subsystem stays intact.**

**Stage 2 — BUILD + PROVE (S-CTO-26): PARTIAL — the safe fix is CODED but PARKED (not deployed).**
- Wrote a clean, minimal SSR fix: new `isPubliclyVisibleForSSR` predicate keyed on `isVerified` (matches the ruling), rewires `findBySlugForSSR` off `isIndexEligible` (moderated), leaves index/feed untouched (respects public-visibility ≠ syndication-eligibility). 68 tests green, fail-first proven, + a regression guard spec.
- **DECISION: HOLD (not deployed).** Reasons: (a) it's one wave of a multi-wave unit that must land complete; (b) deploying flips SSR visibility for 158 rows we don't yet fully understand; (c) not browser/feed-parity verified yet.
- Remaining NAV build waves (specced, not built): canonical entity-URL builder (P2), image↔unified search convergence, structured notification targets (P3, absorbs FR-9), image-index re-key (guarded, feed-safe).

**Founder gate:** the ENTIRE NAV unit must be complete + tested end-to-end + clean before we return to the FR wave or anything else.

---
## 3. THE BIG UNBLOCK — DB is now readable to sessions (root-cause fix)
- The recurring "DB BLOCKED" was real: ECS-Exec fails `TargetNotConnected` because the VPC is MISSING the `ssmmessages`/`ec2messages` endpoints (confirmed live, not a permissions issue). A staging force-redeploy did NOT fix it (ruled out stale channel).
- **Workaround used (A2, safe, zero shared-infra change):** a read-only one-off ECS task (`RunTask` on `tw-staging-task-back:44`) runs a Node+pg script under `SET TRANSACTION READ ONLY`, sanitized JSON → CloudWatch. Confirmed: **134 tables readable.** DB user `tw_runtime_b11` (read-only).
- **Why this matters:** sessions can now audit the real schema (tables, columns, IDs, constraints) instead of guessing → kills the duplication/mess-by-guessing root cause the founder identified.
- A1 (add scoped `ssmmessages`+`ec2messages` VPC endpoints for real exec) = deferred, founder-gated (touches shared VPC).
- ⚠️ Flagged: the staging back task uses a PROD-NAMED role (`tw-prod-ecs-task-execution-role`) and the staging DB is named `twdbprod` — a staging/prod boundary smell to clean before cutover.

---
## 4. THE FOUNDER'S OPERATING LAWS (locked this week — govern every session)
- **UNDERSTAND-BEFORE-FIX:** nothing is fixed without full understanding of the truth AND the alternative. Names are hypotheses until proven from source + runtime rows. Example the founder gave: he assumed `active` = a feed/boost SLOT — it is NOT. So before touching `active` we must know what slots really are, who captures them, when a listing is swapped/rotated, what it's called today (catalogue management?), and if it's even live.
- **UNIFIED-MULTIMODAL-SEARCH (web-verified vs industry SOTA):** image is a first-class search input that CONVERGES into the same unified result surface as keyword/category/filters — not a separate silo. Confirmed pattern (eBay, Shopee, Amazon Nova, Gemini, UniECS). Current state is the anti-pattern (separate `search-by-image` endpoint). AUTH BOUNDARY: image search stays login-gated + rate-limited (it runs AI/embeddings = costs money = bot/abuse target); cheap paths (keyword/category/filters) stay open to logged-out users.
- **FINDABILITY-END-TO-END:** a listing must be findable by ALL means (keyword, image, category, subcategory, every filter, location, price, condition, individual/pro, type) — no matter what it went through. "Visible but not findable" = a regression. Visibility + findability = ONE acceptance surface.
- **PUBLIC VISIBILITY ≠ SYNDICATION ELIGIBILITY** (do not collapse; feed guards sacred — GMC ban history).
- **CONTINUOUS-UPDATE + REGRESSION-GUARD:** the Brain updates the control docs thoroughly after every session (no lost context → no rework/regression), and every fix that lands+tests ships a durable regression guard registered in `known-regressions.md`.
- Standing: Classic sacred / Smart separate; human moderation gate real; prod protected; staging-first; max 2 concurrent sessions; verify from source + live, never guess; no user-facing copy without founder-approved FR/AR/EN.

---
## 5. THE CURATED → MODERATED → VERIFIED TRIO (founder frame)
- MODERATED (S-CTO-28) ✅ understood · VERIFIED (S-CTO-26 ruling) ✅ understood · **CURATED = owed** (queued): do we crop/straighten/enhance/best-photo-first for quality + conversion + feed acceptance, and to what extent (without misrepresenting the item)? Source already has `sharp` processing, thumbnails, `listing_quality_score` — but whether we curate listing PHOTOS is unclear. Future 360, then build if warranted, GMC-safe.

---
## 6. THE ACTIVE INVESTIGATION — S-CTO-29-PLATFORM-TRUTH-360 (in flight, read-only)
Full-platform truth map. Fixes nothing. Sections:
- A. Entity & listing core (seller types individual/store/premium; every lifecycle flag — status/isVerified/isModerated/is_live/active — source+rows; the real state machine).
- B. Moderation→verification→visibility (reconcile S-CTO-28; the 158 rows origin; owner-preview truth).
- C. Findability/search/indexing (keyword index vs image/vector index agreement; the convergence gap; image-index health; findability gaps).
- D. SLOTS / boost / distribution / feeds / catalogue (the founder's core Q: what is a slot, who captures it, when swapped/rotated, called what today, LIVE or dormant, per-channel real counts; is "silently distribute every eligible listing" actually happening).
- E. Three storefronts (Classic marketplace / Smart View / MCP-ChatGPT) — same listing consistent across all? Smart separate? MCP contracts? divergences.
- F. Seller types & monetization (individual/store/premium capabilities, coins/packages/subscription gates, store-video + for-you, live vs dormant).
- G. NAMING RECONCILIATION table (concept → what the name suggests → what it TRULY is → live/dormant/dead → founder-assumption-vs-reality).
- H. LIVE-vs-DORMANT ledger (whole platform).
- Fallback: if too large, complete A–D fully (supply core + slots) and name E–H for a follow-up.

---
## 7. OPEN FINDINGS CARRIED (from S-CTO-26/27/28 — feed the truth 360)
- I1 `is_live` DORMANT (0 rows true; intended lifecycle not wired) · I2 `Publication.active` redundant/dead · I3 staging DB named `twdbprod` (confusion trap) · I4 THREE meanings of "active" across entities · I5 the 158 moderated-not-verified rows (legit-live or stale?) · I6 boosted write-path not fully traced · I7 active/status lockstep risk.
- DEV1 owner-preview assumption may be wrong (SSR has no owner exemption) · DEV2 fail-first direction inverted vs prompt.
- C3 `ta_publication_image` subquery timeouts (possible index-health issue) · C2 prod DB not readable · C1 GHCR read denied.
- From S-CTO-28: image moderation v2-only (mobile bypass); text moderation advisory-only; `ai_moderation_rejected` never fires; image moderation invisible in Amplitude (S3 only); two `isModerated` flags (Publication vs PublicationImage) easily conflated.

---
## 8. THE ROADMAP (unchanged North Star)
all-functional (staging) → security gate → **structural refactor (5-layer modular monolith)** → sync app + MCP → prod cutover. Prod protected until founder opens go-live. The Bayesian/hidden-state moat is post-refactor, its data foundation being built now by the events/identity work.
Rough progress: functional ~50-55% · structural refactor ~5-10% (only S141 schema foundation built) · whole-program-to-prod ~45-50%. Commerce spine solid; remaining distance = the NAV/findability coherence, search-intelligence, the structural refactor, app/MCP sync, and the moat.

---
## 9. WHAT WE WANT FROM CHATGPT (advise on approach + progress)
1. Is the UNDERSTAND-BEFORE-FIX + truth-360-first approach right, or are we over-investigating vs shipping? Where's the line?
2. The NAV unit sequencing — is HOLDING the SSR fix until the whole unit completes correct, or should the safe fix ship incrementally (with its guard) to reduce risk of a big-bang landing?
3. The image↔search convergence — is routing image into the existing hybrid engine (reusing Titan embeddings) the right minimal path, or is there a simpler/safer approach?
4. The DB-access workaround (read-only one-off task) vs fixing ECS-Exec properly (VPC endpoints) — acceptable long-term, or fix the endpoints now?
5. Anything in the findings (I1–I7, the 158 rows, the prod-named role/DB smell, mobile moderation bypass) that you'd escalate as higher-risk than we've rated it.
6. Any drift you see between the stated North Star (refactor-first) and what we're actually doing (a lot of investigation + stabilization).

Challenge us where the evidence warrants. The point of this repo is an independent second opinion, because Kiro created some of the original mess.


---
---
# ADDENDUM — UPDATE @ 2026-09-07 (later same day): S-CTO-29 LANDED + LIVE FEED INCIDENT FULLY RECALLED
**This addendum supersedes §6 "in flight" above. Read it — it changes the picture materially.**

## A. S-CTO-29-PLATFORM-TRUTH-360 — LANDED, QA-ACCEPTED (status: FINISHED — INCOMPLETE)
The full read-only platform truth map is done (evidence indexed at `08_SESSION_EVIDENCE/S_CTO_29_PLATFORM_TRUTH_360_EVIDENCE_2026_09_07.md`). It did the truth map AND caught a live provider-ban risk mid-session, then took founder-authorized emergency action. Brain (B23) verified every action against live AWS.

## B. 🔴 THE FEED INCIDENT (the important one)
- **What was wrong:** the STAGING feed cron was writing the live Meta product feed INTO THE PRODUCTION media bucket `tw-prod-media-storage-prod` (tagged env=prod, publicly readable by Meta). This is the exact path that caused the earlier Google Merchant (GMC) suspension — staging TEST products reaching a real provider.
- **How it got there (CloudTrail):** a `kiro-ai` session on 2026-07-31 created the EventBridge feed crons ALREADY ENABLED — "born live," never a deliberate switch. Classed as a black-area violation.
- **Founder order:** empty ALL staging feeds entirely — nothing from staging should be on Meta/TikTok/Google (all bullshit test products).
- **DONE + VERIFIED (2026-09-07):** ALL 40 staging feed files emptied to `[]` (verified: 0 non-empty remain). Only Meta ever had files (google/tiktok/chatgpt = 0 objects). Both feed crons (`tw-cron-feed-generation`, `tw-cron-store-video`) DISABLED. Backup saved at `feeds/_recall_backup_S29/`. Prod MCP/ChatGPT untouched.
- **FOUNDER DECISION:** LEAVE IT — do NOT add a `FEEDS_ENABLED` code gate. Founder controls the risk by controlling what listings are created on staging. So no code change; the durable-gate task is closed per founder.
- **Known residual (founder-accepted):** an in-app Bull cron `generate-all-feeds` re-registers on container boot (`feed-generator.service.ts:156`) → a staging back-container restart COULD regenerate feeds into the prod bucket again. Founder accepts + mitigates by controlling staging listing creation. The prod-media-bucket env-boundary defect stays a cutover-checklist item only.

## C. PLATFORM TRUTH — the naming/dormant reality (source + live DB verified)
- **Scale:** 60,001 entities (57,356 individual + 2,645 store; NO `isPremium` column — premium is DERIVED from an active syndication subscription, only 3 active). 24,975 publications (21,451 published). Public-visible = 19,875. Index/feed-eligible = 20,033.
- **FLAG TRUTH (this kills the guessing the founder warned about):**
  - `isVerified` = the PUBLIC-VISIBILITY gate (page + search).
  - `isModerated` = the INDEX/FEED-eligibility gate (population), NOT the same as verified.
  - `is_live` = **DEAD** (0 rows, no reader; auctions use a separate `ta_publication_live` table).
  - `active` = **near-vestigial delete-tombstone** (false only on hard-delete). It is NOT a feed/boost slot — this was the founder's exact assumption, now DISPROVEN. "active" has ≥9 different meanings across entities.
- **The 158 mystery rows:** moderated-but-not-verified published rows — origin = a legacy migration (`BackfillIsModerated`), NOT any runtime approve/publish. They're in the index + feed-eligible but 404 on the page and excluded from search (not verified). Decision-ready: verify them or reset is_moderated. No fix taken.
- **SLOT truth (founder's core question):** a "slot" = an active `ta_syndication_campaign_product` row under a subscription's `max_live_items`, backed by a live subscription OR a live boost. The whole system is called **"syndication"** in code (no "distribution/catalogue/feed" module). **Only 1 live slot platform-wide. No auto rotation/swap.** CRITICAL: distribution is GATED behind a purchased package/boost — NOT automatic for every eligible listing. **This CONTRADICTS the founder's stated policy ("silently distribute every eligible listing").** → founder to reconcile intent vs current code.
- **Storefronts:** Classic + Smart both call the same `publications/search` API; Smart is correctly a SEPARATE additive layer (no Classic import); MCP is live at prod and resolves to the same `/{locale}/p/{slug}`.
- **Monetization:** `ta_monetization_config` table DOES NOT EXIST (constants hardcoded; free tier = 5 items). Trial service LIVE but 0 claimed. Store-video built + cron but social autopost is DRY-RUN gated. Coin orders: 129 PENDING, **0 PAID** on staging.
- **🟠 Sovereignty gaps (flagged, not confirmed):** (a) MCP logs interactions to a LOCAL sqlite/JSONL sink — no `ta_analytics_event` write found (possible Commandment-2 violation); (b) the `for-you` API has no server-side lake write (client impression only).

## D. QUEUED FROM S-CTO-29 (Brain owns; nothing buried)
- P0: (feed durable-gate = CLOSED per founder). Env-boundary (staging→prod bucket) = cutover-checklist item.
- P1: MCP + for-you sovereignty investigation.
- P2: the 158-row decision; document `is_live` DEAD + `active` tombstone.
- P3: image-index health, index-alias target (publications_2 vs -v2), autopost env value, prod-parity of findings.

## E. WHERE THE NAV UNIT STANDS NOW
Unchanged from §2 above: the SSR verified-visibility fix is CODED + PARKED (not deployed, HOLD). The NAV unit is gated to complete end-to-end before the FR wave. S-CTO-29 gives the truth foundation (flag meanings, slot reality, the 158 rows) needed to build the remaining NAV waves on understanding, not assumption.

## F. SPECIFIC QUESTIONS FOR CHATGPT (in addition to §9 above)
7. The "silently distribute every eligible listing" policy vs the code reality (distribution gated behind paid package/boost, only 1 live slot) — which should win, and what's the right target model?
8. The feed incident root cause (staging writing the prod bucket, crons born-enabled by an agent) — beyond the recall, what's the minimal durable guard you'd insist on before prod cutover, given the founder declined a code gate for now?
9. Given ~40% of the "platform" is dormant/dead (feeds except Meta, trials, social autopost, paid coins, monetization-config, 8 of 9 slot channels) — is the right move to WIRE these before refactor, DEFER them, or DELETE the dead ones? How would you sequence it against the refactor-first North Star?
