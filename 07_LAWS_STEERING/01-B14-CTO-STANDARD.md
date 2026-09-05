---
inclusion: always
---

# B14 CTO STANDARD — MANDATORY FOR EVERY TAWADOO SESSION

**Authority:** Founder-approved. Brain B14. 2026-08-30.
**Scope:** Every execution session, QA session, Brain session, and prompt in every Tawadoo repository.
**Purpose:** Ensure every session operates at CTO level: best-in-class architecture, data sovereignty, training pipeline, cost control, and zero regression on the Classic View.

## 0. THE THREE COMMANDMENTS

1. **CLASSIC VIEW IS SACRED.** The existing marketplace (search, listings, dashboard, orders, auth, payments, delivery) must NEVER be broken by Smart View work. Mirror it, don't touch it. Every search query that works in Classic must work in Smart. Every listing that renders in Classic must render in Smart. Every event that fires in Classic must fire in Smart.

2. **EVERY INTERACTION FEEDS THE LAKE.** Every AI call, every user query, every search, every edit, every conversion — logged to the sovereign DB FIRST, then mirrored to providers. This is the training data pipeline for our proprietary model. No interaction may bypass sovereignty.

3. **COST-FIRST, ALWAYS.** Use the cheapest model that works. Nova Micro ($0.035/1M) before GPT-4o-mini ($0.15/1M). Vector search before LLM. Client-side stop-words before server-side NLP. Cache everything. Guest quotas. Rate limits. Every prompt must specify cost implications.

## 1. PROTOCOL AWARENESS (every session must know this)

Tawadoo operates in the agentic commerce ecosystem:

| Protocol | What | Our status |
|----------|------|-----------|
| **MCP** (Anthropic/AAIF) | Tool calls from AI agents | LIVE at mcp.tawadoo.ma |
| **ACP** (OpenAI+Stripe) | Checkout by AI agents, ~7.2% fees | Via MCP/ChatGPT |
| **UCP** (Google+Shopify) | Full lifecycle commerce for AI Mode, ~3.2% fees | .well-known/mcp.json done |
| **A2UI** (Google) | Declarative JSON for agent-generated UIs | NOT YET — adopt for Smart View cards |
| **IndexNow** | Instant Bing/DuckDuckGo indexing | DONE |
| **llms.txt / agents.md** | AI crawler documentation | DONE |

**Every Smart View component that renders a card (listing, bid, draft, comparison) should output A2UI-compatible declarative JSON** so external agents (Gemini, ChatGPT) can render them through our MCP. This is the UCP/ACP bridge.

## 2. DATA SOVEREIGNTY PATTERN (MANDATORY for every AI route)

```
User action → Sovereign DB write (ta_analytics_event)
            → LLM/AI call (Bedrock/OpenAI)
            → Sovereign DB write (response + model + latency + tokens)
            → TrainingShadow to S3 (prompt + response + user_edit = JSONL training pair)
            → Return to user
```

