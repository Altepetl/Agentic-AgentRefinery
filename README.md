# AgentRefinery

AgentRefinery turns a documented process into a **Rail**: a guide an LLM agent can
run repeatedly, that gets *better* every time a more capable model runs it — without
anyone rewriting the process.

> For the full requirements spec — every concept, every document's exact
> structure, every state machine, and the rationale behind every naming
> decision — see [`PRD.md`](./PRD.md). This README is the pitch and the
> quick reference; `PRD.md` is the exhaustive one.

## The gap this project fills

Today there are two ways to get an AI agent to execute a process repeatedly:

- **Automate it fully** (script / workflow / RPA) — freezes quality at the day it
  was written. A more capable LLM underneath doesn't improve anything, because the
  logic no longer runs through the model.
- **Re-prompt it each time** — scales with model capability, but sacrifices
  consistency. Every run can drift, skip critical steps, or be interpreted
  differently.

AgentRefinery compiles a documented process into a guide that fixes the mandatory
path (what must always happen, and how it's verified) while leaving open the zone
where model judgment adds quality. The same guide, run by an older or newer LLM,
follows the exact same path — but the result improves with model capability, not
with rewriting code.

That guide is the Rail.

## Installing the AgentRefinery skills

AgentRefinery ships its 3 commands as installable Agent Skills
(`skills/agentrefinery-design`, `skills/agentrefinery-build`,
`skills/agentrefinery-build-validation`). Once this package is published,
installing all of them into a target tool is one command:

```bash
npx agent-refinery install
```

This prompts for which target(s) to install into and copies the 3 skills
into that tool's skills directory. To skip the prompt:

```bash
npx agent-refinery install --target claude,cursor
```

**Testing against a local clone** (before the package is published, e.g. to
try a change to a `SKILL.md`): run it from inside the cloned repo instead —
`npx .` runs the current directory as the package, `bin/cli.js`'s bin entry
is picked up the same way `agent-refinery` would be once published:

```bash
git clone <this-repo-url>
cd AgentRefinery
npx . install --target claude
# or, equivalently:
node bin/cli.js install --target claude
```

Supported targets (`npx agent-refinery list` prints this same table with
each installed skill):

| Target key    | Platform          | Install path                                                        |
|---------------|--------------------|----------------------------------------------------------------------|
| `claude`      | Claude Code        | `.claude/skills/` (project) or `~/.claude/skills/` (`--global`)     |
| `antigravity` | Google Antigravity | `.agent/skills/` (project) or `~/.gemini/antigravity/skills/` (`--global`) |
| `cursor`      | Cursor             | `.cursor/skills/` (project-scoped only, no personal directory)       |
| `zcode`       | ZCode              | `.zcode/skills/` (project) or `~/.config/zcode/skills/` (`--global`) |
| `kimi`        | Kimi Code CLI      | `.kimi-code/skills/` (project) or `~/.kimi-code/skills/` (`--global`) |
| `codex`       | OpenAI Codex CLI   | `.codex/skills/` (project) or `~/.codex/skills/` (`--global`)        |
| `agents`      | Generic            | `.agents/skills/` — the shared convention also read by Gemini CLI, VS Code Copilot, and other Agent-Skills-compatible tools |

Pass `--dir <path>` to install into a project other than the current
directory, and `--global` to install to a platform's user-level directory
instead of the project-level one (ignored for platforms, like Cursor, that
only support project scope). Run `npx agent-refinery help` for the full
option list.

`agentrefinery-build` and `agentrefinery-build-validation` both have a hard
prerequisite on the separate [skill-creator](https://claude.com/plugins/skill-creator)
skill — install it the same way into whichever tool you're using before
running either command; see "Minimum requirements" below.

## Why "Rail" — and why it's the actual product

A rail fixes the path a train follows; it doesn't drive the train. The
engine supplies the power, the speed, the judgment calls within the boundaries the
rail sets. Put a more capable engine on the same rail and it runs faster and
smoother — no new track required.

That's exactly the mechanism here. A Rail is a bundle of context documents that
fixes the mandatory path of a process (the **fixed core**) while leaving a
**judgment zone** — a zone where the LLM's own judgment decides *how* to execute
each step. A weaker model still completes every mandatory step correctly, because
the fixed core anchors it. A stronger model produces a better result on the exact
same path, because the judgment zone is where its extra capability shows up.

The Rail is not a byproduct or an intermediate file — **it is the product.** What
AgentRefinery ships, for any given process, is a Rail: a reusable, inheritable,
version-tracked artifact that outlives any single run. The deliverables produced by
*running* the Rail matter, but the Rail itself — the guide that can be run again
next month by a better model and get better results on the same path — is the
thing AgentRefinery exists to generate.

## Refinement: what happens when you run a Rail

Running a Rail is not a one-shot pass. Its execution is incremental and resumable
by design — to survive interruptions, and, more importantly, so the same Rail can
be handed to a different or more capable LLM later to *complement or improve* a
prior result, rather than starting over.

Each pass over a Rail is a refining pass, the way raw material is refined in
stages rather than converted in one step:

- Every step's outcome is tracked, including **which model/agent executed it** —
  this is what makes "increasingly capable LLMs improve the same result" an
  auditable claim, not a slogan.
- A finished pass can always be re-opened: "run the process again to
  complement/improve the result?" Nothing is overwritten; prior deliverables are
  extended.
- Every improvement pass is logged permanently (which agent, what changed), so a
  Rail accumulates a visible history of how each generation of model improved on
  the last one, on the exact same path.

This is the "Refinery" in AgentRefinery: not the one-time act of building a Rail,
but the repeated act of refining its output by running it again, pass after pass,
as models improve.

## The 5 defining principles of a Rail

1. **Guides, doesn't dictate the exact "how"** — mandatory steps + hard limits, but
   the agent decides execution details within each step.
2. **Verifiable, not just descriptive** — each step has a way to confirm it was
   satisfied before moving on.
3. **Resolves ambiguity explicitly** — when instructions conflict, the guide says
   what wins.
4. **Has a declared escape point** — when the agent can't comply, it stops and
   asks the user. It does not guess and move on.
5. **Explicit scope** — the guide states when it applies and when it doesn't.

## Pipeline

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
 workflow — everything  as specific input, not a generic validation)
 already disambiguated)
   │                     │
   ▼                     ▼
