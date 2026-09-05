# TAWADOO EVENT NAMING TRUTH — SINGLE SOURCE OF TRUTH

> **LAW 1 applies.** This is THE canonical naming registry. Every event name across frontend, backend, Amplitude, GTM, Meta CAPI, TikTok Events, and GA4 MUST match exactly what's defined here.
> **Convention:** `object_action` (snake_case, past tense for completed actions).
> **DB grounding:** Every event name maps to a real DB entity, a real status transition, or a real user action. No made-up names.

---

## SECTION 1: CURRENT STATE IN AMPLITUDE (89 events as of Aug 20, 2026)

### What EXISTS and is CORRECT (keep — canonical names)

| # | Current Amplitude Name | Status | Source | Maps to DB/Action |
|---|---|---|---|---|
| 1 | `search_performed` | ✅ KEEP (P0) | Web filter.tsx | User submits search query |
| 2 | `search_no_results` | ✅ KEEP (P0) | Web filter.tsx | Zero results returned |
| 3 | `listing_viewed` | ✅ KEEP (P0) | Web product-details-view.tsx | ta_publication_view created |
| 4 | `listing_price_compared` | ✅ KEEP (P0) | Web exploreMore.tsx | User views similar items section |
| 5 | `order_placed` | ✅ KEEP (P0) | Web BuyNowModal.tsx + API | ta_order created (status: pending) |
| 6 | `offer_rejected` | ✅ KEEP | Web offerItemLigne.tsx | ta_offer.accepted = false |
| 7 | `boost_purchased` | ✅ KEEP (P0) | Web BoostDistributeModal + API | ta_publication_boost created |
| 8 | `package_purchased` | ✅ KEEP | Web pricing-plans-page.tsx | ta_syndication_user_subscription created |
| 9 | `package_selected` | ✅ KEEP | Web pricing-plans-page.tsx | User selects a tier |
| 10 | `language_switched` | ✅ KEEP | Web LanguageDropdown.tsx | User changes locale |
| 11 | `store_boost_activated` | ✅ KEEP | Web StoreBoostModal.tsx | ta_store_boost created |
| 12 | `account_deleted` | ✅ KEEP | Web user-profile-page.tsx | ta_user soft-deleted |
| 13 | `lucky_wheel_spun` | ✅ KEEP | Web luckyWheel.tsx | Gamification spin |
| 14 | `display_ad_impression` | ✅ KEEP | Web DisplayAdBanner.tsx | ta_sponsor_event (type: impression) |
| 15 | `display_ad_click` | ✅ KEEP | Web DisplayAdBanner.tsx | ta_sponsor_event (type: click) |

### What EXISTS but has DUPLICATE/LEGACY names (DEPRECATE the legacy, keep the P0)

| # | LEGACY Name (KILL) | P0 Name (KEEP) | Both fire on same action | File |
|---|---|---|---|---|
| 1 | `search` | `search_performed` | Yes — CanonicalSearchBar fires `search`, filter.tsx fires `search_performed` | Kill `search` |
| 2 | `contact_seller` | `listing_contact_clicked` | Yes — productDetails fires both | Kill `contact_seller` |
| 3 | `view_item` (trackEcommerce) | `listing_viewed` | Both fire on listing detail page load | Kill `view_item` — use `listing_viewed` only |
| 4 | `post_listing` | `listing_creation_completed` | Both fire on publish | Kill `post_listing` |
| 5 | `store_visit` | `seller_profile_viewed` | Both fire on store page | Kill `store_visit` |
| 6 | `boost_listing` | `boost_purchased` | Both fire on boost purchase | Kill `boost_listing` |
| 7 | `feed_toggle` | `feed_view_toggled` | Both fire on grid↔feed switch | Kill `feed_toggle` |
| 8 | `add_to_wishlist` (trackEcommerce) | `listing_favorited` | Both fire on heart icon | Kill `add_to_wishlist` |
| 9 | `share` | `listing_shared` | Both fire on share action | Kill `share` |
| 10 | `sign_up` | `signup_completed` | Both fire on successful registration | Kill `sign_up` |
| 11 | `login` | `login_completed` | Both fire on auth success | Kill `login` |

**ACTION REQUIRED:** Remove ALL legacy `track()` calls from code. One event, one name. The P0 name wins.

### What EXISTS but is MISNAMED (RENAME to match taxonomy)

| # | Current Name | Canonical Name (per taxonomy) | Reason |
|---|---|---|---|
| 1 | `lead_captured` | `callback_requested` | "Lead captured" is an internal concept. The USER action is "request a callback." Maps to ta_lead creation with buyer_phone. |
| 2 | `generate_lead` | `callback_requested` | Same action, different legacy name. MERGE with above. |
| 3 | `begin_checkout` | `checkout_coin_started` | Ambiguous "begin_checkout" — is it order? coins? Need to distinguish. This fires on COIN purchase Payzone redirect. |
| 4 | `purchase` (trackEcommerce) | `coin_purchased` | "purchase" is generic. This fires specifically when coin pack payment completes. Maps to ta_wallet_history credit. |
| 5 | `feed_enter` | `feed_entered` | Past tense for consistency |
| 6 | `feed_exit` | `feed_exited` | Past tense for consistency |
| 7 | `video_play` | `listing_video_played` | P0 name already exists, legacy still fires separately |
| 8 | `add_to_cart` (trackEcommerce) | `buy_now_clicked` | There's no "cart" in Tawadoo. This fires when user clicks Buy Now. Maps to the START of order flow. |
| 9 | `coins_page_cta` | `pricing_page_viewed` | Rename to match taxonomy. CTA click on coins page = viewing the pricing/purchase options. |

### What EXISTS and is NOISE (DELETE — no business value or test artifacts)

| # | Event Name | Reason to DELETE |
|---|---|---|
| 1 | `session_14b_verification_test` | Test artifact from Session 14B verification |
| 2 | `session_14b_final_check` | Test artifact |
| 3 | `tab_switched` | Generic — what tab? where? Replaced by specific events like `store_tab_switched` |

---

## SECTION 2: THE 5 TRANSACTION MODELS — EVENT LIFECYCLE (grounded in DB)

### MODEL A: BUY NOW (ta_order)

**DB Entity:** `ta_order`
**Status Enum:** `pending → confirmed → shipped → in_transit → delivered → returned → cancelled`
**Source Channel:** `google | meta | tiktok | chatgpt | direct`

| Step | Canonical Event Name | Who triggers | Where tracked | Properties |
|---|---|---|---|---|
| 1 | `buy_now_clicked` | Buyer | Frontend (BuyNowModal) | listing_id, price, source_page |
| 2 | `checkout_started` | Buyer | Frontend (BuyNowModal) | listing_id, price, delivery_available |
| 3 | `delivery_method_selected` | Buyer | Frontend (BuyNowModal) | method (tawssil_home/relay_point/in_person), cost, city |
| 4 | `order_placed` | Buyer | Frontend + Backend | order_id, reference, amount, delivery_fee, source_channel, utm_*, commission_rate |
| 5 | `guest_order_placed` | Guest buyer | Backend (GuestOrdersController) | order_id, guest_id, phone_verified |
| 6 | `order_confirmed_by_seller` | Seller | Backend (OrdersService.confirmOrder) | order_id, time_to_confirm_hours |
| 7 | `order_shipped` | System (Tawssil) | Backend (tawssil webhook → picked_up) | order_id, tawssil_barcode, tracking_url |
| 8 | `order_in_transit` | System (Tawssil) | Backend (tawssil webhook → in_transit) | order_id |
| 9 | `order_delivered` | System (Tawssil) | Backend (tawssil webhook → delivered) | order_id, delivery_days, amount |
| 10 | `order_returned` | System (Tawssil) | Backend (tawssil webhook → returned) | order_id, reason |
| 11 | `order_cancelled` | Buyer or Seller | Backend (cancelOrder/buyerCancelOrder) | order_id, who_cancelled (buyer/seller), reason |
| 12 | `order_tracking_viewed` | Buyer | Frontend (tracking page) | order_id, status, views_count |

**FUNNEL: Buyer Purchase (Buy-Now)**
```
listing_viewed → buy_now_clicked → checkout_started → delivery_method_selected
→ order_placed → order_confirmed_by_seller → order_shipped → order_delivered → review_submitted
```

---

### MODEL B: OFFERS & NEGOTIATION (ta_offer)

**DB Entity:** `ta_offer`
**Status Model:** `accepted` column: `null` (pending) | `true` (accepted) | `false` (rejected)
**Business Rule:** Max 5 offers per buyer per publication. Price must be ≤ listing price.

| Step | Canonical Event Name | Who triggers | Where tracked | Properties |
|---|---|---|---|---|
| 1 | `offer_form_opened` | Buyer | Frontend (makeOffer.tsx) | listing_id, listed_price |
| 2 | `offer_submitted` | Buyer | Frontend + Backend | listing_id, offered_price, discount_pct, category |
| 3 | `offer_viewed_by_seller` | Seller | Backend (when seller opens offers) | offer_id, listing_id, time_to_view_hours |
| 4 | `offer_accepted` | Seller | Backend (OfferService.update accepted=true) | offer_id, listing_id, final_price, time_to_accept_hours |
| 5 | `offer_rejected` | Seller | Backend (OfferService.update accepted=false) | offer_id, listing_id, reason |
| 6 | `offer_countered` | Seller | Backend (future — counter-offer flow) | offer_id, counter_amount, original_amount |
| 7 | `callback_requested` | Buyer | Frontend (LeadCaptureModal) + Backend (LeadsService.createLead) | listing_id, buyer_phone_prefix, lead_type: 'callback' |
| 8 | `callback_completed` | Seller | Backend (LeadsService.markAsContacted) | lead_id, listing_id, response_time_minutes |

**WHAT HAPPENS AFTER OFFER ACCEPTED:**
- Phone number revealed (both parties can now call/WhatsApp directly)
- This is lead_gen model — no order created (unlike Buy Now). Deal happens offline.
- Maps to: `listing_phone_revealed` event when either party views the other's phone.

**FUNNEL: Negotiation**
```
listing_viewed → offer_form_opened → offer_submitted → offer_viewed_by_seller
→ offer_accepted → listing_phone_revealed → (offline deal)
```

---

### MODEL C: AUCTION & BIDDING (ta_bid_room + ta_bid_entity + ta_bid_transaction)

**DB Entities:**
- `ta_bid_room` — the auction. Status: `not_started → opened → closed → canceled`
- `ta_bid_entity` — a participant. Columns: entity_id, automatic_bid, automatic_max_amount, automatic_step
- `ta_bid_transaction` — a single bid. Columns: price, bid_entity_id, by_automatic_bid

**Business Rules:**
- Join costs coins (from category `coinBidNeed`)
- Bid must be > last_price AND ≤ 150% of last_price
- Auto-bid fires 5s after any manual bid
- When closed: winner = last_bidder. ALL non-winners get deposit refund.
- After auction_won → standard order flow (buyer pays, Tawssil delivers)

