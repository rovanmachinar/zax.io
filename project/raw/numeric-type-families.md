# Raw input: numeric type families

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work defining intrinsic, custom, generic, fixed-point, arbitrary-width, or floating numeric families |
| Applies To | Numeric type relationships and operator availability deferred by operator review |
| Owns | Signedness-counterpart, finite-bit-extent, canonical bit-count type, multiword, reversal, extraction/deposit, type-family, alias, float-representation, and policy applicability questions |
| Does Not Own | Accepted numeric types, layouts, conversions, operators, generic contracts, or [enum behavior](enum-types.md) |
| Source / Provenance | Legacy intrinsic-type, casting, and operator evidence |

## Signedness counterparts

A concretely typed integer may request its equal-width opposite-signedness
counterpart:

```zax
unsignedValue : U8 = 100
signedValue := +unsignedValue // I8(100)
```

Future numeric work must decide how a custom type advertises:

- signedness;
- an equal-width counterpart;
- supported panic/optional/wrapping/saturating/reporting policies;
- generic constraints and reflection;
- layout relationships; and
- conversion versus counterpart operation.

This is one type with a signedness property or relationship, not a requirement
for separate mutable/immutable-style implementations.

## Finite and unbounded families

An arbitrary but finite type such as conceptual `I137`/`U137` can define modular
counterpart operations. A truly unbounded signed integer has no wrapping
conversion to an unbounded nonnegative counterpart without a supplied modulus.

Bit count, shift, rotate, reversal, reduction, set-bit mask, set-bit position,
multiword, and masked extraction/deposit operations all require a finite defined
logical bit extent. A runtime-sized bit vector may provide one. Hidden allocation
capacity and padding must not change results.

## Canonical unsigned bit-count type

The zero-count forms and the optional set-bit-position phrases share one
canonical unsigned bit-count type:

```zax
leadingZeros := #<bits
position := bits trailing set bit position

if ?position
  shifted := x << position.
```

That type must be directly viable as the count operand of every applicable
built-in shift without an explicit conversion. Future numeric work must establish:

- its exact name;
- its representation and width;
- whether it varies with the operand's logical extent or is uniform;
- how an optional payload of that type participates in shift counts; and
- how generic numeric code names it.

## Multiword operations

The operator catalog records these as accepted concepts with exact words
deferred:

```zax
// Illustrative wording; exact multiword phrases are not established.
high shift left through low by count
high rotate left through low by count
result funnel right from high and low by count
```

Exact spellings stay deferred because arithmetic versus logical right shift,
receiver orientation, high/low ordering, result shape, multi-place mutation,
signedness, and aliasing all affect the theoretical prototypes. Future numeric
work must settle:

- whether components must share one logical bit extent;
- whether a writable form mutates one place or several;
- what a funnel shift's fixed-width result window is; and
- how aliasing among component places behaves.

## Reversal and masked extraction/deposit

Bit reversal and byte reversal are distinct finite-extent concepts: byte reversal
preserves bit order within each byte, while bit reversal does not.

Masked extraction packs every source bit selected by a mask into contiguous
low-order positions. Masked deposit scatters contiguous low-order input bits into
positions selected by a mask.

Future numeric work must decide exact words, mask-width behavior, zero fill,
signedness interaction, and unsupported-target costs.

## Aliases

Alias work must distinguish aliases retaining intrinsic identity from new wrapper
types with independent operator domains.

## Numeric interaction with enums

An enum is backed by one fundamental intrinsic type but is an independent type.
Whether an enum inherits numeric, bitwise, reduction, shift, magnitude, or
signedness-counterpart operations from its backing representation is an *enum*
question and is preserved in [raw enum input](enum-types.md).

Numeric work should consult that file when a numeric decision would constrain
enum behavior, and vice versa. This file no longer holds enum-specific questions.

## Floating representation

Floating numeric operations and raw representation access are separate:

```zax
// Illustrative future concept.
bits of floatValue
```

Future work must distinguish numeric conversion, bit-preserving reinterpretation,
unsafe raw access, NaN payloads, canonicalization, byte order, and target
representation. Ordinary float bitwise operators remain reserved and unavailable
until that review.

## Activation and retirement

Activate this input for numeric types, generic numeric contracts, signedness,
fixed-point, arbitrary-width, canonical bit-count type, multiword operations,
reversal, masked extraction/deposit, alias, float, or representation work. Move
accepted behavior into numeric, type, cast, or operator owners and retire this
file after every item is dispositioned. Enum-specific questions are dispositioned
through [raw enum input](enum-types.md) instead.
