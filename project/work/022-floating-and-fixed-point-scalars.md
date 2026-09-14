# 022: Floating and fixed-point scalars

| Field | Value |
| --- | --- |
| Status | Active working material / non-normative / awaiting assignment |
| Work Item | `022` |
| Created | 2026-09-14 |
| Owns | The bounded review defined by the fixed initiating input |
| Does Not Own | Complete generics, literal parsing, exhaustive numeric operators, CPU-provider implementation, ABI/FFI, vectors/SIMD, arbitrary decimal arithmetic, or compiler lowering |

## Non-authority notice

This file is a collaborative working record. Existing statements, new
observations, candidate wording, and aligned findings remain non-authoritative
until a separately discussed, aligned, and explicitly authorized promotion
incorporates them into their lasting owners.

## Fixed initiating input

This section records the information aligned when work item `022` was created.
It is intentionally incomplete and must not be rewritten as work develops.

### Initiating concern

Establish a coherent programmer-facing model for Zax fixed-point and binary
floating scalar families, including concrete format identity, representation,
availability, conversion, costs, and safe versus coercive structural
compatibility.

The [floating-point maintainer notes](../raw/floating-point-maintainer-notes.md)
are the primary input. Treat their complete contents as maintainer-supplied
design evidence to understand and test, not as automatically accepted language
design. The [raw numeric-family input](../raw/numeric-type-families.md)
preserves supplemental generic, fixed-point, float-representation, conversion,
and operation pressure.

### Motivating pressure

Current integer design establishes exact logical widths, two's-complement signed
representation, storage envelopes, profile-selected identities, and explicit
conversion. Current structural design requires each scalar family to define:

1. the complete meaning-bearing properties that must match for safe shape/layout
   compatibility; and
2. the reduced properties that must still match for local coercive conversion.

Those lists remain provisional for fixed-point and binary floating values.
Floating review must finalize them and recheck the integer list rather than
letting equal byte size stand in for compatible numeric format.

Zax also needs a usable explanation of which formats exist, which values they
represent, when target or compiler-host support exists, what operations may
cost, and how programmers choose exact versus profile-selected types without
prematurely committing to one generic syntax or backend implementation.

### Known assumptions

- Type identity, scalar semantic format, storage layout, safe compatibility,
  coercive compatibility, numeric conversion, and ABI compatibility remain
  distinct.
- Integer-backed fixed-point values conceptually add a fixed binary fractional
  position to finite integer representation; exact generic syntax is not yet
  established.
- Binary floating formats conceptually depend on fraction or mantissa width,
  exponent width, explicit or implicit integer bit, endianness, and applicable
  special-value/normalization rules.
- A concrete scalar type has statically known format properties even when its
  implementation uses software support.
- Hardware support is optional and operation-specific; “native” must not imply
  that every operation has one direct instruction.
- Target, compiler-host, and active execution environment remain distinct.
- No compiler implementation exists in this repository.

### Known inclusions

- Human-facing terminology for fixed-point and binary floating scalar families.
- Concrete type identity and the dimensions that determine one format.
- Logical value interpretation, range, precision, storage envelope, alignment,
  padding/non-value bits, encoding, and endianness.
- Zero, signed zero, infinities, NaNs, subnormals, invalid patterns, and other
  format-dependent domains at the depth needed to define concrete types.
- Exact, conventional, target-selected, compiler-host-selected, fastest,
  smallest, widest, and supported aliases where justified.
- Hardware-backed versus software-backed availability and visible cost.
- Default initialization and ordinary value construction.
- Numeric conversion, raw representation access, checked/reporting boundaries,
  and compatibility with integer/fixed-point or other floating formats.
- Final safe-compatibility and coercive-compatibility property lists for
  fixed-point and binary floating values, plus a deliberate recheck of the
  current integer list.
- Literal, operator, generic, reflection, build-profile, and interop
  consequences far enough to give every material deferral a live owner.
- Lasting documentation ownership and complete disposition of the floating
  maintainer notes and applicable numeric-family raw input.

### Known starting boundaries

- Complete generic declaration, factory, inference, specialization, and
  associated-type syntax.
- Exact decimal/binary literal tokenization, parsing, suffix, and commitment
  grammar beyond immediate scalar consequences.
- An exhaustive floating/fixed-point operator catalog when the scalar model can
  remain coherent without it.
- Complete rounding-mode, exception-flag, environment-control, and
  reproducibility APIs.
- CPU-provider file formats, backend capability discovery, instruction
  selection, software-library algorithms, and compiler lowering.
- SIMD/vector/matrix formats and packed-lane operations.
- Arbitrary-precision or decimal floating families.
- ABI, FFI, calling convention, serialization, and wire-format guarantees.
- Formal grammar, conformance, and specification language.

These boundaries do not erase consequences. Preserve every material pressure in
a current owner or indexed raw destination rather than leaving it only in the
eventual archive of this work item.

### Intentionally unresolved framing

- Whether fixed-point values are concrete integer-family specializations, a
  related scalar family, or another relationship.
