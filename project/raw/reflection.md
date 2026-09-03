# Raw input: reflection

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work defining reflection, type metadata, declaration metadata, or source reflection |
| Applies To | Reflection pressure exposed by operator-phrase, type-identity, integer-family, and source-presentation review |
| Owns | Preserved reflection, integer metadata, representative-source, activation-pressure, and retirement questions |
| Does Not Own | Accepted reflection semantics or the current domain facts reflected |
| Source / Provenance | Legacy `meta-types.md`, `meta-functions.md`, and `enums.md` metadata evidence together with operator-phrase review of `type of`, qualifier predicates, phrase fences, and work item `012` optional-layer metadata pressure |

## Why this input exists

Current documentation reserves a small, deliberately bounded set of reflection
concepts and defers everything else. See the
[operator catalog](../../language/operator-catalog.md#reserved-phrase-forms).

Reserved now:

- exact `type of`;
- a type-information operation whose exact words remain open;
- the four direct predicates `is constant`, `is final`, `is immutable`, and
  `is readonly`; and
- future declaration reflection whose exact form remains open.

Not reserved: any general `expression information of` or `access information of`
form. This file keeps the deferred questions live.

## `type of`

```zax
Selected := type of expression
```

`type of` returns the selected expression's concrete static type identity. It
does **not**:

- execute the expression merely to discover its type;
- choose among result-only overloads;
- choose between operator-phrase trees;
- create a type for zero or multiple results; or
- attach expression-specific facts to canonical type identity.

Every successfully selected single expression result has a concrete static type,
even when that type is anonymous, intermediate, structural, or known only to the
compiler.

Future work must decide how `type of` renders an anonymous or intermediate type,
whether the result is usable everywhere a named type is, and how it interacts
with qualification.

## Type-information wording and metadata shape

The type-information operation is reserved in concept only. Illustrative wording
such as:

```zax
// Illustrative wording only; no exact source form is reserved.
info := type information of MyType
```

is not a source reservation. Future reflection work selects the exact words.

The metadata itself should be immutable, readonly, and final. One unified
metadata structure may carry optional type-, expression-, declaration-, or
access-specific sections rather than requiring an unrelated result type for every
subject.

Future work must decide:

- the exact words;
- the metadata structure and how optional sections are discovered;
- whether metadata is a value, a type identity, or both;
- lifetime and storage of metadata; and
- what is available at runtime versus only at compile time.

## Alias versus canonical identity

Canonical type identity must remain independent of which expression produced it.
Future work must decide:

- whether an alias is visible through reflection or erased to its target;
- whether qualified type uses have distinct reflected identities;
- how a generic instantiation reports its parameters; and
- how recursive and self-referential types are reported without infinite
  expansion.

## Declaration reflection

The direct predicate list is intentionally small. Declaration properties such as
`private`, export status, deprecation, and attached directives belong to future
declaration reflection rather than to an expanding `is <property>` family.

Future work must decide:

- the form of declaration reflection;
- which declaration properties are exposed;
- whether reflection can observe a private member from outside its permitted
  context; and
- whether a declaration's replacement permission — as distinct from the
  underlying place stance reported by `is final` — is reflectable.

That last question is a live consequence: `is final` deliberately reports the
resolved type-use or referent-place truth, so a program currently has no way to
ask whether *this* declaration may exercise replacement.

## Optional layer and source metadata pressure

[Zax optional values](../../language/optional-values.md) permits independently
qualified repeated wrapper layers:

```zax
MyType readonly ? writable ? immutable
```

Reflection must preserve:

- the number and order of optional, pointer, and reference layers;
- qualifications attached to each layer;
- wrapper `final`/`varying` truth separately from declaration-side replacement
  permission;
- compact `??` as a different source token from spaced `? ?`;
- aliases that hide optional depth without changing canonical resolved
  structure.

Runtime presence is an expression/value fact, not part of canonical `T?` type
identity. Future reflection must distinguish static wrapper metadata from the
current absent/present state of one value.

## Expression facts versus type identity

```zax
compileValue := 42
runtimeValue := readInteger()
```

Both may have type `Integer`, while only `compileValue` is constant. Constancy is
an expression-evaluation fact and stays a direct query rather than becoming part
of `Integer` identity.

Several distinct result slots each have a type but do not collectively form one
expression type. A future explicit combiner would itself return one typed
structure; see [structural typing](structural-typing.md).

## Source reflection of fences and transparent enclosures

An explicit
[phrase fence](../../language/operator-phrases.md#exact-phrase-fencing) is
required source presentation, yet it creates no node in the final expression
tree. The same is true of the candidate transparent
[bare source enclosure](bare-source.md).

Source reflection and documentation tooling must retain enough information to
reproduce both. Future work must decide:

- how a fence is represented when the tree has no corresponding node;
- how a transparent enclosure is represented;
- whether formatters and documentation generators consume the same
  representation; and
- whether source reflection is a separate facility from type and declaration
  reflection.

## Enum metadata

Legacy enum material assumes compile-time metadata sufficient to convert between
enum values and strings. Preserve that requirement here and route the enum-side
questions to [enum types](enum-types.md).

## Integer-specific metadata

The integer facts themselves are current in
[Zax integers](../../language/integers.md). This section retains the future
reflection shape and tooling exposure required to report them.

Future reflection must let integer tooling and generic code discover:

- logical bit width and signedness;
- minimum and maximum values;
- exact intrinsic specialization;
- transparent alias and public intent-identity relationships;
- admission and exposure choices;
- immediate projection type;
- signedness counterpart and whether it is a named pair or unnamed exact type;
- bit-count, byte-count, delta, distance, and storage associated types;
- storage-envelope byte count, non-value padding extent, and alignment;
- exact, native, or software representation classification;
- requested native-selection constraints;
- selected width, active/target/compiler-host environment, and fallback;
- optimized-maximum selection and software-emulation classification;
- applicable protected operation and conversion families;
- declared range-containment and identity-bridge relationships;
- endian eligibility; and
- normalization requirements for representation-dependent exchange.

Native representation says only that the CPU provider classifies the width as a
native scalar representation. Per-operation instruction availability and cost
remain separate metadata.

Diagnostics should lead with the programmer's source alias or the most useful
public catalog name. Expanded output may show the canonical
`Scalars.Integers...` path and the underlying `Integer$(...)` specialization
without making the generic expansion the ordinary error vocabulary.

Future work must decide whether these facts form an integer-specific metadata
section, general associated-type relationships, convenient phrase operations,
or several layers. This input does not reserve exact reflection phrases.

## Constraints this input places on current work

- current documentation must not present illustrative reflection wording as an
  exact source reservation;
- canonical type identity must not absorb expression-specific facts; and
- the direct predicate family must not expand to cover declaration properties;
- public aliases remain useful diagnostic vocabulary even when reflection exposes
  canonical identity; and
- integer-specific facts need not become universal fields on every type.

## Activation and retirement

Activate this input when reflection, type or declaration metadata, integer-family
metadata, source reflection, or reflection-dependent tooling is reviewed. Move
accepted behavior into a reflection owner and the domain owners whose facts it
exposes, then retire this file after every preserved question is dispositioned.
