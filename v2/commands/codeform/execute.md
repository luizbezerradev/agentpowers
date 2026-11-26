---
description: "Implements ONE sub-issue: reads it with gh issue view, follows its YAML contract to the letter with TDD (one test with the exact name of each contract line; MANDATORY rules, lint) and, when done, spawns the reviewer (and security, if the flag asks): fixes the simple, escalates the sensitive, records decisions on the sub-issue and posts the validations. With --lane, runs as a lane of /codeform:orchestrate."
argument-hint: "<sub-issue number> [--lane]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - Skill
  - Artifact
  - Agent
  - SendMessage
  - mcp__context7__resolve-library-id
  - mcp__context7__query-docs
---

<execution_context>
Before proceeding, read the methodology file below in full — principles and rules of this phase,
especially §0.11 (doubts), §0.12 (what is correct), §2 (records, contract, scope change, validations), §4 (Clean Code,
9 Object Calisthenics, YAGNI/KISS/DRY/Less Code Best Code), §5 (SDD/TDD, test types and scoped, sequential test runs),
§0.7 (delegation), §6.2 (tiers) and §7 (simple × sensitive findings):
`../../skills/codeform/methodology.md`

If the contract has `tela` lines, also read the **wireframe**: the artifact URL is in the "Wireframe" section of the
project context (methodology §2); read its `.html` source in full, or the artifact (`Artifact` with `action: "read"`)
if the source isn't available.
</execution_context>

<source_of_truth>
⚠️ **READ BEFORE ANYTHING ELSE — this block overrides any other instruction in this file, including
"follow the contract to the letter".** What is correct is defined in §0.12 of the methodology; doubts follow §0.11.
In this phase: implement the correct path; a non-critical gap or deviation of the contract is decided and recorded as
a decision comment on the sub-issue; a critical one stops for the user; a change in what was asked follows §2.3.
</source_of_truth>

<delegation>
⚠️ **The implementation is yours, in the main thread; the review belongs to new child agents** (§0.7; how: §0.10 of the methodology).

- **Implementation — never delegated.** Writing code and making design decisions at every line are judgment. If the
  volume seems too big for one session, **stop and tell the user** (a fix bigger than the sub-issue: methodology §2.3).
- **Review and security — new, clean-context, read-only children** in this same workspace (they see your local
  diff), spawned by you **together, in parallel, when the contract is done**
  (steps 11–17), following the playbooks `~/.claude/skills/codeform/playbooks/review.md` and `security.md`.
  They resolve their own doubts directly with the user and report to you when done (§0.10); **you** verify the findings, fix what is simple
  and take the rest to the user (§7 of the methodology).
- **As a lane** (`--lane`) you are an Orca worker and spawn your children the same way (§0.10, "Spawning from a worker").
</delegation>

<lane_mode>
🛤️ **`--lane`: you are a lane of `/codeform:orchestrate`** — an Orca worker in the lane's own Orca worktree
(rules in §6.3 of the methodology). Your prompt has a preamble with Task and Dispatch IDs: follow it, and load
`orca skills get orchestration` for the worker commands (`ask`, `worker_done`). The lane's worktree, lane branch, account,
model/effort, linked issues and isolated test resources (ports, environment name… the environment variables the orchestrator
passed, if the project requires them) have already been prepared — **don't change them**. What changes:

- **Shared state:** write/update `$(git rev-parse --git-common-dir)/codeform-lanes/<Initiative>/<sub-issue>.md`
  (outside the versioned tree) with the fields from §6.3. Update it when starting, at each checkpoint, when stopping for the
  user and when done — and mirror it on the workspace with `orca worktree set --worktree current --workspace-status <id> --comment "<short line>"`
  (skill `orca-cli`): `in-progress` when starting and at milestones; comment `awaiting decision: …`/`blocked: …` on
  `awaiting-decision`/`blocked`; `in-review` on `done`.
