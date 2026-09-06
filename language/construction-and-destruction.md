# Zax construction, replacement, and destruction

| Field | Value |
| --- | --- |
| Status | Current conceptual design |
| Audience | Human developers reading, writing, or evaluating Zax |
| Applies To | Programmer-facing value construction, reconstructive replacement, and destruction; not a formal grammar or specification |
| Implementation State | Not established by this repository |
| Owns | Ordinary constructors and destructors; contextual/explicit constructor participation; automatic and explicit member lifecycle operations; construction packets; lifecycle declaration states; qualifier-complete generated `copy` families; generated assignment result; immutable reconstructive replacement; replacement constructors, resource retention, and results; construction/destruction authority; optional construction and complete-wrapper replacement integration at the shared lifecycle depth; automatic local, body, and flow-header lifetime ending and destruction order across normal and abrupt scope exits; the programmer-visible obligation to prove a live value before access through conditionally live storage; manual and delayed construction boundaries; lifecycle costs, diagnostics, and formatting |
| Does Not Own | Complete transfer meaning and fallback ([transfer stances](transfer-stances.md)); complete [optional behavior](optional-values.md); integer realization and numeric-source candidate behavior ([integer literals and realization](integer-literals.md)); declaration/qualifier behavior ([declarations and bindings](declarations-and-bindings.md), [qualifiers](qualifiers.md)); shared invocation selection ([function invocation](function-invocation.md)); flow-transfer/post-operation behavior ([core flow control](core-flow-control.md)); [reference lifetime](lifetimes-and-references.md); or general [safety contracts](safety-and-analysis.md) |

## Mental model

Zax distinguishes three related states:

1. **Storage exists.** A suitably sized and aligned place is available.
2. **Member lifetimes exist.** Individual contained places may or may not
   currently contain live values.
3. **The complete enclosing lifetime exists.** Ordinary code may use the
   complete instance after construction finishes and before replacement or
   destruction begins.

