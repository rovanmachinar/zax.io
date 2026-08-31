# Raw input: owned composition and exposed identity surfaces

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work defining `own`, type composition, identity-aware surface exposure, or composition sugar |
| Applies To | Private owned composition, exposed/opaque identity surfaces, signature substitution, projection/admission boundaries, suppression, and reflection |
| Owns | Preserved `own` and identity-exposure requirements, representative source, activation pressure, and retirement criteria |
| Does Not Own | Accepted owned-composition semantics or current identity/partial behavior |
| Source / Provenance | Integer identity work and the language direction favoring composition over inheritance |

Current explicit identity, admission, projection, and exposed/opaque behavior is
owned by [Zax identity types](../../language/identity-types.md). This input
retains the future general `own` transformation and filtering mechanism.

## Mental-model pressure

An exposed identity behaves conceptually like a private owned composition:

```zax
MyInteger :: type {
  boxed own private : Integer
}
```

This is a semantic mental model, not a required literal expansion or compiler
lowering. Zax does not gain class inheritance.

The identity remains distinct while preserving the underlying representation
shape and exposing an adjusted subset of behavior. An opaque identity uses the
same representation relationship without automatic surface exposure.

## General-purpose scope

Integer and scalar identities supplied the immediate pressure, but `own` is a
general composition mechanism. It may expose or adapt behavior from any
contained type and must not be designed as numeric-only identity machinery.

```zax
// Illustrative future `own` use on a nonnumeric contained service.
MyLoggingService :: type {
  myService own private : MyService
}
```

Future teaching needs both numeric and nonnumeric examples.

## Signature transformation

Exposure cannot publish source declarations unchanged. Self-type inputs and
results need meaning-preserving substitution:

```text
Integer + Integer -> Integer
```

becomes conceptually:

```text
MyInteger + MyInteger -> MyInteger
```

External result domains such as `Boolean` and associated count types remain
external. Future work must classify:

- self-type parameters and results;
- optional and reporting structures containing self-type;
- associated result and counterpart types;
- constructors and destruction;
- type-receiver operations;
- qualifications and transfer behavior;
- private declarations;
- operations whose meaning does not survive substitution; and
- source additions that would expand an exposed identity later.

## Projection and admission exclusions

Arbitrary source conversion, identity admission, and unsafe boundaries cannot be
forwarded blindly.

An identity owns projection to its immediate source type and admission through
its declared `admit` or `restricted` policy. Equal representation does not create
a sibling identity conversion.

An exposed integer identity may receive the protected exact, optional, and
narrowing integer conversion surface. An opaque identity receives only its
immediate projection unless it declares more.

## Suppression and partial boundaries

An original `own` or identity-exposure declaration may need a way to filter
behavior that cannot or should not be exposed. A later `partial` declaration
only adds and therefore cannot suppress, hide, restrict, remove, or replace an
exposed declaration.

Partial ownership and extension authority are preserved separately in
[partial-type input](partial-types.md).

## Representation and reflection pressure

Future work must decide:

- whether owned single-value composition guarantees equal size, alignment, and
  base address;
- whether the contained value remains directly nameable inside the owner;
- how identity and representation are reported;
- how aliases and partial definitions appear;
- how forwarded versus locally declared operations are distinguished; and
- whether adding behavior to the source is a compatibility event for exposed
  identities.

## Construction, transfer, and same-storage views

By-value identity admission needs a viable copy, move, consuming/`last`
transfer, direct construction, or another declared way to establish the
underlying stored value. `own` cannot manufacture copyability.

A reference result may instead create an identity-typed view of existing
underlying storage:

```zax
// Illustrative future behavior.
myView : MyIdentity & = MyIdentity from myUnderlying
```

This performs no copy and creates no independent destruction obligation. Future
work must define:

- equal shape, size, alignment, and base address;
- aliasing and coexistence of underlying/identity views;
- qualifications and declaration-side permissions;
- lifetime and temporary-source behavior;
- identity invariants and admission policy;
- lifecycle function compatibility; and
- optimizer assumptions about differently typed views.

The compiler must not silently return a reference when source requests a
by-value identity.

### `cast as` pressure

Representation-compatible same-storage identity views motivate a
compiler-recognized operation conceptually like:

```zax
// Illustrative; exact spelling is not established.
myUnderlying cast as MyIdentity &
```

`own` supplies the shape relationship. Identity admission determines whether the
view is ordinary, optional, unchecked, unsafe, or unavailable. The compiler may
generate the bridge because a sealed underlying type cannot know every future
identity.

This input owns the primary future pressure until a dedicated casting owner
exists. Structural-typing, qualifier, and lifetime inputs retain their local
compatibility consequences.

## Activation and retirement

Activate this input for `own`, owned members, composition sugar, exposed/opaque
identity surfaces, signature substitution, identity projection/admission,
same-storage identity views, representation casts, or composition reflection.
Move accepted behavior into composition, identity, type, declaration, operator,
layout, lifetime, casting, and reflection owners, then retire this file after
every requirement is dispositioned.
