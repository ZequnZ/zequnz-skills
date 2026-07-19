# Lessons Learned

This file is the **self-evolving memory** of the skill. It is not
documentation of the workflow (that's `SKILL.md` and `references/`) — it's a
running log of concrete, generalized things learned from real usage across
whatever projects install this skill.

## Why this file exists

An agent running this skill on project A learns something (a new pitfall, a
new answer-shape pattern, a reconciliation case worth remembering) that
would help an agent running it on project B, months later, with no shared
memory. Since agents don't otherwise carry context between sessions or
between projects, this file is the mechanism that lets the skill get better
every time it's used, instead of staying frozen at whatever it looked like
on the day it was written.

## How to add an entry

After a real interview or plan-review cycle completes, spend two minutes
asking: *is there something here that would help next time, on a different
project?* If yes, append an entry using this format:

```markdown
### <short title>
**Context:** one line — what kind of task surfaced this (no company names,
no ticket IDs, no proprietary schema/field names — describe the *shape* of
the situation, not the specifics).
**Lesson:** the generalized takeaway, phrased so it applies beyond the one
project it came from.
```

Keep entries short. If a lesson is genuinely one-off and doesn't generalize,
don't force it in — this file's value comes from signal density, not
completeness.

### Hard rule: no proprietary specifics

This skill is meant to be installed across many different codebases,
including other people's. Before appending anything, strip:
- Company / client / product names
- Ticket or issue IDs
- Internal schema field names, table names, endpoint paths
- Any data that identifies a specific customer or business relationship

Generalize the shape instead. Example — wrong: *"the `invoiceNumber` field
on `SalesInvoice` was already wired to the render function at Acme Corp but
never populated."* Right: *"a producer-consumer pair already had the wiring
for a field, and the consumer already preferred it with a graceful fallback
— it just needed the producer to actually set it. Worth checking for this
class of 'already built, just not populated' situation before designing new
plumbing."*

---

## Entries

### Dormant capability collapses the interview scope
**Context:** A request to surface an extra piece of information through an
existing pipeline (e.g. "show the human-facing identifier instead of the
internal one").
**Lesson:** Before writing schema/transport/UI branches into the design
tree, grep the render/consumption side for a fallback chain that already
anticipates the field (`preferred_value or fallback or generic_default`).
If it's already there, the entire tree collapses from "design new plumbing"
to "figure out which producer populates the field, from what source, with
what exact copy" — a much smaller and different set of questions than you'd
otherwise draft.

### Data-availability asymmetry across producer sites is the real design branch
**Context:** Populating a field across multiple call sites that all build
the same kind of object/record, where the task looks like "just set field X
everywhere."
**Lesson:** Grep every construction site of the object being changed and
note what data each one actually has in hand at that point in the code —
they are rarely symmetric. Some paths have the full backing data available
for free; one path may have been built from a narrower payload (a preview,
a partial response) that never carried the needed value. The asymmetry —
not the field itself — is the actual structural decision, and deserves its
own tree node (ship the easy paths now vs. thread the value through the
narrower path too).

### An answer can flip resolver *behavior*, not just a stored value
**Context:** A permission/gating or auto-resolution mechanism where an
allow-list or requirement tuple determines whether something activates
automatically.
**Lesson:** Changing a tuple from "empty" (never auto-triggers) to
"containing one specific value" doesn't just store a different value — it
can flip the mechanism from "explicit opt-in only" to "auto-resolves for a
subset of callers." Always trace the actual resolution code path an answer
touches, not just the text of what changed, before marking a permission or
gating decision `RESOLVED`.

### A reversal can revive an earlier "over-engineered" recommendation
**Context:** Multi-round interview where round 2 reverses a decision from
round 1 (e.g. "actually, render this on the other side of the system than
we said before").
**Lesson:** When a reversal happens, don't just update the node it directly
touched — check every other node whose recommendation depended on the old
answer. A pattern you dismissed as over-engineered under the old assumption
can become the *consistent* choice under the new one. Say this coupling out
loud so the human sees their answers reinforcing each other rather than
experiencing the change as a surprise contradiction you introduced.

### Persistence-layer granularity must be checked before proposing "just store it"
**Context:** Designing where a piece of per-event or per-turn data should
live in an existing state/config/history mechanism.
**Lesson:** A last-write-wins storage channel only ever holds the most
recent value across the whole lifetime of whatever it's attached to. If the
feature needs to recall a specific past instance (not just "the latest"),
that channel is structurally insufficient regardless of what type it
stores. Confirm the storage mechanism's actual granularity before writing
implementation steps that assume it can do more than last-write-wins.
