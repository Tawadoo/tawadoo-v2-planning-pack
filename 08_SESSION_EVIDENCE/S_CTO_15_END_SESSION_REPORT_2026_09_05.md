# S-CTO-15 — END-SESSION REPORT (for the Brain)

**Session:** S-CTO-15 · EVENT ALLOWLIST FIX + TRIAGE (build-or-ask)
**Date:** 2026-09-06 · **Mode:** MODE B build unit (founder-authorized), `tawadoo_api_js` writable
**Skills:** tawadoo-source-truth + tawadoo-data-sovereignty + tawadoo-runtime-provider-qa (all applied)
**Companion evidence:** `S_CTO_15_ALLOWLIST_FIX_EVIDENCE_2026_09_05.md` (full triage table, live proof, ladders)
**Commit:** `bd5cf2a` on `Ramzi_V2` (parent `673eac0`) · **CI:** `33974665602` GREEN · **Deploy:** staging rolloutState=COMPLETED
**Proposed status:** FINISHED — COMPLETE. Independent QA to re-verify from source per §18 before durable acceptance.

---

## 0. ONE-PARAGRAPH SUMMARY FOR THE FOUNDER

I looked at every analytics event name that was firing or landing but was never officially registered, and I sorted each into three buckets: (A) it already fires in the app → I registered it so it counts as "real"; (B) it does not fire anywhere yet → I did NOT build it, it's on a short list for you to approve or reject; (C) junk. I registered 28 real events (mostly Smart View), applied your two decisions (context-switch and the price-suggestion name), fixed a documentation lie, and — most importantly — built the missing safety net so that any future misspelled/undeclared event is now COUNTED and alarmed instead of slipping into the lake silently. I shipped it to the staging test environment and proved live that a real event now counts as canonical and a fake one gets flagged without being dropped. Nothing broke. Below I list every loose end, small or large, including things outside my scope, so nothing is buried.

---

## 1. WHAT WAS BUILT (and shipped live) — the "back"

All in `tawadoo_api_js`, module `analytics-ingestion`, commit `bd5cf2a`, deployed + live-verified on staging:

1. **28 category-A events declared canonical** in `constants/allowed-events.ts` (21 `smart_view_*`, 4 draft-recovery, `store_delivery_configured`, `context_switch`, `price_suggestion_shown`). Each has a live web emitter cited to file:line in the evidence file.
2. **Unknown-event surfacing** in `analytics-observability.service.ts`: new `ingestion_unknown` counter, `recordIngestionUnknown()`, emission in the 60s health-metric JSON (`ingestion.unknown`), and `[ALARM:INGESTION_UNKNOWN]` warn. Wired from `analytics-ingestion.service.ts`.
3. **Stale doc fixed** (header no longer falsely claims "rejected with HTTP 400").
4. **4 fail-first tests** added; red on clean HEAD, green after.

**Non-breaking guarantee proven:** live POST returned HTTP 200, `ingested:2`; unknown persisted `_is_canonical=false` and surfaced; A event landed canonical; no ingestion failure, delivery `success:2`, `dlq:0`.

---

## 2. WHAT IS "BUILT IN THE BACK" BUT NOT BUILT / NOT VERIFIED ELSEWHERE

This is the cross-surface truth. Nothing buried.

### 2A. Declared canonical in the API allowlist, but the EMITTER still lands under a different/undeclared path (front not aligned)
The allowlist now *recognizes* the 28 A events, but I did **not** touch any web emitter (web was read-only). So:
- **Amplitude tracking-plan registration NOT done.** Declaring a name canonical in our API allowlist is **not** the same as registering it on the Amplitude project (795817) tracking plan. The 28 newly-declared events (and the pre-existing draft events per S-CTO-11 O2) are **still unregistered on Amplitude**. → they land in our sovereign DB canonical, but Amplitude may still treat them as unplanned. **Owner: a tracking-plan registration task (S-CTO-18 or a dedicated Amplitude-admin step).**
- **DB-level canonical proof for the 28 is partial.** I proved live that ONE A event (`smart_view_opened`) lands canonical (absent from the unknown warn, `unknown` count stayed 1 for 2 successes). I did **not** individually fire all 28 through staging. The allowlist membership guarantees canonical tagging for all 28 (same code path), but only 1 was runtime-exercised. **Low risk (identical path), disclosed.**

