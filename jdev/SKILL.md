---
name: jdev
version: 0.5.0
description: |
  Drive a feature through its lifecycle. Takes a feature JOB and a PHASE
  (define, research, design, spike, plan, go, verify, summarize, challenge,
  resolve, status, or anything else), creates a working folder at .temp/<JOB>/
  in the current project, and writes the phase's output to
  .temp/<JOB>/<JOB>-<PHASE>.md.
  Use when asked to "define", "research", "design", "spike", "plan", or "go"
  on a feature, to "draft requirements", "scope a new feature", "write a
  product proposal", to "challenge" or "verify" a feature's artifacts, to
  "summarize" the work for a pull request or changelog, to check a feature's
  "status", or to "resolve" review comments left in a feature's output files.
argument-hint: <phase> <job> [instructions]
---

# /jdev — Drive a Feature Through Its Lifecycle

Work a feature forward one phase at a time — define it, research it, plan it,
build it, check it — and persist each phase's output to a per-feature working
file.

## Arguments

The user invoked: `/jdev $ARGUMENTS`

There are three arguments:

- **PHASE** (first token) — which phase to perform this run. Selects a row in
  the [dispatch table](#what-to-produce). Known PHASEs: `define`, `research`,
  `design`, `spike`, `plan`, `go`, `verify`, `summarize`, `challenge`,
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
   offering `define`, `research`, `plan`, and `go` as options. Do not
   guess one.
5. **If PHASE is `help`, stop parsing here** — ignore JOB and INSTRUCTIONS, skip
   the folder setup entirely, read `phases/help.md`, and follow it. It produces
   no file.
6. If no JOB was provided, ask the user for it with AskUserQuestion (text
   input). Do not guess one. INSTRUCTIONS may legitimately be empty — don't
   prompt for it.

## Set up the working folder

Create the feature folder at the **project root** and confirm the output path.
Run from the repo so `.temp/` resolves against the current project:

```bash
JOB="<normalized-job>"
PHASE="<normalized-phase>"
ROOT="$(git rev-parse --show-toplevel 2>/dev/null || pwd)"
DIR="$ROOT/.temp/$JOB"
OUT="$DIR/$JOB-$PHASE.md"
mkdir -p "$DIR"
[ -f "$OUT" ] && echo "EXISTS: $OUT" || echo "NEW: $OUT"
```

The skill is **re-entrant** — calling `/jdev <same-phase> <same-job> ...` again
refines that phase's existing output instead of starting over. Branch on whether
`$OUT` exists:

- **NEW** (file does not exist) — this is the first run of this PHASE for this JOB.
  Generate the output from scratch and `Write` it to `$OUT`.
- **EXISTS** (file is already there) — this is a refinement run. First make sure
  the current contents are in context: if you have not already `Read` `$OUT`
  during this conversation, read it now. Then apply this run's INSTRUCTIONS by
  `Edit`ing the file in place — preserve everything not implicated by the
  instructions; don't regenerate or reorder untouched sections.

> **`resolve` and `status` are exceptions.** Like `help`, they produce no
> output file. They still need `$DIR` so they can scan the folder, but they
> ignore `$OUT` and the NEW/EXISTS branch above — set up the folder, then go
> straight to their phase spec. (`challenge`, `go`, `verify`, and `summarize`
> also partly deviate: `challenge` re-runs are fresh audit rounds, `go` re-runs
> are remediation rounds that do new implementation work, `verify` re-runs are
> fresh re-checks of every claim, and `summarize` re-runs refresh the summary
> against the current `go` record — each spec explains.)

Earlier phases feed later ones. Before any later phase, check the feature
folder for outputs from prior phases (e.g. `.temp/<JOB>/<JOB>-define.md`) and
read any that exist so this phase builds on them rather than starting blind —
each phase spec names its expected inputs.

## Referencing another job

INSTRUCTIONS may point at a *different* job with the shorthand `jdev:<job>`
(e.g. `jdev:checkout`), or at one of its specific artifacts with
`jdev:<job>:<phase>` (e.g. `jdev:checkout:go`). The referenced job is a separate
feature with its own folder — this is how one job's output seeds another.
Resolve every such reference before producing this run's output:

1. **Locate** the referenced folder at `$ROOT/.temp/<job>/`. If it doesn't
   exist, don't invent its contents — tell the user the reference didn't resolve
   and ask how to proceed.
2. **Read** the referenced artifact(s): the named phase file if `:<phase>` was
   given (`.temp/<job>/<job>-<phase>.md`), otherwise the artifacts in that job's
   folder relevant to this run.
3. **Pull** the relevant material in as seed guidance for *this* run, then follow
   the rest of INSTRUCTIONS for what to focus on. You're starting a new artifact
   in the current JOB, not editing the referenced one — never write into the
   referenced job's folder.

The motivating case is kick-starting a job from another's leftovers. The `go`
phase records a **Follow-ups** list; pointing a new job's `define` at it carries
those forward:

```
/jdev define billing "jdev:checkout:go follow-ups — turn the deferred items into requirements"
```

This reads `checkout`'s implementation record, lifts its Follow-ups, and seeds a
fresh `billing` job from them.

## What to produce

`INSTRUCTIONS` (the third argument) steers every phase:

- On a **NEW** output, treat INSTRUCTIONS as seed guidance for what to draft. If
  empty, fall back to the phase's default structure.
- On an **EXISTS** refinement, treat INSTRUCTIONS as the change request for this
  edit (e.g. "add a risks section", "tighten the success metrics"). If empty,
  ask the user what they want changed rather than rewriting blindly.

When a refinement resolves an item from a **Constraints & open questions**
section, don't just delete it. Move it into a **Decisions** section in the same
file — create that section (placed just before **Constraints & open questions**)
if it doesn't exist yet — and record it as the decision reached, not the
question asked: state what was settled and, briefly, why. Leave still-open items
where they are. This keeps the resolution history visible instead of erasing it.

Then branch on `PHASE`. Each known phase's full spec lives in its own file
under `phases/` **in this skill's directory** (the directory containing this
SKILL.md — not the project). **Read `phases/<PHASE>.md` now, before producing
anything, and follow it.** The one-liners below exist for dispatch and for
`help` — they are not the spec:

