---
title: AgentRefinery — Product Requirements Document
status: active
version: 0.2.0
created: 2026-07-27
updated: 2026-07-28
role: Single, self-contained reference for the entire AgentRefinery project —
  problem, concept, relationship to the sibling AgentRails project, the
  refinement mechanism as far as it's currently settled, and every open
  question still blocking a full command spec. Written so a reader (human
  or AI agent) with no memory of how this project came to be can pick it up
  and continue work correctly, without re-deriving anything from a prior
  conversation.
---

# AgentRefinery — Product Requirements Document

## 0. How to use this document

This is the authoritative specification of AgentRefinery **as currently
scoped**, after the 2026-07-28 split described in §3. Unlike a typical PRD
in this family of documents, large parts of this one are deliberately
**open items** (§7), not settled design — the mechanism was proposed by the
user in outline form but has not yet been fully specified command-by-command.
Do not fill those gaps with invented detail; either ask the user, or leave
the gap marked as open, exactly as it's written here.

- **`README.md`** is the pitch + quick reference for what AgentRefinery is
  and how it relates to AgentRails.
- **This document (`PRD.md`)** is the fuller spec: what's settled, what
  isn't, and why.
- **`DESIGN-NOTES.md`** is the working scratchpad, kept as a session-by-
  session log of how the design evolved — including the original,
  now-superseded combined design.
- **`skills/`** and **`templates/`** are currently **empty** in this repo
  (see §8) — the previous Rail-production content that lived here has
  moved to the sibling AgentRails repo (§1, §3). AgentRefinery's own
  `agentrefinery-design` / `agentrefinery-build` / `agentrefinery-build-
  validation` skills have not been built yet, because their exact behavior
  is one of the open items in §7.

If you are an AI agent picking up this project cold: read this document in
full, then read the sibling AgentRails repo's `PRD.md` (or its README, if
the full repo isn't available) before assuming anything about what a
"Rail" is — AgentRefinery no longer defines that concept itself.

---

## 1. What AgentRefinery is, and what it isn't

**AgentRefinery does not produce Rails.** That job belongs entirely to the
sibling **AgentRails** project (a separate repo/product). A Rail — a
`context/` bundle plus a matched pair of Agent Skills for a given
`process-name` — is built exclusively via AgentRails'
`agentrails-design` → `agentrails-build` / `agentrails-build-validation`
pipeline. AgentRefinery takes an already-built Rail as a **given input**
and never redefines, edits, or rebuilds it.

**AgentRefinery's own job**: given a Rail that already runs consistently
(that's what AgentRails guarantees), compare and improve the *quality* of
its output across N repeated runs, as increasingly capable LLMs execute
the same Rail. AgentRails intentionally does not care about this — a bare
Rail's re-run is a clean, destructive restart (see AgentRails' `PRD.md`
§10.2, Phase 4) that throws away the previous `output-process-name/`
entirely. AgentRefinery exists specifically to keep what's worth keeping
across those destructive restarts, and to decide, run over run, whether a
new pass is actually an improvement before treating it as the new best
result.

This is a narrower, harder problem than producing the Rail itself:
"consistent execution" is externally verifiable (a checklist either
passes or it doesn't — see AgentRails' `Validation.md`), but "this pass is
*better than* the previous best pass" is a judgment call with no
established, generic definition yet. See §7 for the open questions this
raises.

---

## 2. Relationship to AgentRails

- AgentRails and AgentRefinery are **separate repositories, separate
  products**, deliberately split on 2026-07-28 (§3) because they answer
  different questions: AgentRails asks "does this process run the same
  way every time, on any model?"; AgentRefinery asks "did running it again,
  with a better model, actually make the result better?"
- AgentRefinery **depends on** AgentRails: it has nothing to refine without
  a Rail already built and already run at least once by AgentRails'
  `process-name` skill.
- AgentRefinery **never modifies** AgentRails' own commands, generated
  skills, or a Rail's `context/` bundle. It reads a Rail's runtime output
  (`output-process-name/`) as input and writes to its own, separate
  directory (`refinery-process-name/`, see §5) — it does not write into
  `output-process-name/` itself, and it does not change how `process-name`
  or `process-name-validation` behave.
