# TAWADOO LAW — 10/10 EXECUTION PROMPT STRUCTURE AND NON-REGRESSION

**Authority:** Founder-approved permanent cross-session execution law.  
**Scope:** Every future Tawadoo Brain, execution queue, named implementation/QA session, and `KIRO_PROMPT_NEXT.md`.  
**Purpose:** Preserve the standalone B11-S24/B11-S25 prompt discipline that produced useful source, commit, CI, deployment, and evidence results, while strengthening independent QA so no future session can weaken gates or overclaim completion.

## 0. SUPREME RULE — Every law is binding and non-negotiable

**No rule or law in this document may be ignored, skipped, deprioritized, or treated as optional by any session, Brain, or agent — regardless of context length, session complexity, time pressure, or volume of information.**

Every section in this file is a hard constraint. "Too much to read" is not permission to skip. "The session is long" is not permission to forget. "It seemed irrelevant" is not a defense for violation.

If a session violates any law, the violation must be flagged by independent QA as a **prompt-authoring or execution failure** — not excused. If the Brain violates a law, it must self-correct and document the correction.

The volume of rules exists because each one was earned by a real failure that cost real time. They are not suggestions. They are not guidelines. They are law.

## 1. Absolute non-regression rule

Every future execution prompt must use this structure or a demonstrably stronger version.

- A prompt may add stricter sections, tests, evidence, safety controls, or domain-specific gates.
- A prompt may not silently remove, merge away, abbreviate, or weaken a required section.
- If a section is genuinely not applicable, the prompt must retain the heading, state `NOT APPLICABLE`, and explain why with current source evidence.
- No historical prompt, completion report, checklist, Brain claim, or prior green run can waive current source/live verification.
- Founder approval of a queue does not authorize destructive migrations, production mutations, policy invention, secret exposure, or bypassing a completion gate.
- When this law conflicts with a shorter historical prompt convention, this law wins unless Ramzi explicitly approves a named exception in the current session.

## 2. Prompt issuance law

Before creating or firing a prompt:

1. State the named session’s purpose and expected achievements.
2. Give Ramzi a stop/change/approve opportunity.
3. Re-check queue dependencies, repository overlap, dirty work, and currently in-flight sessions.
4. Fire only 2–3 safe, related, repository-isolated sessions at once.
5. Give every session a permanent ID and human-readable name.
6. Create one complete standalone prompt per execution agent/session.
7. Do not use a generic combined prompt when separate agents need separate ownership.
8. `KIRO_PROMPT_NEXT.md` must contain the complete next executable prompt body—not a launcher, reference, or “read another file” stub.
9. Update Brain and canonical queue to `IN FLIGHT` only when a prompt is actually issued and execution is authorized.

## 3. Mandatory prompt identity header

Every prompt begins with:

- canonical session ID and name;
- exact status (`OPEN/PENDING`, `IN FLIGHT`, or staged/not executable);
- batch/wave;
- environment boundary;
- writable repositories;
- forbidden/read-only repositories;
- required evidence-output path;
- dependency/activation gate if staged.

A staged prompt must prominently say `OPEN/PENDING — DO NOT EXECUTE` and must define every activation condition.

## 4. Mandatory five-line session opener

Before any tool call, every execution agent prints exactly these labeled lines:

```text
BRANCH: <actual or UNVERIFIED; never guessed>
MISSION: <session ID/name and concrete outcome>
ORDER: <reality check → findings → fail-first proof → implementation → validation → CI/deploy → live QA → rollback → evidence/status>
BLOCKERS: <known blockers or none known; identify policy/access/destructive-risk boundaries>
TREE STATE: <actual or UNVERIFIED; preserve tracked/untracked work>
```

After Git/source preflight, the agent prints a corrected five-line checkpoint with actual branches, blockers, and tree states before editing.

## 5. Mandatory standalone prompt sections

Every prompt must contain all of these sections in this order or a stronger clearly mapped order:

1. **Identity/status/environment/repository/evidence header**
2. **Mandatory five-line opener**
3. **Mission and measurable expected achievement**
4. **Durable Brain/canonical/spec/audit authority files**
5. **Actual repository steering files and known contradictions**
6. **Required technical competencies**
7. **Non-negotiable laws**
8. **Strict writable/read-only/forbidden repository boundary**
9. **Pre-edit current reality check**
10. **Finding/evidence verdict gate**
11. **Domain-specific implementation outcomes**
12. **Mandatory fail-first regression matrix and validation**
13. **Commit, CI, immutable staging provenance, DB/provider/live-safe QA**
14. **Rollback contract defined before push and safely exercised where possible**
15. **Durable evidence manifest requirements**
16. **Own-session-only Brain/canonical status update contract**
17. **Final response contract**
18. **Absolute stop conditions**

## 6. Durable authority and steering law

Prompts must list real files, not vague “follow steering” language.

- Read current Brain updates, canonical queue, relevant audits, specs, tasks, and source-backed reports.
- List every applicable `.kiro/steering/*.md` path for writable repositories.
- Distinguish always-included, file-match, and manual steering.
- Never invent installed Kiro skills. State required technical competencies truthfully.
- Where steering disagrees about branches or workflows, do not guess. Inspect current branch/upstream, workflow triggers, remote head, running artifact, and deployed commit; record the contradiction.
- Current source and verified live reality override historical completion claims while preserving superseded history.

## 7. Evidence-state law

Never collapse these states:

```text
documented → source → local → CI → deployed → live
```

For each criterion, record the highest state actually proven. Specifically:

- source existence is not local validation;
- local tests are not CI;
- a green build/deploy workflow is not necessarily a test run;
- pushed is not deployed;
- deployed is not healthy;
- an HTTP 200 is not DB/provider correctness;
- a mutable tag is not immutable provenance;
- a written rollback plan is not an exercised rollback;
- a session-authored evidence checklist cannot redefine or shrink its controlling prompt’s gates.

## 8. Source-first reality-check law

Before editing behavior, inspect:

- all callers and emitters;
- implementations and middleware order;
- DTOs, types, entities, schemas, migrations in chronological order;
- tests and fixtures;
- configuration/env/secret references without values;
- workflows/build/deployment wiring;
- downstream consumers/reports/providers;
- current CI runs and individual jobs;
- current staging artifacts/tasks/digests/health/logs/alarms;
- direct DB/provider facts where authorized.

Repository-wide search is mandatory. A similarly named file or old report is not proof.

The pre-edit evidence matrix must use only:

