# CLAUDE.md

Guidance for Claude Code (or any agent) working in this repository.

## What this repo is

**AgentRefinery does not produce Rails.** That's the sibling
**AgentRails** repo's job (`/home/rommelmg/Projects/Altepetl/Code/AgentRails`
if working locally). AgentRefinery consumes an already-built Rail as
input and owns a narrower, separate concern: comparing and improving a
Rail's output across N repeated runs, potentially by increasingly capable
LLMs, without ever modifying the Rail's own definition or how AgentRails
runs it.

This project is currently **mid-redesign**, following a 2026-07-28 split
of what used to be one combined project. Large parts of `PRD.md` are
explicit **open items** (§7), not settled behavior — resist the urge to
fill those gaps with plausible-sounding invented detail. Either resolve
with the user, or leave the gap marked open, exactly as documented.

**Before making any non-trivial change, read `PRD.md` in full.** It is
the canonical spec, and it is unusually explicit about what's settled vs.
still open.

## Directory map

```
AgentRefinery/
├── README.md          — pitch, current scope, pointer to AgentRails
├── PRD.md              — full spec: settled mechanism (§5) + open items (§7)
├── DESIGN-NOTES.md     — session-by-session working design log, incl. the
│                          now-superseded original combined-project design
├── Changelog.md        — this project's own release history
├── CONTRIBUTE.md        — contribution conventions
├── LICENSE              — MIT, Altepetl
├── skills/              — currently EMPTY (agentrefinery-* not built yet)
└── templates/           — currently EMPTY (no AgentRefinery-specific
                            document types designed yet)
```

## Terminology quick reference

(Full glossary: `PRD.md` §10.)

- **Rail** — owned and defined by AgentRails, not this repo.
- **`process-name`** — a Rail's durable identifier, defined by AgentRails;
  AgentRefinery reuses it, never invents its own.
- **`output-process-name/`** — a Rail's runtime output, owned by
  AgentRails; wiped on every destructive Phase-4 restart. AgentRefinery
  reads it, never writes to it.
- **`refinery-process-name/`** — AgentRefinery's own directory,
  accumulating the best-so-far result across N passes of a Rail.
- **Refinación / Refinement** — the not-yet-fully-specified step that
  compares a fresh pass against the accumulated best-so-far and updates it
  only if the fresh pass is genuinely better. See `PRD.md` §5, §7.

## What's settled vs. open

**Settled** (`PRD.md` §5): AgentRefinery mirrors AgentRails' 3-command
naming pattern (`agentrefinery-design`, `agentrefinery-build`,
`agentrefinery-build-validation`), kept entirely separate from AgentRails'
own commands. `agentrefinery-build`'s generated skill copies
`output-process-name/` into `refinery-process-name/` after each Rail run.
A "Refinación" step compares the fresh pass against the existing
`refinery-process-name/` and updates it only if genuinely better.

**Open** (`PRD.md` §7) — do not assume answers to these:
1. Which command (or concept) actually implements "Refinación."
2. What concretely defines "better" — the single biggest open question.
3. Whether comparison happens on the whole output or per-deliverable.
4. What `agentrefinery-build-validation` validates.
5. What `agentrefinery-design` does at all.
6. Whether `refinery-process-name/` needs its own tracking/log file.
7. Who orchestrates running `process-name` a 2nd/3rd/Nth time.

## Operating principles for changes in this repo

1. **English only, everywhere** — no exceptions.
2. **Command/file names are fully hyphen-separated** —
   `agentrefinery-build-validation`, never `agentrefinery-buildvalidation`.
3. **Assume nothing; document everything.** Especially: don't write a
   `skills/agentrefinery-*/SKILL.md` around an unresolved open item from
   `PRD.md` §7 — that just relocates the ambiguity instead of resolving it.
4. **Never modify AgentRails.** No changes to a Rail's `context/`, its
   `output-process-name/`, or either of its generated skills belong in
   this repo — if a change seems to require that, it belongs in the
   sibling AgentRails repo instead.
5. **Update `PRD.md` first** for any conceptual change — specifically §5
   (settled mechanism) and §7 (open items) — then propagate to
   `README.md` and `DESIGN-NOTES.md`.

Full contribution conventions: `CONTRIBUTE.md`.

## Testing

No automated test suite exists, and no `agentrefinery-*` skill has been
built yet — `PRD.md` §7's open items block writing a real command spec.
Once they're resolved and skills exist, the first real validation is
running a full refinement cycle (multiple passes of an AgentRails-built
Rail) and checking that `refinery-process-name/` actually ends up holding
the best result across passes. Build any test artifacts under `/sandbox/`
(gitignored) — never commit a Rail or a refinement run into this repo.
