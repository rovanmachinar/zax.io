# 039: Pointer stances

| Field | Value |
| --- | --- |
| Status | Active working material / non-normative / not started |
| Work Item | `039` |
| Created | 2026-11-16 |
| Owns | The bounded review of whether, and how, a stance can describe what may happen to a value reached through a pointer, an optional, or an array, separately from the stance of the pointer, optional, or array itself |
| Does Not Own | Current transfer-stance, pointer, optional, array, qualifier, or invocation behavior, which remain with their owners until a promotion is authorized; the decision itself, which belongs to the language maintainer |

## Non-authority notice

This file is a collaborative working record. Existing documentation, candidate
syntax, examples, and later aligned findings remain non-authoritative until a
separately discussed, aligned, and explicitly authorized promotion incorporates
them into their lasting owners.

## Fixed initiating input

This section records the aligned information known when work item `039` was
created. It is intentionally incomplete and must not be rewritten as work
develops.

### Initiating concern

A transfer stance applies to one whole value. A pointer's whole value is the
pointer itself, its address and any ownership it holds, so nothing in a
pointer type says what may happen to the value it points to. Should a stance
be able to describe the pointed-to value separately from the pointer, through
any nesting of `*` and `&`, and if so, how? The same question applies to an
optional and the value it contains, and to an array and its elements.

### Motivating pressure

- **A loan cannot travel through a pointer.** Making a raw pointer from a loan
  is an error outside `unsafe`, because the pointer cannot carry the loan's
  promise that the whole value is not copied or taken. A data-oriented
  language that works through pointers, and interoperates with C, needs to
  lend through a pointer without `unsafe`.
- **`_` already does it.** The receiver `_` is pointer-shaped and carries a
  loan in a function with no receiver stance, so the language already has a
  pointer-shaped path whose pointed-to value has a stance. Pointer types cannot
  express the same thing.
- **Optionals and arrays have the same shape.** The maintainer is equally
  concerned with optionals and arrays. Each holds or reaches a second value:
  the optional's contained value, the array's elements. Whether that inner
  value needs its own stance, separate from the optional or array, belongs to
  the same design.

### Why the whole-value model has been outgrown

This explanation was requested by the maintainer as part of the initiating
input. It is the agent's framing for 039 to test, not an aligned conclusion.

**The original model.** A stance was something written at a use, such as
`as move`: a one-time instruction for one complete value at one call.
`qualifiers.md` recorded the consequence: transfer stance is not a fourth
per-layer qualifier axis, because `copy`, `deep`, `move`, and `last` "describe
how one complete source or receiver is offered to a consumer". A one-time
instruction has no reason to be part of a type, so a slot for it on each layer
made no sense.

**What changed in work item 038.** Stances became lasting properties of
places and paths, not only one-time instructions:

- a stance declared on a variable is a requirement on every later transfer
  from it;
- `loan` is the stance of a reference, enforced on everything made from it;
- a reference's stance already describes the value it refers to, not the
  reference: `& loan`, `& move`, and `& last` say what may happen to the
  referent;
- members follow their own declared stance, independent of the value that
  contains them;
- a reference or receiver passes the whole value on with at most the stance it
  was given;
- a `self` result carries the caller's variable with the stance written on the
  call;
- `_` is pointer-shaped and carries a loan.

So a reference already has a stance on its pointed-to layer, while a pointer
has a stance only on its own layer. Members already have stances independent
of the containing value. The rule "a stance applies only to a whole value" now
describes the stance written at a use, but not the stances the language
actually tracks.

**Why independent stances may be better.** A pointer, an optional, and an
array each involve two things that can be treated differently: the container
or pointer itself, and the value inside or pointed to. One stance cannot say
both "you may take ownership of this pointer" and "you may only borrow what it
points to", or "you may move this array" and "you may not take its elements".
Giving the pointed-to or contained value its own stance could:

- let a loan travel through a pointer, an optional, or an array without
  `unsafe`;
- explain `_` as a pointer whose pointed-to value is on loan, instead of a
  special case;
- make references and pointers uniform, if a reference's stance is simply the
  stance of its pointed-to layer;