- `CONFIRMED`
- `PARTIALLY CONFIRMED`
- `NOT REPRODUCED`
- `SUPERSEDED`
- `UNKNOWN/BLOCKED`

Implement only current confirmed defects.

## 9. Dirty-work and Git law

For every touched repository capture branch/upstream, status, diffs, untracked files, recent log, ahead/behind, and worktrees.

Never:

- reset, clean, stash, discard, or overwrite unrelated work;
- force-push;
- use `git add .` or `git add -A`;
- push main/master;
- bypass hooks;
- amend after a hook failure.

Classify overlapping work as `PRE-EXISTING/OWNED ELSEWHERE`, `ADOPTED AFTER REVIEW`, `REVISED`, `REJECTED`, or `BLOCKING`. Stage exact owned files. Use one concern per commit.

## 10. Frozen pipeline and environment law

Without explicit current-session approval, do not change:

- `Dockerfile`
- `.dockerignore`
- `.github/workflows/*`

Default execution environment is staging only. Production mutation is forbidden unless Ramzi gives new explicit approval naming the exact action and risk.

Do not trigger real customer messages, orders, payments, ads spend, destructive provider operations, or unsafe production-like side effects.

## 11. Repository-isolation law

Every prompt names writable, read-only, and forbidden repositories/files.

- If a correct fix requires crossing the boundary, stop that criterion and document the exact coordinated change/owner.
- Do not smuggle cross-repository edits into a session.
- Concurrent sessions must not write the same repository or shared deployment/schema/security surface unless explicitly coordinated and serialized.
- A blocker in one independent workstream does not stop other safe work.

## 12. Fail-first regression law

Every new regression assertion must be demonstrated failing against the actual broken/pre-fix behavior before its green result is trusted.

- An old test failing because it expected insecure behavior is not sufficient fail-first evidence.
- A static string assertion is not a runtime migration/idempotency proof.
- Mocks that bypass the real integration path cannot prove cross-service behavior.
- If direct reproduction is destructive or unsafe, use a reversible isolated harness and document the limitation.
- Required negative, positive, forbidden-role, failure-path, retry/idempotency, compatibility, and log-redaction cases must be explicit.

## 13. Validation and CI law

Use commands discovered from repository configuration. At minimum, where applicable:

- targeted unit/integration regressions;
- full applicable non-watch tests;
- lint;
- typecheck;
- production build/package validation;
- migration tests against a production-like schema;
- concurrency/failure injection;
- container startup/readiness;
- cross-repository contract tests.

Inspect every CI job. If CI only builds and deploys, record tests as local—not CI-certified.

## 14. Deployment and immutable provenance law

For each changed deployed service, record:

```text
commit SHA → workflow run and individual jobs → registry image/tag → immutable digest → task definition revision → running task digest → health/live probes
```

- Resolve mutable tags to running digests.
- Do not claim immutable provenance unless the commit-to-digest mapping is independently reconstructable.
- Verify service stability, ALB/target/container health, relevant logs, metrics, and alarms.
- Verify migrations in the ledger/catalog and direct DB effects; do not infer DB success from service startup logs alone when direct access is required by the prompt.
- Verify provider/destination outcomes where applicable.

## 15. Migration, data, privacy, and policy law

Queue approval alone never authorizes destructive or irreversible DB changes.

- Inspect live schema/history before migration.
- Test up/down or define a safe forward-fix when down is destructive.
- Prove lock/transaction/partition/idempotency compatibility.
- Do not invent retention, deletion, consent, identity, finance, commerce, moderation, or provider policy.
- Keep unknown policy-dependent criteria explicitly blocked with owner.
- Never expose secrets, tokens, PII, private content, or raw customer payloads in output/evidence.

## 16. Rollback law

Before push/deploy, define:

- source revert commits;
- migration down versus forward-fix;
- prior immutable image digest and task definition;
- configuration/secret-reference rollback;
- compatibility and data-repair/replay implications;
- rollback triggers;
- post-rollback probes.

A rollback gate passes only when the prompt-required safe exercise/evidence exists. “Documented and executable” is not the same as exercised.

## 17. Evidence-manifest law

Each named session writes a separate evidence file containing:

1. executive verdict and evidence ladder;
2. both five-line openers;
3. repository/branch/dirty-work provenance;
4. steering contradictions;
5. source and live finding matrices;
6. pre-existing work classification;
7. changed files and migrations;
8. true fail-first red/green evidence;
9. exact local commands/results;
10. commits/remotes;
11. CI runs and individual jobs;
12. artifact digest/task/health chain;
13. direct DB/provider/live-safe verification;
14. observability/audit evidence;
15. rollback plan and exercise;
16. blockers/residual risks/owners;
17. historical supersessions;
18. completion checklist mapped one-to-one to the controlling prompt.

Evidence headers and endings must agree. Do not self-author a smaller checklist and call all original prompt gates passed.

## 18. Independent result-acceptance law

An execution session’s final status is a proposal until an independent Brain/QA session verifies it.

Independent QA must re-check:

- exact prompt requirements;
- source/diffs/commits/remotes;
- local-versus-CI distinction;
- workflow jobs;
- deployed digest/task/health;
- DB/provider/live-safe results;
- audit/observability;
- rollback evidence;
- repository isolation and frozen-pipeline compliance.

If any controlling completion criterion is absent, downgrade to `FINISHED — INCOMPLETE` even when the execution session reports COMPLETE. Preserve the original report and append the QA correction.

## 19. Canonical status law

Use only:

- `OPEN/PENDING` — not fired;
- `IN FLIGHT` — issued and authorized/underway;
- `FINISHED — COMPLETE` — every controlling source, test/CI, migration, deployment, DB/provider/live, QA, audit/observability, and rollback criterion passed;
- `FINISHED — INCOMPLETE` — execution ended with any missing criterion; blocker, evidence state, owner, and next action required;
- `BLOCKED` — cannot proceed because of approval, dependency, or access.

Never use generic `done`. Operational words such as “waiting” do not replace canonical status.

At completion, a session updates only its own proposed status. Independent Brain QA makes the durable accepted status in Brain and canonical queue.

## 20. Brain and queue continuity law

After firing, result receipt, QA correction, or completion:

- update the latest Brain and canonical queue together;
- preserve prompt path, evidence path, commits, runs, digests/tasks, reached evidence state, missing gates, owners, and next action;
- append corrections; do not erase historical claims;
- keep future sessions `OPEN/PENDING` until explicitly fired;
- do not fire a new session merely because a slot opens—re-evaluate dependencies and repository overlap first;
- preserve an exact re-entry procedure so no future Brain depends on chat history.

