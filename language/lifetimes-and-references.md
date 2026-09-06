# Zax lifetimes and references

| Field | Value |
| --- | --- |
| Status | Current conceptual design |
| Audience | Human developers reasoning about where Zax instances live and when non-owning access remains valid |
| Applies To | Programmer-facing instance lifetimes, life paths, instance places, references, reference origin, escape, and synchronous borrowing; not a formal specification |
| Implementation State | Not established by this repository |
| Owns | Life paths; instance places and resident instances; reference binding and origin; references across mutation and replacement; member and nested-place consequences; synchronous parameter and temporary borrowing; returned references; reference capture and storage; reference-facing diagnostics, costs, and unsafe boundaries |
| Does Not Own | How construction and destruction perform lifecycle transitions ([construction and destruction](construction-and-destruction.md)); complete qualifier meaning ([qualifiers](qualifiers.md)); pointer ownership, arenas, and allocation disposition ([pointers and arenas](pointers-and-arenas.md)); transfer stances ([transfer stances](transfer-stances.md)); or general safety-contract behavior ([safety and analysis](safety-and-analysis.md)) |
| Source / Provenance | Legacy pointer, function-capture, scope, construction, and global-lifecycle evidence reconciled with current qualifier, invocation, optional, identity, and transfer design |
| Supersedes | Reference and lifetime teaching formerly distributed through root legacy pages |

## Start with a life path

Every Zax instance is created, lives, and is destroyed within a **life path**.
An instance never outlives its life path.

```zax
process final : ()() = {
  first : Document

  scope {
    second : Document
    inspect(first)
    inspect(second)
  } // second is destroyed as its inner life path ends
} // first is destroyed as the invocation life path ends
```

Outside an unsafe contract violation:

- each instance is constructed once within its life path;
- each instance is destroyed once within that path;
- ordinary construction follows the path's defined order;
- ordinary destruction follows the corresponding reverse order; and
- no reference, pointer, transfer, or allocation makes an instance survive the
  path that owns it.

Some operations create a separate nested life path and transfer or share
ownership of that path. The instance does not escape its path; the path itself
continues because another owner keeps it alive.

This distinction is the foundation for Zax lifetime reasoning.

## Life path, instance place, and resident instance

An **instance place** is the stable, typed position within a life path through
which a live instance is reached. After introducing the complete term, this
document also calls it a **place**.

The **resident instance** is the completely constructed instance currently
available through that place.

```text
allocation or enclosing storage
-> life path
-> instance place
-> resident instance
```

These are language-level distinctions, not promises of separate runtime
objects. A place does not inherently require a wrapper allocation, hidden state,
or extra indirection.

Allocation provides physical storage. It may exist before construction or
remain after destruction. Stable bytes or an unchanged address do not by
themselves prove that a life path, place, or resident instance remains valid.

### Final and varying places

A final place hosts one resident-instance lifetime:

```zax
document final : Document immutable final = makeDocument("one")
```

A varying place may host successive resident instances without the place itself
ending:

```zax
document varying :
  Document immutable writable varying = makeDocument("one")

document = makeDocument("two")
```

The first immutable `Document` is destroyed and a successor is constructed in
the continuing `document` place. Ordinary access is unavailable while a
lifecycle operation has left the place without a complete resident instance.

Type-side `final` or `varying` describes this underlying place truth.
Declaration-side `final` or `varying` independently controls whether one access
path may initiate replacement. See [Zax qualifiers](qualifiers.md).

`resident instance` is distinct from Zax's established term **current
instance**, which means the instance whose type-defined body is executing
through `_`.

## Common life paths

The same model applies across Zax:

| Source | Life-path behavior |
| --- | --- |
| Global execution | Global declarations live within the process path and are destroyed in reverse of their established construction order; exact cross-module, dependency, and `once` ordering remains future work |
| Namespace | A namespace introduces a path within global execution and ends before the containing unnamed global path |
| Flow scope | Blocks, clauses, loops, and explicit `scope` create bounded paths destroyed when control leaves them |
| Type instance | Composition creates contained member places and nested paths bounded by the containing instance |
| Function invocation | Parameters, result slots, and invocation temporaries live through the complete synchronous call and result-mapping boundary |
| Lambda capture | A capturing lambda owns a capture path tied to the lambda instance |
| Expression temporary | A temporary path survives through the complete use or transfer that required it |
| Optional | A present optional owns one nested boxed path; reset ends that path |
| Managed variant | Exactly one active alternative path is resident; changing alternatives ends one path and begins another |
| Array or collection | A collection owns one or more element paths whose stability depends on its operation contract |
| Dynamic allocation | An arena supplies storage for a separately owned path |

