# jdev

`jdev` is a hands-on, maximum-supervision skill for AI-driven development. It covers all phases of a feature's lifecycle, from definition to completion. The goal is to bring some structure and refinement to vibe coding sessions.

## The gimmick

`jdev` maintains a small set of working files in your project. Instead of chatting with the agent to refine its output, instead you iterate with the agent to refine those working files.

## Usage

Here's a rough example of how to use the skill.

### 1. Define the problem

```
/jdev define login "I need to create a login feature for my website."
```

This will write a file to `.temp/login/login-definition.md`. **Don't skip this file!** Go read it in full to ensure the AI understands what you want. 

### 2. Drop comments into the file

At any point, you can drop comments into this working file. This is perhaps the defining feature of `jdev`. You work in these files instead of directly through chat. Comments are denoted by a triple asterix: `*** My comment here... ***`. For example:

```
# `.temp/login/login-definition.md`

## Requirements

- Users can create an account and sign in with a username and password. *** Actually, I want to use a federated login system, like "Sign in with Google". ***
- Users stay signed in for 30 days across browser restarts.
- Users can reset a forgotten password from the sign-in page. *** Does this requirement still make sense if we go federated? ***

## Success metrics

- 80% of new visitors complete account creation without support contact. *** Let's make this 90% ***
```

### 3. Resolve the comments

When your comments are added, you can ask `jdev` to *resolve* the comments. The skill will scan for these comments and address them in place.

```
/jdev resolve login
```

> Make sure you **save** the working file with your comments before running resolve! I make this mistake often.

### 4. Expand your understanding

Subsequent phases read the results of previous phases. If desired, use optional elaboration phases to dig deep into the problem space.

```
/jdev research login
/jdev design login
/jdev spike login "Let's build a prototype to verify our solution to password reset."
```

Continue to use the comment-and-resolve pattern throughout these phases.

### 5. Get ready to ship

When you're satisfied with the definition document, plan it and run it.

```
/jdev plan login
/jdev go login
```

## Interface

```
/jdev <phase> <job> [instructions]
```

- **phase** — which step to run (see [Phases](#phases) below).
- **job** — a short, stable name for the feature (e.g. `checkout`, `billing-v2`). Use the same job name across every phase; it names the folder and the files.
- **instructions** — optional free-form guidance for this run: what to focus on, what to add, what to change.

Output lands in your project under `.temp/<job>/`, consolidated into three working files:

| File | Written by | Holds |
|---|---|---|
| `<job>-definition.md` | `define`, `research`, `design`, `spike` | What the feature is: requirements, plus a section per deepener phase |
| `<job>-plan.md` | `plan` | How to build it |
| `<job>-done.md` | `go`, `verify` | What actually happened: implementation record, work summary (both from `go`), and verification report |

e.g. `/jdev research checkout` writes the `## Research` section of `.temp/checkout/checkout-definition.md`. Unrecognized phases get their own file (`<job>-<phase>.md`); `challenge`, `resolve`, `status`, and `help` write no file at all.

> You'll probably want to add the `.temp/` folder to `.gitignore` when using this skill.

### Re-running a phase

Running the same phase again refines its own file or section in place using your instructions, rather than starting over — other phases' sections are left untouched:

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
| 6 | `go` | Carry out the plan (and refresh the work summary) |
| 7 | `verify` *(optional)* | Check the build |

Each phase reads the earlier phases' output before producing its own, so the work compounds: `research` builds on `define`, `plan` honors `design` contracts and `spike` verdicts, `verify` traces the definition and plan back to real code. `design` and `spike` are optional deepeners — skip them for small features.

Once `go` has run, `go` and `verify` alternate until the feature is actually done: `verify` records what's missing as a checklist of gaps, a `go` re-run closes them, and a `verify` re-run confirms and ticks them off.

A few phases sit outside the lifecycle and can be run at any point:

| Phase | Role |
|---|---|
| `challenge` | Adversarially audit the artifacts with fresh eyes; leaves `*** ... ***` markers and reports in chat |
| `resolve` | Address the `*** ... ***` comments in the job's files, in place |
| `status` | Report where the job stands and suggest the next move |
| `help` | Explain the workflow |
| *anything else* | Write a custom artifact titled by the phase name, guided by your instructions |

## Referencing another job

Instructions can pull in another job's artifacts with `jdev:<job>`. The main use is seeding a new feature from a finished one's leftovers — `go` records a **Follow-ups** list in the done file. For example:

```
/jdev define billing "jdev:checkout follow-ups — turn the deferred items into requirements"
```

This would start a fresh `billing` job from `checkout`'s deferred work.

## Cheatsheet

```
# Define the job:
/jdev define checkout "Let's implement guest checkout for the shopping cart."

# Research the job (optional):
/jdev research checkout

# Design interfaces for the job (optional):
/jdev design checkout "We need a new API endpoint."

# Spike the risky bit with a throwaway prototype (optional):
/jdev spike checkout "Can the session survive a payment redirect?"

# Plan the job:
/jdev plan checkout

# Fresh eyes audit the plan, leaving comments (optional):
/jdev challenge checkout

# Address any comments (optional):
/jdev resolve checkout

# Implement:
/jdev go checkout

# Verify the completed work (optional):
/jdev verify checkout

# Close verify's gaps, if any (Round 2):
/jdev go checkout

# Verify again:
/jdev verify checkout

# Anytime: where does this stand?
/jdev status checkout
```
