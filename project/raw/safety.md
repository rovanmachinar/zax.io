# Raw input: safety boundaries

| Field | Value |
| --- | --- |
| Status | Raw residual future-work input / non-authoritative |
| Audience | A future numbered work item defining safety guarantees and unsafe boundaries |
| Applies To | Unresolved safety guarantees, domain categories, and comparative safety input |
| Owns | Preservation of unresolved guarantee categories, domain-specific safety pressure, and comparison material |
| Does Not Own | Current safe-subset, proof, unsafe-permission, and contract-evolution behavior ([safety and analysis](../../language/safety-and-analysis.md)); or accepted unsafe syntax |
| Source / Provenance | Work items `001`, `005`, `006`, `012`, `015`, `020`, `032`, `038` (the rejected warning for members `last` may have taken), and `040` (exclusive access and a provably safe contract); Zax purpose, lifecycle, invocation safety, optional unsafe-access, composition provenance, and general panic-contract pressure |

Current general behavior has moved to
[Zax safety and analysis](../../language/safety-and-analysis.md). This file
retains the categories and comparative pressure that future safety work must
still disposition.

## Guarantee categories to investigate

- Type integrity
- Initialization
- Bounds
- Pointer validity
- Lifetime and use-after-free
- Aliasing
- Data-race boundaries
- Arithmetic behavior
- Panic behavior
- FFI and assembly trust
- Resource exhaustion
- Deadlock and starvation
- Logic and broader security correctness

The last categories may be explicit non-guarantees rather than properties the
compiler can prove.

## Construction-derived safety pressure

[Zax construction, replacement, and destruction](../../language/construction-and-destruction.md)
requires static lifecycle reasoning without mandatory runtime state flags.

Future safety work must decide which cases the language contract guarantees the
compiler will reject, including:

- use before member construction;
- use after member destruction;
- missing construction or destruction on one normal path;
- two operations affecting the same member lifetime;
- normal return with an incomplete enclosing instance;
- conflicting call-site and constructor-body member construction;
- unsafe manual lifecycle operations hidden behind opaque code;
- incomplete current-instance access; and
- publication or reentrant observation before construction completes.

Bounded helper access and publication are different:

```zax
+++ final : ()() = {
  _.first.+++()
  initializeRemaining(_)
}
```

```zax
+++ final : ()() = {
  registerGlobally(_)
}
```

The second call may retain an access path, notify subscribers, or reenter through
another path. A control that permits the first must not automatically permit the
second.

An unsafe assertion may override incomplete proof or assert the result of an
opaque operation. It cannot make known use after an ended lifetime valid, and it
cannot make any known violation valid. Detailed source-control and provenance
questions are preserved in
[raw analysis-control input](analysis-controls.md).

