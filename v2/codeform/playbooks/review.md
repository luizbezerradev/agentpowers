# Playbook — Post-execution code review

> Read by the **new, clean-context** child agent that `/codeform:execute` spawns when a sub-issue's contract is done (§0.10).
> It is **read-only**: the child points out and classifies; the parent fixes what is simple and escalates what is
> sensitive (§7 of the methodology).

<execution_context>
Before proceeding, read `../methodology.md` in full — especially §4 (Clean Code, 9 Object
Calisthenics, YAGNI/KISS/DRY/Less Code Best Code), §5 (SDD/TDD) and §7 (simple × sensitive findings).
</execution_context>

<source_of_truth>
⚠️ **READ BEFORE ANYTHING ELSE — this block overrides any other instruction in this file, including
the contract fidelity checklist.** What is correct is defined in §0.12 of the methodology. In this phase: code that
follows the contract but isn't the correct path is a finding; you don't fix anything.
</source_of_truth>

<child_protocol>
🤖 **You run as a child agent, spawned by the command's agent (the "parent") — an Orca worker, or a Claude subagent
outside Orca** (§0.7 of the methodology). You resolve your doubts **directly with the user, in your own session**, and
report to the parent **only when done** (§0.10, "Questions" — direct mode; report file + `worker_done`).

- Where this playbook says **STOP**, **ask**, **wait for the choice** or **debate**: don't decide and don't move
  on. Write a `STOP` block **to the user in your session** — the decision, the options (pros and cons), your
  recommendation and the reason — and **end the turn**; the user answers you there. Never send questions to the
  parent. §0.2 and §0.3 still apply.
- **Read in full** every file you base a claim on and cite `path:line` (§0.1). The parent opens
  every line cited in a blocker/critical finding before presenting it; a finding without a verifiable line is
  discarded.
- **State your coverage:** finish by saying what you read and what you did **not** read. "Nothing found" without the list of
  what was read is incomplete.
- **You don't fix anything.** Not even what seems obvious: fixing simple findings belongs to the parent, fixing sensitive ones to the user.
- **Output format:** a report with (1) summary per file (✅/⚠️/❌); (2) the frontend-versus-wireframe table, if applicable; (3) list of findings — each with `id`, `path:line`, violation, suggested fix, **class** (`blocker` | `improvement`), **proposed handling** (`simple` | `sensitive`, by the criteria of §7 of the methodology) and, for a blocker, **catchable by** (`test` | `lint` | `no` — could a test or a lint rule have caught it?); (4) the contract table (`contract line → test exists? → passed? → values match?`); (5) verdict (**Sub-issue approved** | **Sub-issue blocked**); (6) coverage.
</child_protocol>

<objective>
Perform the **code review of the sub-issue given in the brief** after its execution, checking whether the
implemented code (1) matches its contract exactly (values, named tests, tests passing) and its acceptance criteria and (2)
respects the methodology's quality principles (§4 and §5 of the methodology). **This playbook is
read-only** — it points out and classifies; it doesn't fix.
</objective>

<guiding_question>
This review is not only "does the code match the contract line by line" — it is validating whether what was
implemented is, in fact, the path of the `<source_of_truth>`.

Direct consequence for classifying findings: code that "works" but solves it with a
workaround, technical debt or outside the conceptually correct place is a **blocker**, even if it
technically satisfies the acceptance criterion.

Applied to this phase: if the code followed the contract to the letter but it is the *design requested by the
contract* that is conceptually wrong, **that is also a review finding** — don't validate it "because the contract
said so". Record the finding, explain didactically why the contract's path is wrong and which one you recommend, and
classify it as `sensitive` (§7): something that changes the design is never "simple" — even if it seems obvious.
</guiding_question>

<when_to_apply>
**Mandatory** when the sub-issue implements:
- Business logic (calculations, domain rules, state flows)
- Public API (endpoints, contracts, DTOs)
- Critical flow (authentication, payment, authorization, sensitive data)
- **Any frontend screen or component** — the review includes item 7 of the checklist (frontend)

**Optional (but recommended)** for scaffolding/infra without logic.
</when_to_apply>

<checklist>
Evaluate each item below for **each file modified by the sub-issue**. For each violation, cite
`path:line`, describe the problem and suggest the fix — without applying it without approval.

### 1. Fidelity to the contract (methodology §2.2)

- **Exact values:** does each `api` line return exactly the status code and body the contract gives, with the
  required input validated? Does each `regras` line use exactly the numbers (limit, period, duration, what resets
  the count)? Does each `tela` line show exactly what the contract says when the user does it?
- **Test exists:** for each `api`, `regras` and `tela` line, is there a test with **exactly the name** the line gives,
  and does it assert that line (not just execute it)?
- **Test passed:** does the scoped test output the parent pasted show each of those names passing?
- Is every **acceptance criterion** of the sub-issue covered by contract lines that pass?
- Is there anything in the code the contract does **not** ask for, or that `fora_do_escopo` excludes (scope creep)?
- Was the legacy code listed in `impacto.legado_na_area` left uncopied?
- Does every deviation from the contract have a **decision comment** on the sub-issue (methodology §0.11)? A deviation
  without one is a finding.

