# 018: Switch, cases, and value selection

| Field | Value |
| --- | --- |
| Status | Active working material / non-normative |
| Work Item | `018` |
| Created | 2026-09-08 |
| Owns | The bounded review defined by the fixed initiating input |
| Does Not Own | Complete generalized pattern matching, arbitrary destructuring, compile-time selection, generics, reflection, variants and unions, or query/comprehension syntax |

## Non-authority notice

This file is a collaborative working record. Existing statements, new
observations, candidate wording, and aligned findings remain non-authoritative
until a separately discussed, aligned, and explicitly authorized promotion
incorporates them into their lasting owners.

## Fixed initiating input

This section records the information aligned when work item `018` was created.
It is intentionally incomplete and must not be rewritten as work develops.

### Initiating concern

Develop a coherent human-developer-facing model for runtime value selection
through `switch`, `case`, and `default` without prematurely designing a complete
generalized pattern-matching language.

The review should establish:

1. selector evaluation, initialization, binding scope, and body structure;
2. case ordering and comparison behavior;
3. first-match, multi-match, overlap, and unreachable-case behavior;
4. explicit fallthrough or the disposition of legacy `case continue`;
5. `break`, `next`, `continue`, labels, and transfer targets;
6. `default`, exhaustiveness, and incomplete selection;
7. runtime case expressions and user-defined comparison operations;
8. enum members, aliases, unnamed values, flags, and unsafe-admitted values;
9. bounded optional absence, presence, nested-layer, and payload-binding
   pressure;
10. scope, lifetime, cleanup, and result completeness;
11. costs, diagnostics, formatting, and source stability; and
12. lasting documentation ownership and legacy disposition.

### Motivating pressure

`switch`, `case`, and `default` remain a basic programmer-facing control-flow
hole after core flow and iteration promotion. Legacy flow material proposes
runtime cases, complex values, alternative comparison operators, header
sections, and `case continue`, but those forms have not been reconciled with
current flow, operator, enum, optional, and lifetime behavior.

Current core flow distinguishes `next` from `continue`. Selection must decide
whether the legacy fallthrough intent becomes `case next`, another explicit
form, or no fallthrough operation.

Enum declarations are not automatically exhaustive: strict enums may have
reachable unnamed values, relaxed enums admit all backing values, flags admit
unnamed combinations, aliases overlap, and `unsafe from` can establish any
backing representation. Selection must not infer complete coverage merely from
a list of declared member names.

### Known assumptions

- Conditions and guards that require truth produce exactly `Boolean`.
- A selected clause consumes one effective body statement and owns its body
  scope.
- Transfers unwind exited scopes in reverse construction order.
- `next` runs the target's post operation; `continue` skips it.
- Bare transfers do not silently skip a labeled eligible target.
- Comparison and operator selection use the current operator model.
- Optional layers retain independent presence and boxed lifetimes.
- No compiler implementation exists in this repository.

### Known inclusions

- `switch`, `case`, and `default` mental model and ordinary use.
- Selector evaluation count and visible scope.
- Runtime values and user-defined comparable values.
- Case ordering, overlap, reachability, and selection count.
- Alternative comparison operations when coherent with current operators.
- Fallthrough and the legacy `case continue` consequence.
- Labels, transfer targets, post behavior, and exits.
- Enum selection and non-exhaustive reachable values.
- Bounded optional-state and payload-binding pressure.
- Cleanup, construction completeness, costs, diagnostics, and formatting.
- Lasting ownership and complete legacy/raw disposition.

### Known starting boundaries

- Complete generalized pattern matching.
- Arbitrary destructuring or recursive pattern syntax.
- Compile-time `if`, compile-time dispatch, or compile-time execution.
- Generic constraints and generic specialization.
- General reflection or declaration metadata.
- Complete variants, unions, or algebraic data types.
- Query, comprehension, or functional-pipeline syntax.
- Compiler lowering or dispatch-table implementation.

These boundaries do not erase consequences. Record every material pressure in a
live owner or indexed raw destination.

### Intentionally unresolved framing

