# S-CTO-11 — END-SESSION REPORT (for the Brain)

**Session:** S-CTO-11 · Draft/Autosave Finisher (resolve P1 — the uncommitted draft/autosave work in `product-form-v2.tsx`)
**Date:** 2026-09-05
**Repo (writable):** `tawadoo_web_js` ONLY · `tawadoo_api_js` / `admin_bo_tawadoo` / `-tawadoo-mcp-` read-only (respected)
**Branch:** `Ramzi_V2` — started at `36b7183c` (== deployed front tip, 0/0 vs origin), shipped at **`780bdb58`** (pushed, 0/0)
**Skills:** `tawadoo-source-truth` + `tawadoo-runtime-provider-qa` (both applied)
**Proposed status:** **`FINISHED — COMPLETE`** for the shipped scope (founder-approved UI, committed to Ramzi_V2, CI green incl. post-deploy smoke, deployed digest == running digest, browser-verified live Chromium+WebKit+mobile+Arabic-RTL, publish path proven not regressed). Several **open follow-ups** below — none block the shipped feature.
**Companion evidence:** `tawadoo_web_js/S_CTO_11_DRAFT_AUTOSAVE_FINISHER_EVIDENCE_2026_09_05.md` (full matrices, provenance, screenshots list)
**Independent QA:** REQUIRED per §18 before durable acceptance. This report is a proposal to verify, not accepted truth.

---

## 0. TL;DR FOR THE FOUNDER
- The **Save as draft** button and the **auto-recovery banner** are now LIVE on staging, proven working in real browsers (Chrome, Safari, mobile, Arabic RTL). The normal add-photos → publish flow still works exactly as before, no errors.
- It is committed on the truth line (`Ramzi_V2`), deployed, and I can undo it instantly if needed.
- **Small housekeeping left:** the new "draft" tracking events need to be added to the data list so they count as training data. That is the only real open item from this feature; it does not affect what a seller sees.
- **One thing I could NOT fully prove live** (honest): I proved the button is there and the recovery works, but I did NOT click "Save as draft" all the way through to see the draft land in "My listings" with a real account+coins. Source says it works (drafts are first-class); it just wasn't exercised end-to-end by me. Queued below.

---

## 1. WHAT WAS BUILT / SHIPPED (front end — `tawadoo_web_js`)
All from the S-CTO-2 uncommitted body, finished + shipped this session. Commit `780bdb58` (12 files, +791/-40):
- `utils/draft-autosave.ts` (NEW) — per-user localStorage snapshot (7-day TTL, content-gated, version-guarded, `safeLocalStorage`; media files not serialized, counts stored).
- `product-form-v2.tsx` — debounced autosave + flush on tab-hide/pagehide; mount resume banner (Resume/Discard); manual `handleSaveAsDraft` (existing create-in-draft endpoint; reuses `publication.status==='draft'`); mobile Save-as-draft button.
- `components/sidebar-cards.tsx` — desktop Save-as-draft button (create mode only).
- `components/photo-first-step.tsx` — routes the 2+ image same-item check through the shared helper (behaviour preserved: block only on 'different', proceed on 'unknown').
- `utils/media-cross-check.ts` — `crossCheckImagesSameItem()` (image↔image counterpart; consolidates the duplicated check).
- `public/locales/{fr,en,ar}.json` — 10 new `productForm` keys (founder-approved).
- Tests: `draft-autosave.test.ts` (NEW, 8), `media-cross-check.test.ts` (+8), two updated regression guards.

**Verified live (deployed staging, 4 projects, all PASS):** autosave writes snapshot · Save-as-draft button present · **Publish still present/enabled (no regression)** · reload → Resume banner appears (non-clipped, correct RTL) · Resume restores the exact typed title · 0 client crashes. CI post-deploy Playwright smoke also green.

---

## 2. WHAT WAS BUILT IN BACK-END vs NOT BUILT / NOT VERIFIED (cross-surface truth)