### 2. Clean Code

- Do names (variables, functions, classes) reveal intent without needing an explanatory comment?
- Small functions, with a single level of abstraction and no hidden side effects?
- No dead code, debug `console.log`/print or forgotten commented-out section?

#### 2.1 Comments (§4.3 of the methodology) — mandatory check

The repository's standard is **absence of comments**; each one that survives must justify itself.
This is currently the most frequent deviation, so review it explicitly, file by file.

For **each comment** in the diff, decide which of the six allowed categories it falls into — implicit business
rule · non-obvious decision with a link to an ADR · warning about a non-local consequence · public
contract (TSDoc/GoDoc, describing the contract) · external source/justification of an `eslint-disable` ·
traceable `TODO(ID)`. **Doesn't fall into any? It's a finding.** Classify it by the anti-pattern:

| Finding | How it shows up | Fix to propose |
|---|---|---|
| **Changelog comment** | *"X used to be here"*, *"removed in TICKET-452"*, *"it used to be Y"*, *"died in issue Z"*, *"no longer exists"* | Delete it from the code. If the reason is not yet in the commit body / ADR / a decision comment on the sub-issue, **move it there** before deleting — the narrative isn't lost, it changes places. |
| **Paraphrase of the code** | `// increments the counter` | Delete. |
| **Decorative banner** | `═══ SECTION ═══`, `// ---- helpers ----` | Delete; if the file needs a visual index, it is too big (§4.1 no. 7). |
| **Commented-out code** | an entire commented-out section | Delete — `git` keeps it. |
| **Comment that rescues a bad name** | `// here "d" is the document` | Rename the symbol (§4.1 no. 6). |
| **Outdated comment** | describes behavior the diff changed | Update or delete — a lie with authority is worse than absence. |

Also check the **budget** of each file touched by the sub-issue:

```
grep -cE '^\s*(//|\*|/\*)' <file>   # comment lines
wc -l < <file>                       # total lines
```

- **Block > 5 lines** → finding; cut it down to one sentence + link to the ADR/issue.
- **Density > 10%** → requires justification **block by block**. A file that is only a public contract
  declaration can legitimately exceed 10%; an implementation file cannot.
- **A file touched by the sub-issue with an off-standard comment that survived** → finding:
  MANDATORY 10 (§3 of the methodology) requires normalizing the **whole file**, not just the diff lines.
- **A file the sub-issue didn't touch** → **out of scope for this review.** It is not a finding; if you notice
  high density, mention it as a separate observation, without proposing an edit.

### 3. 9 Object Calisthenics (§4.1 of the methodology)

A deviation here is an **improvement** (suggestion), not a blocker — unless the contract explicitly requires the rule.
Don't propose a change that breaks the existing style of the file (methodology §0, item 5).

| # | Principle | What to check |
|---|---|---|
| 1 | One level of indentation per method | Nested `if/for/while` without extraction? |
| 2 | No `else` | `else` where an early return would do? |
| 3 | No exposed primitives | Bare `string`/`int` where there is domain meaning? |
| 4 | Encapsulated collections | `List`/`Map`/`Set` manipulated outside a dedicated class? |
| 5 | One dot per line | `a.getB().getC()` chains? |
| 6 | No abbreviations | Abbreviated names where the full name fits? |
| 7 | Small entities | Class > 50 lines or method > 10 lines? |
| 8 | ≤ 2 instance variables | New class with 3+ state variables? |
| 9 | No getters/setters | Getter/setter where "Tell, Don't Ask" solves it better? |

### 4. Classic Principles (§4.2 of the methodology)

- **YAGNI:** is there code no line of the contract asked for?
- **KISS:** is there a simpler version that satisfies the contract?
- **DRY:** duplicated logic (≥ 3 occurrences) that should be extracted?
- **Less Code Best Code:** is there code that could be removed without losing any contract line
  (unused helper, dead branch)?

### 5. SDD/TDD and test types (§5 of the methodology)

- For isolable logic: are there **unit** tests covering the contract lines (the named tests of item 1)?
- **Coverage:** does new/touched isolable-logic code have **100%** line/branch coverage by
  tests with real assertions — not just execution without validating behavior?
- **Mutation:** for critical business logic (calculation, domain rule, auth, financial flow),
  was mutation testing run **scoped to the changed files** (§5.4)? Is there a surviving mutant without
  a test fix?
- **Integration:** where the sub-issue crosses 2+ real layers (service+repository, API+database), is there an
  integration test without a mock at the point that should validate the real contract?
- **E2E:** does a critical flow or end-to-end journey exposed by the sub-issue have an e2e test that simulates the
  real user path?