## 21. Final prompt quality checklist

Before issuing any future execution prompt, independently verify:

- [ ] standalone complete body, not a launcher;
- [ ] permanent ID/name/status/environment/evidence path;
- [ ] five-line opener;
- [ ] purpose and measurable achievements;
- [ ] actual Brain/canonical/spec/steering paths;
- [ ] truthful required competencies;
- [ ] source-first finding gate;
- [ ] writable/read-only/forbidden boundaries;
- [ ] dirty-work/frozen-pipeline/Git laws;
- [ ] fail-first runtime regression matrix;
- [ ] local validation and individual CI jobs;
- [ ] immutable commit/digest/task chain;
- [ ] direct DB/provider/live-safe QA;
- [ ] logs/metrics/alarms/audit;
- [ ] rollback defined and exercised as required;
- [ ] separate evidence manifest;
- [ ] own-session status mutation only;
- [ ] independent QA acceptance;
- [ ] stop conditions and no production mutation;
- [ ] Brain/canonical re-entry continuity.

If any unchecked item applies, the prompt is not 10/10 and must be strengthened before firing.

## 22. Founder communication law

Ramzi prefers concise, results-first communication.

- Lead with the decision, status, or output.
- Use short direct sentences and compact bullets/tables only when they improve clarity.
- Do not repeat known context, narrate long plans, or add optional explanation by default.
- If blocked, ask one precise question and state the consequence.
- Expand only when Ramzi asks to `elaborate` or when a risk cannot be understood safely without detail.
- Keep execution prompts and evidence complete; concise presentation must never remove a required gate, proof, owner, or warning.

## 23. Novel tool, permission, service, and cost approval law

No execution prompt or agent may silently activate, invoke, provision, subscribe to, or expand access for a novel agent, Kiro Power, MCP server, managed AI/DevOps service, cloud service, marketplace product, paid API, recurring resource, secret, storage system, observability product, or permission set merely because it could help complete the task.

Before first use or provisioning, the prompt and executing agent must:

1. prove the capability is not already available through the approved local tools, existing AWS CLI/API access, current repository code, or already-running infrastructure;
2. state the exact tool/service/resource, why it is necessary, what data it receives, whether project code/secrets/customer data leave the approved boundary, and the cheaper/no-cost alternative considered;
3. state the expected one-time and recurring cost or explicitly say `COST UNKNOWN — BLOCKED`; never describe a paid service as free without current evidence;
4. state the requested IAM action/permission delta, duration, resource scope, environment, owner, tags, budget/alarm plan, expiration, and cleanup/rollback procedure;
5. obtain Ramzi’s explicit current-session approval naming that tool/service/resource and cost/permission scope before use. Firing a broad execution prompt or queue does not supply this approval unless the prompt itself names the exact cost-bearing action and Ramzi approves it;
6. use the least-privilege existing `kiro-ai` access first. An `AccessDenied` is evidence to request a narrowly named permission from Ramzi; it is never authority to attach policies, create inline policies, pass broader roles, or grant itself access;
7. prefer direct deterministic inspection over autonomous paid agents. Activating documentation for a Kiro Power is not permission to invoke its paid/backend tools;
8. record every created/changed billable resource, API/service invocation, permission change, expected cost, actual owner, and deletion date in the evidence manifest and Brain/queue;
9. after validation, immediately remove or stop resources that are no longer required, but never delete a resource whose runtime/security/rollback dependency is unproven. If retention is required, record the reason and scheduled review date;
10. stop with `BLOCKED` rather than hallucinating approval, architecture, ownership, cost, permission, durability, or cleanup.

Independent QA must inspect CloudTrail/audit history, resource inventory, IAM deltas, billing visibility where already enabled, and cleanup evidence. A technically useful result is still `FINISHED — INCOMPLETE` if an unapproved novel paid service, permission expansion, unexplained recurring resource, or missing cleanup gate occurred.

Every future prompt’s identity header or non-negotiable laws must include a `TOOLS / PERMISSIONS / COST` declaration listing approved novel tools and resources or `NONE`; its pre-edit reality check must inventory existing cost-bearing resources; its stop conditions must block unapproved activation/provisioning; and its final checklist must prove cleanup or justified retention.

## 24. Available capability utilization and blocker-proof law

Every execution and independent-QA prompt must assume the Kiro session can inspect the full local workspace and use the machine’s already-installed deterministic tools unless current preflight proves otherwise. Capability must be discovered and exercised; it must never be guessed absent.

### Default available capabilities

Subject to the prompt’s safety and cost boundaries, sessions must inventory and use where relevant:

- read access to every Tawadoo repository, specification, history, diff, worktree, generated artifact, and local document under `/Users/ramzihannachi/Code`;
- the local macOS/zsh environment, package managers, compilers, linters, test runners, containers, Git/GitHub CLI, AWS CLI, database clients, HTTP clients, and repository scripts already installed;
- Playwright and installed Chromium, WebKit, and Firefox-compatible browser capabilities for real browser, cookie, session, responsive, accessibility, and cross-browser QA;
- current authorized staging AWS, GitHub, database, provider, logs, metrics, alarms, registry, task, secret-reference, and deployment inspection paths;
- safe synthetic staging probes, production-handler integration tests, isolated production-like schemas/harnesses, and prompt-authorized rollback/forward-recovery exercises.

This inventory is capability context, not blanket mutation authority. All repositories other than the prompt’s named writable repositories are read-only by default. Secrets may be consumed through approved references for testing but never printed, copied into evidence, command text, logs, screenshots, or source. Production mutation and paid/novel tools remain governed by Sections 10, 11, 15, and 23.

### Required behavior before declaring a blocker

