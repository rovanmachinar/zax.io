# 017: Each iteration and compiler-known traversal

| Field | Value |
| --- | --- |
| Status | Active working material / non-normative |
| Work Item | `017` |
| Created | 2026-09-07 |
| Owns | The bounded review defined by the fixed initiating input |
| Does Not Own | First-class iterator concepts, complete generics, generators, coroutines, async or parallel iteration, or complete collection and range design |

## Non-authority notice

This file is a collaborative working record. Existing statements, new
observations, candidate wording, and aligned findings remain non-authoritative
until a separately discussed, aligned, and explicitly authorized promotion
incorporates them into their lasting owners.

## Fixed initiating input

This section records the information aligned when work item `017` was created.
It is intentionally incomplete and must not be rewritten as work develops.

### Initiating concern

Develop a coherent human-developer-facing model for `each` and compiler-known
traversal without prematurely requiring first-class iterator values or a
concept/generic constraint system.

The review should establish:

1. the role and source shape of `each`;
2. the distinction, if retained, between `each in` and `each from`;
3. initialization, binding, condition, and body structure;
4. traversal order and termination;
5. element value, place, reference, qualification, and transfer behavior;
6. scope, lifetime, cleanup, and control-transfer behavior;
7. compiler-known traversal of enum declarations and case-insensitive matches;
8. the bounded interaction with arrays, ranges, fields, and multi-string flags
   conversion;
9. compile-time structural traversal versus runtime value traversal;
10. costs, diagnostics, and source stability; and
11. a precise handoff to later concept-dependent iterator protocols.

### Motivating pressure

Current enum design needs member and case-insensitive-match traversal in
declaration order. Flags conversion also needs a future way to consume several
strings atomically. Legacy flow material proposes `each in` and `each from`.

Zax cannot yet express a true general iterator protocol because concepts and
generic constraints are not current. The immediate task is therefore to design
direct `each` behavior over compiler-known sources without disguising a local
compiler facility as a completed user-extensible iterator abstraction.

### Known assumptions

- First-class iterator values and general iterable parameters remain deferred
  until concepts or an equivalent constraint system can express their
  requirements.
- A direct `each` construct may operate over language-known sources before that
  abstraction exists.
- Existing core flow rules for effective bodies, scopes, labels, transfer
  targets, unwinding, and completion remain constraints.
- Enum member traversal preserves declaration order and duplicate-valued
  aliases.
- Case-insensitive enum-match traversal visits each distinct matching value once
  in first matching declaration order.
- No compiler implementation exists in this repository.

### Known inclusions

- `each` statement mental model and ordinary use.
- `in` and `from` source roles.
- Header initialization and binding scope.
- Element binding type, qualification, and lifetime.
- Forward progress, exhaustion, and termination.
- `break`, `continue`, `next`, `return`, panic, and cleanup.
- Compiler-known enum member and match traversal.
- Concrete pressure from arrays and ranges where required to make `each`
  coherent.
- Compile-time reprocessing pressure for heterogeneous structural traversal.
- Multiple-string flags conversion pressure without assuming a general iterable
  parameter.
- Costs, diagnostics, formatting, and source stability.
- Lasting documentation ownership and legacy disposition.

### Known starting boundaries

- General concepts, traits, interfaces, or generic constraints.
- First-class iterator and iterable values.
- User-defined iteration protocols.
- Associated element-type contracts.
- Generator or coroutine functions.
- Async, concurrent, or parallel iteration.
- Complete arrays, slices, ranges, variadics, reflection, or collection APIs.
- Query comprehension or functional pipeline syntax.
- Compiler lowering.

These boundaries do not erase consequences. Record each material pressure in a
live owner or indexed raw destination.

### Intentionally unresolved framing

- Exact `each` header syntax.
- Whether `in` and `from` remain distinct.
- Which compiler-known sources are supported initially.
- Whether enum declaration traversal runs at compile time, causes body
  reprocessing, or exposes runtime metadata.
- Whether array/range traversal yields values, places, or references.
- Binding qualification and transfer stance.
- Mutation of a source during traversal.
- Whether `next` performs a traversal step while `continue` skips one, or
  whether `each` needs another transfer model.
- Cleanup and failure behavior when traversal stops early.
- How flags multi-string conversion uses a concrete source before iterable
  concepts exist.
- Which facts must be preserved for a later concepts-based iterator protocol.

### Initial stopping guidance

Stop when the work has:

- established a usable direct `each` model over a justified initial set of
  compiler-known sources;
- integrated header bindings, scopes, lifetimes, transfer, cleanup, and flow
  exits;
- established enum member and match traversal;
- dispositioned legacy `each in` and `each from` evidence;
- preserved concept-dependent iterator requirements without designing them;
- identified exact current owners and indexed deferrals; and
- completed the required documentation-fit dry run.

Do not design concepts, complete generics, first-class iterator protocols,
generators, coroutines, async or parallel iteration, promote findings, archive
this work item, or begin work item `018` without the separately required
discussion, alignment, and authorization.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs focused reading,
  numbered work, promotion, teaching, deferrals, and dry runs.
- [Raw iteration and traversal input](../raw/iteration.md) - preserves legacy
  `each`, enum traversal, flags multi-string, and later concept-dependent
  iterator pressure.
- Focused [deferred enum traversal](../../language/enums.md#deferred-enum-traversal)
  and [several string inputs for flags](../../language/enums.md#several-string-inputs-for-flags) -
  supply the immediate accepted domain requirements without fixing traversal
  syntax.
- Focused [loops and explicit scope](../../language/core-flow-control.md#loops-and-explicit-scope),
  [flow labels and transfer targets](../../language/core-flow-control.md#flow-labels-and-transfer-targets),
  and [unwinding, destruction, and completion](../../language/core-flow-control.md#unwinding-destruction-and-completion) -
  constrain repeated execution, exits, and cleanup.
- Focused [header sections and separators](../../language/source-structure.md#header-sections-and-separators)
  and [braces and body boundaries](../../language/source-structure.md#braces-and-body-boundaries) -
  constrain `each` header and body source.
- Focused [flow-control initialization](../../language/declarations-and-bindings.md#flow-control-initialization) -
  constrains declaration visibility and repeated-body scope.
- Legacy [`each`](../../flow-control.md#each) and focused array/range forms on
  that page - supply primary historical source evidence.

### Consequence-driven

- Read focused [array and collection lifetimes](../../language/lifetimes-and-references.md#arrays-and-collections)
  when traversal yields places or references.
- Read focused [transfer stances](../../language/transfer-stances.md) when
  traversal copies, moves, consumes, or exposes elements.
- Read [raw generic input](../raw/type-parameters-and-generics.md) only when a
  direct `each` rule creates a concrete concepts/constraint dependency that must
  be preserved rather than solved.
- Read [raw indexing and slicing input](../raw/indexing-and-slicing.md) when
  array, slice, range, proxy, or index traversal requires more than a local
  constraint.
- Read [raw reflection input](../raw/reflection.md) when heterogeneous member
  traversal requires metadata beyond the accepted enum facts.
- Read focused callable, construction, or optional material only when a concrete
  traversal rule crosses that owner's boundary.

### Audit-only

- `project/archive/`, including work items `001` through `016`.
- Deleted or superseded iterator implementation sketches recoverable through Git
  history.

Do not read archived work item `016` during ordinary work on `017`. Its accepted
findings are promoted into current owners and its future traversal pressure is
preserved in live raw input.

## Working record

Not started. The agent creating this work item stops here without analyzing its
content.
