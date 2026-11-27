# Zax conversions and casts

| Field | Value |
| --- | --- |
| Status | Current conceptual design |
| Audience | Human developers converting values between types or reinterpreting memory |
| Applies To | Choosing among conversion, stance restatement, structural compatibility, container and member navigation, and unchecked reinterpretation; programmer-declared `as`; `unsafe cast`; pointer and integer conversion; not a formal grammar or specification |
| Implementation State | Not established by this repository |
| Owns | The overview of conversion and cast operations; programmer-declared `as` conversion; `unsafe cast` forms, vacancy, qualification, ownership-role, cost, and diagnostic behavior; the rule that a type cast never changes a stance; `unsafe cast as` for changing a stance; conversion between pointers and pointer-representation integers |
| Does Not Own | Numeric conversion ([integers](integers.md), [fixed-point](fixed-point-scalars.md), [floating-point](floating-point-scalars.md)); admission and projection ([identity types](identity-types.md), [enums](enums.md)); stance restatement ([transfer stances](transfer-stances.md)); compatibility, coercion, and structural applications of `unsafe cast` ([structural shapes](structural-shapes-and-compatibility.md)); outer casting ([composition](composition.md)); interior ownership ([pointers](pointers-and-arenas.md)); dereference ([Nothing instances](nothing-instances.md)); exact forms ([operator catalog](operator-catalog.md)) |
| Source / Provenance | Legacy casting material from the retired root casting page, refined against current operator, structural, pointer, qualifier, and scalar design |
| Supersedes | The retired root casting page |

## Zax has no universal cast

Zax offers several operations where many languages offer one cast, because
speed, safety, and efficiency matter more here than one flexible spelling. You
choose the operation that matches your intent, from a free reinterpretation of
existing memory to a conversion that builds a new value. Each of these lines
asks a different question:

```zax
display := boiling as Fahrenheit            // Celsius chose to provide this conversion
myByte := myCount narrowing as U8           // the language's modular integer conversion
header : PacketHeader * =
  myBuffer unsafe cast PacketHeader *       // treat these bytes as a header; nothing is checked
```

The first line runs code that the `Celsius` type wrote. The second follows the
integer rules for keeping only the low bits. The third converts nothing at all:
it tells the compiler to treat existing memory as a different type, and the
programmer takes responsibility for that being valid.

Keeping these operations apart lets a reader see the cost and the risk of each
line. A conversion can run code. Restating a stance moves nothing by itself. An
unchecked reinterpretation carries `unsafe` in its spelling.

`as` is an ordinary operator word. Most `as` operations are operators that a
type chooses to declare. The language owns only a small set of exact `as ...`
forms, such as stance restatement and structural compatibility, plus the
language-provided conversions of its own scalar types.

## Which operation do I want?

