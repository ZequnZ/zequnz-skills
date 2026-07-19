# Answer-Handling Protocol

When the human answers questions in the interview doc, their reply takes one
of four shapes. Detecting which shape you're looking at determines what you
do next — treating all answers as "plain picks" is the most common way this
workflow goes wrong.

## Finding the answers in the first place

The interview doc is very often **untracked** in git (it's a new file), so
`git diff` and `git status --short` show nothing useful even after the user
edits it. Don't rely on git to find changes. Instead:

```bash
# what changed recently, if the file is tracked or you want a timestamp hint
find . -path ./.git -prune -o -type f -mmin -25 -print

# the notes themselves — grep broadly, users are inconsistent about markers
grep -nE "^(A|Q):" development_docs/interview/<topic>.md
grep -nE "Note:|I think|let's go with|actually|instead" development_docs/interview/<topic>.md
```

Users mix styles across rounds: a terse `A: sounds good`, a `Q:` that's
actually a request to go verify something in code, a `Note:` buried mid
paragraph, or bare free-text dropped after a numbered question with no
marker at all. Grep for several patterns and read every hit in context.
Missing one note means shipping a half-updated doc — worse than not
updating it at all, because it looks complete but isn't.

## Shape 1 — A plain pick

The user chose one of your offered options outright ("B", "let's go with
the narrow scope", "yes to your recommendation").

**Action:** mark the node `RESOLVED` in the tree. If the pick has ripple
effects on other nodes — and it usually does — state the ripple explicitly
in a "Resolved / reconciled" block so the human sees what their answer
implied, rather than silently updating five other sections and hoping
they notice. Example shape: "landing on the fully-generic option means the
field that was required now needs to become optional, and the name in the
schema changes from X to Y — noting that here so it's not a surprise later."

## Shape 2 — An answer that contradicts an earlier resolved node

This **will** happen across multiple rounds. Design understanding evolves;
a user who said "a plain string is enough, backend renders it" in round 1
may say "no, structured data only, frontend renders it" in round 2. This is
normal design evolution, not user error — but it must be reconciled on the
record, never silently picked.

**Action:**
1. Detect the contradiction (this is why the propagation checklist below
   matters — you can't reconcile what you haven't tracked).
2. Name both answers explicitly and which round each came from.
3. State which one now governs and why.
4. Add an explicit reconciliation question if there's any ambiguity about
   whether the reversal was intentional (`Q-R1: confirm this is meant to
   override the earlier answer from round 1`).

**The revival pattern** — a subtlety worth remembering: a late reversal can
make a *different, earlier* decision suddenly correct after being flagged as
over-engineering. If round 1's pick led you to recommend a simple data-only
model (and you called a more elaborate per-type-behavior pattern
over-engineering for that shape), and round 2 reverses the earlier pick,
re-check every node whose recommendation *depended* on the now-reversed
answer. The earlier "over-engineered" option may now be the *consistent*
choice for the new shape. State this coupling explicitly ("your reversal
here flips X, and that revives the pattern you leaned toward in round 1 —
your two answers now reinforce each other") so the user sees the connection
rather than experiencing it as whiplash.

## Shape 3 — "Evaluate this idea" (not a pick)

The user doesn't choose from your options — they float their own idea and
ask you to assess it: "what if we made this a union type instead, evaluate
that", "maybe each caller builds its own version of this, thoughts?"

**This does not resolve the node.** It opens a new evaluation sub-node.

**Action, in order:**
1. **Measure the blast radius in code first.** Grep every construction/
   consumption site of the thing being changed, count them, and note which
   ones are already shaped the way the idea implies — this is often the
   real finding (e.g. "the frontend already treats this as a union
   internally; only the wire format is flat").
2. Present 2-3 concrete sub-options grounded in that measurement, not in
   the abstract.
3. Give a recommendation. When the idea is a scope/appetite tradeoff rather
   than a correctness question, **say so explicitly and hand the decision
   back** — "this is a scope call, not a correctness one: the more general
   shape is the better long-term choice but strictly more work than the
   immediate need requires."
4. Reject sub-options that lose type safety or that fight an
   already-resolved decision, and say why you're rejecting them.

## Shape 4 — "Elaborate / justify with examples"

The user replies to what looked like a resolved or near-resolved question
with "can you show me examples?" or "walk through the pros/cons with real
code for each." **This does not resolve the node either** — it's a signal
that your recommendation wasn't concrete enough to decide on.

**Action:** don't re-ask the same question. Answer it with enumerated,
concrete scenarios:
- For a "do we still need X?" question: list the 2-3 real consumers that
  could plausibly want X and show, for each, whether it actually needs X.
- For a "compare A vs B" ask: write out both with real code snippets side
  by side, not prose paragraphs.

Then restate the recommendation and leave a fresh `A:`/`A2:` slot so the
now better-informed user can confirm. The signal here is that the user is
close to agreeing but wants the reasoning made tangible before committing
— give them something concrete, don't re-litigate the abstract question.

## Propagation checklist (do this for every resolved node)

One answer typically touches more than the section it directly landed in.
After folding a decision in, sweep this list:

- [ ] The section that directly owns the decision (rewrite it from
      "candidate/recommended" language to "decided" language)
- [ ] Any options-considered / decision table elsewhere in the doc
- [ ] Risk callouts (a mitigation column often changes from "consider X"
      to "decided: X")
- [ ] Test-strategy notes, if the doc has them — a decision usually implies
      a new test case
- [ ] Open-questions list (move the resolved item into a "resolved this
      round" subsection instead of just deleting it — keeps a record)
- [ ] **Stale-phrasing sweep** — grep the OLD recommendation's keywords and
      fix every place the new decision invalidates. A decision recorded
      once but contradicted three sections later is worse than not
      recording it at all.

## Cleanup

Convert raw `A: sounds good` markers into a proper prose "Resolved this
round" entry once folded in. Delete the raw `Q:`/`A:` scaffolding lines
after processing so the doc reads as a clean decision record, not a
transcript. Verify with `grep -nE "^(A|Q):" <file>` — it should return
nothing once a round is fully processed (any remaining hits are questions
you haven't answered yet, which is the correct signal to keep iterating).