Each instance lives within a
[life path](lifetimes-and-references.md#start-with-a-life-path). An
[instance place](lifetimes-and-references.md#life-path-instance-place-and-resident-instance)
is the stable typed position through which its complete resident instance is
reached. This document defines how lifecycle operations establish and end those
instances; the lifetime owner defines how references remain valid across those
events.

Allocation may obtain storage, but allocation alone does not construct a value.
Construction establishes a complete value in storage. Destruction ends a
complete value and its remaining member lifetimes. Deallocation is a separate
responsibility of the applicable storage or ownership policy.

Reconstructive replacement reuses the outer storage while recycling the
previous representation and resources into a new complete immutable value
lifetime.

## Ordinary construction

### Constructor declarations

An ordinary constructor is a type-owned `+++` declaration:

```zax
Connection :: type {
  +++ final : ()(endpoint : Endpoint) = {
    // Establish the Connection.
  }
}
```

An ordinary constructor:

- has no result values;
- accepts zero or more ordinary input parameters;
- participates in ordinary parameter qualification and
  [callable selection](function-invocation.md#candidate-selection);
- receives `_` as the current instance under construction;
- may establish final and immutable state without `unsafe pliable`; and
- must establish a complete instance on every normal return.

Construction authority records the value's actual qualifications while allowing
the current instance to be established. It is not a general unsafe escape.

The complete instance's final and immutable guarantees activate only when
construction finishes. Before then, the compiler may know that some members are
live while others are not.

### Automatic and explicit member construction

Before an ordinary constructor body runs, the compiler determines which members
are under explicit construction control.

- A member without an explicit `+++` call is initialized automatically before
  the enclosing constructor body.
- Automatic member initialization follows member declaration order.
- A member with an explicit `_.member.+++()` call is not automatically
  initialized.
- Explicit member constructors execute wherever constructor-body control flow
  reaches them and may run in any order.
- After an explicit member `+++` completes, that member is live.
- A later `_.member = value` performs ordinary operator selection against the
  already-live member.
- Normal completion requires every member not covered by an explicit unsafe
  initialization bypass to contain a valid live value.

```zax
MyType :: type {
  t : T
  u : U
  x : X
  y : Y
  z : Z

  +++ final : ()() = {
    _.t.+++()
    _.t = makeAT()

    _.u = makeAU()

    _.z.+++()
    _.x.+++()
  }
}
```

`u` and `y` initialize automatically before the body, in declaration order.
`t`, `z`, and `x` initialize where their explicit calls execute.

The direct call permits construction of a member whose type has no default
constructor:

```zax
Session :: type {
  connection : Connection

  +++ final : ()(endpoint : Endpoint) = {
    _.connection.+++(endpoint)
  }
}
```

The compiler tracks member lifecycle states through control flow. It diagnoses
required cases such as:

- use before construction;
- missing construction on a normal completion path;
- constructing one still-live member lifetime more than once;
- conflicting call-site and constructor-body construction; and
- normal return with an incomplete instance.

The analysis is static; Zax does not require per-instance runtime flags.
Compiler debugging modes may add checks without making those checks language
guarantees.

## Construction packets

A construction packet distinguishes positional constructor arguments, named
constructor arguments, and stored-member initializers:

```zax
[{
  positionalExpression,
  parameterName: namedExpression,
  : explicitlyPositionalExpression,
  :,
  parameterWithDefault:,
  .memberName = memberExpression
}]
```

- `expression` supplies an ordinary positional constructor argument.
- `parameterName: expression` supplies a named constructor argument.
- `: expression` makes positional intent explicit.
- empty `:` or `parameterName:` explicitly omits a parameter and requires its
  declared default.
- `: :` or `parameterName: :` explicitly supplies the parameter type's default
  value at that packet position.
- `.memberName = expression` supplies direct call-site initialization for a
  stored member.

The categories are explicit. A named argument never falls back to a stored
member, and a stored-member entry never falls back to a parameter.

The packet syntax is current conceptual design and should be used in
construction examples. Its formal grammar may be refined if later concrete
pressure requires a change.

`[{}]` is the canonical zero-entry packet:

```zax
explicit : MyType = [{}]
```

For an ordinary type it selects the same zero-input construction as an
initializer-free declaration when available. For `MyType?`, it explicitly
constructs a present wrapper containing a zero-input-constructed `MyType`;
initializer-free optional construction is absent.

`[{ }]` is a confusable-form intent error because its blank body looks like the
programmer opened a nonempty packet but forgot its arguments. Contiguous `[{}]`
acknowledges intentionally zero inputs. A packet is also not independently an
expression value; anonymous typed construction supplies a destination where an
expression hole needs one. Complete optional and source-structure consequences
are defined by [Zax optional values](optional-values.md#empty-construction-packets)
and [Zax source structure](source-structure.md#optional-layers-and-empty-construction-packets).

### Declaration expressions as inputs

An entry expression may itself declare a temporary:

```zax
[{
  : myValue : String,
  name: temporary : String,
  .member = makeMember()
}]
```

The outer `:` or `name:` categorizes the packet entry. The nested declaration
expression introduces and evaluates the temporary. A bare declaration
expression in a label-capable entry is an intent error; explicit positional
intent or grouping disambiguates it. See
[Zax function invocation](function-invocation.md#argument-declaration-expressions).

### Positional cursor

Argument mapping is tested separately for each constructor candidate:

1. At packet entry, the positional cursor points to the first parameter.
2. A bare or explicitly positional expression binds the current positional
   parameter and advances.
3. `name: expression` binds that named parameter and resets the cursor to the
   parameter following it.
4. An empty positional or named omission maps its parameter and advances or
   resets the cursor in the same way, but evaluates no value during the explicit
   input phase.
5. `.member = expression` does not establish a constructor position and
   invalidates the positional cursor.
6. A later named constructor argument may establish the cursor again.
7. A positional entry while the cursor is invalid is not viable.
8. A positional entry does not skip a defaulted parameter.

```zax
[{
  .legs = 2,
  kind: "grizzly",
  afterKind
}]
```

`afterKind` maps to the parameter after `kind` for each candidate. Different
candidates may map it differently. Ordinary viability and ranking choose a
clear winner; an unresolved tie is an error.

Closely overlapping overload contracts can still create logic mistakes when a
later declaration change makes another candidate a better match. Zax diagnoses
ambiguity but does not prohibit advanced overload sets whose distinctions
require care.

### Number literals in constructor selection

Number literals participate in the same constructor selection after arguments
are mapped. Suppose `MyValue` has constructors accepting `U32` and `I8`:

```zax
myValue : MyValue = 55 // error: both constructors accept 55
myU32Value : MyValue = (: U32 = 55) // selects the U32 constructor
```

The compiler tests only the declared parameter types. Suppose `MyContainer`
accepts `MyInteger`, and `MyInteger` can itself be constructed from an integer.
The compiler does not invent that nested construction:

```zax
myContainer : MyContainer = 55
// error: MyContainer requires MyInteger, not an integer parameter

myContainer : MyContainer = (: MyInteger = 55)
```

The second form explicitly creates the `MyInteger` value. Integer argument
selection and the no-invented-temporary rule are explained by
[Zax integer literals and realization](integer-literals.md#when-a-visible-type-may-complete-an-operand).

### Contextual and explicit constructors

An operator with a visible typed peer may sometimes request the same
construction without the programmer writing the nested declaration:

```zax
myValue := 0 + (: MyInteger = 5)
```

That construction may run user code and has real effects and costs. It is
available only when the constructor and operator both opt in with the
`contextual` completion mode.

Illustrative constructor syntax:

```zax
+++ contextual final : ()(rhs : Integer) = {
}
```

Assume `MyContextualInteger` also provides a contextual `+`. Its constructor
allows the missing left value to be built:

```zax
myValue := 0 + (: MyContextualInteger = 5)
```

For construction, this is equivalent to:

```zax
myValue :=
  (: MyContextualInteger = 0) +
  (: MyContextualInteger = 5)
```

An explicit constructor refuses that inserted construction even when `+`
permits contextual completion:

```zax
myValue := 0 + (: MyExplicitInteger = 5)
// error: MyExplicitInteger requires explicit construction

myValue :=
  (: MyExplicitInteger = 0) +
  (: MyExplicitInteger = 5)
```

The second form supplies both values explicitly and may use the ordinary
operator.

`explicit` requires source to provide the constructed value before the compiler
may select a candidate that needs it, and omission defaults to `explicit`. The
exact final keyword position remains future declaration/source integration.
Operator participation and bounded fallback are defined by
[Zax operators](operators.md#contextual-completion).

Constructors and replacement constructors use the ordinary parameter-default
and omission model. A packet may omit an input only when the tested candidate
declares a default. Complete omission, default evaluation, and fixed-arity
preference are defined by
[Zax function invocation](function-invocation.md#omitted-inputs-and-defaults).

### Evaluation and binding order

Every value-producing packet entry evaluates strictly from left to right. The comma is a
non-overloadable sequencing operation for this purpose. Other expression
precedence rules still apply inside each entry.

For each value-producing entry in source order:

1. evaluate its expression;
2. immediately initialize or bind the selected input slot according to its
   `copy`, `move`, `last`, reference, pointer, or other parameter semantics; and
3. complete the observable effects of that binding before evaluating the next
   entry.

Empty omission entries perform no evaluation in packet order. After all explicit
value-producing entries, including stored-member inputs, complete, declared
defaults for still-unbound constructor parameters evaluate in the selected
constructor prototype's parameter order.

After every input is ready, member construction follows the selected constructor
plan. Packet order does not reorder members.

- A copied input captures the source value when that entry is evaluated.
- A reference input binds at evaluation but observes later changes to its
  referent.
- A by-value `deep`, `move`, or `last` input performs its required parameter
  construction during that entry's binding.
- A `move`- or `last`-stanced reference binds directly to caller-owned storage;
  the selected lifecycle body may take resources later during its complete
  call.
- A call-site member initializer supplies the already evaluated input used when
  the construction plan reaches that member.

```zax
source : Integer = 1

value : Example = [{
  .snapshot = source,
  changeSourceToTwo(source)
}]
```

If `snapshot` selects `copy` semantics, its input captures `1` before
`changeSourceToTwo` runs. If it selects reference semantics, the bound reference
later observes `2`.

The compiler may use a conceptual temporary to preserve this behavior and may
elide it only when observable `copy`, `move`, construction, and destruction behavior
does not change.

A temporary bound to a reference input survives through completion of the
selected lifecycle call. Other packet temporaries survive until their values
have been transferred and their specified destruction point is reached.
Complete temporary and full-expression rules remain future function and
lifetime work.

### Multiple-result inputs

A bare multiple-result invocation in a packet may supply consecutive constructor
parameters:

```zax
pair : Pair = [{
  produceTwo()
}]
```

This uses the shared
[result-forwarding model](function-invocation.md#forwarding-results). It does not
pack the results into one structural value and does not initialize stored members
by result label.

### Mixed call-site member initialization

A `.member = expression` entry:

- suppresses that member's ordinary default initialization;
- evaluates its expression in packet order;
- initializes the member when the construction plan reaches it; and
- completes before the enclosing constructor body begins.

An explicit `_.member.+++()` in the selected constructor body conflicts because
both operations attempt to construct the same member. The compiler diagnoses
that conflict.

Low-level code may eventually use a narrow unsafe lifecycle assertion when
opaque control flow makes the apparent conflict valid. The syntax and formal
guarantees for such assertions remain future safety and analysis-control work.

### Anonymous structural values are distinct

A future anonymous-structure expression would produce one structural value:

```zax
animal : Animal = {{
  value: first,
  kind: "grizzly",
  legs: 2
}}
```

The doubled braces are illustrative future syntax, not current structural-type
design. The semantic distinction is that one anonymous value becomes one
constructor input; it is not a packet containing several arguments and member
initializers.

## Declared and generated lifecycle operations

Zax distinguishes five declaration states:

| Form | Meaning |
| --- | --- |
| Declaration with a body | Use the programmer-defined implementation |
| Bodyless declaration | Declare an unavailable requirement that blocks weaker selection and that composition or later completion may fulfill |
| `= default` | Require the compiler to supply the language-defined default implementation |
| `= existing` | Explicitly satisfy this shape using a compatible existing operation |
| `= forbidden` | Permanently prohibit this operation shape |

`forbidden` is a hard prohibition. Generation or compatibility fallback cannot
restore the operation.

### Bodyless declarations

A bodyless declaration claims its signature and suppresses automatic generation
for that shape. It participates in match-quality comparison while remaining
unavailable for invocation.

- If it is the best match, selection fails because the type does not currently
  support the operation. The compiler does not fall through to a weaker
  candidate.
- If a latent generated candidate would match better, the compiler requires an
  explicit declaration for that better shape rather than generating it
  silently.
- Composition or later completion may supply the missing implementation.

### `default`, `existing`, and `forbidden`

`= default` selects the language-defined default implementation for that exact
declaration shape. The compiler must supply it or diagnose that no default
exists:

```zax
+++ final : ()() = default
```

An arbitrary function does not acquire invented behavior merely because the
syntax parses:

```zax
myFunc : ()() = default // error: no language-defined default implementation
```

`= existing` deliberately reuses a compatible operation:

```zax
+++ final : ()(value : Value copy) = {
  // Programmer-defined copy behavior.
}

+++ final : ()(value : Value move) = existing
```

Callable compatibility alone does not establish lifecycle-contract
compatibility. `existing` makes reuse explicit when `copy`, `move`, source-state, or
cost behavior might otherwise differ.

`= forbidden` prohibits the exact shape:

```zax
replacement +++ final :
  ()(source : Source) = forbidden
```

### Demand-driven generation

The programmer declares only operations intended for use. Zax does not require
an exhaustive list of theoretical variations.

When an actual use could choose between a compatible existing declaration and a
newly generated specialization with meaningfully different behavior, the
compiler does not guess. The programmer resolves the demanded shape by:

- providing a body;
- writing `= existing`;
- writing `= default`;
- writing `= forbidden`; or
- leaving a bodyless requirement for later composition or completion.

No error is required for an unused latent ambiguity.

A generated empty constructor exists only when every required member has a
viable default or declared initialization path. Declaring the default-construction
shape suppresses generation of that shape.

For a containing type, a generated operation selects member operations that
satisfy the generated lifecycle contract. It does not select any callable member
operation merely because one exists.

Generated `copy` is the conservative transfer baseline. Zax does not generally
generate `deep`, `move`, or `last` operations:

- `deep` requires an exact `deep` declaration;
- `move` falls back to `copy` when no `move` declaration exists;
- `last` falls back to `move` and then `copy`;
- and a generated containing `copy` selects the exact member `copy` operations
  required by its contract.

Complete stance meaning and fallback are defined by
[Zax transfer stances](transfer-stances.md#fallback).

Reflection must eventually distinguish declared, generated, explicitly
defaulted, delegated-to-existing, bodyless, and forbidden operations.

### Generated copy construction and assignment

Generated `copy` is one compiler-owned schema exposed as exact,
qualifier-complete declarations. It is not a qualifier-erased prototype.

Construction creates a new value lifetime, so both mutable and immutable values
may have generated `copy` constructors. For a mutable value shape, the compiler
conceptually supplies exact source-place variants:

```zax
+++ final : ()(
  : MyValue mutable readonly final & copy
) = default

+++ final : ()(
  : MyValue mutable readonly varying & copy
) = default
```

The source parameter is compiler-anonymous. Construction has no receiver stance
because no destination lifetime exists yet.

A corresponding immutable value shape may receive immutable source variants.
Ordinarily `mutable` and `immutable` qualify the same representation. Future
mutability-indexed type-family design may instead permit independently defined
shapes; each such concrete shape still receives only its appropriate
constructors.

Canonical same-type `=` treats the destination as a complete value. Its receiver
place must therefore be `varying`: that place permits the complete value to be
assigned as a unit. For a mutable varying receiver, the compiler conceptually
supplies:

```zax
operator binary '=' final : (
  : MyValue mutable readonly varying & copy
)(
  : MyValue mutable readonly final & copy
) mutable writable varying = default

operator binary '=' final : (
  : MyValue mutable readonly varying & copy
)(
  : MyValue mutable readonly varying & copy
) mutable writable varying = default
```

The result and source slots are compiler-anonymous. The language-defined body
returns readonly `copy` access to the assignment receiver `_`, permitting
right-associated assignment:

```zax
first = second = third
```

The `final`-source and `varying`-source forms are different declarations.

A `final` receiver promises that its place continues to hold the same value
lifetime. It may still mutate the contents of a mutable value through eligible
member functions or member assignments, but canonical whole-value assignment is
unavailable:

```zax
stable final : MyValue mutable final

stable.member = replacement // may mutate the current value
stable = anotherValue       // error: canonical whole-value assignment needs a varying place
```

A domain-specific `=` may explicitly accept a `final` receiver when it defines
coherent in-lifetime behavior rather than replacing the complete value.

An immutable value cannot use an ordinary assignment that mutates its contents.
When it occupies a `varying` place, the compiler-owned reconstructive `=`
skeleton may instead construct a new immutable value, disposition the old one,
end the old lifetime, and establish the new lifetime in the same place. That
reconstructive family preserves:

- immutable receiver truth;
- type-side varying capability;
- declaration-side varying replacement permission;
- writable access;
- exact source qualification;
- readonly `copy` access to the new current receiver lifetime; and
- exactly-once old-resource disposition.

One concrete value shape is `mutable` or `immutable`, never both. There is no
runtime branch over mutability or place stance inside one generated declaration.
An independently defined immutable shape receives constructors but no ordinary
mutating assignment; reconstructive `=` remains a separate compiler-owned
lifecycle process.

Generation is suppressed or replaced only by the exact generated shape. A
nearly matching programmer declaration may coexist with the generated
declaration and produce demand-time ambiguity. Diagnostics should show the exact
generated signature and the mismatching layer.

## Reconstructive replacement

### Why replacement exists

Generated reconstructive replacement is the whole-value transition for an
`immutable` destination whose place is type-side `varying`, reached through a
`writable` path whose declaration carries `varying` replacement permission:

- `immutable` prevents mutation within one complete value lifetime;
- type-side `varying` means the place may receive another complete value
  lifetime;
- `writable` permits the current access path to change anything at all; and
- declaration-side `varying` permits *this* path to initiate a whole-value
  replacement.

```zax
message varying : Message immutable writable varying = makeMessage("first")
message = makeMessage("second")
```

Neither immutable instance mutates. The old lifetime shuts down and a new
immutable lifetime begins in the same outer storage.

A restricted same-place path observes that transition without being able to
initiate it:

```zax
observer final :
  Message immutable readonly varying & = message

restricted final :
  Message immutable writable varying & = message

restricted = makeMessage("third")
// error: this declaration lacks replacement permission

message = makeMessage("third") // legal through the varying declaration
display(observer)              // "third"
```

`restricted` remains writable for operations that do not require whole-value
replacement, and its type use still records truthfully that another path may
replace the referent. Type-side capability versus declaration-side permission is
owned by
[Zax qualifiers](qualifiers.md#type-side-truth-versus-declaration-side-permission).

A `mutable`, `writable`, `varying` destination uses ordinary assignment
selection. It does not receive this generated immutable-value replacement
facility merely because its place is varying.

The compiler owns the generated reconstructive `=` lifecycle skeleton.
User-defined code may provide a replacement constructor but does not replace the
skeleton with an ordinary operator body.

### Generated fallback

When no selected custom replacement constructor handles the transition, the
generated fallback:

1. invokes the old instance's ordinary `---`;
2. retains the already allocated outer storage;
3. invokes the selected ordinary `+++`; and
4. performs the ordinary new member-initialization process.

The fallback assumes that no old resource is intentionally retained through the
transition.

### Custom replacement

A custom replacement constructor uses contextual `replacement +++`:

```zax
Registration :: type {
  id : RegistrationId
  settings : Settings

  +++ final : ()(settings : Settings) = {
    _.id = registry.register(settings)
    _.settings = settings
  }

  --- final : ()() = {
    registry.unregister(_.id)
  }

  replacement +++ final : ()(
    settings : Settings
  ) = {
    registry.reconfigure(_.id, settings)
    _.settings = settings
  }
}
```

When selected, the replacement constructor:

- suppresses the enclosing ordinary `---`;
- suppresses the enclosing ordinary `+++`;
- receives the prior representation and resources through `_`;
- has transitional construction authority;
- may retain, assign, move, copy, destroy, reconstruct, or otherwise recycle
  old resources while renewing every member resident instance;
- uses the existing outer allocation; and
- must establish a complete valid replacement instance on every normal return.

The example retains the `id` resource and representation. Running the ordinary
destructor first would unregister it and defeat recycling. Complete replacement
nevertheless renews `id`'s member resident instance along with every other
member. Resource reuse is not member-lifetime retention, and leaving bits at an
address after ending a member lifetime does not preserve that old instance.

For a member carried forward in place, renewal is a language-level lifetime
boundary controlled by the replacement constructor:

- the old member resident instance ends as part of the enclosing replacement;
- no ordinary member `---` or `+++` hook is generated around the carried
  representation;
- the replacement constructor transfers that representation and its resources
  into the successor member instance; and
- normal return publishes the successor only after the complete replacement is
  valid.

The retained resource therefore receives one continuing owner rather than a
destructor followed by reconstruction. References to the old member still cross
a real resident-instance boundary even though no member hook ran.

The compiler tracks member transitions. Normal return requires one live valid
value for every required member and exactly one disposition for every resource
the transition ceased to retain. A carried resource counts as transferred into
the successor member, not left undispositioned.

### Replacement results

A replacement constructor may declare zero or more results:

```zax
BufferOwner :: type {
  buffer : Buffer
  format : Format

  replacement +++ final :
    (retainedCapacity : Boolean)(
      nextFormat : Format
    ) = {
    capacityWasRetained := _.buffer.canReuseFor(nextFormat)

    // Establish the complete replacement instance.

    return capacityWasRetained
  }
}

retained := owner = newFormat
```

The generated reconstructive `=` expression forwards the literal results
returned by the selected replacement constructor. It does not synthesize,
adapt, or infer an arbitrary result.

A resultless replacement declaration supplies no result:

```zax
replacement +++ final : ()(rhs : Source) = {
  // Complete replacement.
}
```

The compiler does not implicitly return the reconstructed destination. A
resultless generated fallback also supplies no expression result.

Every normal return must complete:

- the replacement instance; and
- every declared result.

A result may report resource reuse or another domain outcome. It cannot mean
that the destination is half-constructed. The compiler cannot infer that
programmer intent from a Boolean or name, so API design and documentation must
make the distinction understandable.

Result references and pointers remain subject to ordinary lifetime and alias
rules. Expected-result context participates only at the narrow complete
declaration boundaries defined by
[function invocation](function-invocation.md#narrow-expected-result-selection).

### Candidate selection

Being generated does not make a candidate automatically win or lose. Generated
reconstructive `=` and declared domain-specific `=` candidates participate in
ordinary selection according to their declarations and qualifications. An
unresolved equal match is an error.

Direct compound and mixfix operations remain separate:

```zax
destination += rhs
destination[index] = rhs
```

The compound form invokes its own selected mutation candidate. The indexed form
may select one direct mixfix whose receiver, index, and RHS evaluate once. Neither
is automatically rewritten through a value operation followed by `=`.

A user mixfix that consumes a written `=` component does not acquire the
compiler-owned reconstructive lifecycle skeleton. If its body mutates a current
value, ordinary mutable/writable authority applies. Reconstructing an immutable
varying place remains this document's compiler-owned operation.

See [Zax mixfix operators](mixfix-operators.md#qualifications-and-lifecycle) for
tree matching and the
[integer operator catalog](integer-operator-catalog.md#compound-arithmetic) for
protected integer compounds.

When an actual use exposes an existing-versus-generated choice, the programmer
resolves the demanded shape with a body, `existing`, `default`, `forbidden`, or a
bodyless declaration.

Construction and replacement may be controlled independently:

```zax
+++ final : ()(source : Source) = default

replacement +++ final :
  ()(source : Source) = forbidden
```

This type permits construction from `Source` while prohibiting replacement from
`Source`.

A separate domain-specific operator may return any result allowed by ordinary
operator rules. That is distinct from a replacement constructor's declared
results.

### Self-aliasing and interior aliases

Evaluating a right-hand expression before replacement begins does not guarantee
independence. A reference result may point into the destination:

```zax
Document :: type {
  text : String

  replacement +++ final : ()(
    source : String readonly &
  ) = {
    _.text = source
  }

  stringViewFrom final :
    (result : String readonly &)() = {
    return _.text
  }
}

document varying : Document immutable writable varying
document = document.stringViewFrom()
```

`source` may alias `_.text`. Delegating to `String`'s selected `=` lets `String`
own exact self-alias handling, but it does not remove the general pressure.

The same issue appears in:

```zax
value = value
value = viewOf(value)
value = makeReplacementUsing(value)
```

Safe cases may include:

- a by-value snapshot established before transition;
- an operation designed to handle exact self-aliasing; or
- a lifetime policy that proves nonconflicting access.

The compiler diagnoses conflicts required by the applicable language contract.
A future narrow unsafe permission may allow a use whose safety the compiler
cannot prove. The permission belongs at the risky operation, not on every `&`
declaration.

Complete reference origin, fixed place binding, and the unsafe successor-member
boundary are defined by
[lifetimes and references](lifetimes-and-references.md#direct-member-references-cross-a-renewal-boundary).

### Raw pointers

Same-storage replacement does not categorically invalidate every raw pointer.

- A pointer into retained storage may continue to name the same location.
- A pointer to a value whose lifetime ended no longer identifies that old live
  value, even when another value later occupies the address.
- Reusing an address does not by itself decide whether the intended pointee
  survives.

Raw pointers carry no ownership or intrinsic lifetime guarantee. A use is safe
when analysis proves its required origin and lifetime facts; otherwise it needs
narrow unsafe responsibility. Managed ownership, including direct-member
anchored pointers, is defined by
[pointers and arenas](pointers-and-arenas.md).

### Representation boundary

Same-storage replacement requires the existing place to satisfy the selected
replacement representation's size, alignment, and layout.

If a future type-family design gives mutable and immutable variants different
representations, sharing one family name does not make cross-variant
same-storage replacement viable. A different representation may require
separately suitable storage and a construction, `copy`, or consuming
transformation.

Unions and overlapping storage likewise need future active-member transition
rules before generated replacement can operate on them safely.

### Optional complete-wrapper replacement

Same-type optional `=` replaces the complete wrapper lifetime rather than
assigning or conditionally constructing only its boxed value:

```zax
destinationOptional : MyType?
sourceOptional : MyType?

// ...

destinationOptional = sourceOptional
```

It requires a type-side varying wrapper place, declaration-side replacement
permission, writable access, and a compatible optional source transfer. The old
wrapper and any present payload end; a new absent or present wrapper is directly
constructed from the source state.

This is distinct from packet construction, which retains one mutable wrapper
lifetime while ending and constructing boxed lifetimes:

```zax
destination = [{ sourceValue }]
```

It is also distinct from an operation selected after proven boxed access:

```zax
if ?destination
  destination. = sourceValue
```

The optional family therefore exposes a protected whole-wrapper lifecycle
operation even when the wrapper is mutable; it does not turn ordinary
value-shaped `T? = T` source into construction. Complete state, qualification,
intent, and transfer behavior is owned by
[Zax optional values](optional-values.md#construction-wrapper-replacement-and-boxed-assignment).

## Destruction

### Destructor declarations and sequence

A concrete type has at most one destructor:

```zax
--- final : ()() = {
  // Dismantle the current instance.
}
```

A destructor accepts no ordinary input arguments and returns no results.

Ordinary destruction:

1. enters terminal destruction authority;
2. runs the enclosing `---` body while every not-yet-destroyed member remains
   live;
3. automatically destroys remaining members after the body; and
4. ends the enclosing lifetime.

Automatic member destruction follows reverse declaration/default-construction
order. It does not reconstruct the arbitrary runtime order chosen by explicit
constructor-body `+++` calls.

An explicit `_.member.---()` call may run in any destructor-body control-flow
order and suppresses the ordinary generated destructor call for that member
lifetime. The compiler tracks which member lifetimes remain live at every normal
destructor exit.

Destruction receives terminal mutable and writable authority. It may dismantle
resources but cannot create an access path valid beyond the enclosing lifetime.

### Reconstruction during destruction

A destructor may reconstruct a member:

```zax
--- final : ()() = {
  _.member.---()
  _.member.+++()
  use(_.member)
  _.member.---()
}
```

The two member destructor calls end two different lifetimes. Lifecycle analysis
should distinguish them rather than report double destruction of one lifetime.

Advanced terminal code may need a narrow future unsafe permission when required
analysis cannot prove its lifecycle behavior. Generated destruction suppression
depends on tracked member state at normal exit, not merely on the presence of a
`---` token.

The exact severity of optional diagnostics and any runtime debugging checks
remain future safety and tooling work.

## Scope-exit destruction and flow transfers

Local, body, and flow-header lifetimes end automatically when their scope exits,
whether that exit is a normal completion or an abrupt transfer. Destruction runs
in reverse construction order.

On normal body completion:

1. body-local bindings are destroyed;
2. the construct runs its post operation when it requires one, while header
   bindings remain alive;
3. the construct tests, re-enters, or exits; and
4. header bindings are destroyed when the complete construct exits.

An abrupt transfer such as `break`, `continue`, `next`, or `return` destroys every
body or nested scope it leaves, in reverse construction order, before control
arrives at the target:

```zax
while i := 0 ;; i < 100 ;; ++i {
  resource := acquire()
  if done(resource)
    break // resource is destroyed, then i, as control leaves the loop
}
```

`break` and `return` skip ordinary post operations but still perform this
destruction. `next` runs the target's post operation before proceeding, while
`continue` skips it; both keep the target's own header bindings alive because the
complete target construct has not exited. A target header binding survives a
`next` or `continue` re-entry and is destroyed only when the complete target flow
statement exits.

Which scopes a transfer crosses, whether the post operation runs, and how targets
are selected are owned by
[core flow control](core-flow-control.md#unwinding-destruction-and-completion).
This document owns the automatic local and header lifetime order that results.
Construction and result completeness must still hold on every normal path a
branch, loop, or transfer produces.

## Conditionally live storage and access proof

### Optional construction must establish the contained value

Constructing a present `T?` succeeds only when its selected construction can
establish a valid `T`:

```zax
myValue : MyType
myOptional := (: MyType? = myValue)
```

An optional may forward an initializer into a contextual contained constructor,
but failure remains an error:

```zax
myOptional := (: U8? = 355) // error: 355 does not fit U8
```

It does not become absent, narrow the value, or select default optional
construction after present construction fails. Absence occurs only when the
selected operation explicitly requests or produces it.

Type-default optional construction is absent, while `[{}]` explicitly requests
present zero-input boxed construction. A packet on an existing mutable/writable
wrapper evaluates and binds its inputs, ends any old boxed lifetime, constructs
the fresh value, and marks presence only after completion.

Reset ends the boxed lifetime and leaves the same wrapper absent. Direct
destruction through boxed access is rejected because it could not update wrapper
presence:

```zax
optional.---() // error
```

Complete optional operations, nesting, qualification, and transfer are defined
by [Zax optional values](optional-values.md). The number-literal application is
explained by [Zax integer literals and realization](integer-literals.md#optional-construction-still-has-to-build-a-value).

### Access proof

Some storage may or may not hold a live value on a given path. Access through
such storage requires proof that a live value exists on that path:

> Access through storage whose value lifetime may not be active requires proof
> that a live value exists on that path.

The concern is whether a value lifetime is active, not how a particular feature
spells its presence test. It covers conditionally live versus merely
unconstructed storage, use before construction, and proof that a value lifetime
is active before access.

Optional stored-value dereference is one application. Postfix optional
dereference produces a reference to the stored value and requires static proof
that the optional holds a live value on the dereferencing path. Absent that
proof, the dereference is a semantic error rather than a runtime hazard:

```zax
if ?optionalValue
  use(optionalValue.) // proven live on this path
```

Test presence first and dereference inside the proven body. A compound proof such
as `?optionalValue && ?optionalValue.` proves nothing unless the right expression
is unambiguously exactly `Boolean`, because only then does the protected
short-circuit operation skip the dereference; see
[operators](operators.md#optional-presence-operation).

The proof need not be immediately adjacent. Construction, earlier control flow, a
preceding presence test, or another recognized presence contract may establish
it. An arbitrary user-defined Boolean-returning `?` does not by itself prove
initialization; the analyzer needs a recognized presence contract.

Wrapper qualifications do not replace the boxed qualifications after access.
Once proof permits `optional.`, the resulting path carries the boxed
qualifications. Any operation ending that exact boxed lifetime invalidates the
proof and every reference tied to it; see
[Zax optional values](optional-values.md#presence-proof-and-postfix-access).

The presence operation `?value` itself is owned by [operators](operators.md), and
condition placement is owned by [core flow control](core-flow-control.md). This
document owns only the programmer-visible lifetime obligation: access proves a
live value. Detailed proof algorithms, assertion syntax, contract provenance, and
lint separation remain future analysis work, and validity follows the selected
static-analysis contract rather than one compiler's current cleverness.

## Manual and delayed construction

### `unsafe ???`

The declaration form:

```zax
value : MyType = unsafe ???
```

establishes a binding and storage while explicitly bypassing ordinary
initialization. It leaves representation validity and eventual destruction under
unsafe programmer responsibility.

Explicit delayed construction remains legal:

```zax
value : MyType = unsafe ???
value.+++()
```

The later `+++` performs the initialization, member construction, allocation,
and constructor-body work that ordinary declaration initialization would have
performed. Calling `+++` again on the same still-live lifetime may duplicate
initialization or leak resources.

### Stored members with `unsafe ???`

A member initializer of `unsafe ???` is already the programmer's unsafe
disposition:

```zax
MyType :: type {
  s : S = unsafe ???
  t : T

  +++ final : ()() = {
    _.t.+++()
  }
}
```

For `s`:

- storage exists;
- ordinary default initialization and `S.+++()` are bypassed;
- destruction remains scheduled;
- representation remains indeterminate unless another operation establishes it;
- the containing constructor may complete without `_.s.+++()`; and
- the programmer assumes responsibility for every later use and for valid
  destruction.

The compiler neither initializes `s` automatically nor requires an explicit
`_.s.+++()`. Assembly, foreign code, an opaque operation, or another low-level
mechanism may establish the representation. Leaving `s` unreferenced is not a
language error.

Calling `_.s.+++()` later is legal and transitions the member into an ordinarily
constructed state. It is optional. `unsafe ???` already supplies the unsafe
acknowledgment; delayed construction does not require another unsafe category.

Lifecycle tracking distinguishes ordinary construction, explicit bypass,
ordinary construction after bypass, and an ended lifetime. The bypass satisfies
the enclosing constructor's completion requirement through unsafe programmer
responsibility rather than proof of a valid `S` representation.

## Incomplete current instances and unsafe controls

A helper may receive the current instance while construction is incomplete:

```zax
+++ final : ()() = {
  _.first.+++()
  initializeRemaining(_)
}
```

The compiler may permit the call when it proves that the helper uses only live
members and completes a valid state. Otherwise a future narrow unsafe
partial-instance permission is required.

The current instance retains its actual immutable qualification while
construction authority permits establishment of its state. An
immutable-compatible helper is preferred. Construction authority may permit a
mutable-compatible helper when no immutable form is viable, subject to the same
partial-state analysis.

Publishing an incomplete current instance is a stronger operation:

```zax
+++ final : ()() = {
  registerGlobally(_)
}
```

Publication may retain a reference, notify subscribers, or reenter through
another path. Permission for bounded helper access does not imply permission for
escape.

The language needs distinct future unsafe controls for:

- manual member construction or destruction hidden from analysis;
- construction or destruction that analysis cannot prove occurs on every path;
- apparently overlapping lifecycle calls known to be mutually exclusive;
- bounded partial-instance access;
- partial-instance escape or publication;
- unresolved replacement aliasing; and
- terminal member reconstruction.

The final source syntax, analysis provenance, and contract-version behavior
remain future analysis-control work. The general distinction among proof,
unsafe assertion, unsafe permission, and no valid interpretation is defined by
[safety and analysis](safety-and-analysis.md). No broad unsafe marker can make a
known-ended lifetime valid again.

## Panic and allocation boundaries

Unresolved panic is fatal graceful crashing. A panic may resolve or be
intentionally suppressed through its eventual mechanism; otherwise the process
ends.

If execution continues inside construction or replacement after a resolved
panic, normal completion obligations still apply. Zax does not currently
require exception-style rollback to the old value or recoverable
partial-construction unwinding.

Automatic allocation occurs before construction of the allocated value.
The selected allocation operator decides how exhaustion appears:

- a panicking form prevents normal construction completion and panics;
- a non-panicking form constructs a pointer member containing `Nothing`, so the
  enclosing instance may still complete; or
- an unchecked form makes a false success guarantee undefined.

Complete behavior is defined by
[pointers and arenas](pointers-and-arenas.md#allocation-failure).

Constructors, replacement constructors, and destructors are synchronous.
Suspending lifecycle operations, cancellation, and concurrent replacement remain
future async, lifetime, and concurrency work.

## Identity admission boundary

By-value identity admission requests a new value. It requires an applicable
`copy`, `move`, consuming/`last` transfer, direct construction, or another declared
way to establish the identity's underlying stored value.

Identity syntax does not manufacture copyability. When no applicable
construction or transfer exists, by-value admission is unavailable.

A same-storage identity reference would instead view existing storage and create
no independent value lifetime. Its representation-cast, qualification, alias,
lifetime, and destruction rules remain future identity/owned-composition work.
The compiler does not silently substitute such a view for a requested by-value
result.

General identity behavior is defined by
[Zax identity types](identity-types.md#construction-and-transfer).

## Costs

Programmers must be able to discover:

- automatic versus explicit member lifecycle operations;
- temporaries retained while a construction packet evaluates;
- declared constructor defaults evaluated after explicit packet inputs;
- `copy`, `move`, `last`, and reference binding performed for packet entries;
- generated fallback replacement as `---` followed by `+++`;
- resources retained or reconstructed by custom replacement;
- copies or snapshots required to avoid alias hazards;
- scope-exit destruction of local and header bindings on normal and abrupt exits;
- post operations run on `next` and normal completion but skipped on `break`,
  `continue`, and `return`;
- allocator retention and deallocation behavior;
- static-analysis and unsafe-override boundaries; and
- synchronization or async machinery when future facilities use them.

## Diagnostics

Diagnostics should distinguish:

- no viable ordinary or replacement constructor;
- missing, duplicate, unknown, or ambiguously mapped packet entries;
- a positional entry with no current positional cursor;
- omission of a parameter that has no declared default;
- label-versus-declaration intent that is unclear;
- equal viable overloads;
- unresolved existing-versus-generated behavior at an actual use;
- use of a `forbidden` operation;
- a member that cannot initialize automatically;
- conflicting call-site and constructor-body member construction;
- use before construction or after destruction where required analysis proves
  it;
- an access through conditionally live storage, such as an optional stored-value
  dereference, without proof that a live value exists on that path;
- missing or duplicate lifecycle transitions on normal paths;
- normal completion with an incomplete instance or result;
- possible self or interior alias conflict during replacement;
- partial-instance access or escape requiring a future unsafe control; and
- an inapplicable or unsupported future unsafe semantic assertion.

Exact identifiers, wording, presentation, and mandatory-versus-tooling
boundaries remain future diagnostic and safety work.

## Formatting

Canonical formatting should preserve:

- contextual adjacency in `replacement +++`;
- visible construction-packet entry categories;
- left-to-right packet entry order;
- explicit positional intent;
- omission versus explicit type-default input;
- ordinary qualifier and parameter ordering;
- explicit `= default`, `= existing`, and `= forbidden`; and
- narrow unsafe-control documentation when those controls are later defined.

Formatters must not reorder packet entries because evaluation order is
observable. They must not add or remove explicit positional intent, convert
omission into type-default initialization, or parenthesize a bare multiple-result
invocation in a way that changes it into one expression.

## Boundaries and maturity

This document is current conceptual design, not formal grammar, an
implementation mapping, a compatibility contract, or a conformance
specification.

Later work may refine syntax and adjacent mechanisms while preserving:

- separate storage, member-lifetime, and complete-instance states;
- declaration-order automatic construction and reverse automatic destruction;
- automatic local and header lifetime ending and destruction order across normal
  and abrupt scope exits;
- the programmer-visible obligation to prove a live value before access through
  conditionally live storage;
- arbitrary control-flow order for explicit member lifecycle calls;
- static lifecycle tracking without mandatory runtime flags;
- shared ordinary-call input forms while `.member = expression` remains
  construction-specific;
- strict packet evaluation/binding order;
- constructor defaults after explicit packet inputs and before member
  construction;
- packet multiple-result forwarding remaining distinct from structural packing;
- ordinary constructors remaining resultless;
- demand-driven generated/existing/default/forbidden resolution;
- immutable + writable + varying generated reconstructive replacement;
- custom replacement recycling old representation without enclosing `---` or
  `+++`;
- complete replacement on every normal return, including resultful replacement;
- resultless replacement producing no implicit destination result;
- terminal destruction authority;
- `unsafe ???` satisfying construction through explicit unsafe responsibility;
- known pointer and alias hazards remaining visible;
- async, concurrency, ownership, and formal unsafe-control mechanisms remaining
  separate concerns until their focused reviews;
- differently represented variants requiring suitable storage rather than
  assuming family-name compatibility;
- unions and overlapping storage awaiting active-member transition rules; and
- global and `once` initialization, retry, and teardown ordering remaining
  future lifecycle work.
