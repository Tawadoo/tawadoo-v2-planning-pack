# S-CTO-24 — END SESSION REPORT FOR THE BRAIN
**Session:** S-CTO-24 · FRONTEND REGRESSION 360 (READ-ONLY, 11 items)
**Date:** 2026-09-06 · **Mode:** MODE A (discovery/forensics) · **Type:** read-only diagnosis
**Author:** Kiro execution session (self-reported proposal; requires independent Brain QA per §18)
**Companion evidence:** `S_CTO_24_FRONTEND_REGRESSION_360_EVIDENCE_2026_09_06.md`

---

## 1. PROPOSED STATUS
**FINISHED — COMPLETE** for the read-only diagnosis mandate (all 11 root-caused + fix plan + build grouping, no code changed, HEADs unchanged).
This is a **proposal**; the Brain must independently accept per §18. Note the runtime caveats in §5 below — several findings are source+git only, NOT runtime/browser-confirmed, so any downstream build session must reproduce live first.

## 2. PROVENANCE (verify, don't trust)
- WEB `tawadoo_web_js` branch Ramzi_V2 @ **8b0a815f** (staging). Prod ref **origin/main @ a6499893**.
- API `tawadoo_api_js` branch Ramzi_V2 @ **8d55a20**.
- BO `admin_bo_tawadoo`: inspected only enough to confirm it is NOT involved in lead status (leads are API+web only). BO HEAD not recorded (no BO work).
- No code changed. HEADs re-verified unchanged at session end (web 8b0a815f, api 8d55a20). Only two evidence `.md` files written at workspace root.
- Skills activated this session: `tawadoo-source-truth`, `tawadoo-runtime-provider-qa` (discovery+activation in-session; COMPLETION is partial — see §5 runtime caveat).

## 3. WHAT WAS CONFIRMED FROM SOURCE (the 11) — one line each + layer
- **FR-1** pink strip = `LiveAuctionStrip.tsx:62` raw gradient, homepage-only (add, commit 97dbaa1b). WEB.
- **FR-2** auction = `ProductCard` bid branch (no separate component); subscribe/notify = **PARTIAL** (FCM+email+in-app channels exist; NO subscribe endpoint, NO auction-start trigger/cron, NO UI CTA). CROSS-LAYER.
- **FR-3** pagination unwired — commit `0ffb088f` re-fetch effect excludes `currentPage`/`searchParams`; API `page` param correct. WEB.
- **FR-4** chat — rebuilt websocket-only `SocketProvider` (commit 144ec858 + fix chain); API gateway/CORS healthy; suspect socket connect or ALB WS upgrade. WEB + INFRA.
- **FR-5** detail layout — `product-details-view.tsx` `justify-center`+`flex-1`+ new `w-[300px]` ad rail (commit 523adc9a) → empty space + horizontal AdFallbackPlaceholder. WEB.
- **FR-6** cards — `productCard.tsx` rewrite; `bg-gradient-to-r` badges with NO color stops + coastal repaint 48bb6db7; prod "side banners" = old framed `w-1/4` filter column. WEB.
- **FR-7** save-search — `search-results-view.tsx:371/:376` `SAVEABLE_SEARCH_PARAMS` includes `cat`/`scat`/`scat2`/`type`/`store` → button on bare category page. WEB. **(root cause exactly as prompt predicted, confirmed at :371/:376, not :371/:375.)**
- **FR-8** badge — no `notifications/mark-all-read` endpoint; panel-open never clears; messages clear only on InboxView mount. CROSS-LAYER.
- **FR-9** callback deeplink — `notifySeller` (leads.service.ts:687) omits `service`/`objectId`; web handler has no lead branch → routes to /notification-unavailable; copy DB-driven/weak. CROSS-LAYER.
- **FR-10** unlock transparency — API unlock response drops computed `source`/`coinsSpent`/balance/allowance; web hard-codes "10", no confirm/feedback. CROSS-LAYER.
- **FR-11** mark-contacted — `markAsContacted` throws `ForbiddenException('unlock first')`; web swallows to generic toast. Web call itself is correct. CROSS-LAYER + POLICY.

## 4. BUILT-IN-BACK vs NOT-BUILT-IN-FRONT (nothing buried) — the API/back/DB/front split
This is the key "what exists where" map the Brain asked for:

