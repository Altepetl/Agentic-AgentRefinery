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

### Changed

- **Split the original combined project into two sibling repos
  (2026-07-28).** AgentRefinery no longer produces Rails — that mechanism
  (fixed core/judgment zone, the 3-command Rail-building pipeline, the 5-
  document context bundle, `ProcessTracking.md`/`ValidationTracking.md`,
  and a Rail's own destructive re-run behavior) moved entirely to the new,
  separate **AgentRails** repo. `skills/agentrefinery-design/`,
  `skills/agentrefinery-build/`, `skills/agentrefinery-build-validation/`,
  and `templates/*.md` were moved there (renamed to `agentrails-*`), not
  copied — this repo's `skills/` and `templates/` are empty again.
- Rewrote `PRD.md`, `README.md`, `CLAUDE.md`, `CONTRIBUTE.md`,
  `package.json`, and `bin/cli.js` to reflect AgentRefinery's narrowed
  scope: comparing and improving a Rail's output across N repeated runs,
  consuming Rails built by the sibling AgentRails project as input. Much
  of this narrowed scope is documented as an open item (`PRD.md` §7)
  rather than settled behavior — the exact command specs for
  `agentrefinery-design` / `agentrefinery-build` /
  `agentrefinery-build-validation` are not yet resolved.
- Appended a new dated entry to `DESIGN-NOTES.md` documenting the split
  and its rationale, without rewriting the file's prior history (the
  original combined-project design remains at the top of that file as a
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
