# 014: Reference origins and lifetime strategies

| Field | Value |
| --- | --- |
| Status | Active working material / non-normative |
| Work Item | `014` |
| Created | 2026-09-05 |
| Owns | The bounded review defined by the fixed initiating input |
| Does Not Own | Complete pointer grammar or ownership taxonomy, async suspension/cancellation, cross-thread preparation, complete generics, formal borrow checking, layout/ABI, or compiler implementation |

## Non-authority notice

This file is a collaborative working record. Existing statements, new
observations, candidate wording, and aligned findings remain non-authoritative
until a separately discussed, aligned, and explicitly authorized promotion
incorporates them into their lasting owners.

## Fixed initiating input

This section records the information aligned when work item `014` was created.
It is intentionally incomplete and must not be rewritten as work develops.

### Initiating concern

Develop a coherent human-developer-facing model for reference origins and
selectable lifetime strategies:

1. what exact lifetime a reference names;
2. reference origin from direct storage, members, temporaries, parameters,
   results, optionals, identities, and other eligible sources;
3. the distinction between a reference to one exact value lifetime and a
   reference that tracks a place across replacement;
4. reference validity through content mutation, whole-value replacement,
   reconstructive replacement, `move`, `last`, destruction, and reconstruction;
5. parameter binding and the synchronous call-completion boundary;
6. returned references and escape;
7. reference capture and storage;
8. interior references and aliases;
9. intentional lifetime-policy plurality, guarantees, costs, and failure modes;
10. safe proof versus narrow unsafe lifetime or alias assertion;
11. independent pointer/reference binding replacement or rebinding pressure;
12. pointer-object and pointee lifetime consequences at the depth required by
    references;
13. disposition of legacy `lease` and body-time terminal-capability evidence;
14. diagnostics, source stability, and lasting documentation ownership.

### Motivating pressure

Current design permits a reference to an existing value:

```zax
source : Document
view : Document readonly & = source
```

The programmer still needs to predict what `view` means if another path:

```zax
source = makeReplacement()
consume(source as move)
consume(source as last)
```

or simply lets `source` reach destruction.

The answer may depend on whether `view` names one exact `Document` lifetime,
tracks a place across successive lifetimes, or uses another selected lifetime
policy. Those choices carry different compile-time, runtime, memory,
synchronization, and usability costs.

Function invocation already extends a temporary reference through one
synchronous call:

```zax
inspect(makeDocument())
```

That does not by itself make a returned reference valid:

```zax
view := returnView(makeDocument())
```

Transfer work also establishes reference-shaped `move` and `last` parameters,
direct compiler-managed temporary `last` stance, by-value result terminal
opportunity, explicit destructive renewal, and exactly-once resource
disposition. Reference lifetime work must explain how those accepted behaviors
remain safe and understandable.

### Known assumptions

- A reference is an access path to existing storage rather than an independent
  by-value copy.
- Current conceptual design treats a direct reference as having a hidden handle,
  auto-following its referent, and not being independently rebound. Any revision
  requires concrete pointer/reference pressure and must not overload
  declaration-side `final` with another meaning.
- Mutable/immutable value truth, final/varying place truth, writable/readonly
  access, declaration-side replacement permission, and transfer stance remain
  independent.
- A same-place alias preserves the referent's actual type-side final/varying
  truth and may narrow only its own declaration-side replacement permission.
- Explicit `varying` on an immutable reference currently identifies unusual
  place-tracking behavior that may observe successive immutable lifetimes.
- Explicit arguments bind in caller source order. A reference-bound temporary
  survives through the complete synchronous call.
- A returned reference does not become valid merely because an input temporary
  survived the call.
- Named `move`/`last` references return to `copy` for ordinary nested use unless
  destructive stance is explicitly renewed.
- A direct compiler-managed unnamed by-value temporary may offer `last`; that
  does not make a reference result's referent terminal.
- A false unsafe lifetime, presence, or alias assertion may have undefined
  consequences without a required runtime check.
- Zax has no exception-style rollback model.
- There is no language implementation in this repository.

### Known inclusions

- Reference-origin mental model and terminology.
- Exact-lifetime versus place-tracking reference behavior.
- Direct storage, member, optional, identity, parameter, temporary, result, and
  capture origins.
- Mutation, replacement, move, terminal transfer, reconstruction, and
  destruction consequences.
- Synchronous parameter and temporary lifetime.
- Returned-reference origin and escape.
- Interior aliases and same-storage identity views.
- Lifetime-policy families, selection guidance, guarantees, costs, and failure.
- Static proof, diagnostics, and narrow unsafe responsibility.
- Reference binding replacement/rebinding pressure at the depth required to
  avoid conflating it with referent replacement.
- Pointer ownership/provenance, generic, async, and cross-thread consequences
  only far enough to constrain the reference model.
- Documentation fit, current owners, legacy disposition, and indexed deferrals.

### Known starting boundaries

- Complete pointer syntax, pointer representation, ownership taxonomy, and every
  pointer conversion.
- Final async suspension, cancellation, executor, and concurrency behavior.
- Cross-thread preparation stance/capability design.
- Complete generic, specialization, or qualifier-generic syntax.
- General structural typing and subtyping.
- Formal borrow-checking algorithms or proof calculus.
- Formal layout, ABI, grammar, compiler implementation, or lowering.

