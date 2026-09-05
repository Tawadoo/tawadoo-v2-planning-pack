# SESSION ID S-CTO-19api — ADD `app_install_banner_dismissed` TO ALLOWLIST

**Status proposal:** FINISHED — COMPLETE (independent QA to accept)
**Date:** 2026-09-05 (session window; prompt dated 2026-09-06)
**Repo (writable):** `tawadoo_api_js` ONLY · Read-only: web/bo/mcp
**Branch:** `Ramzi_V2` · **Commit:** `50cfa12814a42438849498011e8c8ee65f28065b`
**Environment:** STAGING only · prod NOT touched
**Skills:** tawadoo-source-truth · tawadoo-data-sovereignty

---

## EXECUTIVE VERDICT
The single canonical event `app_install_banner_dismissed` is now an allowlist entry in the
Banners (app-install) group, alongside `app_install_banner_shown` / `app_install_banner_clicked`.
Fail-first proven red→green. Build/test/lint/typecheck green. Deployed to staging; the running
task carries the exact image digest built from this commit. **Live DB proof:** an emitted
`app_install_banner_dismissed` event persisted to `ta_analytics_event` with **no `_is_canonical`
flag** — i.e. tagged CANONICAL. This unblocks the future web rename
`app_banner_dismissed → app_install_banner_dismissed`.

### Evidence ladder (highest state proven)
`documented → source → local → CI → deployed → LIVE (DB canonical)` — **LIVE reached.**

---

## FIVE-LINE OPENER (as run)
- SESSION: S-CTO-19api — ADD `app_install_banner_dismissed` to allowlist
- BRANCH: `Ramzi_V2` (verified; 0/0 vs origin/Ramzi_V2 at start; clean tree)
- MISSION: add the single canonical event `app_install_banner_dismissed`. No other change.
- ORDER: 360 (confirm absent + domain) → add → fail-first → build/test → deploy → verify canonical → evidence
- TREE STATE: clean at start; only the two owned files changed.

---

## 1. MANDATORY 360 (source-verified)

**File:** `tawadoo_api_js/src/modules/analytics-ingestion/constants/allowed-events.ts`

- **Absent-confirmation:** collision search for
  `app_install_banner_dismissed|app_banner_dismissed|app_banner|install_banner`
  across `tawadoo_api_js/**/*.ts` returned ONLY the two existing app-install entries
  (`app_install_banner_shown`, `app_install_banner_clicked`) + generic `banner_click`/`banner_view`.
  - `app_install_banner_dismissed` was **NOT present** under any spelling.
  - `app_banner_dismissed` (the current web name) is **NOT in this API allowlist** either.
- **Correct domain group (cited):** the block
  `// ─── Banners (generic + app-install) ───` in `allowed-events.ts`, which contains:
  - `'banner_click'`
  - `'banner_view'`
  - `'app_install_banner_shown'`
  - `'app_install_banner_clicked'`
  The dismissed event belongs immediately after `app_install_banner_clicked` (its siblings).
- **Validation semantics (source, `analytics-ingestion.service.ts`):** events NOT in
  `ALLOWED_EVENTS` are persisted with `properties._is_canonical=false` AND surfaced via
  `observability.recordIngestionUnknown(n)` (S-CTO-15 canonical-tag, not a hard gate, §50).
  Events IN the set get NO flag → CANONICAL.

## 2. THE ADD (exact diff)
```
  'app_install_banner_shown',
  'app_install_banner_clicked',
+ 'app_install_banner_dismissed', // S-CTO-19api — unblocks web rename app_banner_dismissed → app_install_banner_dismissed
```
One additive line in the correct domain. No other event added/renamed. No duplicate.

## 3. COLLISION SEARCH
`grep app_install_banner_dismissed|app_banner_dismissed|app_banner|install_banner` (repo-wide,
`tawadoo_api_js/**/*.ts`) → no pre-existing `app_install_banner_dismissed` anywhere. Recorded above.

## 4. FAIL-FIRST (red → green)
New test in `analytics-ingestion.service.spec.ts` (S-CTO-15 canonical-tagging describe block):
`'S-CTO-19api: app_install_banner_dismissed → canonical (unblocks web rename)'`.

- **RED (before the add):** `expect(insert.properties._is_canonical).toBeUndefined()` failed —
  `Received: false` (event was non-canonical/surfaced). 1 failed / 25 skipped.
- **GREEN (after the add):** test passes. Full suite: **26 passed / 26 total.**

