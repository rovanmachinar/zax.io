# Zax pointers and arenas

| Field | Value |
| --- | --- |
| Status | Current conceptual design |
| Audience | Human developers choosing dynamic-storage, ownership, sharing, and pointer-lifetime behavior |
| Applies To | Programmer-facing raw and managed pointers, ownership transitions, weak observation, arenas, allocation disposition, control blocks, and cross-thread lifetime accounting; not a formal grammar or ABI |
| Implementation State | Not established by this repository |
| Owns | Raw, `unique`, `shareable`, `strong`, `weak`, `anchored`, and pointer-layer `atomic` behavior; arena-backed dynamic allocation; destruction/recovery/cycle-probing choices; control-block obligations; pointer presence and weak acquisition; pointer ownership transitions; pointer-facing costs and diagnostics |
| Does Not Own | Non-owning references and life paths ([lifetimes and references](lifetimes-and-references.md)); complete transfer-stance behavior ([transfer stances](transfer-stances.md)); pointer representation integers ([integers](integers.md)); pointee operation thread safety; or exact allocation, arena, control-block, and pointer-cast syntax |
| Source / Provenance | Legacy pointer, memory-allocation, custom-allocator, strong/weak, handle/hint, and `Nothing` evidence reconciled with current transfer and lifetime design |
| Supersedes | Current-purpose portions of the retired root pointer and allocation pages |

## Choose ownership separately from allocation

Every dynamic allocation receives storage from an arena. The ordinary process
heap is one arena; a program may provide others.

The pointer tells readers who owns or observes the allocated life path:

```zax
rawView : MyValue *
soleOwner : MyValue * unique
preparedOwner : MyValue * unique shareable
sharedOwner : MyValue * strong
weakObserver : MyValue * weak
```

The allocation separately determines:

- when the resident instance is destroyed;
- when its backing block becomes reusable;
- whether unreachable strong-reference cycles may be collected;
- which arena supplies object storage; and
- where any ownership control block lives.

Separating these axes prevents one pointer family for every combination of
ownership, allocator, destruction timing, and thread behavior.

## Pointer instances and pointees

A pointer is an ordinary instance with its own life path. Its value identifies
another instance place or `Nothing`.

The pointer place and pointee place are distinct:

```zax
pointer : MyValue * = first
pointer = second // repoints the pointer; does not replace first
```

Replacing or destroying a pointer instance affects the pointee only when that
pointer's ownership contract requires it.

Pointer-value transfer never implies pointee transfer by itself. A raw pointer
move moves only an address-like value. A unique-owner move transfers the owned
life path because `unique` explicitly says it does.

## Pointer ownership roles

### Raw pointers

An unqualified `*` is raw:

```zax
view : MyValue *
```

A raw pointer:

- may point to `Nothing`;
- may be repointed;
- owns no life path;
- performs no automatic pointee destruction;
- provides no automatic lifetime extension;
- and may become invalid if its pointee path ends or its provenance is wrong.

Raw pointers are not categorically unsafe. A particular operation is safe when
analysis proves every required fact. For example, a type may contain a raw
pointer member whose pointee path is automatically allocated, constructed,
destroyed, and recovered by the **containing type's declared lifecycle**. The
raw pointer remains non-owning; the containing lifecycle is the independently
proved owner responsible for ending the nested path. Access within that bounded
relationship requires no unsafe marker.

The proof may be lost when the pointer escapes the container, invocation, or
owner that established it. At that boundary the compiler must prove the extended
relationship, require narrow unsafe responsibility, or reject a known-invalid
use.

Creating a dynamic path with no independently tracked owner or containing
lifecycle—only a raw pointer value—remains unsafe: losing or repointing that
pointer may leave no authority able to end the path.

`?view` proves only that the raw pointer value is not `Nothing`. It does not
by itself prove a live pointee, valid provenance, alignment, or permitted
access. When origin and lifetime analysis already proves those facts, the
presence test may complete the safe proof.

### Unique ownership

```zax
owner : MyValue * unique
```

`unique` means one pointer owns the allocation-root life path. Ownership may be
transferred, but it is never shared concurrently with another owning pointer.

A blockless unique owner has no ownership control block. This is the
minimum-overhead managed pointer form.

It still retains or can recover the allocation-root arena, disposition,
destructor, size, and alignment information required for final release. That
provenance may come from arena metadata, the allocation layout, or another
representation chosen by the implementation; it is not a shared ownership
control block.

