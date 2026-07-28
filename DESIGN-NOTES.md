# AgentRefinery — Design Notes (work in progress)

> This file is the session-by-session working log of how the design evolved.
> For the current, consolidated, authoritative spec, see [`PRD.md`](./PRD.md)
> — it captures everything below plus the rationale for each decision, in one
> place.

Status: architecture agreed, no files/skills built yet. Project renamed from
"AgentRails" to **AgentRefinery** — "Rails" named the mechanism, not the purpose.
The real purpose is: **processes that get better every time a more capable LLM
re-runs them.**

**Decision (reversed from an earlier draft of this note): "Rail" is kept as the
active operational term.** It is not retired branding — it's the name of the
product AgentRefinery actually generates: the context bundle + paired skills for
a given `process-name` (see "The product" below) is **a Rail**. The root
`README.md` documents why the Rail matters as the real deliverable, and why
running an LLM over a Rail is what constitutes the refinement process — the
"Refinery" in AgentRefinery refers to that repeated refining, not just to the
one-time act of building the Rail.

Umbrella noun for "the produced artifact" is resolved: **Rail** (plural
**Rails**). This closes what was previously an open item.

## The gap this project fills

Today there are two ways to get an AI agent to execute a process repeatedly:

- **Automate it fully** (script/workflow/RPA) — freezes quality at the day it was written.
  A more capable LLM underneath doesn't improve anything, because the logic no longer runs
  through the model.
- **Re-prompt it each time** — scales with model capability, but sacrifices consistency.
  Every run can drift, skip critical steps, or be interpreted differently.

**AgentRefinery** compiles a documented process into a guide that fixes the mandatory
path (what must always happen, and how it's verified) while leaving open the zone
where model judgment adds quality. The same guide, run by an older or newer LLM,
follows the exact same path — but the result improves with model capability, not
with rewriting code. Repeated passes — potentially by increasingly capable models —
progressively refine the output, the way raw material is refined in stages. That
compounding refinement across executions is the actual point of the project; the
guide/path-fixing mechanism (formerly branded "Rails") is the tool that makes it
possible without drift.

## The underlying mechanism (5 defining principles) — what makes a Rail a Rail

To explain in the README as the conceptual origin, not as ongoing branding:

1. **Guides, doesn't dictate the exact "how"** — mandatory steps + hard limits, but the
   agent decides execution details within each step.
2. **Verifiable, not just descriptive** — each step has a way to confirm it was satisfied
   before moving on.
3. **Resolves ambiguity explicitly** — when instructions conflict, the guide says what wins.
4. **Has declared escape points** — what to do when the agent can't comply.
   **Decision: the agent stops and asks the user.**
5. **Explicit scope** — when the guide applies and when it doesn't.

### Fixed core vs. judgment zone (the key mechanism)

Per step, split:
- **Fixed core** — the invariant action + verification. Guarantees the same path is
  followed regardless of model capability.
- **Judgment zone** — where the LLM's judgment operates. A more capable model produces
  better quality *here*, without deviating from the path, because the fixed core still
  anchors it.

This is what a workflow lacks (no judgment zone — everything is fixed) and what a raw
prompt lacks (no fixed core — everything is open).

## Project shape: a Refinery, not a single output

AgentRefinery doesn't produce one specific artifact once — it's the factory that turns
a process description into what's needed to run that process N times with minimal
deviation, while allowing increasingly capable LLMs to improve output quality on the
same path, pass after pass.

### Pipeline (3 meta-commands, delivered as Agent Skills)

```
User: prompt + supporting data describing the process
        │
        ▼
agentrefinery-design      ← the ONLY step needing LLM/agent reasoning.
(LLM-driven)                 Detects ambiguity, classifies fixed core vs.
                              judgment zone, resolves ambiguity up front.
        │
        ▼
   Standardized context documents (draft) — see below
        │
   ┌────┴────┐
   ▼         ▼
agentrefinery-build   agentrefinery-build-validation
(deterministic         (deterministic workflow — consumes
 workflow — everything  Validation.md + Backbone.md, which are
 already disambiguated) provided as specific input, not a generic
                         validation)
   │                     │
   ▼                     ▼
process-name        process-name-validation
(skill that runs     (skill that validates a completed
 the process)          execution against Backbone/Validation)
```

Key point: only `agentrefinery-design` requires model judgment. `agentrefinery-build`
and `agentrefinery-build-validation` are mechanical transformations, because by the time
they run, all ambiguity has already been resolved — this is why they can be "fixed
workflows" without contradicting the "not a rigid workflow" philosophy (the rigidity
applies to *this transformation step*, not to how the end agent executes the process).