| Step | Canonical Event Name | Who triggers | Where tracked | Properties |
|---|---|---|---|---|
| 1 | `auction_viewed` | Buyer | Frontend (auction-view.tsx) | auction_id, current_bid, bidders_count, time_remaining |
| 2 | `bid_room_joined` | Buyer | Backend (BidEntityService.joinBidRoom) | auction_id, coins_spent (from category.coinBidNeed) |
| 3 | `bid_placed` | Buyer | Backend (BidTransactionService.placeBid) | auction_id, amount, bid_number, is_winning, auto_bid |
| 4 | `bid_outbid` | System | Backend (on someone else bidding higher) | auction_id, new_highest, my_bid, time_since_my_bid |
| 5 | `bid_auto_set` | Buyer | Frontend (BidEntity update) | auction_id, max_limit, increment |
| 6 | `bid_auto_triggered` | System | Backend (placeAutomaticBid) | auction_id, amount, triggered_by_opponent_amount |
| 7 | `auction_won` | System | Backend (BidRoomService.updateOpenedToClosed) | auction_id, final_price, starting_price, total_bids, my_bid_count |
| 8 | `auction_lost` | System | Backend (BidRoomService — notify losers) | auction_id, final_price, my_max_bid |
| 9 | `bid_deposit_refunded` | System | Backend (wallet refund on auction close) | auction_id, amount_refunded |
| 10 | `auction_watched` | Buyer | Frontend (add to watchlist) | auction_id, time_before_end |

**WHAT HAPPENS AFTER AUCTION WON:**
- Winner and seller are connected (phone revealed)
- Order placed (standard order flow) OR offline deal
- Maps to: `order_placed` with source context indicating auction origin

**FUNNEL: Auction**
```
auction_viewed → bid_room_joined (coins paid) → bid_placed → bid_outbid → bid_placed (retaliation)
→ auction_won → order_placed → order_delivered
```

---

### MODEL D: LEAD GENERATION (ta_lead + ta_lead_unlock)

**DB Entity:** `ta_lead`
**Status Enum:** `new → viewed → contacted → converted → expired`
**Source Channel:** `google | meta | tiktok | chatgpt | direct`
**Distribution Model:** `lead_gen | rental`
**Unlock:** Seller pays coins OR uses included leads from package

| Step | Canonical Event Name | Who triggers | Where tracked | Properties |
|---|---|---|---|---|
| 1 | `listing_contact_initiated` | Buyer | Frontend (productDetails contact button) | listing_id, method (chat/call/whatsapp), gate_context |
| 2 | `callback_requested` | Buyer | Frontend (LeadCaptureModal) + Backend | listing_id, buyer_phone, source_channel, distribution_model |
| 3 | `lead_received` | System | Backend (LeadsService.createLead) | lead_id, listing_id, seller_entity_id, source_channel |
| 4 | `lead_viewed_by_seller` | Seller | Backend (LeadsService.markAsViewed) | lead_id, time_to_view_hours |
| 5 | `lead_unlocked` | Seller | Backend (LeadsService.unlockLead) | lead_id, listing_id, coins_spent, source ('included'/'purchased') |
| 6 | `lead_contacted` | Seller | Backend (LeadsService.markAsContacted) | lead_id, response_time_minutes |
| 7 | `lead_converted` | Seller | Backend (LeadsService.markAsConverted) | lead_id, listing_id |
| 8 | `lead_expired` | System | Backend (cron) | lead_id, hours_elapsed |

**CRITICAL DISTINCTION:**
- `callback_requested` = the BUYER's action (they submit their phone number via OTP-verified form)
- `lead_received` = the SELLER sees a new lead notification
- `lead_unlocked` = the SELLER pays coins to reveal buyer's phone number
- These are THREE different events for THREE different actors at THREE different times.

**FUNNEL: Lead Generation**
```
listing_viewed → listing_contact_initiated → callback_requested (OTP verified)
→ lead_received (seller notified) → lead_viewed_by_seller → lead_unlocked (coins paid)
→ lead_contacted → lead_converted
```

---

### MODEL E: COINS & SUBSCRIPTION (ta_wallet + ta_wallet_history + ta_syndication_user_subscription)

**DB Entities:**
- `ta_wallet` — balance (coins field)
- `ta_wallet_history` — every credit/debit with `source` string
- `ta_coin_package` — purchasable packs
- `ta_syndication_user_subscription` — active subscription with tier/track/channels

**Wallet History Sources (from code):**
- `coinPackagePurchase` — bought a coin pack
- `lead_unlock` — paid to see a lead
- `joinBidRoom` — paid to enter auction
- `refundSecurityDeposit` — refund after auction loss
- `boost` — paid for listing boost
- `subscription` — paid for distribution package
- `wheel` — won from lucky wheel
- `referral` — earned from referral
- `giveaway` — admin gift

| Step | Canonical Event Name | Who triggers | Where tracked | Properties |
|---|---|---|---|---|
| 1 | `pricing_page_viewed` | User | Frontend (pricing-plans-page) | source_page, current_tier |
| 2 | `coin_pack_viewed` | User | Frontend (coin-packages-view) | source |
| 3 | `coin_pack_selected` | User | Frontend | pack_name, price_mad, coins_amount |
| 4 | `checkout_coin_started` | User | Frontend (redirect to Payzone) | pack_name, amount_mad, payment_method (card/cash) |
| 5 | `payment_initiated` | System | Frontend (Payzone form loads) | method, amount, currency |
| 6 | `payment_completed` | System | Backend (Payzone callback) | transaction_id, amount, method, processing_time_ms |
| 7 | `payment_failed` | System | Backend (Payzone decline) | failure_code, failure_reason, retry_attempted |
| 8 | `coin_purchased` | System | Backend (wallet credited) | pack_name, amount_mad, coins_received, bonus_coins |
| 9 | `coin_earned` | System | Backend (any non-purchase credit) | source (wheel/referral/giveaway/refund), amount |
| 10 | `coin_spent` | System | Backend (any debit) | source (boost/bid_room/subscription/lead_unlock), amount, target_id |
| 11 | `subscription_purchased` | User | Backend | tier (elan/portee/rayonnement), track (buy_now/lead_gen), duration_days, coins_spent |
| 12 | `subscription_trial_claimed` | User | Backend (TrialService.claimTrial) | slots (10), source_cta |
| 13 | `subscription_expired` | System | Backend (cron/trial expiry) | tier, track, days_active |
| 14 | `subscription_renewed` | User | Backend | tier, track, coins_spent, consecutive_months |
| 15 | `subscription_cancelled` | User | Backend | tier, track, reason, days_active |
| 16 | `boost_purchased` | User | Backend + Frontend | listing_id, duration_days, coins_spent |
| 17 | `boost_expired` | System | Backend (cron) | listing_id, duration_was, views_during_boost |
| 18 | `lead_unlocked` | Seller | Backend (LeadsService.unlockLead) | lead_id, coins_spent, source (included/purchased) |
| 19 | `wallet_viewed` | User | Frontend | balance, lifetime_earned, lifetime_spent |

**FUNNEL: Monetization**
```
pricing_page_viewed → coin_pack_selected → checkout_coin_started → payment_completed
→ coin_purchased → (boost_purchased | subscription_purchased | lead_unlocked)
```

---

## SECTION 3: SELLER-SIDE EVENTS (what the SELLER experiences)

These are CRITICAL and mostly MISSING from current implementation:

| # | Event | Seller Action | Where to Track | DB Trigger |
|---|---|---|---|---|
| 1 | `order_received` | Seller sees new order notification | Backend (OrdersService.createOrder → notify seller) | ta_order created WHERE seller_entity_id = mine |
| 2 | `order_confirmed_by_seller` | Seller taps "Confirm" | Backend (OrdersService.confirmOrder) | ta_order.status → confirmed |
| 3 | `offer_received` | Seller sees new offer | Backend (OfferService.create → notify seller) | ta_offer created WHERE seller_id = mine |
| 4 | `offer_viewed_by_seller` | Seller opens the offer detail | Backend/Frontend | ta_offer viewed |
| 5 | `offer_accepted` | Seller accepts | Backend (OfferService.update accepted=true) | ta_offer.accepted = true |
| 6 | `offer_rejected` | Seller rejects | Backend (OfferService.update accepted=false) | ta_offer.accepted = false |
| 7 | `lead_received` | Seller gets callback request | Backend (LeadsService.createLead) | ta_lead created WHERE seller_entity_id = mine |
| 8 | `lead_unlocked` | Seller pays to see phone | Backend (LeadsService.unlockLead) | ta_lead_unlock created |
| 9 | `bid_received` | Seller's auction gets a new bid | Backend (BidTransactionService.placeBid) | ta_bid_transaction on seller's bid_room |
| 10 | `auction_completed_seller` | Seller's auction closes with winner | Backend (BidRoomService → close) | ta_bid_room.status → closed WHERE my publication |
| 11 | `listing_viewed_notification` | Seller sees milestone view count | Backend (NotificationService) | ta_publication_view count crosses threshold |
| 12 | `seller_dashboard_viewed` | Seller opens their dashboard | Frontend | Dashboard page load |
| 13 | `seller_insights_viewed` | Seller checks analytics tab | Frontend | Insights/stats page load |

---

## SECTION 4: WHAT'S COMPLETELY MISSING (gaps vs taxonomy)

### CRITICAL MISSING (P0 — must be in next session):

| # | Canonical Event Name | Why critical | Where to add |
|---|---|---|---|
| 1 | `callback_requested` | THE lead gen action. Buyer submits phone. Currently tracked as `lead_captured` (wrong name) | Frontend (LeadCaptureModal) — RENAME existing |
| 2 | `order_confirmed_by_seller` | Without this, we can't measure seller response time | Backend (OrdersService.confirmOrder) |
| 3 | `order_shipped` | Can't measure fulfillment without it | Backend (Tawssil webhook processor) |
| 4 | `order_delivered` | Can't measure delivery success | Backend (Tawssil webhook processor) |
| 5 | `order_cancelled` | Can't measure cancel rate or reasons | Backend (cancelOrder/buyerCancelOrder) |
| 6 | `auction_won` | Users win auctions — no tracking of this key moment | Backend (BidRoomService.updateOpenedToClosed → winner) |
| 7 | `auction_lost` | Users lose auctions — no tracking | Backend (BidRoomService → notify losers) |
| 8 | `bid_room_joined` | User PAYS COINS to enter — a monetization event! | Backend (BidEntityService.joinBidRoom) |
| 9 | `bid_outbid` | User gets outbid — drives re-engagement | Backend (BidTransactionService.placeBid → notify previous) |
| 10 | `offer_accepted` (server-side) | Seller accepts offer — key conversion moment | Backend (OfferService.update) |
| 11 | `lead_unlocked` | Seller pays coins — a monetization event! | Backend (LeadsService.unlockLead) |
| 12 | `lead_received` | Seller perspective of a new lead | Backend (LeadsService.createLead → notify) |
| 13 | `subscription_expired` | Subscription ends — churn signal | Backend (syndication expiry processor) |
| 14 | `subscription_trial_claimed` | Trial claimed — funnel start | Backend (TrialService.claimTrial) |
| 15 | `coin_earned` | Any non-purchase credit (wheel, referral, refund) | Backend (WalletService.createHistory where coins > 0) |
| 16 | `coin_spent` | Any debit (boost, bid, lead, subscription) | Backend (WalletService.createHistory where coins < 0) |
| 17 | `guest_order_placed` | Guest checkout — different from registered user | Backend (GuestOrdersController) |
| 18 | `buy_now_clicked` | THE first step of purchase. Before checkout. | Frontend (BuyNowModal open) — currently fires as `add_to_cart` which is WRONG |

### HIGH PRIORITY MISSING (P1 — behavioral depth):

