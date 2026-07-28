---
name: agentrefinery-build
description: Reads an already-built, already-run, already-validated Rail's context/Backbone.md, its output-process-name/ (the actual first validated pass), and a Refinement Engine specification (the bundled engines/ResearchRefinementEngine.md, or a user-supplied one), and builds the process-name-refine skill that will compare all subsequent passes against the best result seen so far. Invoke this only after a Rail's process-name and process-name-validation have both already run and every row in ProcessTracking.md and ValidationTracking.md is OK — never on a Rail that hasn't completed and passed its first validated pass. Use whenever the user wants to add cross-run refinement to a Rail, says the first validated pass is ready to refine, or asks to "build the refine skill" / "build process-name-refine".
---

# agentrefinery-build

Unlike AgentRails' own build commands, this one carries a small, deliberate
amount of judgment: mapping `Backbone.md`'s objectives/hard limits to the
Refinement Engine specification's stages. There is no `agentrefinery-design`
step to have resolved this ambiguity beforehand — there's no equivalent
ambiguity to resolve up front, because the objectives are already fixed
(`Backbone.md`, already reviewed and built by AgentRails) and the first
pass's actual output already exists (`output-process-name/`, already
validated). This command uses both directly, together with the resolved
Refinement Engine spec, to derive `RefinementPlan.md`. If an objective can't
be mapped to any stage in a checkable way, that's unresolved ambiguity —
stop and surface it to the user rather than guessing. This is the one place
in this pipeline allowed to ask, precisely because no earlier design step
exists to have caught it.

## Hard prerequisite: skill-creator

You must scaffold `process-name-refine/SKILL.md` **through the
`skill-creator` skill** (https://claude.com/plugins/skill-creator) — never
hand-roll the `SKILL.md` file or its packaging yourself. This keeps the
generated skill consistent with the same packaging conventions as every
other Agent Skill, including every Rail-produced one. If `skill-creator`
isn't available in this environment, stop and tell the user it's a hard
requirement rather than improvising a workaround.

## Inputs

1. `<process-name>/context/Backbone.md` — the objectives (`O#`) and hard
   limits (`L#`) every `RefinementPlan.md` step must trace back to.
   `<process-name>/context/Readme.md` is also read, for its escalation rule
   text (carried into the generated skill, same as AgentRails does).
2. `<process-name>/output-process-name/` — the actual deliverables from the
   Rail's first pass, plus `ProcessTracking.md` and `ValidationTracking.md`.
3. A Refinement Engine specification — **optional parameter**. If not
   given, defaults to the bundled `engines/ResearchRefinementEngine.md`. If
   given, must be a document following the same shape: a set of named
   stages, each with concrete instructions for the LLM executing that
   stage.

## Preconditions

Stop and report if any of these isn't met — never proceed on a partial or
unvalidated Rail:

- `context/Backbone.md` and `context/Readme.md` exist.
- `output-process-name/ProcessTracking.md` exists and every row is `✅`.
- `output-process-name/ValidationTracking.md` exists and every row is `✅`
  — i.e. `process-name-validation` has already confirmed this pass, not
  just that `process-name` finished running.

## Step 1 — Read the inputs

Read `Backbone.md`'s objectives/hard limits, `Readme.md`'s escalation rule,
`output-process-name/`'s actual deliverables and both tracking files, and
the resolved Refinement Engine spec's stage list (bundled default or
user-supplied — resolve which one applies before continuing).

## Step 2 — Draft RefinementPlan.md

For each stage in the Refinement Engine spec, write a concrete step that
cites the Backbone `O#`/`L#` ID(s) it serves, split into:

- **Fixed core** — the mechanical part: which files in
  `output-process-name/` vs. `output-process-name-refine/` get compared,
  what gets written where, and under what condition.
- **Judgment zone** — the part the spec's own "instructions for the LLM"
  text governs: semantic matching, evidence scoring, synthesis, and so on.

Ground every step in the real file layout of `output-process-name/` — this
is exactly the advantage a design-time ambiguity-resolution step would have
had, except here it comes from the real first pass already existing rather
than a hypothetical one.

Include an explicit **hard-limit re-check step**: whatever candidate ends
up chosen as the new best-so-far must not violate any `L#`, even if the
Refinement Engine spec's own ranking would otherwise favor it.

Every Backbone `O#`/`L#` touched by the process must be covered by at least
one `RefinementPlan.md` step; every step must cite a real Backbone ID and a
real Refinement Engine stage. If you find an objective that can't be mapped
to any stage in a checkable way, stop and report it to the user rather than
inventing a mapping.

## Step 3 — Freeze the Refinement Engine spec

Copy the resolved Refinement Engine spec verbatim into
`process-name-refine/refinement-engine.md`, so the generated package is
self-contained — `agentrefinery-build-validation` (and any future re-check)
checks against this frozen copy, not the original external file, so
validation stays reproducible even if the original path changes later.

## Step 4 — Compose the SKILL.md content to hand to skill-creator

