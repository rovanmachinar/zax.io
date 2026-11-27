# Zax safety and analysis

| Field | Value |
| --- | --- |
| Status | Current conceptual design |
| Audience | Human developers reasoning about Zax's safe subset, compiler proof, unsafe responsibility, and source portability |
| Applies To | Cross-cutting programmer-visible safety and static-analysis behavior; not a formal proof system or diagnostic specification |
| Implementation State | Not established by this repository |
| Owns | The safe-subset model; required proof; `safe<...>` claims and `unsafe` operations and the test that chooses between them; the current category list; which checks count as proof; known-invalid boundaries; distinction from intent and lints; asking a compiler to verify a claim with `[<verify=...>]`; how loudly verification findings are reported; language-contract versus compiler-analysis evolution; panic-check control attachment, ranges, import boundary, and build starting state; cross-cutting safety diagnostics and source stability |
| Does Not Own | Domain-specific validity and admission rules, including [enum admission](enums.md#admission-and-reachable-values), which define each category's fact; the provisional category registry; intent acknowledgements ([intent acknowledgements](intent-acknowledgements.md)); directive grammar and placement ([compiler directives](compiler-directives.md)); or compiler implementation algorithms |
| Source / Provenance | Current intent, construction, invocation, optional, lifetime, and pointer safety boundaries |
| Supersedes | Accepted general safety direction formerly preserved only as project input |

## Safety is a selected contract

Zax does not organize the entire language around maximizing compiler-enforced
safety. It does define a safe subset with explicit guarantees.

Code using only safe operations and interfaces receives the guarantees assigned
to the selected language contract. Operations outside those guarantees must be
recognizable and auditable.

The central rule is:

> For every property mandatory under the selected language contract, Zax
> attempts to prove the operation valid. If proof cannot be established, the
> operation is rejected unless the programmer writes a narrow `safe<...>` claim
> supplying the missing fact, or uses an `unsafe` operation whose lower-level
> meaning the programmer takes responsibility for. Neither can make a known
> impossibility valid.

The contract decides what must be proved. One executable compiler's current
cleverness does not redefine portable source validity.

## Four analysis outcomes

### Proven safe

The compiler establishes every required fact:

```zax
value : MyValue
view : MyValue readonly & = value

inspect(view)
```

No marker is needed.

### Unproved but valid

The operation may be valid, but analysis lacks a required fact because code is
opaque, alias relationships are complex, or another recognized proof boundary
intervenes. The programmer supplies the fact with a `safe<...>` claim:

```zax
/// The callback consumes the view synchronously and stores nothing.
safe<lifetime-escape>{
  invokeOpaqueCallback(view)
}
```

The claim says that a fact is true. It does not change runtime behavior.

### Unchecked operations

Some operations have a lower-level meaning that the language deliberately does
not check. They are written with `unsafe`:

```zax
header : PacketHeader * = myBuffer unsafe cast PacketHeader *   // treat these bytes as a header
```

No analysis could confirm that `myBuffer` holds a valid `PacketHeader`: the
bytes came from outside anything the language tracks. The programmer takes
responsibility for that.

A `safe<...>` claim and an `unsafe` operation differ in what the compiler could
ever know:

- a `safe<...>` claim supplies a fact about state the language tracks, which a
  compiler could prove with enough analysis;
- an `unsafe` operation relies on something the language does not track, such
  as raw bytes, foreign code, or ownership given up and later taken back.

Both must name the exact property and source region they affect.
[Vouching for safety](#vouching-for-safety-safe) explains how to choose.

### No valid interpretation

Neither `safe<...>` nor `unsafe` can manufacture:

- a resident instance where none exists;
- a life path after it ended;
- ownership that was never acquired;
- initialization that did not occur;
- a reference rebound to another place;
- a value of an incompatible type or representation;
- or any other known-impossible state.

```zax
value.---()
inspect(value) // error: no resident instance exists
```

No marker makes this ordinary access valid without first establishing another
instance through a real lifecycle operation.

The threshold for declaring a case impossible is intentionally high.
Conservative analysis belongs in the unproved-but-valid or unchecked
categories rather than being mislabeled as universal invalidity.

## Vouching for safety: `safe<...>`

Zax has two markers for places where the programmer takes responsibility:

```zax
safe<distinct-element>{                         // "this is safe; here is the fact"
  dispatch(inbox[2 * i] as last)
  dispatch(inbox[2 * i + 1] as last)
}

header : PacketHeader * = myBuffer unsafe cast PacketHeader *   // "this is unchecked"
```

In the first block, the two indexes always name different elements, but the
language's listed proofs do not cover two indexes per loop pass. The
programmer vouches for the fact. In the second line, nothing the language
tracks could say whether the bytes form a valid header.

To choose between them, ask what the claim is about:

- **Use `safe<category>{...}`** when the claim is a fact about state the
  language already tracks, such as lifetimes, aliases, presence, element
  distinctness, or where a member reference came from. A compiler could prove
  such a fact with enough analysis. When a language contract later requires
  that proof, the marker becomes redundant and you delete it, leaving the
  plain code underneath.
- **Use `unsafe`** when the claim concerns something the language deliberately
  does not track, or when code brings something back under tracking from that
  state. Examples are raw reinterpretation, ownership given up with
  `unsafe vacate` and later reclaimed, and an untracked `* unsafe` pointer. No
  analysis of Zax source can make such a marker redundant. Only a new
  declaration that the language recognizes, true on every compiler, could take
  over the vouching: for example, a future foreign-interface declaration
  saying "this call constructs its argument".

A false claim of either kind has undefined consequences. A `safe<...>` claim
is still the programmer vouching that the code is safe; the word names what
the programmer asserts, not what the compiler checked.

To audit source, search for `safe<`, which finds every claim and every
`unsafe<...>` enclosure, and for the word `unsafe`, which finds every
unchecked operation and untracked pointer.

### Current categories

Each category names one fact. The domain owner defines the fact precisely.

| Category | You vouch that | Owner |
| --- | --- | --- |
| `distinct-element` | Two array element accesses name different elements, or an element not finished earlier | [Arrays and slices](arrays-and-slices.md#what-the-compiler-can-prove) |
| `optional-presence` | The optional's boxed value is present here | [Optional values](optional-values.md#presence-proof-and-postfix-access) |
| `pointer-presence` | This pointer is not vacant here | [Nothing instances](nothing-instances.md#possible-vacancy-needs-a-presence-claim) |
| `replacement-alias` | A replacement's destination and right-hand operand do not alias in a way that matters | [Optional values](optional-values.md#presence-proof-and-postfix-access) |
| `lifetime-escape` | A reference's target outlives its use, where the compiler lost track of where the target came from | [Lifetimes and references](lifetimes-and-references.md) |
| `unique-raw-borrow` | A raw pointer does not come from inside a `unique` allocation | [Pointers, allocation, and arenas](pointers-and-arenas.md#raw-pointers) |
| `valid-representation` | These bits are a valid value of this type | [Unions](unions.md#unsafe-union), [floating-point scalars](floating-point-scalars.md) |
| `exact-origin` | A member reference really came from the named container member | [Composition](composition.md#proved-tracked-and-claimed-outer-casting) |
| `opaque-type` | An erased owner or observer really holds this type | [Pointers, allocation, and arenas](pointers-and-arenas.md#safe-owner-recovery) |
| `partial-instance-access` | Code given an incomplete instance touches only members already constructed | [Construction and destruction](construction-and-destruction.md) |
| `partial-instance-escape` | Code that receives an incomplete instance does not use it before it is complete | [Construction and destruction](construction-and-destruction.md) |
| `destruction-path-complete` | Every path through a destructor ended the member | [Construction and destruction](construction-and-destruction.md#destructor-declarations-and-sequence) |
| `destruction-at-most-once` | Paths that look like they overlap never end one member twice | [Construction and destruction](construction-and-destruction.md#destructor-declarations-and-sequence) |
| `terminal-reconstruction` | A member rebuilt during destruction is ended exactly once more | [Construction and destruction](construction-and-destruction.md#reconstruction-during-destruction) |
| `not-finished` | A member or value was not finished by an earlier `as last` the compiler cannot rule out | [Transfer stances](transfer-stances.md#members) |
| `disjoint-storage` | A structural transform's source and destination do not overlap | [Structural shapes](structural-shapes-and-compatibility.md#overlap-is-handled-statically) |
| `no-mutation-while-held` | Nothing writes a value while an `immutable` path to it is held | [Qualifiers](qualifiers.md#ordinary-promise-strengthening) |

Two categories are about things the language does not track, so they use
`unsafe<...>{...}`:

| Category | You take responsibility that | Owner |
| --- | --- | --- |
| `opaque-construction` | An opaque operation constructs the named member, which therefore gets no automatic construction | [Construction and destruction](construction-and-destruction.md#construction-by-an-opaque-operation) |
| `manual-member-destruction` | An opaque operation ends the named member, so its automatic destruction does not run | [Construction and destruction](construction-and-destruction.md#destructor-declarations-and-sequence) |

The unchecked operations are `unsafe cast`, `unsafe cast as`, `unsafe vacate`,
`unsafe pliable`, `unsafe as coercive`, `unsafe from`, `union unsafe`, and
the untracked pointer `* unsafe`. Their owners define them.

A compiler or vendor may define extension categories with `x-` names, such
as `safe<x-acme-ring-index>{...}`; see
[analysis provenance](#analysis-provenance).

### When each category arises

Each example shows code the compiler rejects, the fix that needs no marker
where one exists, and the claim otherwise. Names such as `calibrate` and
`registry` are placeholders.

**`partial-instance-access`.** A constructor passes its incomplete instance to
a helper:

```zax
MyEngine :: type {
  sensor : MySensor
  log : MyLog

  +++ final : ()() = {
    _.sensor .= makeSensor()
    calibrate(_)         // error: the compiler can't confirm calibrate touches only sensor
    _.log .= makeLog()
  }
}
```

Here the compiler cannot confirm that `calibrate`, whose body it cannot
follow, reads only `sensor` and keeps no reference. Better: pass only what is ready,
`calibrate(_.sensor)`. When the helper must take the whole instance, write
`safe<partial-instance-access>{ calibrate(_) }`.

**`partial-instance-escape`.** A constructor publishes its instance early:

```zax
+++ final : ()() = {
  _.sensor .= makeSensor()
  registry.add(_)        // error: other code could reach log before it exists
  _.log .= makeLog()
}
```

Better: register after every member exists. If registration must come first,
for example because the registry only records the address until a later
start-up step, write `safe<partial-instance-escape>{ registry.add(_) }`.

**`destruction-path-complete`.** A destructor destroys a member on two
complementary paths:

```zax
--- final : ()() = {
  if _.pooled {
    pool.returnSlot(_.slot)
    _.buffer.---()                     // destroyed on this path
  }
  if !_.pooled {
    flush(_.buffer)
    _.buffer.---()                     // and on this one
  }
}                                      // error: the compiler can't tell that every path destroyed buffer
```

The compiler destroys members still alive at exit without any runtime flag,
so it must know on every path whether `buffer` was destroyed. When it cannot
connect the two conditions, it reports an error rather than guess. Better: use
`if … else`. Otherwise wrap the second `if` in
`safe<destruction-path-complete>{...}`.

**`destruction-at-most-once`.** Two paths the compiler thinks could both run:

```zax
--- final : ()() = {
  if isPrimary(_) { _.buffer.---() }
  if isMirror(_)  { _.buffer.---() }   // error: buffer may be destroyed twice
}
```

The programmer knows an instance is never both. Better: `else if`. Otherwise
wrap the second line in `safe<destruction-at-most-once>{...}`.

**`terminal-reconstruction`.** A helper rebuilds and ends a member during
destruction:

```zax
--- final : ()() = {
  _.conn.---()
  reopenFlushAndClose(_)               // rebuilds conn, flushes it, and destroys it again
}                                      // error: the compiler can't confirm conn ended exactly once
```

Better: write the steps in the destructor. Otherwise wrap the helper call in
`safe<terminal-reconstruction>{...}`.

**`lifetime-escape`.** The compiler loses track of where a returned reference
came from:

```zax
pick final : (result : MyValue readonly &)(a : MyValue readonly &, b : MyValue readonly &) = {
  return chooser(a, b)                 // error: where does chooser's result come from?
}
```

The programmer knows `chooser` returns one of its arguments, so
`safe<lifetime-escape>{ return chooser(a, b) }`.

**`unique-raw-borrow`.** A raw pointer made from a reference, or from `_`,
whose origin the function cannot see:

```zax
MyValue :: type {
  selfPtr final : (result : MyValue loan *)() = {
    safe<unique-raw-borrow>{
      return _                         // _ might live inside a unique allocation
    }
  }
}
```

A raw pointer made directly from a `unique` owner is a proved violation that
no claim fixes; see [raw pointers](#raw-pointers).

**`pointer-presence`.** An opaque call inside a result that promises presence:

```zax
acquire final : (result : MyValue * last = @)() = {   // = @ allocates first and promises callers a present result
  reset result
  result = foreignAcquire()            // error: foreignAcquire might return vacant
}
```

Better: restore presence before returning, with `if !?result { result = @ }`
after the call. Otherwise wrap the call in
`safe<pointer-presence>{ result = foreignAcquire() }`.
[Nothing instances](nothing-instances.md#possible-vacancy-needs-a-presence-claim)
shows the more common case of passing a possibly vacant pointer to a writer.

**`exact-origin`.** A cast to a container the compiler cannot prove:

```zax
safe<exact-origin>{
  container : Container & = memberReference outer cast Container.member
}
```

**`opaque-type`.** Recovering a typed owner from an erased one without the
runtime check:

```zax
safe<opaque-type>{
  vouched : MyType * unique = opaque as last       // you vouch opaque holds a MyType
}
```

**`valid-representation`.** Reading bits through a type that has invalid bit
patterns:

```zax
value.raw = readRawCode()
safe<valid-representation>{
  use(value.code)                       // you vouch the bits name an admitted code
}
```

The same claim covers a reference into a union lens that a later write might
have invalidated, and a floating-point pattern read without validation.

**`not-finished`.** A use after an `as last` the compiler cannot rule out:

```zax
if mustArchive(message) {
  archive(message.payload as last)     // payload finished on this path
}
if !mustArchive(message) {
  message.payload.send()               // error: payload may have been finished
}
```

Better: `if … else`. Otherwise wrap the use in
`safe<not-finished>{ message.payload.send() }`. Tracking a member finished
inside a called function, through a writable parameter, is future work; today
[transfer stances](transfer-stances.md#members) leaves that to the function
and the type.

**`disjoint-storage`.** A structural transform whose source cannot be copied
aside first:

```zax
// GridTranspose maps each cell to its transposed position;
// MyGrid's cells allow neither copy nor move
adjust final : ()(target : MyGrid writable loan &, source : MyGrid readonly loan &) = {
  target -<>- GridTranspose -<>- source   // error: target and source may be the same grid
}
```

Better: pass grids that are visibly distinct, or make the cells copyable.
Otherwise wrap the transform in `safe<disjoint-storage>{...}`.

**`no-mutation-while-held`.** An `immutable` view of a mutable value, when an
opaque call also receives the value:

```zax
counter : Counter mutable writable = makeCounter()
snapshot : Counter immutable readonly & = counter
logCounters(counter)                 // error: logCounters could write counter while snapshot is held
inspect(snapshot)
```

Better: pass `logCounters` a readonly view, or finish with `snapshot` first.
If `logCounters` takes a writable parameter but never writes, wrap the call in
`safe<no-mutation-while-held>{...}`.

**`distinct-element`.** A loop that finishes two elements per pass, which the
language's listed array proofs do not cover:

```zax
each i : in 0..<inbox.length() / 2 {
  dispatch(inbox[2 * i] as last)
  dispatch(inbox[2 * i + 1] as last)   // error: the compiler can't confirm this is a different element
}
```

The programmer knows `2 * i` and `2 * i + 1` never collide, so wrap both
calls in `safe<distinct-element>{...}`.
[Arrays and slices](arrays-and-slices.md#what-the-compiler-can-prove) lists
the cases that need no claim.

**`optional-presence`.** An optional read where the presence proof is out of
the compiler's sight:

```zax
loadIfMissing(cache)                   // an opaque call that always fills cache
use(cache.)                            // error: the compiler can't confirm cache is present
```

Better: test with `if ?cache`. Otherwise wrap the use in
`safe<optional-presence>{ use(cache.) }`.

**`replacement-alias`.** A replacement the compiler cannot prove is safe from
an alias:

```zax
oldValue : MyValue & = optional.
opaqueObserve(oldValue)                // might keep oldValue somewhere
reset optional                         // error: a retained alias would see the ended value
```

The programmer knows `opaqueObserve` keeps nothing, so wrap the reset in
`safe<replacement-alias>{ reset optional }`.

**`opaque-construction`** and **`manual-member-destruction`** mark an opaque
operation that constructs or ends a member:

```zax
--- final : ()() = {
  unsafe<manual-member-destruction>{
    foreignRelease(_.handle)           // the C library ends handle; don't destroy it again
  }
}
```

### A check counts only if the compiler can follow it

Many claims can be avoided with a check: a `?pointer` test before access, or
an `is type` test before recovering an erased owner. The compiler must be able
to recognize the check:

```zax
if opaque is type MyType {
  typed : MyType * unique = opaque as last         // fine: the test is recognized
}

isWanted := fooBarChecking(opaque is type MyType, extraCheck) || goodToGo
if isWanted {
  typed : MyType * unique = opaque as last         // checked recovery: the compiler can't follow the test
}
```

The language contract defines which check shapes count, so every compiler
agrees on what compiles. A cleverer compiler may recognize more, but that
never changes whether the code compiles; see
[asking a compiler to verify a claim](#asking-a-compiler-to-verify-a-claim).
When a check is too tangled to count, simplify it or write `safe<...>`.

## Markers are narrow

Neither marker is a broad alternate language mode. Every `safe<...>` claim and
`unsafe` operation:

- uses the literal word `safe` or `unsafe`;
- identifies the exact category or operation;
- encloses or names the affected operation, binding, member, or expression;
- applies at the narrowest useful source location;
- permits attached documentation explaining the responsibility;
- alters only the stated proof or boundary; and
- requires no runtime check unless the underlying operation independently
  specifies one.

One enclosure may name several categories without granting anything else:

```zax
safe<optional-presence, replacement-alias>{
  observe(optionalValue.)    // optional-presence: you vouch a value is present
  reset optionalValue        // replacement-alias: you vouch observe kept no alias
}
```

Enclosures create no scope and no value, as
[compiler directives](compiler-directives.md#directives-enclosures-and-compile-time-expressions)
describes.

## False claims

A false `safe<...>` claim, or a false assumption behind an `unsafe` operation,
may have undefined consequences. The language does not require a runtime
check. Compiler debugging modes may instrument claims and panic when misuse is
detected; that is a tooling feature, not a language guarantee.

A claim may fill a proof gap. It cannot contradict a fact the compiler already
knows. A compiler that proves a claim false, with certainty, reports an error:

```zax
value.---()                                 // value's instance is destroyed

safe<lifetime-escape>{
  inspect(value)                            // error: the claim says value is alive; the compiler knows it was destroyed
}
```

No marker makes access to a destroyed instance valid. Sometimes low-level code
genuinely needs to look at the storage left behind, for example a unit test
checking that a destructor cleared a buffer stored in an arena, whose storage
stays allocated after the destructor runs. Such code reads the storage through
an untracked pointer taken while the value was alive. The compiler does not
check accesses through it, which is not the same as making them valid:

```zax
secret : MySecretBuffer = loadSecret()
probe : MySecretBuffer * unsafe = secret   // untracked borrow, taken while secret is alive
secret.---()                               // the destructor clears the buffer
checkCleared(probe.)                       // compiles: untracked, so the check is on you
inspect(secret)                            // error: no instance exists
```

This is an escape hatch, not a pattern to reach for: whether reading the
leftover storage means anything is entirely the programmer's responsibility.

## Diagnosable unchecked Nothing access

Postfix pointer dereference generates no runtime vacancy test. Through an
untracked `* unsafe` pointer, nothing checks vacancy at all:

```zax
pointer : MyValue * unsafe = choosePointer()
view : MyValue & = pointer.     // no check
```

If the compiler proves `pointer` vacant, it reports an error. If `pointer` is
vacant at runtime anyway, the reference reaches the type's Nothing backing.
Reading a prepared member may succeed, reading an unprepared member uses its
defined panic path where available, and a write to compiler-provided shared
backing is invalid.

Zax does not require a generated vacancy check on every read or store. Suitable
target protection traps writes and requested trapping-policy reads; debug
instrumentation may detect additional misuse. When the target lacks the needed
trap, a missed access can read prepared backing, corrupt shared backing, or have
undefined consequences.

For a tracked pointer, every access that might reach Nothing backing needs
proof that the pointer is not vacant, or a `safe<pointer-presence>` claim,
whatever the type's Nothing policy. An access the compiler proves vacant is an
error. The error and the claim belong at the operation that dereferences the
pointer, such as `myPointer.`, not inside a function that later receives the
resulting reference:

```zax
counter : MyCounter * = findCounter(id)    // tracked; vacant when id is unknown
safe<pointer-presence>{
  writeCount(counter.)                     // you vouch counter is not vacant here
}
```

A claim or a recognized `?counter` test establishes that `counter` is not
vacant for later uses on the same path, until something could change the
pointer: a reset, a
reassignment, or a call that can write it. Across calls, tracking follows
source: when every call path into a function passes a non-vacant pointer, the
function's own accesses need nothing. Once the function is compiled into a
fixed exported interface, such as a library binary whose callers are compiled
separately, its callers can no longer be followed, and a possibly vacant
access inside it is an error. Whether owning and managed pointers follow this
tracked rule is future pointer-tracking work.

Exact Nothing behavior is owned by
[Zax Nothing instances](nothing-instances.md#pointer-dereference-is-unchecked).

## Intent, safety claims, unsafe, and linting are different

| Mechanism | Programmer statement |
| --- | --- |
| `intent<...>{...}` | “This defined but suspicious interpretation is deliberate.” |
| `safe<...>{...}` | “This required fact is true although the compiler cannot prove it.” |
| `unsafe` operation or `unsafe<...>{...}` | “I take responsibility for something the language does not track.” |
| Lint suppression | “Do not report this advisory diagnostic here.” |

### Intent does not accept safety responsibility

```zax
intent<terminal-source-reuse>{
  resource.releaseRemainingHandle()
}
```

This confirms defined terminal-source behavior. It does not grant lifetime,
ownership, mutation, or alias authority.

An interior pointer or a member reference may require intent because
replacement will make it observe a new member instance. The operation is
defined; the acknowledgement confirms that surprising meaning, as
`intent<inner-pointer-tracks-replacement>` and
`intent<reference-tracks-replacement>` do.

### Lints do not change semantics

A reference may safely observe a value changed through another authorized path.
A lint can ask whether that observation was intended without making the source
unsafe.

Lint suppression:

- changes no language behavior;
- creates no proof;
- grants no permission;
- suppresses no required semantic error; and
- should not require `safe<...>` or `unsafe`.

## Domain rules remain with their owners

This document owns the reusable safety model. Domain owners define the facts to
which it applies.

Examples:

- [construction and destruction](construction-and-destruction.md) defines when
  instances and members are live or complete;
- [lifetimes and references](lifetimes-and-references.md) defines reference
  targets, escape, and replacement boundaries;
- [pointers and arenas](pointers-and-arenas.md) defines ownership, weak
  acquisition, arena obligations, and raw-pointer responsibility;
- [optional values](optional-values.md) defines presence and boxed access;
- [function invocation](function-invocation.md) defines result and parameter
  completion;
- [composition](composition.md) distinguishes proof-required, tracked checked,
  and claimed outer casting, defines the receiver-origin proof required for
  non-optional outer result remapping, and owns `tracked via` and the
  `exact-origin` claim;
- [transfer stances](transfer-stances.md) defines moved-from and terminal source
  states;
- integer owners define overflow, narrowing, and required-result behavior; and
- [enums](enums.md) defines strict and flags admission domains and the values
  produced by enum operations.

A category does not become a duplicate definition of the domain rule it
qualifies.

In particular, an intent acknowledgement cannot authorize a composition
provenance claim. A `safe<exact-origin>` claim accepts responsibility for an
unproved relationship at the exact forwarding or outer-cast site. A false
claim has the consequences defined by the composition operation.

### Union lens validity

A plain [union](unions.md) admits only passive, all-bit-pattern-valid lenses, so
every lens remains safe after every write.

`union unsafe` admits passive representations with restricted value domains.
The selected language contract may prove one lens valid from zero-fill, a
preceding write, exact representation facts, or preserved flow. When it cannot,
reading the backing through that lens needs a `safe<valid-representation>`
claim:

> The current union backing is a valid ordinary value of the requested lens's
> type.

It neither converts the bits nor starts a lens lifetime. A false claim has
undefined consequences. Known-invalid representation remains an error. The
same category covers other reads of bits through a type with invalid bit
patterns, such as an unvalidated
[floating-point](floating-point-scalars.md) pattern.

A compiler's private analysis may advise, but it does not change which claims
source must write. Stronger source validity belongs to a selected contract as
described under
[language contracts and compiler analysis](#language-contracts-and-compiler-analysis).

### Checked variant access

Named [variant](variants.md) access has a runtime-checkable selection condition:

- proved active removes the check;
- proved inactive is an error;
- unresolved selection performs the registered
  `inactive-variant-access` check and panics on mismatch; and
- disabling that category promises the requested name is active, with undefined
  consequences if the promise is false.

Disabling the check never suppresses a statically proved mismatch and does not
affect `.=` or switch routing, which establish selection through their own
operations.

This supplies a concrete distinction among proved invalid source,
runtime-checkable uncertainty, an uncheckable proof gap, and high-confidence
compiler suspicion. Warning policy for the last category remains future
analysis-control work; source validity does not vary with an implementation's
confidence heuristic.

### Enum raw-admission permission

```zax
raw : U8 = readByte()
value := MyEnum unsafe from raw
```

Enum `unsafe from` is a defined permission to bypass the enum owner's ordinary
admission policy while preserving the exact backing representation. A strict
enum need not name the value, and a flags value may contain bits outside its
safe allowed mask. The result is still a defined enum value whose available
operations follow their declarations.

This permission does not perform integer conversion or accept a different
backing type. It changes no `optional from` result and does not expand ordinary
safe admission. Complete enum consequences are defined by
[Zax enums](enums.md#unsafe-from).

## Representative lifecycle boundaries

### Partial construction

A constructor helper may need access to already-live members before the complete
enclosing instance exists. That bounded access differs from publishing the
incomplete instance through a global, callback, or stored reference.

Claims must distinguish:

- bounded partial-instance access (`partial-instance-access`);
- escape or external publication (`partial-instance-escape`);
- asserting that opaque work established a member, which
  [`opaque-construction`](construction-and-destruction.md#construction-by-an-opaque-operation)
  does at the establishing operation and which also takes the member out of
  automatic construction;
- suppressing a generated destruction because opaque code ends the member
  (`manual-member-destruction`); and
- claiming path completeness or at-most-once behavior
  (`destruction-path-complete`, `destruction-at-most-once`).

### Replacement and aliases

A replacement right-hand operand may alias the destination or one of its
members. Analysis must account for whether the operation:

- copies required values before ending them;
- consumes an alias before renewal;
- delegates exact self-alias handling to the selected operation;
- preserves only storage or resources rather than an old member lifetime.

Where analysis cannot settle a possible alias, `safe<replacement-alias>`
claims it does not matter. A reference that deliberately goes on to observe the
new member after replacement is not a safety question at all; it uses
`intent<reference-tracks-replacement>`, described by
[lifetimes and references](lifetimes-and-references.md#direct-member-references-cross-a-renewal-boundary).

### Returned references

A returned reference must target a place whose life path outlives every use.
Returning a local or temporary is known invalid when it escapes. When the
compiler loses track of where a valid result came from, the function needs a
`safe<lifetime-escape>` claim or a future callable-origin contract.

### Raw pointers

A plain borrowing raw pointer is tracked: the compiler checks, at each access
to what it points to, that the target is still alive. A pointer declared
`* unsafe` is not tracked, and every access through it is the programmer's
responsibility:

```zax
node : MyNode = makeNode()
link : MyNode * = node               // tracked: checked at each use
parent : MyNode * unsafe = link      // untracked from here on
back : MyNode * = parent             // error: an untracked pointer can't become tracked again
```

A non-vacant pointer still does not prove:

- live pointee storage;
- a resident instance;
- valid provenance;
- alignment;
- permitted aliasing; or
- ownership.

A pointer operation is safe when analysis proves every fact required by the
selected contract. `?pointer` proves only non-vacancy. Origin analysis may prove
the remaining facts, or an escape, opaque operation, owner release, reset,
relocation, or arena teardown may invalidate that proof. Where a required fact
cannot be proved, the code needs a `safe<...>` claim for that fact or a
`* unsafe` pointer. A claim cannot make a known-invalid pointee valid, and a
`* unsafe` pointer is simply not checked.

One fact concerns where a raw pointer comes from: a `unique` allocation allows
no raw borrowers.

- A raw pointer made directly from a `unique` owner is a proved violation. No
  claim fixes it; the only escape is an untracked `* unsafe` borrow.
- A raw pointer made from a reference whose origin the function can't see (a
  reference parameter, a reference result, or `_`) might reach a `unique`
  allocation. When the compiler cannot rule that out, the function needs a
  `safe<unique-raw-borrow>` claim. Raw pointers from visible values, such as
  a local, a member of a local, or the value behind an owning raw pointer,
  need none.

```zax
owner : MyValue * unique = @
raw : MyValue * = owner.               // error: unique allows no raw borrowers
escape : MyValue * unsafe = owner.     // allowed: untracked, fully on you

makePtr final : (result : MyValue readonly loan *)(input : MyValue readonly loan &) = {
  safe<unique-raw-borrow>{
    return input                       // you vouch input is not inside a unique allocation
  }
}
```

Complete rules belong to
[pointers, allocation, and arenas](pointers-and-arenas.md#raw-pointers).

Whether a raw pointer is responsible for freeing its allocation is decided by
its stance alone, so allocation, reset, and reassignment need no tracking:

```zax
owner : MyValue * last = @     // owns the allocation and frees it
reset owner                    // safe: frees it now and leaves owner vacant
owner = @{ anotherArena }      // safe: frees any current allocation, then holds the new one

view : MyValue * = owner.      // a borrowing raw pointer
reset view                     // error: a borrowing pointer holds nothing to free
```

Ownership passes between owning raw pointers and managed owners with `as last`,
without `unsafe`. Taking ownership through a borrowing raw pointer is the one
place where the compiler cannot know the facts: whether the address is the
start of an allocation, and whether anything else will free it. That needs the
stance cast, which asserts both:

```zax
restored : MyValue * last = (view unsafe cast as last) as last
```

A pointer proved to identify an interior member, an ended allocation, a
stack or global place, or an allocation another owner will free remains
known-invalid even under `unsafe`. Intent acknowledgement cannot supply
lifecycle proof.

Raw `vacate` follows the same distinction. On a borrowing raw pointer it is
always safe, because no responsibility is lost. On an owning raw pointer,
`unsafe vacate` gives up the responsibility for freeing without freeing.
Managed pointers reject `vacate` even under `unsafe` because bypassing their
release guarantees a leak or corrupts ownership accounting.

### Opaque owner and observer recovery

`OpaqueOwner`, `OpaqueObserver`, and `OpaqueReferenceObserver` preserve a private
exact type/capability witness while hiding typed access.

`value is type MyType` is a safe nonexecuting proof operation. It transfers
nothing, creates no access path, and does not prove liveness. Safe recovery still
checks ownership role, origin, and qualification authority:

- owner or pointer recovery can report failure through a vacant typed pointer;
- reference-observer recovery panics on mismatch because a reference cannot be
  vacant; and
- a successful preceding `is type` test supplies the exact type fact to its
  proven flow path.

A recovery written inside `safe<opaque-type>{...}` skips the type-witness
check because the programmer vouches for the type. It cannot recover an erased
interior target from allocation-root ownership, regain lost qualifications, or
turn a nonowner into an owner. A known mismatch remains an error; a false
claim has undefined consequences. The `is type` test counts as proof only when
the compiler can follow it; see
[a check counts only if the compiler can follow it](#a-check-counts-only-if-the-compiler-can-follow-it).

Complete domain behavior belongs to
[Zax pointers, allocation, and arenas](pointers-and-arenas.md#type-erased-ownership-and-observation).

### Array bounds, slices, and disabled checks

When the compiler can prove that an intrinsic array operation is invalid—for
example, indexing a fixed five-element array at index ten—it rejects the source.
When the deciding index, length, or source count is known only while running,
the required operation checks it and panics on failure.

The array failures are kept separate: element bounds, slice bounds/order,
requested length, initializer element count, invalidated slice, foreign slice
origin, broken iterable count promise, storage-size overflow, and required
backing allocation.

A programmer may independently disable one registered failure category. That
choice tells the compiler to assume the failure cannot happen and permits it to
remove the corresponding check. If the promise is false, behavior is undefined.

Optional/reporting array operations retain the checks required to produce their
declared results even when the corresponding required-operation panic is
disabled. A slice validity check may carry origin and generation cost; disabling
it transfers responsibility rather than turning an invalid slice into an empty
one.

Array and storage providers may cooperate through future stability
kinds/tokens and observed versions. The provider reports changed raw mappings;
the array determines which element-place guarantees ended. A version mismatch
is the runtime evidence for the existing invalidated-slice panic, not a second
kind of slice failure.

Complete array conditions and programmer-visible behavior belong to
[Zax arrays and slices](arrays-and-slices.md#required-reporting-and-unchecked-behavior).
Exact registry syntax and identifiers remain analysis-control work.

## Runtime failure and unsafe failure

Defined runtime failure is not the same as unsafe undefined behavior.

Examples of defined failure include:

- a failed shared-to-unique ownership claim producing an empty unique pointer;
- a failed weak acquisition producing an empty strong pointer;
- a non-panicking allocation producing a vacant pointer;
- a panicking allocation failing its arena request;
- a checked operation returning absence; and
- a required runtime check panicking.

A false `safe<...>` claim, or a false assumption behind an `unsafe`
operation, may instead have undefined consequences with no required check.

Future contracts should state which failure model applies rather than leaving it
to compiler accident.

## Language contracts and compiler analysis

Source may eventually select a language-contract version through source,
project, build, or toolchain configuration. Omitting an explicit version accepts
the active environment's contract; it does not claim compatibility with every
past or future contract.

The selected contract defines portable required analysis.

The source-validity outcomes are:

| Proof status | Portable source consequence |
| --- | --- |
| Selected-contract-required proof succeeds | Use the plain unmarked form; a redundant `safe<...>` claim is an error, while a defined but suspicious alternate safe mode may require intent acknowledgement; every conforming compiler must recognize the proof |
| Selected contract does not require the proof | The code still needs its narrow `safe<...>` claim; one compiler's stronger private proof cannot make omitting it portable or reject the retained claim |
| Explicitly selected stronger extension proof succeeds | That proof may become canonical, making the unmarked form required and a redundant claim an error for that source |
| Any compiler proves the claimed fact false | Reject the operation, with or without the claim |

Composition provenance is a concrete example:

```zax
proved : Container & =
  memberReference outer cast Container.member

deliberatelyTracked : Container & ? =
  intent<redundant-outer-tracking>{
    memberReference tracked outer cast Container.member
  }
```

If the selected contract requires and establishes exact-origin proof at this
site, plain `outer cast` produces the non-optional result without tracking, while
retaining a `safe<exact-origin>` claim around it is an error. A deliberately
retained tracked operation requires `intent<redundant-outer-tracking>`,
preserving its optional result and potential tracking costs without forcing a
physical metadata read.

Without contract-required proof, plain `outer cast` alone is rejected. The
programmer may write the optional `tracked outer cast`, or write the plain
cast inside `safe<exact-origin>{...}` for an unchecked non-optional result,
even if one compiler privately proves the origin. That private proof cannot
require the intent category, change the tracked result shape, or reject the
portable claim. An explicitly selected stronger extension may make plain
`outer cast` canonical and make tracked use acknowledgement-required. A
compiler-proved impossible origin rejects the plain and claimed forms; the
tracked checked form instead produces absence.

The composition owner defines the exact origin proof class and applies the same
distinction to callable result mapping: plain `via` for a proved non-optional
outer result, `tracked via` for a checked optional result, and plain `via`
inside `safe<exact-origin>{...}` for an unproved non-optional result.

### A compiler may prove more

Suppose source contains a `safe<...>` claim because the selected contract does
not require compilers to prove a valid member-lifetime relationship.

A newer compiler may prove it. The claim stays required: removing it would make
the source depend on that compiler's stronger analysis. What the newer compiler
may do is report that it can prove the site, so the programmer can ask it to
keep checking; see
[asking a compiler to verify a claim](#asking-a-compiler-to-verify-a-claim).

### A later or extension contract may require more

A later mainline contract, or an explicitly selected extension contract, may
require conforming compilers for that contract to recognize the case as safe.

Under that explicitly selected contract:

- the obsolete claim is an error, and so is any `[<verify>]` directive above
  it, because it no longer has a claim to verify;
- every conforming compiler must accept the unmarked valid operation;
- a defined alternate safe mode may become acknowledgement-required when the
  contract proves it redundant; and
- a compiler that still demands the claim is nonconforming.

This rule applies to every mandatory analysis, not only lifetime or optional
presence.

### Analysis provenance

Categories need visible provenance:

- language-contract categories;
- compiler or vendor categories, named with `x-`; and
- community extensions implemented by multiple tools, also named with `x-`.

A vendor category is a fact that vendor's compiler defines and checks, as in
`safe<x-acme-ring-index>{...}`. That compiler may report a problem other
compilers miss, or confirm code the programmer wants checked. Another compiler
that does not know the category may ignore it only when it independently
proves the source valid without it. Otherwise it reports that the required
extension is unsupported.

This differs from [asking a compiler to verify](#asking-a-compiler-to-verify-a-claim)
a standard category: there, every compiler understands the claim, and the
directive only asks one of them to check it.

## Asking a compiler to verify a claim

A `safe<...>` claim stays in source even when your compiler could prove it, so
that every compiler accepts the same program. You can still ask a capable
compiler to keep checking the claim, so that an edit that breaks it is caught
instead of silently trusted:

```zax
[<verify=x-acme>]
safe<distinct-element>{
  dispatch(inbox[2 * i] as last)
  dispatch(inbox[2 * i + 1] as last)
}
```

The `[<verify=...>]` directive goes on the line directly above the
`safe<...>` enclosure. It names a compiler or extension specification by an
`x-` name: a specific compiler (`x-specific-compiler-name`) or a registered
extension specification (`x-extension-spec-name`), chosen so that it never
collides with anyone else's. To ask several compilers, repeat the directive:

```zax
[<verify=x-acme>][<verify=x-other>]
safe<distinct-element>{
  dispatch(inbox[j] as last)
}
```

`[<verify=x-acme>]` concerns only the `x-acme` compiler; every other compiler
ignores it. A misspelled name is simply never checked. An enclosure naming
several categories is verified for all of them. A `[<verify>]` with no
`safe<...>` enclosure below it is an error.

A reviewer reading the source can tell a claim a named compiler keeps checking
from one only the programmer vouches for.

### What a compiler does at a claim

| Situation at a `safe<...>` site | Result |
| --- | --- |
| The selected contract's rules prove the site | Error: remove the claim |
| Any compiler proves, with certainty, that the claimed fact is false, even using analysis beyond the contract | Error: fix the code. This happens at every reporting level, including `off` |
| A compiler proves the site only by going beyond the contract, and no `[<verify>]` names it | A finding suggesting `[<verify=x-acme>]`, reported at the [`safe-verification` level](#how-loudly-findings-are-reported) |
| `[<verify=x-acme>]` is present and `x-acme` still proves the site | Nothing reported |
| `[<verify=x-acme>]` is present and `x-acme` can no longer prove the site, without proving it false | A finding asking to fix the code or remove the directive, reported at the `safe-verification` level |

"Proves false" means certainty, not suspicion.

The first and third rows differ in whose proof it is. When the language
contract proves the site, every compiler agrees the claim is redundant, so it
is an error everywhere. When only one compiler can prove the site, the claim
stays valid Zax on every compiler; that compiler reports a finding, and the
build's `safe-verification` level decides whether the finding is advice or
fails the build.

### Checks inside `wellformed`

A `wellformed{...}` answer follows the language contract alone. A cleverer
compiler's analysis never changes it, so every compiler takes the same branch:

```zax
if compile wellformed{ pairwiseDispatch$(MyMessage)(inbox) } {
  pairwiseDispatch$(MyMessage)(inbox)   // its body has a [<verify=x-acme>] claim
} else {
  serialDispatch(inbox)
}
```

Findings from a cleverer compiler's analysis inside a `wellformed` payload,
both "this claim is false" and "this site can be verified", are reported at a
separate `wellformed` level, which defaults to `lint`. Code that is also
compiled outside the payload, like `pairwiseDispatch$(MyMessage)` in the
selected branch above, is judged there under the ordinary rules. The separate
level matters for code that exists only as a probe, which should not break a
build. The same holds for `if compile`, `switch compile`, constraints,
overload and specialization selection, and reflection: none of them sees a
verification result.

### How loudly findings are reported

`[<safe-verification=...>]` chooses whether the two findings above are
silent, advice, or errors:

| Level | Effect |
| --- | --- |
| `off` | The "can be verified" and "can no longer be verified" findings are not reported, so the compiler need not search for proofs. A claim proved false is still an error |
| `lint` | The findings are advice |
| `error` | The findings fail the build |

A separate `wellformed=` level, with the same values, covers findings inside
`wellformed` payloads.

Your own code defaults to `error`, and the `wellformed=` level defaults to
`lint` everywhere. The `error` default means a capable compiler asks you to
resolve every finding in your own code, by adding `[<verify>]` or fixing the
claim, so that a library you publish produces no findings in its users'
builds. Which programs are valid Zax is still the same on every compiler; the
default only decides whether your build stops on the finding. The level for
your own code comes from a compiler or build option. Each imported module
instance gets its level inside the import statement that creates it,
defaulting to `lint`:

```zax
C :: import Module.CModule {
  [<safe-verification=off>]                    // quiet for this whole instance of C
}

D :: import Module.DModule {
  [<safe-verification=error, wellformed=error>]
}

E :: import Module.EModule                     // nothing written: lint, and lint inside wellformed
```

- **The importer's choice applies to the whole instance.** A module's own
  `safe-verification` directives apply when that module is the code being
  built. When it is imported, the importer's setting governs it, so a
  library's strict setting cannot fail its users' builds over findings they
  cannot fix.
- **The setting stops at that instance.** If C imports another module, C's
  own import statement decides that module's level.
- **Re-exported and shared instances keep the level of the import that
  created them.** Each instance has one level, so each claim has one answer.
- **A finding is judged where the claim is written**, under the level of the
  instance whose source contains it. When a generic's claim fails only for one
  instantiation, the diagnostic also names the instantiation, so the author
  whose type broke the proof can see it.
- **Inside ordinary code**, `[<safe-verification=...>]` can also be used as a
  [range](compiler-directives.md#directive-ranges) to tune particular regions;
  this is expected to be rare.

Panic-check state never crosses an import, because no module may make
undefined-behavior promises about code it did not write. The
`safe-verification` level may cross an import because it changes only what
the compiler reports, never what the program does.

## Panic boundary

A panic blocks the operation that encountered its condition. A narrowly
applicable helper may repair that condition and allow the same operation to
complete as though the failure had not occurred. Otherwise the process crashes
gracefully.

Resolution does not:

- skip the failed operation;
- supply an unrelated substitute result;
- unwind completed members;
- roll back construction or replacement;
- continue after incomplete destruction; or
- expose partial lifecycle state to ordinary code.

This exact boundary applies the reusable
[Panic is not alternate control flow](principles.md#panic-is-not-alternate-control-flow)
principle. A concept owner links here when lack of unwinding changes local
construction, mapping, disposal, or replacement behavior rather than redefining
panic independently.

Panic categories can be enabled or disabled independently. Disabling a category
promises that its condition cannot occur, and lets the compiler omit the check.
If the condition occurs anyway, behavior is undefined. Other panic categories
remain enabled.

For allocation, this is not a third unchecked operator: `@` remains panicking
and `@!` must still detect request failure to return a vacant pointer.

### Controlling panic checks

A panic control is a [compiler directive](compiler-directives.md) naming one
category. Attached, `[<panic=no, ...>]` disables the check for one statement
or declaration, and `[<panic=yes, ...>]` enables it:

```zax
// Category names are illustrative; the registry is future work.
[<panic=no, fixed-arithmetic-overflow>]
total = total + delta   // no overflow check for this statement
```

As a [range](compiler-directives.md#directive-ranges), unattached state markers
set the check's state for the code that follows, inside one opening or file:
`never` disables it, `always` enables it, `inherit` restores the state the
opening started with, and `default` restores the language's default:

```zax
MyMixer :: type {

  [<panic=never, fixed-arithmetic-overflow>]

  accumulate final : ()() = {
  }

  [<panic=inherit, fixed-arithmetic-overflow>]

  audit final : ()() = {
  }

}
```

Here `accumulate` has no overflow check, and `audit` is back to the state
`MyMixer` started with. The closing brace restores the surrounding state in any
case.

Rules that keep these promises honest:

- **Name each category.** There is no "disable every check" form, so a category
  added to the language later is never disabled without someone naming it.
- **Imports are a wall.** An importer's panic state never reaches the modules it
  imports, so no module makes undefined-behavior promises about code it did not
  write.
- **Build configuration sets the starting state.** The build may set a module's
  starting state before its own source runs, and the module's directives refine
  it from there. A build that disables a check for a dependency is an explicit,
  auditable choice by whoever owns the build.
- **Not a build mode.** Whether a check exists never depends on debug or release
  mode, only on these controls.

Exact category names remain future analysis-control work.

## Diagnostics

Safety diagnostics should identify:

- the selected language contract;
- the exact required property;
- the source operation and affected place, value, pointer, or result;
- the available proof and the missing or contradictory fact;
- the exact `safe<...>` category or `unsafe` operation that would apply, so
  the programmer never has to look up a category name;
- the consequence of a false claim;
- and any runtime check or failure that the selected operation guarantees.

Diagnostics should distinguish:

- unproved valid operation that a `safe<...>` claim can cover;
- an operation that needs an `unsafe` form;
- known-invalid operation, including a claim proved false;
- a claim made redundant by the selected contract;
- a site a named compiler can verify, and a `[<verify>]` site it can no longer
  verify, each with the `safe-verification` level that applied;
- a `[<verify>]` directive with no claim below it;
- defined but suspicious intent;
- advisory lint;
- unsupported extension category; and
- compiler nonconformance with a selected contract.

Do not report a vague “unsafe” or “lifetime” error when the actual issue is an
ended member path, missing allocation root, possible callback escape, unknown
presence, duplicate disposition, or another precise fact.

## Costs

Static proof may increase compile time and diagnostic complexity while adding no
runtime state.

Runtime-checked policies may add:

- branches;
- generations or state flags;
- control blocks;
- synchronization;
- allocation;
- panic paths; or
- larger values.

Claims can remove required runtime checks where the contract permits that
model, but they transfer correctness responsibility to the programmer. They are
not optimization hints detached from semantics. Verifying a claim adds compile
time and changes no generated code.

## Source stability

Portable source validity follows the selected language contract.

Compiler improvements must not silently:

- reinterpret references or pointers;
- choose another transfer stance;
- add ownership;
- remove required synchronization;
- turn an `unsafe` operation into ordinary safe meaning;
- or change defined runtime failure into undefined behavior.

A new contract may deliberately strengthen mandatory proof or reject obsolete
claims. Such changes must be explicit and reproducible.

A compiler upgrade that proves less than before can turn passing `[<verify>]`
sites into findings. Because your own code defaults to `error`, that can fail
a build until the directive is removed or the level is lowered; the program's
meaning does not change.

## Boundaries and maturity

This document establishes current conceptual safety direction. It is not a
formal safe-subset specification, proof calculus, diagnostic registry, or
compiler conformance suite.

Still deferred:

- final category names, which may be revised, and exact enclosure nesting and
  reflection rules;
- which check shapes each contract version recognizes as proof;
- mandatory proof sets for each contract version;
- who maintains the registry of `x-` extension specification names;
- formal analysis provenance and extension rules;
- debug instrumentation;
- formal panic behavior;
- data-race guarantees;
- FFI and assembly trust boundaries;
- resource exhaustion, deadlock, and starvation;
- and broader security correctness.
