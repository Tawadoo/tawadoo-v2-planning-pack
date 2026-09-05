# S-CTO-25-NAV-MAP — ENTITY & NAVIGATION INTEGRITY MAP (Stage 1, READ-ONLY)
**Evidence file · session S-CTO-25-NAV-MAP · 2026-09-06**
**Status proposed by this session:** FINISHED — COMPLETE (for the MAP). Independent Brain QA owes acceptance (§18).
**Classification:** A — Refactor/Architecture. Read-only. Zero product code changed. Only this evidence file written.

---

## EXECUTIVE RESULT (≤20 lines)
1. **YES — there is a systemic entity-addressing/navigation-integrity defect.** It is not 11 isolated bugs; the FR symptoms share a small number of named root patterns.
2. The **authoritative identity** of a listing is `ta_publication.id` (**UUID**, System of Record). There is **no slug column** on the record; the public slug lives per-language on `PublicationTranslation.slug` (a *projection-grade* attribute, `text`, no unique index declared in the entity).
3. Every surface builds and resolves URLs **on its own**, inline — there is **NO centralized URL builder** anywhere in web/api. This is the mechanical root of the drift.
4. **The single biggest finding, proven LIVE:** the same published+verified listing returns **HTTP 200 from `publications/slug/:slug` but HTTP 404 from `publications/lightssr/:slug`** — because the two endpoints use **different visibility predicates** (`isVerified` vs `isModerated`), and live data has verified listings with **`isModerated = null`** (D1 anomaly, confirmed live). One route (`/p/[slug]`) is served by two endpoints that disagree about whether the listing exists.
5. **FR-9 (notif deadlink) CONFIRMED at source:** lead notifications carry no target id and deep-link to a generic `/dashboard/leads`; many notification `objectId`s are **literal strings** (`'receiveOffer'`, `'acceptOffer'`, `'verifyPublication'`), not entity ids.
6. **D2 (vector search leak) CONFIRMED at source:** the keyword leg filters `status/isVerified`; the vector leg does **not**.
7. Most "item unavailable" cases are **STATE 3 (exists, visibility state wrong)** or **STATE 8/11 (wrong-layer resolution / stale-projection)** — **NOT** STATE 1 (genuinely gone). The moderation gate (STATE 2/3) is *intended* and must be preserved; the *inconsistency of the gate across layers* is the bug.
8. The `/{locale}/p/{slug}` and `/{locale}/su/{slug}` URL shapes are **LIVE PUBLIC CONTRACTS** consumed by ChatGPT/MCP + the mobile app + feeds + OG + notifications. A canonical change must preserve or deliberately version them.
9. **Stage-2 recommendation:** build ONE small canonical module — a **Publication Identity + Visibility resolver** in the api (System-of-Record layer) that (a) unifies the visibility predicate, (b) unifies detail resolution (slug OR uuid) behind one gate, and a **thin shared web link/target helper** — then migrate callers. No framework, no "UniversalEntityRouter".
10. DB direct psql = **BLOCKED** (in-VPC); DB-ground-truth items tagged UNKNOWN-BLOCKED, but the API read path surfaced the key DB fact (`isModerated=null`) honestly.

---

## FIVE-LINE OPENERS (both)

**Initial (before tools):**
```
BRANCH: UNVERIFIED (preflight pending per repo)
MISSION: S-CTO-25-NAV-MAP — determine whether Tawadoo V2 has a SYSTEMIC entity-addressing/navigation-integrity defect; produce the 7 artifacts. READ-ONLY.
ORDER: reality check → trace ~10 entities end-to-end → addressing matrix → classify every "unavailable" (STATE 1–12) → root-cause PATTERNS → canonical target model → migration surface → app/MCP contract inventory → Stage-2 recommendation → evidence
BLOCKERS: DB in-VPC/IP-locked + ECS-Exec down (direct psql likely BLOCKED — use API+index+source, tag DB-ground-truth UNKNOWN)
TREE STATE: UNVERIFIED (preserve all work; this session writes only the evidence file)
```

**Corrected (after preflight):**
```
BRANCH: web=Ramzi_V2 · api=Ramzi_V2 · bo=Ramzi_V2 · mcp=Ramzi_V2 (all HEADs match expected)
MISSION: S-CTO-25-NAV-MAP — systemic entity-addressing/navigation-integrity map, 7 artifacts. READ-ONLY.
ORDER: reality check ✓ → trace entities → addressing matrix → failure matrix → root-cause patterns → canonical model → migration surface → app/MCP inventory → Stage-2 decision → evidence
BLOCKERS: DB direct psql BLOCKED (in-VPC/IP-locked, ECS-Exec down) — API+index+source path; API host IS reachable read-only
TREE STATE: web M yarn.lock + prior-session untracked evidence; mcp/bo untracked evidence/scripts; ALL preserved. This session writes ONLY S_CTO_25_NAV_MAP_EVIDENCE_2026_09_06.md
```

---

