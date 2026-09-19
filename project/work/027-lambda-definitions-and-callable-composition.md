# 027: Lambda definitions and callable composition

| Field | Value |
| --- | --- |
| Status | Active working material / non-normative / awaiting assignment |
| Work Item | `027` |
| Created | 2026-09-19 |
| Owns | The bounded review defined by the fixed initiating input |
| Does Not Own | Complete generics, variadics, async execution, foreign calling conventions, compiler implementation, callable ABI, or runtime representation except where a concrete lambda/callable decision constrains them |

## Non-authority notice

This file is a collaborative working record. Existing statements, new
observations, candidate wording, and aligned findings remain non-authoritative
until a separately discussed, aligned, and explicitly authorized promotion
incorporates them into their lasting owners.

## Fixed initiating input

This section records the information aligned when work item `027` was created.
It is intentionally incomplete and must not be rewritten as work develops.

### Initiating concern

Establish a coherent programmer-facing model for lambda definitions and callable
composition. Current design distinguishes callable prototypes, fixed and varying
function storage, bound/unbound/`once` receiver behavior, captures, minted
implementations, unavailable function values, and composition candidates without
one reviewed owner teaching how a programmer defines, stores, transfers,
composes, resets, and invokes a lambda value.

The maintainer-supplied
[lambda maintainer notes](../raw/lambda-maintainer-notes.md) are the latest
primary input. Read and disposition their complete contents when this work item
is assigned. They may add to, replace, supersede, or reject older material.

The indexed
[function composition and chaining input](../raw/function-composition-and-chaining.md)
preserves capture, receiver application, generated callable, optional callable,
composition, chaining, reflection, and cost pressure not yet given lasting
owners.

### Motivating pressure

Several current owners already constrain the feature:

- function values have visible prototypes, fixed or varying storage, and
  callable-selection behavior;
- `bound`, `unbound`, and `once bound` declarations distinguish receiver slots
  and receiverless routes;
- a default or reset function value is unavailable and must disposition any
  owned capture representation;
- ordinary capture currently defaults to copying the captured value, while
  explicit reference capture borrows one fixed place;
- transfer stance, qualification, lifetime, and source state remain independent;
- a compatible visible prototype reuses one minted implementation without
  silently changing its body;
- contiguous `[[...]]` is reserved for lambda capture rather than compiler
  directives; and
- legacy composition and chaining syntax remains evidence rather than accepted
  design.

Without a cohesive model, lambda syntax can accidentally decide callable
identity, capture lifetime, transfer, receiver binding, composition, reset
cost, or generated implementation behavior in incompatible local ways.

### Known inclusions

- Lambda-definition syntax and the boundary among a declaration, expression,
  body, prototype, and callable value.
- Noncapturing and capturing lambdas.
- Generated callable identity and compatibility with visible prototypes.
- Capture discovery, explicit capture lists, naming, evaluation order, and
  construction.
- By-value and by-reference capture, including qualifications, transfer stance,
  lifetime, mutation, and escape.
- Repeated invocation after `move`/`last` use of captured state.
- Binding or applying an instance receiver to a bound prototype.
- Receiverless generated callables versus original bound, unbound, and `once`
  declarations.
- Function composition, chaining, result-to-input mapping, partial argument
  capture, and candidate source forms such as legacy `>>` and `|>`.
- Copying, moving, terminally transferring, assigning, resetting, and destroying
  callable values and their capture state.
- Callable presence, unavailable state, and interaction with Nothing behavior.
- Recursion, self-reference, and mutually dependent callable values where
  concrete evidence supports them.
- Invocation, overload selection, result routing, optional callable, safety,
  reflection, cost, diagnostics, and source-stability consequences.
- Complete disposition of maintainer notes, indexed raw input, and reached
  legacy/current evidence.

### Known starting boundaries

- Complete generic lambda syntax, constraints, specialization, and type erasure.
- Complete variadic callable behavior.
- Complete async suspension, cancellation, executors, and coroutine state.
- Foreign calling conventions, raw function-pointer ABI, dynamic linking, and
  interoperability representation.
- General operator redesign unrelated to the selected composition/chaining
  surface.
- Compiler data structures, lowering, closure layout, and optimization except
  where programmer-visible cost or behavior constrains them.

These boundaries do not erase consequences. Preserve every material pressure in
a current owner or indexed raw destination rather than leaving it only in the
eventual archive of this work item.

### Intentionally unresolved framing

- Whether every lambda produces one anonymous concrete type or may directly use
  another callable identity.
- Which lambda forms require an explicit prototype.
- Exact capture-list syntax and omission behavior.
- Whether capture defaults differ among values, references, receiver slots, and
  callable composition.
- Whether receiver application, partial argument capture, function composition,
  and chaining use one operation family or several deliberately distinct forms.
- How a generated callable exposes labels, defaults, results, stance, receiver
  origin, and availability.
- Which operations are generated for callable copy, move, `last`, reset, and
  destruction.
