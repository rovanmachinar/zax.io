# 008: Operator catalog, declarations, and expression resolution

| Field | Value |
| --- | --- |
| Status | Active working material / non-normative |
| Work Item | `008` |
| Created | 2026-08-26 |
| Owns | The bounded review defined by the fixed initiating input |
| Does Not Own | Accepted complete literal, word, conversion, generic, metaprogramming, type-identity, formal grammar, or compiler behavior |

## Non-authority notice

This file is a collaborative working record. Existing statements, new
observations, candidate wording, and aligned findings remain non-authoritative
until a separately discussed, aligned, and explicitly authorized promotion
incorporates them into their lasting owner documents.

## Fixed initiating input

This section records the information known and aligned when work item `008` was
created. It is intentionally incomplete rather than exhaustive and must not be
rewritten as the work develops.

### Initiating concern

Develop a coherent human-developer-facing model for the operator system needed
to read and write ordinary Zax expressions:

1. the catalog of recognized operator spellings and operator families;
2. fixity, arity, overloadability, and protected built-in signatures;
3. declaration forms for ordinary pre-unary, post-unary, and binary operators;
4. global and type-defined declarations and receiver operands;
5. operand evaluation order and once-only evaluation;
6. lookup domains, candidate discovery, viability, preference, ambiguity, and
   expected-result interaction;
7. precedence, associativity, grouping, and expression-tree formation;
8. arithmetic overflow-policy families, including wrapping/modulating,
   saturating, and checked operations;
9. compound and mutating operators and their relationship to value operations,
   assignment, and reconstructive replacement;
10. explicit fallback or generated relationships among related operators;
11. costs, failures, diagnostics, and source-stability consequences; and
12. lasting documentation owners and boundaries.

Use current declaration, qualifier, invocation, construction, source-structure,
core-flow, and bounded operator owners as constraints. Recover Zax-specific
intent before borrowing operator behavior from other languages.

### Motivating pressure

Current conceptual owners now establish:

- visible declaration and assignment boundaries;
- receiver, value, access, and indirection qualifications;
- shared callable viability, expected-result limits, and partial-order
  preference;
- assignment versus reconstructive replacement;
- exact source continuation, token spacing, bodies, and layout intent;
- branch-specific callable selection and expression convergence;
- exact-`Boolean` conditions;
- direct and ambiguous `?`/`!` selection, Boolean-only opposite-operator
  fallback, ordinary eager `&&`/`||` overloads, and protected Boolean
  short-circuit operations; and
- required language support for protected primitive operations.

The remaining operator material proposes a large flat token list, global and
type-defined declaration forms, special literals and word operators, and
operator-specific examples without one reviewed catalog, expression-binding
model, or derivation policy.

New aligned initiating input adds explicit overflow-policy families:

```zax
result := a +% b // wrapping or modulating addition
result := a +| b // saturating addition
result := a +! b // checked addition
```

The same review must determine whether corresponding subtraction,
multiplication, division, unary, increment/decrement, shift, and compound forms
exist and where the pattern deliberately stops.

The operator system is an early dependency for selection and matching, optional
and error behavior, numeric and structural types, arrays and iteration, casting,
generics, and metaprogramming. Reviewing it now avoids each future domain
inventing local token, precedence, or fallback rules.

### Known assumptions

- [Zax language vision](../../language/vision.md) is accepted foundational
  direction.
- [Zax language principles](../../language/principles.md) owns reusable
  cross-cutting design and use principles.
- [Zax source structure](../../language/source-structure.md) owns statement
  boundaries, continuation, whitespace intent, contextual keywords, and layout
  validation.
- [Zax declarations and bindings](../../language/declarations-and-bindings.md)
  owns declaration forms, name visibility, assignment boundaries, and the
  ordinary identifier namespace.
- [Zax qualifiers](../../language/qualifiers.md) owns place, value, access,
  receiver, and indirection qualifications.
