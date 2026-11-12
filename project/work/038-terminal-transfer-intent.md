# 038: Terminal transfer intent

| Field | Value |
| --- | --- |
| Status | Active working material / non-normative / not started |
| Work Item | `038` |
| Created | 2026-11-12 |
| Owns | The bounded review of when a provable last use needs a stated transfer stance, and how a type shows whether taking it over can change anything, as defined below |
| Does Not Own | Current transfer-stance, invocation, or type behavior, which remain with their owners until a promotion is authorized; the decision itself, which belongs to the language maintainer |

## Non-authority notice

This file is a collaborative working record. Existing documentation, candidate
syntax, examples, and later aligned findings remain non-authoritative until a
separately discussed, aligned, and explicitly authorized promotion incorporates
them into their lasting owners.

## Fixed initiating input

This section records the aligned information known when work item `038` was
created. It is intentionally incomplete and must not be rewritten as work
develops.

### Initiating concern

When must a programmer state a transfer stance (`copy`, `move`, `last`, or
`deep`) at a provable last use, and when should the compiler accept the use
without one? In particular: which types can never be affected by being taken
over, the role "plain old data" plays in other languages; whether and how a
type states that it does or does not need a stance; and how generic code is
checked when the answer depends on the type each instantiation receives.

### Motivating pressure

- **The current rule is decided by the receiver, not the type.** The
  terminal-opportunity intent error fires when an unstated last use could be
  taken over by a receiver that would accept `last` differently. It does not
  fire when `last` would accept the same contract and produce the same source
  post-state. No owner says which types make that true.
- **Plain values should not need ceremony.** Maintainer direction: plain
  values such as `Integer` should need no stance, and the compiler should not
  complain about them. Examples currently leave `Integer` cases unannotated
  without claiming a rule.
- **Generic code is the hard case.** A body may be fine for one processing and
  need a stance for another. Current examples state a stance whenever generic
  items are passed on for the last time.
- **A long-standing frustration.** The maintainer has always found it unclear,
  in C++ and elsewhere, when a type is plain data and when it needs resource
  transfer. Zax should make that visible and teachable.
- **Captured input.** [Raw terminal transfer intent](../raw/terminal-transfer-intent.md)
  preserves the representative examples and the questions future work must
  decide.

### Starting input

The maintainer has no preferred solution. The agent is asked to produce a
range of options for review, from as simple as a documented definition to
source-level solutions, such as a stance declaration on a type, that make it
clear when a stance should or should not be written. Each option should come
with advice and a recommendation, and with the feasibility pressure it places
on an eventual implementation once the design is aligned. That analysis is
candidate material, not aligned design.

Legacy and raw material is evidence. It is not authoritative over current
design or over the maintainer's direction in discussion.

### Starting boundaries

- In scope: the boundary of the terminal-use intent error; a definition of
  types for which a transfer can never change anything; whether a type can
  state or opt out of that property, and whether it is inferred from members;
  how generic and variadic code is checked; and diagnostics that explain the
  decision.
- The meaning of `copy`, `move`, `last`, and `deep` is current design. A
  conflict with it is recorded as a finding for discussion, not resolved by
  silently changing it.
- Lifetime analysis, ownership of pointers and arenas, and async transfer are
  reviewed only far enough to record constraints.
- Reflection remains outside this item, although a type-level property may
  later be reflected.

### Initial stopping guidance

First establish the current rule precisely and gather the cases a definition
must cover: scalars, fixed arrays and structures of them, references and
pointers, optionals, unions and variants, and types with programmer-declared
copy, construction, or destruction behavior. Then present the options with
examples and a recommendation for discussion. Do not design every type
category in full.

Creating and routing this work item does not authorize analysis. A later
assignment begins it.

Do not promote findings, change current owners, archive this work item, or
create work item `039` without the separately required discussion, alignment,
and authorization.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs focused reading,
  authority, promotion, deferral, and closure.
- [Raw terminal transfer intent](../raw/terminal-transfer-intent.md) - the
  preserved concern, examples, and open questions.
- [Zax transfer stances](../../language/transfer-stances.md), especially the
  terminal-intent rule, source post-state, and declaration and type-alias
  stance overlays - the model this work refines.
- [Terminal opportunity requires intent](../../language/function-invocation.md#terminal-opportunity-requires-intent)
  in function invocation - how the rule applies at a call.
- The implicit-stance acknowledgement in
  [intent acknowledgements](../../language/intent-acknowledgements.md) - the
  existing way to keep implicit behavior deliberately.

### Consequence-driven

Read only the smallest relevant sections when the concern reaches them:

- [Zax construction and destruction](../../language/construction-and-destruction.md),
  when programmer-declared copy, construction, or destruction behavior decides
  whether a type is plain.
- [Zax type definitions](../../language/type-definitions.md), when a type-level
  property or declaration is proposed.
- [Zax unions](../../language/unions.md) and [Zax variants](../../language/variants.md),
  when passive or all-bit-safe layouts or managed alternatives bear on the
  definition.
- [Zax optional values](../../language/optional-values.md),
  [Zax pointers and arenas](../../language/pointers-and-arenas.md), and
  [Zax lifetimes and references](../../language/lifetimes-and-references.md),
  when indirection or ownership layers are classified.
- [Zax generics](../../language/generics.md) and
  [Zax variadics](../../language/variadics.md#a-stance-applies-to-each-item),
  when generic or per-item checking is decided.
- [Zax arrays and slices](../../language/arrays-and-slices.md), when element
  transfer into an array is reached.
- [Zax qualifiers](../../language/qualifiers.md), when a property interacts
  with mutability or access.
- [Raw mutability-indexed type families](../raw/mutability-indexed-type-families.md)
  and [raw safety boundaries](../raw/safety.md), when generated copy shapes or
  transfer-contract blame are reached.

### Audit-only

- Archived numbered work, only when a provenance question cannot be answered
  from current owners or raw input. The transfer-stances review is
  `013-transfer-stances-and-source-state.md`; the variadics review is
  `037-variadic-arguments.md`.

## Working record

Not started.
