---
description: "Release the branch — validates the conventions the project declares (commits, PR title, branch as a warning) in the main thread, checks that the issues are synced, runs the ci:check of the touched units in mechanical child agents (build/lint one per unit, in parallel; full test suite one unit at a time, sequentially, listing each test name) and, if all ✅ and no warnings, opens the PR citing the issue and sub-issues, waits for the PR's CI to be green before handing it to human review, and posts the conclusion and validations comments on the issue. It is the only point in the cycle where the full test suite runs. Generic: discovers the rules and the CI recipe in the current repository."
argument-hint: "[--base <branch>] (comparison branch; default: the working base the project declares, or the repository's default branch)"
allowed-tools:
  - Read
  - Bash
  - Glob
  - Grep
  - Agent
---

<execution_context>
Before proceeding, read `../../skills/codeform/methodology.md` in full — especially §0 (Pair Programming
Contract) and §0.7 (delegation).

**This command knows nothing about the project by heart.** The rules and the CI recipe come from the current repository,
read on every run:
- `CLAUDE.md` (root and services), `CONTRIBUTING.md` and the docs they point to — commit and
  branch format, accepted types, PR title rule, merge method, working base branch, issue/board rules.
- `.github/workflows/` (or whatever CI the project uses) — the workflow that **validates conventions** (if any) and the
  real build/lint/test recipe of each unit, including the reusable workflows it inherits.
</execution_context>

<source_of_truth>
⚠️ **The source of truth is what the project declares and the CI that actually runs, not your memory of other
repositories.**

Architecture is not judged here: **conformance is audited** against a standard that already exists. Given that:

- If the conventions prose (`CLAUDE.md`/`CONTRIBUTING`) diverges from what the validation workflow **actually**
  checks, that is a **finding** (drift between documentation and enforcement) — report it, don't resolve it by silently picking one of the
  two.
- **A rule the project doesn't declare is not validated.** With no PR title rule, no "one unit per
  commit" rule or no branch convention, the corresponding item is skipped **and the report says it was skipped and
  why**. Don't import rules from another project.
- Don't invent build/lint/test commands. If the unit's CI doesn't define a step, use the default of the reusable
  workflow it calls; if that doesn't have the step either, **don't run anything in its place** and say so in the report.
- Repository without readable CI: discover the commands in the manifests (`package.json`, `Makefile`, `go.mod`,
  `pyproject.toml`…), **state the assumption** and ask for confirmation before running.

**Don't fix anything on your own — report it; the user decides** (§0.2 of the methodology).
</source_of_truth>

<delegation>
⚠️ **Child agents only for mechanical execution — never to judge** (§0.7; how: §0.10 of the methodology).

Every child here is **read-only** and its report is the **raw output**: exit code + raw log (stdout+stderr) of each command.

Build, lint and test are **deterministic**: exit code + log, no ambiguity to resolve.

- ✅ **Allowed:** you resolve a unit's real commands (CI + reusable) and spawn children with a
  closed spec: *"run exactly these commands, in this order, in the directory `<unit_path>`, and return the
  exit code and the raw log (stdout+stderr) of each one — nothing else."* **Lint/build:** one child per
  touched unit, in parallel (they are not tests; use the non-fixing variant of lint — a child never changes files).
  **Tests:** a single child runs the full suite of each unit, **one unit at a time**, through the single test lock
  (§5.4: tests never in parallel).
- ❌ **Forbidden:** asking the child to summarize the log, decide whether the failure is serious, suggest a fix or
  interpret the result.

**You read the raw log, decide the verdict and write the report.** If a child returns anything that
isn't exit code + log (e.g. "everything passed"), treat it as incomplete and read the log before reporting.

Convention validation (Block A) and **opening the PR** (Block D) are **never** delegated: they are deterministic,
fit in the main thread, and the second is an action visible to other people.
</delegation>

<objective>
Validate that the current branch is ready and, if it is, **open the PR** in the current repository:

1. **Conventions** (Block A) — those the project declares (commits, PR title, "unit per commit"…),
   replicating the logic of the validation workflow, if it exists; the branch name counts only as a warning.
