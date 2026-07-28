---
title: AgentRefinery — Product Requirements Document
status: active
version: 0.1.0
created: 2026-07-27
updated: 2026-07-27
role: Single, self-contained reference for the entire AgentRefinery project —
  problem, concept, architecture, every command, every document, every
  decision and its rationale. Written so a reader (human or AI agent) with
  no memory of how this project came to be can pick it up and continue work
  correctly, without re-deriving anything from a prior conversation.
---

# AgentRefinery — Product Requirements Document

## 0. How to use this document

This is the authoritative, exhaustive specification of AgentRefinery. It
exists so the project can survive losing all conversational context — every
concept, decision, and rationale that shaped the project should be
recoverable from this file alone, cross-referenced against the actual
files in the repo (which remain the source of truth for exact current
wording of `README.md`, `DESIGN-NOTES.md`, `templates/*.md`, and
`skills/*/SKILL.md`).

- **`README.md`** is the pitch + quick reference: what AgentRefinery is,
  why the Rail matters, and a concise user manual for the 3 commands.
- **This document (`PRD.md`)** is the full requirements spec: every
  concept, every document's exact structure, every state machine, every
  decision made and why, and what's still open.
- **`DESIGN-NOTES.md`** is the working scratchpad where decisions were
  first drafted — largely superseded by this document now that the project
  has stabilized, but kept as a session-by-session log of how the design
  evolved.
- **`templates/*.md`** and **`skills/*/SKILL.md`** are the actual
  implementation artifacts. If this PRD and the implementation ever
  disagree, treat that as a bug to reconcile, not a signal that either one
  is automatically right — check which one is stale.

If you are an AI agent picking up this project cold: read this document in
full before touching any code or generating any Rail. Do not assume
anything not written down here or in the files it references — that
"assume nothing, document everything" rule is itself a project requirement
(see §14).

---

## 1. Problem statement — the gap this project fills

Today there are two ways to get an AI agent to execute a process
repeatedly:

- **Automate it fully** (script / workflow / RPA). This freezes quality at
  the day the automation was written. A more capable LLM underneath
  doesn't improve anything, because the logic no longer runs through the
  model — it's been compiled away into fixed code.
- **Re-prompt it each time** (a raw prompt, re-run from scratch every
  time). This scales with model capability — a better model does a better
  job — but sacrifices consistency. Every run can drift, skip critical
  steps, or interpret the same instructions differently, because nothing
  fixes the mandatory path.

Neither option lets you have both: a process that stays consistent every
time it runs, *and* that gets better every time a more capable model runs
it.