1. Inspect the actual installed tool/package/browser/CLI and record its version or concrete absence.
2. Attempt the safe authorized operation. For browser-required behavior, run the relevant Chromium and WebKit flow when installed; “requires browser” is not a blocker.
3. For AWS/GitHub/DB/provider access, try the least-privilege read or prompt-authorized staging path without exposing values. Record the exact sanitized `AccessDenied`, network timeout, private-network boundary, missing secret reference, or unavailable role if it fails.
4. Use all repositories read-only to trace contracts and coordination requirements. “Needs another repository” is not permission to skip source analysis or contract testing.
5. If the correct fix needs another writable repository, identify exact files, owner, order, compatibility, tests, deployment, rollback, and overlap. Either the prompt must explicitly authorize a serialized multi-repository session before launch or the criterion remains incomplete; do not edit across an unapproved boundary.
6. If a safe staging rollback is required and explicitly authorized, plan a maintenance-safe reversible exercise and execute it. “Would disrupt staging” is not a waiver unless a currently active dependency or quantified safety risk makes the exercise unsafe; record that exact conflict and owner.
7. A large pre-existing lint/test baseline is not a waiver. Run the full command and a non-suppressive changed/runtime-critical scope; all session-owned files must pass, and CI must enforce the accepted scope.
8. A static assertion, copied production logic, indirect health inference, or source comment cannot replace an available runtime/browser/DB/provider test.
9. Evidence must distinguish `CAPABILITY AVAILABLE AND USED`, `CAPABILITY AVAILABLE BUT FAILED` with exact sanitized proof, `CAPABILITY REQUIRES NEW COST/PERMISSION — BLOCKED`, and `CAPABILITY NOT INSTALLED/NOT ACCESSIBLE — PROVEN`.
10. Independent QA must rerun the highest-risk available checks rather than accepting an execution report’s statement that testing was impossible.

### Prompt requirement

Every future prompt must include an `AVAILABLE CAPABILITIES / REQUIRED USE` declaration naming relevant repositories as writable or read-only; installed browser/runtime/test capabilities to verify; AWS/GitHub/DB/provider paths to attempt; safe staging and rollback authority; and exact conditions that constitute a proven blocker. The prompt must not say merely “use available tools.”

If prompt authors knowingly omit a required available capability, or an execution session reports a capability blocker without attempted sanitized proof, the affected completion gate fails and the result is `FINISHED — INCOMPLETE`.

## 25. Mandatory Kiro skill propagation law

Reusable workspace skills are execution controls, not decorative labels. Every future Tawadoo Brain, queue entry, named prompt, independent-QA prompt, and `KIRO_PROMPT_NEXT.md` must select and require the applicable installed skills below:

- `tawadoo-source-truth` — mandatory for every named implementation, investigation, prompt-authoring, and independent-QA session;
- `tawadoo-data-sovereignty` — mandatory for database, migration, idempotency, outbox, identity, money, commerce, moderation, messaging, analytics, or other durable facts;
- `tawadoo-runtime-provider-qa` — mandatory for browser, HTTP, session, DB, queue, staging, callback, Amplitude, Firebase, or other provider/live acceptance;
- `tawadoo-secure-release` — mandatory for auth/security, secrets, IAM, cost, CI, AWS, deployment, observability, rollback, production readiness, and go-live;
- `tawadoo-refactor-hygiene` — mandatory for refactoring, repository cleanup, dead-code removal, dependency consolidation, or AI-fingerprint cleanup, but only after behavior/security/data truth is protected.

### Prompt requirements

Every prompt must include a `REQUIRED KIRO SKILLS` declaration with, for each applicable skill:

1. exact skill name and exact workspace path under `/Users/ramzihannachi/Code/.kiro/skills/<name>/SKILL.md`;
2. why it applies and which controlling gates it owns;
3. activation method: direct `/<skill-name>` invocation where one skill is primary, or explicit discovery/activation proof for every named skill when several apply;
4. a preflight gate confirming exact uppercase `SKILL.md`, valid frontmatter, folder/name equality, current-session discoverability, and selected-agent availability;
5. skill-specific completion checks mapped into the evidence manifest.

The execution agent must record `SKILL DISCOVERY`, `SKILL ACTIVATION`, and `SKILL COMPLETION` in evidence. Existence on disk is only source evidence. Automatic relevance matching, a prompt mentioning a name, or a skill appearing in the `/` list does not by itself prove its workflow was followed.

### Non-regression and safety

- Never invent, rename, or claim an unavailable skill. If a required skill is not discoverable in the current selected agent/session, record the attempted proof and stop as `BLOCKED` until a new session or corrected resource declaration makes it available.
- New skill files take effect in a new Kiro chat; prompt authors must not claim same-session registration without direct discovery/invocation proof.
- Custom agents must explicitly include the required `skill://.kiro/skills/<name>/SKILL.md` resources and validate them after selecting that agent; do not assume inherited defaults.
- Skills do not grant tools, repository writes, provider access, IAM, cost approval, production authority, or permission to expose secrets. Sections 9–16 and 23–24 remain controlling.
- Use the minimal applicable set, but omission of an applicable installed skill fails prompt quality and independent acceptance.
- Skill instructions may strengthen this law but may not abbreviate the mandatory standalone prompt structure or replace direct source/runtime/CI/DB/provider/live evidence.

Every final prompt-quality checklist must therefore prove: exact applicable skills named; discovery and activation gate present; skill-owned completion checks present; unavailable skills truthfully blocked; and no skill used as authority for novel cost, permission, cross-repository mutation, or production action.

## 26. Founder-operated, agent-executed work law

Ramzi is the non-technical founder and decision owner. He must never be assigned terminal commands, code edits, configuration work, dashboard inspection, database queries, provider verification, deployment steps, rollback execution, evidence collection, or technical troubleshooting.

- Kiro and its authorized execution sessions perform all technical discovery, implementation, testing, browser/provider/AWS/DB verification, deployment, rollback, cleanup, and durable documentation end to end.
- “Run alone,” “serialized,” or “founder-approved session” means the automated session must not overlap another risky session; it never means Ramzi performs the work.
- Ask Ramzi only for decisions that cannot be derived safely: a concise plain-language `APPROVE / REJECT / CHANGE` choice covering business policy, production mutation, destructive action, novel cost, permission expansion, or material risk.
- Every approval request must state what Kiro will do, customer/system impact, maximum expected cost, reversible rollback, and the consequence of rejection without technical jargon.
- Do not ask Ramzi to open a console, run a command, inspect a dashboard, retrieve a secret, copy a token, test a browser, or diagnose an error. Use available capabilities first; if access truly requires founder identity, provide one minimal guided confirmation step while Kiro performs all surrounding work.
- Preserve context in the Brain, canonical queue, evidence, prompts, and skills so Ramzi never has to reconstruct technical history or remind future sessions what was already proven.
- Reports must lead directly to named repair ownership and an executable next step. Do not present incomplete findings as a dead end or ask Ramzi to design the remediation.

## 27. Ramzi_V2 near-future production authority law

Ramzi explicitly designates each repository’s verified `Ramzi_V2` line as Tawadoo’s near-future production truth.

