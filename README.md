# AgentRefinery

AgentRefinery does **not** produce Rails. That's the job of the sibling
**[AgentRails](../AgentRails)** project. AgentRefinery takes an
already-built, already-run, already-validated Rail (produced by
AgentRails) as a given input, and answers a different question: **across N
repeated runs of that same Rail, potentially by increasingly capable
models, is the result actually getting better — and if so, how do we keep
the best one?**

> For the full spec — every concept, document structure, and command
> behavior — see [`PRD.md`](./PRD.md). This README is the pitch and
> quick-reference summary; `PRD.md` is the one to trust for detail.

## Why this is a separate project from AgentRails

AgentRails and AgentRefinery used to be one combined project. Splitting
them (2026-07-28) resolved a real conflict: AgentRails needs a Rail's
re-run to be a simple, destructive restart — "run again? previous results
in `output-process-name/` will be deleted" — so that consistency stays
easy to verify regardless of which model is running it. But *comparing*
whether a new pass actually improved on the last one needs the opposite:
something that survives across those restarts and accumulates a
best-so-far result. Trying to do both inside one command made neither
concern clean. Now:

- **AgentRails** owns the mechanism: producing a Rail and running it
  consistently, once, down a fixed path — see its own README for the full
  pitch, the fixed-core/judgment-zone mechanism, and its 3 commands
  (`agentrails-design`, `agentrails-build`, `agentrails-build-validation`).
- **AgentRefinery** (this repo) owns comparing and improving a Rail's
  output across repeated runs, without ever touching the Rail's own
  definition or how AgentRails runs it.

## The 2-command pipeline

Unlike AgentRails, AgentRefinery ships only **2 commands**, not 3 — there's
no `agentrefinery-design`. Building a Rail requires resolving ambiguity in
a raw process description up front; refining one doesn't, because the
objectives are already fixed (`Backbone.md`, already reviewed and built by
AgentRails) and the first pass's actual output already exists
(`output-process-name/`, already validated). See `PRD.md` §3, item 4, for
the full rationale.

```
agentrefinery-build              → builds process-name-refine/SKILL.md
agentrefinery-build-validation   → one-shot QA check on that build
                                     (does not itself produce a runnable skill)
```

The single runtime artifact this project produces, per Rail, is
**`process-name-refine`** — the third command in the overall scheme,
alongside AgentRails' own `process-name` and `process-name-validation`.

## User manual

### 1. `agentrefinery-build`

Delivered as the Agent Skill at `skills/agentrefinery-build/SKILL.md`.

- **Invoke when**: a Rail's first pass has already run and been confirmed
  — `output-process-name/ProcessTracking.md` and `ValidationTracking.md`
  both show every row `✅`. Never invoke this on a Rail that hasn't
  finished and passed its first validated pass.
- **Required inputs**:
  - `<process-name>/context/Backbone.md` (and `Readme.md`, for its
    escalation rule)
  - `<process-name>/output-process-name/` — the actual deliverables from
    the Rail's first validated pass
- **Optional input**:
  - A path to a **Refinement Engine specification** — a document defining
    *how* to compare a fresh pass against a best-so-far result (see "How
    refinement actually works," below). **If omitted, defaults to the
    bundled [`engines/ResearchRefinementEngine.md`](./engines/ResearchRefinementEngine.md).**
- **Example invocation — using the bundled default** (no Refinement Engine
  spec given):
  ```
  /agentrefinery-build standard-builder/
  ```
  - `standard-builder/` — the Rail directory, containing `context/` and
    `output-process-name/`. Since no spec path follows it,
    `agentrefinery-build` resolves the Refinement Engine spec to the
    bundled `engines/ResearchRefinementEngine.md`.
- **Example invocation — with a user-supplied spec**:
  ```
  /agentrefinery-build standard-builder/ /home/Projects/Code/MyEngines/CodeReviewRefinementEngine.md
  ```
  - `standard-builder/` — same Rail directory as above.
  - `/home/Projects/Code/MyEngines/CodeReviewRefinementEngine.md` — a
    Refinement Engine spec tailored to this Rail's own process (e.g. one
    commissioned from a more capable LLM for a code-review process, where
    "better" means something different than it does for research
    synthesis). Overrides the bundled default for this build only.