## PROVENANCE (git / tree / skills)
- **HEADs verified (preflight):** web `8b0a815fa4dadfc445cd29cec8e71e3424bfd3c8` · api `8d55a207c88a6b125b47316cb361c69d463a3b54` · bo `07b3a845bd296675dfb940666bb1343c03fe0a21` · mcp `d8efb4a4bf15f9ebcb9f529f1278e1a74a2fe983`. All on `Ramzi_V2`. **No drift** vs expected (web 8b0a815f · api 8d55a20 · bo 07b3a84 · mcp d8efb4a).
- **Tree state:** web has `M yarn.lock` + prior-session untracked reports/playwright dirs; mcp + bo carry untracked prior-session evidence/scripts. All pre-existing, owned elsewhere, **preserved untouched**.
- **Writable:** only `/Users/ramzihannachi/Code/S_CTO_25_NAV_MAP_EVIDENCE_2026_09_06.md`. Zero product code changed in any repo. No commits, no deploy, no migration.
- **SKILL DISCOVERY / ACTIVATION:** `tawadoo-source-truth` + `tawadoo-runtime-provider-qa` both discovered at their exact paths and activated this session (full SKILL.md bodies loaded).
- **SKILL COMPLETION:** source-truth reconstruction done from repos + live; runtime-provider-qa exercised via read-only HTTP probes on staging (no writes, no secrets emitted).
- **TOOLS/PERMISSIONS/COST:** NONE new. Local read + staging read-only HTTP. `/tmp/scto25_search.json`, `/tmp/s.json` scratch created; not committed.

---

## PER-LAYER "PROVEN vs NOT" LADDER
`documented → source → local → CI → deployed → live`

| Claim | Highest state reached | How |
|---|---|---|
| Listing PK = UUID, no slug column | **source** | publication.entity.ts:42-45 read |
| Public slug is per-language on translation | **source + live** | publication-translation.entity.ts:19-20; live search returns `translations[].slug` |
| Two endpoints (slug vs lightssr) serve one `/p/[slug]` route | **live** | curl: slug 200, lightssr 404 for same listing |
| Visibility predicate diverges (isVerified vs isModerated) | **source + live** | findBySlug service:748 vs findBySlugForSSR service:851 + index-eligibility.ts:26-32; live isModerated=null |
| D1 anomaly: verified listing with isModerated=null | **live (API body)** | slug-endpoint JSON `isModerated: null`, `isVerified: true`, `status: published` |
| D2: vector leg unfiltered | **source** | hybrid-search.service.ts:313 (keyword) vs :443 (vector) |
| FR-9: lead notif no target | **source** | leads.service.ts:691; notification-deep-link.helper.ts lead → generic page |
| No centralized URL builder | **source** | grep: zero exported route builders in web |
| MCP/app depend on /p/{slug} + /su/{slug} | **source** | config.py:184; RootNavigator/index.tsx; share.ts |
| DB row counts / true % of isModerated=null | **UNKNOWN-BLOCKED** | direct psql in-VPC; only one live listing sampled |

---

# ═══════════ ARTIFACT 1 — ENTITY ADDRESSING MATRIX ═══════════
*(source-only + live-confirmed values; no invented data)*

### Legend
- **Authoritative identity** = the System-of-Record key (DB). **Public identity** = what appears in a URL. **API identity** = what the endpoint keys on.

---

### E1 — Normal published LISTING
| Dimension | Value | Evidence |
|---|---|---|
| Authoritative DB identity | `ta_publication.id` **UUID** (`@PrimaryGeneratedColumn('uuid')`) | publication.entity.ts:42-45 |
| Public identity | per-language `PublicationTranslation.slug` (text, **no unique index in entity**) | publication-translation.entity.ts:19-20 |
| API identity (public detail) | **slug** via `GET publications/slug/:slug` (OptionalJwt) — **also accepts UUID** (regex fallback) | publication.controller.ts:359; publication.service.ts:748-773 |
| API identity (SSR) | **slug** via `GET publications/lightssr/:slug` — also UUID fallback | publication.controller.ts:456; publication.service.ts:851 |
| API identity (owner) | **UUID** via `GET publications/:id` (JwtAuth + x-entity-identifier) | publication.controller.ts:1929 |
| Search/index identity | OpenSearch doc `_id` = **publication UUID** (`_source.id`); embeddings index carries `publicationId` | hybrid-search.service.ts:~430,466 |
| Route | `/{locale}/p/[slug]` | web src/app/[locale]/p/[slug]/page.tsx |
| Resolver (web) | SSR → `publications/lightssr/{slug}` (page.tsx:20); client → `publications/slug/{slug}` (product-details-view.tsx:73) — **two endpoints, one route** | — |
| Visibility source | **DIVERGES:** detail(slug)= `status==PUBLISHED && isVerified` (owner-exempt); SSR(lightssr)= `isIndexEligible` = `isModerated===true && published && !deleted` (NO owner exempt) | service:748 vs service:851; index-eligibility.ts:26-32 |
| Authz source | OptionalJwt (guest allowed) for public; JwtAuth+entity ownership for `/:id` | controller:358,1928 |
| Share destination | `/{locale}/p/{slug}` (cards, ShareButton) **OR** `/{locale}/p/{UUID}` (dashboard ShareWidget) | ShareButton.tsx; BuyerPurchases.tsx:212, BuyerOffers.tsx:220, BuyerAuctions.tsx:78 |
| Notification target | `service ∈ {OfferService, publicationService, SaveSearchService}` → `/{locale}/p/{publicationSlug}` (re-derived) else `/notification-unavailable` | web notification.tsx:59-65; api helper type `publication` targetId=publicationId |
| App contract | ui_only: `publications/slug/{slug}` primary + `publications/{id}` fallback; deeplink `/{lang}/p/{slug}` + `tawadoo://`; push `{link, publicationId}` | RootNavigator/index.tsx; ProductDetail/index.tsx:158,171 |
| MCP contract | listing identity = `id` (opaque); public URL = `{BASE}/{locale}/p/{slug}` + UTM; **no single-listing resolve** (search only) | client.py:445,517; config.py:184 |
| **Current status** | 🔴 **DUAL-ENDPOINT / DUAL-PREDICATE divergence proven live** (slug 200 / lightssr 404 same listing) | live curl |

