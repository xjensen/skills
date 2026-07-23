# `research` — Investigate the Problem Space

Investigate the problem space so the plan rests on evidence, not assumptions.
This phase owns the `## Research` section of the definition file — read the
file's existing content first (especially the `define` base, if present).
Produce, as subsections under `## Research`:

- **Prior art** — how this is solved today (in this codebase and elsewhere).
- **Codebase findings** — relevant existing modules, patterns, and seams to
  reuse or extend (cite `file_path:line`).
- **Options** — candidate approaches with trade-offs.
- **Recommendation** — the approach to carry into planning, and why.
- **Risks & unknowns** — what still needs to be de-risked. A `spike` run can
  pick the riskiest of these up directly.

Newly surfaced constraints and open questions go in the file's shared
**Constraints & open questions** tail section, not inside `## Research`.
