# Raw input: CPU provider model

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work defining target and compiler-host CPU profiles, native scalar representations, and integer selection |
| Applies To | Native representation classification, integer factories, machine-word families, pointer/size capacities, mapping stability, metadata, and fallback |
| Owns | Preserved CPU-provider requirements, constraints, representative source relationships, activation pressure, and retirement criteria |
| Does Not Own | Accepted CPU-profile format or current integer/ABI semantics |
| Source / Provenance | Integer type work and existing native/compiler-host/target execution terminology |

Current programmer-visible integer roles, ranges, and profile constraints are
owned by [Zax integers](../../language/integers.md). This input retains the
future profile format, versioning, metadata, and compatibility mechanism.

## Native representation

A CPU provider classifies scalar integer widths that the selected CPU natively
represents. This says only that the CPU has a native representation for that
width. It does not promise direct multiplication, division, atomics, shifts, or
any other individual operation.

For example, an eight-bit target may classify `Byte` as native while a
minimum-sixteen-bit `Integer` uses a software-composed representation. The same
CPU may still lack an eight-bit multiplication instruction.

Per-operation instruction availability and cost are separate metadata.

## Integer selector facts

A provider supplies concrete selections satisfying language family contracts:

- widths classified as native scalar representations;
- provider-preferred fastest native choices;
- smallest qualifying native choices;
- the natural `Word` width;
- widest native signed and unsigned representations;
- ordinary, near, and far pointer-representation widths;
- type-size and index-size capacities;
- ordinary bit-count and byte-count representations; and
- software fallbacks when no native width satisfies a selector.

The integer factory consumes minimum/maximum width, signedness, preference,
environment, and fallback dimensions and returns one exact intrinsic
specialization.

The provider's concrete facts may eliminate checks and determine layout. They do
not ordinarily change source conversion availability established by public
family range contracts.

### Optimized maximum

`OptimizedMaxI`/`OptimizedMaxU` preserve a selection need distinct from native,
supported, and language maxima:

> Select the widest integer family for which the provider supplies its preferred
> general-purpose optimized implementation, whether through native
> representation or deliberately optimized software emulation.

An eight-bit CPU may select `U8` as `NativeMaxU` and an optimized 16-bit software
implementation as `OptimizedMaxU`. A 32-bit profile may select optimized 64-bit
behavior without claiming native 64-bit representation.

Future profiles must expose the selected width, broad cost expectations, and
which important operations use native instructions or software emulation.
“Optimized” does not promise one instruction or universal fastest behavior.

## Machine-word relationships

The provider chooses `Word` width `W`. Zax derives:

```text
HalfWord = ceil(W / 2)
DWord    = W * 2
QWord    = W * 4
OWord    = W * 8
```

The provider cannot assign unrelated meanings to these names. A derived width
without native representation uses software support. A width beyond the
language maximum makes that derived family member unavailable for the profile.

## Count and capacity relationships

The provider chooses ordinary `BitCount` and `ByteCount` representations subject
to language minima and the profile's predefined scalar families. Large count
types cover the language-wide maximum independently of ordinary native width.

Within each memory domain:

```text
IndexSize.maximum = TypeSize.maximum
IndexSize representation = TypeSize representation
```

Across memory domains:

```text
Near pointer capacity <= ordinary pointer capacity <= Far pointer capacity
Near.TypeSize.maximum <= TypeSize.maximum <= Far.TypeSize.maximum
Near.IndexSize.maximum <= IndexSize.maximum <= Far.IndexSize.maximum
```

Mirrored representations do not collapse distinct near/ordinary/far intent
identities.

## Mapping stability

Changing a provider selection may change storage, alignment, overflow boundaries,
generated code, and compatibility. Future work must decide:

- which mappings belong to a versioned target contract;
- how profile versions are selected and recorded;
- which changes are source, ABI, cache, or serialization compatibility events;
- how compiler-host and target profiles coexist in one build;
- how shared code names active versus explicitly pinned environment types; and
- which diagnostics identify an unsupported or incomplete profile.

ABI correspondence remains interoperability work. A native representation does
not by itself promise a foreign ABI type or calling convention.

## Metadata requirements

Tooling and reflection need requested selector dimensions, selected exact width,
native/software-emulation classification, environment, fallback, optimized
maximum, profile identity/version, and per-operation capability/cost information
where later defined.

Diagnostics should lead with public integer role names and expose profile and
exact-specialization detail on demand.

## Activation and retirement

Activate this input for CPU profiles, target descriptions, compiler-host/target
execution environments, native scalar selection, machine-word families, pointer
and size capacities, or mapping compatibility. Move accepted behavior into
target-profile, integer, build, reflection, compatibility, and interoperability
owners, then retire this file after every preserved requirement is dispositioned.
