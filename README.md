# AgentRefinery

AgentRefinery does **not** produce Rails. That's the job of the sibling
**[AgentRails](../AgentRails)** project. AgentRefinery takes an
already-built Rail (a guide an LLM agent can run repeatedly, produced by
AgentRails) as a given input, and answers a different question: **across N
repeated runs of that same Rail, potentially by increasingly capable
models, is the result actually getting better — and if so, how do we keep
the best one?**

> For the fuller picture of what's settled and what's still an open
> question — including the exact mechanism proposed so far and the
> questions it doesn't yet answer — see [`PRD.md`](./PRD.md). This README
> is the pitch and current-scope summary; `PRD.md` is the one to trust for
> detail.

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

## Current status

This repo is **mid-redesign**. The mechanism below is what's been decided
so far; several real questions about it are still open (see `PRD.md` §7)
— most importantly, **there is no agreed definition yet of what makes one
pass's output "better" than another's**. No `skills/agentrefinery-*`
commands have been built yet; `skills/` and `templates/` in this repo are
currently empty placeholders.

## The mechanism, as far as it's settled

AgentRefinery mirrors AgentRails' 3-command naming pattern with its own
commands — `agentrefinery-design`, `agentrefinery-build`,
`agentrefinery-build-validation` — kept entirely separate so nothing about
AgentRails' own commands has to change.

What's settled about how they behave:

- After a Rail's `process-name` skill (from AgentRails) finishes a run,
  `agentrefinery-build`'s generated skill copies the entire contents of
  `output-process-name/` into AgentRefinery's own directory,
  `refinery-process-name/`. This keeps AgentRails' own output directory
  clean and ready for its next destructive restart, while
  `refinery-process-name/` is where AgentRefinery keeps its accumulated
  best-so-far result.
- A step the design conversation referred to as **"Refinación"**
  (Refinement) then compares the fresh `output-process-name/` pass against
  the existing `refinery-process-name/` (if one already exists). Only if
  the fresh pass is genuinely better does it get written into
  `refinery-process-name/`, replacing what was there. Otherwise
  `refinery-process-name/` is left alone.

That's what makes `refinery-process-name/` behave as a monotonically
improving result across N runs, even though each individual `process-name`
run (via AgentRails) has no memory of the prior pass.

**Not yet decided** (see `PRD.md` §7 for the full list): which of the 3
commands actually implements "Refinación"; what concretely defines
"better"; whether the comparison happens on the whole output or
per-deliverable; what `agentrefinery-build-validation` validates; what
`agentrefinery-design` does at all, given refinement isn't a document-
drafting problem the way building a Rail is; and who orchestrates running
`process-name` a 2nd, 3rd, ... Nth time.

## Directory shape implied so far

```
<process-name>/                       ← the Rail, built and owned by AgentRails
├── context/                          ← untouched by AgentRefinery
├── output-process-name/              ← owned by AgentRails; wiped on its own
│                                        destructive restart
├── process-name/SKILL.md             ← untouched by AgentRefinery
├── process-name-validation/SKILL.md  ← untouched by AgentRefinery
└── refinery-process-name/            ← owned by AgentRefinery; accumulates the
                                          best-so-far result across passes
```

## Minimum requirements

- **[AgentRails](../AgentRails)** — a Rail must already exist and be
  runnable before there's anything for AgentRefinery to refine.
- **[skill-creator](https://claude.com/plugins/skill-creator)** — expected
  to remain a hard prerequisite for AgentRefinery's own build commands too,
  once they're built, consistent with how AgentRails scaffolds its
  generated skills. Not yet exercised since no `agentrefinery-*` skill has
  been built yet.
