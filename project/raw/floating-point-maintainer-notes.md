floating-point-maintainer-notes.md

# Maintainer raw input into floating points

This is newer design than the legacy information inside Zax meant to refresh (or even replace) the legacy materials where appropriate.

Zax will support two types of floating points:
- fixed integer floating points
- mantissa floating points

This is all hand crafted so if something seems "wrong" it's possible the maintainer made typos or other errors. Please confirm any mistaken understanding.

# fixed floating points are integers

Zax will follow the Zig model conceptually. Integers are actually just bit-width with fractional 0 bit position, meaning there are no bits dedicated to fractional for normal integers.

As Integers in Zax are actually based on a generic, one of the parameters in the generic will be the fractional bit position. This bit position is base-2 fractional. More dedicated bits is more fractional. But all math operations are actually the same. The only real difference is presentation and conversion. For example of you have "32.8" vs "32.4" converting from "32.8" would require shift by 4. The least significant bits hold the fractional. When an integer generic becomes concrete the fractional part is a concrete part of the definition of the Integer. The integers remain twos compliment for signed numbers. The min/max value for fixed floating point are identical values, where the maximum presentation would have to consider the fractional part.

Right now, without generics, we will "handwave" around the definition.

Conceptually however the alias is made will derive to:
```
MyU32f8 :: alias Integer$(..., FractionalBits = 8)      // this is NOT legitimate syntax

myNormalInteger : U32 = 0
myFractionalInteger : MyU32f8 = 1.5

//...

myNormalInteger = myFractionalInteger as U32            // conceptually this would have to shift by 8 bits to convert
```

For the time being there will be no pre-defined fixed floating point aliases. That's not to say we can't add any (as that would be trivial) but absent a good naming strategy and set of definitions for what are "good" values, I don't have any recommended names.

The following aliases will be added for fixed floating points (U would be for unsigned versions):

`QI7F8` / `QU7F8` / `MicroFixed16` / `UMicroFixed16`:
Q7.8 - 16‑bit
16‑bit micro controller format
Classic embedded fixed‑point

`QI15F16` / `QU15F16` / `ControlFixed32` / `UControlFixed32`:
Q15.16 - 32‑bit
32‑bit ARM DSP format
Very common in control systems

`QI23F8` / `QU23F8` / `AudioFixed32` / `UAudioFixed32`:
Q23.8 - 32‑bit
32‑bit audio DSP format (audio DSP classic)
Great for filters and mixing

`QI31F32` / `QU31F32` / `PrecisionFixed64` / `UPrecisionFixed64`:
Q31.32 - 64‑bit
64‑bit high‑precision fixed‑point
Excellent for simulation and DSP

`QI1F14` / `QU1F14` / `MLFixed16` / `UMLFixed16`:
Q1.14 - 16‑bit
16‑bit ML inference format
High fractional precision

`QI3F12` / `QU3F12` / `ControlFixed16` / `UControlFixed16`:
Q3.12 - 16‑bit
16‑bit embedded control format
Perfect for PID loops

Each of these will be defined in these namespaces:
```zax
Scalars.Floating.Fixed
Scalars.Floating.Fixed.Target
Scalars.Floating.Fixed.CompilerHost
```


# Binary Floating-Points

Zax also uses different bit-width binary floating points based on a generic float which are IEEE-ish. The "ish" is because IEEE only defines a few fixed sizes and any other sizes are thus non-standard, but they have a standard encoding format.

Zax will let you specify two values for floating points:
- fractional bit-width size
- exponent bit-width
- if an explicit Integer bit is present (if not present, 1 is assumed as part of the fractional bits to form the full mantissa; if present, then the fractional bits storage contains the full mantissa which has 0 or 1 explicitly specified for the integer bit)
- little or big endian encoding

Given this information, the actual required bit-width is derived.

