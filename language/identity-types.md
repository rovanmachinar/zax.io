# Zax identity types

| Field | Value |
| --- | --- |
| Status | Current conceptual design |
| Audience | Human developers defining, reading, or converting aliases and representation-related types |
| Applies To | Transparent type aliases and distinct identities over existing types; not a formal grammar or specification |
| Implementation State | Not established by this repository |
| Owns | Transparent aliases; identity boundaries; immediate underlying type/value/place operations; identity declarations and original-owner body authority; admission; identity projection; exposed and opaque surfaces; contextual-posture reset and non-forwarding; declared bridges; construction/transfer requirements; costs, diagnostics, and source stability |
| Does Not Own | Integer-family membership and numeric-source realization ([integers](integers.md), [integer literals and realization](integer-literals.md)); qualifier semantics ([qualifiers](qualifiers.md)); complete owned-composition transformation; partial-extension authority; or structural type equivalence |
| Source / Provenance | Legacy alias and enum evidence refined through fundamental-integer and conversion review |

## Two ways to build on an existing type

Zax can give an existing type another name, or define a new type that uses an
existing type as its underlying type.

```zax
MyShortName :: alias type SomeLongTypeName
MyCount :: identity admit expose type U32
```

`MyShortName` and `SomeLongTypeName` are the same type. The alias only provides
another name.

`MyCount` and `U32` are different types. `MyCount` uses `U32` as its underlying
type, so it begins with the same representation and value capacity. The
identity declaration decides how `U32` values may create `MyCount` values and
whether `U32` functions and operators become available on `MyCount`.

```zax
myRaw : U32 = 10
myCount : MyCount = MyCount from myRaw

myRaw = myCount // error: different identities
```

The explicit `from` states where the program crosses between them.

## Transparent aliases

```zax
MyShortName :: alias type SomeLongTypeName
```

A transparent alias is another name for one existing type:

- both names refer to the same type identity;
- no conversion is required between the names;
- both names have the same values, operations, representation, and
  qualifications; and
- the alias creates no additional overload domain.

```zax
myOriginal : SomeLongTypeName
myAlias : MyShortName = myOriginal
```

Diagnostics may retain the alias spelling when that helps explain source:

```text
MyShortName (alias of SomeLongTypeName)
```

That presentation does not make the alias another type.

## Identity boundary and underlying type

An **identity boundary** separates types that are not interchangeable even when
their representation is the same.

Assignment, argument passing, return, and operator matching do not cross the
boundary unless the language or type declares an explicit operation:

```zax
myCount : MyCount
myRaw : U32

myCount = myRaw // error: identity boundary
myCount = MyCount from myRaw
```

The **identity type** is the new outer type, such as `MyCount`.

The **underlying type** is the existing type on which it is based, such as
`U32`.

The **underlying value** is the value represented before admission or returned
after identity projection.

An identity boundary is semantic, not necessarily expensive. Crossing it may
require no representation change while still requiring explicit source.

## Identity declaration

An identity declaration answers two independent questions:

1. May every valid underlying value create this identity automatically?
2. Which functions and operators from the underlying type become available?

Both answers are explicit:

```text
Name :: identity <admission> <surface> type UnderlyingType
```

Exactly one admission keyword is written:

| Keyword | Meaning |
| --- | --- |
| `admit` | Permit ordinary `from` for every valid underlying value when the requested construction or transfer is viable |
| `restricted` | No ordinary admission is generated; the identity declares validation, unchecked admission, unsafe admission, or no admission |

Exactly one surface keyword is written:

| Keyword | Meaning |
| --- | --- |
| `expose` | Make eligible underlying functions and operators available after adapting them to the new identity |
| `opaque` | Do not make underlying functions or operators available automatically |

The four combinations serve different purposes:

```zax
MyInteger :: identity admit expose type Integer
MyValidatedInteger :: identity restricted expose type Integer
MyToken :: identity admit opaque type U64
MyDocumentHandle :: identity restricted opaque type Integer
```

There is no omission default for either choice. Admission and exposure state
consequential intent.

### Original definition body and sealing

An identity without a body completes and seals as soon as its declaration
finishes:

```zax
MyInteger :: identity admit expose type Integer
```

An original definition body may keep the identity open while its owner adds
constructors, functions, and operators:

```zax
// Illustrative future identity-body/completion syntax.
MyContextualInteger :: identity admit expose type Integer {
  +++ contextual final : ()(rhs : Integer) = {
  }

  operator binary '+' contextual existing

  double final : ()() = {
    _ *= 2
  }
}
```

This is a compile-time definition phase, not a runtime state. No code observes a
partially defined identity. The closing brace completes and seals the type;
later additions require whatever authority the future partial mechanism
provides.

The body belongs to the original identity owner. It does not introduce class
inheritance: the identity remains a distinct representation/composition-based
type with the admission and exposure choices stated by its header.

