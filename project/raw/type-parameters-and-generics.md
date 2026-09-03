# Raw input: type parameters and generics

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work defining generics, type parameters, constraints, or computed type results |
| Applies To | Type-input pressure exposed by operator-phrase, declaration, invocation, and integer-family review |
| Owns | Preserved type-parameter, generic, integer-factory, relational-pair, associated-type, representative-source, activation, and retirement questions |
| Does Not Own | Accepted generic semantics or current declaration/type behavior |
| Source / Provenance | Legacy `meta-types.md` and `meta-functions.md` evidence together with operator-phrase review of type parameters/type receivers and work item `012` optional-depth substitution pressure |

## Why this input exists

Current documentation accepts a deliberately narrow type-input model:

- a prototype may declare a `ParameterType : type` slot;
- a caller completes it with one concrete type identity;
- that type argument has no runtime storage, lifetime, or evaluation; and
- a `operator type` receiver is a concrete type identity rather than an instance.

See
[declarations and bindings](../../language/declarations-and-bindings.md#operator-phrase-declarations-and-type-parameters)
and
[function invocation](../../language/function-invocation.md#type-parameter-slots).

Nothing beyond that is accepted. Generic declarations, constraints, computed type
results, and generated type families remain undecided, and this file keeps those
questions live.

## Accepted narrow baseline

```zax
Source :: type {
  operator binary 'as' final : (
    result : DestinationType
  )(
    DestinationType : type
  ) readonly = {
  }
}

converted := source as DestinationType
```

A value receiver may also accept a type argument without that argument
contributing receiver discovery:

```zax
schema : MyReceiverType
result := schema for SomeType
```

Only *mechanically determined* reserved type results and value results dependent
on concrete type arguments are assumed today.

## Legacy generic evidence

Legacy material uses a `$` sigil for type variables and a `Name$(...)` form for
parameterized types:

```zax
MyType$(TypeA, TypeB) :: type {
    value1 : $TypeA
    value2 : $TypeB
}

addThenMultiply final : (
    result : $Type
)(
    value1 : $Type,
    value2 : $Type
) = {
}
```

That spelling has not been reviewed against the current `ParameterType : type`
slot. Future work must decide whether the two are the same mechanism written
differently, two distinct mechanisms, or one superseding the other.

## Generic type arguments and constraints

Future work must decide:

- how a generic declaration names its type parameters;
- whether a type parameter may be constrained, and by what vocabulary;
- how a constraint interacts with fixed-arity viability and partial-order
  preference;
- whether a type argument may be inferred from value arguments;
- default type arguments; and
- diagnostics when several instantiations are equally viable.

## Dependent value result types

`as` already demonstrates a result type determined by a type argument:

```zax
converted := source as DestinationType
```

Future work must decide how far that generalizes: whether an ordinary user
declaration may state a result type in terms of its own type parameters, how such
a result participates in narrow expected-result selection, and where inference
stops.

## Computed type results

A general computed or runtime-dependent type result is **not** assumed:

```zax
// Illustrative pressure; no general computed type result is accepted.
Selected :: selectType(condition)
```

The mechanically determined reserved type results, such as `as default` and
`type of`, do not authorize arbitrary user-defined runtime-dependent type-result
functions. Future work must decide whether a compile-time function may return a
type identity, what identity such a result has, and how it interacts with
overload selection and reflection.

## Generic and alias type receivers

For a non-generic declaration, the enclosing type name identifies the receiver
type inside its body:

```zax
MyType :: type {
  operator type pre unary 'custom type info for' final : (
    result : MyCustomTypeInfo
  )() = {
  }
}
```

Undecided:

- what the enclosing name means inside a generic definition;
- whether a type-receiver operation is discovered through an alias;
- whether an alias and its target share type-receiver operations;
- how qualified type identities behave as receivers; and
- whether a generic instantiation may add or remove type-receiver operations.

### Qualified type receivers and `once`

Reference admission creates pressure for a qualified type identity to supply
receiver discovery:

```zax
// Illustrative; qualified type-receiver syntax is not established.
myView := MyIdentity & from myUnderlying
```

Future work must decide:

- whether `MyIdentity &` is one qualified type receiver;
- whether expected-result context instead selects a reference-returning
  `MyIdentity from ...` operation;
- how alias and generic identities participate;
- whether an explicit type parameter slot would duplicate the implicit
  `operator type` receiver;
- how `operator type` relates to legacy/static `once final`; and
- whether `once varying` has a corresponding type-receiver role.

The result must never silently change between by-value identity construction and
same-storage reference view.

## Generated type families

Endian enums are described as conceptually generated families with explicitly
illustrative names such as `BigEndianU32`. See
[Zax endianness](../../language/endianness.md) and [enum types](enum-types.md).

Future generic work must decide:

- how a language-provided family is generated and named;
- whether user code may declare a similar family;
- whether family members are ordinary named types or a distinct kind;
- how a family member's identity, reflection, and documentation appear; and
- whether a family may be parameterized over more than the backing type.

## Integer specializations and factory selection

The concrete programmer-visible integer families and relationships are current
in [Zax integers](../../language/integers.md). This section retains only the
future generic/factory mechanism needed to produce them.

Future integer work assumes a built-in generic family conceptually like:

```zax
// Illustrative only.
MyI57 :: alias type Integer$(57, Sign.Signed)
```

Each realized combination is a concrete, closed intrinsic type. It is not still
generic after realization.

A separate integer factory returns one concrete specialization from dimensions
such as:

- minimum and maximum logical width;
- signedness;
- fastest or smallest preference;
- active, target, or compiler-host environment;
- native-representation requirement; and
- software fallback.

Illustrative use:

```zax
// Exact factory and type-result syntax remain unsettled.
MySelectedInteger :: alias type IntegerFactory.select(
  minimumBitCount: 13,
  maximumBitCount: 24,
  signedness: Sign.Signed,
  environment: Target
)
```

Future work must define a type-returning factory function rather than treating
selection as an array or receiverless phrase. A selected public role identity
retains its role when two profiles happen to choose the same exact
specialization.

## Relational signedness pairs

The exact intrinsic family derives its counterpart by changing only signedness:

```text
Integer$(W, Signed) <-> Integer$(W, Unsigned)
```

Named role identities require an explicit relational pair:

```text
Small <-> USmall
FastI16 <-> FastU16
IPointer <-> UPointer
```

The integer factory or another validated generic mechanism should define both
identities together:

1. apply one shared selector except for signedness;
2. realize equal-width intrinsic counterparts;
3. establish both public identity branches;
4. bind them as unique mutual counterparts; and
5. generate their protected counterpart operations.

The compiler must not infer relationships from spelling. Future programmer code
should be able to define owner-authorized pairs through the same mechanism. The
compiler validates equal logical width, opposite signedness, mutual and unique
pairing, and ownership; no declaration may replace the intrinsic pairing or pair
a type owned elsewhere.

One-sided roles have no paired public identity. Their counterpart relation may
return the unnamed exact intrinsic counterpart. Restricted or opaque semantic
identities gain no counterpart merely from integer storage.

## Integer associated types

Generic numeric code needs statically known type relationships:

```zax
MyBitCount :: alias type MyNumericType bit count type
MyByteCount :: alias type MyNumericType byte count type
MyDelta :: alias type MyNumericType delta type
MyDistance :: alias type MyNumericType distance type
MyStorage :: alias type MyNumericType storage type
MyCounterpart :: alias type MyNumericType signedness counterpart type
```

These are type-receiver operations returning concrete type identities with no
runtime construction. Future work must determine whether they are specialized
operator phrases, one general associated-type facility, type metadata queries,
or a combination.

The relationships must remain usable in result declarations and constraints
without forcing generic code to branch manually between ordinary and large
count tiers.

## Constraints this input places on current work

Optional wrapper composition adds a resolved-type intent requirement:

```zax
MyInner :: alias type MyValue?
MyOuter :: alias type MyInner?

inner : MyInner
outer : MyOuter = inner             // error: hidden optional depth is added
outerExplicit : MyOuter = [{ inner }]
```

A type parameter substitution may create the same relationship even when the
generic source does not spell both `?` layers. Future generic work must preserve:

- no automatic flattening of optional layers;
- one independent qualification and presence state per layer;
- construction-packet acknowledgement when a construction adds a layer around
  an already-optional source;
- same-type optional construction remaining ordinary;
- declared operations returning nested optionals retaining their exact result
  type and state contract.

Generic diagnostics must show enough resolved structure to explain why an
apparently simple initialization needs an explicit packet.

- a concrete type argument is never a runtime value;
- a type-receiver operation is not inherently compile-time;
- non-generic enclosing type identity is sufficient for current behavior; and
- current documentation must not imply that a general computed type result
  exists;
- integer-factory and associated-type examples remain illustrative until
  computed type results are designed; and
- public signedness pairs are declared relationally rather than inferred from
  names.

## Activation and retirement

Activate this input when generics, type parameters, constraints, computed type
results, type-receiver identity, integer factories, relational type pairs,
associated types, or generated type families are reviewed. Move accepted
behavior into generic, declaration, invocation, type, numeric, and reflection
owners, then retire this file after every preserved question is dispositioned.
