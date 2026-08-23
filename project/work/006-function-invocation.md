# 006: Function invocation, argument binding, and results

| Field | Value |
| --- | --- |
| Status | Active working material / non-normative |
| Work Item | `006` |
| Created | 2026-08-23 |
| Owns | The bounded review defined by the fixed initiating input |
| Does Not Own | Accepted complete function, ownership, lifetime, generic, async, formal grammar, or compiler behavior |

## Non-authority notice

This file is a collaborative working record. Existing statements, new
observations, candidate wording, and aligned findings remain non-authoritative
until a separately discussed, aligned, and explicitly authorized promotion
incorporates them into their lasting owner documents.

## Fixed initiating input

This section records the information known and aligned when work item `006` was
created. It is intentionally incomplete rather than exhaustive and must not be
rewritten as the work develops.

### Initiating concern

Develop a coherent human-developer-facing model for ordinary function
invocation:

1. how arguments map to parameters;
2. when expressions evaluate and parameters bind;
3. how defaults and omitted arguments work;
4. how zero, one, or multiple results map into surrounding expressions and
   declarations;
5. how overload candidates become viable, preferred, ambiguous, or unavailable;
   and
6. how temporaries and immediate reference, copy, move, or `last` effects cross
   the call boundary.

Use construction as precedent where its argument behavior is general without
forcing ordinary calls to use construction-packet or stored-member syntax.

### Motivating pressure

Work item `005` established strict left-to-right construction-packet evaluation,
immediate binding according to parameter semantics, named and positional cursor
behavior, declaration expressions as inputs, temporary-lifetime requirements,
resultful and resultless replacement, and demand-driven operation selection.

Those findings deliberately defer their general callable form. Existing
declaration and qualifier owners also establish function-value declarations,
parameter/result declaration contexts, recursive binding boundaries, qualified
arguments, receiver operands, and operator selection without yet owning complete
ordinary invocation behavior.

Reviewing function invocation next tests whether those boundaries form one
predictable call model before later pointer/lifetime, unsafe-analysis, async,
generic, variadic, or complete function-value work depends on it.

### Known assumptions

- The current [language vision](../../language/vision.md) is accepted
  foundational direction.
- [Zax source structure](../../language/source-structure.md) is accepted
  conceptual design.
- [Zax declarations and bindings](../../language/declarations-and-bindings.md)
  is accepted conceptual design.
- [Zax qualifiers](../../language/qualifiers.md) is accepted conceptual design.
- [Zax construction, replacement, and destruction](../../language/construction-and-destruction.md)
  is accepted conceptual design.
- Parameters and results are declaration contexts using ordinary name, type,
  initialization, and qualifier concepts.
- Function values use ordinary value declarations; an ordinary function body is
  delayed code rather than executable declaration initialization.
- Instance members are accessed explicitly through `_`; member-function
  invocation and overload selection use a receiver operand.
- Construction establishes strict left-to-right argument evaluation and
  immediate copy, move, `last`, reference, pointer, or other input binding.
  Ordinary invocation should preserve that behavior unless a concrete
  contradiction requires explicit revision.
- Construction's stored-member packet entries are construction-specific and do
  not become ordinary function arguments.
- Generated and declared callable behavior must not be selected through
  source-order guessing when no candidate clearly wins.
- Exact move/copy, `last`, pointer lifetime, ownership, generic, variadic,
  capture, async, and concurrency behavior remains later work unless an
  immediate invocation contradiction requires a constraint.
- There is no language implementation in this repository.

### Known inclusions

- Ordinary function-call syntax and expression boundaries.
- Zero, one, and multiple input arguments.
- Positional and named argument syntax.
- The positional cursor after named arguments.
- Parameter names as source-facing call-contract elements.
- Declarations used as argument expressions.
- Strict argument evaluation and immediate parameter binding.
- References, pointers, copy, move, and `last` at the immediate call boundary.
- Parameter initialization and complete call-entry state.
- Default parameter expressions and omitted arguments.
- Zero, one, and multiple results.
- Result capture, mapping, omission, and discard.
- Resultless calls used as statements or nested expressions.
- Return completion at the depth needed to make result mapping coherent.
- Function overload viability, preference, ambiguity, and unavailability.
- Expected-result context in candidate selection.
- Calling function values versus resolving overload groups.
- Member-function calls and immediate receiver-operand qualification.
- Temporary lifetime through call completion.
- Later overload changes and source-stability pressure.
- Immediate callback and reentrancy consequences when needed to define call
  completion.
- Programmer-visible costs, diagnostics, and canonical formatting.
- Candidate lasting owners and documentation boundaries.
- Preservation of useful but premature pointer, ownership, generic, variadic,
  async, analysis, or safety proposals as indexed raw input.

### Known starting boundaries

- Complete closure and capture representation.
- Function reassignment and recursive function values beyond immediate
  invocation pressure.
- Complete move/copy/`last` ownership and moved-from states.
- Complete pointer/reference lifetime guarantees and provenance.
- Generics, concepts, and specialization.
- Variadic functions and argument splitting or combining.
- Async calls, coroutines, cancellation, executors, and scheduling.
- Arbitrary operator declaration and ranking beyond shared callable pressure.
- Complete callback-effect and reentrancy analysis.
- Foreign calling conventions and ABI.
- Formal grammar, compiler data structures, lowering, or implementation.

