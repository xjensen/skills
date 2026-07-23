# `define` — Capture the Product Requirements

Capture the **high-level product requirements** for the feature. This is the
foundational content other phases build on. `define` owns the **base** of the
definition file: a title line for the feature, then, based on instructions:

- **Problem** — what user/business problem this solves, and for whom.
- **Goals & non-goals** — what success looks like; what's explicitly out of scope.
- **Users & use cases** — who uses it and the key scenarios.
- **Requirements** — the high-level capabilities the feature must provide.
- **Success metrics** — how we'll know it worked.

Close the file with the shared tail section:

- **Constraints & open questions** — known limits, dependencies, unknowns.
  This section (and **Decisions**, created on demand just before it) is shared
  with `research`, `design`, and `spike` — they add their open items here too,
  and it always stays at the end of the file.

Stay at the product/requirements altitude — no implementation detail yet. On
refinement runs, leave the `## Research`, `## Design`, and `## Spike` sections
alone — they belong to their phases.
