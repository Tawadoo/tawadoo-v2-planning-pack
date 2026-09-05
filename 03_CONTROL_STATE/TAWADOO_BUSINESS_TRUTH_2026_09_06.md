# TAWADOO — CORE BUSINESS TRUTH (source-verified, canonical — read before authoring ANY session)
**Purpose:** Kiro/Brain kept re-asking the founder about core business (moderation, human-in-the-loop, distribution, jobs/services, feeds) — knowledge already in the code. This file is the durable answer so no session re-learns the basics. Every prompt author reads this FIRST. Verified from source 2026-09-06 (file:line cited). Update only from source, never from memory.

## 1. MODERATION = HUMAN-IN-THE-LOOP, MANUAL, SACRED (X6)
- A listing publishes as `isModerated=false` + `isVerified=false` (defaults, `publication.entity.ts`).
- A **human BO admin** clicks Verify → `approvePublication()` (`publication.service.ts:3705`) sets **BOTH `isModerated=true` (3729) + `isVerified=true` (3730)** in one transaction + pushes to index.
- **No auto-approver / worker / threshold exists** (S-CTO-20 verified). The human gate is INTENDED and STAYS ALWAYS (founder-locked, X6). AI may pre-screen; it NEVER replaces the human.
- Two flags gate two layers: **`isModerated`** = index-write eligibility; **`isVerified`** = public search-read + authenticated detail. Coupled today only because approve sets both.
- Media rows have their OWN `isModerated` on `ta_publication_image` (separate column — name-collision trap).

## 2. VISIBILITY CHAIN (publish → visible)
publish (`publications/publish/:id`) → DB `isModerated/isVerified=false` → **human BO Verify** → both flags true + index push → OpenSearch write gate=`isModerated`, search read filter=`isVerified` → public search + detail. A fresh listing (photo OR photoless) is INVISIBLE until a human moderates. This is correct, not a bug.

## 3. JOBS / SERVICES (image-optional) — source-verified taxonomy (`constants/leadGenCategories.ts`)
- **JOB_CATEGORY_IDS = {85, 144, 145}** (parent 85 = Jobs; 144 job_offers; 145 job_requests).
- **SERVICE_CATEGORY_IDS = {84, 132–143}** (parent 84 = Services + 12 subcats).
- **JOBS_OR_SERVICES = union.** For these: image is OPTIONAL (founder D1=YES), delivery/courier + auction fields HIDDEN, compensation field labeled "salary" (jobs) / "price" (services). `service` is a CATEGORY, NOT a listing type — never query `type=service` (returns 0).
- LEAD_GEN_CATEGORY_IDS includes Real Estate (1,11,12,15–22) + jobs/services — lead-gen track.
- Photoless fallback visual = the CATEGORY ICON from the carousel (`getIcon(category.icon)` in `icones.ts`), founder-decided — NOT a generic placeholder.

## 4. DISTRIBUTION / FEEDS — founder policy + guards (SERIOUS, ban history)
- **FOUNDER POLICY (locked):** silently distribute EVERY ELIGIBLE listing (free/paid/boosted alike) to the RIGHT feed, from the start, to maximize traffic before paid kicks in. Eligibility + correct-feed-routing is the ONLY gate. NEVER communicate distribution status to users. Distribution is silent + automatic — NOT a second approval (moderation is the only human gate).
- **Feed eligibility runs off:** `SyndicationCampaignProduct` + `SyndicationUserSubscription` tracks (`syndication-eligibility.service.ts`). A plain moderated listing is in a feed only via an active campaign/subscription (X1, by design) — boost/paid affects campaign/ranking, not whether an eligible listing distributes.
- **Channels:** google-xml, meta-json, tiktok-json, tiktok-csv, chatgpt-json. Per-channel quality gates (min resolution, description length). Each channel has its OWN image requirement.
- **GUARDS ARE SACRED (GMC SUSPENSION history — `GOOGLE_MERCHANT_APPEAL_2026_08_23.md`):** feed-quality-gate + eligibility guards exist to prevent provider BANS. NEVER weaken one. A mislabeled/ineligible/mismatched item reaching a provider can ban the account — the real cost.
- **KNOWN BUG (F1, S-CTO-22api fixing):** `feed-generator.service.ts:479` `.innerJoinAndSelect('pub.images')` drops ELIGIBLE photoless listings from ALL feeds — violates the silent-distribution policy. Fix = LEFT-join/placeholder while preserving every guard + per-channel image requirement.
- GMC is SUSPENDED (appeal pending); Google push held. Meta/TikTok feeds generate.

## 5. MONEY / COMMERCE (never wrong, in the Core, locks)
- Money spine: offer/bid/coin/payment/order/lead/subscription. Wallet uses pessimistic row-locks + shared transactions. Payzone callback signature verified. Coin grant/deduct = super-admin (Ramzi) only, audited.
- Money events now stamp the acting user_id (S-CTO-3), entity id kept as a property.

## 6. ANALYTICS / SOVEREIGNTY
- DB-first (`ta_analytics_event`) → transactional outbox (`ta_analytics_delivery`) → async workers → Amplitude/sGTM/Meta/TikTok/GA4/S3 lake. Lake NEVER on the synchronous commerce path. Every interaction feeds the lake first, then providers.
- Unknown events persist as `_is_canonical=false` (surfaced, not dropped). Allowlist enforced (S-CTO-15). Unknown event = BUILD (if real UI) or ASK founder (if no UI) — never silently ignore.

## 7. THE 3 AGENTS / MOAT (endgame, mostly parked)
Buyer agent (UCP/MCP/ACP), Seller agent (Premium), Advertiser agent (RTB). MCP LIVE at mcp.tawadoo.ma. Bayesian/HMM hidden-state prediction model = the moat, post-refactor, fed by session corpus + attribution + convert-reward.

## 8. RED LINES (never violate)
Classic View sacred (never broken by Smart). Classic + Smart = SEPARATE (FACE-001=C). DB = system of record; projections (OpenSearch/feeds/Amplitude/lake) rebuildable, never the truth. Human moderation gate stays. Money in Core with locks. No microservice sprawl (modular monolith). No prod mutation without founder. Smallest safe change; prove broad patterns with one bounded slice.

---
**Author rule:** if a session prompt would ASK the founder something answered above, answer it from HERE instead. Only ask the founder for genuine business/policy decisions NOT already decided here.