- The exact concrete parameters and derived properties for each fixed-point and
  binary floating format.
- Which conventional aliases are language-defined, CPU-provider-defined, or
  deliberately absent.
- Which formats and operations every complete toolchain must support.
- The exact safe and coercive property lists, especially when two values share
  total storage but differ in sign, fractional position, exponent partition,
  explicit integer bit, endianness, or special-value policy.
- Whether and how unsupported formats remain valid type identities.
- Which raw representation operations exist and how they differ from numeric
  conversion, coercive conversion, and `unsafe cast`.
- The minimum ordinary arithmetic, comparison, reporting, and conversion
  surface needed to make each scalar family useful.

### Initial stopping guidance

Stop when the work has:

- established a teachable scalar mental model for fixed-point and binary
  floating values;
- defined which properties determine concrete identity, value meaning, storage,
  safe compatibility, and coercive compatibility;
- described ordinary exact/profile-selected type use and availability;
- explained representative construction, conversion, failure, raw access, and
  cost behavior with valid and invalid source;
- dispositioned special values, endianness, padding, support, and portability;
- rechecked and either confirmed or revised the current integer compatibility
  property lists;
- assigned every generic, literal, operator, reflection, CPU-provider, interop,
  and implementation consequence to a live owner or indexed raw destination;
- identified exact lasting owners and raw/legacy dispositions; and
- completed the required documentation-fit dry run.

Do not promote findings, archive this work item, begin work item `023`, or
design complete generics, literal grammar, exhaustive operators, CPU-provider
implementation, ABI/FFI, or compiler lowering without the separately required
discussion, alignment, and authorization.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs focused reading,
  decision maturity, promotion, deferral, and closure.
- [Floating-point maintainer notes](../raw/floating-point-maintainer-notes.md) -
  supplies the maintainer's primary fixed-point, binary floating, aliases,
  capabilities, support, and integer-recheck input.
- [Language vision](../../language/vision.md) - supplies explicit-cost,
  data-representation, low-level access, portability, and toolchain direction.
- [Integers](../../language/integers.md) - owns the finite integer model,
  storage envelopes, exact/profile-selected identities, conversions, and the
  current scalar compatibility list that this work must recheck.
- [Structural shapes and compatibility](../../language/structural-shapes-and-compatibility.md)
  - owns scalar format versus identity/layout, safe and coercive compatibility,
  and raw-view boundaries.
- [Endianness](../../language/endianness.md) - owns absolute/native byte-order
  meaning and semantic conversion versus raw storage.
- [Raw numeric type families](../raw/numeric-type-families.md) - preserves
  generic numeric-family, fixed-point, floating representation, conversion, and
  operation pressure that this work must disposition.

### Consequence-driven

- [Integer literals and realization](../../language/integer-literals.md),
  [raw literals maintainer notes](../raw/literals-maintainer-notes.md), and
  [raw literal operators](../raw/literal-operators.md) when fixed/floating
  literal source, commitment, suffixes, or compile-time value formation becomes
  material.
- [Operators](../../language/operators.md), the
  [operator catalog](../../language/operator-catalog.md), and the
  [integer operator catalog](../../language/integer-operator-catalog.md) when
  arithmetic, comparison, reporting, conversion, raw-bit, or protected form
  availability becomes necessary.
- [Raw CPU-provider model](../raw/cpu-provider-model.md) when native/software
  support, aliases, target/compiler-host choice, or capability stability becomes
  concrete.
- [Raw type parameters and generics](../raw/type-parameters-and-generics.md)
  when concrete family parameters, factories, associated types, or
  specialization identity become material.
- [Raw reflection](../raw/reflection.md) when format properties, support,
  aliases, hidden specializations, or operation availability need metadata.
- [Raw compile-time execution](../raw/compile-time-execution.md) when
  compiler-host/target evaluation, constant folding, or unavailable-format
  behavior crosses the scalar boundary.
- [Raw interop](../raw/interop.md) when foreign format, ABI, calling convention,
  serialization, or wire representation becomes concrete.
- [Enums](../../language/enums.md) only when scalar backing eligibility or
  semantic enum interaction becomes material.
- Legacy floating, numeric, casting, or literal material only when a concrete
  question cannot be resolved from the primary notes, current owners, and
  indexed raw input.

### Audit-only

- Archived work only when a concrete provenance or regression question cannot
  be answered from current owners and live raw input.

## Initial working posture

Begin from concrete programmer needs: selecting one supported format, storing a
known value, predicting precision/range, converting deliberately, and
understanding whether work uses hardware or software support. Reconstruct the
maintainer notes into a candidate model without treating generic placeholders,
alias lists, or capability structures as accepted merely because they are
detailed.

Use current integer, structural, and endian owners as constraints. Follow
literal, operator, provider, generic, reflection, and interop consequences only
when a concrete scalar decision crosses those boundaries.

## Working record

Awaiting assignment. Creating and routing work item `022` does not authorize
analysis or begin the floating/fixed-point review.
