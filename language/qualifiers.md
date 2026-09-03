# Zax qualifiers

| Field | Value |
| --- | --- |
| Status | Current conceptual design |
| Audience | Human developers reading, writing, or evaluating Zax |
| Applies To | Programmer-facing qualifier behavior; not a formal grammar or specification |
| Implementation State | Not established by this repository |
| Owns | Place-replacement, value-mutability, and access qualifiers; type-side truth versus declaration-side replacement permission; qualifier attachment, defaults, inheritance, restatement, ordering, ordinary promise strengthening, explicit unsafe weakening, deep immutability, semantic-indirection qualification boundaries, unsafe pliability, varying immutable places, reconstructive replacement at the depth required by qualifiers, receiver-operand constraints, and immediate construction, destruction, indirection, concurrency, and structural-typing boundaries |
| Does Not Own | Declaration/binding behavior ([declarations and bindings](declarations-and-bindings.md)); invocation/result preference ([function invocation](function-invocation.md)); lifecycle behavior ([construction and destruction](construction-and-destruction.md)); complete [optional behavior](optional-values.md); or complete pointer/lifetime rules |

## Mental model

Zax separates three questions that other languages often combine:

1. May this storage place receive another value lifetime?
2. May the current value lifetime's contained state change?
3. May this particular access path perform an otherwise available change?

Each question has its own qualifier axis:

| Axis | Stances | Governs |
| --- | --- | --- |
| Place replacement | `final` / `varying` | Whether the qualified storage place may ordinarily receive another value lifetime, and whether a particular declaration may exercise that authority |
| Value mutability | `mutable` / `immutable` | Whether the current value lifetime's contained state may change through ordinary behavior |
| Access capability | `writable` / `readonly` | Whether the current path may perform an otherwise permitted content mutation or place replacement |

These are called **place-replacement qualifiers**, **value-mutability
qualifiers**, and **access qualifiers**.

Begin with omitted value and access defaults:

```zax
source final : Buffer = makeBuffer()

source.append(data)   // legal: mutable and writable are defaulted
source = makeBuffer() // error: source's place is final

reader : Buffer readonly final & = source
reader.append(data)   // error: this path is readonly
```

`final` governs replacement of `source`'s place. It does not prevent mutation of
the defaulted mutable value through the defaulted writable source path.
`readonly` removes change authority from `reader` without changing the value's
mutability or the place's final stance.

## Type-side truth versus declaration-side permission

`final` and `varying` occupy two positions that answer two different questions:

| Position | Meaning |
| --- | --- |
| Type-use side | Whether the underlying place is actually `final` or `varying` |
| Declaration-name side | Whether this declaration or access path may exercise replacement authority |

This is the same shape as the distinction between a mutable value and a readonly
access path. Teach it as **capability versus permission**:

| Kind of change | Underlying capability | Permission through this path |
| --- | --- | --- |
| Mutate the current value lifetime's contents | `mutable` | `writable` |
| Replace the entire value lifetime in place | type-side `varying` | `writable` plus declaration-side `varying` |

The restrictive counterparts are:

| Concern | Underlying restriction | Path restriction |
| --- | --- | --- |
| Content mutation | `immutable` | `readonly` |
| Whole-value replacement | type-side `final` | declaration-side `final` |

`readonly` and `writable` appear in the qualified type use but semantically
describe the current access path rather than the nominal base type. `readonly` is
broad: it removes both content-mutation and whole-value-replacement authority
through that path, so a second declaration-name `readonly` would only repeat the
same fact.

Declaration-side `final` is narrower. It removes only whole-value replacement
authority while preserving writable content mutation when the value is mutable:

```zax
source varying : MyType mutable varying

reader : MyType mutable readonly varying & = source
restricted final : MyType mutable writable varying & = source

reader.member = replacement     // error: readonly path
restricted.member = replacement // legal: mutable value and writable path
restricted = makeMyType()       // error: declaration-side final
```

That is why `final`/`varying` needs both positions while `readonly`/`writable`
does not.

So:

```zax
myType final : MyType varying
```

means:

- the underlying type/place is varying;
- an authorized path may replace its value lifetime in place;
- this declaration promises not to exercise that replacement authority; and
- the declaration may still mutate contents when the value is mutable and the
  path is writable.

### Same-place aliases

A restricted same-place view preserves the underlying truth while narrowing
replacement authority through that declaration:

```zax
myType varying : MyType varying

myAliasType final : MyType varying & = myType
```

`myType` retains replacement access. `myAliasType` cannot initiate replacement,
but its type use still records truthfully that another path may replace the
referent. A same-place alias never misreports the referent's actual stance, and an
omitted type-side stance inherits that actual stance rather than changing it.

### Permitted and rejected combinations

```zax
name final : Type             // type-side omission resolves final for new storage
name final : Type varying     // legal: restrict replacement through this path
name varying : Type varying   // legal: retain replacement access
name final : Type final       // legal
name varying : Type final     // error: access cannot exceed underlying capability
```

Declaration-side `final` may narrow replacement access from varying to final. It
may never widen final storage to varying.

Declaration-side replacement permission must survive aliases, argument mapping,
results, and captures. A declaration-final path cannot regain replacement
authority merely by being supplied to a callable whose referent type remains
varying. See
[Zax function invocation](function-invocation.md#declaration-side-replacement-permission).

### Immutable varying places seen through several paths

```zax
value varying : MyType immutable varying

observer final :
  MyType immutable readonly varying & = value

replacer varying :
  MyType immutable writable varying & = value

restricted final :
  MyType immutable writable varying & = value

replacer = makeMyType()   // reconstructive replacement
restricted = makeMyType() // error: this declaration lacks replacement authority
```

The underlying place can undergo compiler-recognized reconstructive replacement,
ending one immutable lifetime and beginning another. `observer` and `restricted`
must understand that `replacer` may do so. `restricted` remains writable for
operations that do not require whole-value replacement, but its declaration-side
`final` prevents it from initiating reconstructive replacement.

Effective whole-value replacement through one path therefore requires:

- underlying type/place stance `varying`;
- declaration/access replacement permission `varying`;
- writable access;
- a viable replacement operation; and
- for reconstructive replacement, the applicable immutable-lifetime conditions.

### Inspecting the stance

The reserved `is final` query inspects the resolved type-use or referent-place
truth, not this declaration's replacement permission:

```zax
restricted final : MyType varying

restricted is final // false
```

Whether a particular declaration may exercise replacement belongs to future
access or declaration reflection.

### Pointer and reference bindings

Declaration-side `final` restricts whole-value replacement *through that
declaration*. It is not a statement about independently replacing or rebinding
the pointer or reference binding itself. Exact syntax for that separate operation
must be established by focused pointer, reference, and lifetime work rather than
by overloading the same word with two meanings.

## What each axis promises

### `final` and `varying`

Type-side `final` says the qualified place is not ordinarily replaceable.
Type-side `varying` says it may receive another value lifetime when a writable
path with declaration-side replacement permission and a viable replacement
operation are available.

An ordinary alias to the same place preserves that actual type-side stance. A
varying referent may not be presented as final; declaration-side `final` or
`readonly` supplies the access restriction instead.

```zax
config final : Config mutable = makeConfig()

config = makeOtherConfig() // error: config is final
config.refresh()           // legal when this access is writable
```

### `mutable` and `immutable`

`mutable` permits the current value lifetime's contained state to change. It
does not grant every access path permission to perform that change.

`immutable` guarantees recursively that the value and its contained values do
not mutate through ordinary safe behavior after construction completes. A
varying place may later end that immutable lifetime and begin another; that is
place reconstruction, not mutation of either instance.

```zax
value : Document mutable = makeDocument()
reader : Document mutable readonly & = value
```

The value is mutable, but `reader` cannot mutate it.

### `writable` and `readonly`

`writable` permits the current path to perform a change when the applicable place
or value axis and a viable operation also permit it.

`readonly` prevents the current path from mutating the value or reconstructing
the place. It does not freeze the underlying value, make a varying place final,
or suspend independent writable aliases:

```zax
source : Counter
reader : Counter readonly & = source

source.increment() // legal through the original writable path
reader.increment() // error: reader is readonly
```

`reader` remains readonly for its lifetime. Writable access is not recovered
from it when the view ends; `source` retained a separate writable path
throughout.

The axes combine when deciding whether a change is available:

| Operation | Required qualifications |
| --- | --- |
| Mutate the current value lifetime's contents | `mutable` + `writable` |
| Use generated reconstructive replacement for an immutable value | `immutable` + type-side `varying` + declaration-side `varying` + `writable` |
| Observe a stable immutable place | `immutable` + `readonly` + `final` |
| Observe successive immutable lifetimes in one replaceable place | `immutable` + `readonly` + explicit type-side `varying` |

## Varying places with immutable lifetimes

A varying place may hold successive immutable values:

```zax
message varying : Message immutable = makeMessage("first")

observer final :
  Message immutable readonly varying & = message

replacer varying :
  Message immutable writable varying & = message

display(observer)                 // "first"
replacer = makeMessage("second")  // reconstructive replacement
display(observer)                 // "second"
```

Neither immutable instance mutates. The first lifetime ends and another
immutable lifetime begins in the same storage. `replacer` needs
declaration-side `varying` in addition to writable access; a declaration-side
`final` alias of the same place could observe the transition but never initiate
it.

The referent's `varying` stance must be explicit when creating an immutable
place-tracking reference. Omission must not silently introduce this unusual
behavior:

```zax
implicit : Message immutable readonly & = message
// error: varying place tracking must be explicit
```

A final referent view is also unavailable:

```zax
stable : Message immutable readonly final & = message
// error: message's place is varying
```

A future lifetime strategy could temporarily prevent replacement while a stable
borrow exists. Baseline qualifier behavior does not require that alias-tracking
mechanism.

## Qualifier attachment

Qualification follows the entity being constrained:

- a declared binding has its own place qualification;
- a directly stored value has a value-mutability qualification;
- an alias or projection has an access qualification;
- a referenced or pointed-to place has a place qualification independent from
  the binding that stores the reference or pointer; and
- every further pointer or reference level may introduce another independently
  qualified place and access path.

The syntax must therefore distinguish a final pointer binding from a pointer to
a final place. A varying pointer may point to a final pointer that in turn points
to another final place. Replacing the outer pointer replaces neither inner
place.

The exact grammar for multiple pointer and reference levels remains later
indirection design. Whatever spelling is selected must preserve these distinct
layers.

## Defaults and resolution

The baseline omission defaults are:

- place replacement: `varying`;
- value mutability: `mutable`; and
- access capability: `writable`.

Each default may be changed for an applicable source context. Exact directive
syntax, lexical scope, stacking, and precedence between future default sources
belong to compiler-directive design.

Qualifiers resolve independently at every applicable layer:

1. preserve inherent or inherited qualifications supplied by the source,
   referent, or resolved type;
2. apply explicit qualifications at their stated layers;
3. apply contextual defaults only to still-unresolved axes; and
4. diagnose contradictions in the resulting combination.

A default never overrides a qualification carried by the source:

```zax
source final : Payload immutable = makePayload()

sameView : Payload immutable final & = source
reader : Payload readonly final & = source
writer : Payload writable final & = source // error
```

If `immutable` and `writable` are configured simultaneously as defaults, the
configuration itself is legal. A declaration to which both become applicable is
contradictory and fails at that declaration. This allows a project to force an
explicit choice in selected contexts.

A possible future mode with no fallback for an axis is deferred. Baseline Zax
provides sensible defaults.

## Restatement, duplication, and conflict

Compatible qualification may reach one entity from distinct sources:

```zax
source final : Payload immutable = makePayload()
reader : Payload immutable final & = source
```

The explicit `immutable` on `reader` restates the qualification inherited from
`source`. This is legal and may be useful for aliases, generic substitution, or
deliberate clarity.

The same word may also appear at multiple pointer or reference levels because it
qualifies different entities.

Repeating one qualifier token at one syntactic point is an error:

```zax
value final final : Payload = makePayload() // error
```

Opposing stances applied to the same entity and layer are also errors. A
diagnostic must identify the entity and layer rather than treating identical
words at distinct indirection levels as duplicates.

## Canonical ordering

On the declaration-name side, canonical ordering is:

1. place replacement;
2. `unsafe pliable` or explicit `unpliable`; and
3. the declaration colon.

On the type-use side, read outward from the base type:

1. base type;
2. value mutability;
3. access capability;
4. referent-place replacement; and
5. optional, pointer, or reference marker;
6. then the independently qualified outer layer, when another layer follows.

```zax
source final : Payload immutable = makePayload()
view final unpliable : Payload immutable readonly final & = source
escape final unsafe pliable : Payload immutable readonly final & = source
```

A formatter may normalize ordering, spacing, and layout. It must not add or
remove explicit qualifiers. A linter may identify compatible restatement or an
explicit default as redundant without changing source.

## Ordinary promise strengthening

Ordinary views may narrow authority and strengthen the promise made by the new
path:

- `writable` to `readonly`;
- declaration-side `varying` to declaration-side `final`;
- a mutable value to a readonly path; and
- an immutable value to a readonly path.

The reverse directions weaken promises and are not ordinary conversions:

- `readonly` to `writable`;
- declaration-side `final` to declaration-side `varying`; and
- `immutable` to `mutable`.

Mutable or readonly state does not thereby become immutable. Readonly constrains
one path; immutable makes a guarantee about one value lifetime.

Type-side final/varying is not an ordinary same-place authority conversion. An
alias preserves the actual referent-place stance, even when its own
declaration-side permission is narrower. A new independent destination place
resolves its own stance:

```zax
source varying : Payload = makePayload()
copy final : Payload = source
```

`copy` is a new final place; `source` remains varying.

## Unsafe casts and new values

An explicit unsafe cast may weaken qualification, including producing mutable
or writable pointer/reference access from immutable or readonly input. The
complete cast lattice and exact cast syntax remain later casting design.

An ordinary mutable alias cannot safely become an immutable alias. Other mutable
aliases may still change the same value. Mutable-to-readonly access is safe
because it constrains only the new path.

A new immutable value may be produced safely from mutable input when the
operation establishes the complete deep-immutability guarantee. This may
involve:

- constructing an independent immutable value;
- making a sufficiently independent copy; or
- consuming mutable storage for which the compiler proves exclusive authority
  over all storage covered by the immutable guarantee.

A consuming transition ends the former mutable capability. A scoped
non-mutating view that later permits mutation again is readonly, not temporarily
immutable. Exact uniqueness proofs, freezing operations, copy selection, cost
visibility, and syntax remain future work.

## Construction and deep immutability

Immutability is recursively deep over direct structural containment. Semantic
indirection boundaries retain independently stated qualifications. Pointer,
reference, and optional layers therefore do not silently rewrite the
qualifications of their pointee, referent, or boxed value:

```zax
stablePresence :
  Payload mutable writable? immutable readonly
```

The optional wrapper's presence is immutable and readonly through this path.
After proven access, the boxed `Payload` has its separately written
mutable/writable qualifications. The compiler may rely on stable wrapper state
but not stable payload bytes.

This independence does not remove lifetime ownership. An optional wrapper owns
its boxed lifetime and may end it through an otherwise authorized wrapper
operation. Complete behavior is defined by
[Zax optional values](optional-values.md#wrapper-and-boxed-qualifications).

Construction has authority to establish final and immutable state without an
unsafe bypass. The guarantees activate when the full instance, including all
contained parts, has completed construction and becomes ordinarily observable.

If construction does not complete, no fully constructed instance becomes live.
Member sequencing, completion, panic boundaries, premature publication, and
escaped construction-time aliases are defined or bounded by
[Zax construction, replacement, and destruction](construction-and-destruction.md).

## Reconstructive replacement

`=` has one compiler-recognized lifetime scenario in addition to arbitrary
domain-specific operator candidates. When an existing destination is immutable
and type-side varying, the current declaration has declaration-side varying
replacement permission, and the current path is writable, the compiler may select
a generated **reconstructive replacement** candidate.

A mutable, varying destination uses ordinary operator selection. It does not
receive this generated immutable-value lifecycle transition merely because its
place is varying.

The generated candidate ends one enclosing immutable value lifetime and
establishes another in the same storage. Its lifecycle skeleton is
compiler-owned and cannot be replaced by an ordinary user-defined `=` body. A
type customizes the transition with a
[replacement constructor](terms.md#replacement-constructor):

```zax
replacement +++ final : ()(
  rhs : Input readonly &
) = {
  // `_` initially contains the previous current-instance state.
}
```

`replacement` is contextual syntax only when it immediately precedes `+++`
where a constructor declaration is legal. Elsewhere it remains an ordinary
identifier:

```zax
replacement := makeValue()
value.replacement()

Example :: type {
  replacement : String
}
```

The replacement constructor:

- uses the destination's existing allocation;
- receives `_` with the old representation and resources;
- has transitional mutable and writable construction authority without
  `unsafe pliable`;
- may retain, destroy, replace, move, copy, or initialize members in place;
- selects its declared right-hand operand through ordinary parameter and
  overload rules;
- may return additional declared results; and
- must establish a complete valid instance before returning normally.

An untouched resource may remain in its existing field and allocation. Reusing
an address does not automatically preserve raw interior pointers when a
contained pointee lifetime ends or is reconstructed.

This document owns why reconstructive replacement is required by the qualifier
model and its qualification boundary. Complete fallback, member transition,
resource retention, result, destruction, and alias behavior is defined by
[Zax construction, replacement, and destruction](construction-and-destruction.md#reconstructive-replacement).
Complete move/copy/`last` per-slot preference, recoverable panic, callbacks,
reentrancy, async, and concurrency remain future focused work.

If panic is terminal, partially transitioned storage does not return to ordinary
execution. Any future recoverable panic model must define separate partial
cleanup behavior.

## Type-family boundary

This qualifier model does not decide whether one named type may provide distinct
mutable and immutable implementations. Such variants could have different
layouts, stored state, operations, and construction strategies.

If a future type-family design adopts that capability:

- readonly access retains the selected variant's underlying mutability;
- a mutable readonly variant remains distinct from an immutable readonly
  variant;
- differently represented variants cannot be ordinary requalified views; and
- conversion requires construction, copying, or a consuming transformation that
  establishes the destination representation and guarantees.

Exact family identity, shared APIs, defaults, self-reference inside family
variants, conversion, reflection, structural compatibility, and syntax remain
future type-definition and structural-typing work.

## `unsafe pliable`

`unsafe pliable` creates an explicit access path that locally ignores final,
readonly, and immutable restrictions while retaining the underlying
qualifications.

```zax
source final : LegacyValue immutable = makeLegacyValue()
escape final unsafe pliable :
  LegacyValue immutable readonly final & = source
```

`unsafe pliable` also bypasses the declaring binding's own final stance for
operations through that path. The final, readonly, and immutable qualifications
remain recorded, but none is exempt from the explicit local bypass.

This escape supports narrow cases such as mutex use, hidden caches, lazy
bookkeeping, and legacy integration. It must be used with extreme caution:
other aliases may continue relying on the retained qualifications.

### `unpliable`

`unpliable` is ordinary behavior. An unpliable path respects all retained
qualifications. The word may be written explicitly but is normally omitted.

`unsafe pliable` cannot be selected through an omission default or a default
directive. Its purpose requires a local `unsafe` marker. Because `unpliable` is
the only ordinary behavior, no directive is needed to select it.

### Difference from an unsafe cast

An unsafe cast produces a differently qualified view. `unsafe pliable` retains
the original qualifications and ignores them locally:

```zax
source final : Payload readonly = makePayload()
escape unsafe pliable : Payload readonly final & = source

explicitView unpliable : Payload readonly final & = escape
ordinaryView : Payload readonly final & = escape
```

Both new views respect the retained readonly qualification.

### Non-propagation

Pliability is removed at every new boundary unless explicitly reintroduced:

- assigning or aliasing from an `unsafe pliable` source does not make the
  destination `unsafe pliable`;
- dereferencing does not carry pliability into another pointer or reference
  level;
- an operator may use an `unsafe pliable` operand's bypass for the current
  operation, but its result receives only the selected candidate's declared
  qualifications;
- function results cannot carry pliability as a returned property; and
- captures preserve the source qualifications but do not inherit source
  pliability.

An explicit capture may create a new `unsafe pliable` capture binding. The
capture clause is then the visible unsafe introduction site. Exact capture
syntax remains function design.

### Local aliases and members

A local alias can minimize the unsafe region:

```zax
useValue final : ()(input : LegacyValue immutable) = {
  local unsafe pliable : LegacyValue immutable & = input
  local.callMutableFunction()
}
```

A member may advertise an interior-mutation escape:

```zax
CacheOwner :: type {
  cache unsafe pliable : LegacyCache
}
```

`mutable` on a member describes that member value's mutability. It does not
authorize mutation through a readonly or immutable container path. Such a bypass
must be written `unsafe pliable`.

### Parameters

A parameter may introduce a local bypass:

```zax
useValue final : ()(
  input unsafe pliable : LegacyValue immutable
) = {
  input.callMutableFunction()
}
```

Candidate selection uses the parameter's declared `LegacyValue immutable`
requirement. After binding, the parameter path gains the explicit unsafe bypass.
The caller need not pass an `unsafe pliable` argument, and the bypass does not
cascade through every source-level caller.

A function may instead accept writable access, with its caller creating a
localized `unsafe pliable` alias before the call. A local alias minimizes the
unsafe region; an `unsafe pliable` parameter advertises a function-wide argument
escape at the declaration boundary.

Compiler and tooling analysis must preserve the possibility of mutation through
transitive aliases. It may track an internal effect without requiring every
source-level caller to repeat `unsafe pliable`.

### Safety and optimization

`unsafe pliable` has defined mechanical semantics: the compiler must preserve
the requested access and mutation. The operation is not itself undefined
behavior.

The programmer assumes responsibility for consequences. Broken invariants, data
races, invalid states, lifetime violations, or other misuse may produce
undefined behavior. Debug tooling may insert checks and panic when it detects a
problem, but such instrumentation is not a language guarantee.

An `unsafe pliable` escape is an optimization invalidation boundary. A compiler
may rely on immutable stability only where it proves that no relevant unsafe
escape may mutate the storage. Ordinary aliases remain typed immutable, but
their observed values cannot be assumed unchanged across a potentially aliasing
unsafe mutation.

Language documentation owns this programmer-visible requirement, not any
particular implementation or backend mapping.

## Parameters, results, and captures

Parameters resolve all three axes at the layers expressed by their prototypes.
Ordinary call matching may preserve or strengthen promises but may not silently
weaken an argument's qualifications.

Function results are governed by the resolved prototype:

- a returned pointer or reference carries the prototype's per-level
  qualifications;
- a returned by-value value carries its declared value mutability; and
- the caller's destination introduces an independently resolved place.

An inferred or generic result ultimately resolves to such a prototype. Complete
generic result resolution remains later function design.

Captures preserve or strengthen the captured source qualifications and never
inherit pliability implicitly.

Declaration-side replacement permission survives parameter binding, result
routing, and capture. A declaration-final argument cannot regain replacement
authority merely because the parameter's referent type use remains varying:

```zax
replaceThrough final : ()(target varying : MyType immutable writable varying &)

restricted final : MyType immutable writable varying & = value
replaceThrough(restricted) // error: restricted may not delegate replacement authority
```

The invocation owner applies these qualification truths to parameter binding,
result routing, expected-result matching, compatible prototype adaptation, and
partial-order candidate preference. See
[Zax function invocation](function-invocation.md#declaration-side-replacement-permission).

Changing the labels, defaults, or result-acknowledgement policy of a compatible
visible prototype does not permit its ordered implementation slots to gain
authority or require qualification-changing executable adaptation.

## Receiver operands

The [receiver operand](terms.md#receiver-operand) is the implicit operand
representing the instance on which a type-defined function or operator acts.
All three qualifier axes may constrain it and participate in candidate
selection.

No operator, including `=` or `+=`, receives conventional qualifier behavior
merely because of traditional meaning. The reconstructive `=` scenario is
special only because the compiler recognizes an immutable value in an existing
varying place through a writable path. A domain-specific operator may still
accept a final, readonly, or immutable receiver operand when its declaration is
compatible.

```zax
MyType :: type {
  x : Integer

  operator binary '=' final :
    (result : Boolean)(rhs : String) writable final = {
    _.x = 42
    return true
  }
}
```

`_` has the declared receiver-operand qualifications like any other qualified
place. `final` does not prohibit an operator named `=`. An operation involving
`_` selects only candidates compatible with its final stance. Likewise,
readonly prevents selection only of a candidate requiring writable access
through that operand; it does not prohibit arbitrary effects elsewhere.

The generated reconstructive candidate requires immutable, type-side varying,
declaration-side varying, and writable. It is unavailable for a mutable value,
through a readonly receiver operand, or through a declaration-side final path
even when the underlying place is varying.

A temporary supplies the qualifications of its resolved result and
compiler-managed temporary place. An operation through a pointer or reference
uses the dereferenced instance's qualifications, not the pointer binding's place
qualification.

Omitted receiver-operand qualifiers use the defaults applicable where the
operation is defined. Complete generated operator families, temporary destruction
beyond synchronous call completion, and default-source precedence remain future
work.

Ordinary member-call evaluation, fixed-arity viability, and ambiguity are
defined by [Zax function invocation](function-invocation.md). Operator-specific
lookup and selection are defined by [Zax operators](operators.md).

### Mixfix receiver anchors

A type-defined mixfix belongs to one receiver anchor. That receiver and every
operand hole carry ordinary place, value, access, and indirection
qualifications:

```zax
// Illustrative mixfix declaration syntax.
operator mixfix
  index 1
  binary '='
final : (
  result : MyResult
)(
  indexValue : MyIndex,
  rhs : MyValue
) writable = {
}
```

A writable declaration is nonviable through readonly access. Mutation still
requires mutable value state plus writable access. A final place may permit
content mutation but not ordinary place replacement.

Mixfix punctuation grants no authority. A user mixfix that consumes `=` does not
acquire the compiler-owned reconstructive-replacement lifecycle skeleton.
Complete tree matching and protected barriers are defined by
[mixfix operators](mixfix-operators.md).

## Optional qualification layers

`?` forms an outward qualification layer:

```zax
T q0 ? q1 * q2
```

This reads as a `q2` pointer to a `q1` optional wrapper containing a `q0` boxed
`T`. Qualifiers before `?` describe the boxed layer; qualifiers after `?`
describe the wrapper layer:

```zax
innerReadonly : Payload readonly?
wrapperReadonly : Payload? readonly
```

`innerReadonly` may change wrapper state through otherwise mutable/writable
wrapper defaults, but postfix access produces readonly `Payload` access.
`wrapperReadonly` cannot change state through this wrapper path, but the outer
readonly qualifier does not rewrite the boxed qualifications.

Postfix optional access crosses the wrapper boundary. Once presence is proven,
the result carries the boxed qualifications rather than the wrapper
qualifications:

```zax
if ?innerReadonly
  inspect(innerReadonly.) // readonly Payload access
```

### Wrapper axes

The three qualifier axes remain independent:

| Axis | Optional-wrapper consequence |
| --- | --- |
| `mutable` / `immutable` | Whether presence may change during this wrapper lifetime |
| `writable` / `readonly` | Whether this path may perform an otherwise available wrapper change |
| Type-side `varying` / `final` | Whether the place may receive another complete optional-wrapper lifetime |

Declaration-side `varying`/`final` separately says whether this declaration may
exercise complete-wrapper replacement.

A final mutable wrapper may reset and construct successive boxed lifetimes while
retaining one wrapper lifetime:

```zax
optional final : Payload? mutable final = value

reset optional
reset optional
optional = [{}]
optional = anotherOptional // error: complete wrapper place is final
```

An immutable varying wrapper cannot change presence within its current lifetime,
but an authorized writable varying path may replace that complete wrapper:

```zax
optional varying :
  Payload? immutable writable varying = first

reset optional    // error: current wrapper is immutable
optional = [{}]   // error: current wrapper is immutable
optional = second // replace the complete wrapper lifetime
```

Boxed `readonly`, `immutable`, or `final` does not prevent the mutable/writable
wrapper owner from ending that conditional lifetime:

```zax
optional : Payload immutable readonly final? mutable writable

reset optional
optional = [{}]
```

Those operations remove and create boxed places. They do not mutate or
independently replace the old payload through postfix access.

Complete optional operations, nested optional layers, and transfer effects are
defined by [Zax optional values](optional-values.md).

## Indirection

Dereferencing does not change the qualifications of the dereferenced instance.
Converting between pointer and reference forms preserves pointee-level
qualifications.

An ordinary alias to the same referent preserves that place's type-side
final/varying stance, though it may narrow its own declaration-side replacement
permission. Changing indirection form cannot present a varying referent as final
or a final referent as varying.

The declaration-side permission of the pointer or reference binding constrains
replacement through that declaration. Independently replacing or rebinding the
pointer or reference binding itself is a separate operation whose exact syntax
remains later indirection design.

Changing indirection form may not silently strip referent qualifications or
increase ordinary access authority. Complete pointer/reference grammar,
rebinding, ownership, and alias analysis remain later work.

## Members and nested access

A readonly path to a container ordinarily produces readonly paths to projected
members:

```zax
source final : Container
view : Container readonly final & = source

view.member = 5 // error
source.member = 5 // legal through the original writable path
```

A member requiring mutation through readonly or immutable access must introduce
explicit `unsafe pliable`.

Postfix optional access is not ordinary member projection. It crosses the
semantic `?` indirection boundary and therefore exposes the independently
qualified boxed layer as described above.

## Move-out and destruction

A final live place cannot be ordinarily replaced. Move and copy are contextual
operations rather than persistent qualifier axes.

A final place may permit destructive move-out when the operation truly consumes
the source as part of ending its lifetime. This supports reclaiming or recycling
expensive resources. It does not permit transfer from a still-live readonly or
immutable source into mutable or writable state without an explicit unsafe
operation.

During destruction, the current instance receives terminal mutable and writable
authority without requiring `unsafe pliable`. Destruction may dismantle the
value and extract resources because its lifetime is ending. That authority
cannot create an alias usable after the instance's lifetime.

Destructor sequencing is defined by
[Zax construction, replacement, and destruction](construction-and-destruction.md#destruction).
Moved-from states and proof of terminal consumption remain lifetime and
ownership work.

## Concurrency boundary

Immutable qualification addresses value change, not lifetime, shared backing
storage, synchronization, safe transfer, allocation, or reference-count safety.
It may make sharing easier but is not a complete thread-safety guarantee.

Lifetime management must independently ensure that an instance outlives every
reference to it. Mutable shared state requires synchronization. An
`unsafe pliable` path can additionally invalidate assumptions made by other
aliases and requires particular caution across threads.

## Structural-typing boundary

Qualifier-preserving conversion and structural identity are different
questions. Conversion may not increase ordinary authority. Future
structural-typing work must decide separately whether qualifiers participate in:

- nominal identity;
- name-and-type shape compatibility;
- qualifier-aware layout compatibility;
- qualifier-erased layout compatibility;
- reflection; and
- safe substitutability at a particular use site.

Zax should eventually provide an explicit anonymization operation for
intentional use of compatible differently named structures. Anonymization may
not become a qualifier-elevation mechanism. Exact syntax and equivalence rules
remain deferred.

## Diagnostics and formatting

Diagnostics should:

- identify the entity and indirection level carrying each qualification;
- distinguish underlying capability from permission through the current
  declaration or path;
- distinguish unavailable authority from a conflicting qualification;
- reject duplicate qualifier tokens at one syntactic point;
- reject a declaration-side stance that would exceed the underlying capability;
- report contradictions where contextual defaults become concrete;
- explain required and available receiver-operand qualifications when no
  candidate is viable; and
- distinguish an unsafe bypass from ordinary promise strengthening;
- show the qualification layer that makes an invocation or result transfer
  unavailable; and
- distinguish an incomparable transfer from a strictly worse match.

Canonical ordering is a formatting rule, not semantic precedence. Formatters
preserve explicit qualification. Linters may identify compatible restatement or
explicit defaults as redundant.

## Costs and risks

The qualifier model makes useful promises visible, but those promises have
costs:

- additional indirection levels require additional qualification decisions;
- configurable defaults make omitted source context-dependent;
- deep immutable construction may require copying, exclusive authority, or
  specialized construction;
- readonly aliases do not prevent mutation through other paths;
- explicitly varying references may observe successive value lifetimes in one
  place;
- immutable reconstructive replacement introduces constructor, lifetime, and
  alias constraints;
- `unsafe pliable` can invalidate invariants and optimization assumptions relied
  upon elsewhere; and
- qualifier-sensitive overload selection can make candidate sets more complex.

These costs are reasons to keep defaults sensible and unsafe escape paths local,
not reasons to combine the three axes.

## Boundaries and maturity

This document is accepted conceptual design, not a formal grammar,
implementation mapping, compatibility contract, or conformance specification.

Later work may refine syntax and adjacent mechanisms while preserving:

- the independence of place, value, and access qualification;
- the separation of type-side capability from declaration-side permission;
- per-layer attachment through indirection;
- defaults filling only unresolved axes;
- ordinary conversions never increasing authority;
- same-place aliases preserving the referent's actual type-side final/varying
  stance while being free to narrow their own replacement permission;
- mutation requiring mutable + writable;
- generated reconstructive replacement requiring immutable + type-side varying +
  declaration-side varying + writable;
- deep immutability across direct structural containment while optional,
  pointer, and reference indirection layers retain independently written
  qualifications;
- optional wrapper mutability governing presence, independently from boxed
  qualification and complete-wrapper place replacement;
- explicit, non-propagating unsafe pliability;
- qualifier-aware receiver-operand selection;
- function invocation applying qualification truth without silently increasing
  authority; and
- the separation of mutability from lifetime and thread safety.

Independent replacement or rebinding of a pointer or reference binding itself
remains future pointer, reference, and lifetime work.
