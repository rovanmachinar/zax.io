# Raw input: pointer and stance deferrals

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work on stance fallback, interop, replacement, pointer ownership, and `unique` guarantees |
| Applies To | Long-term questions left open when stances were extended to pointed-to values |
| Owns | Each deferred question, the example that shows it, why it can wait, what reactivates it, and any constraint it places on current design |
| Does Not Own | Current transfer stances ([transfer stances](../../language/transfer-stances.md)); current pointer ownership ([pointers, allocation, and arenas](../../language/pointers-and-arenas.md)); current construction and fallback ([construction, replacement, and destruction](../../language/construction-and-destruction.md)) |
| Source / Provenance | Work item `039` (pointer stances), part C "Deferred elsewhere" |

## Reading posture

Each item below is a question current design deliberately leaves open. None of
them invalidates current design. Read an item only when its reactivation
pressure appears.

## Making a stance's use an error instead of a fallback (`forbid`)

Today, leaving a stance out of a type's `transfer` list means the operation
doesn't exist, and fallback passes over it: on a `transfer copy` type,
`as move` copies. A type can't write `= forbidden` on an unlisted stance.

Two ways to make `as move` an error were considered: a `forbid` word in the
`transfer` list, or allowing `= forbidden` on an unlisted stance's signature.
Both break three places that rely on fallback:

```zax
// candidate syntax, not adopted
MyPoint :: type transfer copy forbid move {
  x : Integer
}

makePoint final : (result : MyPoint)() = {
}

existing : MyPoint
existing = makePoint()          // error: the unnamed result is treated as written as last,
                                // which falls back through move, which is forbidden

points : MyPoint[0..]
points.append(makePoint())      // error for the same reason; arrays also move
                                // elements when they grow or relocate

keepAll$ final : (T : type)()(item : T via move) = {
}
keepAll(point as move)          // error: generic code that moves any T breaks for MyPoint
```

- **Why it can wait:** a type that must never be duplicated leaves `copy` out
  of its list. A type that wants to discourage `as move` because it only
  copies is better served by a warning, which never changes what compiles.
- **Reactivates:** a concrete type that needs `as move` to be an error, with
  answers for temporaries, array growth, and generic code.
- **Constraint if it returns:** a declared stance that `forbid` makes unusable
  is an error at the declaration, as in `point : MyPoint move`.

## Foreign-function declarations that take `&`

Lending a value to foreign code today needs `unsafe`, because a raw pointer
made from a reference of unseen origin needs `unsafe`. A foreign function's Zax
declaration could instead take a reference and pass an address at the
boundary, placing trust in the declaration:

```zax
// illustrative syntax only
foreignInspect final : ()(value : MyValue readonly loan &) foreign = { }
foreignInspect(myValue)         // the address is passed at the boundary; no unsafe at the call
```

- **Why it can wait:** no interop design exists yet.
- **Reactivates:** interop and foreign-function design.

## Replacing a pinned `loan` variable in place with `.=`

A pinned value keeps its address and nothing is built from it. Whether `.=`
may end it and build a successor in the same place is undecided:

```zax
device : MyDevice loan = openDevice()
device .= openDevice()          // undecided: same address, new value
```

- **Reactivates:** replacement design, or the first example that needs it.

## Owning raw pointers compared with blockless `unique`

An owning raw pointer (`* move` or `* last`) and a blockless `unique` both own
one allocation alone with no control block, and neither can point inside an
allocation. They differ in that only the raw owner allows raw borrowers:

```zax
rawOwner : MyValue * last = @
uniqueOwner : MyValue * unique = @
a : MyValue * = rawOwner.       // fine: a raw owner allows raw borrowers
b : MyValue * = uniqueOwner.    // error outside unsafe: unique allows no raw borrowers
```

- **Reactivates:** another overlap or divergence that makes the two need to
  merge or separate further.

## Raw borrows made before a raw-to-`unique` conversion

A raw owner may hand out raw borrowers and then become `unique`. The earlier
borrowers may still exist:

```zax
rawOwner : MyValue * last = @
early : MyValue * = rawOwner.
owner : MyValue * unique = rawOwner as last
use(early.)                     // accepted for now: unique's "no raw borrowers" isn't complete here
```

- **Why it can wait:** it is the programmer's job to keep raw borrowers valid,
  as for any raw owner.
- **Reactivates:** a need for `unique`'s guarantee to be complete.

## `deep` on owning pointer layers

`deep` never clones what a pointer points to. Copying a `strong` pointer deeply
would allocate, and a copy has no way to name the arena:

```zax
// candidate syntax, not adopted
shared : MyValue deep * strong deep = @
```

- **Reactivates:** a use that can name an allocation arena for the copy.

## Retirement

Retire each item when a work item decides it, moving the result into its
current owner. Remove this file from [the raw index](README.md) when no item
remains.
