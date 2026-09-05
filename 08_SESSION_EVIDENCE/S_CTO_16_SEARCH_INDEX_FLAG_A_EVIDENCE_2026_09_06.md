# SESSION ID S-CTO-16 — SEARCH-INDEX FLAG-A DIAGNOSIS (READ-ONLY)

**Date:** 2026-09-06 · **Status proposed:** FINISHED — COMPLETE (verdict rendered; one gate BLOCKED, documented) · **Mode:** READ-ONLY diagnosis, no code/DB/index mutation.
**Founder:** Ramzi. **Skills:** tawadoo-source-truth + tawadoo-runtime-provider-qa.
**Evidence ladder reached:** source (CONFIRMED) · live public-API (CONFIRMED) · direct DB/OpenSearch read (BLOCKED — no mutation-free path).

---

## FIVE-LINE OPENER (actual)

```
SESSION: S-CTO-16 — SEARCH-INDEX FLAG-A DIAGNOSIS (READ-ONLY)
BRANCH: Ramzi_V2 (tawadoo_api_js) — verified, clean tree, HEAD bd5cf2a6ddb78b2b5aef5d53847bab2c8d81a753
MISSION: confirm/refute — do photoless job/service listings get indexed into search? If not, the exact cause. NO fix.
ORDER: source trace (index/eligibility rule) → live probe (search a known photoless listing) → root-cause verdict → evidence
TREE STATE: clean; touched NOTHING but this evidence file
```

Nothing was changed. No commit, no deploy, no prod, no reindex/backfill, no browser sweep. `Ramzi_V2` HEAD unchanged.

---

## THE OBSERVATION UNDER TEST (FLAG-A, single, unverified)

S-CTO-R-RENDER observed that a **photoless SERVICE** the seller published did NOT appear in `/publications/search` (0 photoless returned). Hypothesis: photoless job/service listings are not discoverable by buyers → undercuts the whole image-optional feature.

---

## 1. SOURCE TRACE — does anything require ≥1 image to be INDEXED or to appear in /publications/search?

Layers are distinct: **index-eligibility** (does a doc get written to OpenSearch) vs **search-query filters** (does an existing doc surface in results) vs **feed-eligibility** (external channels) vs **render** (UI). FLAG-A is about the SEARCH INDEX + SEARCH QUERY.

### 1a. Index-eligibility rule — NO image required
`tawadoo_api_js/src/modules/publication/index-eligibility.ts:29` — `isIndexEligible()`:
```
isModerated === true && status === PUBLISHED && deletedAt == null
```
The canonical `INDEX_ELIGIBILITY_WHERE` (same file, line ~19) is identical. **No image / photo condition anywhere.** This is the single choke point every index write flows through.

### 1b. Index-write path — NO image gate
`publication-search.service.ts` `reindexOne()` (the single choke point, called by `pushToIndex`): if `!isIndexEligible(pub)` it REMOVES the doc, else it indexes/updates. It never inspects images.
`publication-index-body.ts` `buildPublicationBody()` serializes `images: (publication.images || []).map(...)` — an empty array is fine; **no image gate, no throw** for a photoless listing.

### 1c. Search-query filters — NO image filter (both search paths)
- **Legacy / filter-only + count + random path:** `publication.service.ts` `buildEsFilters()` (line 1674+). Hard filters pushed: `{ term: { status: 'published' } }` and `{ term: { isVerified: true } }` (lines 1674–1675). Optional filters: bid window, text multi_match, location, category, price, store, newProduct, announceType(=type), storeType, properties, hasVideo. **There is NO image/photo filter.** (Same two hard filters also appear at lines 1302, 1485, 2568, 2626, 4538, and the `andWhere(isVerified=true)` DB paths at 4287/4582.)
- **Text-search path (hybrid):** `searchPublications()` routes any `searchText` query to `searchPublicationsHybrid()` → `HybridSearchService`. `hybrid-search.service.ts:313`: hard filters are exactly `[{ term: { status: 'published' } }, { term: { isVerified: true } }]`. Image count is used ONLY as a soft quality **boost** (lines 628–638: `imageCount >= 3 → +0.04`, `>= 1 → +0.02`), never as a filter. A photoless listing simply gets `+0` boost — it is NOT excluded.

**Conclusion of source trace:** No layer — index-eligibility, index-write, legacy search, or hybrid search — requires an image for a listing to be indexed or to appear in `/publications/search`.

