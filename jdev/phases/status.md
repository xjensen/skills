# `status` — Where Does This Job Stand?

Orientation, not new work. This phase produces no file and skips the Finish
report — scan the feature folder and report back. If the folder is empty or
missing, say the job has no artifacts yet and suggest starting with `define`.

Gather:

- **Working files present** — which of `$JOB-definition.md`, `$JOB-plan.md`,
  and `$JOB-done.md` exist (plus any custom artifacts), and which phase
  sections each holds, with one line each on what they currently contain
  (read them, or skim enough to summarize honestly). The sections tell you
  which phases have run: `## Research`/`## Design`/`## Spike` in the
  definition file; `## Implementation` and `## Verification` in the done file
  (`## Summary` rides along with `## Implementation` — `go` always writes it).
- **Unresolved markers** — `grep -rnE '\*\*\* .+ \*\*\*' "$DIR"/*.md`, counted
  per file (`challenge:`-prefixed markers are outstanding audit findings).
- **Open questions** — items still sitting in **Constraints & open questions**
  sections across the files.
- **Verify verdict** — the latest verdict in the done file's `## Verification`
  section, if present, and any unchecked gaps.

Report it as a short dashboard: phases done, markers outstanding, open
questions, then a **suggested next move** — the next lifecycle phase not yet
run, or `resolve` if markers are outstanding, or `verify` if `go` happened but
was never checked. Keep the whole thing skimmable; this phase's job is to
re-orient the user in under a minute.