- Current deployed production and `main` remain operational evidence for compatibility, rollback, traffic, data shape, cost, and incident learning. They do not override safer, source-verified `Ramzi_V2` architecture merely because legacy production currently works.
- Every implementation, QA, prompt, Brain update, queue decision, and production-readiness review must treat verified `Ramzi_V2` source plus accepted staging evidence as the target architecture intended to ship.
- Dirty or uncommitted files are not automatically accepted truth. Preserve them, identify ownership and completeness, and integrate only after source review, validation, commit, CI, staging, rollback, and independent QA.
- Security, migration, data, identity, money, moderation, messaging, and provider defects found in legacy production must be repaired in `Ramzi_V2` and proven in staging before any separate production approval.
- Temporary security branches or clean worktrees may isolate implementation from unrelated dirty work, but they are candidates derived from `Ramzi_V2`, never competing product truth. Their accepted commits must return to `Ramzi_V2` before production readiness can be claimed.
- Never backport an unsafe production shortcut into `Ramzi_V2` solely for behavioral parity. Preserve required business compatibility while replacing excessive privilege, plaintext secrets, mutable provenance, runtime DDL, and other historical debt with the approved future model.
- Production remains immutable unless Ramzi separately approves the exact production action, risk, window, cost, and rollback. Design authority does not grant production mutation authority.

---

**Permanent rule:** retain this structure, strengthen it with domain-specific requirements, and never regress to generic prompts, hidden context, launcher-only files, reduced evidence checklists, self-certified completion, unapproved autonomous tools, silent permission expansion, unowned recurring cost, invented access limitations, untested available capabilities, unactivated named skills, or founder-assigned technical work.

## 28. `Ramzi_V2` single-source-of-truth law

**Founder ruling, 2026-08-22:** `Ramzi_V2` is the **only** complete and authoritative line in every Tawadoo repository. It must be ahead of, or equal to, every other branch, worktree, subbranch, and deployed artifact. Nothing may be more complete than `Ramzi_V2` for longer than one session.

This law exists because a real regression trap was found: in `admin_bo_tawadoo`, the deployed staging artifact was built from a commit that was **not** an ancestor of `Ramzi_V2`. Any session branching from `Ramzi_V2` — the normal, correct instinct — would have silently reverted a completed security fix. The drift was invisible until directly measured.

### 28.1 Mandatory preflight gate in every execution prompt

Before the first edit, every session must:

1. record `Ramzi_V2`'s current SHA for each writable repository;
2. record the commit the **deployed** artifact was built from;
3. prove the deployed commit **is an ancestor** of `Ramzi_V2`;
4. enumerate every local and remote branch that is **ahead** of `Ramzi_V2`.

If the deployed commit is not an ancestor of `Ramzi_V2`, the session must **stop as `BLOCKED`** and report the exact drift, unless its controlling prompt explicitly authorizes branching from the deployed commit and names the required integration follow-up. Branching from a stale `Ramzi_V2` is a regression, not a starting point.

### 28.2 Mandatory integration duty

A session that produces an accepted commit has **not finished** until that commit is integrated into `Ramzi_V2`.

- Integration is a named, reviewed step with its own validation, not an afterthought.
- A session may propose `FINISHED — COMPLETE` only if its commits are in `Ramzi_V2`, **or** its controlling prompt explicitly defers integration and names the owner and the follow-up session.
- Independent QA must verify integration state and downgrade to `FINISHED — INCOMPLETE` when commits remain outside `Ramzi_V2` without an explicit deferral.
- Never force-push `Ramzi_V2`. Never rewrite its history. Integrate by merge or reviewed fast-forward only.

### 28.3 Distinguish drift types before acting

Not all drift is equal, and a session must classify it rather than merging blindly:

- **Session drift** — a security or feature branch this programme created that is ahead of `Ramzi_V2`. Integrate it.
- **Legacy production drift** — `main`, or a hotfix branch, ahead of `Ramzi_V2` because production received fixes that never reached the future line. This requires a **founder architecture decision** about which commits belong in `Ramzi_V2`. A session must **not** silently merge these. Report the count and the subjects, and stop.
- **Abandoned drift** — stale feature or stash branches. Report; never delete without explicit approval.

### 28.4 Uncommitted work is not truth, and is not disposable

Dirty working trees are neither authoritative nor discardable. Preserve them, classify them per Section 9, and never reset, clean, stash, or overwrite them to make an integration succeed. A repository with a large dirty tree must be integrated in an isolated worktree, leaving the dirty tree untouched.

### 28.5 Cutover prerequisite

Production is built from the `Ramzi_V2` line. Any commit absent from `Ramzi_V2` **will not ship**. Therefore no production cutover may be approved while an accepted security or correctness commit sits outside `Ramzi_V2`. This is a hard gate, not hygiene.

### 28.6 Prompt and documentation requirements

Every future prompt must carry a `RAMZI_V2 INTEGRATION` declaration stating: the expected `Ramzi_V2` SHA per writable repository; whether the session integrates or defers; and, if it defers, the exact owner and follow-up session. Every Brain and queue update must record integration state alongside status, so no future session has to rediscover drift by measurement.


## 29. Velocity and cutover-deferral law

**Authority:** Founder ruling, 2026-08-22.

### 29.1 Complete the queue, defer the polish

The objective is to finish all 17 batches and the full canonical queue cleanly and completely. Any item that is not urgent for functional correctness, security of the staging environment, or queue progression — including alarm tuning, digest pinning of non-critical services, retained-credential rotation, `DEV_MODE` toggling, fleet-wide observability expansion, and similar infrastructure polish — is **deferred to a pre-production cutover checklist** rather than blocking batch progression.

Staging is Ramzi's personal build-and-test environment. It is not shared with customers. Hardening it beyond what the queue requires is wasted time until the queue is done.

### 29.2 Session sizing and parallelism

- Sessions should be **reasonably large** so each one moves the queue meaningfully. Do not split naturally cohesive work into many tiny sessions.
- Fire **2–3 repository-isolated sessions at a time** to maximize throughput without deployment overlap.
- After each wave completes, Brain performs independent QA **from source and live evidence, never from documents alone**, then immediately writes and fires the next 2–3 prompts.

### 29.3 Independent QA from source, never from documents

Independent QA must reconstruct truth from:
- actual Git commits, diffs, and branches;
- actual CI runs and individual job results;
- actual deployed digests and running task health;
- actual direct DB/provider state where authorized;
- actual browser/HTTP/session probes.

