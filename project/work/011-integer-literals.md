# 011: Integer literals and compile-time realization

| Field | Value |
| --- | --- |
| Status | Active working material / non-normative |
| Work Item | `011` |
| Created | 2026-08-31 |
| Owns | The bounded review defined by the fixed initiating input |
| Does Not Own | Complete non-integer literal families or internal compiler mechanisms |

## Non-authority notice

This file is a collaborative working record. Existing statements, new
observations, candidate wording, and aligned findings remain non-authoritative
until a separately discussed, aligned, and explicitly authorized promotion
incorporates them into their lasting owners.

## Fixed initiating input

This section records the information aligned when work item `011` was created.
It is intentionally incomplete and must not be rewritten as work develops.

### Initiating concern

Develop a coherent human-developer-facing model for integer literals and
compile-time integer realization:

1. how integer source produces a mathematical/compiler-known value;
2. decimal, binary, octal, hexadecimal, and other accepted integer bases;
3. attached prefix forms such as `h'...'`, where retained;
4. positive, unsigned-requested, and negative source;
5. whether a sign participates in realization or applies an operator afterward;
6. unconstrained default realization as `Integer` or `UInteger`;
7. direct realization against a typed slot;
8. compiler-known uncommitted integer values or an equivalent source model;
9. arbitrary-precision compile-time evaluation before concrete realization;
10. exact fit, overflow, and underflow;
11. the minimum signed value without requiring an impossible positive
    intermediate;
12. narrow expected-result context and its limits;
13. interaction with exact, optional, and narrowing integer conversion;
14. active, target, and compiler-host type requests;
15. costs, failures, diagnostics, formatting, and source stability; and
16. lasting documentation owners and boundaries.

Recover Zax-specific intent before adopting another language's literal rules.

### Motivating pressure

Current integer teaching relies on ordinary source such as:

```zax
myValue := 56
myUnsigned := +56
myNegative := -27
myByte : Byte = 255
myMinimum : I8 = -128
```

Programmers need to predict which type each expression receives, when a typed
slot realizes the value directly, and when compilation rejects a value that does
not fit.

Current design also allows a compile-time expression to supply a concrete type:

```zax
// Illustrative type-producing expression; exact generic syntax remains future.
MySelectedType :: alias type chooseIntegerType()
myValue : MySelectedType = 42
```

This work must explain the programmer-visible result without describing internal
compiler communication.

### Known assumptions

- [Zax integers](../../language/integers.md) owns current integer types, ranges,
  identities, storage, and conversion relationships.
- [Zax integer operator catalog](../../language/integer-operator-catalog.md)
  owns protected signedness, negation, and conversion behavior.
- [Zax declarations and bindings](../../language/declarations-and-bindings.md)
  owns typed and inferred declaration forms and direct initialization.
- [Zax function invocation](../../language/function-invocation.md) owns narrow
  expected-result context and result selection.
- [Zax source structure](../../language/source-structure.md) owns token,
  whitespace, quote, and continuation boundaries already established.
- [Zax operator catalog](../../language/operator-catalog.md) owns exact operator
  forms and precedence.
- Every successfully selected single expression result ultimately has one
  concrete static type.
- A typed slot may directly realize an uncommitted compile-time integer value;
  this is not implicit conversion from a hidden preselected `Integer`.
- An expression already committed to `Integer` does not gain another conversion
  merely because it is constant.
- There is no language implementation in this repository.

### Programmer-visible compile-time boundary

A type-producing compile-time expression may supply the concrete type used to
realize an integer literal. This work accepts that capability at the
programmer-visible level.

The required model is only:

1. the compile-time expression supplies a concrete type identity;
2. that identity is available before the typed initializer is realized;
3. the integer value realizes directly against that type; and
4. an unrepresentable value produces a compiler diagnostic.

Do not design or explain:

- internal type-metadata representation;
- compiler-generated bridge code;
- reflection metadata transport;
- compiler-host ABI communication;
- compiler callbacks;
- lowering or evaluation-engine architecture; or
- internal compiler phase/data-structure mechanics.

If a concrete source-level contradiction depends on one of those mechanisms,
capture the pressure for implementation/future specification work without
solving it here.

### Known inclusions

- Default type and signedness for unconstrained integer source.
- Typed-slot and expected-result realization.
- Compile-time-known uncommitted integer values.
- Positive source, pre-unary unsigned request, and negative source.
- Minimum signed values.
- Exact-value fit and compile-time failure.
- Supported bases, prefixes, digits, separators, and readability.
- Grouping that must not change type, value, width, or failure.
- Interaction with exact, optional, and narrowing integer conversion.
- Environment-selected integer identities at the depth required to realize a
  literal.
