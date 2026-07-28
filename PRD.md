---
title: AgentRefinery — Product Requirements Document
status: active
version: 0.3.0
created: 2026-07-27
updated: 2026-07-28
role: Single, self-contained reference for the entire AgentRefinery project —
  problem, concept, relationship to the sibling AgentRails project, the
  settled 2-command mechanism that builds and validates the
  `process-name-refine` skill, and the pluggable Refinement Engine
  specification concept. Written so a reader (human or AI agent) with no
  memory of how this project came to be can pick it up and continue work
  correctly, without re-deriving anything from a prior conversation.
---

# AgentRefinery — Product Requirements Document

## 0. How to use this document

This is the authoritative specification of AgentRefinery. Two prior design
attempts preceded this one (see §3): a combined AgentRails/AgentRefinery
project, then a first cut at a split-off AgentRefinery with a fully open
command spec. This version resolves that spec concretely, following a
2026-07-28 design conversation.

- **`README.md`** is the pitch + quick reference for what AgentRefinery is
  and how it relates to AgentRails.
- **This document (`PRD.md`)** is the fuller spec: every concept, every
  document's structure, the 2-command pipeline, and the naming/design
  history.
- **`DESIGN-NOTES.md`** is the working scratchpad, kept as a session-by-
  session log of how the design evolved — including the original combined
  design and the first, fully-open split-off attempt, both now superseded.
- **`engines/ResearchRefinementEngine.md`** is the bundled, default
  Refinement Engine specification — see §5 for what that concept is and
  why it's user-replaceable.
- **`skills/`** holds `agentrefinery-build/SKILL.md` and
  `agentrefinery-build-validation/SKILL.md` — the 2 commands this project
  ships. There is no `agentrefinery-design`; see §3, item 4, for why.

If you are an AI agent picking up this project cold: read this document in
full, then read the sibling AgentRails repo's `PRD.md` before assuming
anything about what a "Rail" is — AgentRefinery does not define that
concept itself.

---

## 1. What AgentRefinery is, and what it isn't

**AgentRefinery does not produce Rails.** That job belongs entirely to the
sibling **AgentRails** project. A Rail — a `context/` bundle plus a
matched pair of Agent Skills for a given `process-name` — is built
exclusively via AgentRails' `agentrails-design` → `agentrails-build` /
`agentrails-build-validation` pipeline. AgentRefinery takes an
already-built, already-run, already-validated Rail as a **given input**
and never redefines, edits, or rebuilds it.

