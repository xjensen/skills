# `go` — Carry Out the Plan

Carry out the work described by the plan (`$JOB-plan.md` — read it first if it
exists). Make the actual code changes in the project, then record them in the
`## Implementation` section of the done file (`$OUT`), which this phase owns.
Each run is a round — write a `### Round N — <one-line purpose>` subsection
(`### Round 1 — initial implementation` on the first run) containing:

- **What changed** — the edits made, by file (cite `file_path`).
- **Deviations** — where reality differed from the plan, and why.
- **Verification** — tests/builds run and their results.
- **Follow-ups** — anything deferred or newly discovered. Another job can pick
  these up as a starting point via `jdev:<this-job>` — see
  "Referencing another job" in SKILL.md.

Honor any active safety or scope restrictions before editing files.

## Then refresh the summary

`go` also owns the done file's `## Summary` section — there is no separate
summarize phase. After recording the round, write or rewrite `## Summary` so it
reflects the **full current state** of the work, not just this round. Place it
last, after `## Implementation` and any `## Verification` section.

**The audience is outside this workflow.** The summary describes the work
itself, never the process that produced it: no mention of jdev, phases,
rounds, GAP IDs, working files, `.temp/` paths, or any other skill plumbing.
Fold every round into one coherent account of the finished work — a reader
should not be able to tell how many passes it took. Write the section so its
body can be copied out whole and pasted into a pull request or changelog as-is.
If a `## Verification` section exists, let its open gaps and drift findings
feed the concerns list (described plainly, not by ID).

Write `## Summary` with:

- An opening **summary** — a short description of the work: what was built or
  changed and why. A paragraph or two directly under the heading, ready to
  paste as a PR description opener.
- **Functional changes** — a short list of the practical changes to the
  application: new or changed behavior, user-visible effects.
- **Code changes** — a concise description of the changes to the codebase,
  referencing the files touched (cite `file_path`), grouped sensibly rather
  than as a raw file dump.
- **Concerns & follow-ups** — anything a reviewer or maintainer should know:
  deferred work, known limitations, open questions, unresolved verification
  gaps. If there are none, say so.

Keep the summary tight — it's the artifact someone pastes elsewhere, not
another planning document.

## Re-runs: remediation, not refinement

A re-run of `go` means more implementation work, not editing the record's
prose. Decide what this run builds, in order:

1. If INSTRUCTIONS names the work, do that.
2. Otherwise, if the done file's `## Verification` section exists with
   unchecked items in its **Gaps** checklist, close those gaps — this is the
   default remediation loop.
3. Otherwise, ask the user what this run should build.

Then record the work by **appending a new round** to `## Implementation` —
never rewrite earlier rounds — with the same What changed / Deviations /
Verification / Follow-ups structure, plus a **Gaps addressed** list of the GAP
IDs worked on when the run was driven by the verification report. Do **not**
tick checkboxes in `## Verification`, even though it sits in the same file —
only a `verify` re-run may do that, after independently confirming the fix.

`## Summary` is the exception to the append-only rule: **rewrite it in full**
each round so it always reflects the complete, current work — fold the new
round in seamlessly, leaving no trace of how many rounds it took.

After a remediation round, point the user at `/jdev verify <job>` to re-check.