| PHASE | Produces | One-liner |
|---|---|---|
| `define` | `$JOB-define.md` | High-level product requirements: problem, goals, users, success metrics. |
| `research` | `$JOB-research.md` | Investigate the problem space: prior art, codebase findings, options, recommendation. |
| `design` | `$JOB-design.md` | The feature's outside surface: API contracts, data shapes, UX flows. |
| `spike` | `$JOB-spike.md` + throwaway code | Disposable prototype answering the single riskiest unknown. |
| `plan` | `$JOB-plan.md` | Implementation plan: approach, ordered steps, files, testing, rollout. |
| `go` | `$JOB-go.md` + code changes | Carry out the plan in the project; record what changed. |
| `verify` | `$JOB-verify.md` | Trace define/plan requirements to actual code and tests; coverage matrix. |
| `summarize` | `$JOB-summarize.md` | Distill the `go` record into a PR/changelog-ready work summary — no workflow plumbing. |
| `challenge` | `$JOB-challenge.md` + markers | Fresh-context subagent adversarially audits the artifacts; leaves `*** ... ***` markers for `resolve`. |
| `resolve` | edits files in place | Address `*** ... ***` comments across the folder's files. |
| `status` | report only | Dashboard: phases done, markers outstanding, open questions, next move. |
| `help` | report only | Explain the workflow. |

The nominal lifecycle order is `define` → `research` → `design` → `spike` →
`plan` → `go` → `verify`, with `design` and `spike` optional deepeners. When
`verify` finds gaps, `go` and `verify` alternate — `go` re-runs close the open
gaps, `verify` re-runs confirm and tick them off — until the verdict is ready.
`summarize` runs any time after `go` to turn the implementation record into an
external-facing report of the work. `challenge`, `resolve`, and `status` are
maintenance phases runnable at any point — `challenge` is most valuable
between `plan` and `go`.

### default (any other PHASE)

The PHASE is not in the table, so there is no `phases/` file to read. Don't
reject it — treat the PHASE token as the **title of the artifact** and let
INSTRUCTIONS define what to produce. Write a focused, well-structured markdown
document for that phase to `$OUT`, using INSTRUCTIONS as the primary guidance.
If INSTRUCTIONS is empty and the intent is unclear, ask the user what this
phase should produce before writing.

## Finish

This applies to every file-producing phase (not `help`, `resolve`, or `status`,
which report their own way). After writing the file, report back concisely:
- the JOB and the PHASE used,
- whether this was a new draft or a refinement,
- the full path to the file (`$OUT`),
- a one-line summary of what was captured or changed this run.

Keep it short — no preamble.
