# 007: Core flow control, clause headers, and exits

| Field | Value |
| --- | --- |
| Status | Active working material / non-normative |
| Work Item | `007` |
| Created | 2026-08-24 |
| Owns | The bounded review defined by the fixed initiating input |
| Does Not Own | Accepted complete iteration, selection, pattern matching, error propagation, lifetime, async, formal grammar, or compiler behavior |

## Non-authority notice

This file is a collaborative working record. Existing statements, new
observations, candidate wording, and aligned findings remain non-authoritative
until a separately discussed, aligned, and explicitly authorized promotion
incorporates them into their lasting owner documents.

## Fixed initiating input

This section records the information known and aligned when work item `007` was
created. It is intentionally incomplete rather than exhaustive and must not be
rewritten as the work develops.

### Initiating concern

Develop a coherent human-developer-facing model for core synchronous flow
control:

1. what a condition must produce and when it is evaluated;
2. how `if`, `else`, and related clauses select bodies;
3. how flow-header declarations, conditions, and post operations compose;
4. how pre-test, post-test, and unconditional loops execute;
5. how `break`, `continue`, and `return` choose targets and exit scopes;
6. how bindings, destruction, construction obligations, and result obligations
   behave on every path; and
7. how source structure, formatting, diagnostics, and visible cost make that
   behavior predictable.

Use current statement/body, declaration, invocation/result, qualifier, and
lifecycle owners as constraints. Establish the common clause, scope, ordering,
and exit model before expanding into iteration protocols, large selection
constructs, async flow, or specialized error propagation.

### Motivating pressure

Current owners now establish:

- statement-level newlines, effective bodies, semicolon composition, and
  mandatory indentation intent;
- declarations in flow headers and their clause-visible scope;
- strict call evaluation, result slots, complete return obligations, and
  resultless calls;
- qualifier-aware access and mutation;
- construction/destruction obligations and incomplete-lifetime tracking; and
- reusable principles for clear intent, trustworthy order, defensible
  preference, visible cost, and explicit shape changes.

The remaining root flow-control material proposes `if`, loops, header
initializers, `;;`, ternary expressions, `break`, `continue`, `return`, `switch`,
`each`, `using`, and value-polymorphic dispatch without one reviewed common
model.

Core flow control is now the earliest missing foundation for reading and writing
ordinary Zax programs. It should be reviewed before advanced function captures,
iteration protocols, pattern-like selection, error short-circuiting, or async
flow depend on unclear scope and exit behavior.

### Known assumptions

- [Zax language vision](../../language/vision.md) is accepted foundational
  direction.
- [Zax language principles](../../language/principles.md) owns reusable
  cross-cutting design and use principles.
- [Zax source structure](../../language/source-structure.md) owns effective
  statements, bodies, semicolon composition, continuation, and mandatory layout.
- [Zax declarations and bindings](../../language/declarations-and-bindings.md)
  owns declaration forms, visibility, shadowing, and the existing
  flow-initializer binding boundary.
- [Zax function invocation](../../language/function-invocation.md) owns call
  evaluation, result slots, return completion, multiple-result capture, and
  synchronous call completion.
- [Zax qualifiers](../../language/qualifiers.md) owns place, value, access,
  receiver, and indirection qualifications.
- [Zax construction, replacement, and destruction](../../language/construction-and-destruction.md)
  owns value/member lifetime transitions and normal completion obligations.
- A flow-control clause consumes one effective body statement, which may be
  simple, composed, or braced.
- A flow-control clause establishes a scope around its complete body; braces do
  not make body-local names escape.
- A binding introduced by a flow initializer is visible to the condition,
  applicable clause bodies, and corresponding false or `else` clauses, then
  leaves scope after the complete flow statement.
- Resultful functions must complete every result on every normal exit. A bare
  return or fallthrough is valid only when all result slots are already complete.
- No source-order, indentation, or implementation guess may silently choose
  among plausible flow meanings.
- Exact operator declaration/ranking, casting, ownership, async, generic,
  iteration-protocol, and pattern-matching behavior remains later work unless an
  immediate core-flow contradiction requires a constraint.
- There is no language implementation in this repository.

### Known inclusions

