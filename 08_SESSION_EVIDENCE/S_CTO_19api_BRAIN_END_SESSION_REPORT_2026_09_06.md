# S-CTO-19api — END-OF-SESSION REPORT FOR THE BRAIN

**For:** Brain (state + queue). **From:** execution session S-CTO-19api.
**Date:** 2026-09-05 (prompt dated 2026-09-06). **Repo:** `tawadoo_api_js` @ `Ramzi_V2` `50cfa12`.
**Proposed status:** FINISHED — COMPLETE (independent QA to accept). **Evidence file:** `S_CTO_19api_EVIDENCE_2026_09_06.md`.

This report exists to make sure **nothing is buried**. It separates: (1) what was actually built and proven,
(2) what is only true on the back end and NOT on the front / elsewhere, (3) every open/incomplete/odd thing I
hit — including outside my scope — for the Brain to queue.

---

## 1. WHAT WAS ACTUALLY BUILT + PROVEN (in scope)

- **Back end (API):** added one canonical event `app_install_banner_dismissed` to the allowlist
  `tawadoo_api_js/src/modules/analytics-ingestion/constants/allowed-events.ts`, in the
  "Banners (generic + app-install)" group, next to `app_install_banner_shown` / `app_install_banner_clicked`.
- **Test:** fail-first regression test added to `analytics-ingestion.service.spec.ts` (red before, green after).
- **Verified end-to-end on STAGING:**
  - Local: 26/26 suite, lint 0, `tsc --noEmit` 0, `nest build` 0.
  - CI run `33977662385` success; commit→digest `sha256:527a29ad2a19…cf5`; **running ECS task carries that exact digest**; rollout COMPLETED, 2/2 tasks.
  - **Live DB truth:** emitted the event to `api-staging.tawadoo.ma`; read back `ta_analytics_event` via a bounded one-off ECS task — persisted row has **no `_is_canonical` flag** → tagged CANONICAL. (HTTP 200 alone was NOT treated as proof.)

**Net effect:** the API now accepts `app_install_banner_dismissed` as canonical. The future web rename is unblocked.

---

## 2. BUILT IN BACK, NOT (YET) IN FRONT / ELSEWHERE — the gap, explicit

- **FRONT (web):** the web still emits the OLD name. `tawadoo_web_js/src/components/OpenInAppBanner.tsx:231`
  fires `trackEvent('app_banner_dismissed', …)`. **Nothing in web emits `app_install_banner_dismissed` yet.**
  → Until a web session renames it, the canonical allowlist entry I added has **no producer**. The dismiss
  event currently still lands as NON-canonical (`app_banner_dismissed`, `_is_canonical=false`, surfaced as drift).
  This is expected (the prompt scoped web out) but must not be read as "the dismiss event is now canonical in production behavior" — it is not, until web ships.
- **DB:** the only `app_install_banner_dismissed` rows in `ta_analytics_event` right now are **my probe row(s)**
  (`event_id=scto19api-live-…`, `properties.probe="S-CTO-19api"`). No real user traffic uses this name yet.
  → Minor housekeeping: these are synthetic probe rows in staging analytics. Harmless, but noted so they aren't mistaken for organic data.
- **BO (admin_bo_tawadoo):** zero references to either name (grep confirmed). No BO work needed or done.
- **MCP (-tawadoo-mcp-):** zero references (grep confirmed). Out of scope and untouched, as required.
- **AWS:** no infra change. One bounded one-off Fargate task (read-only SELECT via existing runtime role) ran and self-terminated. No new standing resource, no IAM change, no secret exposed.

---

## 3. OPEN / INCOMPLETE / ISSUES TO QUEUE (nothing buried)

### 3.1 (Primary follow-up) Web rename is required to realize the value — NOT done here
- The whole point of this add is a downstream web rename `app_banner_dismissed → app_install_banner_dismissed`.
- **Recommended next session: `S-CTO-20web`** (writable `tawadoo_web_js`): rename the emitter at
  `OpenInAppBanner.tsx:231`, browser-verify (Chromium + WebKit) the dismiss still fires, and confirm it lands
  canonical via the same DB read path. Also remove/adjust the `// P0: app_banner_dismissed` comment.

### 3.2 (NEW FINDING — surfaced this session, outside my scope) Two allowlist entries have NO web emitter
- **`app_install_banner_shown` and `app_install_banner_clicked` are in the API allowlist but have ZERO emitters in web** (grep across `tawadoo_web_js/**/*.{ts,tsx}` → no matches).
- What web actually emits from the banner (`OpenInAppBanner.tsx` + `utils/analytics.ts`):
  - `banner_view` (on show)
  - `banner_click` (via `trackBannerClick`, with `action: "open" | "dismiss"`)
  - `app_banner_dismissed` (on dismiss)
