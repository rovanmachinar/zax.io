# Raw input: pointer and arena mechanics

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work defining complete pointer, arena, control-block, provenance, or allocation syntax |
| Applies To | Mechanics deferred by current lifetime and pointer design |
| Owns | Preservation of unresolved allocation grammar and failure operators, arena interfaces, custom control blocks, deeper/unsafe ownership anchoring, pointer provenance and casts, pointer-layer presence mechanics, cycle tracing, recovery and prompt-disposition policy, and thread-affine release |
| Does Not Own | Current reference lifetime or pointer ownership semantics; future `Nothing` resident-instance and representation behavior ([Nothing input](nothing-instances.md)) |
| Source / Provenance | Former raw lifetime input; legacy `pointers.md`, `memory-allocation.md`, `custom-allocators.md`, `strong-weak.md`, and `handle-hint.md`; work item `014` |

## Current constraints

Current behavior is owned by:

- [Zax lifetimes and references](../../language/lifetimes-and-references.md);
- [Zax pointers and arenas](../../language/pointers-and-arenas.md);
- [Zax transfer stances](../../language/transfer-stances.md); and
- [Zax safety and analysis](../../language/safety-and-analysis.md).

Future work must preserve:

- permanent reference binding;
- raw, `unique`, `unique shareable`, `strong`, `weak`, and anchored pointer roles;
- pointer-layer `atomic` as shared lifetime accounting rather than pointee thread
  safety;
- arena-backed dynamic allocation;
- independent destruction timing, storage recovery, and cycle probing;
- inline or detached control blocks;
- destination-directed ownership transitions;
- weak probing without acquisition;
- weak-to-strong `copy`;
- direct-member `anchored by`;
- and no anchored-to-`unique` conversion.

## Allocation source syntax

Future grammar must express:

- object arena;
- optional separate control-block arena;
- prompt or attached destruction;
- prompt or arena-wide storage recovery;
- cycle probing;
- pointer ownership result;
- inline or detached control-block preference;
- custom arena or control-block implementation;
- constructor inputs; and
- allocation failure.

Illustrative shape only:

```zax
value : MyValue * strong =
  allocate MyValue
    in myObjectArena
    control in myControlArena
    destruction prompt
    recovery arena
    cycles enabled
```

No spelling in this example is accepted syntax.

The final form should avoid one operator or pointer type for every policy
combination while keeping material costs visible.

## Allocating shareability

A blockless unique owner cannot become strong until a control block is
allocated:

```zax
owner : MyValue * unique

// Illustrative future source.
prepared : MyValue * unique shareable =
  owner with control in myControlArena
```

Future work must decide:

- whether the operation is a constructor, operator phrase, allocation packet, or
  library-backed language operation;
- how failure is represented;
- how an inline block can be added without relocating the resident instance;
- when only a detached block is possible;
- how the allocation disposition is registered;
- and how custom block implementations expose local and atomic shared modes.

Shedding a detached block may recover its allocation. Shedding an inline block
retires its semantic role but normally cannot recover its bytes without
relocating the instance.

## Deeper and unsafe ownership anchoring

Current `anchored by` accepts a statically recognized direct member:

```zax
member : Item * strong anchored =
  container.item anchored by container
```

Future work must decide whether one uninterrupted chain of direct composition is
also safe:

```zax
leaf : Leaf * strong anchored =
  container.branch.leaf anchored by container
```

The path must not cross optional presence, pointer/reference dereference,
variant activity, dynamic-array relocation, unmanaged overlay state, or another
allocation.

Legacy `unsafe lifetime of` preserved arbitrary adoption of another pointer's
ownership control block. If an unsafe successor exists, it must state the exact
relationship being claimed:

- target lies within the ownership anchor's allocation;
- target instance place remains valid for every use;
- replacement or relocation behavior is intentional;
- representation and alignment are suitable; and
- the anchored target cannot become a unique allocation root.

Illustrative future source:

```zax
/// myTarget is a stable direct subplace of owner despite opaque projection.
myTarget unsafe anchored by owner
```

The spelling is not accepted.

## Pointer provenance and conversion

