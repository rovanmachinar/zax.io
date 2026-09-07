# Zax pointers, allocation, and arenas

| Field | Value |
| --- | --- |
| Status | Current conceptual design |
| Audience | Human developers allocating dynamic storage and choosing ownership, sharing, disposition, and pointer-lifetime behavior |
| Applies To | Declaration-bound allocation, programmer-facing raw and managed pointers, ownership transitions, weak observation, arenas, allocation disposition, control blocks, collection, and cross-thread lifetime accounting; not a formal grammar or ABI |
| Implementation State | Not established by this repository |
| Owns | The `@` allocation family and policy enclosure; declaration-attached and open-ended raw allocation; raw, `unique`, `shareable`, `strong`, `weak`, `anchored`, and pointer-layer `atomic` behavior; arena-backed dynamic allocation; allocation records; destruction/recovery/collection choices; control-block obligations; pointer `reset`; pointer presence and ownership transitions; allocation/pointer costs and diagnostics |
| Does Not Own | General declaration initialization ([declarations and bindings](declarations-and-bindings.md)); constructor and packet behavior ([construction and destruction](construction-and-destruction.md)); complete result mapping ([function invocation](function-invocation.md)); non-owning references and life paths ([lifetimes and references](lifetimes-and-references.md)); complete transfer-stance behavior ([transfer stances](transfer-stances.md)); execution-context construction and replacement ([execution context](execution-context.md)); pointer representation integers ([integers](integers.md)); pointee operation thread safety; arena API/registration; or pointer-cast syntax |
| Source / Provenance | Legacy pointer, memory-allocation, custom-allocator, strong/weak, handle/hint, context, and `Nothing` evidence reconciled with current declaration, transfer, construction, and lifetime design |
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

## Allocate through a declaration

Every direct source allocation is a declaration initializer. The declaration
supplies the resident type and resulting pointer role:

```zax
owner : MyValue * unique = @
shared : MyValue * strong = @
scoped : MyValue * = @
```

`@` requests storage from an arena, constructs one `MyValue`, establishes the
destination pointer's ownership or cleanup relationship, and then publishes the
pointer. Allocation obtains storage; ordinary construction still establishes
the resident instance in that storage.

A pointer declaration without `@` does not allocate:

```zax
empty : MyValue * unique
view : MyValue *
anotherView : MyValue * = view
```

`empty` and `view` contain `Nothing`. `anotherView` copies the raw address-like
value from `view`.

Allocation cannot appear as an independent expression. Assignment requires an
already typed pointer destination:

```zax
ordinary : MyValue *
ordinary = @ // legal open-ended raw allocation

missing = @ // error: no typed destination
```

A declaration expression can instead supply a destination inside a larger
expression:

```zax
ordinary =
  (: MyValue * = @) as last
```

The explicit `last` transfers the temporary declaration's ownership or scheduled
disposition instead of copying only a pointer value.

### Direct allocation assignment

Any typed pointer destination supplies enough pointee and pointer-role context
for direct allocation assignment:

```zax
scheduled : MyValue * = @
reset scheduled
scheduled = @{ anotherArena }

ordinary : MyValue *
ordinary = @
```

The destination contract determines the result:

| Destination | Direct allocation behavior |
| --- | --- |
| Scheduled raw | Reset the old allocation and install a scheduled replacement |
| Open-ended raw | Do not reset; install another open-ended allocation |
| Ordinary undecorated raw | Do not reset or create a schedule; result is open-ended |
| `unique`, `unique shareable`, or `strong` | Release old ownership and install the new managed allocation |

`ordinary = @` is the direct counterpart of allocating through a typed
declaration expression and terminally mapping into an undecorated raw
destination. It is legal, but the resulting allocation must be manually tracked
and reset.

For an open-ended destination, allocation assignment never performs an implicit
reset:

```zax
open : MyValue * = @<
open = @<{ anotherArena }
// error when the prior open allocation is still live
```

This is safe when analysis proves `open` is already `Nothing` or its previous
allocation was dispositioned. Overwriting a proved live open allocation is a
non-acknowledgeable lifecycle error. When opaque code prevents proof of prior
disposition, narrow unsafe responsibility—not intent acknowledgement—must supply
the missing fact.

