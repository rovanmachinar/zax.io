# Raw input: replacement construction

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | A future numbered work item reviewing constructors, destruction, lifetime transitions, move/copy, aliasing, generated assignment, panic, async, or concurrency |
| Applies To | Detailed mechanics and unresolved consequences of reconstructive replacement in existing storage |
| Owns | Provenance, aligned boundary dependencies, unresolved questions, activation pressure, and retirement criteria |
| Does Not Own | Accepted qualifier behavior, final constructor semantics, generated-operator ranking, lifetime policy, panic recovery, syntax beyond the current contextual boundary, or implementation |
| Source / Provenance | Work item `004` post-promotion review of varying places containing successive immutable lifetimes |

## Reading posture

[Zax qualifiers](../../language/qualifiers.md#reconstructive-replacement) owns
the current programmer-visible reason and boundary for reconstructive
replacement. This file preserves detailed questions that qualifier documentation
must not answer prematurely.

It is not required reading for ordinary qualifier or constructor use.

## Current boundary to preserve

Future work starts from these current constraints:

- one storage place may contain successive value lifetimes when it is varying;
- a generated reconstructive `=` scenario requires both a varying destination
  and a writable current path;
- readonly access cannot initiate replacement;
- a domain-specific `=` candidate does not receive conventional assignment
  meaning merely from its token;
- the compiler owns the reconstructive lifetime skeleton;
- a type customizes the transition through contextual `replacement +++`;
- `replacement` remains an ordinary identifier outside that narrow constructor
  declaration context;
- a replacement constructor cannot return results;
- `_` uses the destination's existing allocation and initially contains the
  previous receiver representation and resources;
- replacement construction has transitional mutable and writable authority
  without `unsafe pliable`;
- the right-hand operand uses the constructor's ordinary declared type,
  qualifications, and overload selection; and
- normal completion establishes a complete valid new value lifetime in the same
  place.

This input may refine details without violating those current programmer-facing
constraints.

## Why detailed work remains

In-place replacement is neither ordinary mutation nor ordinary construction
into empty storage.

At entry:

- the old enclosing lifetime is transitioning;
- members may still contain live resources;
- raw or typed pointers may refer to those members; and
- the right-hand operand may alias the destination or its contents.

At successful return:

- every required member must be valid;
- every old resource must be retained, moved, or destroyed exactly once;
- no invalid old-state access may remain; and
- the new enclosing lifetime must satisfy its invariants.

The language needs enough structure to make this predictable without forcing
all implementations to discard reusable resources.

## Member-state transition questions

Future construction work must decide:

1. Whether an untouched member's lifetime continues across replacement or is
   adopted into a new containing lifetime.
2. How the compiler tracks live, moved, destroyed, retained, and reconstructed
   member states.
3. When reading an old member becomes invalid.
4. Whether assigning a member performs mutation, field-level reconstruction, or
   selectable operator behavior.
5. How nested final, immutable, readonly, and unsafe-pliable members participate.
6. Whether a replacement constructor must mention every member or may rely on
   generated retention.
7. Which invariants the compiler can validate at normal return.
8. How unions, anonymous storage, padding, and address-sensitive members
   transition.

## Resource retention

The motivating behavior includes retaining expensive resources in place:

- database or service connection handles;
- allocated capacity;
- synchronization state;
- caches;
- mapped files;
- device or operating-system handles; and
- self-relative data structures.

Reusing one address does not automatically preserve a contained resource's
lifetime. The future design must distinguish:

- retaining a member and its lifetime;
- moving ownership without changing physical storage;
- ending and reconstructing a member at the same address; and
- leaving raw bits untouched without preserving a valid language-level value.

## Raw and typed pointer consequences

Pointers into the old representation may:

- remain valid when the exact pointee lifetime is retained;
- become invalid when the pointee is destroyed;
- point to a new object at the same address after reconstruction;
- require explicit unsafe acknowledgement; or
- prohibit a replacement operation while they remain usable.

Same-address reconstruction alone does not decide pointer validity. Complete
pointee lifetime, provenance, alias, and replacement rules remain necessary.

## Aliasing and self-replacement

Future work must address:

```zax
value = value
value = viewOf(value)
value = makeReplacementUsing(value)
```

Questions include:

- when the right-hand operand is evaluated;
- whether it borrows, copies, or moves from the destination;
- whether evaluation completes before the transition begins;
- how aliases into destination members are handled;
- whether explicit varying place-tracking references remain usable during the
  transition;
- whether callbacks may observe transitional state; and
- which cases are rejected, require unsafe behavior, or use a generated
  temporary.

## Generated candidate and constructor selection

The generated reconstructive `=` scenario may coexist with arbitrary
domain-specific `=` candidates.

Future operator and constructor work must define:

- when the generated scenario enters the candidate set;
- whether it has reserved priority for same-type or constructible replacement;
- how a type disables it;
- how ambiguity with a user-defined `=` candidate is diagnosed;
- how the best replacement constructor is selected from the right-hand operand;
- whether a default replacement constructor is generated;
- fallback to ordinary destruction plus move/copy construction;
- behavior when copy, move, construction, or destruction is disabled;
- the result shape of generated reconstructive `=`; and
- reflection of generated versus declared operations.

## Move, copy, `last`, and destruction

Replacement may retain old members, consume the right-hand operand, or combine
both sources.

Future work must define:

- terminal access to resources that are not retained;
- interaction with `last` and other ownership/lifetime proposals;
- whether replacement can claim resources before the old enclosing lifetime
  ends;
- destruction order for unretained members;
- cleanup of partially consumed right-hand values; and
- whether move/copy operations remain independently selectable inside the
  replacement constructor.

## Panic and partial transition

Panic is often terminal. If execution cannot recover, partially transitioned
storage need not become available to resumed ordinary code.

If future panic behavior supports recovery, construction and lifetime work must
define:

- which members remain live;
- which destructors run;
- whether the old or new enclosing lifetime owns retained resources;
- how a valid state is restored or abandoned; and
- whether replacement constructors must opt into, prohibit, or prove recoverable
  behavior.

## Reentrancy, async, and concurrency

Replacement constructors may call other functions, invoke callbacks, or interact
with async and concurrent code.

Future work must decide:

- whether transitional `_` may escape;
- whether callbacks can access aliases to the destination;
- whether replacement constructors may suspend;
- whether a varying place may be reconstructed while another task holds a
  place-tracking reference;
- what synchronization is required;
- what an immutable varying observer may assume across suspension; and
- how compilers preserve optimization correctness across possible replacement.

## What this input does not decide

This file does not establish:

- field-transition syntax;
- a stable borrow mechanism;
- generated candidate priority;
- recoverable panic;
- move/copy defaults;
- pointer provenance;
- async replacement;
- thread safety;
- the final ownership model;
- implementation data structures; or
- a transpiler/backend mapping.

## Activation pressure

Read this file when focused work begins on any of:

- constructors or destructors;
- replacement construction;
- generated `=` candidates;
- place and value lifetimes;
- move, copy, or `last`;
- member initialization-state tracking;
- pointer/reference provenance and aliasing;
- panic recovery;
- callbacks and reentrancy;
- async suspension; or
- concurrent replacement.

## Intended disposition and retirement

The future work must:

1. map each accepted rule to its construction, lifetime, operator, pointer,
   safety, async, or concurrency owner;
2. keep the qualifier owner focused on programmer-visible place/access
   constraints;
3. give every question above an accepted, deferred, rejected, or superseded
   disposition;
4. remove obsolete candidate syntax and assumptions; and
5. remove this file from the raw index and retire it when no unresolved material
   remains.
