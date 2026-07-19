# Interview: <Topic>

**Status:** Iteration 1 | In progress | Resolved
**Related plan:** (link once implementation-plan phase starts)

## How to use this doc

Answer inline. Each question gets a `Q:` and an `A:` slot right below it —
fill in the `A:` when you've decided. Use `Note:` for anything that doesn't
fit the question/answer shape (a stray idea, a constraint you just
remembered). Status tags used throughout this doc:

- `OPEN` — not yet answered
- `REC` — the agent has a recommendation, awaiting your confirmation
- `RESOLVED` — decided, ripple effects folded into the doc
- `DEFERRED` — explicitly punted, with a one-line reason

When you're done making notes for this round, just say something like "made
some notes, check the file" — no need to restate them in chat.

---

## Baseline — how this works today

_Not up for debate here, just grounding context. Verified in code, with
file:line references, not assumptions._

| Aspect | Current behavior | Source |
|---|---|---|
| | | |

**Key finding (if any):** _state here if a dormant/partial version of the
requested capability was found during the code read — this can collapse
the rest of the tree significantly._

---

## Design tree

```
ROOT: <the core scope/shape decision>
├── Branch A: <e.g. schema shape>            [depends on ROOT]
├── Branch B: <e.g. producer/data source>    [depends on ROOT, Branch A]
└── Branch C: <e.g. consumer/rendering>      [depends on Branch A]
```

---

## Iteration 1

_Only the questions whose answers unlock the next layer. Later iterations
get previewed with a stated lean but aren't finalized until upstream nodes
resolve._

### Q1 — <question> `OPEN`
**Recommendation:** <your recommended answer with reasoning>

Q: <question text>
A:

### Q2 — <question> `OPEN`
**Recommendation:** <your recommended answer with reasoning>

Q: <question text>
A:

---

## Answer log / parking lot

- Resolved so far: (none yet)
- Awaiting: Q1, Q2
- Parking lot (noticed but not yet scheduled):
