# Workflow

## How to work
- Reply to the user in Brazilian Portuguese (pt-BR). Generated artifacts (contracts, wireframe notes, issue comments) are also written in pt-BR; issue and PR bodies follow the repository's language, as the commands define. Headings, field labels and fixed values of those artifacts are English, exactly as in the templates; only the prose content is pt-BR. Code identifiers stay as originally written, and commit conventions stay as already defined.
- Understand before changing: read the code and trace the real flow. Reuse what already exists before writing, unless it is legacy or a workaround.
- TDD when the logic is isolable (criterion in §5.2 of the methodology): failing test (run it and watch it fail) → minimum to pass → refactor.
- Tests: only those for what changed (scoped) and one at a time (sequential). Full suite only in the `ci:check` of `/codeform:release` (§5.4 of the methodology).
- Never say "done" without the output of a command run after the last edit.
- Code: KISS, YAGNI, DRY, less code is better.

- Explain only in the simple version — direct, literal, human, didactic and objective, with no ambiguous metaphor and no unexplained jargon (the user is AUTISTIC). Give a technical version only when the user asks for it; it must not contradict the simple one.

## Light track × full flow
- Larger, ambiguous or risky: `/codeform:brainstorm` → `/codeform:plan` → `/codeform:execute` (or `/codeform:orchestrate`, in Orca, for parallel blocks) → `/codeform:release` (understand → TDD with principles → isolated reviewer). Cycle rules: `~/.claude/skills/codeform/methodology.md`.
- Issue rule, doubt order and scope change: `~/.claude/skills/codeform/methodology.md` (header, §0.11, §2.3).

## ALWAYS stop and ask before
- A decision on the critical list (`~/.claude/skills/codeform/methodology.md` §0.11)
- Payment
- New dependency
- Destructive operation, deploy, infra, push to main
- Changing a test that already existed (never change a test to make it pass)
- A scope change that affects what was asked (methodology §2.3)

## Child agents
- Inside Orca, a child agent is an Orca worker; outside it, a Claude subagent (how: `~/.claude/skills/codeform/methodology.md` §0.10).
- Parallelize only independent work (distinct areas, disjoint files). When in doubt, sequential.
- Give each child: goal, boundaries and output format.
