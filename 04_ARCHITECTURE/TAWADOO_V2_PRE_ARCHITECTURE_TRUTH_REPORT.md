# TAWADOO V2 — PRE-ARCHITECTURE TRUTH REPORT

**Date:** 2026-09-01
**Author:** Brain B16 (final discovery gate, MODE A)
**Method:** Verified from live git, live AWS (S3/RDS/ECS/CLI), HTTP probes, and source. Read-only. No prod mutation, no repo writes, no migrations, no refactoring. Where a fact could not be established read-only, marked UNKNOWN-BLOCKED.
**Governing rule:** VERIFY THE SYSTEM, NOT THE STORY. Do not discover forever — close gaps, then synthesize.
**Hard stop:** This report ends discovery. Next phase is Architectural Synthesis — awaiting CTO/founder review. NO refactoring begins from this report.

**Three perspectives kept distinct (never conflated):** PRODUCTION (trusted, running today, `main`) · RAMZI_V2 (evolved refactor line, staging) · TARGET (to be designed after synthesis).

---

## 1. PRODUCTION BASELINE
- Branch `main`. Running images: api `:0.1.419`, web `:0.1.938`, bo `:0.1.62`, mcp ECR `:5ad5bc5`. ECS prod all 1/1 healthy: back task-def :101, front :170, bo :48, mcp :11.
- **Schema strategy: `synchronize: true`** (verified `git show origin/main:src/app.module.ts`) — TypeORM auto-syncs schema from entities at boot; NO formal migrations. This is why prod never hit the migration-privilege crash.
- Release: version tags (`0.1.x`, CI run-number based) + PR flow (commit subjects `(#7)(#8)(#9)`). Disciplined.
- DB: `tw-postgres-cluster-prod` Aurora PG 16.11, 1 member (NOT Multi-AZ), 7-day backup, encrypted, private.
- Endpoints: www 307 (alive), api 404 root (behind CloudFront, no public health path), bo 000 (IP-restricted, correct), mcp 404 root (protocol-only).

## 2. RAMZI_V2 BASELINE
- Branch `Ramzi_V2` (staging/refactor line). api HEAD `88908dd` (post-S139 revert, == cd5b8de tree), web `735690d8`, bo `ffde480`, mcp `d8efb4a`.
- **Schema strategy: `synchronize: false` + `migrationsRun: true`** — runs formal migrations at boot. This CRASHED in S138 because the runtime DB user lacks table ownership (DDL). CONFIRMED platform-wide (ta_entity/ta_message/ta_lead log "must be owner" as caught WARNs; only migrationsRun turns it into a crash).
- ECS staging all healthy: front :18, back :44 (:staging-v2 → digest d3c09fff, reverted no-migration image), bo :39, mcp :5, sgtm :2.
- Built on the shared human-CTO base (merge-base 317cf774, an ancestor of Ramzi_V2) + ~623 Kiro-era commits.

## 3. PRODUCTION DRIFT — COMPLETE CLASSIFICATION
Merge-base 317cf774. Drift counts: api main+8, web +38, bo +1, mcp +0.

### API (8 main-only) — verified by subject match
| Commit | Behavior | Class | Destination | Risk |
|---|---|---|---|---|
| a0a5ac2 sitemap endpoint | PG-direct sitemap | ALREADY IN RV2 (364197e) | — | none |
| a1ab58e posts PUBLISHED filter | status filter | ALREADY IN RV2 (ed0a4c5) | — | none |
| d58813c CONTENT_ENGINE_ENABLED kill-switch | cost guard | ALREADY IN RV2 (9145a39) | — | none |
| dd7cdf3 saved_search_match disable | spam/email fix | ALREADY IN RV2 (a115601) | — | none |
| **6e06278 JwtAuthGuard on message+report endpoints** | auth guard on unprotected endpoints | **MUST PORT** | message + report controllers | **SECURITY (unauthed access)** |
| **1c714d6 Cache-Control no-store on message endpoints** | prevent proxy caching private data | **MUST PORT** | message controller | **SECURITY (private-data cache leak)** |
| c06a305 sitemap isModerated removal | prod uses published status | MUST PORT (verify) | sitemap/publications service | low (data-correctness) |
| a1987ae blog article generation restore (July incident) | content-engine fix | MUST PORT (verify likely-covered) | content-engine | low |

