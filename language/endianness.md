# Zax endianness

| Field | Value |
| --- | --- |
| Status | Current conceptual design |
| Audience | Human developers reading, writing, or evaluating Zax code that stores, transmits, or interprets byte-ordered integers |
| Applies To | Programmer-facing endian semantic enum behavior; not a formal grammar, layout contract, or specification |
| Implementation State | Not established by this repository |
| Owns | The endian mental model; absolute, native, compiler-host, and target endianness; how the four generated enum operations behave for endian values; receiver-correct encode, transcode, decode, adoption, and raw extraction; backing-value validity for endian enums; the focused endian operation surface; native right-operand as-if semantics; unavailable and deferred endian operations; storage/shape compatibility without implicit transfer; endian costs, diagnostics, and examples |
| Does Not Own | Complete enum validity/generation; integer families and backing eligibility ([integers](integers.md)); exact forms and shared selection ([operator catalog](operator-catalog.md), [operators](operators.md)); or general type compatibility/layout rules |
| Source / Provenance | Legacy [enums](../enums.md) and [basics](../basics.md) endian conversion evidence, refined against current operator and enum behavior |

## Mental model

> For every operation that an endian type supports, behave as though the value
> were being operated on natively by an environment with that byte order.

That single sentence is the whole model. An endian value is not a byte-swapping
gadget bolted onto an integer; it is a distinct type whose supported operations
mean what they would mean inside a machine that natively uses its byte order.

```zax
headerBits : U32 = 4660       // hexadecimal 00001234
flagMask : U32 = 2147483648   // hexadecimal 80000000

header := BigEndianU32 from headerBits
flagged := header | flagMask
// `flagged` means "the big-endian value 0x80001234",
// not "the native value whose bytes happen to look shuffled".
```

Zax deliberately supports only operations whose meaning is unambiguous under that
sentence. Everything else is unavailable rather than approximate.

## Endian semantic enum families

