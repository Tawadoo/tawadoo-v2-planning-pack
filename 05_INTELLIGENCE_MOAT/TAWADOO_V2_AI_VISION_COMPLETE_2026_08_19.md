# TAWADOO V2 — THE COMPLETE AI VISION

> **"Tawadoo, the marketplace that gets you."**
> 
> This is not a feature list. This is the blueprint for Africa's first fully autonomous AI commerce platform.
> Every user has an agent. Every seller has an agent. Every advertiser has an agent.
> The platform itself IS an agent — learning, predicting, acting, optimizing 24/7.
> Humans steer. AI executes.

---

## THE THREE AGENTS

### 🛒 AGENT 1: THE BUYER AGENT (UCP — Universal Commerce Protocol)

**What it does:** Finds, negotiates, buys, and protects — on behalf of the buyer. Across every channel simultaneously.

**WHERE IT LIVES:**
- In-app widget (chat bubble on every page)
- ChatGPT (via MCP + ACP — "Buy in ChatGPT")
- WhatsApp (buyer sends photo or text, agent responds)
- Voice (Web Speech API → Darija/FR/AR transcription → agent acts)

**WHAT IT DOES:**

| Capability | How | Data it learns from |
|---|---|---|
| **Conversational search** | "Bghit chi telfon b moins de 3000 f Casa" → instant results | 1.5M search queries + Darija dict + click signals |
| **Image search** | Upload photo → "I found 12 similar items near you" | Titan embeddings (1024-dim) on 210K images |
| **Voice search** | Speak in Darija/FR/AR → transcription → search → results read aloud | Web Speech API + Bedrock transcription |
| **Price intelligence** | "Is this a good price?" → compares against 891K historical views + sold prices | ta_publication_view + ta_offer (accepted prices) |
| **Negotiation assistant** | "This seller typically accepts 8% below asking. Suggest 4200 MAD?" | ta_offer history (131 accepted, avg discount) |
| **Trust advisor** | "This seller responds in 15min avg, has 4.8★, verified ID" | ta_review + ta_activity + verification tables |
| **Auction strategist** | "You have a 67% chance of winning at 3500. Snipe at 10 seconds?" | ta_bid_transaction (488 bids, winning patterns) |
| **Price alerts** | "iPhone 13 under 3500 in Casa" → notifies via WhatsApp when found | ta_save_search + real-time indexing |
| **Delivery tracking** | "Where's my order?" → Tawssil status in natural language | Tawssil webhooks + ta_order lifecycle |
| **Purchase protection** | AI flags suspicious listings/sellers before buyer commits | Fraud model + report history |
| **Comparison shopping** | Shows side-by-side: price, condition, seller rating, delivery time | OpenSearch + computed features |
| **Personal shopper memory** | Remembers preferences across sessions (budget, brands, categories) | Amplitude user properties + session history |

**THE KILLER FEATURE: BUY VIA ANY CHANNEL**
- User in ChatGPT: "Find me a used iPhone in Casablanca under 3000 MAD" → MCP finds → ACP checkout → purchased without leaving ChatGPT
- User in WhatsApp: sends photo of something they want → image search → results with prices → replies "2" → offer placed → seller notified
- User on web: widget conversation → "This one, negotiate to 2500" → AI makes offer on their behalf → seller accepts → order confirmed
- User by voice: "Tawadoo, cherche-moi un vélo pour enfant à Rabat" → voice search → results read → "Le deuxième" → listing opened

---

### 🏪 AGENT 2: THE SELLER AGENT (Premium tier — « Rayonnement »)

**What it does:** Sells FOR the seller. 24/7. Across all channels. The seller only confirms orders and ships.