| # | Event | Why needed |
|---|---|---|
| 1 | `listing_dwell_time` | How long users spend on a listing = interest signal |
| 2 | `offer_viewed_by_seller` | Seller response time measurement |
| 3 | `bid_auto_set` | Auto-bid configuration = commitment signal |
| 4 | `bid_auto_triggered` | System action on behalf of user |
| 5 | `offer_countered` | Counter-offer (future flow, not yet in DB but PLANNED) |
| 6 | `listing_gallery_swiped` | Photo browsing depth = purchase intent |
| 7 | `checkout_started` | Explicitly entering the checkout form (distinct from buy_now_clicked) |
| 8 | `delivery_rate_fetched` | Tawssil returns delivery estimate |
| 9 | `listing_callback_requested` | = `callback_requested` (same event, listing context) |
| 10 | `store_followed` | Social signal |
| 11 | `notification_opened` | User acts on notification = engagement |
| 12 | `deep_link_arrived` | Attribution — where did this user come from? |
| 13 | `search_image_used` | Camera/image search usage |
| 14 | `listing_shared` | Viral signal — which listings get shared |
| 15 | `auth_gate_shown` / `auth_gate_converted` / `auth_gate_abandoned` | Conversion gate funnel |

---

## SECTION 5: THE MASTER NAME REGISTRY (canonical names — use ONLY these)

### Naming Rules:
1. `object_action` format (snake_case)
2. Past tense for completed actions: `order_placed`, `bid_placed`, `offer_submitted`
3. Present participle ONLY for ongoing states: never use in event names
4. Object = the DB entity or UI component: `listing`, `order`, `offer`, `bid`, `auction`, `lead`, `coin`, `subscription`, `boost`, `feed`, `search`, `auth`, `notification`, `widget`, `store`
5. NEVER use generic verbs alone: no `purchase`, `search`, `share`, `login` — always prefix with object
6. NEVER use internal jargon the user wouldn't recognize

### The Registry (by domain):

**IDENTITY (8 canonical names):**
`signup_completed` · `login_completed` · `logout` · `auth_gate_triggered` · `auth_gate_converted` · `auth_gate_abandoned` · `account_deleted` · `profile_completed`

**SEARCH (10 canonical names):**
`search_performed` · `search_no_results` · `search_result_clicked` · `search_filter_applied` · `search_sort_changed` · `search_image_used` · `autocomplete_selected` · `saved_search_created` · `category_browsed` · `search_abandoned`

**LISTING ENGAGEMENT (12 canonical names):**
`listing_viewed` · `listing_favorited` · `listing_shared` · `listing_contact_initiated` · `listing_video_played` · `listing_gallery_swiped` · `listing_image_zoomed` · `listing_description_expanded` · `listing_dwell_time` · `listing_price_compared` · `listing_phone_revealed` · `listing_reported`

**SELL FLOW (10 canonical names):**
`listing_creation_started` · `listing_photo_uploaded` · `listing_ai_generated` · `listing_ai_edited` · `listing_category_selected` · `listing_price_set` · `listing_creation_completed` · `listing_creation_abandoned` · `listing_edited` · `listing_deleted`

**BUY NOW / ORDERS (12 canonical names):**
`buy_now_clicked` · `checkout_started` · `delivery_method_selected` · `order_placed` · `guest_order_placed` · `order_confirmed_by_seller` · `order_shipped` · `order_in_transit` · `order_delivered` · `order_returned` · `order_cancelled` · `order_tracking_viewed`

**OFFERS & NEGOTIATION (8 canonical names):**
`offer_form_opened` · `offer_submitted` · `offer_viewed_by_seller` · `offer_accepted` · `offer_rejected` · `offer_countered` · `callback_requested` · `callback_completed`

**AUCTION & BIDDING (10 canonical names):**
`auction_viewed` · `bid_room_joined` · `bid_placed` · `bid_outbid` · `bid_auto_set` · `bid_auto_triggered` · `auction_won` · `auction_lost` · `bid_deposit_refunded` · `auction_watched`

**LEADS (8 canonical names):**
`lead_received` · `lead_viewed_by_seller` · `lead_unlocked` · `lead_contacted` · `lead_converted` · `lead_expired` · `listing_contact_initiated` · `callback_requested`

**COINS & PAYMENTS (12 canonical names):**
`coin_pack_viewed` · `coin_pack_selected` · `checkout_coin_started` · `payment_initiated` · `payment_completed` · `payment_failed` · `coin_purchased` · `coin_earned` · `coin_spent` · `wallet_viewed` · `pricing_page_viewed` · `coin_purchase_abandoned`

**SUBSCRIPTIONS & BOOSTS (10 canonical names):**
`subscription_purchased` · `subscription_trial_claimed` · `subscription_expired` · `subscription_renewed` · `subscription_upgraded` · `subscription_cancelled` · `boost_purchased` · `boost_extended` · `boost_expired` · `store_boost_activated`

**FEED & DISCOVERY (8 canonical names):**
`feed_entered` · `feed_exited` · `feed_scrolled` · `feed_action_taken` · `feed_view_toggled` · `listing_impression` · `store_card_clicked` · `category_clicked`

**MESSAGING (5 canonical names):**
`chat_initiated` · `message_sent` · `message_received` · `seller_response_time` · `chat_thread_archived`

**NOTIFICATIONS (5 canonical names):**
`notification_sent` · `notification_opened` · `notification_dismissed` · `deep_link_arrived` · `whatsapp_link_clicked`

**STORE (6 canonical names):**
`store_created` · `store_followed` · `seller_profile_viewed` · `store_video_played` · `store_boost_activated` · `seller_dashboard_viewed`

**GAMIFICATION (4 canonical names):**
`lucky_wheel_spun` · `referral_link_shared` · `referral_reward_earned` · `review_submitted`

**DISPLAY ADS (5 canonical names):**
`display_ad_impression` · `display_ad_click` · `display_ad_converted` · `ad_slot_filled` · `ad_slot_empty`

**AI & MCP (6 canonical names):**
`mcp_query_received` · `mcp_results_returned` · `mcp_user_converted` · `ai_classification_completed` · `ai_generation_completed` · `ai_embedding_indexed`

**SYSTEM (5 canonical names):**
`page_performance_measured` · `api_error_occurred` · `javascript_error` · `feed_health_checked` · `deployment_completed`

---

## SECTION 6: IMMEDIATE ACTION ITEMS (Session 14D MUST do these)

### STEP 1: KILL LEGACY DUPLICATES (11 events to remove)
Remove ALL legacy `track()` calls that fire alongside P0 equivalents. One action = one event name.

### STEP 2: RENAME MISMATCHES (9 events)
Change the event name strings in code to match this registry.

### STEP 3: ADD MISSING CRITICAL (18 server-side events)
Wire the 18 P0-critical events listed in Section 4 into the API backend via AmplitudeService.

### STEP 4: REBUILD FUNNELS IN AMPLITUDE
Delete the shallow funnels I created today. Replace with the 12 PRIMARY funnels from the taxonomy (Section 2 above) using the CORRECT canonical event names.

---

## SECTION 7: DB ENTITY → EVENT NAME MAP (for developers)

| DB Table | Status/Action | Event Name |
|---|---|---|
| `ta_order` created | Buyer places order | `order_placed` |
| `ta_order` status → confirmed | Seller confirms | `order_confirmed_by_seller` |
| `ta_order` status → shipped | Tawssil picks up | `order_shipped` |
| `ta_order` status → delivered | Tawssil delivers | `order_delivered` |
| `ta_order` status → cancelled | Cancel | `order_cancelled` |
| `ta_order` status → returned | Return | `order_returned` |
| `ta_offer` created | Buyer makes offer | `offer_submitted` |
| `ta_offer` accepted=true | Seller accepts | `offer_accepted` |
| `ta_offer` accepted=false | Seller rejects | `offer_rejected` |
| `ta_lead` created | Buyer requests callback | `callback_requested` + `lead_received` (seller-side) |
| `ta_lead` status → viewed | Seller sees lead | `lead_viewed_by_seller` |
| `ta_lead_unlock` created | Seller pays coins | `lead_unlocked` |
| `ta_lead` status → contacted | Seller calls | `lead_contacted` |
| `ta_lead` status → converted | Deal done | `lead_converted` |
| `ta_bid_room` status → opened | Auction starts | (system cron, no user event) |
| `ta_bid_room` status → closed | Auction ends | `auction_won` + `auction_lost` |
| `ta_bid_entity` created | User joins auction | `bid_room_joined` |
| `ta_bid_transaction` created | User bids | `bid_placed` |
| `ta_publication_boost` created | User boosts listing | `boost_purchased` |
| `ta_store_boost` created | User boosts store | `store_boost_activated` |
| `ta_wallet_history` coins > 0 | Any credit | `coin_earned` (source property distinguishes) |
| `ta_wallet_history` coins < 0 | Any debit | `coin_spent` (source property distinguishes) |
| `ta_syndication_user_subscription` created | Package bought | `subscription_purchased` |
| `ta_syndication_user_subscription` active=false | Expires | `subscription_expired` |
| `ta_trial` created | Trial claimed | `subscription_trial_claimed` |
| `ta_trial` status → expired | Trial ends | `subscription_expired` (with is_trial=true property) |
| `ta_trial` status → converted | Bought after trial | `subscription_purchased` (with from_trial=true property) |
| `ta_publication` created | Listing published | `listing_creation_completed` |
| `ta_publication_view` created | Someone views listing | `listing_viewed` |
| `ta_review` created | Review submitted | `review_submitted` |
| `ta_guest` created | Guest order | `guest_order_placed` |

---

*This document is the SINGLE SOURCE OF TRUTH for event naming. Any event name not in this registry is either LEGACY (to be killed) or NOT YET IMPLEMENTED (to be added per the taxonomy). When in doubt: this file wins.*

*Updated: Aug 20, 2026. Grounded in: DB entity schemas verified from source + V2 Full Event Taxonomy (312 events) + V2 AI Vision (372 events) + current Amplitude state (89 events).*


---
---

# PART 2: THE NAMING LAW (NON-NEGOTIABLE — APPLIES TO EVERY SESSION, EVERY REPO, FOREVER)

> **LAW 15 — EVENT NAMING IS SACRED (added Aug 20, 2026 by Ramzi+CTO)**
>
> No session, no developer, no AI may create, rename, or fire an analytics event without checking this registry FIRST. If the event exists here, use the EXACT name. If it doesn't exist here, ADD it to this registry BEFORE coding. NEVER invent event names on the fly. NEVER create duplicates. One action = one canonical name = everywhere.

## THE 12 NAMING RULES (unbreakable)

1. **`object_action` format.** Always. No exceptions. The object is the thing acted upon. The action is what happened. Example: `listing_viewed`, `order_placed`, `bid_placed`.

2. **snake_case only.** No camelCase, no PascalCase, no kebab-case. `search_performed` not `searchPerformed`.

3. **Past tense for completed actions.** `order_placed` not `order_place`. `bid_placed` not `bid_placing`. `offer_submitted` not `offer_submit`.

4. **Same name EVERYWHERE.** Frontend track() call, Backend AmplitudeService.track(), Amplitude dashboard, GTM dataLayer event name, Meta CAPI event_name, TikTok event — ALL use the identical string.

5. **One action = one event name.** If a user clicks "Buy Now", that fires `buy_now_clicked`. PERIOD. Not `buy_now_clicked` + `add_to_cart` + `begin_checkout`. ONE name for ONE user action.

6. **Object names match DB entities.** `order` = ta_order. `offer` = ta_offer. `bid` = ta_bid_transaction. `lead` = ta_lead. `listing` = ta_publication. `subscription` = ta_syndication_user_subscription. `boost` = ta_publication_boost. `coin` = ta_wallet_history. `store` = ta_entity (type=store). `auction` = ta_bid_room.

7. **Properties carry context, not the event name.** WRONG: `order_placed_via_google`, `order_placed_via_chatgpt`. RIGHT: `order_placed` with property `source_channel: 'google'`. The event name is the WHAT. Properties are the WHERE/HOW/WHY.

