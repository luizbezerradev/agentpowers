---
description: "Plan — from the issue's acceptance criteria, write the local PROJECT_CONTEXT.md (with parallelization plan) and one sub-issue per behavior with its YAML contract (risk flags and tier included) → [approve] → dry-run in a clean child agent → [resolve gaps] → publish the sub-issues on the issue. With --regenerate, regenerates the contract of a sub-issue from the issue."
argument-hint: "[<issue number>] [--regenerate <sub-issue>]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash
  - Agent
  - SendMessage
  - mcp__context7__resolve-library-id
  - mcp__context7__query-docs
---

<execution_context>
Before proceeding, read in full:
- `../../skills/codeform/methodology.md` — especially §0 (Pair Programming Contract, including §0.11 doubts and
  §0.12 what is correct), §2 (canonical records, contract, scope change), §5.1 (impact list) and §6 (parallelization
  plan and tiers).
- The project's `CLAUDE.md` / `AGENTS.md` (methodology §0.11, step 2).
- The two templates that define the exact format of what you write:
  - `../../skills/codeform/templates/project-context.template.md`
  - `../../skills/codeform/templates/contract.template.yaml` (the contract fields — single definition)
- The parent issue and its comments (`gh issue view <n> --comments`): its acceptance criteria are the source of the
  contracts.
- The approved study: the `/codeform:brainstorm` conversation in this session. Without an approved
  study in the conversation, **STOP** and tell the user to run `/codeform:brainstorm` (not needed with `--regenerate`).
- The approved wireframe (the **artifact URL** and its `.html` source, given in the brainstorm conversation). Read it in
  full. If the initiative touches frontend and there is no approved wireframe, **STOP** and tell the user to run
  `/codeform:brainstorm`.
</execution_context>

<source_of_truth>
⚠️ **READ BEFORE ANYTHING ELSE — this block overrides any other instruction in this file.** What is correct is defined
in §0.12 of the methodology; doubts follow §0.11. In this phase: the correct design goes into the contract, and the
deviation from what exists goes into its `decisoes` with the source; a critical one (§0.11) stops for the user.
</source_of_truth>

<delegation>
⚠️ **The write belongs to the parent; the dry-run belongs to a new child agent** (§0.7; how: §0.10 of the methodology).

The dry-run runs in a **clean-context**, **read-only** child, following the playbook
`~/.claude/skills/codeform/playbooks/dry-run.md`. Nothing that inherits or copies the conversation: it would carry the
bias of whoever wrote the contracts, and the dry-run exists precisely to disagree. It doesn't talk to the user: it
delivers the report, and **you** verify it, present it, collect the decisions and apply the updates. A `STOP` goes to
the user, and the user's answer goes back to the same child.
</delegation>

<objective>
Turn the issue and the study into execution-ready records — the local **`PROJECT_CONTEXT.md`** and **one sub-issue per behavior
with its contract** (methodology §2.2) — **and validate them** with an independent dry-run before any line of code.
This command also records the **parallelization plan** (project context §11.1) and, in each contract, the **risk
flags** and the **tier**, which `/codeform:orchestrate` and `/codeform:execute` consume.

The contracts are what `/codeform:execute` follows **to the letter**. That is why **no doubt may become a gap in a
contract**: resolve it by §0.11 and record it in `decisoes` with its source; never fill a field with an assumption, a
disguised placeholder or a "decision to be defined later".

Arguments: $ARGUMENTS — optional parent issue number (without it, the issue of the brainstorm session, step 1) and
`--regenerate <sub-issue>` for the regenerate mode (step 18).
</objective>

<guiding_question>
We are in **dev, but production already exists** (MANDATORY 4, §3): what the initiative replaces in **code** is mapped
for removal, not for coexistence; **data** already in production is preserved — every contract that changes the
schema, format or location of existing data maps its **migration** in `impacto.dados`.