**WHERE IT LIVES:**
- Dashboard (seller sees AI actions, overrides when needed)
- WhatsApp (seller gets notifications of AI actions, approves with one tap)
- ChatGPT (buyers interact with the seller's AI agent directly)
- All distribution channels (Google Shopping, Meta, TikTok — AI optimizes placement)

**WHAT IT DOES:**

| Capability | How | Data it learns from |
|---|---|---|
| **Auto-respond to buyers** | AI answers questions about condition, availability, delivery in seller's voice | ta_message history (4,436 messages) → learn seller's tone |
| **Auto-negotiate** | AI accepts/counters offers within seller's bounds (min price, max discount %) | ta_offer (131 accepted patterns, avg negotiation rounds) |
| **Auto-promote** | AI decides WHEN to boost (peak demand hours), WHAT to boost (highest-value items) | ta_publication_boost (3,772 boosts) + activity patterns |
| **Auto-price** | AI suggests price adjustments based on demand, competition, views vs no-contact | 891K views + 595 offers + market data from lake |
| **Auto-relist** | Expired/unsold listing → AI re-titles, adjusts price, re-publishes | Listing quality model + search demand signals |
| **Listing creation** | Seller uploads photos → AI does EVERYTHING (title, desc, category, price, publish) | 2,860 AI-generated listings + acceptance patterns |
| **Multi-channel optimization** | AI decides which channels to emphasize per listing (Google for electronics, Meta for fashion) | Distribution feed performance + ads-history data |
| **Demand alerts** | "Searches for 'iPhone 13 Casa' up 45% this week — list yours NOW" | ta_search_history (1.5M) real-time trend detection |
| **Competitor intelligence** | "3 similar items priced lower appeared today. Consider adjusting." | OpenSearch similar-items + new listing indexing |
| **Review solicitation** | AI sends review request at optimal time after delivery (learned from data) | ta_review (248) + delivery timestamps + open patterns |
| **Store video optimization** | AI suggests when to post store videos for maximum engagement | organic-history/meta + TikTok engagement patterns |
| **WhatsApp auto-reply** | Buyer sends WhatsApp → AI responds with listing details, availability, price | WhatsApp WABA + listing data + seller rules |
| **Order confirmation** | AI prepares order → presents to seller → ONE TAP to confirm → shipping instructions generated | ta_order lifecycle + Tawssil integration |

**THE LEARNING LOOP:**
```
Day 1: AI uses rules (seller sets min price, max discount, response template)
Day 7: AI learns seller's PATTERNS (when they accept, what language they use, how they counter)
Day 30: AI can replicate the seller's STYLE (tone, negotiation strategy, pricing instinct)
Day 90: AI performs BETTER than the seller (access to market data seller can't see)

EVERY override by the seller = training signal:
- Seller edits AI response → AI learns their voice
- Seller rejects AI price suggestion → AI adjusts pricing model
- Seller accepts an offer AI would have rejected → AI broadens acceptance criteria
- Seller declines an offer AI would have accepted → AI tightens criteria
```

**WHAT THE SELLER ACTUALLY DOES (at full AI maturity):**
1. Uploads photos (or just takes them — AI does the rest)
2. Sets boundaries (min price, max discount, delivery preferences)
3. Confirms orders (one tap in WhatsApp or dashboard)
4. Ships items (Tawssil handles logistics)
5. Counts money

**EVERYTHING ELSE = AI.**

---

### 📊 AGENT 3: THE ADVERTISER AGENT (Tawadoo Retail Media Network)

**What it does:** Turns Tawadoo into a SELF-SERVE AD PLATFORM for brands and businesses. Like Amazon Ads but for North Africa.

> **Research insight (Topsort):** "Unlike traditional digital advertising, retail media has a clean feedback loop: did the person who saw the ad go on to buy the product? First-party purchase data makes this possible."
> **Research insight (Kevel):** "Audience segmentation harnesses first-party data into a single customer view for AI-powered segmentation and ad personalization."

**WHY TAWADOO ADS ARE UNIQUELY POWERFUL:**

We have what NO external ad network has:
1. **Purchase intent signals** — auth gate context (buy_now/bid/offer) = ACTUAL buying intent, not inferred
2. **Search queries** — 1.5M searches = what people WANT to buy right now
3. **Auction behavior** — bid aggressiveness = spending willingness
4. **Category affinity** — browsing patterns = interest graph
5. **Price sensitivity** — offer amounts, filter usage = budget signals
6. **Closed-loop attribution** — we own the ENTIRE funnel from impression to purchase to delivery
7. **Trilingual targeting** — reach users in their ACTUAL language (not assumed from geo)

**THE ADVERTISER PLATFORM:**

| Feature | What it Does | Data Source |
|---|---|---|
| **Self-serve campaign creation** | Brand uploads creative, sets budget, picks targeting | BO module (already scaffolded as display-ads) |
| **AI targeting** | "Show to users likely to buy electronics in Casa this week" | Search intent + category affinity + purchase history |
| **Behavioral targeting** | "Show to users who searched for X but haven't purchased yet" | ta_search_history + ta_publication_view + ta_order (absence) |
| **Lookalike audiences** | "Find users similar to my best customers" | Amplitude cohorts + embedding similarity |
| **Real-time bidding (internal)** | Multiple advertisers compete for same slot → highest bid × quality score wins | Internal auction engine |
| **Creative optimization** | AI tests multiple creatives, auto-allocates budget to winner | CTR prediction model on creative features |
| **Performance dashboard** | Real-time: impressions, clicks, CTR, conversions, ROAS, cost per acquisition | ta_sponsor_event + ta_order attribution |
| **AI recommendations** | "Your best audience is women 25-34 in Casablanca searching for fashion" | Derived from campaign performance + demographic signals |
| **Budget optimization** | AI paces spend to maximize conversions within budget | Reinforcement learning on campaign performance |
| **Cross-channel attribution** | "This user saw your ad on Tawadoo, then bought on your site" | UTM + fbclid + ttclid tracking |
| **Seasonal prediction** | "Demand for your category spikes in 2 weeks (Ramadan/Eid/school). Increase budget now." | Historical seasonal patterns from GA4 + search trends |

**THE ADS DATA FLYWHEEL:**
```
More ads run → More impression/click/conversion data collected
→ Better CTR prediction model → Higher quality ads shown
→ Higher CTR → More revenue per impression → More advertiser ROI
→ More advertisers → More competition → Higher bids → More revenue
→ Better user experience (relevant ads) → More engagement → LOOP
```

**HOW ADS/ORGANIC DATA PLAYS IN (from lake):**

| External Data Source | How it Feeds the Ads Platform |
|---|---|
| **Google Ads search terms** (in lake) | Shows WHAT buyers search for → informs ad targeting suggestions |
| **Meta Ads audience insights** (in lake) | Shows demographic performance → refines Tawadoo's targeting AI |
| **TikTok creative performance** (in lake) | Shows which video styles convert → AI coaches advertisers on creatives |
| **GSC organic queries** (48K impressions) | Shows organic demand → identifies categories where ads would accelerate |
| **GA4 channel attribution** | Shows which external channels feed which user types → cross-channel optimization |
| **Bing indexed pages** (44K) | Shows content gaps where ads could fill demand → programmatic content + ads synergy |
| **MCP/ChatGPT queries** | Shows AI-referred demand → "ChatGPT users search for X — target them" |

**REVENUE MODEL FOR TAWADOO:**
- CPC (cost per click): Advertisers pay when users click
- CPA (cost per acquisition): Advertisers pay ONLY when purchase happens (risk-free for them)
- Sponsored listings: Pay to appear first in search results (like Google Shopping but inside Tawadoo)
- Premium placement: Homepage banners, category headers, feed interstitials
- Data insights reports: Sell aggregated market intelligence (anonymized) to brands

---

## THE PLATFORM AI (what ties it all together)

### AI-POWERED SEARCH ENGINE (self-improving)

The search engine isn't static. It LEARNS from every interaction:

```
CURRENT: BM25 (keyword) + kNN (vector) + cross-lingual tags → static weights (0.6/0.3/0.1)

V2: ADAPTIVE RANKING
- Weights adjust PER QUERY TYPE:
  - Exact product name ("iPhone 13 Pro") → keyword weight ↑ (0.8)
  - Descriptive query ("vélo pour enfant pas cher") → vector weight ↑ (0.5)
  - Darija query ("tomobile occasion") → expansion weight ↑, fuzzy ↑
  
- USER-PERSONALIZED RANKING:
  - User A always clicks cheapest → boost low price
  - User B always clicks best condition → boost quality
  - User C always clicks nearest → boost location proximity
  
- REAL-TIME DEMAND SIGNALS:
  - Query volume spike → boost new listings in that category
  - Zero-result queries accumulate → trigger supply recruitment notification
  
- ADVERTISER BOOST (paid placement):
  - Sponsored results interleaved naturally (labeled "Sponsored")
  - Position determined by bid × quality score × user relevance
```

**Training signals (all from events we're tracking):**
- `search_click_signal` → positive relevance (clicked = relevant)
- `search_skip_signal` → negative relevance (visible but not clicked)
- `search_purchase_signal` → STRONG positive (searched → clicked → BOUGHT)
- `listing_dwell_time > 30s` → moderate positive (interested)
- `listing_dwell_time < 5s` → moderate negative (irrelevant)
- `offer_submitted after search` → very strong positive (intent to transact)
- `bid_placed after search` → very strong positive

### AI-POWERED RECOMMENDATIONS (replacing static sections)

```
CURRENT: "Trending" = just latest. No personalization.

V2: EVERY section is AI-driven:
- "For You" = collaborative filtering (users like you bought these)
- "Trending in [your city]" = real trending from view velocity
- "Price Drop Alerts" = items you viewed that got cheaper
- "Similar to your last search" = vector similarity from session context
- "Sellers you follow have new items" = social graph
- "Back in stock" = items you viewed that were sold and relisted
- "Finishing soon" = auctions you watched nearing end
```

### AI-POWERED NOTIFICATIONS (optimal timing + content)

```
CURRENT: Send notifications on lifecycle triggers (boost expiring, order shipped, etc.)

V2: AI decides WHEN, WHAT, and HOW:
- WHEN: optimal send time per user (learned from open patterns)
- WHAT: which notification has highest P(action) for this user right now
- HOW: which channel (push vs WhatsApp vs in-app) converts best for this user
- CONTENT: personalized copy (seller name, listing thumbnail, price in their currency preference)
- FREQUENCY: respects fatigue (never > 3/day, backs off if ignored)
```

### AI-POWERED FRAUD & TRUST

```
SIGNALS THAT INDICATE FRAUD:
- Multiple accounts from same device fingerprint (ta_guest: 1.5M fingerprints)
- Price dramatically below market (computed from category averages)
- Stock photos (Bedrock image analysis → detect generic/stolen images)
- Seller with 0 response rate + many listings (typical scammer pattern)
- New account + immediate high-value listings
- VPN/proxy usage (from ta_guest device analysis)
- Copy-paste descriptions from other listings (embedding similarity > 0.95)
- Bid manipulation (bidding on own auctions from different account)

AI ACTIONS:
- Flag for review (low confidence)
- Auto-reject listing (high confidence scam)
- Shadow-ban (user doesn't know they're flagged, but listings get 0 distribution)
- Alert admin (critical: financial loss pattern)
```

### AI-POWERED MOOD/INTENT PREDICTION

```
SIGNALS THAT REVEAL USER STATE:
- Fast browsing, no dwell → "window shopping" / casual browsing
- Deep search + filter refinement + price comparison → "ready to buy"
- Repeated returns to same listing → "wants it but hesitant" (price? trust?)
- Added to favorites but no contact → "waiting for price drop"
- Auth gate encounter → the ACTION they tried to take tells us exactly what they want
- Time of day + day of week → "lunch break browser" vs "evening serious shopper"
- Session frequency increasing → engagement growing (opportunity)
- Session frequency decreasing → churn signal (intervene)
- Auction: bid in last 30 seconds → "sniper" personality
- Auction: bid early and auto-bid → "committed buyer" personality
- Offer always 30%+ below → "aggressive negotiator" → match with flexible sellers
- Never makes offers, always buys at asking → "convenience buyer" → show Buy Now prominently

THE AI USES THIS TO:
- Personalize the UI (show Buy Now for convenience buyers, Make Offer for negotiators)
- Time notifications (ping the "lunch break browser" at 12:30)
- Target ads (show premium products to high-intent users)
- Coach sellers ("This buyer is a sniper — set auto-bid protection")
- Predict conversion ("78% chance this user buys within 24h — boost relevant listings")
```

---

## WHAT THE DATA LAKE + EXTERNAL SOURCES FEED

### FROM GOOGLE ADS (in lake: campaign_daily, keywords, search_terms)
→ **Demand intelligence:** What are buyers searching for ON Google that we can capture?
→ **Keyword insights for sellers:** "People search for 'iphone 13 pro max occasion maroc' — title your listing with these exact words"
→ **Category opportunity detection:** High Google search volume + low Tawadoo supply = recruitment target

### FROM META ADS (in lake: insights_ad_daily, audiences, video_retention)
→ **Creative best practices:** Which ad formats/styles convert? → Coach sellers on photos/videos
→ **Audience lookalike signals:** High-value Meta converters → identify similar users on Tawadoo
→ **Video engagement patterns:** Which intros keep attention? → AI guides store video creation

### FROM TIKTOK ADS (in lake: full_pull)
→ **UGC style insights:** What organic-feeling content converts? → Inform AI content generation
→ **Young demographic behavior:** TikTok audience = Tawadoo's growth segment

### FROM GSC (48K impressions, 991 clicks)
→ **Organic opportunity map:** Queries with high impressions but low clicks = fix meta descriptions
→ **Content gap identification:** Queries with 0 Tawadoo results = create programmatic pages
→ **Category SEO health:** Which categories rank? Which don't? → Focus AI content engine

### FROM BING (44K indexed pages, 14.45% CTR)
→ **AI discoverability:** Bing feeds Copilot/ChatGPT. Indexed = AI-findable.
→ **Page quality signals:** High Bing CTR = content that satisfies intent

### FROM MCP/CHATGPT (155KB metrics, 272 web users from "AI Assistant" channel)
→ **AI commerce demand:** What do people ASK ChatGPT about Morocco commerce?
→ **Tool usage patterns:** Which MCP tools are most called? → Optimize those endpoints
→ **Conversion patterns:** MCP → visit → purchase attribution → prove AI channel value

### FROM MICROSOFT CLARITY (session recordings, heatmaps)
→ **Rage click detection:** Where do users get frustrated? → Fix UX
→ **Dead click detection:** What looks clickable but isn't? → Fix affordance
→ **Scroll drop-off:** Where do users stop scrolling? → Reorder content
→ **Form abandonment:** Where in checkout do users leave? → Simplify

---

## COST & SCALE ANALYSIS

### COST OF THE AI LAYER (realistic)

| Component | Current Monthly | V2 Monthly | Notes |
|---|---|---|---|
| AWS Bedrock (7 models) | ~$150 (budget alarm) | ~$300-500 | More invocations (seller agents, content engine) |
| Amplitude | $0 (100M free) | $0 | Still under free quota even at 5M events/month |
| OpenSearch (1 node) | ~$80 | ~$160 (2 nodes) | Scale for real-time recommendations |
| ECS Fargate (4 services) | ~$200 | ~$350 (auto-scaling) | Agent processing + widget backend |
| S3 + Data Lake | ~$10 | ~$30 | Amplitude export + expanded lake |
| Redis | ~$30 | ~$50 | Feature cache + real-time predictions |
| CloudFront | ~$20 | ~$40 | Media CDN (fix the S3 direct issue) |
| **TOTAL** | **~$490/month** | **~$1,130/month** | +$640/month for full AI layer |

**Revenue needed to justify:** At current coin-pack revenue (35.7K MAD lifetime), the AI layer pays for itself if it increases monetization by just 2%. That's 3 extra coin pack sales per month.

### SCALE PLAN

| Users | Events/month | Bedrock Calls/month | ECS Tasks | Cost/month |
|---|---|---|---|---|
| 60K (now) | 5M (after instrumentation) | ~50K | 5-7 | ~$1,130 |
| 150K (6 months) | 15M | ~150K | 8-12 | ~$2,500 |
| 500K (12 months) | 50M | ~500K | 15-25 | ~$6,000 |
| 1M (18 months) | 100M | ~1M (SageMaker transition) | 30-50 | ~$12,000 |

At 1M users, self-hosted models (NVIDIA NIM) become cheaper than Bedrock API calls. That's when NVIDIA credits become critical.

### RISK ANALYSIS

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| AI hallucination in negotiation | Medium | High (bad deal) | Seller approval required for counter-offers > seller's max discount |
| AI responds incorrectly about product | Medium | Medium | Responses grounded in listing data only, never hallucinated details |
| Ad click fraud | Low | Medium | Fingerprint dedup + velocity limits + ML anomaly detection |
| Data privacy (CNDP/GDPR) | Low | High | All AI trains on BEHAVIORS not IDENTITIES. No PII in models. |
| Seller over-reliance on AI | Medium | Low | AI quality degrades without seller feedback → natural balance |
| Cost overrun on Bedrock | Low | Medium | Budget alarms + rate limits + request caching |
| ChatGPT platform changes (ACP) | Medium | Medium | MCP is OUR server (we control). ACP is open standard. |
| Competitor copies approach | Low | Very Low | 5.7M data rows + 22 integrations = 12+ months head start |

---

## THE AGENTIC COMMERCE PROTOCOLS (ACP → UCP → MCP)

### MCP (Model Context Protocol) — WE HAVE THIS LIVE
- ChatGPT app at mcp.tawadoo.ma
- 16 tools (search, recommendations, market insights, etc.)
- Users can DISCOVER and BROWSE Tawadoo products in ChatGPT

### ACP (Agentic Commerce Protocol) — NEXT STEP
- OpenAI + Stripe open standard (launched 2025)
- Enables "Buy it in ChatGPT" — instant checkout without leaving
- Tawadoo implements: product feed → ChatGPT indexes → user asks → ChatGPT shows → user buys → our API processes
- We apply to OpenAI's Instant Checkout partner program (we already have the ChatGPT app!)

### UCP (Universal Commerce Protocol) — OUR VISION
- Not just buy in ChatGPT. SELL in ChatGPT.
- Seller uploads photos to ChatGPT → our MCP tool creates listing → AI optimizes → published
- Seller manages business from ChatGPT conversations ("How are my listings doing?" → MCP returns analytics)
- Buyer negotiates with seller's AI agent — BOTH parties in ChatGPT, marketplace is the invisible middleware

**THE ULTIMATE STATE:**
```
Buyer: "I want a used iPhone 13 in Casablanca under 3000 MAD"
ChatGPT (via MCP): finds 5 options, shows with prices, ratings, delivery estimates
Buyer: "The second one. Can I get it for 2700?"
ChatGPT (via ACP + Seller Agent): sends offer, seller's AI considers, counters at 2800
Buyer: "Deal."
ChatGPT (via ACP): processes payment (Stripe/Payzone), creates order, arranges Tawssil delivery
Buyer: gets notification when shipped, when delivered
Seller: got a WhatsApp "Sale confirmed! 2800 MAD. Ship to Casablanca address. Tap to print label."

NO ONE VISITED TAWADOO.MA. Yet we processed the entire transaction.
The marketplace is invisible infrastructure. AI is the interface.
```

---

## EVENTS FOR THE NEW DOMAINS (adding to taxonomy)

### DOMAIN 20: AUTONOMOUS SELLER AGENT (15 events)

| # | Event | Properties | Phase |
|---|---|---|---|
| 313 | `seller_agent_auto_responded` | thread_id, response_length, buyer_satisfaction_predicted, seller_override_needed | P2 |
| 314 | `seller_agent_offer_handled` | offer_id, action (accept/counter/reject), within_bounds, seller_notified | P2 |
| 315 | `seller_agent_override` | action_type, original_ai_action, seller_correction, learning_signal | P1 |
| 316 | `seller_agent_boost_triggered` | listing_id, reason (demand_spike/expiry_prevention/underperforming), coins_spent | P2 |
| 317 | `seller_agent_price_suggested` | listing_id, current_price, suggested_price, market_context, accepted | P1 |
| 318 | `seller_agent_listing_relisted` | listing_id, changes_made[], price_change_pct | P2 |
| 319 | `seller_agent_demand_alert_sent` | category, city, demand_signal_strength, channel | P2 |
| 320 | `seller_agent_whatsapp_replied` | thread_id, message_type (inquiry/availability/price), response_time_ms | P2 |
| 321 | `seller_agent_order_prepared` | order_id, actions_taken (label/confirm/instructions), seller_tap_time_ms | P2 |
| 322 | `seller_agent_review_solicited` | order_id, timing_reason, channel, response_received | P2 |
| 323 | `seller_agent_competitor_alert` | listing_id, competitor_listing_id, price_difference, action_suggested | P2 |
| 324 | `seller_agent_performance_report` | period, views_driven, contacts_generated, sales_attributed, roi_vs_manual | P2 |
| 325 | `seller_agent_activated` | seller_id, tier, rules_configured (min_price/max_discount/auto_respond) | P1 |
| 326 | `seller_agent_deactivated` | seller_id, reason, days_active, performance_during | P2 |
| 327 | `seller_agent_learning_signal` | signal_type (override/correction/approval), category, confidence_change | P2 |

### DOMAIN 21: BUYER AGENT & VOICE/IMAGE (12 events)

| # | Event | Properties | Phase |
|---|---|---|---|
| 328 | `buyer_agent_search_conversational` | query_natural_language, intent_parsed, results_quality | P1 |
| 329 | `buyer_agent_recommendation_shown` | listing_ids[], algorithm, user_context, personalization_score | P1 |
| 330 | `buyer_agent_recommendation_clicked` | listing_id, position, algorithm, relevance_feedback | P1 |
| 331 | `buyer_agent_price_alert_triggered` | alert_id, listing_id, target_price, actual_price, notification_channel | P1 |
| 332 | `buyer_agent_negotiation_assisted` | listing_id, suggested_amount, buyer_accepted_suggestion, outcome | P2 |
| 333 | `buyer_agent_trust_check_performed` | seller_id, trust_score, warnings[], buyer_proceeded | P2 |
| 334 | `voice_search_initiated` | language_detected, duration_seconds, transcription_confidence | P1 |
| 335 | `voice_search_results_spoken` | results_count, items_read_aloud, user_action_after | P2 |
| 336 | `image_search_initiated` | source (camera/gallery/paste), image_size, processing_time_ms | P0 |
| 337 | `image_search_results_shown` | results_count, top_category, similarity_scores[], user_clicked | P1 |
| 338 | `buyer_agent_whatsapp_query` | query_type (text/image/voice), response_type, action_taken | P2 |
| 339 | `buyer_mood_predicted` | mood (browsing/researching/ready_to_buy/hesitant/frustrated), confidence, signals_used[] | P2 |

### DOMAIN 22: RETAIL MEDIA / ADVERTISER AGENT (10 events)

| # | Event | Properties | Phase |
|---|---|---|---|
| 340 | `advertiser_campaign_ai_optimized` | campaign_id, optimization_type (budget/targeting/creative/timing), improvement_expected_pct | P2 |
| 341 | `advertiser_audience_auto_refreshed` | campaign_id, audience_size_before, audience_size_after, criteria_adjusted | P2 |
| 342 | `advertiser_creative_auto_paused` | campaign_id, creative_id, reason (low_ctr/high_cost/fatigue), replacement_suggested | P2 |
| 343 | `advertiser_budget_auto_reallocated` | from_campaign_id, to_campaign_id, amount, reason (roas_differential) | P2 |
| 344 | `advertiser_roi_milestone` | campaign_id, milestone (first_click/first_conversion/roas_2x/roas_5x), time_to_milestone | P1 |
| 345 | `advertiser_insight_generated` | campaign_id, insight_type, recommendation, data_source | P2 |
| 346 | `ad_internal_auction_completed` | slot, winning_campaign_id, bid_amount, quality_score, competing_bids_count | P2 |
| 347 | `ad_predictive_ctr_scored` | campaign_id, creative_id, user_segment, predicted_ctr, actual_ctr | P2 |
| 348 | `retail_media_revenue_daily` | total_revenue_mad, campaigns_active, avg_cpc, avg_cpa, fill_rate | P1 |
| 349 | `advertiser_self_serve_onboarded` | advertiser_id, category, initial_budget, targeting_type | P1 |

### DOMAIN 23: PREDICTIVE INTELLIGENCE (8 events — computed, not user-triggered)

| # | Event | Properties | Phase |
|---|---|---|---|
| 350 | `prediction_conversion_scored` | user_id, score, top_signals[], recommended_action | P2 |
| 351 | `prediction_churn_scored` | user_id, risk_score, days_to_predicted_churn, intervention_sent | P2 |
| 352 | `prediction_ltv_computed` | user_id, predicted_ltv_mad, segment (low/medium/high/whale), features_used | P2 |
| 353 | `prediction_price_suggested` | listing_id, suggested_price, confidence, comparable_listings[], market_velocity | P2 |
| 354 | `prediction_demand_forecasted` | category, city, forecast_7d, confidence, trend (rising/stable/falling) | P2 |
| 355 | `prediction_fraud_flagged` | user_id_or_listing_id, risk_score, signals[], action_taken (flag/block/shadow) | P1 |
| 356 | `prediction_notification_timed` | user_id, notification_type, optimal_hour, optimal_channel, confidence | P2 |
| 357 | `prediction_match_scored` | buyer_id, listing_id, match_score, factors[], surfaced_in (feed/recommendation/search) | P2 |

---

## FINAL TOTALS

| Domain | Events |
|---|---|
| 1-13 (original) | 240 |
| 14. Conversational AI & Widget | 15 |
| 15. Display Ads Intelligence | 15 |
| 16. Blog & Content Engine | 10 |
| 17. Conversations & Social | 12 |
| 18. Lifecycle & Retention | 12 |
| 19. AI Search Engine | 8 |
| **20. Autonomous Seller Agent** | **15** |
| **21. Buyer Agent & Voice/Image** | **12** |
| **22. Retail Media / Advertiser Agent** | **10** |
| **23. Predictive Intelligence** | **8** |
| **GRAND TOTAL** | **357 events** |

### FUNNELS: 25 total
Original 20 + 5 new:
- 21: Seller Agent Effectiveness (activation → auto-actions → seller approval rate → sales attributed)
- 22: Buyer Agent Conversion (widget/voice/image → search → results → action → purchase)
- 23: Advertiser Self-Serve (onboard → create campaign → first impression → first click → first conversion → ROAS milestone)
- 24: Predictive Action Loop (prediction scored → intervention sent → user responded → outcome matched prediction?)
- 25: Cross-Channel Commerce (ChatGPT query → MCP results → ACP checkout → delivery → review)

---

## THE INVESTOR PITCH (one paragraph)

> Tawadoo is building Africa's first agentic commerce platform. Every seller gets an autonomous AI agent that sells for them 24/7 — negotiating, promoting, pricing, and responding across WhatsApp, ChatGPT, and 5 distribution channels. Every buyer gets a personal shopping AI that finds, compares, negotiates, and purchases in their language (including Moroccan Darija). Every advertiser gets an AI media buyer with first-party purchase intent data that no external ad network can match. The platform learns from 5.7M behavioral data points and 22 API integrations, with models retraining weekly from real transaction outcomes. Built on AWS Bedrock, OpenSearch, and 100M free events/month from Amplitude. Already live in Morocco with 60K users, 26K listings, and 35.7K MAD in coin-economy revenue. Comparable: Agenz raised $5M seed (June 2026, Breega) for AI real-estate valuation in Morocco. Tawadoo is the full marketplace AI stack — price intelligence + seller agents + buyer agents + media network — across EVERY category.

---

*This is the destination. 357 events. 25 funnels. 3 autonomous agents. 1 retail media network. The path starts with Session 14A (65 P0 events). Everything builds toward this.*

*Tawadoo, the marketplace that gets you — because AI literally understands you.*


---

## PRICE INTELLIGENCE ENGINE (External Data + Internal Signals)

### THE PROBLEM
A seller asks: "How much should I sell my iPhone 13 for?" Today we have NO answer. Tomorrow, our AI knows EXACTLY — because it has:
- Internal: 891K views with prices + 595 offers (131 accepted at what prices) + 42.8K wallet transactions
- External: Avito.ma competitor prices, Jumia.ma new prices, international benchmarks

### EXTERNAL PRICE DATA SOURCES (legal, public scraping)

| Source | What to Scrape | Frequency | Legal? | Storage |
|---|---|---|---|---|
| **Avito.ma** (competitor) | Category × city × condition → prices | Weekly per category | ✅ Public listings (no auth required) | S3: `lake/price-intelligence/avito/` |
| **Jumia.ma** (new prices) | Category → brand → model → new retail price | Weekly | ✅ Public catalog (no auth required) | S3: `lake/price-intelligence/jumia/` |
| **Amazon.fr** (reference) | Electronics/fashion brand × model → EU price | Monthly | ✅ Public prices | S3: `lake/price-intelligence/amazon/` |
| **eBay sold items** (global used market) | Category × condition → sold prices | Monthly | ✅ eBay API (free tier) | S3: `lake/price-intelligence/ebay/` |
| **Google Shopping** (price comparison) | Product name → multiple retailer prices | Monthly | ✅ Google Shopping API or scrape | S3: `lake/price-intelligence/google/` |

**Implementation:** Apify actors (cloud scraping, $0.01/result) running on schedule via EventBridge. Results stored as NDJSON in the lake. No code on our servers — Apify handles the scraping, we just pull results.

### THE PRICE INTELLIGENCE MODEL

```
INPUTS:
- Internal sold prices (ta_offer accepted_price by category × city × condition)
- Internal asking prices (ta_publication.price by category × city)
- Internal view velocity (views/day for a given price point)
- External competitor prices (Avito: same category × city × condition)
- External new prices (Jumia: same brand × model retail price)
- Time on market (days listed without sale at various price points)
- Demand signals (search volume for this item type this week)
- Seasonality (Eid, Ramadan, back-to-school from historical patterns)

OUTPUT:
- Suggested price (MAD)
- Price range (min likely to sell / max before it sits)
- Time-to-sell prediction at different price points
- Confidence level
- Comparison: "15% below Avito average, 40% below Jumia new"

DISPLAY:
- Seller sees: "Suggested price: 3,200 MAD (sells in ~5 days)"
- Seller sees: "At 3,500 MAD it may take 2-3 weeks"
- Seller sees: "Similar items on Avito: 3,000-3,800 MAD"
- Buyer sees: "This price is 12% below market average ✓ Good deal"
```

### EVENTS FOR PRICE INTELLIGENCE

| # | Event | Properties | Phase |
|---|---|---|---|
| 358 | `price_suggestion_shown` | listing_id, suggested_price, actual_price, market_data_source, confidence | P1 |
| 359 | `price_suggestion_accepted` | listing_id, suggested_price, final_price_set, adjustment_pct | P1 |
| 360 | `price_suggestion_rejected` | listing_id, suggested_price, seller_price, reason_inferred | P2 |
| 361 | `price_comparison_viewed` | listing_id, competitor_prices[], market_median, user_type (buyer/seller) | P1 |
| 362 | `external_price_scraped` | source (avito/jumia/ebay), category, items_count, avg_price, freshness | P2 (server) |
| 363 | `price_drop_detected` | listing_id, old_price, new_price, drop_pct, alert_triggered | P1 |
| 364 | `price_alert_set` | category, target_price, condition, city, notification_method | P1 |
| 365 | `price_alert_triggered` | alert_id, listing_id, price_matched, user_action_after | P1 |

---

## LISTING CREATION VIA CHATGPT ("Sell in ChatGPT")

### HOW IT WORKS

```
SELLER IN CHATGPT:
"I want to sell this" + [attaches 3 photos]

CHATGPT (via our MCP tools):
1. Calls `create_listing` MCP tool with photo URLs
2. Our API: Bedrock Nova Lite classifies → category detected
3. Our API: Bedrock text model generates title + description (trilingual)
4. Our API: Titan embeds images for future search
5. ChatGPT responds: "I found this is an iPhone 13, 128GB, good condition.
   Here's what I've prepared:
   - Title: iPhone 13 128Go Blanc Très Bon État
   - Category: High-Tech > Smartphones
   - Suggested price: 3,200 MAD (market average: 3,400 MAD)
   - Description: [generated]
   Should I publish this on Tawadoo? You can adjust anything."

SELLER: "Change price to 3000 and publish"

CHATGPT: 
6. Calls `publish_listing` MCP tool with adjustments
7. Listing goes live on Tawadoo + all distribution channels
8. "Done! Your listing is live. I'll notify you when someone's interested."

AFTER PUBLICATION (seller's AI agent takes over):
- Buyer contacts → agent responds
- Buyer makes offer → agent negotiates (within seller's bounds)
- Deal reached → seller gets WhatsApp "Confirm order? [Yes/No]"
- Seller taps Yes → Tawssil creates shipment → done
```

### MCP TOOLS NEEDED (additions to existing 16)

| Tool | What it Does | Parameters |
|---|---|---|
| `create_listing_from_photos` | AI generates full listing from images | photos[], seller_notes, price_hint |
| `publish_listing` | Makes a draft listing go live | listing_id, price_adjustments, publish_now |
| `get_my_listings_status` | Seller checks performance | seller_id, include_stats |
| `respond_to_buyer` | Agent replies to inquiry | thread_id, response_text, auto_generated |
| `accept_offer` | Seller confirms deal via ChatGPT | offer_id, seller_confirmation |
| `get_price_suggestion` | Market price for an item | category, brand, model, condition, city |

### EVENTS

| # | Event | Properties | Phase |
|---|---|---|---|
| 366 | `listing_created_via_chatgpt` | listing_id, photos_count, ai_category, ai_price_suggestion, seller_adjustments | P1 |
| 367 | `listing_published_via_chatgpt` | listing_id, time_from_photos_to_live_seconds | P1 |
| 368 | `seller_chatgpt_session` | actions_taken[], session_duration, listings_managed | P2 |

---

## THE WIDGET AS SEARCH BAR (importing ChatGPT-style conversational UI)

### THE CONCEPT

Replace (or augment) the traditional search bar with a conversational input:

```
CURRENT: [Search bar] → type keywords → see grid of results

V2: [Conversational search] → type/speak naturally → AI finds + presents + acts

Examples:
- "Bghit chi telfon samsung b moins de 2000" → AI returns filtered results
- "Montrez-moi des appartements 3 chambres Casablanca" → category + filter auto-applied
- [uploads photo of a shoe] → "I found 8 similar items near you"
- [voice] "Cherche-moi un vélo pour enfant" → voice transcription → search → results
- "What's a good price for an iPhone 13?" → price intelligence response (no listing needed)
- "Show me what's trending in Rabat" → real-time demand data
```

### TECHNICAL APPROACH

NOT importing OpenAI's widget (their widget stays in ChatGPT). Instead:
- Our OWN conversational UI component, powered by our MCP backend
- The same API that serves ChatGPT (MCP tools) powers our in-site widget
- This means: ONE intelligence backend, TWO interfaces (ChatGPT + our widget)
- Widget uses Bedrock for NLU (intent classification) → MCP tools for execution → response rendering

### WHY OUR OWN WIDGET (not embedding ChatGPT):
1. We control the experience (no OpenAI branding/limits)
2. We control the data (every interaction goes to OUR lake, not OpenAI's)
3. We can upsell (widget can show boost CTAs, suggest premium features)
4. We avoid API costs (Bedrock is cheaper than OpenAI API at scale)
5. We own the training data (interactions improve OUR models, not OpenAI's)

### EVENTS

| # | Event | Properties | Phase |
|---|---|---|---|
| 369 | `conversational_search_initiated` | input_type (text/voice/image), language, query_raw | P0 |
| 370 | `conversational_intent_classified` | intent (search/price_check/sell/negotiate/track_order), confidence, model_used | P1 |
| 371 | `conversational_results_rendered` | results_count, response_type, latency_ms | P1 |
| 372 | `conversational_voice_transcribed` | language_detected, confidence, transcription_length, darija_detected | P1 |

---

## THE TRAINING DATA PIPELINE (Surgically Clean, Ready to Curate)

### THE PRINCIPLE

Every AI interaction is a training opportunity. The system SHADOWS every decision:
- When AI generates → we store input + output + what the human did with it
- When AI classifies → we store the classification + whether human agreed
- When AI suggests → we store suggestion + acceptance/rejection
- When AI negotiates → we store the conversation + outcome

**The moment we're ready to train proprietary models, the data is ALREADY curated, labeled, and formatted.**

### THE SHADOW LEDGER (what gets logged to the lake)

```
S3: tawadoo-core-intelligence-lake/training-pipeline/

Structure:
├── classification/
│   ├── category_classification/
│   │   └── {timestamp}_{listing_id}.json
│   │   Each entry: {
│   │     "input_images": ["s3://..."],
│   │     "ai_prediction": {"category": "electronics/smartphones", "confidence": 0.94},
│   │     "human_action": "accepted" | "changed_to:vehicles/cars",
│   │     "model_id": "nova-lite-v1",
│   │     "timestamp": "...",
│   │     "latency_ms": 1200
│   │   }
│   └── intent_classification/
│       └── {timestamp}_{session_id}.json
│       Each entry: {
│         "query": "bghit chi tomobile",
│         "ai_intent": "search:vehicles",
│         "darija_expanded": "voiture|automobile|سيارة",
│         "results_returned": 47,
│         "user_clicked_position": 3,
│         "user_purchased": true/false
│       }
│
├── generation/
│   ├── listing_titles/
│   │   └── {timestamp}_{listing_id}.json
│   │   Each entry: {
│   │     "input": {"images": [...], "category": "...", "properties": {...}},
│   │     "ai_output": {"title_fr": "...", "title_ar": "...", "title_en": "..."},
│   │     "human_action": "accepted_as_is" | "edited",
│   │     "human_edit": {"title_fr": "...", "edit_distance": 0.23},
│   │     "model_id": "gpt-4o-mini",
│   │     "tokens_in": 450, "tokens_out": 120,
│   │     "cost_usd": 0.002
│   │   }
│   ├── listing_descriptions/
│   │   └── (same structure)
│   ├── chat_responses/ (when seller agent auto-responds)
│   │   └── {timestamp}_{thread_id}.json
│   │   Each entry: {
│   │     "buyer_message": "C'est disponible?",
│   │     "ai_response": "Oui, disponible. Vous pouvez passer quand vous voulez.",
│   │     "seller_override": null | "Oui disponible, mais uniquement le matin.",
│   │     "buyer_satisfied": true (inferred from continued engagement),
│   │     "model_id": "...",
│   │     "seller_id": "..."
│   │   }
│   └── price_suggestions/
│       └── {timestamp}_{listing_id}.json
│       Each entry: {
│         "category": "...", "brand": "...", "condition": "...", "city": "...",
│         "ai_suggested_price": 3200,
│         "market_data": {"avito_avg": 3400, "jumia_new": 5999, "internal_median": 3100},
│         "seller_final_price": 3000,
│         "time_to_first_lead_days": 2.5,
│         "sold_price": 2800 (if sold),
│         "model_version": "price-v1"
│       }
│
├── search_relevance/
│   └── weekly_batches/
│       └── week_{YYYY_WW}.parquet
│       Each row: {
│         "query": "...",
│         "results_ranked": [listing_ids in order shown],
│         "user_clicked": [listing_ids clicked],
│         "user_purchased": [listing_ids purchased],
│         "dwell_times": [seconds per clicked listing],
│         "filters_applied": {...},
│         "user_locale": "fr",
│         "darija_expansion_used": true/false
│       }
│
├── negotiation/
│   └── {timestamp}_{offer_id}.json
│   Each entry: {
│     "listing_id": "...",
│     "listed_price": 5000,
│     "offer_chain": [
│       {"actor": "buyer", "amount": 3500, "message": "..."},
│       {"actor": "seller_agent", "amount": 4200, "message": "..."},
│       {"actor": "buyer", "amount": 4000},
│       {"actor": "seller", "action": "accept"}  // or "seller_agent" if AI handled
│     ],
│     "outcome": "sold_at_4000",
│     "seller_agent_active": true,
│     "seller_overrode": false,
│     "time_to_close_hours": 3.5
│   }
│
├── embeddings/
│   └── publication_embeddings/
│       └── {listing_id}.json
│       {
│         "image_embedding": [1024 floats],
│         "text_embedding": [1024 floats],
│         "model_versions": {"image": "titan-v2", "text": "titan-v2"},
│         "indexed_at": "..."
│       }
│
├── darija_nlp/
│   └── query_expansion_log.ndjson
│   Each line: {
│     "raw_query": "bghit chi telfon",
│     "detected_darija_terms": ["telfon"],
│     "expanded_to": ["téléphone", "هاتف", "phone"],
│     "results_before_expansion": 3,
│     "results_after_expansion": 47,
│     "user_satisfied": true (clicked result)
│   }
│   
│   └── unmatched_darija_queries.ndjson  (GOLD: queries we couldn't expand)
│   Each line: {
│     "query": "bghit chi 7aja zwinaaaa",
│     "zero_results": true,
│     "language_confidence": 0.89,
│     "candidate_for_dictionary": true
│   }
│
└── bedrock_invocations/
    └── daily/{YYYY-MM-DD}.ndjson
    Each line: {
      "model_id": "...",
      "purpose": "classification|generation|embedding|content",
      "input_tokens": 450,
      "output_tokens": 120,
      "latency_ms": 1200,
      "cost_estimate_usd": 0.002,
      "success": true,
      "timestamp": "..."
    }
```

### WHAT THIS ENABLES (when time comes to train)

| Training Target | Data in Pipeline | Format Ready? |
|---|---|---|
| **Category classifier fine-tune** | classification/category_classification/ (2,860+ samples with human corrections) | ✅ input→output→correction triplets |
| **Listing generator fine-tune** | generation/listing_titles/ + listing_descriptions/ | ✅ input→output→human_edit (SFT data) |
| **Price model training** | generation/price_suggestions/ + external scrapes | ✅ features→prediction→actual_outcome |
| **Search ranking (LTR)** | search_relevance/weekly_batches/ | ✅ query→ranked_results→clicks→purchases (graded relevance) |
| **Darija NLP expansion** | darija_nlp/query_expansion_log + unmatched queries | ✅ term→expansion→effectiveness |
| **Negotiation agent** | negotiation/ (full chains with outcomes) | ✅ conversation→outcome pairs (RLHF compatible) |
| **Chat response model** | generation/chat_responses/ (with seller overrides = preference data) | ✅ response pairs: AI vs human-preferred (reward model training) |
| **Recommendation model** | search_relevance/ + embeddings/ | ✅ user×item interactions with outcomes |
| **Fraud detection** | (from activity patterns, reports, blocks) | ✅ labeled positive/negative examples |

### THE SWITCH MOMENT

```
DAY 0 (now): Pipeline logs EVERYTHING. No training.
DAY 30: 50,000+ classification samples. 10,000+ generation samples. 100K+ search relevance rows.
DAY 60: Data quality review. Remove garbage. Label ambiguous cases.
DAY 90: FLIP THE SWITCH.
  - NeMo Curator cleans + deduplicates the corpus
  - SFT fine-tune on classification (LoRA on Nova Lite equivalent)
  - SFT fine-tune on generation (LoRA on GPT-4o-mini equivalent)  
  - LambdaMART trains on search_relevance data
  - XGBoost trains on price prediction data
  - Reward model trains on negotiation pairs (for RLHF)
  
DAY 91: A/B test: fine-tuned model vs base model. Measure:
  - Classification: accuracy improvement (fewer human corrections)
  - Generation: acceptance rate improvement (fewer edits)
  - Search: NDCG@10 improvement (better click-through)
  - Price: MAE reduction (closer to actual sold price)
  - Negotiation: deal closure rate improvement

IF BETTER: Deploy. IF SAME/WORSE: More data needed. Continue collecting.
```

### COST OF THE PIPELINE

| Component | Monthly Cost | Notes |
|---|---|---|
| S3 storage (training data) | ~$5-10 | Parquet + JSON, compressed |
| Apify scraping (price intel) | ~$20-50 | Weekly scrapes of Avito/Jumia/eBay |
| EventBridge triggers | ~$1 | Scheduling |
| Lambda for log processing | ~$5 | Transform + store |
| **Total pipeline cost** | **~$30-65/month** | Until training starts |
| Training cost (Day 90) | $200-500 one-time | DGX Cloud credits (NVIDIA) or SageMaker Spot |

---

## UPDATED GRAND TOTALS

| Metric | Count |
|---|---|
| **Events (tracked)** | **372** |
| **Funnels** | **25** |
| **Derived Signals** | **50+** |
| **Autonomous Agents** | **3** (buyer, seller, advertiser) |
| **ML Model Specs** | **15** |
| **External Price Sources** | **5** (Avito, Jumia, Amazon, eBay, Google Shopping) |
| **Training Data Categories** | **9** (classification, generation, search, negotiation, price, embeddings, darija, chat, bedrock) |
| **MCP Tools** | **22** (16 existing + 6 new for sell-in-ChatGPT) |
| **BO Dashboard Panels** | **10** |
| **External API Integrations** | **22+** (adding Apify for price scraping) |

---

## THE ONE-LINE SUMMARY

**Tawadoo V2 = an autonomous marketplace where AI agents buy, sell, negotiate, advertise, and learn — across web, ChatGPT, and WhatsApp simultaneously — while a training pipeline shadows every interaction to build proprietary models that no competitor can replicate.**

---

*Document complete. This is the vision Ramzi has been waiting years for. It's specific enough to build, ambitious enough to fund, and grounded in infrastructure that already exists. The path starts with Session 14A (65 P0 events). Everything else builds from there.*
