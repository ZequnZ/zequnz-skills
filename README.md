# zequnz-skills

My Agent skills collection — reusable, self-evolving procedural knowledge
for AI coding agents (Claude Code, Hermes Agent, and any other
Anthropic-style "skills" runtime that reads a `SKILL.md`).

A skill here is not a prompt template. It's a workflow the agent follows —
with pitfalls, worked examples, and reference docs — captured from real
usage and refined over time. Some skills also maintain their own
`LESSONS.md`: a log the agent appends to after using the skill, so the
skill gets better the more it's used, across whatever project installs it.

## What's in here

```
skills/
  <skill-name>/
    SKILL.md              # the workflow itself: triggers, steps, pitfalls
    LESSONS.md             # (optional) self-evolving log of generalized lessons
    references/            # deep-dive docs loaded only when relevant
    templates/              # copy-paste starting points for generated docs
```

### Available skills

| Skill | What it does |
|---|---|
| [`design-tree-interview`](skills/design-tree-interview/) | Structured, whiteboard-first design alignment before writing code. Produces a dependency-ordered design tree the human answers inline in a markdown file, then a separate implementation plan once the design is stable. Self-evolves via `LESSONS.md`. |

More will be added as they're extracted from real work.

## Using a skill

These are plain markdown + optional scripts/templates — no special install
step is required to *read* them. How you wire them into an agent depends on
the runtime:

- **Claude Code / Claude.ai:** drop the skill directory under your project's
  `.claude/skills/` (or wherever your Claude Code skills path points), or
  package it with the skill-creator tooling if you're using Claude's skill
  packaging format.
- **Hermes Agent:** copy the skill directory into your profile's
  `skills/` directory (or a category subdirectory under it) — see the
  [Hermes Agent docs](https://hermes-agent.nousresearch.com/docs) for the
  exact path and any manifest requirements.
- **Anything else that reads `SKILL.md` + YAML frontmatter:** the format
  here follows the common convention — `name`, `description` frontmatter,
  markdown body, optional `references/`, `templates/`, `scripts/`
  subdirectories loaded on demand. It should drop in with minimal
  adaptation.

Each skill's own `SKILL.md` states its trigger conditions in the
`description` frontmatter field — that's what an agent uses to decide when
to load it.

## Philosophy

1. **Trigger-first descriptions.** The `description` field is the primary
   mechanism agents use to decide whether a skill applies — it states both
   what the skill does and when to reach for it.
2. **Progressive disclosure.** `SKILL.md` stays lean; deep detail lives in
   `references/` and is loaded only when the workflow actually needs it.
3. **Explain the why, not just the what.** Skills here favor explaining the
   reasoning behind a step over issuing bare imperative rules — a
   capable agent given good reasoning generalizes better than one given a
   rigid checklist.
4. **No proprietary specifics.** Nothing in this repo should reference a
   specific company, client, ticket ID, internal schema, or customer detail.
   Lessons and examples are generalized before being committed here — see
   `CONTRIBUTING.md`.
5. **Self-evolving where it matters.** Skills that benefit from accumulated
   experience (design discussions, debugging methodologies, review
   processes) carry a `LESSONS.md` the agent is instructed to update after
   real usage, so the skill compounds instead of staying static.

## Contributing

See [`CONTRIBUTING.md`](CONTRIBUTING.md) — covers the anonymization bar for
anything derived from real work, the expected `SKILL.md` shape, and how to
propose a new skill or a `LESSONS.md` addition.

## License

MIT — see [`LICENSE`](LICENSE).
