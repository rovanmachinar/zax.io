# Raw input: safety boundaries

| Field | Value |
| --- | --- |
| Status | Raw placeholder / non-authoritative |
| Audience | A future numbered work item defining safety guarantees and unsafe boundaries |
| Applies To | Safe-subset guarantees, unsafe mechanisms, and comparative safety input |
| Owns | Preservation of aligned pressures, the contract-qualified prove-or-narrowly-assert philosophy, and unresolved comparison material |
| Does Not Own | Accepted safety guarantees or unsafe syntax |
| Source / Provenance | Work items `001`, `005`, `006`, and `012`; Zax purpose, lifecycle, invocation safety, optional unsafe-access, and general panic-contract pressure |

## Aligned direction

Maximizing compiler-enforced safety is not Zax's organizing goal. Zax should
nevertheless define a safe subset with explicit guarantees.

Code using only safe operations and interfaces should receive the guarantees
assigned to that subset. Operations leaving those guarantees must be
recognizable and auditable without forcing the whole language into one
restrictive ownership model.

The final mechanism need not use an `unsafe { ... }` scope. Narrow operation
markers, unsafe declarations or interfaces, effect propagation, and reviewed
safe wrappers around unsafe implementations require evaluation.

## Prove or narrowly assert, qualified by contract

The generalized philosophy behind Zax's static analysis is:

> For every property mandatory under the selected language contract, Zax attempts
> to prove the operation valid. If proof cannot be established, the operation is
> rejected unless a narrow semantic assertion explicitly accepts responsibility
> for an unproved but valid boundary. An assertion cannot make a known violation
> valid.

Two consequences must survive into the future safety owner:

1. what must be proved is decided by the selected contract, not by how clever the
   executable compiler happens to be; and
2. an assertion is a claim about an unproved but valid boundary, never a way to
   permit a known violation.

The contract also decides what a now-redundant control means: advisory when the
contract does not require the case, potentially a hard error when the selected
contract requires the case to be recognized as safe, and nonconforming when a
compiler demands an assertion for a contract-mandated passing case. The detailed
matrix and provenance questions are preserved in
[raw analysis-control input](analysis-controls.md#contract-dependent-redundant-controls).

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

`unsafe ???` already acknowledges bypassed initialization. It can satisfy a
containing constructor through programmer responsibility and may later receive
explicit `+++` without another unsafe marker.

## Invocation-derived safety pressure

[Zax function invocation](../../language/function-invocation.md) requires static
reasoning about:

- use before a result slot is constructed;
- incomplete or duplicate result completion;
- use after a move or `last` transfer;
- references returned from argument or result temporaries;
- source and destination result slots consumed more than once; and
- reentrant observation between completed earlier bindings and not-yet-bound
  later parameters.

Future safe-subset work must decide which cases the language guarantees are
rejected, which rely on a selected lifetime strategy, and which narrow unsafe
control can acknowledge an unproved but valid operation.

## Panic boundary

Unresolved panic is currently fatal graceful crashing. If a panic resolves and
execution continues within construction or replacement, ordinary completion
obligations still apply.

Do not infer general exception-style rollback or recoverable partial-construction
unwinding. Reopen that design only for a concrete nonfatal panic use case.

Required-result fixed-width arithmetic supplies that concrete pressure. Bare
overflow panics and produces no result. A future handler cannot simply resume the
failed operator with wrapping unless a resumable-panic contract defines:

- how a replacement expression result is supplied;
- once-only operand evaluation;
- nested call and scope transfer;
- destruction and partially completed mutation;
- handler failure; and
- optimization boundaries.

Until then, overflow remains fatal graceful crashing. Programs select optional,
wrapping, saturating, or reporting operators when they need another local policy.

Future safety work should test a general default-check contract across other
runtime failure domains. Where a check is required by default, violation panics.
An explicit unchecked contract may permit omitting the check and make violation
undefined, provided the cost and responsibility remain visible and reproducible.

This does not require runtime handles for static lifetime proof. A false unsafe
optional-presence, reference-lifetime, or alias assertion may have undefined
consequences with no required check. Debug modes may add detection and panic, but
that instrumentation is not a language guarantee.

A source-program panic during compile-time execution must become a compiler
diagnostic with source and evaluation-path context rather than an internal
compiler crash. This does not imply a compile-time panic handler.

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
  concurrency, resources, and logic.

Future work must define Zax's promises first, then compare them carefully.

## Activation and retirement

Activate this input before publishing safety guarantees, defining unsafe
boundaries, or making construction lifecycle diagnostics mandatory. Consume its
findings through that work and retire or archive this placeholder afterward.