The ordinary repair is explicit:

```zax
reset open
open = @<{ anotherArena }
```

The replacement token must preserve the declaration's scheduled/open contract.
Using `@<` to refill a scheduled slot or plain `@` to refill an open-ended slot
is a non-acknowledgeable conflict.

### Constructor inputs

The existing construction packet follows the allocation form:

```zax
owner : MyValue * unique =
  @ [{
    endpoint
  }]
```

The packet retains its ordinary constructor meaning. Allocation policy does not
share that packet's namespace.

Allocation first obtains all required object and control-block storage. Only
then do packet entries evaluate from left to right and ordinary construction
begin. A storage failure therefore does not evaluate constructor inputs or run
the constructor.

### Allocation forms

The exact reserved forms are:

| Form | Storage-request failure | Raw result |
| --- | --- | --- |
| `@` | Panic | Destination declaration schedules disposition |
| `@!` | Return `Nothing` | Destination declaration schedules disposition after success |
| `@<` | Panic | Open-ended; no automatic disposition owner |
| `@!<` | Return `Nothing` | Open-ended after success |

These are indivisible tokens, not compositions of independently invoked `@`,
`!`, and `<` operators. `@!` reports request failure through the pointer's
ordinary `Nothing` state; plain `@` still checks failure and panics.

`@<` and `@!<` apply to raw pointer destinations. They do not make sense for
`unique` or shared destinations whose pointer role already owns disposition.
Such a combination is a non-acknowledgeable intent error:

```zax
invalid : MyValue * unique = @<
// error: unique ownership and open-ended disposition conflict
```

There is no separate unchecked allocation form. A future safety control may
disable the allocation-failure panic category under a promise that the condition
cannot occur. That general panic control does not change allocation syntax, does
not affect `@!`, and gives undefined behavior if the promised-impossible failure
actually occurs.

### Default allocation

The short forms deliberately choose useful defaults:

```zax
prepared : MyValue * unique shareable = @
preparedInArena : MyValue * unique shareable = @{ myArena }
```

| Concern | Default |
| --- | --- |
| Object arena | Current thread context's default object arena |
| Control block for `unique shareable` or `strong` | Inline with the object allocation |
| Disposition | `AllocationDisposition.Prompt` |
| Collection | `AllocationCollection.Disabled` |
| Construction | Zero-input constructor |

The current thread's `___` execution context supplies omitted object and
detached-control-block arenas. An explicit object arena replaces only that
selection:

```zax
owner : MyValue * unique = @{ myArena }
```

The [execution-context owner](execution-context.md) defines the general `___`
model. An enclosing allocation's arena does not automatically flow into nested
allocations.

### Allocation policy enclosure

`@{ ... }` carries arena and allocation-policy inputs. The opening brace attaches
directly to the allocation token:

```zax
valid : MyValue * unique = @{ myArena }
invalid : MyValue * unique = @ { myArena }
// error: the policy enclosure must attach to @
```

The enclosure permits at most one positional input, the object arena. Remaining
inputs are named:

```zax
owner : MyValue * strong =
  @{
    arena: myObjectArena,
    controlArena: myControlArena,
    disposition: AllocationDisposition.DeferredRecovery,
    collection: AllocationCollection.Collectable
  } [{
    endpoint
  }]
```

Supplying `controlArena:` requests a detached control block. To request a
detached block from the context's default control-block arena:

```zax
owner : MyValue * strong =
  @{ control: ControlBlockPlacement.Detached }
```

The aligned policy values are:

```text
AllocationDisposition.Prompt
AllocationDisposition.DeferredRecovery
AllocationDisposition.AttachedLifespan

AllocationCollection.Disabled
AllocationCollection.Collectable

ControlBlockPlacement.Inline
ControlBlockPlacement.Detached
```

An empty enclosure has no accepted form; use `@`:

```zax
first : MyValue * unique = @{}  // error
second : MyValue * unique = @{ } // error
```

Two positional arenas and duplicate named inputs are likewise errors:

```zax
invalid : MyValue * strong =
  @{ myObjectArena, myControlArena }
// error: name controlArena
```

Explicit `Detached` plus `controlArena:` has one defined meaning but redundantly
states placement. It requires an applicable intent acknowledgement. `Inline`
plus `controlArena:` is contradictory and cannot be acknowledged into validity.

### Allocation order and failure

The programmer-visible order is:

1. Resolve the resident type, pointer role, policies, and required arena
   capabilities.
2. Evaluate written policy inputs once in source order.
3. Resolve omitted arenas through the current `___`.
4. Request object storage.
5. Request detached control-block storage when needed.
6. After every required request succeeds, evaluate construction-packet entries
   from left to right.
7. Run ordinary member construction and the selected constructor.
8. Establish ownership or declaration-attached disposition.
9. Publish the complete pointer.

For `@!` and `@!<`, failure of a later storage request releases every earlier
uncommitted reservation exactly once before producing `Nothing`.

For `@` and `@<`, a failed request enters panic while the operation remains
blocked. A narrowly applicable helper may repair the condition and allow that
same request to succeed. Otherwise the program crashes gracefully. Panic does
not return a substitute result, skip the failed operation, continue with partial
state, or introduce exception-style unwinding.

`@!` controls allocation-request failure only. It does not translate a
constructor or nested-operation panic into `Nothing`.

### Adopt an existing allocation

A trailing source expression changes the operation from new-object construction
to existing-allocation adoption:

```zax
originalValue : MyValue * unique = @

prepared : MyValue * unique shareable =
  @ originalValue as last
```

The existing resident instance is preserved. `@` allocates only metadata required
by the destination role:

- a proved raw allocation may become blockless `unique` without `@`;
- raw or blockless `unique` may use `@` to allocate a detached dormant block for
  `unique shareable`;
- raw or blockless `unique` may use `@` to allocate and activate a detached block
  for `strong`; and
- a source that already satisfies the destination role uses ordinary transfer
  without redundant `@`.

A construction packet is unavailable in this form because no new resident
instance is constructed.

A function result may supply the source directly:

```zax
prepared : MyValue * unique shareable =
  @ makeScheduledRawValue()
```

The producer's declared result stance and disposition authority—not its name—
determine whether adoption is safe.

When the source is runtime `Nothing`, both `@` and `@!` produce a `Nothing`
destination without requesting metadata or entering panic. Applying this
operation to a source statically proved `Nothing` is a non-acknowledgeable intent
error because no work can occur.

For `@!`, metadata-allocation failure leaves the destination at `Nothing`, keeps
the source's original authority unchanged, and releases partial metadata. `@`
blocks in panic until the failure is repaired or the program crashes.

## Pointer instances and pointees

A pointer is an ordinary instance with its own life path. Its value identifies
another instance place or `Nothing`.

The pointer place and pointee place are distinct:

```zax
pointer : MyValue * = first
pointer = second // ordinary unscheduled raw assignment only repoints
```

Replacing or destroying a pointer instance affects the pointee only when that
pointer's ownership contract or declaration-attached allocation requires it.

Pointer-value transfer never implies pointee transfer by itself. A raw pointer
copy or move ordinarily transfers only an address-like value. A `unique` owner
transfers its owned life path because `unique` explicitly says it does. A
scheduled raw declaration may transfer its separate disposition responsibility
through an accepted `last` operation.

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

#### Scheduled raw allocations

For a raw destination, `@` or `@!` attaches successful allocation disposition to
the destination declaration's life path:

```zax
scoped : MyValue * = @
maybeScoped : MyValue * = @!
```

The pointer remains raw. The declaration independently schedules disposition
when its path ends or when the slot replaces the allocation:

```zax
alias : MyValue * = scoped
```

`alias` copies only the address. It does not copy the schedule and cannot safely
outlive the allocation path owned by `scoped`.

Assignment to the scheduled slot dispositions its old allocation and then adopts
the new source's disposition authority:

```zax
scoped = uniqueOwner as last
scoped = otherScheduledRaw as last
```

The source becomes `Nothing`. Its own scheduled cleanup remains in place but
later observes `Nothing` and performs no disposition.