- [Zax function invocation](../../language/function-invocation.md) owns shared
  callable viability, source-order input evaluation, expected-result limits,
  preference, ambiguity, and branch-specific callable selection.
- [Zax construction, replacement, and destruction](../../language/construction-and-destruction.md)
  owns assignment-versus-replacement lifetime behavior and hidden-cost
  constraints.
- [Zax core flow control](../../language/core-flow-control.md) owns exact-Boolean
  conditions, conditional-expression selected-arm order, and convergence.
- [Zax operators](../../language/operators.md) is the bounded current operator
  owner and must grow cohesively rather than be replaced by another competing
  owner.
- Operator result types may be arbitrary unless a specific protected operation
  constrains them.
- Direct operator selection, ambiguity, and opposite-operator fallback retain
  their current meaning.
- Exact `Boolean && Boolean` and `Boolean || Boolean` are language-provided,
  protected, left-to-right short-circuit operations that user declarations
  cannot replace.
- Other valid `&&`/`||` operand shapes are ordinary eager overloads and may
  return arbitrary types.
- Optimization may preserve established operations but may not silently select
  a different user-defined operator.
- No source-order, import-order, layout, or implementation guess may resolve an
  otherwise ambiguous operator meaning.
- There is no language implementation in this repository.

### Known inclusions

- Inventory every operator spelling found in current and legacy material.
- Classify every spelling as current, candidate, deferred, rejected, reserved,
  protected, or superseded.
- Decide whether Zax has a fixed recognized symbolic catalog, permits arbitrary
  symbolic declaration, or uses another bounded rule.
- For each reviewed operator, record token spelling, fixity, arity,
  overloadability, protected signatures, operand evaluation, result constraints,
  precedence, associativity, related compound forms, costs, failures,
  diagnostics, and source-stability effects.
- Pre-unary, post-unary, and binary declaration forms.
- Global and type-defined declarations and receiver operands.
- Lookup domains and candidate discovery at the depth required for ordinary
  symbolic operators.
- Reuse of function-invocation viability, preference, expected-result selection,
  ambiguity, and unavailable-best behavior.
- Parentheses, precedence, associativity, chaining, and expression-tree
  formation for the accepted catalog.
- Arithmetic baseline behavior for ordinary `+`, `-`, `*`, `/`, and `%`.
- Wrapping/modulating arithmetic candidates such as `+%`, `-%`, and `*%`.
- Saturating arithmetic candidates such as `+|`, `-|`, and `*|`.
- Checked arithmetic candidates such as `+!`, `-!`, `*!`, and possibly `/!`.
- Applicability of overflow policies to signed, unsigned, fixed-width,
  pointer-sized, floating, fixed-point, compile-time, and user-defined numeric
  types.
- Overflow, underflow, division-by-zero, invalid-shift, and constant-expression
  behavior at the depth required by the catalog.
- Compound forms such as `+=`, `+%=`, `+|=`, and `+!=`, where justified.
- Increment/decrement relationships to arithmetic or compound operations.
- Assignment and compound mutation versus reconstructive replacement.
- Explicit derivation or fallback candidates, including:
  - `!=` from Boolean `==` plus negation;
  - relational operators from `<=>`;
  - pre/post increment from compound arithmetic;
  - compound mutation from a value operation plus assignment/replacement;
  - overflow-policy compound forms from their value forms;
  - swapped-operand candidates; and
  - already accepted `?`/`!` Boolean fallback.
- The default presumption that distinct spellings are independent operations
  unless an explicit reviewed rule establishes a derivation.
- The question whether any derivation is implicit, explicitly generated, opt-in,
  or prohibited.
- Once-only place and operand evaluation across any generated relation.
- Hidden copy, temporary, allocation, destruction, mutation, conversion, alias,
  and result-shape costs.
- Tokenization, longest-token behavior, and whitespace/source-stability pressure
  from spellings such as `+!`, `+|`, `+%`, and their compound forms.
- Protected language-provided operations and diagnostics for conflicting user
  declarations.
