# 010: Fundamental integer types and finite bit domains

| Field | Value |
| --- | --- |
| Status | Active working material / non-normative |
| Work Item | `010` |
| Created | 2026-08-29 |
| Owns | The bounded review defined by the fixed initiating input |
| Does Not Own | Accepted complete floating-point, fixed-point, arbitrary-width, enum, generic, literal, reflection, iteration, ABI, formal grammar, or compiler behavior |

## Non-authority notice

This file is a collaborative working record. Existing statements, new
observations, candidate wording, and aligned findings remain non-authoritative
until a separately discussed, aligned, and explicitly authorized promotion
incorporates them into their lasting owner documents.

## Fixed initiating input

This section records the information aligned when work item `010` was created.
It is intentionally incomplete rather than exhaustive and must not be rewritten
as the work develops.

### Initiating concern

Develop a coherent human-developer-facing model for Zax's fundamental finite
integer types and the bit domains on which current integer operations depend:

1. what makes a type fundamental or intrinsic;
2. signed and unsigned fixed-width integer families;
3. concrete public names, aliases, and type identity;
4. value domains, storage representation, and logical bit extent;
5. width, size, alignment, padding, and portability;
6. context-selected or CPU-natural integer forms;
7. integer conversion and signedness relationships;
8. the current arithmetic, policy, shift, rotate, count, mask, reduction,
   reversal, and extraction/deposit operation domains;
9. the canonical shift-compatible bit-count result type;
10. endian backing eligibility and representation consequences;
11. costs, failures, diagnostics, reflection pressure, and source stability; and
12. lasting documentation owners and boundaries.

Use the current operator, catalog, qualifier, declaration, and endianness owners
as constraints. Recover Zax-specific intent before borrowing a numeric model
from another language.

### Motivating pressure

Current conceptual owners already rely on concrete intrinsic integer domains:

- fixed-width signed and unsigned operands have protected operator signatures;
- arithmetic policy forms distinguish required, optional, wrapping, saturating,
  and reporting behavior;
- shifts and rotates depend on a finite logical bit extent;
- count, set-bit mask, and set-bit position operations depend on exact width;
- set-bit positions return an optional canonical count directly usable as a
  built-in shift count;
- multiword shifts, reversal, and masked extraction/deposit remain accepted or
  deferred concepts constrained by finite extent;
- endian semantic enums require a fundamental integer backing type with known
  size, alignment, and representation; and
- future enums, literals, generics, reflection, and compatibility need a stable
  programmer-facing integer foundation.

Legacy material proposes many concrete, context-sized, CPU-sized, aliased, and
generic-looking integer forms. Checked-in names and explanations remain evidence
rather than accepted design merely because they exist.

### Known assumptions

- [Zax language vision](../../language/vision.md) is accepted foundational
  direction.
- [Zax language principles](../../language/principles.md) owns reusable
  cross-cutting design and use principles.
- [Zax operators](../../language/operators.md) owns shared operator discovery,
  selection, results, evaluation, protection, and failure categories.
- [Zax operator catalog](../../language/operator-catalog.md) owns current exact
  operator forms, precedence, protected integer operation families, and accepted
  arithmetic/bit behavior.
- [Zax endianness](../../language/endianness.md) owns the current semantic enum
  model that requires applicable fundamental integer backing types.
- [Zax qualifiers](../../language/qualifiers.md) owns value, access, and
  place-replacement qualifications applied to integer values and places.
- [Zax declarations and bindings](../../language/declarations-and-bindings.md)
  owns type/value naming intent, aliases at the currently established depth, and
  declaration boundaries.
- [Raw numeric type-family input](../raw/numeric-type-families.md) preserves
  future signedness, finite-extent, representation, and policy questions.
- Every successfully selected single expression result has a concrete static
  type, including compiler-known intermediate results.
- Concrete programmer-visible integer types can be defined before their future
  shared generic skeleton is known.
- A future generic model must explain or generate established concrete types
  without silently changing their names, identity, ranges, representation,
  layout, protected signatures, conversions, or source behavior.
- There is no language implementation in this repository.

### Known inclusions

- The conceptual distinction between fundamental, intrinsic, language-provided,
  alias, and custom integer types.
