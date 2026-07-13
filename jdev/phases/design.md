# `design` — Shape the Feature's Surface

Design what the feature looks like *from the outside* before planning how to
build it: interfaces, contracts, and interaction shapes. Read the `define` and
`research` outputs first if they exist. This phase sits between `research` and
`plan` — use it when the feature has a user-facing or API surface worth
settling before implementation sequencing. Produce (as applicable — skip
sections the feature doesn't have):

- **Surface overview** — the feature's externally visible shape in a few
  sentences.
- **API contracts** — endpoints or function signatures, request/response
  shapes, status/error responses.
- **Data shapes** — new or changed models and schemas, with their fields.
- **UX flows** — screens, states, and transitions, as prose or ASCII sketches.
- **Error & edge states** — what the user or caller sees when things go wrong.
- **Alternatives set aside** — surface-level options considered and why they
  lost.
- **Constraints & open questions** — known limits, dependencies, unknowns.

Stay at the interface altitude — what the surface *is*, not how it's built.
Implementation sequencing belongs to `plan`, which reads this file.
