# Raw input: mutability-indexed type families

| Field | Value |
| --- | --- |
| Status | Raw candidate / non-authoritative |
| Audience | A future numbered work item reviewing type definitions, type families, structural compatibility, representation specialization, or qualifier-indexed APIs |
| Applies To | Preserved legacy motivation and a candidate framing for distinct mutable and immutable implementations under one named family |
| Owns | Provenance, use cases, current constraints, unresolved questions, activation pressure, and retirement criteria |
| Does Not Own | Accepted type-family behavior, syntax, identity, layout, conversion, defaults, reflection, structural compatibility, or implementation |
| Source / Provenance | Distinct mutable/immutable implementation proposals formerly in `mutable.md`, recovered during work item `004` promotion review and reframed through language-maintainer discussion |

## Reading posture

This file preserves a design pressure that does not belong in ordinary qualifier
documentation and was not rejected during work item `004`.

The former mutability page proposed more than qualifying one shared
representation. It allowed one named type to have distinct mutable and immutable
implementations with different members, operations, and layouts. That material
was removed from the public path because its detailed behavior crossed
unreviewed type-definition and structural-typing boundaries, not because the
underlying use case lacked merit.

Everything below remains candidate input.

## Motivating use case

A mutable collection may need representation and behavior that an immutable
collection does not:

- spare capacity;
- growth and compaction bookkeeping;
- mutation-specific synchronization;
- copy-on-write state;
- mutation-only operations; or
- different allocation and storage strategies.

The legacy proposal used a form resembling:

```zax
MyType :: type mutable {
    count : Integer
    capacity : Integer
}

MyType :: type immutable {
    count : Integer
}

mutableValue : MyType mutable
immutableValue : MyType immutable
```

This syntax is evidence, not accepted design.

The immutable form can omit `capacity` because it never grows. It may also expose
only operations meaningful for stable data. The mutable form can carry
additional state and mutating operations without forcing the immutable
representation to pay their storage or API cost.

The idea was partly motivated by mutable and immutable collection families in
Objective-C. Separately named mutable and immutable types can express the same
distinction, but a shared family name may provide a clearer conceptual
relationship and more direct contextual selection.

## Candidate framing

A useful framing is a **mutability-indexed type family**.

Under this candidate model:

- `MyType mutable` and `MyType immutable` are distinct concrete variants in one
  named family;
- the variants may have different stored members, functions, layouts, and
  construction strategies;
- the family may instead use one shared implementation when specialization has
  no value;
- qualification selects or preserves a concrete variant rather than merely
  changing permissions on one representation;
- family membership does not by itself make the variants layout-compatible or
  reference-convertible; and
- conversions exist only when explicitly supplied or justified by a future
  consuming transformation.

This framing is a proposal for later work, not terminology accepted by current
language design.

## API-surface pressure

An immutable variant may intentionally expose a smaller API surface:

- observation;
- traversal;
- comparison;
- serialization; and
- other operations that do not require mutation.

A mutable variant may add mutation, reserve-capacity, growth, compaction, and
other operations.

Future structural compatibility may allow either variant to satisfy a smaller
readonly contract without requiring every compatible type to implement the
mutable operations. This could reduce accidental API requirements while keeping
mutable functionality available where requested.

That pressure is distinct from representation specialization:

- structural or composition-based interfaces can describe a common API;
- distinct implementations can optimize representation; and
- a named family can express the relationship and contextual variant selection.

The language may ultimately use one, two, or all three mechanisms.

## Composition alternative

Legacy composition material proposes that `own` can express separately named
types sharing an interface-like component:

```zax
ReadableType :: type {
    count : Integer
    read override : (result : Value)(index : Integer)
}

MyImmutableType :: type {
    readable own : ReadableType
}

MyMutableType :: type {
    readable own : ReadableType
    capacity : Integer
}
```

This remains a viable fallback and may be simpler than introducing type-family
semantics.

It does not necessarily provide:

- one discoverable public family name;
- contextual mutable/immutable default selection;
- family-aware overloads or generics;
- an explicit counterpart relationship; or
- a natural location for declared cross-variant construction.

Whether those advantages justify a type-family feature remains open.

## Constraints from current qualifier design

Any future design must preserve the accepted qualifier model.

### Readonly retains underlying mutability

Readonly access does not erase which concrete variant exists:

```zax
mutableSource final : MyType mutable
immutableSource final : MyType immutable

mutableReader :
    MyType mutable readonly final & = mutableSource
immutableReader :
    MyType immutable readonly final & = immutableSource
```

