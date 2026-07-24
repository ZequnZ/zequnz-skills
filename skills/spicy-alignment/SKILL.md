---
name: spicy-alignment
description: Structured, whiteboard-first alignment before action. Produces a resolved interview doc with every decision node marked `RESOLVED` or `DEFERRED`.
license: MIT
disable-model-invocation: true
metadata: 
  author: ZequnZ
  version: 1.0
  updated: "2026-07-23"
---

# Spicy Alignment

A workflow for driving a decision conversation to convergence *before* anyone starts executing.
The core idea: instead of asking clarifying questions in chat (which get lost) or guessing (which erodes trust), write a structured **design tree** to a markdown file, resolve it one dependency-ordered layer at a time, and let the human answer inline in the file itself.

The output is a single artifact: a **resolved interview doc** with every decision node marked `RESOLVED` or `DEFERRED`.
What you do with it next — implementation plan, action items, a separate design doc — is outside this skill's scope.

## When this applies

- The user explicitly asks for a discussion-first / whiteboard-first approach, or says something like "don't just do this, let's align first."
- The task is a non-trivial decision with real tradeoffs: multiple viable approaches, unclear scope, competing constraints, or consequences that are hard to reverse.
- If the user says "just do X" / "write the patch" / "go ahead" — skip this workflow entirely and go straight to execution. This is for alignment thinking, not for confirmed requests.

## Workflow

### 1. Do your homework first

Before writing a single question, research the topic yourself.
Every question in the interview doc should be a genuine unknown — something you could not resolve on your own.

General homework checklist:
- What exists today? (The baseline — verified with evidence, not assumptions.)
- Has this been decided before? (Check prior docs, meeting notes, decision records.)
- Is there a partial or dormant version of what's being asked for? (Check existing systems, processes, or artifacts before designing new ones.)

If you find yourself about to ask the human something on this list, that's research, not a question.

**Domain-specific pre-work:** Check `references/` for a pre-work doc matching this domain.
If one exists, read it before writing the interview doc.

### 2. Write the interview doc

Create the interview doc in a location that makes sense for the project (e.g. `development_docs/interview/<topic>.md` for a codebase, a shared docs folder for a team process).
Use `templates/interview-template.md` as a starting point.

The structure:

- **How to use this doc** — the inline `Q:`/`A:` protocol and status tags (`OPEN`, `REC`, `RESOLVED`, `DEFERRED`).
- **Baseline** — grounding context about how things work today, verified with evidence, not assumptions. This section is "not up for debate, just context."
- **Design tree** — an ASCII tree showing the ROOT decision and its branches, with `[depends on X]` annotations showing what each branch needs resolved first.
- **Iteration 1** — only the questions whose answers unlock the next layer. Not a flat dump of every question in the tree.
- **Answer log / parking lot** — what's resolved, what's pending, loose threads noticed but not yet scheduled.

The discipline that makes this work:

- **Resolve in dependency order.** A root-level answer usually collapses most of the leaf questions below it — don't ask leaf questions before the root is settled.
- **Always give a recommended answer.** Never hand the user a blank question. State your lean and why, then let them confirm, override, or redirect.
- **Later iterations can be previewed** with your leaning stated, so the user can redirect early — but don't finalize them until the upstream node actually resolves.

Summarize the doc's key points and open questions in chat too — don't just say "I wrote a file," restate what's in it so the human doesn't have to open it to know what you're asking.

### 3. Process answers each round

The user will often not reply in chat.
They'll open the file, drop notes next to questions, and say something like "I made some notes, check the file."
Read `references/answer-handling-protocol.md` for the full protocol — it covers the four shapes an answer can take (plain pick, contradiction/reversal, "evaluate this idea", "elaborate with examples") and exactly how to handle each.

The critical distinction: not every answer is a plain pick.
Contradictions reopen resolved nodes, "evaluate this idea" opens new sub-nodes, and "elaborate with examples" signals your recommendation wasn't concrete enough.
Treating all four shapes as plain picks is the most common way this workflow goes wrong.

The one-line summary: **find every note by searching** (check whether the file is git-tracked — if yes, `git diff` is your best tool; if not, grep for answer markers + check timestamps); trace the *mechanism implication* of each answer, not just the text; propagate the decision to every dependent section; and when a later answer reverses an earlier one, name both, say which one now governs, and ask an explicit reconciliation question.

### 4. Repeat until the tree is fully resolved

Keep the **interview**iterating until every node in the design tree is `RESOLVED` or `DEFERRED`.
Promote parking-lot items into new iteration questions when their upstream dependencies resolve.
The interview doc is done when the tree shows all nodes resolved and the answer log has no remaining `Awaiting` items.

### 5. Self-evolve: capture the generalizable lesson

After a real interview cycle completes, spend two minutes checking whether anything you learned would help next time, on a different project.
If yes, append an entry to `LESSONS.md` following the format and anonymization rules in that file's header.
Read `LESSONS.md` before starting a new interview — it contains accumulated wisdom from prior usage.

Not every session needs to grow this file.
If the lesson is genuinely one-off and doesn't generalize, don't force it in.

## Pitfalls

- **Jumping straight to execution.** The #1 failure mode of any interview. "Any ideas?" does not mean "write a patch" or "draft the plan." If the user wanted action, they'd have said so.
- **Asking questions you could resolve yourself.** A question the human has to answer that you could've figured out wastes their attention and signals you skipped the homework.
- **Silently picking a side when an answer contradicts an earlier one.** This will happen — understanding evolves over rounds. Reconcile it explicitly (see the answer-handling reference), never paper over it.
- **Treating "evaluate this idea" as a decision.** When the user floats an idea instead of picking from your options, that opens a new evaluation node — investigate the implications, present sub-options, recommend, and if it's a scope/appetite call rather than a correctness call, say so and hand the decision back.
- **Skipping the implication check.** The text of an answer can look like a simple substitution and still change real behavior elsewhere. Trace what the answer actually implies for other decisions before writing it down as resolved.
- **Writing proprietary specifics into the shared `LESSONS.md`.** This skill is meant to be installed across different projects and by different people — company names, ticket IDs, internal details, and customer data don't belong. Generalize the shape of the lesson, drop the specifics.

## References

- `references/answer-handling-protocol.md` — the four answer shapes, worked examples of contradiction/reversal handling, and the "reversal can revive a previously-shaky decision" pattern.
- `references/software-pre-work.md` — domain add-on for software design interviews: code reading, dormant capability check, prior-decision check, and the "generalizing an existing mechanism" sub-pattern.
- `LESSONS.md` — growing, self-evolving log of generalized lessons from real usage. Read it before starting a new interview; append to it after finishing one (see step 5 above).
- `templates/interview-template.md` — copy-paste starting point for interview docs.