These areas are not part of the initiating question and need not be explored to
begin. They do not prohibit following, recording, or dispositioning material
consequences.

### Intentionally unresolved framing

- Exact ordinary-call syntax for positional and named arguments.
- Whether ordinary calls use the construction positional-cursor model without
  change.
- Whether positional arguments may follow named arguments in every callable
  category.
- How declarations nested inside arguments are formatted and scoped.
- When parameter default expressions evaluate relative to explicit arguments.
- Whether omitted parameters use the same marker or mechanism as explicitly
  requested type-default initialization.
- Whether parameter names are always part of the public source contract.
- How expected-result context affects overload viability and ranking.
- How zero, one, and multiple results participate in expression typing.
- How result capture, omission, discard, and declaration interact.
- Whether overload groups and function-valued expressions use one invocation
  surface.
- Exact temporary destruction points after copy, move, reference, or `last`
  binding.
- How a later overload or parameter-name change affects source compatibility.
- Which findings belong in current function documentation versus future
  lifetime, ownership, generic, variadic, operator, async, safety, or diagnostic
  owners.

### Initial stopping guidance

Stop when the work has:

- established a coherent ordinary invocation syntax and programmer model;
- established positional, named, defaulted, and omitted argument mapping;
- established strict evaluation and immediate binding behavior;
- established coherent zero, one, and multiple result production and capture;
- established immediate overload viability, preference, ambiguity, and
  unavailable-candidate behavior;
- constrained copy, move, `last`, references, pointers, temporaries, receiver
  operands, callbacks, and result context without designing their complete
  future domains;
- identified programmer-visible costs, diagnostics, and formatting;
- dispositioned the indexed function-invocation input;
- preserved useful deferred function, lifetime, analysis, generic, variadic, or
  async material in appropriate indexed input;
- identified plausible lasting owners and their boundaries; and
- gathered enough information for a pre-promotion documentation fit dry run.

Do not promote findings, update language examples broadly, archive this work
item, or begin work item `007` without the separately required discussion,
alignment, and authorization.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs ownership,
  focused reading, numbered work, and promotion.
- [Language vision](../../language/vision.md) - provides explicit-cost,
  progressive-complexity, readability, and source-stability principles.
- [Zax source structure](../../language/source-structure.md) - provides
  expression, statement, body, semicolon-composition, and scope boundaries.
- [Zax declarations and bindings](../../language/declarations-and-bindings.md) -
  provides function-value declarations, parameters/results as declaration
  contexts, recursive bindings, name visibility, and result-capture boundaries.
- [Zax qualifiers](../../language/qualifiers.md) - provides parameter, result,
  capture, receiver-operand, and access qualification.
- [Zax construction, replacement, and destruction](../../language/construction-and-destruction.md) -
  provides the accepted evaluation, binding, cursor, temporary, generated
  operation, and result precedents that ordinary invocation must test.
- [Zax language-design terms](../../language/terms.md) - provides current
  cross-cutting vocabulary.
- [Raw function-invocation input](../raw/function-invocation.md) - preserves the
  construction-derived constraints and examples intended for this work.
- [Legacy functions](../../functions.md) - preserves primary evidence for
  parameters, results, calls, overloads, defaults, member functions, transfers,
  and argument-composition proposals.

### Consequence-driven

- Read focused [operator](../../operator.md) material when callable ranking,
  expected-result selection, or operator/function consistency becomes necessary.
- Read focused [pointer and reference](../../pointers.md) material and the
  [raw lifetime-strategies input](../raw/lifetimes.md) when reference origin,
  temporary extension, move/copy/`last`, or ownership becomes necessary.
- Read the [raw analysis-controls input](../raw/analysis-controls.md) when
  invocation requires compiler proof, unsafe overrides, or diagnostic
  provenance.
- Read the [raw safety input](../raw/safety.md) when invocation behavior crosses
  a promised safe-subset boundary.
- Read focused [variadic](../../variadic.md) material when fixed-arity invocation
  cannot be established without a variadic constraint.
- Read focused [flow-control](../../flow-control.md) or [Nothing](../../nothing.md)
  material when result mapping or resultless calls cross those boundaries.
- Read the [raw async input](../raw/async.md) when call completion, suspension,
  cancellation, or callback behavior becomes materially async.
- Expand beyond this list whenever discussion or a material finding establishes
  a concrete need. State why the additional reading is relevant and keep it
  focused.

Material not listed in this reading scope is neither required nor prohibited. It
remains available when discussion or a concrete consequence makes it relevant.

### Audit-only

- `project/archive/`, including work items `001` through `005`.

Do not read work item `005` while conducting ordinary work on `006`. Its
applicable findings have been promoted into
`language/construction-and-destruction.md` and preserved in
`project/raw/function-invocation.md`; reading the archived work would add
historical discussion rather than current authority.

Read work item `005` only when the language maintainer explicitly requests a
targeted audit or provenance investigation. Other audit-only material likewise
requires direct language-maintainer instruction or a specific audit or recovery
need.

## Working record

Not started. The agent creating this work item stops here without analyzing its
content.
