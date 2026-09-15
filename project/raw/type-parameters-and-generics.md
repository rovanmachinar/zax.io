# Raw input: type parameters and generics

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work defining generics, type parameters, constraints, or computed type results |
| Applies To | Type-input pressure exposed by operator-phrase, declaration, invocation, scalar-family, and composition review |
| Owns | Preserved type-parameter, generic, whole-type contract, intrinsic endian/fixed/floating specialization, numeric factory, bounded probing, language-limit, relational-pair, associated-type, representative-source, activation, and retirement questions |
| Does Not Own | Accepted generic semantics or current declaration/scalar/type behavior |
| Source / Provenance | Legacy `meta-types.md` and `meta-functions.md` evidence together with operator-phrase review of type parameters/type receivers, work item `012` optional-depth substitution pressure, work item `015` allocation-policy pressure, and work item `020` composition constraints |

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

### Whole-type abstract-contract pressure

Member-level composition roles do not yet provide a whole-type generic
constraint. Future work may need a no-storage contract that describes the
required shape of a type and can be used by generic constraints, structural
matching, or composition.

No source form for that whole-type contract is established, so this input does
not invent a representative declaration. Future work must decide:

- how the required value, callable, and operator shapes are written;
- whether matching is nominal, structural, explicitly fulfilled, or a
  combination;
- which names, qualifications, transfer stances, labels, and result-origin
  guarantees participate;
- how a generic body is checked against the contract; and
- how contract evolution affects existing specializations and fulfillments.

The facility must not turn member-level `abstract` metadata into inheritance,
runtime interface storage, a vtable, or implicit dynamic dispatch. A preferred
route also cannot manufacture a whole-type contract or cause one to be inferred.

[Zax structural shapes and compatibility](../../language/structural-shapes-and-compatibility.md)
now supplies the accepted distinction among identity, direct and flattened
shape, layout, operation permission, and whole-type conformance. Future generic
constraints must use those meanings without treating a compatibility posture as
a generic contract, making an anchor invent a deduction target, or turning
shape equality into implicit admission.

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

## Qualifier-parameterized declaration families

[Zax qualifiers](../../language/qualifiers.md) and
[transfer stances](../../language/transfer-stances.md) require every ordinary
callable prototype to resolve one exact mutable/immutable, final/varying,
writable/readonly, and transfer shape. Omission supplies concrete defaults; it
does not create qualifier erasure or implicit generic specialization.

A future explicit generic mechanism should let an author produce exact
specializations without hand-writing every valid combination.

Illustrative syntax:

```zax
relay$(Mutability, PlaceStance) final : (
  result : MyValue Mutability readonly PlaceStance &
)(
  input : MyValue Mutability readonly PlaceStance &
) = {
  // ...
}
```

The spelling is not accepted. Future work must decide:

- how qualifier parameters are named and constrained;
- whether qualifier arguments are inferred from value inputs;
- how a result repeats inferred input metadata;
- when the generic body is checked against all constraints;
- whether invalid specializations remain latent until demanded;
- how concrete declarations compare with inferred specializations;
- how generated lifecycle families interact with programmer generic families;
- whether stance itself may be a generic parameter;
- how diagnostics show the inferred concrete prototype;
- and how source compatibility changes when another specialization becomes
  viable.

Every selected specialization remains one exact concrete prototype. Only
demanded specializations need executable code, and implementation deduplication
must not erase language-level identity.

### Generic processing versus compatible prototypes

