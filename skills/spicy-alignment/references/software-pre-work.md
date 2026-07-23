# Software Pre-Work

Domain-specific preparation for using the spicy-alignment interview method on **software design decisions** — architecture, schema, API contracts, generalizing existing mechanisms, etc.
Do this before writing the interview doc.
The core method in `SKILL.md` is domain-general; this reference adds the software-specific research steps.

## 1. Read the relevant code paths

Before writing a single question, read the actual code involved.
Every question in the interview doc should be a genuine unknown — something you could not resolve by reading the codebase.
If you find yourself about to ask "how does feature X work today?", that's a `grep` task, not a question for the human.

Batch independent reads — don't make one tool call per file when you can read several at once.

## 2. Check for dormant capability

A surprisingly common outcome: the capability being requested already exists somewhere in the code but was never wired up or populated.
Grep the mechanism end-to-end for a field, hook, or branch that already anticipates the ask.

If you find one, the whole interview scope collapses from "design new plumbing" to "populate a field that already exists."
Lead the interview doc's baseline section with a **"Key finding — the mechanism is already built"** callout.
The design tree then only needs to resolve *which producers set the field, from what data source, and with what exact value* — not the schema/transport/consumer branches you'd otherwise need.

Also check `development_docs/`, ADRs, or equivalent for a prior interview or plan that already reasoned about the dormant capability — its rationale is stronger evidence than re-deriving the argument from first principles.

## 3. Check for prior decisions

A structural pattern someone's asking you to reconsider is often the resolved output of an earlier design discussion.
Before reasoning from first principles:

- Grep `development_docs/` (or the project's equivalent) for the file or pattern in question.
- Check `AGENTS.md`, `CLAUDE.md`, or the project's conventions file for mentions.
- Run `git log --oneline -- <file>` and `git status` to see if the structure is recent or still-in-flight work on the current branch.

Citing a prior decision record ("this was resolved as Q-C3 in the earlier interview doc, here's the rationale") beats re-arguing general architecture principles, and it also tells you whether you're being asked to revisit your own earlier work from this session.

## 4. Software-specific pitfalls

These are the software-flavored versions of the general pitfalls in `SKILL.md`:

- **Asking questions you could answer by grepping.** "How is feature X used today?" is a `grep -rn` away. Don't ask the human to explain their own codebase to you when you have file access.
- **Skipping the mechanism check on gating/permission decisions.** Changing a value in an allow-list or requirement tuple doesn't just store a different value — it can flip auto-resolution behavior from "never activates" to "activates for a subset of callers." Trace the actual resolution code path an answer touches before marking it `RESOLVED`.
- **Cross-repo read_file returning empty.** When reading a second repo via relative paths, `read_file` may resolve relative to the wrong working directory and silently return empty content. Resolve the absolute path first (e.g. `find /Users/<user> -maxdepth 6 -type d -name '<repo>'`), then use absolute paths for all subsequent reads.
- **Assuming persistence support without checking granularity.** Before proposing "just store it in state/config," check whether the storage layer actually supports the history/scoping granularity the feature needs. A last-write-wins store only ever holds the most recent value — confirm this before writing implementation steps.

## 5. When the topic is "generalize an existing mechanism"

A recurring sub-pattern: the ask isn't "design something new," it's "generalize this existing single-purpose implementation so more callers can reuse it."
The interview shape here is different from a fresh design — the spine of the analysis is finding the **hard-coded couplings** that block reuse, not brainstorming new options from scratch.

### Find the couplings

Read the mechanism end-to-end first: every place it's constructed, every place it's consumed, every field it carries.
Then enumerate the hard-coded couplings. Common shapes:

- **A domain-specific required field** that only makes sense for the one use case (e.g. a required integer ID when a second use case might not have an ID yet, or needs a different type).
- **A closed enum whose vocabulary fits exactly one use case** (e.g. `approved`/`rejected`/`skipped` describes a confirmation decision, not a general operation outcome — it has no vocabulary for "created" or "failed").
- **A hard-wired lookup key on the consuming side** — a switch-statement or i18n key set keyed on one specific value, where a second caller has nowhere to hang its own behavior.

Each coupling becomes a branch in the design tree.
Lead with a short numbered list ("the N hard-coded couplings that block generalization") before diving into the tree.

### Frame the ROOT as a scope question

The root question is almost always "what is the right scope of the generalized primitive?"
Present it as narrow / medium / broad:

- **Narrow** = today's single use case, unchanged. Too narrow if the concrete near-term need (a second caller you already know is coming) would immediately break it.
- **Medium** = the near-term class of uses you can already name. Usually the right answer — absorbs the known need at low cost.
- **Broad** = "any conceivable future caller." Usually premature — adds a registry/dispatch layer that isn't needed yet.

Anchor the scope recommendation in the actual near-term roadmap, not hypothetical future flexibility.

### Design for opt-in extension

The strongest generalization pattern: make emission opt-in via the return value — a producer participates if and only if it returns a value shaped the right way, with a discriminator field the shared reader checks.
Adding a new caller never touches the shared plumbing — only the new producer's own file changes.
