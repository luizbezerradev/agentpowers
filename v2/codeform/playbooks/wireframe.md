# Playbook — High-fidelity wireframe

> Read by the child agent that `/codeform:brainstorm` spawns after the study is approved (§0.10). It is also the
> **update mode** used by `/codeform:execute` when the frontend changes during implementation.

<execution_context>
Before proceeding, read in full:
- `../methodology.md` — especially §0 (Pair Programming Contract).
- The `CLAUDE.md` of the frontend app (and the root one) and the design system docs it points to (e.g. typography,
  text/icon/field primitives, inputs, controls, tabs, listing pages). **Find out where they are** from the
  project's `CLAUDE.md` or by looking for the frontend app and a design system folder (`docs/design-system/`,
  Storybook, tokens); with no documented design system, the real code is the only source — say so. **Re-read them on every run**: they change without this playbook being updated.
- The approved study (in the message that started you; you don't inherit the conversation). The project context
  (`PROJECT_CONTEXT.md`, methodology §2) doesn't exist yet in the first round; if it exists (re-run or update
  mode), read it.
- If the wireframe's `.html` source already exists (`$D/WIREFRAME_*.html`, methodology §2 "Wireframe"), read it — or
  the published artifact, if the source is missing: it is the current state to evolve, not to redo from scratch.
</execution_context>

<source_of_truth>
⚠️ **READ BEFORE ANYTHING ELSE — this block overrides any other instruction in this file.**

The wireframe exists to **evaluate a decision**, and an evaluation is only worth anything if the design is **faithful to the
product**: what the user sees here must be what the frontend will render. Two sources, in order:

1. **The real frontend code** (the UI components, the shared code, the modules, the global CSS and the
   token files it imports, in the project's frontend app) — what actually exists.
2. **The design system**, if the project has one — what *should* exist.

If the two diverge (legacy screen off-standard, a component that violates the cardinal rules), **don't
choose silently**: show the deviation and bring it up for debate (§0.2). Design from the design system when the
screen is new; from the existing component only when the goal is precisely to evolve it.

> What is the most enterprise-grade and conceptually correct path for this screen, using the building blocks the
> frontend **already has** — and, when a block is missing, saying explicitly that it **doesn't exist**?

**Don't invent components.** If the design needs something the frontend doesn't have, mark it in the wireframe
as `NEW` (visually distinct), list it in the report and leave the decision to the user. Never draw it
as if it already existed — that crystallizes a false promise in `/codeform:plan`.

**Don't implement anything in the frontend: you only design; deviations go into your report for the user to decide.**
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
- **You don't implement anything in the frontend.** The only files written are the `WIREFRAME_*.html` and the artifact.
- **Output format:** two phases.
  - **Phase 1 (steps 1–4):** present to the user, in your session, the fidelity inventory and **all** pending decisions (`🆕 NEW`, deviations between code and design system, choice of states) in a `STOP` block — even if there is no `NEW`, the user needs to approve the choice of states before you draw.
  - **Phase 2 (steps 5–9), after the user's decisions:** report to the parent (§0.10): the artifact URL, the `.html` path, the final inventory, the decisions the user made and what was not reproduced faithfully.
  Adjustments the user asks for in your session go back through steps 2–8 only for what changed; then report again to the parent (`worker_done` with the updated report).
</child_protocol>

<objective>
Create a **Canvas Design Artboard Artifact** — a **high-fidelity** wireframe — of the screen/flow given in the brief,
built with the real frontend components of the codebase, so the user can evaluate **how the feature
would look** before the contracts are written.

"Real components" in an HTML artifact means: **reproduce exactly** what the real component
renders — same DOM structure, same utility classes, same tokens (`--control-*`,
`bg-accent`, `bg-active`, `border-border`, `text-content-*`…), same variants and sizes, same
icons (the `lucide-react` the component uses) — and **never** an approximation "in the style of". The wireframe
is disposable; the **fidelity** is not.

This phase **doesn't create product code**. The deliverable is the artifact (and its source `.html`).
</objective>

<guiding_question>
For each element drawn, ask yourself:

> Did I open the component that renders this and copy what it actually emits — or did I draw it from memory?

And, at the screen level:

> Does this design serve the goal of the implementation, or is it just a pretty screen? Each region answers a
> requirement of the goal; a region without a requirement is visual scope creep — cut it.

We are in **dev, but production already exists**: the wireframe shows the **target** state, without preserving a legacy
screen "for compatibility". But if the change alters a screen with **data already in production**, also draw
how that existing data appears (migrated state), not just the empty case.
</guiding_question>

<process>
Follow the **Pair Programming Contract** (§0 of the methodology) in each step.

1. **Lock down what to draw.** If the brief is empty or vague (you can't tell which screen, which
   flow, which persona), **STOP and ask**. State assumptions; confirm the risky ones. List the
   **screens/states** you will draw (e.g. empty, filled, loading, error, permission denied,
   confirmation modal) and confirm with the user if the choice of states has more than one reasonable form.
2. **Locate what already exists.** Find the neighboring screen(s) and the candidate components. Open and read each one **in full**. Record
   the `path:line` of each component you will use.
3. **Read the relevant design system** (see `<execution_context>`) and the official docs via **context7** for
   the Radix/shadcn/UI lib touched — variants, states and accessibility come from the docs, not from memory.
4. **Build the fidelity inventory** — before writing HTML, present a table in the chat:

   | Wireframe region | Real component | `path:line` | Variant/size | Status |
   |---|---|---|---|---|
   | Page header | `<ExistingComponent>` | `src/…:12` | — | ✅ exists |
   | Level selector | — | — | — | 🆕 NEW (doesn't exist) |

   Every `🆕 NEW` and every deviation between code and design system **stops here** (§0.2): present the options
   with pros and cons, recommend one and **wait for the choice** before drawing the region.
5. **Load the `artifact-design` skill** (via `Skill`) and follow its page contract (title,
   tokens in `:root`, light/dark theme, layout at mobile width). The **wireframe tokens come
   from the real frontend tokens** (values read from `globals.css`/token files), mapped to
   the artifact's `:root` variables — not invented values. The wireframe must work in both
   themes if the frontend has them.
6. **Write the HTML** in the session's temp folder (scratchpad), as `WIREFRAME_<TOPIC_IN_UPPERCASE>.html`
   — that is where the artifact is published from (step 7). Requirements:
   - **One block per state** from the list in step 1, navigable (tabs/anchors) — evaluating only the happy path
     hides exactly what still needs to be decided.
   - **Minimal interactivity** that helps the evaluation (open modal, switch tab, expand section), in simple inline
     JS — no framework, no dependency the real component doesn't use.
   - **Discreet annotations** (numbered) linking each region to the requirement of the goal it serves,
     with a legend showing the real component and the `path` — it is the link `/codeform:plan` reuses.
   - `🆕 NEW` regions with their own visual mark (dashed border + badge) and a legend of what is missing.
   - Real product copy (in the product's language — e.g. pt-BR with accents), example data **plausible for the domain** — never
     "Lorem ipsum" or `Item 1/Item 2`.
   - Basic accessibility preserved (roles, visible focus, contrast), because the real component
     has it and the design must reflect it.
7. **Publish the artifact** with the `Artifact` tool pointing to the scratchpad HTML (private by
   default), with an `icon` and a one-sentence `description`. **Then copy the same HTML** to
   `$D/WIREFRAME_<TOPIC_IN_UPPERCASE>.html` (methodology §2, "Wireframe"; `$D` from the brief): it is the source that
   allows editing the wireframe in the future, outside the versioned tree.
   For adjustments, **edit that copy and republish it passing the artifact's `url`** (same
   URL) — never create a new artifact each round. **The artifact URL is a deliverable**: state it in the
   report, along with the `.html` path — `/codeform:plan` records it in the "Wireframe" section of the
   project context, and `/codeform:execute` (with its reviewer) start from it. If the project context already
   exists, confirm that its "Wireframe" section has this URL.
8. **Check before delivering:** re-read the produced HTML against the inventory from step 4 — does each region
   have the declared component, classes and tokens? If something was drawn "from memory", fix it
   now or mark it honestly in the legend. Say in the report what you could **not** reproduce faithfully
   (e.g. a component that depends on live data, rich editor, canvas) and why.
9. **Report in the chat** (§0.8): (1) artifact link and `.html` path; (2) what the design shows,
   per state; (3) final component inventory (real × `NEW`); (4) deviations between code and design
   system found; (5) decisions the user made with you; (6) next steps. The parent leads the approval gate.
10. **Iterate:** the user's adjustments arrive in your session; each one goes back through steps 2–8 only for what
    changed, followed by a new report to the parent.
</process>

<update_mode>
This playbook is also the wireframe's **update mechanism** during `/codeform:execute`: when the
implemented frontend diverges from the design (component adjustment, new state, decision made during
implementation), the execute agent follows this mode — in its own thread or via a child agent (§0.10). In this mode, **don't
start over**: read the artifact URL in the project context (section "Wireframe"), edit the existing `.html` source
(or rebuild it from the artifact, if missing) only where it changed, re-read the changed real components (steps 2 and 8), republish **at the same
URL** and say in the report what changed and why. The URL never changes.
</update_mode>

<guardrails>
- **No product code.** Frontend is read-only; the only files written are the
  wireframe's `.html` source in `$D` (and the published artifact).
- **Fidelity over beauty.** A component that looks right but doesn't match the real DOM/classes is a
  failure, even if the result pleases the eye.
- **Never draw as existing what doesn't exist** (`🆕 NEW` always explicit).
- **Pair Programming Contract** (§0 of the methodology): cite `path:line`; surface confusion before
  moving on; the design decision belongs to the user (§0.2).
- **No real data.** Don't put PII, customer names or production data in the wireframe examples.
- **It's not the spec.** The wireframe is input for a decision; acceptance criteria are still born in
  `/codeform:plan` (contract `tela` lines). A future divergence between wireframe and contract is resolved by the user, not silently.
- **Surface confusion, don't hide it** (§0.3): any ambiguity between goal, code and design
  system — at any step — is a reason to stop and ask **before** drawing the region.
</guardrails>

<gate>
Deliverable to the parent (Phase 2): artifact URL, path `$D/WIREFRAME_<TOPIC>.html`,
final inventory (real × `NEW`), deviations, the decisions the user made and what was not reproduced faithfully. **The parent leads the approval of
the design with the user**; the project context must contain the artifact link and the
`.html` name, and each component marked 🆕 NEW must appear in some contract.
</gate>