An ordinary borrowed raw source is adoptable only when analysis proves that it
identifies an allocation root and carries the sole disposition authority.
Otherwise the adoption requires narrow unsafe responsibility. A known interior,
ended, or independently owned target is rejected.

Assigning `Nothing` dispositions a scheduled slot's current allocation before
emptying it:

```zax
scoped = (: MyValue *)
```

#### Open-ended raw allocations

`@<` and `@!<` deliberately create no declaration-attached cleanup owner:

```zax
manual : MyValue * = @<
maybeManual : MyValue * = @!<
```

They are safe only when analysis proves another lifetime/disposition owner or a
complete manual protocol. Otherwise creation or escape requires narrow unsafe
responsibility.

Assigning `Nothing` to an open-ended raw pointer discards only the address:

```zax
manual = (: MyValue *)
```

It does not disposition the allocation.

#### Returning a scheduled raw allocation

A callable that allocates a raw result should declare terminal transfer in its
prototype:

```zax
makeValue final : (
  result : MyValue * last = @
)() = {
  // result is allocated and scheduled before body entry.
}
```

`= @` makes the allocation and schedule part of the visible result contract.
`last` transfers that responsibility to the caller destination:

```zax
value : MyValue * = makeValue()
```

The producer result pointer becomes `Nothing`; its scheduled cleanup remains but
is a no-op. If the result is discarded, its own slot retains responsibility and
dispositions the allocation.

`@!` produces the same contract with a possible `Nothing` result:

```zax
tryMakeValue final : (
  result : MyValue * last = @!
)() = {
}
```

An omitted stance defaults to `copy` and requires
`intent<implicit-stance-at-terminal-use>` acknowledgement because `last` would
normally transfer the schedule. Explicit `copy` is legal and deliberately
exposes a borrowed result: callers may use it only within the producer result
slot's lifetime unless they restate `last` or prove another owner.

Explicit raw `move` likewise does not transfer declaration-attached disposition
and remains subject to the borrowed lifetime boundary. `deep` is available only
when an exact deep-capable consumer exists; it never implies built-in pointee
cloning.

For results, `@` and `@<` promise presence on every normal exit; `@!` and `@!<`
permit `Nothing`. A compatible visible prototype may change scheduled versus
open-ended outward cleanup without reminting the implementation body, but it
cannot change definite versus maybe-present guarantees.