- **Human stop:** when you need the user (critical decision — methodology §0.11 —, sensitive/CRITICAL finding), mark `awaiting-decision`, send the decision
  to the orchestrator with `orca orchestration ask --question "#<sub-issue>: <decision, options with pros/cons, recommendation>" --options "<a,b,…>"`
  and **end your turn**; Orca wakes you with the user's answer (relayed by the orchestrator). If the user answers you directly
  in this chat, that answer counts. Never decide for the user.
- **Reporting the end:** when done, report with the `worker_done` from your preamble: `--outcome succeeded` only with the
  validations comment posted (Gate log included) and all gates ✅; otherwise `--outcome failed` with the blocker. Include SHAs, files modified and evidence.
- **Commits:** authorized by the orchestrator gate **only on this lane branch**. Commits in the format and
  granularity the project's `CLAUDE.md` declares (e.g. one per unit/service, conventional message),
  with the attribution trailer required there. **Never `git push`.**
- **Lint/typecheck only on the lane's units** — a lint with auto-fix on the whole repository reformats files
  of other lanes.
- **Tests with the cross-lane lock** (§5.4): every test, coverage or mutation command goes through the
  `codeform-lanes/test.lock` lock, so two lanes never test at the same time.
- **Scope:** only the files of your contract's `impacto.arquivos`. Needing a contention file the plan (§11.1) assigns to
  another lane is a **fork**: stop and ask for a decision.

Without `--lane`: **don't commit** unless the user asks.
</lane_mode>

<objective>
Implement the given sub-issue (`$ARGUMENTS`) following its contract to the letter, with TDD, and then submit it to
**review** (and, if applicable, **security**) by independent child agents. It is a **real project** — nothing may be
simulated or hardcoded.
</objective>

<frontend_wireframe>
🖼️ **All frontend in this sub-issue is implemented based on the wireframe** (artifact linked in the project
context, section "Wireframe").

- Structure, components, states and copy come from the wireframe — which was already built with the
  real frontend components. Reuse the same components and tokens it declares in its legend.
- Region marked `NEW` in the wireframe: the component is created in this sub-issue, following the design.
- **Divergence between the wireframe and what the implementation requires** (component doesn't behave as
  designed, missing state, new decision): resolve it by methodology §0.11 (a change in what the user sees that the
  issue didn't answer is a business rule → critical) and **update the artifact** before moving on.
- **If the frontend changes during execute for any reason, the artifact must be updated**:
  edit the wireframe's `.html` source (methodology §2, "Wireframe") only where it changed and republish **at the same URL** (follow the `<update_mode>` of the
  playbook `~/.claude/skills/codeform/playbooks/wireframe.md`, in your own thread or via a child agent, §0.10). A frontend sub-issue **doesn't
  close** with an outdated wireframe.
- Never implement from memory a screen the wireframe covers, and never let the wireframe fall behind the code.
</frontend_wireframe>

<mandatory>
The MANDATORY rules of **§3 of the methodology** are **non-negotiable** and apply in full in this phase, along with
§4 (Clean Code, 9 Object Calisthenics, YAGNI/KISS/DRY/Less Code Best Code, comments §4.3) and §5
(SDD/TDD, test types §5.3, test runs §5.4). They are **not** repeated here; read them there. What is
**specific to this phase**:

- **MANDATORY 4 (production data):** if the work changes the schema, format or location of existing data and the
  contract didn't make the migration explicit, **STOP** (critical, §0.11) before running anything destructive against
  production data.
- **MANDATORY 7:** execution only starts after the pre-reading (steps 1–2) and context7 (step 3).
- **MANDATORY 8:** Clean Code + Object Calisthenics apply from the first line written, not only
  in the review. In legacy code touched by the task: **improve what is changed, never make it worse.**