### 1d. The only non-obvious hard gate is `isVerified=true` — and it cannot desync from moderation
`isVerified` is distinct from `isModerated`, so in principle a moderated-but-unverified doc would pass the index gate (`isModerated`) yet be filtered out of search (`isVerified`). Verified this cannot happen:
- `moderation-state-machine.ts:143` — `approved` → `{ status:'published', isModerated:true, isVerified:true }` (set as one unit).
- `publication.service.ts:3730` — `approvePublication()` sets `isModerated=true` AND `isVerified=true` together, then pushes to index.
- `publication.service.ts:3415` — reject/edit sets both back to `false` together.
So an approved photoless job/service is BOTH moderated and verified → it passes the index gate AND the search `isVerified` filter. No photo-independent desync exists in source.

### 1e. Publish path explicitly supports photoless jobs/services (image-optional)
- `publication.service.ts:3470` `isImageOptionalCategory()` — TRUE only when the listing's ROOT category code is `jobs` (id 85) or `services` (id 84); keyed off root category CODE, not `store_track` (so vehicles/real-estate stay image-required).
- `autoFixMainImage(pubId, imageOptional=true)` does NOT throw `PUB_NO_IMAGES_OR_VIDEOS` for jobs/services with no media (line 3533+).
- `image-optional-publish.spec.ts` is the fail-first regression guard for exactly this.
So jobs/services are designed to publish with zero images, and nothing downstream in indexing/search re-imposes an image requirement.

**Job/service is NOT a `type`.** `PublicationType` enum = `bid | classic | offer` only (`publication.entity.ts:25`). Jobs/services are identified by ROOT CATEGORY (84/85), and a photoless job/service still has `type='classic'` (or offer). The FLAG-A phrase "photoless service" therefore means "a listing under the services category with no image", not `type=service`.

---

## 2. LIVE PROBE (staging, read-only)

**Target resolved from AWS (read-only):** ECS `tw-staging-cluster`, API service `tw-staging-svc-back` (task-def `tw-staging-task-back:44`), env `INDEX_PUBLICATIONS=publications-v2`, `OPENSEARCH_NODE=…tw-staging-os…`. Public host `api-staging.tawadoo.ma` (front `staging.tawadoo.ma`). OpenSearch domain `tw-staging-os` (eu-west-1).

**Read-only HTTP probes against the live API (`GET /publications/search`), the exact endpoint a buyer hits:**

| Probe | reported_total | photoless in page |
|---|---|---|
| `?limit=1` (health) | HTTP 200 | — |
| `?limit=100` (broad, no filter) | **39** | **0** |
| `?type=service` | 0 | 0 (note: not a valid `type` value — see 1e) |
| `?cat=84` (services) | 0 | 0 |
| `?cat=85` (jobs) | 0 | 0 |
| `?scat=84` / `?scat=85` | 0 | 0 |
| `?store_type=lead_gen` | 0 | 0 |

**Key live fact:** the ENTIRE staging search index currently holds only **39** published+verified docs, and the `type` distribution is `classic: 21, offer: 18`. **Zero** docs under jobs/services categories; **zero** photoless docs of any kind. So there is currently **no photoless job/service listing searchable on staging** — and apparently none in the index at all to test against.

### 2b. Direct DB / OpenSearch read — BLOCKED (no mutation-free path)
To distinguish "photoless job/service was published-but-not-indexed (broken)" from "no photoless job/service was ever successfully published/approved on staging (empty population)", I need the DB flags of the specific S-CTO-R-RENDER listing and whether its OpenSearch doc exists. Every read path is blocked without a mutation:
- OpenSearch `tw-staging-os` endpoint is public but **IP-restricted** (allowlist: 3.250.16.176, 63.32.202.176, VPC CIDRs) — my IP is not allowed → direct query = 403.
- RDS `tw-staging-instance-1` is `PubliclyAccessible: false` (VPC-private).
- ECS Exec IS enabled on `tw-staging-svc-back` (`enableExecuteCommand: true`), but **both running tasks' SSM agents report `TargetNotConnected`** (they were launched before the flag took effect). The other services (`mcp`, `bo`, `front`) have exec DISABLED.
- No SSM-managed EC2 / bastion exists (`DescribeInstanceInformation` = empty).
- Connecting a fresh exec-capable task requires forcing a new deployment = a **mutation**, which is FORBIDDEN in this read-only session.

