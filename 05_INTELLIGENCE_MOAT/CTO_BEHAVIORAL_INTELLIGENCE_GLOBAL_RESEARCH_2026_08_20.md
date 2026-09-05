# GLOBAL BEHAVIORAL INTELLIGENCE RESEARCH — STATE OF THE ART (Aug 20, 2026)

> **Purpose:** What the world's best marketplaces track to understand users better than anyone. Every signal documented here informs Tawadoo's proprietary behavioral capture layer. This research feeds Session 14E (extended behavioral intelligence).
> **Sources:** Amazon, Alibaba/Taobao, Meesho (India), Pinduoduo/Temu (China), Dubizzle/OLX (MENA), Flipkart (India), TikTok Shop, Rakuten (Japan), academic research (NIH, Nature, Springer, arXiv), industry analysis.
> **Saved for:** Next prompt execution. LAW 16 applies — all captured to YOUR DB first.

---

## 1. THE SCALE BENCHMARKS (what the leaders process)

| Platform | Users | Signals/Day | What they track |
|---|---|---|---|
| **Amazon** | 310M+ active accounts | Billions | Cursor movement, hover duration, scroll depth, image zoom time, video rewind points, cart abandonment triggers, review reading depth, Q&A engagement, price check frequency, cross-category sequences, tab switching, comparison shopping patterns |
| **Alibaba/Taobao** | 1B+ users | Hundreds of billions | Full user behavior SEQUENCES (not individual events). Behavior Sequence Transformer (BST) models capture temporal ordering. Billion-scale item graph from browsing history. 22-day behavioral windows for intent modeling. |
| **Meesho** | 264M transacting | 6 BILLION/day | PRISM engine (Personalised Ranking & Intent Signal Module). Every product impression with viewport position. Scroll direction AND speed. Time between taps. Share-before-purchase patterns. 73% of orders come from RECOMMENDATIONS not search. |
| **Pinduoduo/Temu** | 900M+ (Pinduoduo) | Not disclosed (massive) | Social shopping signals: who shared what to whom, group buying formation patterns, countdown urgency responses, gamification engagement (lucky draws, team discounts), live stream shopping behavior. AI personalizes at scale. |
| **Dubizzle Group** | 240M+ visitors/month | 3 BILLION events/month | Petabyte-scale warehouse with 135 billion records. Custom single tech stack across 8 MENA countries. 660K verified users. Scam detection from behavioral patterns (cut 27% YoY). |
| **Flipkart** | 500M+ registered | Not disclosed | FCC Recommendations infers user intent from behavioral signals + catalog data. Conversational AI replacing traditional search. Hyper-personalization beyond simple recommendations. |
| **TikTok Shop** | 150M+ buyers | Not disclosed | Average watch time, saves, comments, shares, video completion rate, pinned product CTR, viewer-to-buyer conversion, live stream engagement duration, replay patterns. $112B GMV projected 2026. |

---

## 2. WHAT AMAZON TRACKS THAT WE DON'T (deep signals)

From research (Content was rephrased for compliance with licensing restrictions):

### A. Browsing Telemetry (the "body language")
- **Cursor hover patterns** — which elements the mouse pauses over (price, image, reviews, buy button)
- **Hover duration on images** — longer hover = more interest in that specific product angle
- **Scroll velocity mapping** — fast scroll = browsing. Deliberate slow scroll = reading. Full stop = considering
- **Image zoom timing per photo** — WHICH photo gets zoomed and for how long (reveals what feature matters)
- **Video segment engagement** — not just 25/50/75% but which SECOND they pause, rewind, or skip (reveals feature interest)
- **Tab switching patterns** — opening 5 products in new tabs = comparison shopping mode (high intent, undecided)
- **Cart composition analysis** — what combinations of items go into cart together (reveals use case: "furnishing apartment" = couch + table + lamp)
- **Review engagement depth** — did they read all reviews? Only negative? Only verified? Which reviews they expand (reveals concern areas)
- **Q&A section engagement** — reading questions = uncertain buyer. ASKING question = very high intent but blocked by info gap
- **Price check frequency** — same item visited 3+ times across days = waiting for price drop (trigger: price alert)
- **Cross-category browsing SEQUENCES** — bought house → browsing furniture → PREDICT: appliances, paint, garden tools coming next
- **Search refinement patterns** — "laptop" → "laptop 16gb ram" → "macbook pro m3" = narrowing intent (ready to buy specific item)
- **Wishlist-to-purchase timing** — how long items sit in wishlist before purchase (or abandonment) = urgency measure

### B. Intent Classification (what Amazon infers)
- **High-intent buyer signals:** Multiple visits to same item, zooming photos, reading reviews, checking delivery date, adding to cart
- **Comparison shopper signals:** Opening multiple similar items, sorting by price, filtering by features, rapid switching between pages
- **Window shopper signals:** Fast scrolling, low dwell, no zoom, no review reading, quick page exits
- **Price-sensitive signals:** Always sorting low-to-high, using deal filters, checking "other sellers", waiting for sales
- **Quality-focused signals:** Sorting by rating, reading top reviews, checking brand, viewing certifications

---

## 3. WHAT ALIBABA/TAOBAO TRACKS (behavior SEQUENCES, not individual events)

From Alibaba's research papers (arXiv: 1803.02349, 1905.06874):

### The Key Insight: SEQUENCES matter more than individual events

Taobao doesn't just track "user viewed item X". They track the SEQUENCE:
```
User viewed A → viewed B → viewed C → bought A → viewed D → bought D
```

This sequence tells you:
- A and D are similar (bought both)
- B and C were considered but rejected (why? price? condition? seller?)
- The TIME between viewing and buying A = decision speed (impulse vs research)
- If many users follow A→D, then D should be recommended after A purchase

### Behavior Sequence Transformer (BST)
- Captures the **temporal ordering** of user actions using Transformer architecture
- Each action is embedded as a vector (like words in a sentence)
- The MODEL learns that "view phone case AFTER buying phone" is a signal, not just "viewed phone case"
- Deployed at billion scale with real-time inference