### 2B. RENAME family — declared targets exist, but emitters still emit the OLD name (front work outstanding = S-CTO-16)
These land in the lake under a wrong spelling; the canonical twin exists but the **web emitter was not changed** (out of my writable scope):
- `feed_slide_change` (live emitter `ImmersiveFeed.tsx:374`) → should emit `feed_slide_changed`.
- `app_banner_dismissed` (live emitter `OpenInAppBanner.tsx:231`) → target `app_install_banner_dismissed` **does not exist in the allowlist yet** (I did NOT add it — it would be a paper name with no emitter until the web rename lands; adding it now violates the "A = has emitter" rule). S-CTO-16 must add the canonical AND rename the emitter together.
- `ai_generated` → `ai_generation_completed` (target already declared): only the emitter rename remains (the only `ai_generated` in the API is a DB column `posts.entity.ts:91`, not an event).
- `session_start`/`session_end`/`feed_enter`/`feed_exit`/`login`: I found **no literal string emitter** in current web `src` for these. They may already be resolved, or emitted via a variable/constant I could not match with a literal grep. **UNVERIFIED whether they still land** — needs a web-side check (or a lake query) in S-CTO-16. Not assumed either way.
- `promo_dismiss`, `video_play`: source-verified **already removed/renamed** (a source-guard test forbids `promo_dismiss`; `productDetails.tsx:1729` comments the legacy `video_play` was killed for `listing_video_played`). No action. If either still shows landings in the lake, they are historical rows, not live emission.

### 2C. Category-B events — NOT built (correctly), awaiting founder
`store_search`, `store_categories_updated`, `smart_view_query`, `smart_view_intent_classified`, `smart_view_guidance_response` have **no emitter anywhere** (grep of web + api). The prior reports (S-CTO-13a/14) proposed adding `store_search`/`store_categories_updated` and implied the smart_view ones — **that was wrong**; source shows no emitter. They are on the founder B-list (evidence §3). Recommendation: build B1/B2 later with their surfaces; reject B3–B5 as redundant.

### 2D. DB — verified for the surfacing path, NOT for full parity
- **Verified live (from CloudWatch DB-query logs of the running task):** the unknown probe's `INSERT INTO ta_analytics_event` carried `properties = {"probe":"scto15","_is_canonical":false}` and a matching `ta_analytics_delivery` outbox row (`status:pending`). So the persist-non-canonical-and-queue path is DB-confirmed.
- **NOT verified:** I did **not** run a direct `psql` query against staging (DB host lives only in ECS secrets; connecting would risk secret handling, and the log-based proof was sufficient and safer). So DB↔Amplitude↔S3-lake parity for the 28 newly-declared events is **UNVERIFIED** at the row level. This is the same open item S-CTO-14 ISSUE-5 / S-CTO-13a O12 — still open. **Owner: S-CTO-18 DB/lake read session.**
- **Observed, not investigated:** the health metric shows `partitions.default_count: 8` (8 events in the DEFAULT partition). The observability code has an `[ALARM:DEFAULT_PARTITION]` for exactly this — the DEFAULT partition is supposed to stay empty. **8 rows is small but non-zero.** Not caused by my change (pre-existing). → **FLAG for Brain: investigate why 8 events fell into the DEFAULT partition (partition-routing gap or events with out-of-range timestamps).**

### 2E. BO (`admin_bo_tawadoo`) — untouched, gap carried forward
Not in scope; not inspected this session beyond confirming it's read-only. S-CTO-14 §2C already flagged BO emits essentially no sovereign analytics (only `ask_ramzi_interaction`). **Still open.** If admin actions should feed the lake (Commandment 2), that's a BO session. **Owner: S-CTO-18.**

### 2F. AWS — read-only inspection only
- ECS describe (service/task/deployment), CloudWatch FilterLogEvents — read-only, no mutation.
- The only AWS mutation was the **CI-driven** `ecs update-service --force-new-deployment` (part of the normal pipeline), which succeeded (services-stable).
- **GHCR image manifest→digest resolution NOT done directly** — the package is private under `embendev24`; resolving the tag digest via the registry API needs an auth token, which would risk secret exposure in a request. I relied on the deploy chain + live behavioral proof instead. The running per-image config digest is `sha256:02947ef8…` (both tasks identical). **Belt-and-suspenders digest==tag mapping is therefore UNVERIFIED at the registry level** — low risk given the live behavioral proof that the running code is `bd5cf2a` (the new `unknown` metric field only exists in my commit).

### 2G. MCP — untouched (correct), gap carried forward
No MCP name or contract touched. The MCP-is-a-separate-island gap (business events persist to local SQLite/`/tmp`, never reach `/api/analytics/events`; the 4 canonical `mcp_*` events are never emitted; `search_refined` name collision) is **still open**, founder-gated, → **S-CTO-17**.

---

## 3. ISSUES / ERRORS / OPEN ITEMS — FOR THE BRAIN TO QUEUE (including small + out-of-scope)

