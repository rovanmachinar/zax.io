# 019: `using` and scoped resource lifetimes

| Field | Value |
| --- | --- |
| Status | Active working material / non-normative |
| Work Item | `019` |
| Created | 2026-09-09 |
| Owns | The bounded review defined by the fixed initiating input |
| Does Not Own | Exceptions or panic recovery, generalized cleanup hooks, async cancellation, complete owned composition, generic resource concepts, or compiler lowering |

## Non-authority notice

This file is a collaborative working record. Existing statements, new
observations, candidate wording, and aligned findings remain non-authoritative
until a separately discussed, aligned, and explicitly authorized promotion
incorporates them into their lasting owners.

## Fixed initiating input

This section records the information aligned when work item `019` was created.
It is intentionally incomplete and must not be rewritten as work develops.

### Initiating concern

Establish a coherent human-developer-facing model for scoped resource use and
decide the disposition of legacy `using`.

The central question is whether:

```zax
using resource := acquireResource() {
  use(resource)
}
```

provides any material behavior beyond an ordinary declaration in the current
explicit `scope` construct:

```zax
scope resource := acquireResource() {
  use(resource)
}
```

If no distinct semantic guarantee survives review, retire `using` rather than
preserving two spellings that imply different lifetime or cleanup behavior. If a
distinct construct remains justified, define its exact additional contract.

### Motivating pressure

Legacy flow control presents `using` as an always-true, scoped initialization
construct and also proposes `using value own`, which appears to expose or inject
member access. Current Zax already has:

- ordinary declaration and initialization;
- explicit `scope` with initializer, post, labels, and transfers;
- automatic reverse destruction on normal and abrupt scope exit;
- synchronous temporary lifetime rules;
- explicit transfer stances; and
- separate unresolved owned-composition and callable-capture pressure.

The review must distinguish a genuine resource-lifetime operation from redundant
flow sugar and unrelated member-exposure or capture syntax.

### Known assumptions

- Every local, body, and flow-header lifetime ends automatically when its scope
  exits.
- Normal and abrupt exits destroy applicable values in reverse construction
  order.
- A post operation is ordinary normal-completion work, not guaranteed cleanup,
  `finally`, or `defer`.
- Explicit `scope` accepts an initializer, optional post, one effective body, a
  flow label, and applicable `break`/`continue`/`next`/`goto` transfers.
- Direct body re-entry destroys the old body scope, retains active header
  lifetimes, and starts a fresh body scope.
- A declaration can name a temporary and give it the surrounding declaration
  lifetime under existing construction and reference rules.
- No compiler implementation exists in this repository.

### Known inclusions

- Ordinary scoped resource acquisition, initialization, use, and destruction.
- Whether `using` differs from explicit `scope`.
- Binding visibility and temporary lifetime.
- Normal completion and every applicable transfer or panic boundary.
- Post behavior and whether a resource construct needs one.
- Direct re-entry while an acquired header value remains alive.
- Several resources through ordinary composed initialization.
- Acquisition failure through existing construction, optional, result, or panic
  behavior rather than an assumed special mechanism.
- Cost visibility, diagnostics, source stability, and formatting.
- Complete disposition of legacy `using value own`.
- Lasting documentation ownership and legacy/raw disposition.

### Known starting boundaries

- Exceptions, panic recovery, or resumable failure.
- General `defer`, `finally`, cleanup callbacks, or scope guards.
- Async suspension, cancellation, and cross-thread teardown.
- Complete owned-composition or member-exposure syntax.
- General callable capture or closure representation.
- Generic resource concepts, constraints, or context-manager protocols.
- Compiler lowering or runtime implementation.

These boundaries do not erase consequences. Record every material pressure in a
live owner or indexed raw destination.

### Intentionally unresolved framing

- Whether `using` remains a language construct at all.
- Whether a retained `using` is semantic or only narrow source sugar.
- Whether it can express anything not expressible through explicit `scope`.
- Whether acquisition failure changes body entry.
- Whether a retained construct supports labels, posts, or direct re-entry.
- Whether `using value own` belongs to resource lifetime, owned composition,
  member projection, or callable capture.
- Whether resource-specific terminology is justified when ordinary values
  already have deterministic destruction.

### Initial stopping guidance

Stop when the work has:

- established the ordinary scoped-resource mental model;
- decided whether `using` remains, narrows to justified sugar, or retires;
- integrated initializer scope, temporary lifetime, transfers, posts,
  destruction, and completion;
- dispositioned `using value own` without silently designing owned composition;
- preserved exception, cleanup-hook, async, generic, capture, and implementation
  consequences in live destinations;
- identified exact current owners and legacy/raw changes; and
- completed the required documentation-fit dry run.

Do not promote findings, archive this work item, begin work item `020`, or design
general cleanup, owned composition, async cancellation, or generic resource
protocols without the separately required discussion, alignment, and
authorization.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs focused reading,
  numbered work, promotion, teaching, deferrals, and dry runs.
- Legacy [`using` statement](../../flow-control.md#using-statement) - supplies
  the primary historical source and `using value own` pressure.
- Focused [loops and explicit `scope`](../../language/core-flow-control.md#loops-and-explicit-scope),
  [header schemas and binding scope](../../language/core-flow-control.md#header-schemas-ordering-and-binding-scope),
  and [unwinding and completion](../../language/core-flow-control.md#unwinding-destruction-and-completion) -
  establish the current competing flow and transfer model.
- Focused [flow-control initialization](../../language/declarations-and-bindings.md#flow-control-initialization) -
  establishes header declaration visibility.
- Focused [scope-exit destruction](../../language/construction-and-destruction.md#scope-exit-destruction-and-flow-transfers) -
  establishes automatic lifetime ending and explicit lifecycle pressure.
- Focused [common life paths](../../language/lifetimes-and-references.md#common-life-paths)
  and [synchronous temporary extension](../../language/lifetimes-and-references.md#temporary-extension-is-synchronous) -
  constrain resource and temporary lifetime.
- Focused [header sections and separators](../../language/source-structure.md#header-sections-and-separators)
  and [braces and body boundaries](../../language/source-structure.md#braces-and-body-boundaries) -
  constrain any retained source form.

### Consequence-driven

- Read [raw owned composition](../raw/owned-composition.md) only when
  dispositioning legacy `using value own` or a concrete member-exposure claim.
- Read [raw function composition and chaining](../raw/function-composition-and-chaining.md)
  only when callable capture or generated callback behavior becomes material.
- Read [optional values](../../language/optional-values.md) only when a concrete
  fallible acquisition uses absence.
- Read [pointers, allocation, and arenas](../../language/pointers-and-arenas.md)
  only when a concrete resource example depends on allocation ownership or arena
  lifetime.
- Read raw async, safety, analysis-control, generic, or interop input only when a
  concrete scoped-resource rule creates a dependency that must be preserved
  rather than solved.

### Audit-only

- `project/archive/`, including work items `001` through `018`.
- Deleted or superseded resource-management implementation sketches recoverable
  through Git history.

Do not read archived work item `018` during ordinary work on `019`. Its accepted
flow and transfer findings are promoted into current owners.

## Working record

Not started. The agent creating this work item stops here without analyzing its
content.