### 2.1 API (`tawadoo_api_js`) — READ-ONLY this session, NO change made, NONE needed
- **Already built (source-verified this session):** `PublicationStatus.DRAFT = 'draft'` is a first-class enum (`publication.entity.ts`); the `status` column **defaults to `DRAFT`** (`@Column(... default: PublicationStatus.DRAFT)`); publish is a separate step; drafts are correctly **excluded from index/feed eligibility** (`INDEX_ELIGIBILITY_WHERE = { isModerated:true, status:PUBLISHED, deletedAt:IsNull }`; unit specs `index-eligibility-gate`, `publication-creation-defaults`, `index-recovery` all assert draft → not indexed); the moderation state machine **rejects draft→approved** (`approve-publication-idempotent.spec.ts`). So a saved draft is private, unindexed, unsearchable by design.
- **Conclusion:** the draft feature is 100% additive on the client; the server already supported it. No migration, no schema change, no api deploy required.
- **NOT verified this session (API):** server-draft **dedupe on repeated Save-as-draft** (code relies on the existing `publicationItem?.id || id` guard so a 2nd save PATCHes rather than creating a duplicate — NOT live click-twice tested); exact coin behaviour on Save-as-draft (does saving a draft debit coins, or only publishing? — not confirmed; likely no debit until publish, but unverified).

### 2.2 Front-end (`tawadoo_web_js`) — WRITABLE, shipped (see §1)
- **Built + verified live.** One residual: the manual **Save-as-draft round-trip** (click Save → routed to `/dashboard/announcements` → draft visible → open → publish) was NOT exercised live by me (see §3 O1). Source confirms `my-announcements-view.tsx` has a `draft` status filter (`params.status='draft'`) and bubbles drafts to the top — so drafts should appear under "Mes ventes → Brouillon" — but this is source-truth, not runtime-truth.

### 2.3 DB — no change; NOT directly inspected this session
- Draft status column already exists and defaults to `draft` (source). Local autosave writes only to the browser; nothing hits the DB until explicit Save-as-draft (existing endpoint). **NOT verified via psql** (no direct DB access used; no DDL/migration involved so none required).

### 2.4 BO (`admin_bo_tawadoo`) — NOT touched, NOT verified
- Drafts are a status the model already understands. **NOT independently verified** that draft-status listings render correctly in the BO admin listing views, or that moderation/BO does not accidentally surface a private draft. Low risk (drafts are excluded from index + feeds by the API predicate), but unproven in BO. Queue if the Brain wants BO-side confirmation.

### 2.5 AWS / infra — no change
- No new services, no IAM, no cost, no secrets. `Dockerfile` / workflows / forbidden files untouched. Deploy used the existing CI (mutable `staging-v2` tag per §42; ECS `tw-staging-svc-front` force-new-deployment). Running digest `sha256:a3c37d58…0b1ba4` on both tasks == the image CI built from `780bdb58`.

### 2.6 MCP — untouched, irrelevant to this feature.

### 2.7 Lake / Amplitude (project `795817`) — events fire but NOT registered
- The manual save reuses existing `listing_draft_saved`; 4 NEW recovery events (`listing_draft_autosaved_local/recovered/resumed/discarded`) fire via the `track()` passthrough.
- **Taxonomy check this session:** NONE of the 5 draft event names are currently registered in the Amplitude taxonomy — not even the pre-existing `listing_draft_saved`. The `listing_*` family is otherwise well-populated. So draft events are not yet first-class training signal. (See §3 O2 — the one real housekeeping item.)

---

## 3. OPEN / INCOMPLETE ITEMS (including small ones) — for the Brain queue