These boundaries do not erase consequences. Record each material pressure in a
live owner or indexed raw destination.

### Intentionally unresolved framing

- Whether ordinary `T &` names one exact value lifetime, uses a default lifetime
  policy, or needs another explicit distinction.
- Exact source syntax for place-tracking versus exact-lifetime references.
- The final set of lifetime strategies and their selection mechanism.
- Whether one policy temporarily prevents replacement while a stable reference
  exists and another tracks place transitions.
- Which policies require runtime state, synchronization, ownership, or
  allocation.
- Reference validity during and after whole-value or reconstructive replacement.
- Returned-reference origin through nested calls, result forwarding, and
  temporaries.
- Reference capture, storage, repeated invocation, and escape.
- Whether a terminal reference/capability may survive beyond one synchronous
  consumer body.
- Whether references remain categorically non-rebindable and how pointer or
  reference binding replacement is spelled.
- Exact unsafe assertion categories and diagnostics.
- Interaction with cross-thread lifetime transferability and thread-affine
  resources.

### Initial stopping guidance

Stop when the work has:

- established a usable programmer model for reference origin;
- distinguished exact-lifetime and place-tracking references;
- established mutation, replacement, move, last, reconstruction, and destruction
  effects on reference validity;
- established synchronous parameter binding, temporary extension, result
  origin, and escape behavior;
- established the useful lifetime-policy families and their visible guarantees,
  costs, and failure modes;
- reconciled optionals, identities, members, results, captures, and immediately
  necessary pointer behavior;
- dispositioned rebinding, unsafe proof, generic, async, cross-thread, and
  ownership pressure;
- identified exact current owners and legacy changes;
- and completed the required documentation-fit dry run.

Do not promote findings, archive this work item, or begin work item `015`
without the separately required discussion, alignment, and authorization.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs focused reading,
  numbered work, promotion, teaching, deferrals, and dry runs.
- [Raw lifetime strategies](../raw/lifetimes.md) - supplies lifetime-policy
  plurality, reference origin, place tracking, rebinding, pointer/provenance,
  alias, terminal-capability, and cross-thread pressure.
- Focused [qualifier capability versus permission](../../language/qualifiers.md#type-side-truth-versus-declaration-side-permission),
  [optional qualification layers](../../language/qualifiers.md#optional-qualification-layers),
  [indirection](../../language/qualifiers.md#indirection), and
  [transfer and destruction](../../language/qualifiers.md#transfer-and-destruction) -
  supply the concrete value, place, access, alias, and replacement model.
- Focused [declaration-side replacement permission](../../language/function-invocation.md#declaration-side-replacement-permission),
  [evaluation and immediate binding](../../language/function-invocation.md#evaluation-and-immediate-binding), and
  [temporary lifetime and call completion](../../language/function-invocation.md#temporary-lifetime-and-call-completion) -
  supply synchronous binding, result, temporary, and escape pressure.
- Focused [by value and by reference](../../language/transfer-stances.md#by-value-and-by-reference),
  [projection and aliases](../../language/transfer-stances.md#projection-and-aliases),
  [receiver stance](../../language/transfer-stances.md#receiver-stance), and
  [terminal opportunity when mapping a result](../../language/transfer-stances.md#terminal-opportunity-when-mapping-a-result) -
  supply accepted transfer/reference consequences.
- Focused [reconstructive replacement](../../language/construction-and-destruction.md#reconstructive-replacement),
  [self-aliasing and interior aliases](../../language/construction-and-destruction.md#self-aliasing-and-interior-aliases),
  and [destruction](../../language/construction-and-destruction.md#destruction) -
  supply lifetime-ending, member-origin, and exactly-once disposition pressure.
- Focused [references, pointers, and other boxed types](../../language/optional-values.md#references-pointers-and-other-boxed-types)
  and [optional references](../../language/optional-values.md#optional-references) -
  supply conditional reference-handle and referent-lifetime behavior.
- Focused [identity construction and transfer](../../language/identity-types.md#construction-and-transfer) -
  supplies same-storage identity-reference and origin pressure.

### Consequence-driven

- Read [raw safety input](../raw/safety.md) and
  [raw analysis controls](../raw/analysis-controls.md) when mandatory proof,
  unsafe lifetime/alias assertion, panic, or diagnostic severity becomes
  concrete.
- Read focused legacy [pointers and references](../../pointers.md) when exact
  syntax, `lease`, pointer ownership/provenance, or terminal-capability evidence
  is required.
- Read [raw async input](../raw/async.md) when a synchronous conclusion
  materially constrains suspension, cancellation, or cross-thread preparation.
- Read [raw generic input](../raw/type-parameters-and-generics.md),
  [raw callable-selection input](../raw/callable-selection.md), or
  [raw `own` input](../raw/owned-composition.md) when reference origin must
  survive specialization, exact-prototype selection, or exposed-surface
  transformation.
- Read other pointer, allocation, capture, structural, or interop material only
  when an immediate reference-origin consequence cannot be dispositioned
  coherently without it.

### Audit-only

- `project/archive/`, including work items `001` through `013`.
- Superseded implementation sketches and compiler-internal explanations.

Do not read archived work item `013` during ordinary work on `014`. Its accepted
findings are promoted into current owners and its future pressures are preserved
in live raw inputs.

## Working record

Not started. The agent creating this work item stops here without analyzing its
content.
