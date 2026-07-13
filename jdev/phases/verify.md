# `verify` — Trace Requirements to Reality

Check that what was built actually satisfies what was asked. Run after `go`.
Read the `define`, `plan`, and `go` outputs first — if no `go` record exists,
tell the user there's nothing to verify yet and stop.

1. **Extract the checkable claims**: each requirement from `define`, each step
   from `plan`, and each "what changed" / verification claim from `go`. If
   INSTRUCTIONS narrows the scope (e.g. "just the API requirements"), honor it.
2. **Verify each claim against the actual project**, not against the
   documents' say-so: read the implicated code and confirm the behavior is
   really implemented; find the tests that cover it; where a cheap check
   exists (build, test suite, lint), re-run it rather than trusting the `go`
   record's report.
3. **Assign each claim a status** — `Verified`, `Partial`, `Missing`, or
   `Unverifiable` — always with evidence: `file_path:line`, a test name, or
   command output. Never mark `Verified` on the strength of the `go` record
   alone.

Write `$OUT` as a **verification report**:

- **Coverage matrix** — one row per requirement/step: status + evidence.
- **Gaps** — a checklist, one item per `Partial`/`Missing` claim, each with a
  stable ID and what it would take to close it:

  ```markdown
  - [ ] **GAP-1** — <the claim that fell short>: <what closing it takes>
  ```

  This checklist is the work ledger for the go↔verify loop: a re-run of `go`
  picks up the unchecked items (see `go`'s re-run rules), and only a re-run of
  `verify` may check one off.
- **Drift** — places the code does something the artifacts don't describe
  (undocumented behavior is a finding too).
- **Verdict** — ready, or gaps to close first.

To kick off remediation: `/jdev go <job>` — with empty INSTRUCTIONS, a `go`
re-run defaults to closing the open gaps.

## Re-runs are fresh re-checks

A re-run of `verify` is not a prose refinement — it's a new verification round
(like `challenge`'s audit rounds). Re-verify every claim against the current
code, then update `$OUT` in place:

- Update each coverage-matrix row's status and evidence.
- Re-check each existing gap. If it's now closed, tick its checkbox and append
  the evidence (`file_path:line`, test name, or command output); if still open,
  leave it unchecked. Keep IDs stable — never renumber or delete entries. Add
  newly found gaps with fresh IDs continuing the sequence.
- A gap changes state only on this run's own evidence — never because the `go`
  record says it was addressed.
- Refresh the verdict.