- The concrete signed and unsigned fixed-width family.
- Exact public type names and whether names such as `Byte`, `U8`, `Integer`, and
  `UInteger` are distinct identities, aliases, or context-selected forms.
- Minimum and exact widths, value ranges, zero, sign representation, and whether
  every storage bit participates in the logical value.
- Logical bit extent versus storage size, alignment, allocation capacity, and
  padding.
- Host, target, native, CPU-natural, and portable width pressure.
- Default initialization and zero/default value pressure at the depth needed to
  understand integer use.
- Literal realization pressure at the depth needed to establish integer value
  domains, without completing literal grammar.
- Widening, narrowing, signedness changes, exact-value conversions, panic,
  truncation, and explicitness.
- Equal-width signedness counterparts and their relationship to ordinary
  conversion.
- Current protected arithmetic, comparison, bitwise, count, reduction, shift,
  rotate, magnitude, mutation, and swap domains.
- Required, optional, wrapping, saturating, and reporting operation policies as
  they constrain the integer type model.
- Shift-count input requirements.
- The canonical unsigned count type shared by leading/trailing-zero counts and
  optional set-bit positions, including direct viability as a built-in shift
  count.
- Leading/trailing-zero counts, leading/trailing set-bit masks, and optional
  set-bit positions.
- Finite-extent pressure on population count, reductions, reversal, rotates,
  masked extraction/deposit, and multiword operations.
- Signed mask results and the distinction between bit-pattern behavior and
  arithmetic overflow.
- Fundamental backing eligibility for full-domain endian semantic enums.
- Storage/shape compatibility pressure without treating compatibility as
  conversion permission.
- Costs, panic and optional paths, compile-time failures, diagnostics,
  portability, optimization freedom, formatting, and source stability.
- Documentation fit, including whether detailed integer teaching warrants a
  dedicated current owner.
- The future possibility of a concise basic-types guide that summarizes and
  routes to mature integer, floating-point, string, enum, literal, and other
  foundational type owners.

### Known starting boundaries

- Complete floating-point semantics and representation.
- Complete fixed-point and arbitrary-width integer behavior.
- Complete enum declaration, validity, iteration, reflection, and operation
  inheritance.
- Complete generic syntax, constraints, specialization, instantiation, and
  generated type-family behavior.
- Complete literal parsing, prefix, escaping, payload, and custom realization.
- Complete structural identity, subtyping, storage compatibility, and truncating
  transfer.
- Complete reflection and compile-time execution.
- Complete ABI, foreign interoperability, serialization framework, and memory
  layout contract.
- Formal grammar, compiler implementation, lowering, instruction selection, and
  toolchain internals.

These areas need not be designed to begin. They do not prohibit following,
recording, or dispositioning material consequences.

### Intentionally unresolved framing

- The exact concrete intrinsic integer catalog.
- Whether `Integer` and `UInteger` are fixed, minimum-width, optimal CPU-width,
  target-context, or other selected forms.
- Which names are concrete type identities versus identity-preserving aliases.
- The relationship between `Byte`, character storage, raw bytes, and `U8`.
- Whether signed integers use one required representation or only a value-domain
  contract.
- Whether fundamental integer storage may contain padding outside logical bit
  extent.
- Exact size and alignment portability guarantees.
- Exact widening, narrowing, truncation, panic, and unsafe conversion behavior.
- The canonical bit-count type's name and representation.
- Which deferred bit operations receive exact phrase forms.
- How context-sized types behave during compiler-host versus target execution.
- How future generic syntax explains the concrete family.
- Whether a basic-types overview is justified now, later, or not at all.
- Which findings belong in an integer owner, the operator catalog, endianness,
  literal, compatibility, generic, reflection, or future owners.

### Documentation-structure pressure

Detailed integer behavior may justify a dedicated current owner such as
`language/integers.md`. A future concise `language/basic-types.md` could provide
a mental map and route to mature detailed owners for integers, floating point,
strings, enums, literals, and other foundational types.

This work does not presume either file must be created. The documentation-fit
dry run must decide whether integer teaching already warrants its own owner and
whether `index.md` remains sufficient basic-type routing until several sibling
owners exist. Do not scaffold empty or speculative type-document families.

