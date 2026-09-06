# Raw input: global and once lifetimes

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work defining exact global and lazy-once lifecycle ordering |
| Applies To | Cross-file global construction, dependency ordering, circular initialization, `once`, teardown, and concurrency |
| Owns | Preserved legacy ordering and failure pressure, activation pressure, and retirement criteria |
| Does Not Own | The current process life-path mental model |
| Source / Provenance | Legacy `ctor-dtor.md`, `basics.md`, and work item `014` |

## Current boundary

[Zax lifetimes and references](../../language/lifetimes-and-references.md)
establishes that global instances live in the process life path and are destroyed
in reverse of their established construction order.

This file preserves how that exact order is selected.

## Ordering pressure

Legacy design proposed:

- declaration/source order as the ordinary global construction order;
- dependency detection that may require one global to initialize before another;
- a diagnostic for circular global initialization dependencies;
- reverse destruction order;
- lazy `once` construction interleaved with global construction history; and
- thread-aware initialization for `once`.

Future work must reconcile those proposals with:

- module/source loading order;
- compile-time versus target execution;
- imported module initialization;
- repeated or failed initialization;
- panic;
- concurrency;
- dynamic library loading;
- and source stability when dependency analysis improves.

No exact ordering rule in this file is accepted.

## Activation and retirement

Activate this input when global initialization, `once`, module startup, process
teardown, or cross-file lifecycle ordering is reviewed. Move accepted behavior
into the lifecycle and source/module owners, then retire this file.
