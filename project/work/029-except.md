# 029: Except result-flow control

| Field | Value |
| --- | --- |
| Status | Active working material / non-normative / awaiting assignment |
| Work Item | `029` |
| Created | 2026-09-21 |
| Owns | The bounded review defined by the fixed initiating input |
| Does Not Own | C++-style exceptions, catch/throw semantics, stack unwinding, panic recovery, or compiler implementation except where distinguishing them constrains programmer-visible `except` behavior |

## Non-authority notice

This file is a collaborative working record. Existing statements, new
observations, candidate wording, and aligned findings remain non-authoritative
until a separately discussed, aligned, and explicitly authorized promotion
incorporates them into their lasting owners.

## Fixed initiating input

This section records the aligned information known when work item `029` was
created. It is intentionally incomplete and must not be rewritten as work
develops.

### Initiating concern

Recover and refine Zax `except` as flow control related to function results.
These controls intentionally route execution through an exceptional result path.
They are not exceptions, stack unwinding, C++-style `throw`/`catch`, or an
alternate panic-recovery mechanism.

The maintainer-supplied
[except notes](../raw/except-maintainer-notes.md) are the latest primary input
for this work item. They may add to, replace, supersede, or reject conflicting
legacy evidence.

Legacy [`except`](../../except.md) contains useful ideas to mine and disposition
by value. It is not authoritative over the refreshed notes merely because it is
older, more detailed, or already checked in. The refreshed model is the
preferred direction unless concrete semantic, cross-feature, teachability, or
feasibility pressure exposes a flaw. Legacy precedence by itself is not such a
flaw.

Current `language/` owners remain current for the concerns they already own.
Any conflict between the refreshed notes and a current owner requires explicit
review.

### Motivating pressure

The work must give programmers one teachable model for:

- which function results can establish an exceptional result path;
- how a caller recognizes, routes, binds, forwards, handles, or deliberately
  ignores that path;
- how ordinary success and exceptional result flow relate to named and several
  results;
- which scopes, posts, disposal operations, and destructors run;
- how result transfer stance, ownership, reference origin, and lifetime remain
  valid;
- how `except` differs from `return`, panic, optional absence, enum/variant
  selection, and ordinary conditional control flow;
- how source exposes exceptional flow and its costs without implying exception
  stack machinery; and
- how adding or changing exceptional results affects source compatibility.

### Known starting boundaries

- C++-style exceptions, `throw`, `catch`, and exception-object hierarchies.
- Stack unwinding as an exception mechanism.
- General panic recovery or resumable panic handling.
- Compiler lowering, hidden control structures, runtime implementation, and ABI
  except where programmer-visible behavior constrains them.
- General algebraic effects, coroutines, async cancellation, or generalized
  pattern matching beyond concrete `except` pressure.

These boundaries do not erase consequences. Preserve every material pressure in
a current owner or indexed raw destination rather than leaving it only in the
eventual archive of this work item.

### Intentionally unresolved framing

- Exact `except` declaration and use-site syntax.
- How a function marks one or several results as exceptional.
- Whether exceptional routing is expression-level, statement-level,
  result-mapping behavior, callable-prototype behavior, or a combination.
- Binding, forwarding, acknowledgement, and completeness rules.
- Interaction with multiple results, defaulted results, optional values, enums,
  variants, references, ownership, and transfer stances.
- Exact scope-exit, post-operation, `using` disposal, and destruction ordering.
- Static diagnostics, source-compatibility policy, and any intent or safety
  categories.
- Lasting documentation ownership and terminology.

### Initial stopping guidance

Stop when the work has:

- established a cold-reader-friendly programmer model with representative
  success, exceptional-result, forwarding, handling, and cleanup examples;
- reconciled result declaration, invocation, routing, transfer, lifetime,
  scope-exit, disposal, panic, safety, and source behavior;
- clearly distinguished exceptional result flow from exception unwinding and
  catch/throw systems;
- dispositioned every useful maintainer-note and reached legacy finding;
- preserved all material deferrals in live owners; and
- completed the required documentation-fit dry run.

Do not promote findings, archive this work item, begin work item `030`, or
design C++-style exception machinery without the separately required
discussion, alignment, and authorization.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs focused reading,
  decision maturity, promotion, deferral, and closure.
- [Except maintainer notes](../raw/except-maintainer-notes.md) - latest
  maintainer-supplied primary input; read and disposition completely after
  assignment.
- [Legacy except](../../except.md) - useful historical result-flow evidence to
  mine by value rather than authority.
- [Function invocation](../../language/function-invocation.md) - owns callable
  results, result slots, result routing, mapping order, completion, and visible
  prototypes.
- [Declarations and bindings](../../language/declarations-and-bindings.md) -
  owns result declaration contexts, labels, initialization, and binding scope.
- [Core flow control](../../language/core-flow-control.md) - owns effective
  bodies, `return`, transfers, posts, labels, and scope exit.
- [Construction, replacement, and destruction](../../language/construction-and-destruction.md) -
  owns result/member completeness, destruction order, abrupt scope exit, and the
  no-unwinding lifecycle boundary.
- [Transfer stances](../../language/transfer-stances.md) - owns result and
  source `copy`/`deep`/`move`/`last` behavior.
- [Lifetimes and references](../../language/lifetimes-and-references.md) - owns
  result/reference origin, call completion, temporary lifetime, and escape.
- [Safety and analysis](../../language/safety-and-analysis.md) - owns the panic
  boundary, proof, unsafe responsibility, and source-stability model.
- [Source structure](../../language/source-structure.md) - owns source
  continuation, bodies, mapping presentation, and contextual recognition.
- [Language-design terms](../../language/terms.md) - owns cross-cutting result,
  completion, panic, lifetime, and flow vocabulary.

### Consequence-driven

- [Zax `using`](../../language/using.md) when exceptional flow crosses resource
  enrollment, disposal, or disposal bypass.
- [Optional values](../../language/optional-values.md),
  [enums](../../language/enums.md), [variants](../../language/variants.md), and
  [switch](../../language/switch.md) when exceptional results use or resemble
  those state/selection mechanisms.
- [Qualifiers](../../language/qualifiers.md) when routing changes result,
  destination, or reference authority.
- [Intent acknowledgements](../../language/intent-acknowledgements.md) and
  [raw analysis controls](../raw/analysis-controls.md) when suspicious omission,
  unchecked routing, or failure controls become concrete.
- [Asynchronous execution](../raw/async.md) only when suspension, cancellation,
  or async result propagation creates a material constraint.
- Other legacy error, result, function, flow, or cleanup material only when a
  concrete question cannot be resolved from the refreshed notes, current
  owners, and `except.md`.

### Audit-only

- Archived work only when a concrete provenance or regression question cannot
  be answered from current owners and live raw input.

## Initial working posture

Begin with concrete source for:

1. a function producing an ordinary success result and one exceptional result;
2. a caller handling both paths locally;
3. a caller forwarding the exceptional path;
4. several results where only one path is exceptional;
5. a reference- or ownership-bearing exceptional result; and
6. cleanup showing exactly which locals, posts, disposal operations, and
   destructors run.

For each example, state which result slots are constructed, which body or route
executes, what transfers, what is destroyed, and why no exception stack
unwinding or catch search occurs.

Use the refreshed maintainer notes as the preferred primary model. Preserve
current owner constraints and treat legacy material as evidence rather than
authority.

## Working record

Awaiting assignment. Creating and routing work item `029` does not authorize
analysis. The maintainer will complete the primary notes before handoff.