---

### E2 — Listing reached FROM SEARCH
| Dimension | Value | Evidence |
|---|---|---|
| Search result identity carried | `id` (UUID) + `type` + `status` + `isVerified` + `translations[].slug` + `entity.slug` | **live** `/publications/search`; serializeLeanResponse ~controller:1008 |
| Link built from | **SEARCH-DOCUMENT slug field** (`translations[].slug` / `slugPublication`), UUID fallback | productCard.tsx:151-155,362; StoreSearchCard.tsx:138 |
| Read gate | keyword leg filters `status=published` + `isVerified=true` | hybrid-search.service.ts:313 |
| **Current status** | 🟡 link built from projection (search doc) not canonical DB identity; correctness leans on the 301 canonical-slug redirect as safety net | page.tsx:35-38 |

---

### E3 — Listing FROM IMAGE / VECTOR search
| Dimension | Value | Evidence |
|---|---|---|
| Result identity | `publicationId` from embeddings index (`_source.publicationId`) | hybrid-search.service.ts:466 |
| Read gate | **NONE** — `executeVectorSearch` has no `isVerified`/`status` filter, only a KNN score floor | hybrid-search.service.ts:443,461 |
| **Current status** | 🟠 **D2 CONFIRMED** — vector recall can surface unverified/unpublished publication ids; keyword/vector asymmetry |

---

### E4 — Listing FROM CATEGORY / FEED
| Dimension | Value | Evidence |
|---|---|---|
| Feed item link | `{FEED_SITE_BASE_URL||staging.tawadoo.ma}/fr/p/{canonicalSlug}` (UUID last-resort fallback) | feed-generator.service.ts:~631-647 |
| Feed eligibility gate | `status=published` + `is_moderated=true` | feed-generator.service.ts:~589 |
| **Current status** | 🟡 feed URL hardcodes `/fr/` locale + defaults to a **staging** base; UUID fallback link relies on web UUID handling |

---

### E5 — Seller's OWN listing (pending / unmoderated)
| Dimension | Value | Evidence |
|---|---|---|
| Detail gate (client `findBySlug`) | owner **exempt** — `if (!entityID || entityID != publication.entity.id) throw 404` → owner passes | publication.service.ts:~830-834 |
| Detail gate (SSR `findBySlugForSSR`) | `isIndexEligible` only — **NO owner exemption** | publication.service.ts:851-895 |
| **Current status** | 🔴 **M-C6 root** — owner opening own pending listing hits SSR first → 404, because the SSR gate has no owner branch that the client gate has |

---

### E6 — LEAD
| Dimension | Value | Evidence |
|---|---|---|
| Authoritative identity | `ta_lead.id` **UUID**; FKs `publication_id` + `seller_entity_id` (UUID) | lead.entity.ts |
| API identity | `PATCH /leads/:id/status` (viewed/contacted/converted); `GET /leads/:id` **fetches up to 1000 entity leads + `.find()`** (no direct by-id) | leads.controller.ts:~528,~470 |
| Route (web) | `/{locale}/dashboard/leads/[id]` → 307 redirect → `/dashboard/listings?tab=inquiries&lead={id}` | web dashboard/leads/[id]/page.tsx |
| Notification target | `leadReceived` — **NO service/objectId**; deep-links to generic `/dashboard/leads` (empty targetId) | leads.service.ts:~691; notification-deep-link.helper.ts lead branch |
| **Current status** | 🔴 **FR-9 CONFIRMED** — lead notif has no addressable target |

---

### E7 — NOTIFICATION target (generic)
| Dimension | Value | Evidence |
|---|---|---|
| Storage | `Notification` entity: PK UUID + `service` + `objectId` + `publication` FK + `deepLink` jsonb | notification.entity.ts:17,31,34,40 |
| Deep-link contract | `NotificationDeepLink {type, targetId, webUrl, appUrl, fluidUrl}` built by `buildNotificationDeepLink` (base `ORIGINE_FRONT||www.tawadoo.ma`) | notification-deep-link.helper.ts:66 |
| Target completeness | **INCONSISTENT:** offer/review → targetId=publicationId; conversation → thread id; lead/wallet/syndication/order → **empty targetId (generic page)** | helper lead/wallet branches |
| `objectId` reality | frequently a **literal string**: `'receiveOffer'`, `'firstOffer'`, `'acceptOffer'`, `'verifyPublication'`, `'unverifyPublication'` | publication.service.ts:1011,1054,3303,3358; offer.service.ts:267 |
| Web handler | switch on `service`; `publicationSlug` **re-derived** from a separate `publicationDetail` lookup — if that lookup fails, slug routing dead-ends even when the listing exists | web notification.tsx:34-96; notificationsModal.tsx:84-98 |
| `listing_approved` | type mapped in helper but **NO producer**; `approvePublication()` fires no notification | helper:12; publication.service.ts:3705 |
| **Current status** | 🔴 overloaded `objectId`, incomplete targets, web/api target-shape mismatch |

