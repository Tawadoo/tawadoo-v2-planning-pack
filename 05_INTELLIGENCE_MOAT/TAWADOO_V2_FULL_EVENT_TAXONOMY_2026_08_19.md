# TAWADOO V2 — FULL EVENT TAXONOMY + FUNNELS (200+ Events, 12 Funnels)

> **THE DESTINATION:** 200+ tracked events + 50 derived signals + 12 measurable funnels
> **THE PATH:** Session 14A (55 P0) → Session 14B (55 P1) → Session 14C (50 P2) → Session 16 (40 system/server)
> **Naming:** `object_action` (snake_case). Past tense for completed. Present for in-progress.
> **Properties:** Every event carries: `timestamp, session_id, user_id (if authed), device, platform, locale`

---

## DOMAIN 1: IDENTITY & SESSION (20 events)

| # | Event | Trigger | Properties | Phase | Funnel |
|---|---|---|---|---|---|
| 1 | `session_started` | New session begins | session_number, referrer, utm_*, landing_page, device, os, screen | P0 | All |
| 2 | `session_heartbeat` | Every 30s active | time_on_page, scroll_depth, tab_focused, idle_seconds | P2 | Engagement |
| 3 | `session_ended` | Tab close / timeout | duration_seconds, pages_viewed, events_count, converted | P1 | All |
| 4 | `identify` | Auth state set | user_id, has_store, locale, city, signup_method, email_verified, coin_balance, subscription_tier | P0 | All |
| 5 | `signup_started` | Click register / gate opens for signup | method, referrer, landing_page, gate_context | P0 | Signup |
| 6 | `signup_otp_requested` | Send OTP | channel (sms/whatsapp), retry_count | P0 | Signup |
| 7 | `signup_otp_verified` | OTP success | attempts, time_to_verify_seconds | P0 | Signup |
| 8 | `signup_completed` | Account created | method, total_time_seconds, referral_code, ab_variant | P0 | Signup |
| 9 | `login_started` | Open login form | method, source_page | P1 | Retention |
| 10 | `login_completed` | Auth success | method, days_since_last_login, new_device | P0 | Retention |
| 11 | `login_failed` | Auth failure | method, failure_reason, attempt_number | P1 | Retention |
| 12 | `logout` | Click logout | session_duration, pages_viewed | P1 | — |
| 13 | `auth_gate_shown` | Gate renders | context (10 values), listing_id, page_type | P0 | Conversion |
| 14 | `auth_gate_interaction_started` | First keystroke/click in gate | context, method | P1 | Conversion |
| 15 | `auth_gate_converted` | Auth success from gate | context, duration_ms, method | P0 | Conversion |
| 16 | `auth_gate_abandoned` | Gate dismissed | context, duration_ms, had_input | P0 | Conversion |
| 17 | `profile_viewed` | Visit own/other profile | own_profile, sections_viewed | P2 | — |
| 18 | `profile_edited` | Save changes | fields_changed[], completion_pct | P2 | Seller |
| 19 | `social_auth_clicked` | Click Google/Facebook/Apple | provider, source_page | P1 | Signup |
| 20 | `account_deleted` | Delete account | reason, tenure_days, lifetime_value | P2 | Churn |

---

## DOMAIN 2: DISCOVERY & BROWSE (25 events)

| # | Event | Trigger | Properties | Phase | Funnel |
|---|---|---|---|---|---|
| 21 | `page_viewed` | Any page load | page_type, path, locale, referrer, device | P0 | All |
| 22 | `homepage_section_viewed` | Section enters viewport | section_name (trending/categories/stores/money), position | P1 | Discovery |
| 23 | `category_clicked` | Click category icon/card | category_id, category_name, position, source_page | P0 | Discovery |
| 24 | `subcategory_selected` | Drill into subcategory | subcategory_id, parent_category, depth | P1 | Discovery |
| 25 | `listing_impression` | Card visible 1s (IntersectionObserver) | listing_id, position, page_type, source (search/feed/home/category) | P0 | Discovery |
| 26 | `listing_card_clicked` | Click card anywhere | listing_id, position, page_type, click_area (image/title/price) | P0 | Buyer |
| 27 | `listing_card_wishlisted` | Heart icon from card | listing_id, page_type, position | P1 | Buyer |
| 28 | `listing_card_shared` | Share from card | listing_id, method, page_type | P2 | Viral |
| 29 | `feed_entered` | Open feed mode | context (search/browse), total_listings, source | P0 | Feed |
| 30 | `feed_slide_changed` | Swipe in feed | listing_id, index, direction, time_on_prev_slide | P1 | Feed |
| 31 | `feed_exited` | Leave feed | items_viewed, duration_seconds, actions_taken | P1 | Feed |
| 32 | `feed_action_taken` | Any CTA in feed | action (contact/offer/bid/buy/share/favorite), listing_id | P0 | Feed |
| 33 | `grid_feed_toggled` | Switch view mode | from_mode, to_mode, device | P1 | UX |
| 34 | `page_scroll_depth` | 25/50/75/100% milestone | page_type, depth_pct, time_to_depth_ms | P1 | Engagement |
| 35 | `section_impression` | Homepage section visible | section_name, fallback, position | P1 | Discovery |
| 36 | `trending_item_clicked` | Click in trending | listing_id, position, algorithm | P2 | Discovery |
| 37 | `store_card_clicked` | Click store in discovery | entity_id, source_page, position | P1 | Discovery |
| 38 | `promo_banner_viewed` | Ad/promo enters viewport | slot, creative_id, campaign_id | P1 | Ads |
| 39 | `promo_banner_clicked` | Click promo | slot, creative_id, campaign_id, destination | P1 | Ads |
| 40 | `promo_banner_dismissed` | Close promo | slot, creative_id, time_visible | P2 | Ads |
| 41 | `app_install_banner_shown` | Open-in-app banner | platform, page_type | P2 | App |
| 42 | `app_install_banner_clicked` | Click open app | platform | P2 | App |
| 43 | `trial_banner_shown` | Trial CTA visible | location, variant | P1 | Trial |
| 44 | `trial_banner_clicked` | Click trial CTA | location | P1 | Trial |
| 45 | `trial_banner_dismissed` | Click "Later" | location, times_shown | P2 | Trial |

---

## DOMAIN 3: SEARCH & FILTER (20 events)

| # | Event | Trigger | Properties | Phase | Funnel |
|---|---|---|---|---|---|
| 46 | `search_focused` | Click search bar | search_bar_location, time_since_page_load | P2 | Search |
| 47 | `search_typed` | Typing in search | query_length, typing_speed, backspace_count | P2 | Search |
| 48 | `search_autocomplete_shown` | Suggestions appear | suggestions_count, query_prefix | P1 | Search |
| 49 | `search_autocomplete_clicked` | Click suggestion | suggestion_text, position, query_typed | P1 | Search |
| 50 | `search_performed` | Submit/results load | query, results_count, category, city, has_filters, locale, is_darija, execution_ms | P0 | Search |
| 51 | `search_zero_results` | 0 results | query, category, city, suggested_alternatives | P0 | Search |
| 52 | `search_result_clicked` | Click result in list | listing_id, position, query, time_since_results | P0 | Search |
| 53 | `search_result_scrolled_past` | Result visible but not clicked | listing_id, position, time_visible | P2 | Search |
| 54 | `search_filter_applied` | Apply filter | filter_type, filter_value, results_before, results_after | P0 | Search |
| 55 | `search_filter_removed` | Remove filter | filter_type, time_since_applied | P2 | Search |
| 56 | `search_sort_changed` | Change sort order | sort_value, previous_sort, results_count | P1 | Search |
| 57 | `search_pagination` | Load more / next page | page_number, results_shown_so_far, query | P1 | Search |
| 58 | `search_refined` | Change query after results | original_query, new_query, results_delta | P1 | Search |
| 59 | `search_saved` | Save search alert | query, filters, alert_frequency | P1 | Search |
| 60 | `search_image_used` | Camera/image search | results_count, top_category_matched | P1 | Search |
| 61 | `search_abandoned` | Leave search without clicking result | query, results_count, time_on_page | P2 | Search |
| 62 | `darija_query_detected` | Expansion triggered | original_query, expanded_terms, dictionary_match | P1 | AI |
| 63 | `darija_query_no_expansion` | Darija detected but no match | query, language_confidence | P2 | AI |
| 64 | `category_browse_started` | Enter category page | category_id, total_results, source | P1 | Discovery |
| 65 | `location_filter_changed` | Change city/region | city, previous_city, results_delta | P1 | Search |

