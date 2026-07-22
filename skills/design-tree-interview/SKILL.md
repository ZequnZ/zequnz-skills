---
name: design-tree-interview
description: >
  Structured, whiteboard-first design alignment before any code gets written.
  Use whenever a user wants to think through a non-trivial design decision
  before implementing it — "let's align on the design first", "walk me
  through the tradeoffs", "I want to interview through this", "put together
  a design doc and let's iterate", or any request to resolve an
  architecture/scope/schema decision collaboratively rather than have an
  agent guess and build. Produces a living markdown whiteboard the human
  edits inline, then a separate implementation plan once design is stable.
  Also use when picking up an existing interview/plan doc mid-cycle ("I left
  some notes in the file, check it out").
---

# Design Tree Interview

A workflow for driving a design conversation to convergence *before* an agent
starts writing code. The core idea: instead of asking clarifying questions in
chat (which gets lost) or guessing (which erodes trust), write a structured
**design tree** to a markdown file, resolve it one dependency-ordered layer
at a time, and let the human answer inline in the file itself.

This produces two artifacts in sequence:

1. An **interview doc** — pure design alignment, no implementation detail.
2. An **implementation plan** — the executable checklist, written only after
   the interview is stable.

Keeping these separate matters: the interview doc stays readable as a record
of *why* a decision was made, and the plan stays focused on *what* to do.

## When this applies

- The user explicitly asks for a discussion-first / whiteboard-first
  approach, or says something like "don't just implement this, let's align
  first."
- The task is a non-trivial design decision: new schema/contract shape,
  generalizing an existing single-purpose mechanism, choosing between
  architectural approaches, or scoping a feature with real tradeoffs.
- If the user says "just implement X" / "write the patch" — skip this
  workflow entirely and go straight to code. This is for design thinking,
  not for confirmed, concrete edit requests.

## Workflow

### 1. Ground yourself in the code first

Before writing a single question, read the relevant code paths. Every
question in the interview doc should be a genuine unknown — something you
could not resolve yourself by reading the codebase. If you find yourself
about to ask "how does X work today?", that's a research task, not a
question for the human.

**Check for dormant capability before designing new plumbing.** A
surprisingly common outcome: the capability being requested already exists
somewhere in the code but was never wired up or populated. Grep the
mechanism end-to-end for a field, hook, or branch that already anticipates
the ask. If you find one, the whole interview collapses from "design new
plumbing" to "populate a field that already exists" — lead the doc with a
callout about this, and only ask about which producer sets it and what data
source feeds it, not the schema/transport/UI questions you'd otherwise need.

Also check whether this exact decision was already made and documented
somewhere (previous design docs, ADRs, commit messages on the current
branch). A structural pattern someone's asking you to reconsider is often
the resolved output of an earlier design discussion — citing that record
beats re-deriving the argument from first principles, and it also tells you
whether you're being asked to revisit your own earlier work.

### 2. Write the interview doc

Create `development_docs/interview/<topic>.md` (create the directory if
missing — adjust the path to match the project's convention if one already
exists). Structure:

```markdown
# Interview: <Topic>

## How to use this doc
Answer inline: `Q:` questions get an `A:` slot right below them. 
Add `Note:` for anything that doesn't fit the Q/A shape. 
Status tags: 
- OPEN (unanswered),
- REC (agent has a recommendation, awaiting confirmation),
- RESOLVED (decided),
- DEFERRED (explicitly punted).

## Baseline — how this works today (not up for debate, just context)
[A table or short prose grounding every later question in verified code
facts. Cite file:line, not vibes.]

## Design tree
[ASCII tree. ROOT decision at top, branches below with `[depends on X]`
annotations showing what each branch needs resolved first.]

## Iteration 1
[Only the questions whose answers unlock the next layer — not a flat dump
of every question in the tree. Each question gets your recommended answer
with reasoning FIRST, then the Q:/A: slot.]

## Answer log / parking lot
[What's resolved, what's pending, loose threads noticed but not yet
scheduled.]
```

The discipline that makes this work:

- **Resolve in dependency order.** A root-level answer usually collapses
  most of the leaf questions below it — don't ask leaf questions before the
  root is settled.
- **Always give a recommended answer.** Never hand the user a blank
  question. State your lean and why, then let them confirm, override, or
  redirect.
- **Later iterations can be previewed** with your leaning stated, so the
  user can redirect early — but don't finalize them until the upstream node
  actually resolves.

Summarize the doc's key points and open questions in chat too — don't just
say "I wrote a file," restate what's in it so the human doesn't have to
open it to know what you're asking.

### 3. Process answers each round

The user will not reply in chat. They'll open the file, drop notes next to
questions, and say something like "I made some notes, check the file." Read
`references/answer-handling-protocol.md` for the full protocol — it covers
the four shapes an answer can take (plain pick, contradiction/reversal,
"evaluate this idea", "elaborate with examples") and exactly how to handle
each without silently overwriting history or fence-sitting.

The one-line summary: **find every note by searching, not diffing** (the
file is often untracked, so `git diff` shows nothing); trace the *mechanism*
implication of each answer, not just the text; propagate the decision to
every dependent section, not just the one it landed in; and when a later
answer reverses an earlier one, name both, say which one now governs, and
ask an explicit reconciliation question rather than picking silently.

### 4. Transition to an implementation plan — only once the tree is stable

When every node is `RESOLVED` or `DEFERRED`, write a **separate** file:
`development_docs/implementation-plan/<topic>-impl-plan.md`. Read
`references/implementation-plan-phase.md` for the full plan structure,
checklist-execution discipline, and the note-answer loop applied to plan
review rounds (the same mechanism-over-text discipline from step 3 applies
here too — plan review rounds get edited inline the same way).

Do not inline the checklist into the final iteration of the interview doc.
Mark all interview nodes `RESOLVED`, add a one-line pointer to the plan
file, and let the plan carry the ordered checklist, files-touched map, test
plan, and edge cases as its own document.

Before starting execution: confirm the target branch with the user if the
project has a "checkout before code changes" convention, and flag any
cross-repo surface explicitly (e.g. "the backend ships a new field — a
separate frontend repo needs to render it, confirm that's in scope").

### 5. Retrospective (optional, confirm scope first)

If the user wants a retrospective after the work ships, confirm the table
of contents with them before writing it — don't assume the shape.

### 6. Self-evolve: capture the generalizable lesson

After a real interview cycle completes (not every single question — the
cycle as a whole), spend two minutes updating `LESSONS.md` in this skill's
own directory with anything you'd want to know next time:

- A new answer-shape pattern you handled that isn't already in
  `references/answer-handling-protocol.md`.
- A category of "dormant capability" you found (without naming the
  client/project — generalize it: "a resolver already existed but nobody
  had wired a caller to it" rather than the actual field/company names).
- A reconciliation pattern (an answer in round 2 reversing round 1) worth
  remembering the shape of.
- A pitfall that cost you a wasted round (e.g. you proposed a persistence
  approach before checking the storage layer actually supported the
  history-granularity the feature needed).

Write it anonymized and generalized — this file is shared across every
project that installs this skill, so it should never contain a specific
company name, internal ticket ID, proprietary schema, or customer detail.
See the header of `LESSONS.md` for the exact format. If a lesson doesn't
generalize (it's genuinely one-off), don't force it in — not every session
needs to grow this file.

## Pitfalls

- **Jumping straight to code.** The #1 failure mode. "Any ideas?" does not
  mean "write a patch." If the user wanted code, they'd have said so.
- **Asking questions you could answer by reading the code.** Grep first.
  A question the user has to answer that you could've resolved yourself
  wastes their attention and signals you didn't do the reading.
- **Silently picking a side when an answer contradicts an earlier one.**
  This will happen — design understanding evolves over rounds. Reconcile it
  explicitly (see the answer-handling reference), never paper over it.
- **Treating "evaluate this idea" as a decision.** When the user floats an
  idea instead of picking from your options, that opens a new evaluation
  node — measure the blast radius in code, present sub-options, recommend,
  and if it's a scope/appetite call rather than a correctness call, say so
  and hand the decision back.
- **Skipping the mechanism check.** The text of an answer can look like a
  simple substitution and still flip real behavior (e.g. a value going from
  "excluded from an allow-list" to "member of an allow-list" changes what
  auto-resolves, not just what's stored). Trace the code path the answer
  touches before writing it down as resolved.
- **Writing proprietary specifics into the shared `LESSONS.md`.** This repo
  is meant to be installed by other people on other codebases — company
  names, ticket IDs, internal schema field names, and customer data don't
  belong here. Generalize the shape of the lesson, drop the specifics.

## References

- `references/answer-handling-protocol.md` — the four answer shapes, worked
  examples of contradiction/reversal handling, and the "reversal can revive
  a previously-shaky decision" pattern.
- `references/implementation-plan-phase.md` — plan doc structure, the
  iterative note-answer review loop, checklist-execution discipline, and
  the cross-repo frontend-impact follow-up check.
- `references/generalizing-a-mechanism.md` — the sub-pattern for when the
  ask is "generalize this existing single-purpose thing" rather than "build
  something new": how to find the hard-coded couplings blocking reuse and
  frame the scope decision.
- `LESSONS.md` — growing, self-evolving log of generalized lessons from
  real usage. Read it before starting a new interview; append to it after
  finishing one (see step 6 above).
- `templates/interview-template.md`, `templates/implementation-plan-template.md`
  — copy-paste starting points for the two doc types.