2. **Issues** — acceptance criteria of the parent and sub-issues all `[x]`, sub-issues closed with their validations comment; otherwise, **the PR is not opened** (step 6).
3. **`ci:check`** (Block B) — for each unit the branch touched, its real CI recipe: lint, build
   and the **full test suite**. It is the **only** place in the cycle where the full suite runs (methodology
   §5.4); in `execute`, in the review and in the lanes, tests are only scoped.
4. **PR** (Block D) — only if all ✅ **and with no warnings at all**; it goes to human review **only with the PR's CI green**.

Up to Block D, this command is **read-only**: it reports what is broken and waits for the user's decision before
any fix.

Arguments: $ARGUMENTS — optional `--base <branch>`, the comparison branch (step 1); without it, the base comes from
"Working base branch" in the methodology.
</objective>

<when_to_apply>
**Always mandatory**, for every branch, before opening or updating a PR. It is **per branch, not per
sub-issue**: it runs once, after **all** of the issue's sub-issues have gone through `/codeform:execute` (review and
security included) — and, if there was parallelism, after `/codeform:orchestrate --merge`.
</when_to_apply>

<scope>
The scope is the **current branch compared with a base branch** (`--base`).

1. **Commits in scope:** `git log <base>..HEAD` — every commit the current branch has and the base doesn't.
2. **Merge commits are ignored in Block A**: a commit with more than one parent (e.g. syncing the base into the branch)
   has a diff that crosses units by nature and cannot be treated as a violation. Confirm in the project's validation
   workflow, if there is one, that it does the same.
3. **Touched units:** the union of the units (service, package, app) the files of the in-scope commits
   touch. The `path → unit` map comes from the project (validation workflow, path filters of the CI workflows
   or `CLAUDE.md`); in a single-unit repository, it is the root. Only the touched ones enter Block B.
4. **Out of scope:** units the branch didn't touch are not built/linted/tested. Something broken in
   passing in an untouched one is an observation, not a scope expansion.
</scope>

<process>
Follow the **Pair Programming Contract** (§0 of the methodology) in each step.

### Setup

0. **Release preconditions** — any failure: **stop and say what is missing**.
   - The current branch is **not** a protected branch (methodology, "Working base branch") nor a lane branch (listed in `codeform-lanes/<Initiative>/LANES.md`, methodology §6.3, if there is one).
   - `git status --porcelain` **empty** (there is uncommitted work → the PR wouldn't represent the branch).
   - **Every** sub-issue of the parent has a **validations comment** with the Gate log filled in (methodology §2.4:
     review or waiver with a reason; security or "not applicable" with a reason; **accepted risks** listed). A
     sub-issue without it = the execution didn't go through the gates: **block** and tell the user to run
     `/codeform:execute <sub-issue>`.

1. **Resolve the base:** `--base`; otherwise follow "Working base branch" in the methodology. Confirm it exists:
   `git rev-parse --verify <base>`. If it doesn't exist or there is more than one candidate, **STOP and ask**.
2. **Read the real sources** (fresh, not from memory): the project's convention rules (`CLAUDE.md`,
   `CONTRIBUTING`), the convention validation workflow (accepted types, path→unit map, exceptions such as
   managed files) and, for each unit from step 4, the CI workflow + the reusable ones it inherits.
3. **Gather the in-scope commits:** `git log <base>..HEAD --format='%H %P'` (merge commits by the number of
   parents) and, for each non-merge commit, `git show --name-only --format=`.
4. **Map the touched units:** apply the `path → unit` map to the files of all commits from step 3,
   respecting the exceptions the project declares (e.g. files managed by a release tool).

---

### Block A — Conventions (never delegated)

Each item below applies only **if the project declares the rule** (step 2). No rule: skip it and record that.

5. **Branch name:** if the project declares a pattern, report as a **warning** (never a failure) whatever doesn't
   follow it.