| I want to... | Representative source | Taught by |
| --- | --- | --- |
| Convert a number to another numeric type | `myWide := myU16 as U32`, `myMaybe := myU16 as U8?` | [integers](integers.md#conversion-to-exact-intrinsic-integers), [fixed-point](fixed-point-scalars.md), [floating-point](floating-point-scalars.md#numeric-conversion) |
| Convert my type to another type | `display := boiling as Fahrenheit` | [Writing your own `as`](#writing-your-own-as) below |
| Wrap a value in an identity or enum type | `myCount := MyCount from myRaw` | [identity types](identity-types.md#admission), [enums](enums.md#generated-boundary-operations) |
| Hand a value to a consumer as a move | `consume(myBuffer as move)` | [transfer stances](transfer-stances.md) |
| Use compatible storage as another named type | `pixel : PixelPoint = kilometers as shape PixelPoint` | [structural shapes](structural-shapes-and-compatibility.md#protected-compatibility-conversions) |
| Get the container from a member | `car : Car & = engine outer cast Car.engine` | [composition](composition.md#outer-casting-to-an-immediate-container) |
| Own a member of a shared allocation | `engine : Engine * strong = inner car.engine` | [pointers](pointers-and-arenas.md#interior-pointers) |
| Take an address, or follow a pointer | `pointer : MyValue * = value`, `view : MyValue & = pointer.` | [declarations](declarations-and-bindings.md), [Nothing instances](nothing-instances.md#pointer-dereference-is-unchecked) |
| Store an address as an integer | `myAddress : UPointer = myPointer as UPointer` | [Pointers and integers](#pointers-and-integers) below |
| Reinterpret memory without checks | `header : PacketHeader * = myBuffer unsafe cast PacketHeader *` | [Unchecked reinterpretation](#unchecked-reinterpretation-with-unsafe-cast) below |
| Use a value with a stance it wasn't declared with | `restored : MyValue * last = (view unsafe cast as last) as last` | [Changing a stance with `unsafe cast as`](#changing-a-stance-with-unsafe-cast-as) below |

Taking an address needs no cast. A pointer initialized from a value takes that
value's address, and a trailing `.` follows a pointer to its target. Both rules
are explained where they are owned.

## Writing your own `as`

A type provides a conversion by declaring binary `as` with a type parameter
slot:

```zax
Fahrenheit :: type {
  degrees : Binary64
}

Celsius :: type {
  degrees : Binary64

  operator binary 'as' final : (
    result : DestinationType
  )(
    DestinationType : type
  ) readonly = {
    result .= [{
      .degrees = degrees * 9.0 / 5.0 + 32.0
    }]
  }
}

boiling : Celsius = [{
  .degrees = 100.0
}]

display := boiling as Fahrenheit
```

The left value's type owns the conversion, so `boiling as Fahrenheit` finds the
operator on `Celsius`. `DestinationType` is a
[type argument](operator-phrases.md#type-arguments-and-type-qualified-operators):
it names a type and has no runtime storage, lifetime, or evaluation.

The result starts unconstructed. `result .= [{ ... }]` constructs it from the
packet, just as the same packet would initialize a declaration. Writing
`result.degrees = ...` first would be an error, because ordinary `=` never
constructs. See
[constructing a result later](function-invocation.md#opt-in-result-initialization).

A plain `type` slot accepts any type, so the declaration above is also
selected for an unrelated destination such as `String`, and then fails inside
its body. A [constraint](generics.md#constraints) states which destinations the
conversion accepts:

```zax
isTemperature final : (result : Boolean)(UseType : type) = {
  return UseType == Fahrenheit
}

Celsius :: type {
  degrees : Binary64

  operator binary 'as' final : (
    result : DestinationType
  )(
    DestinationType : isTemperature
  ) readonly = {
    result .= [{
      .degrees = degrees * 9.0 / 5.0 + 32.0
    }]
  }
}

display := boiling as Fahrenheit   // accepted
label := boiling as String         // error: no viable as; isTemperature(String) is false
```

The constraint lists each destination the body can produce. A destination it
rejects never reaches the body, and the error names the constraint.

A declaration may name a pointer or reference type as its destination. The
result then follows the ordinary rules for
[returned references](lifetimes-and-references.md#returned-references). There
is no built-in `as MyType *`; initializing a pointer from a value already takes
its address.

A type may also declare longer phrases that begin with `as`, such as
`as json text`. They work, but a lint discourages them: the language may add
new exact `as ...` forms later, and a collision would change what existing
source means. Plain binary `as` with a type argument is always safe to declare
on your own types.

Words inside a phrase are phrase words, not keywords. A type may name an
operation `unsafe as` to warn its readers, but that word grants no unsafe
authority. The language's own unchecked reinterpretation is `unsafe cast`.

### Your types and the language's scalars

Integers, fixed-point, and floating-point conversions belong to the language.
Your type can still convert to and from them:

```zax
myCount := myTally as I32                  // MyTally declares its own `as`

myTally : MyTally = [{
  .count = myCount                         // build a MyTally by construction
}]

myId : UserId = UserId from myRaw          // an identity type admits its underlying value
```

A scalar value is the receiver of `myCount as MyTally`, so `MyTally` cannot
supply that conversion. Use construction instead, or an identity type's
admission.

A [partial](partials.md) does not solve this. A partial can add operators to
`I32`, but an `as` declaration accepts every destination type, so a partial
`as` on `I32` would also claim `myCount as I64`, which belongs to the language.
It becomes a practical route only once a declaration can restrict its
destination type.

## Unchecked reinterpretation with `unsafe cast`

`unsafe cast` tells the compiler to treat existing memory as another type
without converting or checking anything. The result is always a view, a pointer
or a reference, never a new value:

```zax
otherPointer := myPointer unsafe cast OtherType *     // pointer to pointer
valueView := myValue unsafe cast OtherType &          // a value's storage viewed as OtherType
referenceView := myReference unsafe cast OtherType &  // reference to reference
```

The cast verifies no compatibility, qualification, invariant, lifetime, bounds,
alignment, or layout guarantee. Those facts become the programmer's
responsibility, and a false claim has undefined consequences. The compiler
still rejects a reinterpretation it proves impossible, even when `unsafe` is
written.

Before reaching for `unsafe cast`, check whether a checked operation expresses
the same intent. The
[structural compatibility and coercion forms](structural-shapes-and-compatibility.md#coercive-reference-views)
prove layout and validity facts for related types, and a declared `as` performs
real conversion.

### New values come from construction

A cast produces no new value, so there is no by-value form. To obtain a new
value, form the view and construct from it:

```zax
copy := myValue unsafe cast OtherType      // error: unsafe cast produces a view
copy : OtherType =
  myValue unsafe cast OtherType &          // view the storage, then construct a copy
```

A by-value spelling could not say whether it meant a byte copy, a constructor
call, truncation, or the start of a new lifetime. Separating the view from
construction keeps each responsibility visible: the cast owns the unchecked
reinterpretation, and construction owns copying, resources, and the new
lifetime.

### Vacant pointers stay vacant

A vacant pointer converts to a vacant pointer of the destination type:

```zax
vacantPointer : MyType *
otherPointer := vacantPointer unsafe cast OtherType *
// ?otherPointer is false: still vacant
```

Each type may represent vacancy differently, because a vacant pointer targets
its own type's [Nothing instance](nothing-instances.md). Without this rule, the
result would point at `MyType`'s Nothing backing while claiming to be an
`OtherType` pointer, and a presence test would report it as present. A
non-vacant address is reinterpreted unchanged.

To keep a vacant pointer's exact raw representation instead, convert it to a
[pointer-representation integer](#pointers-and-integers) and back. That round
trip does not preserve vacancy.

### Qualification

The written destination type states the complete qualification of the result.
`unsafe cast` can therefore remove `immutable`, `readonly`, or `final`:

```zax
frozen : MyType immutable = makeValue()
writable := frozen unsafe cast MyType mutable writable &   // you take responsibility for every holder of frozen
```

Adding `immutable` does not need `unsafe cast`. An `immutable` path promises
that the value does not change while the path is held, so a mutable value can
make that promise when analysis proves nothing writes it during that time, or
when a `safe<no-mutation-while-held>` claim vouches for it:

```zax
myMutable : MyType = makeValue()

asReadonly : MyType readonly & = myMutable            // ordinary: readonly narrows one path
asImmutable : MyType immutable & = myMutable          // fine if nothing writes myMutable while asImmutable is held
```

[Qualifiers](qualifiers.md#ordinary-promise-strengthening) describes the
complete rule.

When the only goal is to change qualification on the same type, prefer
[`unsafe pliable`](qualifiers.md#unsafe-pliable). It keeps the recorded
qualifications and bypasses them locally for one explicit path, while
`unsafe cast` creates a differently qualified view that later code cannot
distinguish from an ordinary one.

### A type cast never changes a stance

The destination type states the type and the qualifiers, but never a stance.
`x unsafe cast T` keeps every stance `x` has, and writing a different stance in
`T` is an error:

```zax
view : MyValue loan * = myValue
other := view unsafe cast MyOther *           // other is MyOther loan *: the stance is kept
wrong := view unsafe cast MyOther copy *      // error: a type cast can't change a stance
```

A cast that changed the type and the stance together would hide mistakes: a
typo in the type would go unnoticed while the stance changed. Changing a
stance has its own cast,
[`unsafe cast as`](#changing-a-stance-with-unsafe-cast-as).

### Pointer ownership roles

For raw, `unique`, `unique shareable`, `strong`, and `weak` pointers, a type
cast may change the pointee type, the qualifiers, and the pointer kind, the
last under the ordinary transition rules. It never changes a stance. The
change of pointer kind and every lifetime rule are exactly those of the
ordinary [pointer conversions](pointers-and-arenas.md#ownership-transitions):

```zax
myCar : Car * strong = makeCar()

shared : Truck * strong = myCar unsafe cast Truck * strong               // another strong owner
observer : Truck * weak = myCar unsafe cast Truck * weak                 // ordinary strong to weak
exclusive : Truck * unique = (myCar as last) unsafe cast Truck * unique  // ordinary unique-claim rules
```

In the last line, `myCar as last` is evaluated first and the cast applies to
it. Its `as last` is the stance written for the transfer, as on any
transfer, not a stance the cast changes.

A raw pointer owns its allocation because of its stance, `move` or `last`, so
a type cast can't make a borrowing raw pointer into an owner. That takes the
stance cast:

```zax
owner : MyValue * last = @
raw : MyValue * = owner.                  // a borrowing raw pointer
unsafe vacate owner                       // owner gives up responsibility without freeing

// each line below is a separate alternative
owned : MyValue * last = raw unsafe cast MyValue * last       // error: a type cast can't change a stance
owned : MyValue * last = (raw unsafe cast as last) as last    // the stance cast; the type is unchanged
```

A pointer with a control block (`strong`, `weak`, or `unique shareable`) still
disposes the allocation its control block records. Viewing a `Car` allocation
as a `Truck` does not make release destroy a `Truck`.

A blockless `unique` pointer may also change its pointee type. Whether that is
valid depends on the program. The compiler cannot tell, which is why the form
requires `unsafe`. Which type's destruction runs when a retyped blockless
`unique` releases is not yet specified. Code that retypes one takes
responsibility for correct disposal.

## Changing a stance with `unsafe cast as`

A stance says what may be built from a value. Sometimes low-level code knows
more than the stances say: a container managing its own storage, interop where
a foreign API's real behavior is known, or ownership given up earlier with
`unsafe vacate` and now taken back. `x unsafe cast as S` gives `x` itself with
the stance `S` on its outermost layer. The type and the qualifiers stay the
same:

```zax
owner : MyValue * last = @
view : MyValue * = owner.                    // a borrowing raw pointer
unsafe vacate owner                          // owner gives up responsibility without freeing
restored : MyValue * last = (view unsafe cast as last) as last
// the cast gives view itself with the stance last; writing as last hands it
// over, so restored takes over the allocation and will free it.
// view counts as finished afterward, so it can't be used by accident.
```

Writing `unsafe` means the programmer takes responsibility for what the stance
protected. Here that is the assertion that `view` points at the start of an
allocation that nothing else will free, as
[owning raw pointers](pointers-and-arenas.md#owning-raw-pointers) explains.
The cast works the same way on any stance, including the stance a reference
parameter was given:

```zax
remember final : ()(input : MyMessage copy &) = {
  keep(input as last)                          // error: input was given copy
  keep((input unsafe cast as last) as last)    // compiles: the function claims last on the
                                               // caller's value. The caller still believes it
                                               // is unchanged; keeping that true is now the
                                               // programmer's responsibility
}
```

A reference has no layer of its own, so its outermost stance is the stance of
the value it refers to. A pointer's inner layers are reached through `.`, one
layer per cast. Making a pointer from the result keeps the address, vacant
included:

```zax
lent : MyValue loan * = myValue
widened : MyValue copy * = (lent.) unsafe cast as copy
// the MyValue that lent points to, seen as copy instead of loan
```

The cast gives the same value with a new stance, not a new value, so a
transfer still writes its stance, as in `(view unsafe cast as last) as last`.
Four consequences follow:

1. **It changes the stance, not which operations exist.** The type's
   `transfer` list still decides what exists, and fallback still applies:

   ```zax
   MyPoint :: type transfer copy {
     x : Integer
   }

   pinned : MyPoint loan
   moved := (pinned unsafe cast as move) as move   // compiles: MyPoint has no move, so copy runs
   deeper := (pinned unsafe cast as deep) as deep  // error: MyPoint has no deep, and deep never falls back
   ```

2. **A cast that only narrows is legal but pointless.** Ordinary binding
   already narrows a stance. A compiler may warn that the `unsafe` is
   unnecessary, but it can't reject the cast:

   ```zax
   plain : MyValue copy * = myValue
   lent : MyValue loan * = plain                       // ordinary narrowing: no unsafe needed
   lent2 : MyValue loan * = plain unsafe cast as loan  // legal; may draw a warning
   ```

3. **It changes stances, never qualifiers.** Making a `readonly` path writable
   stays with [`unsafe pliable`](qualifiers.md#unsafe-pliable) and the type
   cast:

   ```zax
   reader : MyValue readonly copy & = myValue
   taken := (reader unsafe cast as last) as last   // the stance becomes last, but the path is
                                                   // still readonly: a last or move constructor
                                                   // that needs a writable source can't accept
                                                   // it, so fallback reaches copy, which runs
   ```

4. **A pinned value can be moved.** Other code may rely on a pinned value's
   address staying fixed. Having written `unsafe`, the programmer gets what
   they asked for:

   ```zax
   device : MyDevice loan
   relocated := (device unsafe cast as move) as move  // compiles; pointers to device may now be stale
   ```

## Pointers and integers

An address can be stored as an integer and turned back into a pointer:

```zax
myAddress : UPointer = myPointer as UPointer               // observe the address
myRestored : MyType * = myAddress unsafe cast MyType *     // assert it is a valid MyType address
```

Converting a pointer to an integer is safe. Observing an address cannot break
memory safety. The integer carries no provenance, though, so it proves nothing
if it is later turned back into a pointer.

Converting an integer to a pointer is unchecked reinterpretation. The
programmer asserts that the address is valid for the destination type, suitably
aligned, and live for every use.

- The integer must match the pointer's memory domain: `UPointer` for ordinary
  pointers, `Near.UPointer` for near pointers, and `Far.UPointer` for far
  pointers. See
  [pointer-representation integer roles](integers.md#pointer-representation-integer-roles).
- A vacant pointer converts to its raw representation. That value is not
  portable, and converting it back produces a pointer that is not vacant.
- The pointer it produces is a borrowing raw pointer. To own the allocation
  at that address, also change its stance with
  [`unsafe cast as last`](#changing-a-stance-with-unsafe-cast-as).
- The difference between two addresses uses `PointerDelta`, the exact
  difference type of `UPointer`.

## Costs

- A declared `as` costs whatever its body does, including allocation or other
  resource work.
- `unsafe cast` performs no conversion work. Preserving vacancy may need one
  test of the source pointer. That test disappears when the source and
  destination use the same vacant representation, or when the compiler proves
  that the source is never vacant.
- `myPointer as UPointer` and integer-to-pointer `unsafe cast` copy an address.
- `unsafe cast as` changes only what the compiler allows; it does no work at
  run time.
- Every `unsafe` form adds audit cost: a reviewer must confirm the facts the
  compiler did not check, and the result may be less portable across targets.

## Diagnostics

Useful diagnostics include:

- by-value `unsafe cast`, with the view-then-construct correction;
- a reinterpretation the compiler proves impossible, even under `unsafe`;
- an ordinary view that tries to remove a qualification, with
  `unsafe pliable` and `unsafe cast` shown as the explicit alternatives;
- an ordinary view that adds `immutable` while analysis cannot rule out a
  write, with `safe<no-mutation-while-held>` shown as the claim;
- a pointer-to-integer conversion whose integer role does not match the
  pointer's memory domain;
- a type cast whose destination writes a different stance, with
  `unsafe cast as` shown as the explicit alternative;
- a transfer after `unsafe cast as` that needs an operation the type doesn't
  have, such as `as deep` on a type without `deep`, saying that the cast
  changed the stance but can't create the operation;
- an `unsafe cast as` that only narrows a stance, as an optional warning;
- a declared `as` selected for a destination its body cannot produce; and
- a lint for programmer-declared phrases that begin with `as`.

## Boundaries and maturity

These concerns remain future work:

- restricting which destination types one `as` declaration accepts;
- adding operators to the language's scalar types from programmer code;
- helpers that view raw memory as an array or iterator of bytes;
- pointer provenance, alignment, segment, and comparability rules beyond the
  conversions described here; and
- which type a retyped blockless `unique` destroys.
