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
