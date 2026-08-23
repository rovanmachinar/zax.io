# 005: Construction, destruction, and replacement

| Field | Value |
| --- | --- |
| Status | Active working material / non-normative |
| Work Item | `005` |
| Created | 2026-08-22 |
| Owns | The bounded review defined by the fixed initiating input |
| Does Not Own | Accepted complete construction, lifetime, ownership, formal grammar, or compiler behavior |

## Non-authority notice

This file is a collaborative working record. Existing statements, new
observations, candidate wording, and aligned findings remain non-authoritative
until a separately discussed, aligned, and explicitly authorized promotion
incorporates them into their lasting owner documents.

## Fixed initiating input

This section records the information known and aligned when work item `005` was
created. It is intentionally incomplete rather than exhaustive and must not be
rewritten as the work develops.

### Initiating concern

Develop a coherent human-developer-facing model for how Zax value lifetimes:

1. begin through initialization and construction;
2. transition through reconstructive replacement; and
3. end through destruction.

Define ordinary constructors, replacement constructors, destructors,
construction authority, same-storage resource retention, generated behavior,
failure boundaries, and immediate qualifier interactions.

### Motivating pressure

Work item `004` established varying places containing successive immutable value
lifetimes, generated reconstructive `=`, contextual `replacement +++`, terminal
destruction authority, and a detailed set of unresolved replacement mechanics.

Those accepted qualifier constraints depend on construction and destruction
behavior that is not yet technically complete. The legacy constructor corpus
also contains substantial proposals for constructor selection, generated
constructors, arguments, member initialization, ordering, move/copy,
destruction, and failure behavior.

Reviewing these concerns next tests whether reconstructive replacement and
ordinary construction can form one coherent programmer model without silently
committing to a complete ownership or lifetime strategy.

### Known assumptions

- The current [language vision](../../language/vision.md) is accepted
  foundational direction.
- [Zax declarations and bindings](../../language/declarations-and-bindings.md)
  is accepted conceptual design.
- [Zax qualifiers](../../language/qualifiers.md) is accepted conceptual design.
- Ordinary constructor declarations use `+++`.
- Destructor declarations use `---`.
- Replacement constructors use contextual `replacement +++`.
- `replacement` remains an ordinary identifier outside that narrow constructor
  declaration context.
- Generated reconstructive `=` requires an existing varying place and a
  writable current path.
- Reconstructive replacement uses the destination's existing storage.
- A replacement constructor receives `_` with the previous representation and
  resources and must establish a complete valid new lifetime before normal
  return.
- Replacement-constructor right-hand operands use ordinary declared parameter
  types, qualifications, and selection.
- Constructors and replacement constructors do not return results.
- Construction authority establishes final and immutable state without
  `unsafe pliable`.
- Destruction receives terminal mutable and writable authority.
- Panic is often terminal. Complete recoverable-panic behavior remains
  unresolved.
- Exact move/copy, `last`, alias, pointer, async, concurrency, ownership, and
  lifetime behavior remains later work unless an immediate construction
  contradiction requires a constraint.
- There is no language implementation in this repository.

### Known inclusions

- Default initialization, direct initialization, and constructor invocation
  boundaries.
- Ordinary constructor declarations and selection.
- Multiple constructor arguments and their mapping.
- Named constructor arguments and their interaction with declaration syntax.
- Default arguments and omission where construction requires them.
- Constructor overloading, viable candidates, and immediate ambiguity rules.
- Generated, defaulted, disabled, and programmer-declared constructors.
- Member initialization order and construction authority.
- Complete-instance activation of final and immutable guarantees.
- Contextual `replacement +++` recognition and selection.
- Same-storage replacement and the transitional receiver operand.
- Member retention, destruction, move, copy, and reconstruction during
  replacement.
- Right-hand operand evaluation and aliasing with the destination.
- Destructor declaration, selection, ordering, and terminal authority.
- Partial construction and partial destruction at the conceptual depth needed
  for a coherent model.
- Panic and non-recoverable failure boundaries.
- Raw and typed pointers into constructing, replacing, or destroying storage.
- Immediate interactions with `last`, move/copy, allocation, callbacks,
  reentrancy, async, and concurrency.
- Programmer-visible costs, diagnostics, and canonical formatting.
- Candidate lasting owners and documentation boundaries.
- Preservation of useful but premature argument, lifetime, operator, or safety
  proposals as indexed raw input when their lasting owner is not yet known.

### Known starting boundaries

- Complete ownership and selectable lifetime strategies.
- Complete pointer/reference grammar and rebinding.
- Complete move/copy overload selection and moved-from states.
- Complete function declaration, named/default argument, generic, and overload
  behavior outside construction pressure.
