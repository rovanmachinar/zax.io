# 013: Transfer stances, source state, and lifetime completion

| Field | Value |
| --- | --- |
| Status | Active working material / non-normative |
| Work Item | `013` |
| Created | 2026-09-02 |
| Owns | The bounded review defined by the fixed initiating input |
| Does Not Own | Complete pointer/reference grammar, final lifetime-policy selection, async/concurrency, complete generics, optional combinators, or formal layout/ABI rules |

## Non-authority notice

This file is a collaborative working record. Existing statements, new
observations, candidate wording, and aligned findings remain non-authoritative
until a separately discussed, aligned, and explicitly authorized promotion
incorporates them into their lasting owners.

## Fixed initiating input

This section records the information aligned when work item `013` was created.
It is intentionally incomplete and must not be rewritten as work develops.

### Initiating concern

Develop a coherent human-developer-facing model for transfer stances, source
state, and lifetime completion:

1. the distinct meaning of `copy`, `deep`, `move`, and `last`;
2. whether each stance permits, requests, or requires a specialized transfer;
3. ordinary/default transfer when no stance is written;
4. candidate availability and preference among value, reference, pointer, copy,
   move, and terminal forms;
5. when a consumer accepts a stance and when source effects occur;
6. argument, construction-packet, assignment, result, and return timing;
7. source lifetime and valid state after successful copy, move, or terminal
   transfer;
8. source-shell destruction and exactly-once resource disposition;
9. outermost stance propagation through optional, pointer, reference, member,
   and other structural layers;
10. generated versus declared transfer operations;
11. destination/result qualification and prohibition on silently increasing
    authority;
12. inferred declarations such as `result := move source` at the depth required
    by transfer;
13. self-aliasing, interior aliases, and operation-specific same-object behavior;
14. costs, diagnostics, source stability, and lasting documentation ownership.

### Motivating pressure

Current conceptual design now uses transfer stances across several owners:

```zax
source : MyType?
anotherSource : MyType?

// ...

copied : MyType? = source
moved : MyType? = move anotherSource
terminallyTransferred : MyType? = last source
```

Optional behavior fixes the state consequences after a consumer accepts a
stance, but general Zax still needs to decide source syntax, preference, timing,
qualification, propagation, and the valid source lifetime afterward.

Construction packets and function inputs bind in source order. Qualifiers
distinguish value capability, access permission, and place replacement.
Pointers, references, optionals, and structural values may have several layers
through which an outer transfer stance must remain understandable.

### Known assumptions

- A transfer-qualified source expression has no state effect until a selected
  consumer accepts it.
- Caller source order and immediate binding remain observable.
- `copy`, `deep`, `move`, and `last` are distinct intents.
- Combined stances such as `move copy` and `last deep` do not exist.
- A stance on the outermost composite source is expected to carry inward through
  generated transfer:

  ```zax
  MyType readonly ? writable * immutable * varying deep
  ```

- Wrapper, pointer, referent, and boxed qualifications remain independently
  attached.
- Optional-specific source consequences already documented in
  [Zax optional values](../../language/optional-values.md) must be preserved.
- A false unsafe lifetime assertion may have undefined consequences without a
  required runtime check.
- Zax has no exception-style rollback model.
- There is no language implementation in this repository.

### Known inclusions

- Mental model and terminology for all four transfer stances.
- Default transfer and explicit source intent.
- Transfer-qualified source views and their result type.
- Candidate viability, partial-order preference, and ambiguity.
- Immediate binding and consumer timing.
- Source lifetime, moved-from state, terminal consumption, and destruction.
- Outermost-to-inner propagation through generated operations.
- Custom interception and containing-type generation.
- Optional, pointer, reference, member, parameter, result, and capture
  consequences at the depth required by transfer.
- Inferred declaration pressure at the depth required by transfer.
- Self-aliasing, resource disposition, costs, diagnostics, and source stability.
- Documentation fit and lasting ownership.

### Known starting boundaries

- Complete pointer/reference grammar and independent binding rebinding.
- Final lifetime-policy selection or a complete borrow/ownership system.
- Async suspension, cancellation, executors, and concurrency.
- Complete generics, specialization, and generic inference.
- Optional combinators and general functional composition.
- Formal layout, ABI, grammar, compiler implementation, or lowering.

These boundaries do not erase consequences. Record each material pressure in a
live owner or indexed raw destination.

### Intentionally unresolved framing

- Whether a stance permits or mandates its specialized operation.
- Exact preference among copy, deep copy, move, terminal transfer, and reference
  binding.
- Canonical pre-unary, post-unary, parameter, and declaration syntax.
- Whether `as copy`, `as deep`, `as move`, and `as last` are accepted exact
  phrases.