6. **Issues completed:** follow "Locate" in §2 of the methodology and re-read the parent and **all** sub-issues
   (`gh issue view <n> --json state,body`).
   **❌ Blocks the PR** if any of these is true: (a) the parent has an acceptance criterion `- [ ]`; (b) some
   sub-issue has a criterion `- [ ]`; (c) some sub-issue is open; (d) a sub-issue has no `### Contract` or no
   validations comment (there's no way to prove it was completed). List number + what is missing. Don't edit here
   (read-only): propose running step 19 of `/codeform:execute` (or, after a `--merge`, step 16 of
   `/codeform:orchestrate`). A criterion the user decides not to fulfill is their decision: they update the issue
   (or explicitly authorize proceeding); you never check `[x]` to unblock the PR.
7. **PR title**, if the project declares a rule and a PR already exists (`gh pr view --json title` — if it fails because
   there is no PR yet, **skip it and say so**): fails if it violates the rule (e.g. requiring or forbidding the conventional
   commit format).
8. **Commits:** apply **only** the commit rules the project declares — message format and, if any,
   "at most one unit per commit". For the unit rule, collect per non-merge commit the set of
   units its files touch (step 4, per commit, not the union); more than 1 is a violation — record `sha`,
   the units and the message. The rule is **per commit**, not per branch: a legitimate multi-unit branch, with
   one commit per unit, is correct.

---

### Block B — `ci:check` (mechanical delegation allowed)

9. For each unit from step 4: resolve the real recipe (step 2) and build the exact list of commands, in the CI order
   of that toolchain, separating the **lint/build** steps from the **test** steps. The test command is the
   CI's, **without a scope filter** (the unit's full suite), with the runner in serial mode and listing each test by
   name (Jest: `--runInBand --verbose`; another runner: the equivalent, checked in the docs/`--help`), so every contract
   test name shows in the log (methodology §2.2). If the project's `ci:check` script already exists, use it with those
   flags. Those two flags are the only allowed difference from the CI — **state it in the report**.
10. **B1 — lint/build:** spawn **one child per touched unit, in parallel** (`<delegation>`, §0.10; child-id
    `release-lint-<unit>`): spec with the exact commands and the `unit_path`, required report =
    exit code + raw log of each command, nothing more.
    **B2 — tests (after B1):** **a single** child (child-id `release-tests`) runs the full suite of each
    unit, **one unit at a time, in the order of step 4**, never two at the same time; same return. A unit
    whose build failed in B1 doesn't enter B2: mark the test as `— (blocked)`.
11. **Read each child's raw log yourself** (its report file, §0.10) before classifying anything as ✅/❌. An exit
    code 0 without a readable log still needs to be checked — don't trust only the number.

---

### Block C — Report

12. **Result matrix:**

    | Unit | Toolchain | Lint | Test | Build | Evidence |
    |---|---|---|---|---|---|
    | `<unit>` | `<toolchain>` | ✅ | ❌ | — (blocked) | log excerpt |

    A `—` cell means "this unit's CI doesn't have this step" (with the reason), never "I didn't run it".
    Distinguish the two explicitly. Report exactly the real CI coverage; never "complete" a unit
    with a step it doesn't have.

13. **Violations and warnings** (Block A), in the format:
    ```
    ### ⚠️ <warning>: <detail>
    **Expected:** <rule declared by the project, with the source>
    ```
    ```
    ### ❌ Commit <sha> violates <rule>: <detail>
    **Message:** <first line of the commit>
    **Recommended fix:** <what the rule asks for>
    ```

---

### Block D — Verdict, decision and PR

14. **Verdict:** ✅ **Ready for PR** (preconditions, Block A with no violations **and no warnings**, `ci:check`
    (Block B) all ✅ or `—`) | ⛔ **Blocked** (any ❌) | ⚠️ **With warnings** (no ❌, but there are warnings — e.g. branch
    off-convention). A pending issue (step 6) is ❌, not a warning.
