# `design` — Shape the Feature's Surface

Design what the feature looks like *from the outside* before planning how to
build it: interfaces, contracts, and interaction shapes. This phase owns the
`## Design` section of the definition file — read the file's existing content
first (the `define` base and `## Research`, if present). This phase sits
between `research` and `plan` — use it when the feature has a user-facing or
API surface worth settling before implementation sequencing. Produce, as
subsections under `## Design` (as applicable — skip subsections the feature
doesn't have):

- **Surface overview** — the feature's externally visible shape in a few
  sentences.
- **API contracts** — endpoints or function signatures, request/response
  shapes, status/error responses.
- **Data shapes** — new or changed models and schemas, with their fields.
- **UX flows** — screens, states, and transitions, as prose or ASCII sketches.
- **Error & edge states** — what the user or caller sees when things go wrong.
- **Alternatives set aside** — surface-level options considered and why they
  lost.

Newly surfaced constraints and open questions go in the file's shared
**Constraints & open questions** tail section, not inside `## Design`.

Stay at the interface altitude — what the surface *is*, not how it's built.
Implementation sequencing belongs to `plan`, which reads this file.
