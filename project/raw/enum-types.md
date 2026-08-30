# Raw input: enum types

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work defining complete enum behavior, validity, inheritance, or generation |
| Applies To | Fundamental-backed enum identity and the enum questions deferred by operator-phrase review |
| Owns | Preserved enum questions, representative source, activation pressure, and retirement criteria |
| Does Not Own | Accepted enum grammar, members, conversions, operator availability, or generic behavior |
| Source / Provenance | Legacy `enums.md` and `casting.md` evidence, the enum portion formerly held in [numeric type families](numeric-type-families.md), and operator-phrase review of the generated enum baseline |

## Why this input exists

Current documentation accepts a narrow enum baseline: four generated operations,
recorded by the
[operator catalog](../../language/operator-catalog.md#generated-enum-operations),
plus the endian semantic families taught by
[Zax endianness](../../language/endianness.md).

Everything else about enums — members, validity, inheritance, safe creation,
generation, and conversion beyond the underlying type — remains undecided. This
file keeps those questions, and the evidence behind them, out of archived
numbered work.

## Preserved baseline that future work must not silently contradict

An enum is an independent type backed by one fundamental intrinsic type. It owns
its own behavior and implementation surface while remaining constrained to the
backing type's instance storage:

- an enum cannot add stored instance members or hidden per-value storage;
- it may add `final` functions, type-receiver operators, and other behavior that
  requires no additional instance storage;
- storage and shape compatibility with the backing type do not grant implicit
  transfer or shared type identity; and
- the four generated signatures are protected and cannot be replaced by an enum
  declaration.

```zax
BackingType := EnumType underlying type
rawValue := enumValue underlying value
converted := enumValue as UnderlyingType
enumValue := EnumType unsafe from rawValue
```

`unsafe from` preserves the backing value without membership, range, or semantic
validation, so it may produce a value outside a restricted enum's ordinarily
named domain.

## Safe creation pressure

There is no baseline safe `EnumType from value` for a restricted enum. Endian
enums have a safe `from` because they are full-domain, so every bit pattern is a
valid value.

Future enum work must decide whether a restricted enum receives a safe creation
facility and, if so, whether it:

- validates membership and reports failure;
- returns an optional or a reporting result shape;
- distinguishes named values from arbitrary in-range values; or
- remains unavailable, leaving `unsafe from` as the only adoption route.

```zax
// Illustrative pressure; no safe restricted-enum creation is accepted.
maybeFruit := Fruit from rawValue
```

## Restricted versus full-domain validity

| Kind | Every backing bit pattern is valid? | Consequence |
| --- | --- | --- |
| Restricted enum | No | `unsafe from` can create an out-of-domain value; closed operations could too |
| Full-domain semantic enum | Yes | Language-provided closed operations cannot create an invalid representation |

Future work must decide what "valid" means for a restricted enum, whether flag
combinations are valid, and what an operation is permitted to do when a value is
outside the named domain.

## Operation independence or inheritance

Legacy material assumes enums can be combined arithmetically and bitwise:

```zax
Fruit :: enum U8 {
    Apple           = (1 << 0),
    Banana          = (1 << 1),
    Pear            = (1 << 2),
    Orange default  = (1 << 3),
    Pineapple       = (1 << 4)
}

fruitInBasket := Fruit.Apple | Fruit.Banana | Fruit.Pear
```

Nothing in the accepted baseline decides that. Future enum work must decide
whether an enum:

- directly inherits numeric, bitwise, reduction, shift, magnitude, or
  signedness-counterpart operations from its backing representation;
- inherits a subset chosen by the enum's declared intent, such as a flag enum;
- receives generated operations only when the enum opts in; or
- requires explicit conversion for every such operation.

Endian enums are evidence that a fundamental-backed enum may supply a *focused*
operation surface rather than the backing type's full surface. Do not infer from
them that every enum inherits its backing operations.

## Legacy `unsafe as` conversion evidence

Legacy enum material proposes two operations beyond the generated baseline:

```zax
bitValue := fruitInBasket as U8              // baseline: exact underlying type
bitValueInInt := fruitInBasket unsafe as Integer // legacy: non-underlying intrinsic
toAnotherValue := fruitInBasket as U16       // legacy: rejected direct conversion
safeValue := fruitInBasket as U8 as U16      // legacy: two-step conversion
```

The claims were:

- `as` to the exact underlying type is allowed;
- `as` directly to another intrinsic type is an error even when it could not
  overflow;
- `unsafe as` to another intrinsic type is permitted and ignores overflow; and
- a two-step conversion through the underlying type is the sanctioned route.

Only the first is part of the accepted baseline. Overflow, narrowing, panic, and
safety behavior for the rest remains deferred to enum and casting work. Preserve
the evidence rather than silently accepting or discarding it.

## Endian enum motivation

Endian families motivated much of the baseline: they needed independent nominal
identity, unchanged instance storage, an explicit receiver-correct conversion
model, and a distinction between semantic decoding and raw extraction.

Complete endian behavior belongs to [Zax endianness](../../language/endianness.md).
The questions that remain *enum* questions are:

- exact generated enum names and the generic mechanism producing a family;
- whether generated families are ordinary enums or a distinct declaration kind;
- non-byte-multiple widths, padding, and representation extent; and
- how a full-domain semantic enum declares that every bit pattern is valid.

## Numeric interactions retained here

The enum-specific portion formerly held in
[numeric type families](numeric-type-families.md) now lives in this file.
Numeric questions that merely *touch* enums — signedness counterparts, finite bit
extent, and the canonical bit-count type — stay with that input.

## Other undecided enum concerns

- enum metadata and its relationship to reflection, routed to
  [reflection](reflection.md);
- default value selection and duplicate values;
- explicit member values and implicit successors;
- exported, private, and nested enum declarations;
- alias relationships between an enum and its backing type; and
- whether an enum may declare its own conversions to unrelated types.

## Activation and retirement

Activate this input when enum members, validity, inheritance, safe creation,
generic enum generation, or enum conversion is reviewed. Move accepted behavior
into an enum owner and the applicable operator, casting, numeric, and reflection
owners, then retire this file once every preserved question has an accepted
owner, explicit deferral, or discarded status.
