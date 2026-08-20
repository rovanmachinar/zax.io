# Raw input: assembly and foreign interoperability

| Field | Value |
| --- | --- |
| Status | Raw placeholder / non-authoritative |
| Audience | A future numbered work item defining assembly, FFI, ABI, or binding behavior |
| Applies To | Direct assembly and selected C/C++ interoperability |
| Owns | Preservation of aligned requirements and boundaries |
| Does Not Own | Accepted foreign layouts, calling conventions, or binding contracts |
| Source / Provenance | Work item `001`, Zax purpose and design principles |

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

## Activation and retirement

Activate this input before defining assembly integration, FFI, ABI controls, or
binding behavior. Consume its findings through that work and retire or archive
this placeholder afterward.