- So the canonical `app_install_banner_shown` / `app_install_banner_clicked` names may be **dead allowlist entries** (declared, never produced) — a taxonomy/reality mismatch. Meanwhile the banner's real "shown"/"click" signals ride on the generic `banner_view` / `banner_click`.
- **Queue item for Brain (investigate, don't implement):** decide the intended taxonomy for the app-install banner — either (a) web should emit `app_install_banner_shown`/`clicked` and retire the generic `banner_view`/`banner_click` for this banner, or (b) the two unused canonical names should be removed from the allowlist. This is a **founder/Brain taxonomy decision** (§52.1 — business/policy naming), not a silent code choice. I did not touch it.

### 3.3 (Pre-existing, not introduced) Staging task-def has no container-level health check
- Task-def `tw-staging-task-back:44` container health = `UNKNOWN` (no `healthCheck` block). Stability is only
  provable via ECS rollout state + running count + ALB target health, not a container probe.
- Not mine to fix; flagged for the pre-production cutover checklist (aligns with §29 defer-polish). Queue as a
  hardening item, not a blocker.

### 3.4 (Observation) Parallel API worktrees on disk — drift risk to watch
- Five API worktrees exist: `tawadoo_api_js` (the live one), `tawadoo_api_b02`, `tawadoo_api_b11_db_c1`,
  `tawadoo_api_s25c3`, `tawadoo_api_s34`. Each has its own `allowed-events.ts`.
- My change landed ONLY in `tawadoo_api_js` (correct — it's the `Ramzi_V2` line CI deploys). The other four now
  **lag** on this entry. Per §28, `Ramzi_V2` is the single source of truth; the others are candidates/history.
- **Queue note:** if any of those worktrees is later integrated, ensure it does not revert this allowlist entry
  (the S-CTO-15 + S-CTO-19api additions). Low risk today, but it's exactly the invisible-drift class §28 warns about.

### 3.5 (Method note) Live DB read required a workaround worth recording
- First one-off task failed (`Cannot find module 'pg'` — `pg` isn't resolvable from `/tmp`). Fixed by requiring
  `pg` from `/usr/src/app/node_modules/pg` and running from the app workdir. Second task exit 0.
- No repo `db-query` helper script exists in the image for ad-hoc reads. **Suggestion (housekeeping, optional):**
  a tiny committed read-only verification script (like the existing `scripts/s-cto-8-live-qa.js`) would make future
  live-canonical DB checks one-step instead of hand-rolled inline node. Not urgent.

### 3.6 (No issue, stated for completeness) Errors encountered
- Cosmetic zsh `command not found: #` / `timeout` noise in a couple of heredoc/portability commands — did not
  affect results (files written, DNS/private-VPC confirmed by other means). No functional error.
- No test failures other than the intended fail-first RED. No CI failures. No crash-loop.

---

## 4. VERIFICATION LEDGER (what state each claim reached)

| Claim | State proven | How |
|---|---|---|
| Event added to allowlist | SOURCE + LIVE | file diff + live DB canonical read |
| Fail-first red→green | LOCAL | jest RED then GREEN |
| Build/lint/typecheck | LOCAL | nest build / eslint / tsc all 0 |
| Deployed = committed code | DEPLOYED (digest) | running task digest == commit build digest |
| Event tags canonical live | LIVE (DB) | `ta_analytics_event` row has no `_is_canonical` |
| Web emits canonical name | **NOT DONE** | web still emits `app_banner_dismissed` |
| shown/clicked have emitters | **DISPROVEN** | zero web emitters (§3.2) |
| BO / MCP impact | NONE | grep both repos, no refs |

---

## 5. SUGGESTED QUEUE ENTRIES (for the Brain to schedule)

1. **S-CTO-20web** (web, next): rename `app_banner_dismissed → app_install_banner_dismissed` in
   `OpenInAppBanner.tsx`, browser-verify, confirm canonical via DB. *Reason: realizes the unblock this session enabled.*
2. **S-CTO-2x-taxonomy** (investigate/MODE A): resolve the app-install banner taxonomy mismatch (§3.2) —
   founder decision on whether to emit `app_install_banner_shown`/`clicked` or retire them. *Reason: dead canonical names vs real generic-banner signals.*
3. **Cutover-checklist item:** add a container health check to `tw-staging-task-back` (§3.3). *Reason: real stability probe, deferred polish.*
4. **Housekeeping (optional):** commit a reusable read-only DB verification script (§3.5); purge synthetic probe rows if desired (§2).

---

## 6. ROLLBACK (this session)
`git revert 50cfa12` — removes the single allowlist line + the test. Additive, reversible, no migration/schema/data change.

---
*Own-session status is a proposal. Durable accepted status + queue updates are the Brain's to write (§18/§19/§20).*
