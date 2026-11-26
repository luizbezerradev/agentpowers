---
description: "Creates a GitHub issue from a reported problem: a brief study of the codebase (possible causes and solutions), title and body from the repository's issue templates, and filling in the board, if the project defines one."
argument-hint: "<description of the problem or improvement>"
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
  - Agent
---

Reported problem: $ARGUMENTS

## 1. Understand the report

Classify it as a **bug**, **improvement** or **technical task**. If the report is ambiguous
enough to change the investigation, ask before continuing; otherwise, assume and state the assumption.

## 2. Brief study (read-only)

Locate the point in the codebase where the problem originates. Be quick: the goal is a grounded hypothesis, not an
audit. Read in full the files you are going to cite. Don't change code.

Produce:
- **Affected area/service**, among the options the template chosen in step 3 offers (if any).
- **Possible causes**, with `path/file.ts:line`. Mark what was **verified in the code** and what is a **hypothesis**.
- **Possible solutions**, with the trade-off of each and the recommended one.

If you can't find the cause, say so in the issue; never make it up.

## 3. Issue template

**Title and body come from the repository's templates**, never from a default of this skill. Read
`.github/ISSUE_TEMPLATE/` (and `config.yml`, if present) and pick the template by the `name`/`description`/`labels`
that matches the type from step 1. From the template, use:

- `title` and any title rule described in the `markdown` block (prefix, format, length). A rule that
  requires a decision from you (e.g. which prefix) and that the report doesn't answer: ask before creating. Never copy
  the prefix from an example.
- `labels`.
- Body: for each `body` field that is not `markdown`, in this order, a `### <label>` section followed by the
  content (the chosen option, for a dropdown). Fill in all `required` fields and whatever the study from step 2
  produced; an optional field without content is omitted. Write in the repository's language.

`gh issue create` doesn't render the form, so you build the body in the format above. A Markdown
template (`.md`): use it as is, and `gh issue create --template` works.

**Blank issue** only when the case **fits none** of the templates, or when the repository has no
`.github/ISSUE_TEMPLATE/`. Tell the user that no template fits and why, follow the project's `CLAUDE.md`
for title and body and, if it has nothing, use a short title and the sections from step 2. When in doubt between a
template and blank, use the template.

## 4. Project board (if any)

If the project's `CLAUDE.md` describes a GitHub board/project (fields such as Size, Estimate, Priority, Status,
criteria to derive them, ids), derive the values **from the study, not from a guess**, show the user the summary
(title, area, probable cause, board values and the one-line justification) and **wait for confirmation
before creating**. No board described: skip the board steps, show the summary (title, area, probable cause)
and wait for confirmation the same way.

## 5. Create the issue and fill in the board

After confirmation, save the body to a temporary file and create the issue:

```bash
url=$(gh issue create -t "<title>" -l <template labels> -F <file.md>)
```

With a board described in `CLAUDE.md`: get the issue's item (if it comes back empty, wait a few seconds or
add it with `gh project item-add`), fill in the fields with `gh project item-edit` using the ids the
`CLAUDE.md` provides. Nonexistent id: re-read `gh project field-list <n> --owner <org> --format json` and use the
current ones. In zsh, keep the `-q` argument in quotes (brackets become a glob).

## 6. Verify and report

Re-read the issue (`gh issue view`) and, if there is a board, the item's values via GraphQL (`fieldValues`). Report the
URL and the values. If something wasn't saved, say which field failed; don't declare success without the read-back.