A prior execution evidence file, Brain section, QA report, or any other document is a **hypothesis to verify**, not a fact to accept. The law is: verify from the real system, then compare against claims. Never the reverse.

### 29.4 Prompt quality for speed

Every prompt must still be 10/10 per the mandatory structure (Sections 1–28). Speed comes from:
- larger scoped sessions (fewer handoffs);
- parallel repo-isolated execution;
- immediate QA turnaround;
- deferring non-blocking items to cutover;
- never repeating work that was already proven from source.

Speed does not come from weakening gates, skipping evidence, or shrinking prompts.


## 30. User-facing content approval law

**Authority:** Founder ruling, 2026-08-22.

### 30.1 Absolute rule

No execution session may ship, commit, or deploy **any user-facing text, message, label, notification copy, pre-filled message, email template, WhatsApp template, error message, toast, modal copy, button label, placeholder, CTA, tooltip, banner text, or any string a user will read** without Ramzi's explicit prior approval of the exact final wording in all three languages (French, Arabic, English).

### 30.2 Prompt-level inclusion

All user-facing copy must be **defined and approved inside the execution prompt itself**, before the session is fired. The session must not stop mid-flight to ask for copy — it must already have the approved text embedded in the prompt's outcomes section.

**Procedure:**
1. The Brain (prompt author) proposes all user-facing strings in FR/AR/EN as part of the prompt draft.
2. Ramzi approves, corrects, or rewrites them.
3. Only the approved strings go into the final prompt.
4. The execution session uses those exact strings — no invention, no placeholder, no AI-generated alternative.

### 30.3 What counts as user-facing

- UI labels, buttons, CTAs, headings, descriptions, empty states, error messages
- Pre-filled messages (WhatsApp, email, SMS, push, in-app notification)
- Email/WhatsApp/FCM template text
- Toast messages, confirmation dialogs, modal copy
- SEO meta titles and descriptions
- Onboarding text, tooltip text, placeholder text
- Any string that appears in `public/locales/*.json` or notification/email templates

### 30.4 What does NOT require approval

- Internal log messages, error codes, developer-facing strings
- Variable names, CSS classes, comments in code
- Purely structural/technical configuration

### 30.5 Non-regression

A session that introduces user-facing text not present in its controlling prompt is in violation. Independent QA must check that every deployed user-facing string matches the prompt-approved copy exactly. A mismatch fails the evidence-integrity gate.


## 31. Strong preparation law — no hesitation, no back-and-forth during execution

**Authority:** Founder ruling, 2026-08-22. Based on observed pattern: sessions waste hours going "actually it's this, oh no it's that, oh this isn't what I thought" — all of which is weak preparation disguised as execution.

### 31.1 The rule

An execution session must **never hesitate, backtrack, or discover basic facts about the codebase during implementation**. By the time a prompt fires, the session must already know:

- every file it will touch and why;
- every caller, consumer, and downstream dependency of those files;
- the exact current behavior (from source, not from documents or assumptions);
- the exact target behavior;
- the exact technical approach (library, pattern, API, data model);
- the exact infrastructure it will use (AWS services, ECS tasks, DB access patterns);
- what can go wrong and how to handle it.

### 31.2 Where preparation happens

Preparation is the **Brain's job** (prompt authoring phase), not the execution session's job. The Brain:

1. Reads the actual source code of every relevant file.
2. Traces all callers, emitters, middleware, and consumers.
3. Verifies the current behavior against the claimed gaps.
4. Identifies the exact technical solution (not "fix the thing" but "change line X of file Y to use pattern Z").
5. Identifies potential obstacles (missing packages, access limitations, schema differences).
6. Embeds all of this into the prompt so the execution session has zero ambiguity.

### 31.3 What the execution session does

The execution session:
- Prints the five-line opener.
- Runs its preflight reality check (confirming what the prompt says is still true).
- Implements the defined changes with confidence.
- Validates.
- Deploys.
- Proves.

It does NOT spend time "exploring the codebase" or "understanding the architecture" or "trying different approaches." That exploration was already done. If the prompt is correctly authored, the session executes linearly from start to finish.

### 31.4 Signs of a weak prompt (must never ship)

- The session would need to `grep` or `find` to discover where things are.
- The session would need to read more than 2–3 files to understand what to do.
- The session might choose between multiple implementation approaches.
- The session might discover that a claimed gap doesn't exist.
- The session might hit an infrastructure limitation nobody anticipated.
- The session includes vague language like "investigate," "explore," or "determine the best approach."

All of these mean the **Brain did not prepare enough**. The prompt must be rewritten before firing.

### 31.5 Prompt quality gate

Before issuing any prompt, the Brain must answer YES to all of these:

1. Do I know the exact files that will be modified? **List them.**
2. Do I know the current content of those files? **I read them.**
3. Do I know every caller/consumer that will be affected? **Traced them.**
4. Is the technical approach specific enough that there is only ONE way to interpret it? **Yes.**
5. Are all obstacles pre-identified and mitigated in the prompt? **Yes.**
6. Could a competent developer execute this prompt without asking a single question? **Yes.**

If any answer is NO, the prompt needs more preparation work before firing.

### 31.6 Non-regression

Independent QA must flag any session that spent significant time discovering facts that should have been in the prompt. This is a prompt-authoring defect, not an execution defect. It goes back to the Brain for process improvement, not to the execution session for blame.


## 32. Definition of COMPLETE — end-to-end live in staging

**Authority:** Founder ruling, 2026-08-23.

### 32.1 The rule

`FINISHED — COMPLETE` means the feature is **fully deployed, running, tested, and usable in staging end-to-end**. Specifically ALL of:

- Backend API code merged to `Ramzi_V2`, deployed to ECS, running, healthy, live-probed.
- Frontend code (if the feature has UI) merged to `Ramzi_V2`, deployed, visually working in the browser.
- Database migrations applied, tables/columns exist, data flows correctly.
- BO admin (if affected) deployed and functional.
- GitHub branches clean — no dangling unmerged work.
- AWS infrastructure correct — task definitions, secrets, alarms as needed.
- QA'd and tested: the feature works as a user would use it. Not just "the endpoint returns 200" but "a buyer can actually contact a seller via WhatsApp and the tracking records it."

### 32.2 What is NOT complete

- "API done, frontend pending" → INCOMPLETE.
- "Code merged but not deployed" → INCOMPLETE.
- "Deployed but untested" → INCOMPLETE.
- "Works in one repo but the other repo hasn't been updated" → INCOMPLETE.
- "The endpoint exists but no UI calls it yet" → INCOMPLETE.