[Zax function invocation](../../language/function-invocation.md#minted-implementation-model)
defines a non-generic callable body as minted once under its implementation
prototype. A compatible visible prototype reuses that implementation without
reprocessing its body.

Future generics differ. A generic body may need to be reprocessed for each
demanded concrete type, qualification, stance, value, or other generic argument:

```text
generic body + concrete argument set
-> substitute the concrete contract
-> check and select body operations
-> mint one concrete implementation
```

Each concrete result has its own implementation-prototype facts. Reusing backend
code for equivalent layouts must not erase those language-level specializations.
Future work must define which facts are generic inputs, when reprocessing occurs,
and how diagnostics distinguish the generic source from one minted concrete
implementation.

Runtime surfaces are different. Composition `expose`, independently of `own`,
future function-value work, and any future dispatch work must select a finite
exact set rather than reserve space for every theoretical specialization of an
open generic family.

Future generic deduction must decide whether and when an independent
`preferred` route can participate after deduction. Composition currently
permits projection only after a concrete expected type or complete shape exists.
It must not invent a deduction target, widen member, callable, or operator
discovery, or outrank an exact direct match. Generic work must preserve that
boundary unless a later design explicitly replaces it.

## Intrinsic endian and numeric families

Endianness is now an intrinsic scalar specialization dimension rather than a
generated enum family. See [Zax endianness](../../language/endianness.md).

Future generic work must decide:

- how `Little`, `Big`, active, target, and compiler-host selectors appear as
  type arguments or factory inputs;
- when an environment selector resolves to one concrete endian identity;
- how absolute-endian, active, and public role aliases are generated;
- how fixed-point width, signedness, and bounded `F` are validated;
- how ordinary floating exponent/fraction dimensions select a closed encoding
  policy;
- how closed legacy formats avoid incoherent free parameter combinations; and
- how concrete specializations and aliases appear in reflection and
  diagnostics.

Every selected result is one exact closed specialization. Generic generation
does not create a second scalar semantic model.

## Integer specializations and factory selection

The concrete programmer-visible integer families and relationships are current
in [Zax integers](../../language/integers.md). This section retains only the
future generic/factory mechanism needed to produce them.

Future integer work assumes a built-in generic family conceptually like:

```zax
// Illustrative only.
MyI57 :: alias type Integer$(
  LogicalBits = 57,
  Signedness = Sign.Signed,
  FractionalBits = 0,
  Endianness = Endian.Little
)
```

Each realized combination is a concrete, closed intrinsic type. It is not still
generic after realization.

A separate integer factory returns one concrete specialization from dimensions
such as:

- minimum and maximum logical width;
- signedness;
- concrete or environment-selected endianness;
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
Integer$(W, Signed, F, Endian) <-> Integer$(W, Unsigned, F, Endian)
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

## Fixed-point specializations and full products

Current format and arithmetic semantics are defined by
[Zax fixed-point scalars](../../language/fixed-point-scalars.md). Generic work
retains:

- declaration syntax for `Integer$(W, Signedness, F, Endianness)`;
- validation of `0 <= F <= W - 1` for signed and `0 <= F <= W` for unsigned;
- normalization of `F = 0` to the existing integer specialization;
- generation of `I<W>F<F>`/`U<W>F<F>` and Q aliases;
- constraints requiring one exact fixed-point identity;
- associated full-precision-product result selection; and
- exposure of coefficient, quantum, range, and endian facts to generic code.

For two like `W`, `F` operands, `full precision product` needs an associated
type sufficient for the complete coefficient product and `2F` fractional bits.
Exact result-type query syntax remains unsettled.

## Floating specializations and language limits

Current ordinary, profile-selected, and legacy format semantics are defined by
[Zax binary floating-point scalars](../../language/floating-point-scalars.md).
Generic work retains:

- ordinary IEEE-style exponent/fraction parameter syntax;
- selection among closed encoding policies;
- prevention of freely combined explicit-integer-bit modes until another
  format requires one;
- `Float`, `FastF<N>`, `LeastF<N>`, and maximum-role factories;
- support-class and operation constraints;
- exact environment and endianness selection; and
- immutable type-owned `Scalars.Floating.LanguageLimits` values.

The numeric value of `LanguageLimits.maximumWidth`, and any separate exponent,
fraction, storage, or alignment limits, remain future language-version
decisions. A limit must be available while concrete types are formed; a
type-receiver or `once` declaration is not inherently compile-time merely
because it belongs to a type.

### Bounded size probing

Maintainer literal input preserves a concrete generic compile-time selection
need:

```zax
mySmall := h'FF'
myLarge := h'FFFFFFFFFFFFFFFFFFFFFFFFFFFFF'
```

A numeric literal implementation receives a payload `String`, but the parsed
magnitude may determine whether its final result is an ordinary or much wider
exact integer. The baseline literal facility may require one declared concrete
result. Future generic work should still permit a literal declaration or
associated type factory to compute one concrete result identity from its
compile-time payload before the literal enters its surrounding expression.

More generally, compile-time code should be able to begin with an ordinary
numeric size, test bounded conversion/support requirements, and deliberately
advance through larger candidates until a language/profile maximum is reached.
It should not be forced to instantiate one extreme maximum-width integer or
floating value merely to discover a workable type.

Future generic and compile-time work must define this as bounded static
selection rather than exception-driven runtime fallback, source-order overload
search, or reopening the selected literal as an uncommitted integer.

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

### Cursor-protocol constraints

[Zax iteration](../../language/iteration.md#cursor-driven-from-traversal)
defines the concrete cursor protocol without requiring a generic concept:

```text
iterate source
value at cursor
advance cursor
erase and advance cursor
erase cursor
```

Future generic work must let a declaration constrain an unknown source or
cursor by the protocol pieces it actually needs. It must be possible to express:

- whether `iterate` returns one cursor or an optional cursor;
- the stable concrete cursor type;
- whether `value at` exists and its value, reference, or proxy result shape;
- the relationship between a yielded reference and the source or cursor life
  path;
- whether progression mutates cursor state and returns `Boolean` or returns a
  replacement cursor, optionally absent;
- whether erase-and-advance or erase-and-exit is available;
- required source, cursor, and yielded-value qualifications;
- whether the cursor may be copied, moved, stored, or returned; and
- cursor/source mutation and invalidation guarantees.

A `#` traversal requires progression but no `value at` result. Constraints must
therefore compose protocol capabilities rather than require one monolithic
iterator interface.

Generic syntax must describe these existing operations. It must not introduce a
second cursor protocol or reinterpret concrete `each from` behavior.

### Allocation-policy constraints

[Zax pointers, allocation, and arenas](../../language/pointers-and-arenas.md)
allows the same pointer type to carry different allocation disposition,
collection, arena, and control-block history. `T * strong` alone therefore
cannot prove prompt destruction, prompt recovery, collectability, a particular
arena capability, or cross-thread release.

Future generic work must define how a declaration constrains:

- object and control-block arena capabilities;
- prompt, deferred-recovery, or attached-lifespan disposition;
- collection eligibility;
- declaration-attached versus open-ended raw allocation;
- and whether a supplied allocation policy may be propagated to a nested
  declaration.

Those constraints must remain visible in callable compatibility and
specialization. They must not infer a guarantee from hidden allocation history.

- a concrete type argument is never a runtime value;
- a type-receiver operation is not inherently compile-time;
- non-generic enclosing type identity is sufficient for current behavior; and
- current documentation must not imply that a general computed type result
  exists;
- integer-factory and associated-type examples remain illustrative until
  computed type results are designed; and
- fixed/floating factory, full-product, and `LanguageLimits` examples remain
  illustrative until the same mechanism is designed; and
- public signedness pairs are declared relationally rather than inferred from
  names.

## Activation and retirement

Activate this input when generics, type parameters, constraints, whole-type
abstract contracts, computed type results, type-receiver identity, intrinsic
integer/fixed/floating families, endian selection, numeric factories,
language-limit values, bounded size probing, relational type pairs, associated
types, cursor-protocol constraints, or generated type families are reviewed.
Move accepted behavior into generic, declaration, invocation, type, numeric,
iteration, and reflection owners, then retire this file after every preserved
question is dispositioned.
