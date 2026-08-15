# AI Tools Policy

The challenge explicitly invites AI assistance but grades **your judgment**, not typing speed. These
rules keep that judgment visible and honest.

## Rules

1. **Disclose everything.** In the README (Part E), list which AI tools you used and where.
2. **State accept vs reject.** For each non-trivial AI suggestion, note whether you accepted or
   rejected it and why. "Generated as-is" is not an acceptable answer for graded areas.
3. **You own correctness.** AI output on invariants (atomic publish, uniqueness, artwork specs) must
   be verified by tests and reasoning — these are exactly the areas scored hardest.
4. **No silent boilerplate dumps.** Code, schemas, and docs introduced from AI must be reviewed the
   same as human-authored code.
5. **Keep secrets out.** Never paste secrets into an AI tool.

## Suggested disclosure format (README, Part E.5)

> **AI tools used:** e.g. "ChatGPT (schema brainstorming), Copilot (boilerplate CRUD)."
> **Accepted:** suggested temp-file+rename atomic write; rejected its 'overwrite the file' first pass
> because it breaks the atomicity requirement.

## Why this matters

The grading rubric says "Using them is fine — we want your judgment." Transparent disclosure plus
explicit accept/reject decisions demonstrates that judgment rather than hiding it.