```zax
nextOwner : MyValue * unique = owner as move
```

After successful transfer, `owner` is vacant and `nextOwner` owns the path.

Unique ownership does not prove that the pointee's operations are thread-safe.
It may cross a thread boundary only when the instance and its destruction
contract permit that movement.

### Shareable unique ownership

```zax
owner : MyValue * unique shareable
```

`shareable` is unique ownership with a reserved, dormant control block. It
allows the owner to enter local or atomic shared ownership without allocating a
control block at that transition.

Ownership is still unique while this form is active:

- no strong peers exist;
- no weak observers exist;
- reference-count synchronization is unnecessary; and
- `atomic` therefore has no separate unique form.

A blockless unique owner must explicitly allocate a control block before it can
become shareable. That operation may select a control-block arena and may fail
because allocation fails. Exact source syntax remains future allocation work.

The legacy pointer qualifier `own` is superseded by `unique shareable`. Any
future non-pointer use of `own` for composition is a separate concept and does
not revive the old pointer family.

A shareable unique owner may shed its block:

```text
MyValue * unique shareable
-> MyValue * unique
```

Detached block storage can be released. An inline block can be logically
retired, but its coallocated bytes normally remain until the object allocation
ends.

### Strong ownership

```zax
owner : MyValue * strong
```

Every nonempty strong pointer participates in one control block. Copying it
creates another owner:

```zax
another : MyValue * strong = owner
```

The owned allocation remains available while at least one strong owner exists.
When strong ownership closes, weak pointers cannot reopen it.

`strong` uses local shared accounting. Copies must not cross thread boundaries
without first passing through unique ownership.

### Weak observation

```zax
observer : MyValue * weak = owner
```

A weak pointer:

- shares the control block;
- does not increment the strong count;
- does not keep the target ownership lifetime open;
- may survive target termination when its control block remains alive; and
- cannot resurrect ownership after the final strong release.

Weak observation breaks strong-reference cycles when the graph does not require
that edge to own its target.

## Presence and weak acquisition

Every pointer supports `?`, but its guarantee depends on the pointer role:

| Pointer | Meaning of `?pointer` |
| --- | --- |
| `unique` or `unique shareable` | This pointer currently owns a target |
| `strong` | This pointer currently participates in live strong ownership |
| `weak` | At the instant of this non-owning probe, strong ownership remained open |
| Raw | The pointer value is not `Nothing`; no pointee-validity guarantee |

Weak probing does not acquire ownership:

```zax
if ?observer {
  // Strong ownership existed at this instant.
  // observer is still weak and cannot be dereferenced as a strong owner.
}
```

A successful probe may become stale immediately. A failed probe is permanent
for that ownership lifetime because weak ownership cannot resurrect a target.

Actual acquisition uses ordinary destination-directed `copy`:

```zax
owner : MyValue * strong = observer

if ?owner {
  use(owner.)
}
```

The operation:

- leaves `observer` unchanged;
- acquires strong ownership if ownership remains open;
- produces an empty strong pointer on failure; and
- pins the target until `owner` releases its strong participation.

The presence test and strong increment are one indivisible ownership operation
for an atomic weak pointer.

## Pointer-layer `atomic`

```zax
owner : MyValue * strong atomic
observer : MyValue * weak atomic
```

`atomic` qualifies the pointer layer. It makes:

- strong-count changes;
- weak-count changes;
- weak-to-strong acquisition; and
- ownership closure

safe for the declared cross-thread lifetime-accounting contract.

It does not make `MyValue` operations atomic, synchronized, or data-race safe.
It also does not necessarily create an atomic cell whose pointer value may be
concurrently replaced. Atomic pointer-value storage is a separate future
container concern.

Local weak acquires local strong. Atomic weak acquires atomic strong. Changing
between local and atomic shared families requires exclusive ownership through
`unique shareable`.

## Ownership transitions

The destination pointer type and ordinary `copy`, `move`, or `last` stance state
the intended transition.

### Shareable unique to shared

```zax
prepared : MyValue * unique shareable
anotherPrepared : MyValue * unique shareable

localShared : MyValue * strong = prepared as last
atomicShared : MyValue * strong atomic = anotherPrepared as last
```

The unique source becomes vacant. Its reserved block enters the destination's
local or atomic shared mode.

