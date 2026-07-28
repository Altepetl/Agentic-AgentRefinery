# CLAUDE.md

Guidance for Claude Code (or any agent) working in this repository.

## What this repo is

**AgentRefinery does not produce Rails.** That's the sibling
**AgentRails** repo's job (`/home/rommelmg/Projects/Altepetl/Code/AgentRails`
if working locally). AgentRefinery consumes an already-built, already-run,
already-validated Rail as input and owns a narrower, separate concern:
comparing and improving a Rail's output across N repeated runs, potentially
by increasingly capable LLMs, without ever modifying the Rail's own
definition or how AgentRails runs it.

This project's command spec is now **settled**, following a 2026-07-28
design conversation that resolved the open items left by the first,
fully-open split-off attempt. **Before making any non-trivial change, read
`PRD.md` in full.** It is the canonical spec.

## Directory map

```
AgentRefinery/
├── README.md          — pitch, current scope, pointer to AgentRails
├── PRD.md              — full spec: concepts, 2-command pipeline, command specs
├── DESIGN-NOTES.md     — session-by-session working design log, incl. the
│                          now-superseded original combined-project design
│                          and the first, fully-open split-off attempt
├── Changelog.md        — this project's own release history
├── CONTRIBUTE.md        — contribution conventions
├── LICENSE              — MIT, Altepetl
├── engines/
│   └── ResearchRefinementEngine.md   — bundled default/reference
│                                        Refinement Engine spec
└── skills/
    ├── agentrefinery-build/SKILL.md
    └── agentrefinery-build-validation/SKILL.md
```

## Terminology quick reference

(Full glossary: `PRD.md` §12.)

- **Rail** — owned and defined by AgentRails, not this repo.
- **`process-name`** — a Rail's durable identifier, defined by AgentRails;
  AgentRefinery reuses it, never invents its own.
- **`output-process-name/`** — a Rail's runtime output, owned by
  AgentRails; wiped on every destructive Phase-4 restart. AgentRefinery
  reads it, never writes to it.
- **Refinement Engine specification** — a standalone, pluggable,
  user-replaceable document defining *how* to compare a fresh pass against
  a best-so-far result and decide "better." See `PRD.md` §5.
- **`process-name-refine/`** — AgentRefinery's generated skill package
  (`SKILL.md`, `RefinementPlan.md`, `refinement-engine.md`), built by
  `agentrefinery-build`.
- **`output-process-name-refine/`** — AgentRefinery's own runtime output;
  accumulates the best-so-far result across N passes; survives
  `output-process-name/` being wiped. See `PRD.md` §7, §9.

## The settled mechanism

AgentRefinery ships exactly **2 commands** — `agentrefinery-build` and
`agentrefinery-build-validation` — **not 3**. There is no
`agentrefinery-design`; see `PRD.md` §3, item 4, for why: unlike building a
Rail, there's no upstream ambiguity to resolve before these commands can
run, since `Backbone.md` is already fixed and the first pass's real output
already exists.

- `agentrefinery-build` reads `context/Backbone.md`,
  `output-process-name/` (the first validated pass), and a Refinement
  Engine spec (bundled default or user-supplied), and builds
  `process-name-refine/SKILL.md` plus its companion `RefinementPlan.md` and
  frozen `refinement-engine.md`.
- `agentrefinery-build-validation` is a one-shot QA check confirming
  `RefinementPlan.md`'s traceability against `Backbone.md` and that
  `SKILL.md` implements it faithfully — it does **not** produce a second
  runnable skill, unlike AgentRails' own `agentrails-build-validation`.
- The generated `process-name-refine` skill is the single runtime artifact
  this project produces per Rail. It bootstraps (seeds
  `output-process-name-refine/` if empty) or compares (decides
  `IMPROVED` / `NOT_IMPROVED` / `BLOCKED`) every time it's invoked.

Full detail: `PRD.md` §6–§9.

## Operating principles for changes in this repo

1. **English only, everywhere** — no exceptions.
2. **Command/file names are fully hyphen-separated** —
   `agentrefinery-build-validation`, never `agentrefinery-buildvalidation`.
3. **Assume nothing; document everything.** The Refinement Engine spec is
   pluggable by design — don't hardcode a definition of "better" into a
   build command; it must always be derived from whichever spec is
   resolved (bundled or user-supplied).
4. **Never modify AgentRails.** No changes to a Rail's `context/`, its
   `output-process-name/`, or either of its generated skills belong in
   this repo — if a change seems to require that, it belongs in the
   sibling AgentRails repo instead.
5. **Update `PRD.md` first** for any conceptual change, then propagate to
   `README.md` and `DESIGN-NOTES.md` (append a new dated entry, never
   rewrite prior history).

Full contribution conventions: `CONTRIBUTE.md`.

## Testing

No automated test suite exists. The first real validation of a change to
either `SKILL.md` is running a full refinement cycle against a real,
AgentRails-built Rail: `agentrefinery-build` →
`agentrefinery-build-validation` → repeated invocations of the generated
`/process-name-refine`, ideally interleaved with fresh `/process-name` →
`/process-name-validation` passes on varying LLMs — and checking that
`output-process-name-refine/` actually ends up holding the best result
across passes, with `RefinementTracking.md` reflecting an accurate history.
Build any test artifacts under `/sandbox/` (gitignored) — never commit a
Rail or a refinement run into this repo.
