# Zax variadics

| Field | Value |
| --- | --- |
| Status | Current conceptual design |
| Audience | Human developers writing, calling, or reading functions that accept any number of arguments |
| Applies To | Variadic parameters and packs: declaring, calling, walking, counting, indexing, passing on, transforming, returning, holding, checking, selecting, and capturing them |
| Implementation State | Not established by this repository |
| Owns | The pack model and how its pieces fit together; teaching for every variadic task; variadic costs, diagnostics, and source-stability consequences |
| Does Not Own | Call mapping around a pack, overload rules, and result-pack slots ([function invocation](function-invocation.md#inputs-around-a-variadic-pack)); variadic `$` parameters and every-element constraints ([generics](generics.md#a-variable-number-of-compile-time-parameters)); `each` traversal rules ([iteration](iteration.md#packs)); stance meaning ([transfer stances](transfer-stances.md#a-stance-on-several-items)) |
| Source / Provenance | The retired legacy variadic-functions page and legacy compile-time variadic catalog input, rebuilt on current function-invocation, generics, iteration, and transfer-stance design |

## Accept any number of arguments

Some functions should take as many arguments as the caller has: a sum of
numbers, a print of mixed values, a log line with any number of fields. Here are
two such functions and a few calls to them:

```zax
write final : ()(text : String) = {
}

write final : ()(number : Integer) = {
}

write final : ()(flag : Boolean) = {
}

sum final : (result : Integer)(values ... : Integer) = {
  total := 0
  each value : in values {
    total += value
  }
  return total
}

print$ final : (Items ... : type)()(items ... : Items) = {
  each item : in items {
    write(item)                  // selects the write for this item's type
  }
}

myTotal := sum(1, 2, 3)          // sum compiled for three values
myOther := sum(4, 5)             // and separately for two
print("total:", myTotal, true)   // print compiled for String, Integer, Boolean
```

`values ... : Integer` accepts any number of arguments, all of them `Integer`.
`items ... : Items` accepts any number of arguments, and each one may have its
own type.

`print$` is a [generic](generics.md#a--list-of-compile-time-parameters)
function, so its declaration has three parenthesized lists: compile-time
parameters, then results, then inputs. `Items ... : type` is a list of types,
one per argument. The caller usually doesn't write that list. The compiler
**works it out** from the arguments at each call, as described in
[working out from arguments](generics.md#working-out-from-arguments).

What happens at each call is the key to the whole feature. The compiler
compiles `sum` separately for each number of arguments it is called with, and
compiles `print` separately for each list of argument types. In the copy of
`sum` made for three arguments, `values` is simply three `Integer` parameters,
as if you had written the function with three. In the copy made for
`print("total:", myTotal, true)`, the loop body is checked three times: once
with `item` as a `String`, once as an `Integer`, and once as a `Boolean`. That
is how each `write(item)` can pick the right overload while compiling.

So keep this cost in mind from the start: **each distinct argument count, or
each distinct list of argument types, is another compiled copy of the
function.** Code size grows with the variety of calls, not with how often the
function is called. In return, nothing about the arguments is looked up at
runtime. There is no hidden array, no untyped argument list to walk, and no
allocation.

This page calls each compiled copy a **processing** of the function. It is the
same [processed once per input](generics.md#when-code-is-processed-once-per-input)
rule that generics follow for instantiations.

The arguments a variadic parameter receives are called a **pack**: separate
bindings that the function reaches through one name. A pack is not one value.
It is not an array, a slice, or a tuple, even when every item has the same
type, and its count is always known at compile time.

Later examples on this page reuse the `write` overloads declared here.

## Declaring a variadic parameter

To make a parameter variadic, write `...` between its name and its colon:
`name ... : Type`. The canonical spelling puts one space on each side of the
marker, as described in
[the variadic `...` marker](source-structure.md#the-variadic--marker).

### Three kinds of pack

Packs come in three kinds, depending on what the declaration says about the
items' types. That choice also decides whether the function needs a `$` list:

```zax
sum final : (result : Integer)(values ... : Integer) = {
}

largest$ final : (UseType : type)(result : UseType)(values ... : UseType) = {
}

print$ final : (Items ... : type)()(items ... : Items) = {
}
```

| Items | Declaration | Compiled once per |
| --- | --- | --- |
| One stated type | `values ... : Integer`, no `$` list | Distinct count |
| One shared type worked out from the arguments | `values ... : UseType` with an ordinary `$` parameter | Distinct element type and count |
| Each item its own type | `items ... : Items` with a variadic `$` parameter `Items ... : type` | Distinct list of types |

`sum` needs no `$`, because the `...` marker already tells the compiler to
compile it per count, much as a
[`[<constant>]` input](generics.md#compile-time-inputs-in-the-ordinary-input-list)
makes a function compiled per value without one. Write a `$` parameter only when
the element types must be named or supplied.

In `print$`, `Items ... : type` is a **variadic `$` parameter**: a list of
types, one per item. `items ... : Items` then gives the first item the type
`Items[0]`, the next `Items[1]`, and so on. Each type is worked out from its own
argument the same way a single generic parameter is, keeping qualifiers and
references. The full rules, including how to state the types with `$(...)`, are
in [generics](generics.md#a-variable-number-of-compile-time-parameters).

Because each item is its own parameter, a qualifier or reference in the
declaration applies to each item separately:

```zax
joinAll final : (result : String)(parts ... : String readonly &) = {
}
// three arguments: three separate reference parameters, not an array of references

inspectAll$ final : (Items ... : type)()(items ... : Items readonly &) = {
}
// every item is a readonly reference, whatever its type
```

In `inspectAll$`, an argument that is already a reference keeps that one
reference rather than gaining a second. The rule is in
[working out from arguments](generics.md#working-out-from-arguments).

### Variadic functions are `final`

A variadic function must be `final`, for the same reason that
[generic functions are `final`](generics.md#generic-functions-are-final). A
`varying` slot holds exactly one implementation, the one most recently assigned
to it. A variadic function has a separate implementation for every count or
type list it is called with, and a slot cannot choose among them at each call.
So a variadic function can't be stored in a slot directly. You can still pick
out one of its implementations as a
[typed function value](#selecting-one-processing).

### Parameters after a pack, and several packs

Other parameters may come before and after a pack. Once the pack starts, every
further positional argument joins it, so a parameter after the pack can be
reached only by its label:

```zax
log final : ()(messages ... : String, level : Integer = 0) = {
}

log("starting", "cache")             // both join messages; level keeps its default
log("starting", "cache", level: 2)   // level only by label
log("starting", "cache", 2)          // error: 2 joins messages and fails by type
```

A second pack is simply another parameter after the first, so it too starts at
its label. A label always starts the pack it names, and that pack keeps
absorbing until the next label:

```zax
tagged$ final : (Tags ... : type, Values ... : type)()(
  tags ... : Tags,
  values ... : Values
) = {
}

tagged("draft", "urgent", values: 1, 2.5)
// tags receives "draft" and "urgent"; values receives 1 and 2.5
tagged(values: 1, 2.5)
// tags is empty
```

When the types are stated with `$(...)`, the same labels separate the type
lists, using the `$` parameters' names:
`tagged$(Tags: String, String, Values: Integer, Float)("draft", "urgent", values: 1, 2.5)`.

Each pack is used on its own: walked, counted, indexed, or passed on. Zax has
no form that walks two packs together item by item, so a pack cannot be paired
up with another one.

## Calling with a pack

Callers do nothing special: they write ordinary arguments. Parameters before
the pack map as in any call, with their defaults and labels, and the pack takes
whatever remains:

```zax
connect final : ()(
  host : Host,
  attempts : Integer = 3,
  extras ... : Option,
  where : SourceLocation = Compilation.caller
) = {
}

myHost : Host
myFast : Option
mySecure : Option
myPlace : SourceLocation

connect(myHost)                              // attempts defaulted; extras is empty
connect(myHost, 5, myFast, mySecure)         // 5 maps attempts; both options join extras
connect(myHost, extras: myFast, mySecure)    // the label starts the pack; it keeps absorbing
connect(myHost, 5, myFast, where: myPlace)   // a parameter after the pack only by label
connect(myHost, myFast, mySecure)            // error: myFast maps attempts, then fails by type
```

`where` normally receives
[the caller's source location](function-invocation.md#the-callers-source-location-as-a-default).
Because it follows the pack, a caller who wants to pass a different location
must use its label.

The last call fails because a positional argument never skips a defaulted
parameter. `myFast` lands on `attempts`, and the call never reaches the pack.
To get past `attempts`, either write `:` to acknowledge its default, as in
`connect(myHost, :, myFast)`, or start the pack by label.

The position that the next positional argument fills is called the cursor. Its
rules are defined in
[inputs around a variadic pack](function-invocation.md#inputs-around-a-variadic-pack).

### Supplying a fixed-length array

The values you want to pass may already sit in an array. `from` supplies
several values as separate positional arguments. A fixed-length array's length
is part of its type, so its elements can fill a pack:

```zax
fixedScores : Integer[3] = [ 1, 2, 3 ]

myScore := sum(from fixedScores)   // sum compiled for three values
```

`from` works through the ordinary cursor, so the same elements can fill ordinary
parameters just as well. Given `plot final : ()(x : Integer, y : Integer)` and
`myCoords : Integer[2]`, the call `plot(from myCoords)` fills `x` and `y`.

`from` is positional. Spreading a structured value's members by name is a
different operation,
[`>-` decomposition](structural-shapes-and-compatibility.md#decompose-one-value-with--).

### Lists known only at runtime

A pack's count must be known at compile time, so a resizable array or a slice
cannot be spread into one. An API that accepts a list whose length is known
only at runtime declares an array or slice parameter instead. Below,
`Integer[0..100]` is a
[resizable array](arrays-and-slices.md#fixed-arrays-resizable-arrays-and-slices)
whose length may change between 0 and 100 at runtime:

```zax
sumAll final : (result : Integer)(values : Integer[]) = {
  total := 0
  each value : in values {
    total += value
  }
  return total
}

scores : Integer[0..100] = [ 4, 5 ]

badTotal := sum(from scores)   // error: a pack's count must be known while compiling
myTotal := sumAll(scores)      // the runtime-length API
```

The separation holds in the other direction too. A pack stays several
parameters even when its items all share one type, so the way a function
receives its arguments never changes with how it happens to be called. When a
variadic function wants a runtime list, it builds one explicitly:

```zax
sumByArray final : (result : Integer)(values ... : Integer) = {
  return sumAll([ from values ])   // the items become a fixed array's elements
}
```

How pack items become array elements is described in
[contributions from a pack](arrays-and-slices.md#contributions-from-a-pack).

## Inside the body

### A pack is not a value

A pack is several bindings, so it cannot be stored or passed as a single value:

```zax
send$ final : (Items ... : type)()(items ... : Items) = {
}

relay$ final : (Items ... : type)()(items ... : Items) = {
  saved := items           // error: a pack is not a value
  send(items)              // error: a bare pack name is not a list of arguments
}

relayFixed$ final : (Items ... : type)()(items ... : Items) = {
  send(from items as copy) // from passes the items as separate arguments
}
```

In `relayFixed$`, `as copy` says that each item is copied rather than handed
over. [Passing items on](#passing-items-on) explains why this last use of the
items has to state a stance.

A bare pack name is accepted only where a pack itself is expected: as the
source of `each item : in items`, before `count`, with a compile-time index, and in a
lambda capture list. Anywhere the items should become values, write
`from items`.

### Walking the items with `each`

The most common thing to do with a pack is visit each item. `each in` walks a
pack item by item, in argument order:

```zax
print$ final : (Items ... : type)()(items ... : Items) = {
  each item : in items {
    write(item)            // checked once for each item's type
  }
}

keep$ final : (UseType : type)()(value : UseType) = {
}

keepAll$ final : (Items ... : type)()(items ... : Items) = {
  each item : copy in items {
    keep(item)             // copy must be viable for every item
  }
}
```

In `keepAll$`, `each item : copy in items` binds an owned copy of each item
rather than referring to the item itself, so copying must be possible for every
item's type.

The body is checked separately for each item's type, which is why `write(item)`
may select a different `write` for each item. This is a rule about typing, not a
promise about how the generated code is laid out.

A list of types can be walked the same way:

```zax
describeType$ final : (UseType : type)()() = {
}

describeTypes$ final : (Items ... : type)()() = {
  each Item : in Items {
    describeType$(Item)()
  }
}

describeTypes$(Integer, String)()   // describeType$ for Integer, then for String
```

The full traversal rules are in [iteration](iteration.md#packs).

### Counting the items

`items count` and `Items count` give the number of items as a compile-time
`IndexSize`:

```zax
countOf$ final : (Items ... : type)(result : IndexSize)(items ... : Items) = {
  return items count       // 3 in the processing for countOf(1, "two", true)
}
```

Because the count is a compile-time value, it can appear in types, as in the
fixed array `Byte[Items count]`, and in `requires` clauses.

### Selecting one item by position

To reach one particular item, index the pack. `items[N]` selects one item, and
`Items[N]` its type, as long as `N` is known at compile time:

```zax
isText final : (result : Boolean)(UseType : type) = {
  return UseType == String
}

title$ final : (Items ... : type)()(items ... : Items)
  requires Items count >= 1 && isText(Items[0]) = {
  write(items[0])          // the first item, a String
}

pickAt$ final : (Items ... : type)()(position : IndexSize, items ... : Items) = {
  write(items[position])   // error: position is a runtime value, and items may differ in type
}
```

The index has to be a compile-time value for every pack. A pack's items are
separate parameters, not an array, so `items[N]` picks one of those parameters
at compile time. The items may also differ in type, and the compiler must know
which one you mean.

An index at or past the count is an error in that processing. `title$` avoids
that with its `requires` clause: a call with no items is not viable instead.

## Passing items on

### Expanding with `from items`

A variadic function often wraps another one: it adds a little behavior and
passes the items along. `from items` passes the items on as separate positional
arguments.

The examples here write a **transfer stance** after the items, as in
`from items as copy`, to say how each value is offered to the receiver: `copy`
leaves the source unchanged, while `last` offers it as finished. When an item
is passed for the last time to a receiver that could take ownership of it, Zax
does not choose for you, and the code must state `copy`, `move`, `last`, or
`deep`. All four are defined in [transfer stances](transfer-stances.md).

```zax
logAll$ final : (Items ... : type)()(items ... : Items) = {
  each item : in items {
    write(item)
  }
}

trace$ final : (Items ... : type)()(items ... : Items) = {
  write("trace:")
  logAll(from items as copy)
}

trace("cache", 3, true)    // logAll receives "cache", 3, and true
```

The expanded items land wherever the cursor puts them, so they can fill
ordinary parameters too. Each processing knows its count, so that mapping is
checked separately for each call shape:

```zax
MyHeader :: type { }
MyBody :: type { }

sendTwo final : ()(header : MyHeader, body : MyBody) = {
}

forward$ final : (Items ... : type)()(items ... : Items) = {
  sendTwo(from items as copy)   // fills two ordinary parameters when there are two items
}

myHeader : MyHeader
myBody : MyBody

forward(myHeader, myBody)       // fine
forward(myHeader)               // error in this processing: sendTwo needs two inputs
```

### A stance applies to each item

A stance written on an expansion applies to each item separately. Each item
keeps its own type, including whether it is a reference, so an item that is a
reference stays a reference.

The rule that a last use needs stated intent also applies item by item. It
applies whenever the receiver could take an item over. An ordinary by-value
parameter could, and so could one declared with `last`, such as `sendAll$`'s
below. It does not apply when taking the item over would change nothing, as
for a receiver that can only copy. Because a generic item's type may own
resources, the examples on this page state a stance whenever they pass generic
items on for the last time:

```zax
sendAll$ final : (Items ... : type)()(items ... : Items last) = {
}

forwardAll$ final : (Items ... : type)()(items ... : Items) = {
  sendAll(from items)
  // error: this is each item's final use, and sendAll could take it
  // terminally; state copy, move, last, or deep intent
}

forwardLast$ final : (Items ... : type)()(items ... : Items) = {
  sendAll(from items as last)   // last applies to each item separately
}
```

The exact spelling of a stance after a type such as `Items` is not final.

Zax has no silent perfect forwarding. You write the stance, just as you would
for a single value. To give different items different stances, write the items
as separate arguments, selected by compile-time index:

```zax
sendPair final : ()(header : MyHeader move, body : MyBody) = {
}

forwardPair$ final : (Items ... : type)()(items ... : Items)
  requires Items count == 2 = {
  sendPair(items[0] as move, items[1] as copy)
}
```

The full rules are in
[a stance on several items](transfer-stances.md#a-stance-on-several-items).

## Transforming each item

A common need is to run each item through a call before passing it on: describe
each value, parse each document, pad each line. `from each` does this right
inside an argument list.

It builds on the `|>` chain, which passes a value along. `x |> f()` calls `f`
with `x` as its first input, and `x |> f() |> g()` passes `f`'s results on to
`g`. Chains are taught in
[chain calls immediately](lambdas-and-callable-composition.md#chain-calls-immediately).
`from each` runs such a chain once per item, and the results form a new pack at
that position:

```zax
describe$ final : (UseType : type)(result : String)(value : UseType) = {
}

summarize$ final : (Items ... : type)(result : String)(items ... : Items) = {
}

render final : ()(texts ... : String) = {
}

renderAll$ final : (Items ... : type)()(items ... : Items) = {
  render(from each item : in items |> describe())
  // describe once per item; the descriptions form a new pack for render
}

renderAll("label", 42, true)
// render receives describe("label"), describe(42), describe(true)
```

Read the form in three parts. `from` says that several values are contributed
here. `each item : in items` is the ordinary `each` header, and the binding name
is required. `|> describe()` is the chain each item runs through. The mapping
ends at the comma or closing parenthesis of the argument that contains it.

Using the same declarations, here are two ways a call can use a pack, side by
side: passing the whole pack to one call, and mapping each item:

```zax
compareForms$ final : (Items ... : type)()(items ... : Items) = {
  render(summarize(from items as copy))
  // summarize receives every item, once; render receives one String

  render(from each item : in items |> describe(), "end")
  // the mapped pack, then one ordinary argument
}
```

Plain `from items` would pass the items unchanged, which suits `render` only
when every item is already a `String`.

`from each` is an expression, not a loop. To do something once per item for its
effect, use an ordinary `each` statement:

```zax
logEach$ final : (Items ... : type)()(items ... : Items) = {
  each item : in items {
    render(describe(item))
  }
}
```

### Placing the item

The first stage of the chain decides where the item goes. If you write the
item's name in that stage, the item goes there. If you don't, the item becomes
the stage's first positional input:

```zax
format final : (result : String)(value : Integer, width : Integer) = {
}

pad final : (result : String)(width : Integer, value : Integer) = {
}

show final : ()(lines ... : String) = {
}

formatAll final : ()(width : Integer, values ... : Integer) = {
  show(from each value : in values |> format(width))
  // not written: each value is format's first input, as in format(value, width)

  show(from each value : in values |> pad(width, value))
  // written: each value goes only where it is written

  show(from each value : in values |> pad(width: width, value: value))
  // labels mean what they mean in any call

  show(from each value : in values |> pad(width: width))
  // error: the unwritten value fills width, then width: supplies it again
}
```

Writing the name suppresses the implicit placement only for a `from each`
binding. In an ordinary chain such as `x |> f(x)`, `x` is just a variable, so it
is passed twice.

The item's name is yours, and it is never compared with parameter names, so it
never routes the item by matching a parameter label. This follows the general
rule that
[names you choose never route](function-invocation.md#names-you-choose-never-route).
Stage results route as in any chain, following
[how forwarded results find their parameters](function-invocation.md#how-forwarded-results-find-their-parameters).

### Several stages

A mapping can have several stages. Later stages receive the previous stage's
results as in any chain, and the item's name stays visible in every stage.

The remaining examples in this section reuse declarations from earlier ones,
such as `show`, `describe$`, and `render`.

```zax
trimmed final : (result : String)(text : String) = {
}

annotated final : (result : String)(text : String, original : String) = {
}

showTrimmed final : ()(lines ... : String) = {
  show(from each line : in lines |> trimmed() |> annotated(line))
  // for each line: trimmed(line), then annotated(that result, line)
}
```

To relabel a stage's results, write a postfix `reshape` on that stage's
invocation, as shown in
[reshape a stage's results](lambdas-and-callable-composition.md#reshape-a-stages-results).

### In array expressions

A mapping is accepted anywhere several values are: in an argument list, or in an
[array expression](arrays-and-slices.md#contributions-from-a-pack). In an array,
every item must produce one common element type:

```zax
measureWidth$ final : (UseType : type)(result : IndexSize)(value : UseType readonly &) = {
}

widths$ final : (Items ... : type)(result : IndexSize[Items count])(items ... : Items) = {
  return [ from each item : in items |> measureWidth() ]   // one width per item
}

descriptions$ final : (Items ... : type)()(items ... : Items) = {
  all := [ from each item : in items |> describe() ]       // fine: every description is a String
  render(from all)
}
```

### Order

Each item runs through every stage before the next item starts. The whole
mapping occupies one argument position, so the enclosing call's other arguments
still evaluate in source order around it. Each result binds as soon as it is
produced:

```zax
header final : (result : String)() = {
}

footer final : (result : String)() = {
}

page final : ()(lines ... : String) = {
  show(header(), from each line : in lines |> trimmed() |> annotated(line), footer())
  // header(); then trimmed and annotated for the first line, then for the next;
  // then footer()
}
```

### One value per item

Intermediate stages may pass several results along, but each item's last stage
contributes exactly one value, so the new pack has as many items as the source
pack. That value is the last stage's first result. Any later results must be
discardable, and they go unused. This matches a declaration such as
`value := measure()`, which binds the first result and requires any later
results to be discardable:

```zax
split final : (head : String, tail : String)(text : String) = {
}

join final : (result : String)(first : String, second : String) = {
}

rejoin final : ()(lines ... : String) = {
  show(from each line : in lines |> split() |> join())
  // fine: split passes two results to join, which produces one

  show(from each line : in lines |> split())
  // error: split's tail is not discardable, so each item would offer two values
}
```

When the last stage has one result plus discardable ones, only its first result
is contributed:

```zax
MyShape :: type { }

measure final : (width : Integer, height # : Integer)(shape : MyShape) = {
}

showWidths final : ()(widths ... : Integer) = {
}

drawAll final : ()(shapes ... : MyShape) = {
  showWidths(from each shape : in shapes |> measure())
  // each shape contributes its width; the discardable height goes unused
}
```

This rule protects the code built on a producer. A discardable result never
joins the pack, so adding one to a producer cannot grow or shift the packs built
from it. If each item's results were spread into the pack instead, adding a
result to a producer would silently grow every pack built from it and shift
every later position.

When each item really does have several parts, return one structured value so
the parts keep their names:

```zax
MyRange :: type {
  low : Integer
  high : Integer
}

rangeAround final : (range : MyRange)(center : Integer) = {
}

setRanges final : ()(ranges ... : MyRange) = {
}

markAll final : ()(centers ... : Integer) = {
  setRanges(from each center : in centers |> rangeAround())   // one MyRange per item
}
```

When the producer already returns its parts as separate results, combine them
into one value per item at the call site with
[`-<`](structural-shapes-and-compatibility.md#recompose-several-results-with--):

```zax
MyPoint :: type {
  x : Integer
  y : Integer
}

toPoint final : (x : Integer, y : Integer)(sample : Integer) = {
}

drawPoints final : ()(points ... : MyPoint) = {
}

drawShapes$ final : (Shapes ... : type)()(shapes ... : Shapes) = {
}

plotAll final : ()(samples ... : Integer) = {
  drawPoints(from each sample : in samples |> MyPoint -< toPoint())
  // each sample's x and y become one MyPoint

  drawShapes(from each sample : in samples |> -< toPoint())
  // each sample's x and y become one anonymous value with members x and y

  drawPoints(from each sample : in samples |> toPoint())
  // error: each item would offer two values; write -< to combine them
}
```

`-< toPoint()` is still the stage's invocation, so each sample is carried into
`toPoint` as usual, and `-<` has greater precedence than `|>`. Because the results
become named members, a producer that later gains a result adds a member rather
than shifting positions. That includes discardable results: every labeled
result becomes a member. A receiver that reads `.x` and `.y` keeps
working.

### Stances while mapping

A stance can go in either of two places: on the source, where it applies to
each item, or at the use inside a stage:

```zax
MyMessage :: type { }
MyPacket :: type { }

encode final : (packet : MyPacket)(message : MyMessage last) = {
}

sendPackets final : ()(packets ... : MyPacket) = {
}

sendMessages final : ()(messages ... : MyMessage) = {
  sendPackets(from each message : in messages as last |> encode())
}

sendMessagesAtUse final : ()(messages ... : MyMessage) = {
  sendPackets(from each message : in messages |> encode(message as last))
}
// Both offer last for each message separately.
```

The header accepts the same binding forms as the `each` statement, for example
`from each item : copy in items |> describe()`.

### The first failure stops the rest

The mapped calls behave as if each were written out as its own argument. They
are made before the enclosing callable starts, so a handler on the enclosing
call covers them. If one item selects an exceptional outcome:

- the values already produced are cleaned up as already-bound arguments;
- the remaining items and later arguments do not run; and
- the handler runs.

```zax
MyTree :: type { }
MyParseError :: type { }

parse final : (
  tree : MyTree,
  failure except : MyParseError
)(
  text : String readonly &
) = {
}

MyPicture :: type { }

drawTrees final : (picture : MyPicture)(trees ... : MyTree) = {
}

display final : ()(picture : MyPicture) = {
}

drawAll final : ()(docs ... : String) = {
  picture := drawTrees(from each doc : in docs |> parse()) catch failure {
    return
  }
  display(picture)
}
// With three docs, the drawTrees call behaves as:
//   drawTrees(parse(doc0), parse(doc1), parse(doc2)) catch failure { ... }
```

Failures are not collected: each call completes with one outcome. To
keep processing the other items after one fails, handle the outcome inside the
stage or use an `each` statement. The complete behavior is in
[mapping each item of a pack](except.md#mapping-each-item-of-a-pack).

## Returning and holding packs

### Result packs

A variadic function can also return a variable number of results by declaring
a **result pack**. The number is fixed for each processing. A pass-through,
which returns one result per input, is the simplest case:

```zax
passThrough$ final : (Items ... : type)(results ... : Items)(items ... : Items) = {
  return from items as last
}

x : Integer, y : Integer, z : Integer = passThrough(1, 2, 3)
// a caller that knows the count captures the items with typed declarations
```

Like an input pack, a result pack is not one value or a tuple. Each processing
has one result slot per item, and each item behaves like a separately declared
result. The [result packs](function-invocation.md#result-packs) rules cover how
those slots route, and how acknowledgement, elision, and `return #` apply to
them. Discardability is declared once for the pack and applies to each item.

The pack's types come from the inputs or from `$(...)`, never from the body or
from an expected result. A result declared after a result pack is reachable only
by label.

A body can fill a result pack by mapping, as in
`return from each item : in items |> normalize()` for a per-item `normalize`.
It can also construct the items one at a time:

```zax
makeDefaults$ final : (Items ... : type)(results ... : Items)() = {
  each result : in results {
    result .= [{}]          // checked once per item type
  }
}

first : Integer, second : String = makeDefaults$(Integer, String)()
```

Each result slot starts empty. `.=`
[constructs a value into an empty place](construction-and-destruction.md#construct-into-an-existing-place-with-dot-equals),
and `[{}]` is an empty
[construction packet](construction-and-destruction.md#construction-packets),
so each item is default-constructed as its own type.

Result-pack items have no labels, so they route by position. To steer one item
to a particular parameter, a reshape can name it by position with `[N]`:

```zax
highlight final : ()(chosen : String, others ... : String) = {
}

highlight(passThrough("a", "b", "c") reshape [1]: chosen:)
// "b" binds chosen by its new label; "a" and "c" fill others, in order.
// Without the reshape, "a" would fill chosen.
```

The `[N]` form is described in
[naming results by position](structural-shapes-and-compatibility.md#naming-results-by-position-with-n).

### Several result packs, and pack labels

Individual items have no labels, but the pack as a whole does: its declared
name. That label lets a function return several packs, and lets each one find
its destination:

```zax
makeBoth$ final : (Firsts ... : type, Seconds ... : type)(
  firsts ... : Firsts,
  seconds ... : Seconds
)() = {
  each first : in firsts {
    first .= [{}]
  }
  each second : in seconds {
    second .= [{}]
  }
}

storeBoth$ final : (Firsts ... : type, Seconds ... : type)()(
  firsts ... : Firsts,
  seconds ... : Seconds
) = {
}

storeBoth(makeBoth$(Firsts: Integer, String, Seconds: Float)())
// the firsts pack binds the firsts pack as a group, and seconds binds seconds
```

A pack label matches only an input pack with the same label, never an ordinary
parameter, even when the pack holds a single item. A reshape relabels a whole
pack, as in `reshape firsts: lefts:`, while `[N]` still names individual items.

A body can return values into several packs by label, the same way a call
supplies them. A label starts the pack it names, which keeps absorbing until the
next label:

```zax
regroup$ final : (Lefts ... : type, Rights ... : type)(
  rights ... : Rights,
  lefts ... : Lefts
)(
  leftInputs ... : Lefts,
  rightInputs ... : Rights
) = {
  return lefts: from leftInputs as last, rights: from rightInputs as last
  // each label starts its own result pack, whatever order the packs are declared in
}
```

The counts still come from the inputs, never from the body. The rules for
labeled returns are in
[forwarding from a return](function-invocation.md#forwarding-from-a-return).

### Holding a pack in local declarations

Inside a variadic function, a call may return a result pack whose number of
items depends on how the function was called, so there is no fixed list of
locals to declare for it. A local pack declaration holds it as several locals. This
example uses `passThrough$` from above and `logAll$` from
[passing items on](#expanding-with-from-items):

```zax
relay$ final : (Items ... : type)()(items ... : Items) = {
  outs ... := passThrough(from items as copy)   // several locals, one per item
  logAll(from items as copy)                    // items stay usable
  logAll(from outs as last)
}
```

A local pack follows every rule of a pack parameter. It is not a value, even
when it holds a single item, so `saved := outs` is an error, and its items are
reached through `from outs`, `each`, `outs count`, or a compile-time index.

When a call returns several packs, select each one by its label into its own
local pack. This uses `makeBoth$` from
[several result packs](#several-result-packs-and-pack-labels):

```zax
firsts: lefts ... :, seconds: rights ... : = makeBoth$(Firsts: Integer, String, Seconds: Float)()
// lefts holds the Integer and String items; rights holds the Float item
```

A result pack cannot become a member of a value made with `-<`, because a
member is one value and a pack is not. Route the pack out first, then combine
the rest:

```zax
produceReport$ final : (Items ... : type)(total : Integer, items ... : Items)() = {
}

items: outs ... :, summary : -< produceReport$(Integer, String)()
// outs holds the items; total becomes the only member of summary

report : -< produceReport$(Integer, String)()
// error: the result pack items cannot become a member; route it out first
```

The entries before `-<` take their results first, by label, and `-<` gathers
only the results that are left, just as in `count:, remaining : -< produce()`.

The declaration rules are in
[declarations and bindings](declarations-and-bindings.md#multiple-results).

## Checking packs

### Every element satisfies a constraint

To require something of every item's type, write a constraint function in place
of `type` on a variadic `$` parameter:

```zax
isScalar final : (result : Boolean)(UseType : type) = {
}

total$ final : (Items ... : isScalar)(result : Integer)(items ... : Items) = {
}

myByte : U8 = 7

grand := total(1, 2, myByte)   // fine: every element is scalar
wrong := total(1, "two")       // error: no viable total; the item at position 1 (the second item), a String, fails isScalar
none := total()                // fine: an empty pack has no element that fails
```

A rejection names the failing item's type and its position, counted from zero
as in `items[N]`. When specializations are ranked, an every-element check counts
as one constraint term, as described in
[every element of a pack](generics.md#every-element-of-a-pack).

### Counts and empty packs

An empty pack satisfies every every-element constraint, since it has no item
that could fail. A declaration that needs at least one item adds a `requires`
clause on the count. The count is a compile-time input even when no `$` list
names the element type, so a `requires` clause works on a stated-type pack too:

```zax
average final : (result : Float)(values ... : Float)
  requires values count >= 1 = {
}

myAverage := average(1.0, 2.0)   // fine
noAverage := average()           // error: no viable average; values count >= 1 is false
```

A clause can also check the type at one position, or pass the whole list of
types to a compile-time function. When ranking overloads, the compiler treats
each such function call as one opaque check. It doesn't examine what the
function tests:

```zax
allSameWidth final : (result : Boolean)(Types ... : type) = {
  // walks Types with each, comparing the target size of each
}

packed$ final : (Items ... : type)()(items ... : Items)
  requires allSameWidth(from Items) = {
}
```

The general rules are in [`requires` clauses](generics.md#requires-clauses).

### Recursion through a pack

A variadic function may call itself on each item. Recursion through a pack is
often intentional, and the compiler cannot see whether it stops, so it is not
diagnosed. To exclude a count the function should never handle, use an ordinary
constraint:

```zax
list final : ()(value : Integer) = {
}

list final : ()(value : String) = {
}

list$ final : (Items ... : type)()(items ... : Items)
  requires Items count != 1 = {
  each item : in items {
    list(item)
  }
}

list("marbles", 10, "ducks", 7)
// each item selects the fixed list for its type
```

Without the clause, an item of a type with no fixed `list`, such as a `Boolean`,
would select `list$` again with one item, and the program would recurse without
end. With the clause, that call is a compile error.

## Choosing between overloads

A variadic overload often sits beside fixed ones. When the candidates are
otherwise equally good matches, three rules decide between them. They are
defined in full in
[candidates with a variadic pack](function-invocation.md#candidates-with-a-variadic-pack).

**A fixed parameter beats an absorbed argument.** A fixed declaration accepts a
strict subset of the pack's call shapes, so it is more specific:

```zax
largest final : (result : Integer)(first : Integer, second : Integer) = {
}

largest final : (result : Integer)(values ... : Integer) = {
}

pair := largest(1, 2)        // the two-parameter largest wins
triple := largest(1, 2, 3)   // only the variadic largest is viable
```

**An empty pack counts as one omitted default.** A candidate that needs no pack
wins over one that receives an empty pack:

```zax
inspect$ final : (First : type)()(first : First) = {
}

inspect$ final : (First : type, Rest ... : type)()(first : First, rest ... : Rest) = {
}

inspect(1)          // the first inspect wins: the second needs an empty pack
inspect(1, "two")   // only the second is viable
```

**A pack with a stated type and no `$` list is not generic.** Its function is
compiled per count, yet it is an ordinary candidate. That candidate beats an
equally good generic candidate. A call written with `$(...)` considers only
generic candidates, so it never selects this one:

```zax
isNumeric final : (result : Boolean)(UseType : type) = {
}

sum final : (result : Integer)(values ... : Integer) = {
}

sum$ final : (Items ... : isNumeric)(result : Integer)(items ... : Items) = {
}

plain := sum(1, 2)                      // the sum without $ wins
generic := sum$(Integer, Integer)(1, 2) // only the generic is a candidate
```

## Compile-time values in packs

`[<constant>]` on a pack requires every item to be a compile-time value. The
function is then compiled once per distinct list of values:

```zax
tags final : ()(names ... [<constant>] : String) = {
}

tags("alpha", "beta")   // compiled once for this exact list
```

The most common use pairs one compile-time input with a runtime pack. A format
string, for example, can be checked against the argument types at compile time,
while the arguments themselves stay ordinary runtime values. In the body below,
[`if compile`](compile-time-code.md) chooses a branch while compiling, and
`Compilation.error` reports a compile error with the author's message. The
exact spelling of compile-time error reporting is not final.

```zax
formatMatches final : (result : Boolean)(format : String readonly, Types ... : type) = {
}

printf$ final : (Args ... : type)()(
  format [<constant>] : String readonly,
  args ... : Args
) = {
  if compile !formatMatches(format, from Args) {
    Compilation.error("printf: format does not match the arguments")
  }
  // ...
}

myName := "Ada"
myScore := 42
printf("%s scored %d", myName, myScore)   // checked while compiling, run at runtime
```

A variadic `$` parameter may hold compile-time values instead of types. Two
such packs are the same argument when they have the same length and each item
compares equal. An empty pack is an argument too:

```zax
MyMask$(bits ... : Integer) :: type { }

lowBits : MyMask$(1, 4, 9)
sameBits : MyMask$(1, 4, 9)   // the same type as lowBits
noBits : MyMask$()            // a different type
```

Argument equality in general is covered in
[comparing arguments](generics.md#comparing-arguments).

## Lambdas and function values

### Capturing a pack

A lambda inside a variadic function can capture the whole pack by name. Inside
the lambda, the captured items are still a pack, used just as in the enclosing
function. In the example, `[[ items ]]` is
the capture list, and `()() bound unique` is a callable with no results or
inputs that uniquely owns its captured state. That callable type is explained in
[callable prototypes and storage](lambdas-and-callable-composition.md#callable-prototypes-and-storage).

```zax
deliverAll$ final : (Items ... : type)()(items ... : Items) = {
}

deferDelivery$ final : (Items ... : type)(
  callback : ()() bound unique
)(
  items ... : Items
) = {
  return [[ items ]] ()() bound unique {
    deliverAll(from items as copy)
  }
}
```

Because the enclosing function is compiled for each list of argument types, the
lambda stores one captured value per item, each with its known type. A stance
on the capture applies to each item and leaves the lambda's own prototype
unaffected.

### Variadic lambdas

A lambda can declare a pack parameter of its own. Like a variadic function, it
is compiled per count, so it must be held in `final` storage. A declaration
without a storage qualifier is `varying`, so `final` has to be written:

```zax
tally final := [[]] (result : Integer)(values ... : Integer) {
  running := 0
  each value : in values {
    running += value
  }
  return running
}

grandTotal := tally(1, 2, 3)

counter := [[]] (result : Integer)(values ... : Integer) {   // error: a variadic lambda needs final storage
  return 0
}
```

When the items may differ in type, the lambda needs a list of types, which makes
it a generic lambda. The `$` goes on the binding name, and the lambda's first
parenthesized list holds the compile-time parameters:

```zax
prefix := "> "

echo$ final := [[ prefix ]] (Items ... : type)()(items ... : Items) {
  each item : in items {
    write(prefix)
    write(item)
  }
}

echo("a", 2, true)                 // worked out: echo$(String, Integer, Boolean)
echo$(String, Integer)("a", 2)     // stated
```

A lambda written inline, whether passed, returned, or invoked immediately, has
no name to carry `$`. It takes concrete types from its destination, from types
written directly, or from an enclosing generic's parameters. Variadic lambdas
are covered in
[lambdas with packs](lambdas-and-callable-composition.md#lambdas-with-packs),
and named and inline generic lambdas in
[generic lambdas](generics.md#generic-lambdas).

### Selecting one processing

A `varying` slot or a callback parameter needs an ordinary function value. A
variadic function or lambda has no single function value, because it is
compiled once per count or type list. A fully typed function value has one fixed
prototype, though, and binding the variadic function to it selects the one
processing that matches. Below, `sum` is the one from the
[opening example](#accept-any-number-of-arguments), and `tally` is the variadic
lambda above:

```zax
pickSum : (result : Integer)(first : Integer, second : Integer) = sum
pairTotal := pickSum(1, 2)    // calls sum's two-item processing

handler : (result : Integer)(first : Integer, second : Integer) = tally
// handler holds tally's two-item processing
```

Binding may need compiler-generated glue between that processing and the fixed
prototype. The assignment is supported either way, and the glue is a cost that
tools should show. The details are in
[a function value for a variadic function](function-invocation.md#a-function-value-for-a-variadic-function).

## Costs

Programmers must be able to discover:

- each processing of a variadic function or lambda, one per distinct count,
  list of element types, or list of compile-time values used;
- that a pack involves no runtime list, temporary array, or allocation, while
  building an array from a pack with `[ from items ]` is explicit and costs what
  that array costs;
- the copies, moves, and terminal transfers of each item, as its stance states;
- the calls a `from each` mapping makes for each item, in order;
- compile points in a variadic body, which run once per processing;
- compile-time work in every-element constraints and `requires` clauses, which
  runs once per argument set; and
- compiler-generated glue that binds a fixed function value to one processing.

## Diagnostics

Diagnostics should distinguish:

- a runtime-length array or slice expanded with `from` into inputs;
- a pack used as one value, or passed without `from`;
- a runtime index into a pack, and an index at or past the count in one
  processing;
- a pack label that names an ordinary parameter rather than a pack;
- a result pack that `-<` would have to make into a member;
- a variadic function, or a variadic or `$` lambda, outside `final` storage;
- an expansion whose items do not fit the receiving parameters in one
  processing;
- an implicit terminal use of each item where the receiver could take it
  terminally;
- a per-item chain whose last stage produces no result, or has a result after
  its first that is not discardable, suggesting `-<` when the results could
  become one value;
- mapped results of different types in an array expression;
- an every-element constraint failure, naming the failing item's type and its
  zero-based position, as in "the item at position 1 (the second item)";
- an item carried into the first stage of a per-item chain when that stage
  has no remaining input to take it; and
- a `requires` clause on a count that rejects the call.

When a diagnostic arises in one processing, it should name that processing's
count or type list.

## Source stability

- Calling a variadic function with a new count or type list creates another
  processing. No source changes, but code size grows.
- Adding a defaulted parameter after a pack never captures existing positional
  arguments, because positional arguments after the pack's start keep joining
  the pack. Adding a parameter before the pack does capture the first argument.
- Adding a fixed overload beside a variadic one takes over calls of exactly its
  shape, because a fixed parameter beats an absorbed argument.
- Changing a stated-type pack to a `$` pack turns a non-generic candidate into a
  generic one, which can change selection.
- Tightening an every-element constraint or a count requirement can make
  existing calls not viable.
- A per-item chain contributes only its last stage's first result. If that
  stage gains a result that is not discardable, the chain is rejected. If it
  gains a discardable one, that result goes unused. Either way, a change to the
  producer cannot silently grow or shift a pack.
- A per-item chain ending in `-<` gains a member when its producer gains a
  result, discardable or not. Receivers that read members by name keep
  working. An array expression that needs one common element type reports the
  changed type.
- Adding a second pack after an existing one never captures existing
  arguments, because the new pack starts only at its label. Renaming a pack
  breaks callers that use its label, and reroutes result packs that matched it.
- Renaming a `from each` binding never reroutes a call.

## Boundaries and maturity

This document is current conceptual design, not a formal grammar,
implementation mapping, or conformance specification.

Later work must preserve:

- every variadic parameter as a pack of separate bindings with a compile-time
  count, even when its items share one type;
- runtime-length lists as arrays or slices, never packs;
- `from` as the visible sign that several values are contributed;
- per-item stance and the per-item terminal-use intent rule;
- one value per item at the end of a per-item chain, with `-<` as the explicit
  way to combine several results into that value; and
- each pack used on its own: two packs are never walked together.

The following remain future work:

- tuple-like generic types with one member per pack element, which packs never
  silently become;
- folding across a pack whose items have different types;
- spreading several values per item into a pack at the end of a per-item
  chain, rather than combining them with `-<`;
- complete variadic composition, variadic call components in mixfix operators,
  and variadic operator phrases and holes;
- identifying which item of a mapped pack selected an exceptional outcome,
  which today depends on the payload;
- C-style untyped variable arguments for foreign functions, which never enter
  ordinary Zax calls; and
- generating declarations by walking a list of types, given that `each` over a
  type pack produces runtime bodies only.

For Zax's accepted foundational direction, see the
[language vision](vision.md).