- Programmer-visible costs, failures, diagnostics, formatting, and source
  stability.
- Candidate lasting owners and documentation boundaries.
- Preservation of useful but premature literal, word, conversion, type,
  generic, metaprogramming, and generated-operator findings in existing or newly
  justified live raw input.

### Known starting boundaries

- Complete literal parsing and user-defined literal operators.
- Complete word and compound-word operator semantics.
- Complete `as`, `unsafe as`, conversion, and casting design.
- Compiler-generated operator families beyond derivation questions necessary to
  evaluate the ordinary symbolic catalog.
- Generics, concepts, specialization, and generic operator constraints.
- Complete type identity, structural compatibility, layout, and conversion.
- Complete optional, error, selection, iteration, and pattern semantics.
- Complete allocator, pipeline, invocation, indexing, and function-composition
  operator domains when ordinary catalog coherence does not require them.
- Arbitrary compile-time operator generation and metaprogramming.
- Formal grammar, parser implementation, lowering, ABI, and toolchain internals.

These areas are not part of the initiating question and need not be designed to
begin. They do not prohibit following, recording, or dispositioning material
consequences.

### Intentionally unresolved framing

- The authoritative operator catalog and whether arbitrary symbolic operators
  are permitted.
- The exact meaning of ordinary arithmetic overflow once explicit policy
  variants exist.
- Whether `%`, `|`, and `!` are the accepted wrapping, saturating, and checked
  policy suffixes.
- Which base operations and built-in type families support each policy.
- Whether saturating operations clamp to both representable extrema.
- Whether checked operations panic, return a flag/error/optional result, use
  another result shape, or vary by operator/type.
- The exact spelling and availability of compound policy forms.
- Whether `a += b` and `a = a + b` are always independent operations.
- Whether any compound operation can be derived from a value operation plus
  assignment/replacement without hidden semantic change.
- Whether a non-mutating value operation can ever be derived from a mutating
  compound operation.
- Which comparison or logical operations have safe derivations.
- Which derivations are automatic, explicit, opt-in generated, or prohibited.
- Exact global, member, imported, and future lookup domains.
- Exact precedence and associativity for every accepted operator.
- Longest-token and whitespace rules needed to prevent future tokens from
  reinterpreting existing source.
- Which exact built-in signatures are protected and how their availability is
  taught without promising implementation mapping.
- Which findings belong in the current operators owner versus source structure,
  declarations, qualifiers, invocation, construction, vision, type, casting,
  diagnostics, or future raw owners.

### Initial stopping guidance

Stop when the work has:

- dispositioned every known operator spelling and family;
- established one organized operator catalog rather than a flat legacy list;
- established ordinary symbolic declaration forms, fixity, arity, and receiver
  behavior;
- established lookup, candidate discovery, viability, preference, ambiguity,
  and expected-result interaction;
- established precedence, associativity, grouping, and operand evaluation for
  the accepted catalog;
- established ordinary arithmetic overflow behavior and the wrapping,
  saturating, and checked policy families;
- established compound/mutating operator behavior and its boundary with
  assignment and reconstructive replacement;
- produced an explicit derivation/fallback matrix with once-only evaluation,
  hidden-cost, qualification, alias, and result-shape constraints;
- established protected built-in signatures and conflicting-declaration
  diagnostics;
- constrained literal, word, conversion, generated, generic, type, selection,
  iteration, optional, error, and metaprogramming consequences without
  prematurely designing those complete domains;
- identified costs, failures, diagnostics, formatting, and source-stability
  pressure;
- dispositioned useful legacy and newly supplied operator material;
- preserved every deferred consequence in a live owner or indexed raw
  destination;
- identified plausible lasting owners and their boundaries; and
- gathered enough information for a pre-promotion documentation-fit dry run.

Do not promote findings, update language examples broadly, archive this work
item, or begin work item `009` without the separately required discussion,
alignment, and authorization.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs ownership,
  focused reading, numbered work, promotion, and archival.