---

## DOMAIN 4: LISTING ENGAGEMENT (25 events)

| # | Event | Trigger | Properties | Phase | Funnel |
|---|---|---|---|---|---|
| 66 | `listing_viewed` | Detail page load | listing_id, category, price, source, seller_id, has_video, city, condition | P0 | Buyer |
| 67 | `listing_dwell_time` | Leave listing page | listing_id, seconds, scrolled_to_bottom, actions_taken[], images_viewed | P0 | Buyer |
| 68 | `listing_gallery_opened` | Open image fullscreen | listing_id, image_index | P1 | Buyer |
| 69 | `listing_gallery_swiped` | Browse images | listing_id, images_viewed, total_images, time_per_image_avg | P1 | Buyer |
| 70 | `listing_gallery_zoomed` | Pinch-zoom image | listing_id, image_index, zoom_duration | P2 | Buyer |
| 71 | `listing_video_played` | Click play | listing_id, video_duration, autoplay, source | P0 | Buyer |
| 72 | `listing_video_progress` | 25/50/75/100% watched | listing_id, progress_pct, watch_time_seconds | P1 | Buyer |
| 73 | `listing_video_completed` | Video finished | listing_id, total_duration, replayed | P1 | Buyer |
| 74 | `listing_description_expanded` | Click "Read more" | listing_id, description_length | P2 | Buyer |
| 75 | `listing_properties_viewed` | Scroll to specs | listing_id, properties_visible | P2 | Buyer |
| 76 | `listing_seller_clicked` | Click seller name/avatar | listing_id, seller_id, seller_rating | P1 | Trust |
| 77 | `listing_delivery_checked` | View delivery options | listing_id, delivery_available, estimated_cost | P1 | Buyer |
| 78 | `listing_phone_revealed` | Reveal seller phone | listing_id, seller_id | P0 | Lead |
| 79 | `listing_bookmarked` | Save to favorites | listing_id, source_page, wishlist_count_after | P0 | Buyer |
| 80 | `listing_unbookmarked` | Remove from favorites | listing_id, time_in_wishlist | P2 | Buyer |
| 81 | `listing_shared` | Share listing | listing_id, channel (whatsapp/copy/native), page_type | P0 | Viral |
| 82 | `listing_reported` | Report listing | listing_id, reason, description_length | P2 | Safety |
| 83 | `listing_similar_viewed` | Click "Similar items" | listing_id, similar_listing_id, position | P2 | Discovery |
| 84 | `listing_return_visit` | Same user views same listing 2nd+ time | listing_id, visit_number, days_since_first | P2 | Intent |
| 85 | `listing_price_comparison` | View 3+ in same category in one session | category, listings_viewed[], price_range | P2 | Intent |
| 86 | `listing_contact_initiated` | Click contact button | listing_id, seller_id, method (chat/call/whatsapp), gate_context | P0 | Lead |
| 87 | `listing_callback_requested` | Request callback form | listing_id, buyer_phone_prefix | P0 | Lead |
| 88 | `store_viewed` | Store page load | entity_id, source, listing_count, has_video, is_premium | P0 | Discovery |
| 89 | `store_listing_clicked` | Click listing in store | entity_id, listing_id, position | P1 | Discovery |
| 90 | `store_video_played` | Play store intro | entity_id, duration, completion_pct | P1 | Trust |

---

## DOMAIN 5: AUCTION & BIDDING (20 events)

| # | Event | Trigger | Properties | Phase | Funnel |
|---|---|---|---|---|---|
| 91 | `auction_viewed` | View auction listing | auction_id, current_bid, bidders_count, time_remaining, reserve_met | P0 | Auction |
| 92 | `auction_rules_viewed` | Click rules/info | auction_id, read_duration | P2 | Auction |
| 93 | `auction_bid_history_viewed` | Click bid history | auction_id, bids_count, unique_bidders | P1 | Auction |
| 94 | `bid_initiated` | Click "Place bid" button | auction_id, current_bid, suggested_increment | P0 | Auction |
| 95 | `bid_amount_entered` | Type bid amount | auction_id, amount, minimum_required, above_minimum_pct | P1 | Auction |
| 96 | `bid_placed` | Bid confirmed | auction_id, amount, bid_number, time_remaining, is_winning, auto_bid | P0 | Auction |
| 97 | `bid_outbid` | Someone outbid user | auction_id, new_highest, my_bid, time_since_my_bid | P0 | Auction |
| 98 | `bid_retaliated` | Bid again after outbid | auction_id, time_to_retaliate_seconds, new_amount, increase_pct | P1 | Auction |
| 99 | `bid_auto_set` | Enable auto-bid | auction_id, max_limit, increment | P1 | Auction |
| 100 | `bid_auto_triggered` | Auto-bid fires | auction_id, amount, triggered_by_opponent_amount | P2 | Auction |
| 101 | `auction_won` | User wins | auction_id, final_price, starting_price, total_bids, my_bid_count | P0 | Auction |
| 102 | `auction_lost` | User lost | auction_id, final_price, my_max_bid, winner_bid_count | P0 | Auction |
| 103 | `auction_watched` | Add to watchlist | auction_id, time_before_end | P1 | Auction |
| 104 | `auction_reminder_set` | Set end-time reminder | auction_id, reminder_offset_minutes, method | P2 | Auction |
| 105 | `auction_countdown_viewed` | Watch final 60s | auction_id, seconds_watched, bid_placed_during | P2 | Auction |
| 106 | `auction_sniped` | Bid in last 30s | auction_id, seconds_remaining, amount, success | P2 | Auction |
| 107 | `auction_revisited` | Return to same auction | auction_id, visits_count, bid_status_changed_since_last | P2 | Auction |
| 108 | `auction_shared` | Share auction link | auction_id, channel, time_remaining | P2 | Viral |
| 109 | `bid_room_joined` | Pay coins to enter | auction_id, coins_spent | P0 | Auction |
| 110 | `bid_deposit_refunded` | Auction ends, non-winner refund | auction_id, amount_refunded | P2 | Auction |

---

## DOMAIN 6: OFFERS & NEGOTIATION (15 events)

| # | Event | Trigger | Properties | Phase | Funnel |
|---|---|---|---|---|---|
| 111 | `offer_form_opened` | Click "Make offer" | listing_id, listed_price, suggested_offers[] | P0 | Negotiation |
| 112 | `offer_amount_entered` | Type/select amount | listing_id, amount, discount_pct, is_suggested | P1 | Negotiation |
| 113 | `offer_message_added` | Add note to offer | listing_id, message_length | P2 | Negotiation |
| 114 | `offer_submitted` | Send offer | listing_id, amount, discount_pct, listed_price, message_included | P0 | Negotiation |
| 115 | `offer_viewed_by_seller` | Seller opens offer | listing_id, time_to_view_hours, buyer_id | P1 | Negotiation |
| 116 | `offer_accepted` | Seller accepts | listing_id, final_price, negotiation_rounds, time_to_accept_hours | P0 | Negotiation |
| 117 | `offer_rejected` | Seller declines | listing_id, reason, counter_offered | P0 | Negotiation |
| 118 | `offer_countered` | Seller counter-offers | listing_id, counter_amount, original_amount, concession_pct | P1 | Negotiation |
| 119 | `offer_expired` | Time out | listing_id, hours_elapsed, follow_up_sent | P2 | Negotiation |
| 120 | `offer_cancelled` | Buyer withdraws | listing_id, reason, time_since_submitted | P2 | Negotiation |
| 121 | `negotiation_completed` | Deal reached (any path) | listing_id, final_price, rounds, total_duration_hours | P1 | Negotiation |
| 122 | `callback_requested` | Request seller callback | listing_id, buyer_phone_prefix, message | P0 | Lead |
| 123 | `callback_completed` | Seller called back | listing_id, response_time_minutes | P2 | Lead |
| 124 | `meeting_proposed` | Suggest in-person meeting | listing_id, proposed_date, location_type | P2 | Negotiation |
| 125 | `meeting_confirmed` | Both parties agree | listing_id, meeting_date, location | P2 | Negotiation |