- **Test types (§5.3):** classified per contract line in step 4. Non-negotiable minimum: isolable logic →
  unit + **100% coverage** (line/branch covered by real assertions, not just execution); every sub-issue →
  scoped regression passing; critical business logic → **scoped mutation** on the changed files, with no
  surviving mutant; flow crossing real layers → integration without a mock at the validated point; critical/end-to-end
  flow → e2e.
- **Test runs (§5.4):** **only scoped** to what the task created or changed and **always sequential**
  (runner in serial mode, one command at a time; in `--lane`, with the cross-lane lock). **Never** run the full
  suite in this phase — it belongs to the `ci:check` of `/codeform:release`.
- **MANDATORY 10 (comments, §4.3):** a new file is **born** following the standard; **every file the task edits
  gets the comments of the whole file normalized** — the narrative of a changelog comment goes into the
  commit body, the ADR or a decision comment on the sub-issue before being deleted; a file the task did **not** touch: **don't
  edit it** — record path + density for the wrap-up (step 20).
</mandatory>

<process>
Follow the **Pair Programming Contract** (§0 of the methodology) in each step.

**Setup (once):**
1. **Read the sub-issue:** `gh issue view <s> --comments` — criteria checklist, `### Contract` (YAML) and the decision
   comments already there; then the parent issue (`gh issue view <n> --comments`) and the project context
   (methodology §2, "Locate"). No `### Contract`: **STOP** and tell the user to run `/codeform:plan`. Every sub-issue in
   `depends_on` must already be closed (in `--lane`, merged per the plan); otherwise **STOP** and say which.
2. **Mandatory pre-reading** (MANDATORY 7 + §0.1): the existing files in `impacto.arquivos`, the **wireframe** (if there
   are `tela` lines — `<frontend_wireframe>`) and the project's `CLAUDE.md`/`AGENTS.md` (§0.11, step 2). Report the current
   state citing `path:line`. **Don't assume what exists.** Also record:
   - `BASE_SHA` = `git rev-parse HEAD` (where the work starts from);
   - the **snapshot of what was already dirty** (`git status --porcelain`): those files are **not** part of the work
     and stay out of the diff the reviewers receive;
   - the **tier** and **risk flags** (`risco`).
3. **context7** for each lib/framework/SDK the contract touches.

**Loop per contract line** (`api` responses, `regras`, `tela`; hardest first, §5.1):
4. **Line plan:** for each line, the test it names and how it is proved. **Classify whether the logic is isolable**
   (§5.2) — that decides whether TDD (step 6) applies — and **which test types** it requires (§5.3 matrix).
   **Impact (§5.1):** confirm `impacto` against the code before coding; what is missing becomes a decision comment.
5. **Doubts:** methodology §0.11 — doubt order; critical stops for the user (in `--lane`, `awaiting-decision`); every
   decision is recorded as a comment on the sub-issue (`gh issue comment <s> -F <file>`). A change in what was asked:
   methodology §2.3.
6. **TDD when applicable** (MANDATORY 9 + §5.2): **🔴 Red** — write the test **with exactly the name the contract line
   gives** and run **only that test** (scoped and sequential, §5.4), **pasting the output that confirms the failure**.
   If the line falls into the §5.2 exception (external integration, UI, scaffolding), record why and go to step 7
   without skipping this decision silently.
7. **Implement surgically** (§0.5) applying Clean Code + Object Calisthenics + DRY (MANDATORY
   8): **🟢 Green** — the minimum code to satisfy the test from step 6 (or, if there was no TDD
   for this line, the direct implementation of the line); only what the contract asks for; existing style; no
   legacy (MANDATORY 4); no weak typing (MANDATORY 1); dead code you notice → mention it, don't
   delete it. **Before closing the file, normalize its comments in full** (MANDATORY 10):
   delete what doesn't fit the six categories — especially every changelog comment — and, if its reason
   isn't yet in the commit/ADR/issue, **move it there** before deleting.
