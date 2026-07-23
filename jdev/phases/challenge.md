# `challenge` — Adversarial Audit of the Artifacts

Have fresh eyes try to tear the feature's planning artifacts apart before they
get acted on. Most valuable after the plan exists and before `go`, but it
audits whatever is present. This phase produces no file of its own — its
outputs are `*** ... ***` markers injected into the working files and an audit
report delivered in chat.

1. **Determine scope.** The files named in INSTRUCTIONS, or by default the
   planning-altitude files: the definition file and the plan file — not the
   done file.
2. **Spawn a fresh subagent** with the Agent tool (`general-purpose`). Context
   isolation is the point: the auditor must form its own view of the codebase
   and the artifacts rather than inherit this conversation's assumptions — so
   do NOT use a fork, and don't paraphrase the artifacts into the prompt; give
   it the file paths and let it read them. If INSTRUCTIONS names a model (e.g.
   "in opus"), pass it as the Agent `model` override.
3. **Give the auditor a refutation brief**, not a summarization brief. It
   should hunt for: a requirement in the definition the plan doesn't satisfy;
   an assumption `## Research` asserts but never checked against the code; a
   cheaper approach the options analysis ignored; an edge case that breaks the
   `## Design` contracts; contradictions between or within the files; success
   metrics nothing measures. Have it return findings as a structured list —
   file, location (section or quoted phrase), severity
   (`blocker`/`major`/`minor`), and the finding itself — and instruct it to
   return "no findings" over manufactured nitpicks.
4. **Triage the findings yourself.** Drop anything the artifacts already
   address or that misreads them — you have the context the auditor lacks;
   spending a finding's credibility on a false positive is worse than dropping
   it. Also drop duplicates of unresolved `*** challenge: ... ***` markers
   already sitting in the files from a prior round.
5. **For each surviving finding, inject a comment marker into the implicated
   file** at the relevant spot, in the standard `resolve` format:
   `*** challenge: <the finding> ***`. This closes the loop — the user reviews
   the markers and runs `/jdev resolve <job>` to address them.
6. **Report the audit in chat** (this phase skips the Finish report): scope
   audited (and which model, if overridden), a findings table with severities
   and where each marker was placed, findings dropped in triage and why, and
   an overall verdict (sound / needs work before `go`).

Every run is a **fresh audit round** — there is no report file to refine.
Markers still in the files from earlier rounds show what remains unresolved;
note them in the report, but don't re-inject them.
