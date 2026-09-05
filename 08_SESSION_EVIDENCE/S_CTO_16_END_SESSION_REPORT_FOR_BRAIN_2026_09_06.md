# S-CTO-16 — END-OF-SESSION REPORT FOR THE BRAIN

**Session:** S-CTO-16 · SEARCH-INDEX FLAG-A DIAGNOSIS (READ-ONLY)
**Date:** 2026-09-06 · **Author:** Kiro (execution session) · **Reader:** Brain (queue owner)
**Repo/branch:** `tawadoo_api_js` @ `Ramzi_V2`, HEAD `bd5cf2a6ddb78b2b5aef5d53847bab2c8d81a753` (clean, unchanged)
**Companion evidence file:** `S_CTO_16_SEARCH_INDEX_FLAG_A_EVIDENCE_2026_09_06.md`
**Proposed status:** FINISHED — COMPLETE (verdict rendered) with ONE gate BLOCKED (direct DB/OS read) — needs independent QA acceptance.
**Rule note:** This session's status is a PROPOSAL until independent Brain QA verifies it from source + live (§18, §29.3).

---

## 0. ONE-PARAGRAPH SUMMARY

FLAG-A ("photoless service didn't appear in search") is **REFUTED at the source level**: no layer of the search path (index-eligibility, index-write, legacy search filters, hybrid search filters) requires an image; image count is only a soft ranking boost. BUT the specific live sighting is **INCONCLUSIVE (verdict d)** because staging's search index currently holds only 39 docs (all `classic`/`offer`, zero job/service, zero photoless), and I could not read the exact listing's DB/OpenSearch state without a forbidden mutation. Nothing was built or changed. The user then raised a **business decision** ("make photo mandatory") — I stopped and surfaced it as APPROVE/CHANGE/REJECT; the user said stop. No decision taken, nothing queued to build.

---

## 1. WHAT THIS SESSION WAS SUPPOSED TO DO vs DID

| Goal | Result |
|---|---|
| Trace search index/eligibility rule from source | ✅ DONE — CONFIRMED, cited file:line |
| Live-probe a real photoless job/service in `/publications/search` | ⚠️ PARTIAL — probed live API (CONFIRMED index contents), but no photoless job/service exists in the index to test; specific listing unreadable |
| Verdict a/b/c/d + fix location if broken | ✅ DONE — verdict (a-for-mechanism / d-for-instance), no fix warranted in search layer |
| Do NOT build/fix/commit/deploy/mutate | ✅ HELD — read-only throughout |

---

## 2. WHAT ALREADY EXISTS PER LAYER — and what I VERIFIED vs DID NOT

This session built NOTHING. This table records the **current state of the image-optional / search-index feature** across layers, and my verification level for each. Verification ladder: SOURCE-CONFIRMED / LIVE-CONFIRMED / NOT-VERIFIED / BLOCKED.

### BACKEND (`tawadoo_api_js` @ Ramzi_V2) — SOURCE-CONFIRMED, partially LIVE-CONFIRMED
- **Image-optional publish for jobs/services EXISTS in source.** `isImageOptionalCategory()` (publication.service.ts:3470) → true only for root category `jobs`(85)/`services`(84). `autoFixMainImage(pubId, imageOptional=true)` skips the no-media throw (line ~3533). Guarded by `image-optional-publish.spec.ts`. **SOURCE-CONFIRMED.** NOT LIVE-VERIFIED (could not publish/inspect a real photoless listing this session).
- **Text-only embedding path for imageless listings EXISTS** (publication.controller.ts:594 `imageless` branch; publication.service.ts:3587+ zero image-vector). **SOURCE-CONFIRMED.** NOT runtime-verified.
- **Search index eligibility does NOT require an image** (index-eligibility.ts:29). **SOURCE-CONFIRMED.**
- **Search query (both engines) does NOT filter on images** (publication.service.ts:1674-1675 buildEsFilters; hybrid-search.service.ts:313 hard filters + 628-638 boost-only). **SOURCE-CONFIRMED.**
- **Live API `/publications/search` responds 200** and returns 39 docs total. **LIVE-CONFIRMED.**