8. **Prove** (§0.6 + MANDATORY 9 + §5.3 + §5.4): run the sub-issue's **scoped tests** — those
   related to the files it created or changed, never the full suite — in **sequential** mode, and
   lint/typecheck **only on the touched units**, **pasting both outputs** — clean lint is not evidence
   of a satisfied criterion, a passing test is. For isolable logic, also run **coverage**
   on the same scope and **paste the report**: 100% line/branch of the new code, with no unjustified
   exception. If the line involves critical business logic, run **mutation testing scoped** to the
   changed critical-logic files (§5.4, e.g. Stryker with `--mutate` and `--concurrency 1`) and paste the
   result — fix any surviving mutant at the source (a test that executes the line but doesn't validate the
   behavior), never suppress the mutant. If the line crosses real layers, confirm the integration test runs
   without a mock at the validated point; if it is a critical/end-to-end flow, confirm the e2e.
   **🔵 Refactor** — clean up the code (Object Calisthenics, DRY, KISS) without breaking green and run it
   again. Fix at the source until everything passes clean. **Only then** is the contract line done (its named test passing).
9. **Checkpoint:** when all lines are done, show the increment to the user (in `--lane`, also update the lane state — `<lane_mode>`). On the
    frontend, say whether the wireframe was updated (and what changed) or that the implementation matches it.

**Implementation wrap-up:**
10. Confirm every contract line has its named test passing and report what changed and the evidence — including, on
    the frontend, the list of wireframe updates (URL unchanged).

**Quality gates (review and security):**
11. **Decide which gates run.** Read the contract's **risk flags** (`risco`) and the **real diff**
    (`git diff --name-only $BASE_SHA` + `git ls-files --others --exclude-standard`, minus the snapshot from
    step 2) and apply the review and security triggers of **§1 of the methodology** (`/codeform:execute`):
    waiving the review requires a written reason; the flag can **promote** security, but **never waive** a
    surface the diff actually touches. With no risk surface in the diff, only the review runs — and say so.
    Announce the decision and the why to the user (one line each). Don't ask, **decide** — the decision is yours.
