# B16-S140 — Production Baseline & Drift Audit (READ-ONLY)

**Date:** 2026-09-01
**Auditor:** Brain B16
**Method:** Read-only. Verified from live git, live AWS (RDS/ECS/CLI), and HTTP probes. No prod mutation, no repo writes, no branch switches, no secret values read. Where a fact could not be read read-only, marked UNKNOWN-BLOCKED.
**Scope:** Production (`main` line) as (a) §28 cutover compatibility target and (b) human-CTO + release-discipline reference. Prod is NOT in the refactor scope.

---

## 1. VERDICT
Prod is a **healthy, disciplined baseline** running a DIFFERENT, simpler schema strategy than the refactor line. The §28 drift is **small and concrete: 4 API commits to port (2 security), 38 web commits, 1 BO commit** — not a platform risk. The human-CTO platform is already the shared base of Ramzi_V2. Two corrections to earlier B16 findings (recorded below). The single most important discovery: **prod avoids the migration-crash entirely because prod runs `synchronize: true`, not migrations** — that is the direct answer to the H2b design question.

---

## 2. PROD SOURCE TRUTH (verified)

| Repo | Prod branch | Prod running image | main vs Ramzi_V2 drift |
|---|---|---|---|
| tawadoo_api_js | main (`c06a305`) | `:0.1.419` | main +8 / Ramzi_V2 +623 (merge-base 317cf774) |
| tawadoo_web_js | main | `:0.1.938` | main +38 / Ramzi_V2 +1303 |
| admin_bo_tawadoo | main | `:0.1.62` | main +1 / Ramzi_V2 +163 |
| -tawadoo-mcp- | main | ECR `:5ad5bc5` | main +0 / Ramzi_V2 +46 |

- **Version tags are CI-run-number based** (`github.run_number + offset`), NOT git tags → tag→commit mapping needs the GHCR/ECR image label or CI run history, not local git. Recorded as a partial-UNKNOWN (git side resolved; exact tag→SHA needs registry label read).
- The human-CTO foundation lives in the SHARED BASE at/below merge-base `317cf774`, which IS an ancestor of Ramzi_V2 → **already in the refactor line.** Ramzi_V2 is built on top of it (+623). Prod does NOT hold "lost human commits."

## 3. §28 DRIFT — THE PORT LIST (verified by subject match)

### API — 8 main-only commits: 4 already in Ramzi_V2, 4 MUST-PORT candidates
ALREADY IN Ramzi_V2 (cherry-picked/re-implemented):
- `a0a5ac2` sitemap endpoint → Ramzi_V2 `364197e` ✅
- `a1ab58e` posts PUBLISHED filter → Ramzi_V2 `ed0a4c5` ✅
- `d58813c` CONTENT_ENGINE_ENABLED kill-switch → Ramzi_V2 `9145a39` ✅
- `dd7cdf3` saved_search_match disable → Ramzi_V2 `a115601` ✅

**MUST-PORT before cutover (NOT in Ramzi_V2):**
- `6e06278` **fix(security): add JwtAuthGuard to unprotected message and report endpoints** — SECURITY, port.
- `1c714d6` **fix(security): Cache-Control no-store on message endpoints** — SECURITY (private-data proxy caching), port.
- `c06a305` fix(sitemap): remove isModerated filter (prod uses published status only) — the follow-up to the sitemap already ported; verify Ramzi_V2's sitemap has this fix.
- `a1987ae` fix(blog): restore article generation + visibility (July 2026 incident) — verify Ramzi_V2's content-engine already covers this (likely, given the kill-switch is ported).

→ **Real cutover port list: 2 confirmed security commits + 2 to verify.** Small, concrete.

### Web — 38 main-only commits, BO — 1, MCP — 0
Not yet individually classified (larger set for web). MUST be classified in the cutover phase the same way (already-in / superseded / must-port). MCP is clean (0 drift). BO is 1 commit to check.

## 4. THE H2b REFERENCE — HOW PROD AVOIDS THE MIGRATION CRASH (key discovery)
- **PROD `main` app.module.ts: `synchronize: true`** (verified `git show origin/main:src/app.module.ts`). No `migrationsRun`.
- **Ramzi_V2: `synchronize: false` + `migrationsRun: true`** (the line that crash-looped in S138).
- **Meaning:** prod lets TypeORM auto-sync the schema from entities at boot — so prod NEVER runs formal migrations, and never hits the runtime-user-not-owner DDL crash. The `revert/b11-s34-migration-privilege-blocker` branch confirms prod hit the migration-privilege wall once (B11-S34) and REVERTED to stay on the synchronize model.
- **H2b design implication:** prod's answer (`synchronize:true`) is NOT the right target — `synchronize:true` in production is dangerous (can drop/alter columns from entity drift, no review, no history). The refactor's D-INFRA must choose the PROPER model: **`synchronize:false` + `migrationsRun:false` + a dedicated migrator identity (owner creds) running migrations via a bounded task/CI step**, app boots without DDL rights. Prod's synchronize model is the "why it never crashed" explanation, not the fix to copy.

