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

Current [declarations](../../language/declarations-and-bindings.md#type-callable-once-functions)
and [invocation](../../language/function-invocation.md#type-and-instance-calls-to-once-functions)
also establish the narrow function rule: a `once` function has one type-owned
implementation and is callable through either the type or an instance.

This file preserves global and `once` **value** lifecycle behavior, plus the
remaining capture, concurrency, teardown, and generic-specialization questions.
It does not reopen the accepted type/instance call surface.

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

## Generative module-state pressure

Current [namespace/module design](../../language/namespaces-and-modules.md)
establishes:

- every import creates a distinct module instance;
- each instance has individualized module-global state;
- equal source and injection inputs do not merge that state;
- exact injected namespace aliases may deliberately let several module
  instances use one already imported dependency and its state; and
- there is no automatic super-module global facility.

Future lifecycle work must decide:

- when each generative module instance constructs and destroys its globals;
- how import/source order and detected dependencies interact;
- whether a module instance initializes eagerly, on first reachable use, or
  under another explicit policy;
- how an exactly injected shared dependency is initialized once and ordered
  against each consumer;
- failure, retry, panic, and partial-initialization behavior;
- reverse destruction when several import paths reference one injected shared
  instance; and
- whether a broader shared-state facility is needed after concrete use cases
  that exact injection cannot express.

Cache or generated-code reuse must not merge state. Conversely, different local
aliases to one deliberately injected module instance must not duplicate its
state accidentally.

## Activation and retirement

Activate this input when global initialization, `once`, module startup, process
teardown, or cross-file lifecycle ordering is reviewed. Move accepted behavior
into the lifecycle and source/module owners, then retire this file.
