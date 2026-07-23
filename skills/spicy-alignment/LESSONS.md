# Lessons Learned

This file is the **self-evolving memory** of the skill.
It is not documentation of the workflow (that's `SKILL.md` and `references/`) — it's a running log of concrete, generalized things learned from real usage.

## Why this file exists

An agent running this skill on project A learns something that would help an agent running it on project B, months later, with no shared memory.
Since agents don't carry context between sessions or projects, this file is the mechanism that lets the skill get better every time it's used.

## How to add an entry

After a real interview cycle completes, ask: *is there something here that would help next time, on a different project?*
If yes, append an entry:

```markdown
### <short title>
**Context (<domain>):** one line — what kind of task surfaced this.
**Lesson:** the generalized takeaway.
```

Domain tags: `general`, `software`, `product`, `process`, or whatever fits.
When this file grows past ~20 entries, consider splitting into `lessons/general.md` + `lessons/<domain>.md` — the tags make that a grep-and-move operation.

### Hard rule: no proprietary specifics

Before appending anything, strip company/client/product names, ticket IDs, internal schema/field names, and customer data.
Generalize the shape of the situation, not the specifics.

---

## Entries

### Dormant capability collapses the interview scope
**Context (software):** A request to surface an extra piece of information through an existing pipeline.
**Lesson:** Before writing branches for schema/transport/consumer changes, check whether the consumption side already has a fallback chain that anticipates the field but nobody ever populated it.
If it does, the whole tree collapses from "design new plumbing" to "which producer sets the field, from what source, with what exact value" — a much smaller set of questions.

### Asymmetry across producer sites is the real design branch
**Context (software):** Populating a field across multiple call sites that all build the same kind of record.
**Lesson:** Check every construction site and note what data each one actually has in hand — they are rarely symmetric.
Some paths have the full backing data for free; one path may have been built from a narrower payload that never carried the needed value.
The asymmetry — not the field itself — is the actual structural decision and deserves its own tree node.

### An answer can flip behavior, not just a stored value
**Context (general):** A decision involving a gating or auto-resolution mechanism where membership in an allow-list determines whether something activates automatically.
**Lesson:** Changing a value from "absent" to "present" in a list doesn't just store a different value — it can flip the mechanism from "opt-in only" to "auto-activates for a subset of cases."
Always trace the actual resolution logic an answer touches, not just the text of what changed, before marking a gating decision `RESOLVED`.

### A reversal can revive an earlier "over-engineered" recommendation
**Context (general):** Multi-round interview where round 2 reverses a decision from round 1.
**Lesson:** When a reversal happens, don't just update the node it directly touched — check every other node whose recommendation depended on the old answer.
A pattern you dismissed as over-engineered under the old assumption can become the *consistent* choice under the new one.
State this coupling explicitly so the human sees their answers reinforcing each other.

### Confirm storage granularity before proposing "just store it"
**Context (general):** Designing where a piece of per-event or per-item data should live.
**Lesson:** A last-write-wins storage mechanism only holds the most recent value.
If the feature needs to recall a specific past instance (not just "the latest"), that mechanism is structurally insufficient regardless of what type it stores.
Confirm the storage layer's actual granularity before writing it into the tree as a viable option.
