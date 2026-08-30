# Raw input: type parameters and generics

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work defining generics, type parameters, constraints, or computed type results |
| Applies To | Type-input pressure exposed by operator-phrase, declaration, and invocation review |
| Owns | Preserved type-parameter and generic questions, representative source, activation pressure, and retirement criteria |
| Does Not Own | Accepted generic syntax, constraints, instantiation, identity, reflection, or compile-time behavior |
| Source / Provenance | Legacy `meta-types.md` and `meta-functions.md` evidence together with operator-phrase review of type parameters and type receivers |

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

## Constraints this input places on current work

- a concrete type argument is never a runtime value;
- a type-receiver operation is not inherently compile-time;
- non-generic enclosing type identity is sufficient for current behavior; and
- current documentation must not imply that a general computed type result
  exists.

## Activation and retirement

Activate this input when generics, type parameters, constraints, computed type
results, type-receiver identity, or generated type families are reviewed. Move
accepted behavior into generic, declaration, invocation, type, and reflection
owners, then retire this file after every preserved question is dispositioned.
