---
name: agentrefinery-build
description: Compiles a Rail's context/ bundle (Backbone.md, Workflow.md, Readme.md — already disambiguated by agentrefinery-design) into the runnable process-name skill that executes the process, progressively and resumably, and improves on repeat passes by more capable models. Deterministic — invoke this only after agentrefinery-design has produced a context/ bundle for the process-name in question, never on a raw process description. Use whenever the user wants to turn a finished Rail context bundle into an actual runnable skill, says the design/context docs are approved and ready to build, or asks to "build the Rail" / "package process-name".
---

# agentrefinery-build

By the time you run, all ambiguity in this Rail was already resolved by
`agentrefinery-design`. Your job is a mechanical transformation: turn
`context/Backbone.md` + `context/Workflow.md` + `context/Readme.md` into a
runnable `process-name/SKILL.md`. Do not re-interpret or second-guess the
context documents — if something in them looks ambiguous or wrong, stop and
tell the user to fix it via `agentrefinery-design` rather than resolving it
yourself here. Silently patching ambiguity at this stage would defeat the
reason design and build are separate steps.

## Hard prerequisite: skill-creator

You must scaffold `process-name/SKILL.md` **through the `skill-creator`
skill** — never hand-roll the `SKILL.md` file or its packaging yourself.
This keeps every Rail-produced skill consistent with the same packaging
conventions as every other Agent Skill, instead of drifting into a
bespoke format. If `skill-creator` isn't available in this environment,
stop and tell the user it's a hard requirement rather than improvising a
workaround.

## Inputs

- `<process-name>/context/Backbone.md`
- `<process-name>/context/Workflow.md`
- `<process-name>/context/Readme.md`
- (`Design.md` and `Validation.md` are not needed here — Design.md is for
  human orientation only, and Validation.md belongs to
  `agentrefinery-build-validation`.)

If any of the three required files is missing, stop — this Rail's context
bundle isn't complete enough to build from.

## Step 1 — Read the context

Read `Backbone.md`, `Workflow.md`, and `Readme.md`. Confirm every step in
`Workflow.md` cites Backbone O#/L# IDs that actually exist — if you find a
step that doesn't trace back to Backbone.md, that's a defect from the design
phase; stop and report it rather than silently dropping or fixing the step.

## Step 2 — Compose the SKILL.md content to hand to skill-creator

The skill you're producing is named exactly `<process-name>`. Its purpose:
execute the process defined by `Workflow.md`, progressively and resumably,
writing live progress to a tracking file, and offering repeat passes so a
later, more capable model can improve on a prior run without starting over.

Give `skill-creator` the following as the skill's required behavior — it
must end up in the generated `process-name/SKILL.md` body, adapted to
`skill-creator`'s own conventions but not stripped of substance:

### a. The fixed step sequence

Transcribe `Workflow.md`'s steps verbatim (in order, with their fixed core /
judgment zone split and Backbone refs). The executing agent
follows this sequence exactly — it does not invent, skip, or reorder steps,
per `Readme.md`'s precedence rules and escalation rule (which must also be
carried into the generated skill: stop and ask the user when a step's
fixed core can't be satisfied).

### b. ProcessTracking.md

Lives in `output-process-name/`, schema: `STATUS | AGENT | STEP | DETAILS |
START | END`.

- STATUS: `✅` done, `❌` error/blocked, empty = pending.
- AGENT: which model/agent executed that step — this is what makes
  "increasingly capable LLMs improve the same result" auditable.
- STEP: short step name, matching a Workflow.md step.
- DETAILS: problems found / notes, empty if none.
- START/END: timestamps.

Generate this file (one row per Workflow.md step, empty STATUS) before
execution starts, if it doesn't already exist. Write/flush per step, before
moving to the next one — never hold it open for the whole run, so progress
is visible live.

### c. The state machine the generated skill must implement

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
        # this skill never writes to ValidationTracking.md — read-only.
        # Only process-name-validation writes/clears its own file.

# Phase 4 — everything resolved on both sides -> offer another pass
if ProcessTracking.md fully OK and (ValidationTracking.md doesn't exist or fully OK):
    ask user: "run the process again to complement/improve the result?"
    if yes:
        clear STATUS/START/END/DETAILS in BOTH tracking files
        (do NOT delete output-process-name/ contents)
        re-run from Phase 1 — complement existing deliverables, never overwrite
        append one entry to Changelog.md: which LLM/agent ran this pass +
        a brief description of what was complemented/improved
        on completion, show:
          "Complement and correction pass finished — to properly close this
           cycle, run the process validation again."
    if no:
        stop
```

### d. Changelog.md

Lives in `output-process-name/`, alongside the tracking files. Append-only,
no frontmatter, columns `DATE | AGENT | SUMMARY`. Only written to on a
Phase 4 "yes" branch — never cleared on a tracking-file reset, since it's
the permanent record of every improvement pass across models and time.

### e. Escalation

Carry `Readme.md`'s escalation rule into the generated skill's own text,
explicitly: if a step's fixed core cannot be satisfied as written, the
agent stops and asks the user. It does not guess, skip, or substitute its
own judgment for what the fixed core requires.

## Step 3 — Invoke skill-creator

Hand skill-creator the composed spec from Step 2 as the target skill's
required behavior, with:
- **name**: `<process-name>`
- **description**: what the process does (drawn from `Backbone.md`'s
  objectives) plus when to trigger — e.g. "runs the <process-name>
  process end to end, progressively and resumably; use when the user wants
  to execute/continue/improve the <process-name> process."
- **output path**: `<process-name>/SKILL.md`, alongside the `context/` and
  `output-process-name/` directories for this Rail.

Let skill-creator handle the actual `SKILL.md` scaffolding and packaging
conventions — your job was composing the correct, complete content, not
formatting it.

## Step 4 — Confirm and hand back

Once skill-creator has produced `process-name/SKILL.md`, tell the user it's
ready to run, and remind them that `agentrefinery-build-validation` still
needs to run against `context/Validation.md` + `context/Backbone.md` to
produce `process-name-validation` before this Rail's execution/validation
cycle is complete.
