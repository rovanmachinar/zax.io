# 038: Terminal transfer intent

| Field | Value |
| --- | --- |
| Status | Historical / non-normative / audit-only; closed 2026-11-16 |
| Work Item | `038` |
| Created | 2026-11-12 |
| Owns | The bounded review of when a provable last use needs a stated transfer stance, and how a type shows whether taking it over can change anything, as defined below |
| Does Not Own | Current transfer-stance, invocation, or type behavior, which remain with their owners until a promotion is authorized; the decision itself, which belongs to the language maintainer |

## Non-authority notice

This file is a collaborative working record. Existing documentation, candidate
syntax, examples, and later aligned findings remain non-authoritative until a
separately discussed, aligned, and explicitly authorized promotion incorporates
them into their lasting owners.

## Fixed initiating input

This section records the aligned information known when work item `038` was
created. It is intentionally incomplete and must not be rewritten as work
develops.

### Initiating concern

When must a programmer state a transfer stance (`copy`, `move`, `last`, or
`deep`) at a provable last use, and when should the compiler accept the use
without one? In particular: which types can never be affected by being taken
over, the role "plain old data" plays in other languages; whether and how a
type states that it does or does not need a stance; and how generic code is
checked when the answer depends on the type each instantiation receives.

### Motivating pressure

- **The current rule is decided by the receiver, not the type.** The
  terminal-opportunity intent error fires when an unstated last use could be
  taken over by a receiver that would accept `last` differently. It does not
  fire when `last` would accept the same contract and produce the same source
  post-state. No owner says which types make that true.
- **Plain values should not need ceremony.** Maintainer direction: plain
  values such as `Integer` should need no stance, and the compiler should not
  complain about them. Examples currently leave `Integer` cases unannotated
  without claiming a rule.
- **Generic code is the hard case.** A body may be fine for one processing and
  need a stance for another. Current examples state a stance whenever generic
  items are passed on for the last time.
- **A long-standing frustration.** The maintainer has always found it unclear,
  in C++ and elsewhere, when a type is plain data and when it needs resource
  transfer. Zax should make that visible and teachable.
- **Captured input.** [Raw terminal transfer intent](../raw/terminal-transfer-intent.md)
  preserves the representative examples and the questions future work must
  decide.

### Starting input

The maintainer has no preferred solution. The agent is asked to produce a
range of options for review, from as simple as a documented definition to
source-level solutions, such as a stance declaration on a type, that make it
clear when a stance should or should not be written. Each option should come
with advice and a recommendation, and with the feasibility pressure it places
on an eventual implementation once the design is aligned. That analysis is
candidate material, not aligned design.

Legacy and raw material is evidence. It is not authoritative over current
design or over the maintainer's direction in discussion.

### Starting boundaries

- In scope: the boundary of the terminal-use intent error; a definition of
  types for which a transfer can never change anything; whether a type can
  state or opt out of that property, and whether it is inferred from members;
  how generic and variadic code is checked; and diagnostics that explain the
  decision.
- The meaning of `copy`, `move`, `last`, and `deep` is current design. A
  conflict with it is recorded as a finding for discussion, not resolved by
  silently changing it.
- Lifetime analysis, ownership of pointers and arenas, and async transfer are
  reviewed only far enough to record constraints.
- Reflection remains outside this item, although a type-level property may
  later be reflected.

### Initial stopping guidance

First establish the current rule precisely and gather the cases a definition
must cover: scalars, fixed arrays and structures of them, references and
pointers, optionals, unions and variants, and types with programmer-declared
copy, construction, or destruction behavior. Then present the options with
examples and a recommendation for discussion. Do not design every type
category in full.

Creating and routing this work item does not authorize analysis. A later
assignment begins it.

Do not promote findings, change current owners, archive this work item, or
create work item `039` without the separately required discussion, alignment,
and authorization.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs focused reading,
  authority, promotion, deferral, and closure.
- [Raw terminal transfer intent](../raw/terminal-transfer-intent.md) - the
  preserved concern, examples, and open questions.
- [Zax transfer stances](../../language/transfer-stances.md), especially the
  terminal-intent rule, source post-state, and declaration and type-alias
  stance overlays - the model this work refines.
