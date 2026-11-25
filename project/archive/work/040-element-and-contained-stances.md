# 040: Element and contained stances

| Field | Value |
| --- | --- |
| Status | Historical / non-normative / audit-only; completed 2026-11-24 and archived |
| Work Item | `040` |
| Created | 2026-11-22 |
| Owns | The bounded review of whether, and how, array elements, slices, optional references, optionals' `move source` and `last source` forms, and lambda reference captures can state the stance of the value they reach |
| Does Not Own | Current transfer-stance, array, slice, optional, lambda, or pointer behavior, which remain with their owners until a promotion is authorized; non-vacant (`present`) pointers; the decision itself, which belongs to the language maintainer |

## Non-authority notice

This file is a collaborative working record. Existing documentation, candidate
syntax, examples, and later aligned findings remain non-authoritative until a
separately discussed, aligned, and explicitly authorized promotion incorporates
them into their lasting owners.

## Fixed initiating input

This section records the aligned information known when work item `040` was
created. It is intentionally incomplete and must not be rewritten as work
develops.

### Initiating concern

Work item 039 gave pointers a stance on the value they point to, and gave
optionals two stances: one for the contained value and one for the optional
itself. Arrays, slices, optional references, and lambda reference captures
still reach a second value without a way to state that value's stance. Should
each be able to, and how?

### Starting input

These are the follow-ups 039 deferred to this work item:

- **A stance on each array element.** Elements are governed by their type
  today. A stance before `[` could apply to each element, as in
  `messages : MyMessage move [4]`, so that `keep(messages[0])` is an error.
- **Slice parameters.** A slice parameter is a borrower that writes no stance
  for the elements it shows, as in `values : MyMessage readonly loan []`.
- **Optionals' `move source` and `last source` forms**, which may be explained
  as the optional's owner writing a stance on the contained value.
- **Optional references** as parameters, results, or members, such as
  `MyValue loan & ?`, which would need a stance as borrowers that stand alone.
  `language/optional-values.md` still has unmarked optional-reference examples,
  left as they are on purpose during 039: the type-composition examples
  (`MyValue readonly & ?`, `MyValue readonly & * writable`), the composed forms
  (`MyValue & * & * &`, `MyValue & ? * & ?`), the
  `view : MyValue readonly & ?` declaration under "Optional references", and
  the local `inner : MyValue? & = nested.`