### 32.3 Consequence for batch closure

A batch is COMPLETE only when **every user-facing flow it defines works end-to-end in staging**. If a batch needs both API + Web work, both must land and be verified together before the batch is closed.

### 32.4 Session sizing implication

Per Law §29.2, sessions should be reasonably large. If a batch naturally spans API + Web, the prompt should either:
- include BOTH repositories as writable (if no deployment overlap risk), OR
- be split into two sequential sessions (API first → Web second) with the batch remaining INCOMPLETE until both land.

The Brain must track which sessions a batch needs and not close the batch until all are live.


## 33. Regression guard law — every fix must prevent its own return

**Authority:** Founder ruling, 2026-08-23. Based on observed pattern: bugs get fixed, then come back weeks later because nothing enforces the fix permanently.

### 33.1 The rule

Every bug fix — especially UI/UX fixes — MUST include a **permanent automated regression guard** that will FAIL if the bug ever returns. No fix is complete without its guard.

### 33.2 What counts as a regression guard

- A **Playwright E2E test** that reproduces the exact user-visible behavior (e.g., "language dropdown is fully visible and clickable above all other elements on the search page").
- A **unit/component test** for the specific condition (e.g., "dropdown z-index is higher than header z-index").
- A **CSS/style assertion** in the test suite (e.g., computed style check that a popover's z-index > parent container's z-index).
- A **CI gate** that blocks deployment if the test fails.

### 33.3 What does NOT count

- A code comment saying "don't change this."
- A steering note without enforcement.
- A manual QA pass.
- "It works now" without a test proving it will keep working.

### 33.4 Known-fragile areas registry

Every repository with a frontend MUST maintain a **known-fragile areas file** at `.kiro/steering/known-regressions.md` that lists:
- What broke before
- When it was fixed
- What commit fixed it
- What test guards it now
- What causes it to regress (the root pattern)

Every execution session that touches those areas MUST run the regression tests for them AND check that the guards still pass after their changes.

### 33.5 Prompt requirement

Every future execution prompt that modifies frontend UI must include:
1. A `KNOWN FRAGILE AREAS` section listing which known regressions their changes might affect.
2. A `REGRESSION GUARDS ADDED` section in the completion checklist requiring the session to name the exact test files added to prevent recurrence.
3. A mandate to RUN all existing Playwright regression tests after their changes and report any failures.

### 33.6 The z-index/overflow class of bug specifically

This family of bugs (dropdowns hiding behind headers, modals clipping popovers, selects scrolling behind parent containers) has recurred 3+ times across different components. The root cause is always: a new CSS change introduces `overflow: hidden`, `position: relative` with implicit stacking context, or a lower z-index that clips a child portal/popover.

**Mandatory for all future frontend sessions:** after any CSS/layout change, run a z-index sweep test that opens every dropdown/popover/modal on the affected page and verifies it renders above all other content. This is NOT optional.


---

# CONSOLIDATED LAWS §34–§51 + THE THREE COMMANDMENTS (single-source consolidation)

**Consolidated by Brain B16, 2026-08-31.** These laws were earned in Brain sessions B12–B16 and previously lived only in handoff files, causing a durability gap (a session reading only this file would miss binding laws). They are reproduced here VERBATIM/tightly-paraphrased from their source handoffs so this file is the single canonical law source. **Nothing in §0–§33 above was altered.** All laws below are binding per §0 (Supreme Rule). Sources cited per law.

## THE THREE COMMANDMENTS (from `.kiro/steering/01-B14-CTO-STANDARD.md` — also always-included)

1. **CLASSIC VIEW IS SACRED.** The existing marketplace (search, listings, dashboard, orders, auth, payments, delivery) must NEVER be broken by Smart View work. Mirror it, don't touch it.
2. **EVERY INTERACTION FEEDS THE LAKE.** Every AI call, query, search, edit, conversion → sovereign DB FIRST, then mirrored to providers. This is the proprietary-model training pipeline. No interaction may bypass sovereignty.
3. **COST-FIRST, ALWAYS.** Cheapest model that works. Vector/keyword before LLM. Cache everything. Guest quotas, rate limits. Every prompt states cost implications.

## §34B — Strong preparation law (source: B12A retrospective)
Hidden failures (empty columns, dead workers, empty sitemap for a month) are born from weak preparation. Every investigation and prompt must establish current reality from source/live BEFORE conclusions. (Superset of §31; retained by number for cross-references.)

## §34C — Full-capability investigation law (source: BRAIN_B12B_CHECKPOINT lesson #61)
Every investigation needs: web search + provider docs + AWS inspection + design/BI skill. Not one source. (Aligns with §6 web-search mandate and §24 capability law.)

## §35 — DB Credential Separation Awareness (MANDATORY for all API/schema prompts) (source: BRAIN_B12B_CHECKPOINT, B11-SEC-DB-CREDENTIAL-C1)
Staging/prod use a split-credential model. The runtime DB user (e.g. `tw_runtime_b11`) CANNOT run DDL (ALTER/CREATE/DROP TABLE), CANNOT INSERT into `api_migrations`, CANNOT write tables owned by `prodmasteruser`. Schema changes require running as `prodmasteruser` via a bounded ECS task or direct DB access.
Every API/schema prompt MUST verify YES:
- [ ] Touches DB schema? If yes, references B11-SEC-DB-CREDENTIAL-C1 and §35.
- [ ] Verifies actual `synchronize` and `migrationsRun` values from source (`grep -A5 "synchronize\|migrationsRun" src/app.module.ts`).
- [ ] Verifies target table ownership (`SELECT tableowner FROM pg_tables WHERE tablename='X'`) and runtime-user DDL/`api_migrations` rights.
- [ ] Specifies the migration execution path (runtime startup vs bounded task vs manual).
A prompt that causes a staging crash-loop is a BLACK FLAG. NOTE: this must be reconciled against the current live reality — some services run `migrationsRun: true`; the session must confirm which path actually applies before schema work.

## §41 — FORBIDDEN FILES (source: B14→B15 handoff)
Every prompt must list the sacred files as FORBIDDEN: `src/app/[locale]/layout.tsx`, `AnalyticsTracker.tsx`, `AmplitudeInit.tsx`, `ThirdPartyScripts.tsx`, `next.config.js`, `Dockerfile`, `.github/workflows/*`. (Born from the S130 layout.tsx regression that killed Classic View. See §4B in 01-B14-CTO-STANDARD.)

