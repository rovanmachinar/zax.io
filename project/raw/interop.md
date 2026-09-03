# Raw input: assembly and foreign interoperability

| Field | Value |
| --- | --- |
| Status | Raw placeholder / non-authoritative |
| Audience | A future numbered work item defining assembly, FFI, ABI, or binding behavior |
| Applies To | Direct assembly and selected C/C++ interoperability |
| Owns | Preservation of aligned requirements and boundaries |
| Does Not Own | Accepted foreign layouts, calling conventions, or binding contracts |
| Source / Provenance | Work items `001` and `012`, Zax purpose/design principles, and optional layout/ABI pressure |

## Aligned requirements

- Direct integration with assembly is required.
- Binding to C and C++ is important.
- Interoperability is selective in both directions.
- Zax-native facilities need not all have foreign equivalents.
- Foreign-language facilities need not become Zax concepts.
- C-compatible representations, functions, symbols, and calling conventions are
  likely the most direct boundary.
- Adapters are appropriate when semantic models differ.
- Manual ABI work may remain possible through explicit low-level mechanisms
  without first-class guarantees.

## Representative boundary

C++ virtual functions are not a first-class Zax concept.

A C++ adapter, explicit function table, or manually interpreted known ABI may
bridge a particular case. Direct vtable interpretation is compiler-, ABI-,
layout-, and version-sensitive and belongs at an explicit low-level trust
boundary.

The general pressure is:

> Zax supports selected foreign representations and calling conventions without
> importing the foreign language's complete semantic model.

## Optional representation and ABI pressure

[Zax optional values](../../language/optional-values.md) defines semantic states
without fixing a tag, niche, layout, or calling convention. Future interop work
must decide explicitly:

- whether a selected foreign boundary uses a separate discriminant or a valid
  niche;
- how nested optional states remain distinguishable;
- whether optional pointers/functions preserve their inner null/`Nothing` state;
- size, alignment, padding, parameter passing, and result return;
- stability across modules, toolchains, targets, and language versions;
- adapter behavior when a foreign optional/nullability model has fewer states.

Representation coincidence on one target does not make two semantic states or
type layers equivalent. Manual ABI use remains an explicit low-level boundary
until a named contract accepts an exact optional representation.

## Activation and retirement

Activate this input before defining assembly integration, FFI, ABI controls, or
binding behavior. Consume its findings through that work and retire or archive
this placeholder afterward.
