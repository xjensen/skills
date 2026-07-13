# `resolve` — Address Inline Comments

Scan the feature folder's markdown files for inline comments and address them
**in place** — this phase produces no `$JOB-resolve.md`; it edits the existing
files (`$JOB-define.md`, `$JOB-plan.md`, etc.).

A comment is any text wrapped in triple asterisks with surrounding spaces — e.g.
`*** tighten this section ***` or a standalone `*** is this still true? ***`
line. The spaces matter: they keep the marker from rendering as markdown
bold-italic and make it greppable. Markers come from the user's own review
notes or from a `challenge` run (prefixed `challenge:`) — treat both the same
way. Find every comment across the folder:

```bash
grep -rnE '\*\*\* .+ \*\*\*' "$DIR"/*.md
```

If INSTRUCTIONS names a subset (e.g. "only the plan file", "skip the open
questions"), narrow the scan accordingly. If no comments are found anywhere,
say so and stop — don't touch the files.

For each comment found:

1. **Read the file** if it isn't already in context, so you understand the
   surrounding text the comment refers to.
2. **Interpret the comment** as an instruction or question about the nearby
   content — e.g. "expand this", "wrong, we dropped X", "add a risks section",
   "why?".
3. **Apply the change** by `Edit`ing the file to satisfy the comment, keeping
   the document's voice and structure intact and touching only what the comment
   implicates. If the comment settles an item in **Constraints & open
   questions**, move it into the file's **Decisions** section as described in
   SKILL.md's "What to produce" rather than deleting it.
4. **Remove the marker** once addressed, so a later run doesn't re-process it.
5. **If you can't resolve it confidently** — it's ambiguous, asks for a decision
   only the user can make, or needs information you don't have — leave the
   marker in place and collect it to raise in your report rather than guessing.

This phase has its own wrap-up (it skips Finish): report the files touched,
how many comments you addressed, and any markers you left unresolved and why.
