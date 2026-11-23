# 040: Element and contained stances

| Field | Value |
| --- | --- |
| Status | Active working material / non-normative / not started |
| Work Item | `040` |
| Created | 2026-11-22 |
| Owns | The bounded review of whether, and how, array elements, slices, optional references, optionals' `move source` and `last source` forms, and lambda reference captures can state the stance of the value they reach |
| Does Not Own | Current transfer-stance, array, slice, optional, lambda, or pointer behavior, which remain with their owners until a promotion is authorized; non-vacant (`present`) pointers; the decision itself, which belongs to the language maintainer |

## Non-authority notice

This file is a collaborative working record. Existing documentation, candidate
syntax, examples, and later aligned findings remain non-authoritative until a
separately discussed, aligned, and explicitly authorized promotion incorporates
them into their lasting owners.

## Fixed initiating input

This section records the aligned information known when work item `040` was
created. It is intentionally incomplete and must not be rewritten as work
develops.

### Initiating concern

Work item 039 gave pointers a stance on the value they point to, and gave
optionals two stances: one for the contained value and one for the optional
itself. Arrays, slices, optional references, and lambda reference captures
still reach a second value without a way to state that value's stance. Should
each be able to, and how?

### Starting input

These are the follow-ups 039 deferred to this work item:

- **A stance on each array element.** Elements are governed by their type
  today. A stance before `[` could apply to each element, as in
  `messages : MyMessage move [4]`, so that `keep(messages[0])` is an error.
- **Slice parameters.** A slice parameter is a borrower that writes no stance
  for the elements it shows, as in `values : MyMessage readonly loan []`.
- **Optionals' `move source` and `last source` forms**, which may be explained
  as the optional's owner writing a stance on the contained value.
- **Optional references** as parameters, results, or members, such as
  `MyValue loan & ?`, which would need a stance as borrowers that stand alone.
  `language/optional-values.md` still has unmarked optional-reference examples,
  left as they are on purpose during 039: the type-composition examples
  (`MyValue readonly & ?`, `MyValue readonly & * writable`), the composed forms
  (`MyValue & * & * &`, `MyValue & ? * & ?`), the
  `view : MyValue readonly & ?` declaration under "Optional references", and
  the local `inner : MyValue? & = nested.`
- **Lambda reference captures**, which should follow the same rule as other
  references bound to one visible value (they take a stance from that value,
  limited to stances that leave the owner's value unchanged); the exact capture
  spelling is unsettled.

### Starting boundaries

- The 039 model is current design: a stance says what may be built from a
  value; owners and borrowers; `via`; pinned values; one stance per layer,
  written last on it; and the optional's two stances. A conflict with it is
  recorded as a finding for discussion, not resolved by silently changing it.
- Non-vacant (`present`) pointers are out of scope. They are a separate concern
  with their own raw input, `project/raw/present-pointers.md`, and trigger.
  **Stated overlap:** that raw input's sixth open question asks whether
  `present` should extend to optionals. If 040 settles optional references, it
  notes the effect on that question rather than designing `present`.

### Initial stopping guidance

First present the candidate model for each item, with code and a
recommendation, the most important uncertainty, and the decisions the
maintainer must make. Do not design every combination in full before the
direction is aligned.

Creating this work item does not authorize analysis. A later assignment
begins it.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs focused reading,
  authority, promotion, deferral, closure, and the writing rules.
- [Zax transfer stances](../../language/transfer-stances.md): "What a stance
  says", "Owners and borrowers", "Declaring stances", "Members", "Optional
  values", and "Arrays and slices" - the model this work extends. "Members"
  matters because elements and contained values behave much like members:
  pinning a whole value does not pin its members, and writable access decides
  whether a member can be taken through a borrower.
- [Zax arrays and slices](../../language/arrays-and-slices.md): slices, array
  and slice parameters, and "Copy, deep, move, and last".
- [Zax optional values](../../language/optional-values.md): "Move and last
  source adapters", "Optional references", and "Stances on an optional and its
  contained value".
- [Zax lambdas and callable composition](../../language/lambdas-and-callable-composition.md):
  captures.

### Consequence-driven

Read only the smallest relevant sections when the concern reaches them:

- [Zax qualifiers](../../language/qualifiers.md), "Canonical ordering", when
  stance position on an element or optional layer is reached.
- [Zax function invocation](../../language/function-invocation.md), when
  parameter selection for slice or optional-reference parameters is reached.
- [Zax iteration](../../language/iteration.md), when loop bindings over
  elements are reached.
- [Zax generics](../../language/generics.md), when type parameters that may be
  arrays, slices, or optionals are reached.
- [Zax pointers, allocation, and arenas](../../language/pointers-and-arenas.md),
  when element pointers or pointers to optionals are reached.

### Audit-only

- Archived numbered work, only when a provenance question cannot be answered
  from current owners or raw input. The pointer-stance review is
  `project/archive/work/039-pointer-stances.md`.

## Working record

Not started.