- Source stability when defaults, prefixes, or type context change.
- Diagnostics that show requested type, value, range, and deciding context.
- Documentation fit and lasting ownership.

### Known starting boundaries

- Complete floating-point rounding and literal representation.
- Complete fixed-point, decimal, and unbounded numeric literals.
- Complete string, character, byte-string, and encoding literal behavior.
- General user-defined literal operators.
- Complete compile-time execution, scheduling, sandboxing, and resource policy.
- Complete generic and computed-type syntax.
- Complete reflection and metadata representation.
- Formal grammar, compiler implementation, lowering, or ABI bridges.

These boundaries do not erase consequences. Record each material pressure in a
live owner or indexed raw destination.

### Intentionally unresolved framing

- Whether an uncommitted integer value is a compile-time-only type, another
  static category, or an equivalent result mechanism.
- Whether sign is part of literal source formation or a selected unary operation
  in every applicable context.
- Exact base and prefix spellings.
- Digit separators and grouping rules.
- Default width growth when a value exceeds the ordinary `Integer` range.
- Whether arbitrary-precision compile-time integers always remain available.
- How explicit target/compiler-host type requests constrain realization.
- Exact diagnostics and formatting.
- Which findings belong in an integer owner, literal owner, source structure,
  operator catalog, compile-time owner, or generic/reflection future input.

### Initial stopping guidance

Stop when the work has:

- established how ordinary integer source obtains a concrete type;
- established typed-slot direct realization without implicit conversion;
- established positive, unsigned-requested, negative, and minimum-signed source;
- established bases, prefixes, digit rules, and fit diagnostics at
  human-developer depth;
- reconciled grouping, expected-result context, conversion, and environment
  selection;
- preserved compile-time/generic/reflection implementation pressure without
  designing internal machinery;
- assigned every deferred consequence a live owner and activation pressure;
- identified lasting owners and an exact promotion set; and
- completed the required documentation-fit dry run.

Do not promote findings, design compiler internals, archive this work item, or
begin work item `012` without the separately required discussion, alignment, and
authorization.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs focused reading,
  numbered work, teachability, dry runs, promotion, and archival.
- Focused [Zax integers](../../language/integers.md#initialization-and-compile-time-realization) -
  provides the current direct-realization boundary.
- Focused [integer operator catalog](../../language/integer-operator-catalog.md#signedness-counterpart) -
  provides unary signedness and conversion constraints.
- Focused [declarations and bindings](../../language/declarations-and-bindings.md#core-value-forms) -
  provides typed/inferred declaration and direct-initialization behavior.
- Focused [function invocation](../../language/function-invocation.md#narrow-expected-result-selection) -
  provides the accepted expected-result boundary.
- Focused [operator catalog](../../language/operator-catalog.md#fixity-and-adjacency) -
  provides unary/operator source formation.
- Focused [source structure](../../language/source-structure.md#phrase-fences-and-literal-coordination) -
  provides quote attachment and phrase-fence boundaries.
- [Raw literal input](../raw/literal-operators.md) - preserves unresolved
  realization, prefix, payload, and custom-literal pressure.
- [Raw compile-time execution input](../raw/compile-time-execution.md) -
  preserves the programmer-visible execution-context and type-result boundary.
- Focused [legacy intrinsic system literals](../../basics.md#intrinsic-system-literals) -
  preserves detailed candidate bases, prefixes, separators, and literal
  examples not yet consumed.

### Consequence-driven

- Read [raw type-parameter and generic input](../raw/type-parameters-and-generics.md)
  only when a concrete source-level type result cannot be described without it.
- Read [raw CPU-provider input](../raw/cpu-provider-model.md) only when
  active/target/compiler-host realization cannot be resolved from current
  integer contracts.
- Read [raw reflection input](../raw/reflection.md) only when literal provenance
  or realization must be observable to source/tooling.
- Read [raw numeric-family input](../raw/numeric-type-families.md) when an
  unusual width or arbitrary-precision value constrains current literal
  behavior.
- Read floating, fixed-point, string, or custom-literal input only when a shared
  source decision would otherwise contradict those future families.
- Read focused legacy compiler directives, meta-functions, or meta-types only
  when a concrete programmer-visible contradiction requires their evidence.
  Do not read them merely to explain how the compiler transports or returns a
  type identity.

### Audit-only

- `project/archive/`, including work items `001` through `010`.
- Superseded implementation sketches and compiler-internal explanations.

Do not read archived work item `010` during ordinary work on `011`. Its current
findings are promoted into language owners and its future pressures are indexed
in raw inputs.

## Working record

Not started. The agent creating this work item stops here without analyzing its
content.
