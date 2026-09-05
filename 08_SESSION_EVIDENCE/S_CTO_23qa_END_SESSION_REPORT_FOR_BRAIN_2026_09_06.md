# S-CTO-23qa — END-SESSION REPORT FOR THE BRAIN

**Session:** S-CTO-23qa — Moderation→Visibility Runtime QA (read-only + one human verify)
**Date:** 2026-09-05/06 · **Env:** STAGING only (prod untouched) · **Author:** Kiro (execution)
**Proposed status:** FINISHED — COMPLETE (mission) with open items below. **Independent Brain QA required per §18/§29.3** — re-verify from source + live, not from this document.
**Primary evidence file:** `S_CTO_23qa_MODERATION_VISIBILITY_EVIDENCE_2026_09_06.md`

> Purpose of THIS file: leave nothing buried. Every open item, drift, anomaly, out-of-scope observation, and "built-in-back-but-not-verified-in-front/DB/BO/AWS" gap is listed for the Brain to triage into future units. Discovery ≠ authorization — everything below is a CANDIDATE, not authorized work.

---

## A. ONE-PARAGRAPH TRUTH

The manual human-in-the-loop moderation→visibility chain **works live on staging**: a published-but-unverified listing is invisible in public search; after the founder clicked **Verify** in the BO (real 2FA = the human gate), the listing flipped `isVerified=true` + `isModerated=true`, entered the keyword index, and became searchable. During the session I RAISED a "vector-search leak" finding and then **WITHDREW it** after proper reproduction (false alarm from one ambiguous snapshot). No code changed; no repo HEAD moved. The only writes were synthetic staging test data + the one founder BO Verify.

---

## B. WHAT WAS PROVEN vs NOT PROVEN (by layer — nothing buried)

### API / backend (tawadoo_api_js @ Ramzi_V2)
- **PROVEN (live):** create → publish (`x-api-version: v2`) → pending; `isVerified=false` keeps a listing out of public search; `POST /publications/verify/:id` path (via BO) sets both flags + indexes. Endpoints exercised live: `/auth/register`, `/auth/login`, `/auth/protected`, `/publications`, `/publications/publish/:id`, `/publications/:id`, `/publications/search`, `/entities/slug/:slug`, `/categories`.
- **PROVEN (source only, NOT run live):** `POST /publications/reject/:id` and `/unverify/:id` (rejection/unverify path). Detail-page 404 gate for unverified (source `publication.service.ts` ~756/~827) was NOT hit live — I proved invisibility via search + index, not via a slug 404 (I never captured the public slug because the owner readback shape has no slug field; see D3).

### Web / frontend (tawadoo_web_js)
- **NOT VERIFIED THIS SESSION (gap):** I did NOT load the public web detail page or the web search UI in a browser for the pending/verified listing. §47 (browser-verify every UI change) was not exercised — this was an API/OpenSearch-level proof, not a Chromium/WebKit proof. The web-side rendering of "pending → hidden / verified → visible" is UNVERIFIED. Candidate for a follow-up browser QA.

### BO / admin (admin_bo_tawadoo @ Ramzi_V2, tw-staging-svc-bo)
- **PROVEN (live, by founder):** the AdminJS `verifyPublication` action works — founder clicked Verify, flags flipped, keyword index updated. Screenshot on record showed the review page ("✗ Not Verified", PUBLISHED, "No images uploaded", "Verify Publication" button).
- **NOT VERIFIED:** the `bo_moderation_audit_log` row insert (source `publication.resource.ts:203`) — I could not read the staging DB (VPC-private) so I did NOT confirm the audit row was actually written. Candidate to verify.
- **NOT VERIFIED:** the seller **WhatsApp + email** side-effects that `approvePublication`/`verifyPublication` fire on approval — not observed (synthetic seller had a company-domain email, no real phone). Whether these dispatched/failed gracefully (§50) is UNVERIFIED.

### DB (tw-staging-cluster, Aurora PostgreSQL)
- **NOT ACCESSIBLE this session:** VPC-private, tcp 5432 unreachable from laptop. All flag/state reads came from API + OpenSearch, never direct DB. Any claim requiring DB ground truth (audit log, exact column values) is UNVERIFIED at the DB layer.

### AWS / infra
- **OBSERVED:** OpenSearch domain `search-tw-staging-os-…` reachable via SigV4 (kiro-ai); indices `publications-v2` (keyword) and `publication-embeddings-v2` (kNN). Back task def `tw-staging-task-back:44`, image `ghcr.io/embendev24/tawadoo-api-js:staging-v2` (mutable tag — consistent with §42; not a defect).
- **BLOCKERS (infra):** ECS exec disabled on BO task (`enableExecuteCommand:false`); ECS exec on back task shows agent RUNNING but channel `TargetNotConnected` (failed twice). Not fixed (would require redeploy; out of scope for read-only QA).

---

## C. DRIFT vs the S-CTO-20 source map (flag for Brain)

1. **BO route wrapping:** BO Verify → API `POST /publications/verify/:id` → `verifyPublication` (service:3295) which **wraps** `approvePublication` (service:3705). The prompt/map implied the route hits `approvePublication` directly. Minor, documented.
2. **Gate-layer clarification:** search-visibility is gated by **`isVerified`**; keyword-index *presence* is gated by **`isModerated`** (via `isIndexEligible`). Distinct layers; the map slightly conflated them.

---

## D. OPEN ITEMS / ANOMALIES / ERRORS (nothing buried — including small + out-of-scope)