There's alway a required sign bit. Legacy formats like MBF-40 are not supported (e.g. BASIC 6502 legacy format). The minimum exponent size is 2 bits. The minimum fractional bits is 1 bit, and 2 bits when integer bit is present. That makes the minimum floating point bit-width 4 bits. When stored, the size is always fixed into a larger storage aligned to the nearest 8-bits just like Integer values. The extra unused bits located in the "most significant" bits are undefined. Those bits cannot be accessed nor obtained (outside of direct pointer storage access). What is physically in storage for those extra bits is undefined.

The IEEE defined bit ordering is always applied. The MSB is the sign, and the LSBs contains the mantissa or fractional bits. There are no padding bits between. The integer bit if specified is always the MSB of the mantissa value. If custom hardware uses padding bits or custom encodings, they must have a "load" and "store" that re-encode to their internal format to be compatible and never project their internal bit format into into a float backed value.



Each CPU target provider will define these "well-known" aliases:

`Half` / `Binary16`:
Sign: 1 bit
Exponent: 5 bits
Mantissa: 10 bits
Exponent bias: 15
Status: Official IEEE‑754

`Single` / `Binary32`:
Sign: 1 bit
Exponent: 8 bits
Mantissa: 23 bits
Exponent bias: 127
Status: Official IEEE‑754

`Double` / `Binary64`:
Sign: 1 bit
Exponent: 11 bits
Mantissa: 52 bits
Exponent bias: 1023
Status: Official IEEE‑754

`Quad` / `Binary128`:
Sign: 1 bit
Exponent: 15 bits
Mantissa: 112 bits
Exponent bias: 16383
Status: Official IEEE‑754

`BF16`:
Sign: 1
Exponent: 8
Mantissa: 7
Exponent bias: 127
Status: Not IEEE‑754, but industry standard

`E4M3`:
Exponent: 4
Mantissa: 3
Exponent bias: 7

`E5M2`:
Exponent: 5
Mantissa: 2
Exponent bias: 15

`X80M64`:
Sign: 1
Exponent: 15
Mantissa: 64
Exponent bias: 16383
Integer bit: present (this is an exception)
Status: Not IEEE‑754, but historically standard on x86
Endianness: always little endian

`MBF40`:
Sign: 1 bit
Exponent: 7 bits
Mantissa: 32 bits
Exponent bias: 129
Status: 6052 BASIC
Endianness: always big endian


A definition can include "not supported". Each definition will also specify if the value is hardware CPU-backed or software library based. Hardware backed does not mean the value supports all math operations in hardware, it means that at least one hardware operation is supported with direct floating point support.

The hardware package can include other formats and aliases within the defined as a sub-namespace floating point namespace but those will be non-standard.

There is no requirement that any floating point at all be supported in hardware nor in software.

The target will also define:

| Name | Meaning |
| --- | --- |
| `NativeMaxF` | Widest scalar width the CPU profile classifies as native |
| `OptimizedMaxF` | Widest family for which the profile supplies its preferred general-purpose optimized implementation |
| `SupportedMaxF` | Widest exact specialization supported by this toolchain/version/configuration |
| `LanguageMaxF` | Widest exact specialization permitted by this Zax language version |

| Name | Meaning |
| --- | --- |
| `FastF8`through `FastF128` | CPU-provider-preferred implementation having at least the requested width |
| `LeastF8` through `LeastF128` | Smallest native width having at least the requested width |

Each of these will be marked as supported or not and if there's a required endianness to be supported.

This implies that both hardware and software based floating points might entirely be unavailable, or fully available.

Zax allows generic based floating points type expressed as a concrete type to express their "values" as compile time known values so that the compiler can determine what is and is not supported. The `Target` and `CompilerHost` have their own namespaces and the "current" namespace maps the floating points into the current compile time, or target runtime environment. Thus target and compiler host need not be using the same definitions for their namespaces for these values. One could support some values while the other does not.