Zax has no declaration form that leaves storage indeterminate; the legacy
`unsafe ???` form no longer exists. Construction performed by an opaque
operation is asserted
at that operation with `unsafe<opaque-construction>{ ... }`, which also takes the
named member out of automatic construction. See
[construction by an opaque operation](../../language/construction-and-destruction.md#construction-by-an-opaque-operation).
When the compiler cannot tell whether a place is constructed, the code must be
restructured; there is no unsafe override for construction state.

## Invocation-derived safety pressure

[Zax function invocation](../../language/function-invocation.md) requires static
reasoning about:

- use before a result slot is constructed;
- incomplete or duplicate result completion;
- operations unavailable in moved-from or terminal source state;
- references returned from argument or result temporaries;
- source and destination result slots consumed more than once; and
- reentrant observation between completed earlier bindings and not-yet-bound
  later parameters.

Future safe-subset work must decide which cases the language guarantees are
rejected, which rely on a selected lifetime strategy, and which narrow unsafe
control can acknowledge an unproved but valid operation.

[Zax transfer stances](../../language/transfer-stances.md) and
[intent acknowledgements](../../language/intent-acknowledgements.md) now
distinguish:

- defined terminal-source reuse, which requires intent acknowledgement;
- an unavailable operation in terminal state, which remains invalid;
- and an unproved lifetime or alias claim, which may require unsafe
  responsibility.

Transfer implementation blame belongs at the declaration when analysis can
prove duplicate exclusive ownership, resource loss, double disposition,
incomplete destination state, or a source left unsafe to destroy. Failure to
visibly take a resource is not itself a violation; `copy`, `move`, and `last`
implementations may legally preserve more source state than their maximum
authority permits.

### Rejected: warnings for members `last` may have taken

A type's `last` operation may take a member's resource out, leaving the old
value fit only to be destroyed or replaced. Its destructor and replacement constructor
(`+++ replacement`) must then handle that state. The compiler can see which
members `last` transfers out, so it was proposed that it warn when a
destructor or replacement constructor uses such a member without checking:

```zax
+++ replacement final : ()(nextQuery : Query) = {
  _.connection.resetSession()   // the proposed warning: last may have taken connection
}
```

The type instead decides how it knows whether the resource is present, for
example with its own method or a member that has a visible empty state:

```zax
+++ replacement final : ()(nextQuery : Query) = {
  if _.connection.wasTaken() {    // the type's own check; no warning proposed
    _.connection .= PrivateConnection.open()
  } else {
    _.connection.resetSession()
  }
  _.query .= nextQuery
}
```

*Rejected because:* how a type knows whether a resource is present is part of
its design, and destructors already carry this responsibility without compiler
aid. Reopen only with new programmer pressure, such as recurring defects of
this kind that the type's own design cannot reasonably prevent.

## Composition-derived diagnostic pressure

Current [composition](../../language/composition.md) distinguishes three
immediate outer-provenance operations. Plain `outer cast` or outer-result `via`
requires selected-contract proof and produces a non-optional result.
`tracked outer cast` or `tracked via` performs the checked optional operation
through the `outer tracked` capability. `unsafe outer cast` or `unsafe via`
asserts unproved non-optional provenance. An `intent` acknowledgement cannot
grant unsafe authority.

Future safety work must formalize the initial site-specific exact-origin proof
candidate. At one cast site, every origin that can reach the operand must be the
exact resident member path named by the target. Same-typed sibling members,
arbitrary parameters, callbacks, raw-pointer or opaque ingress, and exported
standalone-member acceptance can prevent that proof. Unrelated external values
that cannot reach the site do not.

When the selected contract requires and establishes that proof, plain
`outer cast` needs neither `outer tracked` placement metadata nor a runtime
check and produces a non-optional result. Without that proof, it is rejected
rather than silently changing result shape or mechanism. The exact mandatory
proof classes and algorithms remain future specification and implementation
work.

Required source spelling must remain portable under the selected contract:

- a mainline contract-required proof requires the safe unmarked form and makes a
  redundant `unsafe` a hard error; deliberately retaining tracked optional
  semantics requires `intent<redundant-outer-tracking>`;
- without contract-required proof, plain `outer cast` is unavailable; checked
  optional behavior requires `tracked outer cast`, while the unchecked
  non-optional form still requires `unsafe` even when one compiler privately
  proves the claim;
- a stronger compiler or shared extension may make that proof canonical only
  when source explicitly selects the extension contract; and
- a proved-false plain or unsafe claim is invalid, while the checked tracked
  operation produces absence.

Private stronger analysis may optimize or advise, but cannot silently change
mainline source validity, require a redundant-tracking intent acknowledgement,
or change the optional result shape of a tracked operation. The same matrix
applies to outer-result provenance through ordinary, `tracked`, and
`unsafe via`.

The intent category acknowledges the selected-contract-proved redundancy and
the optional tracked contract; it grants no provenance and does not force a
physical metadata read. Future proof formalization must apply it at the portable
source contract boundary rather than making it compiler-cleverness-dependent.

Future safety and diagnostic work may decide whether deliberately choosing an
unsafe form when an available `tracked outer cast` could express the same
operation should require a dedicated intent acknowledgement or advisory
diagnostic. That future policy must not make intent authorize provenance,
change the tracked form's optional result, or turn a known-false relationship
into a valid unsafe claim.

Future callable preconditions and postconditions add proof pressure:

- a precondition may let a minted body assume pointer/optional presence, a
  numeric range, collection size, ownership state, or another predicate;
- every normal exit must prove each promised result postcondition;
- callers may rely on the selected visible prototype's guarantees;
- compatible visible prototypes must not promise facts the minted
  implementation cannot establish;
- opaque but valid proof may require a narrow unsafe assertion at the operation
  that hides the fact; and
- a known violation cannot be made valid through intent acknowledgement or
  unrelated unsafe permission.

Exact contract syntax and the portable minimum proof set remain future work.

## Panic boundary

An unresolved panic is fatal graceful crashing. A matching helper may resolve a
panic only by repairing the condition while the operation that encountered it
remains blocked. That same operation then completes as though the failure had not
occurred. Otherwise the program crashes.

Resolution does not skip the failed operation, supply an unrelated substitute
result, unwind completed members, roll back construction, continue after
incomplete destruction, or expose partial lifecycle state to ordinary code.

For example, an allocation-failure helper may obtain more backing storage, extend
the arena, and let the same blocked request succeed. It cannot turn a failed
`@` allocation into a vacant pointer; source selects `@!` when that result is
wanted.

Required-result fixed-width overflow likewise cannot resume with wrapping or a
substitute result. Programs select optional, wrapping, saturating, or reporting
operators when they need another local policy.

Future panic work must define helper discovery, eligibility, recursion, helper
failure, and the exact continuation point while preserving once-only operand
evaluation and the no-partial-continuation rule.

Future safety work should test a general default-check contract across other
runtime failure domains. Where a check is required by default, violation panics.
An explicit unchecked contract may permit omitting the check and make violation
undefined, provided the cost and responsibility remain visible and reproducible.

Panic categories are independently enabled or disabled rather than controlled
as one all-or-nothing mode. Allocation has no separate unchecked operator:
disabling the allocation-failure panic category applies the general promise to
ordinary `@`, while `@!` must retain its check to report vacancy.

This does not require runtime handles for static lifetime proof. A false unsafe
optional-presence, reference-lifetime, or alias assertion may have undefined
consequences with no required check. Debug modes may add detection and panic, but
that instrumentation is not a language guarantee.

A source-program panic during compile-time execution must become a compiler
diagnostic with source and evaluation-path context rather than an internal
compiler crash. This does not imply a compile-time panic handler.

Source-level panic-check controls are current design in
[safety and analysis](../../language/safety-and-analysis.md#controlling-panic-checks).

### Panic function pressure

The root legacy compiler-directive page keeps its panic-function section as
deferred input: a replaceable `___.panic(...)` function receiving an error code
and source location. The maintainer's direction for that future work:

- a custom panic function may only capture state, such as logs or crash
  reports; it cannot repair the condition or resume;
- when it returns, the program ends gracefully;
- an internal pre-panic capture always runs first, regardless of the installed
  function, so a corrupted function pointer still leaves a record; and
- the legacy runtime lookup table that let execution continue past a failed
  check without panicking is rejected: disabling a category removes its check
  instead.

## Nothing-instance safety pressure

[Zax Nothing instances](../../language/nothing-instances.md) establishes a
vacant pointer's special target and leaves several cross-feature safety
classifications for future work:

- a proved dereference of a vacant pointer is diagnosed, while an unproved
  postfix dereference performs no universal vacancy check;
- trapping policy uses target read protection when available, but an access
  relying on unavailable hardware trapping has undefined behavior rather than
  a universal generated software check;
- access to an unreadable compiler-prepared member panics;
- a write to compiler-provided Nothing backing is invalid, may be diagnosed
  statically, traps on targets with suitable write protection, may be noticed by
  optional instrumentation, and otherwise can corrupt shared backing or have
  undefined consequences;
- a custom Nothing instance may be writable, but mutation is ordinary shared
  state with programmer-owned synchronization;
- disabling an applicable pointer-arithmetic or access check accepts narrow
  unsafe responsibility; and
- unsafe cannot bypass managed-pointer disposition or legalize a guaranteed
  leak.

Future safety work must integrate these boundaries with independently
selectable panic categories without promising one fixed proof algorithm or a
check on every store. It must keep vacant pointers, absent optionals,
unavailable function values, uninitialized storage, and ended lifetimes
distinct in diagnostics.

It must also distinguish certainty from suspicion. A compiler that proves an
invalid Nothing write may reject it. When a pointer may flow into a later
writer but invalidity is not proved, any standard unsafe responsibility likely
belongs at that handoff or call boundary rather than on the eventual store.
The provisional unsafe catalog records `possible-nothing-write` and
`possible-nothing-read-trap`; the latter applies only to built-in trapping
Nothing or a defined type selecting `= trap`.
Whether stronger non-mandatory findings use standard compiler-dependent
analysis, only language-defined cases, or `x-` extension diagnostics remains
future safety/error design.

## Unchecked versus unsafe admission

Current identity admission behavior is owned by
[Zax identity types](../../language/identity-types.md#admission). This section
retains the future safe-subset classification.

Identity work exposes a safety distinction future work must preserve:

- `optional from` validates and returns absent on rejection;
- `unchecked from` skips semantic validation but has defined mechanical behavior;
- `unsafe from` bypasses an invariant on which memory, lifetime, representation,
  concurrency, or optimizer guarantees may rely.

An unchecked document handle may be meaningless without making construction
itself undefined. A forged capability or invalid representation may require the
stronger unsafe boundary. `restricted` identity means only that automatic
ordinary admission is absent; the identity owner decides which validated,
unchecked, unsafe, or unavailable paths exist.

Future safety work must define when an invalid semantic value remains a logic
error and when it can violate the safe subset. Existing enum and endian
`unsafe from` operations need reconsideration under that distinction rather than
automatic renaming.

## Unchecked required arithmetic contract

Current programmer-visible integer contract behavior is owned by
[the integer operator catalog](../../language/integer-operator-catalog.md#unchecked-build-contracts).
This section retains its future safe-subset and auditing classification.

An explicit build policy may replace required overflow checks with the contract
that every required arithmetic result is representable. Contract-respecting code
retains the exact result. A violation has no promised behavior and may be
optimized under the assumption that it never occurs.

Future safety work must decide whether code relying on that build contract is
inside a contract-qualified safe subset, how the assumption is audited, and
which non-representability panic conditions the policy covers. Defined modular
arithmetic remains the separate wrapping operator family.

## Exclusive access and a provably safe contract

### Two levels of safe code

- **No `unsafe` at all,** chosen at compile time. Such code is safe apart from
  logic errors, but only for the source compiled in that mode: the language's
  own library and compiler remain trusted, as Rust's
  `#![forbid(unsafe_code)]` covers one crate while its standard library uses
  `unsafe`.
- **`unsafe` allowed only in narrow, named, auditable places,** where a
  programmer asserts that code is actually safe. An `unsafe` assertion says a
  fact is true; it is a claim of safety made by a person instead of proved by
  the compiler.

### `exclusive`: a promise the compiler validates

The maintainer's preferred direction for exclusive write access is an opt-in
path meaning that while it exists, no other path may read or write the value,
checked by lifetime analysis like Rust's `&mut`. It is not a lock. Code that
does not use it keeps today's low friction.

```zax
// compiles in Zax today: two writable paths to one buffer
send final : ()(buffer : Buffer mutable writable loan &) = {
  notifySent()       // a callback resets the same buffer through another path
  buffer.read()      // the buffer changed underneath
}

// illustrative only: no exclusive form exists
send final : ()(buffer : Buffer mutable writable exclusive &) = {
  notifySent()       // error if the callback can reach buffer through another path
  buffer.read()
}
```

Related options considered:

- **Borrowing a writable view of an `immutable` value:** advised against.
  `immutable` promises the value never changes for its whole lifetime, and
  every reader may rely on it. The reverse already exists: an immutable value
  may be built from mutable storage over which the compiler proves exclusive
  authority.
- **A runtime lock wrapper,** like `Mutex` or `RefCell`: complementary, for
  threads and dynamic sharing. Its reentrancy failure should be a defined
  error, not silent corruption.

### Three reentrancy cases

1. **Mutation while access is held throughout.** An exclusive path prevents it.
2. **Lifetime around relocation.** Holding a view while the array relocates:

   ```zax
   values : Integer[0..] = [ 10, 20, 30 ]
   view := values[0..2]
   values.reserve(100)  // may relocate storage
   inspect(view)        // compile-time error when known; otherwise a checked use that can panic
   ```

   Zax answers this partly at runtime today. Rust answers it at compile time
   through exclusivity.
3. **Access released and taken again around a call-out.** No borrow rule can
   prevent it, because releasing is deliberate. Rust allows it too:

   ```rust
   fn send(state: &Rc<RefCell<Buffer>>, on_sent: impl FnOnce()) {
       { state.borrow_mut().prepare(); }   // take access, then release it
       on_sent();                          // the callback may reset the buffer
       state.borrow().read();              // compiles; the data is gone
   }
   ```

   It needs other tools: checking state again after calling out, not calling
   out while an invariant is broken, state machines that make an in-progress
   state explicit, or declared effects such as "this callback does not change
   the buffer" (see [purity and effects](purity-and-effects.md)).

### Exclusive access to array regions

Handing disjoint regions of one array to different threads, as Rust's
`split_at_mut` allows, needs the source array and overlapping views to be
unable to reach a region while an exclusive view of it exists. A slice has no
stance of its own, and `as` on a slice applies to its elements, so an
exclusive view would need a kind word after `[]`, by analogy with `* unique`,
and a separate way to hand itself over:

```zax
// illustrative: no exclusive views exist
window : MyMessage mutable writable loan[] unique = messages[2..<5]
other := messages[3..<6]      // nothing forbids this overlap today,
messages[3] = fresh()         // and the owner still reaches element 3
```

A plain `unique` slice is not enough: `unique` means one owner, not one path,
and a slice owns nothing.

### What a provably safe mode needs

- a switch that forbids `unsafe`;
- a contract that states what "safe" includes, where data-race freedom is
  currently missing, because Zax has no `Send`/`Sync` equivalent or
  exclusivity rule (immutability "is not a complete thread-safety
  guarantee");
- an explicit statement that defined panics count as safe outcomes, because
  several Zax checks happen at runtime where Rust's happen at compile time.

### The maintainer's position on complexity

Programmers need to know only some of Zax's features to program. Most stances,
qualifiers, and storage controls matter when building enduring libraries and
tuning performance. The test is that the common path needs few concepts and
the rest appear only when asked for. Some stance words are needed upfront
because there is no default, which is a deliberate honesty.

- **Reactivates:** threading work, which the maintainer intends to take up
  once core aspects of the language are settled.

## Comparative input

Rust is a comparison point, not an opponent.

Preserve investigation of:

- the value of safe Rust's memory-safety and data-race guarantees;
- borrow-checker restrictions and ownership-oriented redesign;
- `Arc`, `Mutex`, `RefCell`, state machines, and indirection as both real costs
  and possible cognitive overhead;
- unsafe implementations in standard libraries and dependency chains;
- FFI invariants;
- deadlock, starvation, panic, resource, logic, cryptographic, authorization,
  and business-rule non-guarantees;
- C++ mitigation through RAII, ownership conventions, smart pointers,
  sanitizers, static analysis, review, and tooling;
- AI-assisted analysis as broad but probabilistic support rather than a
  deterministic guarantee; and
- CHERI-style pointer capabilities and their limits around temporal lifetime,
  concurrency, resources, and logic;
- Rust preventing mutation while a borrow is active but not a stale assumption
  carried across a call-out, where its protection is that sharing is visible
  in the types;
- bypassing a type's process as an encapsulation problem rather than a
  mutability one;
- C++ safe patterns being unchecked while Rust checks its patterns;
- Swift enforcing exclusive access to variables statically where it can and at
  runtime otherwise, which is close to `exclusive` with runtime-inclusive
  proof, and Swift's costly late addition of data-race checking (`Sendable`),
  which argues for settling thread safety before the rest of the language
  hardens around its absence; and
- runtime-checked proof guaranteeing no undefined behavior but not no crash,
  so Zax should state which properties it proves at compile time and which it
  checks at runtime.

Future work must define Zax's promises first, then compare them carefully.

## Activation and retirement

Activate this residual input when a future contract assigns exact guarantees to
one of the unresolved categories above, formalizes domain diagnostics, or
performs the preserved language comparison. Move accepted details into
[Zax safety and analysis](../../language/safety-and-analysis.md) and applicable
domain owners, then retire this file.