- Nothing about the fixed core / judgment zone mechanism, the 5-document
  context bundle, the 3-command Rail-building pipeline, the
  `ProcessTracking.md` / `ValidationTracking.md` state machines, or the
  destructive Phase-4 restart is owned here anymore. For all of that, see
  the sibling AgentRails repo's `PRD.md` — repeating it here would create
  exactly the kind of duplicate-source-of-truth problem this whole project
  is designed to avoid.

---

## 3. Naming decisions — history and rationale

This section exists specifically to prevent re-litigating settled
decisions, and to preserve *why* they were made, not just what was
decided.

1. **Project started as "AgentRails," was renamed to "AgentRefinery,"
   then split back into two sibling projects, one under each name
   (2026-07-28).** Originally, a single combined project was renamed from
   "AgentRails" (naming the mechanism) to "AgentRefinery" (naming the
   purpose: processes that get better every time a more capable LLM
   re-runs them). In practice this conflated two genuinely different
   concerns inside one Phase 4 state-machine step: (a) keeping a process
   execution consistent and verifiable, and (b) deciding whether a repeat
   pass is actually an improvement worth keeping. Trying to serve both
   inside one command (`process-name`'s own Phase 4, which used to
   "complement, never overwrite" and log to a per-Rail `Changelog.md`)
   made the destructive-vs-accumulating re-run behavior ambiguous and
   made a bare Rail's execution harder to verify in isolation. The fix:
   split back into two sibling repos. **AgentRails** keeps the original
   name and owns the mechanism (fixed core / judgment zone, the 3-command
   Rail-building pipeline, and a Rail's own re-run behavior — now a plain,
   destructive restart, no complement-not-overwrite logic, no per-Rail
   `Changelog.md`). **AgentRefinery** keeps this repo and narrows to just
   the cross-run refinement concern described in §1, consuming Rails
   produced by AgentRails as its input.
2. **The old per-Rail `output-process-name/Changelog.md` concept was
   removed, not migrated.** In the combined design, every "improvement"
   pass appended an entry to a Changelog.md living inside a Rail's own
   output directory. Since a Rail's re-run is now destructive (owned by
   AgentRails), that file no longer has anywhere consistent to live across
   runs. AgentRefinery may end up wanting an equivalent log of its own
   (e.g., inside `refinery-process-name/`) once its full command spec is
   settled — see §7, open item 4 — but nothing should be assumed about its
   shape yet.
3. **"Rail" / "Riel," "fixed core" / "judgment zone," and
   `agentrefinery-buildvalidation` → `agentrefinery-build-validation`** —
   all pre-date the 2026-07-28 split and are documented in AgentRails'
   `PRD.md` §3, since they concern the Rail-production mechanism, not
   AgentRefinery's own concerns. Not repeated here.

---

## 4. The refinement concept