- **Lambda reference captures**, which should follow the same rule as other
  references bound to one visible value (they take a stance from that value,
  limited to stances that leave the owner's value unchanged); the exact capture
  spelling is unsettled.

### Starting boundaries

- The 039 model is current design: a stance says what may be built from a
  value; owners and borrowers; `via`; pinned values; one stance per layer,
  written last on it; and the optional's two stances. A conflict with it is
  recorded as a finding for discussion, not resolved by silently changing it.
- Non-vacant (`present`) pointers are out of scope. They are a separate concern
  with their own raw input, `project/raw/present-pointers.md`, and trigger.
  **Stated overlap:** that raw input's sixth open question asks whether
  `present` should extend to optionals. If 040 settles optional references, it
  notes the effect on that question rather than designing `present`.

### Initial stopping guidance

First present the candidate model for each item, with code and a
recommendation, the most important uncertainty, and the decisions the
maintainer must make. Do not design every combination in full before the
direction is aligned.

Creating this work item does not authorize analysis. A later assignment
begins it.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs focused reading,
  authority, promotion, deferral, closure, and the writing rules.
- [Zax transfer stances](../../language/transfer-stances.md): "What a stance
  says", "Owners and borrowers", "Declaring stances", "Members", "Optional
  values", and "Arrays and slices" - the model this work extends. "Members"
  matters because elements and contained values behave much like members:
  pinning a whole value does not pin its members, and writable access decides
  whether a member can be taken through a borrower.
- [Zax arrays and slices](../../language/arrays-and-slices.md): slices, array
  and slice parameters, and "Copy, deep, move, and last".
- [Zax optional values](../../language/optional-values.md): "Move and last
  source adapters", "Optional references", and "Stances on an optional and its
  contained value".
- [Zax lambdas and callable composition](../../language/lambdas-and-callable-composition.md):
  captures.

### Consequence-driven

Read only the smallest relevant sections when the concern reaches them:

- [Zax qualifiers](../../language/qualifiers.md), "Canonical ordering", when
  stance position on an element or optional layer is reached.
- [Zax function invocation](../../language/function-invocation.md), when
  parameter selection for slice or optional-reference parameters is reached.
- [Zax iteration](../../language/iteration.md), when loop bindings over
  elements are reached.
- [Zax generics](../../language/generics.md), when type parameters that may be
  arrays, slices, or optionals are reached.
- [Zax pointers, allocation, and arenas](../../language/pointers-and-arenas.md),
  when element pointers or pointers to optionals are reached.

### Audit-only

- Archived numbered work, only when a provenance question cannot be answered
  from current owners or raw input. The pointer-stance review is
  `project/archive/work/039-pointer-stances.md`.

## Working record

### How to read this record

- **Open items** lists what still needs a decision or a check. Start there.
- **Aligned findings** were agreed with the language maintainer on
  2026-11-22 and 2026-11-23. They are not promoted and remain
  non-authoritative until an authorized promotion.
- **Discarded** and **Deferred** record what was rejected and what waits for
  other work, with reasons and reactivation pressure.
- Examples in this record use the aligned spacing rule ("Spacing of layer
  markers"). The fixed initiating input keeps its original spelling.

### Open items

None. Every item is aligned, deferred with a destination, or discarded. The
next step is the pre-promotion documentation fit dry run, which needs
separate authorization.

**Checks run on 2026-11-24 against current owners.** Iteration already
requires `move &` and `last &` loop variables to write the stance on the loop
source, which matches the aligned model; only its wording about unmarked
elements changes. Pointers and arenas has no conflicting element-pointer
rule. Source structure already requires `MyType? ?` and rejects a spaced `?`
after qualifiers; no existing token begins with `?[`, `*[`, or `&[`, and
lambda captures use `[[`. The remaining checks produced decisions, recorded
under aligned findings: how `take` groups with `as`, custom `take`,
deduction, slice-to-array conversion, and storage-clause order.

**Closed during review.** The numbering below was used in discussion:

| Discussed as | Where it is now |
| --- | --- |
| Slice model A or B | "Slices: no stance of their own" |
| Partial-finish options 1, 3, A, B, C, D | "Partial finishing" |
| Alignment 2, arrays to slice parameters | "Arrays passed to slice parameters" |
| Alignment 3, pinned and `deep` elements | "Relocation inside an array" |
| Alignment 4, unmarked outer stance | "An unmarked outer stance takes the inner stance" |
| Alignment 5, inner stances in borrowed types | "Element stances inside a reference type", "Finishing elements through a borrowed array" |
| Alignment 6, `take` details | "`take` and `last X[range]`" |
| Alignment 7, owning-array element qualifiers | "Qualifiers on arrays and their elements" |
| Alignment 8, spellings | "`take` and `last X[range]`" |
| Alignment 10, optional source forms | "The optional's `move source` and `last source`" |
| Alignment 11, optional references | "Optional references" |
| Alignment 12, lambda reference captures | "Lambda reference captures" |
| Alignment 13, spacing | "Spacing of layer markers" |
| Item 14, what `move` leaves | "What `move` and `last` leave in the source array" |
| Open item 1, type of an extracted array | "`take` and `last X[range]`" |
| Open item 2, members versus elements | "Members keep today's rule for now"; Deferred |
| Checks 3a and 3b, grouping and custom `take` | "`take` and `last X[range]`" |
| Check 3c, deduction | "An unmarked outer stance takes the inner stance" |
| Check 3d, slice to by-value array | "Arrays passed to slice parameters" |
| Check 3e, storage-clause order | "Spacing of layer markers" |
| Default storage profile | Deferred, into an existing raw input |
| Allow-with-tracking principle | Deferred, with a raw destination |

### Aligned findings (not promoted)

#### Storage terms used below

An array's **storage profile** says how it keeps its elements. It is chosen
when the array is declared, by a `storage` clause or by the declaration's
default storage profile.

- **Inline storage:** the elements live inside the array value itself. The
  array cannot hand its storage to another array, and cannot grow past the
  room built into it, so a resizable inline array needs a fixed maximum.
- **Provider-backed storage:** the array value holds a handle to a block that
  a **storage provider**, such as an arena, manages. The array can hand the
  block to another array, and grows by asking the provider. When no provider
  is written, an arena-backed profile may take a compatible provider from the
  current `___` execution context.

```zax
grid : Integer[9]                                // inline, if the default profile is inline
samples : Integer[0..] storage provider          // provider-backed: this provider instance
pooled : Integer[0..] storage type MyStorage     // provider-backed: a provider of this type
```

Which profile a declaration without a `storage` clause gets is deferred (see
Deferred).

#### Arrays: an element stance, with elements handled like members

An array has two stances. The stance before `[` is each element's; the stance
after `]` is the array's own.

```zax
consume final : ()(message : MyMessage) = {
}

messages : MyMessage move[4] move    // each element: move; the array itself: move

consume(messages[0])                 // error: the element is move; write as move or as last
consume(messages[0] as move)         // fine: the array's owner writes stances on its elements
other := messages                    // error: the array is move; write as move or as last
other := messages as move            // fine: moves the array, and so moves each element

queue : MyMessage[0..] loan          // a pinned array of copy elements
consume(queue[0] as move)            // fine: pinning the array does not pin its elements
```

- The array's own stance governs only transfers of the whole array. It never
  changes how individual elements are handled.
- Elements are handled like members. The element stance decides what may be
  built from each element. The array's owner writes stances on elements by the
  owner table. Through a writable reference to the array, the member rule
  applies to `move`: writable access decides whether an element can be moved
  out. (`last` is stricter; see "Finishing elements through a borrowed
  array".)
- The influence runs one way, from the elements to the whole. Copying or
  moving an array copies or moves each element, so the element stance limits
  the array's stance by the optional's table: elements `move` allow the array
  `loan`, `move`, or `last`; elements `loan` allow only `loan`; and so on. A
  whole-array transfer passes its stance on to each element, as a present
  optional passes its stance on to its contained value.
- Existing arrays keep their meaning: unmarked elements are `copy`, which
  allows any array stance.

#### An unmarked outer stance takes the inner stance

When an array's or optional's own stance is unmarked, it is the same stance as
its elements or contained value. Nested layers apply this from the inside
out. The matching stance is always allowed, so generic and alias-based
declarations never fail merely because the outer stance is unmarked:

```zax
messages : MyMessage move[4]           // the array is move
plain : MyMessage[4]                   // copy elements, so the array is copy, as today
pinned : MyDevice loan[4]              // the array is loan
slot : MyPayload move?                 // the optional is move

OwnedPayload :: alias type Payload move
batch : OwnedPayload[4]                // the elements are move through the alias, so the array is move

Holder$ :: type (T : type) {
  items : T[4]                         // works for every T
}
```

This is the one-way influence from the elements to the whole. Other places
keep their current rules:

- **User types and lambdas** stay unmarked `copy`. A member or capture that
  forbids copies makes copying unavailable, and the error is reported where a
  copy is attempted. Generic code never fails at the declaration.
- **Pointers** are unaffected. Copying a pointer never copies what it points
  to, so the pointed-to stance does not limit the pointer's stance.
- **Inferred declarations** do not carry element stances. The elements of a
  new array are new places, and an inferred declaration never takes a
  declared stance:

```zax
messages : MyMessage move[4]
moved := messages as move              // each element is built by moving from messages
// moved is MyMessage[4]: elements unmarked, so copy; the array is copy
moved2 : MyMessage move[4] = messages as move    // writing the stance keeps the restriction
```

- **Generic deduction** drops declared stances, matching inference. A
  stance reaches `T` only when the type argument is written explicitly as an
  alias that carries one:

```zax
keepAll$ final : (T : type)()(values : T[4]) = {
}

messages : MyMessage move[4]
keepAll(messages as move)            // T is deduced as MyMessage, not MyMessage move
batch : OwnedPayload[4]
keepAll$(OwnedPayload)(batch as move)   // illustrative explicit type argument:
                                        // T is OwnedPayload, which carries move
```

A guarantee for every value of a kind, wherever it is stored, still comes
from a `transfer` list on the type. Teaching note: a user type with `move`
members is a `copy` variable that cannot be copied, while an array of `move`
elements is a `move` array. The array's element stance is part of its type;
a user type's copyability is decided by its definition.

#### What `move` and `last` leave in the source array

`move` always moves every element out, so the destination receives all of
them. Afterward the source has its declared minimum length, and every element
it keeps is moved-from. Elements above the minimum are destroyed from the
highest index downward, as a shrink destroys them. Capacity does not change.

```zax
recent : MyMessage[0..8]           // minimum 0
recent <+>= [ a, b, c, d, e ]
moved := recent as move            // moved gets all 5
recent.length()                    // 0: the minimum is 0, so nothing is kept

window : MyMessage[3..8]           // minimum 3
window <+>= [ a, b, c, d, e ]
moved2 := window as move           // moved2 gets all 5
window.length()                    // 3: elements 0, 1, 2 are kept, moved-from;
                                   // elements 3 and 4 are destroyed

pair : MyMessage[4]                // fixed: minimum and maximum are both 4
moved3 := pair as move             // moved3 gets all 4
pair.length()                      // 4: every element kept, moved-from
```

- A fixed array is the case where the minimum equals the maximum, so nothing
  is destroyed.
- A provider-backed array may hand over its whole storage block on `move`
  only when its minimum is 0; the source is then left with no elements and no
  block. With a minimum above 0, each element is moved and the source keeps
  its block. Getting a new block for the minimum would hide an allocation
  inside `move`.
- `last` may always hand over the block, including for a fixed
  provider-backed array, because after `last` the source only has to be safe
  to destroy. An inline array still passes each element on for `last`,
  because inline storage cannot be handed over.
- Costs to teach: `move` on a resizable array runs one destructor for each
  element above the minimum.

Promotion corrects the transfer-stances sentence "After `move`, the original
must still be a usable, empty array" to this rule.

#### Relocation inside an array

Some operations move elements to new places inside the array: growing into
new storage, reserving capacity, insertion and removal that shift later
elements, and `take` closing its gap. Relocation is the array managing its
own places. It is not code building something from an element, so:

- relocation passes each element on with `last`, falling back to `move` and
  then `copy`, chosen from what the element's type can do and the qualifiers
  on the element;
- the declared element stances `copy`, `move`, `last`, and `deep` do not
  restrict relocation;
- when the type can do none of them, any operation that would relocate
  elements is unavailable, as the arrays owner already states;
- **`loan` is the exception.** A pinned element promises that its address
  never changes, so any operation that could relocate pinned elements is an
  error, unless the storage promises never to relocate (a provider
  capability).

```zax
archives : MyArchive deep[0..]
archives.reserve(64)            // fine: relocation uses MyArchive's own last, move, or copy
keep(archives[0])               // error: only independent copies may be built from the element

frozen : MyMessage immutable[0..]
frozen <+>= [ makeMessage() ]   // if this relocates, each element is copied:
                                // last and move would need to change an immutable element

devices : MyDevice loan[0..]
devices <+>= [ makeDevice() ]   // error: growing may relocate pinned elements,
                                // unless the storage promises never to relocate
fixedDevices : MyDevice loan[4]
fixedDevices[0].start()         // fine: a fixed inline array never relocates its elements
```

Costs to teach: relocating immutable elements copies them.

#### Qualifiers on arrays and their elements

An array's qualifiers and its elements' qualifiers are independent, as an
optional's wrapper and payload qualifiers already are. The array's own
qualifiers, after `]`, cover its structure: length, capacity, and storage.
The element qualifiers, before `[`, cover the elements:

```zax
roster : MyMessage mutable[8] immutable      // no insert, remove, or resize
roster[0] = fresh()                          // fine: the elements are mutable
roster.resize(9)                             // error: the array is immutable

journal : MyMessage immutable[0..] mutable   // entries cannot change
journal <+>= [ entry ]                       // fine: the array can grow
journal[0] = fresh()                         // error: the element is immutable
```

- An unmarked element qualifier takes the ordinary default. Making both layers
  immutable is written on both: `Integer immutable[4] immutable`.
- An immutable empty array can never be filled. No special rule is needed.
- Slices already separate element qualifiers from descriptor qualifiers;
  this extends the same separation to owning arrays.
- Promotion fixes every example that relies on the old reading. The arrays
  owner's `Integer[4 in 4..10] immutable writable varying` example currently
  makes `myArray[0] = 10` an error because the immutable array covered its
  elements; under this rule that line is fine unless the elements are also
  `immutable`. Teaching must say so, because C++ programmers expect a
  `const std::array` to make its elements const.

#### Spacing of layer markers

The spacing is required, not a style choice:

- **`?` attaches** to a name or to a qualifier or stance word:
  `MyValue?`, `MyPayload move? loan`, `MyValue readonly?`. After other
  punctuation it is separated by a space: `MyValue readonly & ?`,
  `MyValue? ?` (`??` is a different token).
- **`[` attaches** to whatever precedes it, including punctuation, so it is
  clear what the array or slice is of: `Integer[5]`, `MyMessage move[4]`,
  `MyMessage readonly loan[]`, `MyPayload move?[40]`, `MyPayload move *[40]`.
- **Several dimensions form one layer** and are written together:
  `Integer[4][3]`.
- **`*` and `&` never attach.** Each is its own layer:
  `MyValue loan * unique`, `MyValue readonly loan &`.

```zax
slot : MyPayload move? loan          // optional: ? attached to the stance word
maybeMany : MyPayload move?[40]      // an array of 40 optionals
view : MyValue readonly loan & ?     // an optional reference: ? after & is spaced
nested : MyValue? ?                  // nested optionals
matrix : Integer[4][3]               // one two-dimensional layer
owners : MyPayload move * unique[8]  // an array of 8 unique pointers

spaced : MyPayload move ? loan       // error: ? must attach to move
```

This explains the existing optional-owner error on
`MyValue immutable readonly final ? mutable writable varying`. Promotion
converts every affected example, including the transfer-stance and optional
examples written `MyPayload move ? loan`.

**Order on an array layer.** The dimensions come first, then the storage
clause, then the array's qualifiers, then its stance, so the stance stays
last on its layer and the storage clause still follows the dimension list:

```zax
pool : MyMessage move[0..] storage type MyStorage immutable move
```

#### Slices: no stance of their own

A slice has no stance. Each element it shows is reached as if through a
`<stance> &` reference, and the ordinary reference rules apply. The descriptor
keeps its qualifiers, which narrowing needs, but it can always be copied and
is never moved or finished.

```zax
inspectSome final : ()(values : MyMessage readonly loan[]) = {
  show(values[0])                    // fine: lends the element
  consume(values[0])                 // error: nothing may be built from a loan element
}

drainSome final : ()(values : MyMessage mutable writable move[]) = {
  consume(values[0] as move)         // fine: the slice was given move for each element
  consume(values[0] as last)         // error: a borrower given move may write only as move
}

total final : ()(values : Integer[]) = {   // error: write an element stance, for example Integer copy[]
}

messages : MyMessage move[8] move
window := messages[1..<4]            // bound to one visible array: the elements are move,
                                     // so an unmarked view gets loan for each element
```

- A slice parameter stands alone, so its element stance must be written, as a
  reference parameter's must. Every plain-data slice parameter gains a word.
- A slice bound to one visible array takes its element stance from the
  elements by the "references you declare" table.
- `as` written on a slice applies to its elements.
- The current `slice as last` error and its reason ("a slice is a borrower
  that was not given `last`") must be reworded at promotion: the descriptor has
  no stance, and what may be built from the elements depends on the stance
  the slice was given.

#### What a `last` view allows

The ordinary `last &` rules apply to every element of a view given `last`:

```zax
whole : MyMessage last[] = messages[..] as last

peek : MyMessage readonly loan[] = whole      // fine: whole is lent, not given up
twin : MyMessage last[] = whole               // error: whole may only lend or write as last

keep final : ()(message : MyMessage) = {
}

show final : ()(message : MyMessage copy &) = {
}

keep(whole[0])                                // error: whole holds last
keep(whole[0] as last)                        // fine: keep owns the new MyMessage and may copy it
show(whole[0] as last)                        // fine: show can only copy, but the caller
                                              // counts as having given the element up
```

Copying out of a `last` view is possible only through `as last`. Afterward,
`whole` is partially finished, as the next section describes.

#### Partial finishing: `last` on elements that stay in place

Writing `last` on an element or range that stays inside its array or view
makes the holder **partially finished**. The elements are terminated, not the
array itself; the array refuses most operations only because they would reach
terminated elements. A partially finished array or view:

- does not survive as a whole: copying it, moving it, passing it whole, and
  ordinary iteration over it are errors;
- may still be destroyed or replaced, like any finished value;
- allows a use of one element (reading, lending, or writing `last` on it) only
  when the element is proven not to have been finished;
- requires `unsafe<...>{ }` for an element use that cannot be proven. The
  future compiler-verified tag `[<safe=x-compiler>]` lets a named compiler
  check such a site.

The first `last` is always allowed. The error lands on a later use that cannot
be proven safe:

```zax
consumeAll final : ()(values : MyMessage last[]) = {
}

messages : MyMessage[0..]
consumeAll(messages[2..<5] as last)  // fine: messages is now partially finished
messages[0].send()                   // fine: 0 is proven outside 2..<5
other := messages                    // error: messages is partially finished

dispatch(messages[i] as last)        // fine: the first last on this element
dispatch(messages[j] as last)        // error: cannot prove j differs from i;
                                     // unsafe<...> takes responsibility for it

pair : MyResource[2]
first := pair[0] as last             // fine
second := pair[1] as last            // fine: proven to be a different element
pair[0].close()                      // error: pair[0] was given up

each i : in 0..<messages.length() {
  dispatch(messages[i] as last)      // fine: each pass has a different i
}
```

**What counts as proven.** The specification lists the proven cases, not each
compiler. A compiler that can prove more may only warn that an `unsafe` looks
unnecessary, or verify the site through the tag. This follows the
compiler-agreement direction recorded in
`project/raw/compiler-agreement-principle.md`. The specification starts with a
small list and may grow it later, which keeps existing programs compiling:

1. constant indexes (`pair[0]` and `pair[1]` are different elements);
2. constant ranges (`messages[0]` is outside `messages[2..<5]`);
3. the language-controlled loop `each message : in messages as last`;
4. a plain index loop: the loop variable is the only index used on the array,
   it is not changed inside the loop, and each pass touches only
   `messages[i]`.

After a plain index loop, every element the loop covered counts as possibly
finished:

```zax
each i : in 0..<messages.length() {
  if urgent(messages[i]) {
    dispatch(messages[i] as last)    // element i may now be finished
  }
}
messages[0].send()                   // error: cannot prove messages[0] was not finished
```

**Replacing terminated elements.** A finished value may be destroyed or
replaced. A terminated element can be replaced in place, and once every
terminated element is proven replaced or gone, whole-array operations are
allowed again. Proof uses the same working list:

```zax
each i : in 0..<messages.length() {
  dispatch(messages[i] as last)       // element i is finished
  messages[i] .= makeMessage()        // replaced in place: element i is live again
}
send(messages)                        // fine: every element was proven replaced

messages[2..<5] = [ a, b, c ]         // direct splice over a finished range: the old ones are
                                      // destroyed, the new ones built; the range is live again
messages .= freshArray()              // replacing the whole array ends every old element

messages[3] = fresh()                 // error: = assigns into the old value, which is finished;
                                      // use .= to replace it
```

Where every replacement cannot be proven, the array stays partially
finished. A function that was given `last` and replaced elements in its own
body does not restore the caller's view: the caller still treats the
elements it gave up as finished.

The same rules apply to a view given `last`. `take` is the way to keep the
whole array live, because it removes elements instead of leaving finished
ones behind.

#### `take` and `last X[range]`

`take` and `last X[range]` are the canonical spellings. `take`, written before
a place and a range, removes that range from the source and hands it on. It
is one form on a place and a range, like direct splice mutation
(`values[2..4] = []`), not an operation on a temporary slice. It binds before
`as`, and the `as` that follows is the ordinary stance on what `take`
produces.

**Recognition and grouping.** The operator catalog lists `take X[range]` and
`last X[range]` as recognized forms, as it recognizes a range on the left of
`=` as one direct splice form. Without that, indexing would bind first and
`take` would apply to a temporary slice. A pre-unary phrase applies before a
following post-unary phrase at the same level, so `take X[range] as last`
means `(take X[range]) as last`.

**Custom types.** A type may declare its own `take` as a mixfix with a
`pre-unary 'take'` component and a `splice 1` component. `last X[range]` is
never declared separately: it always means that type's `take`, followed by
`as last`. On intrinsic arrays and slices, both forms are protected.

```zax
MySequence :: type {
  operator mixfix
    pre-unary 'take'
    splice 1
  final : (result : MySequence)(start : IndexSize?, endExclusive : IndexSize?) writable = {
  }
}
```

**On an array, `take` extracts a new owning array.** The array's owner gives
the elements to the new array: each element is passed on with `last`, falling
back as relocation does, and the source closes the gap immediately. The
source stays fully live and becomes shorter.

**The extracted array's type follows the `<+>` join precedent.** Its length
range is every count the `take` could produce, and its storage follows
join's rule: it may use the source's provider or the applicable default. When
the destination's type is written, `take` builds directly into it, with no
second move.

```zax
messages : MyMessage[0..8]
window := take messages[2..<5]       // MyMessage[3]: the range is constant
tail := take messages[k..]           // MyMessage[0..8]: at most what messages can hold

bounded : MyMessage[3..8]
spare := take bounded[k..]           // MyMessage[0..5]: bounded must keep at least 3

kept : MyMessage[0..] storage type MyStorage = take messages[2..<5]
                                     // built directly into kept
```

```zax
messages : MyMessage[0..]
window := take messages[2..<5]           // window is a new owning array of 3 elements;
                                         // messages closes the gap at once
consumeAll(take messages[2..<5] as last) // the temporary array's elements are given last
consumeAll(last messages[2..<5])         // the same, with the shortcut
messages[0].send()                       // fine: messages is live
```

- The element stance must allow the elements to leave: pinned elements
  cannot be taken, and `deep` elements allow only independent copies.

```zax
devices : MyDevice loan[0..]
take devices[0..<1]                  // error: nothing may be built from pinned elements

archives : MyArchive deep[0..]
take archives[0..<1]                 // error: deep elements allow only independent copies
```

- `take` is governed by the element stance and the array's qualifiers
  (`mutable writable`, as removal needs), never by the array's own stance.
- A fixed array cannot `take`, and `take` cannot take a resizable array below
  its minimum: a compile-time error when the count is known, and a checked
  panic when it is known only at runtime, as with `resize`.

```zax
window : MyMessage[3..8]
window <+>= [ a, b, c, d, e ]
consumeAll(take window[0..<2] as last)    // fine: 3 elements remain
consumeAll(take window[..] as last)       // error: would leave 0, below the minimum of 3
consumeAll(take window[0..<k] as last)    // checked: panics if it would go below 3
```

- Any stance may follow `take`, but taking elements out of an array is
  practically always followed by `as last`. A lint may point that out for
  `take … as move`.
- Costs to teach: each extracted element is built into the new array's
  storage, and the later elements shift down. Taking from the back shifts
  nothing.

**On a slice, `take` shrinks the slice.** The elements stay where they are in
the array; the result is a shorter slice holding the stance the source held
for those elements, and the source no longer covers them. The range must be
written from the start (`[..<n]`, `[..n]`), to the end (`[n..]`), or as all
(`[..]`). The compiler sees the shape in the source text, so no runtime proof
is needed. A middle piece takes two steps, which are cheap.

```zax
whole : MyMessage last[] = messages[..] as last    // covers elements 0..<8
front := take whole[..<2] as last        // fine: whole now covers 2..<8
middle := take whole[..<3] as last       // middle covers 2..<5; whole covers 5..<8
back := take whole[1..] as last          // fine: to the end; whole covers 5..<6
rest := take whole[..] as last           // fine: all that remains; whole is now empty

bad := take whole[1..<2] as last         // error: a slice can only give up a range
                                         // from its start or to its end
alsoBad := take whole[a..<b] as last     // error, even if a is 0 at runtime: write [..<b]
peek := take whole[..<1]                 // fine: no stance, so peek may only look; nothing
                                         // can finish that element now, and the array's
                                         // destructor destroys it later
```

- `last X[range]` is an explicit shortcut for `take X[range] as last`, on
  arrays and slices alike. Both spellings stay valid. On a `copy` or `move`
  slice, `last X[range]` is an error, because that slice was never given
  `last`.
- Without `take`, the holder becomes partially finished; with `take`, the
  source stays fully live. That distinction is the teaching point.

**No split primitive.** Splitting a slice several ways is repeated `take`,
which costs only descriptor arithmetic:

```zax
whole : MyMessage last[] = messages[..] as last   // no element moves
partA := take whole[..<4] as last
partB := take whole[..<4] as last
partC := take whole[..] as last
```

For an array whose minimum is 0, `take messages[..]` and `messages as move`
both leave a usable, empty array, but they differ: `take` builds a new array
and moves each element into it, and `messages` keeps its storage and
capacity; `as move` transfers the whole array, and a provider-backed array
can hand over its storage block.

#### Element stances inside a reference type

An element stance written inside a reference type says what the function may
do with the elements. The caller's elements must allow it, by the existing
table for passing a value you own to a reference:

| Caller's element stance | Element stances a parameter may name |
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
  consume(values[0] as move)            // fine: elements move, and writable access
}

copyAll final : ()(values : MyMessage[4] readonly loan &) = {
  keep(values[0])                       // fine: unmarked elements are copy
}

messages : MyMessage move[4] move
inspect(messages)                       // fine: move elements allow loan
drain(messages)                         // fine: move elements allow move
copyAll(messages)                       // error: move elements do not allow copy

plain : MyMessage[4]
copyAll(plain)                          // fine: today's examples keep working
```

Unmarked still means `copy`, so existing examples such as
`values : Integer[5] loan &` keep their meaning. A parameter that names
`loan` elements accepts any array and only looks. The same rule applies to
the contained value in a reference to an optional
(`MyPayload move? loan &`) and to the value behind a pointer reached through
another pointer.

#### Finishing elements through a borrowed array

Finishing an element in place always needs a `last` that the caller writes
where it can be seen. Partial finishing is full termination of the elements
carried out in steps.

- Through a borrowed array, `move` on an element follows the member rule:
  writable access decides, and the element stays usable.
- Through a borrowed array, `last` on an element is an error unless the
  reference itself holds `last`, which means the caller wrote `as last` on the
  whole array.
- `take` is not a terminal transfer. It removes the elements, so the caller's
  array stays fully valid and becomes shorter. Through a writable array
  reference, `take` needs nothing written at the call, as `resize` and direct
  splice removal already need nothing.

```zax
finishFirst final : ()(values : MyMessage[0..] mutable writable loan &) = {
  consume(values[0] as move)              // fine: member rule, the element stays usable
  consume(values[0] as last)              // error: a borrowed array allows element move, not last
  consumeAll(take values[0..<1] as last)  // fine: the element leaves the caller's array
}

finishAll final : ()(values : MyMessage[4] mutable writable last &) = {
  consume(values[0] as last)              // fine: the caller gave up the whole array
  consume(values[1] as last)
}

messages : MyMessage[0..]
finishFirst(messages)                     // nothing written: messages may be shorter afterward

fixed : MyMessage[4]
finishAll(fixed as last)                  // the caller writes the terminal stance on the whole array
```

#### Members keep today's rule for now

Members keep the current Members rule: through writable access, a function
may write `as last` on a member, and nothing tracks it. Elements are stricter
because an element at a runtime index cannot be reasoned about the way a
named member can, and a member's type author controls what a finished member
means. Promotion teaches that difference in those terms. It does not promise
later tracking, because current owners do not route readers to raw input or
future work.

```zax
refresh final : ()(input : Message mutable writable copy &) = {
  keepPayload(input.payload as last)    // members: allowed for now
}

finishFirst final : ()(values : MyMessage[0..] mutable writable loan &) = {
  consume(values[0] as last)            // elements: an error through a borrowed array
}
```

The direction for tracking finished members, and possibly elements, is
deferred to a follow-up work item (see Deferred).

#### Arrays passed to slice parameters

When an array is passed to a slice parameter, `as` on the array applies to
its elements, as `as` on a reference argument applies to the value it refers
to:

```zax
drainSome final : ()(values : MyMessage move[]) = {
  consume(values[0] as move)
}

messages : MyMessage[8]
drainSome(messages)                   // error: copy elements give an unmarked view copy;
                                      // the parameter needs move
drainSome(messages as move)           // each element is given move; messages stays usable
drainSome(messages as last)           // each element is given last; messages is partially
                                      // finished, every element given
```

**Overloads.** A call that fits both a by-value array overload and a slice
overload is ambiguous, by the existing rule that a better stance match never
wins over a worse type match:

```zax
drain final : ()(values : MyMessage[0..]) = {       // by value: exact type, via copy
}

drain final : ()(values : MyMessage move[]) = {     // a slice: needs the array-to-slice
}                                                   // conversion, but the stance matches

drain(messages as move)               // error: ambiguous
drain(messages[..] as move)           // the slice overload: the argument is already a slice
drain(messages as copy)               // the by-value overload: the slice needs move
```

**A slice never converts implicitly into a by-value array.** Copying every
element is a cost the reader should see, and an array expression shows it.
This is also what keeps `drain(messages[..] as move)` from selecting the
by-value overload:

```zax
consume final : ()(values : MyMessage[0..]) = {
}

window := messages[2..<5]
consume(window)                      // error: a slice is not an array
consume([ from window ])             // fine: builds a new array from the slice's elements
```

Making the whole-array overload win was rejected. The two outcomes differ
observably (a provider-backed array may be left empty, while the slice form
never changes the length), and adding a by-value overload later would
silently change existing calls. No new form is added to force the by-value
choice for `move` or `last`. When callers need that, the functions should
have different names, which becomes advice for library authors.

#### The optional's `move source` and `last source`

An optional has two forms written before it, in addition to the ordinary
`as`. They work whether or not the optional holds a value, and `last source`
leaves the optional empty and usable instead of finished:

```zax
source : MyPayload?

keep(source as last)   // ordinary stance on the whole optional: source is finished as a whole
keep(last source)      // the payload is passed on with last; when keep returns, the rest of the
                       // payload is destroyed and source is empty, but still usable
keep(move source)      // the payload may be moved from; source still holds it, moved-from
```

Both forms are explained as a stance written on the payload, so the payload's
stance must allow them:

```zax
frozen : MyPayload loan?             // the payload is pinned
keep(move frozen)                    // error: nothing may be built from a pinned payload
keep(last frozen)                    // error, for the same reason

slot : MyPayload move? loan          // a pinned optional with a move payload
keep(last slot)                      // fine: pinning the optional does not pin its payload
```

Through a reference to someone's optional, the array rules apply, because a
payload is like an element:

```zax
refill final : ()(cache : MyPayload? mutable writable loan &) = {
  keep(move cache)          // fine: like move on an element, the payload stays usable
  keep(last cache)          // fine: like take, the payload leaves and cache becomes empty;
                            // nothing finished is left in the caller's optional
  if ?cache {
    keep(cache. as last)    // error: like last on an element of a borrowed array, this would
  }                         // leave a finished payload inside the caller's optional
}

peek final : ()(cache : MyPayload? readonly loan &) = {
  keep(move cache)          // error: readonly access cannot change the payload
}
```

`cache` writes no payload stance, so by "Element stances inside a reference
type" its payload is `copy`, and only optionals whose payload is `copy` may be
passed.

#### Optional references

A stance must be written wherever there is no single value to take it from.
An optional reference is that case: it can later refer to a different value,
which may have a different stance. So an optional reference always writes its
stance before `&`, including a local with an initializer:

```zax
view : MyValue readonly loan & ?           // fine: absent now; any later reference is a loan
view .= [{ firstValue }]                   // refers to firstValue
view .= [{ secondValue }]                  // a new reference to a different place, same stance

guess : MyValue & ? = firstValue           // error: write the stance before &
```

A plain reference bound to one visible value takes that value's stance,
because it can never refer to anything else.

`inner : MyValue? & = nested` is not an optional reference. It is a reference
to an optional, bound to one visible value, so it takes the optional's stance
from `nested` by the existing table, and its unmarked payload layer is
`copy`.

Optional-reference results already use this form in current owners, for
example `possibleOwner final : (result : Car loan & ?)()` in function
invocation and composition, and `failure except : MyError readonly loan & ?`
in exceptional flow. Promotion gives a stance to the unmarked examples in
`optional-values.md`. The composed type-formation examples
(`MyValue & * & * &`, `MyValue & ? * & ?`) illustrate which layers may nest;
promotion either marks them illustrative or gives each `&` a stance.

**Effect on `present` pointers (stated overlap).** The optional's own stance
sits after `?`, the position a `present` modifier would also use for
optionals. This work does not constrain the sixth open question in
`project/raw/present-pointers.md` beyond noting that `present` and the
optional's stance would share the outer layer, in the established
qualifier-then-stance order.

#### Lambda reference captures

The transfer-stances owner is followed and the lambdas owner changes. A
reference capture follows the "references you declare" table: an unmarked
capture of a `copy` variable gets `copy`; of a `move` or `last` variable, it
gets `loan`. Under the lambdas owner's current rule (always `loan`), even
passing a captured `Integer` to a by-value function would be an error:

```zax
log final : ()(value : Integer) = {
}

total : Integer
accumulate := [[ total: capturedTotal : & ]] ()(delta : Integer) {
  capturedTotal += delta        // changes total through the reference
  next := capturedTotal + 1     // reading to compute is not building from it
  log(capturedTotal)            // fine: the capture is copy, so log may copy it
}
```

An explicit stance goes before `&`, as for every reference, and a grant goes
on the source:

```zax
owner : Buffer move
drainer := [[ owner as move: mover : move & ]] ()() {
  consume(mover as move)
}
// from here until drainer ends, owner must be treated as possibly emptied
```

A captured `move &` may empty `owner` at any later call of the lambda, so the
owner assumes "possibly emptied" from the capture onward while the lambda
lives; a captured `last &` finishes `owner` at the capture. That follows from
"a reference stance is a permission, not a promise", and needs explicit
teaching because the window is wider than for a local reference.
`unsafe pliable` stays uninherited.

#### Other effects to carry into promotion

- Element raw pointers (`owner.[0]`) and typed loop bindings
  (`each value : MyValue readonly & from values`) are references bound to one
  visible value, so they follow the "references you declare" table. Elements
  `move` give an unmarked binding `loan`. The current sentence "a loop binding
  over elements of a type with `copy` gets `copy`" becomes the case of
  unmarked elements.
- `qualifiers.md` "Canonical ordering" lists optional, pointer, and reference
  markers at step 6. Array and slice markers join that step, with the spacing
  rule above.

### Discarded, with reasons

- **A slice as a view of members (model B).** Under it, writable access
  through a slice would decide whether elements may be moved out, as for
  members reached through a reference. Discarded because the stance before
  `[]` would have no clear meaning, and a writable slice parameter could empty
  the caller's elements with nothing at the call to show it. Accepted cost of
  the chosen model: a writable array reference and a writable slice give
  different element access.
- **A stance on the slice descriptor, including `move` and `last` forms that
  vacate the source slice.** Copying, moving, and finishing a descriptor all
  do the same harmless thing, and `deep` would wrongly suggest copying the
  elements. `window as last` would finish only the descriptor, the opposite of
  what a reader expects. Vacating the source gives `move` a meaning but
  protects nothing while copies remain possible.
- **A `unique` slice.** Zax `unique` means one owner, not one path. The
  pointers owner allows references and slices alongside a `unique` owner. A
  slice owns nothing, and forbidding copies does not stop another slice over
  the same range, so a `unique` slice guarantees nothing without a new
  exclusivity rule.
- **A split primitive.** Repeated `take` on a slice does the same at almost
  no cost.
- **Other answers to finishing one element in place.** Giving up the whole
  array on any element `last` blocked index loops and consuming a fixed
  array's elements one by one. Treating elements like untracked members let
  finished elements be used again with nothing to catch it. Tracking only
  constant indexes was subsumed by partial finishing. A "being finished"
  state that allowed any further `last` grant could not detect the same
  element being finished twice with runtime indexes.
- **`take` on an array as a view with delayed removal.** The taken elements
  would have stayed in place until the receiving call ended, which avoided
  one relocation but left a hole of unknown duration and forbade keeping the
  result in a named variable. Replaced by extraction into a new array.
- **Direction words for slice `take`** (`take front`, `take back`,
  `take middle`). The range's shape already shows the end, the words could
  disagree with it, and a middle take would need two results.

### Deferred

- **Which storage profile applies when no `storage` clause is written.** An
  array with no maximum (`[0..]`) must be provider-backed; for fixed and
  bounded arrays (`[4]`, `[0..8]`) the arrays owner does not say. A programmer
  needs it to predict whether `move` can hand over storage and what an array
  costs. The maintainer's direction: likely an explicit keyword where the
  choice is unclear, or a default with an explicit keyword override. An
  arena-backed profile may take its provider from the `___` context.
  Destination: the existing raw input `project/raw/array-storage-strategies.md`,
  which already lists "an omitted value-declaration strategy resolved from
  defaults/context" among its pressures. The promotion change set adds the
  inline-versus-provider-backed question, the maintainer's direction, and
  these examples there:

  ```zax
  grid : Integer[9]                    // inline or provider-backed? not stated today
  recent : MyMessage[0..8]             // a bounded resizable array: either is possible
  samples : Integer[0..]               // no maximum: must be provider-backed
  pooled : Integer[0..] storage type MyStorage   // explicit today
  ```

  Reactivates with future storage work or with any promotion that teaches
  `move` costs. Constraint on current work: promoted teaching must not state
  which profile an unmarked fixed or bounded array gets.
- **Handing over a storage block with pinned elements.** A provider-backed
  array could hand over its block without moving pinned elements, so
  `MyDevice loan[0..] move` might be safe there, despite the "elements `loan`
  allow the array only `loan`" table. Belongs to future storage work, with the
  exact provider and handle protocol the arrays owner already defers.
- **Exclusive access and a provably safe contract.** Raised while comparing
  Zax with Rust; outside this item's scope. Destination: a new section of the
  existing raw input `project/raw/safety.md`, which already lists aliasing
  and data-race boundaries as guarantee categories to investigate, written
  in the promotion change set. It preserves:
  - **Two levels of safe code.** Compiling with no `unsafe` at all is a
    compile-time choice that is safe apart from logic errors, but only for the
    source compiled in that mode: the language's own library and compiler
    remain trusted, as `#![forbid(unsafe_code)]` covers one Rust crate while
    its standard library uses `unsafe`. The other level allows `unsafe` only
    in narrow, named, auditable places where a programmer asserts that the
    code is actually safe; an `unsafe` assertion "says that a fact is true".
  - **`exclusive` as a promise the compiler validates, not a lock.** The
    maintainer's preferred direction for exclusive write access. An opt-in
    path meaning that while it exists no other path may read or write the
    value, checked by lifetime analysis like Rust's `&mut`, keeps today's low
    friction for code that does not use it. Borrowing a writable view of an
    `immutable` value was advised against, because it breaks the lifetime
    promise every reader relies on; the reverse direction already exists
    (building an immutable value from mutable storage over which the compiler
    proves exclusive authority). A runtime lock wrapper, like `Mutex` or
    `RefCell`, is complementary for threads and dynamic sharing, and its
    reentrancy failure should be a defined error.
  - **Three reentrancy cases.** Mutation while access is held throughout,
    which an exclusive path prevents; lifetime safety around resizing, which
    Zax currently answers partly with runtime validity checks; and access
    released and taken again around a call-out, which no borrow rule can
    cover and which needs re-checking after call-outs, state machines, no
    call-outs while an invariant is broken, or declared effects (overlapping
    `project/raw/purity-and-effects.md`). Rust allows the third case too:

    ```rust
    fn send(state: &Rc<RefCell<Buffer>>, on_sent: impl FnOnce()) {
        { state.borrow_mut().prepare(); }   // take access, then release it
        on_sent();                          // the callback may reset the buffer
        state.borrow().read();              // compiles; the data is gone
    }
    ```

    ```zax
    // compiles in Zax today: two writable paths to one buffer
    send final : ()(buffer : Buffer mutable writable loan &) = {
      notifySent()       // a callback resets the same buffer through another path
      buffer.read()      // the buffer changed underneath
    }

    // illustrative only: no exclusive form exists
    send final : ()(buffer : Buffer mutable writable exclusive &) = {
      notifySent()       // error if the callback can reach buffer through another path
      buffer.read()
    }

    values : Integer[0..] = [ 10, 20, 30 ]
    view := values[0..2]
    values.reserve(100)  // may relocate storage
    inspect(view)        // compile-time error when known; otherwise a checked use that can panic
    ```

  - **Exclusive access to array regions.** Handing disjoint regions of one
    array to different threads, as Rust's `split_at_mut` allows, needs the
    source array and overlapping views to be unable to reach a region while an
    exclusive view of it exists. Constraint from this item: `as` on a slice
    already applies to its elements, so an exclusive view would need a kind
    word after `[]` (by analogy with `* unique`) and a separate way to hand
    itself over.

    ```zax
    // illustrative: no exclusive views exist
    window : MyMessage mutable writable loan[] unique = messages[2..<5]
    other := messages[3..<6]      // nothing forbids this overlap today,
    messages[3] = fresh()         // and the owner still reaches element 3
    ```

  - **What a provably safe mode needs.** A switch forbidding `unsafe`; a
    contract stating what "safe" includes, where data-race freedom is
    currently missing because Zax has no `Send`/`Sync` equivalent or
    exclusivity rule (the qualifiers owner says immutability "is not a
    complete thread-safety guarantee"); and an explicit statement that
    defined panics count as safe outcomes.
  - **Comparative points for the existing "Comparative input" section.**
    Rust prevents mutation while a borrow is active, but not a stale
    assumption carried across a call-out; there its protection is
    visibility of sharing in the types. Bypassing a type's process is an
    encapsulation problem, not a mutability one. C++ safe patterns are
    unchecked, while Rust checks its patterns. Swift enforces exclusive
    access to variables statically where it can and at runtime otherwise,
    which is close to `exclusive` with Zax's runtime-inclusive proof; Swift
    also shows the cost of adding data-race checking (`Sendable`) years after
    launch, which argues for settling thread safety before the rest of the
    language hardens around its absence. Runtime-checked proof guarantees no
    undefined behavior, not no crash, so Zax should state which properties
    it proves at compile time and which it checks at runtime.
  - **The maintainer's position on complexity.** Programmers need to know
    only some of Zax's features to program; most stances, qualifiers, and
    storage controls matter when building enduring libraries and tuning
    performance. The test is that the common path needs few concepts and the
    rest appear only when asked for.

  Reactivates with threading work, which the maintainer intends to take up
  once core aspects of the language are settled.

- **The exact list of proven cases for partial finishing, and the name of its
  `unsafe` category.** The working list is in "Partial finishing". The lasting
  list belongs with future safety-and-analysis work, together with the
  compiler-agreement principle (`project/raw/compiler-agreement-principle.md`)
  and the compiler-verified tag (`project/raw/analysis-controls.md`).
  Reactivates when that work is next reviewed or when a promotion needs the
  exact list. Constraint on current work: promoted teaching may show the
  working list as the current set and must not claim that a compiler may
  accept more.
- **Tracking finished members, and provenance, through source.** Direction
  from the maintainer, for a follow-up work item (proposed as `041` at
  closure):
  - A spec-defined analysis tracks use after `last` through source. A
    finished member that no path touches again is safe. Where the analysis
    cannot tell, `unsafe` is required.
  - The error lands on the later use that causes the problem, and the
    diagnostic also shows where the member was finished.
  - Replacing the member before returning is safe; callers then see nothing.
  - Tracking stops at a compiled, exported ABI. There, the `last` inside the
    exported function becomes the error, unless the function provably
    replaces the member before returning or the site is inside `unsafe<...>`.
  - The same mechanism answers the `unique` concern: a raw pointer made from
    a reference is safe when source tracking proves no path reaches it from a
    `unique` source, an error at a fixed ABI export, and `unsafe` when it
    cannot be tracked.
  - Accepted cost: editing a function body can create an error in a caller,
    because inference through source makes body changes visible downstream.
    Rust avoids this for borrows (its signatures are the whole contract) and
    has the same cost only where body facts leak, such as `Send` through
    `impl Trait` and `async fn`. The diagnostic's pointer back to the origin
    keeps the cost manageable.
  - Open for that work: whether elements should use the same tracking, for
    example a constant index the caller never touches again. Until then the
    aligned element error stays.
  - The specification must define the tracking rules exactly.
  - Prior decision to reconcile: `project/raw/safety.md` records a rejected
    proposal (from work item 038) to warn when a destructor or replacement
    constructor uses a member that the type's own `last` may have taken; how
    a type knows whether a resource is present stays part of its design. This
    direction is different: it tracks uses, outside the type, of a member
    finished through a borrower. The follow-up must keep the type's own
    lifecycle code outside the tracking, unless that rejection's reopen
    condition is met.

  ```zax
  // module: messaging (available as source)
  refresh final : ()(input : Message mutable writable copy &) = {
    keepPayload(input.payload as last)      // origin
  }

  // module: app
  message : Message
  refresh(message)
  message.payload.send()
  // error: use after last
  //   here: message.payload is used after it may have been finished
  //   finished by: refresh, messaging line 2, through input.payload as last
  ```

  Destination: a new indexed raw input,
  `project/raw/usage-and-provenance-tracking.md`, written in the promotion
  change set and linked with `project/raw/analysis-controls.md` (static
  analysis contracts and the compiler-verified tag) and
  `project/raw/pointer-ownership-deferrals.md` (completing `unique`'s
  no-raw-borrowers guarantee). Reactivates with the follow-up work item.
- **The allow-with-tracking principle.** A reusable language principle
  candidate, which the maintainer considers worthy of promotion: prefer
  allowing a construct with spec-defined tracking and a clear diagnostic over
  forbidding it outright, provided the tracking is conservative wherever it
  cannot see. The maintainer's framing of the two extremes: forbidding
  everything risky would, taken to the limit, allow only functions without
  side effects, which is close to perfect but very restricted; allowing
  everything and reporting nothing is the other end. The middle ground lets
  an API author cause some problems, but they are trackable, and the
  language tries to keep the author safe and honest. Partial finishing,
  member tracking, and the `unique` reference case all follow it. It is
  bounded by the compiler-agreement direction: the tracking must be the
  specification's, not each compiler's. Destination: a new indexed raw input,
  `project/raw/allow-with-tracking-principle.md`, written in the promotion
  change set next to `project/raw/compiler-agreement-principle.md`, for
  acceptance into `language/principles.md` when the principles catalog is
  next reviewed.
- **A default stance for stand-alone references and slice parameters.**
  Today the stance must be written, and this item made slice parameters
  require it too (`values : Integer copy[]`), so a learner meets `loan` and
  `copy` early. Each candidate default fails where a learner will notice:

  ```zax
  // if an unmarked reference parameter meant loan:
  show final : ()(value : MyReport &) = {
    keep(value)            // error inside the function just written: a loan can't be copied
  }

  // if it meant copy:
  show final : ()(value : MyReport &) = {
  }
  report : MyReport move
  show(report)             // error at the caller: report forbids copies
  ```

  The maintainer's position: requiring the word is more honest than a default
  that frustrates in other places; the diagnostic ("write `loan` if `show`
  only looks, or `copy` if it keeps a copy") is where the upfront cost is
  repaid. Destination: the existing raw input
  `project/raw/pointer-ownership-deferrals.md`, written in the promotion
  change set. Reactivates if learner evidence shows the required word is the
  larger cost.
- **Learning routes: primers and language bridges.** To be written after the
  main specification is done. Destination: a new indexed raw input,
  `project/raw/learning-routes.md`, written in the promotion change set and
  linked with `project/raw/similarity.md`. It preserves:
  - **A primer** is a step-by-step learning guide. Each step assumes the
    previous steps were read, so it can be short and unrepetitive. It must
    link to owners rather than redefine them, and is costly to keep correct,
    because one language change can affect every later step.
  - **A language bridge** ("if you know X, here is how Zax does it") can be
    entered anywhere, explains Zax in the reader's existing terms, and links
    into owners for the rules. Each bridge claim names where the analogy
    stops, because a near-right analogy misleads.
  - **Why:** an AI trained on Rust already knows Rust's prerequisites; for
    Zax, a reader knows only what it has read, which is why owners repeat
    prerequisites today. A compiler with good diagnostics is the long-term
    fix for AI and human mistakes alike.
  - **The beginner-corpus test:** write ordinary programs (reading a file,
    parsing lines into structs, sorting and filtering, a type with methods, a
    callback) with defaults only, and record which concepts each forces.
  - **Open question:** may owners assume primer knowledge and repeat less?
    That would change the direct-entry rule in `project/documentation.md`
    and needs separate discussion; the agent's lean is no, or only for a
    short, explicitly named baseline.
- **A prerequisite map for concepts.** For each concept, the concepts it
  depends on, so a reader or agent knows what to read first. Destination: a
  new entry in the existing sweep record `project/raw/cross-cutting-audit.md`,
  next to "Example prerequisite and placement audit", written in the
  promotion change set. Activation: the documentation clean-up after the
  main specification.
- **`terms.md` is too code-heavy.** A glossary serves lookup: a definition
  in words plus a link to the owner that teaches the code, with code kept
  only where it explains the term best. Destination: a new entry in
  `project/raw/cross-cutting-audit.md`, written in the promotion change set.
  Activation: the same clean-up.
- **Chunked traversal.** `each chunk : in whole as last chunks of 4` could be a
  convenience over repeated `take` (illustrative spelling). It would be a
  traversal form owned by [iteration](../../language/iteration.md).
  Reactivates when repeated-`take` loops become common in real examples.

### Legacy tensions found, all with a direction

- **Lambda reference captures:** the lambdas owner changes to follow
  transfer stances.
- **`slice as last` rationale:** reworded at promotion.
- **Owning-array qualifiers:** element and array qualifiers become
  independent; examples that relied on the old reading are fixed.
- **`?` spacing:** resolved by the spacing rule; examples are converted.
- **What `move` leaves:** the transfer-stances sentence is corrected to the
  minimum-length rule.

### Likely documentation-owner boundaries

- [Transfer stances](../../language/transfer-stances.md): the element and
  slice rules; the unmarked-outer-stance rule; partial finishing; the
  "Declaring stances" table; every "elements are governed by their type"
  sentence; "Members"; "Arrays and slices", including what `move` leaves;
  "Optional values"; "Callable values and captures"; optional-reference
  rules.
- [Arrays and slices](../../language/arrays-and-slices.md): storage terms;
  element stance and qualifier syntax; slice qualification and narrowing;
  slice parameters (the `Integer[]` examples); `slice as last`; `take` and
  `last X[range]` next to direct splice mutation and `remove`; relocation and
  element-place stability for pinned and immutable elements; what `move` and
  `last` leave; costs; joining.
- [Optional values](../../language/optional-values.md): the source forms, the
  optional-reference examples, the stance section, spacing.
- [Lambdas and callable composition](../../language/lambdas-and-callable-composition.md):
  "Explicit reference capture borrows" and capture spelling.
- [Qualifiers](../../language/qualifiers.md): "Canonical ordering" gains
  array and slice markers and the spacing rule.
- [Operator catalog](../../language/operator-catalog.md): `take X[range]` and
  `last X[range]` as recognized protected forms, and the rule that a
  pre-unary phrase applies before a following post-unary phrase at the same
  level.
- [Mixfix operators](../../language/mixfix-operators.md): a custom `take`
  skeleton, and `last X[range]` as never separately declared.
- [Source structure](../../language/source-structure.md): spacing of `?` and
  `[`, and array-layer order (dimensions, storage clause, qualifiers,
  stance).
- [Generics](../../language/generics.md): deduction drops declared stances.
- [Iteration](../../language/iteration.md): wording about unmarked elements.
- [Function invocation](../../language/function-invocation.md): the
  array-versus-slice overload ambiguity, if its overload section should
  show it.
- Existing raw input `project/raw/array-storage-strategies.md`: add the
  default-profile question; update "Complete-array and range transfer",
  because `take` is now the consuming range operation rooted in the owning
  array (chunk detachment stays there as an optimization); and retire its
  sentence that `slice as last` transfers only descriptor state, which the
  stance-less slice supersedes.
- Existing raw input `project/raw/safety.md`: a new section on exclusive
  access and a provably safe contract, including exclusive access to array
  regions.
- New indexed raw inputs: usage and provenance tracking; the
  allow-with-tracking principle; and learning routes.
- Existing raw input `project/raw/pointer-ownership-deferrals.md`: the
  default-stance question. Existing sweep record
  `project/raw/cross-cutting-audit.md`: the prerequisite map and the
  `terms.md` clean-up.

## Dispositions and promotion dry run

### Promotion, 2026-11-24

The maintainer authorized promotion with teachability as the overriding
concern: guides must serve learning, not only accuracy and completeness. The
change set recorded below was applied: current owners in `language/`, the
root legacy page `compiler-directives.md`, the router `index.md`, three new
raw inputs (`allow-with-tracking-principle.md`,
`usage-and-provenance-tracking.md`, `learning-routes.md`), five updated raw
inputs, and the raw index. Nothing was staged or committed.

**Validation.**

- Every link and heading anchor in the changed files resolves; the checker
  was confirmed to catch a broken anchor.
- No current owner cites numbered work or routes readers into raw input; no
  "governed by their type" wording or spacing that conflicts with the aligned
  rule remains.
- A cold-reader review by an agent that had not read this record found about
  twenty reader problems, mostly in the agent's own examples: examples that
  contradicted their own rules (an unprovable `last` marked fine, a
  replacement example in the wrong order, a member example that undercut its
  justification, an overload example mixing fixed and resizable arrays, a
  slice moved without writable access), phrases describing earlier behavior,
  terms used loosely ("arena-backed", "storage profile"), and exceptions
  stated far from their rules. All were fixed; a second pass by the same
  reader confirmed them, and its last four items were fixed afterward.

**Decisions made during promotion.**

- **`move` on an `immutable` array** keeps its current length instead of
  dropping to its minimum; mutable elements are left moved-from, and
  immutable elements make `move` fall back to copying. Promoted in arrays and
  slices, "What `move` leaves behind", and in the transfer-stances summary.
- **A partially finished array still answers `length()` and capacity
  queries,** because they touch no element. Promoted in arrays and slices,
  "Finishing elements in place".
- **The `terminal` marker,** captured for later examination: an author-side
  marker on a function saying, in the maintainer's words, "this acknowledges
  that `_` might already be terminal when you call it, and that's okay". It
  is not a stance. Captured in `project/raw/usage-and-provenance-tracking.md`
  with an example and the points to settle; array length and capacity are
  noted as the first language-defined instances.

**Maintainer review, 2026-11-24.** The maintainer read the promoted text and
found phrasing the cold reader had passed: code constructs that "write"
stances, pronouns without a clear antecedent ("theirs"), unfinished
consequences ("so less has to be left behind"), places without a referent
("here"), vague verbs ("say so", "worked on", "reaches", "take from"), a
subject drifting between paragraphs, and a self-contradictory-sounding rule
about `unsafe` and compilers that prove more. All quoted cases were rewritten,
and the promoted text was swept for the same patterns. Decisions in the same
review:

- **Storage for new arrays.** The result of `take`, a join, or a by-value
  array parameter gets new storage from the destination's written storage
  clause, otherwise from the `___` context's default, never implicitly from
  the source's provider, because that provider may serve only the source. A
  whole-block hand-over (`as last`, or `as move` with minimum zero) is
  unaffected. Promoted in arrays and slices, transfer stances, and function
  invocation. A scoped override of the context default, suggested by the
  maintainer as a `using`-style block, is captured in
  `project/raw/array-storage-strategies.md`.
- **`unsafe` with compilers that prove more.** Teaching now says a use
  outside the listed cases needs `unsafe` even when a compiler can prove it,
  and that such a compiler can confirm the block with a `[<safe=…>]`
  directive, used as if it exists and marked as still being designed.
- **Documentation guidance.** `project/documentation.md` gained an
  explanation of why these slips recur, with each form and its reason, and
  the validation step now tells the cold reader to use that guidance, flag
  sentences it had to infer, and name what each pronoun in a rule refers to.
- **Scopes that produce a value.** The maintainer's idea, with `break` as the
  preferred keyword for every construct `break` can target, is captured in
  the new raw input `project/raw/scope-expressions.md`, with the legacy
  `basics.md` label spelling noted there.

**Closure, 2026-11-24.** Authorized by the maintainer. Before archival, the
deferrals whose destinations had not yet been written were captured:

- the lasting list of proven cases for partial finishing, and its `unsafe`
  category name, in `project/raw/analysis-control-registry.md` (unassigned
  unsafe pressure);
- handing over a storage block whose elements are pinned, in
  `project/raw/array-storage-strategies.md`;
- the stated `present` overlap, as a note under question 6 of
  `project/raw/present-pointers.md`;
- chunked traversal: dropped, with no capture, on the agent's recommendation,
  because a repeated `take` loop already does the job. Reopen it if
  repeated-`take` loops become common in real examples.

The maintainer also asked that every fresh work item begin by producing a
workable set of decisions with examples and recommendations;
`project/documentation.md` now says so under "The first deliverable is a set
of decisions". Use and provenance tracking was set aside as the next item,
because the maintainer wants the language settled first, threading
especially. Item 041 is the compiler-verified `unsafe` directive
(`[<safe=…>]`) alone, without specific tracking rules. This file was then
marked historical and moved to `project/archive/work/`, the archive index
gained a row, and the current-work index points to 041.

### Re-run, 2026-11-24: PASS

The maintainer aligned A (a reference to a slice writes the element stance
before `[]`; only descriptor qualifiers may follow `[]`), aligned B (correct
the five owning-array uses of `T[]` to owning arrays and reword the
identity-types comment), and accepted the recommendations for C (the split
between arrays and slices and transfer stances; no new owner) and D (the
`index.md` arrays line gains "element and slice stances, `take`").

The re-run rechecked every slice reference and every `T[]` in the corpus:

- the stand-alone slice references are `values : Integer[] loan &` (arrays
  and slices) and `data : Byte[] readonly &` (compiler directives), both
  corrected per A, plus the prose "`T[] &` refers to that description"
  (function invocation), reworded; `alias : Integer[] & = original` is
  already correct, because it is bound to one visible slice;
- exactly five uses of `T[]` mean an owning array: three in compile-time
  code and two in identity types, all corrected per B; every other `T[]` is
  a real slice parameter, which gains an element stance, or a view bound to
  an array.

Every aligned finding has one lasting owner; every affected current owner,
legacy page, raw input, and router is in the change set below; every
deferral has a live destination; the teaching plan covers each changed
section. The capture contradiction across lambdas, qualifiers, transfer
stances, and lifetimes, and the qualifiers sentence on immutable arrays, are
observations that follow from aligned findings and are in the change set.

PASS does not authorize promotion. Promotion is a separate authorized act,
followed by validation that includes a cold-reader review by an agent that
has not read this working record.

The first run's FAIL result follows, unchanged except where the decisions
above resolved it.

### Dry run, 2026-11-24: FAIL (resolved by the re-run)

Read-only. Two design gaps need alignment before promotion can begin, and
two integration choices need confirmation. Everything else has a coherent
owner, reading path, deferred destination, and change set, recorded below so
that a re-run only has to check the corrections.

#### Blocking findings

**A. A stand-alone reference to a slice has no rule.** The slice descriptor
has no stance (aligned), but references that stand alone must write the
stance of the value they refer to, and for a reference to a slice that value
is the descriptor. Current owners write a stance or qualifier where the
descriptor's would go:

```zax
narrow final : ()(values : Integer[] loan &) = {     // arrays and slices, "Passing a slice"
}
legacyChecksum final : (result : U32)(data : Byte[] readonly &) = {   // compiler directives
}
```

Candidate for alignment: a reference to a slice writes the element stance
before `[]`, which satisfies the stand-alone rule, and nothing between `[]`
and `&` except descriptor qualifiers. A reference bound to one visible slice
takes the element stance from that slice, as the existing narrowing example
`alias : Integer[] & = original` already does.

```zax
narrow final : ()(values : Integer loan[] &) = {                    // may narrow the caller's slice
}
legacyChecksum final : (result : U32)(data : Byte readonly loan[] readonly &) = {
}                                                                   // elements read-only; window can't be narrowed
```

**B. Several current owners use `T[]` as an owning array.** Under the arrays
owner, `T[]` is always a slice. These examples build, return, or declare
storage, which a slice cannot own, and identity types relies on wording this
item supersedes:

```zax
buildSineTable final [<execute=compile>] : (result : Integer[])(size : Integer) = {   // compile-time code
}
myTable : Integer[] = buildSineTable(myTableSize)                                      // compile-time code
makeBuffer$ final : (UseType : type)(result : Byte[])(count : Integer) = {             // compile-time code
}
reports : MyType[] = [ report as move ]       // identity types: a slice of a temporary
held : MyMoveOnlyType[]                       // identity types: a slice with nothing to show
copied := reports[0]              // fine: array elements are governed by MyType
```

Candidate for alignment: the promotion change set corrects these to owning
arrays (`Integer[0..]`, `Byte[0..]`, `MyType[0..]`), which is their evident
intent, and rewrites the identity-types comment to "unmarked elements are
`copy`; `MyType` governs whether a copy is possible". The alternative is to
log the compile-time-code cases in the cross-cutting audit and fix only
identity types, which this promotion must touch anyway.

#### Integration choices to confirm

**C. Where the array-specific findings live.** Proposed structure:

- [Arrays and slices](../../language/arrays-and-slices.md) owns array and
  slice behavior: element and slice stance syntax, storage terms, relocation,
  what `move` and `last` leave, partial finishing of elements, replacing
  terminated elements, `take` and `last X[range]`, the extracted array's type,
  slices to by-value arrays, and the array-versus-slice overload consequence.
- [Transfer stances](../../language/transfer-stances.md) owns the model and
  the cross-cutting rules: one stance per layer including arrays and slices,
  the unmarked-outer-stance rule, inference and deduction, element stances
  inside reference types, finishing elements through a borrowed array, the
  member contrast, optional-reference rules, the optional source forms'
  meaning, captures, and a short "Arrays and slices" section that teaches the
  element model and hands off to arrays and slices.

The alternative, a new dedicated owner for element and contained stances,
is not proposed: arrays and slices already teaches array storage, transfer,
and parameters cohesively, and transfer stances already hands off to it.

**D. A routing sentence in `index.md`.** The arrays line ("fixed and resizable
arrays, array expressions, usable capacity, storage strategies, indexing,
slicing, element places, joining, and traversal") would add "element and
slice stances, `take`". Optional; routers need only enough to choose a
document.

#### Ownership map

| Aligned finding | Lasting owner | Also changed |
| --- | --- | --- |
| Storage terms | Arrays and slices, "Storage strategies" | Terms ("Array storage provider and profile") |
| Element stance and member-like elements | Arrays and slices; transfer stances (model) | Qualifiers ("Canonical ordering") |
| Unmarked outer stance; inference; deduction | Transfer stances, "Declaring stances" | Optional values (stance section); generics (deduction) |
| What `move` and `last` leave | Arrays and slices, "Copy, deep, move, and last" | Transfer stances ("Arrays and slices" sentence corrected) |
| Relocation inside an array | Arrays and slices, "Element-place stability" | Transfer stances (pinned values) |
| Qualifiers on arrays and elements | Arrays and slices, "Qualifying an owning array" | Qualifiers ("Array and slice qualification") |
| Spacing and array-layer order | Source structure | Qualifiers ("Canonical ordering"); every affected example |
| Slices with no stance | Arrays and slices, slices and parameters; transfer stances | Terms ("Slice and splice"); function invocation (array and slice parameters) |
| What a `last` view allows; partial finishing; replacing terminated elements | Arrays and slices (new section) | Transfer stances (summary and hand-off) |
| `take` and `last X[range]`, extracted type, recognition | Arrays and slices (new section next to direct splice mutation) | Operator catalog; mixfix operators |
| Element stances inside a reference type | Transfer stances, "Passing a value to a reference" | Arrays and slices ("Passing an array by reference") |
| Finishing through a borrowed array; members keep today's rule | Transfer stances, "Members" | Arrays and slices (hand-off) |
| Arrays passed to slice parameters; overloads; no slice-to-array conversion | Arrays and slices, "Array and slice parameters" | Function invocation (overload section, one example) |
| The optional's source forms | Optional values, "Move and last source adapters" | Transfer stances ("Optional values"); operator catalog (rows) |
| Optional references | Optional values, "Optional references" | Transfer stances ("References that stand alone") |
| Lambda reference captures | Lambdas, "Explicit reference capture borrows" | Qualifiers (capture paragraph); transfer stances ("Callable values and captures") |
| Iteration wording | Iteration, typed loop bindings | none |

#### Affected files: exact promotion change set (A to D resolved as above)

Current owners in `language/`:

- `transfer-stances.md`: "Where a stance is written" (array and slice
  layers); "Declaring stances" table and "Locals and members" (remove "array
  elements are governed by their type"; unmarked outer stance; inference);
  "References you declare" (loop-binding sentence); "References that stand
  alone" (slice parameters, references to slices, optional references);
  "Passing a value to a reference" (inner stances); "Members" (borrowed-array
  contrast; members keep today's rule); "Callable values and captures";
  "Optional values" (source forms, spacing); "Arrays and slices" (element
  model, what `move` leaves, hand-off); metadata `Owns`.
- `arrays-and-slices.md`: "Fixed arrays, resizable arrays, and slices"
  (element stance, slice with no stance); "Qualifying an owning array" and
  its example; "Slice qualification and narrowing"; "Element-place
  stability" (relocation, pinned and immutable elements); "Storage
  strategies" (storage terms); "Copy, deep, move, and last" (minimum-length
  rule; `slice as last` rewording); new sections for finishing elements and
  for `take` and `last X[range]`; "Array and slice parameters" (by-reference
  element stances, slice parameters, references to slices, arrays to slice
  parameters, overloads, no slice-to-array conversion); costs and
  diagnostics; metadata `Owns`.
- `optional-values.md`: source forms and borrowers; optional references and
  their examples; composed forms marked illustrative or given stances;
  stance section; `MyPayload move? loan`, `MyValue * unique?`, and
  `T q0? q1 * q2` spacing.
- `lambdas-and-callable-composition.md`: "Explicit reference capture borrows"
  (the table, explicit spelling, the wider window for `move &` captures).
- `qualifiers.md`: "Canonical ordering" (array and slice markers, array-layer
  order); "Array and slice qualification" (independent element qualifiers,
  replacing "making the array immutable also constrains contained element
  state"); the capture paragraph ("a `loan` when no stance is written"); the
  `T q0 ? q1 * q2` schematic.
- `source-structure.md`: "Optional layers and empty construction packets"
  and the array-type material (spacing of `?` and `[`, required; array-layer
  order).
- `operator-catalog.md`: `take X[range]` and `last X[range]` rows; the
  pre-unary-before-post-unary rule; `last value` row wording beside the new
  form.
- `mixfix-operators.md`: a custom `take` skeleton; `last X[range]` derived.
- `function-invocation.md`: "Array values, bounded references, and slices"
  (`T[] &` wording; slice element stances); the overload example.
- `generics.md`: deduction drops declared stances.
- `iteration.md`: typed loop-binding wording.
- `terms.md`: "Array storage provider and profile" (inline and
  provider-backed); "Slice and splice" (no stance; `take`); "Declared
  stance" and "Owner and borrower" (slices, element stances); "Pinned value"
  (pinned elements and relocation); a new entry for partially finished.
- `variadics.md`, `construction-and-destruction.md`: slice parameters gain an
  element stance (`values : Integer loan[]`, `candidates : Item loan[]`,
  `queries : Query copy[]`).
- `compiler-directives.md` (current owner): `data : Byte[] readonly &`, per A.
- `identity-types.md`, `compile-time-code.md`: per B.

Root legacy page:

- `compiler-directives.md`: `inputs : String[]` gains an element stance
  (`String readonly loan[]`).

Raw inputs (written in the same change set, each indexed in
`project/raw/README.md`):

- new `usage-and-provenance-tracking.md`, `allow-with-tracking-principle.md`,
  and `learning-routes.md`;
- `array-storage-strategies.md`: default-profile question; "Complete-array
  and range transfer" updated for `take`; the `slice as last` sentence
  retired;
- `pointer-ownership-deferrals.md`: default-stance question;
- `safety.md`: exclusive access and a provably safe contract;
- `cross-cutting-audit.md`: prerequisite map; `terms.md` clean-up;
- `similarity.md`: a link to learning routes.

Router: `index.md`, per D.

Unchanged: `project/README.md` (changes only at closure), the archive, and
both operating-prompt sources.

#### Teaching plan for each changed owner section

- **Arrays and slices, element stances.** The reader is learning arrays and
  knows stances from transfer stances. What changes: an element stance before
  `[`, and elements handled like members. They might wrongly expect the
  array's stance to restrict its elements, or `immutable` after `]` to
  freeze the elements (the C++ `const std::array` expectation).
- **Arrays and slices, slices.** The reader knows references. What changes:
  a slice has no stance; each element is reached as `<stance> &`; parameters
  write the element stance. They might write `values : Integer[]` and expect
  it to work, or read `window as last` as finishing the window.
- **Arrays and slices, finishing elements and `take`.** The reader knows
  `last`. What changes: partial finishing, proven cases, replacing
  terminated elements, and `take` as the way to keep the array live. They
  might expect `finish(messages[i] as last)` in a loop to need nothing, or
  `take` on a slice to accept any range.
- **Arrays and slices, storage and `move`.** The reader may never have
  chosen storage. What changes: inline versus provider-backed terms, the
  minimum-length rule, relocation. They might expect a moved-from array to
  be empty, as a moved-from `std::vector` is in practice.
- **Transfer stances, model sections.** The reader knows owners and
  borrowers. What changes: arrays and slices join the layer model; unmarked
  outer stances; deduction. They might expect an alias stance to vanish on
  `OwnedPayload[4]`.
- **Transfer stances, "Members".** The reader knows the member rule. What
  changes: elements are stricter through borrowed arrays. They might expect
  elements and members to behave the same.
- **Optional values.** The reader knows the source forms. What changes: they
  are stances on the payload, with borrower rules, and optional references
  always write a stance. They might expect `cache. as last` through a
  reference to be allowed.
- **Lambdas.** The reader knows reference captures. What changes: an
  unmarked capture follows the references-you-declare table. They might
  expect every reference capture to be a `loan`.
- **Qualifiers, source structure, operator catalog, mixfix, generics,
  iteration, function invocation, terms.** Local rule changes with one example
  each; the reader arrives for one rule and needs that rule plus a link to
  arrays and slices or transfer stances.

#### Deferred material, all with destinations

Default storage profile; block hand-over with pinned elements; exclusive
access and a provably safe contract; the list of proven cases; member and
provenance tracking; the allow-with-tracking principle; the default stance;
learning routes; the prerequisite map; the `terms.md` clean-up; chunked
traversal. Each is listed under Deferred with its destination and
reactivation pressure. Teaching-debt observations: none recorded.

#### To pass on re-run

Align A and B, and confirm C and D. Done; see the re-run result above.