- How recursive and repeatedly invoked captures preserve source state.
- Which facts are conceptual language guarantees and which remain future
  representation, ABI, reflection, or implementation choices.

### Initial stopping guidance

Stop when the work has:

- established a teachable lambda/callable mental model and terminology;
- defined representative noncapturing, value-capturing, reference-capturing,
  receiver-binding, composition, chaining, reset, and invocation behavior;
- reconciled declarations, prototypes, capture construction, lifetime,
  qualification, transfer, invocation, result mapping, unavailable state,
  operators, source structure, safety, diagnostics, and costs;
- separated accepted callable behavior from future generic, variadic, async,
  FFI, ABI, and implementation mechanics;
- dispositioned all useful refreshed and reached legacy/raw material; and
- completed the required documentation-fit dry run.

Do not promote findings, archive this work item, begin work item `028`, or
redesign complete generic, variadic, async, FFI, ABI, reflection, or
implementation behavior without the separately required discussion, alignment,
and authorization.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs focused reading,
  decision maturity, promotion, deferral, and closure.
- [Lambda maintainer notes](../raw/lambda-maintainer-notes.md) - latest
  maintainer-supplied primary input; read and disposition it completely after
  assignment.
- [Function composition and chaining](../raw/function-composition-and-chaining.md) -
  preserves unresolved capture, receiver application, composition, chaining,
  optional callable, and reflection pressure.
- [Declarations and bindings](../../language/declarations-and-bindings.md) -
  owns function storage, prototypes, binding, initialization, aliases,
  receiver categories, and declaration-facing capture constraints.
- [Function invocation](../../language/function-invocation.md) - owns callable
  contracts, receiver and argument binding, results, selection, minted
  implementations, and call completion.
- [Lifetimes and references](../../language/lifetimes-and-references.md) - owns
  current capture defaults, fixed reference binding, escape, and callable
  capture life paths.
- [Transfer stances](../../language/transfer-stances.md) - owns `copy`, `deep`,
  `move`, `last`, receiver stance, projection, and repeated-use source state.
- [Nothing instances](../../language/nothing-instances.md) - owns unavailable
  function values, presence, reset, capture disposition pressure, and
  receiverless `once` behavior.
- [Construction, replacement, and destruction](../../language/construction-and-destruction.md) -
  owns callable/capture construction, reset lifecycle, destruction, and
  exactly-once resource disposition.
- [Source structure](../../language/source-structure.md) - owns `[[...]]`
  capture delimiters, declaration/expression boundaries, and physical source
  presentation.
- [Operators](../../language/operators.md) and the
  [operator catalog](../../language/operator-catalog.md) - own recognized
  composition/chaining source forms, receiver discovery, selection, precedence,
  and reservation.
- [Language-design terms](../../language/terms.md) - owns cross-cutting callable,
  capture, receiver, lifetime, and source-state vocabulary.
- [Legacy functions](../../functions.md) - principal reached legacy callable,
  reassignment, `once`, capture, and composition evidence to disposition by
  value.

### Consequence-driven

- [Qualifiers](../../language/qualifiers.md) when capture mutability, access,
  replacement, or receiver qualification becomes concrete.
- [Safety and analysis](../../language/safety-and-analysis.md) and
  [raw analysis controls](../raw/analysis-controls.md) when capture escape,
  repeated destructive use, unavailable invocation, or unsafe callable
  assertions become concrete.
- [Raw callable selection](../raw/callable-selection.md) when generated callable
  preference, exact prototypes, pre/postconditions, receiver origins, or route
  preservation becomes concrete.
- [Optional values](../../language/optional-values.md) when optional callable
  construction, reset, nesting, or combinators become concrete.
- [Raw type parameters and generics](../raw/type-parameters-and-generics.md) when
  anonymous callable identity, generic lambda parameters, specialization, or
  type erasure becomes concrete.
- [Raw reflection](../raw/reflection.md) when callable identity, capture,
  generated declaration, source, or runtime metadata becomes concrete.
- [Raw asynchronous execution](../raw/async.md) when suspension would extend a
  capture, argument, result, receiver, or callable lifetime.
- Relevant legacy lambda, scope, operator, or variadic material only when a
  concrete question cannot be resolved from the primary notes, required
  callable input, current owners, and focused raw destinations.

### Audit-only

- Archived work only when a concrete provenance or regression question cannot
  be answered from current owners and live raw input.

## Initial working posture

Begin with programmer-visible examples of:

1. a noncapturing lambda;
2. a lambda capturing one value;
3. a lambda explicitly borrowing one reference;
4. a bound receiver converted into a receiverless callable;
5. two callables composed or chained; and
6. a varying callable reset to its unavailable state.

State what is constructed, owned, borrowed, callable, transferable, and
destroyed in each example before proposing one general lambda syntax or
composition operator family.

Use the maintainer notes as the latest primary evidence. Preserve current owner
constraints and treat legacy material as input rather than authority.

## Working record

Awaiting assignment. Creating and routing work item `027` does not authorize
analysis.
