---
name: codeform
description: Personal specs methodology for GitHub issues, in 5 /codeform commands. Use to study an issue and design its wireframe (brainstorm), plan sub-issues with YAML contracts (plan), implement one sub-issue with TDD and review (execute), parallelize in Orca (orchestrate), open the PR (release) or create an issue (issue).
---

# Personal Specs — Issue and Contract-Based Planning and Execution Methodology (flow v2)

This skill makes Claude Code **know on its own** which command of the flow to use when the user talks about studying a
feature, planning the sub-issues and their contracts, implementing them, parallelizing them or opening the PR.
The detailed source of truth is in `skills/codeform/methodology.md`; the child agents' instructions are
in `skills/codeform/playbooks/` (how children run: methodology §0.10).

The unit of work is the **sub-issue** (one behavior of the issue) with its YAML **contract**, in the format of
`templates/contract.template.yaml`. The GitHub issue plus its sub-issues is the single record (methodology §2).

## The cycle → which command to use

| The user wants… | Use | Inside it (human gates between sections) |
|---|---|---|
| Understand the codebase/issue and, if there is a screen, design the wireframe | **`/codeform:brainstorm <issue-number>`** | study → **approve** → create the branch → wireframe (child agent, only if it touches frontend) → **approve** |
| Plan the sub-issues and their contracts from the issue | **`/codeform:plan [--regenerate <sub-issue>]`** | project context + one contract per behavior (with parallelization plan, flags and tier) → **approve** → dry-run (clean child agent) → **resolve gaps** → publish on the issue |
| Implement ONE sub-issue (serially, or as a lane with `--lane`) | **`/codeform:execute <sub-issue>`** | read the sub-issue → implement the contract with TDD → spawn reviewer (and security, per the flag) → fix the simple, escalate the sensitive |
| Parallelize independent sub-issues (Orca only) | **`/codeform:orchestrate`** (`--status`, `--merge`) | plan gate (with account, model and effort per lane) → N Orca workers running `execute --lane` → decision queue → merge |
| Validate the branch and open the PR | **`/codeform:release [--base <branch>]`** | project conventions + synced issues (parent) → `ci:check`: build/lint and full test suite per unit (child agents) → PR if all ✅ → human review only with green CI |
| Create an issue from a problem | **`/codeform:issue`** | independent of the cycle |

**Route after `plan`:** the verdict of the parallelization plan (project context §11.1) decides. `serial` →
`/codeform:execute` sub-issue by sub-issue. `partial`/`full` → `/codeform:orchestrate` for the parallel blocks. Without two
or more independent sub-issues in a block, the orchestrator **does not orchestrate**: it tells you to run `execute`. Outside
Orca, parallel blocks also go sub-issue by sub-issue through `execute`.

**Respect the human gates:** each section ends with a user review; don't jump to the next one without
explicit approval. If the user describes the task without naming the command, identify which point of the cycle
they are at and offer/run the matching command.

## Where each rule lives

This skill only **routes**; it doesn't repeat rules. Everything is in `methodology.md`:

| Topic | Section |
|---|---|
| Pair Programming Contract, delegation (parent × child agents: Orca workers, Claude subagents outside Orca; lanes are sessions), reporting and explanation rule | §0 (§0.7–§0.10) |
| What is correct — the source of truth of every decision | §0.12 |
| Output language (pt-BR for replies and artifacts) | header of `methodology.md` |
| Issue rule, working base branch | header of `methodology.md` |
| Canonical records (issue, sub-issues, local `PROJECT_CONTEXT.md`), contract, scope change, validations | §2 (§2.2–§2.4) |
| MANDATORY rules | §3 |
| Object Calisthenics, YAGNI/KISS/DRY, comments | §4 |
| SDD/TDD, test types, scoped and sequential test runs | §5 |
| Parallelization plan, tiers (complexity), lanes in Orca; model/effort choice | §6, §0.10 |
| Simple × sensitive findings and the fix loop | §7 |
| Doubt order, critical list, decision record, repeated corrections → lint/test or a `CLAUDE.md`/`AGENTS.md` rule | §0.11 |
| Impact list before writing code | §5.1 |

Formats: `templates/contract.template.yaml` (contract fields) and `templates/project-context.template.md`. This skill covers the **cycle**, it doesn't redefine those formats.