---

## DOMAIN 7: MESSAGING (12 events)

| # | Event | Trigger | Properties | Phase | Funnel |
|---|---|---|---|---|---|
| 126 | `chat_initiated` | Open new thread | listing_id, seller_id, context (listing/offer/order) | P0 | Communication |
| 127 | `message_sent` | Send message | thread_id, listing_id, is_first, length, contains_price, contains_offer | P0 | Communication |
| 128 | `message_received` | Get message (server) | thread_id, sender_role (buyer/seller), response_time_ms | P1 | Communication |
| 129 | `message_read` | Read receipt | thread_id, time_to_read_seconds | P1 | Communication |
| 130 | `chat_media_shared` | Send image/video | thread_id, media_type, file_size | P2 | Communication |
| 131 | `chat_location_shared` | Share meeting point | thread_id, location_type | P2 | Communication |
| 132 | `seller_response_time` | Calculated on reply | thread_id, response_minutes, listing_id | P1 | Seller Quality |
| 133 | `chat_thread_archived` | Archive conversation | thread_id, messages_count, outcome (sold/lost/ghosted) | P2 | Communication |
| 134 | `chat_blocked` | Block user | blocked_user_id, reason, messages_before_block | P2 | Safety |
| 135 | `chat_reported` | Report conversation | thread_id, reason, evidence | P2 | Safety |
| 136 | `chat_unread_count` | Unread badge update (passive) | unread_count, oldest_unread_hours | P2 | Engagement |
| 137 | `chat_notification_opened` | Open app from message notification | thread_id, time_to_open | P1 | Engagement |

---

## DOMAIN 8: ORDERS & TRANSACTIONS (20 events)

| # | Event | Trigger | Properties | Phase | Funnel |
|---|---|---|---|---|---|
| 138 | `buy_now_clicked` | Click buy now button | listing_id, price, source_page | P0 | Purchase |
| 139 | `checkout_started` | Enter checkout flow | listing_id, price, delivery_available | P0 | Purchase |
| 140 | `delivery_method_selected` | Choose delivery/pickup | method (tawssil_home/relay_point/in_person), cost, city | P0 | Purchase |
| 141 | `delivery_rate_fetched` | Tawssil returns rate | origin_city, dest_city, amount, estimated_days | P1 | Purchase |
| 142 | `recipient_info_entered` | Fill shipping details | city, has_phone, time_to_fill_seconds | P1 | Purchase |
| 143 | `order_placed` | Confirm order | order_id, amount, delivery_fee, source_channel, utm_*, fbclid, ttclid, commission_rate | P0 | Purchase |
| 144 | `order_confirmed_by_seller` | Seller accepts order | order_id, time_to_confirm_hours | P1 | Fulfillment |
| 145 | `order_shipped` | Seller ships / Tawssil picks up | order_id, carrier, tracking, tawssil_barcode | P0 | Fulfillment |
| 146 | `order_in_transit` | Tawssil status update | order_id, location, next_hub | P2 | Fulfillment |
| 147 | `order_out_for_delivery` | Last mile | order_id, estimated_delivery_time | P2 | Fulfillment |
| 148 | `order_delivered` | Delivery confirmed (Tawssil webhook) | order_id, amount, delivery_days, actual_vs_estimated | P0 | Fulfillment |
| 149 | `order_returned` | Return initiated | order_id, reason, days_since_delivery | P1 | Fulfillment |
| 150 | `order_cancelled` | Cancel before ship | order_id, reason, who_cancelled (buyer/seller) | P1 | Fulfillment |
| 151 | `order_tracking_viewed` | Check delivery status | order_id, status, views_count | P2 | Engagement |
| 152 | `review_prompted` | Show review request | order_id, days_since_delivery, channel | P2 | Trust |
| 153 | `review_submitted` | Write review | order_id, listing_id, rating, text_length, photos | P0 | Trust |
| 154 | `review_edited` | Edit existing review | review_id, original_rating, new_rating | P2 | Trust |
| 155 | `cod_payment_collected` | Cash collected (Tawssil) | order_id, amount | P1 | Purchase |
| 156 | `cod_payment_refused` | Customer refused COD | order_id, reason | P1 | Purchase |
| 157 | `guest_order_placed` | Order from non-registered user | order_id, guest_id, phone_verified | P1 | Purchase |

---

## DOMAIN 9: PAYMENTS & COINS (20 events)

| # | Event | Trigger | Properties | Phase | Funnel |
|---|---|---|---|---|---|
| 158 | `pricing_page_viewed` | Open pricing/packages | source_page, current_tier | P0 | Monetization |
| 159 | `coin_pack_selected` | Click pack | pack_name, price_mad, coins_amount | P0 | Monetization |
| 160 | `checkout_coin_started` | Redirect to Payzone | pack_name, amount_mad, payment_method (card/cash) | P0 | Monetization |
| 161 | `payment_initiated` | Payzone form loaded | method, amount, currency | P1 | Monetization |
| 162 | `payment_3ds_shown` | 3DS challenge | bank_issuer, challenge_type | P2 | Monetization |
| 163 | `payment_success` | Payzone confirms (server) | transaction_id, amount, method, processing_time_ms | P0 | Monetization |
| 164 | `payment_failed` | Payzone decline (server) | failure_code, failure_reason, retry_attempted | P0 | Monetization |
| 165 | `payment_retried` | User retries | retry_number, new_method, success | P1 | Monetization |
| 166 | `coin_purchased` | Coins credited to wallet (server) | pack_name, amount_mad, coins_received, bonus_coins | P0 | Monetization |
| 167 | `coin_earned` | Any credit (server) | source (wheel/publish/offer/review/referral/refund), amount | P0 | Economy |
| 168 | `coin_spent` | Any debit (server) | source (boost/bid_room/listing/subscription/lead_unlock), amount, target_id | P0 | Economy |
| 169 | `boost_purchased` | Boost a listing | listing_id, duration_days, coins_spent | P0 | Monetization |
| 170 | `boost_extended` | Extend existing boost | listing_id, new_duration, coins_spent | P1 | Monetization |
| 171 | `subscription_purchased` | Buy distribution package | tier (elan/portee/rayonnement), track (buy_now/lead_gen), duration_days, coins_spent | P0 | Monetization |
| 172 | `subscription_trial_claimed` | Activate free trial | slots, source_cta | P0 | Trial |
| 173 | `subscription_expired` | Package ends | tier, track, days_active, listings_distributed | P1 | Monetization |
| 174 | `lead_unlocked` | Pay to see contact | listing_id, coins_spent, lead_id | P0 | Monetization |
| 175 | `lucky_wheel_spun` | Daily spin | coins_won, streak_days, total_spins | P0 | Gamification |
| 176 | `wallet_viewed` | Open wallet page | balance, lifetime_earned, lifetime_spent | P1 | Monetization |
| 177 | `coin_purchase_abandoned` | Leave pricing without buying | time_on_page, last_pack_viewed, source_page | P1 | Monetization |

---

## DOMAIN 10: SELLER LIFECYCLE (20 events)