Pointer-representation integer capacities are owned by
[Zax integers](../../language/integers.md#pointer-representation-integer-roles).
Numeric fit does not establish pointer validity.

Future pointer work must define:

- raw pointer creation and address-of behavior;
- how a blockless unique pointer retains or recovers its object arena,
  disposition, destructor, size, and alignment without a shared control block;
- pointer-to-integer and integer-to-pointer conversion;
- provenance preservation;
- alignment;
- segment or address-space identity;
- comparability and ordering;
- pointer subtraction and `PointerDelta`;
- one-past and range boundaries;
- casts among pointer layers;
- ownership-preserving versus ownership-losing conversions;
- and FFI adoption.

Near, ordinary, and far representations may share machine layouts while
retaining distinct source identities and intent.

`pointerA delta pointerB` must not overflow for a semantically valid difference,
but that does not make arbitrary pointers comparable.

## Pointer-to-`Nothing` behavior

Current pointers may contain `Nothing`, and `?pointer` reports the guarantee
appropriate to the pointer role. Real monitored sentinels, custom global
`Nothing`, generated member-access checks, compatible overlap, and exact
representation are preserved by
[raw Nothing-instance input](nothing-instances.md).

No representation may weaken the rule that a raw non-`Nothing` pointer does not
by itself prove a live pointee.

## Arena interfaces

Every dynamic allocation uses an arena, but complete interfaces remain open:

- allocation request shape;
- size and alignment types;
- backing block acquisition, release, and bulk reset;
- concurrency and thread-affinity declarations;
- capacity and memory-pressure reporting;
- object and control-block arena relationships;
- failure and panic behavior;
- nested arenas;
- arena ownership;
- and queued release for thread-confined arenas.

The arena supplies basic storage operations. Language-managed machinery owns
object-by-object construction/destruction registration, control blocks,
attached-lifespan and deferred-recovery lists, collector metadata, and teardown
validation.

Arena implementation may recover memory immediately, lazily, in batches, or at
teardown. Current allocation disposition controls the programmer-visible
destruction and recovery contract; an implementation must not silently change
it.

## Allocation failure operators

Future source design must expose:

- a panicking allocation form that returns a non-`Nothing` pointer or panics;
- a non-panicking form that returns `Nothing` on arena exhaustion;
- an unchecked form whose false success guarantee has undefined consequences;
- automatic member allocation during stack/global/container construction;
- and propagation of allocation choice into nested automatic allocations.

The underlying arena request may report failure independently from the operator
that chooses panic, `Nothing`, or unchecked behavior.

## Prompt-disposition requirements

Ordinary `strong` ownership may use prompt, deferred-recovery, attached, or
collected allocation behavior. A generic API that accepts only `T * strong`
therefore cannot assume that final release immediately runs the destructor or
recovers storage.

Future work must decide how an API requests:

- prompt destruction;
- prompt storage recovery;
- an explicit resource-closing operation independent of lifetime;
- rejection of attached or collected disposition;
- or a generic allocation-disposition constraint.

The requirement must remain visible in source and callable compatibility rather
than depending on hidden allocation history.

## Teardown and recovery

Future arena work must define exact ordering among:

- closing the arena to new allocations;
- cycle probing;
- attached-lifespan destruction;
- prompt-destroyed deferred-recovery blocks;
- externally owned allocations;
- inline control blocks retained by weak pointers;
- detached control blocks in another arena;
- destructor-created work;
- panic during teardown; and
- final backing-storage release.

The current contract requires panic rather than successful teardown when a
dependency would outlive required backing storage.

## Cycle collection

Cycle probing is optional allocation behavior layered on strong ownership.

Future work must define:

- compiler emission of collector support only when reachable build output uses
  collectable allocations;
- root registration or stack/global metadata;
- how external roots are distinguished from strong edges inside a candidate
  cycle;
- how anchored pointers participate;
- whether custom types expose outgoing strong edges through reflection or
  generated traversal;
- collector interaction with local versus atomic control blocks;
- concurrent graph mutation;
- eviction ordering;
- destructor reentrancy;
- weak termination publication;
- attached-lifespan behavior after cycle eviction; and
- library-provided memory-pressure events and explicit programmer-triggered
  collection APIs.

An unreachable cycle is not merely “strong count greater than zero.” Collection
requires proving that no ownership path from outside the candidate graph
remains.

Mark state may live in the existing control block, so collectable pointers need
not be wider. The compiler still needs generated traversal or equivalent type
metadata to discover outgoing strong edges.

## Thread affinity and release

Arenas are not inherently thread-safe. An arena may be thread-confined,
synchronized, movable under exclusive ownership, or able to queue work to an
owning thread while resident instances remain separately thread-affine.

Future work must define:

- whether ownership may move to another thread;
- where final destruction may execute;
- whether the arena itself may receive allocation, release, or teardown on that
  thread;
- executor or thread scheduling for affine destruction;
- what happens when scheduling fails;
- whether control-block arenas may be released from another thread;
- interaction with cancellation;
- and which guarantees an atomic pointer actually exposes.

Pointer-layer `atomic` protects shared lifetime accounting only.

## Arrays and relocated elements

An array implementation may keep elements contiguous, segmented, or otherwise
organized. If future design provides an owning element view, it must not claim
that keeping the array allocation alive keeps one relocated element place valid.

Array operation contracts and element-place stability are retained by
[raw indexing and slicing input](indexing-and-slicing.md). Pointer provenance
work must consume those guarantees rather than inventing a hidden stability
promise.

## Activation and retirement

Activate this input when complete pointer syntax, allocation grammar, arena
interfaces, control-block customization, unsafe anchoring, provenance, pointer
casts, pointer-layer presence mechanics, cycle collection, memory recovery, or
thread-affine release is reviewed. `Nothing` resident-instance, representation,
and dereference behavior belongs to [raw Nothing input](nothing-instances.md).

Move accepted behavior into domain-oriented current owners, preserve remaining
future concerns in narrower indexed inputs, then retire this file.