### Item Graph from Behavior History
- Every user's browsing history creates EDGES between items
- If 1000 users view both item A and item B in the same session → A and B are connected
- Graph embedding produces item vectors that encode "relatedness" from BEHAVIOR (not just category)
- Cold-start items get vectors from their neighbors in the graph

### What Tawadoo should replicate:
- **Session-level behavior sequences** — not just individual events, but the ORDER
- **Item-to-item graph from co-viewing** — "users who viewed this also viewed..."
- **Temporal intent modeling** — "this user has been researching for 5 days → ready to buy"
- **Cross-category sequence prediction** — "bought car → will need insurance/accessories"

---

## 4. WHAT MEESHO TRACKS (6 BILLION signals/day — India)

From Meesho engineering blog + Economic Times (Content was rephrased for compliance with licensing restrictions):

### PRISM: Personalised Ranking & Intent Signal Module
- Processes 6 billion behavioral signals DAILY
- 73% of orders do NOT begin with a search — they come from RECOMMENDATIONS
- This means: Meesho's behavioral understanding is SO good that they can PREDICT what you want before you search

### Signals that drive PRISM:
- **Product impression with viewport position** — not just "was visible" but WHERE on screen (top = more valuable attention)
- **Scroll direction AND velocity** — upward scroll = re-checking something. Rapid downward = skipping. Slow = reading.
- **Time between taps** — long pause between seeing and tapping = hesitation (price concern? trust issue?)
- **Share-before-purchase** — Indian users often share products with family/friends before buying (social validation seeking)
- **Return patterns** — same user coming back to same product over multiple days (building conviction)
- **Session time-of-day** — morning = casual discovery. Evening = serious shopping. Late night = impulse
- **Seasonal patterns** — Diwali, Eid, back-to-school, wedding season → category demand shifts

### Key takeaway for Tawadoo:
If Meesho drives 73% of orders from recommendations (not search), then behavioral understanding IS the product. The better you understand users, the less they need to search — you just SHOW them what they want.

---

## 5. WHAT PINDUODUO/TEMU TRACKS (social + gamification signals — China)

From ChoZan 2026 report + Columbia + ResearchGate (Content was rephrased for compliance with licensing restrictions):

### Social Commerce Signals (unique to their model):
- **Group formation patterns** — who invites whom, acceptance rate, deal-sealing speed
- **Countdown response behavior** — does urgency ("2 hours left") increase conversion? For which users?
- **Lucky draw participation** — engagement frequency predicts purchase frequency
- **Team discount sharing patterns** — viral coefficient per user (some are super-spreaders)
- **Live stream shopping** — join time, watch duration, comment frequency, product pin clicks, purchase during stream
- **Gamification engagement** — daily check-ins, task completion, reward accumulation patterns

### What this means for Tawadoo:
- Lucky wheel engagement → predicts overall platform stickiness
- Share patterns (WhatsApp shares of listings) → viral coefficient measurable
- Auction countdown behavior → "snipers" vs "early bidders" = personality typing
- Referral chain depth → who are your organic growth engines?

---

## 6. WHAT DUBIZZLE GROUP TRACKS (MENA classifieds — our direct competitor class)

From dubizzlegroup.com + Imperva case study (Content was rephrased for compliance with licensing restrictions):

### Their scale:
- **135 BILLION records** in their data warehouse
- **3 BILLION events/month** collected
- **Petabyte-scale** infrastructure
- Single tech stack across 8 MENA countries
- 660K verified users (UAE alone)

### What they track for fraud detection (AI-driven, 27% reduction):
- Behavioral anomaly patterns (posting speed, image reuse, contact patterns)
- Device fingerprinting across accounts
- Price outlier detection (listings priced suspiciously below market)
- Communication pattern analysis (scammer script detection)
- Listing similarity clustering (same photos/text across different accounts)

### Key insight for Tawadoo:
Dubizzle processes 3B events/month with 240M visitors. We have 60K users. If we capture RICH signals now (while small), we'll have the DENSEST behavioral understanding per-user of any MENA marketplace when we scale. Quality over quantity at our stage.

---

## 7. WHAT FLIPKART TRACKS (intent-led commerce — India)

From Flipkart Commerce Cloud + stories.flipkart.com (Content was rephrased for compliance with licensing restrictions):

### FCC Recommendations Engine:
- Infers user intent from behavioral signals + catalog data
- Surfaces most relevant products (or stores/categories) for given user in given context
- Beyond simple "similar items" — understands INTENT

### The shift to conversational commerce:
- Traditional search being REPLACED by AI conversational interface
- AI understands broader customer intent (not just keywords)
- "I need something for my daughter's birthday under 2000 rupees" → AI finds it

### What Tawadoo should learn:
- Conversational intent > keyword search (our ChatGPT MCP already does this!)
- Context matters: SAME user searching "phone" on a Saturday afternoon vs Tuesday morning = different intent
- Combine behavioral signals WITH catalog data for relevance (don't just match keywords, understand WHY they're searching)

---

## 8. WHAT TIKTOK SHOP TRACKS (content + commerce fusion)

From TikTok Ads documentation + industry analysis (Content was rephrased for compliance with licensing restrictions):

