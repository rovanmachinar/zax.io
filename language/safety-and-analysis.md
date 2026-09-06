# Zax safety and analysis

| Field | Value |
| --- | --- |
| Status | Current conceptual design |
| Audience | Human developers reasoning about Zax's safe subset, compiler proof, unsafe responsibility, and source portability |
| Applies To | Cross-cutting programmer-visible safety and static-analysis behavior; not a formal proof system or diagnostic specification |
| Implementation State | Not established by this repository |
| Owns | The safe-subset model; required proof; narrow unsafe assertions and permissions; known-invalid boundaries; distinction from intent and lints; language-contract versus compiler-analysis evolution; cross-cutting safety diagnostics and source stability |
| Does Not Own | Domain-specific validity rules; exact future unsafe syntax and category registry; intent acknowledgements ([intent acknowledgements](intent-acknowledgements.md)); or compiler implementation algorithms |
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
> operation is rejected unless a narrow unsafe assertion or permission provides
> a valid lower-level interpretation. Unsafe source cannot make a known
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

No unsafe marker is needed.

### Unproved but valid

The operation may be valid, but analysis lacks a required fact because code is
opaque, alias relationships are complex, or another recognized proof boundary
intervenes.

Illustrative future syntax:

```zax
/// The callback consumes the view synchronously and stores nothing.
unsafe<lifetime-escape>{
  invokeOpaqueCallback(view)
}
```

The assertion says that a fact is true. It does not change runtime behavior.

### Defined unsafe permission

Some operations have a coherent lower-level meaning but are deliberately
excluded from ordinary safe source:

```zax
// Illustrative category identifier; exact name remains unsettled.
/// The member place and successor representation remain suitable.
unsafe<replacement-member-successor>{
  inspect(memberView)
}
```

Here the enclosing instance was completely replaced, so the old member resident
instance ended. The permission deliberately accesses the completely established
successor through the continuing member place.

An unsafe permission differs from an assertion:

- an assertion supplies an unproved fact required by an otherwise safe
  operation;
- a permission deliberately crosses a defined safe-language boundary.

Both must identify the exact property and source region they affect.

### No valid interpretation

Unsafe source cannot manufacture:

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

There is no unsafe spelling that makes this ordinary access valid without first
establishing another instance through a real lifecycle operation.

The threshold for declaring a case impossible is intentionally high.
Conservative analysis belongs in the unproved or defined-unsafe categories
rather than being mislabeled as universal invalidity.

## Unsafe is narrow

Unsafe control is not a broad alternate language mode.

Every unsafe operation or enclosure must eventually:

- use the literal `unsafe`;
- identify the exact category;
- name or enclose the affected operation, binding, member, or expression;
- apply at the narrowest useful source location;
- permit attached documentation explaining the responsibility;
- alter only the stated proof or permission boundary; and
- require no runtime check unless the underlying selected operation independently
  specifies one.

One enclosure may identify several exact categories without granting unrelated
authority.

Exact syntax, category identifiers, nesting, reflection, and extension
provenance remain future analysis-control work.

## False unsafe claims

A false lifetime, alias, presence, construction, or transition assertion may
have undefined consequences. The language does not require a runtime check.

Compiler debugging modes may instrument unsafe claims and panic when misuse is
detected. Such instrumentation is a tooling feature, not a portable language
guarantee.

An assertion may fill a proof gap. It cannot contradict a fact the compiler
already knows:

```zax
value.---()

unsafe<lifetime>{
  inspect(value) // error: known-ended lifetime
}
```

## Intent, unsafe, and linting are different

| Mechanism | Programmer statement |
| --- | --- |
| `intent<...>{...}` | “This defined but suspicious interpretation is deliberate.” |
| Unsafe assertion | “This required fact is true although the compiler cannot prove it.” |
| Unsafe permission | “I deliberately accept this defined lower-level boundary.” |
| Lint suppression | “Do not report this advisory diagnostic here.” |

### Intent does not accept safety responsibility

```zax
intent<terminal-source-reuse>{
  resource.releaseRemainingHandle()
}
```

This confirms defined terminal-source behavior. It does not grant lifetime,
ownership, mutation, or alias authority.

An anchored pointer may require intent because replacement will make it observe
a successor member resident instance. The operation is defined; the
acknowledgement confirms that surprising meaning.

### Lints do not change semantics

A reference may safely observe a value changed through another authorized path.
A lint can ask whether that observation was intended without making the source
unsafe.

Lint suppression:

- changes no language behavior;
- creates no proof;
- grants no permission;
- suppresses no required semantic error; and
- should not require `unsafe`.

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
- [transfer stances](transfer-stances.md) defines moved-from and terminal source
  states; and
- integer owners define overflow, narrowing, and required-result behavior.

An unsafe category does not become a duplicate definition of the domain rule it
qualifies.

## Representative lifecycle boundaries

### Partial construction

A constructor helper may need access to already-live members before the complete
enclosing instance exists. That bounded access differs from publishing the
incomplete instance through a global, callback, or stored reference.

Unsafe controls must distinguish:

- bounded partial-instance access;
- escape or external publication;
- asserting that opaque work established a member;
- suppressing a generated lifecycle operation; and
- claiming path completeness or at-most-once behavior.

### Replacement and aliases

