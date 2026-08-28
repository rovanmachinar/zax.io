# Raw input: numeric type families

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work defining intrinsic, custom, generic, fixed-point, arbitrary-width, enum, or floating numeric families |
| Applies To | Numeric type relationships and operator availability deferred by operator review |
| Owns | Signedness-counterpart, finite-bit-extent, type-family, alias, enum, float-representation, and policy applicability questions |
| Does Not Own | Accepted numeric types, layouts, conversions, operators, or generic contracts |
| Source / Provenance | Legacy intrinsic-type, enum, casting, and operator evidence |

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

Bit count, shift, rotate, reversal, and reduction require a finite defined
logical bit extent. A runtime-sized bit vector may provide one. Hidden allocation
capacity and padding must not change results.

## Enums and aliases

Future enum work must decide whether an enum directly inherits numeric, bitwise,
or signedness-counterpart operations from its underlying representation or must
be converted explicitly first:

```zax
flags := Fruit.Apple | Fruit.Banana
```

Alias work must distinguish aliases retaining intrinsic identity from new wrapper
types with independent operator domains.

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
fixed-point, arbitrary-width, enum, alias, float, or representation work. Move
accepted behavior into numeric, type, cast, enum, or operator owners and retire
this file after every item is dispositioned.