Complete result construction and mapping are defined by
[Zax function invocation](function-invocation.md#opt-in-result-initialization).

#### Allocation roots and records

An **allocation root** is the top-level life path and typed instance place
created directly by one dynamic allocation operation. Its allocation record
governs the complete allocated storage and final disposition.

```zax
container : Container * unique = @
member : Item * = pointerTo(container.items[0])
```

`container` targets the allocation root. `member` may target bytes and a live
place inside that allocation, but the element is not an independently
dispositionable allocation root.

The distinction determines:

- which complete destructor runs;
- which object arena receives storage recovery;
- which allocation record applies;
- whether a pointer may become `unique`;
- whether raw `reset` may disposition the allocation; and
- what an anchored pointer keeps alive.

An array's complete allocated array place is its root, not any individual
element. The allocation root is a language-level place and path, not a promise
that its address is the first physical byte.

Every dynamic allocation retains or can recover an allocation record containing:

- its allocation root;
- object arena;
- destructor;
- size and alignment;
- disposition;
- and any link to additional metadata.

The record may be represented before the resident storage, through arena
metadata, or another implementation-defined mechanism. Its physical layout is
not a language or ABI promise.

An allocation record is not an ownership control block. Raw and blockless
`unique` allocations have the former without the latter. Ownership control
blocks add dormant shareability, strong and weak accounting, local or atomic
mode, termination state, and collection metadata.

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
become shareable:

```zax
originalValue : MyValue * unique = @

prepared : MyValue * unique shareable =
  @!{
    controlArena: myControlArena
  } originalValue as last
```

Because the existing object has no reserved inline block, this operation
allocates a detached block. `@` blocks in panic on allocation failure;
`@!` returns an empty `prepared` pointer.

The operation is transactional. On success, `owner` becomes `Nothing` and
`prepared` owns the same resident instance through a dormant control block. On
`@!` failure, `prepared` is `Nothing`, `owner` remains an unchanged usable
unique owner, and any partial block reservation is released.

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

## Resetting a pointer

Protected pre-unary `reset` ends the relationship for which a pointer or its
declaration has authority, then leaves the pointer containing `Nothing`:

```zax
reset pointer
```

| Pointer | Effect |
| --- | --- |
| Scheduled raw | Apply the allocation's recorded disposition |
| Open-ended raw | Explicitly disposition the allocation; requires allocation-root and authority proof |
| Borrowed raw | Unavailable unless analysis proves complete disposition authority |
| `unique` | Release unique ownership |
| `unique shareable` | Release unique ownership and its dormant block |
| `strong` | Release this strong participation; destroy only when ownership closes |
| `weak` | Release this weak observation and any final retained block |
| Already `Nothing` | No-op |

An anchored strong or weak pointer follows its shared ownership role: reset
releases participation in the enclosing allocation's control block and leaves
the anchored pointer at `Nothing`. It does not destroy the targeted member as an
independent allocation.

For raw pointers, safe reset requires proof that the pointer identifies the
allocation root and that no competing authority will disposition it. An opaque
but potentially valid relationship may use narrow unsafe responsibility. A
proved interior pointer, ended allocation, stack/global place, or competing
owner is known-invalid.

Reset applies the disposition recorded at allocation:

- `Prompt` destroys and recovers promptly;
- `DeferredRecovery` destroys promptly while the arena retains storage; and
- `AttachedLifespan` closes the pointer's local responsibility while destruction
  and recovery remain attached to arena teardown.

Resetting `Nothing` is a harmless no-op because this protected operation examines
the pointer state rather than invoking a member through its pointee.

Assignment to `Nothing` remains ordinary destination behavior. It releases a
scheduled or managed destination's current relationship, but for an open-ended
raw pointer it discards only the address:

```zax
manual : MyValue * = @<
manual = (: MyValue *) // does not disposition the allocation
```

Resetting the same pointer again is harmless after the first reset leaves
`Nothing`. A disposition attempt through a stale alias is rejected when analysis
proves that the allocation already ended; an unproved alias relationship
requires narrow unsafe responsibility.

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

### Scheduled raw and managed ownership

`last` may transfer declaration-attached raw disposition into managed ownership:

```zax
scheduled : MyValue * = @
owner : MyValue * unique = scheduled as last
```

On success, `scheduled` becomes `Nothing`; its cleanup remains scheduled but
later performs no disposition. `owner` becomes the allocation's unique owner.

A scheduled raw destination may likewise adopt `unique` ownership or another
scheduled raw allocation:

```zax
scheduled = uniqueOwner as last
scheduled = otherScheduled as last
```

The destination first dispositions its old allocation, then adopts the new
allocation. The accepted source becomes `Nothing`.

Transferring either authority into an ordinary unscheduled raw destination makes
the allocation open-ended:

```zax
raw : MyValue * = uniqueOwner as last
// requires another proved disposition owner or narrow unsafe responsibility
```

An ordinary borrowed raw source can enter a scheduled or managed destination
only when analysis proves its allocation root, provenance, and sole disposition
authority. A known-invalid adoption is rejected.

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

An allocation with no explicit object arena uses the current thread execution
context's default object arena. A requested detached block with no explicit
control-block arena uses that context's default control-block arena. Each
allocation reads the applicable context when its own `@` operation executes; an
outer allocation's arena does not flow implicitly into contained allocations.

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

Allocation independently chooses one disposition and whether the allocation may
participate in cycle collection.

### Disposition

| Value | Destruction | Storage recovery |
| --- | --- | --- |
| `AllocationDisposition.Prompt` | When ordinary ownership or scheduled disposition closes | Prompt |
| `AllocationDisposition.DeferredRecovery` | Prompt | Arena teardown/reset |
| `AllocationDisposition.AttachedLifespan` | Arena teardown | Arena teardown/reset |

These are the complete valid combinations. Attached destruction cannot use
prompt storage recovery because storage cannot be recovered while its resident
instance remains live.

### Cycle probing

Shared allocations may select `AllocationCollection.Collectable` to opt into
explicit detection of unreachable strong-reference cycles. Omission selects
`AllocationCollection.Disabled`. Eviction closes ownership and applies each
allocation's selected disposition.

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
collection pause. An arena or platform library may report global pressure; the
programmer decides if and when to invoke one process-wide cycle traversal.
Ignoring the signal leaves later allocation to its ordinary failure contract.

The collection graph is process-wide rather than partitioned by object or
control-block arena. A cycle may span several arenas. Exact root discovery,
cross-thread coordination, traversal, eviction ordering, and trigger spelling
remain future collector work.

`unique` does not participate because no strong ownership graph exists. A
`unique shareable` allocation may select `Collectable`; its eligibility and
metadata remain dormant until it enters `strong` or `strong atomic` ownership.

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
`discard`, which is superseded. A unique or strong allocation may both select
prompt destruction with arena-delayed recovery.

## Allocation failure

The underlying arena request may fail and produce no allocation. `@` and `@!`
determine how source observes that failure.

### Panicking allocation

A `@` allocation either returns a non-`Nothing` pointer or enters panic. This is
the ordinary default for automatic allocation performed while constructing a
stack, global, or containing resident instance. Failure prevents normal
construction completion.

The operation remains blocked at the failure condition. A narrowly applicable
panic helper may repair that condition and allow the same request to succeed as
though it had not failed. Otherwise the program crashes gracefully. It does not
continue with partial allocation or construction state.

### Non-panicking allocation

A `@!` allocation produces a pointer to `Nothing` when the arena cannot
satisfy the request. Automatic pointer members may therefore remain `Nothing`
while the containing instance completes successfully.

This permits deliberately small or exhaustible arenas.

There is no independent unchecked allocation mode. A future general safety
control may disable the allocation-failure panic category while leaving other
panic categories enabled. That promise may permit omission of the corresponding
check and gives undefined behavior if failure actually occurs. It does not
change `@!`, whose defined result requires detecting failure.

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
2. coordinates any required process-wide collector disposition for dependencies
   involving the arena;
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
| Borrowed raw | Address-sized pointer and no ownership work; static proof or unsafe responsibility |
| Scheduled raw | Address-sized pointer plus declaration-attached disposition tracking and allocation-record access |
| Open-ended raw | Address-sized pointer; programmer/proved manual disposition responsibility |
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
- open-ended raw allocation lost its last usable address;
- raw reset target is not a proved allocation root;
- raw allocation was dispositioned through a stale alias;
- scheduled raw adoption lacks sole disposition authority;
- scheduled raw result used `copy` where its cleanup must transfer;
- panicking allocation could not satisfy its request;
- disabled allocation-failure panic occurred despite its promised-impossible
  contract;
- allocation policy enclosure is empty, has two positional arenas, or duplicates
  one named slot;
- detached placement redundantly accompanies `controlArena:` without an intent
  acknowledgement;
- inline placement contradicts `controlArena:`;
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
- collection eligibility;
- contextual, defaulted, or explicit policy origin;
- declaration-attached or open-ended raw state;
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

Adding, removing, or changing an allocation token, policy input, default, or
intent requirement is a source-compatibility event. Source reflection and
tooling must preserve whether each effective choice was explicit, supplied by
the current context, implied by the destination pointer role, or defaulted.

## Boundaries and maturity

This document is current conceptual design, not a pointer ABI or arena interface
specification.

Still deferred:

- custom arena and control-block interfaces;
- custom control-block implementations;
- deeper or unsafe ownership anchoring;
- pointer representation and tagging;
- complete casts, arithmetic, and provenance;
- pointer-to-`Nothing` dereference behavior;
- exact process-wide collection trigger spelling, cycle-root discovery,
  traversal, and concurrent coordination;
- prompt-disposition generic constraints;
- exact category-specific panic-control syntax;
- thread-affinity syntax;
- atomic pointer-value containers; and
- async cancellation and executor behavior.
