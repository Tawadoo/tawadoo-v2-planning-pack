# TAWADOO V2 — PLANNING PACK (for ChatGPT co-planning)

> ⚠️ **TEMPORARY / DISPOSABLE REPO.** This repository exists for ONE task: to let an independent planner (ChatGPT) read the full Tawadoo V2 program state + the systemic navigation finding and help plan the fix, refactor, and road to launch. **It will be DELETED right after this planning task.** It is a private snapshot of internal planning docs — do not fork, share, or rely on it long-term.

---

## WHY YOU (ChatGPT) ARE READING THIS
Tawadoo V2 is a Moroccan marketplace mid-refactor. The engineering agent (Kiro) has done many sessions — and, candidly, **some of the current mess was created by earlier Kiro sessions.** So we do NOT want the same agent that created the problems to also be the sole planner of the fix. Your job: read everything here, independently, and help the founder (Ramzi, non-technical) plan **what to build, how, when, and how to refactor** — with a critical, outside eye. Challenge Kiro's proposed plan where the evidence warrants.

## THE SITUATION IN ONE PARAGRAPH
A read-only audit (S-CTO-25-NAV-MAP) just confirmed a **systemic entity-addressing / navigation-integrity defect**: the "item no longer available", broken sharing, and dead deep-links are not 11 isolated bugs — they collapse into **3 root patterns**. The sharpest proof: the SAME published+verified listing returns HTTP 200 from one endpoint (`publications/slug/:slug`) and HTTP 404 from another (`publications/lightssr/:slug`) because the two use **different visibility flags** (`isVerified` vs `isModerated`) and live data has verified listings with `isModerated = null`. One page served by two endpoints that disagree about whether the listing exists.

## READ IN THIS ORDER
1. **`02_PLAN_AND_FINDING/`**
   - `TAWADOO_V2_MASTER_PLAN_FOR_CHATGPT_2026_09_06.md` — **START WITH THIS.** The complete master plan: how the program operates, the full journey B16→now, the whole queue across all phases, the events/Smart-View/ACP picture, the Bayesian moat, red lines, and an explicit "what to help plan" section.
   - `S_CTO_25_NAV_MAP_EVIDENCE_2026_09_06.md` — the navigation audit: the 7 artifacts (Entity Addressing Matrix, Failure Matrix, 3 root-cause patterns, Canonical Target Model, Migration Surface, App/MCP Compat, Stage-2 decision).
   - `NAV_INTEGRITY_STEERING_DIRECTIVE.md` — the founder directive that spawned the audit (the spec it answered).
   - `KIRO_EXEC_PROMPT_S_CTO_25_NAV_MAP_2026_09_06.md` — the exact prompt the audit ran under (shows the discipline/guardrails).
2. **`03_CONTROL_STATE/`** — the live control plane + full state: `TAWADOO_V2_CTO_OPERATING_SYSTEM.md` (laws + session ledger + FR register + moderation M-ZONE + parked registers), `TAWADOO_V2_MASTER_QUEUE.md`, `TAWADOO_BUSINESS_TRUTH_2026_09_06.md`, `HANDOFF_B22_CTO_2026_09_06.md`, `B22_MASTER_RECONCILIATION_2026_09_05.md`, `BRAIN_B16_MASTERY_2026_08_31.md`.
3. **`04_ARCHITECTURE/`** — the approved target: `TAWADOO_V2_TARGET_ARCHITECTURE.md` (5-layer modular monolith + 10 invariants), `TAWADOO_V2_PRE_ARCHITECTURE_TRUTH_REPORT.md`, `AD-001_SCHEMA_OWNERSHIP_DETAILED_PLAN.md`, `TAWADOO_V2_FRONTEND_ARCHITECTURE.md`, `B16_S140_PROD_BASELINE_AUDIT_2026_09_01.md`.
4. **`05_INTELLIGENCE_MOAT/`** — the Bayesian/hidden-state model + events/taxonomy: `SESSION_REPORT_INTELLIGENCE_LAKE_HIDDEN_STATE_2026_08_31.md`, `CTO_BEHAVIORAL_INTELLIGENCE_GLOBAL_RESEARCH_2026_08_20.md`, `TAWADOO_V2_AI_VISION_COMPLETE_2026_08_19.md`, `TAWADOO_V2_FULL_EVENT_TAXONOMY_2026_08_19.md`, `TAWADOO_EVENT_NAMING_TRUTH_2026_08_20.md`.
5. **`06_DISTRIBUTION/`** — `GOOGLE_MERCHANT_APPEAL_2026_08_23.md` (the ban history that makes feed guards sacred).
6. **`07_LAWS_STEERING/`** — the binding operating laws: `00-EXECUTION-PROMPT-NON-REGRESSION-LAW.md` (§0–§52), `01-B14-CTO-STANDARD.md` (3 Commandments), `02-REFACTOR-PROGRAM.md`.
7. **`08_SESSION_EVIDENCE/`** — every S-CTO session's evidence/report (the raw source-verified findings behind the state).

## WHAT WE WANT FROM YOU
Ground your planning in the master plan's §11 ("what ChatGPT should help plan"). Specifically:
1. Sanity-check the **canonical entity-addressing contract** the audit proposes (Artifact 4 + 7) — is it minimal, correct, in the right layer, free of over-abstraction? Would it actually make the product harder to break?
2. Sequence **Stage 2 (NAV-FIX)** — canonical mechanism first, then which callers to migrate in what order, with what regression matrix, without breaking Classic, mobile, or the live MCP/app contracts.
3. Sequence the **rest of functional completion** (the FR wave, auth flow, events wire-and-prove, search-intelligence) to ~90–95% functional with fewest sessions and zero regression (max 2 concurrent Kiro sessions).
4. Advise on the **security gate**, the **5-layer refactor slicing (R0→R6)**, and **when the moat (Bayesian) data foundation is ready** to start.
5. **Where you disagree with Kiro's proposed plan, say so and why** — that is the point of this independent review.

## GROUND RULES FOR THE PLAN
Respect the North Star (clean human-quality refactor) + the phase order. Prod stays protected. Inventory/notice ≠ roadmap. Every recommendation = a bounded unit with verification + rollback. Surface founder decisions (don't pre-decide them). Pattern over instance. Smallest correct change. No AI fingerprints.

## THE TWO OPEN BLOCKERS THE AUDIT COULD NOT CLOSE (need Stage 2)
1. **Blast radius of `isModerated = null`** — how many live verified listings are affected — needs an authorized DB/API sweep (psql is in-VPC/IP-locked).
2. **Fresh-browser visual confirm** of the 200/404 symptom — the audit proved it at the HTTP level only.

*(Snapshot date: 2026-09-06. Live git at snapshot: web `8b0a815f` · api `8d55a20` · bo `07b3a84` · mcp `d8efb4a`, all `Ramzi_V2`.)*
