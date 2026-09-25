# 034: `partial`

| Field | Value |
| --- | --- |
| Status | Active working material / non-normative / awaiting assignment |
| Work Item | `034` |
| Created | 2026-09-25 |
| Owns | The bounded review of `partial`: reopening a non-sealed type to add declarations, as defined below |
| Does Not Own | Async context propagation; the ordering of words after `+++`; unsafe-category syntax; the decision itself, which belongs to the language maintainer |

## Non-authority notice

This file is a collaborative working record. Existing documentation, candidate
syntax, examples, and later aligned findings remain non-authoritative until a
separately discussed, aligned, and explicitly authorized promotion incorporates
them into their lasting owners.

## Fixed initiating input

This section records the aligned information known when work item `034` was
created. It is intentionally incomplete and must not be rewritten as work
develops.

### Initiating concern

What should `partial` mean in Zax, and how can it be designed so that the
capability it opens stays bounded and predictable?

A partial reopens a type that is not sealed and adds declarations to it:
stored members, fixed or varying functions, operators, and nested types or
enums. The maintainer's illustrative starting shape:

```zax
// Illustrative syntax from the maintainer's note.
MyPartialInteger :: partial Integer {
  operator binary '+' final : (result : ResultType)(rhs : MyType) = {
    //...
  }
}
```

### Motivating pressure

- **Receivers that cannot be extended.** Every user-defined operator belongs to
  its receiver type, so `myType + myInteger` can be declared on `MyType` but
  `myInteger + myType` cannot be declared anywhere. A partial on `Integer`
  could add that operator as if `Integer` had declared it.
- **The execution context.** Every thread receives a `___` context instance
  holding built-in defaults such as the allocation arena. The context is
  intentionally not sealed: programmers need to add their own functions and
  stored members to it, including per-thread storage. This is a hard need, not
  a convenience.

### Starting input

The maintainer's newest thinking is
[`project/raw/maintainer-notes/partial.md`](../raw/maintainer-notes/partial.md).
It replaces the legacy framing where they differ. The legacy
[`project/raw/partial-types.md`](../raw/partial-types.md) remains evidence for
insights and constraints but is not authoritative over the concept or the
maintainer's note.

The note identifies these complexities:

- conflicting names between partials, and names that become ambiguous after
  code that used them was already compiled and linked;
- added stored members, which block `size of` until the type is complete;
- construction, destruction, assignment, and replacement that cannot know about
  storage added later;
- ordering across modules; and
- whether one module's additions are visible to another.

It proposes these starting points, all open to better syntax and spelling:

- **Order-sensitive visibility through `forward`.** A use considers only
  partials already defined, or explicitly forwarded within the visible nesting,
  when it compiles. Later partials do not change what earlier code sees.
- **Graded sealing,** for example `seal storage`, `seal callable`, and
  `seal once`, with bare `seal` sealing everything.
- **Hooks through an abstract contract.** A type calls declared hook points in
  its own lifecycle and operations; partials that explicitly implement the
  contract are linked to them; with no partials, the calls compile away. Hooks
  return nothing, and their order is defined but must not be relied upon.
- **Delayed `size of`,** with an earlier opt-in seal, and likely denying
  `partial` on types that must be complete for compile-time work.
- **A named type for the context.** `___` needs a system type name within a
  namespace so a partial can target it. The name can be chosen now, with the
  namespace left as future pressure, as for `OpaqueObserver`.

### Deliberately unresolved framing

- **Order independence versus order-sensitive visibility.** The legacy input
  repeatedly requires that import, declaration, and build order never resolve
  conflicts or select a surface. The maintainer's note proposes that
  compilation order and `forward` visibility decide which partials a use sees.
  Reconciling or choosing between these is central to this item.
- Who may declare a partial on intrinsic, identity, foreign-owned, or
  language-owned types, and whether an owner must opt in.
- Whether and how partials may add stored members, and how those members take
  part in construction, destruction, copy, assignment, and replacement.
- How partials interact with protected signatures, composition, enums, unions,
  variants, and structural shape.
- The context type's name.

### Starting boundaries

- Do not design async context propagation. The note marks it as a separate
  concern.
- The ordering of words after `+++` and unsafe-category syntax are separate
  deferred pressures, captured in the
  [cross-cutting audit](../raw/cross-cutting-audit.md) and the
  [analysis-control input](../raw/analysis-controls.md).

### Initial stopping guidance

Creating and routing this work item does not authorize analysis. A later
assignment begins it.

Do not promote findings, change current owners, archive this work item, or
create work item `035` without the separately required discussion, alignment,
and authorization.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs focused reading,
  authority, promotion, deferral, and closure.
- [Maintainer notes on `partial`](../raw/maintainer-notes/partial.md) - the
  primary input and the maintainer's current direction.
- [Raw partial type extensions](../raw/partial-types.md) - legacy constraints
  and pressures to test against the note: protected signatures, intrinsic and
  identity authority, composition merges, candidate relatching, the
  execution-context shape, and enums.
- [Zax execution context](../../language/execution-context.md), the
  [one context shape](../../language/execution-context.md#one-context-shape-one-current-instance-per-thread)
  section - the current context model that partials must extend.
- [Zax type definitions](../../language/type-definitions.md), the
  [what a type body can contribute](../../language/type-definitions.md#what-a-type-body-can-contribute),
  [forward anchors](../../language/type-definitions.md#forward-anchors), and
  [partial and generic completion boundaries](../../language/type-definitions.md#partial-and-generic-completion-boundaries)
  sections - what a partial adds to and when a type is complete.
- [Zax declarations and bindings](../../language/declarations-and-bindings.md#forward-anchors),
  forward anchors, and
  [Zax namespaces and modules](../../language/namespaces-and-modules.md), the
  [visibility and export baseline](../../language/namespaces-and-modules.md#visibility-and-export-baseline)
  and [forward module and namespace roots](../../language/namespaces-and-modules.md#forward-module-and-namespace-roots)
  sections - the `forward` and visibility rules the note's proposal depends on.

### Consequence-driven

Read only the smallest relevant sections when the concern reaches them:

- [Zax operators](../../language/operators.md) and
  [Zax operator phrases](../../language/operator-phrases.md) for receiver
  ownership, discovery, and protected signatures.
- [Zax composition](../../language/composition.md#abstract-roles-and-explicit-fulfillment)
  for `abstract` roles, fulfillment, and `own`, which the hook proposal uses.
- [Zax construction, replacement, and destruction](../../language/construction-and-destruction.md)
  when added storage meets lifecycle operations.
- [Zax structural shapes and compatibility](../../language/structural-shapes-and-compatibility.md)
  and [Zax arrays and slices](../../language/arrays-and-slices.md) for layout
  and `size of`.
- [Zax identity types](../../language/identity-types.md),
  [Zax integers](../../language/integers.md), and
  [Zax enums](../../language/enums.md) for intrinsic, identity, and enum
  extension.
- [Zax function invocation](../../language/function-invocation.md#candidate-selection)
  for candidate relatching and ambiguity.
- [Zax conversions and casts](../../language/casting.md#your-types-and-the-languages-scalars)
  for `as` on scalar receivers.
- Raw inputs on [compile-time execution](../raw/compile-time-execution.md),
  [reflection](../raw/reflection.md),
  [global and once lifetimes](../raw/global-and-once-lifetimes.md),
  [library surface and namespaces](../raw/library-surface-and-namespaces.md),
  and [export and visibility directives](../raw/export-and-visibility-directives.md)
  when their concerns are reached.

### Audit-only

- Archived numbered work, only when a provenance question cannot be answered
  from current owners or raw input.

## Working record

Not started.
