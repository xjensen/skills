---
name: jdev
version: 0.7.0
description: |
  Drive a feature through its lifecycle. Takes a feature JOB and a PHASE
  (define, research, design, spike, plan, go, verify, challenge,
  resolve, status, or anything else), creates a working folder at .temp/<JOB>/
  in the current project, and maintains three consolidated working files
  there: a definition file (define, research, design, spike), a plan file
  (plan), and a done file (go, verify). The done file always carries a
  PR/changelog-ready summary of the work, refreshed automatically by go.
  Use when asked to "define", "research", "design", "spike", "plan", or "go"
  on a feature, to "draft requirements", "scope a new feature", "write a
  product proposal", to "challenge" or "verify" a feature's artifacts, to
  check a feature's "status", or to "resolve" review comments left in a
  feature's output files.
argument-hint: <phase> <job> [instructions]
---

# /jdev — Drive a Feature Through Its Lifecycle

Work a feature forward one phase at a time — define it, research it, plan it,
build it, check it — and persist each phase's output into a small set of
per-feature working files.

## Arguments

The user invoked: `/jdev $ARGUMENTS`

There are three arguments:

- **PHASE** (first token) — which phase to perform this run. Selects a row in
  the [dispatch table](#what-to-produce). Known PHASEs: `define`, `research`,
  `design`, `spike`, `plan`, `go`, `verify`, `challenge`,
  `resolve`, `status`, and `help` (explains the workflow; takes no other
  arguments). Anything else falls back to the default behavior.
- **JOB** (second token) — a short, stable identifier for the feature, used as
  both the folder name and the output filename prefix.
- **INSTRUCTIONS** (everything after the first two tokens) — a free-form blurb
  that guides this run: what to focus on, what to add, what to change. May be
  empty. May reference another job with the `jdev:<job>` shorthand — see
  [Referencing another job](#referencing-another-job).

To parse:

1. Take the first whitespace-delimited token of `$ARGUMENTS` as `PHASE`. Normalize
   it to lowercase and strip anything that isn't `[a-z0-9-]`.
2. Take the second whitespace-delimited token as `JOB`. Normalize it to a
   filesystem-safe slug: lowercase, spaces → hyphens, strip anything that isn't
   `[a-z0-9-]`.
3. Treat the remainder of `$ARGUMENTS` (after the first two tokens, trimmed) as
   `INSTRUCTIONS`.
4. If no PHASE was provided, ask the user which phase to run with AskUserQuestion,
   offering `define`, `plan`, and `go` as options. Do not guess one.
5. **If PHASE is `help`, stop parsing here** — ignore JOB and INSTRUCTIONS, skip
   the folder setup entirely, read `phases/help.md`, and follow it. It produces
   no file.
6. If no JOB was provided, ask the user for it with AskUserQuestion (text
   input). Do not guess one. INSTRUCTIONS may legitimately be empty — don't
   prompt for it.

## Set up the working folder

The lifecycle phases share three consolidated files; only unknown PHASEs get a
file of their own. Create the feature folder at the **project root** and
resolve the output path by phase. Run from the repo so `.temp/` resolves
against the current project:

```bash
JOB="<normalized-job>"
PHASE="<normalized-phase>"
ROOT="$(git rev-parse --show-toplevel 2>/dev/null || pwd)"
DIR="$ROOT/.temp/$JOB"
case "$PHASE" in
  define|research|design|spike) OUT="$DIR/$JOB-definition.md" ;;
  plan)                         OUT="$DIR/$JOB-plan.md" ;;
  go|verify)                    OUT="$DIR/$JOB-done.md" ;;
  challenge|resolve|status)     OUT="" ;;
  *)                            OUT="$DIR/$JOB-$PHASE.md" ;;
esac
mkdir -p "$DIR"
if [ -n "$OUT" ]; then [ -f "$OUT" ] && echo "EXISTS: $OUT" || echo "NEW: $OUT"; fi
```

### Files, owners, and section order

Each shared file has a canonical layout, and each phase **owns** one slice of
it. A phase writes only what it owns (plus the definition file's shared tail
sections) — never rewrite, reorder, or "tidy" another phase's sections.

**`$JOB-definition.md`** — everything about what the feature is:

| Owner | Owns |
|---|---|
| `define` | The document base: title, `## Problem`, `## Goals & non-goals`, `## Users & use cases`, `## Requirements`, `## Success metrics` |
| `research` | `## Research` |
| `design` | `## Design` |
| `spike` | `## Spike` |
| *(shared)* | `## Decisions` and `## Constraints & open questions`, always the last two sections — every definition-file phase records its open items and settled decisions here, not in its own section |

Canonical order: define's base sections, then `## Research`, `## Design`,
`## Spike` (each present only once its phase has run), then `## Decisions`
(created on demand) and `## Constraints & open questions` last.

**`$JOB-plan.md`** — owned entirely by `plan`.

**`$JOB-done.md`** — everything about what actually happened:

| Owner | Owns |
|---|---|
| `go` | `## Implementation` and `## Summary` |
| `verify` | `## Verification` |

Canonical order: `## Implementation`, `## Verification`, `## Summary`.
`go` writes both the implementation record and the summary; the summary is not
a separate phase — every `go` run refreshes it to match the current work.

### Re-entrancy

The skill is **re-entrant** — calling `/jdev <same-phase> <same-job> ...` again
refines that phase's existing output instead of starting over. For shared
files the unit of re-entrancy is the phase's **section**, not the file. Branch:

- **File NEW** — `Write` the file with this phase's content in place. If the
  phase isn't the file's base owner (e.g. `research` runs before `define`
  ever has), still create the file — a title line plus this phase's section —
  and let later phases slot their sections in at canonical positions.
- **File EXISTS, own section absent** — first run of this PHASE. Make sure the
  current contents are in context (if you have not already `Read` `$OUT`
  during this conversation, read it now), generate the section, and `Edit` it
  in at its canonical position.
- **File EXISTS, own section present** — a refinement run. Read the file if
  needed, then apply this run's INSTRUCTIONS by `Edit`ing within the section —
  preserve everything not implicated by the instructions; don't regenerate or
  reorder untouched content, and don't touch other phases' sections.

> **`challenge`, `resolve`, and `status` produce no output file.** Like
> `help`, they skip `$OUT` and the branches above (the case statement sets
> `OUT=""` for them). They still need `$DIR` so they can scan the folder —
> set it up, then go straight to their phase spec. `challenge` writes
> `*** ... ***` markers into the working files and reports its audit in chat.
> (`go` and `verify` also partly deviate: `go` re-runs are remediation rounds
> that do new implementation work — appending an implementation round while
> rewriting the single `## Summary` to match — and `verify` re-runs are fresh
> re-checks of every claim — each spec explains.)

Earlier phases feed later ones. Before any later phase, check the feature
folder for the working files (e.g. `.temp/<JOB>/<JOB>-definition.md`) and
read any that exist so this phase builds on them rather than starting blind —
each phase spec names its expected inputs.

## Referencing another job

INSTRUCTIONS may point at a *different* job with the shorthand `jdev:<job>`
(e.g. `jdev:checkout`). The referenced job is a separate feature with its own
folder — this is how one job's output seeds another. Resolve every such
reference before producing this run's output:

1. **Locate** the referenced folder at `$ROOT/.temp/<job>/`. If it doesn't
   exist, don't invent its contents — tell the user the reference didn't resolve
   and ask how to proceed.
2. **Read** the referenced job's working files relevant to this run — its
   definition, plan, and done files, whichever exist and matter to what
   INSTRUCTIONS asks for.
3. **Pull** the relevant material in as seed guidance for *this* run, then follow
   the rest of INSTRUCTIONS for what to focus on. You're writing into the
   current JOB's files, not the referenced ones — never write into the
   referenced job's folder.

The motivating case is kick-starting a job from another's leftovers. The `go`
phase records a **Follow-ups** list in the done file; pointing a new job's
`define` at it carries those forward:

```
/jdev define billing "jdev:checkout follow-ups — turn the deferred items into requirements"
```

This reads `checkout`'s done file, lifts its Follow-ups, and seeds a fresh
`billing` job from them.

## What to produce

`INSTRUCTIONS` (the third argument) steers every phase:

- On a **new** file or section, treat INSTRUCTIONS as seed guidance for what to
  draft. If empty, fall back to the phase's default structure.
- On an **existing** section's refinement, treat INSTRUCTIONS as the change
  request for this edit (e.g. "add a risks section", "tighten the success
  metrics"). If empty, ask the user what they want changed rather than
  rewriting blindly.

When a refinement resolves an item from a **Constraints & open questions**
section, don't just delete it. Move it into the **Decisions** section in the
same file — create that section (placed just before **Constraints & open
questions**) if it doesn't exist yet — and record it as the decision reached,
not the question asked: state what was settled and, briefly, why. Leave
still-open items where they are. This keeps the resolution history visible
instead of erasing it.

Then branch on `PHASE`. Each known phase's full spec lives in its own file
under `phases/` **in this skill's directory** (the directory containing this
SKILL.md — not the project). **Read `phases/<PHASE>.md` now, before producing
anything, and follow it.** The one-liners below exist for dispatch and for
`help` — they are not the spec:

| PHASE | Writes | One-liner |
|---|---|---|
| `define` | definition file (base) | High-level product requirements: problem, goals, users, success metrics. |
| `research` | `## Research` in definition file | Investigate the problem space: prior art, codebase findings, options, recommendation. |
| `design` | `## Design` in definition file | The feature's outside surface: API contracts, data shapes, UX flows. |
| `spike` | `## Spike` in definition file + throwaway code | Disposable prototype answering the single riskiest unknown. |
| `plan` | `$JOB-plan.md` | Implementation plan: approach, ordered steps, files, testing, rollout. |
| `go` | `## Implementation` and `## Summary` in done file + code changes | Carry out the plan in the project; record what changed and refresh the PR/changelog-ready work summary. |
| `verify` | `## Verification` in done file | Trace definition/plan requirements to actual code and tests; coverage matrix. |
| `challenge` | markers in working files; audit reported in chat | Fresh-context subagent adversarially audits the artifacts; leaves `*** ... ***` markers for `resolve`. |
| `resolve` | edits files in place | Address `*** ... ***` comments across the folder's files. |
| `status` | report only | Dashboard: phases done, markers outstanding, open questions, next move. |
| `help` | report only | Explain the workflow. |

The nominal lifecycle order is `define` → `research` → `design` → `spike` →
`plan` → `go` → `verify`, with `design` and `spike` optional deepeners. When
`verify` finds gaps, `go` and `verify` alternate — `go` re-runs close the open
gaps, `verify` re-runs confirm and tick them off — until the verdict is ready.
Every `go` run also refreshes the done file's `## Summary` — an
external-facing, PR/changelog-ready report of the work — so it always tracks
the current implementation. `challenge`, `resolve`, and `status` are
maintenance phases runnable at any point — `challenge` is most valuable
between `plan` and `go`.

### default (any other PHASE)

The PHASE is not in the table, so there is no `phases/` file to read. Don't
reject it — treat the PHASE token as the **title of the artifact** and let
INSTRUCTIONS define what to produce. These ad-hoc artifacts get their own
standalone file: write a focused, well-structured markdown document for that
phase to `$OUT` (`$DIR/$JOB-$PHASE.md`), using INSTRUCTIONS as the primary
guidance. If INSTRUCTIONS is empty and the intent is unclear, ask the user
what this phase should produce before writing.

## Finish

This applies to every file-writing phase (not `help`, `challenge`, `resolve`,
or `status`, which report their own way). After writing, report back concisely:
- the JOB and the PHASE used,
- whether this was a new draft or a refinement,
- the full path to the file (`$OUT`) and, for a shared file, the section written,
- a one-line summary of what was captured or changed this run.

Keep it short — no preamble.