| # | Event | Trigger | Properties | Phase | Funnel |
|---|---|---|---|---|---|
| 178 | `listing_creation_started` | Enter sell flow | source (fab/dashboard/cta), category_hint | P0 | Seller |
| 179 | `listing_photo_uploaded` | Add photo | count, is_first, total_size_mb, ai_analysis_triggered | P0 | Seller |
| 180 | `listing_ai_generated` | AI writes content | category_suggested, title_length, desc_length, confidence | P0 | Seller AI |
| 181 | `listing_ai_edited` | User changes AI output | fields_changed[], edit_distance_pct | P0 | Seller AI |
| 182 | `listing_ai_accepted` | User accepts AI as-is | category, title_length | P1 | Seller AI |
| 183 | `listing_category_selected` | Choose category | category_id, was_ai_suggested, was_changed | P0 | Seller |
| 184 | `listing_price_set` | Enter price | price, category, price_vs_market_pct | P1 | Seller |
| 185 | `listing_video_uploaded` | Add video | duration, resolution, file_size | P1 | Seller |
| 186 | `listing_step_completed` | Wizard step done | step_number, step_name, time_on_step_seconds | P1 | Seller |
| 187 | `listing_creation_abandoned` | Leave without publish | last_step, time_spent, had_photos, had_ai | P0 | Seller |
| 188 | `listing_draft_saved` | Save draft | listing_id, completion_pct | P1 | Seller |
| 189 | `listing_published` | Go live | listing_id, category, city, price, has_video, ai_used, type, creation_time_seconds | P0 | Seller |
| 190 | `listing_edited` | Edit live listing | listing_id, fields_changed[], price_changed | P1 | Seller |
| 191 | `listing_relisted` | Re-list after unsold | listing_id, days_since_expired, price_change_pct, photos_changed | P2 | Seller |
| 192 | `listing_deleted` | Remove listing | listing_id, reason, days_live, views_count | P2 | Seller |
| 193 | `store_created` | Create store page | entity_id, store_type | P0 | Seller |
| 194 | `store_boost_purchased` | Boost store | entity_id, duration, coins_spent | P1 | Seller |
| 195 | `seller_dashboard_viewed` | Open dashboard | active_listings, total_views_7d, total_revenue | P1 | Seller |
| 196 | `seller_insights_viewed` | Check analytics/tips | metrics_viewed[], date_range | P2 | Seller |
| 197 | `seller_pricing_suggestion_seen` | AI suggests price | suggested_price, current_price, accepted | P2 | Seller AI |

---

## DOMAIN 11: NOTIFICATIONS & ENGAGEMENT (15 events)

| # | Event | Trigger | Properties | Phase | Funnel |
|---|---|---|---|---|---|
| 198 | `notification_sent` | Any notification dispatched (server) | type, channel (push/whatsapp/in_app/email), template, user_id | P1 | Engagement |
| 199 | `notification_delivered` | Confirmed delivery | type, channel, delivery_latency_ms | P2 | Engagement |
| 200 | `notification_opened` | User taps/clicks notification | type, channel, time_to_open_seconds, listing_id | P0 | Engagement |
| 201 | `notification_dismissed` | Swipe away / ignore | type, channel, time_to_dismiss | P2 | Engagement |
| 202 | `whatsapp_sent` | WhatsApp dispatched (server) | template_name, language, status, user_id | P1 | WhatsApp |
| 203 | `whatsapp_delivered` | Meta confirms delivery | template_name, delivery_time_ms | P2 | WhatsApp |
| 204 | `whatsapp_read` | User opens WhatsApp msg | template_name, time_to_read_seconds | P2 | WhatsApp |
| 205 | `whatsapp_link_clicked` | User clicks CTA in WhatsApp | template_name, destination, listing_id | P1 | WhatsApp |
| 206 | `push_permission_requested` | Browser/app asks permission | platform, granted | P1 | Engagement |
| 207 | `push_permission_granted` | User accepts | platform, device_type | P1 | Engagement |
| 208 | `deep_link_arrived` | User arrives from notification/share | source (whatsapp/push/email/share), template, listing_id | P0 | Attribution |
| 209 | `referral_link_shared` | User shares referral | channel, user_id | P1 | Viral |
| 210 | `referred_user_arrived` | New user from referral | referrer_id, landing_page | P1 | Viral |
| 211 | `referral_reward_earned` | Both parties credited | referrer_id, referred_id, coins_each | P2 | Viral |
| 212 | `saved_search_match_fired` | Alert triggers (server) | search_id, listing_id, user_id | P2 | Engagement |

---

## DOMAIN 12: AI & MCP (12 events)

| # | Event | Trigger | Properties | Phase | Funnel |
|---|---|---|---|---|---|
| 213 | `ai_classification_completed` | Bedrock classifies photo | category_suggested, confidence, model_id, latency_ms | P1 | AI |
| 214 | `ai_generation_completed` | Bedrock generates title/desc | listing_id, languages[], model_id, latency_ms | P1 | AI |
| 215 | `ai_embedding_indexed` | Listing embedded + indexed | listing_id, embedding_dim, model_id | P2 | AI |
| 216 | `ai_content_generated` | Content engine blog post | topic, language, word_count, model_id | P2 | AI |
| 217 | `mcp_query_received` | ChatGPT calls our MCP | tool_name, query, parameters | P0 | MCP |
| 218 | `mcp_results_returned` | We respond to ChatGPT | tool_name, results_count, latency_ms | P1 | MCP |
| 219 | `mcp_user_converted` | MCP user visits site | referrer_chatgpt, listing_id, action_taken | P1 | MCP |
| 220 | `mcp_listing_served` | Listing shown via ChatGPT | listing_id, query_context | P2 | MCP |
| 221 | `image_search_performed` | Camera search | query_embedding_generated, results_count, top_category | P1 | AI |
| 222 | `voice_search_performed` | Voice input (future) | transcription, language_detected, results_count | P2 | AI |
| 223 | `darija_expansion_triggered` | Dictionary match | term, expanded_to, results_improvement | P1 | AI |
| 224 | `bedrock_invocation` | Any Bedrock call (server) | model_id, input_tokens, output_tokens, latency_ms, cost_estimate | P2 | AI |

---

## DOMAIN 13: SYSTEM & PERFORMANCE (16 events)

| # | Event | Trigger | Properties | Phase | Funnel |
|---|---|---|---|---|---|
| 225 | `api_request_completed` | Any API call (server) | endpoint, method, status_code, response_time_ms, user_id | P2 | Performance |
| 226 | `api_error_occurred` | 4xx/5xx response | endpoint, status_code, error_type, user_impact | P1 | Performance |
| 227 | `page_performance_measured` | Core Web Vitals | lcp_ms, cls, inp_ms, ttfb_ms, page_type | P1 | Performance |
| 228 | `javascript_error` | Uncaught exception | error_message, component, page, user_action_before | P1 | Performance |
| 229 | `video_buffer_event` | Video stalls | listing_id, buffer_duration_ms, connection_type | P2 | Performance |
| 230 | `image_load_failed` | Broken image | listing_id, image_url, fallback_shown | P2 | Performance |
| 231 | `offline_detected` | Connection lost | cached_content_available, actions_queued | P2 | Performance |
| 232 | `slow_interaction` | INP > 200ms | interaction_type, delay_ms, component | P2 | Performance |
| 233 | `feed_health_checked` | Feed regen completes (server) | channel, items_count, errors, duration_ms | P1 | Ops |
| 234 | `feed_fetched_by_platform` | External platform fetches our feed | channel (google/meta/tiktok), items_served, response_time | P1 | Ops |
| 235 | `external_api_health` | Periodic health check (server) | service (payzone/tawssil/whatsapp/opensearch), status, latency_ms | P2 | Ops |
| 236 | `cron_job_completed` | Scheduled job finishes (server) | job_name, duration_ms, items_processed, errors | P2 | Ops |
| 237 | `opensearch_query_executed` | Search engine query (server) | query_type (text/image/hybrid), results_count, latency_ms | P1 | Performance |
| 238 | `redis_cache_hit_rate` | Cache stats (periodic) | hit_rate_pct, keys_count, memory_used_mb | P2 | Ops |
| 239 | `ecs_task_health` | Task status (CloudWatch) | service, cpu_pct, memory_pct, task_count | P2 | Ops |
| 240 | `deployment_completed` | CI deploys new version | service, image_tag, deploy_time_seconds, success | P2 | Ops |