- Exact `switch`, `case`, and `default` source shape.
- Whether selection stops at the first match or may execute several cases.
- Whether fallthrough exists and how it is spelled.
- Whether a case is independently labelable or a transfer target.
- Whether `switch` itself accepts initializer and post sections.
- How case expressions select equality or another comparison operation.
- Whether overlap is legal, acknowledged, ordered, or rejected.
- What exhaustiveness means for open value domains and enum reachable values.
- Whether `default` is required, optional, or replaced by another catch-all.
- How optional presence and nested optional states are selected and bound.
- Whether selection is a statement only or later supports an expression form.
- Which pattern-like pressures belong here versus a later focused work item.

### Initial stopping guidance

Stop when the work has:

- established a usable runtime `switch`/`case`/`default` model;
- integrated selector evaluation, case ordering, body selection, labels,
  transfers, cleanup, and completion;
- dispositioned legacy fallthrough, alternative-comparison, complex-value, and
  header evidence;
- established enum and bounded optional selection behavior;
- preserved generalized pattern, variant, generic, reflection, and compile-time
  consequences without designing them prematurely;
- identified exact current owners and indexed deferrals; and
- completed the required documentation-fit dry run.

Do not design complete pattern matching, variants/unions, generics, reflection,
compile-time execution, promote findings, archive this work item, or begin work
item `019` without the separately required discussion, alignment, and
authorization.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs focused reading,
  numbered work, promotion, teaching, deferrals, and dry runs.
- [Raw selection and matching input](../raw/selection.md) - preserves the
  selection concern, `case next` pressure, flow constraints, enum coverage, and
  bounded optional matching requirements.
- Legacy [`switch`, `case`, and `default`](../../flow-control.md#switch) through
  focused [`switch` statement and `case continue`](../../flow-control.md#switch-statement-and-case-continue) -
  supply primary historical source evidence.
- Focused [effective bodies and header boundaries](../../language/core-flow-control.md#effective-bodies-and-header-boundaries),
  [header schemas, ordering, and binding scope](../../language/core-flow-control.md#header-schemas-ordering-and-binding-scope),
  [flow labels and transfer targets](../../language/core-flow-control.md#flow-labels-and-transfer-targets),
  and [unwinding, destruction, and completion](../../language/core-flow-control.md#unwinding-destruction-and-completion) -
  constrain clauses, headers, exits, and cleanup.
- Focused [viability, preference, and availability](../../language/operators.md#viability-preference-and-availability),
  [evaluation order](../../language/operators.md#evaluation-order), and
  [assignment and mutation boundaries](../../language/operators.md#assignment-and-mutation-boundaries) -
  constrain comparison selection, evaluation count, and case operations.
- Focused [enum comparisons and operation selection](../../language/enums.md#comparisons-and-operation-selection)
  and [selection pressure](../../language/enums.md#selection-pressure) -
  constrain enum cases, aliases, unnamed values, flags, and exhaustiveness.
- Focused [presence proof and postfix access](../../language/optional-values.md#presence-proof-and-postfix-access)
  and [nested optionals](../../language/optional-values.md#nested-optionals) -
  constrain absence/presence selection and payload lifetimes.
- Focused [header sections and separators](../../language/source-structure.md#header-sections-and-separators),
  [braces and body boundaries](../../language/source-structure.md#braces-and-body-boundaries),
  and [`else` attachment and layout](../../language/source-structure.md#else-attachment-and-layout) -
  constrain selection source, clauses, continuation, and bodies.

### Consequence-driven

- Read [raw variants and unions](../raw/variants-and-unions.md) only when a
  concrete selected-value shape requires variant coverage or payload binding.
- Read [raw function composition and chaining](../raw/function-composition-and-chaining.md)
  only when callback or combinator selection becomes material.
- Read focused [construction completeness](../../language/construction-and-destruction.md)
  only when case bindings or selected results create a concrete lifecycle
  dependency.
- Read [intent acknowledgements](../../language/intent-acknowledgements.md) when
  defined but suspicious overlap or ordering requires an acknowledgement
  category.
- Read raw compile-time, generic, reflection, or callable-selection input only
  when a concrete selection rule creates a dependency that must be preserved
  rather than solved.

### Audit-only

- `project/archive/`, including work items `001` through `017`.
- Deleted or superseded selection implementation sketches recoverable through
  Git history.

Do not read archived work item `017` during ordinary work on `018`. Its accepted
findings are promoted into current owners, and selection pressure is preserved
in live raw input.

## Working record

Not started. The agent creating this work item stops here without analyzing its
content.