### WEB (38 main-only) — classified by subject match
- **ALREADY IN RV2 (10):** aba22453 sitemap use-endpoint, cae7020f sitemap status filter, c3d3a7ab footer box1 aspect, c74ccbb9 hide SSR article list, 59bc51cc unify footer, 990a666b footer 404 fix, 364322a3 footer merge, 71015f91 ChatGPT app badge, ad0f90b8 hide parent categories disambiguation, 1eac66a0 categoryId int parse, 520784f5 notifications unavailable route.
- **PORT?-CANDIDATES (28)** — cluster into themes; classify precisely in cutover phase, NOT now:
  - **SEO (3):** a6499893 IndexNow, 6e019f8e Array.from, 30ec65cd unicode-safe truncation → likely MUST-PORT (SEO correctness).
  - **Performance (9):** perf(tbt) GA4 lazyOnload, perf(lcp) AppPromoSection server component, perf(images) lazy, perf(css) tailwind, perf(lcp) hero — plus a Revert of one lazy-load. → likely INTENTIONAL DIFFERENCE or SUPERSEDED (RV2 may have its own perf posture); VERIFY.
  - **App Banner + Deep Links (10):** banner v2, Firebase deep links, iOS Universal Links Team-ID, white-screen fix, deeplinking. → INTENTIONAL DIFFERENCE candidate (RV2 is web-only, app not synced yet) — VERIFY relevance.
  - **Analytics (3):** dual-tag web events into unified lake, stop dropping early GA4, rename platform→detected_platform. → likely MUST-PORT (sovereignty/event correctness).
  - **Blog/footer (3):** blog P0 i18n hotfix, trust status not legacy publish, footer privacy→/cgu 404 fix. → VERIFY (may overlap RV2 footer work already IN-RV2).
- **NET:** the security-relevant + sovereignty-relevant web commits (analytics dual-tag, SEO) are the priority to classify; perf/banner are likely intentional-diff/superseded given RV2's evolution.

### BO (1 main-only)
- `c79c922` fix(posts): make status the real source of truth (admin writes too) → MUST PORT (verify) — data-correctness, aligns with the API posts-status fix already in RV2.

### MCP (0 main-only)
- CONFIRMED clean. No drift. MCP prod == a subset of Ramzi_V2 history.

**§28 verdict:** the real cutover port list is SMALL and concrete — **2 confirmed API security commits (6e06278, 1c714d6)** first, then ~6-10 verify-and-port web/api/bo items (SEO + analytics-sovereignty priority). NOT a platform risk. Full per-commit web classification deferred to the cutover phase (not this gate).

## 4. RUNTIME TRUTH (critical paths — BUILT/TESTED/STAGING-PROVEN/PROD-PROVEN)
| Path | State | Evidence |
|---|---|---|
| **Search** (query→retrieve→rank→result→event) | **PROD-PROVEN** | HybridSearchService (keyword+vector+suggested-cat, video-boost, ghost-penalty, keyword fallback); running on prod api :0.1.419; search_performed events flowing to lake (analytics-events populated daily). |
| **Sovereign Data** (event→writer→outbox→S3 lake→training) | **PROD-PROVEN + verified live** | See §5 — daily NDJSON export 08/28-08/31 at 01:00 UTC, lock present, ai_outputs written today. |
| **Coins** (wallet→debit→lock→ledger→consequence) | **BUILT + TESTED, staging-proven** | wallet.service debitWithLock (pessimistic), wallet-security.spec + coin-package-security.spec green; running on staging; prod-proven inferred (prod runs same core) but NOT independently transaction-tested this session → mark STAGING-PROVEN. |
| **Commerce** (listing→txn→order/offer/auction→pay/COD→delivery) | **BUILT; SPLIT proof** | Fixed-price/offer/auction all built; orders COD→Tawssil built but Tawssil NEVER carried a real parcel (no live creds) = BUILT-UNPROVEN for the delivery leg; payment (Payzone) prod-proven (live money runs on prod). |
| **Video** (upload→storage→process→output→playback) | **BUILT, staging-proven** | file.service ffmpeg compress+thumbnail, video.service validation+moderation, IVS live; runs; no HLS/adaptive-bitrate (not built — not a gap, a design choice). |
| **Distribution** (listing→readiness→feed→channel→provider) | **BUILT-UNPROVEN (held)** | feed-generator + quality-gate + formats built; HELD by per-channel env flags (GMC_SYNC_ENABLED false); GMC SUSPENDED (appeal); feeds not sent = BUILT, operationally UNPROVEN end-to-end. |