**Refinement** = running an already-built Rail's `process-name` skill N
times, potentially with increasingly capable models each time, and keeping
the best result seen so far — without touching the Rail's own definition
(`context/`) and without relying on the Rail's own output directory to
survive across runs (it doesn't; see §2).

A pass is "refining" only in the sense that raw material is refined in
stages: each pass is a fresh, independent run of the Rail (per AgentRails'
own destructive Phase 4), and it is AgentRefinery's job — not AgentRails' —
to decide whether that fresh run is worth keeping over the prior best one.

This makes the "increasingly capable LLMs improve the same result" claim
into something AgentRefinery is responsible for demonstrating, by keeping
an accumulated best-so-far result somewhere that survives a Rail's own
destructive restarts. See §5 for the mechanism proposed for this so far.

---

## 5. Proposed mechanism (settled so far)

This is what has actually been decided, from the user's own description
during the 2026-07-28 design conversation. Everything not stated here is
an open item (§7), not an implicit default.

- AgentRefinery mirrors AgentRails' 3-command naming pattern:
  `agentrefinery-design`, `agentrefinery-build`,
  `agentrefinery-build-validation` — kept as separate commands from
  AgentRails' own `agentrails-design` / `agentrails-build` /
  `agentrails-build-validation` specifically so nothing about those has to
  change.
- **`agentrefinery-build`'s generated skill, after a `process-name` run
  completes**, copies the entire contents of `output-process-name/` into
  AgentRefinery's own directory, `refinery-process-name/`. This keeps
  AgentRails' `output-process-name/` clean and ready for its own next
  destructive restart (per AgentRails' Phase 4), while
  `refinery-process-name/` is where AgentRefinery accumulates its
  best-so-far result across passes.
- A separate step — referred to by the user as **"Refinación"**
  (Refinement) — reads **both** directories: the fresh
  `output-process-name/` from the pass that just completed, and the
  existing `refinery-process-name/` (the prior best-so-far, if one
  exists). It compares the two and, **only if the fresh pass is genuinely
  better**, writes the improved result into `refinery-process-name/`
  (replacing what was there). If the fresh pass isn't better,
  `refinery-process-name/` is left untouched.
- This "Refinación" step is what makes `refinery-process-name/` behave as
  an accumulating, monotonically-improving result across N runs, even
  though each individual `process-name` run (via AgentRails) is a clean,
  from-scratch restart with no memory of the previous pass.

**Explicitly not yet decided**: which of the 3 commands (or what 4th
concept) implements "Refinación" — see §7, open item 1.

---

## 6. Directory shape implied so far

```
<process-name>/                       ← the Rail, built and owned by AgentRails
├── context/                          ← untouched by AgentRefinery
├── output-process-name/              ← owned by AgentRails; wiped on its own
│                                        Phase 4 destructive restart
├── process-name/SKILL.md             ← untouched by AgentRefinery
├── process-name-validation/SKILL.md  ← untouched by AgentRefinery
└── refinery-process-name/            ← owned by AgentRefinery; accumulates the
                                          best-so-far result across N passes,
                                          survives output-process-name/ being
                                          wiped and re-run
```

Nothing about the exact internal structure of `refinery-process-name/`
(does it mirror `output-process-name/`'s file layout? does it carry its
own tracking file? see §7) has been decided.

---

## 7. Open items — must be resolved before a real command spec exists

These are genuine, currently-unanswered questions, not implementation
details to fill in casually. Each one blocks writing a real
`skills/agentrefinery-*/SKILL.md` — resolve with the user rather than
guessing, per this whole project family's own "assume nothing" rule (§9).

1. **Which command (or concept) is "Refinación"?** The user described it
   as "a process as such, called Refinement" separately from the 3 named
   commands, but also asked to keep exactly 3 commands
   (`agentrefinery-design/build/build-validation`) mirroring AgentRails'
   pipeline. Is Refinación part of `agentrefinery-build`'s own generated
   skill (i.e., the copy-then-compare happens in one step), a separate
   phase of that same generated skill, or the actual job of
   `agentrefinery-design` or `agentrefinery-build-validation`? Not
   specified yet.
2. **What defines "better"?** The comparison between a fresh
   `output-process-name/` and the existing `refinery-process-name/` needs
   a concrete, checkable definition of improvement — otherwise this is
   exactly the kind of unverifiable judgment call AgentRails' own
   Backbone/Validation model exists to avoid. Candidates not yet decided
   between: re-running `process-name-validation`'s checklist against both
   and preferring more checklist items passed; a model-driven qualitative
   comparison with no fixed rubric; some hybrid. This is the single
   biggest open question in the whole project.
3. **What granularity does the comparison operate at?** Whole-output
   (replace all of `refinery-process-name/` if the new pass wins overall),
   or per-deliverable (keep whichever version of each individual file is
   better, potentially mixing files from different passes)? Not decided.
4. **What does `agentrefinery-build-validation` validate?** Unlike
   AgentRails' `process-name-validation` (which checks a run against
   `Validation.md`'s fixed checklist), there is no equivalent
   "RefinementValidation.md" concept yet. Does this command validate that
   a refinement pass was applied correctly (mechanically, e.g. "did
   `refinery-process-name/` actually get compared and possibly updated"),
   or does it participate in answering open item 2 ("is this actually
   better")? Not decided.
5. **What does `agentrefinery-design` do at all?** In the mirrored
   3-command pattern, `agentrefinery-design` would be the only
   judgment-requiring step, matching AgentRails' own division of labor
   (§6 of AgentRails' PRD). But refinement isn't a document-drafting
   problem the way building a Rail is — there's no obvious "draft context
   bundle" equivalent for a refinement process. Possibly this command
   defines the not-yet-invented rubric/criteria for open item 2, once per
   Rail, so `agentrefinery-build`'s generated skill has something concrete
   to apply on every pass. Not decided.
6. **Does `refinery-process-name/` need its own tracking file or log?**
   The old, now-removed per-Rail `Changelog.md` (§3, item 2) used to log
   improvement passes. AgentRefinery may want an equivalent inside
   `refinery-process-name/` once the above are resolved, but nothing
   should be assumed about its schema yet.
7. **Orchestration**: who/what decides to run `process-name` a 2nd, 3rd,
   ... Nth time, and with which model? Out of scope for AgentRails (its
   own Phase 4 just asks the user "run again?" once resolved) — presumably
   in scope for AgentRefinery, but not yet designed.

---

## 8. Repository structure of AgentRefinery itself

```
AgentRefinery/
├── README.md          ← pitch + current scope + pointer to AgentRails
├── PRD.md              ← this file — what's settled, what's open
├── DESIGN-NOTES.md     ← working design log (session history, incl. the
│                          original combined-project design)
├── Changelog.md        ← this project's own release history (software,
│                          not to be confused with the removed per-Rail
│                          concept — see §3, item 2)
├── CONTRIBUTE.md        ← contribution conventions
├── LICENSE              ← MIT, Altepetl
├── skills/              ← currently EMPTY; agentrefinery-design/build/
│                          build-validation don't exist yet (§7 blocks them)
├── templates/           ← currently EMPTY; no AgentRefinery-specific
│                          document types have been designed yet
├── package.json         ← npm package metadata for a future installer
└── bin/cli.js           ← `npx agent-refinery install` entry point
                             (currently has nothing real to install — see
                             package.json's own note)
```

This is the **builder repo** for the refinement concern only — it is not
itself a Rail, and it no longer builds Rails either (that's AgentRails).
`skills/` and `templates/` are placeholders until §7's open items are
resolved and a real `agentrefinery-*` command spec can be written.

---

## 9. Documentation & language standards (operating principles)

Unchanged from before the split — still apply to everything in this repo:

1. **English only, everywhere.** No exceptions for etymology notes,
   parenthetical asides, or terms that started as shorthand in a design
   discussion.
2. **Command names are fully hyphen-separated** — no concatenated compound
   words.
3. **Assume nothing; document everything.** Especially important here
   given how much of this project is currently open items (§7) rather
   than settled spec — resist the temptation to fill gaps with plausible-
   sounding invented behavior. Mark it as open, or ask.
4. **Bold emphasis is a formatting choice, independent of translation.**

---

## 10. Glossary

- **Rail** — defined and owned by the sibling **AgentRails** project, not
  here. See AgentRails' `PRD.md` §2.
- **AgentRails** — the separate, sibling repo that owns Rail production
  (the `agentrails-design` / `agentrails-build` /
  `agentrails-build-validation` pipeline) and a Rail's own, destructive
  re-run behavior. AgentRefinery depends on it and never modifies it.
- **`process-name`** — a Rail's durable identifier, defined by AgentRails.
  AgentRefinery uses the same identifier to refer to the same Rail; it
  never invents one of its own.
- **`output-process-name/`** — a Rail's runtime output directory, owned by
  AgentRails; wiped on every Phase 4 destructive restart. AgentRefinery
  reads from it but never writes to it.
- **`refinery-process-name/`** — AgentRefinery's own directory,
  accumulating the best-so-far result across N passes of a Rail. Survives
  `output-process-name/` being wiped and re-run. See §5, §6.
- **Refinación / Refinement** — the not-yet-fully-specified step that
  compares a fresh `output-process-name/` pass against the existing
  `refinery-process-name/` best-so-far, and updates the latter only if the
  fresh pass is genuinely better. See §5, §7 (open item 1) for what's
  unresolved about it.
- **`agentrefinery-design` / `agentrefinery-build` /
  `agentrefinery-build-validation`** — the 3 commands AgentRefinery will
  eventually ship, mirroring AgentRails' naming pattern. None of their
  full behavior is specified yet beyond what's in §5 — see §7.
