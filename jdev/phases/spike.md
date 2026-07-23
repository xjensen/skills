# `spike` — De-risk the Riskiest Unknown

Build a deliberately throwaway prototype to answer the **single riskiest open
question** before committing to a plan. This phase owns the `## Spike` section
of the definition file — read the file's existing content first; the
`## Research` section's **Risks & unknowns** is the natural source of spike
questions. INSTRUCTIONS may name the question directly; otherwise pick the
unknown whose answer most changes the plan, and say which one you picked.

Rules of engagement:

- **One question per run.** A spike that answers three questions vaguely is
  worth less than one that answers one question decisively. Re-run the phase
  for the next unknown.
- **Prototype quality, on purpose.** No tests, no polish, no error handling
  beyond what the question needs — the shortest path to signal.
- **Keep it disposable.** Prefer putting spike code under `.temp/<JOB>/spike/`
  in the project so it never mixes with real source. If the question can only
  be answered by touching the project tree (e.g. wiring into an existing app),
  record exactly which files were touched so they can be reverted, and don't
  commit them.

Record each spike as its own subsection under `## Spike`, titled by its
question (e.g. `### Can the session survive a payment redirect?`):

- **Question** — the unknown this spike targets, and why it was the riskiest.
- **What was built** — what the prototype does, where it lives, how to run it.
- **Findings** — what running it demonstrated, with evidence (output, timings,
  errors).
- **Verdict** — the answer to the question and what it means for the plan
  (confirms the recommendation, forces a different option, etc.).
- **Disposal** — confirmation the tree is clean, or the exact paths still to
  remove/revert.

Re-runs targeting a **new** unknown append a new subsection; INSTRUCTIONS
naming an already-spiked question refine that question's subsection instead.
