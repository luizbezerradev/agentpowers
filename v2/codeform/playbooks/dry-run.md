# Playbook — Pre-execution dry-run

> Read by the **new, clean-context** child agent that `/codeform:plan` spawns (§0.10) after the user approves the
> drafts. The context is clean on purpose: whoever wrote the contracts is biased toward marking ✅.
> This playbook is **read-only**: the child produces the report (Blocks A–C and D1); the parent applies the
> approved updates (D2–D4).

<execution_context>
Before proceeding, read `../methodology.md` in full — especially §0 (Pair Programming Contract, §0.11 and §0.12),
§2 (records, contract, scope change) and §6.1 (parallelization plan), and `../templates/contract.template.yaml`
(the contract fields).
</execution_context>

<source_of_truth>
⚠️ **READ BEFORE ANYTHING ELSE — this block overrides any other instruction in this file.** What is correct is defined
in §0.12 of the methodology. In this phase: a contract that is clear enough to code but isn't the correct path is a
finding; you don't rewrite anything — everything goes into the report (Block C → D1).
</source_of_truth>

<child_protocol>
🤖 **You run as a child agent, spawned by the command's agent (the "parent") — an Orca worker, or a Claude subagent
outside Orca. You have no channel to the user** (§0.7 of the methodology). Ask and report exactly as §0.10 says
("Questions" — through the parent; report file + `worker_done`).

- Where this playbook says **STOP**, **ask**, **wait for the choice** or **debate**: don't decide and don't move
  on. Send a `STOP` block (§0.10) with the decision, the options (pros and
  cons), your recommendation and the reason, and **end the turn**. The parent takes it to the user and answers you
  with the user's answer. §0.2 and §0.3 still apply; only who talks to the user has changed.
- **Read in full** every file you base a claim on and cite `path:line` (§0.1). The parent opens
  every line cited in a blocker/critical finding before presenting it; a finding without a verifiable line is
  discarded.
- **State your coverage:** finish by saying what you read and what you did **not** read. "Nothing found" without the list of
  what was read is incomplete.
- **When in doubt between ✅ and ⚠️, mark ⚠️.** You were spawned clean precisely so you don't inherit the optimism of whoever wrote the contracts.
- **Output format:** the D1 report — executive summary, table of gaps that require a decision, pending decisions from Block A3 grouped by sub-issue, and the validation of the parallelization plan (Block B3). Nothing is written to disk except your report file (§0.10).
</child_protocol>

<objective>
Perform a **complete review of all the contracts of the issue** before any line of code is written. Guarantee that,
at execution time, no critical decision (§0.11) is left open and every other doubt has its answer in the contract,
the issue, the project's `CLAUDE.md`/`AGENTS.md` or the codebase.

This phase operates in two simultaneous modes:

1. **Implementation dry-run:** mentally simulate each contract as if you were really implementing it — go through
   each line of `api`, `regras` and `tela`. If something isn't clear enough to code and test immediately, it is a gap.
2. **Issue ↔ contracts gap analysis:** compare what the issue's acceptance criteria and the project context promise
   with what the contracts actually cover. Every criterion must appear in some contract line; every contract line
   must have a basis in the issue.

At the end, deliver a gap report for decision.
</objective>

<guiding_question>
When going through each contract in Block A, don't validate only "is it clear enough to code" — validate whether the
described path is the **right** one (§0.12). Fixing a contract now (through the issue, §2.2) is always cheaper than
discovering the problem during execution. Explain didactically why a path is wrong and which one you recommend, as a
recommendation (Block C).
</guiding_question>

<process>
Follow the **Pair Programming Contract** (§0 of the methodology) in each step.

### Setup

1. **Resolve the drafts** from the brief: `PROJECT_CONTEXT.md` (methodology §2), the contracts (`$D`) and the parent issue
   (`gh issue view <n> --comments`).
2. **Load everything:** the issue and its comments, the project context, every contract, and the **wireframe** (if
   the initiative touches frontend: the project context's "Wireframe" section has the artifact URL; read its `.html`
   source in full).
3. **context7 (mandatory):** for each lib/framework/SDK cited in the contracts, run
   `resolve-library-id` → `query-docs`, to validate that API names, formats and integrations match
   the library's reality — not memory.

---

### Block A — Dry-run per contract

For **each contract**, in the plan's order, go through:

#### A1. Implementation simulation
- Are all fields of the template filled in, with no placeholder (`objetivo`, `fora_do_escopo`, `depends_on`,
  `impacto`, `risco`, `api`, `regras`, `tela`, `decisoes`)?
- `api`: route, required input and **each response with its exact status code and body**?
- `regras`: **every number explicit** — limit, period, duration, what resets the count?
- `tela`: "when the user does X, Y appears" for each interaction, citing the wireframe region/state?
- Does **every** line of `api`, `regras` and `tela` name its test, with a name specific enough to be unique?
- Do the paths in `impacto.arquivos` exist in the project / make sense within the expected structure? Is the legacy in
  the area (`impacto.legado_na_area`) mapped, and is what the contract replaces mapped for removal (imports,
  re-exports, usages)?
