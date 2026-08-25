# Raw input: asynchronous execution

| Field | Value |
| --- | --- |
| Status | Raw placeholder / non-authoritative |
| Audience | A future numbered work item defining async language behavior |
| Applies To | Async I/O, coroutines, concurrency, scheduling, and optional runtime support |
| Owns | Preservation of aligned pressures and unresolved boundaries |
| Does Not Own | Accepted async semantics or runtime contracts |
| Source / Provenance | Work items `001`, `005`, and `006`; foundational async, lifecycle, and invocation completion pressure |

## Aligned pressure

Asynchronous programming, coroutines, and concurrency are foundational Zax
concerns. Computer I/O is inherently asynchronous even when exposed through
blocking interfaces.

Zax should provide first-class async semantics without imposing their runtime
cost on programs that do not use them.

## Costs and boundaries to preserve

Using async may require:

- coroutine state machines;
- executors or event loops;
- scheduling;
- allocation;
- synchronization;
- cancellation support;
- platform I/O integration; and
- runtime or library services.

These costs should remain visible and selectable where practical.

Async I/O, coroutines, concurrency, scheduling, and parallelism are related but
distinct. A future design must not collapse them merely because they interact.

## Suspending lifecycle pressure

Current construction, replacement, and destruction are synchronous. A future
suspending constructor would expose partial state across a suspension:

```zax
+++ final async : ()() = {
    _.first.+++()
    await acquireResource()
    _.second.+++()
}
```

Before adopting such behavior, future async work must define:

- where the partial current instance is stored;
- which access paths may exist across suspension;
- cancellation after only some members are live;
- cleanup and allocator responsibility;
- which executor or thread resumes the lifecycle operation;
- whether callbacks or other tasks may observe the storage;
- when final and immutable guarantees activate; and
- whether replacement or destruction may also suspend.

An async factory can perform asynchronous work and then invoke synchronous
construction. That pattern remains available without making lifecycle operations
suspend.

## Ordinary-call suspension pressure

[Zax function invocation](../../language/function-invocation.md) defines call
completion only for synchronous calls. Future async work must decide:

- when an initiating async call is considered complete;
- where parameters, delayed defaults, argument temporaries, and result
  destinations live across suspension;
- what cancellation does after only some inputs or results are complete;
- whether callbacks may run after the initiating expression returns;
- how references into caller or producer temporaries remain valid;
- which executor or thread performs result mapping; and
- how coroutine, allocation, scheduling, and synchronization costs remain
  visible.

## Activation and retirement

Activate this input before promoting async documentation, defining the
language/runtime lowering boundary, or permitting lifecycle suspension. Consume
its findings through that work and retire or archive this placeholder afterward.