### Commerce signals from content engagement:
- **Average watch time** — longer watch = higher product interest
- **Saves** — save-to-purchase conversion is the strongest predictor
- **Comments asking price/availability** — purchase intent signal
- **Share to messaging** — social validation (like Meesho's share-before-buy)
- **Video completion rate** — completes = high engagement with the product
- **Pinned product CTR** — clicking the product pin = explicit purchase intent
- **Replay patterns** — rewatching = considering the product seriously
- **Live stream engagement** — join duration, comment frequency, purchase during stream

### Predictive signals for GMV (from their research):
- Videos with 3+ predictive signals generate 7.2x more attributed GMV
- Key signals: saves > watch time > shares > comments
- Seller fulfillment speed affects future algorithmic distribution

### What Tawadoo should learn:
- Video engagement depth (we have listing videos + store videos) is a PURCHASE signal
- Live auction viewing = same as TikTok live stream (high intent while watching)
- Store video watch time → store follow → listing view → purchase (the content-to-commerce funnel)

---

## 9. WHAT RAKUTEN TRACKS (Japan — loyalty + ecosystem)

From Rakuten Insight + marketing platform (Content was rephrased for compliance with licensing restrictions):

### Cross-ecosystem behavioral mapping:
- Rakuten tracks behavior across: marketplace + banking + mobile + travel + insurance + sports
- The SAME user's behavior across services reveals life-stage and purchasing power
- Points accumulation across services → predicts lifetime value

### What Tawadoo can replicate:
- Coin economy behavior ACROSS features (bought coins for boost → also bid in auctions → also bought subscription) reveals user value tier
- Cross-category browsing within our ecosystem → life-stage prediction
- WhatsApp engagement + web behavior + notification response = multi-channel behavioral profile

---

## 10. THE SIGNALS WE'RE MISSING (complete expanded list for Tawadoo)

Based on all research above, organized by CAPTURE DIFFICULTY:

### EASY TO CAPTURE (frontend events — implement immediately):

| # | Signal | What it predicts | How to capture |
|---|---|---|---|
| 1 | Listing photo zoom per-image | Which product feature matters most to this buyer | touch/click event with image_index + duration |
| 2 | Scroll velocity on listing page | Casual browser vs serious reader | scroll event with velocity calculation |
| 3 | Time between search and first click | Result relevance (fast click = relevant) | timestamp diff between search_performed and search_result_clicked |
| 4 | Multiple views of same listing across days | High intent, building conviction | session_id + listing_id + visit_count over time |
| 5 | Tab/window switching during browse | Comparison shopping mode | visibilitychange + focus events |
| 6 | Share-before-purchase pattern | Social validation seeker (common in Morocco) | track share → track if same user later buys |
| 7 | Price filter tightening sequence | Budget crystallizing (broad→narrow = deciding) | filter_applied sequence with price ranges |
| 8 | Listing revisits within session | Undecided between options (show comparison) | same listing_viewed within same session |
| 9 | Cart/offer abandonment point | WHERE in the flow they drop (price? delivery? trust?) | step tracking in checkout/offer flows |
| 10 | Video rewind to specific moments | Which product feature they want to re-examine | timeupdate events with direction detection |
| 11 | Time spent reading seller profile | Trust evaluation in progress | dwell_time on store page |
| 12 | Review section engagement | Concerns about quality/trust | scroll_to_reviews + time_in_section |
| 13 | Notification-to-action speed | Urgency level per notification type | notification_opened → next_action timestamp |
| 14 | Session frequency acceleration/deceleration | Engagement growing (opportunity) vs declining (churn risk) | session_started frequency over time |
| 15 | Search refinement patterns | Intent narrowing ("voiture" → "voiture occasion casa" → "golf 7 diesel") | search query sequence within session |
| 16 | Feed swipe speed per listing | Quick swipe = not interested. Slow/stop = considering | swipe velocity in ImmersiveFeed |
| 17 | Camera permission grant context | What are they trying to do? (image search? listing photos?) | permission_granted with context |
| 18 | Language switch patterns | Bilingual user → serve content in preferred language per context | language_switched + page_type |
| 19 | Auction last-minute viewing | "Sniper" personality → different monetization approach | auction_viewed with time_remaining < 60s |
| 20 | Offer amount relative to asking | Aggressive vs polite negotiator → personality typing | offer_submitted.discount_pct histogram |

### MEDIUM DIFFICULTY (backend computation — needs data correlation):

| # | Signal | What it predicts | How to compute |
|---|---|---|---|
| 21 | Cross-category purchase sequences | "Bought house → needs furniture" life-stage prediction | Order/offer history correlation by user + category |
| 22 | Seller response time → deal probability | Fast-responding sellers close more deals | message timestamp analysis per thread |
| 23 | Price-to-market positioning | Is this listing priced competitively? (affects buyer behavior) | Compare listing price vs category/city average |
| 24 | Listing quality → lead velocity | Better photos/descriptions get more leads faster | Correlate listing features with lead count/speed |
| 25 | Seasonal demand patterns | Ramadan/Eid/school/rent patterns by city | Historical order/search data by date + category |
| 26 | User lifetime behavior sequence | LSTM/Transformer on full action history → predict next action | Full event sequence per user → model |
| 27 | Item-to-item co-viewing graph | "Users who viewed this also viewed..." from BEHAVIOR | Session co-occurrence matrix → graph embedding |
| 28 | Negotiation success prediction | Will this offer be accepted? (price × seller history × category) | Historical offer acceptance rates |
| 29 | Churn timing prediction | WHEN will this user stop coming? (not just IF) | Session frequency decay analysis |
| 30 | Payment method → conversion rate | Card users vs cash users → different abandonment patterns | Payment method × completion rate correlation |

### ADVANCED (ML models needed — future sessions):

| # | Signal | What it predicts | Model type |
|---|---|---|---|
| 31 | User embeddings from behavior | User similarity for collaborative filtering recommendations | User2Vec from action sequences |
| 32 | Listing embeddings from engagement | Which listings are "similar" in terms of who engages with them | Item2Vec from co-viewing graph |
| 33 | Intent classification per session | browsing/researching/ready-to-buy/window-shopping | Multi-class classifier on session features |
| 34 | Next-purchase category prediction | What category will this user buy from next? | Sequence prediction model (LSTM/Transformer) |
| 35 | Optimal notification timing per user | When to send for maximum engagement | Per-user time-series analysis |
| 36 | Price sensitivity per user | How much does price affect THIS user's decisions? | Elasticity model from filter usage + offer patterns |
| 37 | Fraud probability scoring | Is this listing/user suspicious? | Anomaly detection on behavioral features |
| 38 | Listing time-to-sale prediction | How long until this listing sells? | Survival analysis on listing features |
| 39 | Seller agent decision model | When to auto-accept offers, when to counter | RL from seller override patterns (RLHF) |
| 40 | Dynamic pricing suggestions | Optimal price for this item in this city this week | Demand forecasting × price elasticity model |

---

## 11. THE CONVERSATION INTELLIGENCE LAYER (unique to marketplaces with messaging)

Amazon doesn't have buyer-seller messaging. WE DO. This is a MASSIVE data advantage:

| Signal from Messages | What it tells you | Training use |
|---|---|---|
| "disponible?" (available?) | Ready to transact if yes | Intent classifier: HIGH |
| "dernier prix?" (final price?) | Negotiating — will buy if price is right | Intent + price sensitivity |
| "livraison possible?" (delivery?) | Logistics is the blocker, not intent | Remove friction: show delivery options |
| "échange possible?" (trade?) | Wants barter — different transaction model | Transaction type classification |
| Response time (seller) | Fast = good experience. Slow = lost sale. | Seller quality score component |
| Message length (buyer) | Long = serious buyer with specific needs | Intent depth classification |
| Multiple messages without response | Seller ghosting → buyer frustration → churn risk | Seller intervention trigger |
| Message after hours (late night) | Impulse interest — might not convert if delayed | Optimal timing for seller notification |
| Voice note sent | High engagement (Morocco WhatsApp culture) | Preference for voice/audio content |
| Photo shared in chat | Showing specific needs/concerns | Visual intent signal |
| "Wallah" / "inchallah" usage | Cultural commitment signals in Darija | NLP for Darija intent analysis |

---

## 12. THE LIFE-STAGE PREDICTION MODEL (cross-category intelligence)

This is what makes a marketplace IRREPLACEABLE. Single-category stores can't do this:

| Purchase/Browse Pattern | Life Stage Inference | Next-Purchase Predictions |
|---|---|---|
| Browsing real estate (sale) in new city | Relocating | Furniture, appliances, movers, renovation services, school (if family) |
| Selling baby items | Child growing up | Kids bikes, school supplies, bigger clothes, toys for older age |
| Buying used car | New driver OR car replacement | Insurance (lead), accessories, mechanic services, car wash |
| Browsing wedding fashion | Getting married | Home furnishing, kitchen, electronics (gifts), honeymoon-related |
| Selling electronics (multiple) | Financial pressure OR upgrading | Offer coin advance, cheaper alternatives, payment plans |
| Browsing rental apartments | Moving soon | Furniture, appliances, internet service, moving help |
| First listing ever posted | New seller | Onboarding tips, photography advice, pricing guidance |
| Suddenly browsing jobs category | Career change | Professional services, courses, work equipment |
| Buying gaming console | Gamer | Games, accessories, gaming chairs, second screen |
| Selling multiple items same category | Closing business OR decluttering | Storage, moving services, bulk listing tools |

**The model:** User's FULL history (what they browse, buy, sell, search, favorite) → sequence model → predict next category + optimal timing for recommendation.

---

## 13. TEMPORAL SIGNALS (when things happen matters as much as what)

| Signal | Insight | Application |
|---|---|---|
| Morning browse (6-9 AM) | Commute browsing — low intent, high discovery | Show trending, inspiration, new arrivals |
| Lunch browse (12-14 PM) | Quick break — moderate intent, impatient | Show deals, quick-buy items |
| Evening browse (18-22 PM) | Serious shopping — highest intent | Show recommendations, complete-your-purchase nudges |
| Late night (23-02 AM) | Impulse + insomnia — emotional decisions | Show aspirational items, auctions ending soon |
| Weekend vs weekday | Weekend = leisure shopping. Weekday = need-based | Adjust recommendation algorithms by day |
| Payday (25th-5th) | Purchase spike in Morocco (salary cycle) | Time promotions, boost suggestions, coin pack offers |
| Ramadan patterns | Food, home, gifts spike. Vehicles/RE slow. | Category-level demand adjustment |
| School season (Sept) | Supplies, electronics, rentals near schools | Programmatic category emphasis |
| First visit after long absence | Reactivation moment — show what's new | "Since you've been away: X new in your categories" |

---

## 14. SUMMARY: TOTAL BEHAVIORAL SIGNALS FOR TAWADOO

| Layer | Signals Count | Currently Tracked | Gap |
|---|---|---|---|
| Basic interactions (clicks, views, purchases) | 65 (P0) | 65 ✅ | 0 |
| Behavioral depth (scroll, dwell, impressions, abandonment) | 85 (P1 — in flight) | 0 (session running) | 85 |
| Micro-behaviors (cursor, velocity, hesitation, comparison) | 40 | 0 | 40 |
| Lifecycle predictions (cross-category, life-stage) | 15 | 0 | 15 |
| Conversation intelligence (message patterns, response times) | 12 | 0 | 12 |
| Temporal/environmental (time-of-day, seasonal, payday) | 10 | 0 | 10 |
| Social/network (sharing, household, referral, competitive) | 8 | 0 | 8 |
| Sequence-level (behavior chains, co-viewing graph) | 10 | 0 | 10 |
| **TOTAL** | **~245 unique signals** | **65** | **~180 to add** |

Adding the 85 P1 events (in flight) → 150 captured. Still **~95 signals missing** that the world leaders track.

---

## 15. THE TAWADOO ADVANTAGE (what we have that they DON'T at our stage)

1. **Buyer-seller messaging** — Amazon doesn't have this. We can analyze CONVERSATIONS for intent.
2. **Multi-transaction models** — Buy now + offers + auctions + leads in ONE platform = richer behavioral variety per user
3. **Trilingual + Darija** — Language switching patterns + Darija NLP = cultural intelligence nobody else has
4. **Live auctions** — Real-time bidding behavior = urgency + psychology signals (snipers, early birds, auto-bidders)
5. **Cross-category in ONE marketplace** — Cars + real estate + electronics + fashion = life-stage prediction possible
6. **WhatsApp-native** — Voice notes, photos in chat, "wallah" commitment signals = Moroccan behavioral vocabulary
7. **ChatGPT MCP** — Conversational commerce queries = intent in natural language (no other African marketplace has this)
8. **Small user base NOW** — Can instrument DEEPLY without performance concerns. By the time we scale, the models are trained.

---

*This research is COMPLETE. Save for Session 14E+ prompt. The behavioral intelligence layer should grow from 85 events to 245+ signals. This is what builds the AI moat that makes Tawadoo irreplaceable.*

*Sources: Amazon recommendation papers, Alibaba arXiv (1803.02349, 1905.06874), Meesho engineering blog + Economic Times India, Pinduoduo/ChoZan 2026 report, Dubizzle Group corporate, Flipkart Commerce Cloud docs, TikTok Ads documentation, Rakuten Insight, academic papers (NIH, Nature, Springer, Taylor & Francis), industry analysis (Kissmetrics, Shopify, HubSpot).*

*Last updated: Aug 20, 2026. Author: CTO Pilot (Kiro Brain 6).*


---

## 16. VERTICAL-SPECIFIC INTELLIGENCE (per Tawadoo category)

### REAL ESTATE (Zillow, Bayut, PropertyGuru)

**Zillow's "AI-Driven User Memory" (2026 — this is THE state of the art):**

Zillow built a system they call "persistent context across sessions" — it REMEMBERS:
- Where the user is in their housing JOURNEY (browsing, researching, ready to tour, ready to buy)
- What they've explored across MULTIPLE sessions (not just this one)
- What they can AFFORD (inferred from viewing patterns + saved search price ranges)
- WHEN they plan to act (urgency signals from behavior frequency)

Their "high intent buyer" model: users flagged as high-intent are **4.2x more likely to transact in the next 3 months**. Signals: time spent viewing homes, number of homes favorited/shared, agent profile views.

**Zillow's "Next Best Action" platform:** Uses contextual bandits (not just rules) to decide what to show each user next. The system learns IN REAL TIME which action leads to better outcomes for EACH user type.

**Bayut (UAE — our MENA competitor):**
- BayutGPT: world's first AI property search assistant
- TruCheck: verified listing badge (time-stamped authentication)
- Priority ranking for verified listings

**What Tawadoo should capture for Real Estate:**
- `property_visit_frequency` — same property viewed 3+ times = serious buyer
- `neighborhood_research_depth` — viewing multiple properties in same area = area decision made
- `affordability_signals` — price filter patterns over time reveal budget progression
- `journey_stage` — first visit vs returning vs ready-to-tour vs ready-to-offer
- `mortgage_calculator_usage` — indicates financial planning stage
- `agent_profile_views` — researching the seller = trust evaluation
- `property_comparison_behavior` — which features they compare (rooms? price? location? garden?)
- `floor_plan_engagement` — time on floor plan = serious spatial evaluation
- `virtual_tour_completion` — finished virtual tour = HIGH intent (hard to achieve with low intent)

---

### VEHICLES (Carvana, AutoTrader, Carsome)

**Carvana ($18B revenue):** AI + logistics transformed car buying. Generates real car values from: vehicle features, accident history, mileage, service records, market conditions.

**AutoTrader 2026 research:** 83% of consumers believe AI will reshape car buying. 30% already use AI to compare options and validate pricing BEFORE contacting dealers.

**Oliver Wyman 2026:** 62% of consumers used AI to find and purchase products. "AI commerce wins every time" when it validates pricing, compares options, builds confidence.

**What Tawadoo should capture for Vehicles:**
- `vehicle_history_check_requested` — indicates serious buyer who does due diligence
- `price_vs_market_comparison` — user checking if price is fair (show price intelligence)
- `multiple_similar_models_viewed` — comparison shopping between models (Golf vs Polo vs Leon)
- `brand_loyalty_signal` — same user always views one brand = send new listings of that brand
- `mileage_sensitivity` — filters by mileage tightly = maintenance-conscious buyer
- `year_range_narrowing` — narrowing year filter = budget crystallizing
- `test_drive_request` — if we add this feature, it's the HIGHEST intent signal
- `financing_interest_signal` — if user views payment options / instalment info
- `trade_in_inquiry` — "échange possible?" in messages = lifecycle signal (replacing old car)

---

### FASHION (SHEIN, Vinted)

**SHEIN's proprietary model:** Measures customer preferences in real-time, predicts demand using ML on browsing + purchasing histories. Tracks MICROTRENDS across social media before they peak.

**Vinted's recommendation engine:** Uses Vespa search engine with explicit preferences (sizes, styles user inputs) + implicit preferences (clicks, purchases, browsing patterns). ML models learn from historical interactions.

**EDHEC research (2026):** "By analyzing user behavior data, AI tools identify customers most likely to make a purchase and tailor advertising messages they receive."

**What Tawadoo should capture for Fashion:**
- `size_preference_implicit` — which sizes they repeatedly view/filter (even without explicit setting)
- `style_cluster_behavior` — viewing similar aesthetic items reveals style preference
- `brand_affinity_pattern` — always views Zara/H&M → recommend similar brands
- `seasonal_buying_pattern` — buys coats in October, swimwear in June (predict and pre-show)
- `condition_preference` — some users ONLY view "new with tags", others prefer "good condition" (= price sensitivity vs quality priority)
- `photo_engagement_by_type` — which listing photos they zoom (worn photo vs flat lay vs detail shot)
- `return_to_saved_timing` — fashion items saved but not bought for 3 days = send "still available" notification

---

### AUCTIONS (eBay, Catawiki)

**eBay auction research (Stanford, Berkeley, Yale, Penn):**
- Snipers (last-minute bidders) are MORE LIKELY to win and pay LESS
- Experienced bidders are more likely to snipe
- Bidders who get sniped are **4-18% less likely to return to the platform** (CRITICAL retention insight)
- Bid activity pattern: few bids at start, relatively few in middle, LARGE SPIKE in final minutes
- Incremental bidders (bid small amounts repeatedly) are VULNERABLE to snipers

**What Tawadoo should capture for Auctions:**
- `bid_timing_personality` — classify as: early_bird, mid_game, sniper (last 60s), auto_bidder
- `snipe_victim_detection` — user got outbid in last 30s → HIGH churn risk → intervene (suggest auto-bid next time)
- `bid_increment_pattern` — small increments = incremental bidder (vulnerable). Large jumps = aggressive (committed)
- `auction_watching_without_bidding` — stalking behavior = interested but not confident enough to bid
- `auto_bid_ceiling_vs_actual_win` — did they set max way above what was needed? (= would have paid more)
- `post_auction_behavior` — won: does buyer complete order? Lost: does user search for similar item immediately?
- `auction_return_rate` — users who participate in auctions → do they come back for more? (addictive loop detection)
- `bid_war_engagement` — rapid back-and-forth bidding = emotional engagement (dopamine, competitive, will pay premium)

---

### JOBS & SERVICES (LinkedIn, Indeed, Fiverr)

**LinkedIn 2025 report:** 89% of talent leaders say measuring quality of hire is critical but only 25% can do it. Hiring signals = open roles, velocity, department growth.

**Indeed analytics:** Turns activity from millions of job seekers into on-demand reports. Tracks: application velocity, resume keyword matching, employer response rates.

**Fiverr:** Trained on "billions of interactions" for matching. Spend per buyer $342 (up 13.3%), driven by complex AI projects.

**What Tawadoo should capture for Jobs:**
- `job_listing_save_without_apply` — interested but not ready (missing requirement? salary?)
- `multiple_similar_jobs_viewed` — comparison shopping for opportunities
- `skills_match_signal` — user profile skills vs job requirements (can we infer match quality?)
- `employer_response_time_to_application` — fast response = engaged employer (good experience)
- `job_category_switch` — user browsing dev jobs then switches to marketing = career pivot
- `service_request_specificity` — vague request = early stage. Detailed brief = ready to hire
- `freelancer_portfolio_engagement` — time on portfolio = evaluating quality
- `repeat_hiring_pattern` — same client hiring same category again = ongoing need (upsell subscription)

---

### ELECTRONICS (Back Market)

**Back Market's trust architecture:** Quality grading (Grade A/B/C) + warranty + verified reviews. Research shows "confusion tax" — subjective labels erode value.

**Consumer research (2026):** 80% prefer higher-quality refurbished over lower-quality new at same price. Key driver: trust signals (warranty, reviews, certification).

**What Tawadoo should capture for Electronics:**
- `condition_grade_filter_pattern` — which conditions they accept reveals price/quality tradeoff preference
- `warranty_section_viewed` — looking at warranty = trust evaluation in progress
- `specification_comparison` — comparing RAM/storage/screen → technical buyer (show specs prominently)
- `brand_vs_price_preference` — always filters by brand first OR by price first (= loyalty vs sensitivity)
- `previous_model_research` — viewing both iPhone 13 and 14 = deciding which generation (price step)
- `accessory_viewing_after_device` — viewed phone → now viewing cases = purchase cycle intelligence
- `compatibility_check` — views specs to check compatibility with existing devices

---

## 17. THE INNOVATIONS NOBODY ELSE DOES (Tawadoo-specific signals that exploit our unique position)

These are signals that NO other marketplace captures because they don't have our combination of features:

### A. AUCTION × MESSAGING CROSSOVER
When a user who LOST an auction then messages the seller → they're trying to buy directly (circumventing the system). Track: `post_auction_direct_contact` — signals we need better "second chance" offers.

### B. LISTING-TO-LISTING SELLER MIGRATION
When users who viewed Seller A's listing then view Seller B's similar listing → competitive intelligence signal. Track: `seller_switching_behavior` — helps sellers understand their competition.

### C. DARIJA INTENT CLASSIFICATION (unique globally)
Moroccan users express intent differently in Darija vs French. "Bghit" (I want) is stronger intent than "Je cherche" (I'm looking). Track: `query_language_intent_strength` — Darija queries may have higher conversion rates.

### D. WHATSAPP-FIRST DEAL SIGNAL
Users who take conversations from in-app chat to WhatsApp are signaling: deal is serious enough to use their real number. Track: `chat_to_whatsapp_migration` — this is a STRONGER intent signal than continued in-app messaging.

### E. PRAYER TIME BEHAVIOR GAPS
Morocco has 5 daily prayer times. Users who consistently have activity gaps during prayer times → devout users (different product preferences, different promotional calendar). Track: `session_gap_prayer_correlation` — for Ramadan/Eid targeting precision.

### F. FAMILY DEVICE SHARING
In Morocco, families often share devices. Multiple distinct browsing patterns on same device_id → multiple users. Track: `multi-profile_device_detection` — affects recommendation accuracy (don't recommend men's items based on wife's browsing).

### G. SEASONAL MIGRATION PATTERNS (MRE — Moroccans Abroad)
Moroccans living in Europe return in summer (July-August). Their browsing patterns change: from browsing in Europe (FR/NL/BE) to buying in Morocco (Casablanca/Tangier). Track: `mre_return_signal` — high-spending users arriving seasonally (target with premium items, real estate, vehicles).

### H. CROSS-PLATFORM ATTRIBUTION (ChatGPT → Web)
User searches on ChatGPT via our MCP → visits tawadoo.ma → purchases. The MCP query CONTENT + the eventual purchase = the strongest intent-to-action signal. Track: `mcp_query_to_purchase_attribution` — which ChatGPT queries lead to sales (feeds search relevance model).

### I. AUCTION EMOTION MAP
During live auctions, the PACE of bidding reveals emotional state. Rapid successive bids = emotional/competitive. Long pauses between bids = rational/strategic. Track: `bid_velocity_emotional_state` — use to optimize auction UX (show different messaging to emotional vs rational bidders).

### J. SELL-TO-BUY LIFECYCLE
User sells items in one category, then immediately browses another. "Sold old phone → browsing new phones" = trade-up cycle. Track: `sell_to_buy_category_transition` — predict and recommend the UPGRADE they'll want.

---

## 18. OPEN-SOURCE TOOLS TO CAPTURE THIS (for the proprietary event store)

| Tool | What it does | License | Our use |
|---|---|---|---|
| **Microsoft Clarity** (open-source lib) | Behavioral analytics: DOM mutations + user interactions → aggregated insights. Privacy & performance focused. | MIT | Cursor/scroll/rage-click capture — run self-hosted, data stays OURS |
| **PostHog** (self-hostable) | Full product analytics + session replay + feature flags + A/B testing | MIT (Free tier: 1M events/month) | Could REPLACE Amplitude if needed. Self-hosted = full sovereignty. |
| **Seentics** (new, open-source) | Real-time dashboards, session replays, heatmaps, funnels, behavioral automations | Open-source | Alternative analytics platform, fully self-hosted |
| **tracelog-lib** (GitHub) | Lightweight TypeScript: auto-captures clicks, scrolls, page views, performance metrics | MIT | Lightweight client-side capture, send to OUR API |
| **Jitsu** (open-source CDP) | Event collection from every source → YOUR data warehouse. Self-hosted. MIT. | MIT | Replaces Segment. Collects events → sends to our PostgreSQL/S3 |

**The pattern:** Use open-source capture libraries on the frontend. Send all events to YOUR API first. Then mirror to Amplitude for their ML/visualization. You own the raw data. They provide the compute.

---

*End of vertical-specific research. Total new signals identified: ~65 per-vertical + 10 Tawadoo-unique innovations = ~75 additional signals beyond the general behavioral layer. Combined with Section 10's 95 signals → total gap is ~170 signals to reach world-class behavioral intelligence.*

*This feeds directly into Session 14E prompt specification.*


---

## 19. USER ORIGIN, ATTRIBUTION & LIFESTYLE PROFILING (the biggest gap)

### THE PROBLEM: We throw away acquisition intelligence

**What we HAVE on ta_order and ta_lead:** utm_source, utm_medium, utm_campaign, utm_content, fbclid, ttclid
**What we DON'T HAVE on ta_user (signup):** NOTHING. Zero acquisition data.
**What we DON'T HAVE on ta_offer, ta_bid_entity, ta_syndication_user_subscription, ta_coin_package_order, ta_publication:** NOTHING.

**This means:** We can attribute ORDERS to channels, but we can't attribute SIGNUPS, FIRST LISTINGS, SUBSCRIPTIONS, BOOSTS, or COIN PURCHASES to channels. The entire LTV-per-channel calculation is impossible.

### WHAT UTM GIVES US (if we capture it everywhere)

| UTM Parameter | What it tells us | Example values |
|---|---|---|
| `utm_source` | Which PLATFORM sent them | google, meta, tiktok, chatgpt, whatsapp, bing, organic, direct, referral |
| `utm_medium` | What TYPE of traffic | cpc (paid), social (organic social), organic (SEO), email, referral, mcp |
| `utm_campaign` | Which specific CAMPAIGN | ramadan_2026, summer_mre, launch_tiktok_q3 |
| `utm_content` | Which specific AD/CREATIVE | video_casa_phones, carousel_re_rabat |
| `utm_term` | Which KEYWORD (paid search) | "voiture occasion casablanca", "iphone maroc" |
| `fbclid` | Facebook click ID | Links click to specific Meta ad (for CAPI matching) |
| `ttclid` | TikTok click ID | Links click to specific TikTok ad |
| `gclid` | Google click ID | Links click to specific Google ad |

### WHAT SSO GIVES US (that we DON'T store)

**Google OAuth payload:**
- `email` (verified) — ✅ we store this
- `name` (first + last) — ✅ we likely store this
- `picture` (Google profile photo URL) — ❌ NOT stored (could auto-fill store avatar)
- `locale` (e.g., "fr", "ar", "nl") — ❌ NOT stored (their REAL language preference from Google account)
- `sub` (permanent Google user ID) — ❌ NOT stored (useful for cross-device identity matching)

**Facebook OAuth payload:**
- `email` — ✅ stored
- `name` — ✅ stored
- `id` (Facebook user ID) — ❌ NOT stored
- `picture` — ❌ NOT stored
- Available with permission: `gender`, `age_range`, `location`, `friends_count` — ❌ NOT requested

### WHAT WE SHOULD CAPTURE AT EVERY TOUCHPOINT

**At SIGNUP (ta_user needs new columns):**
```sql
ALTER TABLE ta_user ADD COLUMN acquisition_source VARCHAR(50);      -- google_ads/meta_ads/tiktok_ads/organic/direct/referral/chatgpt
ALTER TABLE ta_user ADD COLUMN acquisition_medium VARCHAR(50);      -- cpc/social/organic/email/referral/mcp
ALTER TABLE ta_user ADD COLUMN acquisition_campaign VARCHAR(200);   -- specific campaign
ALTER TABLE ta_user ADD COLUMN acquisition_landing_page VARCHAR(500); -- first page ever seen
ALTER TABLE ta_user ADD COLUMN acquisition_referrer VARCHAR(500);   -- document.referrer on first visit
ALTER TABLE ta_user ADD COLUMN acquisition_fbclid VARCHAR(255);     -- Facebook click ID
ALTER TABLE ta_user ADD COLUMN acquisition_ttclid VARCHAR(255);     -- TikTok click ID
ALTER TABLE ta_user ADD COLUMN acquisition_gclid VARCHAR(255);      -- Google click ID
ALTER TABLE ta_user ADD COLUMN sso_provider VARCHAR(20);            -- google/facebook/apple/phone/email
ALTER TABLE ta_user ADD COLUMN sso_locale VARCHAR(10);              -- locale from Google/FB SSO
ALTER TABLE ta_user ADD COLUMN sso_profile_photo VARCHAR(500);      -- profile photo URL from SSO
ALTER TABLE ta_user ADD COLUMN first_device_type VARCHAR(20);       -- mobile/desktop/tablet
ALTER TABLE ta_user ADD COLUMN first_city VARCHAR(100);             -- GeoIP at registration
ALTER TABLE ta_user ADD COLUMN first_country VARCHAR(50);           -- GeoIP country
```

**At EVERY TRANSACTION (add to tables missing UTM):**
- `ta_offer` → add utm_source, utm_medium, utm_campaign, fbclid, ttclid
- `ta_bid_entity` → add utm_source, utm_medium, utm_campaign, fbclid, ttclid
- `ta_syndication_user_subscription` → add utm_source, utm_medium, utm_campaign
- `ta_coin_package_order` → add utm_source, utm_medium, utm_campaign
- `ta_publication` → add utm_source, utm_medium, utm_campaign (which channel drove this listing creation?)
- `ta_publication_boost` → add utm_source, utm_medium, utm_campaign

### INFERRED USER PROFILING (computed from behavior, no questions asked)

| Computed Property | How to compute | Stored in |
|---|---|---|
| `inferred_occupation` | Category browsing patterns + time-of-day + listing types | ta_user_profile_computed |
| `inferred_income_bracket` | Price filters + purchases + coin pack tier chosen | ta_user_profile_computed |
| `inferred_family_status` | Categories: baby/kids/family vehicles/school supplies | ta_user_profile_computed |
| `inferred_housing_status` | RE browsing: rental vs sale vs none | ta_user_profile_computed |
| `inferred_tech_level` | AI usage + image search + multi-device + feature adoption | ta_user_profile_computed |
| `inferred_location` | Listing cities + search filters + IP geo patterns | ta_user_profile_computed |
| `inferred_buyer_seller_ratio` | Buying actions / selling actions over lifetime | ta_user_profile_computed |
| `inferred_price_sensitivity` | Offer discounts + filter tightness + pack choice | ta_user_profile_computed |
| `inferred_trust_threshold` | Time from first view to first contact (per user) | ta_user_profile_computed |
| `is_mre` | IP from EU/US + searches Morocco + FR/NL/DE locale | ta_user_profile_computed |
| `lifecycle_stage` | first_visit / activated / engaged / power_user / declining / churned | ta_user_profile_computed |
| `predicted_next_category` | Cross-category purchase sequence model | ta_user_profile_computed |

### THE FLOW (how acquisition data gets captured)

```
User clicks ad (Meta/TikTok/Google) → lands on staging.tawadoo.ma/?utm_source=meta&utm_medium=cpc&utm_campaign=summer26&fbclid=xxx

Frontend captures UTM from URL → stores in sessionStorage + first-party cookie (30-day window)

User browses → signs up (via phone OTP or SSO)

At signup API call: frontend sends UTM params + SSO data with the registration request

Backend stores on ta_user: acquisition_source='meta_ads', acquisition_medium='cpc', acquisition_campaign='summer26', fbclid='xxx', sso_provider='google', sso_locale='fr'

User later buys coins → ta_coin_package_order gets utm_source (from current session OR original acquisition if same session)

User later creates listing → ta_publication gets utm_source (same logic)

BO dashboard shows: "Users from Meta ads convert to sellers at 12%, vs Google organic at 8%. Meta LTV: 450 MAD. Google LTV: 280 MAD."
```

---

## 20. SESSION SUMMARY — TOTAL INTELLIGENCE LAYER SCOPE

### Everything identified in this research session (Brain 6):

| Layer | Signals/Items | Status |
|---|---|---|
| P0 core events (Session 14C) | 65 events | ✅ Deployed |
| P1 behavioral depth (Session 14D old) | 85 events | 🔄 In flight (~20% progress) |
| Naming reconciliation + server wiring (14D definitive) | 11 kills + 9 renames + 18 server events | 📋 Prompt written |
| Proprietary event store (14D.1) | ta_analytics_event table + API endpoint + sovereignty pattern | 📋 Spec'd in Brain V5 |
| Attribution/acquisition gap (14D.1 addition) | 14 new columns on ta_user + UTM on 6 transaction tables | 📋 Spec'd above |
| Extended behavioral intelligence (14E) | 170+ additional signals from global research | 📋 Research complete |
| Enrichment import (from Amplitude/Meta/TikTok/Google/MCP back to DB) | 8 enrichment tables | 📋 Spec'd in Brain V4 |
| Inferred user profiling | 12 computed properties | 📋 Spec'd above |
| Tawadoo-unique innovations | 10 signals nobody else captures | 📋 Research complete |
| Amplitude configuration | Dashboards, funnels, cohorts, S3 export, Session Replay, cohort syncs | 📋 Will rebuild after naming is clean |
| **TOTAL** | **372 named events + 170 behavioral signals + 14 acquisition columns + 12 computed properties + 8 enrichment tables** | — |

### CORRECTED EXECUTION SEQUENCE (after 14D finishes):

```
1. QA 14D (verify from source — what did it actually deliver?)
2. Fire 14D.1: SOVEREIGNTY SESSION
   - Create ta_analytics_event (proprietary event store, partitioned)
   - Create POST /api/analytics/events endpoint
   - Rewire frontend: track() → YOUR API → mirror to Amplitude
   - Add acquisition columns to ta_user (migration)
   - Add UTM columns to ta_offer/bid/subscription/coin/publication/boost (migrations)
   - Store SSO metadata (provider, locale, photo) on signup
   - Frontend: capture UTM on first visit → send with every API call
   - Configure Amplitude S3 export → lake
   - Move MCP observability from SQLite → PostgreSQL

3. Fire 14E: EXTENDED BEHAVIORAL INTELLIGENCE
   - 170+ signals from the global research
   - Micro-behaviors (cursor, velocity, hesitation)
   - Lifecycle predictions (cross-category sequences)
   - Conversation intelligence (message NLP)
   - Temporal signals (time-of-day, payday, seasonal)
   - Social/network (sharing, household, referral)
   - Vertical-specific (per real estate, vehicles, fashion, auctions, jobs)
   - Tawadoo-unique (Darija intent, WhatsApp migration, MRE, prayer gaps, auction emotion)
   - ALL write to ta_analytics_event FIRST → then mirror to Amplitude

4. Fire 14F: ENRICHMENT IMPORT
   - Amplitude predictions → ta_user_prediction
   - Amplitude cohorts → ta_user_cohort
   - Meta/TikTok/Google → ta_channel_performance
   - MCP queries → ta_mcp_interaction
   - Computed user profiles → ta_user_profile_computed (daily cron)

5. BO Intelligence Cockpit (Session 16) — reads ALL from YOUR DB
   - MetricsService queries ta_analytics_event + ta_user + enrichment tables
   - Zero dependency on Amplitude API for operational dashboards
```

---

*Research document COMPLETE. 20 sections. Everything captured. Next session: QA 14D → fire 14D.1 with full sovereignty + attribution scope.*