---

## TOTAL: 240 EVENTS

| Phase | Count | Description |
|---|---|---|
| **P0** | 65 | Core conversions + critical touchpoints (Session 14A) |
| **P1** | 85 | Behavioral depth + engagement signals (Session 14B) |
| **P2** | 90 | Advanced analytics + system telemetry (Session 14C + 16) |
| **TOTAL** | **240** | Full marketplace intelligence |

At 3-28K DAU with 240 events, we'd generate **2-8M events/month** — still only 2-8% of our 100M free Amplitude quota.

---

## THE 12 FUNNELS (built from the events above)

### FUNNEL 1: SIGNUP FUNNEL
```
session_started (utm_source=X) → auth_gate_shown (any context) → signup_started 
→ signup_otp_requested → signup_otp_verified → signup_completed → identify
```
**Measure:** Conversion rate per step. Drop-off by method. Time to complete. Gate context that converts best.

### FUNNEL 2: BUYER PURCHASE FUNNEL (Buy-Now)
```
listing_impression → listing_card_clicked → listing_viewed → listing_dwell_time (>10s)
→ listing_contact_initiated OR buy_now_clicked → checkout_started → delivery_method_selected
→ order_placed → order_delivered → review_submitted
```
**Measure:** Impression-to-click rate. View-to-contact rate. Contact-to-order rate. Delivery completion rate.

### FUNNEL 3: BUYER AUCTION FUNNEL
```
auction_viewed → bid_initiated → bid_placed → (bid_outbid → bid_retaliated)* 
→ auction_won → order_placed → order_delivered
```
**Measure:** View-to-bid rate. Bid-to-win rate. Retaliation rate. Win-to-complete rate.

### FUNNEL 4: BUYER NEGOTIATION FUNNEL
```
listing_viewed → offer_form_opened → offer_submitted → offer_viewed_by_seller
→ (offer_accepted | offer_countered → offer_submitted)* → negotiation_completed
→ order_placed → order_delivered
```
**Measure:** View-to-offer rate. Offer acceptance rate. Rounds per deal. Counter-to-close rate.

### FUNNEL 5: SELLER ACTIVATION FUNNEL
```
signup_completed (context=sell) → listing_creation_started → listing_photo_uploaded
→ listing_ai_generated → listing_category_selected → listing_price_set 
→ listing_published → first_lead_received → first_sale
```
**Measure:** Signup-to-first-listing time. Step drop-off. AI adoption rate. Time to first lead. Time to first sale.

### FUNNEL 6: SELLER MONETIZATION FUNNEL
```
listing_published → pricing_page_viewed → coin_pack_selected → checkout_coin_started
→ payment_success → coin_purchased → (boost_purchased | subscription_purchased)
```
**Measure:** Listing-to-monetization rate. Pack conversion rate. Payment success rate. Coin utilization (spent/earned).

### FUNNEL 7: SEARCH-TO-CONVERSION FUNNEL
```
search_performed → search_result_clicked (position) → listing_viewed → listing_dwell_time
→ (listing_contact_initiated | offer_submitted | bid_placed | buy_now_clicked)
```
**Measure:** Search CTR by position. Zero-result rate. Query-to-conversion rate. Filter effectiveness.

### FUNNEL 8: TRIAL CONVERSION FUNNEL
```
trial_banner_shown → trial_banner_clicked → auth_gate_shown (context=sell)
→ signup_completed → subscription_trial_claimed → listing_published (within trial)
→ subscription_purchased (after trial)
```
**Measure:** Banner-to-click rate. Click-to-claim rate. Trial-to-publish rate. Trial-to-paid conversion rate.

### FUNNEL 9: NOTIFICATION ENGAGEMENT FUNNEL
```
notification_sent → notification_delivered → notification_opened
→ deep_link_arrived → (listing_viewed | auction_viewed | wallet_viewed)
→ action_taken (purchase/bid/offer/boost)
```
**Measure:** Delivery rate. Open rate. Click-through rate. Action rate. Revenue per notification.

### FUNNEL 10: REFERRAL VIRAL LOOP
```
referral_link_shared → referred_user_arrived → signup_completed → listing_published OR order_placed
→ referral_reward_earned → referral_link_shared (next generation)
```
**Measure:** Share rate. Arrival rate. Conversion rate. Viral coefficient (k-factor). Generation depth.

### FUNNEL 11: LEAD GENERATION FUNNEL (Lead-Gen/Rental track)
```
listing_impression (lead_gen category) → listing_viewed → listing_contact_initiated
→ lead_captured → lead_unlocked (seller pays) → callback_completed → deal_closed
```
**Measure:** Impression-to-view. View-to-lead. Lead unlock rate. Lead-to-deal conversion. Revenue per lead.

### FUNNEL 12: CHATGPT/MCP FUNNEL
```
mcp_query_received → mcp_results_returned → mcp_user_converted (visits site)
→ listing_viewed → (offer_submitted | buy_now_clicked | bid_placed)
→ order_placed
```
**Measure:** Query-to-visit rate. Visit-to-action rate. MCP conversion rate. Revenue attributed to ChatGPT.

---

## DERIVED SIGNALS (50 computed metrics — from combinations of events)

| # | Signal | Formula | Use |
|---|---|---|---|
| 1 | `conversion_probability` | (views × dwell × contacts) / sessions | Predict purchase |
| 2 | `churn_risk_score` | days_inactive + session_decline + notification_ignore | Predict inactivity |
| 3 | `seller_quality_score` | response_time + reviews + photos + ship_speed | Rank sellers |
| 4 | `listing_quality_score` | photos + video + desc_length + price_vs_market | Predict time-to-lead |
| 5 | `price_sensitivity_index` | filter_usage + offer_discount_pct + comparison_behavior | Price elasticity |
| 6 | `auction_aggression` | bid_retaliation_rate + snipe_frequency + max_vs_start | Bidder type |
| 7 | `negotiation_willingness` | offers_made / listings_viewed | Deal propensity |
| 8 | `engagement_score` | (searches + views + contacts + bids + offers) / days_active | User quality |
| 9 | `viral_coefficient` | (shares × share_conversions) / active_users | Growth rate |
| 10 | `category_demand_heat` | search_volume × (1/zero_result_rate) × (1/supply_count) | Supply gap |
| 11 | `channel_ltv` | revenue_from_channel / users_from_channel | Channel value |
| 12 | `notification_fatigue` | dismiss_rate + unsubscribe_rate + time_to_dismiss_declining | Over-messaging risk |
| 13 | `seller_velocity` | listings_published / days_since_signup | Seller activity |
| 14 | `buyer_intent_stage` | gate_context + dwell_time + actions (browse→research→ready→bought) | Intent classification |
| 15 | `coin_utilization_rate` | coins_spent / coins_earned | Economy health per user |
| 16 | `search_satisfaction` | (clicks + purchases_from_search) / searches | Search quality |
| 17 | `feed_engagement_rate` | feed_actions / feed_slides_viewed | Feed content quality |
| 18 | `delivery_satisfaction` | (on_time_deliveries) / total_deliveries | Tawssil quality |
| 19 | `ai_adoption_rate` | ai_listings / total_listings | AI feature value |
| 20 | `mcp_attribution_rate` | mcp_conversions / mcp_queries | ChatGPT channel value |
| 21-50 | ... (per-category, per-city, per-device variants of above) | Segmented versions | Granular optimization |

---

## IMPLEMENTATION PATH

| Session | Events Implemented | Focus |
|---|---|---|
| **14A** (first) | 65 P0 events | Core conversions + identity + key touchpoints |
| **14B** (second) | 85 P1 events | Behavioral depth + engagement + search intelligence |
| **14C** (third) | 50 P2 client events | Advanced analytics + gamification + trust |
| **16** (fourth) | 40 P2 server/system events | Performance telemetry + ops monitoring + AI metrics |
| **TOTAL** | **240 events** | Complete marketplace intelligence |