## 5. INFRASTRUCTURE-AS-CODE — CORRECTED (my earlier "no IaC" was WRONG)
- **Terraform DOES exist:** `tawadoo_web_js/infra/cloudfront/` — `cloudfront.tf`, `waf.tf`, `acm.tf`, `variables.tf`, `outputs.tf`, `versions.tf`, README, .gitignore. Committed on Ramzi_V2 (`02df7ca9`, message: **"Terraform config for CDN layer (review only, not applied)"**).
- **Accurate statement:** IaC is **partial and aspirational** — covers only the web CloudFront/WAF/ACM (CDN) layer, and is explicitly NOT applied. ECS, RDS, Redis, ALB, ECS task-defs, secrets, alarms remain hand-managed (no IaC). So the D-INFRA refactor goal stands (capture ECS/RDS/etc. as applied IaC), but the CloudFront Terraform is a starting point to build on, not greenfield.

## 6. DATABASE (prod, live)
- `tw-postgres-cluster-prod` — Aurora PostgreSQL **16.11**, **1 member (NOT Multi-AZ)**, backup retention **7 days**, **StorageEncrypted: true**, private.
- Same engine/version as staging (good — parity). Resilience gap = single-instance (no Multi-AZ failover). Backups + encryption present. Multi-AZ decision belongs on the cutover/hardening checklist, not urgent.

## 7. API ENDPOINTS / RUNTIME (prod, live HTTP probe)
- `www.tawadoo.ma` → **307** (alive, redirects — normal).
- `api.tawadoo.ma` (+`/health`) → **404** (API responds but no public root/health route; it's behind CloudFront/ALB — this is expected, not a fault. No public health path is itself a small observability gap).
- `bo.tawadoo.ma` → **000** (not publicly reachable — IP-restricted back office, correct security posture).
- `mcp.tawadoo.ma` → **404** (responds on MCP protocol paths, not root — expected).
- Prod ECS (from earlier verified state): back:101, front:170, bo:48, mcp:11, all 1/1 healthy.

## 8. GITHUB / README / STRUCTURE
- **READMEs:** api ✅, web ✅. **BO ❌, MCP ❌** — documentation gap (two repos with no README). Add during refactor doc hygiene.
- **Release convention (prod):** version tags `0.1.x` (CI run-number based) + immutable per-build. More disciplined than staging's mutable `:staging-v2`. Prod ECS pins version tags; staging pins mutable/digest. The refactor's D-INFRA should standardize on immutable digests for BOTH environments.
- **Branch topology (api):** ~30 branches — active human release flow (deploy/*, hotfix/search-*, hotfix/content-engine-*, feat/*, revert/*). Prod merges via PRs (commit subjects show `(#7)`, `(#8)`, `(#9)` — real PR flow). Ramzi_V2 does NOT use PRs (direct pushes). The refactor should adopt prod's PR discipline.

## 9. CORRECTIONS TO EARLIER B16 FINDINGS (recorded per §57)
1. **"No IaC anywhere" — WRONG.** Partial Terraform exists for web CloudFront/WAF/ACM (review-only, unapplied). Corrected in §5.
2. **"8 commits = human-CTO work at risk" — WRONG** (already corrected earlier). They're Kiro hotfixes; 4 already in Ramzi_V2, 4 to port (2 security). Confirmed by subject match in §3.
3. **Prod migration model** was an open question; now ANSWERED: prod uses `synchronize:true` (auto-sync, no migrations) — which is WHY it never crashed, and is NOT the target model for H2b (§4).

## 10. FEEDS INTO
- **Cutover checklist:** port the 4 API commits (2 security first), classify the 38 web + 1 BO drift, add BO/MCP READMEs, standardize immutable digests, Multi-AZ decision, adopt PR flow.
- **Refactor §13 Entry Report:** D-INFRA now has the H2b answer (proper migrator separation, NOT synchronize:true), the partial CloudFront Terraform as a base, and the prod release discipline as reference.

## 11. GUARDRAILS HONORED
Zero prod mutation. Zero repo writes. No branch switches (explicit-ref git reads only). No secret values. No deploy. HTTP probes read-only. $0. All working trees untouched.

## 12. UNKNOWN-BLOCKED (honest limits)
- Exact prod version-tag → git SHA mapping (tags are CI-run-number based, not git tags; needs GHCR/ECR image label read or CI run history — not resolvable from local git alone).
- Prod `api.tawadoo.ma` deep liveness (404 on root/health is expected behind CloudFront; full authenticated flow not probed read-only from this workstation).
