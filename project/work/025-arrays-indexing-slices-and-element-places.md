# 025: Arrays, indexing, slices, and element places

| Field | Value |
| --- | --- |
| Status | Active working material / non-normative / awaiting assignment |
| Work Item | `025` |
| Created | 2026-09-17 |
| Owns | The bounded review defined by the fixed initiating input |
| Does Not Own | General user-defined generics, complete general collections, runtime string libraries, compiler vectorization, parallel algorithms, compile-time execution, reflection, or implementation architecture |

## Non-authority notice

This file is a collaborative working record. Existing statements, new
observations, candidate wording, and aligned findings remain non-authoritative
until a separately discussed, aligned, and explicitly authorized promotion
incorporates them into their lasting owners.

## Fixed initiating input

This section records the information aligned when work item `025` was created.
It is intentionally incomplete and must not be rewritten as work develops.

### Initiating concern

Establish a coherent programmer-facing model for intrinsic arrays, indexing,
slices, and element places: how element type, shape, length, dimensions,
construction, storage, bounds, mutation, views, lifetime, and traversal interact
without depending on unresolved user-defined generics.

The
[arrays and slices maintainer notes](../raw/arrays-and-slices-maintainer-notes.md)
are primary maintainer-supplied input. The language maintainer will populate
that file before work item `025` begins. Read and disposition its complete
contents as the latest design evidence. It may add to, replace, supersede, or
obsolete any portion of the legacy corpus; retain legacy pages as evidence only
for useful material the refreshed notes do not cover.

The indexed
[indexing and slicing input](../raw/indexing-and-slicing.md) preserves current
operator, multi-index, slicing, range, dynamic allocation, capacity, bit-place,
proxy, traversal, and element-stability pressure.

### Motivating pressure

Arrays are foundational to Zax's data-oriented purpose, but no current concept
owner defines their complete type, storage, construction, indexing, slicing, or
mutation model.

Several current owners already rely on bounded array behavior:

- fixed-array traversal visits stable element places in increasing index order;
- lifetime design requires every dynamic operation to say which element places
  continue, end, or relocate;
- pointer design treats one allocated array as an allocation root rather than
  each element as an independent allocation;
- strings depend on sequence capacity, array relationships, and eventual
  slicing/view behavior;
- operator design recognizes `operator index`, multi-index forms, and direct
  indexed mixfix mutation; and
- `IndexSize` and `TypeSize` distinguish logical positions/counts from storage
  extent and overflow.

These pressures should be resolved before runtime strings, general collections,
collection-aware generics, SIMD/parallel APIs, and compiler transformations
build on an unspecified sequence model.

### Known assumptions

- An intrinsic array family may use language-provided element/count
  parameterization without establishing general user-defined generics.
- A fixed-shape array owns stable element places for its life path.
- A dynamic collection operation must state which existing element places
  remain, relocate, or end.
- Index and call are postfix delimited forms, not arbitrary bracket
  punctuation.
- Direct indexed read/write/compound behavior may be one operation; no proxy is
  implied unless the selected result really provides one.
- A receiver, every index, and a direct indexed-mutation source evaluate once in
  visible source order.
- Valid element indexes and valid slice/splice endpoints have different upper
  bounds.
- `IndexSize` and `TypeSize` are distinct intent identities even when their
  ranges and representations agree.
- Dynamic allocation obtains all required storage before element constructors
  or initializers execute.
- Source, declaration, import, or traversal order does not silently resolve an
  ambiguity.

### Known inclusions

- Fixed-length array identity, element type, dimensions, length, and
  multidimensional shape/layout at programmer depth.
- Dynamic array identity, logical length, capacity, allocation ownership, and
  growth/reallocation contracts.
- Default, list, construction-packet, repeated, copied, moved, partially
  initialized, and explicitly unsafe element construction where each has a
  demonstrated need.
- Element destruction and partial-construction failure.
- Zero-based indexing, multiple indices, direct indexed read/write/compound
  operations, bounds checks, and result/place/proxy behavior.
- Slice/view identity, range/endpoint source, ownership versus borrowing,
  qualification, lifetime, mutation, overlap, and invalidation.
- Element-place stability under replacement, growth, insertion, removal,
  slicing, and traversal.
- `IndexSize`/`TypeSize` arithmetic, capacity, count-to-byte conversion, and
  overflow.
- Direct and cursor-driven `each` participation.
- Array source versus lambda-capture/source-enclosure ambiguity.
- Array-of-structures versus structure-of-arrays pressure where programmer
  layout, indexing, iteration, or transformation behavior materially differs.
- Diagnostics, costs, formatting, reflection pressure, and source stability.
- Complete disposition of the maintainer notes, focused raw input, and reached
  legacy array/index/slice material.

### Known starting boundaries

- General maps, sets, lists, trees, queues, and application/library collection
  APIs.
- General user-defined generic declarations, deduction, specialization,
  constraints, and generic collection concepts.
- Complete runtime string conversion, mutation, normalization, grapheme,
  presentation, locale, and code-page APIs.
- Complete compiler auto-vectorization, SIMD intrinsic catalogs, MIMD
  scheduling, parallel algorithms, and GPU execution.
- Complete compile-time execution, reflection metadata shape, and generated
  collection families.
- Complete pointer representation, allocator implementation, object-file
  layout, ABI, and lowering architecture.

These boundaries do not erase consequences. Preserve every material pressure in
a current owner or indexed raw destination rather than leaving it only in the
eventual archive of this work item.

### Intentionally unresolved framing

- Exact fixed/dynamic array and slice type syntax.
- Whether dimensions associate inside-out or outside-in and how source teaches
  that order.
- Whether array literals are expressions, construction-only source, or one
  destination-directed family.
