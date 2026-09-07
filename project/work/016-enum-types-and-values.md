# 016: Enum types and values

| Field | Value |
| --- | --- |
| Status | Active working material / non-normative |
| Work Item | `016` |
| Created | 2026-09-07 |
| Owns | The bounded review defined by the fixed initiating input |
| Does Not Own | General reflection, generics, computed members, complete selection/pattern matching, partial enum extension, ABI/FFI contracts, or compiler lowering |

## Non-authority notice

This file is a collaborative working record. Existing statements, new
observations, candidate wording, and aligned findings remain non-authoritative
until a separately discussed, aligned, and explicitly authorized promotion
incorporates them into their lasting owners.

## Fixed initiating input

This section records the information aligned when work item `016` was created.
It is intentionally incomplete and must not be rewritten as work develops.

### Initiating concern

Develop a coherent human-developer-facing model for enum types and values:

1. enum declaration and member source;
2. enum type and member identity;
3. ordinary enums versus flags/bitset-like enums;
4. optional explicit underlying integer types;
5. valid, invalid, and unknown underlying values;
6. default initialization;
7. construction, admission, projection, and conversion;
8. equality, ordering, bitwise, and generated operation availability;
9. exhaustive-selection pressure without designing complete pattern matching;
10. representation, portability, costs, and diagnostics;
11. interaction with existing endian semantic enums; and
12. disposition of legacy enum syntax and behavior.

### Motivating pressure

Zax has current integer, identity, operator, and endian behavior that refers to
enums, but no cohesive current enum owner. Programmers need to understand forms
such as:

```zax
// Illustrative starting shape; exact enum syntax requires review.
Color :: enum {
  Red
  Green
  Blue
}

color : Color = Color.Red
```

The language must explain whether every underlying bit pattern is a valid enum
value, how explicit underlying values behave, which operations are generated,
and how ordinary semantic enums differ from flags intended for bitwise
composition.

### Known assumptions

- Enum values have a concrete enum type identity rather than behaving as
  untyped integer constants.
- Existing current endian semantic enum families and the four generated
  underlying/admission operations are evidence and constraints.
- An underlying integer representation does not automatically grant every
  integer operation to the enum identity.
- Representation and value validity are distinct concerns.
- Current work may identify reflection or generic pressure without designing
  those mechanisms.
- There is no language implementation in this repository.

### Known inclusions

- Ordinary enum declaration and member mental model.
- Member identity and explicit values.
- Underlying integer selection and defaults.
- Default enum value behavior.
- Validity of unnamed underlying values.
- Construction, admission, projection, and conversion.
- Generated equality, ordering, and applicable bitwise operations.
- Ordinary versus flags-style behavior.
- Interaction with integer identities and endian semantic enums.
- Exhaustiveness pressure needed to constrain future selection work.
- Costs, diagnostics, portability, and source stability.
- Lasting documentation ownership and legacy disposition.

### Known starting boundaries

- General compile-time reflection and enum-to-string metadata.
- Generic or generated user enum families.
- Computed or runtime-dependent enum members.
- Complete `switch`, pattern matching, and exhaustiveness syntax.
- Partial/open enum extension.
- Formal layout, ABI, FFI, or compiler lowering.
- Serialization frameworks and schema generation.

These boundaries do not erase consequences. Record each material pressure in a
live owner or indexed raw destination.

### Intentionally unresolved framing

- Exact enum declaration and member syntax.
- Whether explicit values are required, optional, or mixed.
- Default underlying integer type.
- Whether ordinary enums admit unnamed underlying values.
- Whether flags are a separate declaration kind, policy, or ordinary enum
  contract.
- Default initialization and whether one member is distinguished as default.
- Which operations are generated versus unavailable.
- Ordering semantics for ordinary enums.
- How unsafe/raw admission differs from ordinary construction.
- How exhaustive selection treats unknown or unnamed values.

### Initial stopping guidance

Stop when the work has:

- established a usable enum declaration and value model;
- established ordinary and flags-style boundaries;
- established underlying representation and value-validity rules;
- established construction, conversion, and generated operation behavior;
- established default initialization;
- constrained future exhaustive selection without designing it;
- dispositioned legacy enum evidence;
- identified exact current owners and indexed deferrals; and
- completed the required documentation-fit dry run.

Do not design reflection, generics, complete selection, partial enum extension,
ABI/FFI, promote findings, archive this work item, or begin work item `017`
without the separately required discussion, alignment, and authorization.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs focused reading,
  numbered work, promotion, teaching, deferrals, and dry runs.
- [Legacy enum design](../../enums.md) - supplies the primary declaration,
  member, underlying-value, flags, conversion, and operation evidence.
- [Raw enum input](../raw/enum-types.md) - preserves current enum validity,
  operation, generation, and future-owner pressure.
- Focused [generated underlying and enum forms](../../language/operator-catalog.md#generated-underlying-and-enum-forms) -
  supplies current generated operation names and protected behavior.
- Focused [endian semantic enum families](../../language/endianness.md#endian-semantic-enum-families)
  and [generated enum operations](../../language/endianness.md#the-four-generated-enum-operations) -
  supply current concrete enum use and receiver-correct behavior.
- Focused [enum and endian boundaries](../../language/integers.md#enum-and-endian-boundaries)
  and [integer identity types](../../language/integers.md#integer-identity-types) -
  constrain underlying representation without granting integer membership.
- Focused [identity boundary and underlying type](../../language/identity-types.md#identity-boundary-and-underlying-type),
  [admission](../../language/identity-types.md#admission), and
  [construction and transfer](../../language/identity-types.md#construction-and-transfer) -
  supply distinct-identity, admission, projection, and construction pressure.
- Focused [non-value definitions](../../language/declarations-and-bindings.md#non-value-definitions),
  [direct initialization](../../language/declarations-and-bindings.md#direct-initialization),
  and [default initialization](../../language/declarations-and-bindings.md#default-initialization) -
  supply declaration and value-initialization integration.

### Consequence-driven

- Read [raw selection input](../raw/selection.md) when enum exhaustiveness,
  unknown values, or flags create immediate selection constraints.
- Read [raw reflection input](../raw/reflection.md) when member iteration,
  names, strings, or metadata become necessary.
- Read [raw generic input](../raw/type-parameters-and-generics.md) when a proposal
  requires generic enum families or constraints.
- Read [raw partial-type input](../raw/partial-types.md) when enum extension or
  externally added members becomes concrete.
- Read [raw interop input](../raw/interop.md) when underlying representation
  requires ABI or foreign-language guarantees.
- Read focused construction, safety, source, or operator material only when a
  concrete enum rule crosses that owner's boundary.

### Audit-only

- `project/archive/`, including work items `001` through `015`.
- Deleted or superseded enum implementation sketches recoverable through Git
  history.

Do not read archived work item `015` during ordinary work on `016`. Its accepted
findings are promoted into current owners and its future pressures are preserved
in live raw inputs.

## Working record

Not started. The agent creating this work item stops here without analyzing its
content.
