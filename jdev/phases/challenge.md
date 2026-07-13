# `challenge` — Adversarial Audit of the Artifacts

Have fresh eyes try to tear the feature's planning artifacts apart before they
get acted on. Most valuable after `plan` exists and before `go`, but it audits
whatever is present.

1. **Determine scope.** The artifacts named in INSTRUCTIONS, or by default
   every planning-altitude file in the folder (`define`, `research`, `design`,
   `spike`, `plan`) — not `go`, `verify`, or a prior `challenge` report.
2. **Spawn a fresh subagent** with the Agent tool (`general-purpose`). Context
   isolation is the point: the auditor must form its own view of the codebase
   and the artifacts rather than inherit this conversation's assumptions — so
   do NOT use a fork, and don't paraphrase the artifacts into the prompt; give
   it the file paths and let it read them. If INSTRUCTIONS names a model (e.g.
   "in opus"), pass it as the Agent `model` override.
3. **Give the auditor a refutation brief**, not a summarization brief. It
   should hunt for: a requirement in `define` the `plan` doesn't satisfy; an
   assumption `research` asserts but never checked against the code; a cheaper
   approach the options analysis ignored; an edge case that breaks the
   `design`; contradictions between files; success metrics nothing measures.
   Have it return findings as a structured list — file, location (section or
   quoted phrase), severity (`blocker`/`major`/`minor`), and the finding
   itself — and instruct it to return "no findings" over manufactured nitpicks.
4. **Triage the findings yourself.** Drop anything the artifacts already
   address or that misreads them — you have the context the auditor lacks;
   spending a finding's credibility on a false positive is worse than dropping
   it.
5. **For each surviving finding, inject a comment marker into the implicated
   file** at the relevant spot, in the standard `resolve` format:
   `*** challenge: <the finding> ***`. This closes the loop — the user reviews
   the markers and runs `/jdev resolve <job>` to address them.
6. **Write `$OUT`** as the audit report: scope audited (and which model, if
   overridden), a findings table with severities and where each marker was
   placed, findings dropped in triage and why, and an overall verdict
   (sound / needs work before `go`).

**Re-runs override the usual EXISTS refinement rule**: a second
`/jdev challenge <job>` performs a *fresh audit round*, not an edit of the old
report. Read the prior report first, note which earlier findings are now
resolved, and update the report with the new round's results.
