# Zax lambdas and callable composition

| Field | Value |
| --- | --- |
| Status | Current conceptual design |
| Audience | Human developers defining, storing, composing, and invoking callable values |
| Applies To | Lambda expressions, captures, callable storage and receiver lifetime, partial application, composition, and chaining; not a formal grammar or ABI |
| Implementation State | Not established by this repository |
| Owns | Lambda expression and capture behavior; generated lambda receivers; fixed/varying and bound/unbound callable storage integration; receiver-lifetime modes; installed binding kind and weak callable invocation; callable transfer, reset, recursion, and repeated capture state; pack capture; variadic and generic lambdas in `final` storage; partial argument capture and prototype adaptation; composition and reshape stages; preservation and relabeling of exceptional outcomes through retained composition; immediate chaining as the nested call, including postfix reshape on stage invocations and per-item mapping chains; callable-specific costs and diagnostics |
| Does Not Own | General declaration rules ([declarations and bindings](declarations-and-bindings.md)); call phases and the rule by which results find inputs ([function invocation](function-invocation.md#how-forwarded-results-find-their-parameters)); reshape declarations and positional `[N]` source paths ([structural shapes and compatibility](structural-shapes-and-compatibility.md#reusable-mapping-with-reshape)); the pack model ([variadics](variadics.md)); the generic rules that generic lambdas follow ([generics](generics.md#generic-lambdas)); cohesive [exceptional result flow](except.md); complete transfer meaning ([transfer stances](transfer-stances.md)); reference lifetime ([lifetimes and references](lifetimes-and-references.md)); pointer ownership and opaque lifetime facilities ([pointers and arenas](pointers-and-arenas.md)); exact operator registration ([operator catalog](operator-catalog.md)); or async, reflection, FFI, ABI, and lowering |
| Source / Provenance | Maintainer-supplied lambda design, legacy function composition evidence, and current declaration, invocation, lifetime, transfer, Nothing, source, and operator design |
| Supersedes | Current-purpose lambda, capture, callable-storage, composition, and chaining material formerly distributed through root legacy function pages and raw project input |

## Start with a lambda

A lambda combines captured state with a callable body:

```zax
factor := 3

scale := [[ factor ]] (
  result : Integer
)(
  input : Integer
) {
  return input * factor
}

factor = 4
scaled := scale(5) // 15
```

Constructing `scale` copies `factor` into a generated receiver. Calling `scale`
later runs the body against that retained receiver; it does not capture
`factor` again.

The declaration and lambda expression have separate roles:

```text
scale :=                             declare and initialize scale
[[ factor ]]                        construct its capture receiver
(result : Integer)(input : Integer) state its call prototype
{ ... }                             define its delayed body
```

An empty capture is explicit:

```zax
doubleLater := [[]] (
  result : Integer
)(
  input : Integer
) {
  return input * 2
}
```

`[[]]` says that this is a lambda with zero captures. A nonempty single-line
capture uses interior spaces, as in `[[ factor ]]`; multiline captures place the
entries on indented lines. Complete token and spacing rules are in
[source structure](source-structure.md#array-expressions-capture-delimiters-and-slicing).

### Lambdas may declare exceptional outcomes

A lambda uses the same completion contract as a named function:

```zax
reader := [[ source ]] (
  value : MyValue,
  failure except : MyFailure
)() {
  if cannotRead(source)
    except failure: makeFailure(source)

  return readValue(source)
}
```

The exceptional outcome is part of the minted callable prototype. Invocation,
storage, catch, forwarding, outcome reshape, and composition preserve it under
ordinary callable rules.

Selecting `failure` ends one invocation. It does not destroy or reset `reader`
or its capture receiver. A reference exceptional payload projected from
captured state retains that capture origin and cannot outlive the receiver
relationship that keeps it valid.

## Capture values, references, and producer results

### Ordinary capture copies

Each ordinary capture constructs a value in the generated receiver:

```zax
message := "ready"

report := [[ message ]] ()() {
  print(message)
}

message = "changed"
report() // prints ready
```

The default is `copy`. If the source name is a reference, ordinary capture
copies the referent rather than preserving the reference:

```zax
source : Document
view : Document readonly & = source

snapshot := [[ view ]] ()() {
  inspect(view) // a captured Document value
}
```

If the referent cannot be copied, this capture is unavailable.

### Rename or construct a capture

Capture mapping reads from source to destination:

```zax
sourceCount : Integer = 42

report := [[ sourceCount: capturedCount: ]] ()() {
  print(capturedCount)
}
```

A complete destination declaration requests direct construction:

```zax
report := [[ sourceCount: capturedCount : MyCount ]] ()() {
  print(capturedCount)
}
```

Construction happens when `report` is created. It may invoke user code,
allocate, or panic like the equivalent ordinary typed declaration.

An empty destination name retains the source name:

```zax
report := [[ sourceCount: : MyCount ]] ()() {
  print(sourceCount)
}
```

A `#` destination stores the capture and introduces no body name:

```zax
keeper := [[
  source: # : StoredResource,
  visible: kept:
]] ()() {
  use(kept)
}
```

`source` is constructed as `StoredResource` when `keeper` is created and
destroyed with `keeper`'s receiver. The body cannot name it. `source: #`
without a type copies the source the same way and likewise introduces no name.
The empty-name form above is different: `sourceCount: : MyCount` still binds
`sourceCount`.

The bare destination name and the receiver member are the same capture:

```zax
report := [[ sourceCount: capturedCount: ]] ()() {
  print(capturedCount)
  print(_.capturedCount)
}
```

The capture list inserts `capturedCount` as a body binding for that receiver
member. A read of either spelling uses the capture. A `#` destination has no
member name, so the discarded `source` above has no `_.source`.

### Explicit reference capture borrows

```zax
total := 0

accumulate := [[ total: capturedTotal : & ]] ()(
  delta : Integer
) {
  capturedTotal += delta
}

accumulate(5)
print(total) // 5
```

`capturedTotal` is another reference bound to `total`'s fixed place. It owns
nothing, never rebinds, and cannot outlive that place. Qualification and
declaration-side replacement permissions cannot become stronger through
capture.

Reference capture does not inherit `unsafe pliable` or a destructive declaration
stance. A capture that deliberately needs either must state its own applicable
source.

### Source stance and capture stance are separate

A source expression can state how lambda construction consumes it:

```zax
callback := [[ value as last: myValue ]] ()() bound strong {
  use(myValue)
}
```

`value as last` is a one-use offer for construction of `myValue`. It does not
make later body uses terminal automatically.

The completed capture declaration may state its own future stance:

```zax
callback := [[
  value as last: myValue : MyType move
]] ()() bound strong {
  consume(myValue)
}
```

Here the outer source offers `last` during capture construction, while ordinary
body use of `myValue` offers `move`. A source-relative `anchor` may likewise
select the captured region under the ordinary structural rules.

### Capture producer results once

```zax
report := [[
  count: capturedCount:,
  text: capturedText: = makeReportInputs()
]] ()() {
  print(capturedCount, capturedText)
}
```

`makeReportInputs()` runs once while `report` is constructed. Its results map to
capture declarations through the ordinary source-to-destination routing rules.

Capture expressions and producer groups evaluate in source order and bind
immediately. The body becomes callable only after every required capture is
complete. Owned captures are dispositioned in reverse construction order when
their generated receiver ends.

## Callable prototypes and storage

Four independent choices describe a callable:

1. Its prototype states explicit results, inputs, labels, defaults,
   qualifications, and transfer stances.
2. A `bound` implementation has a receiver available through `_`; an `unbound`
   implementation does not.
3. A `final` declaration fixes one implementation; a `varying` declaration
   provides replaceable callable storage.
4. Receiver-capable storage may borrow or retain receiver lifetime through
   `unique`, `strong`, `strong atomic`, `weak`, or `weak atomic`.

Receiver-lifetime words require `bound`: an unbound implementation has no
receiver lifetime to retain.

A callback is a callable used through a parameter or stored value rather than a
separate callable category. Its visible prototype includes every ordinary and
exceptional result:

- every installed callback implementation must satisfy that outcome contract;
- the invoker handles or forwards outcomes at the callback call site;
- an implementation with additional outcomes cannot enter a slot that omits
  them; and
- a wrapper lambda may handle or reshape outcomes to expose another explicit
  callback contract.

Runtime callback selection therefore performs no handler discovery. The static
prototype already determines every possible completion outcome.

The prototype also carries an
[execution policy](compile-time-code.md#letting-a-function-decide-execution-policies),
which says whether calls may or must run while compiling. A lambda takes a
policy in the same slot as a declared function and defaults to `auto`. A
replacement installed in varying storage must be able to run everywhere the
slot can be called from, and a lambda created by compile-time code can become
part of the program only if it captures nothing. The complete rules are
described by
[compile-time code](compile-time-code.md#function-values-and-replaceable-implementations).

### Fixed and varying examples

```zax
MyType :: type {
  value : Integer

  fixed final : ()() bound = {
    ++_.value
  }

  receiverless final : ()() unbound = {
    // `_` is unavailable.
  }

  typeAndInstance final once : ()() bound = {
    if ?_
      ++_.value
  }

  replaceable : ()() bound = {
    ++_.value
  }

  retaining : ()() bound strong = {
    ++_.value
  }
}
```

`fixed` has one implementation and no replaceable per-instance slot.
`receiverless` is owned by `MyType` but receives no instance.
`typeAndInstance` supports an ordinary instance call and a receiverless type
call through the type's Nothing instance.

`replaceable` stores an implementation and, when that implementation is bound,
its receiver. Assignment may install a compatible implementation whose receiver
has another concrete type:

```zax
left : MyType
right : OtherType

left.replaceable = right.otherFunc
left.replaceable()
// `_` inside OtherType.otherFunc is right.
```

`left` locates the slot; it does not replace the receiver stored in that slot.
The receiver's type belongs to the minted implementation and is erased from the
visible compatible prototype.

For new direct storage, declaration-side `final` also resolves the omitted
type-side place stance to final. A final and varying callable therefore have
different resolved storage types even when their explicit input/result
prototype looks the same.

### Storage capacity and installed target

A receiver-capable slot can hold a target requiring no receiver:

- unbound storage accepts unbound targets;
- plain bound storage also accepts borrowed-bound targets;
- `bound unique` additionally accepts a uniquely owned receiver;
- `bound strong` and `bound strong atomic` additionally accept the corresponding
  shared receiver; and
- `bound weak` and `bound weak atomic` additionally accept weak observation.

The capacity does not force every target to use it. An unbound target remains
unbound; a borrowed target remains borrowed; assigning a strong target into a
weak slot establishes a weak observation rather than retaining strong ownership.

## Lambda receivers and receiver lifetime

A capturing lambda constructs one anonymous receiver containing its captures.
Different lambda definitions retain distinct language identities even when
their capture layouts happen to match.

Representative forms are:

```zax
plain := [[]] ()() unbound {
  performWork()
}

recursive := [[]] ()(
  value : Integer
) bound {
  if value > 0
    _(value - 1)
}

localCapture := [[ value ]] ()() {
  use(value)
}

borrowed : ()() bound = localCapture

uniqueCallback := [[ value ]] ()() bound unique {
  use(value)
}

sharedCallback := [[ value ]] ()() bound strong {
  use(value)
}

atomicCallback := [[ value ]] ()() bound strong atomic @{ myArena } {
  use(value)
}

observer : ()() bound weak = sharedCallback
```

A noncapturing lambda may be unbound. It may instead be bound when it needs its
self receiver, including recursion through `_(...)`. Capturing necessarily
creates a generated bound receiver.

Plain `bound` erasure borrows `localCapture`; its lifetime must remain valid.
Owned forms continue the receiver in arena-backed storage. Omitted allocation
policy uses the applicable default arena; `@{...}` states nondefault policy.

`atomic` changes ownership accounting, not data-race safety of the receiver.

### Inherit a complete prototype

An explicit callable destination can supply the complete lambda prototype:

```zax
MyCallback :: alias type ()() bound strong

callback : MyCallback = [[ value ]] {
  use(value)
}
```

Prototype inheritance is all-or-nothing. The destination supplies results,
inputs, labels, defaults, qualifications, binding category, and lifetime
capacity together. An inferred declaration writes the complete prototype.

An inherited parameter is in scope even though this lambda did not write it.
Leaving it unread is the same intent error as any other unread name. Acknowledge
it with the trailing marker:

```zax
MyHandler :: alias type ()(
  value : Integer
) bound strong

handler : MyHandler = [[]] {
  value #
}
```

`value #` does not declare `value`. The declaration rule is taught by
[declarations and bindings](declarations-and-bindings.md#names-the-body-may-leave-unread).

A `>|>` target may similarly supply the complete remaining prototype when target
selection is unambiguous.

### Lambdas with packs

Lambdas can work with packs in two ways: a lambda can capture a pack its
enclosing function received, or it can declare a pack parameter of its own.

A function that accepts any number of arguments receives them as a
[pack](variadics.md), which is several separately bound parameters under one
name. Suppose such a function wants to hand its arguments to a callback that
runs later. A lambda inside it can capture the whole pack by name:

```zax
sendAll$ final : (Items ... : type)()(items ... : Items) = {
}

deferSend$ final : (Items ... : type)(
  callback : ()() bound unique
)(
  items ... : Items
) = {
  return [[ items ]] ()() bound unique {
    // inside the lambda, the captured items are still a pack
    sendAll(from items as copy)
  }
}
```

`deferSend$` is compiled separately for each list of argument types it is
called with. In each of those versions the item types are known, so the
generated receiver simply has one member per captured item. Inside the
lambda, the captured items are still a pack, used just as in the enclosing
function. A stance written on the capture applies to
each item on its own, and capturing a pack does not change the lambda's own
prototype.

A lambda can also declare a pack parameter of its own:

```zax
total final := [[]] (result : Integer)(values ... : Integer) {
  running := 0
  each value : in values {
    running += value
  }
  return running
}

grandTotal := total(1, 2, 3)
```

Like a variadic function, `total` is compiled once for each number of arguments
it is called with. That is why it must be held in `final` storage: replaceable
storage holds one implementation, but a lambda with a pack parameter has one per
count.

```zax
counter := [[]] (result : Integer)(values ... : Integer) {   // error: a variadic lambda needs final storage
  return 0
}
```

You can still install `total` in a `varying` slot or pass it as a callback.
That destination has one fixed prototype, so installing `total` there selects
the one processing that matches it:

```zax
handler : (result : Integer)(first : Integer, second : Integer) = total
// handler holds total's two-argument processing
```

When the items may have different types, the lambda needs a compile-time list
of types, and that makes it a generic lambda. The `$` goes on the name, as for
every generic. The binding name carries `$`, and the lambda's first
parenthesized list holds the compile-time parameters:

```zax
prefix := "> "

show$ final := [[ prefix ]] (Items ... : type)()(items ... : Items) {
  each item : in items {
    write(prefix)
    write(item)
  }
}

show("a", 2, 3.0)                  // Items worked out from the three arguments
show$(String, Integer)("a", 2)     // Items stated
```

A `$` binding must be `final`. Constraints and `requires` work as they do for
any generic. Specialization does not apply, because a single lambda expression
has no overloads to rank.

A lambda written inline cannot be generic this way, whether it is passed,
returned, or invoked immediately, because it has no name to carry `$`. It is
always headed for a fixed prototype or a single use. Its types therefore come
from its destination, from types written directly, or from an enclosing
generic's parameters:

```zax
relayFirst$ final : (Items ... : type)()(items ... : Items)
  requires Items count >= 1 = {
  ([[]] ()(value : Items[0]) {
    write(value)
  })(items[0] as copy)
  // Items[0] is a concrete type in each processing of relayFirst$
}
```

[Generic lambdas](generics.md#generic-lambdas) covers the generic rules in
full. The pack side, including how packs travel through function values, is in
[lambdas and function values](variadics.md#lambdas-and-function-values).

## Presence, liveness, and installed binding kind

`?callback` asks whether an implementation is assigned. It does not ask whether
an assigned weak receiver can currently be promoted:

```zax
assigned := ?callback
liveNow := liveness probe callback
```

`liveness probe` returns `Boolean`, performs no acquisition, and grants no
lifetime proof. It is true for assigned unbound, borrowed, unique, and strong
targets. For an installed weak target, it reports whether strong ownership is
open at that instant. The answer may become stale immediately.

| Installed binding | `liveness probe` |
| --- | --- |
| Unavailable | `false` |
| Unbound | `true` |
| Bound borrowed | `true` |
| Bound unique | `true` |
| Bound strong | `true` |
| Bound strong atomic | `true` |
| Bound weak | Whether strong ownership is open at this instant |
| Bound weak atomic | Whether strong ownership is open at this instant |

Actual weak-to-strong construction is the operation that pins a receiver.

One query reports the installed mode:

```zax
kind := binding kind of callback
```

The result is a language-provided enum:

```zax
// `Callables` is provisional library placement; the enum surface is current.
Callables.BindingKind :: enum {
  Unavailable
  Unbound
  Bound
  BoundUnique
  BoundStrong
  BoundStrongAtomic
  BoundWeak
  BoundWeakAtomic
}
```

`Bound` is the borrowed/nonowning bound mode. The query reports the installed
target rather than the slot's maximum static capacity. The enum's canonical
namespace and any root alias remain deferred library-surface placement.

### Weak invocation is conditionally empty

A `bound weak` callable permits only a truly empty result contract: no ordinary
results and no exceptional results. An `except` result is a result and therefore
makes weak storage unavailable. A valid zero-result weak callable never panics
merely because its slot or observed receiver is unavailable.

Invocation keeps ordinary caller-side setup:

1. evaluate the callable expression;
2. evaluate explicit arguments in source order and bind them;
3. evaluate visible-prototype defaults for omitted inputs;
4. inspect the installed binding;
5. invoke an installed unbound or borrowed target normally;
6. temporarily promote an installed weak target and invoke it on success; or
7. complete as a no-op when the slot is unavailable or weak promotion fails.

Argument effects, defaults, construction, temporary lifetimes, and destruction
still occur when no body runs. This makes absence change only body invocation,
not caller-side evaluation.

No ordinary or exceptional result can be manufactured, so every resultful weak
callable prototype is unavailable. `?callback` remains false for an unavailable
slot and true when any target is assigned; callers need not guard a valid
zero-result weak invocation merely to avoid panic.

## Callable transfer, reset, and recursion

Capture expressions run only during construction. Callable transfer never
recaptures:

- `copy` copies the receiver/lifetime relationship;
- a borrowed copy remains another borrow;
- a strong copy adds participation in the same receiver lifetime;
- a weak copy adds another weak observation;
- unique ownership cannot be copied;
- `move` and `last` transfer the relationship and leave the source initialized
  but unavailable without releasing what moved away; and
- callable `deep` is undefined.

Reset dispositions the active target and restores unavailable state:

- unbound clears its implementation;
- borrowed clears only the borrow;
- unique releases the owned receiver;
- strong releases one participation;
- strong atomic releases atomically; and
- weak releases the observation, never the target.

Replacing a varying slot constructs the new callable first, dispositions the
old target, and publishes one complete successor. A panic leaves replacement or
reset as the blocked operation; it does not expose a half-updated slot.

### Repeated destructive capture use

Invocation leaves the callable alive. A capture consumed by `move` remains
moved-from; a capture consumed by `last` remains terminal. Static analysis
diagnoses later body operations invalid for that state.

Zax does not add a one-shot or self-resetting lambda mode. A programmer may
invoke and explicitly reset a slot. A later generic/reflection abstraction can
provide fire-once policy when an application needs it.

### Self-recursion uses the lambda receiver

```zax
countDown := [[]] ()(
  value : Integer
) bound {
  if value > 0
    _(value - 1)
}
```

`_(...)` calls this receiver and its minted implementation. The parentheses are
the call, as in `myValue(...)`. The dot is member access, as in
`_.capturedCount`, not part of the call. Replacing a slot
through which this lambda was reached does not redirect self-recursion. An
unbound lambda has no `_`.

## Capture a member callable

A member expression already carries its receiver:

```zax
counter : Counter

readCounter := [[]] >|> counter.read
current := readCounter()
```

The empty capture says that no additional values or ordinary inputs are bound.
`counter.read` supplies the receiver and prototype. A qualified declaration path
may participate in lookup and `type of`, but Zax has no runtime callable value
that merely waits for a receiver to be reattached.

## Partial application

`>|>` can bind selected ordinary inputs without invoking the target:

```zax
printPair final : ()(
  number : Integer,
  text : String
) unbound = {
  print(number, text)
}

number := 42

printNumberWith := [[ number: number: ]] ()(
  remainingText : String
) >|> printPair

printNumberWith("apples")
```

Construction binds `number` once. The written prototype then lists the inputs
that remain, in order, and may give them new names. Here `remainingText` is
`printPair`'s `text` under another name.

The match is by position. Writing a prototype in front of `>|>` adapts the
target, the same way installing a callable behind a compatible visible
prototype does. It is not a case of results finding inputs, so a written name
never matches a target parameter by label, even when the two names are equal:

```zax
printTriple final : ()(
  number : Integer,
  label : String,
  text : String = ""
) unbound = {
  print(number, label, text)
}

number := 42

printLabelled := [[ number: number: ]] ()(
  text : String
) >|> printTriple

printLabelled("apples")
// calls printTriple(42, "apples", "")
```

The written `text` takes the next input left after `number`, which is `label`.
`printTriple`'s own `text` is not reached and keeps its default.

When no visible change is needed, omit the written prototype and the target
supplies its remaining inputs under their own names:

```zax
printNumberWith := [[ number: number: ]] >|> printPair
```

Target defaults remain delayed until the generated callable is invoked unless a
value is explicitly captured now.

## Compose functions for later use

`>|>` builds a new callable from existing ones without calling anything. The
results of the first become inputs of the second:

```zax
parse final : (
  parsed : Parsed
)(
  source : String
) unbound = {
}

render final : (
  text : String
)(
  parsed : Parsed
) unbound = {
}

sourceText := "1 + 2"

parseThenRender := parse >|> render
text := parseThenRender(sourceText)
```

Constructing `parseThenRender` calls neither function. Each invocation calls
`parse`, passes its results to `render`, calls `render`, and exposes the final
result. Several results remain separate slots.

Results reach `render` the same way they would in the nested call
`render(parse(sourceText))`. A result whose label names one of the next
callable's parameters binds that parameter. The remaining results fill the
remaining parameters in order. Here `parsed` binds `render`'s `parsed` by
label.

A label match settles where a result goes. If the result's type does not fit
that parameter, the composition is an error, and Zax does not look for another
position to put it in. The full rule, including leftover results and
collisions, is in
[how forwarded results find their parameters](function-invocation.md#how-forwarded-results-find-their-parameters).

### Remap with `reshape`

When the labels do not line up, a reshape between the stages renames results
before they reach the next callable:

```zax
parse final : (
  tree : Tree,
  notes : Notes
)(
  source : String
) unbound = {
}

render final : (
  text : String
)(
  document : Tree,
  diagnostics : Notes
) unbound = {
}

ParseToRender :: reshape {
  tree: document:
  notes: diagnostics:
}

parseThenRender := parse >|> ParseToRender >|> render
```

`ParseToRender` sits between the two callables as a stage of its own. A reshape
stage relabels the callable on its left, so
`parse >|> ParseToRender >|> render` means
`(parse reshape ParseToRender) >|> render`. The relabeled results then reach
`render` under the usual rule: `tree` binds `document` by label, and `notes`
binds `diagnostics`.

The three-part source is still one composition operation. The reshape is not
invoked, and it creates no intermediate aggregate.

Only retained `>|>` composition has reshape stages. In an immediate `|>` chain,
you apply the reshape postfix to one stage's invocation instead, as
[reshape a stage's results](#reshape-a-stages-results) shows.

All of a reshape's entries apply at once, so one reshape can swap labels or
shift several along. What it cannot do is leave two results with the same
label:

```zax
produce final : (a : Integer, b : Integer)() unbound = {
}

consume final : ()(b : Integer, c : Integer) unbound = {
}

AToB :: reshape {
  a: b:
}

broken := produce >|> AToB >|> consume   // error: two results labeled b

AToBAndBToC :: reshape {
  a: b:
  b: c:
}

pipeline := produce >|> AToBAndBToC >|> consume   // a feeds b, b feeds c
```

`AToB` renames `a` to `b` while `produce`'s own `b` keeps its label, which
leaves two results labeled `b`. `AToBAndBToC` moves both at once, so nothing
collides.

A reshape can also pick out a result that has no label by its position, as
described in
[naming results by position with `[N]`](structural-shapes-and-compatibility.md#naming-results-by-position-with-n).

### Exceptional outcomes in retained composition

`>|>` exposes the union of its stages' exceptional outcomes. A later stage runs
only after every prior stage selected success:

```zax
parse final : (
  tree : Tree,
  failure except : ParseError
)(
  source : String
) unbound = {
}

render final : (
  text : String
)(
  tree : Tree
) unbound = {
}

pipeline := parse >|> render
```

If `parse` selects an exceptional outcome, `render` is not invoked. Calling
`pipeline` must handle or forward that parse outcome. An exceptional outcome
declared by a later stage remains independently exposed.

A retained callable prototype requires unique exceptional labels. If a later
stage also declares `failure`, reshape one of them before composing:

```zax
renderChecked final : (
  text : String,
  failure except : RenderError
)(
  tree : Tree
) unbound = {
}

ParseFailureNames :: reshape {
  failure: parseFailure:
}

renamedParse := parse reshape ParseFailureNames
checkedPipeline := renamedParse >|> renderChecked
```

Applying the reshape adapts `parse`'s visible outcome label without invoking it.
The composed callable preserves payload type, qualifications, transfer stance,
origin, and ordinary-versus-exceptional category. It never merges same-typed
failures or chooses a destination by type.

Complete outcome production, catch, forwarding, and conditional elision are
taught by [Zax exceptional result flow](except.md).

## Chain calls immediately

`|>` performs calls now:

```zax
result := 5 |> double() |> square() |> half()
```

It evaluates the value on its left once, passes it to the next call, completes
that call, and advances with its results. `>|>` instead constructs a retained
callable for later invocation.

Each step of a chain is the nested call written in a different order.
`x |> f(rest)` means `f(x, rest)`, and `g() |> f(rest)` means `f(g(), rest)`.
The values arriving from the left and the arguments written in the call are
routed together, as one argument list:

```zax
produce final : (text : String, number : Integer)() = {
}

consume final : ()(flag : Boolean, number : Integer, text : String) = {
}

produce() |> consume(true)
// the same call as consume(produce(), true): number and text bind by label,
// and true fills the remaining flag
```

The same holds for a plain value. It goes where the nested call would put it,
and a label written later in the call moves the cursor just as it does in any
other call:

```zax
place final : ()(a : Integer, b : Integer = 0, c : Integer, d : Integer) = {
}

place(2, c: 1, 3)
2 |> place(c: 1, 3)   // the same: 2 → a, 1 → c, 3 → d; b keeps its default
```

Because a chain is an ordinary call, it follows the general routing rule in
[how forwarded results find their parameters](function-invocation.md#how-forwarded-results-find-their-parameters).

### Reshape a stage's results

Sometimes a stage's result labels steer its results to the wrong parameters of
the next stage. To rename them first, write `reshape` after that stage's
invocation. Unlike retained `>|>` composition, an immediate `|>` chain has no
separate reshape stage, so the reshape attaches to a call:

```zax
split final : (head : String, tail : String)(text : String) = {
}

joinReversed final : (result : String)(tail : String, head : String) = {
}

SwapEnds :: reshape {
  head: tail:
  tail: head:
}

myText := "first second"

reversed := myText |> split() |> joinReversed()
// by label: head → head, tail → tail

swapped := myText |> split() reshape SwapEnds |> joinReversed()
// the same call as joinReversed(split(myText) reshape SwapEnds):
// split's head now reaches tail, and its tail reaches head
```

Every stage after the starting value is an invocation, so there is always a
call to attach a reshape to. A plain value at the start of a chain never needs
one. Write it where it belongs in the call directly.

### Exceptional outcomes in a chain

Any stage of a chain may fail. In the example below, both `parse` and `render`
declare an outcome labeled `failure`. If `parse` fails, `render` never runs,
and the handlers at the end of the chain see the failure. The reshape renames
`parse`'s outcome to `parseFailure`, so each stage's failure gets its own
catch:

```zax
parse final : (
  tree : Tree,
  failure except : ParseError
)(
  source : String
) = {
}

render final : (
  text : String,
  failure except : RenderError
)(
  tree : Tree
) = {
}

showSource final : ()(source : String) = {
  text := source |>
    parse() reshape failure: parseFailure: |>
    render() catch parseFailure {
      return
    } catch failure {
      return
    }
  print(text)
}
```

The general rule is that a stage selecting an exceptional outcome skips every
later stage. A trailing handler sees the exceptional outcomes of every stage
that was reached.

Without the reshape, the two `failure` outcomes could share one catch at this
immediate use. That catch is specialized separately for each stage the outcome
may come from. Use a reshape, as above, when the stages need distinct names.

### Run a chain for each item of a pack

Sometimes every argument in a pack needs the same treatment before it moves on,
such as padding each line or splitting and rejoining each string. Inside a
function that accepts any number of arguments as a [pack](variadics.md),
`from each item : in pack |> stages` runs the chain once for each item. The
results take the pack's place in the call as a new group of arguments:

```zax
split final : (head : String, tail : String)(text : String) = {
}

join final : (result : String)(first : String, second : String) = {
}

pad final : (result : String)(width : Integer, text : String) = {
}

show final : ()(lines ... : String) = {
}

showAll final : ()(width : Integer, lines ... : String) = {
  show(from each line : in lines |> pad(width, line))
  // pad(width, line) for each line; the padded lines go to show

  show(from each line : in lines |> split() |> join())
  // split(line), then join, for each line

  show(from each line : in lines |> split())
  // error: split's tail is not discardable, so each item would offer two values
}
```

The first stage decides where the item goes. If the stage writes the item's
name, as in `pad(width, line)`, the item goes only there. If it does not, as in
`split()`, the item becomes that stage's first positional input. Later stages
receive the previous stage's results as in any chain, and the item's name stays
visible in every stage.

Each item runs through every stage before the next item starts. The whole
mapping occupies one argument position, so the enclosing call's other
arguments still evaluate in source order around it.

Intermediate stages may pass several results along, but the last stage
contributes exactly one value per item: its first result. That keeps the new
group the same length as the pack. Any later results of the last stage must be
discardable, and they go unused, as in a declaration such as
`value := measure()`, which binds the first result. That is why the third
call above fails: the chain ends at `split`, whose `tail` cannot be discarded.
To keep both parts, end the chain with a
[`-<`](structural-shapes-and-compatibility.md#recompose-several-results-with--)
stage, as in `|> -< split()`, which makes one value per item with members
`head` and `tail`.

If a stage selects an exceptional outcome, the remaining items do not run.
[Exceptional result flow](except.md#mapping-each-item-of-a-pack) covers how
that outcome is handled.

[Transforming each item](variadics.md#transforming-each-item) teaches the
complete mapping form, including arrays and transfer stances.

## Optional callable values

Optional absence and callable unavailability remain distinct:

```zax
outerAbsent : MyCallback?
presentUnavailable : MyCallback? = (: MyCallback)
presentCallable : MyCallback? = callback
```

Resetting the boxed callable restores inner unavailability. Resetting the
optional ends the boxed callable lifetime and leaves outer absence.

## Costs and diagnostics

Programmers and tools must expose:

- capture evaluation, construction, and destruction order;
- copies, conversions, borrows, moves, and terminal transfers;
- direct versus indirect calls;
- receiver and erased-destructor storage;
- arena allocation and policy;
- unique/strong/weak and local/atomic accounting;
- liveness probes and weak promotion;
- composition call sequence and mapping;
- the calls a per-item chain makes for each item of a pack;
- the processings a variadic or generic lambda receives, one per count or type
  list, and any adaptation when one is installed in a fixed prototype;
- retained outcome sets, exceptional-label reshape, and skipped later stages;
- reset/replacement work and panic paths; and
- static lifetime, origin, transfer, and repeated-state analysis.

Diagnostics should distinguish:

- malformed capture presentation;
- unavailable capture copy;
- reference escape;
- binding or storage mismatch;
- missing ownership capacity;
- an unread capture name, unless a `#` destination stored it without a name or
  the body read the bare name or `_.name`;
- resultful weak storage;
- failed composition mapping;
- a reshape stage whose relabel leaves two results with one label;
- a per-item chain whose last stage produces no result, or a result after its
  first that is not discardable;
- a variadic or `$` lambda outside `final` storage;
- repeated invalid capture use;
- unavailable invocation;
- ineligible reset;
- colliding retained exceptional labels; and
- outcome reshape that changes result category or cannot find its source.

## Boundaries and maturity

This document is current conceptual design, not a formal grammar, ABI,
implementation mapping, or conformance contract.

Still deferred:

- complete variadic composition;
- short-form lambdas, for a compact stored lambda;
- spreading several values per item into a pack at the end of a per-item
  chain, which stays an error; `-<` combines them into one value per item
  instead;
- async suspension and cancellation;
- foreign callable ABI and dynamic linking;
- canonical namespace placement for callable metadata;
- reflection schemas;
- optional callback combinator naming;
- one-shot library abstractions; and
- closure layout, control blocks, optimization, and lowering.