A blockless `unique` cannot use these ordinary transfers because it has no
control block:

```zax
blockless : MyValue * unique
shared : MyValue * strong = blockless as last
// error: allocate shareable control first
```

Control-block allocation is an explicit cost-bearing operation rather than a
hidden side effect of ordinary stance transfer.

### Shared to unique

```zax
plain : MyValue * unique = sharedOwner as last
prepared : MyValue * unique shareable = anotherShared as last
```

The claim succeeds only when:

- exactly one strong owner remains;
- no weak observers remain;
- the pointer targets the control block's allocation root; and
- an atomic source can claim that state atomically.

On success:

- the source becomes vacant;
- a `unique` destination retires the block; and
- a `unique shareable` destination retains the dormant block.

On failure:

- the destination is a valid empty pointer to `Nothing`;
- the source retains its ownership and active block;
- `as last` leaves the source in terminal state; and
- ordinary destruction later releases that retained strong ownership.

The empty pointer is the failure result; no optional wrapper is needed.

### Transfer state and lifetime are independent

An explicit stance can select or forward ownership authority. It does not prove
that a reference, pointer, target place, or arena outlives the destination.
Lifetime and origin analysis still apply.

Complete stance fallback and source state are defined by
[Zax transfer stances](transfer-stances.md).

## Anchored interior pointers

Sometimes code needs an owning pointer to a direct member while keeping the
complete containing allocation alive:

```zax
container : Container * strong = ...

member : Item * strong anchored =
  container.item anchored by container
```

The anchored pointer has:

```text
target place:       container.item
ownership anchor:   container's allocation root
control block:      container's control block
```

Creating it increments the same strong count as `container`. Releasing the last
anchored owner disposes the complete container according to the allocation
contract, because the control block still owns the container root.

### Direct containment only

Ordinary `anchored by` requires a statically recognized direct member. It cannot
cross:

- optional payloads;
- pointer or reference dereferences;
- variant alternatives;
- dynamic array elements;
- unmanaged overlays; or
- separately allocated members.

Those paths may disappear, relocate, or belong to another life path while the
proposed ownership anchor remains alive.

Whether one uninterrupted chain of direct composition may be anchored in one
operation remains deferred. Unsafe ownership anchoring also remains future work.

### Replacement intent

An anchored pointer names the stable direct-member place. Complete replacement
of the container renews that member's resident instance while the allocation and
place continue.

Creating such a pointer causes an intent error when the target or an enclosing
direct place may be replaced:

```zax
// Illustrative category identifier; exact name remains unsettled.
intent<anchored-pointer-tracks-replacement>{
  member : Item * strong anchored =
    container.item anchored by container
}
```

The acknowledgement confirms that later access may observe a completely
established successor member. It is not unsafe permission, and ordinary access
still cannot observe a half-complete transition.

The category identifier shown above is provisional pending the ordinary
intent-category review.

### Anchored weak pointers

```zax
observer : Item * weak anchored = member
restored : Item * strong anchored = observer
```

Weak acquisition reconstructs the same member target and ownership anchor. It
produces an empty anchored strong pointer after ownership closes.

### Anchored pointers are never unique

```zax
owner : Item * unique = member as last
// error: the pointer target is not the control block's allocation root
```

This conversion is statically unavailable rather than conditionally empty.
`Item * unique` would otherwise appear to own an independently destructible
`Item` while its control block actually disposes a `Container`.

## Arenas

An arena supplies storage for dynamic life paths. The process heap is one arena;
custom arenas may choose locality, reuse, bulk recovery, and concurrency
capabilities.

An arena does not by itself keep every allocated instance alive. Owning pointers
and the allocation disposition determine when ownership closes and when
destruction occurs.

An arena is not inherently thread-safe. It may be thread-confined, movable under
exclusive ownership, synchronized for concurrent requests, or able to queue
cross-thread release for processing by an owning thread or executor. The
language tracks the selected capability and must not call the backing arena from
an incompatible thread.

The arena and every metadata region it backs must outlive all resident
instances, retained blocks, and inline control blocks that depend on it.

### Object and control-block arenas

Every dynamic allocation selects an object arena. A managed allocation may:

- coallocate its control block in the object arena; or
- select a separate control-block arena.

An inline block minimizes allocation count but keeps the combined allocation
reserved while weak observers remain. The object arena cannot end while an
external weak pointer still requires that block.