## 5. DATA TRUTH — SOVEREIGN LAKE (mandatory gate, VERIFIED LIVE)
Bucket `s3://tawadoo-core-intelligence-lake` (eu-west-1) — EXISTS, populated.
- **Latest export:** `analytics-events/2026/08/31/events.ndjson.gz` written **2026-09-01T01:00:01** (90KB gz). **The daily cron IS RUNNING** at the scheduled 01:00 UTC.
- **Continuity:** 08/28 (16:41, backfill), 08/29, 08/30, 08/31 — continuous daily, each with a `_lock` object (distributed lock working = idempotency proven).
- **Structure:** partitioned `analytics-events/YYYY/MM/DD/events.ndjson.gz`; prefixes `ai_outputs/` (TrainingShadow — 08/29→09/01, written TODAY), `weekly-refresh/`, `fresh-counts/`, `_db_schema/`, `archive/`, `_scripts/`.
- **IAM behavior:** the B13-S110 "broken IAM / manual trigger" concern is **RESOLVED** — exports run automatically on schedule. (Historical: IAM was once broken; now working.)
- **Training-ready data:** ai_outputs actively populated (shadow AI calls). Reward-tuple COMPLETENESS (prompt+response+conversion for Smart View) — NOT deep-inspected this session; latest ai_output is 513 bytes (single shadow row) → training PAIRS assembly is a separate question for the synthesis phase. Mark: LAKE HEALTHY + POPULATED; tuple-completeness for fine-tuning = PARTIAL/UNKNOWN.
- **Retention/partitioning:** monthly PG partitions on ta_analytics_event; S3 daily partitions; retention policy not read this session (UNKNOWN).

**Verdict: the sovereign lake is REAL, LIVE, CONTINUOUS, and IDEMPOTENT.** B15's "training data extremely weak" was about tuple richness for fine-tuning, NOT lake health — the lake pipeline itself is proven working.

## 6. INFRASTRUCTURE TRUTH
- **IaC: PARTIAL.** `tawadoo_web_js/infra/cloudfront/` Terraform (cloudfront/waf/acm/variables/outputs/versions) — committed Ramzi_V2 02df7ca9, "review only, NOT applied". Covers ONLY web CDN. ECS/RDS/Redis/ALB/task-defs/secrets/alarms = hand-managed, no IaC.
- **Compute:** ECS Fargate, 2 clusters (prod, staging). Images: web/api/bo via GHCR, mcp via ECR.
- **Data:** Aurora PG 16.11 (prod + staging, single-member each, encrypted, 7d backup, NOT Multi-AZ). Redis 7.0.7 ElastiCache (Bull queues).
- **Deploy:** push Ramzi_V2 → GHCR/ECR → `update-service --force-new-deployment`. Staging automated; prod manual. §42 SHA-pin trap fixed in S139 (staging back now on mutable :staging-v2).
- **Alarms:** Bedrock-High-Daily-Usage (ALARM, expected — founder staging testing), cron dead-man alarms OK, CPU alarms.

