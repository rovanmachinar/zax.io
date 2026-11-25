# Raw input: tracking finished members and provenance through source

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative; direction set by the maintainer |
| Audience | A follow-up numbered work item on specification-defined use and provenance tracking |
| Applies To | Members finished through a borrower; raw pointers made from references that may reach a `unique` source; possibly array elements |
| Owns | The maintainer's direction, representative examples, the accepted cost, open questions, the `terminal` marker for functions safe on a finished receiver, and the prior decision to keep distinct |
| Does Not Own | Current member and element rules ([transfer stances](../../language/transfer-stances.md#members), [arrays and slices](../../language/arrays-and-slices.md#finishing-elements-in-place)); the compiler-verified tag ([analysis controls](analysis-controls.md)); `unique`'s current guarantee ([pointer and stance deferrals](pointer-ownership-deferrals.md)) |
| Source / Provenance | Work item `040` (element and contained stances), member-versus-element discussion |

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
  cannot tell, `unsafe` is required.
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
  `unsafe<...>`.
- The same mechanism answers the `unique` concern. Today a raw pointer made
  from a reference of unseen origin needs `unsafe`, because the reference might
  reach a `unique` allocation. Under this direction, source tracking that proves
  no path reaches it from a `unique` source makes it safe; a fixed ABI export
  makes it an error; anything that cannot be tracked needs `unsafe`.

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
  more precise without `unsafe`.
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

Activate with the follow-up work item on use and provenance tracking. Move
accepted behavior into transfer stances, arrays and slices, pointers and
arenas, lifetimes and references, and safety and analysis, then retire this
file and remove it from [the raw index](README.md).
