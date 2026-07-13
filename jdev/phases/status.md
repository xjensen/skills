# `status` — Where Does This Job Stand?

Orientation, not new work. This phase produces no file and skips the Finish
report — scan the feature folder and report back. If the folder is empty or
missing, say the job has no artifacts yet and suggest starting with `define`.

Gather:

- **Phase files present** — which `$JOB-<phase>.md` files exist, with one line
  each on what they currently hold (read them, or skim enough to summarize
  honestly).
- **Unresolved markers** — `grep -rnE '\*\*\* .+ \*\*\*' "$DIR"/*.md`, counted
  per file.
- **Open questions** — items still sitting in **Constraints & open questions**
  sections across the files.
- **Checker verdicts** — the latest `challenge` and `verify` verdicts, if
  those files exist.

Report it as a short dashboard: phases done, markers outstanding, open
questions, then a **suggested next move** — the next lifecycle phase not yet
run, or `resolve` if markers are outstanding, or `verify` if `go` happened but
was never checked. Keep the whole thing skimmable; this phase's job is to
re-orient the user in under a minute.
