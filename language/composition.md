# Zax composition

| Field | Value |
| --- | --- |
| Status | Current conceptual design |
| Audience | Human developers building types from stored components and choosing which component surfaces the container presents |
| Applies To | Named containment; independent `own`, `preferred`, and `expose`; semantic-indirection boundaries; published data paths; singular and family composition routing and filtering; abstract roles and fulfillment; outer casting and exact-origin proof; costs, diagnostics, formatting, and source stability; not a formal grammar or specification |
| Implementation State | Not established by this repository |
| Owns | The complete programmer-facing composition model; data publication and collisions; expected-type projection; generated behavior exposure and unchanged results; singular and family `via` and `existing`; exact and outer-family fences; composition-specific mapping eligibility; `abstract`, `abstract relaxed`, and `fulfill`; the shared mechanical filter used by identity exposure; `outer`, `outer tracked`, `outer cast`, `unsafe outer cast`, and composition-specific exact-origin proof |
| Does Not Own | Ordinary declarations and member lookup ([declarations and bindings](declarations-and-bindings.md)); callable selection and compatible visible prototypes ([function invocation](function-invocation.md)); shared operator discovery and selection ([operators](operators.md)); qualification meaning ([qualifiers](qualifiers.md)); general transfer semantics ([transfer stances](transfer-stances.md)); reference origin and lifetime ([lifetimes and references](lifetimes-and-references.md)); ordinary lifecycle behavior ([construction and destruction](construction-and-destruction.md)); identity admission and projection ([identity types](identity-types.md)); pointer ownership ([pointers and arenas](pointers-and-arenas.md)); or the reusable unsafe model ([safety and analysis](safety-and-analysis.md)) |
| Source / Provenance | Legacy composition intent, reconciled with current declaration, invocation, operator, transfer, lifetime, construction, identity, and safety design |
| Supersedes | Legacy composition design formerly published at the repository root |

## Mental model

Ordinary named containment is the foundation of Zax composition:

```zax
Engine :: type {
  rpm : Integer

  start final : ()() = {
  }
}

Car :: type {
  engine : Engine
  wheels : Integer
}

prepare final : ()(car : Car mutable writable &) = {
  car.engine.rpm = 2000
  car.engine.start()
  car.wheels = 4
}
```

`Car` contains an `Engine`. The stored member has its own name, place, value,
and lifecycle. Nothing implies that a `Car` *is an* `Engine`.

The `own` word can make selected data paths more convenient:

```zax
Car :: type {
  engine own : Engine
  wheels : Integer
}

prepare final : ()(car : Car mutable writable &) = {
  car.engine.rpm = 2000 // physical path
  car.rpm = 2000        // published path to the same place
}
```

`car.rpm` does not name another declaration or another integer. It is a
published access path to `car.engine.rpm`. Both paths have the same storage,
identity, lifetime, qualifications, and access cost.

Composition offers three independent choices:

1. Should stored data names be available by a shorter path? Use `own`.
2. Should an outer value supply this member when that member type is already
   expected? Use `preferred`.
3. Should mechanically safe member behavior appear on the outer type? Use
   `expose`.

Each word can stand alone:

```zax
Assembly :: type {
  publicData own : PublicData
  engine preferred : Engine
  controls expose : Controls
}
```

`Assembly` publishes names from `publicData`, may project to `engine` in an
`Engine`-typed context, and exposes eligible `controls` behavior. None of those
choices silently enables either of the others. In particular, `engine` remains
available as a physically named member rather than publishing its data, and
`controls` behavior can be called through `Assembly` without publishing
`controls` data.

Any subset may be combined:

| Form | Selected conveniences |
| --- | --- |
| `member own : Type` | Stored instance-member names |
| `member preferred : Type` | Expected-type projection |
| `member expose : Type` | Mechanically eligible instance behavior |
| `member own preferred expose : Type` | All three |

When words are combined, their canonical relative order is
`own preferred expose`. Placement of this modifier group among unrelated
declaration words awaits the general grammar and formatting sweep.

Examples below reuse short type names such as `Engine` and `Car` independently.
Each block supplies the declarations relevant to that example rather than
extending every earlier block.

These conveniences remain composition:

- physical member paths always survive;
- published data creates no storage;
- generated behavior is statically selected forwarding, not virtual dispatch;
- expected-type projection is one-way and context-bounded;
- no form creates a class hierarchy, vtable, or hidden outer reference; and
- ambiguity is diagnosed rather than resolved by source order.

## Publishing stored data with `own`

### What is published

Bare `own` publishes the contained type's stored instance-member names:

```zax
Address :: type {
  city : String
  postalCode : String
}

Customer :: type {
  address own : Address
}

printDestination final : ()(customer : Customer readonly &) = {
  print(customer.city)
  print(customer.address.postalCode)
}
```

It does not publish:

- instance functions;
- operators;
- nested types;
- aliases;
- constants; or
- `once` declarations.

Behavior crosses a composition boundary only when the boundary explicitly uses
`expose` or the container declares a forwarding operation.