---

### E8 — SHARE target
| Dimension | Value | Evidence |
|---|---|---|
| Canonical widget | `ShareButton.tsx` — relative `/{locale}/p/{slug}` → absolute via `window.location.origin` | ShareButton.tsx:~78-81 |
| Dashboard widget | `ShareWidget.tsx` — callers pass **UUID** urls `/p/{publicationId}` | BuyerPurchases.tsx:212; BuyerOffers.tsx:220; BuyerAuctions.tsx:78 |
| Store share | `shareEntity.tsx` — independent `{base}/{locale}/su/{slug}` | shareEntity.tsx:~30-33 |
| OG / canonical | built **independently** in `p/[slug]` + `su/[slug]` generateMetadata, hardcoded `BASE_URL="https://www.tawadoo.ma"` | p/[slug]/page.tsx (BASE_URL const); su/[slug]/page.tsx |
| **Current status** | 🟡 ≥4 independent share builders; slug-vs-UUID mixed; 3 base-URL sources (www / staging / ORIGINE_FRONT) |

---

### E9 — SAVED / FAVORITED target
| Dimension | Value | Evidence |
|---|---|---|
| Listing favorite link | `/{locale}/p/{locale-slug}` with fallback **`null`** → `/p/null` dead link | wishlistItem.tsx:91-94 |
| Store favorite link | `/{locale}/su/{entity.slug}` | wishlistItem.tsx:151 |
| Saved-search notif | `SaveSearchService` → `/{locale}/p/{publicationSlug}` | web notification.tsx:59-65 |
| **Current status** | 🟡 null-fallback produces a real dead link (`/p/null`) |

---

### E10 — AUCTION target
| Dimension | Value | Evidence |
|---|---|---|
| Authoritative identity | `BidRoom.id` **UUID** (one bid-room per publication, OneToOne) | bid-room.entity.ts:8-13 |
| Route | `/{locale}/auction/[id]` where `[id]` = **bid-room id** | auction-view.tsx:185,260 (`bid-entities/bid-room/{id}`) |
| Also reachable as | `/{locale}/p/{slug}` (bid listings render as product cards) — **two addressing schemes for one economic object** | productCard.tsx |
| Notification target | `AuctionService` → `objectId` = bidRoom id | bid-room.service.ts:297-301; web notification.tsx:66-71 |
| Share | bid-room id | ShareButton.tsx (entityType auction) |
| **Current status** | 🟡 dual addressing (publication slug vs bid-room UUID) for the same object |

---

### E11 — SELLER / STORE PROFILE
| Dimension | Value | Evidence |
|---|---|---|
| Authoritative identity | `EntityModel` UUID; public identity = `entity.slug` (e.g. live `ram_r_1`) | live search `entity.slug`; su/[slug]/page.tsx |
| Route | `/{locale}/su/[slug]` resolved by `entities/slug/{slug}` | su/[slug]/page.tsx:14 |
| Inconsistency | `FeaturedStoresSection.tsx:133` addresses store by **entityName (NAME)**, not slug | FeaturedStoresSection.tsx:133 |
| **Current status** | 🟡 name-vs-slug inconsistency; otherwise slug-consistent |

---

# ═══════════ ARTIFACT 2 — FAILURE MATRIX ═══════════
*(every observed navigation/deeplink/unavailable failure → true STATE 1–12 + file:line)*

STATE key: 1 doesn't exist · 2 intentionally unavailable · 3 visibility state wrong · 4 projection/index stale · 5 wrong id · 6 wrong entity type · 7 wrong slug · 8 wrong route · 9 wrong environment · 10 authz mismatch · 11 cache/client-state · 12 unknown.

