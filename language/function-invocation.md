# Zax function invocation

| Field | Value |
| --- | --- |
| Status | Current conceptual design |
| Audience | Human developers reading, writing, or evaluating Zax calls |
| Applies To | Programmer-facing synchronous function invocation, argument and default binding, results, and callable selection; not a formal specification |
| Implementation State | Not established by this repository |
| Owns | Ordinary call syntax; visible callable contracts; bound/unbound callable invocation and installed receiver use; instance and narrow type-callable `once` invocation; the parameter/argument distinction; type parameter slots and type arguments at the shared callable depth; concrete result specialization before literal invocation; positional, named, omitted, type-default, and contextual construction-packet inputs; the positional cursor and labels at a variadic pack, and `from` expansion in argument lists; transfer-aware value/reference binding, including composition preferred projection after an expected type exists; compatibility-posture and source-anchor call/result mapping; evaluation and binding order; ordinary and exceptional completion shapes; result slots, stance, completion, destination ordering, conditional publication, and elision; the callable-facing `self` result contract; multiple-result expression and mapping modes; result packs in result slots; operator result integration; result routing, including structural decomposition, recomposition, transforming groups, catch destinations, and exceptional forwarding; the single routing rule for forwarded result sequences into calls, returns, and construction, including collisions, the single-result intent error, and incidental labels; overload viability and preference, including selection rules for variadic candidates; receiver-slot comparison; minted concrete implementations and compatible visible-prototype adaptation, including a function value selecting one processing of a variadic callable; preservation of declaration-side replacement permission through mapping, results, and captures; synchronous call completion; `operator call` input/result mapping; `inline` hints; caller-location defaults; call/index mixfix parameter segmentation at the shared callable depth; invocation diagnostics, costs, and formatting |
| Does Not Own | The variadic pack model: declaring, walking, counting, indexing, mapping, and holding packs ([variadics](variadics.md)); the `label` directive catalog entry ([compiler directives](compiler-directives.md#directive-catalog)); the cohesive exceptional-flow model ([exceptional result flow](except.md)); complete transfer meaning ([transfer stances](transfer-stances.md)); uncommitted integer evaluation and realization ([integer literals and realization](integer-literals.md)); literal source, declaration, payload, merge, and join behavior ([literal source and operators](literal-source-and-operators.md)); complete [optional behavior](optional-values.md); lambda/capture representation and callable composition ([lambdas and callable composition](lambdas-and-callable-composition.md)); composition publication, exposure, and route eligibility ([Zax composition](composition.md)); `using` resource enrollment and disposal ([Zax `using`](using.md)); operator forms and selection ([operators](operators.md), [operator catalog](operator-catalog.md)); or complete [reference origin and lifetime](lifetimes-and-references.md) |
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
7. Select and complete either the ordinary success shape or one declared
   exceptional outcome.
8. Map or transfer only that selected outcome into the surrounding context.

The body cannot observe a partially bound parameter list. Every parameter is
complete before body entry.

Zax permits flexible call forms when programmer intent remains clear. An
ambiguous call is an error until the programmer makes the intended operation
explicit.

This applies the broader
[Expressiveness serves clear intent](principles.md#expressiveness-serves-clear-intent)
and [Preference must be defensible, not guessed](principles.md#preference-must-be-defensible-not-guessed)
principles.

Inputs may be selected, labeled, omitted, or defaulted flexibly. Every
invocation completes one full declared outcome: all ordinary success results or
one named exceptional result. The programmer-facing model is taught by
[Zax exceptional result flow](except.md).

## Callable prototypes and visible contracts

Parameter and result declarations form a callable's visible source contract.
Their:

- ordered slots;
- names and labels;
- types and qualifications;
- transfer stances;
- input defaults;
- result-acknowledgement policy; and
- [execution policy](compile-time-code.md#letting-a-function-decide-execution-policies),
  which says whether calls may or must run while compiling

affect how callers can use the callable.

Function values use the same call surface as directly selected declarations:

```zax
declaredFunction(input)
functionValue(input)
instance.memberFunction(input)
```

A resolved function value supplies one prototype. It does not dynamically search
declarations that happen to share its source name.

### Bound and unbound callables

A bound implementation executes with an installed receiver available through
`_`. An unbound implementation has no receiver:

```zax
Utilities :: type {
  compare final : (result : Boolean)(
    lhs : String readonly &,
    rhs : String readonly &
  ) unbound = {
  }
}
```

`Utilities` owns the declaration, but invoking `Utilities.compare` evaluates no
instance receiver and `_` is unavailable inside the body.

`type of` inspects the callable type without evaluating the instance expression:

```zax
myValue : MyType
BoundPrototype :: alias type type of myValue.process
```

The type query does not create a runtime value with an unfilled receiver. A
member expression such as `myValue.process` forms the bound callable value.
Receiver-capable varying storage may later replace it with another compatible
unbound or bound implementation.

Declaration defaults, `final` storage, and the `once final unbound` intent error
are defined by
[declarations and bindings](declarations-and-bindings.md#bound-and-unbound-function-storage).
Lambda-generated receivers, installed binding kinds, and receiver lifetime are
defined by
[Zax lambdas and callable composition](lambdas-and-callable-composition.md#callable-prototypes-and-storage).

### Default unavailable function values

Default initialization establishes a live but unavailable function value as
defined by
[Zax declarations and bindings](declarations-and-bindings.md#default-function-values).

The compiler diagnoses an invocation it can prove still targets that default
state. An otherwise unhandled invocation panics. It does not execute a no-op,
manufacture arbitrary results, or invoke an unrelated overload.

One deliberate exception is a `bound weak` callable whose complete result
contract is empty: it has neither ordinary nor exceptional results. An
unavailable slot and a failed temporary strong promotion both complete as no-ops
rather than panicking. Any `except` result makes the callable resultful and
therefore ineligible for weak storage. Explicit arguments, defaults, parameter
construction, and other caller-side setup still occur under the visible
prototype before no body runs. This is the declared weak callable contract, not
ordinary unavailable-function behavior. Complete behavior belongs to
[Zax lambdas and callable composition](lambdas-and-callable-composition.md#weak-invocation-is-conditionally-empty).

Recognized function presence returns exactly `Boolean`. A known fixed `final`
function is statically present; a varying function value may require a runtime
presence observation:

```zax
if ?callback
  callback()
```

`reset callback` releases the callable representation and any owned captures,
then restores the unavailable state. Complete presence, reset, and relationship
to type Nothing behavior is defined by
[Zax Nothing instances](nothing-instances.md#function-values-have-presence).

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

### Type and instance calls to `once` functions

A type-owned `once bound` function may be selected through either the type or an
instance:

```zax
MyType.inspect()

value : MyType
value.inspect()
```

For `MyType.inspect()`, the type identity supplies member lookup without a
receiver source expression. This is a **receiverless type call**. `_` identifies
`MyType`'s Nothing instance and `?_` is false. The implementation may perform
type-level work, read compiler-prepared members, use custom Nothing behavior, or
branch on receiver presence. Trapping or unprepared member access uses the
available generated panic or hardware trap; missing target trap capability has
the undefined boundary defined by the Nothing owner.

For `value.inspect()`, `value` evaluates once before the explicit arguments and
supplies the ordinary receiver. `_` identifies that instance and `?_` is true.

Both forms select the same declared callable and explicit parameter/result
contract. For a `final` function, `once` adds the type-qualified
receiverless call rather than another implementation slot. `once` does not
create an overload tie between a hidden type-callable implementation and an
instance implementation.

The receiverless route synthesizes a `copy`-stanced Nothing receiver before
selection. An exact `last` receiver declaration therefore remains available to
an instance source but not to the type call:

```zax
(value as last).consume()
MyType.consume() // error when consume requires exact last receiver
```

There is no runtime change from `last` to `copy`. Aliases, forwarding, and
bound callable values preserve whether they represent receiverless
lookup or an instance receiver.

`_` is pointer-shaped, while `_.` supplies the dereferenced form. Mapping to an
expected pointer prefers `_`; mapping to an expected `MyType` may use `_.`.
Either form may be passed or returned, including through a verified `self`
result. Pointer dereference remains unchecked unless the type selected trapping
Nothing behavior.

Declaration, `final`, generated default, replacement, and prohibition behavior
is defined by
[declarations and bindings](declarations-and-bindings.md#type-callable-once-functions).
Complete Nothing policies and receiver access behavior are defined by
[Zax Nothing instances](nothing-instances.md#receiverless-and-instance-calls).

This call rule does not define global or `once` value initialization, capture,
concurrency, teardown, or generic-specialization behavior.

### `operator call`

Call syntax is a postfix delimited form rather than an ordinary post-unary
operator:

```zax
callable(arguments)
```

A declared function is implicitly callable. A custom callable type may define a
complete call operation:

```zax
// Illustrative declaration syntax.
operator call final :
  (result : Result)(
    number : Integer,
    text : String = "default"
  ) = {
}
```

The input prototype supplies argument count, labels, defaults, qualifications,
and transfers. Its result prototype uses ordinary result mapping.

Inside a flattened mixfix declaration, `call N` states how many prototype
inputs belong to one call component. Each index bracket instead contributes one
`index 1` component:

```zax
// Illustrative mixfix declaration fragment.
operator mixfix
  call 2
  binary '+'
```

Omitted `:` still occupies one call slot and requires the mixfix parameter's
declared default. A consumed call/index component produces no intermediate
call result or proxy; the complete mixfix owns the final result shape.

Exact default timing across a flattened mixfix, result forwarding,
lambda/generated callable types, construction-like call syntax, and indexing
details remain future integration. Complete tree behavior is defined by
[mixfix operators](mixfix-operators.md#call-index-and-splice-components).

### Inlining hints

A call normally jumps to the function's code and returns. Inlining instead
copies a function's body into each caller, which avoids the call overhead and
can enable further optimization, at the price of larger code. A compiler
decides this by itself, and a `final` function may express a preference with
`[<inline>]` in the slot after `final`:

```zax
smallHelper final [<inline>] : (result : Integer)(value : Integer) = {
  return value + 1
}

hotPath final [<inline=always>] : ()() = {
}

coldPath final [<inline=never>] : ()() = {
}
```

| Form | Meaning |
| --- | --- |
| no directive | `maybe`: the compiler decides |
| `[<inline>]` | `likely`: prefer inlining |
| `[<inline=unlikely>]` | prefer an ordinary call |
| `[<inline=always>]` | inline wherever possible |
| `[<inline=never>]` | keep every call an ordinary call |

Every level is a [hint](compiler-directives.md#what-a-directive-may-promise):
inlining never changes what a call does, and it is never an error when inlining
is impossible. `never` can always be honored, because a call can always remain
a call, so compilers honor it. `always` is the strongest request: a recursive function or a call
through a function value cannot always be inlined, and the compiler does what
it can, for example by inlining a local recursive helper per use. Variables
declared in an inlined body stay private to it, exactly as in an ordinary call.

## Parameters, arguments, and type arguments

Parameter and argument are distinct:

- a **parameter** is a slot declared by a prototype;
- an **argument** is caller source supplied to complete a parameter; and
- an operator operand supplies an argument to one selected parameter, exactly as
  a call input does.

### Type parameter slots

A prototype may declare a **type parameter slot**, and a caller completes it with
a **type argument**: one concrete type identity.

```zax
Source :: type {
  operator binary 'as' final : (
    result : DestinationType
  )(
    DestinationType : type
  ) readonly = {
  }
}

converted := source as DestinationType
```

A type argument:

- has no runtime storage or lifetime;
- is not evaluated at runtime;
- occupies its parameter slot for viability, preference, and diagnostics like any
  other input; and
- may determine a value result type, as it does for `as`.

Type parameter slots therefore participate at the shared callable depth rather
than being an operator-only concept. A concrete type argument is not a runtime
value and does not become one merely because a value receiver supplied
discovery.

Declaration ownership and type-qualified `unbound`/`once` operators are defined
by
[declarations and bindings](declarations-and-bindings.md#operator-phrase-declarations-and-type-parameters).
A type parameter slot is one of the
[two forms of generic](generics.md#two-forms-of-generic); generic behavior is
defined by [generics](generics.md).

### Literal result specialization

A generic literal result is specialized before its body is invoked:

```zax
myDefault := h'FF'  // selected result UInteger
myByte : U8 = h'FF' // selected result U8
```

The literal declaration supplies one generic result slot, not several
result-only overloads. The payload is a
[compile-time input](generics.md#when-code-is-processed-once-per-input).
Compile-time processing uses the payload, candidate hole, sign intent, and
suggestion to select one concrete result prototype. The body is processed and
invoked under that prototype.

Ordinary invocation then sees exactly the same contract as a declaration that
had stated the concrete type directly. An `uncommitted` generic placeholder:

- is not a runtime or anonymous result;
- cannot enter ordinary result routing or an enclosing operator;
- does not let value magnitude rank overloads;
- does not reopen after invocation; and
- does not retry another result type after selected range failure.

Complete programmer-facing literal behavior is defined by
[Zax literal source and literal operators](literal-source-and-operators.md#every-invocation-has-one-concrete-result).

### Declaration-side replacement permission

Qualification travels with an argument, and declaration-side replacement
permission is part of that truth. A path that may not replace its own value
lifetime cannot regain that authority by being mapped into a parameter,
forwarded through a result, or captured:

```zax
replaceThrough final : ()(target varying : MyType immutable writable varying &)

restricted final : MyType immutable writable varying & = value
replaceThrough(restricted) // error: restricted may not delegate replacement authority
```

The referent type use still records that another path may replace the referent;
only this declaration's permission is narrower. Complete capability-versus-permission
behavior is owned by
[qualifiers](qualifiers.md#type-side-truth-versus-declaration-side-permission).

### Transfer-aware value and reference parameters

A transfer stance first changes which parameter contract the argument may
satisfy. The stance expression performs no transfer because no consumer has been
selected or run yet. After selection, the parameter's binding form determines
which object the function may change.

The ordinary contracts are:

```zax
read final : ()(
  input : Message readonly & copy
) = {
}

consume final : ()(
  input : Message mutable writable & move
) = {
}
```

A reference-shaped `move` or `last` parameter refers directly to caller-owned
storage. The selected function may take ownership of resources from that value
during the call:

```zax
consume final : ()(
  input : Message mutable writable & move
) = {
  send(input.payload as move)
}
```

A by-value `move` or `last` parameter instead constructs an independent
parameter first. The caller's source is preserved by that copy-in barrier;
resource transfer affects the local parameter:

```zax
consume final : ()(
  input : Message move
) = {
  send(input.payload)
}
```

By-value `deep` performs `deep` construction at the binding boundary and retains
`deep` stance in the body. By-value `copy` performs ordinary `copy`
construction.

For an ordinary writable source and otherwise equal `move`/`last` candidates, the
normal mutable/writable reference form is better than by value. A readonly
restatement can make that reference nonviable:

```zax
consume((message as readonly) as move)
```

If a readonly-reference variant also remains viable, neither candidate may be
better overall: the reference may avoid copying, while by value preserves
caller-owned storage. The later selection model calls this **incomparable**. The
call is ambiguous rather than silently choosing one ownership effect.

Complete stance meaning, fallback, projection, and source post-state are defined
by [Zax transfer stances](transfer-stances.md).

### Array values, bounded references, and slices

An owning array by-value parameter constructs an independent local array:

```zax
consume final : ()(
  values : Integer[2..10]
) = {
}
```

The caller's array must currently contain between two and ten elements. If the
compiler already knows that it does not, the call is an error. Otherwise the
call checks the current length and panics before entering the body when it falls
outside that range.

The local parameter is a newly constructed owning array. The source's
`copy`, `deep`, `move`, or `last` stance determines how that construction uses
the caller's elements and storage.

When the parameter names a provider instance, the local array uses that
provider. When it names only `storage type MyStorage`, construction obtains a
provider of that type. With no storage requirement, construction may reuse a
compatible source provider or choose the destination context's default.

A source array using another provider remains acceptable because this by-value
boundary constructs new destination storage.

A bounded array reference may adapt another intrinsic array while retaining its
underlying bounds and storage capabilities:

```zax
change final : ()(
  values : Integer[0..10] &
) = {
  values.resize(10)
}
```

The call first checks that the array's current length fits `0..10`. The
reference still remembers what the original array can actually do. A later
resize must satisfy both the parameter's written limits and the original
array's limits.

Remembering those two sets of limits may require extra metadata, so this
adapting reference can cost more than a reference to one exact array type.

An array reference without a storage requirement is also storage-erased. Inline
arrays and arrays using different provider instances/types can bind to the same
concrete function implementation. The reference carries the common operations
needed to locate elements through the source storage and may therefore be wider
or more indirect than an exact storage-qualified reference.

Binding such a reference creates no second array and copies no elements. A
reference naming an exact provider type or instance accepts only a source using
that storage because no destination storage is constructed.

The source's complete declared length range need not fit inside the parameter
range. Binding checks the current length. Later writable size changes check both
the parameter's visible range and the source's actual range and may panic when
only one accepts the request.

A readonly array reference cannot change length, so after its entry check it
does not need the source to support every size named by the parameter. A future
no-panic callable capability may request stronger static range containment;
ordinary writable references remain dynamically adapting.

A by-value `T[]` parameter copies only the slice's window description; it does
not copy the viewed elements. `T[] &` refers to that description and may bind
to a temporary slice created from an array for the duration of the call.
Complete identity, construction, storage selection, qualification, and failure
behavior belongs to
[Zax arrays and slices](arrays-and-slices.md#array-and-slice-parameters).

### Preferred composition projection

Preferred projection begins only after normal callable discovery has found a
candidate and that candidate supplies an expected input type:

```zax
tune final : ()(target : Engine readonly &) = {
}

Car :: type {
  engine preferred : Engine
}

car : Car
tune(car) // projects the preferred Engine member
```

The call first finds `tune`; it does not search through `car`. The selected
prototype then says that this input needs an `Engine readonly &`. Because `car`
does not match directly, its `preferred` member may supply `car.engine`.

An exact input match always beats preferred projection. If several projected
routes remain equally viable, the call is ambiguous rather than choosing by
declaration order. Projection also does not invent a target for generic
deduction or structural matching.

This is an input-mapping step, not callable or operator discovery. Complete
route declaration, qualification, nesting, and ambiguity behavior belongs to
[Zax composition](composition.md#expected-type-projection-with-preferred).

### Structural compatibility posture

After discovery supplies one concrete parameter type, an argument's
compatibility posture may make a safe binary-compatible recast viable. Exact
identity remains better than posture-based adaptation. Posture never widens
callable or operator discovery, invents an expected type, or ranks several
non-identity conversions by apparent closeness.

A local parameter receives the posture written on its own declaration after
binding; it does not inherit the caller's posture automatically. Complete
posture and anchor behavior belongs to
[Zax structural shapes and compatibility](structural-shapes-and-compatibility.md#compatibility-posture).

An explicit composition adapter uses the same projection rule when its visible
outer parameter must map to a contained input:

```zax
Tank :: type {
  fuel preferred : Fuel

  refill final : ()(source : Tank) = via fuel.refill
}
```

The adapter may map `source` to `source.fuel` automatically because `fuel` is
`preferred`; `own` alone would not authorize that mapping. A written body may
always pass `source.fuel` explicitly without declaring a preferred route.

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
6. The cursor skips every parameter already supplied by name. Two kinds of
   name match claim their parameters before any input is placed by position,
   wherever they are written in the call: a forwarded result's label, and a
   member decomposed with
   [`>-`](structural-shapes-and-compatibility.md#decompose-one-value-with--).
   An explicit label claims its parameter when the call reaches it in source
   order. After a named input, the next positional input therefore fills the
   first parameter after it that is still open. A duplicate arises only when a
   label names a parameter that is already filled. Forwarded results are
   covered under
   [how forwarded results find their parameters](#how-forwarded-results-find-their-parameters).

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

### Inputs around a variadic pack

Some functions take any number of extra arguments, such as a list of options. A [variadic parameter](variadics.md#declaring-a-variadic-parameter)
collects them into a **pack**. To the positional cursor, a pack is simply a
parameter that keeps accepting inputs:

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

connect(myHost)                              // attempts defaulted; extras empty
connect(myHost, 5, myFast, mySecure)         // 5 maps attempts; both options join extras
connect(myHost, :, myFast, mySecure)         // attempts defaulted explicitly
connect(myHost, extras: myFast, mySecure)    // the label starts the pack; it keeps absorbing
connect(myHost, 5, myFast, where: myPlace)   // a parameter after the pack only by label
connect(myHost, myFast, mySecure)            // error: myFast maps attempts, then fails by type
```

Parameters before the pack map as usual. A positional input still never skips
a defaulted parameter, which is why the last call fails: `myFast` lands on
`attempts`. Once the cursor reaches the pack, it stays there and absorbs every
positional input that follows. A parameter declared after the pack can
therefore be reached only by its label. You can also label the pack itself:
`extras:` puts the cursor on the pack, and the inputs after it keep joining.

A prototype may declare several packs. Each one after the first is a parameter
after a pack, so it starts at its label and absorbs until the next label:

```zax
tagged$ final : (Tags ... : type, Values ... : type)()(
  tags ... : Tags,
  values ... : Values
) = {
}

tagged("draft", "urgent", values: 1, 2.5)   // tags takes two inputs; values takes two
```

To pass values you already hold as separate inputs, write `from`. It expands a
pack or a fixed-length array into positional inputs that move through the
ordinary cursor, so they can land in a pack or in ordinary parameters:

```zax
plot final : ()(x : Integer, y : Integer) = {
}

myCoords : Integer[2] = [ 3, 4 ]
plot(from myCoords)   // 3 maps x, 4 maps y
```

The number of inputs to a call must be known while compiling, so a
runtime-length slice or array cannot be expanded this way.
[Variadics](variadics.md#calling-with-a-pack) covers expanding packs, stances
on the expanded items, and mapping each item with
`from each item : in items |> stages`.

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

### Construction packets as arguments

A bare construction packet has no independent expression type. Once callable
mapping identifies one concrete expected parameter type, however, that parameter
supplies the packet's destination:

```zax
draw final : ()(point : Point &) = {
}

draw(point: [{ .x = 4, .y = 5 }])
draw([{ .x = 4, .y = 5 }]) // positional form
```

This constructs an actual `Point` temporary and binds it through the selected
parameter. It is not an anonymous structural value or a posture-based
conversion.

Candidate selection tests packet viability independently for each already
discovered parameter type. If several callable candidates remain equally viable,
the call is ambiguous rather than letting the packet invent a destination.
Nested packets receive their destinations recursively from the selected
construction plan.

The explicit typed forms remain available:

```zax
draw((: Point = [{ .x = 4, .y = 5 }]))
draw(point: (: Point = [{ .x = 4, .y = 5 }]))
```

`(: Point = ...)` is an anonymous typed declaration expression. The
`point:` in the final call is the named parameter mapping; the two roles are
independent.

Complete packet behavior is defined by
[Zax construction, replacement, and destruction](construction-and-destruction.md#construction-packets).

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
referent.

A by-value `move`, `last`, or `deep` parameter performs its required construction
during this immediate binding step. A reference-shaped `move` or `last`
parameter immediately refers to existing storage, but its selected body may take
resources later during the call.

Producing `source as move`, `source as last`, or an optional phrase form has no
effect by itself because it only changes the stance offered to selection. The
selected consumer decides what transfer, if any, occurs.

When a consumer accepts `last optionalValue`, the optional immediately enters
terminal source state for analysis but keeps required boxed state live until the
complete consumer finishes. Complete optional state and cleanup behavior is
defined by
[Zax optional values](optional-values.md#move-and-last-source-adapters).

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
configure(
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

### The caller's source location as a default

Because a default is caller-side setup, a default argument can report where the
call was written. `Compilation.caller` is valid only as a default argument, and
gives the call site's location:

```zax
trace final : ()(where : SourceLocation = Compilation.caller) = {
  print(where.file.path, where.line)
}

trace()   // reports this line, not a line inside trace
```

The [`Compilation`](compile-time-code.md#compilation-information) namespace
describes `SourceLocation` and the other compile-time facts.

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

A raw pointer parameter may allocate its omission default:

```zax
useValue final : ()(
  value : MyValue * = @
) = {
  // value is borrowed raw; the body owns no cleanup obligation.
}
```

`= @` is a default expression, not a cleanup contract on the parameter slot.
When the caller omits `value`, caller-side invocation machinery owns the
scheduled allocation temporary through call completion and binds its raw address
to the parameter. When the caller supplies a value, that source retains its
ordinary ownership or lifetime responsibility.

General default expressions preserve the same boundary:

```zax
useValue final : ()(
  value : MyValue * = makeScheduledValue()
) = {
}
```

If the producer returns a scheduled temporary, that temporary remains alive
through the call and is dispositioned afterward.

A terminally transferred open-ended default can also be adopted into an
invocation-owned cleanup obligation:

```zax
makeOpenValue final : (
  result : MyValue * last = @<
)() = {
}

useValue final : ()(
  value : MyValue * = makeOpenValue()
) = {
}
```

The caller-side boundary retains the allocation through call completion and
resets it afterward. This is valid only when the result carries allocation-root
and disposition authority. A global or ordinary borrowed result causes no
cleanup to be invented. The callee sees a borrowed raw parameter in every case.
Use a managed pointer parameter when ownership must transfer into the callee.

## Result slots

A selected prototype declares an ordered ordinary result shape:

- `()` has zero results;
- `(value : T)` has one result; and
- `(first : A, second : B)` has two results.

An `except` result instead declares one named exceptional completion shape:

```zax
read final : (
  value : MyValue,
  failure except : MyFailure
)() = {
  // ...
}
```

One invocation publishes either ordinary `value` or exceptional `failure`,
never both. A callable with several `except` results has one separate completion
shape for each. Complete declaration, handling, forwarding, reshape, and
acknowledgement behavior is taught by
[Zax exceptional result flow](except.md).

At the current conceptual depth, exceptional outcomes belong to ordinary
function and call-syntax callable prototypes. Constructor, destructor,
replacement-constructor, and non-call operator result shapes cannot declare
`except`; their available results remain ordinary shapes.

Each ordinary result declaration establishes:

- an ordered result slot;
- a source-facing label, type, and qualifications; and
- a transfer stance that becomes active after construction; and
- an obligation to contain one complete value whenever the success outcome is
  selected.

The result place is available before body execution. Its resident instance may
be established before body entry when the prototype declares an initializer, or
the body may remain responsible for constructing it. An exceptional result
accepts no initializer and may not be manually constructed early; producer
`except` or explicit forwarding constructs it when selecting that outcome.

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

An optional result therefore establishes absence or presence directly:

```zax
find final : (
  result : Item?
)(
  shouldReturn : Boolean,
  input : Item
) = {
  if shouldReturn
    return input

  return (: Item?)
}
```

Neither path first constructs an absent result and assigns over it. Complete
optional construction and transfer is defined by
[Zax optional values](optional-values.md#parameters-and-results).

A result's initializer or return expression decides how the result slot is
constructed. Once the slot exists, the stance written on its declaration decides
how later consumers see it:

```zax
make final : (
  result : Item move
)() = {
  return source as copy
}
```

`source as copy` constructs `result`. After construction, ordinary uses of
`result` in the body offer `move`, and the caller also receives a `move`-stanced
result.

A function always returns the types written in its selected prototype, even
when the compiler evaluates the call early. A number literal in `return` can
take that declared result type directly:

```zax
makeByte final : (result : Byte)() = {
  return 55
}
```

Here `55` is checked against `Byte`, and the caller receives a `Byte`. See
[Zax integer literals and realization](integer-literals.md#concrete-results-and-compile-time-execution).

### Result packs

A variadic function can return a variable number of results by declaring a
result pack. The number is fixed for each processing. A pass-through is the
simplest case:

```zax
passThrough$ final : (Items ... : type)(results ... : Items)(items ... : Items) = {
  return from items as last
}

x : Integer, y : Integer, z : Integer = passThrough(1, 2, 3)   // three results
```

`results ... : Items` declares a **result pack**. Like an input pack, it is not
one value or a tuple. Each processing of the function has a fixed number of
ordinary result slots, one per item, and each slot behaves like a separately
declared result: routing, acknowledgement, elision, and `return #` apply to
each slot individually. A caller that knows the count can capture the items with typed
declarations, as the example does.

A few rules shape how result packs are declared and used:

- The pack's types come from the inputs or from `$(...)`. Neither the body nor
  an expected result ever supplies them.
- Acknowledgement and discardability are written once on the pack and apply to
  every item.
- Individual items have no labels. A reshape can name an item by its position
  with
  [`[N]`](structural-shapes-and-compatibility.md#naming-results-by-position-with-n).
- The pack as a whole is labeled by its declared name. When the pack is
  forwarded, that label binds an input pack with the same label, and the items
  go there as a group. A pack label never binds an ordinary parameter, even
  when the pack holds one item. Without a match, the items route by position.
  A reshape such as `reshape results: values:` relabels the whole pack.
- A prototype may declare several result packs. A result declared after a
  result pack, including another pack, can be reached only by its label.

Two result packs with labeled `$(...)` supply look like this:

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
// firsts binds firsts as a group; seconds binds seconds
```

[Variadics](variadics.md#returning-and-holding-packs) shows how to hold a
result pack when its number of items depends on how the function was called,
and how a body builds one item at a time.

### Receiver-origin `self` results

`self` on a reference or pointer result is an explicit callable-boundary
contract that every returning path returns exactly the invocation receiver `_`:

```zax
tune final : (
  result self : Engine &
)(profile : Profile readonly &) = {
  apply(_, profile)
  return _
}
```

The compiler verifies the claim. It is not inferred into a public prototype and
does not apply to copied results. The contract lets composition forwarding
preserve receiver provenance without performing a runtime outer cast. Complete
origin, escape, and lifetime meaning belongs to
[lifetimes and references](lifetimes-and-references.md#receiver-origin-self).

### Result mapping and terminal opportunity

A result's declared stance is part of the producer prototype. Omission means
implicit `copy`; caller context does not silently rewrite that contract.

An owned by-value result slot is temporary source storage, however. When that
slot maps into its one complete consumer, selection can determine whether a
`last` offer would materially change the accepted contract:

```zax
consume final : ()(
  input : Item readonly & copy
) = {
}

consume final : ()(
  input : Item mutable writable & last
) = {
}

make final : (
  result : Item
)() = {
  return source
}

consume(make())
// error: state the terminal result intent explicitly
```

This intent error requires:

- an owned by-value result slot;
- implicit result stance;
- structurally final mapping of that slot;
- and a materially different `last`-capable consumer or source outcome.

The producer may declare `result : Item last` or `result : Item copy`. The
caller may write `make() as last`, or acknowledge deliberately retained implicit
behavior with `intent<implicit-stance-at-terminal-use>{...}`.

No error is needed when no `move`/`last` consumer is available, `last` would
accept the same `copy` contract with the same outcome, or the result is discarded.
Each slot of a multiple-result sequence is considered independently.

A reference result is excluded because its slot owns an access path, not the
referred-to value. Destroying the result slot does not establish that the
referent is terminal. Pointer ownership follows the selected pointer contract,
not temporary result-slot lifetime. See
[lifetimes and references](lifetimes-and-references.md#returned-references) and
[pointers and arenas](pointers-and-arenas.md).

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

A raw pointer result may allocate and attach disposition to its result slot:

```zax
makeValue final : (
  result : MyValue * last = @
)() = {
  // result is allocated, constructed, and scheduled before body entry.
}
```

`= @` establishes the pre-body allocation and scheduled raw result.
`last` is the recommended result stance because caller mapping must transfer that
schedule rather than copy only the raw address:

```zax
value : MyValue * = makeValue()
```

The caller declaration adopts the allocation and schedules its disposition. If
the result is discarded, the result slot retains responsibility and dispositions
the successful allocation.

Reporting allocation transfers either a successful schedule or vacancy:

```zax
tryMakeValue final : (
  result : MyValue * last = @!
)() = {
}
```

A scheduled raw result with omitted stance defaults to `copy`, but that default
is suspicious enough to require acknowledgement:

```zax
intent<implicit-stance-at-terminal-use>{
  borrowValue final : (
    result : MyValue * = @
  )() = {
  }
}
```

The ordinary repair is `last`. Explicit `copy` is also legal and states a
deliberate borrowed-result contract:

```zax
borrowValue final : (
  result : MyValue * copy = @
)() = {
}
```

An immediate use may remain inside the producer result slot's lifetime. Retaining
the copied address beyond that schedule requires another proved owner, a
use-site `as last`, or narrow unsafe responsibility where the relationship may be
valid but is opaque. A proved-invalid escape remains an error.

Explicit raw `move` likewise does not transfer the declaration-attached schedule;
it is subject to the same outward lifetime boundary. `deep` is available only
when an exact deep-capable consumer exists and never implies built-in raw-pointee
cloning. Only `last` ordinarily transfers a scheduled raw result's disposition
responsibility.

For ordinary result declarations, non-reporting `@` and `@<` also promise a
non-vacant pointer whenever success is selected. The body may temporarily reset
or replace the result but must restore presence. `@!` and `@!<` permit a vacant
success result.

| Result initializer | Body-entry state | Normal-exit guarantee |
| --- | --- | --- |
| `@` | Definitely present, scheduled | Definitely present |
| `@!` | Possibly vacant, scheduled after success | Possibly vacant |
| `@<` | Definitely present, open-ended | Definitely present |
| `@!<` | Possibly vacant, open-ended after success | Possibly vacant |
| None | Unconstructed result slot | Body must construct one complete pointer value |

An opaque operation may use narrow unsafe responsibility to assert a
presence fact that analysis cannot prove. A path proved to return vacancy
cannot satisfy a non-reporting contract through unsafe assertion.

Current ordinary-result syntax cannot express a delayed-construction contract
that leaves an owning, raw, or optional result unconstructed on body entry while
promising presence whenever success is selected. `= @` provides the promise by
allocating before body entry; an uninitialized pointer or optional result
permits delayed construction but exposes no presence guarantee to callers.

References provide borrowed inputs with no independent vacancy state, but do
not preserve pointer ownership or rebinding. Unchecked pointer dereference may
mechanically bind a reference to Nothing backing; a statically proved vacant
source is diagnosed. General future callable preconditions and postconditions
should express contracts such as “this pointer is present on entry” or “this
optional/owning result is present on normal exit” without inventing
pointer-only type qualifiers.

Pre-body constructedness is part of body/prototype compatibility. A body that
expects a live result on entry cannot be reused with a prototype that leaves
that result unconstructed. A body that expects to construct a result cannot be
paired with a prototype that already constructed it. Compatible prototypes may
change initializer expressions only when they establish the same per-result
entry state required by the body.

A result without an initializer is declared by the prototype but starts empty.
The body constructs into that existing place with `.=`, the same operator a
constructor uses for its members:

```zax
make final : (
  result : Item
)() = {
  result .= [{
    .name = "example"
  }]
}
```

First construction means exactly what the same source would mean as a
declaration initializer:

```zax
result .= source        // like `result : Item = source`
result .= [{ first, second }]
result .= [{}]          // zero-input construction; for an optional, a present payload
```

The expression produces access to the newly constructed result, with the
result's own qualifications; a readonly result produces readonly access.
Explicit construction through `result.+++(source)` remains available and means
the same thing. A [`.=` routing group](#construct-existing-places-with-a-dot-equals-group)
can construct several result slots from one multiple-result producer.

First construction needs no replacement permission, so a `final`, readonly, or
immutable result can be constructed this way. Once the result is live, `.=` is
[reconstructive replacement](construction-and-destruction.md#reconstructive-replacement)
with its usual requirements; a second `.=` on a `final` result is therefore an
error.

The assignment operator `=` never constructs its destination:

```zax
make final : (
  result : Item
)() = {
  result.name = "example" // error: result is not constructed yet
}
```

Construct the result first, or request pre-body construction with `= :` when
every implementation of the prototype should receive a live result. That
initializer is part of the prototype, not a local convenience.

Whether a `.=` constructs or replaces is decided by the conservative rule in
[How the compiler decides first construction](construction-and-destruction.md#how-the-compiler-decides-first-construction).
For result slots, the compiler rejects:

- use before result construction;
- a `.=` or `+++` where the compiler cannot tell whether the result is already
  constructed, such as inside a loop or after paths that disagree; construct it
  outside that region instead;
- a second `+++` on one still-live result slot;
- a value-bearing `return` into a result that is already constructed; and
- normal completion with an incomplete result.

Ordinary `=` never changes into construction according to tracked state. It
requires a live destination and performs ordinary operator selection.

## Return and completion

A value-bearing return supplies the complete ordinary success shape:

```zax
return makeFirst(), makeSecond()
```

Return expressions evaluate left to right and immediately establish their
corresponding result slots.

A bare return supplies no expressions:

```zax
return
```

It is valid only when every ordinary success result slot is already complete.

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

Partial value-bearing success-result lists are unavailable. Either:

- the return list supplies the complete ordinary success shape; or
- every named ordinary result slot is already complete and the path uses bare
  return or fallthrough.

Producer `except` is the distinct completion form for one exceptional result:

```zax
read final : (
  value : MyValue,
  failure except : MyFailure
)() = {
  if cannotRead()
    except failure: makeFailure()

  return makeValue()
}
```

It constructs exactly the named exceptional slot, performs ordinary function
exit cleanup, and publishes no ordinary result. Every live provisional result
outside the selected shape must be destroyed by its current owner before
completion. Exact cleanup and handling order is defined by
[Zax exceptional result flow](except.md#exact-exceptional-completion-order).

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

Because `#` occupies an ordinary success-result position, the complete-shape
rule still holds: a `#` slot is completed, not omitted. Across ordinary result
contexts, `#`
explicitly declines an accessible value or destination at that position while
the surrounding construct determines the lifetime consequence:

- result declarations use `#` to permit caller omission;
- a parameter, local, or other non-result declaration uses `#` so this scope
  may leave that name unread, as taught by
  [declarations and bindings](declarations-and-bindings.md#names-the-body-may-leave-unread);
- ordinary caller mapping uses `#` to consume a produced result without exposing
  a destination;
- a `using` mapping keeps that result as an anonymous owned entry while
  suppressing body access and disposal; and
- return lists use `#` to preserve or default-complete the corresponding slot.

An exceptional outcome cannot be declared or omitted with `#`. The caller
handles or forwards it. A `catch` may still map the payload destination to
`#`, which handles the outcome and introduces no payload name. That form is
taught by [exceptional result flow](except.md#handler-destinations-use-result-mapping).

`return #` differs materially from a bare return: `return #` may default-complete
an unconstructed ordinary slot, while a bare return requires every ordinary
success slot to be complete already.

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
value := measure() // trailing diagnostic may be omitted
requiredValue: value:, diagnostic: # = measure() // explicit labeled discard
measure() // error: requiredValue was not acknowledged
```

Caller-side `#` explicitly consumes a result without making it accessible to the
caller. It does not destroy that result immediately. The inaccessible source
slot remains part of the complete result consumer and reaches its ordinary
completion position unless a surrounding construct, such as `using`, establishes
an anonymous destination lifetime for it. An unmarked result must be captured,
routed, or explicitly discarded.

A capture with fewer destinations than results, such as `value := measure()`,
consumes the leading results. Every unmentioned trailing result must permit
omission.

Protected assignment forms such as `a = b` and `place .= source` return access
that is discardable in the same way, so each is a complete statement. Operator
authors choose the same way for their own results; see
[Zax operators](operators.md#discardable-access-from-assignment-forms).

### Exceptional outcome clauses

`catch`, forwarding `except`, and outcome `reshape` attach to the complete
evaluation of the immediately preceding invocation: receiver, explicit
arguments, nested calls, and selected outer callable.

```zax
result := transform(makeValue()) catch failure {
  return
}
```

The handler may receive `failure` from `makeValue` before `transform` starts or
from `transform` after its body runs. Equal-label sources create one
branch-specialized handler mapping. An explicit typed handler destination must
accept every specialization.

```zax
result := transform(
  makeValue() reshape failure: makeFailure:
) catch failure {
  return
} catch makeFailure {
  return
}
```

Clauses run before the invocation's ordinary success results map into an
enclosing call, operator, declaration, packet, `return`, or `using` entry.
Consequently, an exceptional outcome publishes none of those success
destinations.

Handler and forwarding destinations reuse ordinary source-to-destination
mapping:

```zax
call() catch failure: localFailure: {
  return
}

call() except failure: outerFailure:
```

`catch failure` and `except failure` are same-name shorthands. Complete behavior
and the no-discard requirement are taught by
[Zax exceptional result flow](except.md).

## Expression mode and result-mapping mode

Zax distinguishes one expression value from a callable's bare result sequence.

```zax
consume(produce())
```

A bare invocation in a mapping-capable argument-list position may expose several
results as inputs. Used this way, its results form a **forwarded result
sequence**. Each result whose label names a parameter binds that parameter, and
the rest take positions under ordinary call rules.
[How forwarded results find their parameters](#how-forwarded-results-find-their-parameters)
walks through the rule.

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
maps the producer's bare result sequence by position. Unlike a bare forwarded
sequence, it ignores result labels entirely. Write it when results should go in
result order, however they happen to be labeled:

```zax
split final : (head : String, tail : String)(text : String) = {
}

joinReversed final : (result : String)(tail : String, head : String) = {
}

myText := "left:right"
reversed := joinReversed(: = split(myText))   // by position: head fills tail, tail fills head
```

Even though both functions use the labels `head` and `tail`, the explicit form
pairs them by position, so `split`'s `head` lands in `joinReversed`'s `tail`.

Parentheses on the mapper's right side force expression mode first:

```zax
consume(: = (produce()))
```

The mapper receives one expression value and cannot recover the parenthesized
call's original multiple-result sequence.

Parentheses are therefore not cosmetic at a multiple-result boundary. A
formatter must not add or remove them as stylistic cleanup. Around a call with
only one result they make no difference. The result binds at its written
position either way, and the
[single-result rule](#a-single-result-stays-at-its-position) applies in both
cases.

A `using` resource list is another mapping-capable context:

```zax
using (produceResources()) {
  // Every result is enrolled in declared result order.
}

using ((produceResources())) {
  // The inner grouping requires one expression value.
}
```

Invocation owns the distinction between the bare result sequence and one grouped
expression. [Zax `using`](using.md#multiple-results-and-names) owns which mapped
results are enrolled, named, disposed, or destroyed.

### Operator result shapes

An operator is a callable operation and may declare zero, one, or several result
slots:

```zax
sum:, carry: = customLeft + customRight
```

An operator nested as one expression input must provide exactly one usable value:

```zax
combined := customLeft + customRight
// error if the selected + has two mandatory results
```

The same requirement applies to a word-spelled operand position, and it is what
lets a selected inner result flow outward to the next receiver:

```zax
sum:, carry: = addWithCarry(a, b)

combined := addWithCarry(a, b) next operation
// error: no single intermediate receiver
```

Several operator results do not become an implicit tuple or anonymous structure.
Grouping does not combine them. At a mapping-capable complete producer boundary,
operator results use the same acknowledgement, labels, cursors, construction,
and discard behavior as another callable result shape.

Forwarding is the one place operators differ. Only a bare invocation can be a
forwarded result sequence, and an operator expression is a single expression
node. When an operator with several results should supply call inputs, write
an explicit [routing group](#result-routing-groups). A one-result operator in
argument position is one expression value and follows the
[single-result rule](#a-single-result-stays-at-its-position), just like a
one-result call.

A built-in operator's result label is [incidental](#incidental-labels), so
ordinary arithmetic never routes by label and never raises an intent error:

```zax
record final : ()(total : Integer, result : Integer) = {
}

myLeft := 2
myRight := 3
myResult := 10
record(myLeft + myRight, myResult)   // the sum fills total by position
```

The sum fills `total` because that is where it is written, even though the next
parameter happens to be called `result`.

A programmer-declared operator follows the labels in its own declaration. If it
uses a conventional `result` label that should never route, mark the label
incidental:

```zax
MyAmount :: type {
  operator binary '+' final :
    (result [<label=incidental>] : MyAmount)(rhs : MyAmount) readonly = {
  }
}

store final : ()(total : MyAmount, result : MyAmount) = {
}

myPrice : MyAmount
myTax : MyAmount
myTotal : MyAmount
store(myPrice + myTax, myTotal)   // the sum fills total by position
```

Without the marking, the sum would carry a `result` label that names `store`'s
second parameter while sitting in the first position. That disagreement is the
single-result intent error.

Zero-result operators remain valid complete operations but cannot supply a value
to a parent expression node.

Exact built-in operator result shapes are defined by the
[operator catalog](operator-catalog.md). Operator discovery, candidate-tree
formation, and outward result flow are defined by
[Zax operators](operators.md#outward-result-flow).

## Result-routing groups

An explicit routing group ends with one producer:

```zax
consume(
  number: first:,
  text: second: = produce()
)
```

The producer executes once. The routing entries:

- select `produce.number` and route it to `consume.first`; and
- select `produce.text` and route it to `consume.second`.

The first label in each pair selects the source result. The second selects the
destination parameter. Result routing consistently reads from source to
destination.

A routing group is written intent. Unlike a bare
[forwarded sequence](#how-forwarded-results-find-their-parameters), it never
matches labels on its own: every entry states where its result goes. A
destination named in a group also moves the call's positional cursor, just as
an explicit `name: value` input does.

For a single result, a named input such as `count: getCount()` is simpler.
Reach for a group when a producer offers several results and you want only
some of them:

```zax
produceDetailed final : (text # : String, number : Integer)() = {
}

consumeNumber final : ()(number : Integer, flag : Boolean) = {
}

consumeNumber(number: = produceDetailed(), flag: true)
// selects number; the discardable text goes unused
```

`using` resource mapping reuses the same source/destination order, but its
destination label introduces a body-visible resource binding rather than
selecting a predeclared call parameter:

```zax
using (
  resource: myResource:,
  grant: myGrant: = acquirePair()
) {
  use(myResource, myGrant)
}
```

The resource owner requires source-result labels whenever a several-result
producer introduces names. A bare producer may instead enroll every result
without naming it. Complete rules are in
[Zax `using`](using.md#capture-by-source-result-label).

### Two cursors

Routing maintains independent cursors:

- a source-result cursor; and
- a destination-parameter cursor.

A named source resets the source cursor after that result. A named destination
then resets the destination cursor after that parameter.

Named and same-name routing may mix:

```zax
consume(
  number: first:,
  text: = produce()
)
```

`number` routes to `first`. The single `text:` uses same-name shorthand and
routes `produce.text` to `consume.text`.

Same-name routing can use shorthand:

```zax
consume(text:, number: = produce())
```

This routes `produce.text` to `consume.text` and `produce.number` to
`consume.number`.

A bare destination `:` instead supplies the current positional destination:

```zax
consume(
  number: first:,
  text: : = produce()
)
```

Here `text` routes directly to the positional parameter after `first`. Bare `:`
does not introduce a caller-visible name or a separate anonymous intermediate;
the existing parameter slot is the destination.

The destination cursor is not separate bookkeeping for the group. It is the
call's own positional cursor, so it skips any parameter that a forwarded
sequence elsewhere in the call has claimed by label. That holds even when the
forwarded sequence is written later in the call:

```zax
assemble final : ()(a : Integer, b : Integer, c : String) = {
}

produceA final : (a : Integer, note # : String)() = {
}

produceX final : (x : Integer)() = {
}

assemble(x: : = produceX(), produceA())
// produceA's a claims a by label, so the bare : sends x to b;
// produceA's note then fills c by position
```

This is action at a distance: a later argument decides where an earlier bare
`:` lands. If `produceA` ever stopped offering a result labeled `a`, the same
group would quietly send `x` to `a` instead. When a result's place should not
depend on the rest of the call, name the destination, as in
`x: b: = produceX()`.

### Existing destinations and new declarations

Capture may mix a new binding with an existing positional destination:

```zax
existingText : String = "previous"

number:,
text: existingText = produce()
```

`number:` selects and introduces the same-named binding. `text:` selects the
labeled source result and routes it into `existingText` through ordinary
assignment.

These mappings execute in their defined order and are not transactional. A
panic does not roll an earlier effect back or unwind to the caller: a matching
helper repairs the blocked operation and lets it resume, otherwise the process
crashes gracefully. See
[Zax safety and analysis](safety-and-analysis.md#panic-boundary).

### Construct existing places with a dot-equals group

An `=` group assigns to every existing destination. To construct existing
places instead, end the group with `.=`:

```zax
MyType :: type {
  a : A
  b : B

  +++ final : ()() = {
    first: _.a, second: mySecond : MySecond .= makeTwo()
    _.b .= mySecond.makeB()
  }
}
```

- `makeTwo()` runs once.
- `_.a` is an empty member, so `first` constructs it directly. The compiler may
  elide the producer's result slot into the member.
- `mySecond : MySecond` is a new declaration, constructed once from `second`.
- `_.b` is then constructed from `mySecond.makeB()`, and `mySecond` is
  destroyed at the end of the body.

The group's final token decides what happens to every existing destination:

```zax
number:, existingText = produce()
// `=` group: new declarations are constructed; existing destinations are assigned.

first: _.a, second: mySecond .= makeTwo()
// `.=` group: each existing destination receives `.=`.
```

A `.=` group never assigns. Each existing destination receives `.=` exactly as
if written `place .= result`: first construction when the place is empty,
replacement when it is live. When `mySecond` is already live, its entry is
replacement:

```zax
+++ final : ()() = {
  mySecond : MySecond = prepareMeTheSecondPlace()
  first: _.a, second: mySecond .= makeTwo()
  // _.a: first construction, with elision possible.
  // mySecond: replacement from `second`.
  _.b .= mySecond.makeB()
}
```

Replacement cannot elide. A live place cannot become the producer's result
slot, so `second` is an input to the replacement and its temporary is
destroyed afterwards. A `+++ replacement` on `MySecond` that accepts that input
can recycle resources the earlier construction prepared; without one, the
earlier construction only adds a construction and a destruction.

Result slots use the same form, which constructs them without completing the
function:

```zax
make final : (
  a : A,
  b : B
)() = {
  first: a, second: b .= makeTwo()
  finishSetup(a, b)
}
```

Rules:

- The [first-construction rule](construction-and-destruction.md#how-the-compiler-decides-first-construction)
  applies to each entry separately.
- In a constructor, a member used as a `.=` group destination is under
  explicit construction control.
- New declarations may appear and are constructed.
- Entries are processed in order and are not transactional, as in `=` groups.
- Replacement entries follow the ordinary replacement rules: inputs are secured
  before the old lifetime ends, and alias hazards still apply.
- A discarded `#` source result is still constructed by the producer and is
  destroyed when the mapping completes.

A `.=` group is a statement. The other routing contexts have no existing places
to construct into:

| Context | Destinations | Group token |
| --- | --- | --- |
| Call routing | Parameters that the call creates | `=` |
| `using` lists | New resource bindings | `=` |
| Capture producer groups | New captures | `=` |
| `return` routing | Result slots that `return` constructs | `=` |
| Statement routing | New declarations and existing places | `=` or `.=` |

### Typed destinations

A source-result selector may be followed by a complete typed destination
declaration:

```zax
number:,
text: message : String = produce()

consume(number, message)
```

`text:` selects the producer result. That result directly initializes the
caller-owned destination `message : String`; normal construction selects the
conversion or transfer accepted by `String`.

A typed declaration may also occur directly in a call routing group. It supplies
the next positional argument after construction:

```zax
consume(
  number: first:,
  text: message : String = produce()
)
```

This names the first destination parameter explicitly. The new `message`
declaration is initialized from `produce.text` and supplies the next positional
parameter. The source-result selector still appears before the destination
declaration; the ordinary destination cursor identifies the parameter that
receives the constructed intermediate.

Omitting the declaration name creates an anonymous typed intermediate:

```zax
consume(
  number: first:,
  text: : MyTextConsumer = produce()
)
```

`produce.text` initializes the anonymous `MyTextConsumer`. That value then
supplies the current positional parameter and remains alive through its complete
consumer. Unlike bare `:`, the type makes this a destination declaration rather
than a direct positional mapping.

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
name resolves the surrounding syntax. Once `: Integer` and `# : Integer` have
both resolved as ordinary anonymous declarations in a non-result-mapping
expression, they have the same value behavior. Canonical formatting may prefer
one form later; their parsing roles remain distinct. Complete ordinary
declaration behavior is defined by
[Zax declarations and bindings](declarations-and-bindings.md#anonymous-declarations-and-discard-names).

Anonymous typed construction is required when an expression hole adds an
optional layer around an already-optional source:

Assume `consumeOptional` accepts `Item?` and `consumeNested` accepts `Item? ?`:

```zax
inner : Item?

consumeOptional(inner) // ordinary same-type Item? argument
consumeNested((: Item? ? = [{ inner }]))
consumeNested([{ inner }])
```

The final two calls perform the same deliberate outer wrapping. The explicit
declaration spells `Item? ?`; the short form receives that destination from the
selected `consumeNested` parameter. If callable selection cannot establish one
unambiguous expected type, the bare packet remains invalid.

In the explicit form, "anonymous" describes the declaration's missing binding
name before `: Item? ?`; it does not mean that `Item? ?` is an anonymous
structural type or that the named source `inner` lacks a name.

The packet supplies constructor inputs rather than producing an anonymous
value. Optional depth intent is defined by
[Zax optional values](optional-values.md#adding-optional-depth-requires-a-packet).

### Source and destination discard

Source and destination discard act on different cursors:

```zax
consume(
  #,
  number: first:,
  text: second: = produce()
)
```

The `#` consumes the current producer result without exposing a destination and
advances the source cursor. It does not advance the destination cursor.

A destination discard acts independently:

```zax
consume(
  #:,
  number: second:,
  text: third: = produce()
)
```

`#:` supplies no consumer destination and invalidates the destination cursor. It
does not consume a source result. Later named source and destination labels
reestablish their respective cursors. Source discard does not shorten the result's
complete-consumer lifetime.

Each source result and destination slot may be consumed at most once.

The distinction is observable even when both forms are legal:

```zax
produceText final : (
  text : String
)() = {
  return "produced"
}

consumeText final : ()(
  text : String = "fallback"
) = {
  print(text)
}

consumeText(
  text: : = produceText()
) // produceText.text supplies consumeText.text

consumeText(
  text: # = produceText()
) // discard produceText.text; use the parameter default
```

Bare `:` supplies and advances the current positional destination. `#` supplies
no destination and does not advance that cursor. The second call is valid only
because the unbound parameter can be completed by its default.

Return-result routing uses the same distinction. Bare `:` supplies the current
outer result slot. `#` does not; that outer slot must already be initialized or
be completed elsewhere under the ordinary complete-result-shape rules.

### Several producer groups

Several groups execute in source order:

```zax
consume(
  number: first:,
  text: : = before(),

  third:,
  fourth: = produce(),

  fifth:,
  # = after()
)
```

`before()`, `produce()`, and `after()` each execute once. One group completes and
routes its results before the next producer starts.

Writing `before()` again performs another invocation. Two textual calls never
share one evaluation automatically.

### Cursor exhaustion and optional tails

Forwarded and explicit mapping both consume producer results for as long as
destinations remain. A discardable result is still mapped while a destination
remains. Being optional does not make the mapper skip it to improve a later
match.

If the producer has three results and only two destinations remain, the third may
remain unused only when it permits omission:

```zax
consumeFour final : ()(
  first : Integer,
  second : Integer,
  third : Integer,
  fourth : Integer
) = {
}

produceThree final : (low : Integer, middle : Integer, high # : Integer)() = {
}

produceTwo final : (start : Integer, extra # : Integer)() = {
}

consumeFour(produceThree(), produceTwo())
// low, middle, and high fill first, second, and third;
// start fills fourth; the discardable extra goes unused
```

None of these result labels names a parameter, so every result takes a
position. All three of `produceThree`'s results map, `high` included, because
destinations remain. `start` then fills `fourth`, which leaves `extra` with
nowhere to go. That is allowed only because `extra` permits omission. Mapping
never backtracks to discard an earlier optional result and make room.

### Structural result mapping

Compatibility posture belongs to each producer result declaration. It may make
that result viable for an already known compatible destination at its immediate
mapping boundary. Exact identity remains better; posture does not make result
labels or later uses invent a destination.

A source-result selector may anchor within one result:

```zax
consume(
  packet anchor .payload.point: point:,
  status: = produce()
)
```

The general form is:

```text
<source-result label> anchor <source-path>: <destination>:
```

The path belongs to the selected source result. A declared default result anchor
may supply it. One result slot remains consumable only once, and an anchored
reference cannot escape a by-value result temporary.

`-<` explicitly recomposes remaining result slots into one named or anonymous
value. It is not ordinary grouping and does not change the producer's original
slot identities before the operation.

Transformation belongs to one selected result, never to the complete producer
sequence. A transformed entry preserves ordinary source-to-destination routing
order:

```text
<source-result label>: -<>- [<reshape> -<>-] <destination>
```

For example:

```zax
Point :: type {
  x : I32
  y : I32
}

DevicePoint :: type {
  deviceX : I32
  deviceY : I32
}

PointMapping :: reshape {
  deviceX: x:
  deviceY: y:
}

Size :: type {
  width : I32
  height : I32
}

DeviceSize :: type {
  deviceWidth : I32
  deviceHeight : I32
}

SizeMapping :: reshape {
  deviceWidth: width:
  deviceHeight: height:
}

consumeParts final : ()(
  point : Point,
  size : Size,
  status : Boolean
) = {
}

produceParts final : (
  oldPoint : DevicePoint,
  oldSize : DeviceSize,
  status : Boolean
)() = {
}

consumeParts(
  oldPoint: -<>- PointMapping -<>- point:,
  oldSize: -<>- SizeMapping -<>- size:,
  status: = produceParts()
)
```

The final `= produceParts()` remains the ordinary one-producer routing boundary.
`status` maps directly; only the two entries containing `-<>-` transform.

An explicit leading bare `:` selects the current positional source-result cursor
before the destination declaration:

```zax
: -<>- PointMapping -<>- myPoint : Point,
: -<>- SizeMapping -<>- mySize : Size,
status: = produceParts()
```

A one-result invocation is an ordinary expression source:

```zax
producePoint final : (
  oldPoint : DevicePoint
)() = {
}

myPoint : Point =
  -<>- PointMapping -<>- producePoint()
```

Complete shape, posture, anchor, decomposition, recomposition, transformation,
and reshape semantics are defined by
[Zax structural shapes and compatibility](structural-shapes-and-compatibility.md).

## Forwarding results

When one call produces exactly what another needs, its results can go straight
into that call without being named first:

```zax
produce final : (text : String, number : Integer)() = {
}

accept final : ()(
  flag : Boolean,
  number : Integer,
  text : String,
  enabled : Boolean = false,
  count : Integer = 0
) = {
}

accept(true, produce())   // number and text bind by label; true fills flag
accept(produce(), true)   // the same call
```

`produce` offers results labeled `text` and `number`, and `accept` has
parameters with those same labels. Each result goes to its namesake, whatever
order either side declares them in. `true` then fills the first parameter still
open, which is `flag`, so the two calls mean the same thing.

The same rule applies wherever results meet inputs: a nested call, a `|>` chain
stage, `>|>` composition, `return`, and a construction packet.

### How forwarded results find their parameters

When a bare invocation is used directly as inputs, its results form a
**forwarded result sequence**. They find their parameters in two steps:

1. Labels match first. Each labeled result binds the parameter with exactly
   that label, and that parameter is used up. This decides only where results
   go, not when anything runs: every input still evaluates in the caller's
   source order. When either label is [incidental](#incidental-labels), no
   match happens. A label written in a postfix `reshape` is different. It is
   written intent, so it binds the parameter it names, even an incidental one.
2. Everything else follows ordinary call rules, in source order. Unmatched
   results sit where their call was written, alongside the inputs written
   around them. Positional inputs follow the cursor, and an explicit label
   moves the cursor as in any call. The cursor skips the parameters used up in
   the first step, along with any already supplied by an explicit label.

Here are both steps at work, using `produce` and `accept` from
[the opening example](#forwarding-results) and one more producer:

```zax
produceMore final : (enabled : Boolean, count : Integer)() = {
}

accept(true, produce(), produceMore())
// every result binds by label; true fills flag

accept(produce(), true, enabled: true, 5)
// true fills flag; enabled by label; 5 follows the cursor into count
```

In the second call, `number` and `text` are taken by label before anything is
placed by position. The cursor then starts at `flag` and moves just as it would
in a call with no forwarding at all.

A parameter you supplied with an explicit label is skipped the same way. After
an explicit label, the cursor moves to the next parameter that is still open:

```zax
place final : ()(x : Integer, y : Integer, z : Integer) = {
}

place(y: 1, x: 2, 3)   // after x:, y is already supplied, so 3 fills z
place(1, x: 2)         // error: x is supplied twice, by position and then by label
```

A duplicate arises only when a label names a parameter that is already filled.

Because label matches go first, a forwarded sequence can push a written input
to a later parameter. That is not an error:

```zax
measure final : (width : Integer, height : Integer)(shape : MyShape) = {
}

frame final : ()(height : Integer, width : Integer, margin : Integer) = {
}

myShape : MyShape
frame(5, measure(myShape))
// width and height bind by label, so 5 fills margin
```

`5` is written first, but `measure`'s results have already taken `height` and
`width`, so it lands in `margin`. A label match against an explicit label is a
different matter: that is a
[collision](#when-two-sources-claim-one-parameter).

A `|>` chain is the same call written in another order: `g() |> f(rest)` means
exactly `f(g(), rest)`. So a positional input after a label behaves the same in
both forms:

```zax
fill final : ()(a : Integer, b : Integer = 0, c : Integer, d : Integer) = {
}

myFirst := 1
myLast := 2
fill(myFirst, c: 3, myLast)     // myFirst fills a; b takes its default; then c, then myLast fills d
myFirst |> fill(c: 3, myLast)   // the same call
```

[Lambdas and callable composition](lambdas-and-callable-composition.md#chain-calls-immediately)
teaches chains in full.

A named argument expects one expression value, so it never receives a whole
sequence:

```zax
accept(true, number: produce())   // error: produce() has two mandatory results
```

### When two sources claim one parameter

Forwarding can supply the same parameter twice. That happens when two
sequences offer the same label, or when a label match meets an explicit label
for the same parameter. Again using `produce` and `accept` from
[the opening example](#forwarding-results):

```zax
accept(produce(), produce())         // error: number and text are each offered twice
accept(produce(), number: 7, true)   // error: number from produce() and from the label
```

A collision is handled by the ordinary [viability](#viability) rule: a
candidate whose inputs supply a parameter twice is not viable. If another
candidate is still viable, ordinary selection picks it. If none is, the
diagnostic shows the collision. A discardable result collides too, because
permission to go unused does not let it step aside.

In this example the collision rules out one overload, and selection falls
through to the other:

```zax
present final : ()(number : Integer, text : String, flag : Boolean) = {
}

present final : ()(
  caption : String,
  count : Integer,
  value : Integer,
  enabled : Boolean
) = {
}

produceOther final : (number : Integer, flag : Boolean)() = {
}

present(produce(), produceOther())
// the first present would receive number twice, so it is not viable;
// the second matches no labels and takes text, number, number, and flag by position
```

### Steering a forwarded sequence

A label match is not always what you want. When it isn't, indicate where the
results go instead:

```zax
split final : (head : String, tail : String)(text : String) = {
}

joinReversed final : (result : String)(tail : String, head : String) = {
}

myText := "left:right"

joined := joinReversed(split(myText))
// by label: head fills head, tail fills tail

swapped := joinReversed(split(myText) reshape {
  head: tail:
  tail: head:
})
// the reshape relabels first, so split's head fills tail

byPosition := joinReversed(: = split(myText))
// explicit positional mapping: no labels are matched
```

A postfix `reshape` relabels the invocation it follows before any of its results
route. It binds more tightly than `|>`, so in a chain it relabels just the stage
it is written on.

A reshape can also give a result the label a parameter expects. Here
`produce`, from [the opening example](#forwarding-results), offers `text`, but
the consumer wants `caption`:

```zax
consumeCaption final : ()(caption : String, number : Integer) = {
}

consumeCaption(produce() reshape text: caption:)   // text now binds caption
```

You could also make a result positional by relabeling it to a name that no
parameter uses. That keeps working only while no parameter has that name, so
the explicit positional form `: = producer()` is the better choice.

[Structural shapes and compatibility](structural-shapes-and-compatibility.md#naming-results-by-position-with-n)
defines reshape declarations and the `[N]` names for unlabeled results.

### A single result stays at its position

A call with only one result works differently. One result is one expression
value, so it binds where it is written, and the label its producer declared
never moves it. Trouble arises when that label names a different parameter that
is still unfilled. The label and the position then disagree, and either one
could be the mistake, so the call is an intent error rather than a guess:

```zax
getCount final : (count : Integer)() = {
}

report final : ()(total : Integer, count : Integer) = {
}

myTotal := 10
report(getCount(), myTotal)     // intent error: count would reach total by position
getCount() |> report(myTotal)   // the same intent error
report((getCount()), myTotal)   // still the intent error: parentheses do not change it
```

To fix it, say which one you meant. Any of these works:

```zax
report(myTotal, getCount())                         // reorder: position and label agree
report(count: getCount(), total: myTotal)           // label the call's inputs
report(: getCount(), myTotal)                       // by position, on purpose
report(getCount() reshape count: total:, myTotal)   // relabel to the intended parameter
report(myTotal, count: = getCount())                // an explicit routing group
report((: Integer = getCount()), myTotal)           // an anonymous typed destination drops the label
```

Alternatively, the producer or the consumer can mark the label
[incidental](#incidental-labels). It then never routes, and never causes this
error.

The check looks at source shape, not at whether the types would fit. It runs on
the candidate that selection has already chosen, so it never moves selection to
another overload, and it fires only when the label names a different parameter
of that candidate. A common label such as `value` will sometimes trigger it.
For example, `pad(getValue(), myWidth)` against
`pad final : ()(width : Integer, value : Integer)` triggers it, and there it
catches a real swap.

### Names you choose never route

Only the labels declared on result slots route, and only while their results
are being forwarded. Once a result is bound to a name, that name is the
caller's private choice:

```zax
report final : ()(total : Integer, count : Integer) = {
}

computeTotal final : (result : Integer)() = {
}

computeCount final : (result : Integer)() = {
}

total := computeTotal()
count := computeCount()

report(total, count)                 // positional
report(count, total)                 // positional, not an error: count fills total
report(count: count, total: total)   // by label, when wanted
report(count:, total:)               // error: empty labels omit both inputs, and report has no defaults
```

`report(count, total)` hands `count` to `total` without complaint. The variable
names are the caller's own, so the call is purely positional.

The last line looks like a by-name shorthand, but it is not. In an ordinary
call, a bare `count:` acknowledges an omitted input. Inside a
[routing group](#two-cursors) such as `text: = produce()`, the same spelling
selects the same-named result instead. Neither meaning passes a local variable
by name.

Renaming a local variable therefore never reroutes a call, and the same holds
for the item name in a
[`from each` mapping](variadics.md#transforming-each-item).

Destinations never match labels implicitly either. New names come from
selecting results by label or from typed declarations, and existing places
receive results in result order. Using `produce` from
[the opening example](#forwarding-results):

```zax
number:, text: = produce()                      // new same-named bindings, selected by label

existingText : String
existingNumber : Integer
existingText, existingNumber = produce()        // assignment in result order
```

`number, text := produce()` is an error. Each destination entry is read on its
own, so `number` would be an assignment into an existing place, and `text`
would take an inferred type from the second result by position alone.
[Declarations and bindings](declarations-and-bindings.md#multiple-results)
lists the complete destination forms.

Put together, every label-related error comes from one of three situations:

- two meaningful declared labels disagree, such as a result label and a
  parameter label;
- one parameter is supplied twice; or
- a [significant](#significant-labels) parameter receives a value that doesn't
  show the parameter's name.

A name the caller chose never routes, and outside a significant parameter it
never causes an error by itself. That is why
`report(getCount(), myTotal)` is an intent error while `report(count, total)`
is not.

### Explicit routing moves the cursor

A label match and an explicit label affect the cursor differently. A label
match uses up its parameter without anything written at a position, so later
positional inputs simply fill whatever remains. An explicit label, whether
`count: value` or a routing group `count: = producer()`, is a written
instruction, and it moves the cursor as in any call:

```zax
getCount final : (count : Integer)() = {
}

report final : ()(total : Integer, count : Integer) = {
}

myTotal := 10
report(count: = getCount(), myTotal)
// error: the cursor moves past count, so myTotal has no parameter

report(myTotal, count: = getCount())
// total by position, then count by label
```

### Incidental labels

Many labels mean nothing beyond "the answer": `result`, `output`, and similar
conventional names. Writing `[<label=incidental>]` after the name marks the
label as carrying no meaning, so it never routes implicitly:

```zax
addUp final : (result [<label=incidental>] : Integer)(left : Integer, right : Integer) = {
  return left + right
}

record final : ()(total : Integer, result : Integer) = {
}

myResult := 7
record(addUp(1, 2), myResult)      // addUp's result fills total by position
record(5, result: = addUp(1, 2))   // explicit selection still works
```

Without the marking, `addUp`'s `result` label would name `record`'s second
parameter while the call sits in the first position. That is the single-result
intent error.

The marking can go on either side of a call:

- An incidental **result** label never binds a parameter by label and never
  raises the single-result intent error.
- An incidental **parameter** label is never claimed by a producer's label. A
  single result whose label names an incidental parameter raises no intent
  error, because no two meaningful labels disagree. A
  [`>-` member](structural-shapes-and-compatibility.md#decompose-one-value-with--)
  with that name does not bind it either, so the parameter must be supplied
  another way.
- Explicit use still works in both cases: `name: value`, routing groups, and
  reshape labels. A reshape label is written intent, so it binds even an
  incidental parameter.

Here the marking is on the parameter side:

```zax
getTotal final : (total : Integer)() = {
}

getCount final : (count : Integer)() = {
}

store final : ()(count : Integer, total [<label=incidental>] : Integer) = {
}

myCount := 3
store(getTotal(), 5)
// no intent error, because total is incidental on store; the call is purely
// positional, so getTotal()'s result fills count and 5 fills total
store(getCount() reshape count: total:, myCount)    // the reshape label binds the incidental total
```

Most declarations whose result is labeled `result` need no marking, because the intent error
fires only when a label names a different parameter. Marking every label
incidental would throw away the routing that a meaningful label provides.

The directive appears only in the declaration; callers write nothing extra.
Like the label itself, the marking belongs to the visible prototype, as
[compatible visible prototypes](#compatible-visible-prototypes) explains. The
directive is listed in the
[compiler directive catalog](compiler-directives.md#directive-catalog).

A built-in operator's result label is incidental, as
[operator result shapes](#operator-result-shapes) shows.

### Significant labels

Some parameters are easy to fill with the wrong value, because position alone
doesn't show what the value means. Writing `[<label=significant>]` after a
parameter's name requires every caller to show that name with the value:

```zax
MyDocument :: type {
  caption : String
}

publish final : ()(body : String, caption [<label=significant>] : String) = {
}

getCaption final : (caption : String)() = {
}

myBody := "Hello"
caption := "Greeting"
myDocument : MyDocument

publish(myBody, caption)               // fine: the variable is named caption
publish(myBody, caption: "Greeting")   // fine: an explicit label
publish(myBody, myDocument.caption)    // fine: the member is named caption
publish(myBody, >- myDocument)         // fine: the decomposed member caption binds by name
publish(myBody, getCaption())          // fine: the result is labeled caption
publish(myBody, "Greeting")            // error: nothing names caption
publish(myBody, : "Greeting")          // error: positional intent does not override it
```

A matching name only confirms the value. It never routes it:

```zax
publish(caption, myBody)
// error: caption fills body by position, and myBody then reaches caption
// without naming it
```

So a significant parameter is the one place where a name the caller chose
matters. Even there, the name is checked, never used to move a value.

A value keeps its name only through built-in constructs whose meaning is fixed
and which pass the value along unchanged:

- a bare variable, or member access, which keeps the member's name;
- grouping, as in `(caption)`;
- a leading `:`, as in `: caption`, which neither adds nor removes a name;
- a stance restatement, such as `caption as copy`;
- a member decomposed with `>-`;
- an [argument declaration](#argument-declaration-expressions) with that name,
  such as `(caption : String = "Greeting")`.

A call result shows a name only through its own declared label, as
`getCaption()` does above. Any operator, conversion, or other call produces a
new value with no name, because the value has changed: neither `area * 2` nor
`sqrt(area)` is still the area.

```zax
publish(myBody, caption as copy)          // fine: the same value, restated
publish(myBody, : caption)                // fine: : does not change the name
publish(myBody, caption + "!")            // error: a new value; write caption: caption + "!"
publish(myBody, caption: caption + "!")   // fine
```

A conversion with `as` is a new value too, unlike a stance restatement that
uses the same keyword:

```zax
MyCaption :: type { }
// assume MyCaption declares a conversion to String, as shown in casting

publishFrom final : ()(caption : MyCaption) = {
  publish("Hello", caption as String)            // error: a conversion makes a new value
  publish("Hello", caption: caption as String)   // fine
}
```

Conversions are described in [casting](casting.md).

A result label marked incidental shows no name, because an incidental label
carries no meaning:

```zax
getText final : (caption [<label=incidental>] : String)() = {
}

publish(myBody, getText())            // error: getText's caption label is incidental
publish(myBody, caption: getText())   // fine
```

A significant pack is checked as a whole. Its items arrive together, so the pack
must be started by its name: an explicit label, a forwarded result pack with that
label, or `from` a source pack with that name. Supplying no items is fine,
because nothing reached the pack without its name:

```zax
publishAll final : ()(body : String, tags ... [<label=significant>] : String) = {
}

publishAll(myBody, tags: "draft", "urgent")   // fine: the label starts the pack
publishAll(myBody)                            // fine: the pack is empty
publishAll(myBody, "draft", "urgent")         // error: the pack is filled without naming tags

forwardTags final : ()(body : String, tags ... : String) = {
  publishAll(body, from tags as copy)         // fine: the source pack is named tags
}
```

A label with no marking is **descriptive**: it describes the value, binds a
forwarded result with the same label, and otherwise takes positional inputs.
`[<label=descriptive>]` states that default explicitly. Every label has one of
three markings:

| Marking | Behavior |
| --- | --- |
| None, or `[<label=descriptive>]` | The default. A forwarded result with the same label binds it; otherwise it takes positional inputs |
| `[<label=incidental>]` | Never routes implicitly; explicit use still works |
| `[<label=significant>]` | Every value supplied to it must show its name |

`significant` can mark parameters only. Like the other markings, it belongs to
the visible prototype.

### Leftover results

A producer can offer more results than the consumer takes. Any forwarded result
that no parameter consumes must be discardable:

```zax
outline final : (width : Integer, height # : Integer)(shape : MyShape) = {
}

area final : (result : Integer)(width : Integer) = {
}

myShape : MyShape
myArea := area(outline(myShape))              // width by label; the discardable height goes unused
myArea2 := myShape |> outline() |> area()     // the same call
```

Pack items are stricter. An item carried into a `|>` stage by
`from each item : in items |> stages` has no discard marking, so if that stage
does not consume it, the call is always an error. Items of a
[result pack](#result-packs) follow whatever discardability the pack declares.

### Forwarding from a return

`return produce()` forwards a complete result sequence by the same rule.
Labeled results fill the result slots with the same labels first, and the rest
fill the remaining slots in order. Here `wrapper` reuses `produce` from
[the opening example](#forwarding-results):

```zax
wrapper final : (
  number : Integer,
  text : String
)() = {
  return produce()   // produce's text and number fill the same-labeled slots
}
```

Return routing can remap labels:

```zax
return text: outputText:, number: outputNumber: = produce()
```

A return can also label the values it supplies, the way a call labels its
inputs. Each label fills the result slot it names, and later unlabeled values
fill the next open slots after it:

```zax
describeAnswer final : (text : String, number : Integer)() = {
  return number: 42, text: "answer"   // each value goes to the slot with its label
}

describeAgain final : (text : String, number : Integer)() = {
  return text: "answer", 42           // 42 fills number, the next open slot
}
```

The same cursor rules as a call apply. A label on a
[result pack](#result-packs) starts that pack, which keeps absorbing values
until the next label:

```zax
regroup$ final : (Lefts ... : type, Rights ... : type)(
  rights ... : Rights,
  lefts ... : Lefts
)(
  leftInputs ... : Lefts,
  rightInputs ... : Rights
) = {
  return lefts: from leftInputs as last, rights: from rightInputs as last
}
```

A labeled value, `label: value`, is not a routing group. A routing group pairs a
source label with a destination label and ends in `= producer()`.

Parentheses change what a return forwards. A parenthesized call must be one
expression value:

```zax
return (produce())
```

Here the parentheses ask for a single value, so the multi-result sequence is
not forwarded. This works only if `produce()` can form one usable value. To
forward every result, leave the parentheses off.

It can also be tempting to wrap a long routing group in parentheses to spread it
across lines. Result routing cannot be grouped as an expression, so this is an
error:

```zax
return (
  text: outputText:,
  number: outputNumber: = produce()
)
// error: result routing cannot be grouped as an expression
```

To lay the group out over several lines, write the routing directly after
`return`, without parentheses, and continue it onto the next lines with
explicit continuation.

### Forwarding into construction

A construction packet may forward several results as constructor inputs:

```zax
pair : Pair = [{
  produceTwo()
}]
```

This selects a compatible `Pair` constructor, and the results route into that
constructor's parameters by the
[same rule](#how-forwarded-results-find-their-parameters) as in a call. Labels
that name constructor parameters bind first, and the rest fill positions. The
packet does not match result labels against `Pair`'s stored members, and it
does not create one anonymous structural value.

Use explicit `-<` recomposition when the remaining results should become one
structural value; see
[Zax structural shapes and compatibility](structural-shapes-and-compatibility.md#recompose-several-results-with--).

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

If `parseValue` were instead one generic, `parseValue$`, the same boundary
would fill its compile-time parameter, which the arguments do not:

```zax
parseValue$ final : (ResultType : type = Integer)(result : ResultType)(
  text : String readonly &
) = {
}

count : U16 = parseValue(input)   // ResultType is U16
other := parseValue(input)        // no declared type: ResultType defaults to Integer
```

See [supplying compile-time arguments](generics.md#supplying-compile-time-arguments).

A typed declaration can choose the final type of arithmetic performed only on
number literals:

```zax
myByte : U8 = 200 + 55
myBits : U8 = ~1 // error: complement has no width-invariant result
```

The first declaration checks the final mathematical result against `U8`. The
second still fails because unknown positive complement has different signed and
unsigned results; the surrounding declaration cannot reach inside and choose
one interpretation.

See [Zax integer literals and realization](integer-literals.md).

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
[operators](operators.md#boolean-operations-and-fallback).

### Expected results never choose a source structure

Expected-result context chooses an implementation *within* one already formed
expression tree. It never chooses between distinct operator-phrase extents or
attachment trees:

```zax
result : Desired = source phrase middle phrase tail
// error if two phrase trees remain viable, even when only one returns Desired
```

The allowed direction is `selected inner result -> enclosing receiver selection`,
never `enclosing receiver requirement -> manufacture an inner result choice`. An
inner result-only ambiguity therefore cannot be resolved by an enclosing
operation; the programmer states the intended boundary with a complete typed
declaration. See
[operators](operators.md#candidate-tree-formation-and-selection) for the shared
rule and [operator phrases](operator-phrases.md#results-flow-outward-never-inward)
for its phrase-specific use.

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
same-family candidates subject to direct `copy`, `move`, reference, and qualification
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

Final uncommitted integer arm results may realize through the conditional's one
concrete convergence hole. That exception does not propagate through an arm to
discover an inner receiver; see
[integer conditional convergence](integer-literals.md#conditional-convergence).

A path may also select a direct mixfix while another path uses an ordinary
operation:

```zax
result := (condition ?? a * b ;; fallback) + c
```

The true path may select a `*`, `+` mixfix; the false path may select binary `+`.
Common operands retain once-only source evaluation, every path is validated, and
the complete result must converge. Mixfix structural matching, ambiguity, and
decomposition are defined by
[mixfix operators](mixfix-operators.md#branch-specific-mixfix-selection).

An operator node under a conditional expression may also be classified
differently on different paths, so one `&&` or `||` may be a protected
short-circuit operation on one path and an ordinary eager overload on another.
Every path is still validated at compile time. That per-path operator-node
behavior is owned by
[operators](operators.md#branch-specific-selection) and is not restated
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
- operators use the protected and receiver-owned domains defined by
  [Zax operators](operators.md#discovery).

A non-generic `name` and a generic `name$` form one set of candidates. A call
without `$` considers both; a call with `$(...)` considers only generic
candidates. See
[generic and non-generic candidates](generics.md#generic-and-non-generic-candidates).

### Viability

A candidate is viable only when:

- every explicit input, omission, and forwarded result maps without
  duplication, so a [collision](#when-two-sources-claim-one-parameter) makes
  that candidate not viable;
- every required parameter receives a value or declared default;
- arguments and receiver operands can bind with required types, qualifications,
  and transfer behavior;
- explicit source-result labels exist;
- an applicable expected-result shape is compatible;
- every [constraint](generics.md#constraints) on a generic candidate is `true`;
- and every required result receives a valid disposition.

A `false` constraint makes a candidate not viable, exactly as a type mismatch
does. An error inside a generic body never makes a candidate not viable.

Private declarations outside the caller's permitted context are ineligible
before preference. Availability of an eligible visible candidate is checked
after preference so a uniquely best bodyless, forbidden, or unsupported
generated declaration can report unavailable-best rather than falling through.

A number literal can directly supply a parameter declared with an integer type:

```zax
consumeByte final : ()(value : U8) = {
}

consumeByte(55)
```

The compiler checks `55` against `U8`. It does not try conversions or invent
nested temporary values merely to make another overload viable.

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
not declare every pair comparable.

Transfer fallback compares accepted stance:

- `copy` accepts `copy`;
- `deep` accepts only `deep`;
- `move` prefers `move`, then falls back to `copy`;
- `last` prefers `last`, then falls back to `move` and `copy`.

Value/reference binding, exact qualification, type conversion, receiver stance,
and applicable result quality remain independent dimensions. Better stance does
not automatically defeat worse ownership or qualification behavior.

An exact mutable/writable `move` or `last` reference is better than ordinary
copy-in processing when the source is mutable/writable and every other comparison
is no worse. A readonly reference versus by value, or exact `last` by value
versus fallback `move` by reference, may have no clear overall winner and remain
incomparable.

Receiver stance is another slot:

- exact receiver stance admits only that stance;
- unstanced receiver declaration is fallback for every stance;
- exact receiver stance beats fallback when other comparisons permit dominance;
- receiver matching does not use the source stance fallback ladder.

Generic candidates fit this structure. Only between otherwise equally good
matches does a non-generic candidate beat a generic one, and one generic
declaration beats another only when its constraint terms are a strict superset;
see [specializations](generics.md#specializations).

The remaining complete type, qualification, indirection, and
generated-candidate preference table is future callable-selection work. Its
eventual rules must fit this partial-order structure.

### Demand-time ambiguity

Distinct declarations remain legal even when no natural call can currently
select one without ambiguity. The compiler does not need a global
declaration-time proof that every callable is naturally selectable.

At a use:

- no viable candidate is an error;
- several nondominated viable candidates are ambiguous;
- an eligible visible declaration may be uniquely best but unavailable to
  invoke, for example because it is bodyless, forbidden, or requires generated
  behavior the language cannot supply;
- that unavailable-best case is an error;
- and selection does not fall through to a weaker available declaration.

A private declaration outside its permitted context is ineligible before this
comparison. It cannot become the unavailable best candidate or block a visible
public declaration.

A resolved function value or exact typed function alias may select one complete
prototype before invocation. A fully typed function value selects one generic
instantiation the same way; see [naming a generic](generics.md#naming-a-generic).
It selects one processing of a variadic function the same way, as
[a function value for a variadic function](#a-function-value-for-a-variadic-function)
shows.
Complete exact-prototype syntax remains later callable-selection work.

### Terminal opportunity requires intent

**Terminal** means that analysis can prove this is the source's final ordinary
use before its existing lifetime reaches destruction. It is an opportunity
because a `last` consumer could take ownership of resources that would otherwise
be released soon.

Invocation never silently upgrades implicit `copy` or explicit `move` to
`last`. When analysis proves that an implicit source use is terminal and
offering `last` could materially change the accepted contract, the call is an
[intent error](intent-acknowledgements.md):

```zax
forward(resource)
// error: state copy, move, last, or deep intent explicitly
```

The programmer states the stable choice:

```zax
forward(resource as last)
forward(resource as move)
forward(resource as copy)
forward(resource as deep)
```

No intent error is needed when `last` would accept the same contract and produce
the same source post-state. For example, a copy-only function has no
terminal-aware alternative:

```zax
inspect final : ()(
  value : Record readonly & copy
) = {
}

inspect(record)
```

Decorating this call would not change the selected function or source state. An
explicit declaration stance is not upgraded.

Branches may contain separate path-terminal uses. Loop analysis may recognize
more cases over time. Improved analysis produces new intent errors rather than
silently selecting another user body. Destruction does not count as ordinary
later use.

If a number literal fits several overloads, ordinary source prefers an
`Integer` parameter; source with unsigned intent prefers a `UInteger`
parameter. The compiler chooses that overload before checking the value's
range, and a range failure does not retry another overload. Complete examples
and failures are defined by
[Zax integer literals and realization](integer-literals.md#selection-and-range-order).

### Fewer defaults

After supplied receiver and input bindings compare equally, a candidate requiring
fewer omitted callable-parameter defaults is better.

This comparison counts only parameter defaults activated by omission. It does not
count nested constructor, conversion, member, result, or explicit type-default
work.

Explicit and implicit omission count equally:

```zax
configure(value)
configure(value, option:)
```

when both forms activate the same callable default.

An empty variadic pack also counts as one omitted default.
[Candidates with a variadic pack](#candidates-with-a-variadic-pack) shows how
that decides between overloads.

### Candidates with a variadic pack

A variadic overload often sits beside fixed ones, such as a general `largest`
next to a two-argument version. When the candidates otherwise match equally
well, three rules decide between them.

First, an argument bound to an ordinary parameter beats one absorbed into a
pack. A fixed declaration accepts a strict subset of the call shapes the pack
accepts, so it is the more specific choice:

```zax
largest final : (result : Integer)(first : Integer, second : Integer) = {
}

largest final : (result : Integer)(values ... : Integer) = {
}

myLargest := largest(1, 2)   // the two-parameter largest wins
```

Second, an empty pack counts as one omitted default, so a candidate that needs
no pack wins:

```zax
show$ final : (First : type)()(first : First) = {
}

show$ final : (First : type, Rest ... : type)()(first : First, rest ... : Rest) = {
}

show(1)   // the first show wins: the second needs an empty pack
```

Third, a pack with a stated element type and no `$` list makes a non-generic
candidate, even though the function is processed once per argument count. That
candidate beats an equally good generic candidate. A call written with `$(...)`
considers only generic candidates, so it never selects this one:

```zax
sum final : (result : Integer)(values ... : Integer) = {
}

sum$ final : (Items ... : isNumeric)(result : Integer)(items ... : Items) = {
}

myTotal := sum(1, 2)   // the sum without $ wins
```

[Variadics](variadics.md#choosing-between-overloads) has more examples.

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

### Minted implementation model

A callable declaration with a body is checked and **minted** against its own
implementation prototype:

```text
implementation prototype + body
-> establish body-entry facts
-> select operations inside the body
-> establish internal parameter/result behavior
-> mint one implementation
```

A compatible visible prototype reuses that minted implementation. It does not
reprocess the body, select different internal overloads, change internal
scheduled assignment into open-ended assignment, or remint the callable.

A generic differs. Its body is processed once for each distinct set of
compile-time inputs, including each qualification of a worked-out type, and each
resulting instantiation is its own minted implementation; see
[generics](generics.md#when-code-is-processed-once-per-input). A variadic
function is processed the same way, and
[a function value for a variadic function](#a-function-value-for-a-variadic-function)
explains what that means for function values.

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

Relabeling has one more effect. Because forwarded results
[bind by label](#how-forwarded-results-find-their-parameters), new labels also
change how results route for callers of the new prototype. Callers always route
by the labels and [incidental](#incidental-labels) markings of the prototype
they call through. A compatible implementation installed in a slot may use
different labels or markings, but installing it never changes routing at a call
site:

```zax
MyHandler :: type {
  onResize : ()(width : Integer, height : Integer)   // the slot's visible prototype
}

resizeImpl final : ()(w : Integer, h : Integer) = {
}

measure final : (width : Integer, height : Integer)(shape : MyShape) = {
}

handler : MyHandler
handler.onResize = resizeImpl        // compatible: same types and order; labels differ

myShape : MyShape
handler.onResize(measure(myShape))   // routes by the slot's labels, width and height
```

`resizeImpl` calls its parameters `w` and `h`, but the call goes through the
slot, so `measure`'s results route by the slot's `width` and `height`.

Compatibility is positional over ordered input and result slots. A visible
prototype supplies a call-boundary contract around the unchanged implementation:

```text
visible inputs/defaults
-> satisfy the minted body's entry requirements
-> invoke the minted body unchanged
-> map its produced results
-> satisfy the visible caller guarantees
```

The new prototype may:

- relabel compatible slots;
- replace call-boundary defaults; and
- change caller result-acknowledgement policy;
- change outward result transfer stance when the minted result can satisfy that
  mapping; and
- expose a scheduled raw result as open-ended, or an open-ended minted result as
  scheduled, while preserving the minted body's internal cleanup behavior.

Relabeling may explicitly rename a corresponding exceptional outcome. The
visible prototype must otherwise preserve each result's
ordinary-versus-exceptional category, the number of exceptional outcomes, their
payload contracts, and body-entry constructedness. It cannot merge several
outcomes under one non-polymorphic result slot.

It may not:

- reorder values;
- convert incompatible values;
- synthesize missing values; or
- enter the body with an unconstructed result when the implementation requires a
  constructed one;
- provide a possibly vacant allocated result to an implementation minted
  under a definitely present result;
- promise callers a definitely present result when the minted implementation
  may produce vacancy; or
- add, remove, discard, or change the category of an exceptional outcome; or
- change operations already selected inside the body.

For allocated raw results, presence and outward cleanup are separate:

```zax
implementation final : (
  result : MyValue * last = @
)() = {
  // Minted scheduled-result behavior.
}

manualResult final : (
  result : MyValue * last = @<
)() = implementation
```

The alias is compatible: both contracts provide and produce a definitely present
pointer. The minted body retains its scheduled replacement behavior, while the
visible boundary leaves final cleanup to `manualResult`'s caller.

`@` and `@!` are incompatible in either direction:

```zax
maybeEntry final : (
  result : MyValue * last = @!
)() = implementation
// error: implementation requires presence on entry
```

Conversely, an implementation minted with `@!` may return vacancy, so an alias
using `@` would overpromise presence to callers.

Changing outward stance does not alter internal stance-driven behavior already
minted into the body. The boundary change is legal only when the produced result
can satisfy it. In particular, exposing a scheduled raw result as explicit
`copy` retains producer cleanup and therefore supports only lifetime-safe
borrowed uses unless a caller restates `last` or proves another owner.

If entering the original body requires executable adaptation, write a wrapper or
lambda. The one exception is the compiler-generated glue that binds a fixed
prototype to one processing of a
[variadic function](#a-function-value-for-a-variadic-function).

Composition `via` is different from an ordinary compatible-prototype alias. The
declaration explicitly names a contained target, so its thin adapter may adjust
the receiver to that member and perform the narrowly permitted input mapping:

```zax
start final : ()() = via engine.start
```

Singular `via` must select one source operation and one complete mechanical
mapping. If permitted adaptation leaves several overloads viable, the
declaration is ambiguous rather than choosing by source order or implementation
preference. An exact private helper can make that choice explicit:

```zax
startExact private final : ()(fuel : Fuel readonly &) =
  via engine.start

start final : ()(tank : Tank readonly &) =
  via startExact
```

The helper's visible prototype anchors one overload, then the public declaration
targets that exact helper. This is legal for the same reason a public body may
call an accessible private helper: `start` is an explicit new API declaration,
not publication of `startExact`. It does not let data `via`, automatic `expose`,
or `own` publish a private stored path.

A route that maps a contained reference or pointer result outward must also make
its provenance mechanism explicit:

```zax
// The selected contract proves this result origin.
knownOwner final : (result : Car &)() =
  via engine.selectedPeer

intent<redundant-outer-tracking>{
  possibleOwner final : (result : Car & ?)() =
    tracked via engine.selectedPeer
}
```

Ordinary `via` uses selected-contract proof for the non-optional outer result.
If the programmer deliberately preserves the optional tracked contract despite
that proof, the complete declaration requires
`intent<redundant-outer-tracking>`.

Without selected-contract proof, the checked and asserted forms instead remain:

```zax
possibleOwner final : (result : Car & ?)() =
  tracked via engine.selectedPeer

assertedOwner final : (result : Car &)() =
  unsafe via engine.selectedPeer
```

`tracked via` selects the checked optional mapping and requires the applicable
`outer tracked` capability. `unsafe via` is the programmer's unchecked
non-optional provenance assertion. Compiler-private proof cannot require the
intent acknowledgement or change either portable form. `= existing` cannot
silently discover a mapping that needs tracked outer-result work. Complete
redundancy and cost behavior belongs to
[Zax composition](composition.md#deliberately-retaining-redundant-tracking).

Family forms make a stronger declaration. Here `Engine` supplies two `reset`
overloads:

```zax
Engine :: type {
  reset final : ()() = {
  }

  reset final : ()(profile : Profile readonly &) = {
  }
}

SingleControls :: type {
  engine expose : Engine

  reset final : ()() = existing
}

Controls :: type {
  engine expose : Engine

  reset final : ()() = existing family
}

RenamedControls :: type {
  engine : Engine

  restart final : ()() = via family engine.reset
}
```

`= existing` adopts one uniquely discovered operation. `= existing family`
discovers one whole callable family, while `= via family` explicitly names and
may rename one. The written prototype is an exact family anchor, not a wildcard
or a conversion policy. Every eligible overload must independently obtain one
mechanical outer mapping. Adding an overload that has no unique mapping makes
the outer family declaration fail; the new overload is never silently omitted.

`= forbidden family` is an outer-name fence. It blocks the complete visible
callable name regardless of whether a signature would be direct, exposed, or
adopted from another source family. A later direct declaration with that name
conflicts rather than bypassing the fence.

An ordinary `= implementation` alias does not silently gain those composition
steps. Complete `via` eligibility is defined by
[Zax composition](composition.md#via-selects-one-target).

When the implementation is statically known and the new contract adds no runtime
state, a compiler may direct-call without materializing another function-value
slot. `final` alone is not a universal no-storage guarantee. A closure or
capturing default may still require representation.

This is a programmer-visible compatibility and cost model, not a promise of one
backend mapping or byte-for-byte generated code.

### A function value for a variadic function

Suppose you want to store `sum` in a variable or pass it as a callback. A
variadic function is processed once per argument count, or once per type list
when its element types vary, so there is no single function value to take. A
fully typed function value picks one processing by mapping its own parameters
onto the pack:

```zax
sum final : (result : Integer)(values ... : Integer) = {
}

pickSum : (result : Integer)(first : Integer, second : Integer) = sum
myTotal := pickSum(1, 2)   // calls sum's two-item processing
```

`pickSum` has two parameters, so it binds the processing of `sum` that takes
two items. Its labels `first` and `second` differ from `sum`'s parameter label
`values`, and that difference is handled by the ordinary compatible-prototype adaptation described
above.

Binding may also need compiler-generated adaptation, such as glue between that
processing's calling convention and the fixed prototype. The assignment is
supported either way. Any glue is a cost, and tools should be able to show it.

[Variadics](variadics.md#lambdas-and-function-values) covers variadic lambdas
and `varying` callback slots.

## Temporary lifetime and call completion

The synchronous completion boundary includes:

1. callee and receiver evaluation;
2. explicit input evaluation and immediate binding;
3. omitted-default evaluation and binding;
4. opted-in result initialization;
5. body execution;
6. production of one complete success or exceptional outcome; and
7. mapping of only that selected outcome into the surrounding context.

A temporary bound to a reference parameter survives through that boundary:

```zax
inspect(makeBuffer())
```

If `inspect` receives `Buffer readonly &`, the temporary remains live throughout
`inspect`.

A directly constructed compiler-managed unnamed by-value temporary has
structurally known destruction after its one complete consumer. It inherently
offers `last`:

```zax
result := (: Source).get()
```

If `get` has an exact `last` receiver variant, that variant may be selected.
This is not analysis-dependent upgrading of a named value. A named receiver uses
its declaration stance, and a programmer can restate another stance on the
temporary explicitly.

This does not permit a reference into the temporary to escape:

```zax
view : Buffer readonly & = returnView(makeBuffer())
// error: the reference would outlive the temporary target
```

The reference result retains the temporary's origin, so this stored escape is an
error. A different owning result may explicitly preserve a dynamic life path;
unsafe source cannot make a temporary survive after its path ends.

For nested calls:

```zax
consume(produce())
```

the producer's result slots survive until their transfers or bindings into
`consume` complete. A temporary result bound to an outer reference parameter
survives through the outer call.

### Result slots, destinations, and elision

Result construction before mapping has two owners. The selected visible
prototype determines which result slots are initialized before body entry and
runs those initializers in result declaration order. This includes allocation
forms such as `@`; the caller establishes the call boundary from that prototype.
The implementation controls explicit construction in the body, and
value-bearing return expressions evaluate left to right. A rebound body and
visible prototype must agree on which result slots are already constructed at
body entry. Selecting results in another order does not rewrite any of these
construction effects.

Exceptional result slots are the narrow exception to prototype-controlled
pre-body initialization: they accept no initializer and may be constructed only
when producer `except` or forwarding selects them.

#### Conditional destination publication

Elision may reserve an outer destination's storage through several nested calls
without publishing an instance there. The destination binding and lifetime
become caller-visible only when the selected success or exceptional outcome
contains that result.

If another outcome is selected:

- raw reserved storage needs no destruction;
- a body-constructed nonselected provisional result is destroyed by the callee;
  and
- a prototype-preinitialized nonselected ordinary result is destroyed by
  caller-side invocation machinery.

Successful inner completion transfers provisional cleanup responsibility
outward. An enclosing function that later selects another outcome destroys the
live provisional value even when it occupies its caller's reserved storage.
Complete examples are in
[Zax exceptional result flow](except.md#conditional-result-elision).

#### Caller-visible destination order

Distinct source result slots complete after mapping in reverse result declaration
order. New caller-owned destinations follow the visible order in which the
caller introduces them:

```zax
resultA: firstA:,
resultB: secondB: = makeResults()
// Later destroy secondB, then firstA.

resultB: firstB:,
resultA: secondA: = makeResults()
// Later destroy secondA, then firstB.
```

`makeResults` uses the same prototype-defined pre-body construction and the same
implementation-defined body or return construction in both cases. The caller
has changed only which destination it introduces first, so it has visibly
changed the destinations' later destruction order.

#### Existing destinations keep their order

Routing into an existing destination establishes no new lifetime:

```zax
resultB: existingB,
resultA: existingA = makeResults()
```

`existingA` and `existingB` retain the destruction positions established by
their owning scopes. Likewise, call parameters are destroyed in reverse
parameter declaration order, and result destinations declared by an enclosing
function follow that function's result declaration order. Call-site
source-selection order does not replace either prototype order.

#### Elision moves destruction

Elision may make one source result slot and its destination the same value. The
unified value then has the destination's lifetime and destruction position; it
does not also undergo a separate source-slot destruction. This can change the
overall destruction order.

For example, suppose `makeResults` declares `resultA` before `resultB`. A wrapper
can discard the first result and construct its own empty result slot from the
second with a `.=` group:

```zax
keepSecond final : (
  kept : ResultB
)() = {
  #,
  resultB: kept .= makeResults()
}
```

When `resultB` is elided into `kept`, `resultA` remains a distinct source slot
and is destroyed when the inner mapping completes. The unified `resultB`
survives as `kept` and is destroyed later at the outer destination's position.
The resulting order is `resultA` before `resultB`, rather than the reverse
declaration order that two distinct source slots would have followed.

```text
producer result declarations:  resultA, resultB
elided mapping:                 resultB -> outer destination
inner mapping completion:      destroy resultA
outer destination completion:  destroy unified resultB
```

Elision does not reorder prototype-defined pre-body initialization or the
implementation's explicitly sequenced body and return construction of `resultA`
and `resultB`. It changes lifetime and destruction by removing the separate
intermediate identity and making the destination the produced value. An
implementation may perform this storage unification only with those language
effects.

Transfer stance does not change when ordinary argument or result temporaries
reach destruction. It changes which resources remain owned by those values at
that point. Each temporary is destroyed once at its ordinary completion
boundary.

After body-local scope exit and complete selected-outcome mapping:

1. distinct source result slots are destroyed in reverse result declaration
   order;
2. parameter instances are destroyed in reverse parameter declaration order;
3. caller-side receiver, argument, and nested-result temporaries are destroyed
   in reverse construction order when their complete consumer no longer needs
   them.

An elided result is absent from the first step because its lifetime has become
the destination lifetime. A `#` result remains in that step unless its
surrounding construct explicitly establishes another anonymous destination
lifetime.

This keeps parameters and referenced caller temporaries alive through result
mapping. Returned-reference origin and escape are defined by
[lifetimes and references](lifetimes-and-references.md#returned-references);
pointer ownership is defined by [pointers and arenas](pointers-and-arenas.md).

Async suspension requires a broader completion and lifetime model. It is not
ordinary synchronous completion.

## Multiline return lists

A comma continues an already established comma-list across the following
physical newline:

```zax
return number: first:,
  text: second: = produce()
```

When the first result begins after an otherwise complete bare `return`, explicit
continuation is necessary:

```zax
return \
  number: first:,
  text: second: = produce()
```

The comma already continues the next newline. Adding `\` there is an intent
error:

```zax
return \
  number: first:, \ // error: comma already continues this newline
  text: second: = produce()
```

A continuation-only line can visibly carry the list across another newline:

```zax
return \
  number: first:,
  \
  text: second: = produce()
```

Complete continuation and indentation behavior is defined by
[Zax source structure](source-structure.md).

## Diagnostics

Invocation diagnostics should distinguish:

- no callable found;
- invocation of a provably unavailable function value;
- a type-qualified call to a non-`once` function;
- `?_` in a non-`once` bound function;
- a receiverless type call whose synthesized `copy` receiver cannot select the
  declaration;
- trapping or unprepared Nothing access through `_`;
- unknown parameter or result labels;
- label-versus-declaration intent errors;
- duplicate parameter, source-result, or destination mapping;
- a collision in which two forwarded results, or a forwarded result and an
  explicit label, supply one parameter, shown for each candidate it made not
  viable;
- a single result whose declared label names a different unfilled parameter;
- a value supplied to a significant parameter without showing its name,
  including a computed value, a conversion, an incidental result label, or a
  significant pack filled without its label;
- a result-pack label that names an ordinary parameter rather than a pack;
- an unconsumed forwarded result that is not discardable, including an
  unconsumed pack item;
- a runtime-length source expanded with `from` into inputs;
- exhausted or invalid positional cursors;
- omitted parameters without declared defaults;
- declared-default omission versus explicit type-default expressions;
- argument, receiver, result, qualification, or transfer mismatch;
- zero results in a value-required context;
- several mandatory results forced into one expression;
- parenthesized expression mode where mapping was intended;
- unconsumed required results;
- an unhandled or multiply handled exceptional outcome;
- exceptional result initialization before producer `except` or forwarding;
- exceptional outcome discard through `#`;
- a handler destination invalid for one nested source specialization;
- incomplete or duplicate result construction;
- a `.=` routing group used in call routing, `using`, capture, or `return`
  routing, where no existing place can receive `.=`;
- an existing destination in an `=` group that has not been constructed yet,
  where a `.=` group would construct it;
- a `return #` slot whose type cannot be default-constructed;
- result context without a complete declaration;
- branch-dependent conditional-expression arms that select incompatible callables
  or fail to converge;
- incomparable undominated candidates;
- a uniquely best but unavailable candidate;
- use after an earlier `move` or `last`;
- a reference escaping a temporary;
- redundant explicit continuation; and
- contradictory continuation indentation.

When candidates map source differently, diagnostics should show the
candidate-specific input and result mapping rather than only list signatures,
including which forwarded results bound by label and which took positions.

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
- compiler-generated glue that binds a fixed function value to one processing
  of a variadic function;
- each processing of a variadic function, one per distinct argument count or
  type list used;
- type-qualified `once` calls without receiver evaluation;
- instance-qualified `once` calls with ordinary once-only receiver evaluation;
- environments retained for closures or default expressions;
- result construction, remapping, omission, and discard;
- selection and mapping of success versus exceptional completion shapes;
- branch-specialized handler mapping for nested equal-label outcomes;
- destruction of nonselected provisional results;
- conditional publication of elided destinations;
- source-slot completion, caller-owned destination order, and any lifetime or
  destruction-order change caused by result elision;
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
- source-result selectors before destination selectors;
- bare positional destinations, anonymous typed destinations, and `#` discard;
- `using` resource-list mapping boundaries;
- explicit catch, forwarding, and outcome-reshape clauses;
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
- renaming a parameter or result label can also reroute forwarded results: a
  result that matched a parameter by label may fall to a position, or start
  matching a different parameter;
- adding a labeled result to a producer can make it collide with an explicit
  label or another forwarded result at existing call sites, making that
  candidate not viable;
- adding or removing `[<label=incidental>]` on a visible prototype changes
  routing and single-result intent errors at call sites written against that
  prototype, without changing its types;
- marking a parameter `[<label=significant>]` makes existing calls that supply
  it without its name into errors, while removing the marking never breaks a
  call; renaming a significant parameter breaks every caller that showed the
  old name;
- reordering parameters or results changes positional cursors;
- changing result arity or discardability changes mappings;
- adding, removing, renaming, or reclassifying an exceptional outcome changes
  every affected call-site disposition;
- replacing a routed bare positional destination with `#` leaves that
  destination unsupplied;
- reordering explicit destination introductions changes their later destruction
  order without changing prototype- or implementation-defined result
  construction;
- enabling or preventing result elision may move destruction from a source-slot
  completion boundary to the destination lifetime;
- adding a result-only overload may make inferred calls ambiguous;
- adding or changing an overload may change a conditional-expression arm's
  selected callable;
- changing a result type's default-constructibility changes `return #` validity;
- changing transfer or qualification requirements changes viability, cost, and
  post-call source availability;
- adding or removing parentheses can change expression mode versus mapping; and
- adding or removing `once` changes whether a type-qualified call is available
  and whether one declaration is shared across instances.

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
- complete ordinary-success or one-exceptional-outcome completion;
- the distinction between one expression and a result sequence;
- one routing rule wherever forwarded results meet inputs, in which a name the
  caller chose never routes;
- explicit result mapping and deterministic cursors;
- visible-prototype-controlled pre-body result construction,
  implementation-controlled body and return construction, caller-visible
  destination ordering, conditional destination publication, and
  elision-driven lifetime unification;
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

Global and `once` value initialization, concurrent first use, teardown, capture,
and generic-specialization behavior remain future work. They must preserve the
type/instance call distinction established here.

The following remain explicit future work:

- the detailed precedence table for types, qualifications, indirection, `copy`,
  `move`, reference, pointer, and `last`;
- complete closure layout, raw function-pointer ABI, and mutually recursive
  callable-family forwarding;
- partial application, composition, and split/combine transforms;
- complete ownership, moved-from state, returned aliases, and destruction order;
- async, cancellation, executors, and concurrency;
- runtime value-polymorphic predicates and hidden `mutator` access;
- parameter and result labels that are anonymous or private to a declaration,
  as distinct from a label that carries no routing meaning, which can already
  be marked [incidental](#incidental-labels);
- [significant](#significant-labels) result labels, which would require a
  result to be consumed by its label, for when a producer needs them;
- label aliases and complete reflection metadata;
- formal grammar, diagnostic identifiers, ABI, and conformance; and
- compiler lowering and representation.
