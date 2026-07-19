# Implementation Plan Phase

Once the interview doc's design tree is fully `RESOLVED` (or explicitly
`DEFERRED`), write a separate, standalone implementation plan. Keep it
separate from the interview doc — the interview is a design-alignment
record; the plan is an executable checklist. Mixing them makes both harder
to read later.

## File location and naming

`development_docs/implementation-plan/<topic>-impl-plan.md` (adjust to the
project's actual convention if `development_docs/` doesn't exist — check for
an existing docs folder pattern first, e.g. `docs/`, `rfcs/`, `adr/`).

If a topic already has one or more draft plans, read them first and state
explicitly at the top of the new plan which earlier docs it supersedes:

```markdown
> This plan supersedes `earlier-draft.md`, which predates the
> interview-resolved approach described below.
```

## Plan structure

```markdown
# Plan: <Feature Title>

**Status:** Draft | Approved | In Progress | Done
**Interview doc:** development_docs/interview/<topic>.md (resolved)

## Summary
One paragraph: what's being built, why, what problem it solves.

## Goal & Scope
In scope / explicitly out of scope / success criteria.

## Design decisions recap
Pull the RESOLVED table straight from the interview doc — don't make the
reader cross-reference two files to know what was decided and why.

## Files touched
Exact paths, grouped by whether they're new or modified.

## Step-by-step checklist
- [ ] 1. <file path> — <what changes and why>
- [ ] 2. ...
Each unchecked item becomes `- [x]` with a one-line result note once done,
e.g. `- [x] 3. modify_writer.py — done, both reject paths now emit the
outcome artifact`.

## Test plan
Grep the actual assertion sites and fixtures BEFORE writing this list —
a vague "update the tests" item is not useful. Name the specific test
files/functions that need new cases or updated assertions.

## Edge cases / non-goals
What this plan deliberately does not handle, and why.

## Definition of done
- [ ] All checklist items complete
- [ ] Tests passing (use the project's actual verify command, not a
      guessed one — check the project's own conventions file)
- [ ] No commits made between steps unless explicitly requested
```

## Executing the checklist

- Use a live todo/task-tracking mechanism for in-session visibility, **but
  also edit the plan file itself**, flipping `[ ]` to `[x]` with a result
  note per step. In-session tracking is ephemeral; the plan file in the
  repo is the durable artifact the human re-opens later. Marking off only
  the ephemeral tracker leaves the actual file looking untouched.
- Verify per step or per logical group, using the project's real test
  invocation (check its own docs/conventions for the exact command — don't
  assume a bare `pytest` or `npm test` is correct if the project needs
  environment variables or a specific test runner flag).
- Don't commit between steps unless explicitly asked. Finish the whole
  checklist, run the full verification once at the end, then stop and
  report — don't declare "done" without having actually run it.
- When an assumption baked into the plan turns out wrong during
  implementation (an import wasn't actually present, a variable assumed
  always-bound turns out conditional), fix it inline as part of that step.
  Note it briefly in the step's result line if material. Don't treat every
  small plan inaccuracy as a blocking defect requiring a stop-and-ask.

## The note-answer loop applies to plan review too

Plan docs go through the same inline-editing review cycle as interview
docs — the human drops notes in the file rather than replying in chat, and
says something like "I made some notes, check it out." Apply the exact same
protocol as `references/answer-handling-protocol.md`:

1. Search for notes (grep, don't diff — the file is often untracked).
2. Trace the *mechanism* implication of each answer, not just the text.
   A plan-review answer reversing a recommendation is just as likely here
   as in the interview phase, and just as likely to carry a non-obvious
   behavioral implication.
3. Propagate the decision to every dependent section (options table, risk
   notes, test plan, PR-split notes, definition of done) — one answer
   commonly touches four to six places.
4. When an inline note asks you to *evaluate* something ("can this existing
   helper be reused? check it out"), actually go read the code and write
   the verdict into the doc where the question was — don't restate the
   question or hand-wave a plausible-sounding answer.
5. Clean up the raw `Q:`/`A:` markers once folded in.

## Clarifying wire-contract / API-shape decisions before writing steps

When the plan involves a contract change that another system (a separate
frontend, another service, an external consumer) has to consume, ask the
human interactively before writing the detailed implementation section —
not as a buried open question in the document. State 2-3 options
concisely with a stated lean, ask them to confirm, and only then write the
steps. This avoids producing a fully detailed plan for an approach that
gets rejected in the first review round.

## Cross-repo follow-up: don't assume "no schema change" means "nothing to do elsewhere"

If the plan is scoped to one repo/service but touches a payload a separate
consumer (a frontend, another backend, a partner integration) reads, check
whether the field is:

- **An opaque passthrough** (e.g. a display string rendered verbatim) — the
  consumer needs zero changes, it renders whatever comes through.
- **A structured/validated field** consumed through a schema (a typed
  validator, a strict parser) on the other side. Structured validators
  frequently strip unknown fields silently — no error, the data just
  doesn't reach the render layer. If the plan adds a new field to a
  payload another system validates, that consumer's schema needs updating
  too, or the new data will arrive over the wire and vanish before anyone
  sees it.

When this applies, go read the actual consuming code (path is usually
documented in the project's own conventions file) rather than guessing
whether it needs updating. Classify each new field as opaque-passthrough
or schema-validated, and only add the second kind to the plan's checklist.
Explicitly list the opaque fields as "no action needed" too, so nobody
re-investigates them later.

## Pitfalls

- **Writing before reading.** Read the actual source files and the
  project's own conventions doc before drafting steps. A plan that
  misdescribes the current architecture is worse than no plan.
- **Skipping alternatives.** Document at least one alternative even if
  quickly dismissed — a single-option plan gives the reviewer nothing to
  weigh it against.
- **Vague steps.** "Update the service layer" is not an actionable step.
  Name the exact file, the exact function/class, and what changes.
- **Open questions buried in prose** instead of a dedicated list a
  reviewer can action.
- **Omitting definition of done.** Without it, "complete" is subjective.
- **Copying a sibling's permission/scope values wholesale** when adding a
  new capability that resembles an existing one. Auto-resolution or
  allow-list mechanisms can behave very differently depending on which
  scope values are present versus absent — verify the actual resolution
  code path rather than assuming "copy the neighbor's config" is safe. A
  narrower or explicitly-scoped value is usually the safer default; pair
  any expansion with an explicit feature flag if the project has that
  mechanism available.
- **Assuming persistence support without checking granularity.** Before
  proposing "just store it in state/config", check whether the storage
  layer actually supports the history/scoping granularity the feature
  needs. A last-write-wins store only ever holds the most recent value —
  if the feature needs per-item or per-turn history, that store is
  insufficient regardless of what type it holds. Confirm this before
  writing implementation steps, not after building the wrong thing.
