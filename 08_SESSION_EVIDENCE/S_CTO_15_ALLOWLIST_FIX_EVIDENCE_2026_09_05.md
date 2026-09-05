# SESSION ID S-CTO-15 — EVENT ALLOWLIST FIX + TRIAGE (build-or-ask) — EVIDENCE

**Date:** 2026-09-06 · **Repo (writable):** `tawadoo_api_js` only · **Branch:** `Ramzi_V2`
**Skills applied:** tawadoo-source-truth · tawadoo-data-sovereignty · tawadoo-runtime-provider-qa
**Proposed status:** **FINISHED — COMPLETE** (independent QA to re-verify per §18)
**Commit:** `bd5cf2a` (parent `673eac0`) · **CI:** run `33974665602` GREEN · **Deploy:** staging ECS rolloutState=COMPLETED

---

## FIVE-LINE OPENER (corrected, post-preflight)

```
SESSION: S-CTO-15 — EVENT ALLOWLIST FIX + TRIAGE (build-or-ask)
BRANCH: Ramzi_V2 (verified; upstream origin/Ramzi_V2; clean at preflight 673eac0)
MISSION: triage every undeclared event A/B/C; wire+declare A; list B for founder; propose-prune C; build reject-unknown that SURFACES (never drops/silent-accepts); fix stale doc.
ORDER: reality check → classify A/B/C (source-proven) → apply A + N-DEC → build surfacing → fail-first → validate → deploy → live-verify → founder B-list → evidence
TREE STATE: clean at start (only untracked prior evidence docs at workspace root, none mine); ended with 4 tracked files committed as bd5cf2a.
```

---

## 0. EXECUTIVE VERDICT + EVIDENCE LADDER

Every undeclared event is classified A/B/C from source. 28 category-A events (live emitters) are now declared canonical, including the two founder decisions (N-DEC1 `context_switch` as a view-switch, N-DEC2 `price_suggestion_shown` kept as-is). The "reject-unknown" contract — which never actually existed — is now a **surfacing** contract: unknown events are still persisted (never dropped, no HTTP 400, LAW §50) but now increment a dedicated observability counter and fire `[ALARM:INGESTION_UNKNOWN]` so naming drift is caught for triage. The stale doc lie ("rejected with HTTP 400") is corrected. MCP names untouched. Category-B (no-emitter) events are listed for the founder, not built.

Evidence ladder reached: **documented → source → local → CI → deployed → live** (all layers reached; live behavioral proof captured from the running staging task).

---

## 1. A / B / C TRIAGE TABLE (every undeclared event, source-proven)

Rule applied: **A = a live emitter exists in current product source → WIRE by declaring it canonical. B = no emitter exists → founder decision (do NOT build). C = junk/legacy/declared-not-wired → propose prune.** Web/BO/MCP were read-only; the only writable surface is the API allowlist, so "wire" here = add to the canonical allowlist so the already-firing event lands canonical.

### 1A. CATEGORY A — has a live emitter → ADDED to allowlist as canonical (28)