| # | Item | State | Severity | Owner / next |
|---|---|---|---|---|
| **I1** | **DEFAULT partition has 8 events** (`partitions.default_count:8` in the live health metric; there's a dormant `[ALARM:DEFAULT_PARTITION]` for it) | Observed live, NOT investigated | MED (data-routing hygiene) | Investigate why 8 rows fell into DEFAULT (bad timestamps? partition horizon gap?). Not mine. → S-CTO-18 or a partition-health session |
| **I2** | **6 pre-existing failing test suites** on `Ramzi_V2` clean HEAD: `entity-boot`, `hybrid-search-sort`, `hybrid-search.service`, `whatsapp-consent-gate.property`, `whatsapp-bridge.service`, `auth/identifier-change` (last is a flaky 5s timeout, passes isolated) | CONFIRMED pre-existing (stashed my files, they fail identically) | MED | These are env-dependent (OpenSearch/DB) OR real test debt. **CI does not catch them** if CI only runs a subset — verify CI test scope. → a test-health session |
| **I3** | **`ai_generated` name is ambiguous / overloaded** — it's both a DB boolean column (`posts.entity.ts:91`) AND a landing event name (per S-CTO-14). N-DEC2 renames the event to `ai_generation_completed`, but the DB column keeps the name | Noted | LOW | Harmless (different namespaces) but note it so nobody "fixes" the column thinking it's the event. → S-CTO-16 context |
| **I4** | **`smart_view_no_results_recovery` declared-not-wired** — in allowlist, no emitter found (grep) | CONFIRMED | LOW | Wire it in Smart View work OR prune. Left in place (harmless paper). → S-CTO-16 |
| **I5** | **`test_event` prune blocked by a live test fixture** — `analytics-ingestion.service.spec.ts:235` uses `trackServerEvent('test_event', …)` | CONFIRMED | LOW | Migrate the fixture to a real event name, THEN prune `test_event`. Not done (would break the suite). → housekeeping (Category-D, founder-authorized) |
| **I6** | **Amplitude tracking-plan registration NOT done** for the 28 newly-declared events (nor the pre-existing draft events, S-CTO-11 O2) | Open | MED (they land sovereign-canonical but Amplitude may treat as unplanned) | Register on Amplitude project 795817. → S-CTO-18 or a tracking-plan-admin step |
| **I7** | **DB↔Amplitude↔S3-lake parity UNVERIFIED** for renamed/added events (inherited S-CTO-14 ISSUE-5 / S-CTO-13a O12) | Still open | MED | A DB/lake read session (needs authorized psql or RDS Data API). → S-CTO-18 |
| **I8** | **BO sovereign-emission gap** — BO emits only `ask_ramzi_interaction` (inherited S-CTO-14 §2C) | Still open, not re-verified | MED (if admin actions should feed the lake) | Founder decision + BO session. → S-CTO-18 |
| **I9** | **MCP → lake bridge does not exist** + `search_refined` collision + 4 unused canonical `mcp_*` (inherited O11/O4) | Still open, founder-gated (external contract) | MED | → S-CTO-17 |
| **I10** | **`session_start`/`session_end`/`feed_enter`/`feed_exit`/`login` emitter status UNVERIFIED** — no literal grep hit in web src; may be resolved or variable-emitted; still land per S-CTO-14 | UNKNOWN (not assumed) | LOW-MED | Confirm via web read + lake query whether these still emit. → S-CTO-16 |
| **I11** | **`ingestion_unknown` counter counts at RECEIPT, not post-dedup** — an unknown event that is later deduped still increments the counter | By design (drift-signal semantics), disclosed | LOW | Intentional: we want to know undeclared names *arrived*. Note so QA doesn't read it as a bug. |
| **I12** | **GHCR tag→digest mapping not resolved at registry level** (private pkg auth) | Disclosed limitation | LOW | If strict provenance is required for cutover, resolve with an authorized token in a secret-safe way. Live behavioral proof already confirms running code = bd5cf2a. |
| **I13** | **Prior reports (S-CTO-13a/14) contained inaccuracies** — proposed ADD for `store_search`/`store_categories_updated` (no emitter) and implied smart_view names with no emitter | CORRECTED this session from source | LOW (process note) | Prompt-authoring/QA note: those were report hypotheses, not source truth. Brain should treat the S-CTO-15 triage as the corrected source of truth over the earlier proposals. |
| **I14** | **`app_install_banner_dismissed` canonical intentionally NOT added** — would be a paper name (no emitter) until the web rename lands | Deliberate | LOW | S-CTO-16 must add canonical + rename emitter in the same unit. |

---

## 4. THINGS LEFT OPEN / SMALL LOOSE ENDS (belt-and-suspenders)

1. Only **1 of 28** A events was runtime-exercised live (`smart_view_opened`); the other 27 rely on identical-code-path reasoning. Low risk, disclosed.
2. Rollback was **defined but not exercised** (additive, non-breaking change; exercising a rollback on shared staging would disrupt the parallel S-CTO-R-RENDER browser session for no safety gain). Rollback is executable: `git revert bd5cf2a` → CI redeploys prior image `sha-673eac0`, task-def rev 43.
3. The health-metric emission I captured showed a **transient split** at 15:35:00 (one task on new code with the `unknown` field, one still on old code without it) — normal mid-rollout; ECS later reported rolloutState=COMPLETED with both tasks on the same digest. If a future session sees a metric without the `unknown` key, it's an old task pre-convergence, not a regression.
4. I did not re-count the total allowlist size (was "492" per S-CTO-14; I added 28, so ~520 — not independently re-counted line-by-line). If an exact count matters for a report, re-count. (S-CTO-13a O13, still applies.)
5. `KIRO_PROMPT_NEXT.md` and `TAWADOO_V2_MASTER_QUEUE.md` / `BRAIN_B16_MASTERY_2026_08_31.md` were **not** updated by me (Brain owns durable status per §19/§20). The Brain should mark S-CTO-15 FINISHED — COMPLETE (pending its own QA) and queue I1–I14.

---

## 5. PARALLEL-SESSION OVERLAP (§11 repo-isolation, honest note)

- I wrote **only** to `tawadoo_api_js` (4 files, commit `bd5cf2a`). Zero edits to web/bo/mcp (read-only greps only).
- **Parallel session S-CTO-R-RENDER** (read-only browser sweep on web) was authorized to run at the same time. No repo overlap — it's read-only on `tawadoo_web_js`; I only read web files, never wrote. No deployment overlap (it deploys nothing).
- At preflight, `tawadoo_api_js` was clean at `673eac0`; I ended with `bd5cf2a` pushed. My only workspace-root output is this report + the evidence file (not a git repo).

---

## 6. EVIDENCE LADDER (per §7)

`documented → source → local → CI → deployed → live` — **all layers reached.**
- source: A/B/C triage grep-proven; local: 25/25 targeted + 52/52 module, lint/tsc/build clean, fail-first red→green; CI: run 33974665602 (quality-gate ran tests = CI-certified); deployed: ECS rolloutState=COMPLETED, 2/2, digest `sha256:02947ef8…`; live: A canonical + unknown surfaced non-canonical + no regression, from `/ecs/tw-staging-back`.

---

## 7. RECOMMENDED NEXT SESSIONS (ranked, with one-line reasons)

1. **S-CTO-16 · Emitter renames/wiring (web-writable, coordinated api):** align the RENAME family (§2B) to the declared canonicals; add `app_install_banner_dismissed` canonical + rename its emitter together; wire dark surfaces; decide `smart_view_no_results_recovery` (I4); confirm I10 emitter status. Sequenced after S-CTO-15 declared the targets.
2. **S-CTO-17 · MCP → lake bridge (founder-gated, external contract):** I9 — forward MCP business events to the sovereign lake; resolve `search_refined` collision as `mcp_search_refined`; do not rename MCP names.
3. **S-CTO-18 · DB parity + BO gap + Amplitude registration + DEFAULT-partition probe:** I1, I6, I7, I8 — a read/admin session to close DB↔Amplitude↔S3 parity, register the new events on Amplitude, address BO emission, and investigate the 8 DEFAULT-partition rows.
4. **(housekeeping, Category-D, founder-authorized) test-health + `test_event` fixture migration:** I2 (6 failing suites + CI test-scope check) + I5.

---

## 8. TOOLS / PERMISSIONS / COST

- Tools: source grep/read (api writable; web/bo/mcp read-only), `git` (stash for fail-first, commit, push to Ramzi_V2), `gh run` (CI read), `yarn`/`jest`/`eslint`/`tsc`, AWS MCP **read-only** (ECS describe + CloudWatch FilterLogEvents), one HTTPS POST to the staging ingestion endpoint with synthetic data (no PII).
- No novel paid tool, no IAM change, no direct AWS mutation (only the CI pipeline's ECS deploy), no secret printed/stored, no prod touched. **COST: none incurred.**

---

_S-CTO-15 end-session report. Every undeclared event classified A/B/C from source; 28 A declared canonical; founder B-list produced (not built); C prune proposed; reject-unknown now surfaces non-breaking; stale doc fixed; fail-first red→green; CI green; deployed to Ramzi_V2 staging; live-verified. 14 open items (I1–I14) flagged for the Brain, including 4 inherited-open (I7 DB parity, I8 BO gap, I9 MCP bridge, I2 test debt) and 1 newly-found (I1 DEFAULT-partition rows). MCP untouched. Commit bd5cf2a on Ramzi_V2. Proposed FINISHED — COMPLETE; independent QA to re-verify from source._
