---
name: agentrefinery-build-validation
description: Confirms that agentrefinery-build's output — process-name-refine/RefinementPlan.md and its generated SKILL.md — was built correctly against context/Backbone.md and the frozen process-name-refine/refinement-engine.md. A one-shot, deterministic QA check, not a skill generator — it does not produce a second runnable skill. Invoke this only after agentrefinery-build has produced process-name-refine/. Use whenever the user wants to check, review, or confirm that process-name-refine was built correctly, or asks to "validate the refine build".
---

# agentrefinery-build-validation

All judgment calls for this build were already made during
`agentrefinery-build` — every `RefinementPlan.md` step's mapping to a
Backbone objective and a Refinement Engine stage is fixed by that point,
not something you interpret or loosen here. Your job is a mechanical
check: confirm `RefinementPlan.md` traces correctly to `Backbone.md` and
`refinement-engine.md`, and that `process-name-refine/SKILL.md` actually
implements it faithfully. If you find a gap, stop and send it back to
`agentrefinery-build` to fix — do not patch the generated skill by hand.

**This command does not produce a second runnable skill.** Unlike
AgentRails' own `agentrails-build-validation` (which produces
`process-name-validation`), the only runtime artifact this whole project
produces, per Rail, is `process-name-refine/SKILL.md` — already produced
by `agentrefinery-build`. This command only checks it.

## Inputs

- `<process-name>/process-name-refine/RefinementPlan.md`
- `<process-name>/process-name-refine/refinement-engine.md` — the frozen
  copy `agentrefinery-build` wrote. Check against this, not the original
  external Refinement Engine spec file, so this validation stays
  reproducible even if the original path changes later.
- `<process-name>/context/Backbone.md`
- `<process-name>/process-name-refine/SKILL.md` (to confirm it faithfully
  implements `RefinementPlan.md`)

If any of these is missing, stop — `agentrefinery-build` hasn't finished
producing something checkable.

## Step 1 — Read the inputs

Read `RefinementPlan.md`, `refinement-engine.md`, `Backbone.md`, and
`process-name-refine/SKILL.md`.

## Step 2 — Check RefinementPlan.md traceability

- Every `RefinementPlan.md` step cites a real Backbone `O#`/`L#` and a real
  stage name from `refinement-engine.md` — nothing invented.
- Every Backbone `O#`/`L#` touched by the process is covered by at least
  one `RefinementPlan.md` step — nothing dropped.
- The hard-limit re-check step (confirming a candidate best-so-far can't
  violate any `L#`, even if otherwise favored by the Refinement Engine
  spec's ranking) is present.

## Step 3 — Check that SKILL.md implements RefinementPlan.md faithfully

- Same step sequence — no invented, skipped, or reordered steps.
- Bootstrap-vs-comparison branching is present (empty
  `output-process-name-refine/` → seed and stop; existing baseline → run
  the comparison).
- `RefinementTracking.md` generation is present, with the schema
  `ITERATION | AGENT | VERDICT | CONFIDENCE | DETAILS | START | END` and
  `VERDICT` restricted to `SEEDED` / `IMPROVED` / `NOT_IMPROVED` /
  `BLOCKED`.
- The escalation rule (stop and ask the user rather than guessing when no
  clear verdict is reached) is present.
- The generated skill's own Readme/description text states that
  `/process-name-refine` can be re-run repeatedly with a non-deterministic
  verdict, and recommends the full `/process-name` →
  `/process-name-validation` → `/process-name-refine` cycle, ideally
  varying the LLM, over re-running `/process-name-refine` alone.

## Step 4 — On any gap

Stop and report it precisely: which `RefinementPlan.md` step is
untraceable, which Backbone ID is uncovered, or which part of
`SKILL.md` doesn't match `RefinementPlan.md`. Send it back to
`agentrefinery-build` to regenerate — `RefinementPlan.md` and/or the
generated skill — rather than patching the generated skill by hand.

## Step 5 — On success

Confirm `/process-name-refine` is ready to run, and remind the user of the
full recommended cycle: running `/process-name` → `/process-name-validation`
→ `/process-name-refine` repeatedly, ideally varying the LLM each time, is
what actually produces improving results. `/process-name-refine` alone, run
repeatedly against the same two directories, can still shift its verdict a
little run to run (since the comparison isn't fully deterministic) but has
less new evidence to work with than a fresh Rail pass would provide.