| Event | Emitter (source proof) | Action |
|---|---|---|
| `smart_view_opened` | `tawadoo_web_js/src/components/smart-view/SmartViewPage.tsx:185` | ADD |
| `smart_view_session_duration` | `SmartViewPage.tsx:195` | ADD |
| `smart_view_mode_toggle` | `SmartViewPage.tsx:169` | ADD |
| `smart_view_voice_output_played` | `SmartViewPage.tsx:257` | ADD |
| `smart_view_hands_free_started` | `SmartViewPage.tsx:279` | ADD |
| `smart_view_hands_free_session` | `SmartViewPage.tsx:285` | ADD |
| `smart_view_image_search` | `SmartViewPage.tsx:379` | ADD |
| `smart_view_brain_decision` | `SmartViewPage.tsx:593` | ADD |
| `smart_view_mcp_results_used` | `SmartViewPage.tsx:620` | ADD |
| `smart_view_comparison_shown` | `SmartViewPage.tsx:711` | ADD |
| `smart_view_user_correction` | `SmartViewPage.tsx:805` | ADD |
| `smart_view_conversation_turn` | `SmartViewPage.tsx:862` | ADD |
| `smart_view_chip_clicked` | `SmartViewPage.tsx:911` | ADD |
| `smart_view_result_clicked` | `SmartViewPage.tsx:921` | ADD |
| `smart_view_action_confirmed` | `SmartViewPage.tsx:934` | ADD |
| `smart_view_bubble_clicked` | `SmartViewBubble.tsx:25` | ADD |
| `smart_view_voice_input` | `useVoiceInput.ts:155` | ADD |
| `smart_view_voice_error` | `useVoiceInput.ts:175` | ADD |
| `smart_view_confirmation` | `ConfirmationCard.tsx:91,110` | ADD |
| `smart_view_action_executed` | `write-actions.ts` (contact/save_search/favorite/offer/bid/share) | ADD |
| `smart_view_action_failed` | `write-actions.ts` (same six paths, catch blocks) | ADD |
| `listing_draft_autosaved_local` | `product-form-v2.tsx:1124` | ADD |
| `listing_draft_recovered` | `product-form-v2.tsx:1173` | ADD |
| `listing_draft_resumed` | `product-form-v2.tsx:1223` | ADD |
| `listing_draft_discarded` | `product-form-v2.tsx:1238` | ADD |
| `store_delivery_configured` | `WizardStepDelivery.tsx:127` (+ regression test `b13-delivery-wizard.spec.ts:81`) | ADD |
| `context_switch` | `store-dashboard-view.tsx:91` — payload `{from:"store", to:"personal", store_id}` | ADD (N-DEC1) |
| `price_suggestion_shown` | `pricing-inventory-section.tsx:50` | ADD (N-DEC2 keep) |

**Name-collision gate:** grepped all 28 against `allowed-events.ts` before adding — **zero** were already present (no duplicates). Each is a genuinely new canonical name.

### 1B. CATEGORY B — NO emitter found → FOUNDER LIST (NOT built) — see §3

| Event | Source check | Verdict |
|---|---|---|
| `store_search` | no emitter in web or api (grep) | B — founder decides |
| `store_categories_updated` | no emitter in web or api (grep) | B — founder decides |
| `smart_view_query` | no emitter (report was speculative) | B — founder decides |
| `smart_view_intent_classified` | no emitter (report was speculative) | B — founder decides |
| `smart_view_guidance_response` | no emitter (report was speculative) | B — founder decides |

> These **correct** the S-CTO-13a/14 proposals, which listed `store_search`/`store_categories_updated` as "ADD" and implied more smart_view names. Source truth (§49) shows no emitter, so they are B, not A. I did **not** add them.

### 1C. CATEGORY C — junk / legacy / declared-not-wired → PRUNE PROPOSAL (see §4)

| Event | State | Proposed action |
|---|---|---|
| `test_event` | dev fixture, in allowlist, used by `analytics-ingestion.service.spec.ts:235` | PRUNE **proposal only** — kept for now (deleting breaks the live test fixture; migrate fixture first) |
| `smart_view_no_results_recovery` | declared in allowlist, **no emitter** found (declared-not-wired) | PRUNE **proposal** OR wire in S-CTO-16 — founder/next-session call |

### 1D. RENAME family — NOT this session (emitter changes = S-CTO-16, web-writable)