The first path observes a mutable variant that another path may change. The
second observes an immutable variant whose underlying value remains stable
through ordinary safe behavior. Both references preserve the final stance of
their concrete source places.

An omitted mutability in `MyType readonly` must resolve from the source,
referent, or contextual default. It must not mean an unresolved union of both
variants.

### Qualification is not representation conversion

If variants have different layouts:

- mutable-to-readonly may create a restricted path to the same mutable variant;
- mutable-to-immutable cannot be an ordinary reference or pointer
  requalification;
- immutable-to-mutable cannot be an ordinary reference or pointer
  requalification;
- `unsafe pliable` bypasses restrictions on one path but does not change the
  concrete variant or representation; and
- an unsafe qualifier cast does not by itself construct a differently laid-out
  variant.

Cross-variant conversion requires construction, copying, or a consuming
transformation that establishes the destination representation and guarantees.

### Defaults select only unresolved mutability

A contextual mutability default may select the family variant when a direct
declaration provides no source or explicit mutability.

Inherited or explicit mutability resolves the variant before the default applies.
Defaults do not override the variant carried by a source or referent.

### Deep immutability remains applicable

An immutable variant remains recursively immutable over its contained values
under ordinary safe behavior. Different layout does not weaken that guarantee.

## Identity and compatibility questions

Future work must distinguish:

- the family name;
- each concrete family variant;
- a type use with access and place qualification;
- structural shape;
- API compatibility;
- layout compatibility; and
- conversion compatibility.

Questions include:

1. Is the family name itself a type, a type constructor, a lookup group, or
   another concept?
2. Is `MyType mutable` nominally distinct from `MyType immutable`, structurally
   distinct, or both?
3. When a variant uses its family name recursively, does that name resolve to
   the selected concrete variant or to the family?
4. Are nested types, compile-time values, and other non-instance declarations
   shared or variant-specific?
5. How are common members and functions declared without duplication?
6. Can one variant compose implementation from another?
7. Can an overload accept exactly one variant, either variant, or the family as
   a constrained set?
8. How do generic parameters express a variant requirement?
9. Does reflection expose family membership separately from concrete identity?
10. How do structural compatibility and anonymization treat family membership?
11. Can independently declared types satisfy a family variant's API without
    adopting its name?
12. What source and binary stability expectations apply when one variant changes
    layout?

## Default-selection questions

The legacy material proposed selecting one variant as the family default.
Future work must decide:

- whether a family must have a default variant;
- whether contextual qualifier defaults are sufficient;
- whether a type definition may establish its own default;
- how an explicit family default interacts with source-level default directives;
- whether omission can be rejected in selected contexts; and
- how diagnostics identify an ambiguous or unavailable variant.

Work item `004` established only that baseline mutability defaults are
configurable and apply after inherent, inherited, and explicit qualification.

## Conversion questions

Separate representations make conversion an operation rather than a view:

- mutable to immutable may copy, build, freeze, or consume;
- immutable to mutable may allocate capacity or initialize mutation-only state;
- either direction may be unsupported;
- conversion cost must remain visible; and
- cross-variant references are invalid unless a future shared representation or
  projection explicitly supports them.

Future work must decide how these operations participate in overload selection,
construction, move/copy policy, failure handling, and reflection.

## What this input does not decide

This file does not establish:

- the example syntax;
- that type families will exist;
- that every type supports mutable and immutable variants;
- that variants share layout, members, functions, or identity;
- automatic conversion in either direction;
- implicit API subtyping;
- a family-level default mechanism;
- how `own` composition interacts with variants;
- how qualifiers participate in structural equivalence; or
- any compiler or transpiler representation.

## Activation pressure

Read this file when a focused work item reviews one or more of:

- type definitions with representation specialization;
- qualifier participation in type identity;
- structural API or layout compatibility;
- anonymization;
- generic constraints over type families;
- reflection of type identity and qualifiers;
- mutable/immutable construction and conversion;
- recursive family-name resolution inside related implementations; or
- common API expression through composition.

It is not required reading for ordinary qualifier use.

## Intended disposition and retirement

The future work item must:

1. evaluate the use cases against separately named types and `own` composition;
2. decide whether a named family abstraction has enough value;
3. disposition the representative syntax and every legacy conversion claim;
4. move accepted behavior into type-definition, qualifier, structural-typing,
   reflection, function, or conversion owners as appropriate;
5. record explicit rejection if the feature is not adopted; and
6. remove this file from the raw index and retire it when no unresolved material
   remains.