- help with allocation, for example lending an allocation without passing its
  ownership.

The costs to weigh include syntax on each layer, defaults that keep existing
pointer code working, generic code over types that may or may not be
pointers, combinations with ownership kinds that make no sense, interop where
nothing can be enforced, and diagnostics that say which layer's stance a use
violates.

### Starting input

- [Raw pointer stances](../raw/pointer-stances.md): the concern, the
  motivating examples, the history of the per-layer decision, the agent's
  candidate direction (a stance on the pointed-to layer, such as
  `MyValue loan *`, unrestricted when unmarked), the questions the work must
  answer, and the captures from the 038 review: pointer allocation, array
  elements, optional values, and who frees a `copy` allocating result.
- The maintainer has no preferred solution. The agent is asked to produce a
  range of options, from keeping the current rule to independent stances on
  every layer, each with examples, advice, a recommendation, and the
  feasibility pressure it places on an eventual implementation.

### Starting boundaries

- In scope: stances on the pointed-to layer of raw, `unique`, `strong`, and
  `weak` pointers; how stances attach through nested `*` and `&`; whether a
  reference's stance is the same thing; the receiver `_`; optional contained
  values; array elements; conversions that add or drop such a stance;
  diagnostics; generic code.
- The meanings of the five stances (`loan copy move last deep`), and the 038
  rules for references, receivers, members, results, and `self` results, are
  current design. A conflict with them is recorded as a finding for
  discussion, not resolved by silently changing them.
- Pointer ownership kinds, allocation, and arenas are reviewed only as far as
  stances touch them.
- Until 039 decides otherwise, making a raw pointer from a loan, including
  passing a loan `_` to a raw pointer parameter, remains an error outside
  `unsafe`.

### Initial stopping guidance

First present the candidate programmer model, the options with code and a
recommendation, the most important uncertainty, and the decisions the
maintainer must make. Do not design every combination of layers in full
before the direction is aligned.

Creating this work item does not authorize analysis. A later assignment
begins it.

Do not promote findings, change current owners, archive this work item, or
create work item `040` without the separately required discussion, alignment,
and authorization.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs focused reading,
  authority, promotion, deferral, closure, and the writing rules.
- [Raw pointer stances](../raw/pointer-stances.md) - the preserved concern,
  examples, candidate direction, and questions.
- [Zax transfer stances](../../language/transfer-stances.md), especially
  "Loan", "References pass on at most their stance", "Receiver stance",
  "Passing the receiver on", and "Pointer ownership transitions" - the current
  model this work extends.
- [Zax qualifiers](../../language/qualifiers.md), how qualifiers attach to
  each level of indirection - the existing per-layer precedent.
- [Zax pointers, allocation, and arenas](../../language/pointers-and-arenas.md),
  "Pointer ownership roles", the raw-pointer sections, and
  `OpaqueReferenceObserver` - what pointers mean today.

### Consequence-driven

Read only the smallest relevant sections when the concern reaches them:

- [Zax optional values](../../language/optional-values.md), when contained
  values are reached.
- [Zax arrays and slices](../../language/arrays-and-slices.md), when elements
  or slices are reached.
- [Zax lifetimes and references](../../language/lifetimes-and-references.md),
  when reference stances or returned references are compared with pointers.
- [Zax Nothing instances](../../language/nothing-instances.md), when the
  pointer shape of `_` is reached.
- [Zax generics](../../language/generics.md), when type parameters that may be
  pointers are reached.
- [Zax function invocation](../../language/function-invocation.md), when
  parameter selection or results through pointers are reached.
- [Raw mutability-indexed type families](../raw/mutability-indexed-type-families.md),
  when per-layer qualifiers and type-declared qualifier allowances interact.

### Audit-only

- Archived numbered work, only when a provenance question cannot be answered
  from current owners or raw input. The transfer-stances reviews are
  `013-transfer-stances-and-source-state.md` and
  `038-terminal-transfer-intent.md`; the reference and pointer review is
  `014-reference-origins-and-lifetime-strategies.md`.

## Working record

Not started.

## Dispositions and promotion dry run