A detached block allows the object arena to end after the target is terminated
and the block records that termination. Weak pointers may remain until the
control-block arena ends. Destroying that arena while such pointers remain
causes the corresponding panic.

The control block stores or reaches the ownership root, destructor, allocation
disposition, arena relationships, counts, and termination state. Exact
representation is not a language promise.

## Allocation disposition

Allocation chooses three related policies.

### Destruction timing

- **Prompt:** destroy the resident instance when ordinary ownership closes.
- **Attached lifespan:** retain the resident instance until arena teardown.

### Storage recovery

- **Prompt recovery:** return the block to the arena for reuse after
  destruction.
- **Arena recovery:** retain the block until arena teardown or reset.

Delayed destruction requires delayed recovery.

### Cycle probing

Shared allocations may opt into explicit detection of unreachable
strong-reference cycles. Eviction closes ownership and applies the allocation's
selected destruction and recovery behavior.

Cycle probing augments reference counting. Acyclic zero-owner allocations still
follow their ordinary prompt or attached policy.

Collectable allocations require graph metadata and root instrumentation. The
compiler emits that support only when reachable build output uses the feature.
Mark state, arena membership, and a type-specific traversal descriptor may live
in the ordinary ownership control block, so collectable pointers need not be
wider than other strong/weak pointers.

The control block alone cannot discover outgoing strong edges. The compiler
must also provide generated traversal or equivalent type metadata.

Memory pressure is a program-visible event rather than a mandatory automatic
collection pause. An arena or platform library may report pressure; the
programmer decides if and when to invoke cycle traversal. Ignoring the signal
leaves later allocation to its ordinary failure contract.

### Common combinations

| Allocation behavior | Destruction | Storage recovery | Cycle probing |
| --- | --- | --- | --- |
| Normal | Prompt | Prompt | Disabled |
| Deferred recovery | Prompt | Arena teardown/reset | Disabled |
| Attached lifespan | Arena teardown | Arena teardown/reset | Disabled |
| Collected normal | Prompt at closure or cycle eviction | Prompt | Enabled |
| Collected deferred recovery | Prompt at closure or cycle eviction | Arena teardown/reset | Enabled |
| Collected attached | Attached policy after closure or eviction | Arena teardown/reset | Enabled |

**Deferred-recovery allocation** is the accepted concept. Legacy material used
`discard`, but no allocation keyword is established. A unique or strong
allocation may both select prompt destruction with arena-delayed recovery.

Exact allocation syntax remains future arena work.

## Allocation failure

The underlying arena request may fail and produce no allocation. The selected
allocation operator determines how source observes that failure.

### Panicking allocation

A panicking allocation either returns a non-`Nothing` pointer or panics. This is
the ordinary default for automatic allocation performed while constructing a
stack, global, or containing resident instance. Failure prevents normal
construction completion.

### Non-panicking allocation

A non-panicking allocation produces a pointer to `Nothing` when the arena cannot
satisfy the request. Automatic pointer members may therefore remain `Nothing`
while the containing instance completes successfully.

This permits deliberately small or exhaustible arenas.

### Unchecked allocation

An unchecked allocation contract may omit the failure check when the programmer
asserts that the request must succeed. If that assertion is false, the
consequences are undefined.

This is unsafe responsibility, not intent acknowledgement. The shape parallels
checked, alternative-result, and unchecked arithmetic; exact allocation
operators remain future syntax.

## Language-managed arena behavior

An arena implementation supplies basic backing-storage behavior:

- capacity or expansion;
- aligned block acquisition and release;
- concurrency and affinity capabilities;
- and memory-pressure information.

The language manages the Zax-specific contents placed in that storage:

- allocation records;
- resident-instance construction and destruction;
- object/control-block relationships;
- shared ownership counts;
- attached-lifespan and deferred-recovery lists;
- collection metadata;
- teardown ordering;
- and diagnostics.

A custom arena therefore need not implement a complex object-by-object lifetime
API. It registers suitable storage operations and capabilities; language-managed
machinery applies Zax lifecycle semantics over them.

## Arena teardown

Arena destruction:

1. closes the arena to new allocations;
2. performs required collector disposition;
3. destroys attached-lifespan instances in the arena's defined order;
4. verifies that no external owner, weak inline block, or other dependency would
   outlive the arena;