12. **Review — spawn the child** (§0.10; child-id `<sub-issue>-review`; read-only) **at the same time as security**
    (step 15), both on the same diff. Spec: *"Follow
    `~/.claude/skills/codeform/playbooks/review.md`. Sub-issue: #<s> (contract in its body; read it with
    `gh issue view <s> --comments`). Parent issue: #<n>. Files: `<list>`. Scoped test output: `<file>`. Wireframe (if frontend): `<URL and
    .html>`. Read-only. Return the report in the playbook's format."* Don't pass on your conclusions or
    what you think is right.
13. **Review triage** (after **both** reports — review and, if it ran, security; their doubts were resolved with the
    user, §0.10). Triage the review and security reports together (this step and step 16), in one fix loop. For each
    **blocker** finding: open the `path:line` and confirm; discard what doesn't hold, **saying what you
    discarded**. Classify **simple × sensitive** by the criteria of §7 (the reviewer's proposal is a
    suggestion, the classification is yours).
    - **Simple:** fix it now, surgically, show the diff, run lint/typecheck **and** the tests
      scoped to the fixed files (sequential, §5.4), **pasting the outputs** (§0.6).
    - **Sensitive:** present it to the user (§0.2): the problem, the options with pros/cons, the recommendation and
      the why, and **wait for the choice before coding**. In `--lane`, mark `awaiting-decision`.
14. **Re-review.** If there are fixes, **resume the same reviewer** (§0.10) restricted to the changed
    files. Round limit and the lint-or-test decision: methodology §7.
15. **Security** (if step 11 said so) — spawned **together with the review** (step 12), in parallel, on the same
    diff. Spawn a new child (§0.10; child-id `<sub-issue>-security`; read-only). Brief: *"Follow `~/.claude/skills/codeform/playbooks/security.md`. Sub-issue #<s> …
    Files: … Read-only. Return the D1 report and the coverage."*
16. **Security triage** (together with step 13, in the same fix loop). Verify **every** 🔴/🟠 finding by opening source and sink. Fix **at the sink**, without
    asking, only what §7 calls simple (local 🔵/🟡) — diff + scoped tests pasted (§5.4). **Every 🔴/🟠, and everything
    that touches auth, persistence, migration or contract**, goes to the user, **who decides to fix, accept
    or debate**. **Accepting a risk is their decision**, at any severity; never yours.
17. **Re-verification.** For each fix, retrace the source → sink path and show why the attack now
    fails and that legitimate behavior didn't change. Was there a fix? **One** new pass of the same security child
    (§0.10) restricted to the changed files.
18. **Gate log.** Gather for the validations comment (methodology §2.4): review (rounds, verdict) or waiver + reason;
    security (verdict, coverage) or "not applicable" + reason; findings and fixes applied; **accepted risks**
    (severity + the user's justification); and, for each blocker, whether a test or lint could have caught it (§7).
19. **Sync the sub-issue** (only with the Gate log complete — review ✅ or a justified waiver, security ✅ or "not
    applicable" —, **no open blocker** and every contract line done):
    - check `[x]` the criteria in the sub-issue body (`gh issue edit <s> -F <body with [x]>`);
    - comment the **validations** (methodology §2.4) with the test names and results taken from the real output;
    - close it: `gh issue close <s> --reason completed -c "Implemented and reviewed (review ✅, security <✅|n/a>) — see the Validations comment"`.
      A pending line or an unresolved accepted risk: **don't close it**, comment what is missing and tell the user;
    - **parent issue** (in `--lane`, **not** — `/codeform:orchestrate --merge` does it once): check `[x]` the parent's
      criteria fully covered by closed sub-issues (methodology §2, "Parent issue"). Never close the parent (the PR closes it);
    - failed (permission, not found)? Report which one; don't declare it synced without re-reading
      (`gh issue view <s> --json state,body`).

**Wrap-up:**
20. **Comments report** (MANDATORY 10): list (a) the files whose comments you normalized, with
    the density before → after; and (b) the files you **read but didn't touch** that are above 10%
    density — only `path` + `%`, without editing. Item (b) is input for the user to decide whether it becomes a task;
    do **not** turn it into a cleanup on your own.
21. **Final report** (§0.8): result; contract lines and criteria done; sub-issue closed/updated; **agents, models and
    effort used** (you, reviewer, security, with the rounds); evidence; decisions recorded and critical stops (§0.11);
    accepted risks; blockers and next step. In `--lane`: state `done` and SHAs in the lane file.
</process>

<guardrails>
- Surgical changes: touch only what is needed; match the project's style; remove only orphans you created.
- One sub-issue at a time, respecting `depends_on`.
- **Doubts and decisions:** methodology §0.11; scope: methodology §2.3; issue rule: methodology header.
- **The contract is not edited by hand** (methodology §2.2).
- **The reviewers are not you.** Don't pass them your conclusions, don't use them to "validate" what you
  already decided, and **never declare "review approved" without the child's report and your verification of it**.
- **Sensitive is never fixed on your own.** When in doubt between simple and sensitive, it is sensitive (§7).
- **No `git push`.** Commits only in `--lane` (on the lane branch) or if the user asks.
</guardrails>

<gate>
Contract implemented, quality gates closed, validations comment posted and **sub-issue closed** (step 19, re-read with
`gh issue view`):

- There is a next sub-issue in the plan: "Sub-issue done. Run **`/codeform:execute <next>`**" — or
  **`/codeform:orchestrate`** if the plan (§11.1) indicates a parallel block.
- It was the last one: "Last sub-issue done. Run **`/codeform:release`** — it validates the whole branch and opens
  the PR."

If there is an open sensitive or CRITICAL finding, or a critical stop: "Sub-issue blocked. Decide the items presented
above before moving on."
</gate>