| # | Item | State | Severity | Owner / next |
|---|---|---|---|---|
| **O1** | **Save-as-draft round-trip not live-verified** | Button presence + autosave/resume PROVEN live; the full manual Save → draft appears in "Mes ventes/Brouillon" → open → publish was NOT clicked through with a real account+coins | MEDIUM | Web QA session: one authed flow on staging. Source strongly supports it (draft filter exists), but prove runtime. |
| **O2** | **5 draft events not on the lake taxonomy** | `listing_draft_saved` + 4 recovery events fire but are absent from Amplitude project 795817 taxonomy | MEDIUM (Commandment 2 — training data) | Founder micro-decision: register them on the tracking plan / 478-event allowlist. Recommended YES. Then confirm ingestion. |
| **O3** | **Server-draft dedupe on repeated save** | Relies on `publicationItem?.id || id` guard (2nd save should PATCH not create) — NOT live click-twice tested | LOW-MEDIUM | Fold into O1's authed session (click Save twice, assert one draft). |
| **O4** | **Coin behaviour on Save-as-draft** | Unverified whether saving a draft debits coins or only publishing does | LOW | Confirm in O1 (read wallet before/after Save-as-draft). Likely no debit until publish. |
| **O5** | **Autosave field coverage** | Restores title/desc/price/category/condition/listing-type/location/property-selections; does NOT restore media files (impossible locally), auction dates, or map canvas image; banner tells seller to re-add photos | PARTIAL BY DESIGN | Founder/Brain confirm acceptable, or expand (e.g. persist auction dates). |
| **O6** | **BO render of draft listings** | Not verified in `admin_bo_tawadoo` | LOW | Optional BO QA: draft-status listings display correctly and are not publicly exposed. |
| **O7** | **Behaviour asymmetry (image vs video same-item on 'unknown')** | image path PROCEEDS on 'unknown'; video path BLOCKS on 'unknown' — deliberate, matches §50 + prior behaviour (S-CTO-2 §5.4) | POLICY | Founder call: should photo same-item check block on 'unknown' like video does? Genuine product decision, not cleanup. |
| **O8** | **Legacy `product-form.tsx` retirement + toggle flip** | PARKED (founder-gated). v2 is now a full superset (draft parity closed) | FOUNDER-GATED | Separate bounded session (S-CTO-2 §7 plan): flip `NEXT_PUBLIC_V2_UI_ENABLED`, delete 2,148-line legacy, orphan-sweep, re-verify. Do NOT bundle with a feature. |
| **O9** | **Legacy dead-path event name mismatch** | Legacy fires `listing_saved_draft`; v2 fires `listing_draft_saved` (different string). Inert (legacy off on staging), dies with O8 | LOW | Resolve as part of O8. |

---

## 4. ISSUES / ERRORS ENCOUNTERED (including outside scope — for the Brain)

### 4.1 Pre-existing flaky test — AI guidance route (OUTSIDE SCOPE, unchanged)
- `src/app/api/ai/guidance/interaction-id-correlation.test.ts` times out under full-suite CPU contention, passes in isolation (~7.4s). NOT my file, NOT fixed. **Brain action:** queue a fix (raise `testTimeout` for that file or stub the network call) — it makes the full suite non-deterministic and will bite CI if unit tests are ever CI-gated. (First flagged by S-CTO-2 §5.1; still open.)

### 4.2 Prompt premise was slightly off (recorded for prompt-authoring quality, §18/§31)
- The S-CTO-11 prompt assumed the dirty tree was "only the S-CTO-2 draft/autosave work." Source truth: the S-CTO-2 body ALSO included the **media same-item consolidation** (`media-cross-check.ts` + `photo-first-step.tsx`), and a LATER session **S-CTO-5** left additional untracked dirt (playwright reports/configs, evidence `.md`) while changing **0 tracked product files**. I classified all of it from the two evidence files before touching anything. No harm — but the Brain should note the working tree had layered dirt from ≥2 prior sessions.