## 7. EXTERNAL PROVIDER TRUTH (matrix)
| Provider | Purpose | Code | Auth (secret) | Runtime | Staging | Production | Last Proven | Failure Handling | Risk |
|---|---|---|---|---|---|---|---|---|---|
| Amplitude | product analytics | delivery-worker | tawadoo-*/env | outbox dest | ✅ | ✅ | live (events flowing) | retry/DLQ | low |
| Meta CAPI | conversions | meta-capi.service | Meta token | fire-forget | ✅ | ✅(prod) | inferred | non-blocking | low |
| TikTok Events | events | tiktok-events.service | tiktok-marketing | fire-forget | ✅ | ✅(prod) | inferred | non-blocking | low |
| WhatsApp | 19+ templates + webhook | whatsapp-bridge | whatsapp-token | dispatcher | ✅ | ✅(prod) | live | non-blocking | med (template approval §30) |
| SendGrid | email | email.service | sendgrid-api-key | SDK | ✅ | ✅(prod) | live | degrades if no key | low |
| FCM/Firebase | push | notification | google-app-creds | admin SDK | ✅ | ✅(prod) | inferred | non-blocking | low |
| GA4 | server analytics | ga4-measurement | ga4-credentials | mp/collect | ✅ | ✅(prod) | inferred | non-blocking | low |
| GSC | SEO enrichment | seo-enrichment | GSC service acct | @Cron | ✅ | ✅(prod) | cron | isolated | low |
| Bing/IndexNow | instant indexing | (bing-webmaster) | bing-api | ping | ✅ | ✅(prod) | inferred | non-blocking | low |
| GMC | Shopping feed | google-merchant | merchant-creds | held | ❌(flag off) | ❌ SUSPENDED | appeal pending | held by flag | HIGH (suspended) |
| Google Ads | campaign data | content-engine/enrich | google-ads-oauth | cron | ✅ | ✅(prod) | cron | isolated | low |
| Meta/TikTok Catalog | product feeds | syndication formats | tokens | held | ❌(flag off) | ❌ | held | env flags | med (unproven) |
| ChatGPT App | in-ChatGPT widget | mcp/ui/widget | apps-challenge-token | gated | ✅ | ? | UNKNOWN | config-gated | med (prod status unknown) |
| MCP | agent tools | FastMCP server | metrics secret | SSE | ✅ | ✅(mcp.tawadoo.ma responds) | live | error payload | low |
| OpenAI | tier-2 LLM | guidance fallback/api | openai-api-key | SDK | ✅ | ✅(prod) | inferred | static fallback | low |
| Bedrock | LLM+embeddings+moderation | bedrock.service | IAM | Converse/Invoke | ✅ | ✅(prod) | live (lake ai_outputs) | staticFallback | low |
| Payzone/PayExpress | payments | payexpress.service | payzone-* | callback | ✅ | ✅ REAL MONEY | live | signature-verified | HIGH (money) |
| Tawssil | COD delivery | tawssil module | (no live creds) | webhook | built | ❌ NEVER a real parcel | never | quote non-blocking | BUILT-UNPROVEN |
| AWS IVS | live streaming | ivs.service | IAM | SDK | ✅ | ? | UNKNOWN | — | med |
| sGTM | server tag mgr | ECS sgtm | — | idle | idle | ? | idle | — | low |

## 8. EXPERIENCE TRUTH
- Classic View (search/listing/dashboard/orders/auth) — PROD-PROVEN, sacred (Commandment 1).
- Smart View — BUILT on staging (Luna+MCP), UX fixed in S137; NOT on prod; guest no-auth + no cost-cap are INTENTIONAL staging diffs (H2/H3, cutover checklist).
- Grid-vs-feed, blog, stores — built; feed had regressions (fixed/reverted).

## 9. COMMERCE TRUTH
- Two orthogonal axes: Distribution Model (buy_now/lead_gen/rental from category.storeTrack) × Transaction Mode (fixed/negotiable-offer/auction-BID). VERIFIED.
- buy_now→Orders→COD→Tawssil (3-confirmation); offer module (accept/counter); auction (bidRoom+bidEntity+bidTransaction, escrow, anti-sniping). Feed-quality-gate excludes bid/offer from external feeds (§50).
- Payment (Payzone) prod-proven with real money. Delivery (Tawssil) BUILT-UNPROVEN (no live parcel).

## 10. SEARCH / AI TRUTH
- HybridSearchService: keyword (OpenSearch publications_2) + vector (Bedrock 1024-dim, publication-embeddings) + suggested-category, merge/score, video-boost, ghost-penalty, keyword-only fallback on Bedrock failure. PROD-PROVEN.
- Reindex via atomic alias swap. Darija dictionary. Image search (k-NN via embeddings).
- Smart View brain = Luna via Bedrock + MCP SSE tool loop (staging).

## 11. VIDEO TRUTH
- THREE systems (NOT a transcoding farm): (1) video.service listing-video validation+moderation, (2) file.service ffmpeg compress+thumbnail at upload, (3) syndication store-video analysis→AI caption→post (held), (4) IVS live. media_module = generic records. No HLS/adaptive-bitrate/VMAF (design choice, not gap).

## 12. SOVEREIGNTY TRUTH
- Single writer ta_analytics_event (partitioned monthly), allowlist ~585+ events (478 stale), deterministic sha256 event_id, dedup ledger, _is_canonical=false for unknowns. Outbox ta_analytics_delivery → delivery-worker (lease/backoff/DLQ) → Amplitude. Lake export daily NDJSON (VERIFIED LIVE §5). PROD-PROVEN.