- [A last use needs a stated stance](../../language/function-invocation.md#a-last-use-needs-a-stated-stance)
  in function invocation - how the rule applies at a call.
- The implicit-stance acknowledgement in
  [intent acknowledgements](../../language/intent-acknowledgements.md) - the
  existing way to keep implicit behavior deliberately.

### Consequence-driven

Read only the smallest relevant sections when the concern reaches them:

- [Zax construction and destruction](../../language/construction-and-destruction.md),
  when programmer-declared copy, construction, or destruction behavior decides
  whether a type is plain.
- [Zax type definitions](../../language/type-definitions.md), when a type-level
  property or declaration is proposed.
- [Zax unions](../../language/unions.md) and [Zax variants](../../language/variants.md),
  when passive or all-bit-safe layouts or managed alternatives bear on the
  definition.
- [Zax optional values](../../language/optional-values.md),
  [Zax pointers and arenas](../../language/pointers-and-arenas.md), and
  [Zax lifetimes and references](../../language/lifetimes-and-references.md),
  when indirection or ownership layers are classified.
- [Zax generics](../../language/generics.md) and
  [Zax variadics](../../language/variadics.md#a-stance-applies-to-each-item),
  when generic or per-item checking is decided.
- [Zax arrays and slices](../../language/arrays-and-slices.md), when element
  transfer into an array is reached.
- [Zax qualifiers](../../language/qualifiers.md), when a property interacts
  with mutability or access.
- [Raw mutability-indexed type families](../raw/mutability-indexed-type-families.md)
  and [raw safety boundaries](../raw/safety.md), when generated copy shapes or
  transfer-contract blame are reached.

### Audit-only

- Archived numbered work, only when a provenance question cannot be answered
  from current owners or raw input. The transfer-stances review is
  `013-transfer-stances-and-source-state.md`; the variadics review is
  `037-variadic-arguments.md`.

## Working record

### Maturity of this record

Sections 1 to 8 are the agent's initial reconstruction (2026-11-12) for
maintainer review. They are raw input and candidate analysis, and parts are
now superseded; each superseded part carries a note. Nothing in sections 1 to 8
is an aligned finding or accepted design.

[Section 9](#9-discussion-record-2026-11-12-to-2026-11-13) records the
maintainer discussion: aligned findings, dispositions of the questions it
raised, and captures.

**A new session starts with
[section 10](#10-brief-for-the-next-session-2026-11-14),** which lists what to
read, how to write, and the pending change. **Read section 9 first.** Aligned findings there remain
non-authoritative until a separately authorized promotion.

Syntax marked *illustrative* is invented for discussion only.

### Review entry point

**Candidate programmer model, one paragraph.** At a provable last use, a
`last` offer can change the outcome for two independent reasons. Either the
*consumer* declared a slot that only a stronger offer can bind (`last`/`move`
parameters, `& last` overloads), or the *value's own type* has a `move` or
`last` operation that a stronger offer would select when the value is
constructed or assigned. Only the second reason is a property of the type. A
type with no reachable `move` or `last` operation, such as `Integer`, can never
trigger the second reason, so ordinary code using it needs no stance. This
reconstruction calls that property *transfer-inert* (working name only). It is
derivable from declarations the compiler already sees; it is not the same as
"owns no resources".

**The most important uncertainty.** Current owners do not say which of the two
reasons the rule means, and they phrase it differently:

- [Transfer stances](../../language/transfer-stances.md#terminal-opportunity-must-be-explicit)
  says no stance is needed when "`last` would accept the same contract and
  produce the same source post-state". That wording covers both reasons and
  exempts `Integer` through the type's fallback to `copy`.
- [Variadics](../../language/variadics.md#a-stance-applies-to-each-item) says
  the rule "applies whenever the receiver could take an item over. An ordinary
  by-value parameter could". That makes the by-value parameter form decide
  regardless of the type. Taken literally, it requires a stance for `Integer`,
  against maintainer direction.

Section 1 shows the two reasons with examples. Under the first reading, the
maintainer's `Integer` direction already holds today for most code. What is
missing is the definition, its teaching, the generic rule, and diagnostics,
rather than a new exemption.

**Decisions needing maintainer review** (each is developed below):

1. *Which reasons the rule counts* (section 1). Recommendation: count both,
   and state them separately. Correct the variadics wording at promotion.
2. *What the defined property is* (section 3). Recommendation: define it by
   reachable transfer operations (transfer-inert), and teach separately why it
   is not "plain old data".
3. *Whether a type can state the property* (section 4). Recommendation: derive
   it with no annotation. Treat an optional verified declaration on the type, in
   either direction, as a separate source-stability decision.
4. *How generic bodies are checked* (section 5). Recommendation, with
   reservations: a terminal use of a value whose type depends on a generic
   parameter needs a stance in the generic body unless that type is known to be
   transfer-inert. Errors are not discovered per instantiation inside library
   code.
5. *Diagnostics* (section 6). Recommendation: always name which reason
   applied and the declaration that caused it.

**Known holes.** Section 2 has a table of type categories. Several rows are
inferred from owners that do not state stance behavior directly. Scheduled raw
pointers make the property depend on the declaration as well as the type.
Structures never get a generated `move`, which makes some resource-owning
structures transfer-inert (section 3).

**Adjacent findings captured but deliberately not solved** are in section 7.

### 1. The current rule, made precise

> **Superseded in part (section 9).** The two reasons are reframed as the
> necessity and intent requirements, decided from three inputs, in 9.1 A. The
> Reason B example uses a plain by-value parameter. It is no error under
> current design, because by-value parameters always copy in. It is an intent
> error under the aligned model, because a `last` offer would run `Resource`'s
> move constructor (9.1 B). The `countDown(value : Integer last)` example is a
> necessity error at any use (9.1 S).

At an implicit (`copy`) offer that analysis proves is the source's final
ordinary use, the compiler compares selection under the actual `copy` offer
with selection under a hypothetical `last` offer. If they differ in selected
consumer, accepted stance, or source post-state, the use is an intent error.
Fallback is `last -> move -> copy`, and a `copy` offer accepts only `copy`
([viability and preference](../../language/function-invocation.md#partial-order-preference)).

The difference can come from two places.

#### Reason A: the consumer declares a stronger slot

The consumer's own declaration decides. The value's type is irrelevant.

```zax
countDown final : ()(value : Integer last) = {
}

tick final : ()() = {
  remaining : Integer = 3
  countDown(remaining)
  // An implicit copy offer cannot bind a `last` parameter. A `last` offer
  // can. Under current wording this is a terminal-intent error even for
  // Integer, because the accepted contract changes.
}
```

```zax
store final : ()(value : Integer readonly & copy) = {
}

store final : ()(value : Integer mutable writable & last) = {
}

record final : ()() = {
  total : Integer = 7
  store(total)
  // intent error: a `last` offer selects the other `store` body.
}
```

In both cases someone deliberately wrote a stance-taking slot for `Integer`.
A type-level property cannot remove this reason without overriding the
consumer author's declaration.

Reconstruction note: at a *non-terminal* use, `countDown(remaining)` is
simply not viable, which is an ordinary selection error. At a terminal use it
is reported as missing intent. The same call therefore gets different
diagnostics depending on later code. That is probably acceptable, but the
diagnostic needs to make it clear (section 6).

#### Reason B: the value's type has its own `move` or `last`

The consumer is an unstanced or `copy` by-value slot, an array element, a
declaration initializer, or an assignment. A constructor or assignment
operation of the value's type performs the binding. The offered stance selects
among that type's own variants:

```zax
Resource :: type {
  +++ final : ()(source : Resource readonly & copy) = {
  }

  +++ final : ()(source : Resource mutable writable & move) = {
  }
}

forward final : ()(value : Resource) = {
}

forwardCount final : ()(value : Integer) = {
}

finish final : ()() = {
  resource : Resource
  count : Integer = 4

  forwardCount(count)
  // No error: Integer has only copy, so a `last` offer falls back to the
  // same copy construction and leaves count unchanged.

  forward(resource)
  // intent error: a `last` offer would select Resource's move constructor
  // through fallback and leave resource moved-from.
}
```

The raw input's `forward(resource)` and `sum` examples are both Reason B. The
`sum` example is already exempt under this reading:

```zax
sumAll final : (result : Integer)(values : Integer[]) = {
}

sum final : (result : Integer)(values ... : Integer) = {
  return sumAll([ from values ])
  // Each Integer item constructs one array element. Integer has no move or
  // last, so no item needs a stance.
}
```

#### Where the existing examples sit

- `inspect(record)` with a `readonly & copy` parameter: neither reason
  applies, since a reference binding constructs nothing and the slot is `copy`.
- `forwardAll$` in transfer-stances and variadics: `sendAll` declares
  `items ... : Items last`, so Reason A applies to *every* item type. The
  example's unconditional error is correct. The explanatory prose ("an
  ordinary by-value parameter could") is the part that overstates.
- Result mapping (`consume(makePayload())`): the same two reasons apply, with
  the result slot as the source.

#### Smaller tensions in the current rule

- **By-value `move`/`last` parameters copy in.** The
  [by-value move](../../language/transfer-stances.md#by-value-move) form
  constructs its parameter with `copy`, so the caller's source post-state never
  changes. Only the selected contract differs. The rule's "same contract *and*
  same post-state" wording therefore still triggers, but "could take it over"
  in variadics is not literally true for these parameters.
- **`= existing` move.** A type may write
  `+++ final : ()(value : Value move) = existing` to reuse its `copy`. The
  accepted stance is then nominally `move`, so the source is analysed as
  moved-from even though nothing was taken. Under Reason B such a type is not
  transfer-inert. That is arguably correct: the author declared a `move`
  contract and may later change its body.

### 2. Cases the definition must cover

"Inert" below means: no reachable `move` or `last` operation for a whole-value
transfer, so Reason B can never apply. Rows marked *inferred* follow from an
owner's general rules but are not stated there.

| Category | Inert? | Basis |
| --- | --- | --- |
| Integer, fixed-point, floating-point, Boolean scalars | Yes | Generated copy only |
| Enums and flags | Yes, *inferred* | No declared transfer operations |
| Fixed array with inline storage | When the element type is | Elementwise `move`/`last` ([arrays](../../language/arrays-and-slices.md#copy-deep-move-and-last)) |
| Provider-backed array, including `Integer[]` | **No** | `move`/`last` may hand over the backing handle |
| Slice | Yes, *inferred* | Transfer changes only the window |
| Structure with generated copy only | **Yes, even with resource-owning members** | `move` and `last` are never generated |
| Structure that declares `move` or `last` | No | Reason B by definition |
| Structure that declares copy and a destructor only | Yes | `last` falls back to copy; the destructor runs either way |
| Single-owner type (copy `forbidden`, `move` declared) | No | A copy offer is not viable at all |
| Reference | Not applicable | References are excluded from the terminal rule |
| Ordinary raw pointer | Yes, *inferred* | Transfer does not make the pointee terminal |
| Scheduled raw allocation (`p : T * = @`) | **No, but per declaration** | Disposition responsibility lives in the declaration, not the type |
| `* unique`, `* strong`, `* weak`, `OpaqueOwner` | No | `last` transfers or retires participation |
| Optional | When the payload is | Stance is forwarded to a present payload |
| Variant | When every alternative's payload is, *inferred* | Stance is forwarded to the active payload; static checking must assume any alternative |
| Unmanaged union | Yes | `move`/`last` fall back to all-bit copy ([unions](../../language/unions.md#whole-union-operations)) |
| Bound callable values | No when they carry a strong or unique relationship | `move`/`last` transfer the relationship |
| Representation-trivial identity type | Open | Whether it inherits the underlying type's operations needs the identity owner |

Three rows matter most for the decision:

- **`Integer[]` is not inert.** Element plainness does not make an owning
  array plain. `sumAll(values)` with a named `values : Integer[]` at its last
  use needs a stance, because a `last` offer could hand over the buffer.
- **A structure of resource-owning members is inert unless its author declares
  `move` or `last`.** See section 3.
- **Scheduled raw pointers** make a purely type-level property incomplete. The
  rule must say "type and declaration", or treat scheduled declarations as a
  stated exception, as result mapping already does.

### 3. Two different definitions

> **Note (section 9).** Definition T remains the basis: what matters is
> whether a stance can change the outcome. The working term
> "transfer-inert" is not adopted. Section 9 speaks of whether a stance
> *matters*, and of a type's transfer operations.

Two definitions both sound like "plain data". They give different answers.

**Definition T, transfer-inert:** no `move` or `last` operation is reachable
for the type, recursively through containers that forward stances (inline
arrays, optionals, variants). This is exactly what Reason B tests. It is
derivable from declarations and generation rules.

**Definition R, resource-free:** the type owns nothing that destruction
releases. It has no destructor anywhere in its members, no owning pointers, and
no provider-backed storage. This is closer to C++ "trivially copyable" or
"plain old data".

They diverge here:

```zax
MyReport :: type {
  title : String
  lines : Integer[]
}

publish final : ()(report : MyReport) = {
}

send final : ()() = {
  report : MyReport
  publish(report)
  // Definition T: no stance needed. MyReport has only generated copy, so a
  // `last` offer falls back to copy and nothing can be taken.
  // Definition R: MyReport owns resources, so this looks like it should need
  // a stance, but stating `as last` would change nothing.
}
```

Definition T is the one that matches what a stance can actually change.
Requiring a stance under Definition R would ask the programmer to state an
intent that has no effect. It would also make `as last` look meaningful when
it is not.

The divergence comes from a construction rule: Zax never generates `move` or
`last`. A programmer who expects aggregates to move their members, as in C++
or Rust, will be surprised that `report as last` copies both buffers. That
surprise belongs to the generation rule, not to this item (section 7). This
item's teaching should still state it plainly, because it is where a
programmer first notices it.

Tentative recommendation: adopt Definition T for the intent rule and teach
that "transfer-inert" means "a stance cannot change anything", not "cheap to
copy" or "owns nothing". Definition R remains useful for other questions
(unions, zero-fill, bit copy) that this item does not own.

### 4. Options for a type-level signal

> **Superseded (section 9).** Option 3 is adopted in a revised form: a
> `transfer` clause listing the complete permitted set of stances (9.1 D).
> The `inert` and `aware` names are dropped. The derived default of Option 1
> survives as the meaning of an undeclared contract. Option 2's predicate is
> not adopted (9.1 Q).

All options assume Definition T. They are ordered from least to most source
surface.

#### Option 1: documented derived definition only

The transfer-stances owner defines the property and the two reasons, adds the
category table, and corrects the variadics wording. No syntax is added.

- *Advice:* this is the minimum, and every other option builds on it. It already
  satisfies the `Integer` direction for concrete code.
- *Gap:* a programmer cannot ask the compiler "is this type inert?", and a
  generic body cannot rely on it.
- *Feasibility:* none beyond the existing rule. The compiler already resolves
  the type's operation family.

#### Option 2: derived property plus a compile-time query

Option 1, plus a compiler-provided compile-time predicate (illustrative name
`isTransferInert(T)`). Constraints and compile-time assertions can use it, and
diagnostics can cite it.

```zax
// illustrative
sumEach$ final : (Items ... : isTransferInert)(result : Integer)(
  items ... : Items
) = {
  // A generic body may omit stances for these items because the constraint
  // guarantees that a stance cannot change anything.
}
```

- *Advice:* this gives generic code a way to avoid ceremony (section 5)
  without a new declaration form.
- *Tension:* [generics](../../language/generics.md#checking-generic-bodies)
  treats a Boolean constraint as opaque: it says whether arguments are
  accepted, not what the body may assume. Using it to relax a body rule means
  the compiler must recognize this specific predicate in a constraint, which is
  new.
- *Feasibility:* the predicate itself is cheap. Recognizing it inside
  arbitrary Boolean constraint expressions is not. It probably requires the
  short form, a top-level `requires` conjunct, or a dedicated parameter
  property.

#### Option 3: verified declaration on the type, in either direction

A type states the property, and the compiler verifies it. Type definitions
already place capability phrases after `type` (`MyTrackedEngine :: type outer
tracked`), so an illustrative form follows that position:

```zax
// illustrative
MyPoint :: type transfer inert {
  x : Integer
  y : Integer
}

MyHandle :: type transfer aware {
  id : Integer
  // No move yet, but downstream terminal uses must already state a stance.
}
```

Its value is source stability, not convenience:

- **Adding `move` to a library type is a downstream break.** Every implicit
  terminal use of that type in client code becomes an intent error. The break
  is loud, which fits Zax's rules, but nothing in the type's declaration warns
  clients that it can happen. `transfer inert` would make "never" an API promise,
  and removing it would be a visible change.
- **`transfer aware` reserves the right to add `move` later.** Clients state
  stances now, so a later `move` does not break them.

- *Advice:* worthwhile only if library evolution is a real concern now. It
  could be added later without invalidating Option 1 or 2 code.
- *Risk:* two words that most types never need. Each would also need a rule
  for whether a containing type inherits it.
- *Feasibility:* verification is a check over the type's declared and
  generated operation family plus member types. It is cheap and local.

#### Option 4, recorded for completeness: a type-level default stance

[Type aliases already overlay a declaration stance](../../language/transfer-stances.md#type-aliases-may-overlay-declaration-stance).
A type could likewise declare that its values ordinarily offer `move`. That
changes the default offer rather than answering when a stance is needed.
Implicit `move` would also weaken the rule that stronger transfer is written.
It is not recommended. It is listed because it is the nearest existing
mechanism.

**Tentative recommendation:** Option 1 now, and Option 2 if the generic
decision in section 5 needs it. Option 3 is a separate decision about library
evolution, not about ceremony.

### 5. Generic and variadic checking

> **Superseded (section 9).** Generic bodies are checked per instantiation
> (G1), with the diagnostic of G3 (9.1 Q). G2 and the predicate are not
> adopted. The `forwardAll$` example is a necessity error at any use (9.1 S).

Non-generic packs with a stated element type (`values ... : Integer`) are
concrete. They follow the concrete rule per item. The hard case is a dependent
item type:

```zax
collect$ final : (Items ... : type)()(items ... : Items) = {
  all := [ from items ]
}

collect(1, 2)          // Integer items: a stance changes nothing
collect(myResource)    // a Resource item: an intent error, but where?
```

**G1: check each instantiation.** This is what the current
[generic checking model](../../language/generics.md#checking-generic-bodies)
implies, because generic bodies are checked once per instantiation.

- `collect(1, 2)` compiles and `collect(myResource)` reports an intent error
  inside `collect$`.
- *Problem:* the caller cannot fix it. The terminal use is of the generic body's
  own by-value parameter, so no caller stance or caller acknowledgement reaches
  it. The library author may never see it if their tests use plain types.
- *Feasibility:* free; it falls out of instantiation.

**G2: require a stance in the generic body for dependent types.** At
a terminal use where the source's type depends on a generic parameter, Reason B
is assumed possible. A stance is required unless the type is known to be
transfer-inert (with Option 2, through a recognized constraint).

- `collect$` must write `[ from items as copy ]` or `as last`, which matches
  the convention current examples already follow.
- *Cost:* ceremony for numeric generic code, unless Option 2 exists. Writing
  `as last` on an inert item is harmless because it falls back to `copy`.
- *Tension:* it introduces a definition-time judgement into a model that
  otherwise checks only instantiations. The check can still run per
  instantiation if it asks "does the type depend on a generic parameter?"
  rather than "is this instantiation's type inert?". It then gives the same
  answer for every instantiation.
- *Feasibility:* the compiler must track type dependence at each terminal
  use. Terminal-use analysis on the body does not depend on the instantiation
  except where overload selection changes control flow, which is a corner to
  confirm.

**G3: G1 with better blame.** Per-instantiation checking, plus a
diagnostic that names the generic body as the place to fix and suggests a
stance or constraint. A library self-test with `compile{ assert(wellformed{
... }) }` could catch it.

- *Advice:* it keeps the model simple but leaves the "caller cannot fix it"
  problem in place.

Reason A in generic code depends only on the consumer slot. It is
instantiation-independent unless the consumer is itself generic.

**Tentative recommendation:** G2. It keeps the error with the author who can
fix it. Its cost is the ceremony the maintainer wants to avoid, but only in
generic code over unconstrained types. Option 2 would remove that ceremony for
generic code that wants it. This is the decision where the agent is least
confident, because G2 is a new kind of rule in the generics model.

### 6. Diagnostics

> **Note (section 9).** The diagnostic shape still applies. Its reasons now
> follow the necessity and intent requirements. 9.1 D adds the
> unavailable-generated-copy diagnostic, 9.1 H the fallback note, and 9.1 Q
> the generic-body diagnostic.

Each intent error names the reason and the deciding declaration, so a
programmer can see why a similar call did not need a stance:

```text
error: last use of `resource` needs a stated stance
  reason: Resource declares a move constructor (resource.zax:3), so `as last`
          would move from `resource` instead of copying it
  state: as copy, as move, as last, or as deep
note: `count` on line 12 needed no stance because Integer has no move or last
```

```text
error: last use of `remaining` needs a stated stance
  reason: countDown's parameter `value` is declared `last` (tick.zax:1)
```

For G2, the reason line names the generic dependence ("`Items` may be any
type") and the constraint that would remove it.

### 7. Adjacent findings captured, not solved

> **Superseded (section 9).** Each item is disposed in section 9. Structures
> get only generated `copy`, with `= default` as the opt-in (9.1 O). Scheduled
> raw pointers need no exception (9.1 R). Non-viable calls are necessity errors
> at any use (9.1 A). Identity types are 9.1 N. Return-as-final-use remains the
> existing transfer-stances open question.

- **No generated `move`/`last` for structures.** This makes resource-owning
  structures transfer-inert and makes `report as last` copy every member. Owner:
  [construction and destruction](../../language/construction-and-destruction.md#demand-driven-generation).
  Reopen if a programmer use case shows aggregate move is expected. Constraint
  on this item: Definition T inherits whatever generation rule exists. If
  generation changes, the category table changes with it.
- **Scheduled raw declarations.** The property is per declaration for these.
  Owner: [pointers and arenas](../../language/pointers-and-arenas.md). This item
  must either name the exception or define the property over type plus
  declaration.
- **Non-viable versus intent error.** The same Reason A call is a selection
  error at a non-terminal use and an intent error at a terminal use. Owner:
  function invocation diagnostics. Constraint: the diagnostic must show both.
- **Returning a parameter as its final use** is already an open question in the
  [transfer-stances boundaries](../../language/transfer-stances.md#boundaries-and-maturity).
  It is the same rule at a different consumer, and Reason B decides it the same
  way. Not reopened here.
- **Reflection** of the property is out of scope per the initiating input. If
  Option 2 or 3 is adopted, reflection would later expose it.
- **Identity types** may inherit the underlying operation family. The identity
  owner must confirm this before the category table is promoted.

### 8. Likely owner boundaries at promotion

- [Transfer stances](../../language/transfer-stances.md): the two reasons, the
  definition, the category table or its teaching subset, the "not plain data"
  boundary, the generic rule, and diagnostics.
- [Function invocation](../../language/function-invocation.md#a-last-use-needs-a-stated-stance):
  local summary of the two reasons.
- [Variadics](../../language/variadics.md#a-stance-applies-to-each-item):
  correct "an ordinary by-value parameter could".
- [Generics](../../language/generics.md#checking-generic-bodies): the generic
  terminal-use rule if G2 is adopted, and the predicate if Option 2 is adopted.
- [Type definitions](../../language/type-definitions.md) and
  [construction and destruction](../../language/construction-and-destruction.md):
  only if Option 3 is adopted.
- [Intent acknowledgements](../../language/intent-acknowledgements.md#implicit-stance-at-terminal-use):
  category text if the reasons change its description.
- `language/terms.md`: the term, once named.
- [Raw terminal transfer intent](../raw/terminal-transfer-intent.md): retire
  when the above absorb it.

> **Note (section 9).** 9.4 extends this list. The aligned model changes
> by-value parameter construction, body defaults, declared stances on locals,
> and overload preference, so promotion reaches further than listed here.
> Invocation viability is unchanged.

### 9. Discussion record (2026-11-12 to 2026-11-13)

Maturity: 9.1 holds **aligned findings** for this work item. They are not
promoted and not authoritative. 9.2 records how each former open question was
disposed. 9.3 holds **captures**: future concerns and rejected proposals. All
syntax is *illustrative* unless noted as current.

The following example declarations are used throughout:

```zax
// illustrative
MyHandle :: type transfer copy move last {
  id : Integer
  // only copy is implemented today; move and last are reserved
}

Message :: type {
  // no transfer clause: has copy and a declared move constructor
  +++ final : ()(source : Message mutable writable & move) = {
  }
}

MyOwnedFile :: type transfer move {
  handle : Integer
  // move-only, like std::unique_ptr
}

relay final : ()(input : Message) = {
}
// plain by-value parameter

sendMessage final : ()(input : Message move) = {
}
// the caller must offer at least move

sendCount final : ()(input : Integer) = {
}

archive final : ()(file : MyOwnedFile) = {
}

hold final : ()(input : Message mutable writable & last) = {
}

logHandle final : ()(input : MyHandle) = {
}

sendHandle final : ()(input : MyHandle move) = {
}
```

`Integer` has only `copy`.

#### 9.1 Aligned findings

##### A. When a stance must be written

Two separate requirements apply:

1. **Necessity, at any use.** No stance means `copy`. If a use can only work
   with `move` or `last`, an unstated stance is an error wherever it appears,
   not only at a last use. The compiler never infers `move` or `last`, because a
   reader seeing no stance assumes the callee took a copy or preserved the
   source.
2. **Intent, at a last use.** The implicit `copy` works, but a `last` offer
   would change the outcome. The programmer states which is meant.

```zax
// declarations from the start of section 9
archive(file)        // necessity, at any use: MyOwnedFile has no copy
hold(message)        // necessity, at any use: the parameter requires `& last`
sendMessage(message) // necessity, at any use: the parameter requires at least move
relay(message)       // copies; at a last use: intent error, because a last offer
                     // would run Message's move constructor
sendCount(count)     // copies; no error anywhere, because nothing could differ
```

The compiler decides both requirements from three inputs:

1. the caller's offer (implicit `copy`, or `as copy`, `as move`, `as last`,
   `as deep`);
2. the consumer's declaration (a parameter, a local or alias initializer, an
   assignment, an array element, or a capture);
3. the type's transfer operations: its declared `transfer` contract, or what
   it currently has when it declares none.

For intent, at a provable last use, the implicit offer is compared with a
`last` offer. A stance is required when the `last` offer would select a
different function body, or run a different construction, assignment, or
replacement operation of the type. Reserved operations count as if implemented
(E). Whether this is a last use is not an input to the outcome. It only decides
whether the intent requirement applies.

Maintainer direction: do not force a stance where it does not matter; do force
one where it does. Wherever `move` or `last` is required, it is written,
because the default reads as `copy`.

##### B. By-value parameters: three separate decisions

Three things are decided when a function with a by-value parameter is called.
Current design and the first version of this finding mixed them.

1. **Choosing the function: current design, unchanged.** A parameter's
   declared stance is the *least* the caller must offer, using the normal
   fallback. An implicit or `as copy` offer reaches only plain parameters. An
   `as move` offer reaches `move` parameters, then plain ones. An `as last`
   offer reaches `last`, then `move`, then plain. So `input : Message move`
   says "the caller must offer at least `move`". An implicit call is not
   viable, which is an error at any use (A, necessity).
2. **Building the parameter: changed.** The caller's offer goes to the type's
   constructors unchanged, using the normal fallback. The parameter's declared
   stance does not limit or rewrite it: `as last` stays `last` and is not
   turned into `move`. This replaces the current copy-in barrier, in which a
   by-value parameter is always copy-constructed even when declared `move` or
   `last`.
3. **Inside the body: changed.** The body starts from `copy` (C2). The
   parameter's declared stance says how the value arrives. It places no
   requirement on how the body later uses it.

By-value parameters are therefore built exactly like local declarations
(M). Locals, assignment, array elements, and captures need no separate rule,
because none of them has a step 1.

```zax
// also uses declarations from the start of section 9
MyMoveOnlyType :: type transfer move {
  handle : Integer

  +++ final : ()(source : MyMoveOnlyType mutable writable & move) = {
    // source refers to the caller's variable during construction; take the
    // handle and leave source vacant
  }
}

func final : ()(input : MyMoveOnlyType) = {
}

myMoveOnlyType : MyMoveOnlyType
func(myMoveOnlyType as move)   // plain parameter accepts move; MyMoveOnlyType's
                               // move constructor builds input; myMoveOnlyType is vacant
func(myMoveOnlyType)           // error at any use: no copy constructor exists

sendMessage(message as last)   // move parameter accepts last; the offer stays last,
                               // so Message's last constructor runs, or move if none
sendMessage(message)           // error at any use: the parameter requires at least move
relay(message as move)         // plain parameter accepts move; move constructor runs
relay(message)                 // copies; at a last use: intent error
sendCount(count)               // copies; no error anywhere
```

The by-value parameter is built by calling the type's constructor. That
constructor's source is a reference to the caller's variable, which is how a
`move` or `last` constructor takes the contents out. The parameter itself is a
new value owned by the function.

`Message` has `copy` and `move`; `MyOwnedFile` has only `move`:

| Type | Parameter | Implicit | `as move` | `as last` | Last use needs a stance? |
| --- | --- | --- | --- | --- | --- |
| `Message` | plain | copy runs | move runs | move runs (no `last`) | Yes: `last` would move instead of copy |
| `Message` | `move` | not viable | move runs | move runs | Already required at any use |
| `Integer` | plain | copy | copy | copy | No |
| `Integer` | `move` | not viable | copy runs | copy runs | Already required at any use |
| `MyOwnedFile` | plain | error: no copy | move runs | move runs | Already required at any use |
| `MyOwnedFile` | `move` | not viable | move runs | move runs | Already required at any use |

The caller's value ends in the state left by the operation that actually ran,
subject to E and I.

*Withdrawn:* the first version of this finding limited construction to the
lower of the offer and the parameter's declared stance, called a "ceiling".
It let a plain implicit offer bind a `move` parameter and copy, and made
`func(myMoveOnlyType as move)` an error. The limit protected nothing, because
the parameter is the callee's own new value and the caller treats its variable
as finished after `as last` anyway. The maintainer found the concept confusing.
It is not taught.

##### C. Reference parameters

Reference binding constructs nothing. The declared stance is a requirement the
body relies on, so the caller's offer must meet it, at any use and for every
type (A, necessity). This holds even for a type with nothing to take, such as
`Integer`. The callee asked for `move` or `last` authority over the caller's
variable, and an unstated offer would read as `copy`.

```zax
// illustrative
bump final : ()(value : Integer mutable writable & last) = {
}

hold final : ()(input : Message mutable writable & last) = {
}

store final : ()(value : Integer readonly & copy) = {
}

store final : ()(value : Integer mutable writable & last) = {
}

bump(count)        // error at any use: the parameter requires `last`, even for Integer
bump(count as last)
hold(message)      // error at any use: the offer must meet `last`
store(total)       // selects the `& copy` store; at a last use: error, because
                   // a `last` offer would select a different body
```

The last case is the intent requirement: an implicit offer works, selecting
the `& copy` body, but a `last` offer would select the other body. No rule
limits which parameters may be declared for a type.

An unmarked reference parameter means `loan` (U). A reference that may copy
from the referent declares `& copy`.

An earlier version of this finding exempted types with nothing to take, so
that `bump(count)` bound implicitly. That exemption was withdrawn because it
hid the callee's requested authority from the reader.

##### C2. Body defaults for parameters and receivers

A parameter's declared stance says how the value arrives. Inside the body,
each default is **the safest stance consistent with what the caller offered**,
and no default imposes a requirement: an explicit stance may restate it.

| Parameter | Whole value in body | Members without their own stance |
| --- | --- | --- |
| By value, any stance | `copy` | `copy` |
| `& loan` (and unmarked references, U) | no transfer | `copy` |
| `& copy` | `copy` | `copy` |
| `& move`, `& last` | `copy` | `copy` |
| `& deep`, and `deep` receivers | `deep` | `deep` |
| Lifecycle source (W) | never transferred as a whole | the operation's authority |

Members that declare their own stance always use it (M).

"Safest" points in different directions:

- **`move` and `last` references:** starting at `copy` is the cautious
  direction. Nothing is taken unless written, so an innocent helper call cannot
  take the value before the intended transfer. Current design already does this
  for reference parameters. C++ treats a named `T&&` parameter the same way.
- **A `deep` reference or receiver:** the caller said any duplicate must be
  independent. Starting at `copy` would silently share the caller's resources,
  so `deep` stays the default. An explicit `as copy` is allowed, and makes the
  callee's choice to share visible.
- **A by-value parameter, including `deep`:** it is the function's own value,
  already built, so later copies cannot affect the caller. `copy` is safe.
- **`loan` is not the body default** for by-value or `& copy` parameters. It
  would force a written stance on uses whose reader already assumes a copy, and
  protect nothing.

```zax
// illustrative
func final : ()(input : MyType move) = {
  something(input)           // copy, as written; MyType must support copy here
  something(input as move)   // the transfer is visible where it happens
}

publish final : ()(input : Archive readonly & deep) = {
  stored : Archive = input            // deep by default
  scratch : Archive = input as copy   // allowed: the callee visibly chose to share
  archive(input.history)              // deep: an unstanced member through a deep reference
}
```

What independence means is defined by the type. A member declared
`styles : StyleSet copy` is the type's author saying that member is shared even
under `deep`, and callers cannot override it.

*Correction:* the first version of this finding made `deep` reference
parameters and `deep` receivers start from `copy`. That would break the caller's
independence request.

##### D. The `transfer` contract on a type

`MyType :: type transfer <stances> { }` declares the **complete permitted
set** of the type's own whole-value operations for each stance. Those
operations are construction from the same type, assignment `=`, and
reconstructive replacement `.=`.

- An unlisted stance's operations are errors, whether written in the main body
  or added by a partial.
- **Without the clause**, the contract is whatever operations the type has now.
  There is no promise, so partials may add operations.
- `deep` is part of the list.
- Forbidding a stance does not forbid *offering* it. A type without `copy`
  can still be loaned, accepts implicit offers on `loan` references and
  receivers, keeps receiver stances on member functions, and keeps its other
  constructors.
- `loan` never appears in the list. It is not an operation of the type; every
  type can be loaned (T).
- **Order is enforced:** `copy move last deep`. It follows the fallback chain,
  with `deep` outside it. Out-of-order lists are errors with an automatic fix.

```zax
// illustrative
MyPoint :: type transfer copy {
  x : Integer
  y : Integer
}
// copy only, ever: a later move, last, or deep is an error, including from a partial

MyOwnedFile :: type transfer move {
  handle : Integer
}

useFile final : ()() = {
  file : MyOwnedFile
  inspect(file)                        // fine: a readonly & copy reference duplicates nothing
  other : MyOwnedFile = file           // error: no copy construction
  owner : MyOwnedFile = file as last   // fine: falls back to move; file is vacant
}

MyConnection :: type transfer copy move last {
  socket : MySocket * unique
}
// deep forbidden: an independent copy of a live connection has no meaning

MyBadOrder :: type transfer last move {
}
// error: transfer stances must be in the order copy move last deep
```

Std::unique_ptr-style move-only and `last`-only types are supported (H).

**Writing an unlisted operation is an error at the declaration.** Fallback only
finds operations that exist, so it can never reach a forbidden one:

```zax
// illustrative
MyOwnedFile :: type transfer move {
  handle : Integer

  +++ final : ()(source : MyOwnedFile readonly & copy) = {
  }
  // error: copy is not in MyOwnedFile's transfer list (transfer move)
}

MyPoint :: type transfer copy {
  x : Integer
  y : Integer

  +++ final : ()(source : MyPoint mutable writable & move) = {
  }
  // error: move is not in MyPoint's transfer list (transfer copy)
}

moved : MyPoint = point as move   // legal offer: falls back to copy; point unchanged
```

**An unlisted stance suppresses every generated shape for it.** A
`transfer move` type gets no generated copy constructor or copy assignment;
otherwise it would quietly receive an operation its own list forbids.

**`= forbidden` remains** for individual shapes of a *listed* stance, and for
its other uses: replacement constructors, assignment shapes, composition's
`forbidden family` fences, and ordinary functions. A type may list `copy` and
still forbid one copy shape:

```zax
// illustrative
MyMixed :: type transfer copy move {
  +++ final : ()(source : MyMixed readonly & copy) = forbidden
}
// legal: copy is permitted as a family; this one shape is removed
```

**A copy constructor is the specific single-source same-type shape.** Any
other constructor is an ordinary constructor, allowed in a move-only type even
when it reads another instance:

```zax
// illustrative
MyOwnedFile :: type transfer move {
  +++ final : ()(source : MyOwnedFile readonly & loan, mode : MyMode) = {
  }
  // legal: not a copy constructor; reopens the file without duplicating source
}
```

`& loan` is the honest stance here. A `& copy` source would grant permission to
copy a value that cannot be copied.

A named function such as `duplicate()` that builds a new instance through
another constructor is likewise allowed. It is not the copy operation, and the
call site shows what it does.

**Listing a stance is a permission, not a provision.** A generated operation is
unavailable when a member cannot supply it. That is an error only when
something tries to use it, following current demand-driven generation. The
diagnostic names the blocking member:

```zax
// illustrative
MyBox :: type transfer copy move {
  file : MyOwnedFile   // transfer move: no copy
}

box2 : MyBox = box   // error here, not at the declaration
```

```text
error: MyBox has no copy: generated copy needs MyBox.file, and MyOwnedFile
       has no copy (transfer move)
```

| Declaration | Copy constructor allowed? | `move` allowed? | What `as move` runs |
| --- | --- | --- | --- |
| `transfer move` | No | Yes | `move` |
| `transfer copy` | Yes, the generated one | No | `copy` |
| `transfer copy move` | Yes | Yes, now or later | `move`, or `copy` until it is written |
| No clause | Yes, unless `= forbidden` | Yes | whatever exists now |

Without a clause, a type may add `move` to an existing `copy`, in its body or
through a partial. The cost appears downstream: implicit last uses that feed a
plain parameter or a declaration start needing a stance. A `transfer copy move`
reservation avoids that.

##### E. Reserved stances count as implemented

A listed but unimplemented stance counts when deciding whether a stance
matters. It also decides the source's state afterward for analysis, even while
the operation falls back to `copy`. Code written against the promise then stays
valid when the implementation arrives.

```zax
// declarations from the start of section 9; MyHandle reserves move and last
// but implements only copy
useHandle final : ()() = {
  handle : MyHandle
  logHandle(handle)           // plain parameter
  // error at a last use: state intent, because a last offer would run the
  // reserved move constructor
}

reuseHandle final : ()() = {
  handle : MyHandle
  sendHandle(handle as move)  // copy runs today
  logHandle(handle)
  // handle counts as moved-from: allowed only because a moved-from value
  // keeps its full contract (G); this code stays valid once move exists
}
```

##### F. Any explicit stance satisfies the requirement

At a last use where the stance matters, `as copy` is a legal deliberate
choice. `intent<implicit-stance-at-terminal-use>{ }` remains available.

```zax
logHandle(handle as copy)   // fine at a last use: deliberate copy
```

##### G. What `move` and `last` leave behind

- **After `move`, the source keeps the type's full contract.** Every ordinary
  operation remains valid, though results may reflect an empty state. A type
  that cannot leave a fully valid value does not offer `move`. This replaces the
  current "the type's contract explains which operations remain meaningful".
- **After `last`, the source only needs to survive destruction.** Destruction
  and `.=` replacement are allowed without acknowledgement. Other uses need
  `intent<terminal-source-reuse>`. No marker identifies which operations are
  valid on a terminal value: the acknowledgement states intent, and the type's
  documentation says which operations make sense. Inside the type's own
  replacement constructor and destructor, no acknowledgement is needed (J).

Which operations a type implements follows from that:

- **`move`** when the type has a cheap, useful empty state.
- **A separate `last`** only when dropping "still usable" lets it take more or
  skip repair work. Examples: a private connection, a registration,
  self-referential repair.
- **Otherwise** `last` falls back to `move`. Most types declare
  `transfer copy move` or `transfer move`. Leaving `last` out of the list does
  not forbid `as last` offers; it promises `as last` behaves as `as move`.

```zax
// illustrative
buffer : MyBuffer                   // transfer copy move
next : MyBuffer = buffer as move
size := buffer.size()               // fine: an empty but valid buffer

command : MyCommand                 // transfer last
submit(command as last)
command .= MyCommand()              // fine: replacement makes command live again
command.run()
```

##### H. `last` without `move` is legitimate

The case is an owner whose every method assumes it holds its resource, such as
a non-null handle, a held lock, or a live registration. A usable empty state
would break that contract, so the type cannot offer `move`. It can offer
`last`, because the hidden released state is seen only by its destructor and
replacement constructor.

```zax
// illustrative
MyLockGuard :: type transfer last {
  // every method assumes the lock is held
}

holdUntilDone(guard as last)   // hands the lock to another scope
holdUntilDone(guard as move)   // error: no move, and no copy to fall back to

MyShared :: type transfer copy last {
  // always points to something: copy adds a reference, last takes the
  // source's reference without adding one; move would need a null state
}
```

There is no "declaring `last` requires `move`" rule. Instead, a diagnostic note
is raised when `as move` falls back to `copy` on a type that lists `last`:

```text
note: MyShared has no move; `as move` copied. At a final use, `as last`
      transfers without copying.
```

##### I. Caller rule: `as move` to reuse, `as last` when done

`as last` means "I am done with this variable". `as move` means "take the
contents; I will reuse the variable". **After `as last`, the source is
terminal for the caller's analysis, whatever operation ran,** even when the type
fell back to `move`. This changes current design, where the accepted consumer
alone determines source state. `as last` then means the same for every type,
and callers stay correct when a reserved `last` is later specialized.

`as last` at a final use is the recommended caller form even for a type with
no `last` operation. The stance is the caller's offer; the type's list says
what the type can do with it.

##### J. Refresh: `move` plus `.=` keeps a resource alive

This is the motivating use case for `move` versus `last`, and it uses the
current replacement-constructor design:

```zax
// illustrative transfer clause; `+++ replacement` is current design
DatabaseCommand :: type transfer copy move last {
  connection : PrivateConnection
  query : Query

  +++ replacement final : ()(nextQuery : Query) = {
    if _.connection.wasTaken() {
      _.connection .= PrivateConnection.open()
    } else {
      _.connection.resetSession()
    }
    _.query .= nextQuery
  }
}

runBatch final : ()(queries : Query[]) = {
  command : DatabaseCommand
  command.prepare(queries[0])
  submit(command as move)        // results move out; the connection stays
  command .= [{ queries[1] }]    // refresh: the replacement constructor reuses it
  submit(command as last)        // done: last may take the connection itself
}
```

**A replacement constructor must handle every state the destructor handles,**
including whatever the type's `last` leaves. After `as last`, `.=` runs the
replacement constructor normally.

How the type knows whether a resource is still present is part of its design.
`wasTaken()` above is the author's own method. Calls like it inside the
type's replacement constructor or destructor need no
`intent<terminal-source-reuse>`. A type may instead hold the resource through
something with a visible empty state, and check it with an ordinary operation:

```zax
// illustrative transfer clause
DatabaseCommand :: type transfer copy move last {
  connection : PrivateConnection * unique
  query : Query

  +++ replacement final : ()(nextQuery : Query) = {
    if _.connection {          // ordinary vacancy check, valid after last vacated it
      _.connection.resetSession()
    } else {
      _.connection = @
    }
    _.query .= nextQuery
  }
}
```

An always-holding type such as `MyLockGuard` (H) can keep that empty state
private: its methods assume the resource is present, and only its destructor
and replacement constructor check.

Withdrawn during discussion:

- a marker declaring that a replacement constructor accepts a terminal old
  value;
- a destroy-and-reconstruct default for `.=` on a terminal source;
- using the receiver-stance position for that marker. A receiver stance means
  the receiver was *offered* that stance for this call, not that it is in a
  terminal state from an earlier transfer.

##### K. Type-head clause order

`transfer` comes before `seal`. The type's own complete contract precedes
what others may add, and `seal`'s trailing modifiers stay at the end.

```zax
// illustrative
MyCatalog :: type transfer copy move seal open storage close callable {
}
```

##### L. Captured principle input: one enforced order

Maintainer input, a reusable language principle and not only a local rule:
where Zax has lists of modifiers or clauses, the compiler enforces one
canonical order so all code reads the same. Orders not yet defined must
eventually be defined. The stance order in D and the clause order in K are
applications of it.

Destination: decided in the dry run. `language/principles.md` is likely. This
item does not define orders for other constructs. Undefined orders are a
concern to raise when relevant, not a blocker.

##### M. A declared stance on a local, alias, or member is a requirement

Current design treats a declared stance as a default offer: with
`myType : MyType move`, an implicit `func(myType)` silently offers `move`, and
`as copy` can override it. Projection likewise makes a member's declared stance
the silent offer whenever the member is used. This finding replaces both. Every
**transfer** from a local, alias, or member must be written as its declared
stance or a stronger one. A transfer is any use that builds or assigns a new
value from the place: a by-value parameter, a local initializer, an
assignment, an array element, a capture, or a returned value. Loaning, which is
binding a `loan` reference or calling a method with a `loan` receiver, is not
a transfer (T). The requirement limits only which stance may be *written* for a
transfer. It does not limit which consumers an offer can reach through
fallback.

```zax
// illustrative
myType : MyType move

func(myType)            // error if func takes MyType by value: myType must not be copied
func(myType as move)    // fine
func(myType as last)    // fine: stronger than move
func(myType as copy)    // error: myType was declared move
show(myType)            // fine: show takes `& loan`; nothing is transferred
myType.describe()       // fine: an unmarked receiver is a loan (U)
inspectCopying(myType)  // error: takes `& copy`, which could copy myType
```

If loans were covered, a place declared `copy`, `move`, or `deep` could never
be passed to most functions or have most of its methods called, because
unmarked references and receivers mean `loan`.

This applies the necessity rule: no stance reads as copy, copying is forbidden
for this variable, so the stance is written. Aliases follow the same rule:

```zax
// illustrative
func final : ()(input : MyType move) = {
  inputAlias : MyType & move = input as move
  // `as move` is needed: input starts from copy in the body (C2)
  something(inputAlias)          // error if something takes MyType by value
  something(inputAlias as move)  // fine
  something(input)               // fine if MyType supports copy: input is unrestricted
}
```

A parameter's stance is a contract with the caller about how the value arrives.
A local's stance is its author restricting how that variable may later be used.
A function that wants the restriction on its input rebinds it, as `inputAlias`
does.

Rules:

- **A local declared `deep`** requires every transfer to be `as deep`. `deep`
  is outside the fallback chain, so nothing counts as stronger.
- **A variable's restriction does not extend to its members.** A member is
  governed by its own declared stance, which is itself a requirement, otherwise
  by the body default (C2). Neither inheritance from the containing declaration
  nor a member's stance as a silent offer survives from current projection
  rules:

  ```zax
  // illustrative
  Package :: type {
    label : Label move
    weight : Integer
  }

  package : Package move
  total := package.weight            // fine: weight declares no stance
  ship(package.label)                // error: label is declared move; write `as move` or `as last`
  ship(package.label as move)        // fine
  ```

- **A declared stance never settles an overload choice.** A place declared
  `copy` still needs `as loan` or `as copy` at a call where both a `loan` and a
  by-value overload fit (P). The reader of the call should not need to look up
  a declaration.
- **Declaring `loan` on a by-value local or member is an error.** `loan` is the
  weakest stance, so as a requirement it would restrict nothing.

- **Results keep their meaning as the producer's offer.** `result : Message
  move` is part of the public prototype and tells the caller what is offered.
  It is not a restriction.
- **Type aliases may carry a stance,** which declarations using them inherit as
  a requirement. `CopyView :: alias type DeepView copy` is fine normally.
- **An unsatisfiable declared stance is an error.** A declared stance on an
  alias or local is an error when the type's declared `transfer` list cannot
  satisfy it through any allowed offer. Fallback can complete a stronger offer
  with `copy`:

  ```zax
  // illustrative
  DeepView :: type transfer deep {
  }
  CopyView :: alias type DeepView copy
  // error: copy, move, and last are all unavailable on DeepView

  point : MyPoint move   // fine: MyPoint is transfer copy; `as move` falls back to copy
  ```

  For a type without a `transfer` clause, operations can still grow, so the
  error is reported where a transfer is attempted, as for D's demand-driven
  generation.

**The restriction travels through references because of `loan`.** A
restricted place may be loaned but not bound to a `& copy` reference, and
anything derived from a `loan` reference stays `loan` (V). The restriction
therefore cannot leak through a call:

```zax
// illustrative
byValue final : ()(input : MyType) = {
}

inspect final : ()(input : MyType readonly & copy) = {
  byValue(input)   // copies the referent
}

show final : ()(input : MyType readonly &) = {
  byValue(input)   // error: input is on loan
}

myType : MyType move
byValue(myType)    // error: myType must not be copied
inspect(myType)    // error: `& copy` could copy a move-restricted value
show(myType)       // fine
```

*History:* before `loan` was aligned, this finding accepted a local-only
restriction, matching C++ and Rust, which make copyability a property of the
type and use wrapper types for a per-object guarantee. `loan` made the
restriction a real guarantee without making stance a per-layer qualifier,
because the promise belongs to the reference's own declaration.

The restriction still covers only declared places. Values stored in array
elements, or in members that declare no stance, are governed by their type.
A guarantee for every value of a kind, wherever it is stored, comes from a
`transfer` list on the type or identity (D, N).

##### N. Identity types and `transfer`

An identity type inherits its underlying type's transfer operations when it
declares no `transfer` clause, and may declare its own. The clause belongs to
the identity, in its head, after the surface keyword and before `type`:

```zax
// illustrative
MyMoveOnlyType :: identity admit expose transfer move type MyType
MyCount :: identity admit expose type U32   // no clause: operations come from what exists
```

A trailing position (`type MyType transfer move`) is avoided because
`alias type DeepView copy` already uses a trailing stance as an overlay on the
type before it. The order follows L: admission, surface, `transfer`, then the
underlying type, with `seal` after `transfer` if identities can be sealed.

The identity's list is its own complete set of permitted stances. A stance
missing from it is forbidden for the identity even when the underlying type
supports it, whether the identity is `expose` or `opaque`. The list governs
**construction, assignment, and replacement from the identity's
representation**, which includes projection and `underlying value`:

```zax
// illustrative; MyType can be copied
mine : MyMoveOnlyType
other : MyMoveOnlyType = mine              // error: copy is not in the list
copyOut := mine as MyType                  // error: projection would copy-construct a MyType
movedOut := (mine as move) as MyType       // fine: moves the representation out
moved : MyMoveOnlyType = mine as last      // fine: falls back to move
```

Operations and functions are never restricted as such. An exposed operation is
affected only when it needs a forbidden construction, such as copy-constructing
a by-value `MyMoveOnlyType` parameter:

```zax
// illustrative
operator binary '+' final : (result : MyMoveOnlyType)(
  rhs : MyMoveOnlyType
) = {
}

a + b             // error: building rhs from b needs copy construction
a + (b as move)   // fine
```

An operator that takes its inputs by reference, and builds its result by a
permitted construction, works normally. Building through admission from a newly
computed underlying value is permitted.

Listing a stance is a permission, not a provision. An identity over a move-only
underlying type may list `copy`, but it gets a copy only if it declares its own
copy constructor, for example one that reopens a handle.

An identity's `transfer` clause is finding D applied to an identity, not a
separate feature. It turns a copyable type into a move-only kind whose every
value is restricted wherever it is stored: array elements, members, optionals,
and generic code. A declared stance on a place (M) restricts only that place.

```zax
// illustrative
report : MyType move
reports : MyType[] = [ report as move ]
copied := reports[0]              // allowed: array elements are governed by MyType

held : MyMoveOnlyType[]
copied2 := held[0]                // error: no value of MyMoveOnlyType can be copied
```

`opaque` is a separate choice: it decides whether the underlying type's other
operations come along automatically, and does not by itself make an identity
move-only.

##### O. `= default` for `move`, `last`, and `deep`

The compiler automatically generates only `copy`. `move`, `last`, and `deep`
are never generated automatically, because what a value leaves behind can
depend on its logical meaning, not only its storage. An `Integer` that counts
packets outstanding on the wire may need special handling in the context of the
type that holds it.

A type may explicitly ask for the language-defined member-by-member operation
with `= default`:

- **`move = default`** moves each member with its own `move`, falling back to
  `copy` for a member without one.
- **`last = default`** works the same way with `last`, falling back through
  `move` to `copy`.
- **`deep = default`** gives each member a `deep` copy, with no fallback.
- **Unavailable** when any member cannot supply the needed operation, as for
  any `default`. Owning pointers have no `deep`, so a type holding one cannot
  default its `deep`.

```zax
// illustrative
MyFolder :: type transfer move {
  file : MyOwnedFile
  name : String

  +++ final : ()(source : MyFolder mutable writable & move) = default
  // file moves with MyOwnedFile's move; name moves with String's move
}
```

Without that declaration, a structure holding a move-only member cannot be
transferred: its generated copy is unavailable and no `move` exists.

##### P. Where stance sits in overload selection

The model matches current design. Viability eliminates candidates first,
including those a `requires` clause rejects. Preference then compares what
remains. If nothing clearly wins, the call is ambiguous.

Stance is one independent dimension of the per-slot comparison, not a
priority over the others. A candidate wins only if it is no worse on every
dimension and better on at least one. A stance match never overrides a worse
conversion or qualification:

```zax
// illustrative; a Message converts to a Packet
send2 final : ()(input : Message) = {
}

send2 final : ()(input : Packet last) = {
}

send2(message as last)
// first: exact type, stance below the offer
// second: exact stance, needs a conversion
// each is better on one dimension: ambiguous, not decided by stance
```

Within the stance dimension, a declared stance equal to the offer is best,
then lower declared stances in fallback order. Value versus reference does not
count against either candidate. Because of B step 1, an implicit
offer never reaches a `move` or `last` parameter, so no-stance calls always
prefer the plain overload:

```zax
// illustrative
emit final : ()(input : Message) = {
}

emit final : ()(input : Message move) = {
}

emit final : ()(input : Message last) = {
}

emit(message)           // plain emit
emit(message as move)   // move emit
emit(message as last)   // last emit
```

`loan` sits at the bottom of the stance order (T). A `loan` parameter is
reachable by every offer, and `as loan` reaches only `loan` parameters.

**When both a `loan` reference and a by-value parameter fit an unstated call,
the caller must write `as loan` or `as copy`.** Neither wins silently. The two
can differ in more than cost: a mutable `loan` lets the callee change the
caller's value, which a by-value copy cannot, and a by-value callee may keep
what it receives:

```zax
// illustrative
func final : ()(input : MyType mutable writable & loan) = {
}

func final : ()(input : MyType) = {
}

func(value)           // error: both fit; write `as loan` or `as copy`
func(value as loan)   // the loan overload
func(value as copy)   // the by-value overload: exact match for copy
```

This is a plain requirement to write the stance, not an intent error: there is
no implicit behavior worth acknowledging. It applies the principle of A: when
nothing is stated and the choice matters, the programmer states it. An implicit
call and an explicit `as copy` therefore resolve differently here. That is
consistent with F, and is why "no stance" is taught as "a copy offer with no
stated intent" rather than as a silent `as copy`. The pair should be rare.

Later tie-breakers are unchanged: fewer defaults, non-generic over generic,
expected results. When a `& last` reference and a by-value `last` parameter both
remain viable, current design's preference for the reference is kept when the
caller's variable is mutable and writable, because the reference avoids building
another value.

##### Q. Generic bodies are checked per instantiation

The intent requirement is decided for each instantiation. No stance is
required in advance merely because some other type might need one:

```zax
// illustrative
sendEach$ final : (Items ... : type)()(items ... : Items) = {
}

forwardEach$ final : (Items ... : type)()(items ... : Items) = {
  sendEach(from items)
}

forwardEach(1, 2)        // Integer items: a last offer would still copy; no error
forwardEach(message)     // Message items: a last offer would run move; intent error
```

The caller cannot fix that line, so the diagnostic names the generic body as
the place to fix, and shows the call that produced the instantiation:

```text
error: last use of `items` needs a stated stance
  in forwardEach$(Message), line 2: sendEach(from items)
  instantiated from: forwardEach(message), main.zax:14
  this line belongs to forwardEach$; it needs `as copy`, `as move`, or `as last`
```

A library can test representative instantiations with
`compile{ assert(wellformed{ forwardEach(myMessage) }) }`. A `requires` clause
can reject item types, but cannot make a stance unnecessary for an accepted
one.

##### R. Raw pointers enter `unique` only with `last`

Any raw pointer, scheduled or ordinary, enters a `unique` destination only with
`as last`. Without `last`, the operation is a borrow, and a `unique` cannot
hold a borrow:

```zax
// illustrative
scheduled : MyValue * = @
owner1 : MyValue * unique = scheduled             // error: unique cannot borrow; write `as last`
owner2 : MyValue * unique = scheduled as last     // adopts; scheduled becomes vacant
owner3 : MyValue * unique = ordinaryRaw as last   // also needs proof of sole ownership, or unsafe
```

Current pointer design already states that a scheduled source offered with
`last` is adopted and left vacant, its cleanup still running but finding
nothing. It also states that an ordinary raw source needs proof of an allocation
root and sole disposition authority, or `unsafe`. This finding adds the `last`
requirement for every raw-to-`unique` transfer.

Disposition moves only into a `unique` pointer or a scheduled declaration, and
either way `last` is required by necessity (A). Scheduled pointers therefore
need no special exception in the intent rule, beyond the current result-mapping
rule for scheduled results. Promotion confirms one detail the pointer owner
does not state: whether an ordinary raw source becomes vacant after `as last`.
The finding expects yes, to match the scheduled case.

```zax
// illustrative
keep final : ()(target : MyValue * unique) = {
}

borrow final : ()(target : MyValue *) = {
}

borrow(scheduled)         // borrows the address: no stance can change anything
keep(scheduled)           // error at any use: write `as last`
keep(scheduled as last)   // keep owns the allocation; scheduled is vacant
```

##### S. Corrections to existing material, applied at promotion

- **Raw input `forward(resource)`.** With a plain by-value parameter and a
  `Resource` that has `move`, this is an intent error at a last use under B.
  The example stays valid, but its explanation must say that the type's `move`
  constructor is the reason.
- **Variadics wording** "an ordinary by-value parameter could" take an item
  over needs the type condition. A plain parameter matters only when the item's
  type has `move` or `last`.
- **`forwardAll$`** in transfer-stances and variadics: `sendAll` declares
  `items ... : Items last`, so an implicit offer is not viable for any item
  type. The error is a necessity error at any use, not a last-use intent
  error, and holds for every instantiation, including `Integer`. Its comment
  should say so.
- **Section 1's `countDown(value : Integer last)`** is a necessity error at any
  use (B step 1), not a last-use intent error.
- **Documentation stance lists** (`copy`, `deep`, `move`, `last` in
  transfer-stances' table and metadata) should follow the enforced order.
- **`identity-types.md`'s `operator binary '+' contextual existing`** lacks `=`
  and a prototype. Everywhere else, `existing` is a declaration state written
  after `=` on a full prototype. `contextual` itself is used correctly as the
  completion mode from `operators.md`, whose keyword position remains open:

  ```zax
  // illustrative; contextual's exact position is still open
  operator binary '+' contextual final : (
    result : MyContextualInteger
  )(
    rhs : MyContextualInteger
  ) = existing
  ```

- **The pointer owner** gains R's `last` requirement and the vacancy detail.
- **Existing `readonly & copy` examples that only read** become unmarked
  (`loan`) references, under U. Examples that do copy through the reference
  keep `& copy`.
- **Transfer-stances' open question about returning a parameter at its final
  use** is closed by X.
- **Transfer-stances' projection precedence** loses inherited and member-offer
  steps (M). Its by-value, receiver, and declaration-stance sections follow B,
  C2, and M.

##### T. `loan`: a reference that never transfers

`loan` is a fifth stance. A `loan` reference promises that nothing leaves the
owner through it: no copy, `move`, `last`, or `deep` is taken from the
referent. The same instance is used in place and comes back. Whether the
borrower may change it is decided separately by `readonly` or
`mutable writable`. Lending someone a car does not stop them filling or
burning its fuel; they may not keep it or have it duplicated.

```zax
// illustrative
show final : ()(input : MyType readonly & loan) = {
  print(input.label)      // fine: reading a member is not a transfer of input
  byValue(input)          // error: input is on loan; this would copy it
}

fill final : ()(buffer : MyBuffer mutable writable & loan) = {
  buffer.append(data)     // fine: mutation is the qualifier's business
}
```

It fits the family as the zero point of "how much of the value leaves its
owner":

| Stance | What leaves the owner | Owner's value afterward |
| --- | --- | --- |
| `loan` | nothing | the same instance, possibly changed through a mutable reference |
| `copy` | a duplicate | unchanged |
| `deep` | an independent duplicate | unchanged |
| `move` | the contents | live and moved-from |
| `last` | everything | finished |

Rules:

- **Order:** `loan < copy < move < last`, with `deep` outside the chain.
  Sequences of stances use the order `loan copy move last deep` (L).
- **Fallback:** an offer may be accepted by a consumer that takes less, so a
  `loan` parameter accepts every offer. A `deep` offer reaches `loan` consumers
  that build nothing, because no independence is lost, and never a `loan`
  constructor.
- **`as loan`** restricts a call to `loan` parameters, so a reader sees that
  the call takes nothing.
- **References only.** A by-value parameter always builds a new value, so
  `input : MyType loan` by value is an error. Declaring `loan` on a by-value
  local or member is an error (M).
- **Never in a `transfer` list** (D).
- **`as last` reaching a `loan` parameter** still leaves the caller's variable
  finished (I). No diagnostic: the caller said it was done, the callee may take
  ownership in a future version, and any later use is diagnosed normally.
- **`loan` results** are allowed: the caller may use the referent under its
  qualifiers, but may not copy or keep it. They follow V, and reference-result
  lifetime rules apply unchanged.

  ```zax
  // illustrative
  current final : (result : MyDocument readonly & loan)() = {
  }
  ```

- **Members are not protected.** A `loan` prevents duplication through the
  type's own copy operation, not deliberate member-by-member reconstruction.
  Teach this plainly. A firm guarantee comes from the type's `transfer` list and
  private members.

*Names considered:* `borrow` collides with "borrowed raw pointer" and a
callable's "borrowed copy"; `view` with slices and identity views; `observe`
with weak-pointer observers; `peek` implies read-only, which is not the promise;
`lend` reads backwards on the receiving side; `loaned` and `lent` are
participles, unlike the other stances. `loan` had no existing use in
`language/`.

##### U. Unmarked references and receivers mean `loan`

An unmarked reference parameter, and an unmarked receiver, mean `loan`. Copying
or taking through a reference is declared with `& copy`, `& move`, `& last`, or
`& deep`, or with the matching receiver stance:

```zax
// illustrative
show final : ()(input : MyType readonly &) = {
}
// unmarked: loan

remember final : ()(input : MyType readonly & copy) = {
}
// declares that it may copy

MyType :: type {
  describe final : (result : String)() = {
  }
  // unmarked receiver: _ is on loan

  snapshot final : (result : MyType)() copy = {
    return _ as copy
  }
  // copies its receiver, so declares a copy receiver
}
```

The unmarked receiver stays the universal fallback in receiver selection; its
body is now a `loan`. Lifecycle sources are unaffected, because their stance is
always written.

*Why:* most functions and methods never copy through a reference, so most
signatures do not change. The ones that do copy become visible, and restricted
places (M) stay usable.

##### V. Everything derived from a `loan` reference stays `loan`

Anything derived from a `loan` reference that can reach the whole value is
itself `loan`. One rule closes every leak path:

- a reference bound from it must be `loan`;
- a lambda that captures it sees a `loan` inside;
- an identity projection through it is a `loan` view, so a projection that
  would copy is refused;
- forming a raw pointer from it is refused outside `unsafe`, because raw
  pointers carry no stance.

```zax
// illustrative
show final : ()(input : MyType readonly & loan) = {
  alias : MyType readonly & copy = input   // error: a copy-capable reference from a loan
  later := [[ input ]] ()() {
    keep(input)                            // error: input is still on loan here
  }
}
```

##### W. Lifecycle sources

Constructors, assignment, and replacement keep their source stances: `& copy`
(the default and generated form), `& deep`, `& move`, and `& last`. In these
signatures the source's stance names the operation being implemented, which is
what makes a `copy` offer select the copy constructor. It does not grant the
body permission.

> Inside a constructor, assignment, or replacement, the source as a whole is
> never transferred. Its members are handled with the authority the
> operation's stance grants.

| Lifecycle source | Whole source | Members without their own stance |
| --- | --- | --- |
| `& copy` | never transferred | `copy` |
| `& deep` | never transferred | `deep` |
| `& move`, `& last` | never transferred | `copy` by default; write `as move` or `as last` to take |

This also prevents accidental recursion:

```zax
// illustrative
+++ final : ()(source : MyType readonly & copy) = {
  _.label .= source.label     // fine: member copy
  whole : MyType = source     // error: would transfer the whole source, and recurse
}

+++ final : ()(source : MyType mutable writable & move) = {
  _.label .= source.label as move   // fine: takes a member
  whole : MyType = source as move   // error: would recurse
}
```

Generated and `= default` versions write the member transfers.

- **A same-type `& loan` source is an error.** It would be the copy operation
  under another name, and serving a `deep` offer it would silently produce a
  value that is not independent.
- **Other parameters** of constructors, assignment, and replacement are ordinary
  parameters, and `& loan` is often their honest stance (D's reopen
  constructor; a replacement that reads a template; an assignment from another
  type's settings).
- *Rejected:* writing `+++ final : ()(rhs : MyType readonly & loan) copy`, with
  the trailing position naming the operation. It would work for ordinary
  constructors only. Assignment already uses that position for its destination
  receiver. Replacement constructors have an old value `_` that a trailing
  stance would appear to describe. `move` and `last` sources need real
  authority and could never be `loan`. Lifecycle declarations would end up in
  two styles.

##### X. Returning a source at its last use

A `return` builds the result from its source, so it follows the same rules as
any transfer. The intent requirement applies when a `last` offer would run a
different constructor:

```zax
// illustrative
relay final : (result : Message)(input : Message) = {
  return input   // last use: Message has move; state `as copy`, `as move`, or `as last`
}

relayCount final : (result : Integer)(input : Integer) = {
  return input   // no stance needed: Integer has only copy
}
```

This closes the existing transfer-stances open question about returning a
parameter as its final use.

##### Y. Teaching obligation: declared stances by location

Maintainer direction for promotion: a declared stance means something
different in each location, so one place must explain every location, with
tables. The locations are by-value and reference parameters, lifecycle
sources, locals and aliases, members, results, receivers, and type and identity
`transfer` lists. The current spread of rules across B, C, C2, D, M, N, T, U,
and W is evidence for the author, not a presentation plan.

##### Z. References you declare versus bindings the language creates

Aligned from dry run 1's DQ1.

- **A reference you declare** is a new reference with its own stance.
  Unmarked, that stance is `loan`, consistent with parameters and receivers
  (U). This includes typed loop bindings:

  ```zax
  // illustrative
  owner : Buffer move
  alias : Buffer mutable writable & = owner             // unmarked: loan
  consume(alias)                                       // error: alias is on loan
  taker : Buffer mutable writable & move = owner as move
  consume(taker as move)                               // fine

  each value : MyValue readonly & from values {        // declared reference: loan
    inspect(value)
  }
  ```

- **A binding the language creates** is a name for an existing path, and
  behaves exactly like writing that path. Variant case bindings and untyped
  loop bindings are of this kind. They follow M for members and V for paths
  through a `loan`. No silent projected stance remains, and no selector stance
  is needed:

  ```zax
  // current variant syntax; rules from M and V
  message : MyMessage

  switch message {
    case owner {
      use(owner)               // same as use(message.owner)
      consume(owner as move)   // same as consume(message.owner as move): allowed
      consume(owner)           // a copy offer; a unique pointer has none, so write `as move`
    }
  }

  inspect final : ()(message : MyMessage readonly &) = {
    switch message {
      case owner {
        consume(owner as move) // error: message is on loan, so message.owner is too
      }
    }
  }
  ```

  The existing stance-on-source form for iteration, such as
  `in messages as last`, still means that stance at each use.

*Withdrawn:* dry run 1 proposed that a stance on the selector, as in `switch
message as move`, would grant the bindings authority. Treating the binding as
its path made that unnecessary.

##### AA. Inference takes the type only

Aligned from dry run 1's DQ2. An inferred declaration takes its initializer's
type, never a declared stance. Since declared stances are requirements (M),
inheriting one would restrict a variable invisibly:

```zax
makeMessage final : (result : Message move)() = {
}

stored := makeMessage()   // the result's move is the producer's offer for
                          // building stored; stored itself is unrestricted
copyOfStored := stored    // fine
```

This replaces current text saying an inferred declaration "may adopt the
concrete value type and transfer stance of its initializer".

##### AB. Answers from the example inventory

These were aligned from dry run 3's DQ3 to DQ9.

- **Temporaries keep their inherent `last` (DQ3).** A directly constructed
  unnamed temporary still offers `last`. It is a property of the source form,
  and nothing observable is left behind. It is not an inference about a named
  place, so it does not conflict with A.

  ```zax
  fromTemporary := (: Source).get()   // the unnamed receiver offers last
  ```

- **A result's declared stance is its outward offer only (DQ4).** Inside the
  producer's body, `result` starts from `copy` like a parameter, and a transfer
  from it is written. The declared stance is what the caller receives.

  ```zax
  // illustrative
  makeMessage final : (result : Message move)() = {
    result.+++(source as copy)
    inspect(result)            // a copy offer, or a loan if inspect loans
    archive(result as move)    // a transfer from result is written
  }
  ```

- **Receiver selection needs no new rule (DQ5).** An exact receiver beats the
  unmarked fallback, so an unstated call selects an exact `copy` receiver when
  one exists, otherwise the unmarked (`loan`) receiver.
- **Generated forwarding keeps its table (DQ6).** Composition's `= via`
  wrappers write their member offers explicitly: `last` for a plain, `move`,
  or `last` by-value outer parameter, `deep` for a by-value `deep` one, and the
  caller's authorization for a reference. Body defaults (C2) apply only where a
  stance is unstated.

  ```zax
  Tank :: type {
    fuel preferred : Fuel
    refill final : ()(source : Tank) = via fuel.refill
  }
  // generated body, by hand: _.fuel.refill(source.fuel as last)
  ```

- **A copy capture of a `loan` reference is an error (DQ7).** It is a transfer
  from a `loan`. A reference capture of it stays `loan` (V). The lambdas owner's
  capture syntax decides how the fix is spelled.

  ```zax
  // illustrative
  view : Document readonly & = source       // unmarked: loan
  callback := [[ view ]] ()() {             // error if this captures a copy of view
    inspect(view)
  }
  ```

- **`_` is the receiver path, not a formed pointer (DQ9).** Passing `_` on
  carries the receiver's `loan` (V). V's refusal of raw pointers applies only
  to pointers the programmer explicitly forms from a `loan`. An
  `OpaqueReferenceObserver` built from a `loan` reference follows V.
- **The generated `=` returns `self` (DQ8).** A `self` result carries no stance
  of its own. It is the caller's receiver path, with that path's authority: its
  declared stance requirement, any `loan` it was reached through, and its
  qualifiers. A chain therefore behaves exactly like writing the destination
  again:

  ```zax
  // illustrative
  first = second = third                 // like second = third; first = second
  restricted : MyType move
  first = restricted = third             // error: like first = restricted
  first = (restricted = third) as move   // fine
  (a = b).describe()                     // fine
  ```

  This extends `self` results. Today `self` only verifies provenance. It now
  also means that the result is the caller's receiver path. A programmer-written
  `=` declares its own result under ordinary rules: a by-value result is a new
  value offered by the producer, a reference result follows its declared stance
  and lifetime, and `result self` chains like the generated form. A `=` cannot
  hand out more than it was given: returning `_` as `& copy` from an unmarked
  (`loan`) receiver is an error under V, and a `copy` receiver cannot be bound to
  a move-restricted place under M.

##### AC. A `self` result is the receiver exactly as the caller wrote it

Aligned 2026-11-14. This replaces the `self` part of AB (DQ8), which made a
`self` result the caller's variable but left the stance written on the call
behind.

A `self` result is the caller's receiver **exactly as the caller wrote it**:
the variable, its declared stance, any loan it was reached through, its
qualifiers, **and any stance written on the call**. The chosen version's own
receiver stance and qualifiers do not change the result. They still decide
which calls can use that version and what its body may do with `_`.

```zax
// illustrative
(builder as move).setName(name).setSize(size).build()
// build receives move: the stance written at the head of the chain carries
// through every self result, just as a stance on an iteration source applies
// at each use

value : Engine
value.peek().peek()          // fine: like value.peek(); value.peek()

(writableValue as move).inspect().refill()
// inspect has no receiver stance (a loan, readonly); its self result is still
// writableValue written as move, so refill receives move and may write
```

Rules:

- **Declaration form.** A `self` result is written with only its type and
  shape: `result self : Engine &` or `result self : Engine *`. The type must
  be the receiver's type. Writing a stance or qualifiers on a `self` result is
  an error, because the result takes both from the caller, and anything
  written there would mislead readers.

  ```zax
  // illustrative
  peek final : (result self : Engine &)() copy readonly = {
    return _
  }
  // fine

  peekRestated final : (result self : Engine readonly & copy)() copy readonly = {
    return _
  }
  // error: a self result takes its stance and qualifiers from the caller
  ```

- **Receiver selection limits what a chain can call.** A receiver written
  `as move` selects only a `move` version or the version with no receiver
  stance. It never selects a `copy` version, because versions with a receiver
  stance do not fall back to each other. So `(value as move).refillCopy()`,
  where `refillCopy` has only a `copy` receiver, is an error, not a call that
  returns `copy`.
- **`as last` chains.** Every call in the chain receives `last`. The first call
  that actually takes the value leaves the variable finished, and any later
  call in the chain is then a use after `as last`, which is an error.
  `(value as last).inspect().finish()` is fine, because the loan call takes
  nothing first. `(value as last).finish().inspect()` is an error.
- **The variable's own rules still apply.** The stance written at the head of
  the chain must satisfy the variable's declared stance:
  `(restricted as move).refill().send()` is fine for `restricted : T move`,
  and a chain on `restricted` with no stance is an error at its first call
  that would copy it.
- **The generated `=` still returns `self`.** `first = second = third` still
  behaves like `second = third` followed by `first = second`. With a stance at
  the head, `(second as move) = third` passes `move` on through the result.
- **Programmer-written functions that return `_` some other way** follow the
  ordinary rules. Returning `_` as a `& copy` reference from a receiver that
  is a loan is still an error.

Owners to update: `function-invocation.md` (`self` results, and the receiver
section if needed), `transfer-stances.md` (the `self` results section, the
assignment chain example, and the receiver section's chain behavior),
`construction-and-destruction.md` (generated `=`), `lifetimes-and-references.md`
(receiver-origin `self`), and `terms.md` (`self` result).

##### AD. Refinements to AC

Aligned 2026-11-14. AD refines AC, and AF refines item 2 of AD; where they
differ, the later finding wins. Syntax is *illustrative*.

**1. A `self` result passes the stance along; it does not use it up.** A call
that returns `self` acts on the owner itself, so it cannot take the variable
over. It hands the caller's receiver on unchanged: the variable, the stance
written on it, and its qualifiers. The stance reaches the first consumer that
does not return `self`. A `readonly` function that returns `self`, called on a
writable variable, gives back a writable path for the same reason.

```zax
// illustrative
takeOwnership(input as last)                   // takeOwnership may take input
takeOwnership((input as last).doSomething())   // doSomething returns self and only passes
                                               // input on; takeOwnership may still take it
```

The rule that `as last` leaves the variable finished, whatever ran, applies to
the whole expression and its final consumer, not to each call in the chain.
This replaces AC's "`as last` chains" rule. AC's examples
`(value as last).inspect().finish()` and `(value as last).finish().inspect()`
assumed that a call in the chain takes the value; they are dropped. When a
call in a chain does not return `self`, the ordinary result rules apply: its
declared result decides what the next consumer receives, and the stance
written on the receiver does not carry through it. The agent checked those
rules against current owners and found no conflict.

**2. Taking from `_` in a function that returns `self`.** Emptying `_` is
ordinary. After `move`, `_` is still a fully usable value (G), so the value
handed on is valid, possibly empty:

```zax
// illustrative
takeOwnershipOfRemainingMessage((myMessage as last).sendNextBuffer())
// inside sendNextBuffer, send(_.latestBuffer as move) is ordinary:
// myMessage stays a valid message, and takeOwnership... takes the rest
```

None of these need an acknowledgement. Whether each is allowed follows from the
receiver stance of the version that runs: a version with no receiver stance
cannot take anything, and a `move` version may.

```zax
// illustrative, inside a function that returns self
send(_.latestBuffer as move)
send(_.buffering.latestBuffer as move)
_.buffering.transferOutBufferViaMove()
_.transferOutBufferViaMove()
_.flush()                               // clears contents: ordinary mutation
```

Taking a member with `last`, such as `send(_.latestBuffer as last)`, also
needs no acknowledgement, in a `move` version as well as a `last` version. A
member follows its own declared stance, and leaving one taken or depleted is
an ordinary use of writable access (AF item 2).

Passing the **whole** `_` on with `move` or `last` is different. It is
covered by AF, which applies in every function, not only those returning
`self`.

**Withdrawn during discussion:**

- Making `result self` on a `last` version a declaration error. A ban was
  more than needed; AF asks for an acknowledgement instead.
- Letting `as last` select a `move` version when that version returns `self`.
  A type may declare a `last` version that returns `self`, so the ordinary
  receiver selection rules stand.
- Treating `return _` after `_ as last` as `terminal-source-reuse`. Passing
  `_` on is the act that needs to be visible, not the later `return`.

**3. A `self` chain whose result is discarded needs no diagnostic.** When the
`self` result may be discarded (AF item 3), `(value as last).inspect()` and
`(second as last) = third` leave the variable finished, and its destructor
runs at the end of its scope as usual. Nothing requires a value to be passed
anywhere; when it is passed on, `as last` is the stance to consider. The
maintainer's reason: ending an unused value is what a destructor is for.

**4. Weaker stances, and replacing a stance partway along a chain.** Writing a
stance weaker than the variable's declared stance is an error wherever it is
written, including partway along a chain, because the `self` result is that
variable. A later stance replaces an earlier one, as in `source as move as
copy`. `as loan` applies to that one use only and never changes the variable.

```zax
// illustrative
value : MyValue move

// each line below is a separate alternative
(value as copy).foobar()            // error: value was declared move
(value as last).foobar()            // fine: stronger than move
(value as loan).foobar()            // fine: nothing is transferred
keep((value as loan).foobar())      // error: the self result is value written as loan,
                                    // and a loan cannot be copied
(value as last).inspect() as move   // fine: move replaces last; value ends moved-from,
                                    // not finished
```

After `(value as loan).foobar()`, `display(value)` and every other use follow
`value`'s declaration as before.

**5. `self` is allowed on reference results only.** A pointer `self` result
can be neither null nor anything but the receiver, so it offers nothing a
reference does not. It would also give `as last` two meanings in one
expression, because on a raw pointer `as last` hands over ownership of the
allocation:

```zax
// illustrative, and no longer valid
owner : Engine * unique = (value as last).tunePtr()
// would read as taking ownership of value's storage
```

This replaces AC's `result self : Engine *` form. Owners that say "a reference
or pointer result declared `self`" change: `function-invocation.md`,
`lifetimes-and-references.md`, `composition.md` (the `self` result contract and
exposure), and `terms.md`. No live example uses a pointer `self` result.

**6. A chain cannot bypass the rules about uses of a variable.** A `self`
result is the variable, so passing it on is a use of that variable. Every rule
about uses of the variable applies at the consumer at the end of the chain:
necessity, the declared-stance requirement, the last-use intent rule, and use
after `as last`.

```zax
// illustrative; build returns self
// each line below is a separate alternative, at message's last use
relay(message)                     // error: state a stance
relay(message.build())             // the same error: relay still receives message
relay((message as last).build())   // fine
relay(message.build() as last)     // fine: the same stance, written later in the chain
```

The diagnostic points at `message`, at the start of the chain.

**Owners to update,** in addition to AC's list: `composition.md` (item 5).

##### AE. Correction: result mapping and reserved stances

Aligned 2026-11-14. `function-invocation.md`, "Result mapping and terminal
opportunity", in a staged hunk, states a condition as "or on a type that
specifically declares as having a transfer of `move` or `last`". It also says
there is no error when the result is discarded "unless a type specifically
indicates `move` or `last` support". Both garble E, which says reserved stances
count as implemented.

The intended meaning, in the maintainer's examples, rewritten in current
syntax:

```zax
// illustrative
MyType :: type transfer copy move last {
  // only copy is written so far
}

myType : MyType
callFunc(myType)    // last use: error without `as last`, `as move`, `as copy`, or intent

SomeOtherType :: type {
}

MyIdentityType :: identity admit expose transfer copy move last type SomeOtherType

myIdentityType : MyIdentityType
callFunc(myIdentityType)    // last use: the same error
```

A type or identity whose `transfer` list includes `move` or `last` needs a
stated stance at a last use, even when only `copy` exists today, because the
list promises that `as last` will run `move` or `last` later.

Fix at promotion: state the condition as "`as last` would call a different
function or run a different operation, counting the stances the `transfer`
list includes as if they were already written (E)". Wherever an owner says a
stance is needed "only where a different function would be called", or that
nothing could differ, the same correction applies. Known places:

- `transfer-stances.md`: "What decides whether a stance matters", the "no
  error when" sentence under "Last use when mapping a result", and the
  matching diagnostics bullet;
- `function-invocation.md`: the result-mapping bullets and the paragraph
  after them, and the summary of the two rules under "Terminal opportunity
  requires intent";
- `intent-acknowledgements.md`: the `implicit-stance-at-terminal-use` row of
  the category list;
- `variadics.md`: "only when the item's type has a `move` or `last`
  operation", which becomes "has or lists".

Search again at promotion for other wordings of the same condition.

**A discarded result is never a last-use error.** Remove the clause about
discarded results. Nothing receives the result, and its destructor ends it,
as for any unused value. This matches `transfer-stances.md` and item 3 of AD.
A result the function did not mark `#` must still be captured or discarded
with `#`; that existing rule is separate and unchanged. The maintainer's
example of a result a caller may reasonably ignore:

```zax
// illustrative
openWithDescriptor final : (
  file : FileResource,
  securityDescriptor # : SecurityDescriptorResource move
)() = {
}

file := openWithDescriptor()   // securityDescriptor is discarded and destroyed: fine
```

##### AF. Passing `_` on, stronger stances, and required `self` results

Aligned 2026-11-14. Syntax is *illustrative*; the category name was chosen by
the agent and may be revised by the maintainer.

**1. Passing the receiver on with `move` or `last` needs an acknowledgement.**
A call on a receiver reads as the value doing something, not as the value
being given away. A reader of `(task as last).submit()` cannot see that
`submit` hands `task` to a queue. So wherever a function passes `_` on with
`move` or `last`, the author states that it is deliberate:

```zax
// illustrative
Task :: type {
  submit final : ()() last = {
    intent<receiver-transfer>{
      queue.push(_ as last)        // passes _ on: needs the acknowledgement
    }
  }

  inspect final : ()() = {
    log(_)                         // no stance means copy: fine
    keep(_ as copy)                // fine
  }
}
```

This applies in every function, not only in functions that return `self`. It
is not a ban: a deliberate hand-off states its intent once, where it happens.

- **A reference made from `_` counts as `_`.** The compiler follows references
  made from `_`, as it already follows references made from a loan (V).
  Calling a function on such a reference is not a hand-off:

  ```zax
  // illustrative, inside a last version
  myself : Task mutable writable & last = _ as last
  myself.foobar()                  // a call, not a hand-off: fine
  intent<receiver-transfer>{
    adoptMe(myself as last)        // passes _ on
  }
  ```

- **A call on `_` is not a hand-off,** even when it is written
  `(_ as last).giveAway()`. If `giveAway` hands `_` on, its own body contains
  the acknowledgement. The rule therefore catches every hand-off exactly once,
  in the function that performs it.
- **Members are not covered.** `send(_.latestBuffer as last)` needs no
  acknowledgement (AD item 2).
- **`copy` and `deep` are not covered.** They leave `_` unchanged.
- **Where `_` is a loan,** passing it on is already an error (V), and the
  acknowledgement does not change that.

The category is `receiver-transfer`. It joins the category list in
`intent-acknowledgements.md`.

**2. A reference or receiver passes the whole value on with at most the
stance it was given.** A reference or receiver reaches someone else's value,
and its stance is the promise the caller relied on. Passing the whole value on
with a stronger stance would break that promise, so it is an error, not an
intent question. The caller cannot acknowledge it.

```zax
// illustrative
drain final : ()(input : Message mutable writable & move) = {
  keep(input as last)              // error: the caller allowed only move for the whole message
  keep(input as move)              // fine
  keep(input.payload as last)      // fine: payload follows its own stance
}

message : Message
drain(message as move)
message.send()                     // fine: the message as a whole is still usable (G)
```

**Members are not limited by this rule.** A member follows its own declared
stance (M). With writable access, a function may already change a member's
contents in any way, including leaving it empty, so taking a member with
`last` adds nothing new. Passing a value gives no promise about its members'
contents. Neither a flat error nor a required repair applies:

- a flat error would forbid taking resources out of an instance, which is an
  ordinary thing to do;
- requiring the member to be replaced before returning would assume the
  resource was meant to stay, which the compiler cannot know.

Teaching consequence: the compiler cannot see, from the caller's side, that a
member was left finished behind a reference.

```zax
// illustrative
drain(message as move)
message.payload.size()             // the compiler cannot tell that drain finished payload
```

This is the same as a function emptying the member through ordinary writable
access. Destructors and replacement constructors already have to cope with
whatever `last` leaves (J). Promotion states it once where this rule is
taught, so readers do not assume `as move` protects member contents.

Passing on with a **weaker** stance is always allowed. Each use picks its own
stance, and the parameter keeps its permission for later uses:

```zax
// illustrative
process final : ()(source : MyRecord mutable writable & move) = {
  show(source)                                 // lends it
  view : MyRecord readonly & copy = source     // a reference that may only copy
  snapshot := source                           // copies: no stance means copy
  keep(source as move)                         // still allowed afterward
}
```

A value the function owns, such as a by-value parameter, a local, or a
result, may be passed on with any stronger stance, apart from the existing
limits on `deep` and `loan`. Composition's generated wrappers already follow
the whole-value rule: an outer `& move` reference passes its member on as
`move`, "never as anything stronger", which the wrapper chooses for the
member it forwards. The consequence for receivers: `_ as last` can be written
only in a `last` version, and `_ as move` only in a `move` or `last` version.

This does not conflict with the advice to write `as last` when you are done
with a variable. That advice is for variables that hold their own value. A
reference's last use is not the last use of the value it reaches, so the
last-use intent rule already leaves references out. Through a reference, you
pass on what you were given, at most. Promotion should state this next to the
"`as move` to reuse, `as last` when done" habit in `transfer-stances.md`.

**3. A `self` result is required unless its author marks it `#`.** This
corrects the agent's earlier suggestion that a `self` result is always
discardable. A required `self` result lets a type insist that a chain
continues to a finishing call:

```zax
// illustrative
prepare final : (result self : MyValue &)() = { … }      // required
stageMore final : (result self : MyValue &)() = { … }    // required
finalize final : ()() = { … }

myValue.prepare().stageMore().finalize()   // fine
myValue.prepare()                          // error: prepare's result is required and unused
```

An author who wants a `self` call to be a complete statement writes `#` after
`self`: words first, then symbols.

```zax
// illustrative
tune final : (result self # : Engine &)(profile : Profile readonly &) = {
  apply(_, profile)
  return _
}

engine.tune(profile)          // fine: the result may be discarded
```

AC's declaration rule changes accordingly: a `self` result is written with its
type, the `&`, and optionally `#`. A stance or qualifiers on it remain errors.

**Examples to fix at promotion.** `operators.md` already says that an
operator which changes a place, such as `=`, declares its result with `#`. The
following `=` declarations do not:

- `construction-and-destruction.md`: the two `= existing` assignment examples,
  and the two generated `=` declarations shown under "conceptually supplies";
- `partials.md`: both `MyCatalog` `=` bodies.

No current example calls a `self` function as a statement and drops its
result.

**Owners to update for AF:** `transfer-stances.md` (receiver stance, `_` as the
receiver path, the stance habit, diagnostics), `function-invocation.md`
(`self` results), `intent-acknowledgements.md` (new category),
`construction-and-destruction.md` and `partials.md` (`#` on `=`), and
`terms.md` (`self` result).

##### AG. Switch tests lend the selector; unstanced results pass on as `last`; pointers and loans

Aligned 2026-11-14, from questions the cold reads raised.

**1. Every `switch` test lends the selector.** Each test calls a function on
the selector, so if the selector carried a stronger stance, an early test
could use it up before later tests run. Tests therefore receive the selector
as a loan, whether it is a named variable or a temporary, unless the `switch`
writes a stance on the selector:

```zax
// illustrative
switch makeReading() {     // a temporary selector
  case 'is hot'            // lends the selector: only a version of `is hot`
    cool()                 // with no receiver stance can be used
  case < upperBound        // the same temporary, untouched
    handleBelow()
}
```

A test operation that has only a `copy` version cannot be used as a test.
Maintainer direction: do not write copy-only test functions.

**2. A by-value result with no declared stance is passed on as `last`.**
Nothing can use a function's result again once the caller has used it, just
as with a value built in place such as `(: Source)`. So both kinds of unnamed
value are treated the same:

```zax
// illustrative
makeItem final : (result : Item)() = {
}

item := makeItem()         // built from the result with last: the move
                           // constructor runs if Item has one
keepCopy final : (result : Item copy)() = {
}
// a function that wants callers to get a copy declares it
```

This replaces the last-use check on results passed straight on (the result
part of A, and AE's result wording). A result with no stance can no longer
hide a choice, because it is always passed on as `last`. "No stance means
`copy`" remains the rule for variables. This applies to failure results too
(aligned 2026-11-14): `except.md`'s earlier "No implicit `move` or `last` is
added" only mirrored the old `copy` default for ordinary results, and no
failure-specific reason was found. A failure value, once bound in a handler, is
a named variable, so later uses of it with no stance are copies. A raw pointer result that allocates
with `= @` and declares no stance now hands ownership to its caller, so its
acknowledgement requirement is withdrawn.

**3. Loans and raw pointers.** A pointer's stance applies to the pointer
itself, not to the value it points to, so a raw pointer cannot carry a loan.
For this work item the rule stays: making a raw pointer from a loan,
including passing a loan `_` to a raw pointer parameter, is an error outside
`unsafe`. `nothing-instances.md`'s example that does this is corrected.

The maintainer directed that work item `039` design stances on pointed-to
values, through any nesting of `*` and `&`. The concern, the motivating
examples, the history of the per-layer decision, and the agent's suggested
direction are captured in
[raw pointer stances](../raw/pointer-stances.md).

**Owners to update:**

- `switch.md`: tests lend the selector.
- `transfer-stances.md`: temporaries and results, last use when mapping a
  result, source stability, and diagnostics.
- `function-invocation.md`: result slots and result mapping.
- `declarations-and-bindings.md`: initializers and result stance.
- `intent-acknowledgements.md`: the result uses of
  `implicit-stance-at-terminal-use`, and the raw-pointer result.
- `terms.md` and `principles.md`: where a stance comes from.
- `nothing-instances.md`: the `_` example.

##### AH. Decisions from the maintainer's review of the promotion

Aligned 2026-11-15. Applied directly to the promoted pages.

1. **A loan restricts only the whole value.** The whole value cannot be
   copied, moved, or taken through a loan. Members follow their own declared
   stance and the reference's access: through a `readonly` loan they can only
   be copied; through a writable loan they can also be moved or taken. This
   revises T, V, and C2, which said a loan's members could be copied but never
   taken. Stances are promises the compiler helps programmers keep about
   intended use, not a security barrier, and the teaching says so up front.
2. **Generated copy operations, listed.** For a type whose mutable and
   immutable values share one representation, the compiler generates eight
   declarations:
   - four copy constructors, from a `mutable readonly final`,
     `mutable readonly varying`, `immutable readonly final`, or
     `immutable readonly varying` source;
   - four `=`, from the same four sources, into a `mutable writable varying`
     destination.

   A reference parameter declared `mutable` accepts only mutable values, and
   `immutable` only immutable ones; binding an immutable value as
   `mutable readonly` is not allowed, so that immutability stays available to
   optimization. With separate shapes, nothing is generated between them.
   Leaving `copy` out of the `transfer` list removes all eight; `= forbidden`
   on one exact signature removes that one. Declaring an exact version with a
   receiver stance as `= forbidden`, such as a `last` receiver on `=`, removes
   only that version, and a call written with that stance is then an error
   instead of falling back.
3. **Generated declarations use named, incidental parameters,** such as
   `rhs [<label=incidental>]` and `result self # [<label=incidental>]`, not
   the anonymous form. Parameter names do not matter when a hand-written
   declaration replaces a generated one.
4. **No note when `as move` falls back to `copy`.** Fallback is the
   documented behavior, both for a type such as `MyShared` and for a reserved
   `move` not yet written. The source-state row for a reserved `move` becomes
   "unchanged": after `move` a value must still be fully usable anyway, so
   treating the original as moved-from added nothing the compiler could check.
5. **A stance written on a `switch` selector is an error.** Tests always lend
   the selector.
6. **A call with no stance on a receiver:**
   1. uses the version with no receiver stance, if the type has one; it lends
      the receiver;
   2. otherwise uses the `copy` version, if there is one, and the variable's
      declared stance must allow copying;
   3. otherwise is an error: any stronger version needs its stance written.

   A variable's declared stance is never applied: `myValue : MyValue move`
   calling `myValue.f()` lends it, and `(myValue as move).f()` uses a `move`
   version. A call on a type follows the same rule. This reverses AB's DQ5,
   under which a `copy` version beat the version with no receiver stance.
   Adding a `copy` version to a type therefore never changes existing calls.
7. **Answers recorded:**
   - `myInt : Integer last` requires `as last` on every transfer;
   - `func(myType as last)` with `func(myType : MyType & last)` on a
     `transfer move` type is legal;
   - `in messages as last`, not `last in messages`, puts a stance on an
     iteration source.

**Captures:**

- **[Raw pointer stances](../raw/pointer-stances.md)** gained:
  - `loan` and the other stances for pointer allocation;
  - array elements;
  - optional values;
  - who frees a `copy` `= @` result.
- **[Raw mutability-indexed type families](../raw/mutability-indexed-type-families.md)**
  gained the question of a type declaring which qualifiers it allows.
- **[Raw cross-cutting audit](../raw/cross-cutting-audit.md)** gained
  repository-wide checks:
  - keywords without backticks;
  - "not final" markers;
  - undefined canonical orders;
  - "automatic fix" language.

##### AI. Basic value types have `deep`

Aligned 2026-11-16, from the maintainer's final review. Without `deep` on
`Integer` and similar types, `= default` for `deep` would fail on almost every
type. The basic value types have `deep`, which does the same as their `copy`:
the integer types, fixed-point and floating-point numbers, `Boolean`,
characters, and enums. Composite types get `deep` only when every part has it.
Pointers, unions, and lambdas keep no `deep`.

This is stated per type, not as a "plain data" rule, because a number can name
a resource. Such an ID is given its own identity type whose `transfer` list
leaves out `deep`, so a type holding it cannot get `deep` from `= default`.

Applied:

- **`transfer-stances.md`:** the "Deep" section, with the handle-ID example,
  and four places that said `Integer` "has only `copy`", which now say it has
  no `move` or `last`.
- **`construction-and-destruction.md`:** the `= default` passage.
- **`identity-types.md`:** leaving `deep` out of an identity's list.

The scalar owners do not list transfer operations, so they were not changed.

#### 9.2 Dispositions of the former open questions

Each question below was raised on 2026-11-12 and decided in discussion. The
decision now lives in the named finding. The question text is not retained.

| Question | Decision | Finding |
| --- | --- | --- |
| O1. How generic bodies are checked | Per instantiation, with a diagnostic that blames the generic body. The `isStanceFree`-style predicate is not adopted. Stance is one independent dimension in overload selection | Q, P |
| O2. Generated `move` for structures | Rejected: only `copy` is generated automatically. `= default` for `move`, `last`, and `deep` is the explicit opt-in | O |
| O3. A containing type listing a stance its member cannot supply | Not an error at the declaration; an error only when the unavailable generated operation is used | D |
| O4. Same-type constructors with extra inputs | Not copy constructors; allowed in a move-only type | D |
| O5. `transfer` versus `= forbidden` | An unlisted stance suppresses its generated shapes; `= forbidden` remains for shapes of listed stances and its other uses | D |
| O6. Scheduled raw pointers | No special exception; every raw-to-`unique` transfer requires `last` | R |
| O7. Marking operations valid in terminal state | No marker. `intent<terminal-source-reuse>` outside the type; no acknowledgement inside its replacement constructor or destructor | G, J |
| O8. Other consumers besides parameters | Dissolved: they follow the type's ordinary construction and assignment rules, and by-value parameters now match them | B, M |
| O9. Identity types | Inherit operations without a clause; may declare their own `transfer` in the identity head | N |

#### 9.3 Captures

##### Resolved: stance as a per-layer qualifier

This was captured as a future concern when a declared stance on a local
protected only its own scope. It is resolved by `loan` (T, U, V): a restricted
place may be loaned but not bound to a `& copy` reference, so the restriction
travels without making stance a per-layer qualifier. No raw input is needed.
Current Zax's rule that "transfer stance is not a fourth per-layer qualifier
axis" stands.

##### Open question for the identity owner: `= existing` on an opaque identity

An `opaque` identity declares its own functions and operators. The natural
way to map an existing underlying operation would be `= existing`. Current
design defines `= existing` as satisfying a shape with "a compatible existing
operation", and does not say whether the underlying type's operations count as
existing for an opaque identity:

```zax
// illustrative identity body
MyToken :: identity admit opaque type U64 {
  operator binary '==' final : (result : Boolean)(rhs : MyToken readonly & copy) = existing
  // maps U64's == onto MyToken, if = existing can reach the underlying operation
}
```

*Why it can wait:* this item depends only on `transfer` governing an identity's
construction, assignment, and replacement (N), not on how its other operations
are declared.
*Reactivates when:* identity-body syntax is reviewed.
*Destination at promotion:* the identity-types boundaries, or an indexed raw
input.

##### Rejected: must-consume types

A must-consume (linear) type would make it an error for a value to reach the
end of its scope without being consumed. The only workable shape found was to
let a value end only inside its own type's member functions, such as a
database library's `commit` and `rollback`:

```zax
// illustrative only; rejected
Transaction :: type must consume {
  commit final : ()() last = {
    // inside its own type, the value may end normally
  }
}

transaction := database.begin()
(transaction as last).commit()
// leaving scope without commit or rollback would have been an error
```

*Rejected because:* something must eventually end the value's life, and
designating consumers adds a substantial feature with no current programmer
pressure. Recorded here only. No raw input is created.

##### Rejected: compiler checks for unguarded use of members `last` may take

The compiler could see which members a type's `last` transfers out, and warn
when its destructor or replacement constructor uses one without checking:

```zax
+++ replacement final : ()(nextQuery : Query) = {
  _.connection.resetSession()   // the proposed warning: last may have taken connection
}
```

*Rejected because:* how a type knows whether a resource is present is part of
its design (J), and destructors already carry this responsibility without
compiler aid.
*Destination at promotion:* [raw safety](../raw/safety.md), as a rejected
proposition with this reason, so it is not proposed again without new
pressure.

##### Enforced-order principle

Captured in 9.1 L. The destination is decided in the dry run.

#### 9.4 Promotion impact beyond section 8

The aligned model changes current rules in owners that section 8 treated as
summary-only:

- **Function invocation.** By-value parameter construction (B step 2) and the
  by-value sections. Preference within the stance dimension (P), including
  revisiting "exact mutable/writable `move` reference better than copy-in"
  now that by-value parameters can move, and the `loan`-versus-by-value rule.
  Unmarked reference parameters as `loan` (U), `loan` results (T), and return
  at a last use (X). Generic diagnostics (Q).
- **Transfer stances.**
  - `loan` as a fifth stance, its order, fallback, and `as loan` (T);
  - unmarked references and receivers (U), and leak paths (V);
  - lifecycle sources (W);
  - the by-value `move` and `deep` sections (B, C2);
  - the declaration-stance section, projection precedence, and type-alias
    overlays (M);
  - receiver body defaults, including `deep` (C2);
  - the open question about returning a parameter at its final use, closed
    (X);
  - the fallback and source-state tables (I);
  - the `move` and `last` sections (G, H);
  - single-owner resources (D);
  - the terminal-intent section (A, Q);
  - pointer transitions (R);
  - diagnostics and source stability.
- **Construction and destruction.** The `transfer` clause's effect on
  generation, `= forbidden`, and the copy-constructor shape (D); `= default`
  for `move`, `last`, and `deep` (O); the replacement constructor obligation
  (J); the lifecycle-source rule and the same-type `& loan` error (W).
- **Lambdas and callable composition.** Captures of `loan` references (V).
- **Every owner with reference examples.** `readonly & copy` examples that only
  read become unmarked references (S, U).
- **Type definitions.** The type-head clause and order (D, K).
- **Identity types.** The `transfer` clause in the identity head and its
  effect on projection and exposed operations (N); projection through a `loan`
  (V); the `contextual existing` correction (S).
- **Pointers and arenas.** The `last` requirement for raw-to-`unique`
  transfer, and the vacancy detail (R); refusing raw pointers formed from a
  `loan` outside `unsafe` (V).
- **Partials.** A declared contract restricts additions (D).
- **Intent acknowledgements.** Category text for both terminal categories (A,
  G).
- **Variadics and generics.** The variadics wording, `forwardAll$` (S), and
  per-instantiation checking (Q).
- **Raw terminal transfer intent.** Correct, then retire. **Raw safety** gains
  the rejected unguarded-member check.
- **`language/terms.md`** and **`language/principles.md`** (L), including the
  term `loan`.
- **Teaching shape.** One place explains declared stances by location, with
  tables (Y).

The scope is large. The maintainer chose to keep it in this work item, and may
revisit that if the dry run shows it is too big to promote in one change set.

### 10. Brief for the next session (2026-11-14)

The promotion is applied but not committed. A previous session made the
changes; the language maintainer may have staged some of them. This brief
tells the next session what to read, how to write, and what to do. It does
not authorize any edit by itself: **research first, report, and wait for the
maintainer's go before editing.**

#### Step 1: understand the current state

- Run `git status`, `git diff --cached --stat`, and `git diff --stat`. Report
  which files are staged and which are unstaged, and preserve that boundary.
- Read the change sets themselves (`git diff --cached` and `git diff`) for the
  files below, enough to know what was changed and how it now reads.

#### Step 2: learn how stances work before editing anything

Read, in this order:

1. `project/documentation.md`, especially "Write the way you would explain it
   to a colleague", "Example obligations", and "Promotion re-authors for a
   different reader".
2. `language/transfer-stances.md` in full. It is the main teaching document
   and the best summary of the model.
3. Section 9.1 of this file: findings A to AC. Later findings and corrections
   win over earlier text; AC replaces the `self` part of AB.
4. In `language/function-invocation.md`: "Transfer-aware value and reference
   parameters", "Receiver-origin `self` results", "Result mapping and
   terminal opportunity", the stance part of "Partial-order preference",
   "A loan or a copy must be stated", and "Terminal opportunity requires
   intent".
5. In `language/construction-and-destruction.md`: "Declaring which transfers a
   type allows", "Opting in to default move, last, and deep", "Generated copy
   construction and assignment", and "Lifecycle sources are never transferred
   whole".

#### Step 3: write for people, not as a specification

The maintainer rejected earlier promoted text because it read like a
specification, and it took a full rewrite to fix. Every sentence you write
must follow the rules in `project/documentation.md`. In particular:

- say who does what: the caller writes, the compiler chooses, the constructor
  runs;
- name the thing instead of writing "it" when more than one thing could be
  meant;
- no metaphors or invented labels ("head", "gains", "authorization",
  "ceiling");
- no design-discussion vocabulary. Write "`as last` would run the move
  constructor", not "a last offer would select the move constructor". Avoid
  "offer", "authority", "least offer", "consumer reaches";
- pair each rule with a short code example whose comments say what happens;
- state an exception in the same sentence as the rule;
- read every sentence you wrote, and every sentence a delegated writer wrote,
  before calling the work done. A search for known bad phrases does not
  replace reading.

#### Step 4: the specific change to make

*Done 2026-11-14; see "Follow-up promotion applied" at the end of this file.*
Apply finding AC, as refined by AD and AF, and the correction in AE, to the
owners they list. See dry run 5 for the exact change set. Today
those owners describe a `self` result as the caller's variable but do not
carry forward a stance written on the call, and some examples write
qualifiers on a `self` result. After the change:

- a `self` result is the receiver exactly as the caller wrote it, including
  any stance written on the call;
- `result self` is written with only type and shape (`&` or `*`), and a stance
  or qualifiers on it are errors. Fix existing examples that write qualifiers,
  such as `result self : Engine readonly &`;
- teach the `as last` chain rule, the receiver-selection limit, and the
  builder example from AC.

Before editing, report to the maintainer: the exact passages that change, in
which files, whether any are in staged or unstaged hunks, and anything AC does
not decide. Then wait for the go.

#### Step 5: remaining work after that

- Review the new rejected-proposal section in `project/raw/safety.md` against
  the writing rules. It has not been read for style.
- Older, unchanged text in the pointer and qualifier owners still uses
  spec-like words such as "disposition" and "provenance". At closure, record
  it in `project/teaching-debt-log.md` as a candidate for a separate teaching
  refresh rather than widening this promotion, unless the maintainer decides
  otherwise.
- Closure, each step after the maintainer's go: mark this file historical,
  archive it, propose work item 039, and create it.

Review status before this brief: all 27 changed language files were reviewed
for plain language. `transfer-stances.md` was read in full; the changed
sections of `function-invocation.md` and `construction-and-destruction.md`
were read in full; the other files were reviewed through their changed lines.
An independent naturalness review rated the result 7 out of 10, and every
issue it listed was then fixed. Local links all resolve.

## Dispositions and promotion dry run

### Dry run 1 (2026-11-13): FAIL

Read-only. It inspected the aligned findings against current owners by
targeted search and section reads. It did not re-read every owner in full;
items marked *verify* need a check during promotion, but do not change the
design.

**Result: FAIL.** Two design questions are not answered by the aligned
findings and would otherwise be decided silently during promotion. Everything
else has a coherent owner, reading path, and change set, recorded below as
provisional. After the two questions are aligned, a second dry run only needs
to revise the affected rows.

#### Design questions requiring alignment

**DQ1. What an unmarked local reference, or an implicit reference-shaped
binding, means.** U makes unmarked reference *parameters* and *receivers*
`loan`. It says nothing about local reference declarations or the bindings the
language creates for you. Current owners teach the old meaning for both:

```zax
// declarations-and-bindings.md, current
owner : Buffer move
alias : Buffer mutable writable & = owner
consume(alias)         // current text: alias offers `copy`
consume(alias as move) // current text: explicit `move` through this alias

// variants.md and transfer-stances.md, current
choice : MyChoice move
switch choice {
  case owner
    consume(owner) // current text: offers the projected move stance
}
```

Under M, binding the move-restricted `owner` to a copy-capable `alias` is an
error. So is the silent projected `move`. Iteration and `using` bindings raise
the same question.

*Agent recommendation:*

- **An unmarked reference anywhere is a `loan`**, including local reference
  declarations and implicit bindings. To copy or take through a local
  reference, declare it, as in `alias : Buffer mutable writable & move = owner
  as move`.
- **A stance written on a selector or iteration source gives the implicit
  bindings that authority.** Taking still has to be written at the use:

  ```zax
  // illustrative
  switch choice as move {
    case owner
      consume(owner as move)   // allowed: the binding has move authority
  }
  sendPackets(from each message : in messages as last |> encode(message as last))
  ```

This keeps one rule, that an unmarked reference never copies or takes, and
reuses the stance-on-source form variadics already teach.

**DQ2. Whether an inferred declaration adopts its initializer's stance.**
Current declarations-and-bindings and transfer-stances say an inferred
declaration "may adopt the concrete value type and transfer stance of its
initializer". Under M, a declared stance is a requirement. Inheriting one by
inference would restrict a variable invisibly:

```zax
makeMessage final : (result : Message move)() = {
}

stored := makeMessage()
copyOfStored := stored   // if stored inferred `move`, this is an error with
                         // nothing in the source saying so
```

*Agent recommendation:* inference adopts the type only, never a declared
stance. A result's stance is the producer's offer for constructing `stored`
(M); a restriction on `stored` must be written.

#### Ownership map

| Finding | Lasting owner | Other owners: local summary or link only |
| --- | --- | --- |
| A, E, F, I, X | transfer-stances | function-invocation (terminal sections), intent-acknowledgements |
| B steps 1 and 2, P | function-invocation | transfer-stances (by value and by reference) |
| B step 3, C, C2 | transfer-stances | function-invocation |
| D: what the list permits, generation, `= forbidden`, copy-constructor shape | construction-and-destruction | transfer-stances (callers' view), partials |
| D, K: type-head syntax and order | type-definitions | |
| G, H, J | transfer-stances (move and last); construction-and-destruction (replacement obligation) | |
| L | principles | terms |
| M | transfer-stances (declared stances by location) | declarations-and-bindings, structural-shapes |
| N | identity-types | |
| O, W | construction-and-destruction | transfer-stances |
| Q | transfer-stances (when a stance must be written) | generics (diagnostics) |
| R | pointers-and-arenas | transfer-stances (pointer transitions) |
| T, U, V | transfer-stances | function-invocation, lambdas, identity-types, pointers-and-arenas |
| Y | transfer-stances | |
| 9.3 `= existing` on an opaque identity | identity-types boundaries (future work by subject) | |
| 9.3 rejected unguarded-member check | `project/raw/safety.md` | |
| 9.3 must-consume types (rejected), per-layer qualifier (resolved) | this record only | |

There are no provisional teaching-debt observations to dispose of.

#### Structure proposal

- **No new files, moves, or routers.** Every concern has an existing owner.
- **transfer-stances remains one cohesive owner** and grows. It is not split:
  stances, where they are declared, and when one must be written are read
  together. Risk: length. The teaching plan below keeps ordinary use at the
  front, and the post-promotion cold-reader review must test it.
- **The `transfer` list is taught in construction-and-destruction**, because
  it limits the type's own operation family. Type-definitions shows only the
  head syntax and links there. Transfer-stances teaches what callers see. This
  avoids readers reconstructing the clause from three equal descriptions.
- **Raw:** `terminal-transfer-intent.md` retires, and its row leaves
  `project/raw/README.md`. `safety.md` gains the rejected proposition.
- **Focused reading:** unchanged. No ordinary route reaches raw or archived
  material.

#### Provisional change set

Current owners (`language/`):

- **transfer-stances.md:** restructure per the teaching plan; add `loan`;
  rewrite by-value and by-reference, declaration stance, projection, receivers,
  terminal intent, and diagnostics; close the return-at-final-use question;
  update Owns and Does Not Own; follow the stance order in lists.
- **function-invocation.md:** transfer-aware parameters (B, C, U);
  viability and preference (P), including the copy-in wording at the
  preference rules; terminal opportunity (A, Q, X); result mapping; `loan`
  results.
- **construction-and-destruction.md:** the `transfer` list (D); `= default`
  for `move`, `last`, and `deep` (O); lifecycle sources (W); the replacement
  constructor obligation (J); generated-operation text.
- **type-definitions.md:** the type-head `transfer` clause and its order
  relative to `seal` (D, K).
- **partials.md:** a declared `transfer` list restricts additions (D).
- **identity-types.md:** the identity-head clause, and its effect on
  projection and exposed operations (N); projection through a `loan` (V); the
  `contextual existing` correction (S); the `= existing` future-work note.
- **pointers-and-arenas.md:** `last` for raw-to-`unique` transfer, the vacancy
  detail (*verify* against the scheduled case), and raw pointers from a `loan`
  (R, V).
- **declarations-and-bindings.md:** reference declarations and inferred
  declarations (DQ1, DQ2); alias property overlays as requirements (M).
- **variants.md:** case bindings (DQ1).
- **lambdas-and-callable-composition.md:** capture stances as requirements
  (M); captures of `loan` references (V).
- **intent-acknowledgements.md:** the category text for
  `implicit-stance-at-terminal-use` (A).
- **variadics.md:** the stance-per-item wording; `forwardAll$` as a necessity
  error (S).
- **generics.md:** the per-instantiation diagnostic for transfer intent (Q),
  as a short link to transfer-stances.
- **operators.md, operator-catalog.md:** `as loan` among the generic stance
  forms.
- **qualifiers.md:** the stance list beside "not a fourth per-layer qualifier
  axis"; `loan` as a reference's stance, not a qualifier.
- **structural-shapes-and-compatibility.md:** anchored regions no longer
  "inherit" a declaration's `last`. A region is governed like a member (M).
  Update the stance lists.
- **terms.md:** `loan`, the `transfer` clause, declared stance as a
  requirement, five stances.
- **principles.md:** the enforced canonical-order principle (L); the stance
  list.
- *verify, likely unchanged:* nothing-instances (receiverless calls with a
  `loan` fallback receiver), optional-values, arrays-and-slices, unions,
  except, lifetimes-and-references, composition (wrapper offers), and
  integer-literals.

Project files:

- `project/raw/terminal-transfer-intent.md`: retire.
- `project/raw/README.md`: remove its row.
- `project/raw/safety.md`: add the rejected unguarded-member check, with its
  reason.

Explicitly not changed: `project/README.md`, the operating-prompt sources, and
the archive. Those change at closure.

#### Teaching plan

This record is evidence for the author, not teachable text. Promotion
re-authors each section from the reader's position: examples first, rules
after, no discovery history.

- **transfer-stances, opening:** the reader wants to know why `x := y` is not
  the whole story. They know ordinary assignment. What changes: five stances,
  introduced as how much of a value leaves its owner, with `loan` last as the
  stance of references. Likely mistake: thinking `as move` performs the move.
- **transfer-stances, when you must write a stance:** the reader got an error
  or wonders why `relay(message)` needs a stance and `sendCount(count)` does
  not. They know the stances. What changes: "no stance means copy, so anything
  that needs more is written", then the last-use rule, decided by the function
  and by the type's operations. Likely mistake: expecting the compiler to infer
  `move` at a final use.
- **transfer-stances, declared stances by location (Y):** the reader sees
  stances written in many places. They know the stances. What changes: one
  table per location, covering parameters by value and by reference, receivers,
  locals, aliases, and members, results, and type and identity lists, plus the
  body-defaults table. Likely mistake: reading a local's declared stance as a
  silent offer.
- **transfer-stances, `move` versus `last`:** the reader is choosing which to
  implement or write. What changes: full contract versus destruction-only; the
  refresh example; `last` without `move`. Likely mistake: implementing both
  when `last` would fall back to `move`.
- **transfer-stances, `loan`:** the reader wonders what an unmarked reference
  promises. What changes: an unmarked reference never copies or takes, and that
  stays true for anything derived from it. Likely mistake: assuming a `loan`
  is read-only, or that it stops member-by-member reconstruction.
- **function-invocation, transfer-aware parameters and preference:** the
  reader wants to know which overload runs and how the parameter is built.
  What changes: the parameter's stance is the least the caller must offer; the
  offer chooses the constructor; the `loan`-or-copy requirement. Likely
  mistake: expecting a `move` parameter to accept a plain call.
- **construction-and-destruction, the `transfer` list and lifecycle sources:**
  the reader is writing a type. What changes: declaring permitted stances;
  generation and `= default`; the source is never transferred as a whole.
  Likely mistake: listing `copy` and expecting it to exist when a member cannot
  supply it.
- **identity-types:** the reader wants a move-only version of a copyable type.
  What changes: the clause in the identity head governs construction,
  assignment, and replacement from its representation, including projection.
  Likely mistake: thinking `opaque` makes an identity move-only.
- **pointers-and-arenas:** the reader is adopting a raw allocation. What
  changes: `last` is required. Likely mistake: expecting `unique` to accept a
  borrow.
- **declarations-and-bindings and variants:** these depend on DQ1 and DQ2.
  They are planned in dry run 2.

Validation after promotion includes an independent cold-reader review by an
agent that has not read this record.

### Dry run 2 (2026-11-13): PASS

Read-only, except for this record. It revises dry run 1 for the two aligned
answers, Z and AA. It also checked iteration and `using` bindings against Z:
an untyped loop binding names an element path, a typed binding is a declared
reference, and `using` already states that explicit transfer syntax keeps its
ordinary meaning.

**Result: PASS.** Every aligned finding has one lasting owner. The structure
proposal stands: no new files, moves, or routers. Deferred and rejected
material has a destination. The change set is exact apart from the *verify*
checks, which confirm wording but do not change design. This result does not
authorize promotion.

#### Changes from dry run 1

Ownership map additions:

| Finding | Lasting owner | Other owners: local summary or link only |
| --- | --- | --- |
| Z, declared references | transfer-stances (declared stances by location) | declarations-and-bindings, iteration |
| Z, created bindings | transfer-stances | variants (case bindings), iteration (loop bindings) |
| AA | declarations-and-bindings (inferred declarations) | transfer-stances |

Change-set revisions:

- **declarations-and-bindings.md:** an unmarked reference declaration is a
  `loan`, replacing "alias offers `copy`" (Z). Inferred declarations take the
  type only (AA). Alias property overlays are requirements (M).
- **variants.md:** a case binding names its payload path. This replaces
  "preserves that selector's projected offered stance" (Z).
- **iteration.md** (added): typed reference bindings are declared references;
  untyped bindings name element paths (Z). *Verify* the replacement-cursor
  receiver wording against U.
- **transfer-stances.md:** additionally, the case-binding example under
  variants and unions; inferred declarations, near "Initialization is
  independent" (AA); the reference-alias example under projection (Z).
- *verify, likely unchanged:* using.md.

Teaching plan additions:

- **declarations-and-bindings, reference and inferred declarations:** the
  reader declares an alias or an inferred local. They know declarations. What
  changes: an unmarked alias only borrows, and `x := f()` takes the type, not a
  restriction. Likely mistake: expecting an alias to copy or take like the
  original.
- **variants, case bindings, and iteration, loop bindings:** the reader works
  with a payload or element in place. They know variant `switch` and `each`.
  What changes: the binding is the path, so `consume(owner as move)` is
  `consume(message.owner as move)`. Likely mistake: expecting the selector's
  declared stance to flow into the binding.

#### Size and readiness

About 22 owner files and 3 project files change. The work is mostly
re-authoring of transfer-stances, function-invocation, and
construction-and-destruction, plus local updates elsewhere. The maintainer may
still choose to split the promotion if that size is too large to review at
once. Splitting is not required for coherence.

### Dry run 3 (2026-11-13): example inventory, FAIL

Dry run 2 identified owners and wording but did not inventory examples. At
the maintainer's request, one subagent with fresh context made a read-only
inventory of `language/` against a summary of the aligned model. The agent
spot-checked two high-impact claims: construction-and-destruction's
`_.text = source` through an unmarked reference is invalid under U, and
function-invocation states that "`_` is pointer-shaped". Coverage: transfer
stances read in full; function-invocation, declarations-and-bindings,
construction-and-destruction, and composition by targeted section reads; the
rest by pattern search.

**Result: FAIL.** The inventory found questions the aligned model does not
answer (DQ3 to DQ9 below). Dry run 2's PASS is withdrawn until they are
aligned.

#### Inventory summary

About 95 affected examples and statements in 20 files. Categories: wording
only; needs a stance; meaning changes; becomes invalid; prose rule to rewrite;
open.

| File | Wording | Meaning changes | Invalid | Prose rule | Open |
| --- | --- | --- | --- | --- | --- |
| transfer-stances | 8 | 7 | 3 | 25 | 4 |
| function-invocation | 3 | 1 | 0 | 5 | 2 |
| declarations-and-bindings | 1 | 2 | 2 | 4 | 1 |
| construction-and-destruction | 1 | 1 | 1 | 1 | 1 |
| lambdas-and-callable-composition | 1 | 0 | 1 | 1 | 1 |
| lifetimes-and-references | 0 | 0 | 0 | 2 | 1 |
| variants, switch | 0 | 0 | 0 | 3 | 1 |
| composition | 0 | 0 | 0 | 1 | 2 |
| structural-shapes-and-compatibility | 0 | 0 | 0 | 2 | 0 |
| qualifiers | 1 | 0 | 0 | 1 | 1 |
| terms | 0 | 1 | 0 | 1 | 0 |
| identity-types | 1 | 0 | 0 | 0 | 0 (1 needs a stance) |
| variadics | 1 | 0 | 0 | 1 | 0 |
| pointers-and-arenas | 0 | 0 | 0 | 2 | 1 |
| operators, operator-catalog, operator-phrases, optional-values, nothing-instances, intent-acknowledgements, principles, using | 10 | 0 | 0 | 0 | 0 |

Representative items by pattern. Line numbers refer to the current files.

- **Declared stance as a silent offer:**
  - transfer-stances 598–620, `transfer(resource)` at 611, and
    `forward(resource) // remains a move offer` at 1204–1209;
  - lambdas 223–228, `consume(myValue)` from a capture declared `move`;
  - lifetimes 817; declarations 1466–1473; structural-shapes 700; qualifiers
    859; using 235.

  These become invalid or need rewording (M).
- **Receiver stance taken from the declaration:**
  - transfer-stances 860–868, `document : Document deep` followed by
    `document.publish()`, and 945–947 and 995;
  - terms 1268–1273; function-invocation 4033.

  Meaning changes: selecting a `deep` receiver needs `(document as deep)`.
- **Projection inheritance:**
  - transfer-stances 91, 706–737, `ship(package.label)` at 712, and 788–800;
  - declarations 1496–1502; variants 485 and 539; switch 1140; the case
    binding at transfer-stances 1349–1358.

  Prose to rewrite (M, Z).
- **Copy-in barrier:**
  - transfer-stances 430–440, 512–522, 638–640, and 668–676;
  - function-invocation 462–489 and 3486; construction-and-destruction 737
    and 940; composition 968; transfer-stances 816.

  Prose to rewrite, and meaning changes (B).
- **By-value bodies keeping their stance:**
  - transfer-stances 423 and 486–494;
  - function-invocation 470–475.

  `send(input.payload)` now offers `copy` (C2).
- **Unmarked references and receivers that copy or take:**
  - declarations 1505–1517, `consume(alias)` and `consume(alias as move)`;
  - transfer-stances 739–748; construction-and-destruction 1391–1393,
    `_.text = source`;
  - transfer-stances 960–1011, the unstanced receiver body `copy`;
  - identity-types 483; nothing-instances 487.

  Invalid, or prose to rewrite (U, Z).
- **Inference adopting a stance:** transfer-stances 677–680; declarations
  1521–1535 (AA).
- **Generation:** transfer-stances 1459–1467; construction-and-destruction
  979–986 (D, O).
- **Pointers:** pointers 330 and 1006–1008 (R).
- **The `forwardAll$` comment:** transfer-stances 1230; variadics 485 (S).
- **Read-only `readonly & copy` parameters to become unmarked:**
  transfer-stances 774, 1101, and 1196; function-invocation 440, 1298, and
  3567.
- **Stance lists missing `loan`, or out of order:** transfer-stances 57–66 and
  300–310; function-invocation 3474; qualifiers 31; identity-types 511;
  variadics 425; operators 935; operator-catalog 592; operator-phrases 259;
  optional-values 1179; intent-acknowledgements 494; principles 50; the
  `forward(...)` lists at transfer-stances 1175–1186.
- **Not enumerated:** many helper calls through unmarked references, such as
  `use(x.y)`, `print(_.label)`, and `inspect(view)`. They are valid when the
  helper takes a `loan`. Promotion must give each helper an explicit signature
  or keep it obviously loan-taking.

#### Design questions requiring alignment

- **DQ3. Compiler-created temporaries.** Current design gives an unnamed
  temporary an inherent `last` offer:

  ```zax
  fromTemporary := (: Source).get()
  ```

  Does that survive "never infer `move` or `last`"? *Agent recommendation:* yes.
  It is a structural property with no observable source left behind, not an
  inference about a named place.
- **DQ4. The result slot inside its producer.** Examples include `result :
  Message move` followed by `inspect(result)`. Is the declared stance a
  requirement inside the body, or the outward offer only? *Agent
  recommendation:* outward offer only. Inside the body, `result` starts from
  `copy` like a parameter, and a transfer from it is written.
- **DQ5. An unmarked receiver beside an exact `copy` receiver.** *Agent
  recommendation:* current receiver selection already decides it: an exact
  receiver beats the unmarked fallback, so an unstated call selects the exact
  `copy` receiver. No new rule is needed.
- **DQ6. Composition wrappers with by-value `deep`.** The case is `refill final
  : ()(source : Tank) = via fuel.refill`. *Agent recommendation:* the
  mechanism writes its own offers, so its current table stands. Body defaults
  do not constrain generated forwarding.
- **DQ7. A copy capture of a `loan` reference.** The case is `view : Document
  readonly & = source` followed by `[[ view ]]`. *Agent recommendation:* a copy
  capture is a transfer from a `loan`, so it is an error. A reference capture
  stays `loan`. The lambdas owner's capture syntax decides the fix spelling.
- **DQ8. The generated `=` result and chained assignment.** Generated `=`
  returns `_` as a `readonly & copy` reference so that `first = second =
  third` works. With an unmarked receiver as `loan`, that result would be
  derived from a `loan` and could not feed the next assignment's copy. Giving
  `=` a `copy` receiver would instead make every assignment *to* a
  move-restricted local bind it as a copy receiver. *Undecided; needs
  discussion.*
- **DQ9. `_` is pointer-shaped.** Examples include `negate(_)` and `apply(_,
  profile)`. V refuses raw pointers formed from a `loan`. Does `_` from a
  `loan` receiver count? *Agent recommendation:* `_` is the receiver path, not
  a formed pointer. Passing `_` on carries the receiver's `loan`, as V does for
  references. V's raw-pointer rule targets explicitly formed pointers. Related:
  `OpaqueReferenceObserver` built from a `loan` reference (pointers
  1236–1237) follows V.

DQ8 is the only question without a recommendation. It touches the generated
assignment shape that construction-and-destruction owns.

### Dry run 4 (2026-11-13): PASS

Read-only, except for this record. It revises dry run 3 for finding AB, which
answers DQ3 to DQ9. No inventory item remains undecided by the aligned model.
The remaining helper calls through unmarked references are a promotion
obligation, not a design question: each helper either receives a signature or
is plainly `loan`-taking.

**Result: PASS.** Every aligned finding, including Z, AA, and AB, has one
lasting owner. The structure proposal stands. The exact change set is dry run
2's, extended by the example-level inventory in dry run 3 and by the rows below.
This result does not authorize promotion.

#### Ownership map additions

| Finding | Lasting owner | Other owners: local summary or link only |
| --- | --- | --- |
| AB, temporaries and results inside producers | transfer-stances | function-invocation (result slots, temporary receivers), qualifiers, switch |
| AB, receivers | transfer-stances | nothing-instances |
| AB, generated forwarding | composition | transfer-stances (wrapper table) |
| AB, captures of a `loan` | lambdas-and-callable-composition | lifetimes-and-references |
| AB, `_` as the receiver path | transfer-stances | operators, composition, function-invocation |
| AB, `self` results and the generated `=` | function-invocation (`self` results); construction-and-destruction (generated `=` shape) | lifetimes-and-references (receiver-origin `self`) |

#### Change-set additions

These files were not in dry run 2's list: **switch.md**, **composition.md**
(beyond the wrapper wording), **lifetimes-and-references.md**,
**operator-phrases.md**, **using.md**, and **nothing-instances.md**.
Wording-level files from the dry run 3 inventory: optional-values,
intent-acknowledgements, principles, and operators.

Each file's line-level items are those listed in dry run 3's inventory, plus:

- **construction-and-destruction.md:** the generated `=` result becomes
  `self`; `_.text = source` declares `& copy` or reads differently.
- **function-invocation.md:** `self` results carry the receiver path's
  authority; a result's declared stance is its outward offer only; temporary
  receivers are unchanged.
- **lifetimes-and-references.md:** receiver-origin `self` gains the path
  meaning; captures follow DQ7.
- **lambdas-and-callable-composition.md:** a copy capture of a `loan` is an
  error; capture stances are requirements.

#### Teaching plan additions

- **function-invocation and lifetimes, `self` results:** the reader wants
  chained calls or assignment. They know reference results. What changes: a
  `self` result is the place you started from, with exactly its permissions.
  Likely mistake: expecting `self` to grant copy permission.
- **construction-and-destruction, generated `=`:** the reader asks what `a = b`
  returns. What changes: it returns `a` itself, so `first = second = third` works
  wherever `second` may be copied. Likely mistake: chaining through a
  move-restricted place.

#### Readiness

The promotion touches about 26 language files and 3 project files. The heavy
re-authoring is in transfer-stances, function-invocation, and
construction-and-destruction. Splitting is still optional, not needed for
coherence.

### Dry run 5 (2026-11-14): PASS

Read-only, except for this record. The main promotion from dry runs 1 to 4 is
already applied and awaiting commit. This dry run covers only the follow-up
change set for findings AC, AD, AE, and AF. It checked every live owner that
mentions `self` results, passing `_` on, the "when a stance is needed"
condition, and members taken through a `move` reference.

**Result: PASS.** Each finding has one owner that teaches it fully, and the
other owners need only a short local statement or a link. No design question
blocks promotion.

*Revised 2026-11-14.* The first version of this record raised members taken
through a `move` source as an item to confirm. Discussion settled it: AF
item 2 limits only the whole value, and members follow their own stance. The
item and its change-set entries are removed. The current sentences about
lifecycle sources need no change.

This result does not authorize promotion.

#### What the dry run found

1. **No live example passes `_` on with `move` or `last`.** The only `_` with
   a strong stance is `(_ as last).finishPublishing()` in
   `transfer-stances.md`, which is a call on `_`, not a hand-off. No existing
   example therefore needs the new acknowledgement. The promotion should
   still add a comment to that example, because it is the natural place to
   show the difference between calling a function on `_` and passing `_` on.
2. **No live example uses a pointer `self` result,** so making `self`
   reference-only breaks nothing. The wording "a reference or pointer result
   declared `self`" appears in `function-invocation.md`,
   `lifetimes-and-references.md`, `composition.md`, and `terms.md`.
3. **`nothing-instances.md` says `_` is pointer-shaped,** and that "either
   form may be passed or returned, including a verified `self` result". The
   pointer form and the dereferenced form of `_` both still exist. Only the
   `self` part changes: a `self` result returns the dereferenced, reference
   form. One sentence changes.
4. **The `=` examples missing `#`** are exactly the ones AF lists. No other
   example drops a `self` result.

#### Ownership map

| Finding | Owner that teaches it fully | Other owners: short statement or link |
| --- | --- | --- |
| AC, AD items 1, 4, and 6: a `self` result passes on the receiver as written, chains count as uses of the variable, later stances replace earlier ones | transfer-stances (`self` results) | function-invocation, lifetimes-and-references, terms |
| AC and AD item 5, AF item 3: declaration form, reference only, required unless `#` | function-invocation (`self` results) | lifetimes-and-references, composition, terms, nothing-instances |
| AD item 2: taking members of `_` is ordinary | transfer-stances (`_` is the receiver path) | none |
| AD item 3: a discarded chain needs no diagnostic | transfer-stances (`self` results) | none |
| AE: the reserved-stance condition and discarded results | transfer-stances (when a stance must be written) | function-invocation, intent-acknowledgements, variadics |
| AF item 1: passing `_` on needs `receiver-transfer` | transfer-stances (`_` is the receiver path) | intent-acknowledgements (category list and a short section) |
| AF item 2: a reference passes the whole value on with at most its own stance; members follow their own stance | transfer-stances (stances inside a body) | the stance habit at the top of transfer-stances |
| AF item 3, examples: `#` on `=` | construction-and-destruction | partials |

#### Exact change set

Staged state is shown per file. Edits to a staged file appear as new unstaged
changes on top of it, so the maintainer's staged comparison point stays
intact.

- **transfer-stances.md** (unstaged):
  - opening habits: one sentence saying that through a reference you pass on
    at most what you were given;
  - "What decides whether a stance matters": count stances the `transfer`
    list includes as written (AE);
  - "Stances inside a body": a short subsection for AF item 2, with the
    `drain` example, the note that members follow their own stance, and the
    note that the caller cannot see a member left finished;
  - "`_` is the receiver path": AF item 1 with the `Task` example, AD item 2,
    and a comment on the `finishPublishing` example;
  - "`self` results carry the caller's path": rewritten for AC and AD items 1,
    3, 4, and 6, with the builder, `takeOwnership`, and `relay(message.build())`
    examples;
  - "Last use when mapping a result": the "no error when" sentence (AE);
  - diagnostics: passing `_` on without `receiver-transfer`; passing a
    reference on with a stronger stance; the reserved-stance wording.
- **function-invocation.md** (staged):
  - "Receiver-origin `self` results": reference only, the declaration form
    with optional `#`, required by default, and a short summary of the
    stance behavior with a link to transfer-stances;
  - "Result mapping and terminal opportunity": rewrite the condition bullets
    and remove the discarded-result clause (AE);
  - the two-rule summary under "Terminal opportunity requires intent" (AE).
- **lifetimes-and-references.md** (staged): "Receiver-origin `self`":
  reference only, and the stance written on the call carries through.
- **construction-and-destruction.md** (staged): `#` on the four `=`
  declarations; the generated `=` passage gains
  `first = (second = third) as move` as the preferred way to pass a stance
  through an assignment.
- **composition.md** (staged): "The `self` result contract": reference only.
- **intent-acknowledgements.md** (staged): a `receiver-transfer` row and short
  section; the `implicit-stance-at-terminal-use` row wording (AE).
- **terms.md** (unstaged): the `self` result entry.
- **partials.md** (unstaged): `#` on both `MyCatalog` `=` bodies.
- **nothing-instances.md** (unstaged): the `self` sentence (found item 3).
- **variadics.md** (unstaged): "has a `move` or `last` operation" becomes
  "has or lists" (AE).
- **project/work/038**: section 10 is replaced at closure; no other project
  file changes.

No raw input changes. `project/raw/safety.md` keeps its pending style review
from section 10, step 5, which is separate from this change set.

#### Teaching plan

- **transfer-stances, `self` results.** The reader wants to chain calls, as in
  a builder, and wonders what happens to the stance they wrote at the start.
  They already know receiver stances. What changes: the stance and qualifiers
  they wrote travel along the whole chain and reach the call at the end;
  the calls in between only pass the value along. Likely mistakes: thinking
  a call in the middle uses up `as last`, or that a chain hides a last use.
- **transfer-stances, passing `_` on.** The reader is writing a `last` or
  `move` version that gives the value to someone else. They know that `_` is
  the receiver. What changes: they write `intent<receiver-transfer>` around
  the hand-off, because callers of `(task as last).submit()` cannot see it.
  Likely mistake: expecting a call such as `(_ as last).finish()` to need it.
- **transfer-stances, references pass on at most their stance.** The reader
  has a `& move` parameter and wants to finish the value. They know the
  "`as last` when done" habit. What changes: that habit is for variables
  holding their own value; through a reference you pass on what you were
  given, at most, for the whole value; members follow their own stance.
  Likely mistakes: writing `as last` on a whole `& move` reference, or
  assuming `as move` protects the contents of the caller's members.
- **function-invocation, `self` result declarations.** The reader is
  declaring a chaining function. What changes: write only the type, `&`, and
  optionally `#`; without `#`, callers must keep chaining. Likely mistakes:
  writing `readonly` or a stance on the result, or using `*`.
- **construction-and-destruction, `=`.** The reader writes their own `=`.
  What changes: the result is declared `result self # : MyValue &`, so
  `a = b` stays a complete statement.
- **intent-acknowledgements, `receiver-transfer`.** The reader met the
  diagnostic. What changes: they learn what was flagged and that the fix is to
  acknowledge a deliberate hand-off or not hand `_` off.

#### Readiness

About 10 language files change, mostly in a few sections each. The main
writing is in transfer-stances (`self` results and `_`) and function-invocation
(`self` results). The change set can be applied in one pass.

### Dry run 6 (2026-11-14): teachability of the unstaged promotion, FAIL

Read-only, except for this record. The maintainer asked for every unstaged
edit from the earlier promotion to be checked against the teachability test
before anything more is promoted. The agent read every unstaged diff itself,
and had read `transfer-stances.md` in full earlier in the same session. No
subagent was used. Staged files were outside this pass, except where dry run
5 already changes them.

The question for each passage: can a competent programmer who never saw this
work record understand what the text is telling them, see it in code, and
know what to do?

**Result: FAIL.** Ownership is still sound, as dry run 5 found. But several
passages make the reader reconstruct the meaning from abstract or
specification-like wording, and one passage contradicts another owner. All of
them are in sections this promotion changed, so the teaching-debt rules make
them promotion blockers rather than log entries. They should be rewritten as
part of the promotion change set.

#### Passes

These unstaged edits read as teaching and need no change beyond dry run 5:
`operators.md`, `operator-phrases.md`, `principles.md` (the enforced-order
section is a good model), `partials.md` (apart from the `#` from AF),
`structural-shapes-and-compatibility.md`, `type-definitions.md`, `using.md`,
and `variants.md`. In `transfer-stances.md`, the opening, the five stances,
copy, deep, move, last, the refresh example, loan, "When a stance must be
written", declared stances by location, stances inside a body, by value and
by reference, members and bindings, captures, optional values, variants,
generated operations, and costs all teach well.

`project/documentation.md` is project guidance, not programmer teaching, and
reads clearly.

#### Must fix: a contradiction between owners

1. **The same declaration means different things in two owners.** Both
   `qualifiers.md` (the opening) and `transfer-stances.md` ("Qualifier
   promises remain exact") use:

   ```zax
   source : Buffer mutable readonly final & move
   ```

   `transfer-stances.md` reads it as a parameter: "permission to move from the
   `Buffer` it refers to". `qualifiers.md` reads it as a local reference: "a
   `move` stance which every transfer from `source` must state". Both
   meanings exist, but the reader cannot tell which one applies, because the
   example has no context. Fix: show it as a parameter in both owners, for
   example `consume final : ()(source : Buffer mutable readonly final & move)`,
   and describe it the same way in both.

#### Must fix: passages that do not teach

Each item names the reader's question, what blocks them, and the direction of
the fix.

**transfer-stances.md**

2. **"Results" and "Results and forwarding", the second declaration.** The
   reader asks: can I give the same function a different result stance? The
   text says the function can be declared again "if calls can be adapted to
   it", and that callers of that declaration "receive the result with its
   stance". There is no code. The raw-pointer example that follows is only
   described in prose, using `last = @`, which this page never teaches. Fix:
   show both declarations and two callers in code, with comments saying what
   each caller receives, or move the topic to function invocation and leave a
   one-sentence link.
3. **"Results and forwarding", exceptional results.** The example
   `value := operation() catch failure: localFailure: { … }` is not explained,
   and the closing sentences are specification-like: "The compiler may build
   the failure value directly in its final destination. The caller becomes
   responsible for destroying the failure value only after it has been built
   there." Fix: say in one plain sentence that a failure value is built like
   any other result, show one move-only failure that needs a `move` failure
   result, and leave construction details to the exceptional-results owner.
4. **"Last use when mapping a result", the closing paragraph.** "The exception
   is a raw pointer result declared `last = @`, because that result is
   responsible for freeing its allocation." The reader has not been taught
   `= @` on a result. Fix: a two-line example, or a plain description with a
   link to pointers and arenas.
5. **"Compatible whole values and anchored regions", the guarantee.** "A
   function that returns a value with `last` can also name one anchor for
   which it gives that guarantee." The reader cannot tell how a type or a
   function gives that guarantee, or what the caller writes. There is no
   code. Fix: show the guarantee in source, or say plainly that the spelling
   belongs to structural shapes and link there.
6. **"Receiver stance", the four rules.** The second rule contains a long
   comparison with parameters in the middle of the list, so the reader loses
   the rule while reading the exception. Rules three and four also restate
   each other. Fix: state the selection rules first, each with its example,
   and then explain separately why receivers differ from parameters on the
   loan-or-copy question.
7. **"Type aliases".** `DeepView` and `CopyView` are shown without saying why
   anyone would write them. Fix: one sentence on the use case, such as a team
   alias that marks every view as needing `deep`, and what declaring a
   variable with it then requires.
8. **"A stance on several items", the `from each` paragraph.** The paragraph
   introduces new pack syntax and two stance positions in one dense
   paragraph before any code. Fix: show the code first, then explain the two
   places a stance can go.
9. **"Arrays and slices".** "An array whose storage comes from a provider"
   and "when the destination accepts it" are not explained. Fix: plain words
   for the provider, or a link where storage providers are taught.
10. **The `Owns` metadata** says "result offers", which is discussion
    vocabulary. Fix: "result stances".

Minor wording in `transfer-stances.md`, to fix in the same pass: "Fallback"
opens with "something that takes less can accept it instead", where an
abstraction acts. The writing rules prefer naming who does what, for example
"the compiler uses a function that takes less instead". The heading "Qualifier
promises remain exact" is also unclear; "Stances do not change qualifiers"
says it directly.

**qualifiers.md**

11. **Captures paragraph.** "Captures preserve or strengthen the captured
    source qualifications and never take on pliability implicitly."
    "Pliability" is not a term a reader knows. Fix: say which qualifiers a
    capture keeps and which it cannot gain.
12. **The readonly paragraph** near the end: "Unusual readonly contracts
    remain possible only when their actual behavior respects those
    qualifications." Fix: say what a programmer may do, with the existing
    example from transfer-stances of a `readonly` `move` source whose
    resource lives outside the value.

**optional-values.md**

13. **The table of what each form leaves in the optional.** The new `loan` and
    `deep` rows are plain, but the older rows beside them say "accepted
    whole-optional `move` contract determines source state" and "mark terminal
    when accepted; after complete consumer, leave wrapper absent and payload
    dispositioned". The sentence before the table says "Generic stance
    restatement has no state effect until a consumer accepts it". The table
    was changed by this promotion, so every row should read the same way.
    Fix: rewrite each row as what the original optional holds afterward.

**operator-catalog.md**

14. **`value as copy` is described as "copy it",** but the next sentence says
    the form performs no transfer. Fix: "ask for a copy", and match the other
    rows.

**pointers-and-arenas.md**

15. "Borrowing needs no stance, because no stance would change what happens"
    does not say why. Fix: "Borrowing an address copies the pointer and
    leaves ownership where it is, so it needs no stance".
16. "An observer made from a loan reference, such as `typed` above, which has
    no stance, still carries the loan": the reader cannot tell what "which
    has no stance" refers to. Fix: split into two sentences that name the
    observer and the reference.
17. The example line using `ordinaryRaw` relies on a name the example never
    declares. Fix: declare it, or describe it in the comment.

**switch.md**

18. "but a receiver stance alone does not transfer the selector as a
    right-hand argument" keeps the older wording in a sentence that was
    otherwise rewritten. Fix: "writing a stance on the selector does not by
    itself pass the selector to anything".

**nothing-instances.md**

19. "so it is still what a call on the type uses": the reader cannot tell what
    "still" contrasts with. Fix: "so a call on the type uses `inspect`".

**terms.md**

20. **The `loan` entry:** "Whether the function may change the value is
    decided separately, by its qualifiers" does not say whose qualifiers.
    Fix: "by the reference's qualifiers, `readonly` or `writable`".

**project/raw/safety.md** (style review from section 10, step 5)

21. "leaving the old value only destruction-valid" is the one phrase in the
    new rejected-proposal section that does not read plainly. Fix: "leaving
    the old value fit only to be destroyed or replaced". The rest of the
    section passes.

#### Addendum: staged pages

The maintainer asked for the staged pages to be checked the same way. The
agent read every staged diff. `arrays-and-slices.md`, `generics.md`,
`iteration.md`, and `declarations-and-bindings.md` pass. Fixes needed:

22. **function-invocation.md, "Transfer-aware value and reference
    parameters".** "then plain or `copy` one" and "If a call declares a stance
    less than every overload requires" are ungrammatical. "taking from it must
    explicitly declare a stance" uses "declare" for something written at a
    use.
23. **function-invocation.md and nothing-instances.md, `_` as a `self`
    result.** "Either form may be passed or returned, including through a
    verified `self` result" needs the reference-only rule (AD item 5).
24. **construction-and-destruction.md, "Declaring which transfers a type
    allows".** "An `= forbidden` clause can remove a `copy` function from
    auto-generating thus disallowing its usage" is ungrammatical and repeats
    the earlier `= forbidden` paragraph.
25. **construction-and-destruction.md, "Generated copy construction and
    assignment".** The opening sentences are specification-like: "one
    compiler-owned schema exposed as exact, qualifier-complete declarations",
    "not a qualifier-erased prototype", "conceptually supplies exact
    source-place variants", "compiler-anonymous", "no destination lifetime
    exists yet".
26. **composition.md, explicit wrappers.** "Each offer then follows ordinary
    fallback, and the remaining terminal outer value is destroyed" uses
    discussion vocabulary, and the paragraph before it repeats the reason
    already given above the table.
27. **intent-acknowledgements.md, implicit stance at a last use.** "when an
    owned by-value result reaches its structurally final consumer and its
    producer left stance implicit" is specification-like.
28. **lambdas-and-callable-composition.md.** Typo: "declare what is wants".
29. **identity-types.md.** The reordered list of ways to build the stored
    value still says "consuming/`last` transfer".
30. **structural-shapes-and-compatibility.md** (unstaged, found while
    checking item 5): "This explicit request is viable only when the
    applicable source contract proves that the complete source remains
    destruction-valid" is specification-like.

#### Not decided by this dry run

Whether to fix these passages inside this promotion or move some to the
teaching-debt log. *Decided 2026-11-14:* the maintainer asked for all of them
to be fixed in this promotion, deciding only where a fix is not obvious. Under `project/documentation.md`, an observation in a
section this promotion materially changed remains a promotion blocker, so the
agent recommends fixing all of them in the same change set as dry run 5.
Items 2 to 5 may also move detail to its owning document instead of
expanding `transfer-stances.md`.

#### What would make the next dry run pass

The promotion change set becomes dry run 5's list plus items 1 to 21 above.
After that change set is applied, a reader who did not take part in this
discussion should read each changed section, as `project/documentation.md`
requires after a substantial promotion. The maintainer has asked that the
agent not use a subagent for writing. That cold-reader check could still be
done by a fresh reader if the maintainer wants it; otherwise the agent reads
every changed section itself.

### Follow-up promotion applied (2026-11-14)

Authorized by the maintainer: the dry run 5 change set, items 1 to 21 of dry
run 6, and the staged-page items 22 to 30, deciding only where a fix was not
obvious. None needed a decision. The agent wrote every change itself and
reread every changed section; no subagent was used. Not committed. Every
edit to a staged file is an unstaged change on top of it, so the staged
comparison point is unchanged.

What changed, by file:

- **transfer-stances.md:** a new "References pass on at most their stance"
  section (AF item 2, including members and the weaker-stance example); a new
  "Passing the receiver on" section (AF item 1); "`self` results carry the
  caller's variable" rewritten with the builder chain, last use through a
  chain, declared stances, and replacing a stance (AC, AD); receiver
  selection rewritten as three rules followed by the comparison with
  parameters; the reserved-stance wording in "What decides whether a stance
  matters", "Last use when mapping a result", and diagnostics (AE); type
  aliases with a use case; the `from each` example shown before its
  explanation; the second result declaration and failure results shown in
  code; the anchor guarantee shown in code; storage providers explained;
  "Stances do not change qualifiers", with the `drainBuffer` parameter; the
  fallback opener and metadata wording.
- **function-invocation.md:** "Receiver-origin `self` results" rewritten:
  declaration form, reference only, required unless `#`, the result as the
  caller's variable, and the `peek` example; the result-mapping conditions
  and the discarded-result sentence (AE); the two-rule summary and `return`
  sentence (AE); a note that a `self` chain does not hide a last use; the
  `_` paragraph; grammar in the by-value rules; a diagnostics entry.
- **lifetimes-and-references.md, composition.md, terms.md,
  nothing-instances.md:** reference-only `self` results and the caller's
  stance carrying through; composition's wrapper cost paragraph.
- **construction-and-destruction.md and partials.md:** `#` on every `=`
  declaration; the generated-copy section in plain words; the generated `=`
  passage, including `first = (restricted as move) = third`; the
  `= forbidden` sentence; the identity admission list.
- **intent-acknowledgements.md:** the `receiver-transfer` category row and
  section; the `implicit-stance-at-terminal-use` row; the by-value result
  sentence; the terminal-source reuse explanation.
- **qualifiers.md, optional-values.md, operator-catalog.md,
  pointers-and-arenas.md, switch.md, variadics.md, identity-types.md,
  lambdas-and-callable-composition.md, structural-shapes-and-compatibility.md,
  project/raw/safety.md:** the one-passage fixes listed in dry run 6.

Validation: all local links and heading anchors in `language/`,
`project/raw/`, `README.md`, and `index.md` resolve. No live text describes a
pointer `self` result or a `self` result written with qualifiers.

Remaining before closure:

- An independent cold read of the changed sections, if the maintainer wants
  one, as `project/documentation.md` asks after a substantial promotion.
- Section 10, step 5: record the older spec-like wording in the pointer and
  qualifier owners, outside the sections changed here, in
  `project/teaching-debt-log.md`.
- Closure: mark this file historical, archive it, propose work item 039, and
  create it, each after the maintainer's go.

### Cold reads after the follow-up promotion (2026-11-14)

The maintainer asked for fresh readers. Three general-purpose subagents, which
had not seen this record, read every changed section, split by file. Their
findings are candidates; the agent checked each against the page and the
aligned findings, fixed real problems, and dismissed the rest.

#### Reader 1: `transfer-stances.md` (46 items)

Fixed:

- **Contradictions:**
  - `as last` replaced by a later stance in a chain: the text now says the
    stance written last counts, as AD item 4 aligned.
  - Temporaries treated as `last` while function results passed straight on
    are not: the page now says why. A result has a declaration whose stance
    the caller can read; a value built in place has none.
  - A member's declared stance: "always use it" became "a requirement", and
    `deep`'s treatment of a `copy` member is described as what the generated
    `deep` does.
  - "nothing restricts how the function uses the parameter" now matches AF
    item 2.
  - The readonly-move explanation in "Choosing value or reference" now says
    the move constructor needs a `mutable writable` source.
  - The `peekValue` comment now says the caller borrows unless it writes
    `as last`.
  - Union `last` now says the variable still counts as finished.
  - The `last source` bullets now give the timing: finished during the call,
    empty afterward.
  - The Move section now says its guarantee is about the type's own `move`,
    separate from what a function with writable access may do to members.
  - Source stability now lists temporaries and loans.
- **Forms used before they were taught:** the "Reading the examples" table
  gained `+++ replacement`, `(: Source)` and `[{ … }]`, generic `$` functions,
  `result #`, `once`, and receiver qualifiers. Short glosses were added for
  the Nothing instance, compatible shape and anchor, `compatible layout`,
  `preferred`, `unique shareable`, `bound strong`, `as readonly`, `failure.`,
  `result.+++`, and identity types. Acknowledgement names now say that
  "terminal" means finished.
- **Examples:**
  - the first `Message` example now explains that `as last` falls back to
    `move`;
  - the "One stance" example shows what it claims;
  - the capture example puts the error on the copy capture and shows a
    reference capture;
  - the fallback example's type was renamed `Ledger`, so it no longer shares
    a name with the later `Archive`;
  - `Message` gained `payload`, `Archive` gained `history`, and
    `prepareTemplate` uses `Archive`;
  - `nextQuery as last` in both `DatabaseCommand` replacement constructors;
  - a by-value `deep` parameter needs `as deep`, matching function
    invocation's table;
  - binding a reference moves nothing;
  - `encode(message as last)` passes each item once, matching variadics.
- **Wording:** the car metaphor was removed; the move-resource bullet, costs,
  diagnostics, and two boundary items were reworded.

Dismissed:

- The "Source / Provenance" metadata row. It is a sanctioned metadata field.
- Item 24's question about an alias's stance used on a parameter. The alias
  section already says that declarations using the alias get the stance as if
  they had written it, so its meaning follows the location table.
- Item 37, wording that is precise and short.
- Item 2's request to move fallback earlier. A forward link was added
  instead.

#### Readers 2 and 3: the other changed pages (36 and 66 items)

Fixed, in the agent's reading of the aligned findings:

- **Contradictions:**
  - an initializer with no stance "copies": a function result uses its
    declared stance;
  - a variable with no stance "counts as declared `copy`" (which would have
    ruled out `as deep`): it has no requirement;
  - the glossary's single exception for where a stance comes from: function
    results and temporaries are now listed;
  - a caller's stance on a call beats the declared result stance, as
    `peekValue() as last` already showed;
  - a bare `&` result is a `loan`, under finding U, except for `self`
    results;
  - a same-type `& loan` source is an error only when it is the constructor's
    only input;
  - `= default` takes members with its own stance; the body-default table is
    about hand-written bodies;
  - an `=` written with `as move` still copies unless the type declares a
    move assignment;
  - reference captures can never allow more than the captured variable;
  - "use after an earlier `move`" in diagnostics became use after `as last`.
- **Wrong examples:**
  - the `as deep` repair on a type without `deep`;
  - `MyType`'s constructors in the identity example;
  - the `transfer move` type with no `move` written, in type definitions;
  - `snapshot` copying a loan receiver in composition;
  - "copied self results" and "`self` … or pointer" in composition;
  - the composition two-copies claim;
  - the inverted `if _.connection` comment;
  - `keepCopy(result as copy)`, which showed nothing being taken.
- **Wording:**
  - "takes nothing" for `as loan` in operators, the operator catalog, and
    optional values;
  - "Stances" as the header of the qualifier table;
  - "its final stance" for a qualifier;
  - "consumer accepts", "disposition", "authority", "terminal opportunity",
    "boxed state", "capture path", and the legacy `lease` sentence in changed
    sections;
  - the two headings in function invocation, renamed "Result mapping and
    last use" and "A last use needs a stated stance", with links updated;
  - examples that used undeclared names (`queue`, `a`, `b`, `forwardEach$`,
    `consume`, `makePayload`, `install`, `selectedText`) now declare them
    or say what they take;
  - the anchor example now shows a caller;
  - the principles text on undefined orders no longer gives process advice.
- **Older text** next to changed sections went to the teaching-debt log.

Dismissed:

- `{ … }` as an elided body. The documentation rules allow it.
- "processing" versus "instantiation" in variadics. Variadics uses
  "processing", and the changed sentence now does too.
- `relay(message.prepare())` with `prepare` not declared on the page. The
  sentence says `prepare` returns `self`.

Raised with the maintainer, now decided by AG:

- **A temporary `switch` selector treated as `as last`** when several case
  tests call functions on it. A `last` version in an early test could finish
  the selector before later tests run.
- **Passing `_` to a raw pointer parameter from a loan receiver.**
  `nothing-instances.md` shows it; finding V forbids making a raw pointer from
  a loan; AB's DQ9 says `_` is not a pointer the programmer formed. The
  combination lets a loan reach a raw pointer.
- **Whether `item := makeItem()`**, with no result stance on `makeItem` and a
  movable `Item`, is a last use of the result that needs a stance. The owners
  describe the check only for a result passed straight on to a call. If
  initializing a variable counts too, every such initializer needs a stance
  or a result stance.

### AG applied, and status for maintainer review (2026-11-14)

The maintainer authorized recording AG, capturing pointer stances as raw
input for work item 039, and promoting AG, then reviewing the whole result.

Applied:

- **Switch tests lend the selector:** `switch.md`.
- **Results with no declared stance pass on as `last`:**
  - `transfer-stances.md`: the temporaries section became "Unnamed values are
    already treated as `last`"; "Last use when mapping a result" became
    "Results with no declared stance"; the scope of the intent rule, the
    location table, "Written `copy` versus no stance", and source stability
    were updated;
  - `function-invocation.md`: "Result mapping and last use" became "Results
    with no declared stance", and the raw-pointer result passage was updated;
  - `declarations-and-bindings.md`: initializers and the result summary;
  - `intent-acknowledgements.md`: the result and raw-pointer-result uses of
    `implicit-stance-at-terminal-use` were removed;
  - `pointers-and-arenas.md`: raw pointer results;
  - `terms.md` and `principles.md`.
- **Loans and raw pointers:** `nothing-instances.md`'s mapping example now
  declares a `copy` receiver.
- **Capture:** [raw pointer stances](../raw/pointer-stances.md), indexed in
  `project/raw/README.md`.

Validation: all local links and anchors in `language/`, the new raw input,
the raw index, and the teaching-debt log resolve. No text describing the
removed result check remains.

**Failure results, decided and applied.** Failure results follow AG item 2.
Updated: `except.md` ("Transfer, ownership, and references" and the owning
payload example), `transfer-stances.md` (the failure-result paragraph and
source stability), and `function-invocation.md` (results with no declared
stance).

**What remains in 038:**

1. The maintainer's review of the complete promotion, staged and unstaged.
2. Closure, each step after the maintainer's go: mark this file historical,
   archive it, create work item 039, and update the current-work index in
   `project/README.md`.

**Proposed initiating input for 039, for discussion:**

- **Title:** `039-pointer-stances`.
- **Initiating concern:** how a pointer type states what may happen to the
  value it points to, so that a loan can be passed where a pointer is
  expected without `unsafe`, through any nesting of `*` and `&`.
- **Motivating pressure:** the receiver `_` is pointer-shaped and carries a
  loan; data-oriented and interop code lends through pointers; the current
  rule forbids any pointer made from a loan outside `unsafe`.
- **Starting input:** [raw pointer stances](../raw/pointer-stances.md),
  including the candidate direction.
- **Required reading:** `project/documentation.md`, the raw input,
  `transfer-stances.md` (loan, references pass on at most their stance,
  receiver stance), `qualifiers.md` (attachment through indirection), and
  `pointers-and-arenas.md` (ownership roles, raw pointers).

## Closure (2026-11-16)

Closed by the language maintainer after the final review. Every finding, A to
AI, is promoted into its current owner. Deferred material lives in
[raw pointer stances](../../raw/pointer-stances.md), which work item 039
consumes, and in the indexed raw inputs and teaching-debt log entries named
above. No provisional teaching observation remains only in this record.
