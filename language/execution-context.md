# Zax execution context

| Field | Value |
| --- | --- |
| Status | Current conceptual design |
| Audience | Human developers using thread-scoped services and context-provided defaults |
| Applies To | The `___` execution context, its application-wide shape, per-thread instance, replacement, and default-arena role; not a formal grammar or runtime ABI |
| Implementation State | Not established by this repository |
| Owns | The programmer-facing execution-context mental model and its relationship to allocation defaults |
| Does Not Own | Allocation syntax and arena behavior ([pointers, allocation, and arenas](pointers-and-arenas.md)); partial-type mechanics; async context switching; arena interfaces; task-local context; or implementation transport |
| Source / Provenance | Legacy context input reconciled with current allocation, lifetime, and concurrency design |
| Supersedes | The retired root context page |

## One context shape, one current instance per thread

`___` names the current execution context for a thread's execution path.
Conceptually:

```zax
currentThread.___
```

Functions reached along that path can use the same current context without
receiving an ordinary explicit parameter at each call.

The complete application has one resolved context type shape:

- the language supplies a core shape;
- permitted partial additions may contribute application-specific services;
- the complete shape is fixed before runtime use; and
- every replacement context instance has that same shape.

Exact partial-type syntax, contribution authority, conflict handling, and build
ordering remain future partial-type work.

## Replacing the current context

A thread may replace its current context:

```zax
currentThread.___ = myReplacementContext
```

`myReplacementContext` must have the application's complete context shape.
Replacement changes what later context-dependent operations observe on that
thread. It does not mutate the shape or retroactively change operations that
already selected and captured a context value.

The in-place context instance need not be internally synchronized merely because
the application has several threads. Each thread has its own current context.
Objects stored in or reached through the context still require whatever
thread-safety, affinity, and sharing contracts their own use demands.

Async tasks, suspension, migration, inheritance into new threads, and restoration
of prior context instances remain future concurrency work. Such work must
preserve well-defined context changes rather than accidentally using whichever
thread resumes a task.

## Default arenas

The core context shape provides access to:

- the default object arena; and
- the default detached control-block arena.

Exact member or function names are not yet established. Conceptually they serve
the roles illustrated by:

```zax
// Illustrative names, not accepted declarations.
___.getDefaultArena()
___.getDefaultControlBlockArena()
```

An allocation with no explicit object arena reads the current context's default
object arena when that allocation operation executes:

```zax
owner : MyValue * unique = @
```

A detached control block with no explicit control-block arena similarly uses the
current context's default:

```zax
owner : MyValue * strong =
  @{ control: ControlBlockPlacement.Detached }
```

An explicit arena overrides only that allocation:

```zax
owner : MyValue * unique = @{ myArena }
```

The complete allocation forms, defaults, ordering, failure, ownership, and arena
requirements are defined by
[Zax pointers, allocation, and arenas](pointers-and-arenas.md#allocate-through-a-declaration).

## Nested allocations

An arena selected for one allocation does not become an implicit default for
allocations performed while constructing that resident instance:

```zax
Parent :: type {
  child : Child * unique = @
}

parent : Parent * unique =
  @{ arena: scratchArena } [{}]
```

`parent` uses `scratchArena`. `child` consults the current `___` when its own
allocation executes.

When nested allocation must use a particular arena, pass that arena through the
constructor or supply a preallocated value through the construction packet. This
keeps the dependency visible and also works when the containing instance uses
stack or global storage rather than an outer dynamic allocation.

Ordinary callees likewise observe the thread's current context. An enclosing
allocation's explicit arena, failure form, disposition, collection policy, and
control placement do not silently rewrite allocations performed by those
callees.

## Costs and diagnostics

Using a context-provided default introduces a dependency on the current thread
context even though source omits an explicit arena operand. Tooling should expose
which context member supplied an effective default.

Representative diagnostics include:

- replacement context has the wrong resolved shape;
- required default arena is unavailable;
- selected context service is incompatible with the current thread;
- context replacement would violate a live operation's captured dependency; and
- async or cross-thread code has no defined context-transfer behavior.

## Boundaries and maturity

This document is current conceptual design, not a context ABI or implementation
mapping.

Still deferred:

- exact core context declaration and member names;
- partial-extension syntax and authority;
- initialization and teardown order;
- context inheritance when creating a thread;
- async task capture, replacement, restoration, and migration;
- task-local or nested context layers;
- reflection;
- compile-time/host/target context relationships; and
- implementation transport, including whether calls physically receive a hidden
  parameter.