- [Language vision](../../language/vision.md) - provides foundational direction
  and the required-language-support boundary.
- [Zax language principles](../../language/principles.md) - provides the
  cross-cutting principles against which operator behavior should be evaluated.
- [Zax source structure](../../language/source-structure.md) - provides token
  spacing, continuation, contextual keywords, layout intent, and expression
  source boundaries.
- [Zax declarations and bindings](../../language/declarations-and-bindings.md) -
  provides declaration forms, name visibility, assignment boundaries, and
  operator-facing declaration constraints.
- [Zax qualifiers](../../language/qualifiers.md) - provides place, value, access,
  receiver, and indirection qualifications used by operator candidates.
- [Zax function invocation](../../language/function-invocation.md) - provides
  shared evaluation, binding, viability, expected-result selection, preference,
  ambiguity, and branch-specific callable selection.
- [Zax construction, replacement, and destruction](../../language/construction-and-destruction.md) -
  provides assignment/replacement lifetime boundaries and hidden-cost pressure.
- [Zax core flow control](../../language/core-flow-control.md) - provides
  exact-Boolean conditions, conditional-expression selected-arm order, and
  convergence.
- [Zax operators](../../language/operators.md) - provides the current bounded
  operator behavior that this work must extend coherently.
- [Zax language-design terms](../../language/terms.md) - provides current
  cross-cutting vocabulary.
- Focused [legacy basics operator catalog](../../basics.md#operators) - preserves
  the flat proposed operator list, protected forms, and word-operator boundary.
- [Legacy operator overloading](../../operator.md) - preserves primary evidence
  for declaration forms, arbitrary results, global/type-defined operators,
  receiver use, literals, and word operators.
- Relevant entries in the
  [cross-cutting audit](../raw/cross-cutting-audit.md) - preserve operator,
  branch-convergence, required-support, example, and source-stability audit
  obligations that must be dispositioned.

### Consequence-driven

- Read focused [casting](../../casting.md) material when conversion operators,
  `as`, or result-shape adaptation becomes necessary.
- Read focused [optional](../../optional.md), [Nothing](../../nothing.md), or
  [except](../../except.md) material when presence, negation, checked-result, or
  error-flow behavior constrains an operator family.
- Read focused [type definition](../../type-definition.md),
  [composition](../../composition.md), [alias](../../alias.md), or
  [raw structural typing](../raw/structural-typing.md) material when operator
  identity, receiver membership, structural compatibility, or result type
  relationships become necessary.
- Read focused [arrays](../../arrays.md), [variadic](../../variadic.md), or
  remaining legacy operator sections only when indexing, invocation, literals,
  or variadic behavior constrains the ordinary catalog.
- Read [raw analysis controls](../raw/analysis-controls.md) or
  [raw safety boundaries](../raw/safety.md) when checked failure, proof
  assertions, lint behavior, or protected-operation diagnostics become
  necessary.
- Read [raw build and dependency input](../raw/build-and-dependencies.md) when
  required primitive support or automatic helper/toolchain behavior becomes
  programmer-visible.
- Read focused [compiler directives](../../compiler-directives.md),
  [meta-functions](../../meta-functions.md), or [meta-types](../../meta-types.md)
  only when ordinary operator generation cannot be separated cleanly from
  compile-time facilities.
- Expand beyond this list whenever discussion or a material finding establishes
  a concrete need. State why the additional reading is relevant and keep it
  focused.

Material not listed in this reading scope is neither required nor prohibited. It
remains available when discussion or a concrete consequence makes it relevant.

### Audit-only

- `project/archive/`, including work items `001` through `007`.

Do not read archived work item `007` while conducting ordinary work on `008`.
Its applicable findings have been promoted into current language and project
owners and incorporated into the fixed initiating input above. Read it only when
the language maintainer explicitly requests a targeted audit or provenance
investigation.

Other audit-only material likewise requires direct language-maintainer
instruction or a specific audit or recovery need.

## Working record

Not started. The agent creating this work item stops here without analyzing its
content.