- **Regression:** do the tests related to the changed files pass after the change (scope of §5.4 — the
  full suite belongs to the release's `ci:check`, not to this review)? If the sub-issue is a bugfix or protects
  existing behavior, is there a dedicated regression case that would fail without the fix?
- Do all scoped tests pass? (cite the runner output and the coverage/mutation report the
  execute agent pasted; if you run any test to confirm, only scoped and sequential, §5.4)
- Was tiering by criticality (§5.3) respected — no applicable type was skipped, and no
  type was applied where it made no sense (e.g. mutation testing on trivial scaffolding)?

### 6. Remaining MANDATORY rules (§3 of the methodology)

- No weak typing without justification?
- Does the project's lint/typecheck pass clean?
- No legacy code kept in parallel?

### 7. Frontend — check on the real screen (only if the sub-issue touches frontend)

Reading the code **is not enough**: the frontend review happens **on the running UI**, against the wireframe.

- **Start the app** and open the sub-issue's screens (skill `run`; to navigate/interact, Orca's embedded browser
  when running in Orca (skill `orca-cli`); for browser tests, whatever harness the frontend app already has, e.g. Playwright). If it can't be started,
  **say so** and don't approve the frontend — never approve a screen you haven't seen.
- **Wireframe:** the artifact URL is in the "Wireframe" section of the project context (methodology §2; read it
  with `Artifact` `action: "read"`, or its local `.html` source). For **each region and each state** of the
  wireframe, compare with the real screen: structure, components, copy, states (empty, loading,
  error, permission denied, modals) and interactive behavior. A divergence is a finding.
- **Artifact up to date:** if the frontend diverges from the wireframe **because it changed during execute**, the
  artifact should have been updated — an outdated wireframe is a **blocker** (execute didn't
  comply with `<frontend_wireframe>`). If it is the design that was wrong, the finding is the user's decision,
  not a silent fix on either side.
- **Fidelity to the design system:** are the components and tokens used those of the project's design system and
  of the rules in the frontend app's `CLAUDE.md` (mandatory primitives, bans on raw styling…)? With no
  design system or documented rules, say so and compare with the existing components.
- **Theme and responsiveness:** does the screen work in the themes the frontend supports and at the relevant
  smaller/larger widths? No overflow, no cut-off text.
- **Basic accessibility:** visible focus and tab order, roles/labels, contrast, keyboard
  navigation on the new controls.
- **Console and network:** no new error/warning in the browser console; API calls according to the contract's
  `api` lines (no duplicated or looping requests).
- **Evidence:** paste what you saw — screenshots/Playwright output and the table
  `wireframe region/state → ✅/⚠️/❌ on the real screen`.
</checklist>

<process>
Follow the **Pair Programming Contract** (§0 of the methodology) in each step.

**Setup:**
1. **Read the sub-issue** from the brief: `gh issue view <s> --comments` (contract, criteria and decision comments).
2. List all files modified by the sub-issue (`git diff --name-only`, minus the parent's dirty snapshot).
3. Read each modified file **in full** — don't evaluate what you haven't read.
3.1. If the sub-issue touches frontend: open the wireframe (URL in the project context) and start the app, to
   fulfill item 7 of the checklist after the code review.

**Review (per file):**
4. Go through the checklist above.
5. Record each finding: `path:line` → violation → suggested fix.
6. Classify by applying the `<guiding_question>`: **Blocker** is everything that violates a contract line, an
   acceptance criterion, a MANDATORY rule (Object Calisthenics only when the contract requires it), **or** solves it with a workaround/technical debt/outside the
   conceptually correct path — even if it "works" and doesn't break any listed rule
   literally. **Improvement** (Object Calisthenics, YAGNI/KISS/DRY/Less Code — a recommendation, not mandatory) is what is already
   correct at the source but could be simpler or leaner. For each blocker, mark **catchable by** (`test` | `lint` |
   `no`): input for the lint-or-test decision (methodology §7).

**Report:**
7. Deliver the report in the format of the `<child_protocol>`: summary per file (✅/⚠️/❌), **frontend
   versus wireframe table** (if applicable), list of findings with class and proposed handling,
   final verdict and coverage.
8. **Stop.** The parent verifies the blocker findings, fixes the simple ones, escalates the sensitive ones and, if there are
   fixes, resumes you for a **re-review restricted to the changed files** (re-read the checklist only
   on them). Round limit: methodology §7.
</process>

<guardrails>
- **Read-only.** You don't edit any file.
- **Always cite `path:line`** before stating anything about the code.
- **Don't "improve" adjacent code** that wasn't touched by the sub-issue.
- At every fork with more than one valid fix path, **STOP** (send `STOP`) and mark the finding as `sensitive`.
- **Surface confusion, don't hide it** (§0.3): if what the code does, what the contract asked for and what is
  conceptually correct don't coincide, **don't silently pick one of the three** — record the
  contradiction as a finding, explain each side and send `STOP` — the decision belongs to the user.
</guardrails>

<gate>
Return the verdict to the parent (**Sub-issue approved** | **Sub-issue blocked**) and the coverage. If the sub-issue touches
external input, authentication/authorization, persistence, sensitive data or LLM/prompts, **say so**: it is
input for the parent to decide about security.
</gate>