| Capability | API / back | DB | Web / front | BO | Gap |
|---|---|---|---|---|---|
| Lead unlock payment (included-allowance→coins) | ✅ full logic + `LeadUnlock`/`WalletHistory` writes, Amplitude event | ✅ tables exist | ⚠️ calls unlock but shows NOTHING about how paid; hard-codes cost 10 | n/a | API **computes** source/cost/balance but **does not return** them; web can't display truth (FR-10) |
| Lead status transitions (viewed/contacted/converted) | ✅ endpoint + service + guards + sovereign events | ✅ | ⚠️ correct call, generic error handling only | n/a | API throws unlock-first 403; web can't explain it (FR-11) |
| Callback/lead notification | ✅ created (`notificationCode:'leadReceived'`) + WhatsApp + deeplink helper computes a URL | ✅ notification + DB-driven translations | ❌ handler ignores API deeplink, no lead branch → dead-ends | n/a | API omits `service`/`objectId`; web has no route (FR-9). Copy in `ta_notification_translation` weak/absent |
| Notifications mark-all-read | ❌ **endpoint does not exist** (only `PATCH :id/read`) | n/a | ❌ no clear-on-view | n/a | Must BUILD API endpoint + web clear (FR-8) |
| Messages mark-all-read | ✅ `messages/mark-all-read` exists | ✅ | ⚠️ called only on InboxView mount | n/a | Web wiring gap (FR-8) |
| Auction subscribe/notify-on-start | ❌ no endpoint, ❌ no start trigger/cron, ❌ no notification code | ❌ no subscription entity | ❌ no CTA (only a countdown) | n/a | Channels (FCM/email/in-app) exist but the whole feature is UNBUILT (FR-2) |
| Realtime chat gateway | ✅ gateway, CORS allows staging, joinThread/sendMessage/notifyMessage | n/a (Redis adapter) | ⚠️ rebuilt websocket-only client; may not reach `connected` | n/a | Web socket connect + **ALB WS upgrade NOT verified** (FR-4) |
| Publications search pagination | ✅ `page`/`limit`, returns totalPages/currentPage | ✅ | ❌ page change never re-fetches | n/a | Web re-fetch wiring (FR-3) |
| Display-ads platform (banners) | (feature) | — | ✅ new components; ⚠️ `listing_sidebar` slot DEFINED but NOT rendered on search; sidebar variant is horizontal `compact` | n/a | Front layout regressions FR-5/FR-6; a defined-but-unrendered slot |
| LiveAuctionStrip | uses `useAuctions()` | — | ✅ homepage only, raw styling | n/a | Front cosmetic FR-1 |

