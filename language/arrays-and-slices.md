# Zax arrays and slices

| Field | Value |
| --- | --- |
| Status | Current conceptual design |
| Audience | Human developers storing, reading, changing, viewing, or traversing ordered data |
| Applies To | Programmer-facing intrinsic arrays, array expressions, indexing, slicing, capacity, layout, and element lifetime; not a formal grammar or specification |
| Implementation State | Not established by this repository |
| Owns | Fixed and resizable array identities; logical length and usable capacity; array expressions and inference; intrinsic index and splice behavior; borrowed slices; stances on elements and slices; element and array qualifiers; direct splice mutation; taking elements with `take` and `last X[range]`; element-place stability and relocation; inline and provider-backed storage; programmer-visible storage selection and capabilities; array transfer, what `move` and `last` leave, and joining; finishing elements in place; storage-erased array references; array and slice parameter behavior; array-specific costs, diagnostics, reflection facts, and future boundaries |
| Does Not Own | General construction packets and lifecycle machinery ([construction, replacement, and destruction](construction-and-destruction.md)); reusable reference lifetimes ([lifetimes and references](lifetimes-and-references.md)); pointer ownership and arenas ([pointers, allocation, and arenas](pointers-and-arenas.md)); exact array storage-provider/handle protocols; shared operator and mixfix selection ([operators](operators.md), [mixfix operators](mixfix-operators.md)); general traversal protocol behavior ([iteration](iteration.md)); qualifier meaning ([qualifiers](qualifiers.md)); transfer-stance meaning ([transfer stances](transfer-stances.md)); general integer roles ([integers](integers.md)); general structural transformation ([structural shapes and compatibility](structural-shapes-and-compatibility.md)); runtime string APIs; user-defined generics ([generics](generics.md)); variadic packs ([variadics](variadics.md)); or implementation layout and lowering |

## Start with an ordinary array

Use `[ ... ]` to provide the values of an array:

```zax
values : Integer[5] = [ 10, 20, 30, 40, 50 ]

third := values[2]
third = 300
// values[2] is still 30; third is an independent Integer.

thirdInArray : Integer writable & = values[2]
thirdInArray = 300
// values[2] is now 300.
```

`Integer[5]` says that `values` always contains exactly five integers. Array
indexes start at zero, so `values[2]` selects the third integer.

What happens after selection depends on how the result is used:

- `third := values[2]` constructs a separate `Integer` from the selected
  element.
- `thirdInArray : Integer writable & = values[2]` keeps access to that element
  inside `values`.

Direct traversal visits every element in increasing index order:

```zax
each element : in values {
  print(element)
}
```

The inferred traversal binding accesses each array element directly. An
explicit value binding would instead construct a separate value for each pass.

## Fixed arrays, resizable arrays, and slices

Zax provides three related sequence forms.

### Fixed arrays

```zax
values : Integer[5]
```

A fixed array always contains its declared number of elements. Its length is
part of its type. It can change individual elements when their types and the
current access allow, but it cannot insert, remove, or resize.

### Resizable arrays

```zax
scores : Integer[0..100]

scores <+>= [ 10, 20 ]
// scores now contains two elements.
```

`Integer[0..100]` allows the array to contain anywhere from zero through one
hundred integers. The array begins with the minimum required count—zero here—and
size-changing operations must keep it inside those limits.

The range is a contract on the array. It does not mean that storage for one
hundred elements is immediately reserved.

### Slices

```zax
values : Integer[5] = [ 10, 20, 30, 40, 50 ]
middle := values[1..<4]

middle[0] = 200
// values[1] is now 200.
```

A slice is a window into part of an existing array. `middle` views the three
elements at indexes `1`, `2`, and `3`; it does not create another copy of them.

The slice can remember a narrower window, but it cannot expand beyond the
elements it was originally allowed to view. The array must remain alive, and
operations that move or remove viewed elements can invalidate the slice.

A slice is a borrower of the elements it shows, checked by lifetime analysis
like a reference. So a slice may view the elements of an array that a `unique`
pointer owns, where a raw pointer to an element would need `unsafe`:

```zax
owner : MyMessage[0..] * unique = @
window : MyMessage[] = owner.[1..<3]     // fine: a slice is checked like a reference
raw : MyMessage * = owner.[0]            // error outside unsafe: unique allows no raw borrowers
```