Applied to this phase: if the study led to a conceptually wrong design, don't write that design just because it came
out of `/codeform:brainstorm` — explain why it is wrong and which path you recommend (§0.12) at Gate 1.
</guiding_question>

<process>
## Part 1 — Write (parent; contract drafts in `$D`, §0.10)

### Phase A — PROJECT_CONTEXT.md

1. **Resolve the parent issue** (argument, or the issue of the brainstorm session) and `<Initiative>` (methodology §2).
   No issue: **STOP** — the flow starts from an issue (`/codeform:issue`).
2. **Write `docs/codeform/<Initiative>/PROJECT_CONTEXT.md`** (methodology §2; create the local folder and its exclude
   line if missing), following the template section by section, from the
   approved study. **No section marked `(required)` may be left empty or with a placeholder.** Missing information for
   a required section: resolve it by §0.11 (a critical gap stops for the user).
3. **Wireframe (initiative with frontend):** fill in the "Wireframe" section with the **artifact URL** and the `.html`
   name. Without the URL, **stop and ask for it**. Every `tela` line of a contract cites the wireframe region/state it
   implements, and each component marked `NEW` appears in some contract. No frontend: "Not applicable — no frontend".
4. **Remove** the template's instruction comments (`<!-- -->`) and `{{...}}` placeholders.

### Phase B — Sub-issues and contracts

5. **Split the issue into behaviors** (e.g. `cadastrar`, `editar`), one sub-issue each, **so they run in parallel
   lanes** — rules of the parallel split in methodology §6.1 (`depends_on`, `unidades`, `arquivos`, foundation
   sub-issue, real dependencies only).
6. **Generate one contract per sub-issue** in `$D/<behavior>.contract.yaml`, from the issue's acceptance criteria,
   following `templates/contract.template.yaml` (fields and rules are there and in methodology §2.2). Each issue
   criterion is covered by at least one contract line; a criterion that can only be written vaguely is a gap (§0.11).
   Fill `impacto` (methodology §5.1) with the `path:line` of who uses what changes and the legacy code in the area not to
   copy. Name the test of every `api`, `regras` and `tela` line.
7. **Validate coherence:** every dependency between sub-issues has a concrete justification; field/entity names
   shared between contracts are consistent; nothing a contract assumes is missing from the one that delivers it. A
   divergence is resolved by §0.11 — never by silently picking one of the versions.

### Phase C — Parallelization plan, flags and tier

8. **Risk flags and tier** in each contract's `risco` (tier table in methodology §6.2). **Execute decides on security by
   reading these flags** — an omitted flag is a gap: when in doubt, declare it.
9. **Parallelization plan** (project context §11.1, content and independence rule in methodology §6.1): from every
   contract's `depends_on` and `impacto.arquivos`, **calculate the intersections yourself** and write blocks, contention
   files, merge order and verdict. A `serial` verdict requires a written justification — retry the split of step 5
   before accepting it.
10. **Present** the behaviors, the blocks, the verdict, each tier, `PROJECT_CONTEXT.md` and the contract drafts.

### Gate 1 — approval of the drafts
Without explicit approval, the dry-run does **not** run. The user's corrections go back through steps 2–9 only for what
changed.

## Part 2 — Dry-run (clean child)

11. **Spawn the child** (§0.10; child-id `plan-dry-run`; read-only). Spec: *"Follow `~/.claude/skills/codeform/playbooks/dry-run.md`.
    Parent issue: #<n>. Read `docs/codeform/<Initiative>/PROJECT_CONTEXT.md` and the contract drafts `$D/*.contract.yaml`; the wireframe, if there is one.
    Read-only. Return the D1 report and the coverage."* Don't pass on your conclusions or the study.
12. **Verify the report.** Every ❌/⚠️ item citing an excerpt: open it and confirm the problem holds. Discard what
    doesn't hold up, **telling the user** what you discarded and why. A `STOP` from the child goes to the user and the
    user's answer goes back to it (§0.10).