**AgentRefinery's own job**: given a Rail that already runs consistently
(that's what AgentRails guarantees) and has already produced one fully
validated pass, build a **third command** — `process-name-refine` — that
compares that pass against whatever the Rail produces on subsequent runs
(potentially by increasingly capable LLMs) and keeps the best result seen
so far. AgentRails intentionally doesn't care about this — a bare Rail's
re-run is a clean, destructive restart (AgentRails' `PRD.md` §10.2, Phase
4) that discards `output-process-name/` entirely on every repeat pass.
AgentRefinery exists specifically to keep what's worth keeping across
those destructive restarts, and to decide, run over run, whether a new
pass is actually an improvement before treating it as the new best result.

This is a narrower, harder problem than producing the Rail itself:
"consistent execution" is externally verifiable (a checklist either
passes or it doesn't — see AgentRails' `Validation.md`), but "this pass is
*better than* the previous best pass" has no single, universal, fixed
definition. AgentRefinery's answer (§5) is to make that definition itself
a **pluggable, user-replaceable specification** — the **Refinement
Engine** — rather than something AgentRefinery hardcodes.

---

## 2. Relationship to AgentRails

- AgentRails and AgentRefinery are **separate repositories, separate
  products**, split on 2026-07-28 because they answer different
  questions: AgentRails asks "does this process run the same way every
  time, on any model?"; AgentRefinery asks "did running it again, with a
  better model, actually make the result better?"
- AgentRefinery **depends on** AgentRails: it has nothing to refine
  without a Rail already built, already run once by `process-name`, and
  already confirmed by `process-name-validation` (§6.1's preconditions).
- AgentRefinery **never modifies** AgentRails' own commands, generated
  skills, or a Rail's `context/` bundle. It reads a Rail's runtime output
  (`output-process-name/`) as input and writes only to directories it
  owns itself: `process-name-refine/` (the generated skill package) and
  `output-process-name-refine/` (that skill's own runtime output). It
  never writes into `context/` or `output-process-name/`, and it never
  changes how `process-name` or `process-name-validation` behave.
- Nothing about the fixed core / judgment zone mechanism, the 5-document
  context bundle, the 3-command Rail-building pipeline, the
  `ProcessTracking.md` / `ValidationTracking.md` state machines, or the
  destructive Phase-4 restart is owned here. For all of that, see the
  sibling AgentRails repo's `PRD.md`.

---

## 3. Naming and design decisions — history and rationale

1. **Project started as "AgentRails," was renamed to "AgentRefinery," then
   split back into two sibling projects (2026-07-28).** A single combined
   project conflated "run this process consistently" and "decide whether
   a repeat pass improved on the last one" inside one Phase 4 step. The
   fix: split into two sibling repos. **AgentRails** kept the mechanism
   (fixed core/judgment zone, the 3-command build pipeline, a Rail's own
   destructive re-run). **AgentRefinery** (this repo) kept the narrower
   cross-run refinement concern, consuming Rails produced by AgentRails.
2. **A first split-off attempt at this repo's own PRD (2026-07-28,
   superseded by this version) left the entire command spec as open
   items** — no agreed definition of "better," no agreed command
   responsible for the comparison step, no agreed answer for what
   `agentrefinery-design` would even do. This version resolves all of
   that; see §5 onward. `DESIGN-NOTES.md` keeps the earlier, fully-open
   version as historical record.
3. **The comparison/synthesis step the user called "Refinación" is
   implemented by `agentrefinery-build`'s generated skill,
   `process-name-refine`, not by a separate command.** There is exactly
   one runtime artifact this project produces per Rail:
   `process-name-refine/SKILL.md`. `agentrefinery-build` builds it;
   `agentrefinery-build-validation` checks that it was built correctly.
   Neither build command runs the comparison itself — that happens every
   time a user invokes the generated `/process-name-refine` skill.
4. **There is no `agentrefinery-design`.** In AgentRails, `agentrails-
   design` exists because building a Rail requires resolving ambiguity in
   a raw process description before two mechanical build steps can run.
   Here, there's no equivalent ambiguity to resolve up front: the
   objectives are already fixed (AgentRails' own `Backbone.md`, already
   reviewed and built upon), and the first pass's actual output already
   exists (`output-process-name/`, already validated). `agentrefinery-
   build` uses both directly, together with a Refinement Engine spec
   (§5), to derive the refinement step list — see §6.1's note on why this
   makes `agentrefinery-build` slightly less purely mechanical than
   AgentRails' own build commands, and why that's a deliberate, documented
   exception rather than an oversight.
5. **`agentrefinery-build-validation` keeps its full hyphen-separated
   name** (not shortened to `agentrefinery-validation`), for naming
   consistency with AgentRails' own `agentrails-build-validation`, even
   though there's no `agentrefinery-design` to disambiguate it from.
6. **The generated skill's own runtime output directory is
   `output-process-name-refine/`, not `process-name-refine/`.** An
   earlier phrasing of this design called the output directory
   `process-name-refine/` — the same name as the generated skill package
   itself (`process-name-refine/SKILL.md`). Renamed to
   `output-process-name-refine/` to follow the same `output-` prefix
   convention AgentRails already uses (`process-name/SKILL.md` vs.
   `output-process-name/`) and to avoid the name collision. See §7 for
   the full directory layout.
7. **The Refinement Engine specification is a pluggable, user-replaceable
   document, not a fixed algorithm AgentRefinery hardcodes.**
   `engines/ResearchRefinementEngine.md` (translated from the user's
   original Spanish draft) ships as the **default/reference**
   specification — tuned for comparing independent LLM research passes
   (semantic matching, clustering, evidence scoring, contradiction
   detection, ranking, synthesis, targeted follow-up research). A user can
   supply their own instead, tailored to their own process, and is
   explicitly encouraged to invest real budget in generating a strong one
   (e.g. commissioning a more expensive, more capable LLM to draft a
   domain-specific engine) — see §5.2. Like a Rail itself, AgentRefinery's
   job is only to provide the guardrail that lets *whichever* engine spec
   is supplied run consistently; it is not the ceiling on refinement
   quality.

---

## 4. The refinement concept

**Refinement** = running an already-built, already-validated Rail's
`process-name` skill N times, potentially with increasingly capable
models each time, and keeping the best result seen so far — without
touching the Rail's own definition (`context/`) and without relying on
the Rail's own output directory to survive across runs (it doesn't; see
§2).

A pass is "refining" only in the sense that raw material is refined in
stages: each pass is a fresh, independent run of the Rail (per AgentRails'
own destructive Phase 4), and it is `process-name-refine` — not
AgentRails — that decides whether that fresh run is worth keeping over
the prior best one.

---

## 5. The Refinement Engine specification

### 5.1 What it is

A **Refinement Engine specification** is a standalone document describing
*how* to compare a fresh pass's output against an accumulated best-so-far
result and decide whether the fresh pass is an improvement. It is the
concept that answers the single hardest open question this project
faced: **what defines "better"?**

Rather than AgentRefinery hardcoding one universal definition (which
can't exist generically — "better" means something different for a
research synthesis process than for, say, a code-generation or a
document-drafting process), the definition itself is an input parameter,
the same way a process description is AgentRails' input. `agentrefinery-
build` reads whichever Refinement Engine spec is given (or the bundled
default) and derives `process-name-refine`'s concrete comparison logic
from it.

### 5.2 The bundled default: `engines/ResearchRefinementEngine.md`

This repo ships one reference implementation,
`engines/ResearchRefinementEngine.md`, aimed specifically at refining the
output of **research-style processes** — where a Rail's deliverable is a
set of findings/claims/conclusions, and multiple independent passes (by
different LLMs) can each surface overlapping, complementary, or
contradictory claims. Its pipeline, summarized (full detail in the file
itself):

1. **Normalization** — convert each pass's output into a common structure
   (claim, explanation, evidence, confidence, source).
2. **Semantic matching** — determine when two differently-worded claims
   express the same idea (not just string similarity).
3. **Clustering** — group related claims around a shared concept.
4. **Evidence scoring** — weigh each claim by consensus, evidence quality,
   independence of sources, and reasoning quality — explicitly not by
   repetition count alone ("three LLMs repeating the same source is not
   three independent evidences").
5. **Contradiction detection** — flag genuine conflicts vs. differences
   that are just context-dependent or a wording difference.
6. **Ranking (Reciprocal Rank Fusion + weighted factors)** — combine
   per-pass rankings into one global ranking.
7. **LLM judge / synthesis** — produce the refined conclusion, explicitly
   distinguishing FACT / INFERENCE / HYPOTHESIS / UNCERTAINTY /
   CONTRADICTION, never treating majority agreement as proof.
8. **Iterative/targeted refinement** — identify which remaining
   uncertainties would actually change the conclusion, and recommend a
   *specific* follow-up investigation rather than "research everything
   again."
9. **Stopping condition** — e.g. confidence ≥ 0.90, no unresolved critical
   contradictions, no HIGH/CRITICAL-impact uncertainty, bounded by a
   maximum iteration count.

This is the **reference example, not the only valid shape a Refinement
Engine spec can take.** A process whose deliverable isn't a set of
research findings (e.g. a generated document, a codebase, a design)
needs a differently-shaped spec — its own definition of what a "claim,"
"contradiction," or "evidence" even means for that kind of output. Users
are expected to supply their own for such processes.

### 5.3 Investing in a better spec is expected, not a fallback

Because the Refinement Engine spec is the concept doing the actual
judgment-call work (deciding "better"), its quality has an outsized effect
on how good `process-name-refine` ends up being — far more than most
other inputs in this pipeline. A user who cares about refinement quality
for a specific, valuable process is expected to invest real effort/cost
into a strong spec: for example, commissioning a more capable, more
expensive LLM specifically to draft a domain-tuned Refinement Engine
specification for their process, rather than relying on the bundled
default outside its intended (research-synthesis) scope. This mirrors
Rails themselves: **AgentRefinery's job is only to provide the guardrail
that lets whichever engine spec is supplied run consistently — it is not,
and should not be treated as, the ceiling on refinement quality.**

---

## 6. System architecture — the 2-command pipeline

```
Prerequisites (all must already be true before agentrefinery-build runs):
  - <process-name>/context/ exists (built by AgentRails' agentrails-design)
  - <process-name>/process-name/SKILL.md and
    <process-name>/process-name-validation/SKILL.md exist and have been run
    (built by AgentRails' agentrails-build / agentrails-build-validation)
  - <process-name>/output-process-name/ContentTracking.md +
    ValidationTracking.md both show every row fully OK — i.e. the Rail's
    first pass is complete AND validated, not just executed
  - A Refinement Engine specification is available (the bundled default,
    engines/ResearchRefinementEngine.md, or a user-supplied one)

        │
        ▼
agentrefinery-build        ← reads context/Backbone.md (+ Readme.md's
(mechanical, but combines     escalation rule), output-process-name/ (the
 already-fixed inputs —        actual first validated pass), and the
 see PRD.md sec.3, item 4)     resolved Refinement Engine spec
        │
        ▼
<process-name>/process-name-refine/     ← the generated skill package:
  ├── SKILL.md                             the runnable comparison/synthesis
  ├── RefinementPlan.md                    skill (scaffolded via skill-creator)
  └── refinement-engine.md                 the traceable step list this
                                            package's SKILL.md implements
                                            a frozen copy of whichever engine
                                            spec was actually used to build
                                            this package
        │
        ▼
agentrefinery-build-validation   ← reads RefinementPlan.md + context/Backbone.md,
(mechanical QA check on the        confirms every step traces to a real
 build output, not a 2nd skill)    Backbone ID and a real engine-spec stage,
                                    and that every Backbone ID is covered
        │
        ▼
   Confirms /process-name-refine is ready to run
```

**Key difference from AgentRails' own 3-command pipeline**:
`agentrefinery-build-validation` does **not** produce a second runnable
skill the way `agentrails-build-validation` produces
`process-name-validation`. It is a one-shot deterministic QA check on
`agentrefinery-build`'s output. The only runtime artifact this whole
project produces, per Rail, is `process-name-refine/SKILL.md`.

---

## 7. Directory layout

```
<process-name>/                            ← the Rail, built and owned by AgentRails
├── context/                               ← untouched by AgentRefinery
├── output-process-name/                   ← owned by AgentRails; wiped on its own
│                                             Phase 4 destructive restart; read-only
│                                             input to agentrefinery-build and to the
│                                             generated process-name-refine skill
├── process-name/SKILL.md                  ← untouched by AgentRefinery
├── process-name-validation/SKILL.md       ← untouched by AgentRefinery
├── process-name-refine/                   ← AgentRefinery's generated skill package
│   ├── SKILL.md                              (built by agentrefinery-build, checked
│   ├── RefinementPlan.md                      by agentrefinery-build-validation)
│   └── refinement-engine.md
└── output-process-name-refine/            ← AgentRefinery's own runtime output;
    ├── RefinementTracking.md                 survives output-process-name/ being
    └── (accumulated best-so-far               wiped and re-run; only
        deliverables)                          process-name-refine writes here
```

---

## 8. Command specifications

### 8.1 `agentrefinery-build`

**File**: `skills/agentrefinery-build/SKILL.md`. **Judgment**: limited —
see §3, item 4, for why this command carries a small amount of judgment
(mapping Backbone objectives to Refinement Engine stages, grounded in the
actual first-pass output) despite otherwise following AgentRails' "build
commands are mechanical" convention.

**Purpose**: given a Rail that has already completed one fully validated
pass, build the `process-name-refine` skill that will compare all
subsequent passes against the best result seen so far.

**Hard prerequisite**: the `skill-creator` skill
(https://claude.com/plugins/skill-creator), same as both of AgentRails'
own build commands — `process-name-refine/SKILL.md` must always be
scaffolded through it, never hand-rolled.

**Required inputs**:
1. `<process-name>/context/Backbone.md` — the objectives (`O#`) and hard
   limits (`L#`) that define what "the process succeeded" means. These
   are what every `RefinementPlan.md` step must trace back to.
   `<process-name>/context/Readme.md` is also read, for its escalation
   rule text (carried into the generated skill, same as AgentRails does).
2. `<process-name>/output-process-name/` — the actual deliverables from
   the Rail's first pass, plus `ProcessTracking.md` and
   `ValidationTracking.md`.
3. A Refinement Engine specification — **optional parameter**. If not
   given, defaults to the bundled `engines/ResearchRefinementEngine.md`.
   If given, must be a path to a document following the same shape (a set
   of named stages, each with concrete instructions for the LLM executing
   that stage).

**Preconditions** (stop and report if not met — do not proceed on a
partial or unvalidated Rail):
- `context/Backbone.md` and `context/Readme.md` exist.
- `output-process-name/ProcessTracking.md` exists and every row is `✅`.
- `output-process-name/ValidationTracking.md` exists and every row is
  `✅` — i.e. `process-name-validation` has already confirmed this pass,
  not just that `process-name` finished running.

**Behavior, in order**:
1. Read `Backbone.md`'s objectives/hard limits, `Readme.md`'s escalation
   rule, `output-process-name/`'s actual deliverables and tracking files,
   and the resolved Refinement Engine spec's stage list.
2. Draft `RefinementPlan.md`: for each stage in the Refinement Engine
   spec, write a concrete step that cites the Backbone `O#`/`L#` ID(s) it
   serves, split into **fixed core** (the mechanical part: which files in
   `output-process-name/` vs. `output-process-name-refine/` get compared,
   what gets written where, on what condition) and **judgment zone** (the
   part the spec's own "instructions for the LLM" text governs — semantic
   matching, evidence scoring, synthesis, and so on). Ground every step
   in the real file layout of `output-process-name/` — this is exactly
   the advantage a design-time ambiguity-resolution step would have had,
   except here it comes from the real first pass already existing.
   Include an explicit hard-limit re-check step: whatever candidate ends
   up chosen as the new best-so-far must not violate any `L#`, even if the
   Refinement Engine spec's own ranking would otherwise favor it.
3. Every Backbone `O#`/`L#` touched by the process must be covered by at
   least one `RefinementPlan.md` step; every step must cite a real
   Backbone ID and a real Refinement Engine stage. An objective that can't
   be mapped to any stage in a checkable way is unresolved ambiguity —
   surface it to the user rather than guessing (this is the one place a
   build command in this pipeline is allowed to ask, precisely because no
   earlier design step exists to have caught it; see §3, item 4).
4. Copy the resolved Refinement Engine spec verbatim into
   `process-name-refine/refinement-engine.md`, so the generated package is
   self-contained and `agentrefinery-build-validation` (and any future
   re-check) doesn't depend on the original spec file still existing at
   its original path.
5. Compose the generated skill's required behavior for `skill-creator`:
   - **Bootstrap phase**: if `output-process-name-refine/` doesn't exist
     or is empty, copy the entirety of `output-process-name/` into it,
     write a `RefinementTracking.md` row with verdict `SEEDED`, and tell
     the user there's nothing to compare yet — recommend re-running the
     Rail's `/process-name` (ideally with a different/more capable LLM),
     then re-running `/process-name-refine`.
   - **Comparison phase** (runs whenever a best-so-far baseline already
     exists in `output-process-name-refine/`): apply `RefinementPlan.md`'s
     steps against the current contents of `output-process-name/` (the
     fresh pass) and `output-process-name-refine/` (the existing
     best-so-far). If the synthesis step concludes the fresh pass is
     genuinely better (per the Refinement Engine spec's own criteria),
     write the improved result into `output-process-name-refine/`,
     replacing what was there; otherwise leave it untouched. Either way,
     append a row to `RefinementTracking.md` (see §9.1 for schema).
   - **Escalation**: if the comparison can't reach a clear verdict per the
     spec's own criteria, stop and ask the user — never guess and pick a
     side. This is the same fixed, non-negotiable escalation rule as
     every Rail's `Readme.md`, carried into this generated skill too.
   - **Re-runnability**: `/process-name-refine` can be invoked repeatedly,
     including without a new Rail pass in between — since the comparison
     step isn't fully deterministic, re-running it (especially with a
     different LLM) can surface a different verdict. The generated
     skill's own `Readme`/description should say so explicitly (§10).
6. Invoke `skill-creator` with: name = `<process-name>-refine`;
   description = what it refines plus a trigger phrase (e.g. "compares the
   latest `<process-name>` pass against the best result seen so far and
   keeps whichever is better; use when the user wants to
   refine/improve/compare passes of the `<process-name>` process"); output
   path = `<process-name>/process-name-refine/SKILL.md`.
7. Confirm to the user that the skill is ready, and that
   `agentrefinery-build-validation` should run next to confirm the build.

### 8.2 `agentrefinery-build-validation`

**File**: `skills/agentrefinery-build-validation/SKILL.md`. **Judgment**:
no — purely mechanical once `agentrefinery-build` has already produced
its output.

**Purpose**: confirm that `process-name-refine/SKILL.md` was built
correctly — every step traceable to a real Backbone ID and a real
Refinement Engine stage, every Backbone ID covered, nothing invented or
dropped. This mirrors exactly how AgentRails' `agentrails-build-
validation` checks `Workflow.md`'s traceability against `Backbone.md` —
just applied to `RefinementPlan.md` instead.

**Does not produce a second runnable skill** — unlike AgentRails' own
`agentrails-build-validation`, this is a one-shot QA check, not a skill
generator. See §6 for why.

**Required inputs**:
- `<process-name>/process-name-refine/RefinementPlan.md`
- `<process-name>/process-name-refine/refinement-engine.md` (the frozen
  copy `agentrefinery-build` wrote — checked against, not the original
  external file, so this validation is reproducible even if the original
  path changes later)
- `<process-name>/context/Backbone.md`

**Preconditions**: all 3 files exist.

**Behavior**:
1. Read `RefinementPlan.md`, `refinement-engine.md`, `Backbone.md`.
2. Check traceability: every `RefinementPlan.md` step cites a real
   Backbone `O#`/`L#` and a real stage name from `refinement-engine.md`.
   Every Backbone `O#`/`L#` is covered by at least one step. The hard-
   limit re-check step (§8.1, step 2) is present.
3. Check that `process-name-refine/SKILL.md` actually implements
   `RefinementPlan.md`'s steps faithfully — same sequence, no invented,
   skipped, or reordered steps, bootstrap-vs-comparison branching present,
   `RefinementTracking.md` generation present, escalation rule present.
4. On any gap: stop and report it — send it back to `agentrefinery-build`
   to fix (regenerate `RefinementPlan.md` and/or the skill), rather than
   patching the generated skill by hand.
5. On success: confirm `/process-name-refine` is ready to run, and remind
   the user of the full recommended cycle (§10) — running `/process-name`
   → `/process-name-validation` → `/process-name-refine` repeatedly, ideally
   varying the LLM each time, is what actually produces improving results;
   `/process-name-refine` alone, run against the same two directories
   repeatedly, can still shift its verdict a little run to run (since the
   comparison isn't fully deterministic) but has less new evidence to work
   with than a fresh Rail pass would provide.

---

## 9. Runtime mechanics of the generated `process-name-refine` skill

### 9.1 `RefinementTracking.md` schema

Lives in `output-process-name-refine/`, one row per invocation of
`/process-name-refine`:

`ITERATION | AGENT | VERDICT | CONFIDENCE | DETAILS | START | END`

- **ITERATION**: sequential counter, starting at 1 (the bootstrap pass).
- **AGENT**: which model/agent executed this invocation — same
  auditability purpose as AgentRails' own tracking files.
- **VERDICT**: `SEEDED` (bootstrap — nothing compared yet), `IMPROVED`
  (fresh pass replaced the prior best-so-far), `NOT_IMPROVED` (fresh pass
  compared but not written, prior best-so-far kept), or `BLOCKED`
  (escalated to the user, no verdict reached).
- **CONFIDENCE**: whatever confidence figure the resolved Refinement
  Engine spec's own synthesis step produces (e.g. the default spec's own
  §13 confidence metric), if it defines one; empty if the spec doesn't.
- **DETAILS**: brief summary of what changed (if `IMPROVED`), why nothing
  changed (if `NOT_IMPROVED`), or what's blocking (if `BLOCKED`) —
  including any remaining uncertainties the spec's own synthesis step
  flagged, so a user deciding whether to run the Rail again knows what a
  next pass should target.
- **START / END**: timestamps.

Never cleared or reset — this is the permanent, accumulating record of
every refinement pass across models and time for this Rail, since
`output-process-name-refine/` itself is never wiped (only
`output-process-name/`, by AgentRails' own Phase 4, is).

### 9.2 State machine

```
if output-process-name-refine/ doesn't exist or is empty:
    copy all of output-process-name/ into output-process-name-refine/
    append RefinementTracking.md row: VERDICT = SEEDED
    report: "Baseline established. Nothing to compare yet — re-run the
             Rail's /process-name (ideally with a different/more capable
             LLM), then run /process-name-refine again to look for an
             improvement."
    stop

# a best-so-far baseline already exists -> attempt a comparison pass
apply RefinementPlan.md's steps to:
    fresh pass  = output-process-name/  (current contents)
    best-so-far = output-process-name-refine/ (current contents)

if the Refinement Engine spec's synthesis step reaches a clear verdict:
    if fresh pass is genuinely better:
        write the improved result into output-process-name-refine/
        append RefinementTracking.md row: VERDICT = IMPROVED
    else:
        leave output-process-name-refine/ untouched
        append RefinementTracking.md row: VERDICT = NOT_IMPROVED
    report the verdict, confidence (if any), and remaining uncertainties
else:
    append RefinementTracking.md row: VERDICT = BLOCKED
    stop and ask the user rather than guessing
```

### 9.3 Re-running the full cycle

The generated skill's own description/Readme text must state explicitly:
`/process-name-refine` can be invoked on its own, repeatedly, and can
produce a slightly different verdict each time since the comparison isn't
fully deterministic — but it has more genuinely new evidence to work with,
and therefore produces better results, when run as part of the full cycle:

```
/process-name             (AgentRails — produces a fresh pass)
/process-name-validation  (AgentRails — confirms the fresh pass is complete)
/process-name-refine      (AgentRefinery — compares it against the best-so-far)
```

repeated N times, ideally with a different (increasingly capable) LLM
executing each command each cycle.

---

## 10. Repository structure of AgentRefinery itself

```
AgentRefinery/
├── README.md          ← pitch + current scope + pointer to AgentRails
├── PRD.md              ← this file
├── DESIGN-NOTES.md     ← working design log (session history, incl. both
│                          the original combined-project design and the
│                          first, fully-open split-off attempt)
├── Changelog.md        ← this project's own release history
├── CONTRIBUTE.md        ← contribution conventions
├── LICENSE              ← MIT, Altepetl
├── engines/
│   └── ResearchRefinementEngine.md   ← bundled default/reference spec (§5)
├── skills/
│   ├── agentrefinery-build/SKILL.md
│   └── agentrefinery-build-validation/SKILL.md
├── package.json         ← npm package metadata for the installer
└── bin/cli.js           ← `npx agent-refinery install` entry point
```

This is the builder repo for the refinement concern only — it is not
itself a Rail, and it does not build Rails (that's AgentRails).

---

## 11. Documentation & language standards (operating principles)

1. **English only, everywhere.** No exceptions for etymology notes,
   parenthetical asides, or terms that started as shorthand in a design
   discussion — including `engines/ResearchRefinementEngine.md`, which was
   translated from an original Spanish draft for this exact reason.
2. **Command names are fully hyphen-separated.**
3. **Assume nothing; document everything.**
4. **Never modify AgentRails.** No changes to a Rail's `context/`, its
   `output-process-name/`, or either of its generated skills belong here.
5. **Bold emphasis is a formatting choice, independent of translation.**

---

## 12. Glossary

- **Rail** — defined and owned by the sibling **AgentRails** project. See
  AgentRails' `PRD.md` §2.
- **AgentRails** — the separate, sibling repo that owns Rail production
  and a Rail's own destructive re-run behavior. AgentRefinery depends on
  it and never modifies it.
- **`process-name`** — a Rail's durable identifier, defined by AgentRails.
  AgentRefinery reuses it, never invents its own.
- **`output-process-name/`** — a Rail's runtime output directory, owned by
  AgentRails; wiped on every Phase 4 destructive restart. AgentRefinery
  reads from it but never writes to it.
- **Refinement Engine specification** — a standalone, pluggable document
  defining how to compare a fresh pass against a best-so-far result and
  decide whether it's an improvement. See §5.
- **`engines/ResearchRefinementEngine.md`** — the bundled default/
  reference Refinement Engine spec, tuned for research-style processes.
  See §5.2.
- **`agentrefinery-build`** — builds `process-name-refine/SKILL.md` and
  its companion `RefinementPlan.md` / `refinement-engine.md`. See §8.1.
- **`agentrefinery-build-validation`** — a one-shot QA check confirming
  `process-name-refine` was built correctly; does not itself produce a
  runnable skill. See §8.2.
- **`process-name-refine`** — the runnable skill this project produces
  per Rail; compares a fresh pass against the accumulated best-so-far and
  updates it only if genuinely better. See §9.
- **`RefinementPlan.md`** — the traceable step list, derived from
  Backbone.md + the resolved Refinement Engine spec, that
  `process-name-refine/SKILL.md` implements. Lives inside the generated
  skill package, not in a Rail's `context/`. See §8.1.
- **`output-process-name-refine/`** — `process-name-refine`'s own runtime
  output directory; accumulates the best-so-far result across N passes;
  survives `output-process-name/` being wiped and re-run. See §7, §9.
- **`RefinementTracking.md`** — the per-invocation log inside
  `output-process-name-refine/`. See §9.1.
- **Escalation rule** — the fixed, non-negotiable rule that an executing
  agent stops and asks the user rather than guessing when it can't reach
  a clear verdict. Carried over from AgentRails' own Rails; applies to
  `process-name-refine` too. See §8.1, §9.2.
- **skill-creator** — the external skill both build commands scaffold
  their output through. Hard prerequisite.