### FRONTEND (`tawadoo_web_js`) — NOT VERIFIED (out of scope; owned by S-CTO-17web browser session)
- **NOT VERIFIED THIS SESSION:** whether the web create-listing form actually lets a seller submit a job/service with zero images (client-side validation may still force an image even though the backend allows it). Open editor files show `product-form-v2.tsx` and `useProductFormState.ts` — these are the likely place a client-side image requirement would live. **FLAG FOR BRAIN:** confirm the web form's image requirement matches the backend's image-optional rule for categories 84/85. If the front forces an image, no photoless job/service can ever be created → which would fully explain "0 photoless in index" without any search bug.
- **NOT VERIFIED:** whether search results render photoless cards correctly (fallback thumbnail). Owned by S-CTO-17web.

### DATABASE (staging RDS `tw-staging-instance-1`, private) — BLOCKED
- **NOT READ.** Could not confirm: does a photoless job/service row exist in `ta_publication`? What are its `status/isModerated/isVerified/deletedAt`? Is there any moderated-but-unindexed listing (reconciliation backlog)? **BLOCKED** — RDS is `PubliclyAccessible:false`, no bastion, ECS Exec agents not connected. Reading requires a mutation (redeploy or SG/policy change) → founder approval.

### OPENSEARCH (staging domain `tw-staging-os`, eu-west-1) — LIVE-CONFIRMED via API only; direct read BLOCKED
- **Index `publications-v2` holds 39 published+verified docs, type distribution {classic:21, offer:18}, zero job/service, zero photoless** — LIVE-CONFIRMED via public API aggregation.
- **Direct OpenSearch query BLOCKED** — domain access policy is IP-restricted to 3.250.16.176 / 63.32.202.176 / VPC CIDRs; my IP is not allowlisted (would 403). Could not run raw `_search`/`_count` or verify a specific doc's presence/fields directly.

### BO (admin_bo_tawadoo) — NOT VERIFIED (out of scope)
- **NOT VERIFIED:** the moderation/approval UI path that sets `isModerated/isVerified=true` and triggers `pushToIndex`. If a photoless job/service is stuck unapproved in the BO queue, that alone explains "0 in search." **FLAG FOR BRAIN.**

### AWS — READ-ONLY INVENTORY DONE
- Staging inventory captured (ECS cluster/services/task-def:44, ALB hosts, OpenSearch domain, RDS instances). No mutation. See §5 for the infra ISSUE found.

---

## 3. OPEN / INCOMPLETE / UNVERIFIED — nothing buried (Brain: queue these)

**O-1 (PRIMARY, closes FLAG-A):** The exact S-CTO-R-RENDER listing's DB + OpenSearch state was never read. Verdict is (d) inconclusive until this one fact is obtained. → Queue `S-CTO-18-flagA-close` (read-only DB/OS confirmation). Needs a mutation-free read path (founder approval for either IP allowlist or a staging redeploy to reconnect ECS Exec).

**O-2 (likely ROOT of the sighting, currently unproven):** Is the FRONT-END create form blocking photoless job/service submission client-side, even though the backend allows it? If yes, no photoless job/service ever reaches the DB/index → "0 photoless" is a front-end gap, not a search bug. → Queue a `tawadoo_web_js` check of `product-form-v2.tsx` / `useProductFormState.ts` image validation for categories 84/85. **This is the cheapest path to explaining FLAG-A and may make O-1 unnecessary.**

**O-3 (moderation lifecycle):** Is there a moderated-but-unindexed backlog, or a photoless job/service stuck unapproved in the BO queue? IndexReconciliationService exists to heal unindexed listings — is it running on staging? → Queue a read-only check of reconciliation cron health + BO moderation queue for job/service listings.