| # | Observed failure | True STATE | Evidence (file:line / live) | Notes |
|---|---|---|---|---|
| F-A | Same published+verified listing: `lightssr` 404 while `slug` 200 | **3** (visibility state wrong) + **8** (two endpoints one route) | live curl; service:748 vs 851; index-eligibility.ts:26-32 | Root = divergent predicate (`isVerified` vs `isModerated`) + live `isModerated=null` |
| F-B | Owner cannot open own pending/unmoderated listing (M-C6) | **3** (visibility) + **10** (authz asymmetry) | service:851-895 (no owner branch) vs :830-834 (owner exempt) | SSR gate lacks the owner exemption the client gate has |
| F-C | "view listing" 404 on pending moderation (fresh publish) | **2/3** (intended gate, but inconsistently applied) | isIndexEligible; S-CTO-20/23qa | Moderation gate is INTENDED (X6); keep it — fix the *consistency*, not the gate |
| F-D | FR-9 — lead/callback notification dead-ends, no deeplink to the lead | **8** (wrong/missing route) + **5** (no target id) | leads.service.ts:~691; helper lead branch (empty targetId) | api omits objectId; web has no lead branch in notification.tsx switch |
| F-E | Notification `objectId` is a literal string (`'receiveOffer'` etc.) used as an id | **5** (wrong id) / **6** (type confusion) | publication.service.ts:1011,1054,3303,3358; offer.service.ts:267 | `objectId` overloaded; not a real entity id |
| F-F | ReviewService fallback treats `objectId` as a slug (`/p/{objectId}#reviews`) | **6/7** (type/slug mismatch) | web notification.tsx:85-92 | objectId is not a slug |
| F-G | `/notification-unavailable` shown when `publicationSlug` lookup fails even though listing exists | **11** (client-state) / **4** (stale re-derivation) | web notification.tsx:59-65; notificationsModal.tsx:84-98 | slug re-derived from a separate lookup that can fail |
| F-H | Favorite link renders `/p/null` when translations missing | **7** (bad slug) / **11** | wishlistItem.tsx:91-94 | null fallback = real dead link |
| F-I | `productItemLigne` link `/p/` (empty) when no slug | **7** (bad slug) | productItemLigne.tsx:107 | '' fallback |
| F-J | Vector search surfaces unverified/unpublished publication ids | **3** (visibility) / **4** | hybrid-search.service.ts:443 (no filter) | D2 — recall bypasses gate |
| F-K | Featured store opened by NAME not slug | **7** (identity mismatch) | FeaturedStoresSection.tsx:133 | works only if entities/slug also matches name |
| F-L | Dashboard share builds `/p/{UUID}`; cards build `/p/{slug}` | **5 vs 7 mix** (two identities) | BuyerPurchases.tsx:212 vs productCard.tsx:362 | both resolve (UUID fallback exists) but inconsistent |
| F-M | Feed URL defaults to `staging.tawadoo.ma` + `/fr/` | **9** (wrong environment) | feed-generator.service.ts:636,647 | env-default base leaks staging host into feed links |
| F-N | Notif/OG hardcode `www.tawadoo.ma` while web base = `staging.tawadoo.ma` | **9** (env) | p/[slug]/page.tsx BASE_URL; helper:68; .env.staging | 3 sources of truth for public base URL |
| F-O | True % of live listings with `isModerated=null` (blast radius of F-A) | **12** (unknown) | **UNKNOWN-BLOCKED** (psql in-VPC) | one live sample confirms the class exists; scale unknown |

**Deliberately NOT a failure (correct behavior):** bogus UUID → 404, nonsense slug → 404 (live curl); 301 canonical-slug redirect (page.tsx:35-38); moderation gate hiding fresh publishes from the public (intended, X6).

---

# ═══════════ ARTIFACT 3 — ROOT-CAUSE PATTERNS ═══════════
*(shared vs distinct — evidence-decided, no forced unification)*

### PATTERN P1 — **Divergent visibility predicate across layers** (the dominant pattern)
The "is this entity visible / does it exist" question is answered by **different predicates in different layers**:
- Public detail (`findBySlug`) + keyword search: **`isVerified`** (+status, +owner-exemption).
- SSR detail (`findBySlqForSSR`) + index-write + feed eligibility: **`isModerated`** (`isIndexEligible`).
- Vector search: **no gate** (score floor only).
Because live data has verified listings with `isModerated=null` (D1), these predicates **disagree on the same row**. 
**Causes:** F-A, F-B, F-C, F-J. **Evidence:** service:748 vs 851; index-eligibility.ts:26-32; hybrid-search.service.ts:313 vs 443; live `isModerated=null`.
→ This is the systemic root. Fixing it collapses the largest cluster of "item unavailable".

### PATTERN P2 — **No canonical identity→destination→target contract (everything inline)**
There is **no shared URL builder and no shared notification-target shape**. Every card, share widget, OG block, feed formatter, and notification handler builds/parses URLs and targets independently, so identity choice (slug vs UUID vs name), fallbacks (`null` / `''` / UUID), and base URLs drift.
**Causes:** F-H, F-I, F-K, F-L, F-M, F-N, and the slug/UUID mix everywhere. **Evidence:** grep (zero exported route builders); ShareButton/ShareWidget/shareEntity; feed-generator; p/[slug] & su/[slug] BASE_URL duplication.

### PATTERN P3 — **Overloaded / incomplete notification target** (a specialization of P2 on the notification surface)
`objectId` is overloaded (bid-room id | thread id | publication id | literal string) and often incomplete (lead/wallet/order → generic page). The web switch and the api helper disagree on target shape.
**Causes:** F-D, F-E, F-F, F-G. **Evidence:** helper branches; publication.service.ts literal objectIds; web notification.tsx switch.