What goes to the lake for EVERY AI interaction:
- `session_id`, `user_id` or `anonymous_id`
- `raw_prompt` (user's exact input)
- `detected_intent` (from classifier)
- `extracted_search_terms` (what the LLM/classifier pulled out)
- `model_used` (nova-micro / gpt-4o-mini / static-fallback)
- `model_response` (full structured JSON)
- `latency_ms`, `input_tokens`, `output_tokens`
- `result_count` (how many listings returned)
- `user_action` (clicked result? refined? abandoned? converted?)
- `user_edit` (if user modified AI suggestion — this is GOLD for training)

This data is what makes our proprietary model possible. Missing it is unrecoverable.

## 3. MODEL ROUTING (cost optimization)

| Tier | When | Model | Cost |
|------|------|-------|------|
| 0 | Keyword/category search, filter changes | Direct OpenSearch | $0 |
| 1 | Intent classification, metadata extraction, stop-word removal | Nova Micro via Bedrock | $0.035/1M tokens |
| 2 | Darija comprehension, complex reasoning, multi-turn negotiation | GPT-4o-mini via OpenAI | $0.15/1M tokens |
| 3 | BO Ask Ramzi (admin only) | Claude Haiku/Sonnet/Opus via Bedrock | $0.80-$75/1M tokens |

**Rule:** Never use Tier 2 when Tier 1 suffices. Never use Tier 1 when Tier 0 suffices.

## 4. SMART VIEW ARCHITECTURE (every Smart View session must follow)

### The Split-Pane Layout
- **Left: Chat Pane** — conversational input, AI reasoning, refinement chips
- **Right: Canvas Pane** — results grid/feed/video, action cards, sponsored slots
- **Mobile:** stacked (chat on top, results below), swipeable

### Card Types (A2UI-ready declarative components)
- `ListingCard` — mirrors Classic listing card exactly
- `ComparisonCard` — side-by-side specs with AI recommendation badge
- `ConfirmationCard` — 2PC human-in-the-loop for write actions (bid, buy, publish)
- `DraftListingCard` — seller auto-draft with editable fields
- `RefinementChips` — conversational filter refinements
- `SponsoredCard` — display ad slot (position 1 in results)

### Sacred Systems (NEVER break these)
- Auth (JWT, OTP, session, guards)
- Analytics sovereignty pipeline (ta_analytics_event → delivery worker → Amplitude)
- TrainingShadow (ai_outputs → S3 lake)
- CSRF on BO routes, RBAC on 22 BO pages
- Payment flow (Payzone callbacks)
- Order state machine (3-confirmation law)
- Classic View search, filters, sort, category browsing
- WhatsApp notifications (19+ templates)
- Distribution feed generation (DISABLED but intact)

## 4B. SACRED FILES — NEVER MODIFY WITHOUT DEDICATED SESSION + RAMZI APPROVAL

These files affect EVERY page. Modifying them in any session that isn't exclusively dedicated to them is FORBIDDEN:

```
src/app/[locale]/layout.tsx          — ROOT LAYOUT. Breaks everything if touched wrong.
src/app/[locale]/AnalyticsTracker.tsx — SPA page view tracking
src/app/[locale]/AmplitudeInit.tsx    — Amplitude SDK initialization
src/app/[locale]/consent/ThirdPartyScripts.tsx — GTM, Meta, TikTok consent
next.config.js                        — Build configuration
Dockerfile                            — Container build (§10)
.github/workflows/*                   — CI pipeline (§10)
```

**If your Smart View work needs to add something to layout.tsx:**
1. Create a NEW component (e.g., `SmartViewProvider.tsx`)
2. Import it in layout.tsx in a SEPARATE, SINGLE-PURPOSE commit
3. That commit touches ONLY layout.tsx and ONLY adds one import + one JSX element
4. Verify before AND after: `fbq`, `AmplitudeInit`, `AnalyticsTracker`, `ResumeActionOnAuth`, `UtmCapture`, OG tags, Schema.org — ALL must be identical

This law exists because B14-S130 destroyed the entire Classic View by rewriting layout.tsx while trying to improve the Smart View. Every user on every page was affected. Lesson #41.

## 5. REGRESSION ZERO-TOLERANCE

### Before ANY code change:
1. Read the file you're changing AND all its callers/consumers
2. Run existing Playwright regression tests
3. Check `tawadoo_web_js/.kiro/steering/known-regressions.md`

### After EVERY commit:
1. `yarn build` — must pass
2. `yarn test` — must pass
3. Run ALL Playwright regression tests for affected pages
4. If CSS/layout changed: z-index sweep test (§33.6)

### The Classic Mirror Rule:
- Every API endpoint the Smart View calls MUST be the same endpoint Classic calls
- Every search parameter Smart View sends MUST match Classic's format
- Smart View is an ADDITIVE layer — it reads the same data, calls the same APIs, fires the same events
- The ONLY new code is: intent classification, LLM guidance, action cards, voice input
- If you find yourself modifying a shared component to make Smart View work, STOP — create a Smart View-specific wrapper instead

## 6. WEB SEARCH MANDATE

Every execution session MUST perform web searches BEFORE implementation. Not generic "best practices" — specific queries:
- "NestJS {exact pattern} production 2026"
- "Next.js {exact feature} React Server Components 2026"
- "marketplace {exact UX pattern} conversational commerce 2026"
- "{exact library} {exact version} integration guide 2026"

This is how B13 achieved zero regressions. Current industry knowledge prevents reinventing broken patterns.

## 7. TRAINING DATA QUALITY RULES

Every piece of data going to the lake must be:
1. **Timestamped** — UTC ISO 8601
2. **Identified** — session_id + user_id/anonymous_id
3. **Typed** — event_type from the 478-event allowlist
4. **Contextualized** — locale, device_type, platform, page_url
5. **Paired** — for AI interactions: prompt + response + user_action = one training tuple
6. **Anonymized** — no raw PII (email, phone) in properties. Use hashed IDs.
7. **Deterministic** — event_id derived from content hash (idempotent retries)

The reward signal for fine-tuning is: **did the user convert after the AI suggestion?** Track:
- `smart_view_ai_suggestion_accepted` (user clicked the suggested listing)
- `smart_view_ai_suggestion_modified` (user changed the AI's search terms)
- `smart_view_ai_suggestion_rejected` (user ignored and typed something else)
- `smart_view_draft_field_edited` (user changed AI-generated listing field)

## 8. COST GUARDRAILS

- Guest users: max 20 AI guidance calls per session (rate limit in guidance route)
- Authenticated users: max 100 AI guidance calls per day
- Image analysis: max 5MB, validated MIME types, one analysis per upload
- Voice transcription: use browser Web Speech API ($0) before cloud STT
- Prompt caching: structure system prompts as static prefix (50% token discount on Bedrock/OpenAI)
- Response caching: 15-minute Redis TTL on identical search queries

## 9. REFERENCE FILES (read when needed, not every session)

- `/Users/ramzihannachi/Code/B14_KNOWLEDGE_ANCHOR_2026_08_30.md` — full research findings
- `/Users/ramzihannachi/Code/B14_CHECKPOINT_2026_08_30.md` — queue + state
- `/Users/ramzihannachi/Code/BRAIN_B14_HANDOFF_2026_08_30.md` — handoff from B13
- `/Users/ramzihannachi/Code/KIRO_PROMPT_NEXT.md` — 13-point prompt standard
- `/Users/ramzihannachi/Code/.kiro/steering/00-EXECUTION-PROMPT-NON-REGRESSION-LAW.md` — all §0-§33 laws
