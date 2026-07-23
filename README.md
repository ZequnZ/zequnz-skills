# zequnz-skills

My Agent skills collection — reusable, self-evolving procedural knowledge for Agent Hardness. 

Each skill is a workflow the agent follows, with pitfalls, worked examples, and reference docs captured from real usage and refined over time.


## What's in here

```
skills/
  <skill-name>/
    SKILL.md              # the workflow itself: triggers, steps, pitfalls
    LESSONS.md            # (optional) self-evolving log of generalized lessons
    references/           # deep-dive docs loaded only when relevant
    templates/            # copy-paste starting points for generated docs
```

### Available skills

| Skill | What it does |
|---|---|
| [`spicy-alignment`](skills/spicy-alignment/) | Structured, whiteboard-first alignment before action. Produces a dependency-ordered design tree the human answers inline in a markdown file. Domain-general core with software as an optional add-on. Self-evolves via `LESSONS.md`. |

More will be added as they're extracted from real work.



## Contributing

See [`CONTRIBUTING.md`](CONTRIBUTING.md) — covers the anonymization bar for anything derived from real work, the expected `SKILL.md` shape, and how to propose a new skill or a `LESSONS.md` addition.

## License

MIT — see [`LICENSE`](LICENSE).