- The condition contract for `if` and core loops.
- Exact versus converted `Boolean` conditions at the depth needed for coherent
  flow without designing the complete conversion system.
- Condition side effects and evaluation order.
- Short-circuit `&&`, `||`, and negation behavior at the depth required by
  conditions.
- `if`, `else`, and `else if`.
- Simple, composed, and braced clause bodies.
- Flow-header initialization, condition, and post-operation roles.
- The `;;` separator and its relationship to statement composition.
- Scope and visibility across conditions, true/false clauses, and loops.
- Pre-test `while` and `until`.
- Post-test `redo while` and `redo until`.
- Unconditional `forever`.
- Loop iteration order.
- `break` and `continue`, including target selection and whether labeled or named
  forms are needed.
- `return` as a flow exit, including complete result obligations.
- Normal fallthrough and early-exit behavior.
- Destruction and cleanup of applicable local scopes on normal completion,
  `break`, `continue`, and `return`.
- Construction and result completeness across branches and loop paths.
- Reachability and nontermination at the depth needed for programmer-facing
  diagnostics.
- Ternary/conditional expression behavior where it depends on the shared
  condition and branch model.
- Immediate callback and reentrancy consequences when condition or header
  evaluation invokes code.
- Programmer-visible ordering, costs, diagnostics, and canonical formatting.
- Source-stability pressure from changed conditions, added clauses, or changed
  exit targets.
- Candidate lasting owners and documentation boundaries.
- Preservation of useful but premature iteration, switch/pattern, error-flow,
  lifetime, analysis, generic, compile-time, or async findings in existing or
  newly justified indexed raw input.

### Known starting boundaries

- Complete `each` iteration protocols, ranges, arrays, metadata iteration, and
  iterator customization.
- Complete `switch`, `case`, pattern matching, exhaustiveness, and fallthrough.
- Complete `using` resource-management semantics.
- Runtime value-polymorphic function declarations using `if`.
- `except`, `catch`, and specialized error-result propagation.
- Function capture and closure representation.
- Complete pointer/reference lifetime strategies and ownership.
- Async suspension, cancellation, executors, and concurrent flow.
- Generics, concepts, specialization, and compile-time branch elimination.
- Arbitrary operator declaration, conversion, and ranking.
- Compiler directives controlling likelihood, reachability, or analysis.
- Formal definite-initialization, reachability, or termination algorithms.
- Foreign control transfer, ABI, stack unwinding, and compiler implementation.

These areas are not part of the initiating question and need not be explored to
begin. They do not prohibit following, recording, or dispositioning material
consequences.

### Intentionally unresolved framing

- Whether conditions require exactly `Boolean`, allow an explicit selected
  conversion, or use another narrowly defined truth-test contract.
- Whether logical short-circuit operators are intrinsic flow mechanisms,
  ordinary operators with protected ordering, or another explicit category.
- Exact syntax and meaning of `;;` in initializer/condition/post headers.
- Which core statements support initialization and post operations.
- Whether `until` remains a distinct statement or an inversion of `while`.
- Exact pre-test and post-test loop spelling.
- Whether `break` and `continue` target only the nearest loop or may name an
  enclosing loop or scope.
- Whether named scopes participate in core exit targeting.
- How loop-header and iteration-local bindings are scoped and destroyed.
- What cleanup is required before each exit transfers control.
- Whether a condition or body that cannot complete normally changes result,
  construction, or reachability obligations.
- How ternary/conditional expressions produce one value or result shape.
- Which findings belong in a core flow owner versus source structure,
  declarations, invocation, construction, lifetime, diagnostics, operators, or
  future iteration/selection owners.

### Initial stopping guidance

Stop when the work has:

- established one coherent condition model;
- established shared clause evaluation, body, and scope behavior;
- established core branch and loop forms;
- established header initialization, condition, post-operation, and `;;`
  behavior;
- established `break`, `continue`, `return`, fallthrough, and target selection;
- established programmer-visible cleanup, construction, and result obligations
  for every core exit;
- constrained short-circuit operators, conversions, callbacks, lifetimes,
  analysis, and nontermination without designing their complete future domains;
- identified costs, diagnostics, canonical formatting, and source-stability
  pressure;