`CAPABILITY AVAILABLE BUT FAILED` (ECS Exec, proven `TargetNotConnected`); `CAPABILITY REQUIRES MUTATION — BLOCKED` (new deployment to reconnect agents / open DB path).

---

## 3. VERDICT

**Primary verdict: (d) INCONCLUSIVE on the exact FLAG-A instance — but the underlying fear is refuted at the source level.**

Broken down:
- **The "search index / search query requires an image" hypothesis is REFUTED from source (verdict (a) for that specific mechanism):** no image condition exists in index-eligibility, index-write, legacy search filters, or hybrid search filters. Image count is only a soft ranking boost. A photoless job/service that is published + approved (hence moderated + verified) WILL be indexed and WILL match `/publications/search`. There is no photo-based exclusion anywhere in the search path.
- **The specific live observation cannot be confirmed or refuted (verdict (d)) because the staging index currently contains zero job/service and zero photoless docs, and the DB/OpenSearch state of the exact S-CTO-R-RENDER listing is unreadable without a forbidden mutation.** The observed "0 photoless" is fully consistent with an empty population (no photoless job/service currently published+approved on staging), which is NOT evidence of a search-index defect.

**Most likely explanation (source + live, to be confirmed by the follow-up):** the photoless service either was never fully approved (still `isModerated/isVerified=false`, e.g. sitting in the moderation queue), or was published to a `back` instance whose index write was recovered/pending — NOT that the search index rejects photoless listings by rule. If a real gap exists, it is in the **moderation/approval → index-push lifecycle for image-optional listings**, not in a photo requirement in search.

**If it were broken (it is not, per source), the fix location would be:** `publication-search.service.ts reindexOne()` / `index-eligibility.ts isIndexEligible()` — but both already correctly omit any image requirement, so **no fix is warranted in the search-index layer.**

---

## 4. RAG / RISK

- **BLUE** on the specific FLAG-A instance (inconclusive — needs one DB/OS read to close).
- **Search-index-requires-image hypothesis: REFUTED (source-certain).** No RED here.
- Reversibility: R0 (read-only session; nothing changed, nothing to roll back).

---

## 5. RECOMMENDED FOLLOW-UP

**Session:** `S-CTO-18-flagA-close` — a **5-minute read-only DB/OS confirmation** with a mutation-free path to staging data.
**One-line reason:** the only missing fact is the exact S-CTO-R-RENDER listing's flags (`status/isModerated/isVerified/deletedAt`) + whether its `publications-v2` doc exists; source already proves search does not gate on images, so this closes (d)→(a) or, if the listing is approved-but-unindexed, points the fix at the approval→index lifecycle (NOT at an image rule).
**How to unblock the read (pick one, founder to approve the mutation-bearing option if chosen):** (i) allowlist the runner IP on `tw-staging-os` temporarily; or (ii) force one staging `back` redeploy so ECS Exec agents reconnect, then run read-only `SELECT` + OpenSearch `GET`. Option (ii) is a deployment (mutation) and needs founder approval; option (i) is a reversible security-group/access-policy change (also a mutation, smaller blast radius).

**Do NOT** build a search-index "photoless fix" — source proves none is needed.

---

## APPENDIX — read-only actions taken (provenance)
- `git rev-parse` on tawadoo_api_js → branch `Ramzi_V2`, HEAD `bd5cf2a…`, clean tree.
- Source reads: `index-eligibility.ts`, `publication-search.service.ts`, `publication-index-body.ts`, `publication.service.ts` (buildEsFilters + searchPublications + isImageOptionalCategory + approve/reject), `publication.controller.ts` (search filter mapping + publish image-optional), `hybrid-search.service.ts`, `moderation-state-machine.ts`, entities (`publication`, `category`, `publication-image`), `image-optional-publish.spec.ts`.
- AWS read-only: opensearch ListDomainNames/DescribeDomain, ecs List/DescribeServices/DescribeTaskDefinition/ListTasks, elbv2 Describe(LoadBalancers/Listeners/Rules), ssm DescribeInstanceInformation, rds DescribeDBInstances. No secret VALUES recorded (only key NAMES).
- Live HTTP: `GET https://api-staging.tawadoo.ma/publications/search` with the filter matrix in §2 (all read-only GET).
- ECS Exec attempted on both `back` tasks → `TargetNotConnected` (agents not connected). No mutation performed.
```