13. **Present the D1 report** (§0.8) and **wait for approval** — nothing is written before that.

### Gate 2 — the user resolves the gaps

14. **Apply the approved updates** to `PROJECT_CONTEXT.md` and the contract drafts. **Never delete** a recorded decision — rewrite or complement it.
    If a gap answer changes what the issue asks, apply the scope change procedure (methodology §2.3) first.
15. **Re-validation.** If the change altered the parallelization plan, the tiers or more than three contracts,
    **resume the same child** (§0.10) restricted to the changed parts. Report: ✅ Ready for execution |
    ⚠️ Items pending user decision.

## Part 3 — Publish (only with ✅)

16. **Plan status:** in `PROJECT_CONTEXT.md` §11.1, set the line `- **Dry-run:** ✅ approved on <date>` —
    `/codeform:orchestrate` checks it.
17. **Sub-issues:** list the ones the parent already has (methodology §2, "Locate"). For each behavior:
    - title and body come from the repository's sub-issue template, if one exists in `.github/ISSUE_TEMPLATE/` (same
      reading as `/codeform:issue` §3); a title rule that depends on the parent (e.g. prefix) is copied from the
      parent's title. No template: title = the behavior preceded by the parent's `[..]` prefix, if any; body =
      methodology §2 ("Sub-issues"): criteria checklist + `### Contract` with the YAML;
    - new: `url=$(gh issue create -t ... -F <file.md>)`, then link it:
      `gh api repos/{owner}/{repo}/issues/<parent>/sub_issues -F sub_issue_id=$(gh api repos/{owner}/{repo}/issues/<n> -q .id)`;
      fill `sub_issue` in the contract with the number and update the body;
    - existing (plan redone): **update** its body instead of duplicating; a behavior removed → close it as `not planned`;
    - **Board:** if the project's `CLAUDE.md` says how sub-issues are handled on the board, follow it right after
      creating and re-read the board to confirm. Without such a rule, don't touch the board.
    List the sub-issue numbers to the user.

## `--regenerate <sub-issue>` mode

18. Re-read the parent issue (including `## Mudanças de escopo`) and the sub-issue. Regenerate **only** that contract
    from the issue (steps 6–8), keep its `decisoes` that still hold, show the diff of the contract to the user and, with
    the "ok", update the sub-issue body. If the parallelization plan changed, update `PROJECT_CONTEXT.md` §11.1 too.
</process>

<guardrails>
- **No product code.** Only planning records.
- **Pair Programming Contract** (§0): read what is real and cite `path:line`; surface confusion and assumptions.
- **Doubts:** methodology §0.11 — critical ones stop for the user; the rest are decided and recorded in `decisoes`.
- **Nobody edits a contract by hand** (methodology §2.2): changes go through the issue and this command.
- **Local files:** only `PROJECT_CONTEXT.md` (never committed) and the contract drafts in `$D`; the contracts live in the sub-issues (methodology §2).
- **Parallelism the plan doesn't prove doesn't exist.** Path intersection calculated by you, not presumed;
  when in doubt, serial.
- **The dry-run isn't yours.** You don't run it in your own thread, you don't pass your conclusions to it, and you don't
  rewrite the contracts before Gate 2.
- **Don't commit, don't push.** Nothing is published on the issue before Gate 2 is closed.
</guardrails>

<gate>
With the plan **published** (✅ Ready for execution), point to the route by the §11.1 verdict:

- `serial`: "Plan approved, no parallelism. Run **`/codeform:execute <first sub-issue>`** (and the others, in the
  plan's order)."
- `partial` or `full`: "Plan approved, with parallel blocks. Run **`/codeform:orchestrate`** in Orca — it
  dispatches the independent blocks; the serial sub-issues (and all of them, outside Orca) go through `/codeform:execute`."

If there are pending items: "Resolve the gaps listed above before starting execution."
</gate>