Each session is a standalone prompt per LAW 11 (13 sections, full skills). They can fire sequentially — each builds on the previous. 14A is the blocker. 14B-C and 16 add depth.

---

## FUNNELS IN AMPLITUDE (configured after 14A deploys)

All 12 funnels above are created in Amplitude's UI (no code). They require the P0 events to exist first. The funnels are the FIRST thing Ramzi sees when he opens Amplitude — the health of his marketplace in 12 conversion paths.

---

*This is the DESTINATION. 240 events. 12 funnels. 50 derived signals. Phased across 4 sessions. The path starts with Session 14A (65 P0 events). Ready when Ramzi says go.*


---

## DOMAIN 14: CONVERSATIONAL AI & WIDGET (15 events)

> **Research insight:** OpenAI launched "Buy it in ChatGPT" with Agentic Commerce Protocol (ACP) — an open standard for AI agents to complete purchases. We ALREADY have MCP. ACP is the NEXT step: users buy directly in ChatGPT without leaving.
> **Research insight:** Conversational commerce converts 3-5x better than static pages. An in-app AI assistant widget (powered by our MCP backend) creates a persistent shopping companion.

| # | Event | Trigger | Properties | Phase | Funnel |
|---|---|---|---|---|---|
| 241 | `widget_opened` | User opens AI chat widget on site | page_type, trigger (fab_button/auto_prompt/nudge), user_segment | P1 | Widget |
| 242 | `widget_message_sent` | User types question to AI | message_length, intent_detected, language, is_darija | P1 | Widget |
| 243 | `widget_response_displayed` | AI responds | response_type (listings/answer/action/clarification), latency_ms, listings_count | P1 | Widget |
| 244 | `widget_listing_clicked` | User clicks a suggested listing from AI | listing_id, query_context, position_in_response | P0 | Widget |
| 245 | `widget_action_taken` | User performs action via widget | action (add_favorite/make_offer/place_bid/contact_seller/buy_now), listing_id | P0 | Widget |
| 246 | `widget_checkout_initiated` | User starts purchase via widget | listing_id, price, via_acp (boolean) | P1 | Widget |
| 247 | `widget_session_duration` | Widget closed | duration_seconds, messages_count, actions_taken, converted | P1 | Widget |
| 248 | `widget_seller_assisted` | Seller uses AI to respond | thread_id, suggestion_accepted, response_time_saved_ms | P2 | Widget |
| 249 | `widget_negotiation_assisted` | AI suggests offer amount | listing_id, suggested_amount, market_context_shown | P2 | Widget |
| 250 | `widget_voice_input_used` | Voice-to-text search | transcription_length, language_detected, query_result | P2 | Widget |
| 251 | `acp_checkout_started` | ChatGPT initiates ACP checkout | listing_id, amount, buyer_context | P1 | MCP |
| 252 | `acp_checkout_completed` | Purchase completed via ACP | order_id, amount, time_to_complete | P1 | MCP |
| 253 | `acp_checkout_abandoned` | ACP checkout dropped | listing_id, step_abandoned, reason | P2 | MCP |
| 254 | `widget_feedback_given` | User rates AI response | rating (helpful/not_helpful), message_id | P2 | AI Quality |
| 255 | `widget_escalated_to_human` | AI can't help, connects to seller | reason, messages_before_escalation | P2 | Widget |

### WIDGET FUNNEL (new)
```
widget_opened → widget_message_sent → widget_response_displayed → widget_listing_clicked 
→ widget_action_taken → (widget_checkout_initiated → order_placed) OR (listing_viewed → standard funnel)
```
**Measure:** Open rate, message-to-response rate, response-to-click rate, widget conversion rate vs standard.

---

## DOMAIN 15: DISPLAY ADS INTELLIGENCE (15 events)

> **Research insight:** 90%+ of digital display runs through programmatic. Pinterest's conversion candidate generation uses "deep causal learning" to optimize ad placement. We already have `display-ads` module with campaigns + events. We need to track advertiser AND viewer behavior to build a self-optimizing ad platform.
> **Research insight:** Marketplace ads are uniquely powerful because we have INTENT data (search queries, category browsing, auth gate contexts) that no external ad network has.

| # | Event | Trigger | Properties | Phase | Funnel |
|---|---|---|---|---|---|
| 256 | `ad_impression` | Display ad enters viewport | campaign_id, creative_id, slot (category_top/search_mid/listing_below/listing_sidebar), targeting_match_reason | P0 | Ads |
| 257 | `ad_viewed` | Ad visible 1s+ (viewability) | campaign_id, creative_id, viewable_seconds, scroll_position | P1 | Ads |
| 258 | `ad_clicked` | Click display ad | campaign_id, creative_id, slot, destination_url, user_intent_context | P0 | Ads |
| 259 | `ad_converted` | Click → action on advertiser's listing | campaign_id, conversion_type (view/contact/purchase), conversion_value | P0 | Ads |
| 260 | `ad_dismissed` | Close/hide ad | campaign_id, reason (x_button/ad_preferences), time_visible | P2 | Ads |
| 261 | `ad_campaign_created` | Advertiser creates campaign (BO/self-serve) | campaign_id, budget, targeting, creative_count, category | P1 | Advertiser |
| 262 | `ad_campaign_paused` | Pause campaign | campaign_id, reason, spend_to_date, impressions_to_date | P2 | Advertiser |
| 263 | `ad_budget_exhausted` | Daily/total budget hit | campaign_id, total_spend, impressions_served, clicks, conversions | P1 | Advertiser |
| 264 | `ad_creative_performance` | Hourly creative metrics (server) | creative_id, impressions, clicks, ctr, conversions, cost | P1 | Advertiser |
| 265 | `ad_targeting_matched` | User matches targeting criteria | campaign_id, match_criteria (category/city/keyword/behavioral), user_segment | P2 | Ads |
| 266 | `ad_auction_won` | Our internal auction (which ad to show) | winner_campaign_id, bid_amount, slot, competing_campaigns_count | P2 | Ads |
| 267 | `ad_revenue_attributed` | Revenue from ad interaction confirmed | campaign_id, revenue_mad, attribution_type (click/view-through) | P1 | Revenue |
| 268 | `ad_slot_filled` | Ad slot has a campaign to show | slot, campaign_id, fill_rate_context | P2 | Ops |
| 269 | `ad_slot_empty` | No campaign for this slot (backfill/house ad) | slot, page_type, fallback_shown | P2 | Ops |
| 270 | `ad_report_generated` | Advertiser views performance report | campaign_id, metrics_viewed[], date_range | P2 | Advertiser |

### ADVERTISER FUNNEL (new)
```
ad_campaign_created → ad_impression → ad_viewed → ad_clicked → ad_converted → ad_revenue_attributed
```
**Measure:** Fill rate per slot, viewability rate, CTR, conversion rate, ROAS per campaign, cost-per-click, cost-per-acquisition.

### DISPLAY ADS FOUNDATION FOR AI:
- **Behavioral targeting:** Use auth gate context (buy_now/contact/bid) + search queries + category affinity to target ads
- **Predictive CTR:** Train model on (user_features × creative_features × slot × time) → P(click)
- **Real-time bidding (internal):** When multiple campaigns compete for a slot, AI picks the highest-value impression
- **Dynamic creative optimization:** Test multiple creatives per campaign, auto-allocate to winner
- **Viewability scoring:** Predict which slots get the most attention (from scroll_depth + dwell_time data)

---

## DOMAIN 16: BLOG & CONTENT ENGINE (10 events)

> **Research insight:** Marketplace SEO strategy shows that "Category pages drive most organic traffic." Programmatic pages (city × category × condition) compound organic traffic at scale. Blog content at 2100-2400 words is the SEO sweet spot.
> **Our reality:** We already have a content engine (Bedrock) generating blog posts + programmatic city×category pages. We have 48K GSC impressions/28d. The blog at /b/ has trilingual content. We need to TRACK content performance.

