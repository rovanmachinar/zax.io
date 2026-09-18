# 026: Nothing values, states, and instances

| Field | Value |
| --- | --- |
| Status | Active working material / non-normative / awaiting assignment |
| Work Item | `026` |
| Created | 2026-09-18 |
| Owns | The bounded review defined by the fixed initiating input |
| Does Not Own | Complete optional APIs, pointer ownership, general function invocation, global/`once` lifecycle ordering, variants/unions, compiler implementation, ABI, or runtime representation except where a concrete `Nothing` decision constrains them |

## Non-authority notice

This file is a collaborative working record. Existing statements, new
observations, candidate wording, and aligned findings remain non-authoritative
until a separately discussed, aligned, and explicitly authorized promotion
incorporates them into their lasting owners.

## Fixed initiating input

This section records the information aligned when work item `026` was created.
It is intentionally incomplete and must not be rewritten as work develops.

### Initiating concern

Establish a coherent programmer-facing model for what Zax means by `Nothing`.
Current and legacy material use the word around pointer vacancy, optional
absence, default function values, missing current instances during type calls,
real or custom instances, generated checks, and representation sentinels without
one reviewed conceptual owner establishing which uses are one concept and which
must remain distinct.

The maintainer-supplied
[Nothing maintainer notes](../raw/nothing-maintainer-notes.md) are the latest
primary input. Read and disposition their complete contents when this work item
is assigned. They may add to, replace, supersede, or reject older material.

The indexed [Nothing instances input](../raw/nothing-instances.md) preserves
global/custom instance, pointer, monitor, representation, and generated-check
pressure not yet given a lasting owner.

### Motivating pressure

Several current owners already rely on behavior called `Nothing`:

- a pointer can contain a vacant state;
- a default function value has a non-callable state;
- a type-qualified call to a `once` function supplies no receiver instance;
- an optional can be absent without constructing its boxed type;
- allocation reporting can produce pointer vacancy;
- source and diagnostics must distinguish absence from uninitialized storage,
  an ended lifetime, a missing result, and an unsupported operation; and
- future custom/global `Nothing` behavior may affect identity, storage,
  construction, lifetime, safety, and reflection.

Without a cohesive model, documentation can reuse one word while implying
different identity, value, lifetime, ownership, representation, or failure
semantics.

### Known starting evidence

- Optional absence is wrapper state and does not silently construct a boxed
  value.
- `unsafe ???` means initialization was bypassed; it is not an ordinary
  `Nothing` state.
- A reference always has a live referent and cannot contain pointer-style
  vacancy.
- Pointer presence, optional presence, and function callability currently have
  distinct domain contracts even where documentation uses similar language.
- A type-qualified `once` call has no receiver instance available through `_`;
  that fact must not manufacture an ordinary instance.
- Source, declaration, import, or analysis order does not silently choose among
  several possible meanings.

These are constraints supplied by current owners, not a conclusion that one
`Nothing` type or representation already exists.

### Known inclusions

- Whether `Nothing` names a type, value, state, instance, family, sentinel, or
  several deliberately related concepts.
- Pointer vacancy and its relationship to pointer identity, representation,
  presence, dereference, reset, ownership, and allocation failure.
- Optional absence and whether it shares any value/identity relationship with
  pointer or function vacancy.
- Default function values, callability checks, and panic behavior.
- The no-receiver state used by type-qualified `once` functions and
  type-receiver operations.
- Construction, default initialization, copying, assignment, destruction, and
  lifetime of any real `Nothing` value or instance.
- Global, language-provided, type-provided, or programmer-defined `Nothing`
  instances where evidence supports them.
- Qualification, transfer stance, reference/pointer behavior, structural
  compatibility, and identity boundaries.
- Presence/truth operations, overload selection, custom operations, and
  diagnostics.
- Storage size, valid representations, sentinel/tag behavior, source
  reflection, and runtime reflection pressure at programmer depth.
- Clear distinction from uninitialized storage, ended lifetimes, zero values,
  empty collections, absent results, and operations returning no results.
- Complete disposition of maintainer notes, indexed raw input, and reached
  legacy/current evidence.

### Known starting boundaries

- Redesigning complete optional construction, nesting, and transfer behavior.
- Redesigning pointer ownership families, allocation policy, or arena
  mechanics.
- Complete callable selection, function capture, async behavior, or invocation
  routing.
- Complete global/module/`once` initialization, concurrency, and teardown.
- General variants, unions, pattern matching, result/error families, or
  exception-like control flow.
- General user-defined generics, compile-time execution, and reflection APIs.
- Compiler data structures, pointer tagging algorithms, ABI, or lowering.

These boundaries do not erase consequences. Preserve every material pressure in
a current owner or indexed raw destination rather than leaving it only in the
eventual archive of this work item.

### Intentionally unresolved framing

- Whether one canonical `Nothing` identity exists.
- Whether vacancy is a value of the pointer/function type or a relationship to
  a separate `Nothing` value.
