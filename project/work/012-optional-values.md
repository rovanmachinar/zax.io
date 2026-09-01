# 012: Optional values, construction, absence, and transfer

| Field | Value |
| --- | --- |
| Status | Active working material / non-normative |
| Work Item | `012` |
| Created | 2026-09-01 |
| Owns | The bounded review defined by the fixed initiating input |
| Does Not Own | General result/error types, pointer validity, pattern matching, async cancellation, or formal layout/ABI rules |

## Non-authority notice

This file is a collaborative working record. Existing statements, new
observations, candidate wording, and aligned findings remain non-authoritative
until a separately discussed, aligned, and explicitly authorized promotion
incorporates them into their lasting owners.

## Fixed initiating input

This section records the information aligned when work item `012` was created.
It is intentionally incomplete and must not be rewritten as work develops.

### Initiating concern

Develop a coherent human-developer-facing model for optional values,
construction, absence, and transfer:

1. what `T?` stores and means when absent or present;
2. default absent initialization;
3. construction of one present contained `T`;
4. forwarding an initializer into contained construction;
5. why failed contained construction is an error rather than absence;
6. the difference between optional construction and optional conversion;
7. copy, move, consuming/`last`, and source-state behavior;
8. contained lifetime start, destruction, replacement, and reset;
9. presence testing and proven dereference;
10. conditional convergence involving optional values;
11. nested optionals and whether `T??` remains meaningfully distinct;
12. optional parameters and results where they materially affect transfer;
13. costs, layout pressure, diagnostics, and source stability; and
14. lasting documentation owners and boundaries.

Recover Zax-specific intent before adopting another language's optional rules.

### Motivating pressure

Current conceptual design already relies on ordinary optional source:

```zax
myValue : FooBar
myOptional := (: FooBar? = myValue)

if ?myOptional
  use(myOptional.)
```

Programmers need to predict when an optional is absent, when a contained value
is constructed, what happens to the source, and when dereference is valid.

Optional construction and optional conversion must remain distinct:

```zax
myWide : I16 = 355
myConverted := myWide as U8? // absent

myConstruction := (: U8? = 355)
// error: constructing the contained U8 fails; this does not mean absence
```

### Known assumptions

- `T?` is one concrete static type distinct from `T`.
- An optional contains either no live `T` or one live `T`.
- Type-default optional initialization is absent.
- Constructing a present optional must successfully establish a valid contained
  `T`.
- Failed contained construction does not mean absence.
- Optional conversion is an explicitly selected operation that may return
  absence.
- Presence testing produces exactly `Boolean`.
- Dereference requires proof that the contained lifetime is active.
- Conditional paths converge to one concrete optional/result shape.
- Optional construction does not introduce implicit narrowing or a conversion
  chain.
- There is no language implementation in this repository.

### Known inclusions

- Absent and present mental model.
- Default initialization.
- Present-value construction and forwarded initialization.
- Copy, move, consuming/`last`, and source state.
- Contained lifetime, destruction, replacement, reset, and failure cleanup.
- Presence testing, dereference, and access proof.
- Conditional convergence.
- Optional parameters and results at the depth required by transfer.
- Optional conversion and validating admission as distinct producers.
- Nested optional behavior.
- Costs, layout pressure, diagnostics, formatting, and source stability.
- Documentation fit and lasting ownership.

### Known starting boundaries

- General result and error types.
- Exception handling or propagation.
- General algebraic sum and variant types.
- Pointer validity and complete `Nothing` semantics.
- Complete pattern matching.
- Async suspension and cancellation.
- General lifetime strategies beyond immediate optional consequences.
- Formal layout, ABI, grammar, compiler implementation, or lowering.

These boundaries do not erase consequences. Record each material pressure in a
live owner or indexed raw destination.

### Intentionally unresolved framing

- Exact generated constructor set for `T?`.
- Whether present construction forwards arbitrary initializer packets or only
  selected source shapes.
- Default copy versus explicit move/`last`.
- Source state after successful or failed transfer.
- In-place construction and temporary-elision guarantees.
- Reset and replacement spelling.
- Destruction ordering and failure cleanup.
- Nested optional behavior.
- Optional references, pointers, functions, and zero-storage values.
- Layout, discriminant, niche optimization, and ABI pressure.
- Whether optional operations are protected, generated, or declared by a
  generic mechanism.
- Exact diagnostics and formatting.

### Initial stopping guidance

Stop when the work has:

- established the ordinary absent/present mental model;
- established present construction and failure;
- separated construction, optional conversion, and default absence;
- established copy/move/`last` and source-state behavior at programmer depth;
- established contained lifetime, destruction, reset, and proven access;
- reconciled conditional convergence, parameters, results, and identities;
- preserved layout/ABI/implementation pressure without designing machinery;
- assigned every deferred consequence a live destination;
- identified exact current owners and promotion changes; and
- completed the required documentation-fit dry run.

Do not promote findings, archive this work item, or begin work item `013`
without the separately required discussion, alignment, and authorization.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs focused reading,
  numbered work, reader-first promotion, disposition, and dry runs.
- [Legacy optional input](../../optional.md) - preserves detailed candidate
  construction, reset, presence, dereference, transfer, and lifetime behavior.
- Focused [construction and destruction](../../language/construction-and-destruction.md#optional-construction-must-establish-the-contained-value) -
  provides present-construction failure and contained-lifetime boundaries.
- Focused [conditionally live storage and access proof](../../language/construction-and-destruction.md#access-proof) -
  provides the current dereference proof obligation.
- Focused [optional presence operation](../../language/operators.md#optional-presence-operation) -
  provides protected presence behavior.
- Focused [conditional expression and branch convergence](../../language/core-flow-control.md#conditional-expression-and-branch-convergence) -
  provides the current optional convergence boundary.
- Focused [optional construction from numeric source](../../language/integer-literals.md#optional-construction-still-has-to-build-a-value) -
  provides the distinction between contained construction failure and absence.
- Focused [exact intrinsic optional conversion](../../language/integer-operator-catalog.md#exact-intrinsic-conversion) -
  provides `as Destination?` as an explicit optional-value producer.
- Focused [identity admission](../../language/identity-types.md#admission) -
  provides validating `optional from` as a distinct producer.

### Consequence-driven

- Read focused function invocation and qualifier material when a concrete
  parameter, result, copy, move, `last`, or access-capability question requires
  it.
- Read [raw lifetime input](../raw/lifetimes.md) only when current contained
  lifetime rules cannot resolve an immediate optional case.
- Read pointer or `Nothing` material only when optional behavior would otherwise
  imply equivalence.
- Read [raw selection input](../raw/selection.md) only when optional matching
  creates a concrete requirement.
- Read raw analysis or safety input only when a proof, failure, or unsafe
  boundary cannot be described through current owners.

### Audit-only

- `project/archive/`, including work items `001` through `011`.
- Superseded implementation sketches and compiler-internal explanations.

Do not read archived work item `011` during ordinary work on `012`. Its current
findings are promoted into language owners and its future pressures are
preserved in live raw or legacy inputs.

## Working record

Not started. The agent creating this work item stops here without analyzing its
content.
