# Contributing

Thanks for considering a contribution. This repo holds AI agent skills
meant to be installed on other people's codebases, so the bar here is
slightly different from a typical project's CONTRIBUTING.md.

## The one hard rule: no proprietary specifics

Every skill in this repo is derived from real usage, and real usage happens
on real (often employer/client) codebases. Before anything gets committed:

- **No company, client, or product names.** If a lesson came from working
  on "Acme Corp's invoicing system," it gets rephrased as "a
  producer/consumer pair for a billing-adjacent record."
- **No ticket/issue IDs.** `FSDEV-292` becomes "a mid-sized feature branch."
- **No internal schema, field, table, or endpoint names** that identify a
  specific system. Generalize field names to their *role* (e.g. "a
  human-facing identifier field with a fallback chain" rather than the
  actual field name).
- **No customer or business-relationship data.**

The test: could this sentence be read by someone at the original company
without them recognizing their own system? If not, generalize further.

This isn't just a legal/ethics concern — over-specific lessons also don't
generalize well to the *next* project that installs the skill, which
defeats the point of a shared skill repo.

## Adding a new skill

1. Create `skills/<skill-name>/SKILL.md` with YAML frontmatter:
   ```yaml
   ---
   name: skill-name
   description: >
     What this does AND when to use it — this is the primary trigger
     mechanism, be specific about the phrases/contexts that should
     activate it.
   ---
   ```
2. Keep the body under roughly 500 lines. If you're approaching that,
   split detail into `references/*.md` and point to them from the body
   with guidance on when to read each one.
3. Prefer explaining *why* a step matters over issuing a bare imperative —
   agents generalize better from reasoning than from rigid rules.
4. Include a **Pitfalls** section — concrete failure modes seen in
   practice, not hypothetical ones.
5. If the skill benefits from accumulated experience across many uses
   (design discussion, debugging, review processes), add a `LESSONS.md`
   following the format already established in
   `skills/design-tree-interview/LESSONS.md`, and add a step in the
   `SKILL.md` body instructing the agent to read it before starting and
   append to it after finishing (generalized, anonymized — see above).

## Updating an existing skill

- If you found the skill wrong, incomplete, or stale while using it on a
  real task, patch it — don't wait for a dedicated session. Small, precise
  edits (a corrected pitfall, an added answer-shape example) are the most
  valuable kind of contribution here.
- If you're adding a `LESSONS.md` entry, keep it to a few lines: context +
  generalized lesson. Don't let it balloon into a full case study — that
  belongs in a `references/` doc if it's substantial enough to need one.

## Style

- Markdown, no exotic extensions assumed (should render fine on GitHub).
- Use tables for decision matrices / comparison points — they're easier to
  scan than prose when weighing options.
- Worked examples are welcome and encouraged, as long as they're
  anonymized per the rule above. A concrete (but generalized) example
  teaches an agent more than an abstract rule.

## Questions / proposals

Open an issue describing the workflow you think deserves a skill, with a
rough sketch of the trigger conditions and steps. Happy to discuss shape
before a full PR.
