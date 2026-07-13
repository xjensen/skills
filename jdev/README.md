# jdev

`jdev` is a hands-on, maximum-supervision skill for driving feature development. It covers all phases of a feature's lifecycle, from definition to completion. My goal is to keep this short, easy to remember, and flexible. 

## The gimmick

Each phase of a `jdev` run outputs a working file to your project. Instead of chatting with the agent to refine its output, instead you iterate with the agent to refine those working files.

## Usage

Here's a rough example of how to use the skill.

Start with the definition phase.

```
/jdev define login "I need to create a login feature for my website."
```

This will write a file to `.temp/login/login-define.md`. **Don't skip this file!** Go read it in full to ensure the AI understands what you want. 

At any point, you can drop comments into this working file. This is perhaps the defining feature of `jdev`. You work in these files instead of directly through chat. Comments are denoted by a triple asterix: `*** My comment here... ***`. For example:

```
# `.temp/login/login-define.md`

## Requirements

- Users can create an account and sign in with a username and password. *** Actually, I want to use a federated login system, like "Sign in with Google". ***
- Users stay signed in for 30 days across browser restarts.
- Users can reset a forgotten password from the sign-in page. *** Does this requirement still make sense if we go federated? ***

## Success metrics

- 80% of new visitors complete account creation without support contact. *** Let's make this 90% ***
```

When your comments are added, you can ask `jdev` to *resolve* the comments. The skill will scan for these comments and address them in place.

```
/jdev resolve login
```

> Make sure you **save** the working file with your comments before running resolve! I make this mistake often.

Subsequent phases read the results of previous phases. Use this comment-and-resolve pattern to walk through each phase of devolpment.

```
/jdev research login
/jdev plan login
/jdev go login
```

### Interface

```
/jdev <phase> <job> [instructions]
```