**Amplitude / sovereignty note:** `auction_reminder_set` and `auction_watched` are in the event allowlist but `auction_reminder_set` is **never emitted** (allowlisted for a feature that doesn't exist yet — consistent with FR-2 being unbuilt). `lead_unlocked` fires with `coins_spent` from the web hard-coded 10, so **analytics is wrong when an allowance was used** (records 10 coins on a 0-coin unlock) — a data-quality bug feeding the lake (FR-10 side effect). Flag for the training-data-quality owner.

## 5. RUNTIME CAVEAT — what is NOT verified (honest, per tawadoo-runtime-provider-qa)
Everything above is **source + git** verified. The following were NOT confirmed and must be the FIRST step of their build sessions:
- **FR-4:** whether the staging socket actually reaches `connected` on desktop, and whether the staging **ALB permits WebSocket upgrade** (client is websocket-only by design — an LB config item, not app code). No browser/network QA was run.
- **FR-1 / FR-5 / FR-6:** exact live rendering not browser-confirmed (no Chromium/WebKit run this session). Diagnosis is from source + the founder's screenshots.
- **Deployed digest:** I did NOT verify the running staging image was built from 8b0a815f. If the deployed artifact differs from HEAD, some "staging" symptoms may map to a different commit (§28.1 drift trap). Build sessions should confirm deployed digest → commit before trusting line numbers against live.
- `tawadoo-runtime-provider-qa` SKILL COMPLETION is therefore **partial by design** (read-only session, browser QA deferred to build sessions).

## 6. OPEN / INCOMPLETE / OUT-OF-SCOPE ISSUES FOUND (nothing buried — Brain to queue)
Small and large, including things outside the FR-11 scope:

**O-1 (RED, cross-layer, FR-8):** No `notifications/mark-all-read` API endpoint exists. New endpoint required (mirror messages). Owner: api + web.

**O-2 (POLICY STOP, FR-11):** "must unlock before mark-contacted" business rule — is it intended? Founder decision gates whether FR-11 is a web messaging fix or an API rule change. Surface before building S-CTO-27.

**O-3 (POLICY STOP, FR-2):** hide-not-started-auctions vs show-with-subscribe. Founder decision changes build size materially (subscribe endpoint + entity + start trigger + UI vs just hide).

**O-4 (POLICY STOP, FR-5/FR-6):** do we want real VERTICAL side ad banners, or the clean prod look with no side banner? The prod "dual side banners" the founder likes were actually the old centered-content margins + a framed filter column — there is NO dual-banner component to restore. Founder must pick the target.

**O-5 (§30 copy):** FR-9 (`leadReceived` translations — currently DB-driven, weak/absent) and FR-10 (unlock confirm/feedback) and FR-11 (specific error) all need founder-approved FR/AR/EN copy before shipping.

**O-6 (DATA QUALITY, out of FR scope but found):** web hard-codes `coins_spent:10` on `lead_unlocked` analytics even when an allowance (0 coins) was used → wrong training/lake data. Fix rides with FR-10 but flag to the sovereignty/training owner independently in case FR-10 slips.

**O-7 (§35 RUNTIME DDL, out of scope, found while reading leads):** `leads.service.ts:119-125 onModuleInit` runs `ALTER TABLE ta_lead ADD COLUMN IF NOT EXISTS buyer_user_id ...` + `CREATE INDEX` at runtime startup. `entity.service.ts` also has runtime `ADD COLUMN IF NOT EXISTS` DDL. Per §35 (DB credential separation), the runtime DB user may not have DDL rights — if `synchronize/migrationsRun` and grants differ from what these assume, this is a crash-loop / silent-skip risk. This is Session-4-era ("Session 4" comment in code). **Not verified against live grants this session.** Queue a §35 audit: does the runtime user actually have DDL on `ta_lead`/entity tables, or does this silently warn-and-skip (the code catches and logs a warning)? If it skips on prod-like grants, `buyer_user_id` may not exist where expected.

**O-8 (BUILD HYGIENE, out of scope, found):** the WEB repo commits `.next/required-server-files.json` containing stale `http://localhost:3010` / `:3000` `NEXT_PUBLIC_API_BASE_URL`. CI ships `.env.staging` so staging is fine, but committing a built `.next/` artifact with a localhost URL is a footgun (any path that reads the committed artifact instead of the fresh build gets localhost). Queue: gitignore `.next/` or stop committing it. Low severity, real.

**O-9 (DEFINED-BUT-UNRENDERED, FR-5/FR-6 adjacent):** `slot-config.ts` defines a `listing_sidebar` ad slot that is NOT rendered on the search results page (only search_top/search_mid are). Dead/parked config — either wire it (vertical rail) or note it as intentionally parked. Flag so a future session doesn't assume it's live.

**O-10 (FRAGILE-SURFACE WARNING, FR-4):** the chat frontend was fully rebuilt (144ec858) then patched 5+ times for connect/auth-timing (9bd71355, 16547e12, 788c0b10, 8fa8515e, 638e0278). This is a known-fragile area (§33.4). The build session must add a regression guard (§33.1) — a Playwright E2E that proves live send+receive on staging — not just a code tweak.

**O-11 (COLLISION RISK):** FR-3 and FR-7 both live in `search-results-view.tsx`/`filter.tsx`. Any parallel session touching search MUST be serialized with S-CTO-25 (§11 repository-isolation / no same-file concurrent writes).

**O-12 (DRIFT, §28):** origin/main is NOT an ancestor of Ramzi_V2 (Ramzi_V2 +38 / main +1335, merge-base 8b0d7440). This is expected legacy-prod-vs-future-line drift, but the Brain should confirm none of the 1335 prod-only commits contain a lead/notification/search fix that Ramzi_V2 is missing (legacy-production-drift class per §28.3). Not investigated this session (out of scope).

## 7. PROPOSED NEXT BUILD SESSIONS (for the queue — OPEN/PENDING, not fired)
Functional (RED) before cosmetic (YELLOW):
1. **S-CTO-25 · SEARCH FUNCTIONAL (R0, WEB)** — FR-3 pagination + FR-7 save-search gate. Same files → one session (O-11). Add pagination E2E guard.
2. **S-CTO-26 · CHAT RECOVERY (R0, WEB+INFRA)** — FR-4. Runtime-reproduce first (O-10, O-5-caveat); verify ALB WS upgrade; add live send/receive E2E guard.
3. **S-CTO-27 · LEADS (R1, CROSS-LAYER)** — FR-11 (needs O-2 decision) + FR-10 (api response extension + web confirm/feedback + O-6 analytics fix). §30 copy (O-5).
4. **S-CTO-28 · NOTIFICATIONS (R1, CROSS-LAYER)** — FR-8 (build O-1 endpoint + web clear) + FR-9 (api service/objectId + web lead branch + O-5 copy).
5. **S-CTO-29 · CARD + AUCTION VISUAL (R2, WEB + FR-2 cross-layer)** — FR-6 + FR-2 render; FR-2 subscribe (O-3 decision) may split out; z-index/visual sweep §33.6.
6. **S-CTO-30 · DETAIL + PINK STRIP (R2, WEB)** — FR-5 (O-4 decision) + FR-1. Coordinate side-banner decision with S-CTO-29.

Independent §35/hygiene follow-ups (any time, small): **audit O-7** (runtime DDL vs grants), **O-8** (stop committing `.next/`), **O-9** (decide `listing_sidebar` slot fate), **O-12** (legacy-prod-drift scan for lead/notif/search fixes).

## 8. LAW COMPLIANCE NOTES
- §26 founder-operated: all technical work done by Kiro; only genuine business/policy decisions surfaced (O-2/O-3/O-4/O-5) as APPROVE/CHANGE/REJECT.
- §52 founder-in-doubt: FR-11 unlock rule and FR-2 hide-vs-subscribe correctly flagged as business decisions, NOT self-decided.
- §10/§11/§15: no code, no commit, no deploy, no prod mutation, no DB write, no cross-repo edit. Read-only respected.
- §49: no guessing — every claim source/git backed; unverified items (runtime, deployed digest) explicitly marked UNKNOWN, not asserted.
- §18: this report is a proposal; independent Brain QA required before durable status.