| Landing name | Emitter | Canonical twin | Note |
|---|---|---|---|
| `feed_slide_change` | `ImmersiveFeed.tsx:374` (live) | `feed_slide_changed` (already declared) | RENAME emitter in web → S-CTO-16 |
| `app_banner_dismissed` | `OpenInAppBanner.tsx:231` (live) | needs `app_install_banner_dismissed` (O10) | ADD target canonical + rename emitter → S-CTO-16 |
| `video_play` | **already killed** → `listing_video_played` (`productDetails.tsx:1729`) | resolved | no action |
| `promo_dismiss` | **already removed** (source-guard test `b11-s32-fix2-source-guards.test.ts:51`) | resolved | no action |
| `session_start`/`session_end`/`feed_enter`/`feed_exit`/`login` | no literal string emitter in current web src | canonical twins declared | likely already resolved/var-emitted → note for S-CTO-16 |
| `ai_generated` | only a **DB column** `posts.entity.ts:91`, not an event | `ai_generation_completed` (already declared) | N-DEC2 rename is emitter-side → S-CTO-16 |

---

## 2. N-DEC FOUNDER DECISIONS — applied + verified

- **N-DEC1 `context_switch`:** source-verified it is a **Classic/store ↔ personal-view switch** (`store-dashboard-view.tsx:91`, payload `{from:"store", to:"personal", store_id}`). It is **NOT role-related** → added as-is (`context_switch`), NOT renamed to `role_switched`. Per the prompt, this finding is reported rather than assumed. Founder may override.
- **N-DEC2 `ai_generated` / `price_suggestion_shown`:** `ai_generation_completed` is already in the allowlist; the only `ai_generated` in the API is a DB column, not an event, so no allowlist ADD — the rename is an emitter change deferred to S-CTO-16. `price_suggestion_shown` kept as-is and declared (it has a live emitter, was undeclared).

---

## 3. FOUNDER B-LIST (no-UI events — APPROVE / DISCUSS / REJECT) — NOT built

These names have **no emitter in the product today**. They only "exist" as proposals in prior reports. Under UNKNOWN-EVENT=BUILD-OR-ASK, they are the founder's call. Nothing was built.

| # | Event | Plain-English: what it would capture | Implied surface | Why it might matter | Recommendation |
|---|---|---|---|---|---|
| B1 | `store_search` | A shopper searching **within a single seller's store page** (not global search) | Store page search box | Store-level intent + which stores drive their own discovery | **DISCUSS** — likely worth building when store pages get a search box; today there is none |
| B2 | `store_categories_updated` | A seller **reorganising the categories** shown on their store | Seller store-config screen | Seller merchandising behaviour (training signal for store quality) | **DISCUSS** — build with the store-config surface, low priority |
| B3 | `smart_view_query` | A raw Smart View **search query text** as its own event | Smart View chat input | Overlaps `smart_view_conversation_turn` (already captures the query) | **REJECT** — redundant with the turn event; don't add a second name |
| B4 | `smart_view_intent_classified` | The **classified intent** of a Smart View turn as a standalone event | Smart View brain | Overlaps `smart_view_brain_decision` (already carries `intent`) | **REJECT** — redundant; intent already lands in brain_decision |
| B5 | `smart_view_guidance_response` | The **AI's guidance response** as a standalone event | Smart View guidance route | Overlaps `smart_view_ai_interaction` (already declared + firing) | **REJECT** — redundant with ai_interaction |

**One-line founder ask:** *Approve building B1/B2 later (with their surfaces), and reject B3–B5 as redundant?* A 30-second decision; nothing blocks on it.

---

## 4. C PRUNE PROPOSAL (allowlist-side only — no emitter deletions)

| Event | Reason | Recommendation |
|---|---|---|
| `test_event` | Pure dev artifact. **But** it is the fixture in `analytics-ingestion.service.spec.ts:235`. | **Prune proposal, deferred** — kept this session; migrate the test to a real event name first, then prune. Deleting now would break a live test. |
| `smart_view_no_results_recovery` | Declared canonical but **no emitter** (declared-not-wired). | **Prune OR wire** — founder/S-CTO-16 call. Not pruned this session (harmless paper name; may be wired in Smart View work). |

