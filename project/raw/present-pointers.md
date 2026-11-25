# Raw input: present (non-vacant) pointers

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work designing non-vacant pointer types and presence proofs |
| Applies To | Pointer-layer non-vacancy (`present`), flow narrowing after `?p`, narrow presence-only `unsafe cast`, and interaction with `last`, constructors, Nothing instances, and callable contracts |
| Owns | Captured motivation, aligned direction, illustrative syntax, counterexamples, open questions, activation pressure, and retirement criteria for `present` pointers |
| Does Not Own | Accepted current pointer behavior ([pointers, allocation, and arenas](../../language/pointers-and-arenas.md)); transfer stances ([transfer stances](../../language/transfer-stances.md)); Nothing-instance policies ([Nothing instances](../../language/nothing-instances.md)); or general callable contracts ([callable selection](callable-selection.md)) |
| Source / Provenance | Maintainer design discussion during work item `039` promotion review of `language/pointers-and-arenas.md` (2026-11-21); work item `040` (element and contained stances) added the note on question 6 |

## Why this input exists

Today every Zax pointer type allows vacancy—targeting the pointee type's
Nothing instance—even when an allocation just succeeded with `@` or the caller
already checked `if ?ptr`. References (`&`) are never vacant, but a reference
cannot own an allocation, cannot be rebound, and cannot be stored in ordinary
long-lived structures.

Without a way to record non-vacancy in the pointer type itself:

- functions that accept owning or borrowing pointers cannot state non-vacancy
  as a precondition on the pointer type;
- struct members initialized with `= @` still look possibly vacant at every
  use; and
- callers and helpers repeat `if ?ptr` checks even when non-vacancy was
  already established upstream.

## What this promises—and what it does not

Two kinds of pointer validity exist:

1. **Non-vacancy:** this pointer is guaranteed not to target the pointee
   type's Nothing instance.
2. **Pointee liveness:** the target a borrowing raw pointer points at is still
   alive and has not ended.

This feature models **only non-vacancy**. For an owning pointer (`unique`,
`unique shareable`, `strong`, or an owning raw pointer), non-vacancy implies a
live owned allocation. For a borrowing raw pointer, non-vacancy means only
that the pointer holds a non-vacant address, never that the target is still
alive.

## Aligned direction and illustrative syntax

All syntax below is **illustrative** until a dedicated work item reviews and
accepts it.

### Spelling and position

`present` is the preferred word (matching Zax's existing presence terminology:
`?p`, "present pointer", "definitely present result"). `here` is the only
alternative kept on file. `valid` was rejected because readers would mistake a
borrowing `MyType * valid` for a liveness guarantee.

The word belongs on the pointer layer, before the stance (since the stance is
always last on its layer):

```zax
// illustrative syntax
rawOwner : MyType * present last = @
soleOwner : MyType * unique present = @
sharedOwner : MyType * strong present = @
```

`weak` pointers **do not** allow `present`: a `weak` pointer remains
non-vacant even after strong ownership closes, so `weak present` would mislead
readers into assuming the target is alive.

```zax
// illustrative syntax
badWeak : MyType * weak present = sharedOwner   // error: weak pointers do not support present
```

### How a pointer becomes `present`

1. **Panicking allocation (`@`)** produces a `present` pointer directly.
   Non-panicking allocation (`@!`), weak-to-strong acquisition, failed
   shared-to-unique / shared-to-owning-raw claims, and `OpaqueObserver`
   recovery can all yield vacancy, so they produce ordinary (possibly vacant)
   pointers.

   ```zax
   // illustrative syntax
   guaranteed : MyType * unique present = @
   maybe : MyType * unique = @!
   bad : MyType * unique present = @!           // error: @! may produce a vacant pointer
   ```

2. **Flow narrowing after `if ?ptr`** proves non-vacancy inside the branch
   until an operation in the branch could make `ptr` vacant again (such as
   reassignment, `reset`, or passing `ptr` by mutable reference):

   ```zax
   // illustrative syntax
   if ?view {
     sure : MyType loan * present = view        // fine: compiler proved view is non-vacant here
     useSure(sure)
   }
   ```

   Remember that forming a raw borrower through `ptr.` from a `unique` owner
   still requires `unsafe`, regardless of `present`.

3. **Narrow presence-only unsafe cast.** Because `unsafe cast T` names a full
   target type, using it to assert presence risks an accidental type mistake.
   A dedicated form should change **only** `present` on the outermost layer,
   leaving the type, qualifiers, ownership role, and stance unchanged (with
   `.` reaching inner layers, parallel to `unsafe cast as S`):

   ```zax
   // illustrative syntax
   takeMyChances : MyType loan * present = view unsafe cast as present
   ```

   By the same single-change principle, a type cast (`view unsafe cast OtherType *`)
   should keep the existing presence status rather than silently adding
   `present`.