**AgentRefinery's answer**: compile a documented process into a guide that
fixes the mandatory path (what must always happen, and how it's verified)
while leaving open the zone where model judgment adds quality. The same
guide, run by an older or a newer LLM, follows the exact same path — but
the result improves with model capability, not with someone rewriting
code.

That guide is called a **Rail**. Producing Rails, and refining their
output through repeated runs, is the entire point of AgentRefinery.

---

## 2. The core concept: the Rail

### 2.1 Definition

A **Rail** is the product AgentRefinery generates for a given process. It
is not a single file — it's a bundle: 5 context documents plus a matched
pair of runnable Agent Skills (see §9, "Anatomy of a Rail," for the exact
file layout).

### 2.2 The metaphor and why the name matters

A physical rail fixes the path a train follows; it does not drive the
train. The engine supplies the power, the speed, and the judgment calls
within the boundaries the rail sets. Put a more capable engine on the same
rail and it runs faster and smoother — no new track required.

That is exactly the mechanism AgentRefinery implements: a Rail fixes the
mandatory path of a process (the **fixed core**, §4) while leaving a
**judgment zone** where the executing LLM's own judgment decides *how* to
carry out each step. A weaker model still completes every mandatory step
correctly, because the fixed core anchors it. A stronger model produces a
better result on the exact same path, because the judgment zone is where
its extra capability shows up.

### 2.3 The Rail is the product — not a byproduct

This is a specific, deliberate decision (see §3 for the naming history
that led here): **the Rail itself — the reusable, inheritable,
version-tracked context bundle — is what AgentRefinery exists to
generate.** The deliverables produced by *running* a Rail matter, but they
are not the product; they're the output of using the product. The product
is the guide that can be handed to a better model next month and get a
better result on the exact same path, without anyone touching its
definition.

### 2.4 Refinement — what happens when a Rail is run

Running a Rail is not a one-shot pass. Execution is incremental and
resumable by design — to survive interruptions (e.g. running out of
tokens mid-run), and, more importantly, so the same Rail can later be
handed to a different or more capable LLM to *complement or improve* a
prior result, rather than starting over. See §10 for the full mechanics
(tracking files, state machines, Changelog).

Each pass over a Rail is a refining pass, the way raw material is refined
in stages rather than converted in one step. This is the "Refinery" in
AgentRefinery: not the one-time act of building a Rail, but the repeated
act of refining its output by running it again, pass after pass, as
models improve. Concretely, refinement is made auditable by:

- Tracking, per step, **which model/agent executed it** — this is what
  makes "increasingly capable LLMs improve the same result" a checkable
  claim instead of a slogan.
- Never overwriting prior deliverables on a repeat pass — only extending
  them.
- Permanently logging every improvement pass (which agent, what changed)
  in `Changelog.md`, so a Rail accumulates a visible history of how each
  generation of model improved on the last one.

---

## 3. Naming decisions — history and rationale

This section exists specifically to prevent re-litigating settled
decisions, and to preserve *why* they were made, not just what was
decided.

1. **Project renamed from "AgentRails" to "AgentRefinery."** "Rails" named
   the mechanism (the fixed-path guide), not the purpose. The real
   purpose is processes that get better every time a more capable LLM
   re-runs them — i.e., the repeated refinement, not the one-time
   mechanism. "AgentRefinery" names the purpose; "Rail" survives as the
   name of what it produces (see next point).

2. **Whether "Rail" (English) / "Riel" (Spanish) survives as a branded
   term went through three stages, in order:**
   - First draft: "Rails/Rieles" was to be explained once in the README
     as a conceptual origin, then retired from ongoing project vocabulary
     — no umbrella noun for "the produced artifact" was settled, defaulting
     to plain language like "the generated process."
   - **Reversed**: "Riel" (then "Rail") was confirmed as the *active,
     permanent operational term* for the produced artifact — not retired
     branding. The reasoning: the produced artifact needed a name, and
     "Rail" is literally more precise than plain language, since the
     rail/track metaphor *is* the mechanism (fixed core = the track,
     judgment zone = the engine's freedom). This resolved the earlier open
     item of "no umbrella noun for the produced artifact."
   - **Corrected to English-only**: the term was initially written in
     Spanish ("Riel"/"Rieles") in some drafts. Since all project documents
     are English-only (see §14), the term was corrected to its English
     form: **Rail** (singular), **Rails** (plural). This is now final and
     consistent across every file in the repo.

3. **`agentrefinery-buildvalidation` → `agentrefinery-build-validation`.**
   The original command name concatenated "build" and "validation"
   without a separator. Corrected so every word in a command name is
   hyphen-separated, matching the convention already used everywhere else
   (`agentrefinery-design`, `agentrefinery-build`, `process-name`,
   `process-name-validation`). This was a pure naming-consistency fix, not
   a behavior change — the skill's directory and all cross-references
   were renamed together in the same pass.

4. **"Núcleo fijo" / "zona de criterio" → "fixed core" / "judgment
   zone."** These Spanish-language concept terms were used in early
   drafts to describe the per-step split that is the actual mechanism
   behind a Rail (§4). They were translated to English for the same
   reason as point 2 — no exceptions to the English-only rule, even for
   terms that started as Spanish shorthand during design discussions.
   Bold emphasis on the terms was preserved through the translation.

---

## 4. The core mechanism: fixed core vs. judgment zone

This is the single mechanism that makes a Rail different from both a
rigid workflow and a raw prompt. Per step in a Rail's `Workflow.md`, split
the step into two parts:

- **Fixed core** — the invariant action, plus a concrete, checkable way to
  verify it before moving on. Worded so the verification gives the same
  answer no matter which model runs it. This guarantees the same path is
  followed regardless of model capability.
- **Judgment zone** — where the executing LLM's judgment operates. A more
  capable model produces better quality *here*, without deviating from
  the path, because the fixed core still anchors it. May be empty for a
  fully mechanical step, but most steps should have one.

This is exactly what a rigid workflow lacks (no judgment zone —
everything is fixed, so a more capable model can't improve anything) and
what a raw prompt lacks (no fixed core — everything is open, so
consistency isn't guaranteed).

---

## 5. The 5 defining principles of a Rail

Every Rail, regardless of the process it encodes, satisfies these five
properties. They are the acceptance criteria for "is this actually a
Rail" as opposed to a workflow or a prompt:

1. **Guides, doesn't dictate the exact "how"** — mandatory steps and hard
   limits are fixed, but the agent decides execution details within each
   step (the judgment zone).
2. **Verifiable, not just descriptive** — each step has a concrete way to
   confirm it was satisfied before moving on (the fixed core's
   verification).
3. **Resolves ambiguity explicitly** — when instructions conflict, the
   Rail's `Readme.md` states what wins (see §8.6, precedence order).
4. **Has a declared escape point** — when the executing agent can't
   comply with a step, it stops and asks the user. It never guesses and
   moves on. This is a fixed, non-negotiable rule across every Rail (see
   §8.6, escalation rule) — it is not something a specific process's
   design can override.
5. **Explicit scope** — the Rail states when it applies and when it
   doesn't, so the executing agent can recognize being asked to run it on
   something out of scope.

---

## 6. System architecture — the 3-command pipeline

AgentRefinery is not a single tool that produces one artifact once — it's
a small pipeline of 3 meta-commands, each delivered as an Agent Skill,
that together turn a process description into a Rail:

```
User: prompt + supporting data describing the process
        │
        ▼
agentrefinery-design      ← the ONLY step needing LLM/agent reasoning.
(LLM-driven)                 Detects ambiguity, classifies fixed core vs.
                              judgment zone, resolves ambiguity up front.
        │
        ▼
   Standardized context documents (draft)
        │
   ┌────┴────┐
   ▼         ▼
agentrefinery-build   agentrefinery-build-validation
(deterministic         (deterministic — consumes Validation.md + Backbone.md
 — everything           as specific input, not a generic validation)
 already disambiguated)
   │                     │
   ▼                     ▼
process-name        process-name-validation
(runs the Rail)      (validates a completed run against the Rail)
```

**Key architectural decision**: only `agentrefinery-design` requires model
judgment. `agentrefinery-build` and `agentrefinery-build-validation` are
mechanical transformations, because by the time they run, all ambiguity
has already been resolved by `agentrefinery-design`. This is why they can
be "fixed workflows" without contradicting the "a Rail is not a rigid
workflow" philosophy from §4 — the rigidity applies to *this
transformation step* (building the Rail's runnable skills), not to how
the end agent executes the process once built. The process execution
itself (`process-name`) still has a judgment zone per step; the *build
tooling* does not, because building is not the process — it's mechanical
packaging of an already-disambiguated spec.

---

## 7. Command specifications

The concise version of this section lives in `README.md`'s "User manual."
This section is the fuller reference; the actual runtime instructions
live in each command's `SKILL.md` (`skills/<name>/SKILL.md`) — if this
section and a `SKILL.md` ever disagree, the `SKILL.md` is what actually
executes and should be treated as current; update this PRD to match.

### 7.1 `agentrefinery-design`

**File**: `skills/agentrefinery-design/SKILL.md`. **Judgment**: yes — the
only command in the pipeline that requires LLM reasoning.

**Purpose**: turn a process description into a draft Rail context bundle
(5 documents), resolving all ambiguity up front so the two build commands
downstream can be purely mechanical.

**Inputs**:
1. Process description (prompt + supporting data) — required. **A PRD is
   an excellent process description on its own.** A well-written Product
   Requirements Document already states objectives, constraints, and
   scope — most of what `Backbone.md` needs — so it can be handed to
   `agentrefinery-design` directly, rather than rewritten into a prompt
   first. The Rail produced from it, once built (§7.2, §7.3) and run
   (§10), is what actually constructs whatever the PRD describes. This
   document (`PRD.md`) is itself an example of the kind of document that
   works well as design input for some other Rail, elsewhere — not
   because it's a PRD *about* AgentRefinery, but because of the shape a
   PRD naturally has (objectives, constraints, scope already separated
   out).
2. `process-name` — required identifier; if not given, ask the user, never
   invent one (it's a durable identifier other Rails may inherit from
   later).
3. Bundles to merge (optional) — local paths and/or Git URLs pointing to
   already-built Rails' `context/` directories, for inheritance.
4. Output location — ask if not obvious from context.

**Disambiguating unlabeled filesystem paths**: an invocation may pass
several bare paths after the process description and `process-name`
without saying which is the output location and which is a bundle to
merge. Resolve each one by inspection, not by position: a path that
already contains a `context/` subdirectory with the 5 Rail documents is a
**bundle to merge**; a path that doesn't is the **output location**. If a
path can't be checked this way (e.g. it doesn't exist yet), ask the user
rather than assume.

**Example invocation**:
```
/agentrefinery-design PRD.md standard-builder /home/Projects/Code/ECC/ /home/Projects/Code/ECCStandards/
```
- `PRD.md` — process description.
- `standard-builder` — `process-name`.
- `/home/Projects/Code/ECC/` — no `context/` bundle inside it yet → output
  location; `standard-builder/` gets created here.
- `/home/Projects/Code/ECCStandards/` — already has a `context/` bundle
  (an existing Rail encoding this org's coding standards) → bundle to
  merge; `standard-builder` inherits from it.

**Behavior, in order**:
1. Read all 5 files in `templates/` at the AgentRefinery repo root
   (`Design.md`, `Backbone.md`, `Workflow.md`, `Validation.md`,
   `Readme.md`) — these define the required frontmatter and section
   skeleton for each document type. Each template's own `TEMPLATE
   INSTRUCTIONS` comment block is guidance for the generator and must
   never leak into the generated output.
2. If bundles to merge were given: read each one's `context/*.md`. If any
   two contradict each other (conflicting objectives, hard limits, etc.),
   **stop and report the full list of contradictions** — no automatic
   resolution, no partial merge; reconciling input contexts is the user's
   responsibility. If consistent, merge by union following the
   inheritance pattern (base Rail's context first, this Rail's context as
   a specific application layered on top).
3. Draft the 5 documents **in this order**, since each depends on IDs
   defined in the one(s) before it:
   - **Backbone.md** — extract positive objectives (IDs `O1, O2, ...`) and
     hard limits as negative objectives (IDs `L1, L2, ...`). Every
     objective/limit must be independently verifiable; if something can't
     be made checkable, that is the ambiguity to resolve now (step 4
     below), not later.
   - **Workflow.md** — derive the fixed step sequence from Backbone.md.
     Each step cites the Backbone ID(s) it fulfills/guards, and is split
     into fixed core + judgment zone (§4). A step whose fixed core can't
     be phrased as a concrete check is unresolved ambiguity — sharpen the
     underlying Backbone objective rather than writing a vague check.
   - **Validation.md** — derive a checklist from Backbone.md +
     Workflow.md. Each item cites Backbone refs and the Workflow step(s)
     it validates, and states a concrete pass/fail condition checkable
     against the eventual output — not a restatement of the objective.
     Every Backbone ID touched by Workflow.md should be covered by at
     least one checklist item.
   - **Design.md** — descriptive big-picture overview + a Mermaid
     flowchart summarizing Workflow.md's steps. Written for a human
     reader who hasn't read Backbone.md yet. Explicitly
     non-authoritative — never introduces an objective, limit, or step
     not already in Backbone.md/Workflow.md.
   - **Readme.md** — meta-instructions: the standard precedence order
     (Backbone > Workflow > Validation > Design), the fixed escalation
     rule (stop and ask the user — not a per-process choice), and this
     Rail's explicit scope.
4. **Ambiguity handling** (applies throughout step 3, not just at the
   end): whenever source material is ambiguous or underspecified, resolve
   it before drafting the next document — either state a reasonable
   assumption inline, visibly (not buried, so a reviewer will notice), or
   ask the user directly if guessing risks producing an incorrect
   Backbone objective or hard limit. Never silently pick an interpretation
   and move on without surfacing it.
5. Fill every document's YAML frontmatter (see §8.1 for the shared
   schema).
6. Write the 5 files to `<process-name>/context/`.
7. Tell the user the bundle is a **draft for review** — recommend reading
   `Backbone.md` first (the source of truth) before running
   `agentrefinery-build`. Do **not** auto-chain into `agentrefinery-build`
   — the point of stopping here is for the user to review disambiguation
   decisions.

**Out of scope**: scaffolding `process-name/SKILL.md` or
`process-name-validation/SKILL.md` — that belongs to the two build
commands.

### 7.2 `agentrefinery-build`

**File**: `skills/agentrefinery-build/SKILL.md`. **Judgment**: no —
deterministic, mechanical transformation only.

**Purpose**: compile an already-disambiguated Rail context bundle into
the runnable `process-name` skill that executes the process.

**Hard prerequisite**: the `skill-creator` skill
(https://claude.com/plugins/skill-creator) must be available.
`agentrefinery-build` must **always** scaffold `process-name/SKILL.md`
through `skill-creator` — it never hand-rolls `SKILL.md` packaging
itself. If `skill-creator` is unavailable, it stops and says so rather
than improvising a substitute. This keeps every Rail-produced skill
consistent with the same packaging conventions as every other Agent
Skill.

**Required inputs**:
- `<process-name>/context/Backbone.md`
- `<process-name>/context/Workflow.md`
- `<process-name>/context/Readme.md`
(`Design.md` and `Validation.md` are not needed — Design.md is for human
orientation only, Validation.md belongs to `agentrefinery-build-validation`.)

**Preconditions**: all 3 required files exist; every step in
`Workflow.md` cites Backbone IDs that actually exist. If a step doesn't
trace back to Backbone.md, that's a defect from the design phase — stop
and report it, don't silently drop or fix it here.

**Behavior**:
1. Read `Backbone.md`, `Workflow.md`, `Readme.md`; verify traceability.
2. Compose the content to hand to `skill-creator` as the target skill's
   required behavior:
   - **The fixed step sequence** — `Workflow.md`'s steps transcribed
     verbatim (order, fixed core / judgment zone split, Backbone refs).
     The executing agent follows this sequence exactly; it never invents,
     skips, or reorders steps. `Readme.md`'s escalation rule is carried
     into the generated skill's text.
   - **`ProcessTracking.md` generation** — see §10.1 for the schema and
     §10.2 for the full state machine the generated skill must implement.
   - **`Changelog.md` logging** — see §10.4.
3. Invoke `skill-creator` with: name = `<process-name>`; description =
   what the process does (from Backbone's objectives) plus a trigger
   phrase (e.g. "runs the `<process-name>` process end to end,
   progressively and resumably; use when the user wants to
   execute/continue/improve the `<process-name>` process"); output path =
   `<process-name>/process-name/SKILL.md`.
4. Confirm to the user that the skill is ready to run, and remind them
   `agentrefinery-build-validation` still needs to run to complete this
   Rail's execution/validation cycle.

### 7.3 `agentrefinery-build-validation`

**File**: `skills/agentrefinery-build-validation/SKILL.md`.
**Judgment**: no — deterministic, mechanical transformation only. (See
§3.3 for why this command's name is fully hyphen-separated.)

**Purpose**: compile a Rail's checklist into the runnable
`process-name-validation` skill that checks a completed (or in-progress)
`process-name` run against that checklist.

**Hard prerequisite**: same as `agentrefinery-build` — must scaffold
through `skill-creator`; stop and say so if unavailable.

**Required inputs**:
- `<process-name>/context/Validation.md`
- `<process-name>/context/Backbone.md`
(`Design.md`, `Workflow.md`, `Readme.md` are not needed directly —
`Validation.md` was already derived from `Workflow.md` during design.
`Readme.md`'s escalation rule still gets carried into the generated
skill's text.)

**Preconditions**: both required files exist; every checklist item in
`Validation.md` cites Backbone IDs that actually exist. Does not depend
on `agentrefinery-build` having already run — the two build commands can
run in either order — though *using* the resulting
`process-name-validation` skill obviously requires `process-name` to have
produced output first.

**Behavior**:
1. Read `Validation.md`, `Backbone.md`; verify traceability. If a
   checklist item isn't concretely checkable, stop and send it back to
   `agentrefinery-design` rather than softening it into something that
   passes mechanically.
2. Compose the content to hand to `skill-creator`:
   - **The checklist** — `Validation.md`'s items transcribed verbatim
     (Backbone refs, Workflow step(s) validated, concrete pass/fail
     condition). Checked against actual deliverables in
     `output-process-name/` — never invents criteria beyond what
     `Validation.md` specifies.
   - **`ValidationTracking.md` generation** — see §10.1 for schema
     (STEP column seeded from `ProcessTracking.md`) and §10.3 for the
     full state machine the generated skill must implement.
   - **Escalation** — if the validation skill itself can't determine
     whether an item passes (a judgment call `Validation.md` didn't
     anticipate), it stops and asks the user rather than guessing
     pass/fail.
3. Invoke `skill-creator` with: name = `<process-name>-validation`;
   description = what it validates plus a trigger phrase; output path =
   `<process-name>/process-name-validation/SKILL.md`.
4. Confirm to the user that this Rail's execution/validation cycle is
   complete: `process-name` runs the process, `process-name-validation`
   checks it, and running either again — potentially with a more capable
   model — is how the Rail keeps improving without anyone touching its
   context documents again.

### 7.4 What comes after both build commands

`<process-name>/process-name/SKILL.md` and
`<process-name>/process-name-validation/SKILL.md` are themselves Agent
Skills — install them into whichever platform's skills directory applies
(§12) and invoke them directly to run and validate the process. Neither
is part of the AgentRefinery pipeline itself; they are what the pipeline
produces, and they persist independently of AgentRefinery after that.

---

## 8. The Rail context bundle — the 5 documents

Every Rail's `context/` directory contains exactly 5 documents, always
generated in dependency order (Backbone → Workflow → Validation → Design
→ Readme, per §7.1). Each document type has a base pattern in
`templates/<Doc>.md` at the AgentRefinery repo root, which
`agentrefinery-design` fills in per-process.

### 8.1 Shared frontmatter schema

All 5 documents (in both the `templates/` base pattern and any generated
Rail's `context/`) start with this YAML frontmatter:

```yaml
---
title: <Doc type> — <one-line title> (<process-name>)
status: draft | active | deprecated
version: <semver-ish, e.g. 0.0.1>
created: <yyyy-mm-dd>
updated: <yyyy-mm-dd>
role: <one-line purpose of this specific document>
derived-from: <parent template version, and/or parent Rail's document + version if built via inheritance>
regeneration-rule: <when/how this doc must be regenerated if its source changes>
---
```

- `status` is an enum: `draft` (not yet reviewed/approved) → `active`
  (in use) → `deprecated`.
- `derived-from` does double duty: it points to the base template
  pattern, and/or to a parent Rail's document when built via inheritance
  (§11) — potentially both at once.

### 8.2 `Backbone.md` — single source of truth

**Role**: the objectives the process must achieve (positive) and the hard
limits it must never violate (negative objectives). Everything else in
the bundle derives from this document, and if any other document ever
conflicts with it, Backbone.md wins (§8.6).

**Structure**:
- **Objectives** — numbered `O1, O2, ...`. Each must be independently
  verifiable; an objective that can't be made checkable either gets
  sharpened until it can be, or gets moved to Design.md as background
  context instead of being treated as a real objective.
- **Hard limits (negative objectives)** — numbered `L1, L2, ...`. What
  the process must never do, regardless of how a step's judgment zone is
  exercised. Hard limits always win over objectives if the two ever
  conflict.
- **Traceability** — every row in Workflow.md and every checklist item in
  Validation.md must cite at least one Backbone ID. An ID never
  referenced downstream signals a redundant objective or a dropped
  reference — worth investigating either way.

**Regeneration rule**: regenerate whenever the process description or
scope changes. Workflow.md and Validation.md must then be regenerated
afterward, in that order, since both derive from Backbone.md.

### 8.3 `Workflow.md` — the fixed step sequence

**Role**: the fixed order of steps the executing agent must follow.
Steps are derived from Backbone.md; the agent must not invent, skip, or
reorder them.

**Structure, per step**:
- **Backbone refs** — the `O#`/`L#` ID(s) this step fulfills or guards. A
  step with no citation signals a skipped objective or an unauthorized
  invented step.
- **Fixed core** — the invariant action, plus a concrete, checkable
  verification. Must produce the same outcome regardless of which model
  runs it — phrased as a concrete condition, not "make sure it's good."
- **Judgment zone** — what's explicitly left to the executing agent's
  judgment within this step; may be empty for a fully mechanical step,
  but most steps should have one.

A step whose fixed core can't be stated as verifiable is unresolved
ambiguity — it gets sent back to Backbone.md rather than guessed at
here.

**Escalation** (also restated in Readme.md, §8.6): if a step's fixed core
cannot be satisfied as written, the agent stops and asks the user. It
does not silently skip the step or substitute its own judgment for the
fixed core.

**Regeneration rule**: regenerate whenever Backbone.md changes.
Validation.md must be regenerated afterward, since its checklist is keyed
to Workflow.md's steps.

### 8.4 `Validation.md` — the checklist

**Role**: confirms a Workflow.md run was correctly and completely
executed. Consumed by `process-name-validation` (built by
`agentrefinery-build-validation`) to check `output-process-name/` against
Backbone.md's objectives and hard limits.

**Structure, per checklist item**:
- **Backbone refs** — the `O#`/`L#` ID(s) checked.
- **Workflow step(s)** — which step(s) this item validates. (Three-way
  traceability: Backbone ↔ Workflow ↔ Validation.)
- **Check** — a concrete, checkable pass/fail condition against the
  actual output in `output-process-name/`. "O3 was addressed" is not
  checkable; "the deliverable contains a section titled X with at least
  one entry per Y" is. Hard limits get checked too — validation must be
  able to catch a violation, not just confirm objectives were met.

This document only defines *what* to check — it does not track status
itself. Runtime status tracking is `ValidationTracking.md`'s job (§10.1),
seeded from this checklist's item list.

**Regeneration rule**: regenerate whenever Backbone.md or Workflow.md
changes.

### 8.5 `Design.md` — descriptive overview (non-authoritative)

**Role**: big-picture overview and diagrams, for human/agent orientation.
Descriptive only — if it ever conflicts with Backbone.md, Backbone.md
wins (§8.6).

**Structure**:
- **What this process is for** — 1–3 paragraphs: purpose, audience, why
  it exists as a Rail rather than a one-off script or a raw prompt.
- **How it flows** — plain-language walkthrough referencing Workflow.md's
  steps by name/number, without duplicating their fixed core / judgment
  zone detail. Includes a Mermaid flowchart (plain text, so the whole
  context bundle stays diffable and git-friendly — no binary/image
  assets).
- **Context and constraints** — background a reader needs to make sense
  of Backbone.md's objectives and hard limits: prior decisions, why a
  limit exists, what was tried and rejected. Explanatory, not normative.

This is the only one of the 5 documents meant to be read by a human
first, an agent second — write it so a newcomer understands the process
without needing to read Backbone.md first.

**Regeneration rule**: regenerate whenever the process's scope or shape
changes enough that the overview/diagrams would mislead a new reader.

### 8.6 `Readme.md` — meta-instructions

**Role**: how to read the rest of the bundle — what wins when documents
conflict, and what to do when the agent can't comply with something in
them. Read first, before the other four. Not itself a source of
objectives, steps, or checks.

**Precedence order when documents conflict** (the default; only deviate
if a specific process has a genuine reason to, stated inline):
1. **Backbone.md** — always wins. Single source of truth.
2. **Workflow.md** — wins over Validation.md and Design.md (direct
   execution derivation of Backbone.md).
3. **Validation.md** — wins over Design.md.
4. **Design.md** — descriptive only; never overrides the other three.

A conflict between Backbone.md and any other document means that other
document is stale and must be regenerated — it is not grounds for the
agent to pick a side and proceed.

**Escalation rule** (fixed across every Rail, not a per-process choice):
if the executing agent cannot comply with a step's fixed core — blocked,
contradicted by real conditions, missing a precondition, or facing
unresolved ambiguity — **it stops and asks the user.** It does not guess,
does not silently skip the step, and does not substitute its own judgment
for what the fixed core requires.

**Scope**: states explicitly when this Rail applies and when it doesn't,
so the executing agent can recognize being asked to run it on something
out of scope.

**Regeneration rule**: regenerate only if the precedence rule or
escalation rule themselves change — not on every Backbone.md/Workflow.md
edit.

---

## 9. Anatomy of a Rail — full directory layout

```
<process-name>/
├── context/                        ← shared by both build commands
│   ├── Design.md       — big-picture overview + diagrams
│   ├── Backbone.md     — objectives (positive) AND hard limits ("negative
│   │                     objectives") — single source of truth
│   ├── Workflow.md     — fixed step sequence derived from Backbone.md
│   ├── Validation.md   — checklist derived from Backbone.md to confirm the
│   │                     workflow was correctly and completely executed
│   └── Readme.md       — meta-instructions: ambiguity-resolution precedence,
│                          escalation rule (stop and ask the user)
├── output-process-name/            ← runtime output of running the Rail
│   ├── ProcessTracking.md          — per-step status, incl. which agent ran it
│   ├── ValidationTracking.md       — validation pass status, same schema
│   ├── Changelog.md                — permanent log of every refinement pass
│   └── (actual process deliverables)
├── process-name/SKILL.md              (runs the process)
└── process-name-validation/SKILL.md   (validates a completed run)
```

Note the `process-name/` subdirectory shares its name with the outer
`<process-name>/` bundle directory — this is intentional, not a naming
bug: the outer directory is the whole Rail bundle, while the inner
`process-name/` directory is the actual Agent Skill package, structured
so it can be copied as-is into a platform's skills directory (e.g.
`.claude/skills/process-name/`) without renaming.

---

## 10. Progressive execution

Running `process-name` is **not all-or-nothing**. It's incremental and
resumable by design — to survive interruptions (e.g. running out of
tokens), and to let the same process be re-run later by a
different/more-capable LLM to complement or improve a prior result,
rather than starting over. `process-name-validation` mirrors this same
resumability logic against its own tracking file.

### 10.1 Tracking file schema

Both `ProcessTracking.md` and `ValidationTracking.md` (in
`output-process-name/`) share the same table schema:

`STATUS | AGENT | STEP | DETAILS | START | END`

- **STATUS**: `✅` done, `❌` error/blocked, *(empty)* pending.
- **AGENT**: which model/agent executed that step — this is what makes
  the "increasingly capable LLMs improve the same path" thesis auditable
  over time.
- **STEP**: short step name.
- **DETAILS**: problems found / notes; empty if none.
- **START / END**: timestamps.

**Operational rule for both files**: generate the file *before* starting;
if it already exists, resume at the first row with empty STATUS. Never
hold the file open for the whole run — write/flush per step, before
moving to the next one, so progress is visible live, not only at the
end.

- **`ProcessTracking.md`** (owned by `process-name`) — task list
  generated from `Workflow.md`, written with empty STATUS before
  execution starts.
- **`ValidationTracking.md`** (owned by `process-name-validation`) — STEP
  column seeded by copying it from `ProcessTracking.md` (one row per
  process step, not per checklist item, so both files stay aligned); the
  rest of the columns are filled in during validation. Doubles as an
  error/improvement log from the validation pass.

### 10.2 State machine for `process-name`

```
if ProcessTracking.md doesn't exist:
    generate task list from Workflow.md -> write with empty STATUS

# Phase 1 — advance pending steps
while a row has empty STATUS:
    execute it (START -> do -> END + STATUS + DETAILS)

# Phase 2 — resolve own flagged errors
while a row has STATUS = error or non-empty DETAILS:
    retry that step, update STATUS/DETAILS/END

# Phase 3 — consume feedback from a prior validation run
if ValidationTracking.md exists:
    for each row with non-empty DETAILS (gap/error reported by validation):
        re-execute the corresponding process step to resolve it
        # process-name never writes to ValidationTracking.md. It only reads it.
        # Only process-name-validation writes/clears its own file, on its own
        # next run, once it re-confirms the step is fixed.

# Phase 4 — everything resolved on both sides -> offer another pass
if ProcessTracking.md fully OK and (ValidationTracking.md doesn't exist or fully OK):
    ask user: "run the process again to complement/improve the result?"
    if yes:
        clear STATUS/START/END/DETAILS in BOTH tracking files
        (do NOT delete output-process-name/ contents)
        re-run from Phase 1 — complement existing deliverables, never overwrite
        append one entry to Changelog.md: which LLM/agent ran this pass + a
        brief description of what was complemented/improved
        on completion, show:
          "Complement and correction pass finished — to properly close this
           cycle, run the process validation again."
    if no:
        stop
```

### 10.3 State machine for `process-name-validation`

Mirrors `process-name`'s resumability logic, but validating instead of
executing — using `ProcessTracking.md`'s step list and `Validation.md`'s
criteria as its guide, and `output-process-name/` contents as what gets
validated:

```
if ValidationTracking.md doesn't exist:
    seed STEP column from ProcessTracking.md -> write with empty STATUS

while a row has empty STATUS:
    validate the corresponding step's output in output-process-name/
    against the relevant Validation.md checklist item(s)
    (START -> check -> END + STATUS + DETAILS)

while a row has STATUS = error / non-empty DETAILS from a PRIOR validation
pass that hasn't been re-checked yet:
    re-check whether process-name has since fixed it
    (process-name reads this file but never writes to it — only this
    skill writes/clears its own rows, on its own next run, once it
    re-confirms a step is fixed)

if every row is confirmed OK:
    report: this Rail's output is fully validated
else:
    report: which steps still have gaps, so the user can re-run
    process-name to consume this feedback (Phase 3 of its own state
    machine) and resolve them
```

### 10.4 `Changelog.md`

- Lives in `output-process-name/`, alongside the tracking files — it
  documents *execution* history, not document history.
- Logs, per improvement pass (Phase 4 "yes" branch of §10.2): which
  LLM/agent ran it, plus a brief description of what was
  complemented/improved.
- **Never cleared** on a Phase 4 reset — it's the permanent, accumulating
  record of every improvement pass across models and time. This is the
  concrete artifact that makes the "Refinery" claim (§2.4) checkable: you
  can read `Changelog.md` and see the actual sequence of models that
  improved a Rail's output over time.
- Format: simple table, no YAML frontmatter — it's an append-only log, not
  a versioned document. Columns: `DATE | AGENT | SUMMARY`.

---

## 11. Merging / inheriting pre-existing Rails

There is no separate command for this — it's an additional parameter set
accepted by `agentrefinery-design` itself (§7.1, input 3): a list of local
paths and/or Git repository URLs, each pointing to an already-built
Rail's `context/` bundle.

- **On contradiction**: if the N input bundles contain conflicting
  objectives, hard limits, etc., `agentrefinery-design` **reports the
  full list of contradictions and terminates.** No automatic conflict
  resolution, no partial merge. Providing coherent input contexts is
  explicitly the user's responsibility.
- **Recommended pattern — inheritance**: build an agnostic/base Rail's
  context first, then a child Rail whose context is a specific
  application of it. Merging a base Rail's Backbone/Validation with a
  child's is the natural, low-contradiction case, since a child extends
  rather than restates the base.
- **Implementation note**: supporting Git URLs means `agentrefinery-design`
  needs to be able to clone/read remote repositories at runtime.

---

## 12. Cross-platform delivery

Confirmed via research at design time: Claude Code, Google Antigravity,
Kimi Code CLI, and ZCode all converged on the same open **Agent Skills**
format — a directory with a `SKILL.md` (frontmatter metadata +
instructions) plus optional supporting files, loaded on demand. No
per-platform adapter logic is needed; only the install path differs:

- **Claude Code**: `.claude/skills/`
- **Google Antigravity**: `.agent/skills/` (workspace) or
  `~/.gemini/antigravity/skills/` (global)
- **Kimi Code CLI**: discovered via layered skill roots
- **ZCode**: plugin/skills directory (can also bundle into a plugin)

Because every generated `process-name/SKILL.md` and
`process-name-validation/SKILL.md` is a standard Agent Skill package (via
the `skill-creator` hard prerequisite, §13), a Rail built once is
installable on any of these platforms without modification.

---

## 13. Minimum requirements / hard prerequisites

- **[skill-creator](https://claude.com/plugins/skill-creator)** —
  `agentrefinery-build` and `agentrefinery-build-validation` must always
  scaffold their target `SKILL.md` packages through `skill-creator`
  rather than hand-rolling them. This is a hard prerequisite for both
  commands; if it's unavailable, they stop rather than improvise.
- **A target platform that supports Agent Skills** — see §12 for the
  list and install paths.

---

## 14. Documentation & language standards (operating principles)

These are standing rules for the project as a whole, not specific to any
one document — restated here explicitly because they were established
through direct correction during the project's design conversations and
are easy to silently regress on without a written rule to check against.

1. **English only, everywhere.** Every generated document — README, this
   PRD, `DESIGN-NOTES.md`, the 5 context-document types, every
   `SKILL.md`, and everything user-facing — is written in English, for
   open-scope reach. This applies even to terms that started as
   Spanish-language shorthand during design discussions (see §3, items 2
   and 4, for two concrete corrections made under this rule). There are
   no exceptions carved out for etymology notes, parenthetical asides, or
   internal design terms.
2. **Command names are fully hyphen-separated.** Every word in a
   command/skill name is separated by a hyphen — no concatenated compound
   words (`agentrefinery-build-validation`, not
   `agentrefinery-buildvalidation`). See §3, item 3.
3. **Assume nothing; document everything.** This project is built to be
   read and executed by AI agents, not only humans, and often by an agent
   with no memory of how a given Rail or command came to exist. Every
   command's required inputs, preconditions, exact output paths, and
   error-handling behavior must be spelled out explicitly — never left as
   something "an agent will obviously figure out." When something is
   ambiguous or missing, every command in this project stops and asks
   rather than guessing (this is also principle 3 and 4 of §5, applied
   reflexively to AgentRefinery's own tooling, not just to the Rails it
   produces).
4. **Bold emphasis is a formatting choice, independent of translation.**
   When correcting terminology (language or naming), preserve whatever
   bold/italic emphasis the term already had — the correction is to the
   words, not the formatting around them.

---

## 15. Repository structure of AgentRefinery itself

```
AgentRefinery/
├── README.md                              ← pitch + user manual + PRD pointer
├── PRD.md                                 ← this file — full requirements spec
├── DESIGN-NOTES.md                        ← working design log (session history)
├── skills/
│   ├── agentrefinery-design/SKILL.md
│   ├── agentrefinery-build/SKILL.md
│   └── agentrefinery-build-validation/SKILL.md
└── templates/                             ← base templates for the 5 context docs
    ├── Design.md
    ├── Backbone.md
    ├── Workflow.md
    ├── Validation.md
    └── Readme.md
```

This is the **builder repo** — it is not itself a Rail. It produces
Rails. Nothing in this repo is process-specific; `templates/` and
`skills/` are the fixed tooling, and every `<process-name>/` bundle it
generates lives outside this structure (wherever the user directs
`agentrefinery-design` to write it).

---

## 16. Glossary

- **Rail** (plural **Rails**) — the product: a context bundle (5
  documents) plus a matched pair of runnable Agent Skills, for a given
  `process-name`. See §2.
- **Fixed core** — the invariant, verifiable part of a Workflow step. See
  §4.
- **Judgment zone** — the part of a Workflow step left to the executing
  agent's judgment. See §4.
- **`process-name`** — the user-supplied, durable identifier for a
  specific Rail. Never invented by any command; always supplied or asked
  for.
- **`agentrefinery-design`** — the LLM-driven meta-command that produces
  a Rail's draft `context/` bundle. See §7.1.
- **`agentrefinery-build`** — the deterministic meta-command that
  compiles `context/` into the runnable `process-name` skill. See §7.2.
- **`agentrefinery-build-validation`** — the deterministic meta-command
  that compiles `context/` into the runnable `process-name-validation`
  skill. See §7.3.
- **Backbone.md** — a Rail's single source of truth: objectives and hard
  limits. See §8.2.
- **Workflow.md** — a Rail's fixed step sequence, derived from
  Backbone.md. See §8.3.
- **Validation.md** — a Rail's checklist, derived from Backbone.md +
  Workflow.md. See §8.4.
- **Design.md** — a Rail's descriptive, non-authoritative overview. See
  §8.5.
- **Readme.md** (inside a Rail's `context/`) — a Rail's meta-instructions:
  precedence order + escalation rule. Not to be confused with this
  repo's own root `README.md`. See §8.6.
- **`ProcessTracking.md`** — runtime, per-step status log owned by
  `process-name`. See §10.1–10.2.
- **`ValidationTracking.md`** — runtime, per-step status log owned by
  `process-name-validation`. See §10.1, §10.3.
- **`Changelog.md`** — permanent, append-only log of every refinement
  pass over a Rail. See §10.4.
- **Refinement / refining pass** — one run (or re-run) of `process-name`
  over a Rail, especially a repeat pass intended to complement or improve
  a prior result. See §2.4.
- **Escalation rule** — the fixed, non-negotiable rule that an executing
  agent stops and asks the user rather than guessing when it can't comply
  with a step. See §5 (principle 4) and §8.6.
- **skill-creator** — the external skill
  (https://claude.com/plugins/skill-creator) that both build commands
  must use to scaffold their target `SKILL.md` packages. Hard
  prerequisite. See §13.

---

## 17. Open items / roadmap

Carried forward from `DESIGN-NOTES.md`'s running list, current as of this
document's `updated` date:

1. **Nothing built so far has been exercised end-to-end.** The first real
   test should be running `agentrefinery-design` against a real, concrete
   process description, to see whether the generated `context/` bundle
   actually holds up (produces a coherent Backbone, a Workflow that
   traces cleanly to it, a Validation checklist that's genuinely
   checkable) before trusting the rest of the pipeline (`agentrefinery-build`,
   `agentrefinery-build-validation`, and the two generated skills they'd
   produce) with real use.
2. Everything else previously tracked as open in `DESIGN-NOTES.md`
   (umbrella noun for the produced artifact, README-vs-templates
   sequencing, the 3 meta-command skills) has been resolved — see §3 for
   the naming decisions and §6–§9 for the resulting architecture, all now
   implemented in `skills/` and `templates/`.