| # | Event | Trigger | Properties | Phase | Funnel |
|---|---|---|---|---|---|
| 271 | `blog_post_viewed` | Read blog article | post_id, post_slug, category, language, word_count, referrer | P1 | Content |
| 272 | `blog_scroll_depth` | 25/50/75/100% of article | post_id, depth_pct, time_to_depth | P2 | Content |
| 273 | `blog_cta_clicked` | Click internal link/CTA in blog | post_id, cta_type (listing/category/signup/sell), destination | P1 | Content |
| 274 | `blog_listing_clicked` | Click embedded listing from blog | post_id, listing_id, position | P1 | Content |
| 275 | `blog_shared` | Share blog post | post_id, channel | P2 | Content |
| 276 | `programmatic_page_viewed` | View city×category page | city, category, total_listings, referrer (google/bing/direct) | P1 | SEO |
| 277 | `programmatic_page_listing_clicked` | Click listing from programmatic page | page_type, city, category, listing_id, position | P1 | SEO |
| 278 | `content_generated` | AI creates blog/page (server) | content_type, language, topic, word_count, model_id | P2 | AI |
| 279 | `content_indexed` | Google/Bing indexes page (GSC/Bing data) | url, impressions, clicks, position | P2 | SEO |
| 280 | `seo_landing_converted` | Organic visitor → action | landing_page, action (signup/listing_view/contact/purchase), time_to_action | P1 | SEO |

### CONTENT → CONVERSION FUNNEL (new)
```
blog_post_viewed (from organic search) → blog_scroll_depth (>50%) → blog_cta_clicked 
→ listing_viewed → (contact_initiated | offer_submitted | buy_now_clicked) → order_placed
```
**Measure:** Organic traffic per post, read-through rate, CTA click rate, content-to-conversion rate, revenue per post.

### BLOG STRATEGY (from research):
- **Programmatic city×category pages:** "/fr/c/classic/maroc/casablanca/high-tech" — 100+ categories × 50+ cities = 5,000+ indexable pages
- **Buying guides:** "Guide achat iPhone occasion Maroc 2026" — intent-rich, links to listings
- **Seller guides:** "Comment vendre rapidement sur Tawadoo" — recruits sellers
- **Trend reports:** "Les voitures les plus recherchées au Maroc" — from our ACTUAL search data
- **AI-generated, human-edited:** Content engine writes, we review, auto-publish on schedule

---

## DOMAIN 17: CONVERSATIONS & SOCIAL (12 events)

> These track the HUMAN interactions that build marketplace trust — buyer-to-buyer (reviews), seller-to-seller (community), and cross-platform social signals.

| # | Event | Trigger | Properties | Phase | Funnel |
|---|---|---|---|---|---|
| 281 | `review_helpful_marked` | Mark review as helpful | review_id, reviewer_id, helpful_count_after | P2 | Trust |
| 282 | `seller_followed` | Follow a seller/store | entity_id, follower_count_after, source_page | P1 | Social |
| 283 | `seller_unfollowed` | Unfollow | entity_id, days_followed, reason | P2 | Social |
| 284 | `collection_created` | Create a wishlist/collection | collection_name, privacy (public/private) | P2 | Social |
| 285 | `collection_shared` | Share collection link | collection_id, channel, items_count | P2 | Viral |
| 286 | `user_reported` | Report a user | reported_user_id, reason, evidence, prior_interaction | P1 | Safety |
| 287 | `user_blocked` | Block user | blocked_user_id, reason, messages_exchanged_before | P2 | Safety |
| 288 | `dispute_opened` | File dispute on order | order_id, reason, evidence_count | P1 | Trust |
| 289 | `dispute_resolved` | Dispute closed | order_id, resolution (buyer_favor/seller_favor/split), refund_amount | P2 | Trust |
| 290 | `seller_response_rated` | Rate seller communication | seller_id, rating, speed_rating | P2 | Trust |
| 291 | `trust_badge_earned` | User earns verification badge | badge_type (email/phone/id/reviews), user_id | P2 | Trust |
| 292 | `social_share_conversion` | Someone clicks a shared link AND takes action | share_id, original_sharer, action_taken, listing_id | P1 | Viral |

---

## DOMAIN 18: LIFECYCLE & RETENTION (12 events)

> Track renewal, upgrade, downgrade, churn, and win-back across subscriptions, boosts, and engagement.

| # | Event | Trigger | Properties | Phase | Funnel |
|---|---|---|---|---|---|
| 293 | `subscription_renewed` | Auto or manual renewal | tier, track, coins_spent, consecutive_months | P0 | Retention |
| 294 | `subscription_upgraded` | Move to higher tier | from_tier, to_tier, coins_delta, motivation | P0 | Retention |
| 295 | `subscription_downgraded` | Move to lower tier | from_tier, to_tier, reason | P1 | Churn |
| 296 | `subscription_cancelled` | Cancel subscription | tier, track, days_active, reason, listings_affected | P0 | Churn |
| 297 | `boost_renewed` | Re-boost after expiry | listing_id, previous_duration, new_duration, gap_days | P1 | Retention |
| 298 | `boost_expired_no_renew` | Boost expires, user doesn't renew | listing_id, duration_was, views_during_boost | P1 | Churn |
| 299 | `user_reactivated` | Return after 14+ days inactive | days_inactive, reactivation_source (notification/organic/ad) | P0 | Win-back |
| 300 | `user_churned` | 30 days no session (server-computed) | last_session_date, lifetime_value, last_action | P1 | Churn |
| 301 | `win_back_notification_sent` | Send re-engagement (server) | user_id, channel, template, days_inactive | P1 | Win-back |
| 302 | `win_back_converted` | Churned user returns and acts | days_inactive, reactivation_action, channel_that_worked | P1 | Win-back |
| 303 | `daily_streak_maintained` | User logs in consecutive days | streak_days, reward_earned | P2 | Gamification |
| 304 | `milestone_achieved` | User hits platform milestone | milestone_type (10_listings/first_sale/100_views/power_seller), reward | P2 | Gamification |

### SUBSCRIPTION LIFECYCLE FUNNEL (new)
```
pricing_page_viewed → coin_pack_selected → payment_success → subscription_purchased
→ listing_published (within subscription) → leads_received → subscription_renewed OR subscription_cancelled
```
**Measure:** Trial-to-paid, renewal rate, upgrade rate, churn reasons, LTV by tier.

### BOOST LIFECYCLE FUNNEL (new)
```
listing_published → boost_purchased → listing_impression (boosted) → listing_viewed → contact_initiated
→ (boost_expired → boost_renewed) OR (boost_expired_no_renew)
```
**Measure:** Boost ROI (views during boost vs after), renewal rate, revenue per boost.

### WIN-BACK FUNNEL (new)
```
user_churned → win_back_notification_sent → notification_opened → session_started
→ user_reactivated → (listing_published | order_placed | boost_purchased)
```
**Measure:** Win-back rate per channel, time-to-reactivate, reactivated user LTV.

---

## DOMAIN 19: AI SEARCH ENGINE (SELF-IMPROVING) (8 events)

> **Research insight (Pinterest):** "Uses fine-tuned cross-encoder LLMs to evaluate search relevance, then distills into lightweight models for real-time rankings."
> **Research insight (Etsy 2025):** "Shifted from keywords to behavioral signals — clicks, favorites, dwell time — to determine visibility."
> **Research insight (LESER paper):** "Learning to Expand via Search Engine-feedback Reinforcement — fine-tunes LLM using real-time search engine feedback as supervision."
> **Our approach:** Every search → click → outcome creates a training signal. Over time, the search engine LEARNS what's relevant from user behavior.