**O-4 (business decision, PARKED by founder):** "Make photo mandatory." Founder raised it, then said stop. NOT decided, NOT built. It reverses founder decision D1=YES (jobs/services image-optional). If revisited, needs a decision (all / physical-only / specific categories) then its own build session (`publication.service.ts isImageOptionalCategory` + test + web form + FR/AR/EN copy per §30). **Do not action without an explicit founder APPROVE.**

**O-5 (data reality):** Staging has only 39 searchable listings and ZERO job/service listings at all. This is itself notable — either the category isn't being used on staging, or job/service publishing is failing upstream. Worth a Brain note: staging is a thin test population, which weakens any "it's not in search" conclusion drawn from staging alone.

---

## 4. ERRORS / ISSUES ENCOUNTERED (incl. outside scope — nothing buried)

**I-1 — ECS Exec unusable on staging `back` (INFRA, outside scope, real).** `tw-staging-svc-back` has `enableExecuteCommand: true`, but BOTH running tasks return `TargetNotConnected` (SSM agent not connected — tasks predate the flag). The other services (`mcp`, `bo`, `front`) have exec DISABLED. Net effect: **there is currently no mutation-free way to run a command inside the staging VPC.** This blocks any future read-only DB/OpenSearch diagnosis too. → Brain should decide whether to (a) enable exec + force one redeploy so agents connect (small mutation, unlocks future read-only diagnosis), or (b) stand up a tiny SSM-managed bastion, or (c) IP-allowlist a runner on the OS domain. Recommend (a) as a one-time staging enablement.

**I-2 — Local `tawadoo_api_js/.env` has no OpenSearch node / DB creds** (only non-secret tuning). Not an error, but means no local-path staging read is possible; everything must go through AWS. Noted so future sessions don't waste time expecting local connectivity.

**I-3 — `FEED_PUBLIC_BASE_URL` on staging `back` points at a `tw-prod-media-storage-prod` S3 bucket** (env value observed: `https://tw-prod-media-storage-prod.s3.eu-west-1.amazonaws.com`). **Outside my scope, flagging anyway:** staging serving media/feeds from a PROD-named bucket may be intentional (shared media) or a staging/prod bleed. Brain should confirm this is deliberate. Not investigated further.

**I-4 — No error, but a correctness trap for future sessions:** "job/service" is NOT the `type` field. `PublicationType` = `bid|classic|offer` only. Jobs/services are ROOT CATEGORY 84/85. Any future prompt that says `type=service` will silently return 0 and mislead. Recorded so the Brain bakes the right identifier into the next prompt.

---

## 5. VERDICT & RAG (carry-forward)

- **FLAG-A search-index mechanism:** REFUTED (SOURCE-CONFIRMED) — search does not require a photo. **No search-layer fix warranted.**
- **FLAG-A specific sighting:** (d) INCONCLUSIVE — BLUE — needs O-1 or (cheaper) O-2.
- **Reversibility:** R0 (nothing changed).
- **Most probable real cause (hypothesis, unproven):** front-end form blocks photoless job/service (O-2), OR the listing was never approved (O-3) — NOT a search bug.

---

## 6. RE-ENTRY FOR NEXT SESSION (so no history is needed)
1. Read `S_CTO_16_SEARCH_INDEX_FLAG_A_EVIDENCE_2026_09_06.md` (full source+live trace) and this report.
2. Cheapest close = O-2 (web form image validation for cat 84/85) — pure source read in `tawadoo_web_js`, no infra needed.
3. Full close = O-1, which needs the I-1 infra unblock (founder approval).
4. Do NOT build a search "photoless fix" — proven unnecessary.
5. Do NOT action O-4 (mandatory photo) without explicit founder APPROVE; it reverses D1.

---

## 7. STATUS LINE FOR THE QUEUE
`S-CTO-16 — FINISHED — COMPLETE (verdict d; search-index-requires-image REFUTED from source). Open: O-1..O-5. Infra blocker I-1 (ECS Exec not connected). No code/DB/index/prod change. Awaiting independent QA acceptance.`