- Does each `decisoes` entry have a source? A kept response that reveals whether an account exists without a recorded
  security decision (template note on `api`) is ❌.

Mark each point as: ✅ clear | ⚠️ ambiguous (needs a complement) | ❌ gap (blocking)

#### A2. MANDATORY checklist per contract
- [ ] No implicit weak typing (types of the new contracts explicit)?
- [ ] If the contract changes the schema/format/location of data **already in production**, is the **migration** of
  that data in `impacto.dados` (not just the removal of the old code)?
- [ ] Do file, folder, function, variable and type names follow the project's convention?
- [ ] Are the paths **relative to the project root**?
- [ ] YAGNI/KISS: does the contract ask only for what the issue's criteria require? No speculative abstraction?
- [ ] Are `risco.flags` and `risco.tier` coherent with the lines (auth, persistence, external input… declared)?

#### A2.1. Frontend × wireframe (if the initiative touches frontend)
- Does every wireframe state (empty, error, loading, modal…) have a `tela` line covering it?
- Does every `NEW` component of the wireframe appear in some contract?
- Does any `tela` line contradict the wireframe? Never resolve it silently: ✏️ update through the issue or ❓ user
  decision (and, if it's the design that changes, the wireframe is updated through the update mode of the wireframe playbook).

#### A3. Pending decisions
List every doubt that execution would hit and that the contract, the issue, the project's `CLAUDE.md`/`AGENTS.md` and the codebase
don't answer (§0.11). **Critical** ones (§0.11 list): describe them with options and pros/cons and **STOP** (§0.2).
Non-critical ones: record your recommended answer, so the parent can put it in `decisoes`. Group them in the D1 report.

---

### Block B — Issue ↔ contracts gap analysis

#### B1. Coverage
- Take each acceptance criterion of the parent issue (and each phase of the project context §11).
- Map: which contract line covers it? (cite the sub-issue and the line)
- What was left without a line? → **coverage gap** (❌)
- What appears in a contract but has no basis in the issue? → **scope creep** (⚠️)

#### B2. Consistency between contracts
- Do field/entity names shared between contracts appear consistent?
- Are the architecture decisions of the project context (§3 — mandatory engineering principles — and §13 — risks
  and implementation decisions) reflected in the contracts?

#### B3. Dependencies and parallelization plan (methodology §6.1)
- Does each `depends_on` describe a **real** dependency (B consumes something A delivers)? A dependency without that
  justification is ⚠️ (it serializes for no reason).
- Are there contracts that could be parallel but are in series?
- **Validate the project context §11.1 against the contracts:** for each block declared parallel, check
  `depends_on` and **calculate the intersection** of `impacto.arquivos`. A dependency inside the block, a non-empty
  intersection, or a contention file without a single owner is ❌: the block is not parallel.
- Is §11.1 in the **dash diagram format (ASCII)**, with the "Paths each sub-issue touches" list below it? A table
  or Mermaid in place of the diagram is a format ⚠️ (the user asked for dashes).
- Are the **verdict** (`serial`/`partial`/`full`), the **blocks**, the **merge order** and each sub-issue's **tier**
  present and coherent with the risk flags?

---

### Block C — Recommendations

For each point from blocks A and B, format per recommendation:
```
**[<behavior> contract | project context]** | **Type:** Ambiguity / Gap / Legacy / Naming / Pending decision / Scope creep
**Problem:** <objective description with a reference to the field or section>
**Recommendation:** <what to do — solves it at the source, without legacy>
**Rationale:** <why this matters>
**Action:** ✏️ Regenerate the contract / ✏️ Update the project context / ✏️ Change the issue (§2.3) / ❓ User decision
```

---

### Block D — Report (D1) — the rest belongs to the parent

**D1. Deliver to the parent:**
- Executive summary: N contracts reviewed, X OK, Y with suggested adjustments, Z with gaps for decision.
- Table of gaps that require a decision.
- List of pending decisions from Block A3 (grouped by sub-issue).
- Result of the parallelization plan validation (B3).

**D2–D4 are not yours.** The parent presents the report to the user, waits for approval, applies the updates to the
`PROJECT_CONTEXT.md` and the contract drafts (never deleting recorded decisions — rewriting or complementing them) and confirms the re-read. If the parent
resumes you to re-validate the changed parts, re-read only those.

---

</process>

<guardrails>
- **Read-only.** You don't change any file.
- **Always cite the contract/section + field + excerpt** when pointing out any problem.
- **Critical decisions belong to the user** (§0.2, §0.11): **STOP**, present options and **wait for the choice** —
  gaps go into the report (D1), never resolved unilaterally.
- **Don't simulate code** — the dry-run validates *the contracts*, it doesn't write implementation.
- **Don't expand scope:** recommendations solve real gaps, they don't add unplanned functionality.
- **Surface confusion, don't hide it** (§0.3): any ambiguity found in any block goes into the D1 report as a pending
  item — never resolved silently by marking ✅ on assumption. When in doubt between ✅ and ⚠️, mark ⚠️.
</guardrails>

<gate>
Return the verdict to the parent: ✅ **Ready for execution** or ⚠️ **Items pending user decision**,
always accompanied by the coverage (what was read).
</gate>