## §42 — NEVER pin ECS task-def to `sha-` tags (source: B14→B15 handoff, S130/S131 disaster)
Task-defs use the mutable deploy tag (e.g. `staging-v2`); CI handles image updates. Never register task-defs manually with pinned SHA tags — it produces no-op deploys where code deployed ≠ code running.

## §43 — Smart View must mirror Classic behavior (source: B14→B15 handoff)
If Smart View exposes a feature, it must produce the SAME results as Classic — same API, same params, same events. (Reinforces Commandment 1 + Classic Mirror rule.)

## §44 — Deployed ≠ visible (source: B14→B15 handoff, S131 invisible mic)
After every deployment: verify the running image matches git HEAD, THEN get visual proof (Playwright screenshot or founder confirmation). Code deployed is not a feature visible.

## §45 — Every word from Ramzi is law (source: B14→B15 handoff)
Every sentence, attachment, screenshot, and remark in the conversation is a binding requirement. Read it, implement it, verify it. (Same principle as §51.)

## §46 — Cap planning, maximize execution (source: B14→B15 handoff)
Bounded research time, then ship. Do not spend hours on docs/research when the work is executable. (Aligns with §29 velocity + §31 preparation — prepare fully, then execute linearly without dithering.)

## §47 — Browser-verify every UI change (source: B15→B16 handoff)
Every UI session MUST load the relevant staging page in a real browser (Chromium + WebKit) BEFORE first commit and AFTER every commit. If you cannot browser-verify, report FINISHED — INCOMPLETE. curl is not eyes.

## §48 — No-collision naming / check before create (source: B15→B16 handoff)
Never create a file, name, or entity without first checking the filesystem/DB for existing names and patterns. Surgical preparation. (E.g. the `offer` entity already exists — do not recreate.)

## §49 — No guessing, no presuming, no assuming, no working from memory (source: B15→B16 handoff; Ramzi explicit)
Every claim verified from source + live. Every web search done before implementation. No matter how long the session. A report or memory is a hypothesis to verify, never a fact to accept.

## §50 — Silently fail to Tawadoo-only, never block users (source: B15→B16 handoff; Ramzi explicit)
If a channel/provider rejects or fails, the listing stays on Tawadoo + ChatGPT. Never block a user, never hurt an account, never disappoint. Safe defaults over hard failures.

## §51 — Every word Ramzi says is law for all future sessions (source: B15→B16 handoff; Ramzi explicit)
Not just instructions — binding requirements, including context, attachments, screenshots, and corrections provided in any conversation.

---

**Cross-reference note:** §45 and §51 both encode "every Ramzi word is law" (earned twice, in B14 and B15). §34B and §31 both encode strong preparation. These duplications are preserved intentionally so historical cross-references in prompts and evidence files resolve correctly. The canonical count is §0–§51 plus the Three Commandments.


## 52. Founder-decision-in-doubt + thoroughness law (founder-issued 2026-09-03)

**Authority:** Founder ruling. **Scope:** every Brain, session, and prompt. **Why it exists:** an S4 session framed a "solicited saved-search alert" as "marketing needing new consent" — an assumed BUSINESS/POLICY decision that was the founder's to make, and it risked unnecessary work. The founder is human, working heavily, and cannot catch every wrong assumption buried in a wall of text — exactly as an AI can hallucinate. The safeguard must live in how sessions BEHAVE, not in founder vigilance.

### 52.1 STOP and ask the founder when a BUSINESS/POLICY decision is in doubt
Before doing work that DEPENDS ON a business, policy, legal, product, user-facing, money, consent/marketing-vs-transactional, data-retention, or "what should the platform do" answer that is NOT unambiguously already decided by the founder or provable from source:
- **STOP and ask the founder a single, plain-language APPROVE / CHANGE / REJECT question BEFORE building on the assumption.** Do not assume the answer and proceed. Do not encode a policy interpretation as if it were a technical fact.
- Distinguish clearly: a TECHNICAL fact must be verified from source/live (§8, §49); a BUSINESS/POLICY decision must be surfaced to the founder, never self-decided.
- "In doubt" is the trigger — err toward asking. A 30-second founder decision is cheaper than a wrong build. This does NOT mean asking about technical minutiae the session can verify itself; it means never silently choosing a business/policy path.
- The Brain must PUSH sessions toward this: every prompt that touches a policy-dependent area must carry an explicit founder-decision STOP gate with the options pre-drafted (APPROVE/CHANGE/REJECT), so the founder decides fast without reconstructing context.

### 52.2 Thoroughness — get the ABSOLUTE truth before proposing
No finding, plan, or fix may be proposed on assumption. Every session must establish truth from BOTH:
- **The code, entirely (§8/§49):** trace the full path across all relevant repos + DB + BO + live — not a partial read. "I read one file" or "the audit said" is not truth; source + runtime is. Negatives ("there is no X") must be PROVEN, not assumed.
- **The web (§6), from the real authorities:** provider documentation and current best practices from the ACTUAL source (e.g. the email/deliverability/legal authorities, the payment/provider/platform docs, the OpenAI/OpenSearch/framework docs) — cited — for any area where external rules, provider behavior, legal requirements, or reputation/compliance apply. Do NOT rely on the model's memory or generic "best practice." Get the current, cited, authoritative truth.
- Where the two disagree with a prior audit/report/assumption, the SOURCE + AUTHORITATIVE-WEB truth wins; record the correction.

### 52.3 Present findings so the founder can't miss the decision
When surfacing to the founder: lead with the DECISION needed (not a wall of narrative), state the options as a short APPROVE/CHANGE/REJECT list with a one-line recommendation and the consequence of each, and clearly separate FACT (verified) from RECOMMENDATION (opinion) from UNKNOWN. The founder must be able to decide from the top of the message without reading everything. Verbosity that buries a decision is itself a violation.

### 52.4 Enforcement
Independent QA must flag as a defect any session that: built on an unsurfaced business/policy assumption; proposed a finding/fix without full source truth + (where applicable) authoritative web research; or buried a required founder decision in narrative. Every future prompt's structure must include, where applicable: a `FOUNDER DECISIONS (STOP)` block with pre-drafted options, a `SOURCE-TRUTH` requirement (full path, not partial), and a `WEB-RESEARCH (AUTHORITATIVE, CITED)` requirement for any externally-governed area. Omitting an applicable one fails prompt quality.
