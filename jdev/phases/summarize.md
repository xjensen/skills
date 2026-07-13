# `summarize` — Report the Work

Distill the `go` record into an external-facing summary of the work, suitable
for a pull request description, a changelog entry, or a status report. Read the
`go` output first — if no `go` record exists, tell the user there's nothing to
summarize yet and stop. If a `verify` report exists, read it too: its open gaps
and drift findings feed the concerns list.

**The audience is outside this workflow.** The summary describes the work
itself, never the process that produced it: no mention of jdev, phases, rounds,
GAP IDs, working files, `.temp/` paths, or any other skill plumbing. If the
`go` record spans multiple rounds, fold them into one coherent account of the
finished work — a reader should not be able to tell how many passes it took.

Write `$OUT` with four sections:

- **Summary** — a short description of the work: what was built or changed and
  why. A paragraph or two, ready to paste as a PR description opener.
- **Functional changes** — a short list of the practical changes to the
  application: new or changed behavior, user-visible effects.
- **Code changes** — a concise description of the changes to the codebase,
  referencing the files touched (cite `file_path`), grouped sensibly rather
  than as a raw file dump.
- **Concerns & follow-ups** — anything a reviewer or maintainer should know:
  deferred work, known limitations, open questions, unresolved verification
  gaps (described plainly, not by ID). If there are none, say so.

Keep the whole thing tight — this is the artifact someone pastes elsewhere,
not another planning document.

## Re-runs re-summarize

A re-run of `summarize` (the EXISTS branch) refreshes the summary against the
*current* `go` record (and `verify` report) — new rounds may have landed since
the last summary. Rewrite `$OUT` to reflect the full current state of the work,
folding any new work in seamlessly, and treat INSTRUCTIONS as the change
request for this pass (tone, length, emphasis, audience).