process-name        process-name-validation
(runs the Rail)      (validates a completed run against the Rail)
```

Only `agentrefinery-design` requires model judgment. `agentrefinery-build` and
`agentrefinery-build-validation` are mechanical transformations — by the time they
run, all ambiguity has already been resolved.

## User manual

This section documents each of the 3 commands precisely enough that an agent
with no other context — no memory of this conversation, no assumptions about
what "should" happen — can invoke them correctly. If any input listed as
required is missing or unclear, the command stops and asks rather than
guessing; none of the three ever infers a `process-name` or a file location
on its own.

### 1. `agentrefinery-design`

The pipeline's entry point. Delivered as the Agent Skill at
`skills/agentrefinery-design/SKILL.md`.

- **Invoke when**: there is a process description (a prompt plus whatever
  supporting material describes how the process should work) and no Rail
  exists for it yet, or an existing Rail's `context/` bundle needs to be
  extended/merged with another one.
- **Required inputs**:
  - The process description itself (prompt + supporting data). A Product
    Requirements Document (PRD) works well as-is for this: a real PRD
    already states objectives, constraints, and scope — most of what
    `Backbone.md` needs — so it can be handed in directly rather than
    rewritten into a prompt first. The Rail that comes out of it, once
    built and run, is what actually constructs whatever the PRD describes.
  - `process-name` — a stable identifier for this Rail. If not given, ask
    the user for it; never invent one, since other Rails may later inherit
    from it by name.
- **Optional inputs**:
  - A list of local paths and/or Git repository URLs, each pointing to an
    already-built Rail's `context/` bundle, to merge into this one via
    inheritance.
  - The output location for `<process-name>/`, if not obvious from context.
- **Example invocation**:
  ```
  /agentrefinery-design PRD.md standard-builder /home/Projects/Code/ECC/ /home/Projects/Code/ECCStandards/
  ```
  - `PRD.md` — the process description.
  - `standard-builder` — the `process-name`.
  - `/home/Projects/Code/ECC/` — no `context/` bundle inside it yet, so
    it's the output location: `standard-builder/` gets created here.
  - `/home/Projects/Code/ECCStandards/` — already has a `context/` bundle
    (an existing Rail encoding this org's coding standards), so it's
    treated as a bundle to merge: `standard-builder` inherits from it.
- **Preconditions**: `templates/Design.md`, `templates/Backbone.md`,
  `templates/Workflow.md`, `templates/Validation.md`, and
  `templates/Readme.md` must exist at the AgentRefinery repo root — this
  command reads them as the base pattern for every document it produces.
- **Produces**: `<process-name>/context/Design.md`,
  `<process-name>/context/Backbone.md`, `<process-name>/context/Workflow.md`,
  `<process-name>/context/Validation.md`,
  `<process-name>/context/Readme.md` — a **draft** Rail context bundle. It
  does not scaffold `SKILL.md` packages, and it does not auto-chain into
  `agentrefinery-build`.
- **On ambiguity**: resolves it before moving to the next document — either
  states a flagged assumption inline (visible to a reviewer, not buried) or
  asks the user directly. Never silently picks an interpretation.
- **On a bundle merge conflict**: stops and reports the full list of
  contradicting objectives/hard limits across the input bundles. No
  automatic resolution, no partial merge — resolving conflicting input
  contexts is the user's responsibility.

### 2. `agentrefinery-build`

Deterministic. Delivered as the Agent Skill at
`skills/agentrefinery-build/SKILL.md`.

- **Invoke when**: a Rail's `context/` bundle exists and has been reviewed
  (`Backbone.md` especially, since it's the source of truth). Never invoke
  this directly on a raw process description — that's what
  `agentrefinery-design` is for.
- **Required inputs**:
  - `<process-name>/context/Backbone.md`
  - `<process-name>/context/Workflow.md`
  - `<process-name>/context/Readme.md`
- **Hard prerequisite**: the `skill-creator` skill must be installed and
  available. This command refuses to hand-roll `SKILL.md` packaging itself
  — if `skill-creator` is missing, it stops and says so instead of
  improvising a substitute.
- **Preconditions**: all 3 required files exist, and every step in
  `Workflow.md` cites Backbone objective/hard-limit IDs that actually exist
  in `Backbone.md`.
- **Produces**: `<process-name>/process-name/SKILL.md` (the literal folder
  name matches `process-name`, so it can be installed as-is into a
  platform's skills directory) — a runnable skill implementing
  `Workflow.md`'s fixed step sequence, `ProcessTracking.md` generation, the
  4-phase progressive-execution state machine, and `Changelog.md` logging.
- **On an inconsistency** (e.g. a `Workflow.md` step with no traceable
  Backbone citation): stops and reports it rather than silently dropping or
  inventing a fix — that's a defect from the design phase to send back, not
  something to patch here.

### 3. `agentrefinery-build-validation`

Deterministic. Delivered as the Agent Skill at
`skills/agentrefinery-build-validation/SKILL.md`.

- **Invoke when**: a Rail's `context/Validation.md` and `context/Backbone.md`
  exist and have been reviewed. It does not depend on `agentrefinery-build`
  having already run — the two build commands can run in either order —
  though *using* the resulting `process-name-validation` skill obviously
  requires `process-name` to have produced output first.
- **Required inputs**:
  - `<process-name>/context/Validation.md`
  - `<process-name>/context/Backbone.md`
- **Hard prerequisite**: the `skill-creator` skill must be installed and
  available, same as `agentrefinery-build`.
- **Preconditions**: both required files exist, and every checklist item in
  `Validation.md` cites Backbone objective/hard-limit IDs that actually
  exist in `Backbone.md`.
- **Produces**: `<process-name>/process-name-validation/SKILL.md` — a
  runnable skill implementing `Validation.md`'s checklist as checks against
  `output-process-name/`, `ValidationTracking.md` generation (STEP column
  seeded from `ProcessTracking.md`), and a state machine mirroring
  `process-name`'s own resumability logic.
- **On a checklist item that isn't concretely checkable**: stops and reports
  it — sends the user back to `agentrefinery-design` to sharpen it, rather
  than softening the check to make it pass mechanically.

### After both build commands have run

`<process-name>/process-name/SKILL.md` and
`<process-name>/process-name-validation/SKILL.md` are themselves Agent
Skills — install them into whichever platform's skills directory applies
(see "Minimum requirements" below) and invoke them directly to run and
validate the process. Neither is part of the AgentRefinery pipeline itself;
they are what the pipeline produces.

## Anatomy of a Rail

```
<process-name>/
├── context/                        ← shared by BOTH commands
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

## Minimum requirements

- [skill-creator](https://claude.com/plugins/skill-creator) — `agentrefinery-build`
  always scaffolds `SKILL.md` packages through skill-creator rather than
  hand-rolling them. This is a hard prerequisite.
- A target platform that supports Agent Skills — see "Installing the
  AgentRefinery skills" above for the full list of supported targets and
  install paths (Claude Code, Google Antigravity, Cursor, ZCode, Kimi Code
  CLI, OpenAI Codex CLI, and any other tool reading the generic
  `.agents/skills/` convention).

## Merging pre-existing Rails

`agentrefinery-design` accepts a list of local paths and/or Git repository URLs,
each pointing to an already-built Rail's context bundle. If contradictions are
found across bundles (conflicting objectives, hard limits, etc.), the tool reports
them and stops — no automatic conflict resolution, no partial merge. Providing
coherent input contexts is the user's responsibility.

The recommended pattern is **inheritance**: build an agnostic/base Rail's context
first, then a child Rail whose context is a specific application of it. Merging a
base Rail's Backbone/Validation with a child's is the natural, low-contradiction
case.