- Whether optional absence contains, references, or merely resembles
  `Nothing`.
- Whether `_` during a type call denotes a real language-provided instance, a
  statically recognized unavailable receiver, or another mechanism.
- Whether custom types may define their own `Nothing`, and what declaration or
  generated-operation authority that requires.
- Whether any `Nothing` instance has storage, address, lifetime, destructor,
  qualifications, or transfer behavior.
- Whether distinct vacancy domains share source spelling, presence operations,
  diagnostics, or representation.
- Which facts are conceptual language guarantees and which remain future
  representation/ABI choices.

### Initial stopping guidance

Stop when the work has:

- established a teachable common model and terminology;
- separated concepts that only share legacy wording;
- defined pointer, optional, function-value, and no-receiver consequences at
  programmer depth;
- defined any accepted real/custom/global `Nothing` value or instance behavior;
- distinguished `Nothing` from uninitialized, ended, zero, empty, and
  no-result states;
- reconciled source, construction, qualifier, transfer, lifetime, operator,
  invocation, safety, storage, diagnostics, and reflection pressure;
- assigned global/`once`, representation, variant, generic, async, and
  implementation consequences to current or indexed future owners;
- dispositioned all useful refreshed and reached legacy material; and
- completed the required documentation-fit dry run.

Do not promote findings, archive this work item, begin work item `027`, or
redesign complete optional, pointer, function, global-lifetime, variant,
generic, reflection, async, or implementation behavior without the separately
required discussion, alignment, and authorization.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs focused reading,
  decision maturity, promotion, deferral, and closure.
- [Nothing maintainer notes](../raw/nothing-maintainer-notes.md) - latest
  maintainer-supplied primary input; read and disposition it completely after
  assignment.
- [Raw Nothing instances](../raw/nothing-instances.md) - preserves unresolved
  global/custom instance, pointer, monitor, representation, and generated-check
  pressure.
- [Legacy Nothing](../../nothing.md) - principal legacy language evidence to
  disposition by value.
- [Optional values](../../language/optional-values.md) - owns current absence,
  presence, boxed access, construction, reset, and transfer behavior.
- [Pointers, allocation, and arenas](../../language/pointers-and-arenas.md) -
  owns current pointer vacancy, presence, ownership, allocation failure, reset,
  and disposition.
- [Declarations and bindings](../../language/declarations-and-bindings.md) -
  owns default function values, type-callable `once` declarations, default
  initialization, and uninitialized storage distinctions.
- [Function invocation](../../language/function-invocation.md) - owns calling a
  vacant function value and type/instance `once` calls.
- [Construction, replacement, and destruction](../../language/construction-and-destruction.md) -
  owns lifecycle states and the distinction between storage, construction,
  absence, and destruction.
- [Lifetimes and references](../../language/lifetimes-and-references.md) - owns
  live instance/place requirements and why references cannot contain vacancy.
- [Operators](../../language/operators.md) and the
  [operator catalog](../../language/operator-catalog.md) - own presence,
  fallback, reset, call, and recognized source-form pressure.
- [Language-design terms](../../language/terms.md) - owns existing
  cross-cutting vocabulary to revise or extend.

### Consequence-driven

- [Identity types](../../language/identity-types.md) when a canonical or custom
  `Nothing` identity, admission, projection, or representation relationship
  becomes concrete.
- [Qualifiers](../../language/qualifiers.md) and
  [transfer stances](../../language/transfer-stances.md) when any real
  `Nothing` value, pointer/function vacancy, or no-receiver state receives
  qualification or transfer behavior.
- [Safety and analysis](../../language/safety-and-analysis.md) and
  [raw analysis controls](../raw/analysis-controls.md) when vacancy checks,
  disabled panic categories, unsafe assertions, or impossible states become
  concrete.
- [Execution context](../../language/execution-context.md) and
  [raw global and once lifetimes](../raw/global-and-once-lifetimes.md) when a
  language-provided/global `Nothing` instance or shared initialization becomes
  concrete.
- [Raw variants and unions](../raw/variants-and-unions.md) when `Nothing`
  becomes an active alternative, tag, or sentinel whose relationship to
  general variants must be preserved.
- [Raw reflection](../raw/reflection.md) when type/value/state metadata or source
  reflection becomes concrete.
- Relevant legacy pointer, function, type, allocation, or compiler-directive
  material only when a concrete question cannot be resolved from the primary
  notes, required legacy page, current owners, and focused raw input.

### Audit-only

- Archived work only when a concrete provenance or regression question cannot
  be answered from current owners and live raw input.

## Initial working posture

Begin with four programmer-visible examples: a vacant pointer, an absent
optional, a default function value, and a type-qualified `once` call. State what
the programmer may observe and do in each case before proposing that they share
one type, value, representation, or instance.

Use the maintainer notes as the latest primary evidence. Preserve current owner
constraints and treat legacy material as input rather than authority.

## Working record

Awaiting assignment. Creating and routing work item `026` does not authorize
analysis.