`T[N]`, ranged arrays, and `T[]` slices are built into Zax. Programmers declare
their own generic types with a `$` list, as described by
[generics](generics.md#generic-types-and-other-generic-declarations).

## Array expressions

An array expression constructs one owning array:

```zax
numbers := [ 1, 2, 3 ]
// Integer[3] using the resolved default storage profile

bytes : U8[3] = [ 1, 2, 3 ]
```

When the destination supplies a type, each element is constructed as that type.
Without a destination type, all entries must establish one common element type.

### Inferring the element type

Number literals that have not yet selected an integer type remain uncommitted
while the array is considered:

```zax
ordinary := [ 1, 2, 3 ]
// Every number uses the ordinary suggested default: Integer.

small := [ (: U8 = 1), 2 ]
// The committed U8 causes 2 to become U8 as well.
```

All committed elements must have the same concrete type:

```zax
myU8 : U8 = 1
myU16 : U16 = 2

bad := [ myU8, myU16 ]
// error: the committed element types differ
```

Inference does not choose the first entry, widen one integer to another, or
invent an implicit union or a conceptual “any” type. Zax has not established
`Any`, `Variant`, or `Union` as the name of such a type.

An empty expression cannot supply an element type:

```zax
empty := [] // error: no element type can be inferred
typedEmpty : Integer[0] = []
openEmpty : Integer[0..] = []
```

`[]` is the explicit zero-entry array expression. `[ ]` is not an alternate
empty spelling. In nested source, `[ [] ]` contains one empty inner array:

```zax
zeroRows : Integer[0..][0..] = []
oneEmptyRow : Integer[0..][0..] = [ [] ]
```

Omitted storage resolves through the declaration's applicable default storage
profile. A concrete provider type or instance can be stated when its storage
behavior is required.

### Construction packets as elements

A construction packet gets its destination type from the array element:

```zax
items : MyItem[3] = [
  [{ "first", 3 }],
  [{ "second", 7 }],
  [{}]
]
```

The outer `[ ... ]` constructs the array. Each inner `[{ ... }]` supplies the
constructor inputs for one `MyItem`.

```zax
unknown := [ [{ "first", 3 }] ]
// error: the packet has no expected element type
```

Packets do not independently infer their destination type.

### Nested arrays and lambda capture

Equal nested shapes infer fixed dimensions:

```zax
matrix := [ [ 1, 2 ], [ 3, 4 ] ]
// Integer[2][2] using the resolved default storage profile
```

The spaces in `[ [` and `] ]` are meaningful. Contiguous `[[]]` is an empty
lambda capture and nonempty capture requires interior spacing:

```zax
callback := [[ matrix ]] ()() {
  print(matrix[0][0])
}
```

Compact `[[matrix]]` is not a legal capture, and compact
`[[1, 2], [3, 4]]` is not an array spelling. Complete capture behavior belongs
to [Zax lambdas and callable composition](lambdas-and-callable-composition.md).

Rows of different sizes need an explicit type:

```zax
invalidJagged := [ [ 1, 2 ], [ 3 ] ]
// error: jagged inference requires an explicit type

jagged : Integer[2][1..2] = [
  [ 1, 2 ],
  [ 3 ]
]
```

The explicit declaration confirms that each inner row is allowed to resize
between one and two elements. Zax does not infer those future permissions from
the two initial row sizes.

## One element or several elements

Each ordinary entry contributes one value. A bare array therefore becomes one
element of an outer array:

```zax
firstRow : Integer[3]
secondRow : Integer[3]

rows := [ firstRow, secondRow ]
// Integer[2][3]
```

It does not automatically unpack into an array of six integers:

```zax
invalid : Integer[6] = [ firstRow, secondRow ]
// error: two Integer[3] values are not six Integer values
```

Writing a slice still contributes one slice value:

```zax
slices := [ firstRow[..], secondRow[..] ]
// Two Integer[] slice elements.
```

Write contextual `from` when the surrounding array should receive the sequence
elements:

```zax
values := [ from firstRow, from secondRow ]
// Integer[6]

alsoValues := [ from firstRow[..], from secondRow[..] ]
// Integer[6]; the full slices are redundant but explicit.
```

The same rule applies to a named slice:

```zax
view := firstRow[..]
views := [ view ]
expanded := [ from view ]
```

`from` means sequence expansion through direct array/slice traversal, the
cursor protocol, or the items of a [pack](#contributions-from-a-pack).
Structural `>-` decomposition remains a different operation.

### Exact contribution count

An exact-sized destination must receive exactly that many elements:

```zax
source : Integer[3] = [ 1, 2, 3 ]
destination : Integer[2] = [ from source ]
// error: three elements cannot initialize exactly two
```

Zax does not silently discard the extra element. When the count depends on a
runtime slice, the same mismatch is checked while the program runs.

If the destination allows several lengths and does not state an initial count,
the contributed count becomes its current length:

```zax
destination : Integer[0..10] = [ from source ]
// destination currently contains three elements.
```

### Runtime-sized slice contributions

An explicit runtime-sized slice produces a resizable result:

```zax
runtimeValues := [ from makeRuntimeSlice() ]
// MyItem[0..] using the resolved default storage profile

withOne := [ from makeRuntimeSlice(), makeItem() ]
// MyItem[1..] using the resolved default storage profile
```

The first result can contain any count supplied by its runtime slice. The second
always has at least the separately written `makeItem()` element.

These are resizable arrays, not runtime-dependent fixed array types.

### Contributions from a pack

A function that accepts any number of arguments receives them as a
[pack](variadics.md#accept-any-number-of-arguments). A pack is not an array.
It is a set of separate parameters whose count is known while compiling. When
you need the arguments as an array, perhaps to hand them to an API that takes
one, you can use `from` to turn the items into array elements:

```zax
sumAll final : (result : Integer)(values : Integer loan[]) = {
  total := 0
  each value : in values {
    total += value
  }
  return total
}

sum final : (result : Integer)(values ... : Integer) = {
  return sumAll([ from values ])   // the items become the array's elements
}
```

`from each` first passes every item through a call, and each call contributes
one result to the array:

```zax
measureWidth$ final : (UseType : type)(result : IndexSize)(value : UseType readonly loan &) = { }

widths$ final : (Items ... : type)(result : IndexSize[Items count])(items ... : Items) = {
  return [ from each item : in items |> measureWidth() ]   // one width per item
}

sizes := widths("label", 42, true)   // three IndexSize elements
```

The ordinary element rule still applies, so every contribution must share one
common element type. Items of different types do not make an array. Neither do
mapped results whose type differs from item to item:

```zax
collect$ final : (Items ... : type)()(items ... : Items) = {
  all := [ from items as copy ]
}

collect(1, 2)       // fine: both items are Integer
collect(1, "two")   // error: the committed element types differ
```

The mapping runs item by item, at its own position among the entries.
Ordinary entries written before and after it keep their written order.

Going the other way, from an array into a pack, depends on the length. A
pack's count must be known while compiling, so a resizable array or slice
cannot be spread into a pack. A fixed-length array can, because its length is
part of its type. An API that must accept a list whose length is known only at
run time declares an array or slice parameter instead, as `sumAll` does:

```zax
fixedScores : Integer[3] = [ 1, 2, 3 ]
scores : Integer[0..100] = [ 4, 5 ]

fixedTotal := sum(from fixedScores)   // fine: three values, known while compiling
badTotal := sum(from scores)          // error: a pack's count must be known while compiling
runtimeTotal := sumAll(scores)        // the runtime-length API
```

[Variadics](variadics.md#transforming-each-item) teaches the complete pack
model, including mapping and evaluation order.

## Iterable contributions

Use contextual `from` to fill an array from any cursor-driven iterable:

```zax
values : Integer[10] = [
  from makeMeRepeatIterable(0, 10)
]
```

The explicit `from` matters because every bare array, slice, or other iterable
is otherwise one ordinary array element:

```zax
fromArray := [ from sourceArray ]
fromSlice := [ from sourceArray[1..<4] ]
```

### When an iterable knows its count

Some iterables know exactly how many values they will produce. They may provide
the post-unary `count` operation:

```zax
MyIterable :: type {
  operator post unary 'count' final : (
    result : IndexSize
  )() readonly = {
    // Return exactly how many values the next traversal will produce.
  }
}
```

The operation is optional, but its returned count is exact. Array construction
may ask for that count once and reserve enough storage before traversal.

If the iterable later produces another number of values, the array operation
panics because the iterable broke its promise.

An iterable without `count` is still valid. A resizable destination grows while
values arrive; a fixed destination already knows its required storage. Explicit
capacity reservation or suggestion can avoid repeated growth.

### Evaluation and construction order

Array construction:

1. evaluates each ordinary entry and iterable-source expression once from left
   to right;
2. keeps the resulting values and iterable sources alive;
3. asks count-capable sources for their exact counts in entry order;
4. reserves final storage when the complete count is known, or at least the
   known minimum when one or more sources have no count;
5. consumes slices and iterables in entry order; and
6. constructs array elements in increasing index order.

For an uncounted iterable, each required storage expansion finishes before the
next element is constructed. The incomplete array is not available to ordinary
code.

For example:

```zax
values : Integer[0..] = [
  mutateThenReturnCurrentCount(firstIterable),
  from firstIterable,
  mutateThenReturnCurrentCount(secondIterable),
  from secondIterable
]
```

If only `firstIterable` supplies `count`, both mutation expressions and both
source captures run first. Construction then asks for the first iterable's
post-mutation count and reserves the two ordinary elements plus that known
count. It fills entries in written order and grows incrementally while consuming
the uncounted second iterable.

### Defaulting the remainder

Final bare `..` default-constructs the exact number of still-missing elements:

```zax
values : MyItem[5] = [
  makeFirst(),
  makeSecond(),
  ..
]
```

Here `..` default-constructs three more `MyItem` values. It requires an exact
target count and a default constructor. It cannot infer an arbitrary larger
length.

An iterable before `..` contributes until it ends. Producing too many values is
a count mismatch; ending early leaves the remaining elements for `..`.

Repeated values use an ordinary iterable such as
`makeMeRepeatIterable(value, count)`. Arrays need no separate repetition
syntax.

### Nothing preparation does not create elements

Compiler-provided Nothing backing does not ordinary-construct array elements.
An array member reached through a Nothing receiver is therefore invalid when the
operation requires an element:

```zax
MyArrayOwner :: type {
  values : Integer[5]

  first final once : (result : Integer)() = {
    return _.values[0] // invalid; trap/panic when target support permits
  }
}
```

Zeroed or reserved storage is not an array with live elements. A type may
instead choose trapping Nothing behavior or prepare a custom Nothing instance.
Complete policy behavior belongs to
[Zax Nothing instances](nothing-instances.md#what-compiler-preparation-provides).
A future narrower prepared-array form must preserve the element-lifetime rules
in this document. Targets without suitable trap support do not receive a
software vacancy check before every element access; relying on the missing trap
has undefined behavior.

## Logical length ranges

Array declaration ranges describe the counts that one array is allowed to have:

```zax
first : Integer[0..3]
// May contain 0, 1, 2, or 3 elements.

second : Integer[0..<3]
// May contain 0, 1, or 2 elements.

third : Integer[1 in 0..3]
// May contain 0 through 3 elements and initially contains 1.

fourth : Integer[2 in 0..3]
// May contain 0 through 3 elements and initially contains 2.

unboundedA : Integer[0..]
unboundedB : Integer[..]
// Both may contain any IndexSize-supported count from zero upward.

onlyEmpty : Integer[0..0]
// Current and only permitted length is zero.

impossible : Integer[0..<0]
// error: this half-open range contains no permitted length
```

`..` includes its upper count. `..<` excludes it.

`N in` states the exact initial count. It does not state capacity and is not
part of the array's type identity. A known invalid `N` is a compile-time error;
a runtime-invalid count panics unless a reporting operation was selected.

Without an initializer, a resizable array begins at its minimum required count
and default-constructs those elements.

`Integer[0..]` and `Integer[..]` resolve to the same array identity. Current
teaching usually writes the explicit zero minimum.

Exact `Integer[0]` and inclusive `Integer[0..0]` can represent an empty array.
The mathematical range `Integer[0..<0]` contains no allowed count and therefore
cannot be used for a value declaration.

## Length, capacity, and suggestion are different

Three numbers answer different questions:

- **Length:** How many elements exist now?
- **Capacity:** How many elements fit in already reserved storage?
- **Suggested capacity:** How much storage has the programmer suggested may be
  useful?

Every array and slice exposes its current logical length:

```zax
current : IndexSize = values.length()
```

A slice's length is the number of elements in its current window.

Type bounds are separate from current length:

```zax
ValuesType :: alias type type of values

minimum : IndexSize = minimum length of ValuesType
maximum : IndexSize? = maximum length of ValuesType
```

Minimum is always concrete. Maximum is absent for an open range. An exact
`T[N]` reports `N` for both.

For an array with finite maximum `M`:

```text
minimum length <= current length <= capacity <= M
```

Capacity beyond current length is only storage. No elements exist there yet.

### Querying capacity

```zax
actual : IndexSize = capacity of values
suggested : IndexSize? = suggested capacity of values
```

`capacity of` always returns the **usable capacity**: how many elements this
array could hold in its current backing without obtaining different element
storage and without exceeding its declared maximum. An array with no backing
storage has zero capacity.

The storage provider may physically own more. For example, a 128-slot chunk
backing an array whose maximum is 100 still produces array capacity 100.
Provider-specific inspection may expose physical storage separately.

`suggested capacity of` returns an optional count. It is an **absent optional**
until a suggestion is supplied.

### Reserving storage

```zax
values.reserveCapacity(128)
```

After normal completion, at least 128 elements fit without obtaining more
element storage. Length and existing values are unchanged. Reserving may move
all existing elements into a larger region, so references and slices can be
affected.

```zax
values.trimCapacity()
```

Trimming may release unnecessary physical storage but never reduces usable
capacity below current length. It may also move elements.

A request beyond the array's finite maximum is a compile-time error when known
and a panic otherwise.

### Setting and clearing a suggestion

```zax
values.suggestCapacity(256)

unset : IndexSize?
values.suggestCapacity(unset)
```

`suggestCapacity` accepts `IndexSize?`. A concrete `IndexSize` becomes a present
optional and sets the suggestion. An absent optional clears it.

Changing the suggestion records guidance only. It performs no immediate
allocation and moves no elements. Usable capacity may remain below or already
exceed the suggestion.

A known suggestion beyond a finite maximum is an error. A runtime value beyond
that maximum panics. A provider-specific physical-backing hint is a different
operation and may legitimately request storage the current array cannot expose
as usable capacity.

The compiler or runtime may use static bounds for private allocation choices,
but such a choice does not become a visible programmer suggestion.

## Slicing

Slice ranges describe element indexes, not permitted array lengths:

```zax
values : Integer[6] = [ 10, 20, 30, 40, 50, 60 ]

inclusive := values[0..5]
// Indexes 0 through 5: six elements.

halfOpen := values[0..<5]
// Indexes 0 through 4: five elements.

one := values[0..0]
// One element.

empty := values[0..<0]
// A valid empty slice at boundary 0.
```

A half-open slice includes its start and excludes its end. Both boundaries may
be equal, which creates an empty slice. The end may equal `values.length()`.

An inclusive slice includes both indexes. Internally, the same interval can be
described with an exclusive end one greater than the written inclusive end.

Omitted endpoints mean:

```text
values[start..<end] -> start through end - 1
values[start..end]  -> start through end
values[..<end]      -> beginning through end - 1
values[..end]       -> beginning through end
values[start..]     -> start through the last element
values[..]          -> the complete array
```

`start..<` is malformed because there is no end to exclude.

A known reversed or out-of-bounds range is a compile-time error. A runtime
required slice checks its boundaries and panics when they are invalid. It never
silently shortens the requested range or substitutes an empty slice.

## Qualifying an owning array

An owning array has two sets of qualifiers. Words written before `[` qualify
each element. Words written after `]` qualify the array itself: its length,
its capacity, and its storage. The two are independent, so an array can be
fixed in shape while its elements change, or the other way around:

```zax
roster : MyMessage mutable[8] immutable      // the array can't grow, shrink, or resize
roster[0] = fresh()                          // fine: the elements are mutable
roster.resize(9)                             // error: the array is immutable

journal : MyMessage immutable[0..] mutable   // entries can't change once added
journal <+>= [ entry ]                       // fine: the array can grow
journal[0] = fresh()                         // error: the element is immutable
```

If you come from C++, note the difference: a `const std::array` makes its
elements const too, but an `immutable` Zax array does not. To make both
immutable, write `immutable` on both layers:

```zax
fixedTable : Integer immutable[4] immutable = [ 1, 2, 3, 4 ]
fixedTable[0] = 10       // error: the element is immutable
fixedTable.resize(2)     // error: the array is immutable
```

An element left unqualified takes the same defaults as any other declaration,
as [qualifier defaults](qualifiers.md#defaults-and-resolution) describes. An
immutable array keeps its starting length for its whole lifetime, so an
immutable array declared empty, such as `none : Integer[0..] immutable = []`,
stays empty.

The qualifiers after `]` also control two separate things: whether the
current array can change, and whether the whole array can be replaced by a new
one. Here the array is immutable, so it can't be resized, but its place is
`varying` and the access path is `writable`, so `.=` may replace the whole
array:

```zax
myArray varying :
  Integer[4 in 4..10] immutable writable varying = [ 1, 2, 3, 4 ]

myArray.resize(6)    // error: resizing changes the current immutable array
myArray .= replacement
// fine when replacement satisfies the array and storage contracts:
// the old array ends and a new one is built in the same place
```

`writable` matters here because it allows replacing the whole array. It does
not make the elements writable; their own qualifiers decide that.

An array declaration can also carry a storage clause and a stance. They come
in this order: the dimensions, then the storage clause, then the array's
qualifiers, then its stance, which is always last on its layer:

```zax
pool : MyMessage move[0..] storage type MyStorage immutable move
```

Storage clauses are explained under [storage strategies](#storage-strategies),
and stances under [stances on elements and slices](#stances-on-elements-and-slices).

## Slice qualification and narrowing

Array and slice syntax can qualify the viewed elements separately from the
slice value:

```text
T <element/type qualifiers> [] <slice qualifiers>
```

Words before `[]` describe access to the elements. Words after `[]` describe
the slice descriptor—its current start, length, origin, and validity state.
The descriptor has qualifiers but no stance: a slice can always be copied,
and copying it copies only the window. What may be built from the elements is
the subject of [stances on elements and slices](#stances-on-elements-and-slices).

Creating a slice may keep or reduce the access the source gives. It cannot make a
readonly element writable, make immutable state mutable, or extend an element's
lifetime.

Copying a slice copies only its descriptor:

```zax
original := values[1..<5]
copy := original
copy = copy[1..<3]
// original is unchanged.
```

A reference to the descriptor changes the original slice:

```zax
alias : Integer[] & = original
alias = alias[1..<3]
// original now describes this narrower window.
```

The replacement must be an equal or narrower view with the same array origin.
It cannot widen the window, adopt a foreign array, revive an invalid slice, or
retain a view into an expiring temporary.

An element reference obtained before descriptor narrowing remains bound to its
original selected element.

## Stances on elements and slices

A [transfer stance](transfer-stances.md) says what may be built from a value:
a copy, a new value that takes its contents, or nothing at all. An array
holds its elements, so, like an optional, it can carry two stances. A slice
holds none of its elements, so the only stance it carries is the stance of the
elements it shows.

### A stance on each element

A stance written before `[` belongs to each element. A stance written after
`]` belongs to the array itself:

```zax
consume final : ()(message : MyMessage) = {
}

messages : MyMessage move[4] move    // each element: move; the array itself: move

consume(messages[0])                 // error: the element is move; write as move or as last
consume(messages[0] as move)         // fine: the array's owner may move an element out
other := messages                    // error: the array is move; write as move or as last
other := messages as move            // fine: moves the array, and so moves each element
```

Elements behave like members of the array. Each element follows its own
stance, and the array's owner writes stances on elements just as it writes
them on members. The array's own stance covers only transfers of the whole
array, so pinning an array does not pin its elements:

```zax
queue : MyMessage[0..] loan          // a pinned array of copy elements
consume(queue[0] as move)            // fine: the element is copy, so its owner may move it
other := queue as move               // error: the array itself is pinned
```

Copying or moving a whole array copies or moves every element, so the element
stance limits which stances the array can have:

| Element stance | Stances the array may have |
| --- | --- |
| `copy` | any |
| `move` | `loan`, `move`, `last` |
| `last` | `loan`, `last` |
| `deep` | `loan`, `deep` |
| `loan` | `loan` |

When you leave the array's own stance out, it is the same as the element
stance, which the table always allows:

```zax
messages : MyMessage move[4]         // the array is move too
plain : MyMessage[4]                 // no element stance: copy elements and a copy array
pinned : MyDevice loan[4]            // the array is pinned too
```

This also means aliases and generics work without specifying explicit
defaults. A stance that arrives through an alias or a type parameter applies
to the element:

```zax
OwnedPayload :: alias type Payload move
batch : OwnedPayload[4]              // move elements, so a move array

MyHolder$ :: type (T : type) {
  items : T[4]                       // valid for every T, whatever stance it carries
}
```

### A new array starts with unmarked elements

A stance restricts what may be built from one place, not every value that
later comes from it. So a new array built from `messages` does not inherit
its element stance, even when its elements were moved in:

```zax
messages : MyMessage move[4]
moved := messages as move            // each element of moved is built by moving
// moved is a MyMessage[4]: its elements are unmarked, so copy

kept : MyMessage move[4] = messages as move   // write the stance to keep the restriction
```

If every value of a kind must be move-only wherever it is stored, put that on
the type, with a `transfer` list, as
[transfer lists on types and identities](transfer-stances.md#transfer-lists-on-types-and-identities)
explains.

### A slice has no stance of its own

A slice reaches elements that an array owns. Each element it shows is reached
as if through a `<stance> &` reference, and the stance written before `[]`
says what the slice was given for each element. The ordinary rules for
[references](transfer-stances.md#owners-and-borrowers) apply: a slice may pass
on only the stance it was given.

```zax
inspectSome final : ()(values : MyMessage readonly loan[]) = {
  show(values[0])                    // fine: lends the element
  consume(values[0])                 // error: nothing may be built from a loan element
}

drainSome final : ()(values : MyMessage mutable writable move[]) = {
  consume(values[0] as move)         // fine: the slice was given move for each element
  consume(values[0] as last)         // error: a slice given move may pass on only move
}
```

A slice parameter, like a reference parameter, must write its element stance.
Even a slice of plain integers needs one word:

```zax
total final : (result : Integer)(values : Integer[]) = {   // error: write an element stance
}

total final : (result : Integer)(values : Integer loan[]) = {
  // fine: total only reads the elements
}
```

A slice declared from one visible array takes its element stance from that
array's elements, limited to stances that leave the elements unchanged, as
for [references you declare](transfer-stances.md#references-you-declare):

```zax
messages : MyMessage move[8] move
window := messages[1..<4]            // the elements are move, so window gets loan:
                                     // it can use them but can't move their contents out
drained : MyMessage move[] = messages[..] as move   // the owner gives move for each element
```

A stance written on a slice with `as` applies to its elements, because the
slice has no stance of its own. Writing `as last` on a slice that was given
`last` passes `last` on for each element; it does not finish the window. Assigning
or passing a slice by value only copies its window, so there is no separate
way, or need, to move a slice.

A reference to a slice is written the same way, with the element stance
before `[]`. Only the descriptor's qualifiers may follow `[]`:

```zax
narrow final : ()(values : Integer loan[] &) = {
  values = values[1..]               // narrows the caller's slice
}

checksum final : (result : U32)(data : Byte readonly loan[] readonly &) = {
  // the elements are read-only, and the window can't be narrowed
}
```

### A slice given `last`

The rules of a `last &` reference apply to every element of a slice given
`last`. The slice may lend the elements or pass `last` on. It may not copy
them, because two holders of `last` for the same element could both finish
it:

```zax
whole : MyMessage last[] = messages[..] as last

peek : MyMessage readonly loan[] = whole      // fine: whole is lent, not given up
twin : MyMessage last[] = whole               // error: whole may only lend or pass last on

keep final : ()(message : MyMessage) = {
}

keep(whole[0])                                // error: whole holds last, not copy
keep(whole[0] as last)                        // fine: the element is passed on with last
```

To get a copy of an element from a `last` slice, pass the element on with
`as last` to code that owns the result, such as `keep` above; that code may
then copy it, because its parameter is its own value. Giving `last` to
elements that stay in place leaves them finished inside their array, which
[finishing elements in place](#finishing-elements-in-place) explains.

## Slice lifetime and invalidation

A slice remembers the elements selected when it was created. It does not mean
“whatever values later occupy these numeric indexes.”

```zax
values : Integer[0..] = [ 10, 20, 30, 40, 50 ]
view := values[0..3]

values[4..<4] = [ 99 ]
// view survives only when this insertion cannot move existing storage.

values[2..<2] = [ 77 ]
// view is invalidated because insertion changes part of its viewed region.
```

The general rules are:

- An operation that may move the backing storage invalidates every slice unless
  its contract proves preservation.
- Insertion or removal invalidates slices that intersect or follow the first
  changed position.
- A slice entirely before that position may survive when storage cannot move.
- Insertion after a slice preserves it when storage cannot move.

An invalidated slice does not become empty, retarget new values, or become valid
again after later growth.

When invalidation is known from source, later use is a compile-time error.
Otherwise checked use may need origin and validity information and can panic.
Tooling must make that check cost visible. Disabling the corresponding panic
category transfers responsibility to the programmer; a false validity promise
has undefined consequences.

## Indexing element places

For current length `L`, valid indexes are `0` through `L - 1`.

```zax
copied := values[index]
direct : Integer writable & = values[index]
values[index] = replacement
values[index] ^= mask
```

The stable position inside an array that can contain an element is called an
**element place**. Intrinsic indexing selects that place. A value destination
constructs from it; a reference destination keeps access to it.

A constant invalid index is a compile-time error. A runtime required index
checks the current length and panics when invalid.

### Evaluation during indexed mutation

For:

```zax
values[chooseIndex()] = makeReplacement()
```

Zax evaluates and binds:

1. `values`;
2. `chooseIndex()`; and
3. `makeReplacement()`.

It then invokes the direct indexed-mutation operation, checks the runtime bounds,
and changes the element or enters panic. The replacement expression therefore
runs before a runtime bounds panic. A statically invalid operation executes
nothing.

Chained multidimensional indexing resolves the inner array first:

```zax
matrix[row][column] = replacement
```

The row selection completes before `column` is evaluated.

## Direct splice mutation

Using a slice on the right of `:=` or `=` normally produces a passive view:

```zax
view := values[2..4]
```

When a range appears on the left of assignment, the complete source form is one
direct structural operation:

```zax
values : Integer[0..] = [ 10, 20, 30, 40, 50 ]

values[3..<3] = [ 5, 10, 15 ] // insert at boundary 3
values[2..4] = []             // remove indexes 2 through 4
values[2..4] = [ 20, 30 ]     // replace three elements with two
```

The operation uses the array, range, assignment component, and replacement
together. It does not first create a slice and then assign through it.

Assignment to a saved slice therefore does not insert into the array:

```zax
insertionPoint := values[3..<3]
insertionPoint = [ 5, 10, 15 ]
// error: an array value cannot replace this slice descriptor
```

A fixed array can replace only with exactly as many elements as the range
contains. Insertion, removal, and unequal replacement require a resizable array,
and the resulting length must remain within its declared limits.

### Removing a saved slice

```zax
selected := values[2..4]
values.remove(selected)
```

`remove` confirms that the slice is still valid and came from this exact array,
then removes the represented interval.

```zax
firstArray : Integer[0..] = [ 1, 2, 3, 4, 5 ]
secondArray : Integer[0..] = [ 6, 7, 8, 9, 10 ]

selected := firstArray[2..<5]
secondArray.remove(selected)
// error: selected belongs to a different array
```

When the different origin is known, this is a compile-time error. Otherwise the
operation checks before mutation and panics on mismatch. It never searches for
equal values or applies the foreign numeric indexes to `secondArray`.

A slice of a slice retains the original array origin. Removing an empty
same-array slice does nothing.

### Overlapping and self-slice input

```zax
values <+>= values[2..4]
values[3..<3] = values[5..6]
```

Both operations use the source values as they existed before structural
mutation. Appending a view of the same array duplicates those values, so their
element type must support `copy`.

In general, an overlapping bulk mutation behaves as if source values needed by
later writes were saved before the first conflicting write. A separate
directional operation may offer a cheaper explicit contract.

## Taking elements out with `take`

Direct splice mutation can remove elements, but it destroys them. To hand
elements on while removing them, write `take` before the array and range:

```zax
messages : MyMessage[0..] = [ a, b, c, d, e, f ]

batch := take messages[2..<5]        // batch is a new array holding c, d, e
messages.length()                    // 3: messages now holds a, b, f
```

`take` builds a new owning array from the range and removes that range from
the source at once. Each element is handed over with `last`, falling back to
`move` and then `copy`, chosen from what the element's type can do. The later
elements then shift down to close the gap. The source stays a normal, fully
usable array; it is just shorter.

Like direct splice mutation, `take messages[2..<5]` is one operation on the
array and the range together. It does not first make a slice and then take
from it.

### Passing taken elements on

`take` binds before `as`, so a stance written after it applies to the new
array. Handing the elements straight to a function usually wants `as last`,
and `last messages[2..<5]` is a shortcut for exactly that:

```zax
consumeAll final : ()(values : MyMessage last[]) = {
}

inbox : MyMessage[0..] = [ a, b, c, d, e, f, g, h ]
consumeAll(take inbox[2..<5] as last)    // the new array's elements are given last
consumeAll(last inbox[2..<5])            // the same, with the shortcut: takes what are now
                                         // elements 2, 3, and 4
```

Any stance may follow `take`, but the taken elements are destroyed when the
new array ends anyway, so `as last` gives the receiver the most freedom at no
extra cost.

### The type of the taken array

The new array's length range is every count the `take` could produce. This
follows the same rule as joining with `<+>`:

```zax
messages : MyMessage[0..8]
window := take messages[2..<5]       // MyMessage[3]: the range always holds three
tail := take messages[k..]           // MyMessage[0..8]: at most what messages can hold

bounded : MyMessage[3..8]
spare := take bounded[k..]           // MyMessage[0..5]: bounded must keep at least three
```

The new array gets its storage the way any new array does. When you write the
destination's type, its storage clause decides, and `take` builds directly
into it, with no second move. Passing the result to a by-value array parameter
works the same way: the parameter's declared storage decides. Otherwise the
new array uses the current [`___` execution context](execution-context.md)'s
default. It never uses the source's provider implicitly, because that
provider may serve only the source; to share it, name it in the destination's
type with `storage provider`.

```zax
kept : MyMessage[0..] storage type MyStorage = take messages[2..<5]
                                     // built directly into a MyStorage provider
loose := take messages[0..<2]        // built with the context's default storage
```

### What `take` needs

- **A resizable array.** A fixed array cannot change length, so it cannot
  `take`.
- **Room above the minimum.** `take` cannot leave fewer elements than the
  array's declared minimum. When the count is known, that is a compile-time
  error; otherwise it is checked, and panics, as `resize` does.
- **Elements that may leave.** `take` hands each element over with `last`, so
  the element stance must allow that. Pinned elements allow nothing to be
  built from them, and `deep` elements allow only independent copies, so
  neither can be taken. To move `deep` elements elsewhere, copy them with
  `as deep` and then remove them.
- **Write access to the array.** `take` changes the array's length, so it
  needs the same `mutable writable` access as `values[2..4] = []`. The array's
  own stance is not involved.

```zax
shortlist : MyMessage[3..8] = [ a, b, c, d, e ]
consumeAll(take shortlist[0..<2] as last)    // fine: three elements remain
consumeAll(take shortlist[..] as last)       // error: would leave none, below the minimum of three

devices : MyDevice loan[0..]
take devices[0..<1]                       // error: nothing may be built from a pinned element
```

`take` through a writable reference to someone else's array needs nothing
written at the call, as `resize` and direct splice removal need nothing: the
caller's array is simply shorter afterward, and nothing finished is left in
it.

### `take` on a slice

On a slice, `take` shrinks the slice instead of the array. The elements stay
where they are; the result is a shorter slice holding the stance the source
held for those elements, and the source no longer covers them. Writing
`as last` passes that stance on to the new slice:

```zax
letters : MyMessage[8]
whole : MyMessage last[] = letters[..] as last    // covers elements 0..<8 of letters
front := take whole[..<2] as last        // front covers 0..<2; whole now covers 2..<8
back := take whole[4..] as last          // back covers 6..<8; whole now covers 2..<6
rest := take whole[..] as last           // rest covers 2..<6; whole is now empty
```

The comments name positions in the underlying array. Indexes written on a
slice count from the slice's own start, so after the first `take`,
`whole[4..]` begins at the array's element 6.

A slice is one window, so it can give up only a range from its start, a range
to its end, or everything. The range must be written in one of those shapes,
`[..<n]`, `[..n]`, `[n..]`, or `[..]`, so you and the compiler can both see
which end it comes from:

```zax
middle := take whole[1..<2] as last      // error: a slice can only give up a range
                                         // from its start or to its end
part := take whole[a..<b] as last        // error, even if a is 0 at runtime: write [..<b]
```

For a middle piece, take twice: first the front part, then the piece you want
from what remains. Each take on a slice costs only a little arithmetic on the
window, so splitting a slice several ways is cheap:

```zax
partA := take whole[..<4] as last
partB := take whole[..<4] as last
partC := take whole[..] as last
```

Without `as last`, the new slice is bound like any slice taken from a holder of
`last`, so it gets `loan` and may only look at the elements. That skips them:
they are not finished, but `whole` no longer covers them and the new slice
can't finish them, so they stay as they are until the array's destructor
destroys them:

```zax
skipped := take whole[..<1]              // skipped may only look at that element
```

This is the one place `take` behaves differently on arrays and slices with no
stance. On an array, `take` always builds a new array that owns the
elements; on a slice, it can only hand on the stance the slice was given.

`last whole[..<4]` is the shortcut for `take whole[..<4] as last` on a slice
too. On a slice given only `copy` or `move`, it is an error, because that
slice was never given `last`.

### Costs

- Taking from an array moves each taken element into the new array, then
  shifts the later elements down. Taking from the end shifts nothing.
- Taking from a slice moves nothing.

For an array whose minimum is zero, `take messages[..]` and
`messages as move` both leave `messages` empty, but they differ. `take`
builds a new array and moves each element into it, and `messages` keeps its
storage and capacity. `as move` transfers the whole array, and an array whose
storage comes from a [storage provider](#storage-strategies) may hand its
storage block over without touching any element.

## Growing, shrinking, and resizing

These required callables return no result:

```zax
values.grow(newLength)
values.shrink(newLength)
values.resize(newLength)
```

- `grow` does nothing when `newLength` is not larger. Otherwise it
  default-constructs the new suffix.
- `shrink` does nothing when `newLength` is not smaller. Otherwise it destroys
  the removed suffix in reverse order.
- `resize` applies the corresponding behavior.

The new length must satisfy both the current access path and the underlying
array's actual size limits. If new elements cannot be default-constructed,
`grow` and the growing path of `resize` are unavailable.

Splice insertion supplies nondefault new elements:

```zax
values[values.length()..<values.length()] = incoming
```

## Element-place stability

References and slices remain useful only while their selected element places
continue to exist.

The ordinary rules are:

- Changing an element's contents preserves its place.
- Replacing one complete element may begin a new element lifetime in the same
  place.
- Growing at the end within existing capacity preserves old places and creates
  new ones.
- Moving to new backing storage relocates every element and invalidates access
  to the old places.
- Shrinking ends the removed suffix and preserves the earlier prefix.
- Insertion preserves an unaffected prefix at most and moves or replaces later
  elements.
- Removal ends selected elements and may move or replace later elements.
- Changing capacity removes no logical element but may relocate every element.

An unchanged index is not enough to prove that the old element still exists
there.

The array and storage provider determine invalidation together. A checked slice
may retain a stability kind/token and the version observed when it was created.
When storage reports that a relevant chunk or mapping changed, the array
advances the corresponding version. Later slice use compares that information
and panics after invalidation.

The exact token representation remains future storage-contract work. A slice
covering several chunks may need a composite token or conservative whole-array
invalidity. The programmer-visible requirement is that storage cannot claim an
ended or moved element place remained valid, and stronger stability must come
from an exposed provider capability.

Moving an element into unused fresh storage uses construction. The old element
is passed on with `last`, so `last`, then `move`, then `copy` is used,
whichever the element type has first. Shifting into an
already live destination uses ordinary assignment or protected `.=` complete
reconstruction as required by the selected element contract.

If an element type cannot perform the required construction, transfer, or
replacement, the array operation is unavailable.

Arrays sometimes relocate elements so they can grow and shrink. The most
efficient way to relocate an element is to pass it on with `last`, so the
array relocates elements with `last`, falling back to `move` and then `copy`
when the element's type has no `last` or the element's qualifiers rule it out.
A stance declared on the elements doesn't change this, except a `loan` stance,
explained below:

```zax
archives : MyArchive deep[0..]
archives.reserve(64)            // fine: relocation uses MyArchive's own last, move, or copy
keep(archives[0])               // error: your code may build only independent copies

frozen : MyMessage immutable[0..]
frozen <+>= [ makeMessage() ]   // if this relocates, each element is copied:
                                // last and move would have to change an immutable element
```

The one exception is a pinned element. A `loan` stance promises that the
element's address never changes, and other code may rely on that, so any
operation that could relocate pinned elements is an error, unless the array's
storage provider offers a capability that guarantees stable element places.
How an array requests such a capability is not settled yet.
Using pinned elements is always fine; only relocating them is refused:

```zax
devices : MyDevice loan[0..]
devices[0].start()              // fine: nothing moves
devices <+>= [ makeDevice() ]   // error: growing may relocate pinned elements

fixedDevices : MyDevice loan[4] // a fixed array never grows, shrinks, or relocates,
                                // so pinned elements are a natural fit
```

## Storage strategies

An array keeps its elements in one of two ways:

- **Inline storage:** the elements live inside the array value itself, the
  way a C array sits inside a struct or on the stack. Such an array cannot
  hand its storage to another array, and cannot grow past the room built into
  it, so a resizable inline array needs a fixed maximum, such as
  `MyMessage[0..8]`.
- **Provider-backed storage:** the array value holds a handle to a block of
  storage that a separate **storage provider**, such as an arena, manages for
  it. The array can hand the whole block to another array, and grows by asking
  the provider for more. When a provider-backed array names no provider, the
  current [`___` execution context](execution-context.md) may supply one,
  such as a default arena.

An array's **storage profile** records which of the two it uses and, for a
provider-backed array, which provider and capabilities. It affects what a
transfer of the whole array can do, as
[copy, deep, move, and last](#copy-deep-move-and-last) shows, and what an
array costs. An array with no maximum, such as `Integer[0..]`, is always
provider-backed, because inline storage needs a fixed amount of room. Which
profile a fixed or bounded array gets when no storage clause is written is not
yet settled; write a storage clause when the difference matters.

An array keeps the same logical behavior while different storage providers
organize its raw backing:

```zax
provider : MyStorage * strong

byDefault : Integer[50]
byInstance : Integer[50] storage provider
byType : Integer[50] storage type MyStorage
```

`byDefault` uses the storage profile selected by its declaration context.
`byInstance` must use that exact provider instance. `byType` obtains or creates
a provider satisfying `MyStorage`'s default/context contract.

Storage does not change the logical identity of an array, but it may change the
owning value's self-contained representation, available operations, cost, and
physical layout.

One storage clause follows the complete dimension list and governs the entire
multidimensional array. A type alias may retain a provider type requirement:

```zax
MyFlatArray :: alias type Integer[4][3] storage type Flat

MySpecificArray :: alias type
  Integer[4][3] storage provider type MyStorage
// error: a type alias cannot capture a runtime provider instance
```

Selecting storage independently for each dimension is unavailable. A single
provider/handle maps the complete array shape and coordinates all row and
element storage.

The storage clause follows the complete dimension list. `storage` introduces a
provider instance expression; `storage type` introduces a provider type.

### Who owns the storage

One provider may serve many arrays. Each array keeps:

- strong participation that keeps the provider alive; and
- one unique handle for that array's independently owned backing region.

The provider owns raw bytes, slot/chunk organization, gross alignment,
physical lookup, physical extent, and storage stability information. It does
not know the Zax element type or whether one slot contains a live element.

The array owns logical bounds and knows which slots hold live values. It runs
element construction, transfer, replacement, and destruction.

For indexing, the array checks the logical position and then asks the provider
handle to locate a suitably sized/aligned raw slot or chunk. For relocation,
the provider supplies fresh destinations or a relocation plan; the array moves
the actual elements. A provider never blindly copies unknown live values as
raw bytes.

The exact provider/handle protocol is deferred to dedicated future storage work.

### Shared and dedicated providers

Supplying a provider instance allows several arrays to share the storage
service while retaining separate unique backing handles.

A storage type may provide a receiverless default-provider factory:

```zax
MyStorage :: type {
  defaultStorage final : (
    result : MyStorage * strong
  )() unbound = {
    return MyStorage.sharedStorage
  }

  // Illustrative future once-value syntax.
  sharedStorage once : MyStorage * strong
}
```

`unbound` lets the factory be called through `MyStorage` without an instance.
`once` on a function would not memoize its result. Persistent shared state
belongs to the future `once` value, the execution context, or another explicit
lifetime owner.

If no shared-provider contract exists, a storage type may obtain or construct a
dedicated provider. The applicable cost must remain visible.

### Provider-specific and erased access

When `storage type MyStorage` is present, code may use safe provider-specific
operations under the provider and handle's declared qualifications.

When the provider type has been erased, ordinary array operations remain
available through the common storage contract. The array may expose an unsafe
raw pointer to the provider object for a programmer who asserts its concrete
type. That pointer:

- does not point to array elements;
- owns no provider lifetime;
- gives no contiguity guarantee; and
- uses ordinary pointer
  [`unsafe cast`](casting.md#unchecked-reinterpretation-with-unsafe-cast)
  responsibility.

A discontiguous provider has no universal raw element pointer. Unsafe source
cannot manufacture one.

### Contiguous storage

`Flat` can be one concrete storage provider:

```zax
values : Sample[500] storage type Flat
```

Its contract can guarantee one contiguous increasing-index element region. A
resizable flat provider may use existing reserved slots or relocate every
element into a larger region.

Other provider types may offer the same contiguous capability. A future
storage-capability requirement should let an API request contiguity without
requiring the exact `Flat` type.

### Multidimensional storage

Dimension order still matches chained indexing:

```zax
matrix : Integer[4][3] storage type Flat
element := matrix[row][column]
```

When `Flat` accepts this shape, its dense multidimensional contract is
rectangular and row-major: neighboring column values are adjacent and the
rightmost dimension changes fastest.

Another provider may support jagged rows:

```zax
rows : Integer[4][1..10] storage type MyJaggedStorage
selectedColumns := rows[row][1..<3]
```

A provider may reject a shape it cannot represent. A padded rectangular
provider may support varying row lengths without claiming that all live values
form one dense flat sequence.

Every bracket supplies one index or one sliced dimension. Multidimensional
access therefore chains brackets as `matrix[row][column]`. Rectangular and
strided block views remain future work.

Slicing the outer dimension preserves the inner row type:

```zax
myValues : Integer[5][6]

rowsView := myValues[..]
// Integer[][6]: a slice of fixed six-element rows.

rowView := myValues[0][..]
// Integer[]: a slice of Integer elements in one row.
```

`Integer[][]` would be an outer slice whose elements are themselves slices.
`myValues[..]` does not have that type because its row elements remain fixed
`Integer[6]` arrays.

### Changing storage

A future explicit storage-change operation obtains suitable target slots,
transfers elements in index order, ends old element places, releases the old
unique handle, and publishes the new provider only after completion.

The transition invalidates element references, slices, pointers, and cursors
unless a stronger stability contract proves continuity.

## Copy, deep, move, and last

Copying an owning array creates another array with independent element places
and backing storage. Each element uses its ordinary `copy`, which may still
share internal resources according to that element type's contract.

`deep` also creates independent array storage and requires every element to
support its exact deep-copy contract. It never falls back to ordinary copy.

### What `move` leaves behind

`moved := recent as move` builds a new array and moves every element of
`recent` into it. Afterward, `recent` has its declared minimum length, and each element it
keeps is moved-from. Elements above the minimum are destroyed, from the
highest index down, as a shrink destroys them. The source keeps its storage,
and with it its capacity, unless it hands its whole storage block over to the
new array, as described below.

```zax
recent : MyMessage[0..8]           // minimum 0
recent <+>= [ a, b, c, d, e ]
moved := recent as move            // moved gets all five
recent.length()                    // 0: the minimum is 0, so nothing is kept

window : MyMessage[3..8]           // minimum 3
window <+>= [ a, b, c, d, e ]
moved2 := window as move           // moved2 gets all five
window.length()                    // 3: elements 0, 1, and 2 are kept, moved-from;
                                   // elements 3 and 4 are destroyed

pair : MyMessage[4]                // fixed: the minimum and maximum are both 4
moved3 := pair as move             // moved3 gets all four
pair.length()                      // 4: every element is kept, moved-from
```

A fixed array is simply the case where the minimum is the maximum, so nothing
is destroyed. On a resizable array, `move` runs one destructor for each
element above the minimum.

An `immutable` array can't change length, so `move` leaves it at its current
length instead of its minimum. Its elements are left moved-from when they are
mutable. When the elements are immutable too, nothing can be moved out of
them, so `move` falls back to copying, as it does for any immutable value:

```zax
frozenList : MyMessage[0..] immutable = [ a, b, c ]
moved := frozenList as move        // moved gets all three
frozenList.length()                // 3: an immutable array keeps its length;
                                   // its mutable elements are now moved-from

sealed : MyMessage immutable[0..] immutable = [ a, b, c ]
copied := sealed as move           // copies: immutable elements can't be moved from
```

How the elements get to the new array depends on the
[storage profile](#storage-strategies):

- An inline array cannot hand its storage over, so `move` moves each element
  into the new array's storage.
- A provider-backed array may hand its whole storage block over, without
  touching any element, only when its minimum is 0: the source is then left
  with no elements and no block. With a minimum above 0, the source still
  needs storage for the elements it keeps, so each element is moved instead,
  and the source keeps its block. Getting a fresh block for the minimum would
  hide an allocation inside `move`.

### What `last` leaves behind

After `last`, the source only has to be safe to destroy, not usable, so it
needn't keep any elements or storage. A provider-backed array may therefore
always hand its block over on `last`, even a fixed one. An inline array still passes each
element on, because inline storage cannot be handed over.

### Passing a slice on

Assigning or passing a slice copies only its window. It never copies or moves
the elements, and it never hands over the array's storage; only the array
that owns the storage can do that. A stance written on a slice applies to its
elements, as
[stances on elements and slices](#stances-on-elements-and-slices) explains.
To hand a range of elements on and remove it from the array, use
[`take`](#taking-elements-out-with-take).

## Finishing elements in place

Writing `last` on an element, or on a range of elements, that stays inside its
array leaves those elements finished: they may only be destroyed or replaced.
The array itself is not finished. It just holds elements that later code must
not use, so it becomes **partially finished**. The same applies to a slice
whose elements were given `last`.

A partially finished array:

- does not survive as a whole: copying it, moving it, passing it whole, and
  ordinary iteration over it are errors;
- still answers questions about itself that don't touch any element, such as
  `length()` and its capacity, so a loop can still count its elements;
- may still be destroyed or replaced, like any finished value;
- allows a use of one element (reading it, lending it, or writing `last` on
  it) only when the compiler can prove that element was not finished.

Writing `last` on elements of an array whose elements are all unfinished is
always allowed. The error appears at a later use that the compiler cannot
prove safe:

```zax
consumeAll final : ()(values : MyMessage last[]) = {
}

messages : MyMessage[0..]
consumeAll(messages[2..<5] as last)  // fine: messages is now partially finished
messages[0].send()                   // fine: 0 is outside 2..<5
other := messages                    // error: messages is partially finished
```

With indexes known only at runtime, the compiler usually can't tell which
elements were finished:

```zax
inbox : MyMessage[0..]
dispatch(inbox[i] as last)           // fine: every element was unfinished
dispatch(inbox[j] as last)           // error: the compiler can't prove j differs from i
```

Constant indexes are different, because the compiler can see them:

```zax
pair : MyResource[2]
first := pair[0] as last             // fine
second := pair[1] as last            // fine: a different element
pair[0].close()                      // error: pair[0] was finished
```

To keep the array whole, use [`take`](#taking-elements-out-with-take)
instead. It removes the elements rather than leaving finished ones behind.

### What the compiler can prove

Every compiler accepts the same programs that finish elements in place,
because the language lists the cases every compiler must prove:

1. **Constant indexes:** `pair[0]` and `pair[1]` are different elements.
2. **Constant ranges:** `messages[0]` is outside `messages[2..<5]`.
3. **The finishing loop:** `each message : in messages as last` finishes each
   element exactly once.
4. **A plain index loop:** the loop variable is the only index used on the
   array, the loop doesn't change it, and each pass touches only
   `messages[i]`. After such a loop, every element it covered counts as
   possibly finished, unless every pass ends by replacing that element, as
   [replacing finished elements](#replacing-finished-elements) shows.

```zax
outbox : MyMessage[0..]
each i : in 0..<outbox.length() {
  dispatch(outbox[i] as last)        // fine: each pass has a different i
}

queue : MyMessage[0..]
each i : in 0..<queue.length() {
  if urgent(queue[i]) {
    dispatch(queue[i] as last)       // element i may now be finished
  }
}
queue[0].send()                      // error: queue[0] may have been finished in the loop
```

A use outside these cases needs an `unsafe` block, even when your compiler
can prove it safe, so the program compiles the same way everywhere; in the
block, you take responsibility for the use, as
[safety and analysis](safety-and-analysis.md#unproved-but-valid) describes. A
compiler that can prove the block safe can confirm it with a `[<safe=…>]`
directive naming that compiler, which records that the `unsafe` was checked
rather than only asserted. The directive's exact form is still being
designed.

### Replacing finished elements

A finished element can be replaced in place with `.=`. Once every finished
element is proven replaced, or gone, the whole array is usable again:

```zax
messages : MyMessage[0..]
each i : in 0..<messages.length() {
  dispatch(messages[i] as last)       // element i is finished
  messages[i] .= makeMessage()        // replaced in place: element i is live again
}
send(messages)                        // fine: every pass ended by replacing its element
```

Replacement must build a new element. Ordinary assignment with `=` runs the
existing element's assignment, which needs a live element, and a finished
element may only be destroyed or replaced:

```zax
batch : MyMessage[0..]
dispatch(batch[3] as last)
batch[3] = fresh()                    // error: = assigns into a finished element
batch[3] .= fresh()                   // fine: ends the finished element and builds a new one
```

Two other ways to replace finished elements:

```zax
consumeAll(batch[2..<5] as last)
batch[2..<5] = [ a, b, c ]            // a direct splice destroys the finished elements and
                                      // builds new ones: the range is live again
batch .= freshArray()                 // replacing the whole array ends every old element
```

A function that was given `last` and replaces elements in its own body does
not restore the caller's view: the caller still treats the elements it gave
up as finished.

### Finishing elements through a borrowed array

Finishing an element in place always needs the caller to write `last` at the
call, so anyone reading the call sees that elements may be finished. So
through a reference to someone else's array, a
function may move an element out, which leaves it usable, but may not finish
it, unless the caller gave up the whole array with `as last`:

```zax
finishFirst final : ()(values : MyMessage[0..] mutable writable loan &) = {
  consume(values[0] as move)              // fine: the element stays usable
  consume(values[0] as last)              // error: a borrowed array allows move, not last
  consumeAll(take values[0..<1] as last)  // fine: the element leaves the caller's array
}

finishAll final : ()(values : MyMessage[4] mutable writable last &) = {
  consume(values[0] as last)              // fine: the caller gave up the whole array
  consume(values[1] as last)
}

messages : MyMessage[0..]
finishFirst(messages)                     // nothing written: messages may be shorter afterward

fixed : MyMessage[4]
finishAll(fixed as last)                  // the caller writes last on the whole array
```

Members follow a looser rule, including members of an element: a function
with writable access may finish a member with `as last`. A finished member
sits inside a value whose type defines how it is copied, replaced, and
destroyed, and those operations must already cope with a finished member. A
finished element would sit inside an array, and the language itself
implements copying, iterating, and destroying an array; those operations
expect every element to be live.
[Members](transfer-stances.md#members) explains the member rule.

## Joining arrays

`<+>` constructs a new owning array:

```zax
first : Integer[3] = [ 1, 2, 3 ]
second : Integer[2] = [ 4, 5 ]

joined := first <+> second
// Integer[5]
```

`<+>=` appends a sequence to an existing resizable array:

```zax
values : Integer[0..] = [ 1, 2 ]
incoming : Integer[2] = [ 3, 4 ]
oneMore : Integer = 5

values <+>= incoming
values <+>= [ oneMore ]
```

Arrays do not provide intrinsic `+` concatenation.

Ordinary `<+>` preserves both inputs by copying their elements. Each operand
is an ordinary source, so its own declared stance applies, and an explicit
stance may permit reuse:

```zax
left : MyMessage[0..] move
right : MyMessage[0..]

joined := left <+> right            // error: no stance means copy, and left is move
reused := left as last <+> right    // fine: the result may reuse left's storage
```

With `as last`, or `as move` when its minimum length is zero, a compatible
provider-backed left array may hand its whole storage block to the result. Otherwise the result gets new
storage the way any new array does: from the storage clause of the
destination's type when one is written, and from the `___` context's default
when not. It never takes new storage from the left array's provider
implicitly, because that provider may serve only the left array. Suggested
capacity does not propagate to the new array.

The current result length is the checked sum of input lengths. Static bounds add
in the same way:

```text
Integer[3] <+> Integer[2]       -> Integer[5]
Integer[1..4] <+> Integer[2..<5] -> Integer[3..8]
```

An open input maximum produces an open result maximum. If the current sum
cannot fit `IndexSize`, the operation panics.

The left input supplies the result element type. A slice left has no storage
block to hand over, so the result always gets new storage, as described
above.

`<+>=` retains the left array's type, so its new length must fit the existing
limits. A fixed array has no mutating join when the source might contribute an
element.

## Array and slice parameters

### Passing an owning array by value

```zax
consume final : ()(
  values : Integer[2..10]
) = {
}
```

The call creates an independent local array. The source must currently contain
between two and ten elements. A known impossible size is a compile-time error;
otherwise the call checks the current length and panics when it does not fit.

The source's `copy`, `deep`, `move`, or `last` stance controls how the local
array is constructed. With `as last`, or `as move` when its minimum length is
zero, a provider-backed source may hand its whole storage block over, when the
parameter can use storage from that provider. Otherwise, with no storage requirement on the parameter, the
local array gets new storage from the `___` context's default, never
implicitly from the source's provider. Once constructed, the local array may
resize within its own declared limits.

A by-value parameter naming a provider instance constructs into that provider.
A parameter naming only `storage type MyStorage` obtains a provider of that
type. Either can accept a source array using another storage profile because a
new destination is being constructed.

### Passing an array by reference

An array reference without a storage requirement is storage-erased:

```zax
useAnyStorage final : ()(
  values : Integer[5] loan &
) = {
  values[2] = 55
}

provider : MyStorage * strong

inlineValues : Integer[5]
providerValues : Integer[5] storage provider
typedProviderValues : Integer[5] storage provider type MyStorage

useAnyStorage(inlineValues)
useAnyStorage(providerValues)
useAnyStorage(typedProviderValues)
```

All three calls use the same concrete implementation. The reference carries
enough common operation information to reach the source array, ask its storage
to locate element slots, and preserve logical bounds and qualifications.

Binding copies no elements and creates no second array. The erased reference
may be wider and its operations more indirect than a reference constrained to
one exact storage type. It guarantees ordinary array operations, not
provider-specific APIs or contiguous backing.

A storage-qualified reference requires the caller's existing array to use that
provider type or exact instance. A reference cannot obtain replacement storage
from the destination context.

Bounded references additionally retain source size capabilities:

```zax
change final : ()(
  values : Integer[0..10] loan &
) = {
  values.resize(10)
}

fixed : Integer[3]
change(fixed) // runtime panic when change attempts to resize to 10
```

The reference remembers both the parameter's allowed range and the source
array's actual capabilities.

At the call, `fixed` currently contains three elements, which fits `0..10`.
Inside `change`, attempting to resize it to ten still fails because the original
array is fixed at three. That runtime failure panics. Writing
`fixed.resize(10)` directly is a compile-time error.

This adapting reference may need both storage operations and size-capability
metadata.

Different parameter ranges can borrow the same resizable source:

```zax
func1 final : ()(values : Integer[100] loan &) = {
}

func2 final : ()(values : Integer[..100] loan &) = {
}

func3 final : ()(values : Integer[50..200] loan &) = {
}

func4 final : ()(values : Integer[..] loan &) = {
}

myValues : Integer[100..]
// Initial length is 100.

func1(myValues)
func2(myValues)
func3(myValues)
func4(myValues)
```

All four calls pass while `myValues.length()` is 100. If it later grows to 150,
only `func3` and `func4` pass their entry checks.

A writable parameter operation must satisfy both the parameter's visible range
and the source array's actual range. For example, `func3` can request a length
from 50 through 200, but this source still panics on a request below its own
minimum of 100.

A readonly parameter cannot resize the source, so after its entry-length check
it needs no size-changing capability. Ordinary writable references retain the
runtime checks; a future no-panic capability contract may impose stronger
static requirements.

### Element stances in an array reference

An element stance written inside a reference parameter says what the function
may do with the elements. The caller's elements must allow it, by the same
table as for passing a value you own to a reference:

| Caller's element stance | Element stances the parameter may name |
| --- | --- |
| `copy` | any |
| `move` | `loan`, `move`, `last` |
| `last` | `loan`, `last` |
| `deep` | `loan`, `deep` |
| `loan` | `loan` |

```zax
inspect final : ()(values : MyMessage loan[4] readonly loan &) = {
  show(values[0])                       // fine: lends the element
}

drain final : ()(values : MyMessage move[4] mutable writable loan &) = {
  consume(values[0] as move)            // fine: move elements, and writable access
}

copyAll final : ()(values : MyMessage[4] readonly loan &) = {
  keep(values[0])                       // fine: unmarked elements are copy
}

messages : MyMessage move[4] move
inspect(messages)                       // fine: move elements allow loan
drain(messages)                         // fine: move elements allow move
copyAll(messages)                       // error: move elements don't allow copy

plain : MyMessage[4]
copyAll(plain)                          // fine
```

An unmarked element stance means `copy`, so `values : Integer[5] loan &`
accepts any array whose elements allow copying. A parameter that names `loan`
elements accepts any array and only looks at it.

A reference to an array refers to the whole array, and the array's elements
are its members. So, as with any member, writable access decides whether the function
may move an element out, within what the element stance allows: `copy`
elements may be moved, because their owner could. A slice is different: it
reaches each element directly, so it may pass on only the stance it was
given. Through a writable array reference, the function may move elements out
but may not finish them, as
[finishing elements through a borrowed array](#finishing-elements-through-a-borrowed-array)
explains.

### Passing a slice

A slice parameter's declaration includes the stance the function needs for
each element:

```zax
inspect final : ()(
  values : Integer loan[]
) = {
}

narrow final : ()(
  values : Integer loan[] &
) = {
}
```

Passing a slice by value copies its window description, not its elements.
Passing it by reference lets the callee narrow the caller's slice descriptor.

Passing an array where a slice is expected creates a temporary slice for the
call. Narrowing that temporary does not change a separately named caller slice.
A stance written on the array at such a call applies to its elements, just as
a stance written on a reference argument applies to the value it refers to:

```zax
drainSome final : ()(values : MyMessage mutable writable move[]) = {
  consume(values[0] as move)
}

messages : MyMessage[8]
drainSome(messages)                   // error: the parameter needs move for each element
drainSome(messages as move)           // each element is given move; messages stays usable
drainSome(messages as last)           // each element is given last: messages is partially
                                      // finished, with every element given last
```

A slice is never converted into a new array implicitly. Building an array
copies or moves every element, and the source should show that:

```zax
keepAll final : ()(values : MyMessage[0..]) = {
}

window := messages[2..<5]
keepAll(window)                      // error: a slice is not an array
keepAll([ from window ])             // fine: builds a new array from the slice's elements
```

### When an array fits both an array and a slice parameter

If a function has one version that takes an array by value and another that
takes a slice, a call with an array and a stance fits both, and is ambiguous:

```zax
process final : ()(values : MyMessage[0..]) = {                      // builds a new array
}

process final : ()(values : MyMessage mutable writable move[]) = {   // borrows the elements
}

backlog : MyMessage[0..]
process(backlog as move)              // error: ambiguous
process(backlog[..] as move)          // the slice version: the argument is already a slice
process(backlog as copy)              // the array version: the slice version needs move
```

The by-value version matches `backlog`'s type exactly, and the slice version
matches the stance exactly, so neither is better; the general rule is in
[function invocation](function-invocation.md#transfer-aware-value-and-reference-parameters).
The two versions would also leave `backlog` in different states: the array
version moves the whole array, which leaves `backlog` empty, while the slice
version leaves its length unchanged and its elements moved-from.
If callers will need `as move` or `as last` with whole arrays, give the two
functions different names.

### Size of an array value and reference

These queries do not report the same number:

```zax
active size of Integer[5]
active size of Integer[5] &
```

The first reports the self-contained owning representation selected for creating
an `Integer[5]` value. Inline storage includes the elements; provider-backed
storage includes only the array's local provider ownership, unique handle,
bounds, and other metadata. External backing bytes are not counted.

The second reports the representation of the array reference itself. A
storage-erased reference may contain more than one machine pointer.

If reflection strips the reference layer, the resulting logical array type can
report the size of a newly created value under its resolved storage profile. It
cannot recover the actual owning representation erased by an already supplied
unconstrained reference.

Zax has no current full-storage size query. Provider-specific APIs may
report external physical use separately.

## Traversal

Fixed arrays, resizable arrays, and slices support direct `each in`:

```zax
each element : in values {
  inspect(element)
}
```

The source is evaluated once and elements are visited in increasing index
order. The default binding accesses each element directly. An explicitly
non-reference binding constructs a separate value for every pass.

Changing an element is allowed when its qualifications permit. An operation
that might change array length, positions, or backing invalidates direct
traversal instead of changing its remaining work.

A stance written on the source gives each element that stance. Writing
`as last` finishes each element exactly once, so it is the usual way to
consume a whole array:

```zax
each message : in messages as last {
  dispatch(message as last)
}
// messages is partially finished, with every element given last
```

`each from` remains available through cursors for controlled erasure, reverse or
filtered order, and specialized progression. Direct traversal does not secretly
create a cursor.

## Construction and destruction

Array elements construct in increasing index order. They are destroyed in
reverse positional order. Shrinking an array likewise destroys its removed
suffix from the highest index downward.

When all source counts are known, final backing storage is obtained before the
first destination element is constructed. An uncounted iterable may require
incremental growth; each storage expansion and relocation finishes before the
next element is constructed.

A panic blocks the incomplete operation. It does not return a partially
constructed array or unwind completed elements. A matching helper may repair
the condition and resume the same operation; otherwise the process crashes.

## Custom indexing and slicing

Intrinsic arrays select real element storage directly. A custom type can make
the same bracket shapes mean another operation and return another declared
type.

```zax
MySequence :: type {
  operator index final : (
    result : MyResult
  )(
    index : IndexSize
  ) readonly = {
  }
}
```

A custom index may return a value, a reference, or a helper object that later
performs access. Such a helper is called a **proxy**. Proxies are useful when no
ordinary addressable element exists—for example, one packed bit—but they require
their own type, lifetime, qualification, and cost rules.

Ordinary intrinsic arrays need no proxy for element mutation. A custom type can
instead define one direct indexed-mutation mixfix so:

```zax
custom[row][column] = replacement
```

The first index result must itself support the second index, or one direct
mixfix can consume both adjacent index components. Either route performs the
declared operation without requiring a captured proxy.

### Custom splice results

A bracket containing a range selects `operator splice 1`, separate from
`operator index`. Each bracket contributes one sliced dimension:

```zax
MySequence :: type {
  operator splice 1 final : (
    result : MyView
  )(
    start : IndexSize?,
    endExclusive : IndexSize?
  ) readonly = {
  }
}
```

Omitted endpoints arrive as absent optional inputs. Written inclusive endpoints
are converted to their equivalent exclusive end:

```text
[..5]   -> absent start, present 6
[..<5]  -> absent start, present 5
[0..<0] -> present 0, present 0
[0..]   -> present 0, absent end
[..]    -> absent start, absent end
```

The custom result need not be `MyType[]`. Its declaration owns the result and
domain-specific validation.

## Counts, storage extent, and allocation

Lengths, capacities, indexes, endpoints, and ordinary iterable counts use the
applicable memory domain's `IndexSize`. Backing byte extent uses `TypeSize`.

The two identities have equal ranges and representations within one memory
domain but remain different because one counts logical positions while the
other measures storage.

Before requesting backing storage, Zax checks:

- element count multiplication;
- multidimensional extent;
- alignment;
- metadata addition; and
- conversion to byte extent.

No failure silently chooses another near, ordinary, or far memory domain.

The array's unique storage handle owns disposition of every raw region used for
that array. A provider may implement the region with one allocation, several
chunks, or individual nodes. No array element becomes an independently
resettable allocation merely because the provider uses several physical blocks.

When no storage is written, the declaration resolves a storage representation
profile before its owning layout is fixed. For a provider-backed profile, the
current `___` execution context may supply a compatible provider instance. The
provider separately chooses its backing arenas. Runtime context replacement
cannot change the already resolved self-contained array size.

Allocating a pointer to the array object itself with `@` remains another outer
allocation.

## Required, reporting, and unchecked behavior

Required operations use compact source:

```zax
element := values[index]
view := values[start..<end]
values.resize(newLength)
```

They check runtime conditions and panic on violation. Separately named reporting
operations instead return a fixed optional or reporting result. Their exact
callable names remain future source integration.

Disabling a registered panic category tells the compiler to assume that
condition never occurs and permits it to remove the check. If the promise is
false, behavior is undefined.

Disabling a category never:

- suppresses a compile-time-known error;
- changes required source into an optional result; or
- removes a check needed by an explicitly reporting operation.

The reusable proof and category model belongs to
[safety and analysis](safety-and-analysis.md).

## Diagnostics

Source presentation rejects spaced `[ ]` where the zero-entry form is `[]` and
compact nested-array spelling where contiguous `[[` begins lambda capture.

Intrinsic array source rejects a known:

- element index outside the current or fixed bounds;
- slice endpoint outside bounds;
- reversed range;
- exact initializer count mismatch;
- requested length outside its declared limits;
- fixed-array resize;
- slice parameter, slice reference, or slice member without an element
  stance;
- stance that an element stance does not allow, on the array or on an array
  reference parameter;
- `take` on a fixed array, below the minimum, or on pinned or `deep`
  elements;
- `take` on a slice with a range not written from its start, to its end, or
  as `[..]`;
- relocating operation on pinned elements;
- use of a partially finished array as a whole, or of an element not proven
  unfinished;
- `last` on an element reached through a borrowed array;
- call that fits both a by-value array version and a slice version;
- capacity request beyond a finite maximum;
- foreign slice used to mutate another array;
- incompatible nested or committed element type; or
- overflowing multidimensional or byte extent.

Runtime panic conditions distinguish:

- index bounds;
- splice bounds or order;
- requested length;
- initializer count;
- invalidated slice;
- foreign slice origin;
- broken iterable count promise;
- extent overflow; and
- required backing allocation failure.

Custom bracket operations receive no inferred array checks merely because they
use index or splice syntax. Their own declarations define their validation.

## Costs programmers must be able to see

Depending on the operation and type, array code may perform:

- default, copy, deep, move, or last element construction;
- reverse element destruction;
- backing allocation, reservation, trimming, and recovery;
- full relocation or affected-suffix movement, including copying elements that
  are immutable and so cannot be moved;
- moving each taken element into the new array, then shifting later elements
  down;
- destroying the elements above the minimum after `move`;
- temporary capture for overlapping mutation;
- slice descriptor copying;
- slice origin and validity checks;
- bounds, range, count, and storage-overflow checks;
- repeated growth for an uncounted iterable;
- cursor acquisition and progression; and
- storage-provider lookup, dispatch, and capability checks.

The compiler may remove work only when the selected behavior, effects, order,
lifetimes, failure, and source-visible results remain unchanged.

## Structural and reflection boundaries

An array is one structural leaf. Structural shape processing does not expand
every element or turn runtime length into a record-member list.

Changing between array-of-structures and structure-of-arrays layout is an
explicit transformation. It may allocate, visit elements, transfer values, and
handle overlap; it is not an implicit shape/layout conversion.

Future type reflection needs:

- element type and qualifications;
- rank and per-dimension limits;
- fixed or resizable role;
- resolved storage provider/profile and statically required capabilities;
- storage-erased versus exact reference profile; and
- explicit versus defaulted source properties where that difference affects
  compatibility.

Future value reflection needs:

- current lengths;
- usable capacities;
- absent or present capacity suggestions; and
- operation, provider, physical-backing, or layout state exposed by a declared
  provider API.

A statically known provider type may expose safe provider/handle access. An
erased provider may expose an unsafe raw pointer to the provider object, not to
element contents. Contiguous element access remains a separate
lifetime- and qualification-bearing capability.

## Source stability and future boundaries

Changing element identity, size limits, storage profile/capabilities, dimension
order, expansion, capacity behavior, place stability, transfer availability,
failure policy, or operator/callable surface can change source validity,
behavior, cost, and lifetime.

Source, declaration, import, allocation, or traversal order never silently
resolves ambiguity.

This document is current conceptual design, not formal grammar, an ABI,
implementation representation, generic constraint system, or conformance
specification.

Still future:

- runtime-fixed owning arrays and inferred fixed-but-jagged rows;
- rectangular and strided multidimensional block views;
- owning or allocation-anchored slices;
- exact array storage-provider and unique-handle protocols;
- storage-clause and storage-capability syntax;
- stability token/version representation;
- shared/default provider factories and context hooks;
- detaching whole storage chunks for `take`, instead of moving each element;
- which storage profile an array declared without a storage clause gets;
- provider-specific and erased handle APIs;
- explicit storage-change source;
- exact optional/reporting callable names;
- weaker suggested iterable-count capabilities;
- a standard set of iterable and array protocol constraints, written with
  [`requires`](generics.md#constraints);
- complete runtime string mutation and builder APIs;
- exact reflection API syntax;
- SIMD, MIMD, parallel traversal, and vectorization contracts;
- formal panic-registry syntax and identifiers; and
- ABI, representation, lowering, and optimization algorithms.
