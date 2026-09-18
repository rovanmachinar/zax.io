# Raw input: export and visibility directives

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work defining exact private, export, selective exposure, and re-export source |
| Applies To | Compiler-directive and declaration syntax layered over the current module visibility baseline |
| Owns | Preserved exact-syntax questions, representative source pressure, activation pressure, and retirement criteria |
| Does Not Own | Current module-internal defaults, explicit-export requirement, import identity, or name lookup ([namespaces and modules](../../language/namespaces-and-modules.md)) |
| Source / Provenance | Legacy compiler-directive and namespacing material narrowed after namespace/module review |

## Current semantic baseline

Future syntax must preserve:

- declarations are module-internal by default;
- source files are not separate default visibility scopes;
- nothing crosses an import unless explicitly exported;
- importing a module does not automatically re-export it;
- a named import binding may be explicitly exported;
- visibility filters eligibility before callable/operator preference;
- imported collision provenance remains visible; and
- no source, declaration, file, import, or module order breaks a tie.

These semantics are current conceptual design. This raw input does not reopen
them.

## Syntax pressure

Legacy material proposed `[[export]]`. Current source reserves `[[ ... ]]` for
lambda capture and uses `[< ... >]` for compiler directives. Future export
syntax therefore starts from:

```zax
// Illustrative export directive; exact export behavior is not accepted here.
[<export>]
MyType :: type {
}

[<export>]
Utilities :: import Module.UtilitiesDefinition
```

and stateful directive forms:

```zax
// Illustrative directive payloads; exact state behavior is not accepted here.
[<export=always>]
[<export=never>]
[<export=yes>]
[<export=no>]
```

Future work must decide whether export is:

- a declaration modifier;
- a compiler directive attached to one declaration;
- a scoped directive state;
- a selective import/exposure list;
- or a bounded combination.

The result must make attachment and source stability explicit. A stateful
directive must not silently export an unrelated later declaration because a
file was reordered, generated, or composed differently.

## Alias and re-export pressure

An alias may deliberately export only its new spelling while the original name
remains unavailable:

```zax
// Illustrative export marker.
[<export>]
PublicType :: alias type InternalType
```

Future work must define:

- whether the target must itself permit re-export;
- how an exact variable or namespace alias is exported;
- how a polymorphic variable family is selected;
- whether an exported import binding preserves one generative module instance;
- how selective exposure renames one declaration;
- whether an exported namespace path recursively exports members or only the
  path;
- how later exports added below that path affect importers; and
- diagnostics when two selected exports collide.

An exported named import must preserve the existing module-instance identity. It
does not perform another import.

## Private pressure

Exact `private` behavior remains unsettled outside already current type-member
eligibility. Future review must decide:

- whether module/namespace-private declarations are visible throughout the
  owning module instance or only an owning declaration context;
- whether source-file privacy is independently useful despite files not being
  namespaces;
- how injected declarations select target-module privacy;
- whether an alias can expose a private target under owner authority;
- how private candidates affect declaration reflection; and
- whether a visibility change can activate a nearer lexical scope or another
  callable candidate.

Ineligible declarations do not compete as worse candidates. Explicitly
qualified inaccessible source reports visibility failure rather than searching
another root.

## Activation and retirement

Activate this input when exact `private`, `[<export>]`, declaration attachment,
selective exposure, alias export, import-module export, or re-export syntax is
reviewed.

Move accepted syntax into the namespace/module, declaration, source, and
compiler-directive owners. Retire this file after every preserved question is
accepted, rejected, or moved to a narrower live destination.