5. panics when such a dependency remains; and
6. releases or resets backing storage only after required destruction completes.

Releasing storage while a dependent live instance remains would otherwise have
undefined consequences.

Teardown, final release, or backing-store access on an incompatible thread is
unavailable, must be scheduled through the arena's declared mechanism, or causes
a defined panic when the violation is detected. Instance affinity separately
constrains where its destructor may run.

## Thread transfer

References do not safely cross thread boundaries.

A unique owner may cross by transfer when:

- no other owner exists by definition;
- the pointee's affinity permits movement; and
- final destruction is legal or schedulable on the destination side.

Local shared ownership crosses by using unique as the bridge:

```text
strong
-> conditional last into unique shareable
-> transfer unique ownership
-> strong atomic
```

The first transition requires one strong owner, no weak observers, and an
allocation-root target. Atomic shared ownership may then operate across threads,
but the pointee remains subject to its separate data-race and affinity contract.

A raw pointer may cross when analysis proves its pointee path, arena,
provenance, and thread-affinity requirements remain valid. Otherwise the
boundary requires narrow unsafe responsibility.

## Costs

| Form or policy | Typical cost |
| --- | --- |
| Raw | Address-sized pointer and no ownership work; static proof or unsafe responsibility |
| `unique` | Exclusive ownership and final disposition; no control block |
| `unique shareable` | Reserved inline or detached control block |
| `strong` / `weak` | Local count updates, control-block storage, possible weak-retained block |
| `strong atomic` / `weak atomic` | Synchronized count and acquisition operations |
| Anchored pointer | Target address plus shared ownership anchor; same count cost as its strong/weak family |
| Detached block | Additional allocation and another arena lifetime |
| Attached lifespan | Delayed destruction and retained resources |
| Deferred recovery | Prompt destruction but retained backing storage |
| Cycle probing | Reachable-build metadata, root/edge traversal, and explicitly invoked collection work |

Custom control blocks and arenas may choose different representations. Their
public guarantees, failure modes, synchronization, and costs must remain
visible.

## Diagnostics

Representative diagnostics include:

- blockless `unique` cannot enter shared ownership without allocating a control
  block;
- shared-to-unique claim failed and produced an empty destination;
- anchored pointer cannot become unique;
- `anchored by` target is not a permitted direct member;
- replacement-tracking anchored pointer requires intent acknowledgement;
- local shared ownership cannot cross this thread boundary;
- pointee or destructor is thread-affine;
- object or control-block arena would end while dependent pointers remain;
- weak acquisition failed and produced an empty strong pointer;
- raw pointer was not proved non-`Nothing` before access;
- raw pointer has no proved live pointee, provenance, alignment, or access
  permission;
- raw pointer escaped the container or life path that established its proof;
- panicking allocation could not satisfy its request;
- unchecked allocation guarantee was false;
- arena operation or teardown occurred on an incompatible thread;
- allocation disposition does not provide prompt destruction; and
- arena storage cannot be recovered while a resident instance remains live.

Diagnostics should distinguish:

- ownership role;
- target place;
- ownership anchor;
- object arena;
- control-block arena;
- allocation disposition;
- pointer-layer atomicity; and
- pointee thread safety.

## Source stability

The ownership meaning of a pointer type must not depend on hidden allocation
history.

- `unique` is blockless at the type level even when an inline allocation retains
  physically unrecoverable retired bytes.
- `unique shareable` proves that a suitable dormant block exists.
- `strong` and `weak` prove active or observable shared ownership.
- `anchored` proves that target and ownership root differ.
- pointer-layer `atomic` states the shared accounting contract.

Allocation disposition may remain metadata rather than a pointer qualifier, but
the allocation site and APIs must expose behavior that changes destruction,
recovery, collection, or thread-affinity guarantees.

## Boundaries and maturity

This document is current conceptual design, not a pointer ABI or arena interface
specification.

Still deferred:

- exact allocation and control-block construction syntax;
- custom arena and control-block interfaces;
- deeper or unsafe ownership anchoring;
- pointer representation and tagging;
- complete casts, arithmetic, and provenance;
- pointer-to-`Nothing` dereference behavior;
- cycle-root discovery;
- prompt-disposition generic constraints;
- thread-affinity syntax;
- atomic pointer-value containers; and
- async cancellation and executor behavior.
