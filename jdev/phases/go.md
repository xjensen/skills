# `go` — Carry Out the Plan

Carry out the work described by the `plan` (read it first if it exists). Make
the actual code changes in the project, then write `$OUT` as an
**implementation record**:

- **What changed** — the edits made, by file (cite `file_path`).
- **Deviations** — where reality differed from the plan, and why.
- **Verification** — tests/builds run and their results.
- **Follow-ups** — anything deferred or newly discovered. Another job can pick
  these up as a starting point via `jdev:<this-job>:go` — see
  "Referencing another job" in SKILL.md.

Honor any active safety or scope restrictions before editing files.

## Re-runs: remediation, not refinement

A re-run of `go` (the EXISTS branch) means more implementation work, not
editing the record's prose. Decide what this run builds, in order:

1. If INSTRUCTIONS names the work, do that.
2. Otherwise, if `.temp/<JOB>/<JOB>-verify.md` exists with unchecked items in
   its **Gaps** checklist, close those gaps — this is the default remediation
   loop.
3. Otherwise, ask the user what this run should build.

Then record the work by **appending a new round** to `$OUT` — retitle the
original body `## Round 1` if it isn't already, and never rewrite earlier
rounds:

```markdown
## Round 2 — <one-line purpose, e.g. "close verify gaps">
```

with the same What changed / Deviations / Verification / Follow-ups structure,
plus a **Gaps addressed** list of the GAP IDs worked on when the run was driven
by a verify report. Do **not** tick checkboxes in the verify report — only a
`verify` re-run may do that, after independently confirming the fix. After a
remediation round, point the user at `/jdev verify <job>` to re-check.