- **Hard prerequisite**: the `skill-creator` skill must be installed and
  available — `process-name-refine/SKILL.md` is always scaffolded through
  it, never hand-rolled.
- **Produces**: `<process-name>/process-name-refine/SKILL.md`,
  `RefinementPlan.md`, and a frozen copy of whichever Refinement Engine
  spec was actually used, `refinement-engine.md`.

### 2. `agentrefinery-build-validation`

Delivered as the Agent Skill at
`skills/agentrefinery-build-validation/SKILL.md`.

- **Invoke when**: `agentrefinery-build` has already produced
  `process-name-refine/`.
- **Required inputs**:
  - `<process-name>/process-name-refine/RefinementPlan.md`
  - `<process-name>/process-name-refine/refinement-engine.md`
  - `<process-name>/context/Backbone.md`
- **No optional inputs** — this command takes no parameters beyond the
  Rail directory; it validates whatever `agentrefinery-build` already
  produced.
- **Example invocation**:
  ```
  /agentrefinery-build-validation standard-builder/
  ```
- **Produces**: no new skill — a one-shot QA report confirming (or
  rejecting) that `process-name-refine` was built correctly.

## How refinement actually works

`agentrefinery-build` reads a Rail's `context/Backbone.md`, the actual
deliverables in `output-process-name/`, and a **Refinement Engine
specification** — a standalone, pluggable document defining *how* to
compare a fresh pass against a best-so-far result and decide whether it's
an improvement. This is AgentRefinery's answer to the hardest question in
this space, **what makes one pass "better" than another**: rather than
hardcoding one universal definition (which doesn't generically exist), the
definition is an input parameter, the same way a process description is
AgentRails' input.

This repo ships one reference Refinement Engine spec,
[`engines/ResearchRefinementEngine.md`](./engines/ResearchRefinementEngine.md),
tuned for refining **research-style processes** (semantic matching,
clustering, evidence scoring, contradiction detection, ranking, LLM
synthesis, targeted follow-up research). It's a reference example, not the
only valid shape — a process whose deliverable isn't a set of research
findings needs a differently-shaped spec, and users are expected to supply
their own. Users who care about refinement quality for a specific,
valuable process are encouraged to invest real budget in a stronger,
domain-tuned spec — for example, commissioning a more capable, more
expensive LLM to draft one. **Like a Rail itself, AgentRefinery's job is
only to provide the guardrail that lets whichever engine spec is supplied
run consistently — it is not the ceiling on refinement quality.**

The generated `process-name-refine` skill:

- **Bootstraps** if `output-process-name-refine/` is empty: copies
  `output-process-name/` there, logs a `SEEDED` row, and tells the user to
  re-run the Rail (ideally with a different/more capable LLM) before
  refining again.
- **Compares** once a best-so-far baseline exists: applies
  `RefinementPlan.md`'s steps to decide whether the fresh pass is genuinely
  better, and if so replaces the accumulated best-so-far — logging
  `IMPROVED`, `NOT_IMPROVED`, or `BLOCKED` (escalated to the user) either
  way.

`/process-name-refine` can be re-run on its own, but since the comparison
isn't fully deterministic, the best results come from repeating the full
cycle — `/process-name` → `/process-name-validation` → `/process-name-refine`
— varying the LLM each time, not from re-running `/process-name-refine`
alone against the same two directories.

### Refinement process diagram

```
/process-name-refine invoked
        │
        ▼
output-process-name-refine/ empty or missing?
        │
        ├─ yes ──► copy output-process-name/ into output-process-name-refine/
        │          log RefinementTracking.md row: VERDICT = SEEDED
        │          report: "baseline established — re-run the Rail
        │                   (ideally with a different/more capable LLM),
        │                   then run /process-name-refine again"
        │          stop
        │
        └─ no ───► apply RefinementPlan.md to:
                     fresh pass  = output-process-name/       (current contents)
                     best-so-far = output-process-name-refine/ (current contents)
                        │
                        ▼
                   did the Refinement Engine spec's synthesis step
                   reach a clear verdict?
                        │
                        ├─ no ──► log VERDICT = BLOCKED
                        │         stop and ask the user rather than guess
                        │
                        └─ yes ─► is the fresh pass genuinely better?
                                  (includes a hard-limit re-check — a
                                   candidate can never violate a Backbone L#)
                                        │
                                        ├─ yes ─► write result into
                                        │          output-process-name-refine/
                                        │          log VERDICT = IMPROVED
                                        │
                                        └─ no ──► leave output-process-name-refine/
                                                   untouched
                                                   log VERDICT = NOT_IMPROVED
                                        │
                                        ▼
                             report verdict + confidence (if any) +
                             remaining uncertainties — tells the user
                             what the next Rail pass should target
```

## Diagram: full command execution flow

Every command below belongs to exactly one of the two sibling domains —
**AgentRails** (builds and runs the Rail) or **AgentRefinery** (this repo,
refines it across repeated passes). Neither domain's commands modify the
other's files; the only thing that crosses the boundary is
`output-process-name/`, read-only, once it's fully validated.

```
[AgentRails domain — sibling repo, builds & runs the Rail]

  agentrails-design
        │
        ▼
  agentrails-build             ──► process-name/SKILL.md
  agentrails-build-validation  ──► process-name-validation/SKILL.md
        │
        ▼
  /process-name                    (runs the Rail — produces a fresh pass
        │                           in output-process-name/)
        ▼
  /process-name-validation         (confirms the fresh pass is complete)
        │
        ▼
  ──────────────────────────────────────────────────────────────────
  output-process-name/ (now fully validated) crosses the boundary as
  AgentRefinery's read-only input
  ──────────────────────────────────────────────────────────────────
        │
        ▼

[AgentRefinery domain — this repo, refines across repeated passes]

  agentrefinery-build             ──► process-name-refine/SKILL.md,
  (reads Backbone.md +                 RefinementPlan.md,
   output-process-name/ +              refinement-engine.md
   a Refinement Engine spec)
  agentrefinery-build-validation  ──► QA report only — no new skill produced
        │
        ▼
  /process-name-refine              (compares fresh pass vs. best-so-far;
        │                            updates output-process-name-refine/
        │                            only if genuinely better)
        ▼
  ──────────────────────────────────────────────────────────────────
  recommended: repeat the whole cycle above, ideally with a different,
  more capable LLM each time — /process-name-refine alone, re-run
  repeatedly against the same two directories, has less new evidence
  to work with than a fresh Rail pass would provide
  ──────────────────────────────────────────────────────────────────
```

## Directory shape

```
<process-name>/                            ← the Rail, built and owned by AgentRails
├── context/                               ← untouched by AgentRefinery
├── output-process-name/                   ← owned by AgentRails; wiped on its own
│                                             destructive restart
├── process-name/SKILL.md                  ← untouched by AgentRefinery
├── process-name-validation/SKILL.md       ← untouched by AgentRefinery
├── process-name-refine/                   ← AgentRefinery's generated skill package
│   ├── SKILL.md
│   ├── RefinementPlan.md
│   └── refinement-engine.md
└── output-process-name-refine/            ← AgentRefinery's own runtime output;
    ├── RefinementTracking.md                 survives output-process-name/ being
    └── (accumulated best-so-far               wiped and re-run
        deliverables)
```

## Minimum requirements

- **[AgentRails](../AgentRails)** — a Rail must already exist, have run
  once via `process-name`, and been confirmed by `process-name-validation`
  before there's anything for AgentRefinery to refine.
- **[skill-creator](https://claude.com/plugins/skill-creator)** — hard
  prerequisite for both `agentrefinery-build` and
  `agentrefinery-build-validation`, consistent with how AgentRails
  scaffolds its own generated skills.