8. **Seller and buyer perspectives get DIFFERENT events when they're different actions.** Buyer submits offer = `offer_submitted`. Seller sees offer = `offer_viewed_by_seller`. Seller accepts = `offer_accepted`. These are THREE different events because they're THREE different human actions at THREE different times.

9. **System-triggered events use passive voice or `_by_system` suffix ONLY when needed for clarity.** `order_confirmed_by_seller` (not just `order_confirmed` — WHO confirmed matters). `bid_auto_triggered` (system did it, not user). `bid_deposit_refunded` (system refunds).

10. **No generic verbs alone.** BANNED: `purchase`, `search`, `share`, `login`, `view`, `click`. ALWAYS prefix with object: `coin_purchased`, `search_performed`, `listing_shared`, `login_completed`, `listing_viewed`, `buy_now_clicked`.

11. **No marketing/internal jargon in event names.** BANNED: `generate_lead` (user didn't "generate" anything — they requested a callback). BANNED: `gate_auth_success` (user doesn't know what a "gate" is — it's `auth_gate_converted`). Use the USER's perspective.

12. **Deprecated events are REMOVED from code, not left firing alongside replacements.** If `post_listing` is replaced by `listing_creation_completed`, the old track('post_listing') call is DELETED. Not commented out. Not left "for backwards compatibility." GONE.

---

## HOW TO USE THIS REGISTRY (for execution sessions)

**BEFORE writing any track() call:**
1. CTRL+F this document for the action you want to track
2. If it exists → use the EXACT canonical name and properties listed
3. If it doesn't exist → ADD a new entry to this registry FIRST, then code it
4. If you find a legacy/duplicate name in existing code → KILL IT and replace with the canonical name

**BEFORE creating any Amplitude chart/funnel:**
1. Use ONLY event names from this registry
2. The funnels in Section 10 below use these exact names — copy them verbatim

**BEFORE any PR/commit that touches analytics:**
1. Grep for the event name across ALL repos (web + API + MCP)
2. Verify it fires from exactly ONE location (no double-fire)
3. Verify the name matches this registry character-for-character


---

## SECTION 8: COMPLETE 23-DOMAIN EVENT REGISTRY (372 events, implementation status)

**Status markers:**
- ✅ LIVE — Currently firing in production/staging code
- 🔧 WIRED — Code exists, event call present, awaiting deployment/QA
- 📋 PLANNED — System exists in DB/API, event instrumentation needed
- 🔮 FUTURE — System not yet built (agents, widget, ML models)

---

### DOMAIN 1: IDENTITY & SESSION (20 events)

| # | Canonical Name | Explanation | Status | Where |
|---|---|---|---|---|
| 1 | `session_started` | A new browsing session begins (>30min gap from last activity) | 🔧 | Web AmplitudeInit |
| 2 | `session_heartbeat` | Pulse every 30s of active engagement (tab focused, scrolling) | 🔮 | Web (future P2) |
| 3 | `session_ended` | Tab close or 30s idle — captures duration and pages viewed | 🔧 | Web (visibilitychange) |
| 4 | `identify` | User identity set in Amplitude (on login/signup/session restore) | ✅ | Web AmplitudeInit + UnifiedAuthScreen |
| 5 | `signup_completed` | New account successfully created | ✅ | Web UnifiedAuthScreen + SignInScreen |
| 6 | `login_completed` | Returning user authenticated | ✅ | Web UnifiedAuthScreen + SignInScreen |
| 7 | `login_failed` | Auth attempt failed (wrong OTP, expired, rate-limited) | 📋 | Web (add to auth flow) |
| 8 | `logout` | User clicks logout | ✅ | Web HubHome + MobileNavSheet |
| 9 | `auth_gate_triggered` | Contextual auth wall shown (user tried an action requiring auth) | ✅ | Web AuthGate.tsx |
| 10 | `auth_gate_converted` | User completes auth from gate (the gate WORKED) | 🔧 | Web gateAnalytics (as gate_auth_success — RENAME) |
| 11 | `auth_gate_abandoned` | User dismisses gate without authenticating | ✅ | Web UnifiedAuthScreen |
| 12 | `profile_completed` | User fills required profile fields (name, city, phone verified) | 📋 | Web (profile section) |
| 13 | `profile_edited` | User modifies profile information | 📋 | Web (profile section) |
| 14 | `social_auth_clicked` | User clicks Google/Facebook/Apple SSO button | 📋 | Web (auth flow) |
| 15 | `signup_otp_requested` | OTP sent during signup (WhatsApp or email) | 🔧 | Web UnifiedAuthScreen (as otp_sent — RENAME) |
| 16 | `signup_otp_verified` | OTP correctly entered during signup | 🔧 | Web UnifiedAuthScreen (as otp_verified — RENAME) |
| 17 | `signup_started` | User initiates registration (clicks register or gate opens for signup) | 🔧 | Web (as auth_started — RENAME) |
| 18 | `account_deleted` | User permanently deletes their account | ✅ | Web user-profile-page |
| 19 | `language_switched` | User changes app language (ar/fr/en) | ✅ | Web LanguageDropdown |
| 20 | `deep_link_arrived` | User arrives via external link with UTM params | ✅ | Web AmplitudeInit |

---

### DOMAIN 2: DISCOVERY & BROWSE (25 events)

| # | Canonical Name | Explanation | Status | Where |
|---|---|---|---|---|
| 21 | `page_viewed` | Any page loads (virtual pageview for SPA) | ✅ | Web AnalyticsTracker (as virtual_page_view in dataLayer) |
| 22 | `homepage_section_viewed` | A homepage section enters viewport (trending/categories/stores) | 📋 | Web (IntersectionObserver on sections) |
| 23 | `category_clicked` | User clicks a category card/icon to browse | ✅ | Web cardCategory |
| 24 | `subcategory_selected` | User drills into a subcategory from parent | 📋 | Web (browse/category pages) |
| 25 | `listing_impression` | A listing card becomes visible for >500ms (batched, max 20 per fire) | 📋 | Web (IntersectionObserver on cards) |
| 26 | `listing_card_clicked` | User clicks a listing card to view detail (= `search_result_clicked` in search context) | ✅ | Web productCard (as search_result_clicked) |
| 27 | `listing_card_wishlisted` | Heart icon tapped from the card (without opening listing) | 📋 | Web (productCard heart) |
| 28 | `listing_card_shared` | Share triggered from card (without opening listing) | 📋 | Web (productCard share) |
| 29 | `feed_entered` | User enters immersive feed mode | ✅ | Web ImmersiveFeed (as feed_enter — RENAME past tense) |
| 30 | `feed_slide_changed` | User swipes to next/prev item in feed | ✅ | Web ImmersiveFeed |
| 31 | `feed_exited` | User leaves feed mode | ✅ | Web ImmersiveFeed (as feed_exit — RENAME) |
| 32 | `feed_action_taken` | User taps any CTA in feed (contact/offer/bid/buy/share/favorite) | ✅ | Web FeedActionRail (split as feed_contact/feed_buy_now/etc — MERGE into one) |
| 33 | `feed_view_toggled` | User switches between grid and feed display | ✅ | Web FeedToggle |
| 34 | `page_scroll_depth` | User scrolls past 25/50/75/100% milestone on any page | 📋 | Web (IntersectionObserver sentinels) |
| 35 | `section_impression` | Any named section becomes visible (for homepage analytics) | ✅ | Web useSectionImpression |
| 36 | `trending_item_clicked` | User clicks an item from the "trending" section specifically | 📋 | Web (trending section) |
| 37 | `store_card_clicked` | User clicks a store card in discovery/directory | ✅ | Web StoreCardUnified + StoreDiscoveryCard |
| 38 | `promo_banner_viewed` | A promotional banner enters viewport | ✅ | Web PromoBanner (as promo_view — RENAME) |
| 39 | `promo_banner_clicked` | User clicks a promotional banner | ✅ | Web PromoBanner (as promo_click — RENAME) |
| 40 | `promo_banner_dismissed` | User closes/dismisses a promo banner | ✅ | Web PromoBanner (as promo_dismiss — RENAME) |
| 41 | `app_install_banner_shown` | Smart app install banner displayed | ✅ | Web OpenInAppBanner (as banner_view — RENAME) |
| 42 | `app_install_banner_clicked` | User clicks app install banner | ✅ | Web OpenInAppBanner (as banner_click — RENAME) |
| 43 | `trial_banner_shown` | Trial/distribution CTA visible on page | 📋 | Web (trial takeover components) |
| 44 | `trial_banner_clicked` | User clicks trial CTA | 📋 | Web (trial components) |
| 45 | `trial_banner_dismissed` | User clicks "Later" on trial prompt | 📋 | Web (trial components) |

---

### DOMAIN 3: SEARCH & FILTER (20 events)

| # | Canonical Name | Explanation | Status | Where |
|---|---|---|---|---|
| 46 | `search_focused` | User clicks/taps the search bar (intent signal) | 📋 | Web CanonicalSearchBar |
| 47 | `search_typed` | User is typing in search (for autocomplete analytics) | 📋 | Web CanonicalSearchBar |
| 48 | `search_autocomplete_shown` | Autocomplete suggestions appear | 📋 | Web (autocomplete component) |
| 49 | `autocomplete_selected` | User clicks a suggestion from autocomplete | ✅ | Web search-results-view |
| 50 | `search_performed` | Search submitted and results loaded | ✅ | Web filter.tsx |
| 51 | `search_no_results` | Search returns zero results (critical UX signal) | ✅ | Web filter.tsx (as search_no_results) |
| 52 | `search_result_clicked` | User clicks a result from search results | ✅ | Web productCard |
| 53 | `search_result_scrolled_past` | A result was visible but user scrolled past without clicking | 🔮 | Web (future P2 — negative signal) |
| 54 | `search_filter_applied` | User applies a filter (price, condition, city, etc.) | ✅ | Web filter.tsx |
| 55 | `search_filter_removed` | User removes a previously applied filter | 📋 | Web filter.tsx |
| 56 | `search_sort_changed` | User changes sort order (newest/oldest/price asc/desc) | ✅ | Web search-results-view |
| 57 | `search_pagination` | User loads more results (infinite scroll or page click) | 📋 | Web (search results) |
| 58 | `search_refined` | User changes query after seeing results (reformulation) | 📋 | Web (detect query change) |
| 59 | `saved_search_created` | User saves a search alert for new matches | ✅ | Web search-results-view |
| 60 | `search_image_used` | User initiates camera/image search | ✅ | Web SearchByImageButton |
| 61 | `search_abandoned` | User leaves search without clicking any result | 📋 | Web (detect navigation away) |
| 62 | `darija_query_detected` | System detects Darija dialect and expands query | 📋 | Backend (searchEnrichment) |
| 63 | `darija_query_no_expansion` | Darija detected but no dictionary match found | 📋 | Backend (searchEnrichment) |
| 64 | `category_browsed` | User enters a category browse page (not search) | ✅ | Web search-results-view |
| 65 | `location_filter_changed` | User changes city/region filter | 📋 | Web filter.tsx |

---

### DOMAIN 4: LISTING ENGAGEMENT (25 events)

| # | Canonical Name | Explanation | Status | Where |
|---|---|---|---|---|
| 66 | `listing_viewed` | User opens listing detail page (= ta_publication_view created) | ✅ | Web product-details-view |
| 67 | `listing_dwell_time` | Time spent on listing page (fired on leave/hide) | 📋 | Web (visibilitychange) |
| 68 | `listing_gallery_opened` | User opens fullscreen image viewer | 📋 | Web productDetails |
| 69 | `listing_gallery_swiped` | User browses through listing photos | 📋 | Web productDetails |
| 70 | `listing_image_zoomed` | User pinch-zooms a listing image | ✅ | Web productDetails |
| 71 | `listing_video_played` | User plays listing video | ✅ | Web productDetails |
| 72 | `listing_video_progress` | Video reaches 25/50/75/100% completion | 📋 | Web (timeupdate listener) |
| 73 | `listing_video_completed` | Video plays to 100% | 📋 | Web (ended event) |
| 74 | `listing_description_expanded` | User clicks "Read more" on description | 📋 | Web productDetails |
| 75 | `listing_properties_viewed` | User scrolls to specifications/properties section | 📋 | Web (IntersectionObserver) |
| 76 | `listing_seller_clicked` | User clicks seller name/avatar to view their profile | 📋 | Web productDetails |
| 77 | `listing_delivery_checked` | User views delivery options/estimates | 📋 | Web productDetails |
| 78 | `listing_phone_revealed` | Seller/buyer phone number revealed (after offer accepted or lead unlocked) | 📋 | Backend (offer.service + leads.service) |
| 79 | `listing_favorited` | User adds listing to favorites (heart icon) | ✅ | Web productDetails |
| 80 | `listing_unfavorited` | User removes listing from favorites | 📋 | Web productDetails |
| 81 | `listing_shared` | User shares listing via any channel | ✅ | Web ShareButton (as listing_shared) |
| 82 | `listing_reported` | User reports a listing for policy violation | 📋 | Web (report flow) |
| 83 | `listing_similar_viewed` | User clicks "Similar items" recommendation | 📋 | Web exploreMore |
| 84 | `listing_return_visit` | Same user views same listing 2nd+ time (high intent) | 📋 | Web (detect repeat view) |
| 85 | `listing_price_compared` | User views price comparison (similar items section) | ✅ | Web exploreMore |
| 86 | `listing_contact_initiated` | User clicks any contact button (chat/call/whatsapp) | ✅ | Web productDetails |
| 87 | `callback_requested` | User submits phone via OTP-verified lead form (= ta_lead created) | ✅ | Web LeadCaptureModal (currently as lead_captured — RENAME) |
| 88 | `seller_profile_viewed` | User views a store/seller page | ✅ | Web public-store-view |
| 89 | `store_video_played` | User plays a store intro video | 📋 | Web (store page) |
| 90 | `store_followed` | User follows a store for updates | 🔧 | Web storeHeaderCard |

---

### DOMAIN 5: AUCTION & BIDDING (20 events)

| # | Canonical Name | Explanation | Status | Where |
|---|---|---|---|---|
| 91 | `auction_viewed` | User views an auction listing page (sees countdown, current bid, bidders) | 🔧 | Web auction-view (as auction_watched — RENAME) |
| 92 | `auction_rules_viewed` | User reads auction rules/info (engagement signal) | 🔮 | Web (future) |
| 93 | `auction_bid_history_viewed` | User checks bid history (research behavior) | 📋 | Web (bid history section) |
| 94 | `bid_initiated` | User clicks "Place bid" button (before entering amount) | 📋 | Web (bid UI) |
| 95 | `bid_amount_entered` | User types a bid amount (intent signal) | 🔮 | Web (future P2) |
| 96 | `bid_placed` | Bid confirmed and recorded (= ta_bid_transaction created) | ✅ | Web auction-view + Backend BidTransactionService |
| 97 | `bid_outbid` | User is outbid by someone else (= notification fired) | 📋 | Backend (BidTransactionService → notify previous bidder) |
| 98 | `bid_retaliated` | User bids again immediately after being outbid (<5min) | 📋 | Backend (detect pattern) |
| 99 | `bid_auto_set` | User enables auto-bid with max limit | 📋 | Web (BidEntity update) |
| 100 | `bid_auto_triggered` | System places auto-bid on user's behalf | 📋 | Backend (placeAutomaticBid) |
| 101 | `auction_won` | User wins the auction (= ta_bid_room.winner_id set) | 📋 | Backend (BidRoomService.updateOpenedToClosed → winner) |
| 102 | `auction_lost` | User did not win (= refund triggered) | 📋 | Backend (BidRoomService → notify losers) |
| 103 | `auction_watched` | User adds auction to watchlist / sets reminder | 📋 | Web (future) |
| 104 | `auction_reminder_set` | User sets end-time reminder for auction | 🔮 | Web (future) |
| 105 | `auction_countdown_viewed` | User watches final 60 seconds of auction (high engagement) | 🔮 | Web (future P2) |
| 106 | `auction_sniped` | Bid placed in last 30 seconds of auction | 🔮 | Backend (future P2 — detect pattern) |
| 107 | `auction_revisited` | User returns to same auction multiple times | 🔮 | Web (future P2) |
| 108 | `auction_shared` | User shares auction link | 📋 | Web (share component) |
| 109 | `bid_room_joined` | User pays coins to enter auction (= ta_bid_entity created, coins debited) | 📋 | Backend (BidEntityService.joinBidRoom) |
| 110 | `bid_deposit_refunded` | Non-winner gets coins refunded after auction close | 📋 | Backend (WalletService — source: refundSecurityDeposit) |

---

### DOMAIN 6: OFFERS & NEGOTIATION (15 events)

| # | Canonical Name | Explanation | Status | Where |
|---|---|---|---|---|
| 111 | `offer_form_opened` | Buyer clicks "Make an offer" button (intent signal) | 📋 | Web makeOffer.tsx |
| 112 | `offer_amount_entered` | Buyer types/selects an offer amount | 🔮 | Web (future P2) |
| 113 | `offer_message_added` | Buyer adds a note/message to their offer | 🔮 | Web (future) |
| 114 | `offer_submitted` | Buyer sends offer (= ta_offer created, accepted=null) | ✅ | Web makeOffer + Backend OfferService.create |
| 115 | `offer_viewed_by_seller` | Seller opens the offer notification/detail | 📋 | Backend/Frontend (seller views offer) |
| 116 | `offer_accepted` | Seller accepts offer (= ta_offer.accepted = true) | 🔧 | Backend OfferService.update (fires but may use wrong name) |
| 117 | `offer_rejected` | Seller declines offer (= ta_offer.accepted = false) | ✅ | Web offerItemLigne + Backend |
| 118 | `offer_countered` | Seller sends counter-offer (future — not yet in DB model) | 🔮 | Backend (future feature) |
| 119 | `offer_expired` | Offer times out without response | 🔮 | Backend (future — no expiry in current model) |
| 120 | `offer_cancelled` | Buyer withdraws their offer before response | 🔮 | Web (future) |
| 121 | `negotiation_completed` | Deal reached via any path (accepted or counter-accepted) | 📋 | Backend (derive from offer_accepted) |
| 122 | `callback_requested` | Buyer submits OTP-verified phone for seller to call back (= ta_lead created) | ✅ | Web LeadCaptureModal (currently `lead_captured` — RENAME) |
| 123 | `callback_completed` | Seller called the buyer back (= ta_lead.status → contacted) | 📋 | Backend (LeadsService.markAsContacted) |
| 124 | `meeting_proposed` | Buyer/seller proposes in-person meeting | 🔮 | Future feature |
| 125 | `meeting_confirmed` | Both parties agree on meeting | 🔮 | Future feature |

---

### DOMAIN 7: MESSAGING (12 events)

| # | Canonical Name | Explanation | Status | Where |
|---|---|---|---|---|
| 126 | `chat_initiated` | New message thread opened for the first time | 📋 | Web/Backend (first message in thread) |
| 127 | `message_sent` | User sends a message in any thread | ✅ | Web quickSendMessage |
| 128 | `message_received` | User receives a message (server-side, for response time calc) | 📋 | Backend (MessageService) |
| 129 | `message_read` | Recipient reads message (read receipt) | 📋 | Backend (WebSocket ack) |
| 130 | `chat_media_shared` | User sends image/video in chat | 🔮 | Future |
| 131 | `chat_location_shared` | User shares meeting location in chat | 🔮 | Future |
| 132 | `seller_response_time` | Computed: time between buyer message and seller reply | 📋 | Backend (compute on reply) |
| 133 | `chat_thread_archived` | User archives a conversation | 🔮 | Future |
| 134 | `chat_blocked` | User blocks another user from messaging | 🔮 | Future |
| 135 | `chat_reported` | User reports a conversation for abuse | 🔮 | Future |
| 136 | `chat_unread_count` | Passive: unread badge update (for engagement scoring) | 🔮 | Future P2 |
| 137 | `notification_opened` | User opens app from a message notification | ✅ | Web notification.tsx (as notification_clicked — RENAME) |

---

### DOMAIN 8: ORDERS & TRANSACTIONS (20 events)

| # | Canonical Name | Explanation | Status | Where |
|---|---|---|---|---|
| 138 | `buy_now_clicked` | Buyer clicks "Buy Now" button (START of purchase, before checkout) | 🔧 | Web BuyNowModal (currently fires as `add_to_cart` — RENAME) |
| 139 | `checkout_started` | Buyer enters checkout form (delivery address, method) | 📋 | Web BuyNowModal (after initial click, form renders) |
| 140 | `delivery_method_selected` | Buyer chooses home delivery / relay / in-person | ✅ | Web BuyNowModal |
| 141 | `delivery_rate_fetched` | Tawssil API returns shipping estimate | 📋 | Backend (TawssilService.getRates) |
| 142 | `recipient_info_entered` | Buyer fills name/phone/address for delivery | 🔮 | Web (future P2) |
| 143 | `order_placed` | Order confirmed by buyer (= ta_order created, status: pending) | ✅ | Web BuyNowModal + Backend OrdersService.createOrder |
| 144 | `order_confirmed_by_seller` | Seller accepts and prepares order (= status → confirmed, Tawssil colis created) | 📋 | Backend (OrdersService.confirmOrder) |
| 145 | `order_shipped` | Tawssil picks up package (= status → shipped) | 📋 | Backend (Tawssil webhook: picked_up) |
| 146 | `order_in_transit` | Package moving between hubs (= status → in_transit) | 📋 | Backend (Tawssil webhook: in_transit/out_for_delivery) |
| 147 | `order_out_for_delivery` | Last mile delivery attempt | 📋 | Backend (Tawssil webhook: out_for_delivery) |
| 148 | `order_delivered` | Package delivered (= status → delivered). THIS fires GA4/Meta/TikTok Purchase. | 📋 | Backend (Tawssil webhook: delivered) — GA4+Meta CAPI+TikTok already fire here |
| 149 | `order_returned` | Package returned (= status → returned) | 📋 | Backend (Tawssil webhook: returned) |
| 150 | `order_cancelled` | Order cancelled before shipping (= status → cancelled) | 📋 | Backend (cancelOrder/buyerCancelOrder) |
| 151 | `order_tracking_viewed` | Buyer checks delivery tracking page | 📋 | Web (tracking page) |
| 152 | `review_prompted` | System shows review request after delivery | 🔮 | Future (notification-driven) |
| 153 | `review_submitted` | User writes and submits a review | ✅ | Web addReview |
| 154 | `review_edited` | User modifies an existing review | 🔮 | Future |
| 155 | `cod_payment_collected` | Cash on delivery collected by Tawssil | 🔮 | Backend (future Tawssil status) |
| 156 | `cod_payment_refused` | Customer refuses COD at door | 🔮 | Backend (future) |
| 157 | `guest_order_placed` | Non-registered user places order via OTP-verified phone | 📋 | Backend (GuestOrdersController) |

---

### DOMAIN 9: PAYMENTS & COINS (20 events)

| # | Canonical Name | Explanation | Status | Where |
|---|---|---|---|---|
| 158 | `pricing_page_viewed` | User opens pricing/packages page | 🔧 | Web pricing-plans-page (as package_viewed — RENAME) |
| 159 | `coin_pack_selected` | User clicks a specific coin pack to buy | ✅ | Web coin-packages-view |
| 160 | `checkout_coin_started` | User redirected to Payzone payment gateway | 🔧 | Web coin-package-checkout (as begin_checkout — RENAME) |
| 161 | `payment_initiated` | Payzone payment form loaded | ✅ | Web coin-package-checkout |
| 162 | `payment_3ds_shown` | 3D Secure challenge presented by bank | 🔮 | Web (future — detect 3DS iframe) |
| 163 | `payment_completed` | Payzone confirms successful payment | ✅ | Web payment-status + Backend |
| 164 | `payment_failed` | Payzone declines payment | ✅ | Web payment-status + Backend |
| 165 | `payment_retried` | User retries after failure | 📋 | Web (detect retry) |
| 166 | `coin_purchased` | Coins credited to wallet after payment (= ta_wallet_history, source: coinPackagePurchase) | 📋 | Backend (WalletService.creditWithLock) |
| 167 | `coin_earned` | Any non-purchase coin credit (wheel/referral/giveaway/refund) | 📋 | Backend (WalletService.createHistory where coins > 0 AND source ≠ coinPackagePurchase) |
| 168 | `coin_spent` | Any coin debit (boost/bid_room/subscription/lead_unlock) | 📋 | Backend (WalletService.createHistory where coins < 0) |
| 169 | `boost_purchased` | User boosts a listing (= ta_publication_boost created) | ✅ | Web BoostDistributeModal + Backend |
| 170 | `boost_extended` | User extends an existing boost | ✅ | Web ExtendBoostModal |
| 171 | `subscription_purchased` | User buys distribution package (= ta_syndication_user_subscription created) | 📋 | Backend (SyndicationSubscriptionService) |
| 172 | `subscription_trial_claimed` | User activates 14-day free trial (= ta_trial created) | 📋 | Backend (TrialService.claimTrial) |
| 173 | `subscription_expired` | Package/trial ends (= active=false or trial status=expired) | 📋 | Backend (syndication-expiry processor / trial processor) |
| 174 | `lead_unlocked` | Seller pays coins to see lead contact (= ta_lead_unlock created) | 📋 | Backend (LeadsService.unlockLead) |
| 175 | `lucky_wheel_spun` | User spins daily wheel | ✅ | Web luckyWheel |
| 176 | `wallet_viewed` | User opens wallet/balance page | 📋 | Web (wallet section) |
| 177 | `coin_purchase_abandoned` | User leaves pricing page without buying | 📋 | Web (detect navigation away) |

---

### DOMAIN 10: SELLER LIFECYCLE (20 events)

| # | Canonical Name | Explanation | Status | Where |
|---|---|---|---|---|
| 178 | `listing_creation_started` | User enters the sell/publish flow | ✅ | Web product-form-v2 |
| 179 | `listing_photo_uploaded` | User adds a photo to their listing | 🔧 | Web media-section (as listing_photo_added — RENAME) |
| 180 | `listing_ai_generated` | AI generates title+description from photos | ✅ | Web basic-info-section |
| 181 | `listing_ai_edited` | User modifies AI-generated content | ✅ | Web basic-info-section |
| 182 | `listing_ai_accepted` | User accepts AI output without changes | 📋 | Web AIConfirmationCard (as user_accepted — RENAME) |
| 183 | `listing_category_selected` | User selects category for listing | ✅ | Web category-section |
| 184 | `listing_price_set` | User enters listing price | ✅ | Web pricing-inventory-section |
| 185 | `listing_video_uploaded` | User adds video to listing | ✅ | Web media-section (as publication_video_uploaded) |
| 186 | `listing_step_completed` | User completes a wizard step (photos/category/details/price/review) | 📋 | Web product-form-v2 |
| 187 | `listing_creation_abandoned` | User leaves sell flow without publishing | ✅ | Web product-form-v2 (beforeunload) |
| 188 | `listing_creation_completed` | Listing published successfully (= ta_publication created) | ✅ | Web product-form-v2 |
| 189 | `listing_draft_saved` | User saves listing as draft | 📋 | Web (draft flow) |
| 190 | `listing_edited` | User edits a live listing | ✅ | Web product-form-v2 |
| 191 | `listing_relisted` | User re-lists an expired/unsold listing | 🔮 | Future |
| 192 | `listing_deleted` | User permanently removes a listing | ✅ | Web productDetails |
| 193 | `store_created` | User creates their store page (= ta_entity created) | ✅ | Web WizardCompletionScreen |
| 194 | `store_boost_activated` | User boosts their store (= ta_store_boost created) | ✅ | Web StoreBoostModal |
| 195 | `seller_dashboard_viewed` | Seller opens their management dashboard | 📋 | Web (dashboard home) |
| 196 | `seller_insights_viewed` | Seller checks analytics/performance tab | 📋 | Web (insights section) |
| 197 | `seller_pricing_suggestion_seen` | AI suggests price and seller sees it | 🔮 | Future (price intelligence) |


---

### DOMAIN 11: NOTIFICATIONS & ENGAGEMENT (15 events)

| # | Canonical Name | Explanation | Status | Where |
|---|---|---|---|---|
| 198 | `notification_sent` | Any notification dispatched (push/whatsapp/in_app/email) — server-side | 📋 | Backend (NotificationService) |
| 199 | `notification_delivered` | Confirmed delivery to device/channel | 🔮 | Backend (delivery confirmation) |
| 200 | `notification_opened` | User taps notification and enters app | ✅ | Web notification.tsx (as notification_clicked — RENAME) |
| 201 | `notification_dismissed` | User swipes away / ignores notification | 🔮 | Frontend (future) |
| 202 | `whatsapp_sent` | WhatsApp template dispatched via WABA API | 📋 | Backend (WhatsAppBridgeService) |
| 203 | `whatsapp_delivered` | Meta confirms delivery (webhook status=delivered) | 📋 | Backend (WhatsApp webhook controller) |
| 204 | `whatsapp_read` | Recipient opens WhatsApp message (webhook status=read) | 📋 | Backend (WhatsApp webhook) |
| 205 | `whatsapp_link_clicked` | User clicks CTA button in WhatsApp message | 📋 | Backend (UTM from deep link arrival) |
| 206 | `push_permission_requested` | Browser/app requests push permission | 📋 | Web (Notification.requestPermission) |
| 207 | `push_permission_granted` | User grants push notification permission | 📋 | Web (permission result) |
| 208 | `deep_link_arrived` | User arrives from any external notification/share link with UTM | ✅ | Web AmplitudeInit |
| 209 | `referral_link_shared` | User shares their referral code/link | 📋 | Web (referral UI) |
| 210 | `referral_reward_earned` | Both referrer and referred get coins (= ta_wallet_history, source: referral) | 📋 | Backend (WalletService) |
| 211 | `referred_user_arrived` | New user arrives via referral link | 📋 | Web (detect referral UTM) |
| 212 | `saved_search_match_fired` | A saved search alert matches a new listing (server-side) | 📋 | Backend (SaveSearchService) |

---

### DOMAIN 12: AI & MCP (12 events)

| # | Canonical Name | Explanation | Status | Where |
|---|---|---|---|---|
| 213 | `ai_classification_completed` | Bedrock Nova Lite classifies photo to category | 📋 | Backend (EnrichmentAIService) |
| 214 | `ai_generation_completed` | GPT-4o-mini generates title+description | 📋 | Backend (EnrichmentAIService) |
| 215 | `ai_embedding_indexed` | Titan embeds listing and indexes to OpenSearch | 📋 | Backend (PublicationService publish pipeline) |
| 216 | `ai_content_generated` | Content engine creates blog post (Bedrock) | 📋 | Backend (ContentEngineService) |
| 217 | `mcp_query_received` | ChatGPT calls our MCP server (any tool) | ✅ | MCP server (observability.py) |
| 218 | `mcp_results_returned` | We respond to ChatGPT with results | ✅ | MCP server (observability.py) |
| 219 | `mcp_user_converted` | MCP user visits tawadoo.ma from ChatGPT | 📋 | Web (detect utm_source=chatgpt) |
| 220 | `mcp_listing_served` | A specific listing shown via ChatGPT | ✅ | MCP server (observability.py) |
| 221 | `image_search_performed` | Camera/gallery image search executed | ✅ | Web SearchByImageButton + Backend |
| 222 | `voice_search_performed` | Voice input transcribed and searched | 🔮 | Future (Web Speech API) |
| 223 | `darija_expansion_triggered` | Darija dictionary match improves results | 📋 | Backend (hybrid-search.service) |
| 224 | `bedrock_invocation` | Any Bedrock API call (for cost tracking) | 📋 | Backend (all Bedrock callers) |

---

### DOMAIN 13: SYSTEM & PERFORMANCE (16 events)

| # | Canonical Name | Explanation | Status | Where |
|---|---|---|---|---|
| 225 | `api_request_completed` | API call finishes (for latency tracking) | 🔮 | Backend (future middleware) |
| 226 | `api_error_occurred` | 4xx/5xx response visible to user | 📋 | Backend (exception filter) |
| 227 | `page_performance_measured` | Core Web Vitals captured (LCP, CLS, INP, TTFB) | 📋 | Web (PerformanceObserver) |
| 228 | `javascript_error` | Uncaught frontend exception | 📋 | Web (window.onerror / ErrorBoundary) |
| 229 | `video_buffer_event` | Video playback stalls/buffers | 🔮 | Web (future P2) |
| 230 | `image_load_failed` | Listing image fails to load | 🔮 | Web (future P2) |
| 231 | `offline_detected` | User loses network connection | 🔮 | Web (future) |
| 232 | `slow_interaction` | INP > 200ms detected | 🔮 | Web (future P2) |
| 233 | `feed_health_checked` | Feed regeneration completes (server cron) | 📋 | Backend (FeedGeneratorService) |
| 234 | `feed_fetched_by_platform` | Google/Meta/TikTok fetches our product feed | 📋 | Backend (SyndicationFeedController access log) |
| 235 | `external_api_health` | Periodic health check of Payzone/Tawssil/WhatsApp/OpenSearch | 🔮 | Backend (future health cron) |
| 236 | `cron_job_completed` | Any scheduled job finishes | 📋 | Backend (Bull job completion) |
| 237 | `opensearch_query_executed` | Search engine query for latency/performance tracking | 📋 | Backend (OpenSearchService) |
| 238 | `redis_cache_hit_rate` | Periodic cache efficiency stats | 🔮 | Backend (future) |
| 239 | `ecs_task_health` | ECS task CPU/memory metrics | 🔮 | CloudWatch (not app-level) |
| 240 | `deployment_completed` | New code deployed to staging/prod | 🔮 | CI/CD (GitHub Actions) |

---

### DOMAIN 14: CONVERSATIONAL AI & WIDGET (15 events)

| # | Canonical Name | Explanation | Status | Where |
|---|---|---|---|---|
| 241 | `widget_opened` | User opens AI chat widget on site | 🔮 | Future (widget not built) |
| 242 | `widget_message_sent` | User types question to AI assistant | 🔮 | Future |
| 243 | `widget_response_displayed` | AI responds with listings/answers | 🔮 | Future |
| 244 | `widget_listing_clicked` | User clicks a suggested listing from AI response | 🔮 | Future |
| 245 | `widget_action_taken` | User performs action via widget (offer/bid/buy/contact) | 🔮 | Future |
| 246 | `widget_checkout_initiated` | User starts purchase from within widget | 🔮 | Future |
| 247 | `widget_session_duration` | Widget closed — total duration and messages | 🔮 | Future |
| 248 | `widget_seller_assisted` | Seller uses AI to draft response to buyer | 🔮 | Future |
| 249 | `widget_negotiation_assisted` | AI suggests offer amount to buyer | 🔮 | Future |
| 250 | `widget_voice_input_used` | Voice-to-text used in widget | 🔮 | Future |
| 251 | `acp_checkout_started` | ChatGPT initiates ACP purchase flow | 🔮 | Future (ACP partner program) |
| 252 | `acp_checkout_completed` | Purchase completed entirely in ChatGPT | 🔮 | Future |
| 253 | `acp_checkout_abandoned` | ACP checkout dropped before completion | 🔮 | Future |
| 254 | `widget_feedback_given` | User rates AI response (helpful/not helpful) | 🔮 | Future |
| 255 | `widget_escalated_to_human` | AI can't help, connects to real seller | 🔮 | Future |

---

### DOMAIN 15: DISPLAY ADS INTELLIGENCE (15 events)

| # | Canonical Name | Explanation | Status | Where |
|---|---|---|---|---|
| 256 | `display_ad_impression` | Ad enters viewport (ta_sponsor_event type=impression) | ✅ | Web DisplayAdBanner |
| 257 | `display_ad_viewed` | Ad visible 1s+ (viewability metric) | 📋 | Web (IntersectionObserver timer) |
| 258 | `display_ad_click` | User clicks display ad (ta_sponsor_event type=click) | ✅ | Web DisplayAdBanner |
| 259 | `display_ad_converted` | Click leads to action on advertiser's listing | 🔮 | Backend (attribution — future) |
| 260 | `display_ad_dismissed` | User closes/hides an ad | 🔮 | Web (future) |
| 261 | `ad_campaign_created` | Advertiser creates campaign (BO) | 📋 | Backend (DisplayAdsService) |
| 262 | `ad_campaign_paused` | Campaign paused | 🔮 | Backend (future self-serve) |
| 263 | `ad_budget_exhausted` | Campaign budget depleted | 🔮 | Backend (future) |
| 264 | `ad_creative_performance` | Hourly creative stats (impressions, clicks, CTR) | 🔮 | Backend (future cron) |
| 265 | `ad_targeting_matched` | User matches campaign targeting criteria | 🔮 | Backend (future) |
| 266 | `ad_auction_won` | Internal auction picks which ad to show | 🔮 | Backend (future RTB) |
| 267 | `ad_revenue_attributed` | Revenue confirmed from ad interaction | 🔮 | Backend (future attribution) |
| 268 | `ad_slot_filled` | An ad slot has a campaign to display | 🔮 | Backend (future) |
| 269 | `ad_slot_empty` | No campaign for a slot (shows house ad or nothing) | 🔮 | Web (fallback detection) |
| 270 | `ad_report_generated` | Advertiser views performance report | 🔮 | BO (future) |

---

### DOMAIN 16: BLOG & CONTENT ENGINE (10 events)

| # | Canonical Name | Explanation | Status | Where |
|---|---|---|---|---|
| 271 | `blog_post_viewed` | User reads a blog article (page /b/{slug}) | 📋 | Web (blog page) |
| 272 | `blog_scroll_depth` | User scrolls 50/100% of article | 📋 | Web (IntersectionObserver) |
| 273 | `blog_cta_clicked` | User clicks internal link/CTA within blog article | 📋 | Web (blog links) |
| 274 | `blog_listing_clicked` | User clicks an embedded listing from blog | 📋 | Web (blog listing cards) |
| 275 | `blog_shared` | User shares blog post | 🔮 | Web (future) |
| 276 | `programmatic_page_viewed` | User views city×category SEO page | 📋 | Web (browse/ routes) |
| 277 | `programmatic_page_listing_clicked` | User clicks listing from programmatic page | 📋 | Web (browse/ listing cards) |
| 278 | `content_generated` | AI creates blog/page content (server) | 📋 | Backend (ContentEngineService) |
| 279 | `content_indexed` | Google/Bing indexes a new page (from GSC data) | 🔮 | External (data lake pull) |
| 280 | `seo_landing_converted` | Organic visitor takes meaningful action | 📋 | Web (detect organic referrer + action) |

---

### DOMAIN 17: CONVERSATIONS & SOCIAL (12 events)

| # | Canonical Name | Explanation | Status | Where |
|---|---|---|---|---|
| 281 | `review_helpful_marked` | User marks a review as helpful | 🔮 | Future |
| 282 | `seller_followed` | User follows a seller/store for updates | 🔧 | Web storeHeaderCard (as store_followed) |
| 283 | `seller_unfollowed` | User unfollows a store | 📋 | Web (unfollow action) |
| 284 | `collection_created` | User creates a wishlist/collection | 🔮 | Future |
| 285 | `collection_shared` | User shares a collection | 🔮 | Future |
| 286 | `user_reported` | User reports another user for policy violation | 🔮 | Future |
| 287 | `user_blocked` | User blocks another user | 🔮 | Future |
| 288 | `dispute_opened` | Buyer files dispute on order | 🔮 | Future |
| 289 | `dispute_resolved` | Dispute resolved (buyer favor/seller favor) | 🔮 | Future |
| 290 | `seller_response_rated` | Buyer rates seller communication quality | 🔮 | Future |
| 291 | `trust_badge_earned` | User earns verification badge (phone/email/ID) | 🔮 | Future |
| 292 | `social_share_conversion` | Someone clicks a shared link AND takes action | 📋 | Web (detect share UTM + action) |

---

### DOMAIN 18: LIFECYCLE & RETENTION (12 events)

| # | Canonical Name | Explanation | Status | Where |
|---|---|---|---|---|
| 293 | `subscription_renewed` | User renews distribution package (consecutive period) | 📋 | Backend (SyndicationSubscriptionService) |
| 294 | `subscription_upgraded` | User moves to higher tier (Elan→Portee→Rayonnement) | 📋 | Backend |
| 295 | `subscription_downgraded` | User moves to lower tier | 🔮 | Backend (future) |
| 296 | `subscription_cancelled` | User explicitly cancels (before natural expiry) | 📋 | Backend |
| 297 | `boost_renewed` | User re-boosts after expiry | 📋 | Backend (detect re-boost same listing) |
| 298 | `boost_expired` | Boost period ends without renewal | 📋 | Backend (boost-expiry processor) |
| 299 | `user_reactivated` | User returns after 14+ days inactive | 📋 | Backend (detect session gap) |
| 300 | `user_churned` | 30 days no session (computed server-side) | 🔮 | Backend (future daily cron) |
| 301 | `win_back_notification_sent` | Re-engagement notification dispatched | 🔮 | Backend (future) |
| 302 | `win_back_converted` | Churned user returns and takes action | 🔮 | Backend (future) |
| 303 | `daily_streak_maintained` | User logs in consecutive days | 🔮 | Future (gamification) |
| 304 | `milestone_achieved` | User hits platform milestone (first sale, 10 listings, etc.) | 🔮 | Future |

---

### DOMAIN 19: AI SEARCH ENGINE — SELF-IMPROVING (8 events)

| # | Canonical Name | Explanation | Status | Where |
|---|---|---|---|---|
| 305 | `search_click_signal` | User clicks a result after search (POSITIVE relevance signal) | 📋 | Backend (combine search_performed + search_result_clicked) |
| 306 | `search_skip_signal` | Result visible but not clicked (NEGATIVE signal) | 🔮 | Web (future — IntersectionObserver + no-click detection) |
| 307 | `search_purchase_signal` | Full loop: search → click → purchase completed | 📋 | Backend (derive from order with search session) |
| 308 | `search_relevance_feedback` | Implicit: dwell > 30s = relevant, bounce < 5s = irrelevant | 🔮 | Backend (future — combine dwell with source) |
| 309 | `search_query_expansion_result` | Darija/synonym expansion improved results (more clicks post-expansion) | 🔮 | Backend (future analytics) |
| 310 | `search_ranking_served` | The exact ranking shown to user (for Learning-to-Rank training) | 🔮 | Backend (future — log ranked IDs per query) |
| 311 | `search_reranking_candidate` | A listing was re-ranked (boosted/promoted) | 🔮 | Backend (future) |
| 312 | `search_model_evaluation` | Offline eval of search quality (NDCG, MRR, precision) | 🔮 | Backend (future batch job) |

---

### DOMAIN 20: AUTONOMOUS SELLER AGENT (15 events)

| # | Canonical Name | Explanation | Status | Where |
|---|---|---|---|---|
| 313 | `seller_agent_auto_responded` | AI auto-replies to buyer message on seller's behalf | 🔮 | Future (Rayonnement tier) |
| 314 | `seller_agent_offer_handled` | AI accepts/counters/rejects offer within seller's bounds | 🔮 | Future |
| 315 | `seller_agent_override` | Seller corrects AI action (= training signal for RLHF) | 🔮 | Future |
| 316 | `seller_agent_boost_triggered` | AI auto-boosts listing based on demand signals | 🔮 | Future |
| 317 | `seller_agent_price_suggested` | AI suggests price adjustment based on market data | 🔮 | Future |
| 318 | `seller_agent_listing_relisted` | AI re-lists expired listing with optimized content | 🔮 | Future |
| 319 | `seller_agent_demand_alert_sent` | AI alerts seller of rising demand in their category | 🔮 | Future |
| 320 | `seller_agent_whatsapp_replied` | AI replies to buyer WhatsApp on seller's behalf | 🔮 | Future |
| 321 | `seller_agent_order_prepared` | AI prepares order details for seller's one-tap confirm | 🔮 | Future |
| 322 | `seller_agent_review_solicited` | AI sends review request at optimal time post-delivery | 🔮 | Future |
| 323 | `seller_agent_competitor_alert` | AI alerts seller of lower-priced competitor listing | 🔮 | Future |
| 324 | `seller_agent_performance_report` | AI generates periodic seller performance summary | 🔮 | Future |
| 325 | `seller_agent_activated` | Seller enables AI agent (configures rules/bounds) | 🔮 | Future |
| 326 | `seller_agent_deactivated` | Seller turns off AI agent | 🔮 | Future |
| 327 | `seller_agent_learning_signal` | Each seller override = RLHF training data point | 🔮 | Future |

---

### DOMAIN 21: BUYER AGENT & VOICE/IMAGE (12 events)

| # | Canonical Name | Explanation | Status | Where |
|---|---|---|---|---|
| 328 | `buyer_agent_search_conversational` | User searches via natural language (widget/voice) | 🔮 | Future |
| 329 | `buyer_agent_recommendation_shown` | AI-powered personalized recommendation displayed | 🔮 | Future |
| 330 | `buyer_agent_recommendation_clicked` | User clicks AI recommendation | 🔮 | Future |
| 331 | `buyer_agent_price_alert_triggered` | Price drops below user's target → notification | 🔮 | Future |
| 332 | `buyer_agent_negotiation_assisted` | AI suggests offer amount to buyer | 🔮 | Future |
| 333 | `buyer_agent_trust_check_performed` | AI evaluates seller trust before buyer commits | 🔮 | Future |
| 334 | `voice_search_initiated` | User taps microphone for voice search | 🔮 | Future (Web Speech API) |
| 335 | `voice_search_results_spoken` | Results read aloud to user | 🔮 | Future |
| 336 | `image_search_initiated` | User uses camera/gallery for image search | ✅ | Web SearchByImageButton |
| 337 | `image_search_results_shown` | Visual search results displayed | ✅ | Web SearchByImageButton (as image_search_completed) |
| 338 | `buyer_agent_whatsapp_query` | Buyer sends search query via WhatsApp to AI | 🔮 | Future |
| 339 | `buyer_mood_predicted` | AI predicts user intent/mood from behavior signals | 🔮 | Future (ML model) |

---

### DOMAIN 22: RETAIL MEDIA / ADVERTISER AGENT (10 events)

| # | Canonical Name | Explanation | Status | Where |
|---|---|---|---|---|
| 340 | `advertiser_campaign_ai_optimized` | AI adjusts campaign targeting/budget automatically | 🔮 | Future |
| 341 | `advertiser_audience_auto_refreshed` | AI updates audience criteria based on performance | 🔮 | Future |
| 342 | `advertiser_creative_auto_paused` | AI pauses underperforming creative | 🔮 | Future |
| 343 | `advertiser_budget_auto_reallocated` | AI moves budget from low-ROAS to high-ROAS campaign | 🔮 | Future |
| 344 | `advertiser_roi_milestone` | Campaign hits ROAS milestone (2x, 5x, 10x) | 🔮 | Future |
| 345 | `advertiser_insight_generated` | AI generates actionable insight for advertiser | 🔮 | Future |
| 346 | `ad_internal_auction_completed` | Internal RTB auction picks winning campaign for slot | 🔮 | Future |
| 347 | `ad_predictive_ctr_scored` | ML model predicts CTR for campaign×user×slot | 🔮 | Future |
| 348 | `retail_media_revenue_daily` | Daily revenue rollup from all ad campaigns | 🔮 | Future (daily cron) |
| 349 | `advertiser_self_serve_onboarded` | New advertiser creates first campaign | 🔮 | Future |

---

### DOMAIN 23: PREDICTIVE INTELLIGENCE (8 events)

| # | Canonical Name | Explanation | Status | Where |
|---|---|---|---|---|
| 350 | `prediction_conversion_scored` | ML scores user's probability of converting | 🔮 | Future (ML pipeline) |
| 351 | `prediction_churn_scored` | ML scores user's churn risk | 🔮 | Future |
| 352 | `prediction_ltv_computed` | ML computes user's predicted lifetime value | 🔮 | Future |
| 353 | `prediction_price_suggested` | ML suggests optimal listing price from market data | 🔮 | Future |
| 354 | `prediction_demand_forecasted` | ML forecasts demand by category×city | 🔮 | Future |
| 355 | `prediction_fraud_flagged` | ML flags suspicious listing/user for review | 🔮 | Future |
| 356 | `prediction_notification_timed` | ML picks optimal send time per user | 🔮 | Future |
| 357 | `prediction_match_scored` | ML scores buyer×listing match quality | 🔮 | Future |

---

### ADDITIONAL EVENTS (from V2 Vision — Domains 20-23 extensions)

| # | Canonical Name | Explanation | Status |
|---|---|---|---|
| 358-372 | Reserved for agent-specific events discovered during implementation | — | 🔮 |

---

## SECTION 9: IMPLEMENTATION STATUS SUMMARY

| Status | Count | Meaning |
|---|---|---|
| ✅ LIVE | ~55 | Currently firing in staging (some with wrong names — fix per Section 1) |
| 🔧 WIRED | ~15 | Code exists but needs rename or deployment |
| 📋 PLANNED | ~85 | System exists in DB/API, needs instrumentation (Session 14D + 16) |
| 🔮 FUTURE | ~217 | System not yet built (agents, widget, ML, advanced features) |
| **TOTAL** | **372** | Complete V2 taxonomy |

**IMMEDIATE WORK (Sessions 14D + server wiring):**
- Fix 11 duplicate/legacy names (KILL old, keep P0)
- Rename 9 mismatched events to canonical names
- Add 18 critical server-side events (order lifecycle, auction lifecycle, lead lifecycle, wallet events)
- Add ~67 more planned events across existing systems (P1 behavioral depth)
- = ~105 events to add/fix in next 2 sessions

**AFTER THAT (Session 16+):**
- Display ads instrumentation (~5 events on existing module)
- Blog/content tracking (~10 events)
- Performance monitoring (~8 events)
- = ~23 more events on existing code

**FUTURE (when systems are built):**
- Widget (15 events) — when widget is developed
- Seller Agent (15 events) — when Rayonnement agent launches
- Buyer Agent (12 events) — when conversational AI launches
- Advertiser Agent (10 events) — when retail media platform launches
- Predictive Intelligence (8 events) — when ML models train
- Self-improving search (8 events) — when L2R model deploys
- = ~68 events that come with new feature launches


---

## SECTION 10: THE 25 FUNNELS (canonical event chains)

Each funnel uses ONLY names from this registry. Copy-paste into Amplitude.

### FUNNEL 1: SIGNUP
```
session_started → auth_gate_triggered → signup_started → signup_otp_requested → signup_otp_verified → signup_completed
```

### FUNNEL 2: BUYER PURCHASE (Buy-Now)
```
listing_impression → listing_card_clicked → listing_viewed → listing_dwell_time (>10s) → buy_now_clicked → checkout_started → delivery_method_selected → order_placed → order_confirmed_by_seller → order_shipped → order_delivered → review_submitted
```

### FUNNEL 3: BUYER AUCTION
```
auction_viewed → bid_room_joined → bid_placed → bid_outbid → bid_placed (retaliation) → auction_won → order_placed → order_delivered
```

### FUNNEL 4: BUYER NEGOTIATION (Offer)
```
listing_viewed → offer_form_opened → offer_submitted → offer_viewed_by_seller → offer_accepted → listing_phone_revealed
```

### FUNNEL 5: SELLER ACTIVATION
```
signup_completed → listing_creation_started → listing_photo_uploaded → listing_ai_generated → listing_category_selected → listing_price_set → listing_creation_completed → lead_received (first) → order_placed (first sale on their listing)
```

### FUNNEL 6: SELLER MONETIZATION
```
listing_creation_completed → pricing_page_viewed → coin_pack_selected → checkout_coin_started → payment_completed → coin_purchased → boost_purchased OR subscription_purchased
```

### FUNNEL 7: SEARCH-TO-CONVERSION
```
search_performed → search_result_clicked → listing_viewed → listing_dwell_time → listing_contact_initiated OR offer_submitted OR bid_placed OR buy_now_clicked
```

### FUNNEL 8: TRIAL CONVERSION
```
trial_banner_shown → trial_banner_clicked → auth_gate_triggered → signup_completed → subscription_trial_claimed → listing_creation_completed (during trial) → subscription_purchased (after trial)
```

### FUNNEL 9: NOTIFICATION ENGAGEMENT
```
notification_sent → notification_opened → deep_link_arrived → listing_viewed OR auction_viewed → order_placed OR offer_submitted OR bid_placed OR boost_purchased
```

### FUNNEL 10: REFERRAL VIRAL LOOP
```
referral_link_shared → referred_user_arrived → signup_completed → listing_creation_completed OR order_placed → referral_reward_earned → referral_link_shared (next generation)
```

### FUNNEL 11: LEAD GENERATION (vehicles/real estate)
```
listing_impression → listing_viewed → listing_contact_initiated → callback_requested → lead_received → lead_viewed_by_seller → lead_unlocked → lead_contacted → lead_converted
```

### FUNNEL 12: CHATGPT/MCP
```
mcp_query_received → mcp_results_returned → mcp_user_converted (visits site) → listing_viewed → offer_submitted OR buy_now_clicked → order_placed
```

### FUNNEL 13: WIDGET CONVERSATIONAL (future)
```
widget_opened → widget_message_sent → widget_response_displayed → widget_listing_clicked → widget_action_taken → order_placed
```

### FUNNEL 14: DISPLAY ADS
```
display_ad_impression → display_ad_viewed (1s+) → display_ad_click → listing_viewed → offer_submitted OR buy_now_clicked → display_ad_converted
```

### FUNNEL 15: CONTENT/SEO
```
blog_post_viewed → blog_scroll_depth (>50%) → blog_cta_clicked → listing_viewed → listing_contact_initiated → order_placed
```

### FUNNEL 16: SUBSCRIPTION LIFECYCLE
```
subscription_purchased → listing_creation_completed (within subscription) → lead_received → subscription_renewed OR subscription_expired → subscription_cancelled OR subscription_upgraded
```

### FUNNEL 17: BOOST LIFECYCLE
```
boost_purchased → listing_impression (during boost) → listing_viewed → listing_contact_initiated → boost_expired → boost_renewed OR boost_expired (no renew)
```

### FUNNEL 18: WIN-BACK
```
user_churned → win_back_notification_sent → notification_opened → session_started → user_reactivated → listing_creation_completed OR order_placed
```

### FUNNEL 19: AI SEARCH SELF-IMPROVEMENT
```
search_performed → search_ranking_served → search_click_signal (positive) + search_skip_signal (negative) → search_relevance_feedback → search_model_evaluation (weekly)
```

### FUNNEL 20: ACP CHECKOUT (ChatGPT native purchase — future)
```
mcp_query_received → mcp_results_returned → acp_checkout_started → acp_checkout_completed → order_delivered
```

### FUNNEL 21: SELLER AGENT EFFECTIVENESS (future)
```
seller_agent_activated → seller_agent_auto_responded → seller_agent_offer_handled → seller_agent_override (learning) → order_placed (attributed to agent)
```

### FUNNEL 22: BUYER AGENT CONVERSION (future)
```
widget_opened OR voice_search_initiated → buyer_agent_search_conversational → buyer_agent_recommendation_shown → buyer_agent_recommendation_clicked → order_placed
```

### FUNNEL 23: ADVERTISER SELF-SERVE (future)
```
advertiser_self_serve_onboarded → ad_campaign_created → display_ad_impression → display_ad_click → display_ad_converted → advertiser_roi_milestone
```

### FUNNEL 24: PREDICTIVE ACTION LOOP (future)
```
prediction_conversion_scored → notification_sent (intervention) → notification_opened → order_placed (outcome matched prediction)
```

### FUNNEL 25: CROSS-CHANNEL COMMERCE (future)
```
mcp_query_received (ChatGPT) → acp_checkout_started → payment_completed → order_shipped → order_delivered → review_submitted
```

---

## SECTION 11: PROPERTIES STANDARD (what goes WITH each event)

### Universal Properties (EVERY event carries these automatically):
- `timestamp` — ISO 8601
- `session_id` — Amplitude session
- `user_id` — entity UUID (if authenticated)
- `device_id` — Amplitude device ID
- `platform` — web/ios/android
- `locale` — ar/fr/en
- `page_type` — homepage/search/listing/dashboard/store/checkout/etc.

### Transaction Properties (orders, offers, bids, leads):
- `source_channel` — direct/google/meta/tiktok/chatgpt (from DB entity)
- `utm_source`, `utm_medium`, `utm_campaign` — attribution
- `fbclid`, `ttclid` — platform click IDs for conversion matching

### Listing Properties (any event involving a listing):
- `listing_id` — ta_publication UUID
- `category` — category code
- `price` — listing price in MAD
- `city` — listing city
- `seller_id` — seller entity UUID
- `listing_type` — buy_now/lead_gen/rental/auction/negotiable
- `has_video` — boolean
- `is_boosted` — boolean

### Coin Properties (any wallet event):
- `coins_amount` — positive or negative
- `source` — matches ta_wallet_history.source exactly (coinPackagePurchase/lead_unlock/joinBidRoom/refundSecurityDeposit/boost/subscription/wheel/referral/giveaway)
- `balance_after` — wallet balance post-transaction

---

*END OF DOCUMENT. This is the SINGLE SOURCE OF TRUTH. 372 events. 25 funnels. 23 domains. 12 naming rules. Implementation status on every line. When in doubt: this file wins.*

*Last updated: Aug 20, 2026. Authors: Ramzi Hannachi (CEO) + CTO Pilot (Kiro Brain 6).*