## 5. LOCAL VALIDATION (all green)
- Targeted: `jest analytics-ingestion.service.spec.ts -t "S-CTO-19api"` → pass (red→green proven)
- Full suite file: `jest analytics-ingestion.service.spec.ts` → **26/26 pass**
- Lint (owned files): `eslint allowed-events.ts analytics-ingestion.service.spec.ts` → exit 0
- Typecheck: `tsc --noEmit -p tsconfig.json` → exit 0
- Build: `yarn build` (`nest build`) → Done, exit 0
(Pre-existing unrelated suites not run/owned by this session.)

## 6. COMMIT / CI
- Commit `50cfa12` on `Ramzi_V2` (one concern), pushed `bd5cf2a..50cfa12`.
- CI run `33977662385` (Build and Push Back-End to GHCR) → **success**.
  Jobs: quality-gate (Run tests ✓, Build verification ✓) + build-and-push
  (Build/Tag/Push ✓, Record immutable provenance ✓, Deploy staging ECS and verify rollout ✓).

## 7. IMMUTABLE PROVENANCE (commit → digest → running task)
- Image tags built: `sha-50cfa12814a42438849498011e8c8ee65f28065b`, `0.1.485`, `staging-v2`
- **All three resolve to digest** `sha256:527a29ad2a195260d6c849b2944680a5eef8cf5f0a68fc5cb99bbd8a62273cf5`
- ECS service `tw-staging-svc-back` (cluster `tw-staging-cluster`, eu-west-1):
  status ACTIVE, 2/2 running, single PRIMARY deployment `rolloutState=COMPLETED`, task-def `:44`.
- **Running task container image digest** = `sha256:527a29ad2a19...cf5` (EXACT match to the commit build),
  `lastStatus=RUNNING`. (Container health `UNKNOWN` = no container-level healthcheck defined;
  ALB target health + ECS rollout COMPLETED govern stability.)

## 8. LIVE CANONICAL VERIFICATION (DB source-of-truth, not HTTP 200)
- Emitted to staging: `POST https://api-staging.tawadoo.ma/api/analytics/events`
  `{event_type: app_install_banner_dismissed, event_id: scto19api-live-1788626103, properties:{probe:"S-CTO-19api"}}`
  → HTTP 200 `{"status":"ok","ingested":1,"duplicates_skipped":0}`.
- **DB read via bounded one-off ECS task** (task-def `tw-staging-task-back:44`, same image/subnets/SG/secrets,
  runtime role SELECT-only, self-terminated exit 0):
  ```sql
  SELECT event_type, properties, created_at FROM ta_analytics_event
  WHERE event_type='app_install_banner_dismissed' ORDER BY created_at DESC LIMIT 3;
  ```
  Result: `[{"event_type":"app_install_banner_dismissed","properties":{"probe":"S-CTO-19api"},"created_at":"2026-09-05T16:35:04.295Z"}]`
- **`properties` has NO `_is_canonical` flag → the event is tagged CANONICAL live.** Proof complete.

## 9. TOOLS / PERMISSIONS / COST
- No novel paid tool/service provisioned. Used existing `ecs:RunTask` capability (same pattern as S141
  migration one-off). One-off Fargate task ran ~seconds and self-terminated — negligible/no standing cost.
- No IAM change. No secret exposed (DB creds injected via existing task-def `secrets`; never printed).

## 10. ROLLBACK
- Source revert: `git revert 50cfa12` (removes the one allowlist line + the test) — reversible, additive change.
- No migration, no schema change, no data mutation. Nothing to roll back beyond the source line.
- Rollback trigger would be: unexpected drift or duplicate — none observed.

## 11. TRAFFIC-LIGHT + RISK
- **RED:** none.
- **YELLOW:** container health probe is `UNKNOWN` (no container-level healthcheck on task-def `:44`) —
  stability instead evidenced by ECS rollout `COMPLETED` + 2/2 running. Pre-existing, not introduced here.
- **BLUE (info):** the web still emits `app_banner_dismissed`; the API now accepts the canonical target.
- **R0/R1/R2/R3:** R0 (source add) ✓ · R1 (local red→green/build) ✓ · R2 (CI + deployed digest) ✓ ·
  R3 (live DB canonical) ✓.

## 12. UNBLOCKED / NEXT
- **Unblocked:** the web rename `app_banner_dismissed → app_install_banner_dismissed` can now land as
  CANONICAL (target name is a canonical allowlist entry). That is a future **web** session (writable
  `tawadoo_web_js`), not this one.
- **Next session (recommended):** `S-CTO-20web` — rename the web emitter
  `app_banner_dismissed → app_install_banner_dismissed` and browser-verify (Chromium+WebKit) the banner
  dismiss still fires, now landing canonical. Reason: closes the drift the API side just enabled.

---
*Own-session status mutation only. Durable accepted status is set by independent Brain/QA per §18/§19.*
