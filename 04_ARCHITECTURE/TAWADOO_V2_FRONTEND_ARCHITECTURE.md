# TAWADOO V2 — FRONTEND (FACE LAYER) ARCHITECTURE

**Design on paper. DESIGN ONLY, nothing built. Staging clean.**
**Date:** 2026-09-01 · Part of `TAWADOO_V2_TARGET_ARCHITECTURE.md` (Face layer, in depth).
**Grounded in verified source (`tawadoo_web_js`, 2026-09-01), not memory.**

---

## 1. VERIFIED CURRENT STATE (from code)
- Next.js 14 (App Router), React 18, Tailwind + Zustand, next-intl (FR/AR/EN).
- Structure: `src/{app, components, sections, stores, hooks, providers, layouts, services, lib, styles, utils, context, config, constants, types}`.
- ~413 component files, 13 sections, 7 Zustand stores (auth-gate, chat, notifications, entity-switch, roles, pending-action, publication-form).
- Design system STARTED: `src/styles/tw-tokens.css`, `src/components/ui/TwButton.tsx` (+ test), `one-design-system` spec in progress (replacing 402 hardcoded hex → semantic tokens, CI hex-lint).
- Verified prior: NOT broken, NOT blocking. It works. This is finish + organize, not rebuild.

## 2. THE SHAPE — two views, one shared engine
Classic View and Smart View are two windows onto the SAME data and SAME API calls. Shared building blocks live in one place; both views consume them. No duplication between Classic and Smart (the dropped Smart-View auth-gate was caused by duplication — the design fixes it by sharing). Smart View adds only: conversational input, AI guidance, action cards, voice — it reads the same Core.

## 3. FOUR LAYERS INSIDE THE FRONTEND
1. **Design system (the look):** tokens + primitives (colors, spacing, type, `TwButton`, cards). Finish the in-progress `one-design-system`; all visuals come from tokens — no scattered hardcoded colors. Enforced by CI hex-lint.
2. **Shared components (building blocks):** listing card, search bar, image uploader, auth gate, confirmation card, etc. Built once, used by Classic + Smart + dashboard.
3. **Pages/sections (screens):** home, search, product, dashboard, auction, live, blog, etc. They ASSEMBLE building blocks; they never reinvent them.
4. **Data/state (memory):** keep the 7 Zustand stores small and single-purpose (one store per job). No giant global store.

## 4. FRONTEND TALKS ONLY TO THE CORE
The frontend calls Tawadoo's own API only — never Google/Meta/AI directly. The API owns the outside world. Frontend stays simple and cannot be broken by a third party.

## 5. PERFORMANCE IS A STANDARD, NOT AN AFTERTHOUGHT
Fold prod's perf work (lazy-load heavy/interactive components, image optimization, defer non-critical scripts) into the standard and guard it against silent regression. (Prod audit: 38 web commits drifted, many perf/banner/analytics — reconcile, don't lose.)

## 6. FRONTEND "NEVER BREAK" RULES (invariants)
1. Classic View keeps working, always.
2. Every visible string stays translated in FR/AR/EN (keep existing locale lint gates).
3. Shared building blocks are SHARED, never forked between Classic and Smart.
4. Dropdowns/popups/modals always render above other content (permanent z-index guard — recurring bug class).
5. Pages stay fast — perf cannot silently regress.
6. Frontend only calls the Core, never external providers directly.

## 7. WHAT THIS IS / IS NOT
- IS: finish + enforce the started design system, consolidate shared components, formalize Classic/Smart sharing, lock perf + i18n + z-index guards, reconcile the 38-commit web drift.
- IS NOT: a frontend rewrite, a framework change, or a from-scratch design system.

## 8. CLASSIFICATION
- **FACT (source-verified):** stack, structure, 413 components / 13 sections / 7 stores, design-system in progress (tw-tokens.css + TwButton + one-design-system spec).
- **DECISION (direction, design-only):** four internal layers, share-don't-duplicate, tokens-only visuals, perf as standard.
- **UNKNOWN (verify at build):** exact scope of the 38-commit web drift classification; which shared components still have Classic/Smart forks.

## 9. HARD STOP
Design complete on paper (back foundation AD-001 + 5-layer shape + this frontend piece). Nothing built. Next, on founder go: sequence bounded build units (AD-001 foundation first, then frontend units) — each staging-first, reversible, with verification + rollback, per the execution law.
