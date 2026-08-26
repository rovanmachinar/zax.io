# Zax function invocation

| Field | Value |
| --- | --- |
| Status | Current conceptual design |
| Audience | Human developers reading, writing, or evaluating Zax calls |
| Applies To | Programmer-facing synchronous function invocation, argument and default binding, results, and callable selection; not a formal specification |
| Implementation State | Not established by this repository |
| Owns | Ordinary call syntax; visible callable contracts; positional, named, omitted, and type-default inputs; evaluation and binding order; result slots and completion; multiple-result expression and mapping modes; result routing; fixed-arity overload viability and preference; compatible prototype adaptation; synchronous call completion; invocation diagnostics, costs, and formatting |
| Does Not Own | Complete function declaration, capture, reassignment, or representation; complete copy, move, or `last` precedence; pointer/reference provenance; generics; variadics; hidden `mutator` access; runtime value polymorphism; arbitrary operator lookup domains; async; ABI; formal grammar; compiler implementation |
| Source / Provenance | Legacy function material together with current declaration, qualifier, construction, and source-structure constraints |

## Mental model

An ordinary call names or produces a callable and supplies its inputs:

```zax
print(message)
```

The selected callable's prototype declares its result slots first and its input
parameters second:

```zax
double final : (
  result : Integer
)(
  input : Integer
) = {
  return input * 2
}

doubled := double(21)
```

A synchronous call proceeds in understandable phases:

1. Select one available callable without executing runtime argument expressions.
2. Evaluate the callee and receiver operand once.
3. Evaluate explicit inputs in caller source order and immediately bind each one.
4. Evaluate defaults for still-unbound parameters in the selected callee
   prototype's parameter order.
5. Initialize any result slots that opt into pre-body construction.
6. Execute the body.
7. Complete every declared result on every normal exit.
8. Map or transfer the results into the surrounding context.

The body cannot observe a partially bound parameter list. Every parameter is
complete before body entry.

Zax permits flexible call forms when programmer intent remains clear. An
ambiguous call is an error until the programmer makes the intended operation
explicit.