A replacement right-hand operand may alias the destination or one of its
members. Analysis must account for whether the operation:

- copies required values before ending them;
- consumes an alias before renewal;
- delegates exact self-alias handling to the selected operation;
- preserves only storage or resources rather than an old member lifetime; or
- deliberately crosses a defined unsafe successor boundary.

### Returned references

A returned reference must target a place whose life path outlives every use.
Returning a local or temporary is known invalid when it escapes. An opaque but
valid origin may need an unsafe assertion or future callable-origin contract.

### Raw pointers

A non-`Nothing` raw pointer does not prove:

- live pointee storage;
- a resident instance;
- valid provenance;
- alignment;
- permitted aliasing; or
- ownership.

A raw-pointer operation is safe when analysis proves every fact required by the
selected contract. A structurally bounded pointer whose allocation,
construction, use, destruction, and recovery remain inside one containing life
path may be fully provable.

`?rawPointer` proves only non-`Nothing`. Existing origin analysis may prove the
remaining facts, or an escape, opaque operation, owner release, reset,
relocation, or arena teardown may invalidate that proof.

Raw-pointer operations require explicit unsafe responsibility only where the
safe contract requires a fact the compiler cannot prove. A known-invalid
pointee remains invalid inside unsafe source.

## Runtime failure and unsafe failure

Defined runtime failure is not the same as unsafe undefined behavior.

Examples of defined failure include:

- a failed shared-to-unique ownership claim producing an empty unique pointer;
- a failed weak acquisition producing an empty strong pointer;
- a non-panicking allocation producing a pointer to `Nothing`;
- a panicking allocation failing its arena request;
- a checked operation returning absence; and
- a required runtime check panicking.

A false unsafe assertion may instead have undefined consequences with no
required check.

Future contracts should state which failure model applies rather than leaving it
to compiler accident.

## Language contracts and compiler analysis

Source may eventually select a language-contract version through source,
project, build, or toolchain configuration. Omitting an explicit version accepts
the active environment's contract; it does not claim compatibility with every
past or future contract.

The selected contract defines portable required analysis.

### A compiler may prove more

Suppose source contains an unsafe assertion because an older compiler cannot
prove a valid member-lifetime relationship.

A newer compiler may prove it under the same language contract. The assertion is
then redundant for that implementation, but it is at most advisory when the
contract did not require every compiler to recognize the case.

Removing it may make the source depend on the newer compiler's stronger
analysis.

### A later contract may require more

A later language contract may require conforming compilers to recognize that
case as safe.

Under that explicitly selected contract:

- the obsolete assertion may be a hard error;
- every conforming compiler must accept the unmarked valid operation; and
- a compiler that still demands the assertion is nonconforming.

This rule applies to every mandatory analysis, not only lifetime or optional
presence.

### Analysis provenance

Future controls need visible provenance:

- language-contract categories;
- compiler or vendor experimental categories; and
- community extensions implemented by multiple tools.

An unknown unsafe extension can be ignored only when the compiler independently
proves the source valid without it. Otherwise the compiler must report that the
required semantic extension is unsupported.

## Panic boundary

Current Zax design has no general exception-style rollback model.

If a panic is fatal, no recoverable partial-construction continuation exists. If
a future panic handler resumes execution during construction, replacement, or
destruction, it must define:

- how incomplete state becomes complete again;
- which result is supplied for a failed expression;
- destruction and cleanup of partial work;
- once-only operand evaluation;
- handler failure; and
- the point at which ordinary access resumes.

Do not infer those behaviors from the existence of a panic.

## Diagnostics

Safety diagnostics should identify:

- the selected language contract;
- the exact required property;
- the source operation and affected place, value, pointer, or result;
- the available proof and the missing or contradictory fact;
- whether an assertion or permission can apply;
- the consequence of a false claim;
- and any runtime check or failure that the selected operation guarantees.

Diagnostics should distinguish:

- unproved valid operation;
- defined unsafe permission;
- known-invalid operation;
- defined but suspicious intent;
- advisory lint;
- unsupported experimental analysis; and
- compiler nonconformance with a selected contract.

Do not report a vague “unsafe” or “lifetime” error when the actual issue is an
ended member path, missing ownership anchor, possible callback escape, unknown
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

Unsafe assertions can remove required runtime checks where the contract permits
that model, but they transfer correctness responsibility to the programmer.
They are not optimization hints detached from semantics.

## Source stability

Portable source validity follows the selected language contract.

Compiler improvements must not silently:

- reinterpret references or pointers;
- choose another transfer stance;
- add ownership;
- remove required synchronization;
- turn an unsafe permission into ordinary safe meaning;
- or change defined runtime failure into undefined behavior.

A new contract may deliberately strengthen mandatory proof or reject obsolete
controls. Such changes must be explicit and reproducible.

## Boundaries and maturity

This document establishes current conceptual safety direction. It is not a
formal safe-subset specification, proof calculus, diagnostic registry, or
compiler conformance suite.

Still deferred:

- exact unsafe syntax and category names;
- mandatory proof sets for each contract version;
- formal analysis provenance and extension rules;
- debug instrumentation;
- formal panic behavior;
- data-race guarantees;
- FFI and assembly trust boundaries;
- resource exhaustion, deadlock, and starvation;
- and broader security correctness.
