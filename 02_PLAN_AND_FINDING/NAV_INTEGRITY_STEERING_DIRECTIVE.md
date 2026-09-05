# TAWADOO V2 — CTO STEERING: SYSTEMIC ENTITY & NAVIGATION INTEGRITY REFACTOR
> This is the founder directive (pasted into the Kiro session on 2026-09-06) that spawned the S-CTO-25-NAV-MAP audit. It is the spec the audit answered. Reproduced here so ChatGPT has the original intent alongside the finding.

## ROLE
Principal Architect + CTO + Refactor Lead + Release Commander for Tawadoo V2. The founder identified a potentially systemic product/architecture defect: widespread entity-linking, navigation, deep-link, sharing and cross-surface wiring failures (item "no longer available", internal links not reaching the intended entity, broken sharing, shared URLs not reopening the correct item, notification deep-links not reaching their target, notification payloads lacking enough info to identify the destination, an object existing but its route/API/index/visibility disagreeing about whether it exists). NOT to be treated as isolated frontend bugs — treat as a cross-system architectural integrity problem.

## MISSION
Do NOT immediately patch known symptoms. First determine whether Tawadoo has a systemic problem in how entities are created → identified → stored → exposed → indexed → linked → routed → resolved → displayed → shared → reopened → deep-linked → authorized → declared available/unavailable. Establish a single coherent, reliable entity-addressing and navigation model across V2. Objective: ANY USER-VISIBLE ENTITY REFERENCE MUST RESOLVE CORRECTLY OR FAIL WITH A TRUE, EXPLAINABLE STATE.

## KEY RULES (condensed from the 38-section directive)
- Classification A (refactor/architecture) primary; FR findings are evidence, not the roadmap. PATTERN OVER INSTANCE — fix the root once; do not invent one abstraction for genuinely different causes.
- FIRST PRINCIPLE: MAP THE ENTITY before changing code. Trace ~10 representative entities end-to-end.
- Build a real Entity Addressing Matrix (source-only values, no invented data).
- Distinguish 12 true states for every "unavailable" (genuinely absent / intentionally unavailable / visibility wrong / projection stale / wrong id / wrong entity type / wrong slug / wrong route / wrong environment / authz mismatch / cache-client-state / unknown). Never collapse them into one generic message.
- DB is the System of Record; projections (OpenSearch/lake/cache) are rebuildable, NEVER the authority for whether a commerce entity exists.
- Canonical: identity → destination → resolution → visibility → authorization. One deterministic way to construct and resolve every entity's destination.
- Do NOT build a giant "navigation framework" / "UniversalEntityRouter". Smallest correct mechanism + migrate callers. Simple, boring, explicit code.
- Never weaken authorization to make links work. Never expose a private entity because a public link carries its id. Never leak private ids.
- Classic sacred; Classic + Smart stay separate implementations (FACE-001), shared design foundation only; never import Smart into Classic.
- Staging only; prod protected. Browser QA mandatory for UI. No panic reverts. Regression tests for every root cause fixed.
- Scope for this program: navigation/routing/entity-reference contracts/deep-links/share-links/notification routing/relevant API DTOs+resolvers/relevant search-index resolution/relevant app routing/relevant MCP entity URLs. NOT a license to redesign the whole frontend, rewrite Classic, merge Smart+Classic, redesign the DB, create microservices, or reopen deferred work.

## REQUIRED DELIVERABLES (the 7 artifacts the audit returned)
1. Entity Addressing Matrix · 2. Failure Matrix (every failure → true root cause) · 3. Root-Cause Patterns (shared vs distinct) · 4. Canonical Target Model · 5. Migration Surface · 6. App/MCP Compatibility Notes · 7. Architectural Decision (what to build in Stage 2, where, why).

## SCOPE APPROVED FOR STAGE 1 (founder, 2026-09-06)
Scope A: Web + API + Search/Index + BO + DB = full investigation. App + MCP = read-only contract inventory only (no implementation). Canonical identity derived from the SYSTEM OF RECORD (DB→Core→API→projection→URL→notif→share→app→MCP), NOT from the web implementation. Stage 1 read-only. Stage 2 (NAV-FIX) builds the mechanism after founder approves the map.