### 4.3 CI does NOT run the unit test suite (evidence-state honesty, §7/§13)
- `deploy.yml` runs `validate-locales` (JSON + lint gates) → `build-and-push` (Docker + ECS) → `smoke-tests` (post-deploy Playwright smoke). It does **NOT** run `vitest`. So the 176/176 create-form unit pass + full `next build` are **local** evidence, not CI-certified. The browser proof + CI smoke are the live gates. **Brain note:** if unit coverage should gate deploys, that's a CI change (frozen-pipeline, §10 — needs its own approval).

### 4.4 Test-harness friction (not a product defect)
- First browser probe filled the header search box instead of the form title (the create form is photo-first; the title only appears after photo → Continue → **Skip video step** → full form). Fixed the probe to traverse the real steps. Also: the draft analytics `track()` events go to Amplitude, not `console.log`, so my console capture reported them as not-seen — this is a capture limitation, not a missing event (the events exist in source and fire; see O2 for the real registration gap).

### 4.5 Node 20 deprecation warning in CI (informational)
- CI annotated: `actions/checkout@v4` / `setup-node@v4` forced onto Node 24 (Node 20 runner deprecation). Non-blocking now; **Brain note:** a future CI-maintenance session should bump action versions (frozen-pipeline change, needs approval).

---

## 5. LAW / SAFETY COMPLIANCE NOTES
- **Founder guard honored:** feature adds seller-visible UI → I STOPPED, reported, got explicit APPROVE before commit/deploy (§52 / prompt guard).
- **Forbidden/sacred files:** none touched (layout.tsx, AnalyticsTracker, AmplitudeInit, ThirdPartyScripts, next.config.js, Dockerfile, workflows). Legacy form NOT deleted; toggle NOT removed. ✅ (§41, §4B)
- **Dirty-work (§9):** feature files staged by name; `yarn.lock` + all prior dirt preserved unstaged/untouched; no `git add .`, no reset/clean/stash.
- **§28 Ramzi_V2 integration:** commit is ON `Ramzi_V2` and pushed — integration duty discharged (not deferred).
- **§42:** task-def uses mutable `staging-v2` tag; no SHA pinning. **§10:** pipeline unchanged.
- **§30 copy:** 10 new FR/AR/EN strings founder-approved with the ship.
- **Storage law:** `safeLocalStorage` used, CI `lint:storage` green.

---

## 6. ROLLBACK CONTRACT
- **Source:** `git revert 780bdb58` (single additive commit).
- **Deploy:** redeploy prior tip `36b7183c` (mutable `staging-v2`, ECS force-new-deployment).
- **Data:** none written server-side by autosave (local storage only); a seller draft is a normal draft row.
- **Post-rollback probe:** re-run `playwright.scto11.config.ts` (expect banner absent, publish OK).

---

## 7. RAG + R-LEVEL
- **BLUE / R0** — shipped scope: committed, CI green incl. smoke, digest verified, browser-proven in 4 environments incl. RTL, publish not regressed, rollback defined.
- **YELLOW / R1** — residual: O1 (round-trip not live-clicked), O2 (events not registered), CI doesn't run unit tests, BO not checked. All low-to-medium, none block the live feature.

---

## 8. NEXT SESSION
**S-CTO-12 (job/service web)** — reason: with draft/autosave shipped, the next create-listing gap is the job/service publish path — the API throws `PUB_NO_IMAGES_OR_VIDEOS` for jobs/services even though the spec says they need no image (S-CTO-5 §2). Needs the founder's A/B/C decision → web-side alignment (Option B, `tawadoo_web_js`) or a coordinated api session (Option A/C, `tawadoo_api_js`).

**Also queue (from §3/§4):** O1 draft round-trip live QA · O2 register 5 draft events on the lake · O7 photo same-item 'unknown' policy (founder) · O8 legacy retirement (founder-gated) · 4.1 flaky guidance test fix · 4.3/4.5 CI unit-gate + action-version maintenance (frozen-pipeline, needs approval).

**END OF S-CTO-11 REPORT.** Proposed `FINISHED — COMPLETE` (shipped scope). Independent QA to re-verify from source + live; open items above to be queued, not lost.