### DISTINCT (not unified — evidence says so)
- **F-C moderation gate** is INTENDED behavior (X6, founder-locked), not a defect — only its *inconsistent application* (P1) is.
- **F-J vector leak (D2)** shares P1's "predicate" theme but is a distinct code site (a missing filter, not a divergent one) — fix independently.
- **F-M/F-N environment base-URL** is a config-hygiene issue (P2-adjacent) that would remain even if identity were unified — track separately.

**Verdict:** the failures are **NOT** independent — they collapse into **P1 (visibility) + P2 (no canonical contract) + P3 (notification target)**, with a few genuinely-distinct config/behavior items. This is a real systemic pattern; not manufactured.

---

# ═══════════ ARTIFACT 4 — CANONICAL TARGET MODEL ═══════════
*(System-of-Record-derived; simple, boring, explicit. NO framework, NO "UniversalEntityRouter".)*

**Chain (founder constraint): DB → Core → API → projection → URL → notification → share → app → MCP.**

### 4.1 Identity (authoritative)
- **A listing's authoritative identity is `ta_publication.id` (UUID).** Slug is a *localized alias*, not the identity. The DB is the System of Record; OpenSearch/feeds/cache are projections and are never the authority for existence (target-arch §4).
- Same rule per entity: store = `EntityModel.id` (UUID, alias `entity.slug`); lead = `ta_lead.id` (UUID); auction = `BidRoom.id` (UUID); offer/bid = own UUID.

### 4.2 Destination (identity → URL)
- ONE canonical public URL shape per entity, **preserving the live contract**: listing `/{locale}/p/{slug|uuid}`, store `/{locale}/su/{slug}`, auction `/{locale}/auction/{bidRoomId}`, lead (private) `/{locale}/dashboard/listings?tab=inquiries&lead={leadId}`.
- Built by **ONE shared web helper** (`buildEntityUrl(entityType, identity, locale)`) and **ONE api helper** (already partially exists as `buildNotificationDeepLink`), both consuming **one base-URL source of truth** (env, not 3 hardcodes). Slug preferred; UUID always a valid alias (it already is, service:750).

### 4.3 Resolution (URL → same entity)
- ONE resolution path per entity behind ONE endpoint. Collapse `findBySlug` + `findBySlugForSSR` to **share a single visibility predicate** (see 4.5). Accept slug OR uuid (already supported). SSR and client must never disagree.

### 4.4 Visibility (the unified predicate — the core fix)
- Define ONE `isPubliclyVisible(pub)` used by: public detail (both SSR + client), keyword search, vector search, index-write, feed eligibility. 
- Decide the canonical flag semantics: today `isModerated`=index-write, `isVerified`=search-read; `approvePublication` sets both, but D1 leaves `isModerated=null` on some verified rows. **Stage-2 must pick ONE source of truth** (recommend: a single derived predicate that treats "publicly visible" = published + human-approved, however that is stored) and **reconcile the null rows** (data forward-fix, not a projection patch).
- **Owner exemption** must be applied consistently (both SSR + client) so an owner can preview their own pending listing (M-C6) — or explicitly not, but the same everywhere.

### 4.5 Authorization (identity → authz → visibility → resolution)
- Order is fixed: resolve identity → check authz (owner/guest) → check visibility → return. **Never** widen authz to make a link work; **never** leak a private id. A private entity behind a public link returns not-found by design (STATE 2), not a leak.

### 4.6 Projection-staleness handling
- Projections (OpenSearch, feeds, cache) may lag; the **DB record decides existence**. A link that a projection produced must resolve against the record; if the record says not-visible, show a real "unavailable" (with a route away, not a dead end). Vector leg must apply the same visibility predicate (closes D2).

### 4.7 Deeplink / notification target
- ONE target shape: `{entityType, entityId (UUID), url}`. Every producer fills a **real entity id** (no literal strings, no empty targetId). The web consumer routes purely from `{entityType, entityId}` (no re-derivation lookup that can fail). Lead/offer/order notifications carry their own entity id.

### 4.8 Share
- Share URL = the canonical destination (4.2). One builder. No independent OG/canonical/share duplication.

### 4.9 App + MCP
- The canonical URL shapes `/{locale}/p/{slug}` + `/{locale}/su/{slug}` and the `publications/slug/:slug` + `publications/:id` dual-resolution are **preserved as-is** (they are live contracts — Artifact 6). If a structured notification target replaces the FCM `{link, publicationId}` string, the app is versioned in its own unit, not silently.

**What this model is NOT:** not a generic router, not a new abstraction layer, not a Smart+Classic merge, not a DB redesign. It is: one visibility predicate + one url helper + one target shape + a data reconcile.

---

# ═══════════ ARTIFACT 5 — MIGRATION SURFACE (Stage-2 callers) ═══════════
*(every caller that must conform to the canonical contract)*