**D1 — [MEDIUM] v2 publish did not set `isModerated=true` at publish time (anomaly).**
Source (`publication.controller.ts` ~626 in the `enableModeration` branch) calls `updatePublicationIsModerated(id, true)` at publish. But live, my first listing read back `isModerated=false` immediately after a v2 publish, then later showed `true`. Either the v2/enableModeration branch did not execute (despite `x-api-version: v2` sent + `ENABLE_CONTENT_MODERATION=true`), or there's a timing/ordering issue. **Not a proven bug; worth a source+runtime trace.** Impacts index-eligibility timing.

**D2 — [LOW/DEFENSE-IN-DEPTH] vector (kNN) leg has no `isVerified`/`status` filter.**
`hybrid-search.service.ts` `executeVectorSearch` (~line 450) queries `publication-embeddings-v2` with a bare `knn` query — no verification/status filter (unlike the keyword leg at :313). Unverified listings DO have embeddings written and ARE present in the embeddings index (confirmed live: `03fceb84` found in embeddings, not in keyword, not returned by search). They are currently kept out of results only by the downstream `paginateResults` min-combined-score gate (~686–698). **This is NOT a proven leak** (reproduction returned 0), but the guarantee rests on a score threshold rather than an explicit filter. Candidate hardening: add `{term:{isVerified:true}}` + `{term:{status:'published'}}` to the kNN query as belt-and-suspenders.

**D3 — [LOW] Owner readback (`GET /publications/:id`) exposes no slug field.**
The response has no `slug`/`slug_fr` (public search hits do have `slug_fr/en/ar`). Made it awkward to build the public detail URL from the owner view. Not a bug, but a note for anyone scripting detail-page checks — get the slug from the search hit or translations, not the owner readback.

**D4 — [INFO] Duplicate detection matches across verification state and blocks re-publish.**
Same-seller near-identical listings were rejected at publish with `400 "A very similar listing already exists."` — the duplicate detector (embeddings/kNN similarity) matched my already-verified test listings. Expected behavior, but worth noting it keys on embeddings regardless of verified state; relevant to D2's subsystem.

**D5 — [INFO] Public `/entities/slug/:slug` does not expose `owner`.**
Had to resolve the caller's entity via authenticated `POST /auth/protected`. Fine, just documented so future sessions don't re-discover it.

**D6 — [BLOCKER, infra] Staging DB unreachable + ECS exec broken (see B/DB + AWS).**
This limits any future session needing DB ground truth or in-container commands on staging. If DB verification becomes necessary, the Brain should authorize either a bastion/SSM path or re-enabling ECS exec (redeploy) as a bounded unit.

**D7 — [PROCESS] I over-claimed the "leak" before reproducing.**
I used "airtight" language on a single ambiguous snapshot; the founder correctly challenged it and it was withdrawn. Process note per §31/§52.2: raise runtime findings only after reproduction. No lasting artifact, but logged honestly.

**D8 — [HYGIENE, R0] Synthetic staging data left behind (must be cleaned).** See section E.

---

## E. SYNTHETIC DATA CREATED (cleanup owed — R0) — current live state

Sellers/entities (staging):
- `2a9e24a4-…` (slug `s23qa-s`) — no listings — DELETE
- `89ba04d0-…` (slug `s23qa-s-1`) — no listings — DELETE
- `adc41841-…` (slug `s23qa-s-2`, seller `s23qa.synthetic.1788632497325@tawadoo.ma`) — listings below — DELETE with listings
- leak seller `s23qa.leak.<ts>@tawadoo.ma` (entity id captured in evidence file) — listing `03fceb84` — DELETE with listing

Listings (verified live via OpenSearch just before writing this):
- `bfa8710e-8233-4229-aa8a-3de39d009432` — **VERIFIED/published** — in keyword index + embeddings. (This is a synthetic listing now live in staging search — should be removed/soft-deleted.)
- `03fceb84-e878-479b-97b5-4ba9514bf05a` — published, **unverified** — in embeddings only, not searchable.
- `cf108cfc-…`, `22f2a052-…` — **draft** — in neither index.

Temp scripts (all deleted this session): `/tmp/s23qa_harness.mjs`, `/tmp/s23qa_leak_repro.mjs`, `/tmp/admin_bo_tawadoo/e2e/s23qa-login-probe.spec.ts`.

No secrets printed to logs/evidence/command text.

---

## F. NON-GOALS respected
No code/deploy/config change · no prod mutation · no verifying a real user's listing · no removing/automating the human gate · no dependency on S-CTO-22api · no novel paid tool/permission provisioned (§23) — only existing kiro-ai read access + authorized synthetic API writes.

---

## G. RECOMMENDED NEXT UNITS (candidates for founder authorization — NOT authorized)

- **S-CTO-24qa (R0, small):** delete/soft-delete the synthetic staging data in section E, verify removal from both indices. Highest priority — a synthetic verified listing is currently live in staging search.
- **R1 (small, hardening):** add explicit `isVerified`+`status` filters to `executeVectorSearch` (D2). Prove with one bounded slice + fail-first.
- **R2 (trace):** confirm whether v2 publish reliably sets `isModerated=true` at publish (D1) — source + runtime.
- **R3 (web QA):** browser-verify (Chromium+WebKit) the public detail page + search UI for pending vs verified (fills the §47 gap in section B/web).
- **R4 (BO/DB):** confirm `bo_moderation_audit_log` row on verify + observe the approval WhatsApp/email side-effects (needs a DB/observability path — ties to D6).

Prod remains out of scope until the founder opens go-live.