## 13. BUILT-BUT-UNPROVEN (capability exists, operational proof missing — NOT auto-debt)
- **Tawssil COD delivery** — full module, zero real parcels, no live creds.
- **Distribution feeds (GMC/Meta/TikTok catalog)** — built + gated off; GMC suspended; never sent end-to-end.
- **Store-video → social reels** — built, posting held by env flag.
- **Smart View on production** — built on staging only.
- **Reward-tuple assembly for fine-tuning** — shadow rows written; prompt+response+conversion PAIR assembly unproven.
- **CloudFront Terraform** — written, "not applied".
- **ChatGPT App widget prod status** — UNKNOWN.
- **IVS live prod usage** — UNKNOWN.

## 14. TECHNICAL DEBT
- **H2b (top): DB migrator model** — Ramzi_V2 migrationsRun:true as non-owner crashes; prod uses synchronize:true (unsafe target). Needs proper migrator-identity separation. BLOCKS all schema work.
- No IaC for ECS/RDS/etc (partial CloudFront only).
- BO + MCP repos lack READMEs.
- Ramzi_V2 uses direct pushes (no PR flow); prod uses PRs.
- Staging mutable tags vs prod version tags — standardize immutable digests.
- Duplicate migration timestamps (pre-existing).
- Smart View sovereignty logs after LLM call (not pre-call).

## 15. SECURITY RISKS
- **2 unported prod security fixes** (6e06278 JwtAuthGuard on message/report; 1c714d6 Cache-Control no-store) — MUST verify Ramzi_V2 has equivalent, else PORT.
- H2/H3 intentional staging diffs (guest no-auth Smart View image search, no AI cost cap) MUST be closed at prod cutover.
- Runtime DB user cannot DDL (good security posture, but couples to migrator problem).
- Secrets in Secrets Manager (no hardcoded found). BO IP-restricted.

## 16. OPERATIONAL RISKS
- GMC suspended (revenue/distribution channel down).
- Prod Aurora single-instance (no Multi-AZ failover).
- Prod api no public health path (observability gap).
- Distribution held entirely by env flags (fragile if flipped without readiness).

## 17. PERFORMANCE RISKS
- No measured baselines yet (API p50/p95, search latency, hybrid vector vs keyword, Luna+MCP round-trip 7-12s observed in S133). Establish in synthesis phase.
- Bedrock daily usage alarm firing (staging testing cost).

## 18. ARCHITECTURAL COUPLING
- Everything writes to the sovereign lake (intended coupling — the moat).
- Syndication module is large (feeds + store-video + readiness + social posting) — internal coupling candidate to decompose.
- searchEnrichment holds search + trending + training-logger + slug-resolver — mixed concerns.
- BedrockService is a shared dependency (LLM + embeddings + moderation + training shadow).
- Wallet is the financial authority all coin paths depend on.

## 19. ARCHITECTURAL SEAMS (candidate boundaries — NOT service decree; heed anti-over-architecture)
Applying the test (business responsibility / data ownership / transaction boundary / consistency / scaling / failure isolation / deploy independence / does it NEED to be a service):
- **Sovereign Data Layer** → its own boundary (library + async workers + S3). It IS the moat; must be isolated but is NOT a request-path service. Outcome: **bounded context + workers**, not a microservice.
- **Search/AI** → heavy, distinct scaling (OpenSearch + Bedrock), could justify a **service** IF latency/scaling evidence demands; otherwise a **module + worker**. Decide on evidence.
- **Distribution/Syndication** → async, provider-facing, failure-isolated, held by flags → strong **worker/adapter** candidate (already queue-based).
- **Financial (wallet/coins/payment)** → strong consistency, one transaction boundary → **modular core, NOT split** (splitting money across services adds distributed-transaction risk).
- **Commerce (listing/order/offer/auction)** → core domain, shared transaction with wallet → **modular core**.
- **External Providers** → **adapters** (Meta/Google/TikTok/Payzone/Tawssil/WhatsApp) behind interfaces.
- **Video** → ffmpeg/IVS, distinct resource profile → **worker**.
- **Content Engine** → scheduled, budget-gated → **worker**.
- **PROPOSED SHAPE (hypothesis, evidence-led):** ONE MODULAR CORE (auth/commerce/wallet/listing) + SPECIALIZED WORKERS (search-index, distribution, video, content, lake-export) + SOVEREIGN DATA LAYER + EXTERNAL ADAPTERS + a FEW justified services only where scaling/failure-isolation evidence demands. NOT 28 microservices.