The details of pointer-owned dynamic paths are in
[Zax pointers and arenas](pointers-and-arenas.md).

## References borrow one fixed place

A reference is a cheap, non-owning access path to an existing instance place:

```zax
source : Document
view : Document readonly & = source
```

The reference instance has its own life path. Creating or destroying it does not
create, destroy, or own `source`.

### References never rebind

A reference binds to exactly one instance place when constructed. That target
never changes during the reference's lifetime.

```text
reference target place       = permanently fixed
resident instance in place   = may change when the place is varying
```

No safe operation, unsafe assertion, transfer stance, cast, capture, parameter
mapping, or result mapping can retarget an existing reference.

Constructing another reference is not rebinding:

```zax
source : Document
first : Document & = source
second : Document & = first
```

`first` and `second` are separate reference instances bound to the same place.

Assignment through a reference acts on its referent:

```zax
source : Document
view : Document writable & = source

view = replacement
```

It does not assign another target into `view`. A pointer is different: its
pointer value may be replaced so that it points elsewhere or to `Nothing`.

### Non-null does not mean owning

A live reference always has a referent. It cannot point to `Nothing`, but it does
not keep the referent's life path alive.

Safe use requires proof that:

- the referenced place still exists;
- the place contains a complete resident instance;
- no forbidden lifecycle transition overlaps the access; and
- the access path retains the required qualifications and permissions.

When the compiler cannot prove a genuinely valid relationship, a narrow unsafe
assertion may accept responsibility. Unsafe source cannot create ownership,
extend a path, rebind a reference, or make a known-absent instance exist.

## Mutation and replacement

Content mutation does not end the resident instance:

```zax
record : MyRecord mutable
view : MyRecord readonly & = record

record.count += 1
inspect(view) // same resident instance, changed contents
```

`readonly` prevents change through `view`; it does not promise that another path
will never change what `view` observes.

### A reference to a varying place observes its successor

```zax
value varying :
  MyValue immutable writable varying = makeValue("first")

view final :
  MyValue immutable readonly varying & = value

value = makeValue("second")
inspect(view) // observes the completely established successor
```

`view` remains bound to `value`'s stable place. It does not rebind when
replacement renews the resident instance.

Ordinary access cannot observe the place while replacement is dismantling the
old instance or establishing the successor. Constructors, replacement
constructors, and destructors receive only the lifecycle authority appropriate
to their tracked partial state.

### Direct-member references cross a renewal boundary

A direct member has its own place:

```zax
record : MyRecord
countView : Integer readonly & = record.count

record.count = 5
inspect(countView) // same member lifetime
```

Complete replacement of `record` renews every member resident instance:

```zax
record varying : MyRecord immutable writable varying
countView : Integer readonly & = record.count

record = makeReplacement()
inspect(countView) // error: the referenced member was renewed
```

This remains a safe-language error even when:

- the member has a trivial representation;
- the address did not change; and
- low-level access to the successor would be technically possible.

A narrow unsafe permission may deliberately use the fixed member-place
reference to reach a completely established successor. It accepts the logical
renewal boundary; it does not claim that the old member instance survived.

Complete replacement and member renewal are defined by
[Zax construction, replacement, and destruction](construction-and-destruction.md).

## Nested places

Reference validity follows the exact target place, not merely its containing
allocation.

### Optional payloads

```zax
wrapper : MyValue?
wrapperView : MyValue? readonly & = wrapper
```

`wrapperView` may remain valid while the optional wrapper remains alive.

A reference reached through a present payload instead names the payload place.
Reset ends that boxed path:

```zax
if ?wrapper {
  payloadView : MyValue readonly & = wrapper.
  reset wrapper
  inspect(payloadView) // error: the boxed path ended
}
```

Later presence creates a new boxed path. The old reference does not retarget to
it.

An optional reference stores a conditional reference instance:

```zax
view : MyValue readonly & ?
```

Reset destroys the stored reference, not the referent. Later presence may
construct a new reference bound to a different place; that is a new reference
lifetime rather than rebinding.

### Arrays and collections

A fixed-shape array keeps its element places stable for the array's life path.
A dynamic collection's operation contract must say which element places remain:

- an in-capacity operation may preserve existing elements;
- contiguous growth may construct new elements, transfer contents, and destroy
  every old element;
- a segmented structure may relocate only affected segments; and
- removal ends the removed element path.

Reference stability is programmer-visible behavior, not an invisible
implementation choice.

Complete dynamic-array operation contracts remain future array work.

### Pointer members and pointees

These references have different origins:

```zax
pointerView : MyValue * readonly & = owner.pointer
pointeeView : MyValue readonly & = owner.pointer.
```

`pointerView` targets the pointer member's place. Repointing that member changes
the pointer value but does not retarget `pointerView`.

`pointeeView` targets the pointee's place. Repointing `owner.pointer` does not
retarget `pointeeView` and does not prove that its old pointee remains alive.

