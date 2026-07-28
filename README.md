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
