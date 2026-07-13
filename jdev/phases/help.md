# `help` — Teach the Workflow

Teach the user how to use this skill — don't write a file, don't set up a
folder, and skip the Finish report. The dispatch table in SKILL.md (already in
context) gives you every phase's one-liner — do **not** read the other
`phases/*.md` files just to explain them. Explain, concisely and in your own
words:

- **What `/jdev` is for** — driving one feature forward through a sequence of
  phases, each persisted to its own file under `.temp/<JOB>/`.
- **The argument shape** — `/jdev <phase> <job> [instructions]`: PHASE picks the
  behavior, JOB names the feature (stable across phases), INSTRUCTIONS steers the
  run and is optional.
- **The core lifecycle** — `define` → `research` → `plan` → `go`, each later
  phase reading the earlier ones; re-running a phase refines its file instead
  of overwriting; any unknown PHASE is accepted as a custom artifact.
- **The optional deepeners** — `design` (settle the API/UX surface) and `spike`
  (throwaway prototype for the riskiest unknown), both between `research` and
  `plan`.
- **The checkers** — `challenge` (a fresh-context subagent adversarially
  audits the planning artifacts and leaves `*** ... ***` markers; best before
  `go`) and `verify` (trace requirements to actual code and tests; after `go`).
- **The maintenance phases, runnable anytime** — `resolve` scans the feature's
  files for `*** ... ***` comments and edits the files to address them in
  place; `status` reports where the job stands and suggests the next move.
  Neither produces a file.
- **A worked example** — walk through a single JOB across the lifecycle, e.g.:
  - `/jdev define checkout "guest checkout for the cart"`
  - `/jdev research checkout`
  - `/jdev design checkout "REST endpoints + cart UI states"` *(optional)*
  - `/jdev plan checkout "keep the existing payment provider"`
  - `/jdev challenge checkout` *(fresh eyes audit the plan…)*
  - `/jdev resolve checkout` *(…and the findings get addressed)*
  - `/jdev go checkout`
  - `/jdev verify checkout`
- **How to refine** — re-run a phase with new INSTRUCTIONS to edit its file in
  place (e.g. `/jdev define checkout "add a risks section"`), or leave
  `*** ... ***` comments in a file and run `/jdev resolve <job>` to address
  them. `/jdev status <job>` re-orients you after time away.

End by inviting the user to start with `define`. Keep it skimmable — short
sections or a tight list, not a wall of text.
