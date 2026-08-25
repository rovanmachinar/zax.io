# Zax construction, replacement, and destruction

| Field | Value |
| --- | --- |
| Status | Current conceptual design |
| Audience | Human developers reading, writing, or evaluating Zax |
| Applies To | Programmer-facing value construction, reconstructive replacement, and destruction; not a formal grammar or specification |
| Implementation State | Not established by this repository |
| Owns | Ordinary constructors and destructors; automatic and explicit member lifecycle operations; construction packets; lifecycle declaration states and generated behavior; immutable reconstructive replacement; replacement constructors, resource retention, and results; construction/destruction authority; manual and delayed construction boundaries; lifecycle costs, diagnostics, and formatting |
| Does Not Own | Complete [declaration and binding behavior](declarations-and-bindings.md), complete [qualifier behavior](qualifiers.md), general [function invocation, parameter defaults, result routing, and callable preference](function-invocation.md), arbitrary operator semantics, complete move/copy and ownership, pointer/reference grammar and provenance, allocator APIs, formal unsafe-control syntax, panic recovery, async or concurrency behavior, structural identity and layout, formal grammar, diagnostic identifiers, or compiler and tooling implementation |

## Mental model

Zax distinguishes three related states:

1. **Storage exists.** A suitably sized and aligned place is available.
2. **Member lifetimes exist.** Individual contained places may or may not
   currently contain live values.
3. **The complete enclosing lifetime exists.** Ordinary code may use the
   complete instance after construction finishes and before replacement or
   destruction begins.

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
   copy, move, `last`, reference, pointer, or other parameter semantics; and
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
- A move or `last` input performs its eventual transfer semantics during that
  entry's evaluation.
- A call-site member initializer supplies the already evaluated input used when
  the construction plan reaches that member.

```zax
source : Integer = 1

value : Example = [{
    .snapshot = source,
    changeSourceToTwo(source)
}]
```

If `snapshot` selects copy semantics, its input captures `1` before
`changeSourceToTwo` runs. If it selects reference semantics, the bound reference
later observes `2`.

The compiler may use a conceptual temporary to preserve this behavior and may
elide it only when observable copy, move, construction, and destruction behavior
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
compatibility. `existing` makes reuse explicit when copy, move, source-state, or
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
operation merely because one exists. Complete copy, move, `last`, and containing
fallback rules remain future ownership and lifetime work.

Reflection must eventually distinguish declared, generated, explicitly
defaulted, delegated-to-existing, bodyless, and forbidden operations.

## Reconstructive replacement

### Why replacement exists

Generated reconstructive replacement is the whole-value transition for an
`immutable`, `writable`, `varying` destination:

- `immutable` prevents mutation within one complete value lifetime;
- `varying` permits the place to receive another complete value lifetime; and
- `writable` permits the current access path to initiate the transition.

```zax
message : Message immutable writable varying = makeMessage("first")
message = makeMessage("second")
```

Neither immutable instance mutates. The old lifetime shuts down and a new
immutable lifetime begins in the same outer storage.

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
  members;
- uses the existing outer allocation; and
- must establish a complete valid replacement instance on every normal return.

The example retains `id`. Running the ordinary destructor first would unregister
it and defeat recycling.

An untouched member retains its exact language-level lifetime and resource.
Leaving bits at an address after ending a member lifetime is not retention.

The compiler tracks member transitions. Normal return requires one live valid
value for every required member and exactly one disposition for every resource
the transition ceased to retain.

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
rules. Expected-result context may participate in selection only as allowed by
the eventual general callable-selection model.

### Candidate selection

Being generated does not make a candidate automatically win or lose. Generated
reconstructive `=` and declared domain-specific `=` candidates participate in
ordinary selection according to their declarations and qualifications. An
unresolved equal match is an error.

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

document : Document immutable writable varying
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
- a reference whose exact pointee lifetime is retained until its final use;
- an operation designed to handle exact self-aliasing; or
- a lifetime policy that proves nonconflicting access.

The compiler diagnoses conflicts required by the applicable language contract.
A future narrow unsafe permission may allow a use whose safety the compiler
cannot prove. The permission belongs at the risky operation, not on every `&`
declaration.

Complete reference origin, borrow, provenance, and lifetime-policy mechanisms
remain future pointer and lifetime work.

### Raw pointers

Same-storage replacement does not categorically invalidate every raw pointer.

- A pointer into retained storage may continue to name the same location.
- A pointer to an exact retained member lifetime may remain valid.
- A pointer to a value whose lifetime ended no longer identifies that old live
  value, even when another value later occupies the address.
- Reusing an address does not by itself decide whether the intended pointee
  survives.

Raw pointers are unsafe. Future pointer and lifetime policies may provide
stronger checked guarantees.

### Representation boundary

Same-storage replacement requires the existing place to satisfy the selected
replacement representation's size, alignment, and layout.

If a future type-family design gives mutable and immutable variants different
representations, sharing one family name does not make cross-variant
same-storage replacement viable. A different representation may require
separately suitable storage and a construction, copy, or consuming
transformation.

Unions and overlapping storage likewise need future active-member transition
rules before generated replacement can operate on them safely.

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
remain future safety and analysis-control work. No broad unsafe marker can make
a known-ended lifetime valid again.

## Panic and allocation boundaries

Unresolved panic is fatal graceful crashing. A panic may resolve or be
intentionally suppressed through its eventual mechanism; otherwise the process
ends.

If execution continues inside construction or replacement after a resolved
panic, normal completion obligations still apply. Zax does not currently
require exception-style rollback to the old value or recoverable
partial-construction unwinding.

Automatic allocation occurs before construction of the allocated value.
Ordinary allocation exhaustion is commonly fatal. Manual allocator APIs may
offer checked failure through ordinary control flow when a program has a real
recovery strategy.

Constructors, replacement constructors, and destructors are synchronous.
Suspending lifecycle operations, cancellation, and concurrent replacement remain
future async, lifetime, and concurrency work.

## Costs

Programmers must be able to discover:

- automatic versus explicit member lifecycle operations;
- temporaries retained while a construction packet evaluates;
- declared constructor defaults evaluated after explicit packet inputs;
- copy, move, `last`, and reference binding performed for packet entries;
- generated fallback replacement as `---` followed by `+++`;
- resources retained or reconstructed by custom replacement;
- copies or snapshots required to avoid alias hazards;
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