### Widening and variance

Widening from `present` to an ordinary possibly-vacant pointer is safe by
value or through a `readonly` path, and forbidden through a writable pointer
or reference (which would allow storing a vacant pointer into a `present`
slot):

```zax
// illustrative syntax
notSure : MyType loan * = sure                  // fine: safe widening
readView : MyType loan * readonly loan & = sure // fine: readonly path cannot store vacancy
writeSlot : MyType loan * mutable writable loan & = sure
// error: writing a vacant pointer through writeSlot would break sure's present invariant
```

### Lifecycle, constructors, and `last`

- **No zero-input default.** A `present` pointer cannot start uninitialized
  with no value:

  ```zax
  // illustrative syntax
  missing : MyType * unique present             // error: a present pointer cannot start vacant
  ```

  A type with a `present` pointer member has no zero-input default constructor
  unless that member has an initializer (such as `= @`). Generic code that
  default-constructs `T` rejects `T = MyType * present` like any other type
  with no zero-input constructor.

- **Constructors (`+++`).** Inside a constructor, a `present` pointer member
  starts unconstructed and must be established before the constructor finishes;
  the first use establishes it as `present`.

- **`as move` is unavailable on owning `present` pointers.** Moving out of an
  owning pointer leaves it vacant while `move` promises a usable value
  afterward, so `unique present` and owning raw `present` pointers cannot
  support `move` (owning raw `present` is `* present last`, not `* present move`).
  `strong present` still supports `copy` and `last`.

- **`as last` is allowed.** After `as last`, the pointer enters the finished
  state—internally vacant, but the compiler already forbids reading or
  dereferencing it until `.=` replaces it or scope end destroys it:

  ```zax
  // illustrative syntax
  owner : MyType * unique present = @
  take(owner as last)                           // fine: owner is now finished
  use(owner.)                                   // error: owner is finished
  owner .= @                                    // fine: re-establishes owner as present
  ```

- **Result slots (`= @` and bodies).** A `present` result cannot be vacant at
  any ordinary read or on any normal exit. Temporary vacancy inside a function
  body is allowed only in the finished (`last`) state, where the compiler
  forbids access until `.=` re-establishes the result before return.

- **Ordinary `reset` and `vacate` do not apply.** Both leave a pointer usable
  and vacant, which violates `present`. Replacing in place uses `.=`, and
  handing off uses `as last`.

## What this input does not decide

Future work must resolve these open questions before promotion:

1. **Reading a `present` member through a vacant outer pointer under a
   non-trapping Nothing policy.** If `outer : Outer *` is vacant and `Outer`
   contains `inner : Inner * present`, dereferencing `outer` traps under a
   trapping Nothing policy before reaching `inner`, but under a non-trapping
   Nothing policy `Outer`'s Nothing instance has no real `Inner` allocation.
   Define the programmer-visible rule for `outer.inner`.
2. **Early release or abandonment of a `present` owning pointer.** Decide
   whether `present` pointers need a terminal `reset` / `unsafe vacate` form
   (putting the variable into the finished state so an allocation can be freed
   or abandoned before scope end without immediately allocating a replacement),
   or whether `.=` plus `as last` is sufficient.
3. **Exact spelling of the narrow unsafe cast** (`unsafe cast as present` vs
   another exact phrase) and its interaction with `unsafe cast as S` and
   `unsafe cast T`.
4. **Interaction with future callable contracts** (`requires ?foo` /
   `ensures ?result` in [callable selection](callable-selection.md)): when
   `present` on the pointer type is preferred over a contract predicate, and
   how prototypes compare them.
5. **Interaction with `OpaqueObserver`:** whether `OpaqueObserver` records
   `present` or whether recovery always yields a possibly-vacant pointer
   because runtime type/stance mismatch itself produces vacancy.
6. **Optional values:** whether the same `present` modifier should extend to
   non-empty optionals (`MyValue? present`) or stay pointer-specific. An
   optional's own stance is written after `?`, the position such a modifier
   would also use, in the established qualifier-then-stance order (for
   example `MyValue move? present move`); the optional-reference rules do not
   otherwise constrain this question.

## Activation pressure

Consume this input in the next pointer/container follow-up work item (or a
dedicated work item on pointer presence) whenever non-vacant pointer types,
pre-validated pointer parameters/members, or callable presence contracts are
designed.

## Retirement

Move accepted rules into [pointers, allocation, and arenas](../../language/pointers-and-arenas.md),
[Nothing instances](../../language/nothing-instances.md),
[conversions and casts](../../language/casting.md),
[qualifiers](../../language/qualifiers.md), and
[terms](../../language/terms.md) (plus any affected invocation, construction,
or safety owners), then remove this file and its entry in
[project/raw/README.md](README.md).
