# Changelog

All notable changes to the **AgentRefinery project itself** are documented in
this file — its own tooling, documents, and conventions.

> This is not the same file type as a Rail's `output-process-name/Changelog.md`.
> That one is a per-Rail, append-only log of refinement passes (which
> agent/model ran a process again and what it improved) — see `PRD.md` §10.4.
> This one is the standard project changelog for AgentRefinery as software.

The format follows [Keep a Changelog](https://keepachangelog.com/), and this
project uses `MAJOR.MINOR.PATCH`-style versioning once a first release is
tagged. Nothing has been released yet — everything so far is under
`[Unreleased]`.

## [Unreleased]

### Added

- Core project concept: a **Rail** — a fixed-core / judgment-zone context
  bundle that lets a documented process stay consistent across runs while
  improving as a more capable LLM re-runs it. Project renamed from
  "AgentRails" to **AgentRefinery** to name the purpose (repeated
  refinement) rather than the mechanism.
- `README.md` — project pitch, the gap thesis, the 5 defining principles of
  a Rail, the pipeline diagram, a full user manual for the 3 commands
  (including an example invocation and the PRD-as-input pattern), the
  anatomy of a Rail, minimum requirements, and the inheritance/merging
  pattern.
- `PRD.md` — full, consolidated requirements specification: every concept,
  every document's exact structure, both progressive-execution state
  machines, the naming decision history and its rationale, and the
  project's documentation/language operating standards.
- `DESIGN-NOTES.md` — session-by-session working design log, kept as the
  historical record of how the design evolved (superseded as the
  day-to-day reference by `PRD.md`).
- `templates/` — the 5 base document templates a Rail's `context/` bundle
  is generated from: `Design.md`, `Backbone.md`, `Workflow.md`,
  `Validation.md`, `Readme.md`.
- `skills/agentrefinery-design/SKILL.md` — the pipeline's only
  LLM-reasoning step: turns a process description (a PRD works well as
  one) into a draft Rail context bundle.
- `skills/agentrefinery-build/SKILL.md` — deterministic: compiles a Rail's
  context into the runnable `process-name` skill, via the `skill-creator`
  hard prerequisite.
- `skills/agentrefinery-build-validation/SKILL.md` — deterministic:
  compiles a Rail's checklist into the runnable `process-name-validation`
  skill, via the same `skill-creator` hard prerequisite.
- `.gitignore`, `LICENSE` (MIT, Altepetl), `CONTRIBUTE.md`, `CLAUDE.md`.

### Changed

- Renamed the produced-artifact umbrella noun from unresolved
  ("the generated process") to **Rail**, then corrected its spelling from
  the Spanish "Riel" to the English "Rail" to match the project's
  English-only documentation rule.
- Renamed `agentrefinery-buildvalidation` to `agentrefinery-build-validation`
  for consistent hyphen-separation across every command name.
- Translated the remaining Spanish concept terms ("núcleo fijo", "zona de
  criterio") to English ("fixed core", "judgment zone") across every
  document, preserving existing bold emphasis.
