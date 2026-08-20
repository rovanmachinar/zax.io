# Raw input: asynchronous execution

| Field | Value |
| --- | --- |
| Status | Raw placeholder / non-authoritative |
| Audience | A future numbered work item defining async language behavior |
| Applies To | Async I/O, coroutines, concurrency, scheduling, and optional runtime support |
| Owns | Preservation of aligned pressures and unresolved boundaries |
| Does Not Own | Accepted async semantics or runtime contracts |
| Source / Provenance | Work item `001`, Zax purpose and design principles |

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

## Activation and retirement

Activate this input before promoting async documentation or defining the
language/runtime lowering boundary. Consume its findings through that work and
retire or archive this placeholder afterward.