A new identity resets inherited completion posture to `explicit`. The owner
must opt its construction and exposed operator into `contextual` completion
separately:

```zax
myValue := 0 + (: MyContextualInteger = 5)
```

Exact identity-body and `contextual existing` grammar remains future source
integration. Completion behavior is defined by
[Zax operators](operators.md#contextual-completion).

### Integer underlying types do not grant integer membership

An identity does not become an intrinsic integer merely because its underlying
type is an integer:

```zax
MyInteger :: identity admit expose type Integer
myValue := 0 + (: MyInteger = 5)
// error by default: identity construction and exposed + are explicit
```

`admit` supplies declared admission and `expose` adapts eligible operations to
the new identity. Neither grants fundamental integer realization or general
implicit admission from numeric source. Contextual completion is an independent
opt-in owned by the identity declaration.

A transparent alias remains the underlying integer type and retains its
behavior. A future custom numeric-family mechanism may define genuine
programmer-supplied integer participation; ordinary identity declarations do
not. See
[Zax integer literals and realization](integer-literals.md#identities-do-not-become-integers-through-storage).

## Admission

Admission creates a value of an identity type from a value of its underlying
type. A type family may also declare admission from another explicitly related
type.

### Ordinary admission

An `admit` identity supplies ordinary `from` for its underlying type:

```zax
myRaw : U32 = 10
myCount : MyCount = MyCount from myRaw
```

Every valid `U32` value is a valid underlying value for `MyCount`, so no
semantic validation is required.

Type-specific families may declare other ordinary admissions. For example,
integer role identities may accept values from another integer type when its
complete value range fits. Those relationships belong to the type family that
defines them.

### Restricted admission

A `restricted` identity receives no automatic ordinary `from`:

```zax
myHandle : MyDocumentHandle =
  MyDocumentHandle from myRaw // error unless MyDocumentHandle declares it
```

The identity may instead declare one or more policies:

```zax
myHandle : MyDocumentHandle? =
  MyDocumentHandle optional from myRaw

myUnchecked :=
  MyDocumentHandle unchecked from myRaw

myUnsafe :=
  MyCapability unsafe from myBits
```

- `optional from` validates and returns absent when the value is rejected.
- `unchecked from` skips semantic validation but retains defined mechanical
  behavior.
- `unsafe from` bypasses an invariant required for memory, lifetime,
  representation, concurrency, or optimizer correctness.

`restricted` does not decide which of these exists. The identity's invariants
decide whether admission is validated, unchecked, unsafe, or unavailable.

The resulting wrapper's construction, qualification, and lifetime behavior is
defined by [Zax optional values](optional-values.md).

Complete safe-subset classification remains future safety work.

## Identity projection

Identity projection takes an identity value and returns its underlying value
without changing the represented value:

```zax
myCount : MyCount
myRaw := myCount as U32
```

`myRaw` is the `U32` value represented by `myCount`. Projection changes the
static type from `MyCount` to its underlying `U32`; it does not make another
identity valid.

An opaque identity receives this immediate projection without receiving every
operation of its underlying type.

Type-specific exposed identities may provide additional conversions. Integer
identity conversion is taught by [Zax integers](integers.md#integer-identity-types).

## Direct bridges between identities

Two identity types defined independently from the same underlying type remain
unrelated:

```zax
MyDocumentHandle :: identity restricted opaque type Integer
MyFileHandle :: identity restricted opaque type Integer

myDocument : MyDocumentHandle
myFile : MyFileHandle = myDocument // error
```

The general route is explicit projection followed by admission:

```zax
myFile :=
  MyFileHandle unchecked from (myDocument as Integer)
```

A direct bridge exists only when the type family declares that relationship.
Representation equality alone is insufficient.

For example, integer size and index identities may deliberately declare exact
bridges because their ranges and representations are equal:

```zax
mySize : TypeSize = myIndex as TypeSize
myIndex : IndexSize = mySize as IndexSize
```

## Exposed and opaque behavior

`expose` makes eligible functions and operators of the underlying type
available on the identity type. Parameter and result types are changed where
necessary so the operation uses the new identity.

```text
Integer + Integer -> Integer
```

becomes conceptually:

```text
MyInteger + MyInteger -> MyInteger
```

Results belonging to another domain remain in that domain. An integer
comparison still returns `Boolean`; a shift count still uses the integer's
associated count type.

Exposure is conceptually related to private owned composition:

```zax
// Conceptual expansion; exact `own` behavior remains future work.
MyInteger :: type {
  boxed own private : Integer
}
```

This is a programmer model, not required compiler lowering and not class
inheritance.

Constructors, admission, projection, nested self-types, private declarations,
and operations whose meaning cannot survive type substitution require
deliberate rules. Complete transformation and filtering remain future
owned-composition work.

`opaque` performs none of this automatic exposure. The identity may still
declare its own functions and operators.

### Immediate underlying operations do not forward

The original definition body normally uses the adapted identity surface:

```zax
double final : ()() = {
  _ *= 2
}
```

When owner code needs the underlying relationship directly, each identity
boundary provides operations for exactly its immediate layer:

```zax
UnderlyingType := MyIdentity underlying type
myUnderlying := myIdentity underlying value
```

`underlying type` returns the immediate underlying type. `underlying value`
returns one copied/projected value of that type. Existing
`as UnderlyingType` projection remains the explicit conversion-shaped route to
the same immediate underlying value.

`underlying place` is generated as a private final post-unary operation. It is
eligible only inside the private context implementing this immediate identity
boundary and grants no stronger place, value, access, or lifetime permission
than the receiver path already carries:

```zax
doubleUnderlying final : ()() = {
  (_ underlying place) *= 2
}
```

The operation produces a place, not a first-class reference with independent
alias or escape guarantees. A function may bind a reference from that place
when ordinary permissions allow it.

`underlying type`, `underlying value`, and `underlying place` describe the
current boundary and are not mechanically forwarded by `expose`. An outer
identity regenerates them for its own immediate layer. It may obtain a copied
deeper value without gaining the inner identity's private place access:

```zax
myDeeperValue := (_ underlying place) underlying value
myDeeperValue.doSomething()
```

The first operation reaches the outer identity's immediate underlying place.
The second invokes the inner value's public copied/projected operation. Changes
to `myDeeperValue` do not mutate the nested stored value.

A second `underlying place` is unavailable unless the current code is also
private-eligible for that inner identity or the inner owner deliberately
publishes another access operation. Wrapping a private identity does not grant
permission to tunnel through it.

Optional naming of the underlying place and complete owned-component
qualification/lifetime behavior remain future owned-composition work.

## Construction and transfer

By-value admission requests a new identity value:

```zax
myIdentity := MyIdentity from myUnderlying
```

The operation needs a viable way to establish the underlying stored value:

- copy;
- move;
- consuming/`last` transfer;
- direct construction; or
- another declared transfer.

Identity syntax does not manufacture copyability. If no applicable construction
or transfer exists, by-value admission is unavailable.

A same-storage identity reference is a different operation. It would create a
view of existing storage rather than a new value, perform no copy, and schedule
no independent destruction. Exact reference-admission, representation-cast,
alias, qualifier, lifetime, and type-receiver rules remain future
owned-composition and lifetime work.

The compiler never silently substitutes a reference view for a requested
by-value result.

## Representation and qualifications

An identity begins with its underlying representation and adds no instance
storage merely because it has another identity. Complete base-address,
alignment, shape, and same-storage-reference guarantees remain future
owned-composition/layout work.

Identity does not weaken qualifications. Projection, admission, construction,
and transfer use the applicable place, value, access, copy, move, and lifetime
rules.

Storage compatibility never grants implicit conversion.

## Partial definitions

A future partial mechanism may let an authorized owner add functions to an
identity:

```zax
// Illustrative future partial syntax.
MyIntegerExtension :: partial MyInteger {
  // Additional MyInteger behavior.
}
```

Partial definitions add; they do not suppress, hide, restrict, remove, or
replace behavior exposed by the original identity declaration.

Whether sealed intrinsic or identity types accept language-, compiler-,
CPU-provider-, owner-, or programmer-supplied partial functions remains future
partial-type work.

## Costs and diagnostics

An identity may share representation with its underlying type while still
introducing meaningful work or constraints:

- explicit construction or transfer;
- validation and optional failure;
- an unchecked or unsafe assertion;
- copies, moves, references, and temporaries;
- another overload domain; and
- exposed or locally declared operations.

Diagnostics should distinguish:

- transparent alias from distinct identity;
- the identity type from its underlying type;
- missing ordinary admission from failed validation;
- optional, unchecked, and unsafe admission;
- immediate identity projection from unavailable identity-to-identity
  conversion;
- equal representation without a declared bridge;
- an operation absent because the identity is opaque; and
- unavailable by-value admission because no construction or transfer applies.

Diagnostics lead with source/public names and make underlying representation
detail available on demand.

## Source stability

- Changing a transparent alias changes the type denoted by that name.
- Changing an identity's underlying type may change layout, value capacity, and
  cost without erasing its identity boundary.
- Changing `admit`/`restricted` or `expose`/`opaque` changes source validity.
- Adding a direct bridge enables previously invalid identity conversion.
- Changing underlying behavior may affect an exposed identity under future
  owned-composition rules.

These are compatibility events, not invisible implementation choices.

## Boundaries and maturity

This document is current conceptual design, not formal grammar, a complete
layout/ABI contract, or an implementation mapping.

Future work owns complete `own` transformation, partial authority,
same-storage identity views, representation casts, generic identity factories,
reflection shape, enum validity, and structural type equivalence. Those areas
must preserve the explicit identity boundary and admission/exposure choices
defined here.