## 20. INVARIANTS (must NOT break — evidence-based)
- User identity/auth (JWT/OTP) — sacred.
- Wallet/coins integrity — no race (pessimistic lock EVERY debit), caps enforced.
- Payment integrity — signature-verified callbacks, no double-credit.
- Order integrity — state machine, 3-confirmation, COD-only.
- Event sovereignty — every interaction → lake FIRST, single writer, idempotent, no PII.
- Search behavior — Classic results reproducible; keyword fallback on vector failure; alias-swap reindex.
- Listing integrity — distributionModel derived from category; bid/offer excluded from fixed-price feeds.
- Media ownership — watermarked url to Tawadoo, clean url to feeds.
- Distribution — silently fail to Tawadoo+ChatGPT, never block user (§50).
- Production isolation — prod protected; Ramzi_V2 the future line.
- Deployment safety — immutable provenance; §42 no sha-pin no-op; rollback to prior digest.
- Rollback — proven (S139).

## 21. UNKNOWNS
- Exact prod version-tag → git SHA (CI run-number tags, needs registry label).
- Reward-tuple completeness for fine-tuning (lake healthy, pairing depth unknown).
- Lake retention policy (not read).
- ChatGPT App widget prod status; IVS live prod usage.
- Full per-commit classification of 28 web PORT?-candidates (themed, not individually confirmed).
- Performance baselines (unmeasured).

## 22. CONFLICTS
- Prod `synchronize:true` vs Ramzi_V2 `migrationsRun:true` — divergent schema strategies; neither is the correct target.
- Prod PR flow vs Ramzi_V2 direct-push.
- Prod version tags vs staging mutable tags.
- (Resolved earlier) "no IaC" vs partial CloudFront Terraform; "8 human commits at risk" vs Kiro hotfixes mostly ported.

## 23. ITEMS REQUIRING FOUNDER DECISIONS
1. H2b migrator model: adopt migrator-identity separation (synchronize:false + migrationsRun:false + bounded owner-cred migrator)? [recommended]
2. Prod Aurora Multi-AZ — enable before cutover? (cost vs resilience)
3. GMC suspension — path (appeal outcome dependent).
4. Tawssil — go-live decision (waiting Yassine) or design-only.
5. Distribution reactivation — after readiness + clean images (H2b + backfill first).
6. Architecture shape — approve the "modular core + workers + adapters" hypothesis as the synthesis starting point (NOT 28 services)?

## 24. CANDIDATE TARGET ARCHITECTURE PRINCIPLES (hypotheses, evidence-led)
1. ONE modular core (identity/commerce/wallet/listing) — strong consistency, single transaction boundary; do NOT split money/commerce across services.
2. Specialized workers for async/scaling-distinct concerns (search-index, distribution, video, content, lake-export).
3. Sovereign Data Layer as an isolated boundary (the moat) — every write goes through it, first.
4. External providers behind adapters (uniform failure handling, §50 silently-fail).
5. Proper migrator separation (D-INFRA) — the FIRST enabling change; no schema work until done.
6. IaC for ECS/RDS/etc (build on the CloudFront Terraform); immutable digests both envs; PR flow.
7. Evidence over elegance — a boundary becomes a service ONLY with scaling/failure-isolation proof. Default to module/worker/adapter.
8. Preserve every §20 invariant; Classic View sacred.

## 25. PROPOSED FIRST ARCHITECTURAL DECISION
**AD-001: Establish the migrator/runtime credential separation (H2b) as the foundational D-INFRA change — before any schema-touching refactor.**
- Rationale: it is the confirmed platform-wide blocker (S138 crash, prod's synchronize:true is an unsafe workaround, every future migration crashes without it). It unblocks ALL domain refactors that need schema changes (seoTitle, image backfill, feed-safety re-land).
- Shape: app boots `synchronize:false` + `migrationsRun:false`; a dedicated migrator identity (table-owner creds) runs migrations via a bounded ECS task / CI step with rollback; verified on staging first.
- This is a PROPOSAL for founder review, NOT an implementation. No code, no migration, no infra change is made from this report.

---

## HARD STOP
Discovery is closed. This report is the evidence base for Architectural Synthesis. NO refactoring, NO architecture implementation, NO production change, NO migration, NO large refactor branch until CTO/founder review approves the synthesis. Next: TRUTH → ARCHITECTURAL SYNTHESIS → TARGET ARCHITECTURE → MIGRATION STRATEGY → APPROVAL → BOUNDED REFACTORING.
