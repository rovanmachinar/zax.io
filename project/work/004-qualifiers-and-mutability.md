# 004: Qualifiers, mutability, and access capabilities

| Field | Value |
| --- | --- |
| Status | Active working material / non-normative |
| Work Item | `004` |
| Created | 2026-08-21 |
| Owns | The bounded review defined by the fixed initiating input |
| Does Not Own | Accepted detailed qualifier semantics, formal grammar, or compiler behavior |

## Non-authority notice

This file is a collaborative working record. Existing statements, new
observations, candidate wording, and aligned findings remain non-authoritative
until a separately discussed, aligned, and explicitly authorized promotion
incorporates them into their lasting owner documents.

## Fixed initiating input

This section records the information known and aligned when work item `004` was
created. It is intentionally incomplete rather than exhaustive and must not be
rewritten as the work develops.

### Initiating concern

Develop a coherent human-developer-facing model for the independent qualifier
axes established by [Zax declarations and bindings](../../language/declarations-and-bindings.md):

- place replacement through `final` and `varying`;
- underlying value change through `mutable` and `immutable`; and
- access capability through `writable` and `readonly`.

Determine how those axes combine, default, propagate, narrow, conflict, and
participate in declaration and call boundaries.

### Motivating pressure

Work item `003` established the qualifier axes and terminology while explicitly
deferring their complete semantics, syntax, conversions, propagation, and
interactions.

The terminology now appears throughout legacy documentation, but detailed
mutability and qualifier behavior remains unreviewed. Functions, references,
pointers, captures, operator selection, move and copy, concurrency, construction,
lifetime, and structural typing all require stable qualifier constraints.

Reviewing qualifiers next prevents mechanically migrated legacy examples from
being mistaken for accepted detailed behavior.

### Known assumptions

- The current [language vision](../../language/vision.md) is accepted
  foundational direction.
- [Zax declarations and bindings](../../language/declarations-and-bindings.md)
  is accepted conceptual design.
- The `final`/`varying`, `mutable`/`immutable`, and
  `writable`/`readonly` axes and names retain their accepted meanings.
- `final` qualifies a storage place and does not imply immutable contents.
- `readonly` constrains one access path and does not imply that no other access
  can mutate the value.
- `immutable` is a stronger underlying-value guarantee.
- Ordinary views may reduce capabilities but may not increase them.
- References and captures preserve or reduce source-place capabilities.
- Move and copy are contextual operations rather than persistent value
  qualifiers.
- Exact reference rebinding, ownership, move, and copy behavior remains later
  work unless an immediate contradiction requires a constraint.
- Legacy `pliable` and `unpliable` behavior is evidence rather than accepted
  design.

### Known inclusions

- Defaults and explicit spellings for all three axes.
- Qualifier attachment to variables, direct values, references, parameters,
  results, captures, and members at the depth needed for a coherent model.
- Compatible repetition and conflicting qualification.
- Safe capability-reduction rules.
- Whether and how writable access can be recovered after a readonly view ends.
- Function and receiver effects at the conceptual depth required by the
  qualifier model.
- Contained-member and nested-value implications.
- Reference and capture propagation constraints.
- Interaction between final places and movement out of a value.
- Disposition of `pliable` and `unpliable`.
- Canonical formatting and immediate diagnostics.
- Constraints on functions, pointers, operators, concurrency, construction,
  lifetime, and structural typing.
- Candidate lasting owner and documentation boundaries.

### Known starting boundaries

- Complete pointer, reference, and aliasing design.
- Complete lifetime and ownership strategies.
- Complete move/copy selection and moved-from states.
- Full function declaration, capture, overload, and invocation behavior.
- Complete operator ranking and generated candidates.
- Complete concurrency transfer guarantees.
- Structural identity, equivalence, layout, conversion, and reflection.
- Formal grammar, compiler data structures, lowering, or implementation.

These areas are not part of the initiating question and need not be explored to
begin. They do not prohibit following, recording, or dispositioning material
consequences.

### Intentionally unresolved framing

- Whether `writable` must be written explicitly or is normally represented by
  omission.
- Whether immutability is recursively deep by default.
- Whether a final place can be moved from when the moved-from value remains live.
- Whether readonly access permits hidden caches or bookkeeping.
- Whether `pliable` expresses a necessary capability or a legacy workaround.
- Whether every qualifier is part of a type-use expression, a place view, a
  declaration modifier, or some combination.
- Whether qualifier ordering is semantically meaningful or only canonical
  formatting.
- Which capability relations are implicit, explicit, unsafe, or impossible.

### Initial stopping guidance

Stop when the work has:

- established a coherent model for all three axes;
- established defaults and attachment;
- defined safe capability-reduction relationships;
- dispositioned `pliable` and `unpliable`;
- constrained references, captures, functions, members, operators, move/copy,
  and concurrency without designing them completely;
- identified programmer-visible diagnostics and formatting;
- identified structural-typing and lifetime consequences;
- identified a plausible lasting owner and its boundaries; and
- gathered enough information for a pre-promotion documentation fit dry run.

Do not promote findings, update language examples broadly, archive this work
item, or begin work item `005` without the separately required discussion,
alignment, and authorization.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs ownership,
  focused reading, numbered work, and promotion.
- [Language vision](../../language/vision.md) - provides the accepted explicit
  policy, visible cost, safety, readability, and progressive-complexity
  principles.
- [Zax declarations and bindings](../../language/declarations-and-bindings.md) -
  provides the accepted qualifier axes and declaration constraints.
- [Mutability](../../mutable.md) - preserves the primary legacy evidence for
  qualifier combinations, pliability, and function effects.
- Focused qualifier-preservation and conversion material in
  [casting](../../casting.md) - preserves legacy evidence for capability
  conversion.

### Consequence-driven

- Read focused [function](../../functions.md) sections when receiver or function
  qualification becomes necessary.
- Read focused [pointer and reference](../../pointers.md) material when
  qualifier propagation through indirection becomes necessary.
- Read focused [compiler directive](../../compiler-directives.md) material when
  default qualifier policy must be tested.
- Read focused [concurrency](../../concurrency.md) material when immutable or
  readonly guarantees cross threads.
- Read the [raw structural-typing input](../raw/structural-typing.md) only when
  qualifier participation in structural identity becomes material.
- Read construction, lifetime, move/copy, operator, or safety material only when
  an immediate qualifier contradiction requires it.
- Propose an update or new raw input when a material finding must be preserved
  for future work. Discussion, alignment, and explicit edit authorization still
  apply.
- Inspect focused Git history only when the origin or former purpose of an
  unusual qualifier choice becomes consequential.
- Expand beyond this list whenever discussion or a material finding establishes
  a concrete need. State why the additional reading is relevant and keep it
  focused.

Material not listed in this reading scope is neither required nor prohibited. It
remains available when discussion or a concrete consequence makes it relevant.

### Audit-only

- `project/archive/`, including work items `001` through `003`.

Audit-only material requires direct language-maintainer instruction or a
specific audit or recovery need.

## Working record

Not started. The agent creating this work item stops here without analyzing its
content.