- Whether length participates in canonical type identity for every fixed form.
- Exact dynamic-array ownership and whether one built-in dynamic form exists or
  several policy-selected forms.
- Bounds failure as panic, optional/reporting result, static diagnostic, unsafe
  unchecked operation, or a documented family.
- Whether index returns a value, reference, place, or proxy for each array/view
  category.
- Exact range and slice syntax, including inclusive/exclusive endpoints,
  omission, empty/reversed ranges, and multidimensional slicing.
- Whether a slice owns storage, borrows one contiguous region, anchors another
  owner, or has distinct forms for those relationships.
- Which mutations preserve element places and which invalidate views,
  references, pointers, and active traversal.
- How array-of-structures and structure-of-arrays transformations relate to
  current structural mapping without becoming hidden layout conversion.

### Initial stopping guidance

Stop when the work has:

- established a teachable fixed/dynamic array and slice/view mental model;
- defined type identity, dimensions, length, capacity, layout, and
  initialization at programmer depth;
- defined deterministic index/multi-index and direct mutation behavior;
- defined bounds and failure families without hidden fallback;
- defined element place, lifetime, invalidation, and traversal consequences;
- reconciled intrinsic array syntax with source, construction, pointer,
  integer-size, operator, mixfix, iteration, string, and structural constraints;
- assigned generic, runtime-string, reflection, vectorization, parallel,
  collection-library, and implementation consequences to current or indexed
  future owners;
- dispositioned all useful refreshed and reached legacy material; and
- completed the required documentation-fit dry run.

Do not promote findings, archive this work item, begin work item `026`, or
design general generics, collection libraries, runtime string services,
compile-time execution, reflection, vectorization, parallel execution, or
implementation architecture without the separately required discussion,
alignment, and authorization.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs focused reading,
  decision maturity, promotion, deferral, and closure.
- [Arrays and slices maintainer notes](../raw/arrays-and-slices-maintainer-notes.md) -
  will contain the maintainer's refreshed primary input before this work item is
  assigned; read and disposition it completely.
- [Legacy arrays](../../arrays.md) - preserves the principal fixed,
  multidimensional, initialization, dynamic-allocation, slicing, and
  array-overhead use cases to disposition by value.
- [Raw indexing and slicing](../raw/indexing-and-slicing.md) - preserves direct
  index/mixfix, range, capacity, dynamic allocation, bit-place, proxy, traversal,
  and element-stability pressure.
- [Declarations and bindings](../../language/declarations-and-bindings.md) -
  supplies type/declaration, initialization, inference, qualifier, and
  assignment boundaries.
- [Construction, replacement, and destruction](../../language/construction-and-destruction.md) -
  owns construction packets, member/aggregate initialization, partial
  construction, replacement, and destruction.
- [Lifetimes and references](../../language/lifetimes-and-references.md) -
  owns current fixed-array element stability and dynamic collection
  continuation/relocation requirements.
- [Pointers, allocation, and arenas](../../language/pointers-and-arenas.md) -
  owns allocation roots, dynamic storage, pointer roles, arenas, and
  disposition.
- [Iteration](../../language/iteration.md) - owns fixed-array `each` traversal
  and the cursor protocol.
- [Operators](../../language/operators.md), the
  [operator catalog](../../language/operator-catalog.md), and
  [mixfix operators](../../language/mixfix-operators.md) - own index recognition,
  direct indexed mutation, candidate trees, result shapes, and evaluation.
- [Source structure](../../language/source-structure.md) - owns bracket,
  delimiter, comma-list, capture, continuation, and intent boundaries.
- [Integers](../../language/integers.md) - owns `IndexSize`, `TypeSize`,
  associated count/storage identities, conversion, and overflow constraints.

### Consequence-driven

- [Structural shapes and compatibility](../../language/structural-shapes-and-compatibility.md)
  when array shape, flattening, decomposition, recomposition, or
  array-of-structures/structure-of-arrays transformation becomes concrete.
- [Strings and characters](../../language/strings-and-characters.md) and
  [raw string runtime/encoding](../raw/string-runtime-and-encoding.md) when
  arrays, slices, capacity, contiguity, or mutation constrain current/future
  string behavior.
- [Safety and analysis](../../language/safety-and-analysis.md) when bounds,
  unchecked indexing, partial construction, overlap, or invalidation creates a
  safe/unsafe guarantee.
- [Transfer stances](../../language/transfer-stances.md) when element,
  aggregate, slice, or dynamic-storage copy/deep/move/last behavior becomes
  material.
- [Raw type parameters and generics](../raw/type-parameters-and-generics.md)
  when intrinsic array parameterization crosses into user generic declarations,
  constraints, or generated families.
- [Raw pointer and arena mechanics](../raw/pointer-and-arena-mechanics.md) when
  array allocation requires new arena interfaces, allocation records, pointer
  casts, provenance, or control-block mechanics.
- Relevant legacy allocation, construction, casting, string, flow, or operator
  material only when a concrete question cannot be resolved from the refreshed
  notes, required array evidence, and current owners.

### Audit-only

- Archived work only when a concrete provenance or regression question cannot
  be answered from current owners and live raw input.

## Initial working posture

Begin with programmer source that declares one fixed array, initializes its
elements, obtains one element place, takes one bounded slice/view, and traverses
the result. Explain type identity, storage ownership, bounds, and lifetime before
designing dynamic growth policies, generalized ranges, proxy families, or
parallel execution.

Treat the maintainer notes as the latest primary evidence once populated. Legacy
pages remain available for useful unrefreshed cases but do not override the
maintainer notes merely because they contain more text.

## Working record

Awaiting assignment and completion of the maintainer-supplied primary notes.
Creating and routing work item `025` does not authorize analysis.