`own` is not a local-variable or flow-scope projection facility. The separate
`shadowable` word remains a one-level lexical-name permission and has no effect
on publication, precedence, or collisions. See
[declarations and bindings](declarations-and-bindings.md#shadowing).

### Direct names and collisions

A stored member declared directly in the container owns its short name:

```zax
Payload :: type {
  id : PayloadId
  bytes : Buffer
}

Envelope :: type {
  payload own : Payload
  id : EnvelopeId
}

inspect final : ()(envelope : Envelope readonly &) = {
  use(envelope.id)         // Envelope.id
  use(envelope.payload.id) // Payload.id
  use(envelope.bytes)      // Payload.bytes
}
```

The direct `Envelope.id` prevents publication of the short `payload.id` path.
It does not hide or alter the physical member.

Two owned paths publishing the same name make that short name ambiguous:

```zax
Reading :: type {
  value : Float
  timestamp : Integer
}

Sensor :: type {
  primary own : Reading
  backup own : Reading
}

inspect final : ()(sensor : Sensor readonly &) = {
  use(sensor.primary.value)
  use(sensor.backup.value)
  use(sensor.value) // error: two owned paths publish `value`
}
```

There is no first-declaration or shortest-path winner.

### Selecting and renaming data with `via`

A data `via` declaration creates a no-storage route to one exact stored path:

```zax
Sensor :: type {
  primary own : Reading
  backup own : Reading

  value via primary.value
  age via backup.timestamp
}

inspect final : ()(sensor : Sensor readonly &) = {
  use(sensor.value)             // primary.value
  use(sensor.age)               // backup.timestamp
  use(sensor.primary.timestamp) // physical path still available
}
```

The route may preserve the target name or give it a new name. It retains the
target place's type truth, qualification, lifetime, transfer stance, and access.
It cannot widen authority or create storage.

A direct `via` route owns its declared outer name just like a direct stored
member. It therefore selects that name over any automatically published short
path. Two direct routes, or a route and another direct declaration, cannot claim
the same outer name; that is a declaration error.

### Visibility follows the complete physical path

A private member of the contained type never enters the `own`-published set. A
published route also cannot be more visible than any physical segment it
abbreviates. A public inner member behind a private carrier therefore does not
acquire a public short path.

`own`, `expose`, and `via` grant no friendship and no private access. A `via`
target must already be accessible from the declaration's context.

A private carrier may nevertheless use `own` to activate its
[abstract roles](#abstract-roles-and-explicit-fulfillment). That does not
publish the carrier or its fields: the immediate container may inspect its own
private path, and only the container's explicitly written `fulfill`
declarations acquire whatever visibility they declare.

### Nested composition

Data publication follows owned data paths transitively:

```zax
Coordinates :: type {
  x : Float
  y : Float
}

Position :: type {
  coordinates own : Coordinates
}

Sprite :: type {
  position own : Position
}

moveToOrigin final : ()(sprite : Sprite mutable writable &) = {
  sprite.x = 0
  sprite.position.y = 0
  sprite.position.coordinates.x = 0
}
```

Every physical path survives. Collisions at any depth remain ambiguous.
Behavior is different: it crosses only boundaries that independently say
`expose`.

### Publication stops at semantic indirection

Recursive publication follows resident by-value containment. It does not
silently enter a pointer, reference, optional, variant, or another wrapper whose
payload has separate presence, selection, alias, or lifetime meaning.

```zax
Coordinates :: type {
  x : Float
  y : Float
}

Position :: type {
  coordinates : Coordinates * = @
}

Sprite :: type {
  position own : Position
}

draw final : ()(sprite : Sprite readonly &) = {
  use(sprite.coordinates) // published path to the stored pointer
  use(sprite.x)           // error: publication does not enter the pointee

  if !sprite.position.coordinates
    return

  use(sprite.position.coordinates.x) // presence and dereference stay explicit
}
```

`Sprite.position` is resident, so `own` publishes its stored `coordinates`
member. That member's value is a pointer. The allocated `Coordinates` is not
resident inside either `Position` or `Sprite`, even when `@` allocated it during
initialization. Consequently, `own` does not manufacture `sprite.x`, and
ordinary outer casting cannot travel from the pointee back to the object that
stores its pointer.

The same principle applies to the other modifiers. `preferred` selects the
declared member value when that exact value type is expected. `expose` considers
behavior of the declared pointer, optional, variant, or wrapper value; it does
not synthesize delegation to a present or selected payload. A future indirect
delegation feature would need to expose its presence, aliasing, lifetime, and
runtime costs explicitly rather than weakening this boundary.

### Construction remains whole-member construction

Published fields are not flattened into the container's construction shape:

```zax
AnimalData :: type {
  animal : String = "bear"
}

Container :: type {
  data own : AnimalData
}

container : Container = [{
  .animal = "mouse"
  // error: `animal` is a published path, not a direct Container member
}]
```

The container can supply constructor arguments or a complete value for its
direct `data` member:

```zax
container : Container = [{
  .data = makeMouseData()
}]
```

The `.data` entry suppresses default initialization for that direct member.
`makeMouseData()` is evaluated in packet order, and its result constructs
`data` once when member-order construction reaches that place. This is not
default construction followed by assignment. Selected member initialization
finishes before the enclosing constructor body begins.

It cannot initialize `data.animal` as though `animal` were declared directly in
`Container`. The `AnimalData` value must be established by its own selected
constructor. A constructor body that also calls `_.data.+++()` conflicts with
the packet entry because both request construction of the same place:

```zax
Container :: type {
  data own : AnimalData

  +++ final : ()(data : AnimalData) = [{
    .data = data
  }] {
    _.data.+++() // error: `.data = data` already constructs this member
  }
}
```

Construction packets, constructors, replacement, and destruction continue to
target the physical stored shape described by
[construction and destruction](construction-and-destruction.md).

## Expected-type projection with `preferred`

`preferred` lets an outer value supply one member when a destination type is
already known:

```zax
Car :: type {
  engine preferred : Engine
  wheels : Integer
}

tune final : ()(target : Engine readonly &) = {
}

service final : ()() = {
  car : Car

  tune(car)        // projects car.engine
  tune(car.engine) // explicit path remains available
}
```

Preferred projection participates in:

- call-input binding;
- explicitly typed initialization; and
- other typed binding where the destination is already established.

It does not perform open-ended member, receiver, or operator discovery. The
direction is outer to member only. An exact type match beats projection.

A reference projection preserves the physical member path, lifetime, and
qualifications. A by-value projection constructs only the selected member value
under the applicable stance rather than pretending the outer value and member
are identical.

`own` and `preferred` can therefore select different aspects of the same
physical relationship:

```zax
PublishedCar :: type {
  engine own : Engine
}

ProjectedCar :: type {
  engine preferred : Engine
}

compare final : ()(
  published : PublishedCar readonly &,
  projected : ProjectedCar readonly &
) = {
  use(published.rpm) // data publication
  tune(published)    // error: `own` creates no Engine projection

  use(projected.rpm) // error: `preferred` publishes no data
  tune(projected)    // projects projected.engine
}
```

### More than one preferred member

Several preferred members with different target types are legal:

```zax
Rig :: type {
  engine preferred : Engine
  radio preferred : Radio
}

configure final : ()(target : Engine readonly &) = {
}

configure final : ()(target : Radio readonly &) = {
}

configureRig final : ()(rig : Rig readonly &) = {
  configure(rig)        // error: both projected overloads are viable
  configure(rig.engine) // explicit path resolves the choice
}
```

Qualification-distinct routes to the same base type may also coexist. Ordinary
viability and partial-order comparison can select one when its complete
capabilities uniquely fit. Cross-axis or nondominated overlap remains a
use-site ambiguity; projection introduces no scoring system.

```zax
BufferViews :: type {
  snapshot preferred : Buffer immutable
  working preferred : Buffer mutable
}

archive final : ()(input : Buffer immutable &) = {
}

edit final : ()(input : Buffer mutable writable &) = {
}

useViews final : ()(views : BufferViews mutable writable &) = {
  archive(views) // selects snapshot
  edit(views)    // selects working
}
```

The routes are distinguishable because the expected complete types require
different capabilities. If two routes remain equally viable after ordinary
qualification comparison, the use is ambiguous.

Two direct preferred routes with the same normalized type and same complete
defaulted shape are indistinguishable. They are rejected when the containing
type completes rather than waiting for a use:

```zax
TwinBuffer :: type {
  first preferred : Buffer
  second preferred : Buffer
  // error: indistinguishable direct preferred projections
}
```

Transparent aliases do not make the routes distinct.

### Nested and operator projection

Every boundary in a nested projected route must independently use `preferred`.
Converging nested routes remain ambiguous; path length and declaration order do
not rank them.

```zax
Car :: type {
  engine preferred : Engine
}

FleetEntry :: type {
  car preferred : Car
}

tuneEntry final : ()(entry : FleetEntry readonly &) = {
  tune(entry) // FleetEntry -> Car -> Engine
}
```

Removing `preferred` from either `FleetEntry.car` or `Car.engine` makes that
projected call unavailable. An `own` on either boundary may shorten a data path,
but it does not repair the missing expected-type route.

Operator discovery completes before preferred projection participates. Once a
discovered candidate provides an expected operand type, the operand may project
through a preferred member. `preferred` never:

- discovers an operator through the outer receiver;
- widens a candidate tree;
- swaps operands;
- rescans source; or
- beats a direct operand match.

Generic deduction and structural matching do not currently acquire targets from
preferred routes. Their future integration must begin with an independently
established expected type or shape.

## Exposing behavior

`expose` asks the compiler to generate thin forwarding for mechanically eligible
instance functions and operators:

```zax
Engine :: type {
  start final : ()() = {
  }

  isRunning final : (result : Boolean)() readonly = {
  }
}

Car :: type {
  engine expose : Engine
}

drive final : ()(car : Car mutable writable &) = {
  car.start()
  if car.isRunning()
    travel(car)
}
```

The visible calls belong to `Car`'s surface. Their implementations forward to
`car.engine`. A compiler may optimize a trivial wrapper away, but the language
does not promise one backend representation or zero pointer adjustment.
`car.rpm` remains unavailable because `expose` does not publish data.

### Mechanical eligibility

Automatic exposure is intentionally narrower than arbitrary handwritten
wrapping:

| Contained operation | Automatic exposure |
| --- | --- |
| Resultless instance function or operator | Eligible when inputs map unambiguously |
| Result in an orthogonal domain, such as `Boolean` | Eligible; result passes through unchanged |
| Contained value, reference, or pointer result | Eligible when the exact contained result type passes through unchanged |
| Receiver reference or pointer result explicitly declared `self` | Eligible; receiver origin is proved |
| Mapping a copied contained result to a complete outer value | Ineligible; the outer's remaining state cannot be invented |
| Constructor, replacement constructor, destructor, generated copy, or generated assignment | Ineligible; lifecycle targets the physical stored shape |
| Mapping requiring a checked or unsafe outer cast | Omitted |
| Ambiguous input, result, qualification, or provenance mapping | Omitted |

Exposure cannot strengthen or launder:

- visibility;
- access capability;
- place or referent qualifications;
- transfer stance;
- result provenance; or
- lifetime.

Automatic exposure also keeps by-copy input types unchanged. An `Engine`
parameter does not silently become a `Car` parameter merely because the
operation is exposed. Representation-trivial identity exposure is the one case
that may additionally substitute by-copy inputs and copied self results; see
[identity exposure](#identity-exposure).

Unchanged contained results are useful and do not require an outer cast:

```zax
Engine :: type {
  snapshot final : (result : Engine)() = {
    return copy _
  }

  selected final : (result : Engine &)() = {
    return locateSelectedEngine()
  }

  selectedPointer final : (result : Engine *)() = {
    return locateSelectedEnginePointer()
  }

  tune final : (result self : Engine &)() = {
    calibrate(_)
    return _
  }
}

Car :: type {
  engine expose : Engine
  wheels : Integer
}

inspectResults final : ()(car : Car mutable writable &) = {
  snapshot : Engine = car.snapshot()
  selected : Engine & = car.selected()
  selectedPointer : Engine * = car.selectedPointer()
  sameCar : Car & = car.tune()
}
```

The first three wrappers return exactly the contained operation's declared
result type. The `Engine` copy does not become a `Car`; no value exists for
`wheels`. The ordinary `Engine &` and `Engine *` also stay ordinary contained
results even if they happen to refer to a resident engine.

`tune` is different. Its verified reference `self` contract proves that the
result is exactly the receiver used for that invocation. The wrapper may
therefore return the original `Car` receiver. This narrow proof-backed
substitution does not authorize checked or unsafe outer casting for other
results.

Passing through safe unchanged results removes the need for broad
`expose everything` or `expose copy` modes. It does not authorize wrappers that
invent missing outer state, add copies, or assert provenance.

Callable viability, overload comparison, and visible-prototype adaptation are
defined by [function invocation](function-invocation.md). Operator discovery
and selection are defined by [operators](operators.md).

### The `self` result contract

`self` on a reference or pointer result means that every returning path returns
exactly the invocation receiver `_`:

```zax
Engine :: type {
  tune final : (
    result self : Engine &
  )(profile : Profile readonly &) = {
    apply(_, profile)
    return _
  }
}
```

The compiler verifies the contract. It is never inferred into a public
declaration, and it never applies to a copied result. This receiver-origin proof
lets an exposed wrapper map a contained receiver reference back to its outer
receiver without performing a runtime outer cast.

Reference origin and escape remain governed by
[lifetimes and references](lifetimes-and-references.md#returned-references).
When a true provenance claim cannot be proved, only a localized unsafe
assertion can accept that responsibility. An `intent` acknowledgement cannot.

### Direct declarations, ambiguity, and `once`

A direct outer declaration with an exact generated shape wins over exposure.
It is ordinary container behavior, not an override relationship. This includes
a bodyless direct declaration: its exact shape remains unavailable rather than
falling through to an exposed implementation. Use `= existing`, `via`, or a
body when the direct declaration should supply behavior.

Equal generated mappings and ordinary overload sets never use declaration or
generation order as a tiebreaker. An unresolved best set is an ambiguity error.

A `once` function is never exposed automatically. Its type-call mode has a
`Nothing` receiver while its instance-call mode has a present receiver, so one
automatic wrapper cannot preserve both meanings. A programmer may deliberately
provide a mode-specific `via`, select a unique existing mapping, or write a
wrapper body.

Lifecycle operations are also excluded. `expose`, `via`, and `= existing`
cannot turn a member constructor, replacement constructor, destructor,
generated `copy`, or generated assignment family into container lifecycle.
Container lifecycle targets the physical stored shape. A container lifecycle
body may invoke member lifecycle explicitly when its ordinary lifecycle
authority permits.

## Explicit behavior routing and filtering

### `via` selects one target

A function or operator `via` declaration names the exact route it adapts:

```zax
Car :: type {
  engine : Engine

  start final : ()() = via engine.start
  ignite final : ()() = via Car.engine.start
}
```

The relative path is ordinary inside the type. A full type-qualified path is
also available. There is no `_.engine.start` selector form.

Unlike a plain compatible visible prototype, `via` may generate a thin adapter
that performs fixed-offset receiver adjustment and mechanically permitted input
projection. It cannot invent policy, construction, conversion, capability, or
provenance. A written wrapper body is required when those decisions are needed.

The data form `name via member.path` is a no-storage declaration. The callable
form keeps a complete visible prototype and uses `= via member.operation` as
its definition.

An operator target repeats the operator's structured name:

```zax
PoweredCar :: type {
  engine : Engine

  operator binary '+' final : (
    result : Power
  )(
    rhs : Fuel
  ) readonly =
    via engine.operator binary '+'
}
```

The visible prototype selects one operation in that family. Zax does not add a
punctuation-only selector or a separate overload identifier.

Singular `via` must obtain one source operation and one complete mapping. If
permitted adaptation leaves several source overloads viable, the declaration is
ambiguous. An exact private helper can make the intended source prototype
explicit:

```zax
StartRequest :: type {
  fuel preferred : Fuel
  code preferred : IgnitionCode
}

Engine :: type {
  start final : ()(fuel : Fuel readonly &) = {
  }

  start final : ()(code : IgnitionCode readonly &) = {
  }
}

AmbiguousCar :: type {
  engine : Engine

  start final : ()(request : StartRequest readonly &) =
    via engine.start // error: either preferred route could fit an overload
}

Car :: type {
  engine : Engine

  startExact private final : ()(fuel : Fuel readonly &) =
    via engine.start

  start final : ()(request : StartRequest readonly &) =
    via startExact
}
```

The helper's exact `Fuel` prototype selects one `engine.start`. The public
declaration then targets that helper and projects `request` only through
`StartRequest.fuel`. This is legal for the same reason a public written body may
call an accessible private helper: the public declaration is a new API surface,
not publication of the helper. The rule does not let `own`, automatic `expose`,
or data `via` publish private data or behavior.

### `= existing` selects a unique mapping

`= existing` may locate one unique mechanically compatible existing operation
and one unique complete mapping through the permitted member surfaces. Unlike
`via`, it does not name the intended target; failure to find one unique mapping
is an error.

```zax
Engine :: type {
  stop final : ()() = {
  }
}

Car :: type {
  engine expose : Engine

  stop final : ()() = existing
}
```

No match or several matches is an error. Source order and overload ranking do
not guess which relationship the declaration meant.

### Adopting a callable family

The singular forms above select one operation. Their `family` counterparts
deliberately adopt a complete source callable family:

```zax
Engine :: type {
  reset final : ()() = {
  }

  reset final : ()(profile : Profile readonly &) = {
  }
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

`= existing family` must discover one unique source family from the exact
written anchor. `= via family` names that family and may give it another outer
name. Operator families use the same structured target:

```zax
PoweredCar :: type {
  engine : Engine

  operator binary '+' final : (
    result : Power
  )(
    rhs : Fuel
  ) readonly =
    via family engine.operator binary '+'
}
```

The written prototype is an exact anchor, not a wildcard. After it identifies a
family, each source member must independently obtain one mechanical outer
mapping. A conversion or other one-off adaptation that made the anchor viable
does not become a family-wide rule.

Family adoption is intentionally evolution-sensitive. If a later source
overload cannot map uniquely, the outer family becomes an error; that overload
is not silently omitted. Selecting only one singular operation avoids taking on
that source-family compatibility commitment.

### Fencing generated behavior

An exact fence permanently blocks one generated signature:

```zax
Car :: type {
  engine expose : Engine

  start final : ()() = forbidden
}
```

Generation and compatibility fallback cannot restore that signature.

`= forbidden family` blocks the complete outer callable family:

```zax
Engine :: type {
  reset final : ()() = {
  }

  reset final : ()(profile : Profile readonly &) = {
  }
}

Car :: type {
  engine expose : Engine

  reset final : ()() = forbidden family
}
```

The complete written prototype is an exact anchor for the outer name or
structured operator form. `family` broadens the prohibition to every generated,
exposed, adopted, or later direct signature in that outer family, regardless of
which contained source could have supplied it. A later direct `reset`
declaration therefore conflicts with the fence rather than bypassing it.

This is an outer-surface name fence, not an originating-family filter. A
same-spelled callable does not escape because it came from another member, and
no unrelated differently named family is affected. The fence introduces no
wildcard prototype, callable, or runtime state.

## Forwarding values, stances, and results

Automatic exposure does not rewrite contained by-copy inputs. Explicit `via` or
`= existing` declarations may instead present an outer by-value parameter and
project one member declared `preferred` to the selected contained input:

```zax
Fuel :: type {
  refill final : ()(source : Fuel mutable writable & move) = {
  }
}

Tank :: type {
  fuel preferred : Fuel
  shell : Shell

  refill final : ()(source : Tank) = via fuel.refill
  // `source` is established as a complete Tank. The wrapper offers
  // `source.fuel` as last, then destroys the remaining terminal Tank.
}
```

The visible outer parameter is constructed according to its declared stance.
The composition-specific consequence is that the selected member then reaches
the contained input through its preferred route rather than by changing the
visible outer contract. `own` alone cannot authorize this automatic input
projection. A written body remains free to name `source.fuel` directly and
therefore does not need `preferred`.

For outer `copy`, `move`, and `last`, the projected member is offered as `last`;
outer `deep` remains `deep`; and reference-shaped mappings preserve the caller's
exact authorization.

The complete fallback, terminal-member destruction requirement, and cost model
are owned by
[transfer stances](transfer-stances.md#explicit-composition-wrappers). In
particular, the mapping can still perform an outer copy followed by another
member copy; composition does not pretend it is free.

For results:

- orthogonal results pass through unchanged;
- a declared `self` reference or pointer may map to the outer receiver;
- a contained value, reference, or pointer may pass through with its exact
  declared result type;
- a copied contained result cannot become a complete outer value;
- an explicit `via` may retain the contained result type when its visible
  prototype declares that type;
- an explicit optional outer result may use checked outer casting; and
- an unproved non-optional outer remapping requires localized `unsafe via`.

The last form is a provenance assertion, not an intent acknowledgement.

For example, suppose `Engine.selectedPeer()` returns an `Engine &` whose
container relationship is not its invocation receiver:

```zax
Engine :: type outer tracked {
  selectedPeer final : (result : Engine &)() = {
    return locatePeer(_)
  }
}

Car :: type {
  engine own : Engine

  checkedPeerOwner final : (result : Car & ?)() =
    via engine.selectedPeer

  assertedPeerOwner final : (result : Car &)() =
    unsafe via engine.selectedPeer
}
```

The optional declaration visibly permits checked immediate outer casting. The
non-optional declaration instead asserts that the returned engine really
resides at `Car.engine`; a false claim has unsafe consequences.

## Abstract roles and explicit fulfillment

A component type used by an `own` member may describe declarations that its
immediate container must supply.

Inside an abstract required type or prototype, `outer` means the immediate
containing type whose `own` activates the requirement. It is a bounded
contextual type spelling, not a generic parameter and not runtime outer access.

```zax
WindowContract :: type {
  start abstract : ()()
  label abstract : String
  handle abstract : (result self : outer &)()
}

Window :: type {
  contract own : WindowContract

  begin fulfill contract.start final : ()() = {
  }

  caption fulfill contract.label : String

  onEvent fulfill contract.handle final : (
    result self : Window &
  )() = {
    return _
  }
}
```

An `abstract` declaration is compile-time role metadata. It creates:

- no storage;
- no implementation;
- no callable hook;
- no vtable or ABI entry;
- no hidden reference to the container; and
- no ability for inner code to call outward.

The immediate `own` activates the role. The immediate container must write a
compatible declaration that explicitly names the role with `fulfill`. A name
published from another member declared `own` does not satisfy anything
accidentally.
After fulfillment, the requirement does not propagate through another outer
composition boundary.

Declaration category, placement, and qualification rules are integrated with
[declarations and bindings](declarations-and-bindings.md#composition-facing-member-declarations).

The role's qualified abstract path is its identity. The fulfilling declaration
may have another name, as `caption` does above. One declaration may fulfill
several compatible roles by naming all of them:

```zax
start fulfill primaryContract.start, backupContract.start final : ()() = {
}
```

Every named path must be an active compatible role. Naming no real role is an
error. Identical requirements do not merge silently; both paths must appear in
the `fulfill` list. Incompatible requirements cannot share one declaration.

The stored carrier may be private even though its roles must be visible to the
immediate container:

```zax
Service :: type {
  contract own private : ServiceContract

  start fulfill contract.start final : ()() = {
  }
}
```

`Service` can name its own private physical path while checking fulfillment.
The carrier and any data it holds remain private; `own` does not launder them
into a public data surface. The direct `start` declaration independently chooses
its visibility, just as a written public declaration may use other private
implementation details.

Abstract metadata has no declaration place. Declaration-side words therefore
do not apply, and `private` is invalid because the immediate container must be
able to inspect the role:

```zax
Contract :: type {
  start abstract final : String // error: declaration-side `final`
  label abstract : String final // valid type qualification
}
```

Qualifications belonging to the complete required type or prototype remain
meaningful.

### Leaving defaulted qualifier axes open

Omitting a qualifier normally chooses the language default for that axis. An
ordinary abstract role follows that rule and therefore asks for one exact
defaulted shape. `abstract relaxed` is the explicit exception: it leaves only
the otherwise defaulted role-matching axes open.

```zax
ObserverContract :: type {
  observeExact abstract : ()()
  observe abstract relaxed : ()()
}

Observer :: type {
  contract own private : ObserverContract

  observeExact fulfill contract.observeExact final : ()() = {
  }

  observeReadonly fulfill contract.observe final : ()() readonly = {
  }

  observeWritable fulfill contract.observe final : ()() mutable writable = {
  }
}
```

The exact role has one fulfillment with the ordinary defaulted receiver shape.
The relaxed role has two distinct qualification-specialized fulfillments. Each
declaration explicitly names and independently satisfies `contract.observe`;
the declarations do not become a dispatch table, merge into one implementation,
or make fulfillment implicit.

For a value role, relaxation applies only to the outer required value layer. For
a callable role, it applies only to omitted receiver qualifier axes. Any
explicitly written qualifier remains mandatory. Relaxation does not recurse
into parameter or result types and does not loosen:

- the base type;
- arity or labels;
- indirection;
- transfer stance; or
- result provenance.

Duplicate normalized fulfillment signatures remain errors. An ordinary exact
role cannot acquire several implementations merely because qualification could
distinguish them.

### Compatibility and routed value fulfillment

Fulfillment uses ordinary safe same-place or compatible-prototype rules.
Transparent aliases may normalize and a same-place view may safely narrow
authority. Fulfillment cannot manufacture compatibility through:

- construction;
- conversion;
- preferred projection;
- capability strengthening;
- a runtime outer cast; or
- a computed temporary.

A direct data `via` declaration may fulfill a value role with resident storage
provided by another contained implementation:

```zax
PipelineContract :: type {
  buffer abstract : Buffer
  flush abstract : ()()
}

BufferedPipeline :: type {
  buffer : Buffer

  flush final : ()() = {
  }
}

Pipeline :: type {
  contract own : PipelineContract
  implementation : BufferedPipeline

  buffer fulfill contract.buffer via implementation.buffer
  flush fulfill contract.flush final : ()() = via implementation.flush
}
```

The value route must:

- be declared directly in the fulfilling container;
- name the abstract role explicitly;
- resolve statically and uniquely to one accessible resident stored place;
- preserve type truth, qualification, lifetime, and access;
- perform no construction, copy, conversion, preferred projection, checked outer
  cast, or temporary creation; and
- contain no route cycle.

The physical target may be nested. An automatically published short name is
never sufficient by itself.

## Identity exposure

Identity-specific application of the shared mechanical filter is owned by
[identity types](identity-types.md#exposed-and-opaque-behavior). The essential
composition distinction is that a representation-trivial, offset-zero identity
consists of exactly its underlying value. Unlike an ordinary container, it may
therefore substitute the identity type for mechanically compatible by-copy
inputs and copied underlying-self results:

```text
Integer + Integer -> Integer
```

can appear as:

```text
MyInteger + MyInteger -> MyInteger
```

When an identity carries additional resident data, by-copy substitution cannot
manufacture that state and this exception is unavailable. An eligible operation
may still remain exposed with its unchanged underlying result type; it simply
cannot claim to have produced the complete identity. The identity owner defines
the remaining eligible subset, invariant responsibility, and source-evolution
consequences.

Identity admission, projection, and underlying-operation behavior are defined
by [identity types](identity-types.md).

## Outer casting to an immediate container

Ordinary forwarding already knows its receiver relationship and needs no cast.
Other code may hold only a reference or pointer to a stored member. An
**outer cast** starts from that member access and casts outward to its immediate
container.

### Exact stored-member targets

Both outer-cast forms name an exact resident stored-member path:

```zax
Engine :: type outer tracked {
  rpm : Integer
}

Car :: type {
  engine own : Engine
  spare : Engine
}

inspectContainer final : ()(input : Engine &) = {
  asserted : Car & = input unsafe outer cast Car.engine

  checked : Car & ? = input outer cast Car.engine
  if ?checked
    use(checked.)

  invalid := input outer cast Car
  // error: target must be an exact resident stored-member path
}
```

The path resolves duplicate same-typed members and supplies the relationship
being claimed or tested. A target is never:

- a bare type;
- a function, including `final` or `once`;
- an alias;
- a published short path;
- a data `via` route; or
- a generated behavior surface.

The operations are protected language forms, not overloadable operators.
Their exact recognition and precedence are recorded by the
[operator catalog](operator-catalog.md#outer-cast-forms).

### Checked and unsafe outer casting

`unsafe outer cast` uses the named member offset under programmer-asserted
provenance. A false claim has undefined behavior.

`outer cast` is the safe optional-result form. When the selected language
contract cannot prove the exact origin statically, it checks one immediate
relationship at runtime. That runtime fallback requires the member type to be
declared `outer tracked` so the relationship can be validated.

Temporaries and references whose origin has expired are ineligible for both
forms. Neither operation revives an ended resident instance.

These operations apply the reusable unsafe and proof model in
[safety and analysis](safety-and-analysis.md). Their reference origins and
lifetimes remain governed by
[lifetimes and references](lifetimes-and-references.md#published-paths-and-outer-casts).

An outer cast crosses one immediate boundary at a time. Reaching a
grand-container requires another cast from the newly obtained immediate
container.
Possible nested-path sugar is future work and must not imply arbitrary ancestor
discovery.

### Static exact-origin proof

Runtime tracking is unnecessary when the selected language contract mandates a
proof that every origin reaching this cast site is the exact named member path:

```zax
Engine :: type {
  rpm : Integer
}

Car :: type {
  engine : Engine
}

inspectKnownCar final : ()(car : Car readonly &) = {
  engine : Engine readonly & = car.engine
  sameCar : Car readonly & ? = engine outer cast Car.engine

  if ?sameCar
    use(sameCar.)
}
```

At this site, `engine` can only have come from `car.engine`. Under a selected
contract that requires this analysis, the safe unmarked cast is accepted without
`Engine outer tracked` or a runtime relationship check. Its declared result is
still optional; flow analysis may additionally know that this particular result
is present.

Proof is about origins that can reach the cast site, not every value of the
member type in the universe. An importer constructing an unrelated standalone
`Engine` does not matter if that value cannot flow into `inspectKnownCar`.
Conversely, same-typed sibling members, arbitrary `Engine &` parameters,
callbacks, raw pointers, opaque ingress, or an exported API that accepts
standalone engines may prevent the exact-path proof because a different origin
can reach the cast.

This initial site-specific analysis is a conceptual proof model rather than a
finished formal algorithm. Source validity follows the selected contract:

| Situation | Required source behavior |
| --- | --- |
| Selected contract mandates the exact-origin proof and it succeeds | Safe unmarked `outer cast` is required; redundant `unsafe` is an error |
| Selected contract does not mandate that proof | Safe `outer cast` uses the tracked fallback; an unchecked assertion still requires `unsafe`, even if one compiler privately proves it |
| Source explicitly selects a stronger compiler or shared extension contract | That contract may make the stronger proof and safe spelling canonical |
| The asserted relationship is proved false | The source is an error under every contract |

A compiler may use extra private analysis to optimize or advise. Without a
selected contract that makes the proof part of source semantics, it cannot make
one program reject `unsafe` while another conforming compiler still requires
that word.

The matrix applies to result provenance in `via` as well:

- when the selected contract mandates and proves the outer result origin,
  ordinary safe `via` is required and `unsafe via` is redundant;
- when that proof is not in the selected contract, an unchecked non-optional
  outer result still requires `unsafe via`, even if this compiler can prove it;
- an explicitly selected extension may make its stronger proof canonical; and
- a `via` provenance assertion that is proved false is always an error.

### `outer tracked` costs and lifecycle

`outer tracked` explicitly permits compiler-owned hidden representation and
lifecycle bookkeeping:

```zax
Engine :: type outer tracked {
}
```

The metadata describes the current resident instance's placement. It is not
ordinary user state copied unchanged from a source value. A standalone `Engine`
and an engine resident at `Car.engine` have different placement relationships.

Ordinary construction, copy, move, and replacement establish the destination's
current metadata. Moving a complete container preserves its internal immediate
relationships in the destination. Removing a member from a container does not
retain the source relationship. Replacement ends the old resident relationship
and establishes the successor's relationship; references to the old resident
remain invalid.

A checked outer cast that uses the tracked fallback performs runtime work. A
contract-proved cast does not. The design does not promise a single stored
pointer, a particular metadata block, or any other backend representation. Raw
byte relocation cannot be assumed to preserve a valid tracked relationship.

## Costs

Composition syntax makes relationships shorter or generates statically selected
adapters; it does not make their consequences free.

Programmers must be able to discover:

- the unchanged cost of following a published data path;
- pointer adjustment performed by a generated wrapper;
- by-value member extraction performed by preferred projection;
- outer and member copies performed by explicit wrappers;
- especially expensive `deep` member mapping;
- fallback runtime work and hidden state permitted by `outer tracked`;
- additional callable surface generated by `expose`; and
- source compatibility deliberately coupled through family adoption; and
- source compatibility risk introduced by published names and preferred routes.

A compiler may inline or eliminate a wrapper when ordinary optimization permits.
That possibility does not change the language-level transfer, lifetime,
qualification, or failure behavior.

## Diagnostics

Required composition diagnostics include:

- `own` on a local or flow-scope declaration;
- ambiguous published data names;
- attempted publication or automatic forwarding through a pointer, reference,
  optional, variant, or comparable semantic wrapper;
- an inaccessible or non-place-preserving data `via`;
- cyclic data routes;
- indistinguishable direct preferred routes;
- ambiguous projected binding;
- attempting operator or member discovery through `preferred`;
- unavailable or ambiguous generated behavior;
- attempted exposure or routing of a member lifecycle operation as container
  lifecycle;
- exposing a mapping that requires capability or provenance invention;
- `self` on a copied result or a returning path that does not return `_`;
- singular `via` with no unique source operation and complete mapping;
- `= existing` without one unique complete mapping;
- family adoption without one unique source family or with a source member that
  has no unique mechanical outer mapping;
- a direct declaration conflicting with an outer `= forbidden family` fence;
- an exact or family fence without one exact anchor;
- a `fulfill` target that is absent, inactive, incompatible, or already
  fulfilled inconsistently;
- an activated abstract role left unfulfilled by its immediate container;
- several fulfillments of an exact role or duplicate normalized fulfillments of
  a relaxed role;
- declaration-side or private words on abstract metadata;
- reach-through construction of a member declared `own`;
- an outer-wrapper mapping whose remaining container cannot be destroyed in a
  valid terminal-member state;
- an outer cast whose target is not an exact resident stored-member path;
- a checked outer cast for which the selected contract supplies neither static
  exact-origin proof nor an `outer tracked` runtime fallback;
- redundant `unsafe` when selected-contract proof requires the safe form;
- omitted `unsafe` when the selected contract does not authorize the asserted
  provenance;
- a provenance assertion proved false; and
- an outer cast from a temporary or expired origin.

Exact diagnostic identifiers and presentation remain future diagnostics work.

## Formatting

Canonical formatting:

- orders the local modifier group as `own preferred expose`;
- preserves explicit physical paths rather than replacing them with published
  short paths;
- preserves every role path in a `fulfill` list;
- presents callable `via` targets as ordinary member paths and operator targets
  with their structured operator names;
- keeps `existing family`, `via family`, and `forbidden family` together as
  declaration states; and
- writes the outer-casting forms as `outer tracked`, `outer cast`,
  and `unsafe outer cast`.

The wider declaration grammar and formatter own placement of the complete
composition-modifier group relative to unrelated declaration and qualifier
words.

## Source stability

The following changes are source- or behavior-visible:

- adding a stored member to the type of an `own` member may create a
  published-name collision;
- adding a direct outer member may take over a published short name;
- adding or changing contained behavior may change an exposed surface;
- adding a preferred route may introduce a new use-site ambiguity;
- changing qualification may alter preferred-route viability;
- adding an abstract role may break immediate containers;
- changing a role may invalidate an existing fulfillment;
- adding a qualification-specialized fulfillment may conflict with an existing
  relaxed-role fulfillment;
- adding an overload to an exposed family may create new candidates;
- adding an overload to an adopted family changes the promised outer surface
  and may make the family declaration invalid if no unique mapping exists;
- adding an outer family fence conflicts with every declaration in that visible
  family, regardless of source provenance;
- changing a physical path may break `via`, `fulfill`, and outer-cast targets;
- adding identity data may make some exposure mappings ineligible; and
- adding or removing `outer tracked` changes representation and lifecycle cost.

None of these changes is resolved by declaration or source order.

## Boundaries and maturity

This document describes current conceptual design, not a formal grammar,
specification, ABI, compiler lowering, reflection contract, or implementation
guarantee.

It does not establish:

- class inheritance or virtual dispatch;
- a general interface, trait, concept, or structural-subtyping facility;
- a whole-type `abstract` contract or required structural shape;
- generic deduction through preferred projection;
- structural equivalence or layout substitutability;
- partial or external authority to change an owner's exposure fences;
- generalized parameter-origin result contracts beyond receiver `self`;
- local or flow-scope `own`;
- generalized delegation through semantic indirection;
- automatic nested outer casting;
- a final mandatory set of static outer-origin proof algorithms;
- reflection identities for published, generated, or fulfilled surfaces; or
- one representation for generated wrappers or `outer tracked`.

Future work may add those facilities only without turning publication into
storage, making projection drive discovery, weakening explicit fulfillment,
inventing provenance, or resolving ambiguity by source order.