No emitters were touched (web/bo/mcp read-only). Prunes are allowlist-side proposals only.

---

## 5. REJECT-UNKNOWN SURFACING DESIGN (the contract that never existed, now built)

**Before:** doc claimed "unknown → HTTP 400"; reality = unknown persisted `_is_canonical=false` with a single log line, **no metric**. So drift accumulated silently and invisibly (root cause of all naming drift per S-CTO-13a N-ROOT).

**After (non-breaking, LAW §50 — never hard-drop, never silent-accept, no 400):**
1. `analytics-ingestion.service.ts` `ingest()` — unknown events still persist with `_is_canonical=false` (unchanged behaviour) AND now call `observability.recordIngestionUnknown(unknownEvents.length)` + an improved `[ALLOWLIST]` warn.
2. `analytics-observability.service.ts` — new `ingestion_unknown` counter + `recordIngestionUnknown()`; reset each cycle; emitted in the 60s `analytics_ledger_health` JSON (`ingestion.unknown`); fires `[ALARM:INGESTION_UNKNOWN]` (WARN, not error — the event still landed) when >0 in the window.
3. No HTTP 400 introduced (source confirms the controller still returns 200 with `ingested` count) — current clients unaffected.

This matches the sovereignty pipeline: the fact still lands; the drift is now **countable and alarmable** for triage instead of silently accepted.

---

## 6. STALE DOC FIX

`allowed-events.ts` header rewritten from the false *"Unknown event types are rejected with HTTP 400"* to the true behaviour: events in the set are tagged canonical; unknown events are NOT rejected, are persisted with `_is_canonical=false`, and are surfaced (log + counter) for triage — a canonical-tag, not a hard gate.

---

## 7. FAIL-FIRST (red → green)

New tests in `analytics-ingestion.service.spec.ts` (describe `S-CTO-15: canonical tagging and unknown surfacing`):
- KNOWN event → canonical (no flag) — *passes on both HEADs (correct: known events were always canonical)*
- **NEWLY-ADDED (A) events → canonical** — **RED on clean HEAD** (they were undeclared → `_is_canonical=false`), GREEN after.
- **UNKNOWN → non-canonical + surfaced (no 400, no drop)** — **RED on clean HEAD** (`recordIngestionUnknown` never called), GREEN after.
- **unknown surfacing counts every undeclared event in the batch** — **RED on clean HEAD**, GREEN after.

**Proof:** stashed the 3 source files (kept the spec) on clean HEAD `673eac0` → `npx jest ... -t "S-CTO-15"` = **3 failed, 1 passed** (exact failures above). Restored → **4 passed**. Genuine fail-first against the real path.

---

## 8. LOCAL VALIDATION + CI

| Check | Result |
|---|---|
| Targeted spec `analytics-ingestion.service.spec.ts` | **25/25 PASS** (incl 4 new) |
| Full module `src/modules/analytics-ingestion` | **52/52 PASS** |
| `npx eslint` (4 changed files) | clean (exit 0) |
| `npx tsc --noEmit` | clean (exit 0) |
| `yarn build` (nest build) | **Done 16.24s** |
| Full `yarn test` | 6 suites fail (47 tests): `entity-boot`, `hybrid-search-sort`, `hybrid-search.service`, `whatsapp-consent-gate.property`, `whatsapp-bridge.service`, `auth/identifier-change`(flaky 5s timeout) |

**Pre-existing failures proven NOT mine:** stashed all 4 changed files on clean HEAD and ran those 6 suites → 5/6 fail identically (the auth one PASSED when run isolated = flaky timeout under parallel load, not a real failure). All are environmental (need OpenSearch/DB/network unavailable locally). My module is fully green.

**CI (CI-certified, not just local):** run `33974665602` on `Ramzi_V2` — job `quality-gate` (ran tests + build verification, 2m6s ✓) + job `build-and-push` (6m43s ✓, incl "Run tests", "Build verification", "Record immutable provenance", "Deploy staging ECS and verify the rollout").