- Whether accepted transfer phrases can remain unfenced.
- Exact moved-from validity guarantees.
- Whether a `last` consumer must steal resources or may still copy.
- Parameter binding versus a terminal reference/capability consumed later in a
  body.
- `:=` value/reference/pointer inference.
- Generated containing-type transfer and custom interception.
- Exact self-assignment behavior.
- Interaction with pointer ownership and lifetime strategies beyond immediate
  transfer consequences.

### Initial stopping guidance

Stop when the work has:

- established a usable programmer model for `copy`, `deep`, `move`, and `last`;
- established default and explicit stance source;
- established candidate availability and defensible preference;
- established when each consumer produces source effects;
- established source lifetime, valid state, and destruction after transfer;
- established outermost stance propagation and custom interception;
- reconciled parameters, packets, assignment, results, captures, optionals, and
  immediately necessary pointer/reference behavior;
- dispositioned inference, alias, safety, async, generic, and ownership pressure;
- identified exact current owners and promotion changes; and
- completed the required documentation-fit dry run.

Do not promote findings, archive this work item, or begin work item `014`
without the separately required discussion, alignment, and authorization.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs focused reading,
  numbered work, reader-first promotion, disposition, and dry runs.
- [Optional values](../../language/optional-values.md#the-three-ordinary-state-and-transfer-operations) -
  supplies current concrete source-state requirements and inward-propagation
  pressure.
- [Raw lifetime strategies](../raw/lifetimes.md#optional--and-layer-derived-transfer-pressure) -
  is the main live input for transfer preference, source state, terminal
  references, reference addressability, and resource disposition.
- Focused [evaluation and immediate binding](../../language/function-invocation.md#evaluation-and-immediate-binding),
  [partial-order preference](../../language/function-invocation.md#partial-order-preference),
  and [temporary lifetime](../../language/function-invocation.md#temporary-lifetime-and-call-completion) -
  provide call timing, consumer selection, result transfer, and completion
  boundaries.
- Focused [construction packet binding order](../../language/construction-and-destruction.md#evaluation-and-binding-order),
  [generated lifecycle operations](../../language/construction-and-destruction.md#declared-and-generated-lifecycle-operations),
  [optional wrapper replacement](../../language/construction-and-destruction.md#optional-complete-wrapper-replacement),
  and [self-aliasing](../../language/construction-and-destruction.md#self-aliasing-and-interior-aliases) -
  provide lifecycle generation, ordering, destruction, and alias constraints.
- Focused [qualifier capability and permission](../../language/qualifiers.md#type-side-truth-versus-declaration-side-permission),
  [optional qualification layers](../../language/qualifiers.md#optional-qualification-layers),
  and [move-out and destruction](../../language/qualifiers.md#move-out-and-destruction) -
  provide the qualification and terminal-authority model.
- Focused [direct initialization](../../language/declarations-and-bindings.md#direct-initialization)
  and [declaration boundaries](../../language/declarations-and-bindings.md#boundaries-and-maturity) -
  provide value construction and unresolved `:=` inference pressure.
- Focused [optional transfer-source operations](../../language/operators.md#optional-reset-and-transfer-source-operations),
  [operator phrases](../../language/operator-phrases.md), and
  [optional forms](../../language/operator-catalog.md#optional-forms) - provide
  current source forms, phrase recognition, fencing, result holes, and optional
  pre-unary behavior.
- [Transfer stance terminology](../../language/terms.md#transfer-stance) -
  provides the current shared term and maturity boundary.
- Focused legacy [default copy, deep, and last constructors](../../ctor-dtor.md#default-constructors) -
  preserves detailed original Zax transfer-generation intent for review rather
  than acceptance.

### Consequence-driven

- Read focused legacy pointer `last`/`lease` material when a concrete reference,
  pointer, terminal-capability, or consumer-timing question requires it.
- Read [raw analysis controls](../raw/analysis-controls.md) and
  [raw safety input](../raw/safety.md) when an unsafe, proof, panic-check, or
  diagnostic question becomes concrete.
- Read [raw callable-composition input](../raw/function-composition-and-chaining.md)
  when transfer through capture, callable construction, or composition becomes
  necessary.
- Read [raw generic input](../raw/type-parameters-and-generics.md) when
  substitution changes stance, generated operation, or inference behavior.
- Read pointer ownership, async, or structural material only when an immediate
  transfer consequence cannot be dispositioned coherently without it.

### Audit-only

- `project/archive/`, including work items `001` through `012`.
- Superseded implementation sketches and compiler-internal explanations.

Do not read archived work item `012` during ordinary work on `013`. Its current
findings are promoted into language owners and its future pressures are
preserved in live raw inputs.

## Working record

Not started. The agent creating this work item stops here without analyzing its
content.