- Complete generated-operator ranking and arbitrary operator semantics.
- Complete allocation-strategy and allocator design.
- Complete panic recovery and general error handling.
- Complete callback, reentrancy, async, and concurrency design.
- Mutability-indexed type-family identity and representation.
- Structural identity, equivalence, layout, reflection, and anonymization.
- Formal grammar, compiler data structures, lowering, or implementation.

These areas are not part of the initiating question and need not be explored to
begin. They do not prohibit following, recording, or dispositioning material
consequences.

### Intentionally unresolved framing

- Whether ordinary and replacement constructors share all declaration and
  selection rules.
- How multiple and named arguments map to constructor parameters.
- Whether constructor default arguments use ordinary function defaults or a
  construction-specific mechanism.
- Which constructors are generated, defaulted, disabled, or reflected.
- Whether a replacement constructor must mention every member.
- Whether untouched member lifetimes continue across enclosing replacement.
- How self-replacement and right-hand aliases are handled.
- How raw interior pointers survive or fail across replacement.
- What fallback exists when no replacement constructor is declared.
- How generated reconstructive `=` ranks against domain-specific `=`.
- Which panic conditions are recoverable and what partial cleanup means.
- Whether replacement constructors may call back, reenter, suspend, or execute
  concurrently.
- Which detailed findings belong in current construction documentation versus
  future lifetime, operator, function, pointer, safety, async, or concurrency
  owners.

### Initial stopping guidance

Stop when the work has:

- established a coherent model for ordinary construction, replacement
  construction, and destruction;
- established constructor declaration, argument, and immediate selection
  behavior;
- established member-state and resource-retention principles during
  replacement;
- established construction completion and destruction boundaries;
- constrained generated behavior, move/copy, `last`, aliases, pointers, panic,
  callbacks, async, and concurrency without designing them completely;
- identified programmer-visible costs, diagnostics, and formatting;
- dispositioned the indexed replacement-construction input;
- preserved useful deferred argument or lifetime material in the appropriate
  indexed input;
- identified plausible lasting owners and their boundaries; and
- gathered enough information for a pre-promotion documentation fit dry run.

Do not promote findings, update language examples broadly, archive this work
item, or begin work item `006` without the separately required discussion,
alignment, and authorization.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs ownership,
  focused reading, numbered work, and promotion.
- [Language vision](../../language/vision.md) - provides explicit-cost,
  progressive-complexity, safety, readability, and data-oriented principles.
- [Zax declarations and bindings](../../language/declarations-and-bindings.md) -
  provides accepted initialization, assignment, generated replacement,
  incomplete-definition, and declaration constraints.
- [Zax qualifiers](../../language/qualifiers.md) - provides accepted construction
  activation, reconstructive replacement, terminal destruction, place/access,
  and unsafe boundaries.
- [Legacy constructors and destructors](../../ctor-dtor.md) - preserves the
  primary legacy evidence for constructor/destructor forms, selection,
  generation, arguments, ordering, and failure behavior.
- [Raw replacement-construction input](../raw/replacement-construction.md) -
  preserves the detailed future-work questions exposed by reconstructive
  replacement.

### Consequence-driven

- Read focused [type-definition](../../type-definition.md) material when member
  initialization, generated constructors, or incomplete types become necessary.
- Read focused [function](../../functions.md) material when constructor
  parameters, multiple/named/default arguments, overload selection, callbacks,
  or captures become necessary.
- Read focused [operator](../../operator.md) material when generated
  reconstructive `=` viability or ranking becomes necessary.
- Read focused [pointer and reference](../../pointers.md) material and the
  [raw lifetime-strategies input](../raw/lifetimes.md) when aliases, interior
  pointers, lifetime transitions, or `last` become necessary.
- Read focused allocation material when construction requires an immediate
  storage or allocator constraint.
- Read focused panic, safety, async, or concurrency material only when a
  construction finding crosses those boundaries.
- Read the [raw mutability-indexed type-family input](../raw/mutability-indexed-type-families.md)
  only when variant construction or cross-variant replacement becomes material.
- Propose an update or new raw input when useful multiple/named/default argument,
  lifetime, operator, or safety material must be preserved for later work.
  Discussion, alignment, and explicit edit authorization still apply.
- Inspect focused Git history only when the origin or former purpose of an
  constructor choice becomes consequential.
- Expand beyond this list whenever discussion or a material finding establishes
  a concrete need. State why the additional reading is relevant and keep it
  focused.

Material not listed in this reading scope is neither required nor prohibited. It
remains available when discussion or a concrete consequence makes it relevant.

### Audit-only

- `project/archive/`, including work items `001` through `004`.

Audit-only material requires direct language-maintainer instruction or a
specific audit or recovery need.

## Working record

Not started. The agent creating this work item stops here without analyzing its
content.