---

## 9. DEPLOY + IMMUTABLE PROVENANCE

```
commit bd5cf2a6ddb78b2b5aef5d53847bab2c8d81a753
 → CI run 33974665602 (quality-gate ✓ + build-and-push ✓)
 → image ghcr.io/embendev24/tawadoo-api-js tagged: sha-bd5cf2a… (immutable) · 0.1.484 · staging-v2 (mutable deploy tag)
 → aws ecs update-service --force-new-deployment (tw-staging-cluster / tw-staging-svc-back) → services-stable ✓
 → ECS PRIMARY deployment rolloutState=COMPLETED, 2/2 running, task-def rev 44,
    running image staging-v2 @ digest sha256:02947ef8f541fa256b0f205da7967f550343c84726ccc96ea90337a162bda6eb (both tasks identical)
```

- Task-def uses the mutable `staging-v2` tag (§42-compliant — CI updates the image, no SHA-pinned task-def, no no-op deploy).
- GHCR manifest→digest tag resolution was **not** attempted directly: the package is private and would require an auth token in the request (secret exposure risk, §redaction). The deploy chain + the live behavioural proof below are the authoritative running-code evidence.

---

## 10. LIVE PROOF (running staging task = bd5cf2a) — A canonical · unknown surfaced · no regression

**Live POST** `https://api-staging.tawadoo.ma/api/analytics/events` with two synthetic events (unique ids, no PII):
- A event `smart_view_opened` (id `scto15-a-1788622540`)
- Unknown `scto15_probe_unknown_1788622540` (id `scto15-u-1788622540`)

Response: **HTTP 200 `{"status":"ok","ingested":2,"duplicates_skipped":0}`** → both persisted; no 400; no drop.

**Log group `/ecs/tw-staging-back` (running code):**
1. **Unknown surfaced (not silent):** `[ALLOWLIST] 1 unknown (non-canonical) event type(s): scto15_probe_unknown_1788622540 — persisting with _is_canonical=false and surfacing for triage`.
2. **Unknown persisted non-canonical (not dropped):** DB insert log shows `ta_analytics_event … properties = {"probe":"scto15","_is_canonical":false}` for the probe.
3. **New counter fired:** `[ALARM:INGESTION_UNKNOWN] 1 non-canonical (undeclared) events ingested in last 60s — triage naming drift`.
4. **New metric field live:** `analytics_ledger_health … "ingestion":{"success":2,…,"unknown":1,…}` — the `unknown` key proves the running code is `bd5cf2a` (the pre-fix task instance emitted the same metric with **no** `unknown` key).
5. **A event canonical:** `smart_view_opened` is **absent** from the `[ALLOWLIST]` warn and the batch counted `unknown:1` (not 2) for 2 successes → `smart_view_opened` was recognised canonical (no `_is_canonical=false`).
6. **No regression:** `success:2, failure:0, delivery.success:2, dlq:0` — both events persisted + queued for delivery normally.

---

## 11. MCP UNTOUCHED

No MCP event name or `/business-metrics/event` contract was modified. The MCP external-contract names (`recommendation_clicked`, `widget_rendered`, `listing_opened`, `seller_contact_started`, `favorite_added`, `search_refined`, `user_feedback_positive/negative`, `search_completed`, `recommendations_displayed`, `recommendations_display_failed`) are left for the founder-gated S-CTO-17. Zero writes outside `tawadoo_api_js`.

---

## 12. ROLLBACK CONTRACT