| # | Event | Trigger | Properties | Phase | Funnel |
|---|---|---|---|---|---|
| 305 | `search_click_signal` | User clicks result after search | query, listing_id, position, dwell_time_after, converted | P0 | AI Search |
| 306 | `search_skip_signal` | Result visible but not clicked | query, listing_id, position, time_visible | P1 | AI Search |
| 307 | `search_purchase_signal` | Search → click → purchase (full loop) | query, listing_id, position, time_to_purchase | P0 | AI Search |
| 308 | `search_relevance_feedback` | Implicit: dwell > 30s = relevant, bounce < 5s = irrelevant | query, listing_id, signal (positive/negative), confidence | P1 | AI Search |
| 309 | `search_query_expansion_result` | Darija/synonym expansion improved results | original_query, expanded_query, results_delta, clicks_after | P1 | AI Search |
| 310 | `search_ranking_served` | The exact ranking shown to user (server) | query, ranked_listing_ids[], algorithm_version, features_used | P2 | AI Search |
| 311 | `search_reranking_candidate` | A listing was considered for re-ranking | listing_id, original_rank, new_rank, boost_factor, reason | P2 | AI Search |
| 312 | `search_model_evaluation` | Periodic offline eval (server batch) | model_version, ndcg@10, mrr, precision@5, training_data_size | P2 | AI Search |

### SEARCH SELF-IMPROVEMENT LOOP:
```
search_performed → search_ranking_served → search_click_signal (positive feedback)
                                         → search_skip_signal (negative feedback)
→ search_relevance_feedback aggregated weekly
→ Learning-to-Rank model retrained
→ search_model_evaluation (new model vs old)
→ A/B test: new ranking vs old
→ If better: deploy. If worse: rollback.
```

**The FOUNDATION for unparalleled AI search:**
- 1.5M historical search queries = training data
- 891K listing views with source = click-through labels
- Behavioral signals (dwell, bookmark, contact, purchase) = relevance grades
- Darija dictionary (397 entries) = query expansion supervision
- Weekly model retraining (like Shopify) once we have 30 days of click signals

---

## UPDATED TOTALS

| Domain | Events | New/Updated |
|---|---|---|
| 1. Identity & Session | 20 | — |
| 2. Discovery & Browse | 25 | — |
| 3. Search & Filter | 20 | — |
| 4. Listing Engagement | 25 | — |
| 5. Auction & Bidding | 20 | — |
| 6. Offers & Negotiation | 15 | — |
| 7. Messaging | 12 | — |
| 8. Orders & Transactions | 20 | — |
| 9. Payments & Coins | 20 | — |
| 10. Seller Lifecycle | 20 | — |
| 11. Notifications & Engagement | 15 | — |
| 12. AI & MCP | 12 | — |
| 13. System & Performance | 16 | — |
| **14. Conversational AI & Widget** | **15** | **NEW** |
| **15. Display Ads Intelligence** | **15** | **NEW** |
| **16. Blog & Content Engine** | **10** | **NEW** |
| **17. Conversations & Social** | **12** | **NEW** |
| **18. Lifecycle & Retention** | **12** | **NEW** |
| **19. AI Search Engine** | **8** | **NEW** |
| **TOTAL** | **312 events** | +72 new |

---

## UPDATED FUNNELS (now 20 total)

| # | Funnel | Events in Chain | What it Measures |
|---|---|---|---|
| 1 | Signup | session → gate → signup → verify → complete | Acquisition conversion |
| 2 | Buyer Purchase (Buy-Now) | impression → click → view → dwell → contact → order → deliver → review | End-to-end buyer journey |
| 3 | Buyer Auction | view → bid → outbid → retaliate → win → order | Auction lifecycle |
| 4 | Buyer Negotiation | view → offer → counter → accept → order | Negotiation success |
| 5 | Seller Activation | signup → create → photo → AI → publish | First listing time |
| 6 | Seller Monetization | publish → pricing → pack → payment → boost/subscribe | Revenue per seller |
| 7 | Search-to-Conversion | search → click → view → action → purchase | Search effectiveness |
| 8 | Trial Conversion | banner → click → claim → publish → subscribe | Trial ROI |
| 9 | Notification Engagement | sent → delivered → opened → action | Notification effectiveness |
| 10 | Referral Viral Loop | share → arrive → signup → action → share again | k-factor |
| 11 | Lead Generation | impression → view → contact → lead → unlock → deal | Lead funnel (RE/vehicles) |
| 12 | ChatGPT/MCP | query → results → visit site → action → purchase | AI channel conversion |
| **13** | **Widget Conversational** | open → message → response → click → action → purchase | Widget effectiveness |
| **14** | **Display Ads** | impression → viewable → click → convert → revenue | Advertiser ROI |
| **15** | **Content/SEO** | blog_view → CTA_click → listing → action → purchase | Content ROI |
| **16** | **Subscription Lifecycle** | purchase → use → renew/upgrade/cancel | Subscription retention |
| **17** | **Boost Lifecycle** | purchase → impressions → contacts → expire → renew/drop | Boost ROI |
| **18** | **Win-Back** | churn → notification → reactivate → action | Win-back effectiveness |
| **19** | **AI Search Self-Improvement** | query → rank → click/skip → feedback → retrain → evaluate | Search quality over time |
| **20** | **ACP Checkout (ChatGPT native)** | ACP_start → cart → payment → confirm → deliver | ChatGPT commerce conversion |

---

## THE VISION FEATURES (what the events ENABLE — future)

### 1. IN-APP AI WIDGET (powered by our MCP backend)
- Persistent chat button on every page
- "Salam! Ana nbghik nlga chi telfon b moins de 3000 dh f Casa" → AI searches, shows results, user buys from widget
- Seller side: AI suggests replies to buyer messages, drafts counter-offers
- Voice input for Darija queries (Web Speech API → transcription → MCP search)
- Gradually replace static UI with conversational UI for specific flows

### 2. AGENTIC COMMERCE PROTOCOL (ACP)
- Users can BUY Tawadoo listings DIRECTLY in ChatGPT without visiting the site
- ChatGPT renders checkout UI, collects shipping/payment, calls our API
- We're already positioned: MCP app is LIVE. ACP is the commerce layer on top.
- Application to OpenAI's partner program with our existing ChatGPT app

### 3. SELF-IMPROVING SEARCH (from click signals)
- Week 1-4: Collect click signals (search_click_signal, search_skip_signal)
- Week 5: First Learning-to-Rank model trained on accumulated data
- Week 6+: Weekly retraining. Search gets smarter every week.
- Darija expansion grows from user queries (queries with 0 results → candidate dictionary entries)
- Price-aware ranking (budget filters → boost cheaper results for price-sensitive users)

### 4. AI-POWERED DISPLAY ADS PLATFORM
- Advertisers self-serve: create campaign, target (category/city/keyword/behavioral cohort)
- Internal auction: when multiple campaigns match a slot, highest bid × quality score wins
- Predictive CTR model: predict which users will click which creatives
- Auto-optimization: pause underperforming creatives, scale winners
- Reporting: real-time BO dashboard with impressions, clicks, CTR, conversions, ROAS
- The DATA from display ads events feeds the model. More ads = better model = higher CTR = more revenue.

### 5. BLOG AS DEMAND ENGINE
- AI-generated buying guides from REAL search data ("Most searched items in Casablanca this week")
- Programmatic city×category pages (5,000+ indexable pages from DB data)
- Each blog post embeds relevant listings (dynamic, from search API)
- Track content → listing → purchase attribution (content ROI per post)
- Auto-refresh: if a guide's listings sell out, regenerate with current inventory

---

## IMPLEMENTATION PATH (updated)

| Session | Events | Focus | Depends On |
|---|---|---|---|
| **14A** | 65 P0 | Core conversions + identity + key touchpoints + search clicks + ads | Nothing (first) |
| **14B** | 85 P1 | Behavioral depth + widget + content + lifecycle | 14A deployed |
| **14C** | 90 P2 | Advanced + self-improving search + display ads AI | 14B deployed |
| **16** | 72 server | System telemetry + AI metrics + server-side events | 14A deployed |
| **TOTAL** | **312** | Complete marketplace intelligence | — |

---

*312 events. 20 funnels. 50+ derived signals. 5 vision features. The destination is clear. The path is phased. Every event serves at least one funnel and one future ML model. This is the taxonomy for Africa's most intelligent marketplace.*
