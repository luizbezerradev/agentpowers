# PROJECT_CONTEXT — {{INITIATIVE_NAME}}

> Local file `docs/codeform/<Initiative>/PROJECT_CONTEXT.md`, written by `/codeform:plan` (where it lives and why it
> is never committed: methodology §2). Fill in each section from the study approved in `/codeform:brainstorm`. Language **clear to any dev**.
> Remove the instructions between `<!-- -->` at the end. Sections marked **(required)** cannot be missing.
> Headings, bold labels and fixed values stay exactly as written here, in English (other commands look them up
> by this text); the prose content you write under them is pt-BR (output language rule of the methodology).

## 1. Purpose of this document
<!-- Keep these three bullets, written in pt-BR in the generated document: it consolidates the functional and
     operational design of the initiative; it makes clear **where we want to get**, **why** this is the right design
     and **how** we will get there; it lists the initiative's mandatory rules. -->
- {{consolidates the functional and operational design of the initiative}}
- {{makes clear **where we want to get**, **why** this is the right design and **how** we will get there}}
- {{lists the initiative's mandatory rules}}

## 2. Final goal of the implementation **(required)**
<!-- Narrative description of the desired final state. Final flow step by step. Mandatory business
     rules. Expected result. -->

## 2.1 Wireframe **(required if the initiative touches frontend)**
<!-- Artifact link (high-fidelity canvas from /codeform:brainstorm) and the name of its .html source, kept outside
     the versioned tree (methodology §2, "Wireframe"). It is the visual source of truth:
     /codeform:execute implements the frontend based on it and updates it when the frontend changes;
     the /codeform:execute reviewer checks the real UI against it. No frontend: write "Not applicable — no
     frontend". -->
- Artifact: {{ARTIFACT_URL}}
- Source: `WIREFRAME_{{TOPIC}}.html` (in `$D`, methodology §0.10)

## 3. Mandatory engineering principles
<!-- E.g.: no supported parallel flow; implement directly at the source; clear responsibilities;
     idempotency and auditability. Include here the MANDATORY rules relevant to this initiative. -->

## 4. Diagnosis of the current state
<!-- What is broken/incomplete today. Observed files that show the problem
     (paths relative to the project root). Diagnostic conclusions. -->

## 5. What the official documentation shows us
<!-- Optional. Decisions grounded in official docs (via context7). List the references. -->

## 6. Candidate domains and sub-issues **(required)**
<!-- One subsection per domain. Lists the behaviors that will cover that domain — each behavior becomes one
     sub-issue with its own contract (methodology §2.2). -->
### 6.1 `{{domain-A}}`
- {{behavior-1, e.g. cadastrar}} — {{one-line summary}} — #{{sub-issue}}
- {{behavior-2, e.g. editar}} — {{summary}} — #{{sub-issue}}
### 6.2 `{{domain-B}}`

## 7. Main domain concepts
<!-- Each concept/entity with its main fields and how they relate. -->

## 8. Final flow, didactic and complete **(required)**
### 8.1 Step 1 — {{NAME}}
### 8.2 Step 2 — {{NAME}}
<!-- All steps of the flow, in order. -->

## 9. Tree of impacted files and folders **(required)**
<!-- Notation [create] / [modify] / [remove]. Paths relative to the project root. -->
```text
src/
  {{module}}/
    {{file}}.ts                 [create]
    {{existing-file}}.ts        [modify]
```

## 10. Target data model
### 10.1 New or remodeled tables/collections
### 10.2 Old tables/collections to remove
### 10.3 Recommended keys and uniqueness

## 11. How we will get there — phases **(required)**
<!-- High-level sequential phases. Each phase becomes one or more sub-issues in /codeform:plan, with the
     expected delivery. -->
### Phase 1 — {{name}} → delivery: {{...}}
### Phase 2 — {{name}} → delivery: {{...}}

## 11.1 Parallelization plan **(required)**
<!-- Contract of §6.1 of the methodology. /codeform:plan writes it, the dry-run validates it and /codeform:orchestrate reads it.
     Parallelism that is not proven here does not exist. Extract the paths from each contract's `impacto.arquivos`
     and calculate the intersections — don't presume. Fully serial initiative: verdict `serial` and the other lines only
     justify it (which path/dependency prevents it). -->
- **Verdict:** {{serial | partial | full}}

<!-- MANDATORY FORMAT: diagram in DASHES (ASCII, inside ```text). No table and no Mermaid.
     One block per row: "BLOCK X  #NNN  title  [TIER]" + the service/paths on the line below. `|` and `v` = dependency
     (the lower one only starts after the upper one). Parallel sub-issues sit side by side, open with `+----+` and close
     with the line "merge: #NNN -> #MMM". After the diagram, a "How to read" line and the list "Paths each sub-issue
     touches" (one line per sub-issue) — it is the intersection input /codeform:orchestrate checks. -->
```text
 BLOCK A    #101 {{title}}                       [{{LIGHT|STANDARD|HEAVY|CRITICAL}}]
            |    {{service}}: {{paths}}
            v
 BLOCK B    #102 {{title}}                       [{{TIER}}]
            |    {{service}}: {{paths}}
            +-----------------------------+
            |                             |
            v                             v
 BLOCK C    #103 {{title}}                #104 {{title}}
 (PARALLEL)      [{{TIER}}]                    [{{TIER}}]
                 {{service}}: {{paths}}        {{service}}: {{paths}}
            |                             |
            +----- merge: #103 -> #104 ---+
            |
            v
 BLOCK D    #105 {{title}}                       [{{TIER}}]
                 {{service}}: {{paths}}
```

**How to read:** {{in pt-BR: each `|` and `v` is a dependency; only the sub-issues side by side within the same block run
at the same time. Each sub-issue's tier is in brackets.}}

**Paths each sub-issue touches** (intersection input):
- #101 `{{paths}}`
- #102 `{{paths}}`

- **Contention files** (edited by more than one sub-issue, or read by one after being edited by another):
  `{{file}}` — single owner: #NNN. Without a single owner, the sub-issues are not parallel.
- **Merge order** of each parallel block: {{#NNN → #MMM}}.
- **Independence verified by:** {{e.g. empty path intersection between the sub-issues of block X}}.
- **Dry-run:** {{⚠️ pending | ✅ approved on YYYY-MM-DD}} <!-- written by /codeform:plan; /codeform:orchestrate requires ✅ -->

## 12. Mandatory test cases
### 12.1 {{scenario}}
<!-- Step by step of the end-to-end test that proves the goal was reached. The tests of each behavior are named in
     its contract (api, regras, tela); don't repeat them here. -->

## 13. Risks and implementation decisions
### 13.1 Risk: {{...}}
- **Risk:** {{what can go wrong}}
- **Decision:** {{what we will do to avoid it}}

## 14. Where we want to get and how we will get there **(required)**
### Where we want to get
### How we will get there
### Final rule of this initiative