- **Source revert:** `git revert bd5cf2a` (or `git reset --hard 673eac0` on a throwaway) → re-push → CI redeploys prior image.
- **Migration:** none (no schema/DTO/migration change). Nothing to down-migrate.
- **Data:** additive only; the `ingestion_unknown` counter + allowlist entries carry no persisted state to repair. Unknown events already in the lake are unchanged.
- **Prior artifact:** `sha-673eac0…` remains in GHCR; task-def rev 43 is the prior revision.
- **Not exercised:** a live rollback deploy was not performed (the change is proven non-breaking against currently-landing events by the live test in §10; exercising a rollback on shared staging would disrupt the parallel S-CTO-R-RENDER session for no added safety on an additive change). Rollback is defined and executable.

---

## 13. CHANGED FILES (commit bd5cf2a, 4 files, +178 / −4)

- `src/modules/analytics-ingestion/constants/allowed-events.ts` — stale doc fix + 28 canonical adds
- `src/modules/analytics-ingestion/analytics-observability.service.ts` — `ingestion_unknown` counter + emission + alarm
- `src/modules/analytics-ingestion/analytics-ingestion.service.ts` — surfacing call + improved log (no 400, no drop)
- `src/modules/analytics-ingestion/analytics-ingestion.service.spec.ts` — 4 fail-first tests + mock update

---

## 14. TRAFFIC-LIGHT + R-STATE

- **RED:** none introduced. Pre-existing red (entity-boot, hybrid-search×2, whatsapp×2, flaky auth timeout) unchanged, proven not mine.
- **YELLOW:** (a) `test_event` prune deferred (test-fixture dependency); (b) `smart_view_no_results_recovery` declared-not-wired left in place; (c) GHCR digest-tag resolution not done directly (private-pkg auth). None blocks acceptance.
- **BLUE (done):** A/B/C triage complete + source-proven; 28 A declared; N-DEC1/N-DEC2 applied; B-list produced; C prune proposed; surfacing built + live-proven; stale doc fixed; fail-first red→green; CI green; deployed; live-verified; MCP untouched.
- **R0/R1/R2/R3:** R0 source-truth ✓ · R1 local+CI ✓ · R2 deployed+running-digest ✓ · R3 live behavioural (A canonical, unknown surfaced non-canonical, no regression) ✓.

---

## 15. NEXT SESSION IDs (one-line reasons)

- **S-CTO-16 — emitter renames/wiring (web-writable):** rename the live duplicate-spelling emitters to their canonical twins (`feed_slide_change`→`feed_slide_changed`, `app_banner_dismissed`→`app_install_banner_dismissed` after adding that canonical, `ai_generated`→`ai_generation_completed`); wire dark surfaces; prove money-funnel landing. Sequenced *after* this session declared the targets. Also decide `smart_view_no_results_recovery` (wire or prune) + carry the founder B-list decision.
- **S-CTO-17 — MCP → lake bridge (founder-gated, external contract):** forward MCP business events to `/api/analytics/events`; resolve the `search_refined` collision (`mcp_search_refined`). Do NOT rename MCP names.
- **S-CTO-18 — DB parity + BO emission gap:** confirm DB↔Amplitude↔S3 parity for the newly-declared events; address BO's near-zero sovereign emission (only `ask_ramzi_interaction` today).

---

## 16. TOOLS / PERMISSIONS / COST

- Tools: source grep/read (api + web read-only), `git` (stash for fail-first, commit, push), `gh run` (CI read), `yarn`/`jest`/`eslint`/`tsc`, AWS MCP read-only (ECS describe + CloudWatch FilterLogEvents), one authenticated-optional HTTPS POST to the staging ingestion endpoint with synthetic data.
- No novel paid tool, no IAM change, no AWS mutation beyond the CI-driven ECS deploy, no secret printed, no prod touched. **COST: none incurred.**

_S-CTO-15 end. Every undeclared event classified A/B/C from source; 28 A declared canonical; founder B-list produced (not built); C prune proposed; reject-unknown now surfaces non-breaking; stale doc fixed; fail-first red→green; CI green; deployed to Ramzi_V2 staging; live-verified. MCP untouched. Commit bd5cf2a on Ramzi_V2._