- **phase** — which step to run (see [Phases](#phases) below).
- **job** — a short, stable name for the feature (e.g. `checkout`, `billing-v2`). Use the same job name across every phase; it names the folder and the files.
- **instructions** — optional free-form guidance for this run: what to focus on, what to add, what to change.

Output lands in your project at:

```
.temp/<job>/<job>-<phase>.md
```

e.g. `/jdev plan checkout` writes `.temp/checkout/checkout-plan.md`.

> You'll probably want to add the `.temp/` folder to `.gitignore` when using this skill.

### Re-running a phase

Running the same phase again refines its existing file in place using your instructions, rather than starting over:

```
/jdev define checkout "add a risks section"
```

## The lifecycle

| Order | Phase | Role |
|---|---|---|
| 1 | `define` | Capture the product requirements |
| 2 | `research` *(optional)* | Investigate the problem space |
| 3 | `design` *(optional)* | Refine the feature's outside interfaces |
| 4 | `spike` *(optional)* | Create throwaway tests or prototypes |
| 5 | `plan` | Write the implementation plan |
| 6 | `go` | Carry out the plan |
| 7 | `verify` *(optional)* | Check the build |
| 8 | `summarize` *(optional)* | Write a PR/changelog-ready summary of the work |

Each phase reads the earlier phases' files before producing its own, so the work compounds: `research` builds on `define`, `plan` honors `design` contracts and `spike` verdicts, `verify` traces `define` and `plan` back to real code. `design` and `spike` are optional deepeners — skip them for small features.

Once `go` has run, `go` and `verify` alternate until the feature is actually done: `verify` records what's missing as a checklist of gaps, a `go` re-run closes them, and a `verify` re-run confirms and ticks them off. See [The go ⇄ verify loop](#the-go--verify-loop).

A few phases sit outside the lifecycle and can be run at any point:

| Phase | Role |
|---|---|
| `challenge` | Adversarially audit the artifacts with fresh eyes; leaves `*** ... ***` markers |
| `resolve` | Address the `*** ... ***` comments in the job's files, in place |
| `status` | Report where the job stands and suggest the next move |
| `help` | Explain the workflow |
| *anything else* | Write a custom artifact titled by the phase name, guided by your instructions |

## Phases

### `define` — product requirements

Captures the high-level requirements: the problem and who it's for, goals and non-goals, users and use cases, required capabilities, success metrics, and constraints/open questions. Stays at product altitude — no implementation detail. This is the foundational artifact every later phase builds on.

### `research` — investigate the problem space

Grounds the plan in evidence: prior art (in the codebase and elsewhere), relevant existing modules and seams with `file:line` citations, candidate approaches with trade-offs, a recommendation, and the risks and unknowns still to de-risk (which `spike` can pick up directly).

### `design` — the feature's outside surface *(optional)*

Settles what the feature looks like from the outside before sequencing the build: API contracts, data shapes, UX flows, error and edge states, and alternatives set aside. Use it when there's a user-facing or API surface worth locking down; `plan` treats its contracts as settled.

### `spike` — de-risk the riskiest unknown *(optional)*

Builds a deliberately throwaway prototype to answer **one** open question decisively — no tests, no polish, shortest path to signal. Spike code lives under `.temp/<job>/spike/` so it never mixes with real source. The record captures the question, what was built, the findings, a verdict for the plan, and confirmation the tree is clean. Re-run it for the next unknown.

### `plan` — implementation plan

Turns everything above into an ordered, reviewable sequence of changes: the approach, the steps, the files and components touched, data flow, edge cases and testing, and risks and rollout. This is the last stop before code — a good moment to run `challenge`.

### `go` — carry out the plan

Makes the actual code changes in your project, then writes an implementation record: what changed by file, deviations from the plan and why, what was run to verify, and follow-ups (deferred or newly discovered work).

Re-running `go` does **more implementation work**, recorded as an appended "Round N" section: it does what your instructions say, or — with no instructions — defaults to closing the open gaps in the verify report.

### `verify` — trace requirements to reality

Run after `go`. Checks that what was built satisfies what was asked — against the actual code and tests, never the go record's say-so. Produces:

- a **coverage matrix**: every `define` requirement, `plan` step, and `go` claim, each marked Verified / Partial / Missing / Unverifiable with evidence;
- a **Gaps checklist** with stable IDs (`GAP-1`, `GAP-2`, …) — the work ledger for remediation;
- **drift** (undocumented behavior the code has that the artifacts don't describe) and a **verdict**.

Re-running `verify` is a fresh re-check: it re-verifies every claim, ticks off gaps that are now closed (with evidence), and adds newly found ones.

### `summarize` — report the work

Run after `go`. Distills the implementation record into a summary you can paste into a pull request, changelog, or status report — written for readers outside this workflow, with all the skill's plumbing (phases, rounds, gap IDs, working files) omitted. It produces:

- a short **summary** of the work;
- the practical, **functional changes** to the application;
- a concise account of the **code changes**, with file references;
- any **concerns and follow-ups** a reviewer should know about (drawing on the verify report's open gaps, if one exists).

Re-running `summarize` refreshes it against the current state of the work — useful after more go rounds have landed.

### `challenge` — adversarial audit

Spawns a **fresh-context subagent** to try to tear the planning artifacts apart: requirements the plan doesn't satisfy, unchecked assumptions, cheaper approaches ignored, contradictions between files, metrics nothing measures. Surviving findings are injected into the implicated files as `*** challenge: ... ***` markers for `resolve` to address, and a report records the round. Most valuable between `plan` and `go`. Re-runs are new audit rounds.

You can pick the auditor's model in the instructions, e.g. `/jdev challenge checkout "in opus"`.

### `resolve` — address inline comments

Scans the job's files for `*** ... ***` markers — your own review notes or `challenge` findings — and edits the files in place to address each one, removing the marker once done. Anything it can't resolve confidently is left in place and reported back. Produces no file of its own.

To leave a note for it, drop a marker anywhere in a phase file:

```
*** is this still true after the API change? ***
```

### `status` — where does this job stand?

A read-only dashboard: which phase files exist and what they hold, unresolved markers per file, open questions still outstanding, the latest `challenge` and `verify` verdicts, and a suggested next move. Use it to re-orient after time away. Produces no file.

### `help` — teach the workflow

Explains the skill interactively. Takes no job.

### Anything else — custom artifact

An unrecognized phase name isn't an error. `/jdev adr checkout "record the payment-provider decision"` writes `.temp/checkout/checkout-adr.md` with your instructions defining the content.

## The go ⇄ verify loop

The build isn't done when `go` finishes — it's done when `verify` says so.

1. `/jdev go checkout` — implement the plan.
2. `/jdev verify checkout` — trace requirements to code; gaps come back as a checklist (`GAP-1`, `GAP-2`, …).
3. `/jdev go checkout` — with no instructions, a re-run defaults to closing the open gaps, logged as a new round in the go record.
4. `/jdev verify checkout` — re-check; gaps that are genuinely closed get ticked off with evidence.
5. Repeat 3–4 until the verdict is **ready**.

Gap state has exactly one writer: only `verify` may tick a checkbox, and only on its own evidence — `go` claiming something is fixed is never enough.

## Referencing another job

Instructions can pull in another job's artifacts with `jdev:<job>` or `jdev:<job>:<phase>`. The main use is seeding a new feature from a finished one's leftovers — `go` records a **Follow-ups** list, and:

```
/jdev define billing "jdev:checkout:go follow-ups — turn the deferred items into requirements"
```

starts a fresh `billing` job from `checkout`'s deferred work. (Gaps found by `verify` are *not* follow-ups — they're unfinished scope of the current job and stay in the go ⇄ verify loop.)

## A full example

```
/jdev define checkout "guest checkout for the cart"
/jdev research checkout
/jdev design checkout "REST endpoints + cart UI states"   # optional
/jdev spike checkout "can the session survive a payment redirect?"   # optional
/jdev plan checkout "keep the existing payment provider"
/jdev challenge checkout        # fresh eyes audit the plan, leave markers…
/jdev resolve checkout          # …and the findings get addressed
/jdev go checkout
/jdev verify checkout           # finds GAP-1, GAP-2
/jdev go checkout               # closes the gaps (Round 2)
/jdev verify checkout           # confirms — verdict: ready
/jdev summarize checkout        # PR-ready summary of the work
/jdev status checkout           # anytime: where does this stand?
```