This applies the broader
[Expressiveness serves clear intent](principles.md#expressiveness-serves-clear-intent)
and [Preference must be defensible, not guessed](principles.md#preference-must-be-defensible-not-guessed)
principles.

Inputs may be selected, labeled, omitted, or defaulted flexibly. Every normal
call still completes the callable's full declared result shape.

## Callable prototypes and visible contracts

Parameter and result declarations form a callable's visible source contract.
Their:

- ordered slots;
- names and labels;
- types and qualifications;
- input defaults; and
- result-acknowledgement policy

affect how callers can use the callable.

Function values use the same call surface as directly selected declarations:

```zax
declaredFunction(input)
functionValue(input)
instance.memberFunction(input)
```

A resolved function value supplies one prototype. It does not dynamically search
declarations that happen to share its source name.

### Default `Nothing` function values

Default initialization of a function value establishes its `Nothing` state as
defined by [Zax declarations and bindings](declarations-and-bindings.md#default-function-values).

The compiler diagnoses an invocation it can prove still targets that default
state. An otherwise unhandled invocation panics. It does not execute a no-op,
manufacture arbitrary results, or invoke an unrelated overload.

### Receiver operands

A type-defined function has a receiver operand representing the instance on
which it acts:

```zax
document.save(path)
```

The receiver expression evaluates once before explicit arguments:

```zax
makeDocument().save(makePath())
```

`makeDocument()` completes before `makePath()`.

Receiver type and qualification participate in candidate selection. Inside the
body, current-instance member access remains explicit through `_`:

```zax
Document :: type {
  save final : ()(
    path : Path
  ) readonly = {
    writeTo(path, _.contents)
  }
}
```

Complete receiver qualification is defined by
[Zax qualifiers](qualifiers.md#receiver-operands).

## Positional and named inputs

An ordinary positional input is an expression:

```zax
send(stream, bytes)
```

A named input uses the parameter label:

```zax
send(stream, encoding: utf8, bytes)
```

Parameter labels are source-facing contract elements. Renaming a public label can
break labeled callers even when the callable's executable types remain
compatible.

### Positional cursor

Input mapping is tested separately for each callable candidate:

1. The positional cursor starts at the first parameter.
2. A positional input maps the current parameter and advances the cursor.
3. A named input maps that label and resets the cursor to the parameter after it.
4. A parameter may be mapped at most once.
5. A positional input never skips the current parameter merely because that
   parameter has a default.

Given:

```zax
configure final : ()(
  host : Host,
  retries : Integer = 3,
  verbose : Boolean
) = {
}
```

this does not skip `retries`:

```zax
configure(host, true) // true maps to retries, then fails by type
```

The caller can acknowledge the omitted slot or use a later label:

```zax
configure(host, :, true)
configure(host, verbose: true)
```

A later positional input follows the most recent named input:

```zax
configure(host: host, 3, true)
```

Here `3` maps to `retries` and `true` maps to `verbose`.

### Explicit positional intent

Leading `:` marks an input as explicitly positional:

```zax
send(: stream, bytes)
```

It is normally unnecessary for an ordinary expression. It becomes useful when a
declaration expression or another form could otherwise resemble a label.

## Whitespace and intent

Whitespace does not create Zax grammar, but it must honestly present programmer
intent.

In a label-capable position:

```zax
encoding: value   // label
encoding : Type   // declaration
```

A bare declaration expression at the top level of that position is an intent
error because it can easily be a mistyped label:

```zax
send(stream, encoding : Encoding, bytes) // error
```

Group the declaration expression or make its positional intent explicit:

```zax
send(stream, (encoding : Encoding), bytes)
send(stream, : encoding : Encoding, bytes)
```

A named input already establishes its outer label:

```zax
send(stream, encoding: local : Encoding, bytes)
```

This rule depends on source form, not the current overload set. Adding or
removing an overload cannot reinterpret the parse.

## Argument declaration expressions

An input expression may introduce a binding:

```zax
send(
  : temporary : Buffer,
  inspect(temporary)
)
```

The binding:

- becomes visible after its initialization completes;
- is available to later explicit inputs in the same call;
- survives through call completion; and
- does not escape into the following statement merely because it has a name.

Grouping supplies the same declaration-expression boundary:

```zax
send((temporary : Buffer), inspect(temporary))
```

Ordinary declaration and visibility behavior is defined by
[Zax declarations and bindings](declarations-and-bindings.md).

## Evaluation and immediate binding

Explicit value-producing inputs evaluate strictly in caller source order. Each
one immediately initializes or binds its already mapped parameter before the
next input begins:

```zax
source : Integer = 1

observe(
  source,
  action: changeSourceToTwo(source)
)
```

If the first parameter copies, it captures `1`. If it references `source`, it
binds before the mutation but may observe `2` when the body later reads the
referent. A move or `last` transfer likewise occurs before
`changeSourceToTwo(source)` starts.

This order is programmer-visible. An optimization may elide storage or operations
only when it preserves:

- callee, receiver, and argument side effects;
- selected candidates;
- copied values and bound referents;
- source availability after transfer;
- construction and destruction effects; and
- result production and mapping.

### Callbacks and reentrancy

A callback invoked while evaluating an input completes before the next input
starts. A callback invoked by immediate parameter construction is part of that
binding step.

Reentrant code may observe completed earlier effects. It must not observe a
fiction that later parameters are already bound.

## Omitted inputs and defaults

A parameter may declare a default expression:

```zax
connect final : ()(
  host : Host,
  attempts : Integer = 3,
  trace : Boolean = false
) = {
}
```

Omission uses that declared default:

```zax
connect(host)
connect(host, trace: true)
```

An empty input explicitly acknowledges omission:

```zax
connect(host, :, true)
connect(host, attempts:, trace: true)
```

The candidate is unavailable when an omitted parameter has no declared default.

### Default evaluation order

Empty omission entries map slots but perform no runtime evaluation. After every
explicit value-producing input completes, defaults for still-unbound parameters
evaluate in input-parameter declaration order in the selected callee's visible
prototype.

```zax
operation(
  third:,
  first: makeFirst(),
  second: makeSecond()
)
```

`makeFirst()` runs before `makeSecond()`. Only afterward does any omitted default
run.

This order allows deterministic dependent defaults:

```zax
operation final : ()(
  first : A,
  second : B = makeB(),
  third : C = makeC(second)
) = {
}
```

When both are omitted, `second` completes before `third`. A default may refer to
the receiver operand, applicable surrounding callable state, and earlier
parameters in prototype order. It may not refer to itself or a later parameter.

### Explicit type-default expressions

Omission and type-default initialization are different:

```zax
connect(host, attempts:, trace: true)  // declared default, evaluated later
connect(host, attempts: :, trace: true) // type-default Integer, evaluated here
```

The inner `:` is an actual expression producing the mapped parameter type's
default value. It runs at that source position and overrides any declared
parameter default.

The positional form is:

```zax
connect(host, : :, true)
```

## Visible defaults and compatible implementations

Defaults belong to the visible callable contract used at the call site:

```zax
callback : ()(
  timeout : Duration = configuredTimeout()
) = implementation
```

Calls through `callback` use `callback`'s label, default, types, and
qualifications. Defaults declared on `implementation` remain part of
`implementation`'s direct call contract and are not layered under
`configuredTimeout()`.

The default expression evaluates once for each actual omission. A captured
environment or retained state needed by that delayed expression is a cost of the
visible contract.

## Result slots

A selected prototype declares an ordered result shape:

- `()` has zero results;
- `(value : T)` has one result; and
- `(first : A, second : B)` has two results.

Each result declaration establishes:

- an ordered result slot;
- a source-facing label, type, and qualifications; and
- an obligation to contain one complete value on every normal exit.

An ordinary result slot begins unconstructed:

```zax
make final : (
  result : Item
)() = {
  return source
}
```

`return source` directly constructs the slot. It does not default-construct an
`Item` and then assign over it.

### Opt-in result initialization

A result can request construction before body entry:

```zax
make final : (
  result : Item = :
)() = {
  result.name = "example"
}
```

`= :` requests type-default construction. A general initializer directly
constructs from its expression:

```zax
make final : (
  result : Item = makeItem(input)
)(
  input : Input
) = {
  result.finishInitialization()
}
```

After every input and omitted default is complete, opted-in result initializers
run in result-slot declaration order in the selected callee's visible prototype.
They may refer to any completed input parameter.

An initially unconstructed result may instead be constructed later:

```zax
make final : (
  result : Item
)() = {
  result.+++(source)
  return
}
```

The compiler tracks construction through control flow. It rejects:

- use before result construction;
- construction of one still-live result slot twice; and
- normal completion with an incomplete result.

Ordinary `=` never changes into construction according to tracked state. It
requires a live destination and performs ordinary operator selection.

## Return and completion

A value-bearing return supplies the complete result shape:

```zax
return makeFirst(), makeSecond()
```

Return expressions evaluate left to right and immediately establish their
corresponding result slots.

A bare return supplies no expressions:

```zax
return
```

It is valid only when every result slot is already complete.

Falling through the closing brace is equivalent to a bare return:

```zax
make final : (
  result : Item = :
)() = {
  result.name = "example"
}
```

A function uses one complete mechanism on each path:

```zax
make final : (
  result : Item
)() = {
  if condition {
    result.+++(first)
    return
  }

  return second
}
```

Partial value-bearing result lists are unavailable. Either:

- the return list supplies the complete result shape; or
- every named result slot is already complete and the path uses bare return or
  fallthrough.

`return result` is not the completion form for a result slot that already denotes
the output. Use bare return or fallthrough rather than suggesting a transfer from
the slot into itself.

### Position-specific `return #`

`#` in a value-bearing return occupies one result position and means that no new
explicit value is supplied for that slot. For that position it:

1. preserves an already completed result;
2. preserves a result initialized by its declaration;
3. otherwise requests type-default construction; and
4. diagnoses the return if that type cannot be default-constructed.

```zax
foo final : (
  first : Integer = 5,
  second : String
)() = {
  return #, "apple" // preserve 5
}

bar final : (
  first : Integer,
  second : String
)() = {
  first.+++(42)
  return #, "apple" // preserve 42
}

foobar final : (
  first : Integer,
  second : String
)() = {
  return #, "apple" // type-default first
}
```

Because `#` occupies a result position, the complete-result-shape rule still
holds: a `#` slot is completed, not omitted. This keeps the broad meaning of `#`
as an explicit "do not supply or retain a value here" acknowledgement while
giving it position-specific behavior:

- result declarations use `#` to permit caller omission;
- caller mapping uses `#` to discard a produced result; and
- return lists use `#` to preserve or default-complete the corresponding slot.

`return #` differs materially from a bare return: `return #` may default-complete
an unconstructed slot, while a bare return requires every slot to be complete
already. Local, type, and memory-policy uses of `#` remain
[legacy discard](../discard.md) input.

## Result labels and acknowledgement

Result declarations use ordinary declaration spacing:

```zax
produce final : (
  number : Integer,
  text : String
)() = {
  return 42, "answer"
}
```

At a result-routing site, adjacent `name:` selects the source-facing result
label:

```zax
number:, text: = produce()
```

This shorthand selects results labeled `number` and `text` and introduces
same-named inferred bindings.

This is an intent error:

```zax
number :, text: = produce()
```

The first entry is neither a complete typed declaration nor an adjacent label.

### Required and discardable results

A result marked with `#` may be silently omitted by the caller:

```zax
measure final : (
  requiredValue : Integer,
  diagnostic # : String
)() = {
  return calculateValue(), describeMeasurement()
}
```

The function still completes both slots. The marker changes caller
acknowledgement:

```zax
value := measure()    // trailing diagnostic may be omitted
value:, # = measure() // explicit discard
measure()             // error: requiredValue was not acknowledged
```

Caller-side `#` explicitly consumes and discards one result. An unmarked result
must be captured, routed, or explicitly discarded.

A short capture consumes a prefix. Every unmentioned trailing result must permit
omission.

## Expression mode and result-mapping mode

Zax distinguishes one expression value from a callable's bare result sequence.

```zax
consume(produce())
```

A bare invocation in a mapping-capable argument-list position may expose several
results to consecutive input parameters.

Grouping requires one expression:

```zax
consume((produce()))
```

This is valid only when `produce()` can form one usable expression value. Several
mandatory results cannot become one value merely through parentheses.

An explicit positional input also expects one expression:

```zax
consume(: produce())
```

Contextual `=` activates result mapping:

```zax
consume(: = produce())
```

It establishes a source-result cursor and a destination-parameter cursor and
maps the producer's bare result sequence.

Parentheses on the mapper's right side force expression mode first:

```zax
consume(: = (produce()))
```

The mapper receives one expression value and cannot recover the parenthesized
call's original multiple-result sequence.

Parentheses are therefore not cosmetic at a multiple-result boundary. A
formatter must not add or remove them as stylistic cleanup.

## Result-routing groups

An explicit routing group ends with one producer:

```zax
consume(
  first: number:,
  second: text: = produce()
)
```

The producer executes once. The routing entries:

- select `produce.number` and route it to `consume.first`; and
- select `produce.text` and route it to `consume.second`.

The first label in each pair names the destination parameter. The second names
the source result.

### Two cursors

Routing maintains independent cursors:

- a source-result cursor; and
- a destination-parameter cursor.

A named destination resets the destination cursor after that parameter. A named
source resets the source cursor after that result.

Named and positional routing may mix:

```zax
consume(
  first: number:,
  : text: = produce()
)
```

`number` routes to `first`. `text` routes to the positional destination after
`first`.

Same-name routing can use shorthand:

```zax
consume(text:, number: = produce())
```

This routes `produce.text` to `consume.text` and `produce.number` to
`consume.number`.

### Existing destinations and new declarations

Capture may mix a new binding with an existing positional destination:

```zax
number:, existingText = produce()
```

`number:` selects and introduces `number`. `existingText` receives the next
positional result through ordinary assignment.

These operations are not atomic. If an earlier destination completes and a later
one panics, the earlier effect remains observable.

### Typed intermediates

A spaced colon introduces a declaration expression rather than selecting a
result label:

```zax
consume(
  first: number:,
  second: text : String = produce()
)
```

`text : String` consumes the current positional source result. It does not select
the result labeled `text`.

To select a labeled result and then construct a differently typed intermediate,
use two clear steps:

```zax
number:, text: = produce()

consume(
  first: number,
  second: widget : Widget = text
)
```

### Anonymous typed declarations

A complete anonymous declaration supplies an explicit expected result type while
forming one expression input:

```zax
anotherFunction(: : Integer = produce())
anotherFunction(: (: Integer = produce()))
anotherFunction((: Integer = produce()))
anotherFunction(# : Integer = produce())
```

These forms differ in how grouping, positional intent, or an anonymous binding
name resolves the surrounding syntax. Canonical formatting may prefer one form
later; their current parsing roles remain distinct.

### Source and destination discard

Source and destination discard act on different cursors:

```zax
consume(
  #: #,
  first: number:,
  second: text: = produce()
)
```

The outer `#:` supplies no consumer destination and invalidates the destination
cursor. The inner `#` consumes and discards the current producer result and
advances the source cursor. A later named destination reestablishes the
destination cursor.

Each source result and destination slot may be consumed at most once.

### Several producer groups

Several groups execute in source order:

```zax
consume(
  first: number:,
  : text: = before(),

  third:,
  fourth: = produce(),

  fifth:,
  #: # = after()
)
```

`before()`, `produce()`, and `after()` each execute once. One group completes and
routes its results before the next producer starts.

Writing `before()` again performs another invocation. Two textual calls never
share one evaluation automatically.

### Cursor exhaustion and optional tails

Simple or explicit mapping consumes producer results while destinations remain.
A discardable result is still mapped when a destination remains; optionality does
not make the mapper skip it to improve a later match.

If the producer has three results and only two destinations remain, the third may
remain unused only when it permits omission.

```zax
consumeFour(
  produceThreeWithLastOptional(),
  produceTwoMore()
)
```

All three results from the first producer map because destinations remain. The
first result from the second producer maps to destination four. Its remaining
result must permit omission. Mapping does not backtrack and discard an earlier
optional result to make room.

## Forwarding results

### Forwarding into another call

Simple forwarding is positional:

```zax
consume(produce())
```

Result labels do not automatically remap parameter names. Use a routing group to
reorder by labels.

A named argument expects one expression value:

```zax
consume(payload: produceTwo())
```

This is an error when `produceTwo()` cannot form one usable expression value.

### Forwarding from a return

A bare invocation can forward a compatible complete result sequence:

```zax
wrapper final : (
  number : Integer,
  text : String
)() = {
  return produce()
}
```

Return routing can remap labels:

```zax
return outputText: text:, outputNumber: number: = produce()
```

Grouping always requires one expression:

```zax
return (produce())
```

It does not preserve a multi-result sequence.

Result routing itself cannot be grouped as an expression:

```zax
return (
  outputText: text:,
  outputNumber: number: = produce()
)
// error: result routing cannot be grouped as an expression
```

Use naked return routing and explicit continuation for multiline layout.

### Forwarding into construction

A construction packet may forward several results as constructor inputs:

```zax
pair : Pair = [{
  produceTwo()
}]
```

This selects a compatible `Pair` constructor. It does not implicitly pack result
labels into stored members or create one anonymous structural value.

An explicit structural-promotion operation remains future structural-type work.

## Narrow expected-result selection

Result context participates in callable selection only at a direct, complete,
explicitly typed declaration boundary. Applicable contexts include:

- a fully typed local declaration;
- a fully typed declaration expression;
- a complete explicitly typed multiple-result capture;
- a result initializer; and
- a value-bearing return against a complete enclosing result prototype.

```zax
integer : Integer = parseValue(source)
floating : Float = parseValue(source)
byte : Byte = parseValue(source)
text : String = parseValue(source)
```

This allows one discoverable operation family instead of names such as
`parseValueInt`, `parseIntegerValue`, or `intParseValue`.

An inferred declaration supplies no expected result:

```zax
inferred := parseValue(source)
```

Later use, ordinary assignment, unresolved operators, and outer overloaded calls
do not invent one:

```zax
value = a + parseValue(source)
value := parseValue(source) + parseValue(source)
consume(parseValue(source))
```

The inner call must resolve independently unless source introduces a complete
typed declaration or explicit result-label constraint.

A flow condition is not one of these contexts. A condition validates the result
already selected under ordinary rules rather than supplying an expected
`Boolean`, so an operator whose direct candidates are ambiguous is an error there.
An anonymous typed declaration is the accepted way to supply that context:

```zax
if : Boolean = ?value
  useValue(value)
```

Which operator that selects, and when the opposite operator may supply a
fallback, are owned by
[operators](operators.md#direct-selection-ambiguity-and-opposite-operator-fallback).

### Same-family matching

Expected-result matching considers:

- result arity;
- base value type;
- qualifications;
- indirection or reference shape; and
- same-family direct transfer viability.

For:

```zax
value : Integer = makeRandom()
```

results declared as `Integer`, a qualified `Integer`, or `Integer &` may be
same-family candidates subject to direct copy, move, reference, and qualification
rules.

A `Widget` result does not participate merely because an `Integer` constructor
could accept `Widget`. Result selection never searches arbitrary converting
constructors.

Make an intended conversion boundary explicit:

```zax
widget : Widget = (: Integer = makeRandom())
```

The anonymous `Integer` declaration selects the result family. Ordinary `Widget`
construction then consumes that completed value.

### Branch-specific selection under a conditional expression

A conditional expression (`condition ?? c ;; d`) evaluates its condition once and
then only the selected arm. The surrounding operation may resolve a different
callable, overload, or constructor for each arm:

```zax
e := f + (condition ?? c ;; d)
```

This may invoke `f + c` or `f + d`, including different overload bodies, while
preserving strict source evaluation:

1. evaluate `f` exactly once in its original position;
2. evaluate `condition` once;
3. evaluate only `c` or `d`; and
4. invoke the operation selected for that path.

Branch-specific selection does not introduce a second inference model. The narrow
expected-result rule still applies independently on each arm from the same direct,
complete, explicitly typed declaration boundary; the conditional expression does
not propagate an expected result backward from a later statement or outer call.
Each runtime path must converge to one statically usable result shape before the
complete operation completes. The shared condition, single-arm evaluation, and
convergence requirement are owned by
[core flow control](core-flow-control.md#conditional-expression-and-branch-convergence);
this document owns which callable each arm selects.

An operator node under a conditional expression may also be classified
differently on different paths, so one `&&` or `||` may be a protected
short-circuit operation on one path and an ordinary eager overload on another.
Every path is still validated at compile time. That per-path operator-node
behavior is owned by
[operators](operators.md#branch-dependent-short-circuiting) and is not restated
here.

## Candidate selection

Callable selection separates:

1. discovery;
2. mapping;
3. viability;
4. preference; and
5. availability.

### Discovery

The callee form determines the initial set:

- a resolved function value supplies one prototype;
- an identifier may name an overload group;
- a member call uses declarations available for its receiver operand; and
- operators use their future operator-specific lookup domains.

### Viability

A candidate is viable only when:

- every explicit input and omission maps without duplication;
- every required parameter receives a value or declared default;
- arguments and receiver operands can bind with required types, qualifications,
  and transfer behavior;
- explicit source-result labels exist;
- an applicable expected-result shape is compatible;
- every required result receives a valid disposition; and
- the declaration is available for invocation.

### Partial-order preference

Preference is a partial order, not a mismatch score.

For every comparable receiver, input, and applicable result slot, one candidate
may be:

- better;
- equal;
- worse; or
- incomparable.

Candidate A dominates B only when A is no worse everywhere and strictly better
somewhere.

```text
A: exact, conversion
B: conversion, exact
```

Neither dominates the other. The call is ambiguous.

The per-slot better relation must be asymmetric, transitive, and acyclic. It need
not declare every pair comparable. Copy versus readonly reference, move versus
`last`, or another semantically different operation may remain incomparable.

The detailed type, qualification, reference, pointer, copy, move, and `last`
precedence table remains future work. Its eventual rules must fit this
partial-order structure.

### Fewer defaults

After supplied receiver and input bindings compare equally, a candidate requiring
fewer omitted callable-parameter defaults is better.

This comparison counts only parameter defaults activated by omission. It does not
count nested constructor, conversion, member, result, or explicit type-default
work.

Explicit and implicit omission count equally:

```zax
operation(value)
operation(value, option:)
```

when both forms activate the same callable default.

### Expected results as a tie-breaker

Result quality can break a tie only when:

- receiver and supplied-input comparisons are equal;
- a permitted complete declaration supplies the expected shape; and
- the same-family result relation is clear.

A candidate with better inputs wins without trading that advantage against
another candidate's better result, provided its result remains viable.

### Ambiguity and unavailable operations

No tie is resolved by source, declaration, import, module, or generation order.
An equal undominated set is an ambiguity error.

Availability remains separate from match quality. A uniquely best bodyless,
forbidden, unresolved generated-versus-existing, or otherwise unavailable
candidate produces an unavailable-operation diagnostic. Selection does not fall
through silently to a weaker candidate.

Labels, casts, explicit transfers, complete declarations, and exact prototype
bindings provide source-level disambiguation.

## Compatible visible prototypes

An exact prototype can select one member of an overload group:

```zax
selected final : ()(
  value : Document readonly &
) = consume

selected(document)
```

A compatible prototype can also present better labels, defaults, and result
acknowledgement while invoking the same body:

```zax
operation final : (
  poorlyNamedResult : Widget
)(
  timeout : Duration = configuredTimeout()
) = {
  // One implementation body.
}

betterOperation final : (
  result # : Widget
)(
  expires : Duration = anotherConfiguredTimeout()
) = operation
```

Calls through `betterOperation` use `result`, `expires`, its `#` permission, and
`anotherConfiguredTimeout()`.

Compatibility is positional over ordered input and result slots. The new
prototype may:

- relabel compatible slots;
- replace call-boundary defaults; and
- change caller result-acknowledgement policy.

It may not require the body or slots to:

- reorder values;
- convert incompatible values;
- synthesize missing values; or
- reinterpret transfer behavior.

If entering the original body requires executable adaptation, write a wrapper or
lambda.

When the implementation is statically known and the new contract adds no runtime
state, a compiler may direct-call without materializing another function-value
slot. `final` alone is not a universal no-storage guarantee. A closure or
capturing default may still require representation.

This is a programmer-visible compatibility and cost model, not a promise of one
backend mapping or byte-for-byte generated code.

## Temporary lifetime and call completion

The synchronous completion boundary includes:

1. callee and receiver evaluation;
2. explicit input evaluation and immediate binding;
3. omitted-default evaluation and binding;
4. opted-in result initialization;
5. body execution;
6. production of all results; and
7. result mapping into the surrounding context.

A temporary bound to a reference parameter survives through that boundary:

```zax
inspect(makeBuffer())
```

If `inspect` receives `Buffer readonly &`, the temporary remains live throughout
`inspect`.

This does not permit a reference into the temporary to escape:

```zax
view := returnView(makeBuffer())
```

The applicable lifetime strategy must reject the escape, establish a defined
owning or extending mechanism, or require an explicit unsafe operation.

For nested calls:

```zax
consume(produce())
```

the producer's result slots survive until their transfers or bindings into
`consume` complete. A temporary result bound to an outer reference parameter
survives through the outer call.

Exact caller temporary destruction order, returned alias validity, moved-from
states, and ownership disposition remain future lifetime work.

Async suspension requires a broader completion and lifetime model. It is not
ordinary synchronous completion.

## Multiline return lists

A comma continues an already established comma-list across the following
physical newline:

```zax
return first: number:,
  second: text: = produce()
```

When the first result begins after an otherwise complete bare `return`, explicit
continuation is necessary:

```zax
return \
  first: number:,
  second: text: = produce()
```

The comma already continues the next newline. Adding `\` there is an intent
error:

```zax
return \
  first: number:, \ // error: comma already continues this newline
  second: text: = produce()
```

A continuation-only line can visibly carry the list across another newline:

```zax
return \
  first: number:,
  \
  second: text: = produce()
```

Complete continuation and indentation behavior is defined by
[Zax source structure](source-structure.md).

## Diagnostics

Invocation diagnostics should distinguish:

- no callable found;
- invocation of a provably default-`Nothing` function value;
- unknown parameter or result labels;
- label-versus-declaration intent errors;
- duplicate parameter, source-result, or destination mapping;
- exhausted or invalid positional cursors;
- omitted parameters without declared defaults;
- declared-default omission versus explicit type-default expressions;
- argument, receiver, result, qualification, or transfer mismatch;
- zero results in a value-required context;
- several mandatory results forced into one expression;
- parenthesized expression mode where mapping was intended;
- unconsumed required results;
- incomplete or duplicate result construction;
- a `return #` slot whose type cannot be default-constructed;
- result context without a complete declaration;
- branch-dependent conditional-expression arms that select incompatible callables
  or fail to converge;
- incomparable undominated candidates;
- a uniquely best but unavailable candidate;
- use after an earlier move or `last`;
- a reference escaping a temporary;
- redundant explicit continuation; and
- contradictory continuation indentation.

When candidates map source differently, diagnostics should show the
candidate-specific input and result mapping rather than only list signatures.

When result context participates, diagnostics should show the explicit expected
shape and why each candidate is exact, same-family compatible, incompatible,
unavailable, or incomparable.

Exact identifiers and presentation remain future diagnostics design.

## Costs

Programmers must be able to discover:

- callee and receiver evaluation;
- explicit-input, default, result-initializer, body, return, and routing order;
- hidden declared-default work;
- copies, moves, `last`, references, pointers, and temporary extension;
- indirect function-value calls;
- environments retained for closures or default expressions;
- result construction, remapping, omission, and discard;
- a `return #` slot preserved, declaration-initialized, or type-default
  constructed;
- branch-specific callable selection under a conditional expression;
- generated or unavailable candidate involvement; and
- future allocation, async, or synchronization machinery used by a call.

Relabeling a statically known compatible function does not inherently require new
runtime machinery. A captured default may add state or work because the
programmer requested that behavior.

## Formatting

Canonical formatting preserves:

- caller source order;
- label adjacency;
- declaration spacing;
- explicit positional intent;
- omission versus type-default expressions;
- parentheses that establish expression mode;
- contextual `=` mapping boundaries;
- source and destination labels;
- explicit transfer intent;
- comma-list continuation; and
- one continuation reason per physical newline.

```zax
send(
  stream,
  encoding: utf8,
  trace: true
)
```

Formatters must not:

- reorder arguments or routing groups;
- replace named mapping with positional mapping;
- add or remove semantically meaningful parentheses;
- convert omission into type-default initialization;
- add or remove transfer semantics; or
- silently repair contradictory continuation intent.

## Source stability

Even deterministic selection cannot prevent every API evolution hazard:

- adding a more specific overload may change the selected body;
- adding a default may make another candidate viable;
- renaming a parameter or result label breaks labeled callers;
- reordering parameters or results changes positional cursors;
- changing result arity or discardability changes mappings;
- adding a result-only overload may make inferred calls ambiguous;
- adding or changing an overload may change a conditional-expression arm's
  selected callable;
- changing a result type's default-constructibility changes `return #` validity;
- changing transfer or qualification requirements changes viability, cost, and
  post-call source availability; and
- adding or removing parentheses can change expression mode versus mapping.

Possible future mitigations include label aliases, deprecation diagnostics,
contract reflection, and versioned compatibility tooling. Source-order fallback
is not a mitigation.

## Boundaries and maturity

This document is current conceptual design, not a formal grammar,
implementation mapping, compatibility contract, or conformance specification.

Later work must preserve:

- strict caller source order and immediate binding;
- omitted defaults after explicit inputs in selected-callee prototype order;
- visible labels, defaults, and result acknowledgement;
- result slots without mandatory default-construction cost;
- all-results-or-no-results normal completion;
- the distinction between one expression and a result sequence;
- explicit result mapping and deterministic cursors;
- narrow complete-declaration result selection;
- position-specific `return #` preservation or default-completion within the
  complete-result-shape rule;
- branch-specific callable selection under a conditional expression with
  once-only source evaluation;
- partial-order overload dominance without source-order ties;
- unavailable best candidates not falling through;
- compatible prototype adaptation without executable body adaptation;
- temporary lifetime through synchronous completion; and
- clear programmer-visible costs and diagnostics.

The following remain explicit future work:

- the detailed precedence table for types, qualifications, indirection, copy,
  move, reference, pointer, and `last`;
- complete closure layout, capture syntax, reassignment, recursive function
  values, and raw function pointers;
- generics, concepts, specialization, and generic result deduction;
- variadics, partial application, composition, chaining, and split/combine
  transforms;
- complete ownership, moved-from state, returned aliases, and destruction order;
- async, cancellation, executors, and concurrency;
- runtime value-polymorphic predicates and hidden `mutator` access;
- arbitrary operator lookup domains;
- anonymous/private parameter and result labels;
- label aliases and complete reflection metadata;
- explicit promotion of several results into one structural value;
- formal grammar, diagnostic identifiers, ABI, and conformance; and
- compiler lowering and representation.