- dispositioned useful legacy flow and scope material;
- preserved future iteration, selection, error-flow, async, and analysis
  findings in appropriate current or indexed raw owners;
- identified plausible lasting owners and their boundaries; and
- gathered enough information for a pre-promotion documentation fit dry run.

Do not promote findings, update language examples broadly, archive this work
item, or begin work item `008` without the separately required discussion,
alignment, and authorization.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs ownership,
  focused reading, numbered work, and promotion.
- [Language vision](../../language/vision.md) - provides foundational direction
  and accepted high-level tradeoffs.
- [Zax language principles](../../language/principles.md) - provides the
  cross-cutting principles against which flow behavior should be evaluated.
- [Zax source structure](../../language/source-structure.md) - provides
  effective statements, bodies, composition, continuation, layout, and comments.
- [Zax declarations and bindings](../../language/declarations-and-bindings.md) -
  provides flow-header declarations, clause-visible scope, shadowing, and
  binding visibility.
- [Zax function invocation](../../language/function-invocation.md) - provides
  call evaluation, result slots, return completion, result capture, and
  synchronous completion.
- [Zax qualifiers](../../language/qualifiers.md) - provides mutation, access,
  place, receiver, and indirection constraints used within flow.
- [Zax construction, replacement, and destruction](../../language/construction-and-destruction.md) -
  provides lifecycle completeness, destruction, and incomplete-state pressure
  across control paths.
- [Zax language-design terms](../../language/terms.md) - provides current
  cross-cutting vocabulary.
- [Legacy flow control](../../flow-control.md) - preserves primary evidence for
  conditions, branch and loop forms, header initialization, `;;`, ternary,
  iteration, selection, exits, `using`, and value-polymorphic proposals.
- [Legacy scope](../../scope.md) - preserves primary evidence for named and
  unnamed scopes, targeted `break`/`continue`, and scope capture proposals.

### Consequence-driven

- Read focused [operator](../../operator.md) material when condition conversion,
  logical short-circuiting, negation, or operator/function consistency becomes
  necessary.
- Read focused [casting](../../casting.md) material when a non-`Boolean`
  condition requires an explicit conversion boundary.
- Read focused [Nothing](../../nothing.md) or [optional](../../optional.md)
  material when condition truth, absence, or optional state becomes necessary.
- Read focused [pointer and reference](../../pointers.md) material and the
  [raw lifetime-strategies input](../raw/lifetimes.md) when scope exit,
  destruction, dangling references, captures, or transfer becomes necessary.
- Read the [raw analysis-controls input](../raw/analysis-controls.md) when flow
  requires proof overrides, reachability assertions, definite completion, or
  diagnostic provenance.
- Read the [raw safety input](../raw/safety.md) when flow behavior crosses a
  promised safe-subset boundary.
- Read focused [except](../../except.md) material when ordinary exits cannot be
  established without constraining error-result short-circuiting.
- Read focused [arrays](../../arrays.md), [variadic](../../variadic.md), or
  remaining `each` material only when core loop behavior cannot be established
  without an iteration-protocol constraint.
- Read focused `switch`, `case`, or `using` sections from
  [legacy flow control](../../flow-control.md) only when their future design
  constrains the shared clause or exit model.
- Read the [raw async input](../raw/async.md) when flow completion, suspension,
  cancellation, or callback timing becomes materially async.
- Read focused [compiler-directive](../../compiler-directives.md) or
  [meta-function](../../meta-functions.md) material when runtime flow cannot be
  separated cleanly from build-time selection or likelihood directives.
- Expand beyond this list whenever discussion or a material finding establishes
  a concrete need. State why the additional reading is relevant and keep it
  focused.

Material not listed in this reading scope is neither required nor prohibited. It
remains available when discussion or a concrete consequence makes it relevant.

### Audit-only

- `project/archive/`, including work items `001` through `006`.

Do not read archived work item `006` while conducting ordinary work on `007`.
Its applicable findings have been promoted into current language and project
owners. Read it only when the language maintainer explicitly requests a targeted
audit or provenance investigation.

Other audit-only material likewise requires direct language-maintainer
instruction or a specific audit or recovery need.

## Working record

Not started. The agent creating this work item stops here without analyzing its
content.
