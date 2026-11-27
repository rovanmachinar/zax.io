# Raw input: tracking finished members, provenance, and pointers through source

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative; direction set by the maintainer |
| Audience | Follow-up numbered work items on specification-defined use, provenance, and pointer tracking |
| Applies To | Members finished through a borrower; raw pointers made from references that may reach a `unique` source; tracked borrowing pointers and `* unsafe`; possibly array elements |
| Owns | The maintainer's direction, representative examples, the accepted cost, open questions, the decisions already taught for tracked pointers and how safety depends on them, the `terminal` marker for functions safe on a finished receiver, and the prior decision to keep distinct |
| Does Not Own | Current member and element rules ([transfer stances](../../language/transfer-stances.md#members), [arrays and slices](../../language/arrays-and-slices.md#finishing-elements-in-place)); current tracked-pointer and `* unsafe` teaching ([pointers, allocation, and arenas](../../language/pointers-and-arenas.md#tracked-and-untracked-borrowers)); safety claims and verification ([safety and analysis](../../language/safety-and-analysis.md)); `unique`'s current guarantee ([pointer and stance deferrals](pointer-ownership-deferrals.md)) |
| Source / Provenance | Work item `040` (element and contained stances), member-versus-element discussion; work item `041` (tracked pointers, `* unsafe`, presence-claim carry-forward) |

## The concern

Through writable access, a function may finish a member with `as last`, and
nothing tracks it. After the call, the caller's value holds a finished member
that later code may use:

```zax
refresh final : ()(input : Message mutable writable copy &) = {
  keepPayload(input.payload as last)    // allowed today, and untracked
}

message : Message
refresh(message)
message.payload.send()                  // only this line is wrong, and it is in the caller
```

Array elements already follow a stricter rule: through a borrowed array, an
element may be moved out but not finished, unless the caller gave up the whole
array.

## The maintainer's direction

- A specification-defined analysis tracks use after `last` through source. A
  finished member that no path touches again is safe. Where the analysis
  cannot tell, a `safe<not-finished>` claim is required.
- The error lands on the later use that causes the problem, and the diagnostic
  also shows where the member was finished:

  ```text
  error: use after last
    here: message.payload is used after it may have been finished
    finished by: refresh, messaging line 2, through input.payload as last
  ```

- Replacing the member before returning is safe, and callers then see nothing:

  ```zax
  refresh final : ()(input : Message mutable writable copy &) = {
    keepPayload(input.payload as last)
    input.payload .= makePayload()      // replaced before return: nothing finished escapes
  }
  ```

- Tracking follows source, not a compiled ABI. When a function is compiled into
  a fixed exported interface, its callers cannot be tracked, so the `last`
  inside the exported function becomes the error, unless the function provably
  replaces the member before returning or the site is written inside
  `safe<not-finished>{...}`.
- The same mechanism answers the `unique` concern. A raw pointer made from a
  reference of unseen origin needs a `safe<unique-raw-borrow>` claim when the
  compiler cannot rule out a `unique` source, as current
  [pointers](../../language/pointers-and-arenas.md#raw-pointers-and-unique-allocations)
  teaching says. Under this direction, source tracking that proves no path
  reaches it from a `unique` source makes it safe without the claim; a fixed
  ABI export makes it an error; anything that cannot be tracked needs the
  claim.

Today, current teaching tracks `as last` within one function only. A member
finished inside a called function, through a writable parameter, is left to
that function and the type, as
[transfer stances](../../language/transfer-stances.md#members) says. The
`safe<not-finished>` category exists now for uncertainty within a function;
this direction extends it across calls.

## Accepted cost

Editing a function's body can create an error in a caller, because inference
through source makes body changes visible downstream. The diagnostic's pointer
back to the origin keeps that manageable. For comparison, Rust avoids this for
borrows, because a function's signature is its whole contract there, and has
the same cost only where facts leak out of a body, such as whether an
`impl Trait` return or an `async fn` future is `Send`.

## Open questions

- Whether array elements should use the same tracking, for example a constant
  index the caller never touches again. Until then, the element rule stays an
  error through a borrowed array.
- Whether inferred effects need an optional declared form for library
  interfaces that want a stable contract.
- The exact tracking rules, which the specification must define, so that
  every compiler accepts the same programs.

## Tracked pointers and `* unsafe`

Current [pointers](../../language/pointers-and-arenas.md#tracked-and-untracked-borrowers)
teaching treats these decisions as true. Follow-up work is expected to refine
them for corner cases; if it revises or reverses one, it must also revisit the
safety classifications listed below that depend on it.

```zax
node : MyNode = makeNode()
link : MyNode * = node             // tracked borrow, like a slice
use(link.)                         // checked: node is still alive here

parent : MyNode * unsafe = link    // fine: leaving tracking is one-way
use(parent.)                       // the programmer's responsibility

back : MyNode * = parent           // error: an untracked pointer can't re-enter tracking
```

Decisions made:

- A plain borrowing `*` is tracked like a slice: the compiler checks the
  target's lifetime at each access to what the pointer points to.
- `* unsafe` marks an untracked pointer. `unsafe` is a dimension of the
  pointer layer. A tracked `*` converts to `* unsafe`, never the reverse.
- Where the compiler cannot tell whether an access through a tracked `*` is
  safe, a `safe<...>` claim asserts it.
- Every possibly vacant access through a tracked `*` needs proof or
  `safe<pointer-presence>`, whatever the type's Nothing policy. The error and
  the claim land on the dereferencing operation, such as `myPointer.`.
- A claim or recognized `?ptr` test carries forward on the same path until
  something could change the pointer. Across calls, tracking follows source:
  when every call path into a function passes a non-vacant pointer, the
  function's accesses are safe, including when exported from its module; once
  compiled into a fixed exported interface, a possibly vacant access inside it
  is an error. A pointer type that cannot be vacant is the subject of
  [present pointers](present-pointers.md).
- A `unique` allocation still allows no tracked raw borrowers: origin tracking
  is separate from borrow tracking. A raw pointer made directly from a
  `unique` owner is an error, and `* unsafe` is the only escape.
- The uses current teaching formerly gave to untracked raw pointers (parent
  links, graphs, addresses stored as data, foreign code, hardware addresses)
  now use `* unsafe`.

Safety classifications that depend on these decisions:

- `safe<pointer-presence>` and its carry-forward rule;
- `safe<unique-raw-borrow>` for uncertain origin, and the proved-violation
  error for direct borrowing from a `unique`;
- `* unsafe` as an unchecked form on the `unsafe` side of the split, including
  inspecting a destroyed value's storage through a `* unsafe` borrow;
- `safe<...>` claims covering gaps in tracked-pointer analysis.

Open questions:

- **Why `unique` forbids tracked raw borrowers.** Earlier teaching gave the
  reason that raw pointers were untracked while references and slices were
  checked. With tracked `*`, that reason no longer holds, yet the maintainer
  kept the rule: a raw pointer made directly from a `unique` owner is an
  error. Current teaching states the rule without a reason. The follow-up must
  state what a tracked raw borrower can do that a reference or slice cannot
  (for example repointing, storage, or conversion to `* unsafe`) that breaks
  `unique`'s promise, or relax the rule.
- **Whether owning and managed pointers follow the tracked presence rule.**
  Current teaching applies "a possibly vacant access needs proof or
  `safe<pointer-presence>`" to tracked borrowing pointers, leaves `* unsafe`
  unchecked, and says that owning (`* move`, `* last`) and managed (`unique`,
  `strong`, `weak`) pointers are future work.
- How a tracked `*` stored in a member or graph edge is tracked.
- Whether a tracked `*` falls back to a runtime check where proof fails, as
  slices do with their invalidated-slice check, or only to a `safe<...>`
  claim.
- Whether re-entering tracking is ever allowed, for example through an
  `unsafe` cast, as `cast as last` re-enters ownership tracking.
- What `unsafe` means on an owning pointer layer, such as `* last unsafe` or
  `* move unsafe`.
- The exact trigger rules for `safe<pointer-presence>`, including where the
  error lands when a pointer parameter, rather than a reference, receives a
  possibly vacant argument.
- How raw pointers crossing thread boundaries interact with tracking, together
  with the threading capture in [safety boundaries](safety.md).

Activation: the follow-up work item on pointer tracking.

## Functions declared safe on a finished receiver (`terminal`)

Today Zax has no declaration that marks which operations are safe on a
finished value. A caller who knows an operation is safe acknowledges it, and
only the destructor and replacement constructor are exempt:

```zax
submit(command as last)

intent<terminal-source-reuse>{
  command.releaseRemainingHandle()   // the caller acknowledges it, trusting the type's documentation
}
```

The type's author knows which operations are safe, but only the caller writes
anything. The maintainer's direction: capture an author-side marker, for later
examination. It is not a stance, because a stance says what may be built from
a value. In the maintainer's words, the marker says: "this acknowledges that
`_` might already be terminal when you call it, and that's okay."

```zax
// illustrative only: no such marker exists
MyCommand :: type transfer copy move last {
  releaseRemainingHandle final : ()() terminal = {
    // _ may be finished here
  }
  run final : ()() = {
  }
}

submit(command as last)
command.releaseRemainingHandle()   // fine: declared safe after last; no acknowledgement needed
command.run()                      // error: run isn't declared safe on a finished value
```

Points to settle:

- **Position and spelling.** It describes the receiver's possible state, so
  it belongs with the receiver's qualifiers, not among the stances. `terminal`
  matches the acknowledgement vocabulary (`terminal-source-reuse`); the
  spelling is open.
- **What the body may do with `_`.** A candidate rule: destroy or replace
  members, read what the type can always read, and call other functions
  marked the same way; never build from `_` or call unmarked functions on it.
  The compiler can check that much; whether the operation makes sense stays
  the author's promise.
- **Relation to tracking.** Calling a marked function on a finished member or
  value would not count as a use after `last`, which makes the tracking above
  more precise without a `safe<not-finished>` claim.
- **Language-defined instances.** Array `length()` and capacity queries are
  already allowed on a partially finished array, because they touch no
  element. Under this marker they would be the language's own terminal-safe
  operations rather than a special case.
- **The acknowledgement keeps a role:** calling an unmarked function when the
  caller knows better.

[Safety boundaries](safety.md) and [analysis controls](analysis-controls.md)
already list operations unavailable in moved-from or terminal state as
pressure; this marker is one answer to it.

## Prior decision to keep distinct

[Safety boundaries](safety.md#rejected-warnings-for-members-last-may-have-taken)
records a rejected proposal to warn when a destructor or replacement
constructor uses a member that the type's own `last` may have taken: how a type
knows whether a resource is present stays part of its design. This direction
is different. It tracks uses, outside the type, of a member finished through a
borrower. Follow-up work keeps the type's own lifecycle code outside the
tracking unless that rejection's reopen condition is met.

## Activation and retirement

Activate with the follow-up work items on use and provenance tracking and on
pointer tracking. Move accepted behavior into transfer stances, arrays and
slices, pointers and arenas, lifetimes and references, and safety and
analysis, then retire this file and remove it from
[the raw index](README.md).