The skill you're producing is named exactly `<process-name>-refine`. Its
purpose: compare the current contents of `output-process-name/` (the
latest pass) against `output-process-name-refine/` (the accumulated
best-so-far), and keep whichever is genuinely better, per
`RefinementPlan.md`.

Give `skill-creator` the following as the target skill's required behavior:

### a. RefinementTracking.md

Lives in `output-process-name-refine/`, one row per invocation of
`/process-name-refine`: `ITERATION | AGENT | VERDICT | CONFIDENCE | DETAILS
| START | END`.

- **ITERATION**: sequential counter, starting at 1 (the bootstrap pass).
- **AGENT**: which model/agent executed this invocation.
- **VERDICT**: `SEEDED` (bootstrap — nothing compared yet), `IMPROVED`
  (fresh pass replaced the prior best-so-far), `NOT_IMPROVED` (fresh pass
  compared but not written, prior best-so-far kept), or `BLOCKED`
  (escalated to the user, no verdict reached).
- **CONFIDENCE**: whatever confidence figure the resolved Refinement
  Engine spec's own synthesis step produces, if it defines one; empty if
  the spec doesn't.
- **DETAILS**: brief summary of what changed (if `IMPROVED`), why nothing
  changed (if `NOT_IMPROVED`), or what's blocking (if `BLOCKED`) —
  including any remaining uncertainties the spec's synthesis step flagged,
  so the user knows what a next pass should target.
- **START / END**: timestamps.

Never cleared or reset — this is the permanent, accumulating record of
every refinement pass across models and time for this Rail, since
`output-process-name-refine/` itself is never wiped (only
`output-process-name/`, by AgentRails' own Phase 4 restart, is).

### b. The state machine the generated skill must implement

```
if output-process-name-refine/ doesn't exist or is empty:
    copy all of output-process-name/ into output-process-name-refine/
    append RefinementTracking.md row: VERDICT = SEEDED
    report: "Baseline established. Nothing to compare yet — re-run the
             Rail's /process-name (ideally with a different/more capable
             LLM), then run /process-name-refine again to look for an
             improvement."
    stop

# a best-so-far baseline already exists -> attempt a comparison pass
apply RefinementPlan.md's steps to:
    fresh pass  = output-process-name/  (current contents)
    best-so-far = output-process-name-refine/ (current contents)

if the Refinement Engine spec's synthesis step reaches a clear verdict:
    if fresh pass is genuinely better:
        write the improved result into output-process-name-refine/
        append RefinementTracking.md row: VERDICT = IMPROVED
    else:
        leave output-process-name-refine/ untouched
        append RefinementTracking.md row: VERDICT = NOT_IMPROVED
    report the verdict, confidence (if any), and remaining uncertainties
else:
    append RefinementTracking.md row: VERDICT = BLOCKED
    stop and ask the user rather than guessing
```

### c. Escalation

Carry `Readme.md`'s escalation rule into the generated skill's own text: if
the comparison can't reach a clear verdict per the spec's own criteria,
stop and ask the user — never guess and pick a side. This is the same
fixed, non-negotiable escalation rule as every Rail's `Readme.md`.

### d. Re-runnability — put this in the generated skill's own Readme/description text

State explicitly that `/process-name-refine` can be invoked repeatedly,
including without a new Rail pass in between — since the comparison step
isn't fully deterministic, re-running it (especially with a different LLM)
can surface a slightly different verdict. But the best results come from
running the full cycle repeatedly, ideally varying the LLM each time, not
from re-running `/process-name-refine` alone against the same two
directories:

```
/process-name             (AgentRails — produces a fresh pass)
/process-name-validation  (AgentRails — confirms the fresh pass is complete)
/process-name-refine      (AgentRefinery — compares it against the best-so-far)
```

Also state, in this same generated text, that the Refinement Engine spec
frozen into `refinement-engine.md` is a guardrail that lets whichever spec
was supplied run consistently — it is not the ceiling on refinement
quality. A user who cares about refinement quality for a specific, valuable
process should consider investing in a stronger, domain-tuned Refinement
Engine spec (for example, commissioning a more capable, more expensive LLM
to draft one) rather than assuming the bundled default is optimal outside
its intended research-synthesis scope.

## Step 5 — Invoke skill-creator

Hand skill-creator the composed spec from Steps 2–4, with:
- **name**: `<process-name>-refine`
- **description**: what it refines plus a trigger phrase — e.g. "compares
  the latest `<process-name>` pass against the best result seen so far and
  keeps whichever is better; use when the user wants to
  refine/improve/compare passes of the `<process-name>` process."
- **output path**: `<process-name>/process-name-refine/SKILL.md`, alongside
  `RefinementPlan.md` and `refinement-engine.md` in the same directory.

Let skill-creator handle the actual `SKILL.md` scaffolding and packaging
conventions — your job was composing the correct, complete content, not
formatting it.

## Step 6 — Confirm and hand back

Once skill-creator has produced `process-name-refine/SKILL.md`, tell the
user it's ready, and that `agentrefinery-build-validation` should run next
to confirm `RefinementPlan.md`'s traceability against `Backbone.md` before
treating the build as complete.
