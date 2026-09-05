# AD-001 — CONTROLLED SCHEMA OWNERSHIP + MIGRATION/RUNTIME PRIVILEGE SEPARATION

**Detailed step plan — DESIGN ONLY. Not authorized to build.**
**Date:** 2026-09-01 · Brain B16 · Governing: `TAWADOO_V2_ARCHITECTURAL_SYNTHESIS_MANDATE_2026_09_01.md`
**Status:** DIRECTION approved by founder. Implementation NOT approved. Staging clean, nothing in flight.
**Grounded in verified source (2026-09-01), not memory.**

---

## 1. THE PROBLEM (plain English)
Every time the app starts, it also tries to *change the shape of the database* (add/alter tables). It runs that with the ordinary "run-the-app" key, which is not allowed to change tables. So the moment any table change is pending, the whole app crashes on boot. This caused the S138 incident. **Until this is fixed, every future improvement that needs a database change is a landmine.**

## 2. VERIFIED CURRENT STATE (from code + AWS, today)
- `tawadoo_api_js/src/app.module.ts:120` → `synchronize: false` (good — no silent auto-changes).
- `app.module.ts:121` → `migrationsRun: true` — app runs migrations at boot using the **runtime** user (`DB_USERNAME`, line 115), which is not the table owner → DDL crashes. **This is the crash cause.**
- `src/datasource.ts` ALREADY EXISTS — a standalone migrator runner: `migrationsRun:false`, documented "scratch/staging — NEVER prod." **The migrator-key concept is already scaffolded.**
- 49 migrations in `src/migrations/`, tracked in table `api_migrations`.
- **CI** (`.github/workflows/deploy.yml`): runs a migration *string-scan test* only; does NOT run migrations against a real DB. Migrations run at app boot. CI identity `github-actions-deploy` is deliberately limited (cannot register task-defs).
- **Prod** (`main`) avoids the crash with `synchronize: true` (auto-sync). Confirmed UNSAFE as a target (can silently drop columns). NOT the model.
- Prior B11 work (`B11-SEC-DB-CREDENTIAL-C1`) STARTED runtime/migrator separation locally but never finished (no prod run, no CI, no staging rollout, no acceptance). Some services (Leads/consent/entity) still attempt DDL at startup and only WARN. **This step completes that.**

## 3. THE DESIGN — three roles, clearly separated (verified best practice, 2026)
| Role | Can do | Cannot do | Used by |
|---|---|---|---|
| **Owner** (owns tables/sequences/indexes) | owns schema objects | NOT a login/app credential; not used to serve traffic | nobody at runtime — only the basis for migrator grants |
| **Migrator** (controlled, time-scoped) | DDL (ALTER/CREATE), write `api_migrations` | serve app traffic | the deliberate migration step only |
| **Runtime** (the app boots with this) | DML (SELECT/INSERT/UPDATE/DELETE) | NO DDL, no `api_migrations` write | the running app |

Plus `ALTER DEFAULT PRIVILEGES` so **future** tables the migrator creates automatically grant the runtime role the right read/write access (no manual grant per table).

## 4. THE PIECES (what a future build session designs in detail — NOT build now)
1. **App boot** → `app.module.ts` set `migrationsRun: false`. App stops changing tables at startup. (One-line intent; everything around it is the real work.)
2. **Postgres roles** → create owner + migrator + runtime with exact grants; `ALTER DEFAULT PRIVILEGES`. Design the exact SQL; verify against the live schema first.
3. **Ownership lifecycle (the founder's added rigor):** who owns tables, sequences, indexes; how existing objects get re-owned to the owner role; how extensions are handled; how grants are preserved on future migrations.
4. **Migration execution path** → migrations run via `datasource.ts` using the migrator credential, as an explicit bounded step (who triggers it, where the credential lives, how it's audited). NOT app boot.
5. **Migrator credential** → stored as a secret reference (Secrets Manager), **bounded/time-scoped, not a standing superuser** (2026 warning on static broad CI/agent DB creds). Never printed.
6. **Startup-DDL cleanup** → move the B11 startup-DDL (Leads/consent/entity WARNs) into proper migrations so runtime never needs DDL at all. Verify each from source before moving.
7. **CI gate** → CI runs migrations against a production-like scratch schema with the migrator role, proving they apply cleanly BEFORE deploy (today CI only string-scans).
8. **Partial-failure handling** → what happens if a migration half-applies (transaction wrapping, resumability, forward-fix vs down).
9. **Rollback contract** → every migration reversible or safe forward-fix; migrator role revocable; prior image/task retained; exercised on staging.

## 5. WHY THIS IS LOW-RISK
- Completes a half-built, already-scaffolded piece (`datasource.ts` exists, B11 started it) — not greenfield.
- Touches boot behavior + DB privileges only; does NOT touch the business Core, money, or the Face.
- Staging-first, reversible.
- Unblocks EVERYTHING else (seoTitle column, image backfill, feed-safety re-land, all future schema work).

## 6. INVARIANTS IT MUST NOT BREAK (target-architecture §3)
Classic View, login, money locks, payments, orders, sovereignty pipeline — byte-for-byte unaffected. This step only changes *who is allowed to alter tables and when*, not any business behavior.

## 7. NOT DECIDED HERE (founder gates before build)
- Exact migrator credential storage/rotation + how the bounded step is triggered (CI job vs one-off task vs guarded boot phase) — cheapest-safe option chosen with evidence in the secure-release pass.
- Any production change — prod stays untouched until a separate, explicitly-approved cutover.

## 8. CLASSIFICATION (per synthesis mandate)
- **FACT:** crash cause, current settings, datasource.ts exists, CI behavior, prod synchronize:true — all source/AWS verified.
- **DECISION (direction approved):** three-role separation, migrationsRun:false, bounded migrator.
- **FOUNDER DECISION (before build):** credential mechanism, trigger location, production cutover.
- **UNKNOWN (verify at build time):** exact current DB roles/grants on the live cluster (must inspect before writing role SQL — §49).

## 9. HARD STOP
This is the DESIGN for step one. Nothing authorized to build. Next, on founder go: convert this into a real build prompt (fail-first tests, staging rollout, exercised rollback, evidence) per the execution law — one careful, reversible step, staging only.
