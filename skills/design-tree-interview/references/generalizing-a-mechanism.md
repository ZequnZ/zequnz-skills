# Generalizing an Existing Mechanism

A recurring sub-pattern: the ask isn't "design something new," it's
"generalize this existing single-purpose implementation so more callers can
reuse it." The interview shape here is a bit different from a fresh design
— the spine of the analysis is finding the **hard-coded couplings** that
currently block reuse, not brainstorming new options from scratch.

## Step 1 — Read the mechanism end to end

Before writing any tree nodes, trace the existing implementation completely:
every place it's constructed, every place it's consumed, every field it
carries. You need this map to find the couplings in step 2.

## Step 2 — Enumerate the hard-coded couplings

Each coupling you find becomes its own branch in the design tree. Common
shapes to look for:

- **A domain-specific required field.** A field name or type that only
  makes sense for the one use case it was built for (e.g. a required
  integer ID when a second use case might not have an ID yet, or might
  need a string/list instead).
- **A closed enum whose vocabulary fits exactly one use case.** An enum
  like `approved`/`rejected`/`skipped` describes a *confirmation decision*,
  not a general *operation outcome* — it has no vocabulary for "created,"
  "deleted," or "failed." Watch especially for enums that silently drop a
  state the single-purpose version never needed (e.g. a "failed" outcome
  that today just results in an empty response rather than a recorded
  state).
- **A hard-wired lookup key on the consuming side.** UI copy, i18n keys, or
  a switch-statement keyed on one specific value — a second caller has
  nowhere to hang its own copy/behavior without either duplicating the
  switch or falling through to nothing.

For each coupling found, lead with a short numbered list like "the N
hard-coded couplings that block generalization" before diving into the
tree — this gives the reader the shape of the problem before the detail.

## Step 3 — Frame the ROOT question as a scope decision

The root question is almost always **"what is the right scope of the
generalized primitive?"** Present it as narrow / medium / broad:

- **Narrow** = today's single use case, unchanged. Too narrow if the
  concrete near-term need (a second caller you already know is coming)
  would immediately break it.
- **Medium** = the near-term class of uses you can already name (e.g. "any
  write-operation outcome, whether or not it goes through a
  confirmation/interrupt step"). This is usually the right answer — it
  absorbs the concretely-known near-term need at low cost.
- **Broad** = "any conceivable future caller deserves this." Usually
  premature — it tends to add a registry/dispatch layer that isn't needed
  yet. Resist it unless the user names a concrete case that needs it now.

Anchor your scope recommendation in the actual near-term roadmap (what's
genuinely coming next), not hypothetical future flexibility.

## Step 4 — Design for opt-in extension, not central registration

The strongest generalization pattern found in practice: make emission
**opt-in via the return value** — a producer participates in the mechanism
if and only if it returns a value shaped the right way, with a discriminator
field (e.g. a `"kind"` string) that the shared reader checks. This means
adding a new caller never touches the shared streaming/consumption
plumbing — only the new producer's own file changes. Keep the single
extraction/read point unchanged; only the payload shape widens.

## Step 5 — Extract shared builders once you're touching multiple call sites

If the single-purpose version hand-rolls its payload construction in
multiple places (e.g. the same object literal built three times in one
file), generalizing is the natural moment to extract a single builder
helper that every producer calls, rather than letting each new caller
re-invent the construction.

## Step 6 — Forward compatibility without over-engineering

Don't add a `schema_version` field or similar defensive plumbing unless
there's a concrete reason to expect old and new payload shapes to coexist
in production data. Do keep strict validation on the write side (reject
unknown fields when constructing), but make the **read side** tolerant of
unknown values in the discriminator (treat an unrecognized value as "no
match" rather than raising) — this means a newer producer shipped mid
rollout doesn't break an older consumer that hasn't been redeployed yet.
