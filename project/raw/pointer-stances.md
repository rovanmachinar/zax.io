# Raw input: stances on pointed-to values

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Work item `039` (pointer stances), which designs stances on pointer, reference, optional, and array layers |
| Applies To | Whether and how a pointer type can say what may happen to the value it points to, through any nesting of `*` and `&` |
| Owns | Preservation of the concern, the motivating cases, a candidate direction, and the questions that work must answer |
| Does Not Own | Current transfer stances ([transfer stances](../../language/transfer-stances.md)), current pointer ownership ([pointers, allocation, and arenas](../../language/pointers-and-arenas.md)), or current qualifier attachment ([qualifiers](../../language/qualifiers.md)) |
| Source / Provenance | Work item `038` (finding AG and the cold reads after its promotion) |

## The concern

A stance on a value applies to the whole value. A pointer variable's whole
value is the pointer: its address and any ownership it holds. So a pointer's
stance says how the pointer itself may be passed on, and nothing about the
value it points to:

```zax
owner : MyValue * unique = makeOwned()
keep(owner as last)          // the stance applies to the pointer: ownership moves
```

A reference has no value of its own that can be passed around, so its stance
can only describe the value it refers to. That is how `& loan` promises that
nothing is copied or taken from the referred-to value.

Once a pointer is dereferenced, nothing restricts the pointed-to value. A
pointer made from a loan would therefore let code copy a value the caller only
lent:

```zax
keep final : ()(value : MyValue) = {
}

show final : ()(input : MyValue readonly &) = {   // the caller lent input
  view : MyValue * = input    // error outside unsafe today
  keep(view.)                 // if allowed: copies the value the caller only lent
}
```

The current design forbids making a raw pointer from a loan outside `unsafe`.
That closes the gap, but it also means a loan can never be passed where a
pointer is expected. The receiver `_` is pointer-shaped and already carries a
loan, so the gap shows up even without the programmer forming a pointer:

```zax
inspectMapped final : ()(value : MyMappedValue *) = {
}

MyMappedValue :: type {
  show final : ()() = {        // no receiver stance: _ is a loan
    inspectMapped(_)           // error outside unsafe under the current rule
  }
}
```

A data-oriented language that works through pointers, and interoperates with
C, will want to lend through a pointer without `unsafe`.

## Why stances were kept off indirection layers

Qualifiers such as `readonly` and `final` attach to each layer of a type, as
in `MyValue readonly * mutable`. Stances do not. The recorded reason was that a
stance described a one-time choice made at a use, `as move`, not a property of
a type or a place. Since then, a declared stance on a variable became a lasting
requirement, `loan` became the enforced stance of a reference, and `_` carries
a loan while being pointer-shaped. That reason no longer holds on its own.

Work item `038` also considered per-layer stances for a different problem, a
local's declared stance not following the value through references, and
`loan` solved that without them. The pointer case was not considered then.

## Candidate direction (agent suggestion, not aligned)

Let a stance be written on the pointed-to layer of a pointer type, in the same
position qualifiers use for that layer. It would mean what the holder may do to
the pointed-to value, exactly as a reference's stance does:

```zax
// illustrative syntax
show final : ()(input : MyValue readonly &) = {
  view : MyValue loan * = input    // fine: a pointer to a lent value
  inspect(view.)                   // fine: lends it on
  keep(view.)                      // error: view's pointee is on loan
  other : MyValue * = view         // error: would drop the loan
}

inspectMapped final : ()(value : MyMappedValue loan *) = {
}

MyMappedValue :: type {
  show final : ()() = {
    inspectMapped(_)               // fine: the parameter accepts a lent pointee
  }
}
```

Suggested starting rules:

- **Default for an unmarked pointee layer:** unrestricted, as raw pointers are
  today. A default of `loan`, as for references, would change nearly all
  pointer code.
- **Allowed stances on a pointee layer:** `loan`, `copy`, `move`, `last`, and
  perhaps `deep`, with the reference meanings.
- **Conversions:** adding a pointee stance that allows less is always allowed;
  dropping or strengthening one is an error outside `unsafe`.
- **Owning pointers:** a `unique` or `strong` pointer whose pointee is on loan
  is a contradiction and an error.
- **Anything made from such a pointer** carries the pointee stance, as
  everything made from a `loan` reference does today.

## Questions the work item must answer

- **Nesting.** How stances attach through several layers, such as
  `MyValue * *`, `MyValue & *`, and `MyValue * &`, and what each layer's stance
  governs. A reference's stance governs its referent; a pointer's outer stance
  governs the pointer. Does a pointer to a pointer need a stance on each layer?
- **Interaction with qualifiers** on the same layer, and the enforced order of
  words on a layer.
- **References:** whether a reference's existing stance is simply "the
  pointee-layer stance of a reference", making references and pointers
  uniform.
- **`_`:** whether the receiver becomes `MyType loan *` in a function with no
  receiver stance, which would explain the current rule rather than add one.
- **Ownership kinds:** which combinations of `unique`, `strong`, `weak`, raw,
  and a pointee stance are meaningful.
- **Generic code:** how a type parameter that may be a pointer carries a
  pointee stance.
- **Interop:** how pointee stances map to foreign pointers, where nothing can be
  enforced.
- **Costs and diagnostics:** none at run time are expected; diagnostics must
  say which layer's stance a use violates.

## Related questions from the 038 review

- **Allocation.** Whether `loan` on a pointer, or the other stances with a
  pointer-specific meaning, can help with the allocation problems pointers
  have, for example lending an allocation without passing its ownership.
  Re-evaluate `loan`, `copy`, `move`, and `last` for raw, `unique`, and shared
  pointers, not only for the pointed-to value.
- **Arrays.** If a pointed-to value can carry a stance, can array elements
  carry their own stance separate from the array, and is an element reached
  through an array the same question as a value reached through a pointer?
- **Optional values.** An optional value holds its contained value much as a
  pointer refers to its pointee. Whether the contained value needs a stance
  separate from the optional itself should be examined in the same design.
- **Who frees a `copy` allocating result.** The current pages say a raw
  pointer result declared `result : MyValue * copy = @` lets callers only
  borrow the pointer, and that the allocation is freed when the call's result
  is destroyed. The maintainer questioned this. Verify who frees the allocation
  in that case, and whether a borrowed result of this kind should exist at
  all.

## Activation and retirement

- **Activates:** work item `039`, assigned to design pointer stances.
- **Constraint on current design:** until then, making a raw pointer from a
  loan, including passing a loan `_` to a raw pointer parameter, is an error
  outside `unsafe`.
- **Retires:** when `039` promotes its findings into the pointer, transfer
  stance, qualifier, and invocation owners, or rejects the direction with a
  recorded reason.
