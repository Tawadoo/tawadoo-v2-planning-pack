# S-CTO-25-NAV-MAP — ENTITY & NAVIGATION INTEGRITY MAP (Stage 1: read-only investigation, binding on Stage 2)
**STATUS:** OPEN/PENDING — authorized to execute on fire. **READ-ONLY. DO NOT IMPLEMENT.**
**CLASSIFICATION:** A — Refactor/Architecture (primary). This is THE refactor unit, ahead of the FR wave. The FR findings (esp. FR-9 notif deadlink, FR-11 mark-lead, "item no longer available", broken share) are INPUTS/evidence, NOT the roadmap.
**BATCH/WAVE:** Systemic entity-addressing refactor, Stage 1 of 2. Runs ALONE (no parallel writer — it's the priority unit and it reads across all repos). Machine: 1 session.
**ENVIRONMENT:** staging only for any runtime probe. Prod FORBIDDEN. **No code, no deploy, no migration, no notification/share/route change, no app/MCP change.**
**READABLE REPOS (full investigation):** `tawadoo_web_js`, `tawadoo_api_js`, `admin_bo_tawadoo` + DB source (schema/entities/migrations) + Search/OpenSearch index shape + live staging runtime (read-only probes).
**READABLE FOR CONTRACT INVENTORY ONLY (no fix, no design-from):** `-tawadoo-mcp-`, `tawadoo_mobile_app`, `tawadoo_app_mobile_ui_only`.
**WRITABLE:** the evidence file ONLY → `/Users/ramzihannachi/Code/S_CTO_25_NAV_MAP_EVIDENCE_2026_09_06.md`. No product code in any repo. (If browser/probe scratch files are unavoidable, put them under `/tmp` and delete them; never commit.)
**TOOLS / PERMISSIONS / COST:** NONE new. Existing local read + authorized staging read (kiro-ai read, HTTP, Chromium/WebKit browser). DB is in-VPC/IP-locked + ECS-Exec down — treat direct psql as likely BLOCKED; use API + index + source as the evidence path and tag DB-ground-truth items UNKNOWN-BLOCKED honestly.
**RAMZI_V2:** read-only; no integration. Expected HEADs (verify in preflight, STOP if drifted): web `8b0a815f` · api `8d55a20` · bo `07b3a84` · mcp `d8efb4a`.
**REQUIRED KIRO SKILLS:** `tawadoo-source-truth` (mandatory — this is a source-truth reconstruction), `tawadoo-runtime-provider-qa` (browser/HTTP/index probes). Discover + activate + record.

## MANDATORY FIVE-LINE OPENER (print before any tool call)
```
BRANCH: <actual per repo or UNVERIFIED>
MISSION: S-CTO-25-NAV-MAP — determine whether Tawadoo V2 has a SYSTEMIC entity-addressing/navigation-integrity defect; produce the map, failure matrix, root-cause patterns, canonical target model, migration surface, app/MCP compat notes, and the Stage-2 recommendation. READ-ONLY.
ORDER: reality check → trace ~10 entities end-to-end → addressing matrix → classify every "unavailable" (STATE 1–12) → root-cause PATTERNS → canonical target model → migration surface → app/MCP contract inventory → Stage-2 recommendation → evidence
BLOCKERS: DB in-VPC/IP-locked + ECS-Exec down (direct psql likely BLOCKED — use API+index+source, tag DB-ground-truth UNKNOWN)
TREE STATE: <actual per repo or UNVERIFIED; preserve all work; this session writes only the evidence file>
```

## MISSION (the ONE question Stage 1 must answer)
> **What is the AUTHORITATIVE identity of each entity, and how should every surface deterministically turn that identity into a destination and resolve it back to the SAME entity?**
Do NOT assume the current model is correct. Specifically determine whether the "item no longer available" class is caused by **bad addressing/resolution** (wrong id/slug/type/route/env/stale-projection/authz) rather than TRUE entity unavailability. The goal: make the next (Stage 2) implementation session OBVIOUS.

## AUTHORITY / STEERING TO READ FIRST
`TAWADOO V2 — CTO STEERING: SYSTEMIC ENTITY & NAVIGATION INTEGRITY REFACTOR` (the founder directive that spawned this unit — its 38 sections are the spec; treat as controlling). Then `HANDOFF_B22_CTO_2026_09_06.md`, `TAWADOO_V2_CTO_OPERATING_SYSTEM.md` (laws + FR register + M-ZONE moderation/index/visibility chain), `TAWADOO_BUSINESS_TRUTH_2026_09_06.md` (moderation/human-in-the-loop, jobs/services, distribution, lead-gen), `TAWADOO_V2_TARGET_ARCHITECTURE.md` (5-layer model + Systems-of-Record vs Projections), `BRAIN_B16_MASTERY_2026_08_31.md` (refactor design). Steering always-on: non-regression law, B14 standard, refactor program.

## HARD ARCHITECTURAL CONSTRAINT (founder-added, binding)
**DO NOT DESIGN THE CANONICAL CONTRACT FROM THE WEB IMPLEMENTATION ALONE.** Derive authoritative identity from the SYSTEM OF RECORD + existing contracts, tracing for every major entity:
`DB identity → Business Core identity → API identity → projection/search identity → public URL identity → notification identity → share identity → app identity → MCP identity`
The DB remains the System of Record; projections (OpenSearch/lake/cache) are rebuildable and are NEVER the authority for whether a commerce entity exists (steering §9/§10). App+MCP are inventoried (not fixed) precisely so the canonical V2 model does NOT accidentally break their live contracts.

## ENTITIES TO TRACE (representative set — steering §3)
1. normal published listing · 2. listing reached FROM search · 3. listing FROM image-search · 4. listing FROM category/feed · 5. seller's OWN listing · 6. a lead · 7. a notification target · 8. a share target · 9. a saved/favorited target · 10. an auction target. (Add seller-profile as an 11th if it surfaces as a distinct addressing case.)
For EACH, capture (steering §3): identity (DB PK / public id / UUID / slug / route id / entity type / owner rel) · backend (source entity, endpoint, DTO, serializer, resolver, visibility rule, authz rule) · search/projection (indexed id, indexed slug, index name, stale behavior) · frontend (card link, internal link, route builder, router destination, detail resolver, client state, fallback) · sharing (canonical/OG/mobile/app URL) · notification (type, target entity, target id, route, payload, resolver) · app (route/deeplink/API/auth) · MCP (reference, returned URL, tool output, public contract).

## ROOT-CAUSE PATTERN HUNT (steering §5 — find the PATTERN, not the first instance)
Grep across repos for: hard-coded route strings · duplicated route/slug/id builders · `id` vs `entityId` vs `objectId` vs `serviceId` · numeric-id vs UUID mismatch · slug-vs-id mismatch · entity-type missing from references · inconsistent route/endpoint names · notification payloads with incomplete targets · share URLs built independently in multiple places · links built from SEARCH DOCUMENTS instead of canonical DB identity · references to projected entities without canonical resolution · generic "unavailable" fallbacks hiding real failures · env-specific/relative-vs-absolute URL inconsistency · web/app route divergence · visibility checks at different layers · authz producing misleading not-found · deleted/unpublished still in projections · published absent from projections · handlers that understand only some entity types. Verify EVERY name from source (file:line) — §49, extra-careful on renames (§27); a name from a report/doc is a hypothesis until grep-confirmed.

## THE CRITICAL DISTINCTION (steering §6 — classify every "unavailable")
For each observed "item unavailable"/deadlink, assign the true state: (1) genuinely doesn't exist · (2) exists, intentionally unavailable · (3) exists, visibility state wrong · (4) exists, projection/index stale · (5) wrong id · (6) wrong entity type · (7) wrong slug · (8) wrong route · (9) wrong environment · (10) authz mismatch · (11) cache/client-state · (12) unknown. Never collapse these into one generic state; the map must name the true reason. Known anchors to fold in as evidence (from prior sessions, verify live): FR-9 notif deeplink dead-ends `/notification-unavailable` (api omits service/objectId + web handler has no lead branch) · the moderation/index two-gate (`isModerated`=index-eligibility, `isVerified`=search-visibility; `isIndexEligible` in `index-eligibility.ts`) · vector-leg has no isVerified filter (D2) · "view listing" 404 on pending moderation (create-publish chain). Do NOT assume these all share one root cause — evidence decides (steering §19/§27).

## STEP ORDER
1. Preflight: verify HEADs; corrected 5-line opener; confirm the entity list + anchors from source.
2. Trace the 10 entities end-to-end (source + live staging read: real search→click→detail, real share URL open in a FRESH browser session, a real notification payload, a category/feed→detail). Browser desktop+mobile where a destination is user-facing.
3. Build the **Entity Addressing Matrix** (artifact 1) — populated ONLY from actual source/runtime, no invented values.
4. Build the **Failure Matrix** (artifact 2) — every failure → true STATE (1–12) + file:line evidence.
5. Derive **Root-Cause Patterns** (artifact 3) — which failures share a cause, which don't; each pattern named with its evidence.
6. Define the **Canonical Target Model** (artifact 4) — identity → destination → resolution → visibility → authorization → projection-staleness handling → deeplink → share → app → MCP. Simple, boring, explicit. NO "UniversalEntityRouter", no framework (steering §18).
7. Enumerate the **Migration Surface** (artifact 5) — every caller (cards, search, category, feed, seller profile, favorites, notifications, saved searches, messages, leads, share, copy-link, OG, email, push) that must conform in Stage 2.
8. **App/MCP Compatibility Notes** (artifact 6) — inventory existing contracts the canonical model MUST preserve or deliberately version. No implementation.
9. **Architectural Decision** (artifact 7) — exactly WHAT to build in Stage 2, WHERE it should live (which layer/repo), and WHY. Include what it explicitly does NOT include.

## NON-GOALS (steering §26 — this session is NOT a license to)
patch individual links · create a route framework · modify notification/share/route logic · modify app or MCP · rewrite Classic · merge Smart+Classic · redesign DB/design-system · reopen the queued FR wave (S-CTO-26 pink/save-search, chat, cards, layout) · broad cleanup · weaken any authorization to make a link work (steering §24 — identity→authz→visibility→resolution; never expose a private entity because a public link carries its id; never leak private ids).

## SECURITY / SAFETY
Read-only. Staging only, prod protected. No secrets in output/logs/evidence. No destructive action. Any synthetic staging data created for a probe is founder's dev env (not a cleanup blocker) but still record what you created.

## REQUIRED FINAL OUTPUT (exactly these 7 artifacts, in the evidence file — steering §38 condensed)
1. **ENTITY ADDRESSING MATRIX** (per entity: authoritative DB identity · public identity · API identity · search/index identity · route · resolver · visibility source · authz source · share destination · notification target · app contract · MCP contract · current status).
2. **FAILURE MATRIX** (every observed navigation/deeplink/unavailable failure → true root cause STATE 1–12 + evidence).
3. **ROOT-CAUSE PATTERNS** (shared vs distinct; no forced unification).
4. **CANONICAL TARGET MODEL** (identity→destination→resolution→visibility→authorization, + projection-staleness, deeplink, share, app, MCP).
5. **MIGRATION SURFACE** (every caller for Stage 2).
6. **APP/MCP COMPATIBILITY NOTES** (contracts to preserve/version — no code).
7. **ARCHITECTURAL DECISION** — WHAT to build in Stage 2, WHERE it lives, WHY; ranked RED/YELLOW/BLUE + R0/R1/R2/R3; and the honest UNKNOWN-BLOCKED list (esp. DB-ground-truth). End so the Stage-2 prompt is obvious.
Plus: executive result (≤20 lines), the two five-line openers, per-layer proven-vs-not, git/tree provenance, and own-session status only (propose FINISHED — COMPLETE for the MAP; independent Brain QA per §18 owes acceptance).

## STOP CONDITIONS
Any temptation to "just fix" a link → STOP (this is Stage 1). Any DB-ground-truth needed but unreachable → tag UNKNOWN-BLOCKED, keep going. Any finding that a canonical change would break a live app/MCP contract → record it in artifact 6, do NOT design around it silently. If the evidence shows there is NO systemic pattern (the failures are genuinely independent) → say so plainly; do not manufacture a pattern.

## COMPLETION CHECKLIST (map 1:1 in evidence)
- [ ] Preflight HEADs verified; anchors + entity list source-confirmed
- [ ] 10 entities traced end-to-end (source + live staging + fresh-browser share test)
- [ ] Artifact 1 Addressing Matrix (source-only values)
- [ ] Artifact 2 Failure Matrix (every failure → STATE 1–12 + file:line)
- [ ] Artifact 3 Root-cause patterns (shared vs distinct, evidence-decided)
- [ ] Artifact 4 Canonical target model (system-of-record-derived, simple, no framework)
- [ ] Artifact 5 Migration surface (all callers)
- [ ] Artifact 6 App/MCP compat notes (contracts to preserve/version)
- [ ] Artifact 7 Stage-2 architectural decision (what/where/why + classification + UNKNOWN-BLOCKED)
- [ ] Zero product code changed; only evidence file written; own-session status only
