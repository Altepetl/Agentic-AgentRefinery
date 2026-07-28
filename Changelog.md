# Changelog

All notable changes to the **AgentRefinery project itself** are documented
in this file — its own tooling, documents, and conventions.

> This is not the same file type as a Rail's runtime output. A Rail's own
> per-run tracking lives in AgentRails now, not here.

The format follows [Keep a Changelog](https://keepachangelog.com/), and
this project uses `MAJOR.MINOR.PATCH`-style versioning once a first
release is tagged. Nothing has been released yet — everything so far is
under `[Unreleased]`.

## [Unreleased]

### Added

- **Settled the full 2-command pipeline (2026-07-28), resolving every open
  item left by the initial split-off PRD.** AgentRefinery ships exactly 2
  commands — `agentrefinery-build` and `agentrefinery-build-validation` —
  with no `agentrefinery-design` (deliberate, documented exception: unlike
  building a Rail, there's no upstream ambiguity to resolve, since
  `Backbone.md` is already fixed and the first pass's real output already
  exists). Together they build and validate a third, generated command,
  `process-name-refine` — the actual runtime artifact this project
  produces per Rail.
- Introduced the **Refinement Engine specification** concept: a
  standalone, pluggable, user-replaceable document defining how to compare
  a fresh pass against a best-so-far result and decide "better" — this
  resolves what had been the single biggest open question. Bundled one
  reference implementation, `engines/ResearchRefinementEngine.md`, tuned
  for research-style processes and translated in full from the user's
  original Spanish draft.
- Wrote `skills/agentrefinery-build/SKILL.md` and
  `skills/agentrefinery-build-validation/SKILL.md` from the settled spec.
- Renamed the generated skill's own runtime output directory from
  `refinery-process-name/` to `output-process-name-refine/`, fixing a
  naming collision with the generated skill package directory
  (`process-name-refine/`) and following AgentRails' own `output-` prefix
  convention.

### Changed

- **Split the original combined project into two sibling repos
  (2026-07-28).** AgentRefinery no longer produces Rails — that mechanism
  (fixed core/judgment zone, the 3-command Rail-building pipeline, the 5-
  document context bundle, `ProcessTracking.md`/`ValidationTracking.md`,
  and a Rail's own destructive re-run behavior) moved entirely to the new,
  separate **AgentRails** repo. `skills/agentrefinery-design/`,
  `skills/agentrefinery-build/`, `skills/agentrefinery-build-validation/`,
  and `templates/*.md` were moved there (renamed to `agentrails-*`), not
  copied — this repo's `skills/` and `templates/` were emptied at that
  point, then repopulated later the same day (see "Added," above) once the
  2-command spec was settled.
- Rewrote `PRD.md` (now v0.3.0), `README.md`, `CLAUDE.md`, `CONTRIBUTE.md`,
  `package.json`, and `bin/cli.js` to reflect AgentRefinery's settled
  scope: comparing and improving a Rail's output across N repeated runs,
  consuming Rails built by the sibling AgentRails project as input.
- Removed the now-unused `templates/` directory — AgentRefinery has no
  document types of its own beyond `RefinementPlan.md` and
  `refinement-engine.md`, both described directly in `PRD.md` §7.
- Appended new dated entries to `DESIGN-NOTES.md` documenting first the
  split and then the later same-day resolution of every open item, without
  rewriting the file's prior history (the original combined-project design
  and the first, fully-open split-off attempt both remain in that file as
  historical record).

### Added (from the original combined project, prior to the split)

- Core project concept: a **Rail** — since moved to AgentRails, see that
  repo's own Changelog for its post-split history.
- `PRD.md`, `README.md`, `DESIGN-NOTES.md`, `templates/`,
  `skills/agentrefinery-{design,build,build-validation}/SKILL.md`,
  `.gitignore`, `LICENSE` (MIT, Altepetl), `CONTRIBUTE.md`, `CLAUDE.md` —
  all authored under the original combined design; the Rail-producing ones
  among these have since moved to AgentRails.

### Changed (from the original combined project, prior to the split)

- Renamed the produced-artifact umbrella noun from unresolved to **Rail**,
  then corrected its spelling from the Spanish "Riel" to the English
  "Rail."
- Renamed `agentrefinery-buildvalidation` to `agentrefinery-build-validation`.
- Translated remaining Spanish concept terms ("núcleo fijo", "zona de
  criterio") to English ("fixed core", "judgment zone").