For every applicable exact fundamental integer type, Zax conceptually supplies a
big-endian and a little-endian enum version backed by that same fundamental type.
An applicable backing type has a whole-byte logical width, no non-value storage
bits, and the representation guarantees defined by
[Zax integers](integers.md#enum-and-endian-boundaries).

```zax
big : BigEndianU32
little : LittleEndianU32
```

The names `BigEndianU32` and `LittleEndianU32` are explicitly illustrative. They
are written here as though ordinary named enum types exist so that examples stay
readable; exact generated names and the generic mechanism producing the family
remain future enum and numeric work.

Every value of the backing integer type represents an endian value. That is why
the language-provided closed bitwise operations below cannot create an invalid
enum representation.

Endian enums:

- are independent nominal enum types rather than proxies or aliases;
- share the fundamental backing type, instance size, alignment, and
  representation extent of the corresponding basic type;
- cannot add stored instance members or hidden per-value storage;
- require explicit conversion despite that compatible storage and shape; and
- add a focused operation surface beyond the four operations every enum receives.

A **semantic** enum is one whose stored bits and its represented numeric meaning
may differ. Endian enums are the current example: the meaning of an endian value
is a number, while its storage is that number arranged in a particular byte
order.

## The four generated enum operations

Every enum, endian or not, receives four protected language-provided
[operator phrase](operator-phrases.md) operations. For an endian enum they read
as follows.

### `underlying type`

```zax
BackingType := BigEndianU32 underlying type // U32
```

A post-unary type-receiver operation returning the exact fundamental intrinsic
type backing the enum, without use-site qualifiers and in its default-qualified
form. It requires no instance.

### `underlying value`

```zax
rawBigStorage := big underlying value
```

A post-unary instance operation returning one value of the exact underlying
fundamental type containing the stored backing value **unchanged**. It performs
no semantic conversion and no validation. For an endian value this is the raw
byte arrangement, not the represented number.

### `as UnderlyingType`

```zax
nativeValue := big as U32
```

The enum value supplies receiver discovery and the right operand is the exact
underlying type argument. For an endian enum, `as` **decodes represented numeric
meaning**.

For an ordinary restricted enum, semantic decode and raw extraction commonly
produce the same value. Endian enums are exactly where they differ:

| Operation | Question it answers |
| --- | --- |
| `big as U32` | What number does this big-endian value represent? |
| `big underlying value` | What bit pattern is actually stored? |

When the active execution byte order matches the enum's absolute order, the two
agree. When it does not, they do not, and the difference is the entire point.

### `EnumType unsafe from rawValue`

```zax
big := BigEndianU32 unsafe from rawBigStorage
```

A binary type-receiver operation that accepts a value of the enum's exact
underlying fundamental type, preserves that backing value unchanged, performs no
membership, range, or semantic validation, and establishes the independent enum
type. It records the programmer's unchecked assertion that the raw storage
already carries the claimed endian meaning.

`unsafe from` belongs to the baseline enum model rather than being an
endian-specific operation, so no endian-specific raw-adoption phrase exists.

## Receiver-correct conversion

Entry into an endian domain is owned by the **destination** enum type receiver:

```zax
big := BigEndianU32 from nativeValue
little := LittleEndianU32 from nativeValue

little := LittleEndianU32 from big
big := BigEndianU32 from little
```

The destination-owned operation also gives a number literal the exact backing
integer type it needs:

```zax
smallBigEndian := BigEndianU32 from 5
```

Here the selected `from` input is `U32`, so argument binding has the same typed
value as `(: U32 = 5)` before the endian value is created. This direct source
capability must survive future refinement of generated endian names and family
syntax. Number-literal input selection is defined by
[Zax integer literals and realization](integer-literals.md#number-literals-filling-typed-inputs).

The additional safe type-receiver `from` operation:

- **encodes** native numeric meaning when the source is the exact fundamental
  basic type;
- **transcodes** numeric meaning when the source is another endian enum in the
  same backing family;
- returns the destination endian enum type; and
- performs no widening or narrowing.

Exit is owned by the enum value receiver:

```zax
nativeValue := big as U32              // decode numeric meaning
rawBigStorage := big underlying value  // exact stored backing value
BackingType := BigEndianU32 underlying type
```

Putting the four directions together:

| Direction | Operation | Meaning |
| --- | --- | --- |
| Native number in | `BigEndianU32 from nativeValue` | Safe encode |
| Other endian in | `LittleEndianU32 from big` | Safe transcode |
| Raw storage in | `BigEndianU32 unsafe from rawStorage` | Unchecked adoption |
| Number out | `big as U32` | Safe decode |
| Raw storage out | `big underlying value` | Exact extraction |

There is no implicit transfer in any direction:

```zax
nativeValue : U32 = big
// error: compatible storage does not grant implicit conversion
```

### Receiver ownership

The destination enum type owns entry into its domain:

```zax
BigEndianU32 from nativeValue
LittleEndianU32 from nativeValue
MyEnum unsafe from rawValue
```

This keeps fundamental types independent of enums that happen to use them as
backing storage. The uniform target-oriented `from` operation owns encoding and
transcoding, while the enum-owned `as UnderlyingType` operation owns semantic
exit.

## Native, compiler-host, and target endianness

**Native endianness** means the active byte order of the environment whose
execution semantics are currently in effect. Distinguish it from:

| Term | Meaning |
| --- | --- |
| Native endianness | The byte order of the environment whose execution semantics are currently active |
| Compiler-host endianness | The byte order of the environment running the compiler |
| Target endianness | The byte order selected for generated target behavior |
| Big endian / little endian | Absolute representation orders, independent of any environment |

At ordinary target runtime, native endianness is the target execution
environment's byte order. During compile-time execution, native endianness is the
byte order of the environment whose execution semantics are currently active.

Do not use `host`, `target`, and `native` interchangeably. A native-endian alias
is useful for generic source but is context-relative by construction, so exact
`native`, `host`, and `target` enum names remain future naming and
compile-time-context work.

## Mixed native operands

When an available operation on an endian receiver accepts a native operand of the
exact underlying fundamental type, its observable result is the same as if the
native operand had first been converted through `EndianType from nativeValue`:

```zax
masked := bigValue & nativeMask
// same observable result as:
masked := bigValue & (BigEndianU32 from nativeMask)
```

This is an as-if semantic rule, not a required implementation sequence. An
implementation may fuse, elide, or replace those conceptual steps with any
equivalent behavior while preserving result type, observable value, failures, and
documented costs.

The endian value must supply receiver discovery:

```zax
nativeMask & bigValue
// unavailable by design: a fundamental left operand owns no enum-aware signature
```

A raw mask that already carries big-endian representation is *not* a native
number and must use raw adoption instead:

```zax
masked := bigValue & (BigEndianU32 unsafe from rawBigMask)
```

Two different endian enum operands require explicit target-oriented transcoding.
Mixed native participation is operation-local mapping, never implicit assignment
or general transfer.

## Supported operation surface

Endian enum families add a deliberately limited set of operations whose meaning
is unambiguous under the mental model:

- same-type construction, copy, move, assignment, destruction, and swap;
- equality and inequality with the same endian enum type, or with the exact
  backing fundamental type on the right;
- complement on one endian enum value;
- AND, OR, XOR, and AND-NOT with the same endian enum type, or with the exact
  backing fundamental type on the right;
- the language-defined `bitwise nand`, `bitwise nor`, `bitwise xnor`, and
  `bitwise or not` phrase forms;
- the corresponding direct mutation forms;
- population count, returning the backing integer's associated bit-count type;
  and
- reduction AND, OR, XOR/parity, NAND, NOR, and XNOR.

```zax
aBits : U32 = 305419896 // hexadecimal 12345678
bBits : U32 = 252645135 // hexadecimal 0F0F0F0F

a := BigEndianU32 from aBits
b := BigEndianU32 from bBits

same := a == b
inverted := ~a
combined := a | b
cleared := a &~ b
neither := a bitwise nor b

a |= b
a bitwise nand assign b

setBits := #a
anySet := #|a
```

Boolean bitwise operations commute with the fixed endian representation
permutation, so operating on matching encoded backing values produces the same
encoded result as operating natively in that endian environment. Population count
and reductions do not depend on bit position. These basic endian signatures are
language-provided and protected.

## Unavailable and deferred operations

The initial endian surface does not include:

- numeric arithmetic or magnitude;
- ordering comparisons;
- increment or decrement;
- shifts, rotates, or multiword shift operations;
- leading/trailing counts, set-bit masks, or set-bit positions;
- bit or byte reversal; or
- masked extraction and deposit.

```zax
shifted := bigValue << 2
// error: shifting is not part of the initial endian operation surface
```

Decode first when a numeric operation is what the program actually means:

```zax
shifted := BigEndianU32 from ((big as U32) << 2)
```

If future enum and endian work adds a direct operation, the mental model already
fixes its meaning: a numeric shift as if executed natively in the enum's endian
environment, never an accidental shift of how backing bytes appear in the active
machine. Hidden conversion cost and the absence of a direct target instruction
are design pressures, not automatic reasons to include or exclude an operation.

## Compatibility is not conversion

An endian enum and its backing fundamental type are storage- and
shape-compatible while remaining independent types.

- **Exact storage compatibility** requires the complete instance storage to agree
  in size, alignment, stored-member offsets and extents, required padding, and
  representation boundaries from the base address.
- **Shape compatibility** additionally requires corresponding storage-bearing
  members to use the same names and recursively compatible storage.
  Zero-instance-storage declarations such as `final` functions do not change
  instance shape.

Compatibility describes structural facts. A conversion rule grants transfer.
Neither form of compatibility permits:

```zax
nativeValue : U32 = big
// error: structural compatibility is not a conversion rule
```

Complete type-compatibility terminology, directional projection, and truncating
transfer remain future type-compatibility work.

## Costs

- `from` may cost nothing when the enum's absolute order matches native order,
  and a byte permutation when it does not. The language promises the result, not
  the instruction.
- `unsafe from` and `underlying value` are representation-preserving and carry no
  conversion cost, but `unsafe from` carries the programmer's unchecked
  assertion instead.
- `as UnderlyingType` costs whatever decoding the active environment requires.
- Boolean bitwise operations, population count, and reductions need no
  conversion, which is exactly why they are in the supported surface.
- A mixed native right operand conceptually converts that operand. Writing the
  conversion once and reusing the endian value avoids repeating it.

## Worked example

Reading a big-endian length field out of a buffer and using it natively:

```zax
rawStorage := readU32(buffer, offset)

// The buffer already holds big-endian bytes, so adopt rather than encode.
length := BigEndianU32 unsafe from rawStorage

// Decode to work with the number.
count := length as U32

// Mask a flag while staying in the endian domain.
flagMask : U32 = 2147483648 // hexadecimal 80000000
withoutFlag := length &~ flagMask

// Write it back out as raw storage.
writeU32(buffer, offset, withoutFlag underlying value)
```

Contrast the two extraction operations on a machine whose native order is little
endian:

```zax
value := BigEndianU32 from 1

numeric := value as U32              // 1
raw := value underlying value        // 0x01000000
```

Encoding a native number for transmission:

```zax
encoded := BigEndianU32 from hostCount
writeU32(packet, position, encoded underlying value)
```

Transcoding between two absolute orders without ever naming a native number:

```zax
little := LittleEndianU32 from big
```

## Diagnostics

Diagnostics should distinguish:

- an unavailable endian operation from an unsupported operand shape;
- an attempted implicit conversion between an endian enum and its backing type;
- a native operand in receiver position where no enum-aware signature exists;
- mixed endian operands requiring explicit transcoding;
- an invalid source or backing-type shape supplied to semantic conversion or raw
  adoption; and
- an attempt to declare or replace one of the protected endian or generated enum
  signatures.

`unsafe from` cannot diagnose that the programmer meant semantic encoding rather
than raw adoption. Its purpose is to accept the programmer's unchecked
representation assertion.

## Boundaries and maturity

This document is current conceptual design, not a formal specification, layout or
ABI contract, serialization framework, or implementation mapping.

Exact generated endian type names, the generic mechanism producing the family,
identity-family integration, ABI, complete enum validity and operation
inheritance, safe restricted-enum creation, complete arithmetic and ordering
behavior, and complete compile-time-context naming remain focused future work.

Exact operator forms, fixity, and precedence are in the
[operator catalog](operator-catalog.md#endianness-reference). The shared operator
selection model is owned by [operators](operators.md), and cross-cutting
vocabulary by [language-design terms](terms.md#native-endianness).