15. **⛔ or ⚠️ — stop.** Report and **wait for the decision** (§0.2). If a violation has more than one valid
    way to fix it, **STOP**, present the options and wait for the choice. **Don't open a PR.** Approved
    fix: surgical, with the diff, running **only the affected check** and pasting the output; then confirm that
    the item became ✅ without reopening those that were already ✅. Touching history (`git rebase`) is a destructive
    operation — only with explicit approval.
16. **✅ — open the PR yourself:**
    a. **Push the working branch:** `git push -u origin HEAD`. **Never** the default branch or the base,
       never `--force`.
    b. **Is there already a PR for the branch?** (`gh pr view`). If there is: the push already updated it — report the URL and
       go to step f; do **not** create another.
    c. **Title:** follow the PR title rule the project declares (format, whether or not it must be
       conventional) and **validate it against that rule** before creating. No rule: a short title that describes the
       goal.
    d. **Template:** if `.github/pull_request_template.md` (or equivalent) exists, use it as the structure of the
       body. Without a template, the body, in the repository's language, has: (i) **Summary** — the goal (project
       context §2); (ii) **Issue** — `Closes #<n>` and the list of sub-issues (`#<s>` — what each one delivered);
       (iii) **Verification** — the Block C matrix; (iv) **Accepted risks** — from the sub-issues' Gate logs
       (severity + justification), or "none"; (v) the merge method the project requires, if declared. Either way
       the body cites the issue and every sub-issue. End with the session's PR attribution line. Don't write a bare
       `@name` in the body (it notifies the person).
    e. `gh pr create --base <base> --title "<title>" --body "<body>"` and report the **URL**.
    f. **Green CI before human review:** follow the PR's checks (`gh pr checks <PR> --watch`). Only with every
       check green is the PR handed to human review. A failing check: **stop**, report it with the link and
       the log excerpt, and wait for the user's decision — the PR doesn't go to review red.
    g. **Validations comment on the parent issue** (methodology §2.4): `gh issue comment <n> -F <file>` — for each
       contract line of every sub-issue, the test (name + file) and its result in the `ci:check` log (Block B); the
       Block C matrix; the PR URL; and the total of critical stops and decisions recorded on the sub-issues.
    h. **Conclusion comment on the parent issue:** `gh issue comment <n> -F <file>`, short, in the repository's
       language: what was delivered (per sub-issue), the scope changes (`## Mudanças de escopo`, if any), the
       decisions recorded on the sub-issues (count and links), what stayed out (new issues opened) and the PR URL.
</process>

<guardrails>
- **Read-only until Block D.** No edits, no `git rebase`, no rewritten commits without explicit
  approval.
- **PR only with completed issues** (parent and sub-issues with criteria `[x]`, sub-issues closed with validations) **and** ✅ with zero warnings; **human review only with the PR's CI green**. A warning or failure = stop and ask. The authorization to open the PR applies
  only to that green case.
- **Push only the working branch**, never the default/base branch, never with `--force`.
- **Never run build/lint/test of a unit outside the scope** (step 4).
- **`ci:check` tests never in parallel** (step 10, B2): one unit at a time, runner in serial mode.
- **Never invent coverage or rules:** nonexistent CI step → `—` with the reason; a rule not declared by the
  project → item skipped and stated in the report.
- **The unit rule is per commit, not per branch** (step 8) — don't block a legitimate multi-unit
  branch.
- **A sub-issue without its validations comment (Gate log) blocks** (step 0): the execution didn't go through the quality gates.
- **Surface confusion, don't hide it** (§0.3): if the documentation and the validation workflow diverge, or if
  a unit doesn't have a clear recipe, record it as a finding and ask.
</guardrails>

<gate>
PR opened and CI green: "Release done. PR: <URL> (base `<base>`), CI green — ready for human review." Add the
merge method the project requires, if declared. CI red: "PR opened, CI failing — not ready for review: <check link>." 

If ⛔ **Blocked**: "Release blocked. Resolve the items in the report — a broken `ci:check` or an issue
with a pending criterion/open sub-issue prevents the PR."

If ⚠️ **With warnings**: "Release with warnings — the PR was not opened. Decide: fix the warnings or authorize me to
open the PR as is."
</gate>