### Initial stopping guidance

Stop when the work has:

- established a usable programmer model for every current fundamental finite
  integer form;
- established concrete names, identity and alias relationships, widths, ranges,
  logical extent, storage, and portability;
- reconciled every current protected integer operator family with its valid type
  domains;
- established conversion and signedness relationships at the depth programmers
  need;
- established the canonical shift-compatible bit-count result requirement;
- established or explicitly deferred the unresolved reversal,
  extraction/deposit, multiword, and unusual-width behavior;
- constrained endian backing, enums, literals, generics, reflection,
  compatibility, and compile-time contexts without silently designing them;
- identified costs, failures, diagnostics, formatting, and source-stability
  consequences;
- preserved every deferred consequence in a live owner or indexed raw
  destination;
- identified plausible lasting owners and whether a detailed integer owner or
  later basic-types guide is justified; and
- gathered enough information for a pre-promotion documentation-fit dry run.

Do not promote findings, create speculative type-document families, archive this
work item, or begin work item `011` without the separately required discussion,
alignment, and authorization.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs ownership,
  focused reading, numbered work, promotion, and archival.
- [Zax language vision](../../language/vision.md) - provides foundational
  purpose, audience, and language direction.
- [Zax language principles](../../language/principles.md) - provides explicit
  cost, readability, optimization, and source-stability constraints.
- [Zax operators](../../language/operators.md) - provides shared operator
  selection, result, evaluation, protection, and diagnostic behavior.
- [Zax operator catalog](../../language/operator-catalog.md) - provides exact
  current integer operation forms, policy families, protected domains, and
  finite-bit behavior that the type model must support.
- [Zax endianness](../../language/endianness.md) - provides the current
  fundamental-backing use case and representation/semantic distinction.
- [Zax qualifiers](../../language/qualifiers.md) - provides qualification truth
  for integer values, places, and receiver operations.
- [Zax declarations and bindings](../../language/declarations-and-bindings.md) -
  provides naming intent, declaration, alias, and type-boundary constraints.
- [Raw numeric type-family input](../raw/numeric-type-families.md) - preserves
  signedness, finite extent, representation, and operation-policy questions.
- Focused [legacy intrinsic types](../../basics.md#intrinsic-types) - preserves
  the existing candidate type catalog, names, widths, and alias intent.
- Focused [legacy casting](../../casting.md) - preserves current evidence about
  intrinsic widening, narrowing, signedness, panic, and unsafe truncation.

### Consequence-driven

- Read [raw literal input](../raw/literal-operators.md) when integer value range,
  default type, base, width, or signedness makes literal realization necessary.
- Read [raw structural-typing input](../raw/structural-typing.md) when storage,
  shape, representation, alias identity, or truncating transfer constrains the
  integer model.
- Read [raw type-parameter and generic input](../raw/type-parameters-and-generics.md)
  when the public identity of a concrete integer depends on future family syntax.
- Read [raw enum input](../raw/enum-types.md) only when a finding constrains
  fundamental backing eligibility or full-domain enum behavior.
- Read [raw reflection input](../raw/reflection.md) when integer family
  relationships or representations must be discoverable.
- Read [raw compile-time execution input](../raw/compile-time-execution.md) when
  host, target, native, or context-sized behavior constrains integer meaning.
- Read [raw interoperability input](../raw/interop.md) when ABI or foreign-layout
  pressure becomes unavoidable.
- Read focused legacy compiler directives, meta-types, or meta-functions only
  when a concrete host/target or generic consequence requires their evidence.
- Expand beyond this list whenever discussion or a material finding establishes
  a concrete need. State why the additional reading is relevant and keep it
  focused.

Material not listed in this reading scope is neither required nor prohibited. It
remains available when discussion or a concrete consequence makes it relevant.

### Audit-only

- `project/archive/`, including work items `001` through `009`.
- The historical `operator.md` path.

Do not read archived work item `009` while conducting ordinary work on `010`.
Its applicable findings have been promoted into current owners and indexed raw
inputs. Read it only when the language maintainer explicitly requests a targeted
audit or provenance investigation.

## Working record

Not started. The agent creating this work item stops here without analyzing its
content.
