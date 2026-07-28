# Contributing to AgentRefinery

AgentRefinery is the **builder repo for the refinement concern only** — it
does not produce Rails (that's the sibling **AgentRails** project) and it
is not one itself. Read `README.md` and `PRD.md` before changing anything;
`PRD.md` is the canonical spec, and it is unusually candid about what's
still an **open item** (§7) rather than settled design — most contribution
work right now should be resolving those open items, not building on top
of assumed answers to them.

## Before you start

1. Read `README.md` (the pitch + current-scope summary).
2. Read `PRD.md` in full, especially §7 (open items) and §3 (naming/split
   history) — this project is mid-redesign after the 2026-07-28 split from
   AgentRails, and most contribution mistakes right now would come from
   assuming an open item is already decided.
3. Do not duplicate AgentRails' own documentation here. If a change you're
   making describes the Rail mechanism itself (fixed core/judgment zone,
   the 5-document context bundle, `process-name`'s own state machine),
   that belongs in the sibling AgentRails repo, not here — see `PRD.md` §2.

## Standing rules

- **English only, everywhere.** No exceptions for etymology notes,
  parenthetical asides, or terms that started as shorthand in a design
  discussion.
- **Command and file names are fully hyphen-separated** — no concatenated
  compound words (`agentrefinery-build-validation`, never
  `agentrefinery-buildvalidation`).
- **Assume nothing; document everything.** Especially important here:
  resist filling `PRD.md` §7's open items with plausible-sounding invented
  behavior. Either resolve them with the user, or leave them marked open.
- **Preserve bold/italic emphasis through edits** — formatting is
  independent of wording.
- **Never modify AgentRails.** AgentRefinery reads a Rail's runtime output
  (`output-process-name/`) and writes to its own directory
  (`refinery-process-name/`); it never writes into a Rail's `context/`, a
  Rail's `output-process-name/`, or either of AgentRails' generated skills.

## Resolving `PRD.md` §7's open items

This is currently the highest-value contribution: each open item blocks a
real command spec for `agentrefinery-design` / `agentrefinery-build` /
`agentrefinery-build-validation`. When one gets resolved (ideally by
asking the user directly, since these are judgment calls about product
direction, not implementation details):

- Update `PRD.md` §5 (settled mechanism) and §7 (remove or narrow the
  resolved item).
- Append an entry to `DESIGN-NOTES.md` documenting the resolution and why.
- Only then start writing the corresponding `skills/agentrefinery-*/
  SKILL.md` — building a command around an unresolved open item just
  moves the ambiguity downstream instead of resolving it.

## Modifying `skills/*/SKILL.md` (once they exist)

- Keep the `skill-creator` hard prerequisite, consistent with how
  AgentRails scaffolds its own generated skills — don't hand-roll `SKILL.md`
  packaging.
- Keep the division of labor AgentRails established: only
  `agentrefinery-design` should require LLM reasoning; `agentrefinery-build`
  and `agentrefinery-build-validation` should be mechanical once design's
  ambiguity is resolved — assuming `agentrefinery-design` ends up owning
  that role at all (see `PRD.md` §7, open item 5, which is itself still
  open).

## Testing changes

No automated test suite exists. Once real `agentrefinery-*` skills exist,
the first real validation is running a full refinement cycle (multiple
passes of an AgentRails-built Rail's `process-name`, followed by
AgentRefinery's own commands) against a real Rail and checking whether
`refinery-process-name/` actually ends up holding the best result across
passes. Build any test artifacts under `/sandbox/` at the repo root
(gitignored) — never commit a Rail or a refinement run into this repo.

## Recording decisions

- **`PRD.md`** is the canonical spec — update §5 and §7 whenever an open
  item is resolved.
- **`DESIGN-NOTES.md`** is the working log — append new entries as you go,
  don't rewrite its history (including the now-superseded combined-project
  design at the top of the file).
- **`Changelog.md`** (this repo's own, not a Rail's) records what shipped,
  not why — keep entries short and factual.