### API (tawadoo_api_js) — the System-of-Record layer (where the fix lives)
- `publication.service.ts` `findBySlug` (:748) + `findBySlqForSSR` (:851) — unify predicate + owner rule.
- `index-eligibility.ts` (:17-32) — the canonical predicate seam (reuse, don't fork).
- `hybrid-search.service.ts` keyword (:313) + **vector (:443, add filter)**.
- `feed-generator.service.ts` (:589 eligibility; :636-647 URL) — base-URL + eligibility alignment.
- `notification-deep-link.helper.ts` (:66) — one target shape; fill real ids.
- Notification producers: publication.service.ts:1011,1054,3303,3358; offer.service.ts:267; leads.service.ts:691; bid-room.service.ts:297; message.service.ts:540 — pass real `objectId`/entityId.
- Data reconcile: rows with `isModerated=null` but published+verified (blast radius UNKNOWN-BLOCKED).

### Web (tawadoo_web_js) — conform to contract
- Cards/links: `productCard.tsx:151-155,362`, `productItemLigne.tsx:107`, `wishlistItem.tsx:91-94,151`, `StoreSearchCard.tsx:138`, `FeaturedStoresSection.tsx:133`.
- Detail resolvers: `p/[slug]/page.tsx:20` (SSR), `product-details-view.tsx:73` (client) — single predicate parity.
- Share: `ShareButton.tsx`, `ShareWidget.tsx` (+ dashboard callers BuyerPurchases/Offers/Auctions), `shareEntity.tsx`.
- OG/canonical: `p/[slug]/page.tsx` + `su/[slug]/page.tsx` generateMetadata (BASE_URL).
- Notifications: `notification.tsx:34-96`, `notificationsModal.tsx:84-98`, `SiteHeader.tsx`.
- Leads: `dashboard/leads/[id]/page.tsx` redirect stub, `LeadsDashboard.tsx`.
- Auction: `auction-view.tsx`.

### BO (admin_bo_tawadoo) — moderation is where flags are set
- `approvePublication` counterpart in BO (the human Verify action) — ensure it sets the canonical flag(s) so D1 stops being produced.

### App + MCP — conform in their OWN units (Artifact 6), not this migration
- ui_only: RootNavigator deeplink, share.ts, ProductDetail resolve, FCM push consumer.
- MCP: `_build_product_url` (client.py:445), config.py:184 — only if URL shape changes (it should NOT).

---

# ═══════════ ARTIFACT 6 — APP / MCP COMPATIBILITY NOTES ═══════════
*(contracts the canonical model MUST preserve or deliberately version — inventory only, NO implementation)*

**MUST PRESERVE (breaking these breaks live ChatGPT + app):**
1. **URL shapes** `/{locale}/p/{slug}` + `/{locale}/su/{slug}` — hardcoded in MCP (`CLIENT_PRODUCT_PATH_TEMPLATE`, config.py:184) AND app (deeplink regexes + `buildShareUrl`, RootNavigator/share.ts). Changing the `p`/`su` segment or slug-vs-id breaks both simultaneously.
2. **Dual resolution** `publications/slug/:slug` (primary) + `publications/:id` (fallback) — app relies on both; push uses `publicationId`. UUID must remain a valid alias.
3. **`entities/slug/:slug`** for store/profile (app).
4. **`GET /publications/search`** param contract (`search_text, cat/scat/scat2, minp/maxp, properties=\d+:\d+, op, page, limit`) — MCP + app + web all share it.
5. **UUID as durable listing id** — MCP uses it for `utm_content`; app uses it for `openProductById`/`publications/{id}` fallback. If listings ever move off UUID, MCP attribution + app fallback need versioning.
6. **Seller entity = UUID** — MCP `tw_seller` = first 8 hex chars.

**MAY VERSION (deliberately, in a dedicated app unit — NOT silently):**
7. **FCM push payload** `{link:<web url string>, publicationId:<id>}` — the app does **not** consume the structured `deepLink{webUrl,appUrl,fluidUrl}` the API already produces. If Stage-2 moves notifications to the structured target shape (Artifact 4.7), the app must be updated in lockstep; until then, keep emitting `link` + `publicationId`.

**SUPERSEDED (not a consumer to preserve):**
8. `tawadoo_mobile_app` (legacy) is on a **different backend** (`preprodapi.tawadoo.com/v1/api/`, Symfony IRIs, id-only, no slug/deeplink). A canonical change on the modern API does not reach it. Treat as superseded; do not design for it.

**⚠️ Flag (do NOT design around silently):** any Stage-2 change that alters the FCM push key names or the `/p//su/` URL segments **will** break the live app and/or ChatGPT links — record as a coordinated cross-repo item, founder-gated.

---

# ═══════════ ARTIFACT 7 — ARCHITECTURAL DECISION (Stage-2) ═══════════

### WHAT to build in Stage-2 (S-CTO-26-NAV-FIX)
A **small, explicit canonical mechanism**, in this order:
1. **ONE visibility predicate** (`isPubliclyVisible`) reused by public detail (SSR+client), keyword+vector search, index-write, feed eligibility — collapsing the `isVerified` vs `isModerated` split. **[the core fix — closes F-A/B/C/J]**
2. **Data reconcile** for `isModerated=null`-on-verified rows (forward-fix on the record, not a projection patch).
3. **ONE api notification-target shape** `{entityType, entityId, url}` with real ids (no literal `objectId` strings, no empty targetId) — plus a web consumer that routes from `{entityType, entityId}` without a failure-prone re-derivation. **[closes F-D/E/F/G — FR-9]**
4. **ONE shared web link helper** `buildEntityUrl()` + ONE base-URL source of truth; migrate the inline card/share/OG/feed builders to it; fix `null`/`''` slug fallbacks. **[closes F-H/I/K/L/M/N — P2]**

### WHERE it lives
- Core visibility predicate + resolution + notification target = **`tawadoo_api_js` (System-of-Record layer)** — this is the authoritative layer per the founder constraint and target-arch §4. `index-eligibility.ts` is the existing seam.
- The link helper = **`tawadoo_web_js`** (a small `src/utils/entityUrl.ts`, consuming the same shapes the api/MCP/app already use).
- Flag reconciliation touches **BO** (the human Verify action) so the anomaly stops being produced.

### WHY (highest leverage)
- P1 (visibility) is the single largest "item unavailable" cluster and is proven live. Fixing the predicate once, in the record layer, makes every surface agree — vs patching each link (which P2 shows never converges).
- Preserving the URL/resolution contracts (Artifact 6) means the fix is invisible to ChatGPT + the app.

### What Stage-2 explicitly does NOT include
- No route framework / `UniversalEntityRouter`. No Smart+Classic merge. No DB redesign / new slug column / design-system change. No app or MCP code (those conform in their own later units). No weakening of the moderation gate (X6) or any authz. No reopening the parked FR wave beyond the FR-9/FR-11 items that this contract subsumes.

### Classification
- **RED (blocks a clean launch):** P1 visibility divergence (F-A/B/C) + FR-9 notification target (F-D). These make live, real listings/leads unreachable.
- **YELLOW (high-leverage, small):** D2 vector filter (F-J), the shared link helper + null-fallback fixes (F-H/I/L), notification `objectId` real-ids (F-E/F/G).
- **BLUE (post-launch/hygiene):** base-URL single-source (F-M/N), FeaturedStores name→slug (F-K), auction dual-addressing tidy (E10).

- **R0 (touch now):** unify the visibility predicate + reconcile null rows (the record-layer contract).
- **R1 (this refactor):** notification target shape + web link helper.
- **R2 (deferred debt):** consolidate the 3 base-URL sources; auction addressing consolidation.
- **R3 (cosmetic):** none blocking.

### HONEST UNKNOWN-BLOCKED list
- **[DB] Blast radius of `isModerated=null`** across all published+verified rows — psql in-VPC/IP-locked, ECS-Exec down. One live listing confirms the class; the *count* is UNKNOWN. Stage-2 must measure it via an authorized DB path (or a bounded API sweep) before the reconcile.
- **[DB] Uniqueness of `PublicationTranslation.slug`** — no unique index declared in the entity; whether a migration enforces it is UNKNOWN (not read). Slug collisions would be a latent STATE-7 source.
- **[runtime] Browser desktop/mobile (Chromium+WebKit) visual proof** of the 404/200 divergence was not run (HTTP-level proof captured instead); a Stage-2 browser check should confirm the user-visible symptom.
- **[api] Whether any downstream merge re-checks the vector-leg source** to mitigate D2 — not fully traced past `executeVectorSearch`.

### → Stage-2 prompt is now OBVIOUS:
Build the unified `isPubliclyVisible` predicate + reconcile the null rows + the `{entityType, entityId, url}` notification target + the shared web `buildEntityUrl` helper, in the api+web(+BO flag), preserving the `/p//su/` URL and `publications/slug|:id` contracts, with a regression matrix: search→detail parity (slug==lightssr), share→fresh-browser, notif→real target, owner→own-pending-preview.

---

## COMPLETION CHECKLIST (1:1 with controlling prompt)
- [x] Preflight HEADs verified; anchors + entity list source-confirmed
- [x] 10 (+seller-profile=11) entities traced end-to-end (source + live staging HTTP; note: fresh-browser Chromium/WebKit visual test not run — HTTP-level proof captured, tagged UNKNOWN-BLOCKED for the visual layer)
- [x] Artifact 1 Addressing Matrix (source-only + live values)
- [x] Artifact 2 Failure Matrix (every failure → STATE 1–12 + file:line)
- [x] Artifact 3 Root-cause patterns (shared P1/P2/P3 vs distinct, evidence-decided)
- [x] Artifact 4 Canonical target model (system-of-record-derived, simple, no framework)
- [x] Artifact 5 Migration surface (all callers)
- [x] Artifact 6 App/MCP compat notes (contracts to preserve/version)
- [x] Artifact 7 Stage-2 architectural decision (what/where/why + RED/YELLOW/BLUE + R0–R3 + UNKNOWN-BLOCKED)
- [x] Zero product code changed; only evidence file written; own-session status only

## OWN-SESSION STATUS (proposal only)
**S-CTO-25-NAV-MAP → PROPOSED: FINISHED — COMPLETE (for the MAP).** Read-only, zero product code changed, all HEADs unchanged. Independent Brain QA owes acceptance per §18 (re-verify from source + live: the slug/lightssr 200/404 divergence, the isModerated=null live fact, D2 at hybrid-search:443, FR-9 at leads.service.ts:691, and the MCP/app URL-shape contracts). One partial: the user-visible symptom was proven at HTTP level, not via a fresh Chromium/WebKit browser render (tagged UNKNOWN-BLOCKED).