An [anchored owning pointer](pointers-and-arenas.md#anchored-interior-pointers)
is a distinct mechanism that keeps an enclosing allocation alive while
targeting a direct member place.

### Same-storage identity views

A same-storage identity reference creates no independent identity instance and
schedules no separate destruction. It forwards the underlying target place,
origin, and permitted access while applying the identity's nominal rules. See
[Zax identity types](identity-types.md#construction-and-transfer).

## Function parameters and temporaries

A reference parameter constructs a local reference bound to caller-owned
storage:

```zax
inspect final : ()(
  input : Document readonly &
) = {
}
```

The callee receives no ownership. The caller preserves the target through the
complete synchronous call.

Explicit arguments evaluate in caller source order and bind immediately. Later
argument evaluation may mutate a referent already bound to an earlier
parameter:

```zax
observe(
  source,
  action: changeSource(source)
)
```

The first parameter is already bound when `changeSource` runs.

### Temporary extension is synchronous

```zax
inspect(makeDocument())
```

When `inspect` accepts `Document readonly &`, the temporary `Document` survives
through body execution and result mapping. It is destroyed only after its
complete synchronous consumer finishes.

Nested calls preserve the same boundary:

```zax
inspect(returnView(makeDocument()))
```

This may be valid when origin analysis proves that the reference is used only
through the complete outer call.

Suspension is not synchronous completion. Reference validity across async
suspension, cancellation, or executor movement remains future async design.

## Returned references

A reference result owns only an access path. Result-slot destruction does not
make the referent terminal and does not extend its life path.

The compiler tracks the returned target place and its origin:

```zax
returnInput final : (
  result : MyValue readonly &
)(
  input : MyValue readonly &
) = {
  return input
}
```

The result remains tied to `input`'s target.

Commonly provable origins include:

- the current instance `_`;
- an input parameter's referent;
- a direct projection from one of those places; and
- a global place.

A local cannot escape its invocation path:

```zax
badView final : (
  result : MyValue readonly &
)() = {
  local : MyValue
  return local // error: local is destroyed when this call completes
}
```

Temporary extension likewise does not justify storing a returned reference:

```zax
view : MyValue readonly & = returnInput(makeValue())
// error: the reference would outlive the temporary target
```

Zax does not currently require programmer-written lifetime parameters. Concrete
bodies use static origin analysis. Opaque bodies, FFI, and separately expressed
callable contracts may eventually need inferred metadata or conservative origin
contracts, but that future need does not make an unproved result valid.

## Capture and stored references

Lambda capture defaults to `copy`, even when the captured name is a reference:

```zax
source : Document
view : Document readonly & = source

callback := [view] {
  inspect(view) // observes a Document copied into the capture path
}
```

The lambda owns that copied `Document` in its capture path. If the referent is
not copyable, ordinary capture is unavailable.

An explicit reference capture instead constructs another reference bound to the
original place. It:

- does not extend or own the target path;
- must be proved valid for every invocation;
- may not silently inherit destructive `move` or `last` stance; and
- defaults ordinary nested use to `copy`.

Exact reference-capture syntax and callable representation remain future
callable work.

Storing a reference in a field, optional, callback, or result never resets its
origin. The destination must not outlive the target path unless a separate owner
keeps that path alive.

## `move`, `last`, and reference lifetime

Transfer stance and lifetime validity are independent.

A reference-shaped `move` or `last` parameter grants the selected synchronous
consumer authority over caller-owned storage:

```zax
consume final : ()(
  input : Resource mutable writable & last
) = {
  inspect(input) // named input ordinarily offers copy
  transfer(input.handle as last)
}
```

`move` and `last` change resource authority and source state. They do not rebind
the reference or by themselves end the outer place.

Explicit renewal may create another terminal-stanced reference:

```zax
stored :
  Resource mutable writable & last = (input as last)
```

That declaration is viable only when independent lifetime analysis also proves
the target outlives `stored`. Named use of `stored` again offers `copy` unless a
later consumer explicitly renews `as last`.

The legacy `lease` concept has no remaining language role. Fixed reference
binding, synchronous borrowing, pointer ownership, copy-default named use, and
explicit destructive renewal cover its useful behavior.

## Thread boundary

Ordinary references do not safely cross thread boundaries. A reference does not
own or synchronize the target path, and the resident instance may be destroyed
or changed on another thread.

Raw pointers may cross when the compiler proves the pointee path, arena,
provenance, and affinity remain valid; otherwise the boundary requires unsafe
programmer responsibility. Managed cross-thread ownership uses the applicable
unique or atomic pointer contract; that still does not make the pointee's
operations data-race safe.

## Costs and guarantees

An ordinary proved reference requires:

- no ownership transfer;
- no required control block;
- no required allocation;
- no required runtime lifetime check; and
- no rebindable handle behavior.

Those low costs depend on static restrictions. A program needing dynamic
ownership, weak observation, cross-thread lifetime accounting, or
allocation-backed continuation selects an explicit pointer strategy and pays its
visible costs.

Compiler debugging modes may instrument unsafe claims or lifecycle state. Such
checks are tooling, not language guarantees.

## Diagnostics

Diagnostics should identify:

- the reference declaration or forwarding boundary;
- the target instance place and originating life path;
- the destruction, replacement, reset, relocation, or escape that invalidates
  the use;
- the later conflicting use;
- any permission lost through an alias, parameter, result, or capture; and
- whether the operation is unproved, a defined unsafe boundary, or impossible.

Representative errors include:

- reference may outlive its target path;
- returned reference targets a local or temporary;
- member reference crosses complete replacement;
- payload reference is used after reset;
- ordinary access may observe an incomplete resident instance;
- reference capture may outlive its target;
- reference cannot be rebound; and
- lifetime assertion contradicts a known-ended path.

Technically valid but surprising observation through another authorized mutable
path may be a lint rather than a safety error.

## Source stability

Reference meaning does not depend on one compiler's current cleverness.

- The selected language contract determines which cases every conforming
  compiler must prove.
- A newer compiler may prove additional cases under an older contract.
- Removing an assertion made redundant only by that compiler may reduce source
  portability.
- A newer selected contract may require the proof and reject the obsolete
  assertion.

The general rule belongs to [Zax safety and analysis](safety-and-analysis.md).

Improved analysis may accept more valid source. It must not silently:

- rebind a reference;
- add ownership;
- change transfer stance;
- extend a life path;
- turn a direct member into an allocation root; or
- change which resident instance a fixed place exposes.

## Boundaries and maturity

This document is current conceptual design, not a formal lifetime calculus or
compiler algorithm.

Still deferred:

- exact compiler proof algorithms;
- opaque callable origin metadata;
- exact reference-capture syntax;
- dynamic-array stability contracts;
- variant and unmanaged-union design;
- async suspension and cancellation;
- formal grammar, ABI, layout, and lowering.

Those deferrals do not weaken the fixed reference-binding model or the
programmer-visible life-path foundation defined here.
