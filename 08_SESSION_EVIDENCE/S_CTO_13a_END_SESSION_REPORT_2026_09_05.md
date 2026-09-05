# S-CTO-13a — END-SESSION REPORT (for the Brain)

**Session:** S-CTO-13a · EVENT NAMING RECONCILIATION (READ-ONLY PROPOSAL)
**Date:** 2026-09-05 · **Mode:** MODE A discovery, strictly read-only
**Skills:** tawadoo-source-truth + tawadoo-data-sovereignty (both applied)
**Companion evidence:** `EVENT_NAMING_RECONCILIATION_2026_09_05.md` (the full founder-decision table lives there)
**Proposed status:** **FINISHED — COMPLETE** (for a read-only proposal session). Independent QA to re-verify from source per §18 before durable acceptance — this is a proposal to verify, not accepted truth.
**Parallel sessions live at the same time:** S-CTO-12 (browser/web QA). Overlap note in §5.

---

## 0. ONE-PARAGRAPH SUMMARY FOR THE FOUNDER

I built the full decision list you asked for: every event name that is drifting, undeclared, a new draft event, or MCP-specific now has a proposed action (rename / add / prune) and a risk flag in one table. The headline: **~23 names are just misspellings of events we already own** (fix = point the code at the name we already have), **~24 Smart View events should simply be added** (they already fire, they're just not registered), the **5 draft events should be added**, one name is junk (`test_event`), and the **MCP runs a totally separate event system that never reaches our data lake** and is partly a published ChatGPT contract — so it needs its own careful, founder-approved session and must not be renamed carelessly. Nothing was built or changed. Below I flag every loose end, including small ones and things outside my scope, and I spell out exactly what is "built in the back but not proven in the front / DB / BO / lake."

---

## 1. STATUS: FINISHED — COMPLETE (read-only proposal)

Acceptance criteria (all met):
- [x] Full name inventory from all sources, cited (companion §2).
- [x] Every drifted / undeclared / draft / MCP name has a proposed action + risk flag in one table (companion §3).
- [x] The `smart_view_*` verdict resolved (ADD, not noise) — companion §4.
- [x] The 5 draft-event proposals given canonical name + domain — companion §3.E.
- [x] MCP alignment proposal with external-contract flags — companion §5.
- [x] Stale-doc correction identified — companion §6.
- [x] "Do first" shortlist + recommended follow-up build sessions — companion §7–§8.
- [x] No code changed; 4 canonical HEADs unchanged (§6 below).

---

## 2. WHAT I ACTUALLY VERIFIED FROM SOURCE (not from prior reports)

Per §49 (no working from memory) I treated S-CTO-14 and S-CTO-11 as hypotheses and re-checked against source:

1. **Canonical allowlist — read in full.** `tawadoo_api_js/src/modules/analytics-ingestion/constants/allowed-events.ts` (Ramzi_V2 `673eac0`). The header says "492 events" per S-CTO-14; I did not re-count line-by-line but read every domain block.
2. **Ingestion contract — confirmed the stale-doc contradiction directly.** `analytics-ingestion.service.ts:52-67` logs unknown events and **persists** them with `_is_canonical=false` — it does **not** reject with HTTP 400 as the allowlist header claims. This is the mechanism that let 40 undeclared names into the lake.
3. **Smart View emitters — grepped the actual code**, which is MORE complete than S-CTO-14's "17 that landed": the code emits **~24 distinct `smart_view_*`** across `SmartViewPage.tsx`, `SmartViewBubble.tsx`, `write-actions.ts`, `ConfirmationCard.tsx`, `useVoiceInput.ts`, and `app/api/ai/guidance/route.ts`. The allowlist declares only 7. So under-declaration is bigger than the landing window showed.
4. **`smart_view_no_results_recovery`** — declared in the allowlist but I found **no emitter** in web source (only `smart_view_ai_interaction` appears, in the guidance route). → declared-not-wired (flag O5).
5. **`search_refined` collision — confirmed real on BOTH sides:** emitted canonically on web at `tawadoo_web_js/src/hooks/useSearchTracking.ts:111` AND present as an MCP behavior event in `_ALLOWED_CLIENT_BEHAVIOR_EVENTS`. Same string, two independent taxonomies.
6. **MCP taxonomy — read from source.** `-tawadoo-mcp-/src/client_mcp/server.py`: `_ALLOWED_CLIENT_BEHAVIOR_EVENTS` (8 client events, L930) + `record_business_event(...)` server events (`search_completed`, `recommendations_displayed`, `recommendations_display_failed`). These persist to a **local SQLite `business_events` table + `/tmp/client_mcp_business_events.jsonl`** (`observability.py`, `config.py`). Grep proved MCP emits **none** of the canonical `mcp_*` events and never POSTs to `/api/analytics/events`.
7. **5 draft events — from S-CTO-11 evidence:** `listing_draft_saved` (already in allowlist), `listing_draft_autosaved_local`, `listing_draft_recovered`, `listing_draft_resumed`, `listing_draft_discarded`; plus legacy `listing_saved_draft` (wrong-order twin, inert, dies with legacy form retirement).

---

## 3. WHAT IS "BUILT IN THE BACK" BUT NOT BUILT / NOT VERIFIED ELSEWHERE

This is the cross-surface truth you asked for, scoped to what this naming session touched.

### 3A. Ingestion (API) — BUILT, but the "reject unknown" contract was never really built
- **Built & live:** the sovereign ingestion path accepts *any* event name and tags unknowns `_is_canonical=false` (`analytics-ingestion.service.ts`, test F8 in `analytics-ingestion.service.spec.ts`).
- **Never built (contrary to the doc):** the HTTP-400 rejection of unknown events described in the allowlist header does **not** exist. So the "allowlist as a gate" is a documentation fiction; it's really a "canonical tag," not a filter. → this is *why* naming drift accumulates silently.

### 3B. Smart View (Web) — BUILT & FIRING, but NOT DECLARED and NOT (fully) VERIFIED landing
- **Built in the front:** ~24 `smart_view_*` emitters exist and fire via `track()`.
- **Not built in the allowlist:** ~17 are undeclared → persisted as `_is_canonical=false`; not first-class training signal (Commandment 2 gap).
- **Not verified in the lake:** S-CTO-14 saw 17 land in the 30-day window; the other ~7 emitters (`smart_view_action_failed`, `smart_view_action_confirmed`, `smart_view_hands_free_session`, `smart_view_voice_output_played`, `smart_view_voice_error`, `smart_view_confirmation`, `smart_view_user_correction`) are wired-but-unproven-landing (fire on rarer paths). Not confirmed at DB (`ta_analytics_event`) or S3-lake level by anyone yet.
- **Declared-not-wired:** `smart_view_no_results_recovery` is in the allowlist with no emitter found.

### 3C. Draft events (Web) — BUILT & FIRING, NOT REGISTERED on the tracking plan
- **Built in the front (S-CTO-11):** `listing_draft_saved` + 4 recovery events fire.
- **Not built in the allowlist:** the 4 recovery events are undeclared (`listing_draft_saved` IS declared).
- **Not built on Amplitude:** per S-CTO-11 O2, **none of the 5** is registered on the Amplitude tracking plan (project 795817) — not even the pre-existing `listing_draft_saved`. So they aren't first-class training signal yet.

### 3D. MCP — BUILT as a SEPARATE island, NOT bridged to the lake (the biggest gap)
- **Built in MCP:** 11 business events, persisted locally (SQLite + JSONL).
- **Not built:** any bridge from MCP business events to the sovereign `/api/analytics/events`. MCP interactions therefore **bypass sovereignty** (Commandment 2 violation in spirit).
- **Not built:** the canonical `mcp_query_received / mcp_listing_served / mcp_results_returned / mcp_user_converted` are declared in the allowlist but **never emitted** by MCP → the lake's MCP funnel is paper while the real signal sits in `/tmp`.
- **External-contract risk:** MCP event names + the `/business-metrics/event` payload are consumed by the published ChatGPT/OpenAI widget. Renaming any of them is founder-gated (could break external telemetry).

### 3E. DB — NOT inspected this session
- I did **not** run psql or query `ta_analytics_event`. "Landing" truth throughout is Amplitude-catalog-level (inherited from S-CTO-14), which is downstream of DB→delivery-worker→Amplitude. **Unverified:** whether every landed name also has DB rows + an S3 JSONL training pair. (Same gap S-CTO-14 flagged as ISSUE-5.)

### 3F. BO (`admin_bo_tawadoo`) — NOT in scope, NOT inspected for naming
- I confirmed BO HEAD unchanged only. I did not check BO for event emitters this session. S-CTO-14 already flagged that BO emits essentially no sovereign analytics (only `ask_ramzi_interaction`) — carried forward, not re-verified here.

### 3G. AWS / infra — NOT touched
- No AWS read or mutation, no ECS/task/log/S3 inspection, no cost, no IAM. Not in scope for a naming proposal.

---

## 4. ISSUES / ERRORS / OPEN ITEMS — FOR THE BRAIN TO QUEUE (including small + out-of-scope)

| # | Item | State | Severity | Owner / next |
|---|---|---|---|---|
| **O1** | **Stale allowlist header comment** — says unknown events "rejected with HTTP 400"; code persists them `_is_canonical=false` | CONFIRMED from source | LOW (but it's a source-of-truth lie) | Fix in S-CTO-15 (1-line, no logic change). Companion §6. |
| **O2** | **~17 undeclared `smart_view_*`** land as non-canonical | CONFIRMED (code emits ~24, allowlist has 7) | MED (Commandment 2 training gap) | ADD in S-CTO-15. Companion §3.D. |
| **O3** | **~7 more `smart_view_*` emitters wired but not proven landing** (`_action_failed`, `_action_confirmed`, `_hands_free_session`, `_voice_output_played`, `_voice_error`, `_confirmation`, `_user_correction`) | Source-confirmed emitters; landing UNVERIFIED | MED | ADD to allowlist (S-CTO-15) + confirm landing in a browser/lake session. |
| **O4** | **`search_refined` name collision** — canonical web event AND MCP behavior event, same string, two taxonomies | CONFIRMED both sides | MED | Resolve in the MCP-bridge session (S-CTO-16): forward MCP's as `mcp_search_refined` / `source:mcp`. Companion §5. |
| **O5** | **`smart_view_no_results_recovery` declared-not-wired** — in allowlist, no emitter found | CONFIRMED (no grep hit) | LOW | Verify surface intent → wire it or prune. Not this session. |
| **O6** | **5 draft events not on Amplitude tracking plan** (incl. pre-existing `listing_draft_saved`) + 4 recovery events undeclared in allowlist | From S-CTO-11 O2, re-noted | MED (training data) | ADD 4 to allowlist (S-CTO-15) + register all 5 on Amplitude. |
| **O7** | **Legacy `listing_saved_draft`** (wrong-order twin of `listing_draft_saved`) | Inert (legacy form off on staging) | LOW | PRUNE with legacy-form retirement (S-CTO-11 O8/O9). |
| **O8** | **`context_switch` semantics UNRESOLVED** — is it `role_switched` (rename) or Classic↔Smart view switch (add new)? | Could NOT resolve from source | LOW (founder decision) | **Founder micro-decision** — the one item I could not settle. Companion §3.A. |
| **O9** | **`ai_generated` rename target ambiguous** — `ai_generation_completed` vs `listing_ai_generated`; **`price_suggestion_shown`** — `seller_pricing_suggestion_seen` vs `prediction_price_suggested` | Two plausible canonical targets each | LOW-MED (founder decision) | Founder picks the target in S-CTO-15b. Companion §3.A. |
| **O10** | **`app_install_banner_*` family lacks a `_dismissed`** — `app_banner_dismissed` lands with no canonical twin | CONFIRMED (family has shown/clicked only) | LOW | ADD `app_install_banner_dismissed` before renaming the emitter. |
| **O11** | **MCP → lake bridge does not exist** — MCP business events never reach `/api/analytics/events`; 4 canonical `mcp_*` never emitted | CONFIRMED from source | MED (sovereignty, Commandment 2) | S-CTO-16, founder-gated (external contract). Companion §5. |
| **O12** | **DB↔Amplitude↔S3 parity for renamed/added events NOT verified** | Not checked (no DB access this session) | MED | A DB/lake read session (S-CTO-14 ISSUE-5, still open). |
| **O13** | **492 count not independently re-counted** — I read every block but relied on S-CTO-14's "492" total | Method limitation | LOW | If an exact count matters for a report, re-count in the S-CTO-15 edit session. |
| **O14** | **BO analytics-emission gap** (only `ask_ramzi_interaction`) | Inherited from S-CTO-14 §2C, not re-verified | MED (if admin actions should feed the lake) | Founder decision + a BO session. |

**Out-of-scope items I did NOT touch but that intersect naming:** the auth-gate family rename (§3.B) spans web+api and is MED-risk because both spellings land — must be sequenced (add canonical → switch emitter → let old spelling age out; never prune the live twin first). Recorded so a future session doesn't prune a canonical whose duplicate is still landing.

---

## 5. PARALLEL-SESSION OVERLAP (honest note, §11 repo-isolation)

- I performed **zero repo writes/edits/commits/deploys**. My only output is `EVENT_NAMING_RECONCILIATION_2026_09_05.md` + this report, both at the workspace root (`/Users/ramzihannachi/Code`, not a git repo) — touches no branch.
- S-CTO-12 (browser) was live on `tawadoo_web_js` in parallel; I only read files there, never wrote. The web HEAD I read was `780bdb58` (S-CTO-11's shipped tip); any further advance during S-CTO-12's turn does not affect a read-only proposal.
- Read-only guarantee holds — see §6.

---

## 6. READ-ONLY GUARANTEE / PROVENANCE

All four canonical HEADs verified unchanged at session start and end (Ramzi_V2):
- `tawadoo_api_js` `673eac0` · `tawadoo_web_js` `780bdb58` · `-tawadoo-mcp-` `d8efb4a` · `admin_bo_tawadoo` `07b3a84`.
- Zero code/allowlist edits, zero commits, zero deploys, zero browser, zero prod, zero AWS/DB mutation, zero cost.

---

## 7. RECOMMENDED NEXT SESSIONS (ranked)

1. **S-CTO-15 · Allowlist reconciliation ADD/PRUNE + stale-doc fix (MODE B, `tawadoo_api_js` writable).**
   *Reason:* ADD the ~17 `smart_view_*` + ~7 wired-unproven (O2/O3), the 4 draft-recovery events (O6), 3 store events, and `app_install_banner_dismissed` (O10); PRUNE `test_event`; fix the header comment (O1). Single-repo, additive, low-risk, unblocks everything downstream. **Do first** — no new work should be built on undeclared names.
2. **S-CTO-15b · Emitter renames (MODE B, `tawadoo_web_js` + coordinated `tawadoo_api_js`).**
   *Reason:* the duplicate-spelling + auth-gate renames (companion §3.A/B) touch live-landing names across two repos; run after S-CTO-15 has declared any missing canonical targets, sequenced per the "never prune the live twin first" rule. Carries founder decisions O8 (`context_switch`) and O9 (`ai_generated`/`price_suggestion_shown` targets).
3. **S-CTO-16 · MCP → lake bridge + `search_refined` collision (MODE B, `-tawadoo-mcp-` writable, FOUNDER-GATED).**
   *Reason:* external ChatGPT/OpenAI contract + a new network path from MCP to the sovereign lake; must be its own bounded, founder-approved unit (O4/O11). Bridge (map), don't rename.
4. **S-CTO-17 · DB/lake parity + BO emission gap (read session).**
   *Reason:* close O12 (DB↔Amplitude↔S3 parity) and O14 (BO barely feeds the lake) — both inherited-open and needed before any "sovereignty complete" claim.

---

## 8. TOOLS / PERMISSIONS / COST

- Tools: source `read_file` + `grep_search` across `tawadoo_api_js`, `tawadoo_web_js`, `-tawadoo-mcp-`; two prior read-only evidence files (S-CTO-14, S-CTO-11) re-verified against source; branch/HEAD checks via `git rev-parse` (read-only).
- No novel paid tool, no IAM change, no AWS/DB mutation, no browser, no commit/deploy, no prod. **COST: none incurred.**

---

_S-CTO-13a end-session report. Read-only naming reconciliation. Full founder-decision table in the companion file; 14 open items flagged (O1–O14), including 2 founder micro-decisions (O8, O9) and 4 inherited-open items (O11 MCP bridge, O12 DB parity, O13 count, O14 BO). All four canonical HEADs unchanged. Proposed FINISHED — COMPLETE; independent QA to re-verify from source._
