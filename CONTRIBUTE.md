# Contributing to AgentRefinery

AgentRefinery is the **builder repo for the refinement concern only** — it
does not produce Rails (that's the sibling **AgentRails** project) and it
is not one itself. Read `README.md` and `PRD.md` before changing anything;
`PRD.md` is the canonical spec and now reflects a settled 2-command
mechanism, following the 2026-07-28 design conversation that resolved the
open items left by the first split-off attempt.

## Before you start

1. Read `README.md` (the pitch + current-scope summary).
2. Read `PRD.md` in full, especially §3 (naming/design decisions and their
   rationale) and §5 (the Refinement Engine specification concept) — this
   project's core design question, "what makes one pass better than
   another," is answered by making that definition a pluggable input
   (§5.1), not something AgentRefinery hardcodes.
3. Do not duplicate AgentRails' own documentation here. If a change you're
   making describes the Rail mechanism itself (fixed core/judgment zone,
   the 5-document context bundle, `process-name`'s own state machine),
   that belongs in the sibling AgentRails repo, not here — see `PRD.md` §2.

## Standing rules

- **English only, everywhere.** No exceptions for etymology notes,
  parenthetical asides, or terms that started as shorthand in a design
  discussion — including `engines/ResearchRefinementEngine.md`, which was
  translated from an original Spanish draft for this exact reason.
- **Command and file names are fully hyphen-separated** — no concatenated
  compound words (`agentrefinery-build-validation`, never
  `agentrefinery-buildvalidation`).
- **Assume nothing; document everything.** Especially: don't hardcode a
  definition of "better" into `agentrefinery-build` or its generated
  skill — it must always be derived from whichever Refinement Engine spec
  is resolved (the bundled default or a user-supplied one), per `PRD.md`
  §5.
- **Preserve bold/italic emphasis through edits** — formatting is
  independent of wording.
- **Never modify AgentRails.** AgentRefinery reads a Rail's runtime output
  (`output-process-name/`) and writes only to directories it owns itself
  (`process-name-refine/` and `output-process-name-refine/`); it never
  writes into a Rail's `context/`, a Rail's `output-process-name/`, or
  either of AgentRails' generated skills.

## Modifying `skills/*/SKILL.md`

- Keep the `skill-creator` hard prerequisite for both
  `agentrefinery-build` and `agentrefinery-build-validation`, consistent
  with how AgentRails scaffolds its own generated skills — don't hand-roll
  `SKILL.md` packaging.
- `agentrefinery-build` is the one command in this pipeline allowed a small
  amount of judgment (mapping `Backbone.md` objectives to Refinement
  Engine stages) — see `PRD.md` §3, item 4, for why that's a deliberate,
  documented exception rather than an oversight. Don't extend that
  judgment scope elsewhere without updating `PRD.md` first.
- `agentrefinery-build-validation` stays purely mechanical — a one-shot QA
  check on `RefinementPlan.md`'s traceability. It does not produce a second
  runnable skill; don't add one.

## Adding or changing a Refinement Engine specification

- `engines/ResearchRefinementEngine.md` is a **reference example**, not the
  only valid shape a Refinement Engine spec can take — see `PRD.md` §5.2.
  If you're adding a new bundled spec for a different kind of process
  (e.g. code generation, document drafting), it needs its own definition
  of what a "claim," "contradiction," or "evidence" means for that output
  type; don't force it into the research-synthesis shape.
- Document, wherever a Refinement Engine spec is introduced or referenced,
  that AgentRefinery (like a Rail) is only a guardrail for consistently
  running whichever spec is supplied — not the ceiling on refinement
  quality. Users are expected to be able to invest real budget in a
  stronger, domain-tuned spec.

## Testing changes

No automated test suite exists. The first real validation of a change is
running a full refinement cycle against a real, AgentRails-built Rail:
`agentrefinery-build` → `agentrefinery-build-validation` → the generated
`/process-name-refine`, invoked both on an empty
`output-process-name-refine/` (bootstrap path) and against an existing
baseline (comparison path) — and checking that
`output-process-name-refine/` ends up holding the best result across
passes, with `RefinementTracking.md` reflecting an accurate history. Build
any test artifacts under `/sandbox/` at the repo root (gitignored) — never
commit a Rail or a refinement run into this repo.

## Recording decisions

- **`PRD.md`** is the canonical spec — update it first for any conceptual
  change.
- **`DESIGN-NOTES.md`** is the working log — append new entries as you go,
  don't rewrite its history (including the now-superseded combined-project
  design and the first, fully-open split-off attempt, both preserved at
  the top of the file).
- **`Changelog.md`** (this repo's own, not a Rail's) records what shipped,
  not why — keep entries short and factual.