```zax
Scalars.Floating.Binary
Scalars.Floating.Binary.Target
Scalars.Floating.Binary.CompilerHost
```

Extraction of binary floating points into Integers must be possible if the floating point is supported.

## Structural safe and coercive compatibility pressure

Current
[structural shapes and compatibility](../../language/structural-shapes-and-compatibility.md)
requires every scalar family to supply two property lists:

1. properties that must all match for safe scalar shape/layout compatibility;
2. a reduced set that must match for local coercive `unsafe as`.

The provisional safe integer/fixed-point list is:

- logical bit width;
- signedness;
- value encoding, including two's-complement representation;
- fixed fractional-bit position;
- endianness role; and
- storage extent, alignment, non-value bits, and normalization.

Integer/fixed-point coercion provisionally requires equal logical bit width and a
sufficient compatible target storage envelope. It may reinterpret sign,
fractional position, or endian meaning.

Floating-point coercion must remain stricter. The future review must finalize at
least:

- total format width;
- exponent width;
- fraction or mantissa width;
- explicit versus implicit integer bit;
- endianness; and
- every representation rule affecting special values or normalization.

Different exponent partitioning is not another compatible viewpoint on the same
number and remains incompatible even under coercion. Floating-point work must
finalize both lists and recheck the integer list rather than treating the
provisional structural summary as its lasting numeric-format owner.

## declared compile time support

I suggest we have some kind of structure in Zax based on the type that indicate each of these at compile time for the definition. Some will be directly derived from the generic inputs, and others will be consequences of the platform (for example, a 6502 cpu floating might not have full representation of NaN, Inf, etc):

```cpp
BinaryFloatCapabilities
{
    SupportsNaN: bool
    SupportsInfinity: bool
    SupportsSubnormals: bool
    SupportsSignedZero: bool
    SupportsIEEE754Rounding: bool
    SupportsExceptionFlags: bool
    HasImplicitLeadingOne: bool
    HasExplicitIntegerBit: bool
    ExponentBias: int
    MantissaBits: int
    ExponentBias: int
}
```

## known supports

While these will come from the platform definition, it's expected these to be defined as such:

### Partial IEEE‑style support
BF16
- Has NaN
- Has Inf
- Has subnormals
- Rounding is coarse (only 7 mantissa bits)

### No IEEE special values

MBF40
- No NaN
- No Inf
- No subnormals
- No signed zero
- No IEEE rounding modes
- No exception flags

### FP8 formats (E4M3, E5M2)

E4M3
- No Inf
- No NaN (only “invalid” patterns)
- No subnormals
- No signed zero

E5M2
- Has Inf
- Has NaN
- No subnormals
- Signed zero exists


# Things maybe missed in the previous Integer definitions

## big vs little

The Integer should indicate if it's "little endian" or "big endian" or "agnostic" meaning it can change at runtime, and the platform will define what each type is for that platform. To be clear, a type is never "agnostic" when a concrete form is made. Every instance must have an endianness so an agnostic really means the current compile target has specified it's default endianness and thus "agnostic" becomes settled to that endianness. There will still be "big endian" and "little endian" fixtures. On "agnostic" platformed, if the endianness of the CPU can change on the fly there's no flag within an instance of a type that indicates the current endianness of the state. Thus the type has to have a concrete settled endianness. If the math routines entered from that type can swap behavior at runtime (and that is allowed within the compilation target) then the math library will have to perform runtime checks to select the correct runtime routines for the type. The math operations must always match the concrete endianness of a type and not switch merely because a CPU decided to behave otherwise. Ideally, the CPU endianness would be marked as fixed for a compilation target to ensure the correct endianness routines are selected without extra runtime check overhead.


## compile time indication of supported

A discoverable compile-time flag should be included on the type to determine if the type is indeed supported. The defined terms help to pick amongst different known sizes, but (for example) a 6502 CPU library might never include 64 bit values.

A target might support what a compiler host might not or vice versa.