## The product: a Rail

For a given `process-name` (an identifier, itself one of the design inputs), the output —
a pair of commands + a shared context — is **a Rail**:

```
<process-name>/
├── context/                        ← shared by BOTH commands
│   ├── Design.md       — big-picture overview + Mermaid diagrams (plain text)
│   ├── Backbone.md     — objectives to fulfill (positive) AND hard limits as
│   │                     "negative objectives" — single source of truth
│   ├── Workflow.md     — fixed step sequence derived from Backbone.md; agent
│   │                     must follow it and not invent new steps
│   ├── Validation.md   — checklist derived from Backbone.md to confirm the
│   │                     workflow was correctly and completely executed
│   └── Readme.md       — meta-instructions: how the context is organized,
│                          ambiguity-resolution precedence rules, and the
│                          escalation rule (stop and ask the user)
├── output-process-name/            ← runtime output of running process-name
│   ├── ProcessTracking.md          — see "Progressive execution" below
│   ├── ValidationTracking.md       — see "Progressive execution" below
│   ├── Changelog.md                — see "Changelog.md" below
│   └── (actual process deliverables)
├── process-name/SKILL.md              (executes the process)
└── process-name-validation/SKILL.md   (validates a completed execution)
```

`agentrefinery-build` must always be executed through the `skill-creator` skill
(https://claude.com/plugins/skill-creator) to actually scaffold the `SKILL.md`
packages — AgentRefinery does not hand-roll the skill packaging itself. This means
**skill-creator is a hard prerequisite**, to be listed in the root README's minimum
requirements.

## Progressive execution (process-name / process-name-validation)

Running `process-name` is **not all-or-nothing**. It's incremental and resumable —
by design, to survive interruptions (e.g. running out of tokens), and to allow the
same process to be re-run later by a different/more capable LLM to complement or
improve a prior result, rather than starting over.

### Tracking files (in `output-process-name/`)

Both tracking files share the same table schema: `STATUS | AGENT | STEP | DETAILS | START | END`.

- **STATUS**: `✅` done, `❌` error/blocked, *(empty)* pending.
- **AGENT**: which model/agent executed that step — this is what makes the
  "increasingly capable LLMs improve the same path" thesis auditable over time.
- **STEP**: short step name.
- **DETAILS**: problems found / notes, empty if none.
- **START / END**: timestamps.

Operational rule for both files: generate the file **before** starting; if it
already exists, resume at the first row with empty STATUS. Do not hold the file open
for the whole run — write/flush per step, before moving to the next one, so progress
is visible live, not only at the end.

- **`ProcessTracking.md`** (owned by `process-name`) — task list generated from
  `Backbone.md`, written with empty STATUS before execution starts.
- **`ValidationTracking.md`** (owned by `process-name-validation`) — STEP column
  seeded by copying it from `ProcessTracking.md`; the rest of the columns are filled
  in during validation. Doubles as an error/improvement log from the validation pass.

### State machine for `process-name`

```
if ProcessTracking.md doesn't exist:
    generate task list from Backbone.md -> write with empty STATUS

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

`process-name-validation` mirrors this same resumability logic against its own
tracking file, using `ProcessTracking.md`'s step list (and `Validation.md`'s
criteria) as its guide, and using `output-process-name/` contents as what gets
validated.

### Merging N pre-existing context bundles

No new command — it's an additional parameter set on `agentrefinery-design` itself:
a list of local paths, and/or a list of Git repository URLs, each pointing to an
already-built context bundle (Design/Backbone/Workflow/Validation/Readme).

- If contradictions are found across the N bundles (conflicting objectives/hard
  limits/etc.): **report the list of contradictions and terminate.** No automatic
  conflict resolution, no partial merge. Providing coherent input contexts is
  explicitly the user's responsibility.
- Recommended pattern (documented in the root `README.md`, the audience invoking
  `agentrefinery-design`): build via **inheritance** — an agnostic/base project's
  context first, then a child project whose context is a specific application of
  that base. Merging the base's Backbone/Validation with the child's is the
  natural, low-contradiction case.
- Implementation note: supporting Git URLs means `agentrefinery-design` needs to be
  able to clone/read remote repos.

### Context document frontmatter

All 5 context documents (Design/Backbone/Workflow/Validation/Readme) start with a
YAML frontmatter header:

```yaml
---
title: Design — Project Overview (<process-name>)
status: draft
version: 0.0.2
created: 2026-07-16
updated: 2026-07-16
role: <one-line purpose of this specific document>
derived-from: <parent template and/or parent process context, with version>
regeneration-rule: <when/how this doc must be regenerated if its source changes>
---
```

- `status`: enum `draft | active | deprecated`.
- `derived-from` does double duty: points either to the base template pattern,
  or to a parent process's document when built via inheritance (see merging
  section above), or both.

### Changelog.md

- Lives in `output-process-name/`, alongside the tracking files — it documents
  execution history, not document history.
- Logs, per improvement pass (Phase 4 "yes" branch): which LLM/agent ran it + a
  brief description of what was complemented/improved.
- Never cleared on a Phase 4 reset — it's the permanent, accumulating record of
  every improvement pass across models/time.
- Format: simple table (no YAML frontmatter) — it's an append-only log, not a
  versioned document. Columns: `DATE | AGENT | SUMMARY`.

## Cross-platform delivery (confirmed via research)

Claude Code, Google Antigravity, Kimi Code CLI, and ZCode all converged on the same open
**Agent Skills** format: a directory with a `SKILL.md` (frontmatter metadata + instructions)
plus optional supporting files, loaded on demand. No per-platform adapter logic is needed —
only the install path differs:

- Claude Code: `.claude/skills/`
- Google Antigravity: `.agent/skills/` (workspace) or `~/.gemini/antigravity/skills/` (global)
- Kimi Code CLI: discovered via layered skill roots
- ZCode: plugin/skills directory (can also bundle into a plugin)

## Language

All generated documents (README, the 5 context docs, everything user-facing) are in
English, for open-scope reach.

## Proposed repo structure for AgentRefinery itself (the builder repo)

```
AgentRefinery/
├── README.md                              ← the gap + minimum requirements (English)
├── DESIGN-NOTES.md                        ← this file
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

## Open items for next session

1. ~~Confirm no replacement umbrella noun is needed for "the produced artifact"~~
   — resolved: the umbrella noun is **Rail** (see above).
2. ~~Decide starting point: root `README.md` vs. the 5 context-document
   templates first~~ — resolved: built `README.md` first, then
   `templates/{Design,Backbone,Workflow,Validation,Readme}.md`. Both now exist.
3. ~~The 3 meta-command skills~~ — done: `skills/agentrefinery-design/SKILL.md`,
   `skills/agentrefinery-build/SKILL.md`, `skills/agentrefinery-build-validation/SKILL.md`
   all exist. Both build/build-validation skills' instructions require going
   through `skill-creator` to scaffold their target `process-name`/
   `process-name-validation` packages, per the hard prerequisite.
4. Next: nothing built yet has been exercised end-to-end. First real test
   should be running `agentrefinery-design` against a real, concrete process
   description to see whether the generated context bundle actually holds up
   before trusting the pipeline further.

---

## 2026-07-28 — Split back into two sibling projects

Everything above this line describes the **original, combined design**:
one project, later renamed AgentRails → AgentRefinery, that tried to own
both "run this process consistently" and "improve this process's result
across repeated runs" inside a single Phase 4 state-machine step. That
combined design is now superseded — kept here only as historical record,
per this file's own "don't rewrite history" rule.

**What changed and why**: the combined Phase 4 (§ above: "clear STATUS/
START/END/DETAILS in BOTH tracking files... complement existing
deliverables, never overwrite... append to Changelog.md") conflated two
different concerns that turned out to need opposite re-run semantics. A
Rail needs a *destructive* restart to stay simple and verifiable regardless
of which model runs it. Deciding whether a fresh pass is an *improvement*
over the last one needs something that survives across restarts. Both
inside one step meant neither was clean.

**Resolution**: split into two sibling repos, at the same directory level.

- **AgentRails** (new sibling repo) takes everything above in this file
  that concerns the Rail mechanism itself: the fixed-core/judgment-zone
  split, the 3-command Rail-building pipeline (renamed
  `agentrails-design` / `agentrails-build` / `agentrails-build-
  validation`), the 5-document context bundle, `ProcessTracking.md` /
  `ValidationTracking.md`, and the escalation rule. Its own Phase 4 is now
  a plain, destructive restart: ask to re-run, and if yes, delete all of
  `output-process-name/` and start over from Phase 1. No more "complement,
  never overwrite," no more per-Rail `Changelog.md`.
- **AgentRefinery** (this repo) keeps only the refinement concern:
  comparing a Rail's output across N runs and deciding whether to keep a
  new pass. `skills/agentrefinery-design/`, `skills/agentrefinery-build/`,
  `skills/agentrefinery-build-validation/`, and `templates/*.md` (the ones
  built under the combined design, described above) were **moved** to
  AgentRails, not copied — this repo's `skills/` and `templates/` are now
  empty again.

**The proposed mechanism for AgentRefinery**, from the user directly:
mirror AgentRails' 3-command naming pattern with AgentRefinery's own
commands. `agentrefinery-build`'s generated skill copies
`output-process-name/` (produced by AgentRails' `process-name` skill) into
AgentRefinery's own `refinery-process-name/` directory after each run —
this keeps AgentRails' output directory clean for its next destructive
restart. A step referred to as **"Refinación"** then compares the fresh
`output-process-name/` pass against the existing `refinery-process-name/`
(the accumulated best-so-far) and, only if the fresh pass is genuinely
better, writes it into `refinery-process-name/`.

This was described in outline, not as a full command spec — see `PRD.md`
§7 for the concrete list of what's still unresolved (which command
implements "Refinación," what defines "better," comparison granularity,
what `agentrefinery-build-validation` validates, what `agentrefinery-
design` even does here, and who orchestrates running `process-name` a
2nd/3rd/Nth time).

---

## 2026-07-28 (continued) — 2-command pipeline settled; all §7 open items resolved

The prior entry above left the entire command spec open — no agreed
definition of "better," no agreed command implementing "Refinación," no
answer for `agentrefinery-design`. This entry resolves all of it, via a
direct design proposal from the user later the same day, confirmed with 6
clarifying answers. **`PRD.md` v0.3.0 is the authoritative record of this
resolution** — this entry summarizes what changed and why, per this file's
own "log, don't duplicate the spec" role.

**No `agentrefinery-design`.** Resolved as a deliberate, documented
exception rather than an open item: unlike building a Rail, there's no
upstream ambiguity to resolve before `agentrefinery-build` can run — the
objectives are already fixed (`Backbone.md`) and the first pass's actual
output already exists (`output-process-name/`, already validated).
AgentRefinery ships exactly 2 commands: `agentrefinery-build` and
`agentrefinery-build-validation` (kept fully hyphen-separated for naming
consistency with AgentRails, even without a `-design` command to
disambiguate it from — user's answer: "Mantener la consistencia").

**"What defines better" resolved by making it a pluggable input**: the
**Refinement Engine specification** concept. Rather than AgentRefinery
hardcoding one universal definition of "better" (which can't exist
generically), it's a standalone, user-replaceable document `agentrefinery-
build` reads as a third input (alongside `Backbone.md` and
`output-process-name/`). This repo bundles one reference implementation for
research-style processes — the user's original Spanish draft,
`ResearchRefinementEngine.md`, translated in full to English (per this
project's standing "English only, everywhere" rule) and relocated to
`engines/ResearchRefinementEngine.md`. The user was explicit that this is a
reference example only, not a universal default: users should supply their
own for other kinds of processes, and are explicitly encouraged to invest
real budget in a stronger, domain-tuned spec — e.g. commissioning a more
capable, more expensive LLM to draft one. Framing, verbatim from the user:
"Al final nuestro proceso de refinación, al igual que los rieles, solo son
los guardarrailes para el agente de IA" — AgentRefinery (like a Rail) is
the guardrail that lets whichever spec is supplied run consistently, not
the ceiling on refinement quality. See `PRD.md` §5.3.

**Which command implements "Refinación": neither build command does — the
generated `process-name-refine` skill does, at runtime, every time it's
invoked.** `agentrefinery-build` runs exactly once per Rail, to construct
`process-name-refine/SKILL.md` (confirmed directly by the user: "Es
correcto, solo se ejecuta una vez, para construir el comando:
/proccess-name-refine"). The actual pass-to-pass comparison logic — the
part that decides SEEDED / IMPROVED / NOT_IMPROVED / BLOCKED — lives
inside that generated skill's own runtime behavior, not inside
`agentrefinery-build` itself.

**What `agentrefinery-build-validation` validates**: traceability, mirroring
how `agentrails-build-validation` validates `Workflow.md` against
`Backbone.md` (user's answer: "Similar al agentrails-build-validation
verifica Validation.md") — here, checking `RefinementPlan.md`'s steps each
cite a real Backbone ID and a real Refinement Engine stage, and that every
Backbone ID is covered. Unlike `agentrails-build-validation`, it does not
produce a second runnable skill — it's a one-shot QA check, since the only
runtime artifact this project produces per Rail is `process-name-refine`
itself.

**Naming collision caught and fixed**: the user's original phrasing reused
`process-name-refine/` for both the generated skill package directory and
its own runtime output directory. Confirmed and renamed the output
directory to `output-process-name-refine/`, following AgentRails' own
`output-` prefix convention (user: "Si, tienes razon.").

**`output-process-name-refine/` does need its own tracking file** —
`RefinementTracking.md`, schema `ITERATION | AGENT | VERDICT | CONFIDENCE |
DETAILS | START | END`, permanent and never cleared (unlike AgentRails' own
tracking files, which get wiped by a Rail's own Phase 4 restart). See
`PRD.md` §9.1.

**Who orchestrates running `process-name` again**: the user, manually,
ideally varying the LLM each cycle. `/process-name-refine` can be re-run on
its own, but since its comparison isn't fully deterministic, the best
results come from repeating the full `/process-name` →
`/process-name-validation` → `/process-name-refine` cycle — this
recommendation must appear in the generated skill's own Readme/description
text, not just be documented here.

**Result**: `PRD.md` rewritten to v0.3.0 as the fully consolidated,
settled spec (§3 for the full naming/rationale history, §5 for the
Refinement Engine concept, §6–§9 for the 2-command pipeline and the
generated skill's runtime mechanics). `skills/agentrefinery-build/SKILL.md`
and `skills/agentrefinery-build-validation/SKILL.md` written from this
spec. `templates/` removed (empty, no longer referenced — AgentRefinery has
no document types of its own beyond what's described in §7's directory
layout).

## Open items for next session (current)

None outstanding from the design conversation itself. Remaining work is
implementation follow-through, not open design questions:

1. Exercise the pipeline end to end against a real, AgentRails-built Rail
   once one exists: `agentrefinery-build` → `agentrefinery-build-
   validation` → the generated `/process-name-refine`, both on an empty
   `output-process-name-refine/` (bootstrap path) and against an existing
   baseline (comparison path).
2. Consider whether additional bundled Refinement Engine specs (beyond the
   research-synthesis-tuned default) are worth adding for other common
   process shapes (e.g. code generation, document drafting) — not blocking,
   since users are already expected to supply their own for such cases.
