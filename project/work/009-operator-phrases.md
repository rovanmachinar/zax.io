# 009: Operator phrases and keyword-neutral source

| Field | Value |
| --- | --- |
| Status | Active working material / non-normative |
| Work Item | `009` |
| Created | 2026-08-28 |
| Owns | The bounded review defined by the fixed initiating input |
| Does Not Own | Accepted complete casting, literal, numeric, allocation, lifetime, reflection, generic, formal grammar, or compiler behavior |

## Non-authority notice

This file is a collaborative working record. Existing statements, new
observations, candidate wording, and aligned findings remain non-authoritative
until a separately discussed, aligned, and explicitly authorized promotion
incorporates them into their lasting owner documents.

## Fixed initiating input

This section records the information known and aligned when work item `009` was
created. It is intentionally incomplete rather than exhaustive and must not be
rewritten as the work develops.

### Initiating concern

Develop a coherent human-developer-facing model for operator phrases and the
source interpretation required to use them safely:

1. the closed language-defined phrase catalog and any bounded custom/open phrase
   facility;
2. phrase words, holes, fixity, arity, precedence, associativity, and grouping;
3. declaration and forward forms for pre-unary, post-unary, binary, circumfix,
   call/index-related, and mixfix phrase components;
4. contextual keyword, identifier, and operator-phrase recognition;
5. longest-phrase and overlapping-phrase behavior;
6. whitespace, attachment, continuation, multiline layout, and future keyword
   stability;
7. keyword-neutral and confusable-form source, including the `bare{...}`
   candidate;
8. local phrase fencing, including the doubled-single-quote candidate and its
   literal interaction;
9. protected intrinsic phrase signatures, eager versus short-circuit behavior,
   and mixfix-consumption barriers;
10. namespace, import, visibility, custom-extension, ambiguity, and source
    stability;
11. costs, failures, diagnostics, reflection pressure, and formatting; and
12. lasting documentation owners and boundaries.

Use the promoted operator, catalog, mixfix, source-structure, declaration,
invocation, qualifier, and terminology owners as constraints. Recover
Zax-specific intent before borrowing phrase behavior from other languages.

### Motivating pressure

Current conceptual owners establish:

- a closed symbolic and circumfix catalog;
- one language-defined precedence and associativity per operator form;
- explicit pre-unary, post-unary, binary, circumfix, call/index, and mixfix
  categories;
- contextual keyword recognition;
- grouped independent unary applications;
- shared callable viability, expected-result limits, partial-order preference,
  ambiguity, and unavailable-best behavior;
- protected intrinsic signatures and distinct mixfix-consumption barriers;
- accepted operation concepts whose exact phrase spellings remain deferred;
- illustrative logical, bitwise, magnitude, shift, transform, scan, and
  multiword phrase operations; and
- a requirement that future phrase syntax not reinterpret ordinary source
  silently.

Legacy evidence proposes built-in phrases such as `as`, `unsafe as`,
`size of`, and host/target layout operations, together with arbitrary custom
phrases such as `run`, `run fast`, and `run fast from`. Checked-in wording does
not decide that those exact phrases or an open custom facility are accepted.

New aligned input adds a concrete keyword-neutral source candidate:

```zax
bare{ if + while }
```

and an intent-gated asymmetric circumfix:

```zax
// Illustrative future syntax.
saturated := bare{ ||value| }
```

The candidate `bare{...}` enclosure:

- uses one contiguous contextual opener, while bare `bare` is not a keyword and
  `bare {` is not the same form;
- creates no scope, runtime operation, final precedence boundary, expression
  node, or mixfix hole;
- requires one independently complete expression or effective statement;
- neutralizes non-operator keyword roles and permits recognized confusable
  forms;
- cannot obtain missing operands, joiners, or header sections outside its
  boundary; and
- becomes transparent before final precedence and mixfix selection.

Another candidate fences phrase words locally:

```zax
value''fast''
''run''value
x ''runs fast'' y
```

That form makes phrase extent and attachment visible but may conflict with empty
or adjacent literals, literal prefixes, escaping, keyboards, and documentation.

This work must refine, replace, or reject those candidates rather than assuming
either spelling is correct.

### Known assumptions

- [Zax language vision](../../language/vision.md) is accepted foundational
  direction.
- [Zax language principles](../../language/principles.md) owns reusable
  cross-cutting design and use principles.
- [Zax source structure](../../language/source-structure.md) owns ordinary
  contextual keywords, whitespace intent, continuation, layout, and diagnostic
  categories.
- [Zax declarations and bindings](../../language/declarations-and-bindings.md)
  owns namespaces, visibility, and name-introduction boundaries.
- [Zax function invocation](../../language/function-invocation.md) owns shared
  callable mapping, defaults, result shapes, viability, expected results,
  preference, ambiguity, and unavailable best.
- [Zax qualifiers](../../language/qualifiers.md) owns receiver and operand
  capability truth.
- [Zax operators](../../language/operators.md) owns the shared operator model and
  selection.
- [Zax operator catalog](../../language/operator-catalog.md) owns current exact
  symbolic/circumfix forms, precedence, built-in behavior, and accepted phrase
  operation concepts.
- [Zax mixfix operators](../../language/mixfix-operators.md) owns tree matching,
  holes, phrase components, protected barriers, and nested outward resolution.
- [Zax language-design terms](../../language/terms.md) owns eager,
  short-circuit, protection, mixfix, keyword-role, and intent terminology.
- Phrase parsing may not depend on runtime operand values.
- Types, namespaces, and imports may not assign another precedence to one phrase.
- Source, declaration, import, and discovery order may not resolve ambiguity.
- User-defined phrase operations do not gain short-circuit or unsafe behavior
  merely from their words.
- No source-order guess may rescue an otherwise ambiguous phrase meaning.
- There is no language implementation in this repository.

### Known inclusions

- Inventory every operator phrase or phrase-like operation in current, legacy,
  and live raw material.
- Classify each as accepted operation concept, candidate spelling, deferred
  domain behavior, rejected, reserved, protected, or superseded.
- Decide whether phrases form a closed catalog, an explicitly fenced custom
  facility, another bounded extension model, or a combination.
- Decide natural versus explicit source for language-defined and custom phrases.
- Pre-unary, post-unary, binary, circumfix, call/index, and mixfix phrase forms.
- Phrase declarations, type-defined receivers, global visibility, forward
  recognition, imports, and namespaces.
- Phrase holes and exact phrase extent.
- Longest phrase, common prefixes/suffixes, overlapping declarations, and
  ambiguity.
- Precedence and associativity for every accepted phrase family.
- Phrase whitespace, physical newlines, continuation, and formatting.
- Independent pre/post phrase composition versus one circumfix phrase.
- Keyword, identifier, and phrase-role conflicts.
- The complete `bare{...}` candidate, including:
  - one complete expression or effective statement;
  - no scope or lifetime boundary;
  - multiline layout without block-style statement separation;
  - transparent final precedence and mixfix behavior;
  - keyword-neutral interpretation;
  - confusable-form intent;
  - invalid joiner/operator boundary crossing;
  - nested enclosures, reflection, documentation, and formatting; and
  - interaction with future categorized unsafe source.
- Doubled-quote or other local phrase fences and literal conflicts.
- The intent taxonomy:
  - layout-intent error;
  - operator-attachment intent error;
  - redundant-structure intent error;
  - confusable-form intent error; and
  - keyword-role conflict.
- Exact Boolean short-circuit phrase operations and eager custom shapes.
- Protected intrinsic phrase signatures and mixfix-consumption barriers.
- Accepted phrase-operation concepts from the operator catalog:
  - logical NAND, AND-NOT, NOR, OR-NOT, XNOR, and mutation;
  - bitwise NAND, NOR, XNOR, OR-NOT, and mutation;
  - saturating magnitude;
  - modulo-count shifts;
  - bit and byte reversal;
  - masked extraction and deposit;
  - first/last set-bit scans;
  - multiword shifts, rotates, and funnel shifts; and
  - future phrase components inside mixfix declarations.
- Legacy conversion, endian, layout, reflection, lifetime, allocator, host, and
  target phrase evidence at the depth needed to design phrase source safely.
- Costs, diagnostics, source stability, tooling, and formatting.
- Lasting owners, phrase catalog/reference boundaries, and future raw
  destinations.

### Known starting boundaries

- Complete `as`, `unsafe as`, conversion, and casting semantics.
- Complete literal parsing, prefixes, payloads, and custom literal execution.
- Complete numeric, fixed-point, arbitrary-width, enum, or floating-point
  behavior.
- Complete pointer, lifetime, allocator, overhead, and layout semantics.
- Complete reflection and host/target compile-time behavior.
- Complete generics, concepts, specialization, and generic phrase constraints.
- Complete function composition, chaining, lambda, and capture behavior.
- Complete indexing, slicing, ranges, and proxies.
- Complete unsafe controls and categorized unsafe source.
- Formal grammar, parser recovery, compiler implementation, lowering, ABI, and
  toolchain internals.

These areas need not be designed to begin. They do not prohibit following,
recording, or dispositioning material consequences.

### Intentionally unresolved framing

- The exact phrase catalog and canonical words.
- Closed versus bounded open/custom phrase operators.
- Whether custom phrases require explicit fencing at every use.
- Whether `bare{...}` is the correct name, delimiter, and source model.
- Whether keyword-neutral parsing prefers identifiers, phrases, or diagnoses
  incomparable interpretations.
- Whether doubled single quotes or another fence can coexist with literals.
- Exact declaration, forward, import, qualification, and namespace syntax.
- Exact phrase precedence and associativity.
- Longest phrase and common-prefix behavior.
- Phrase circumfix and independent pre/post composition.
- Phrase components inside mixfix declarations.
- Exact protected phrase signatures and short-circuit barriers.
- Source reflection and formatting of transparent or fenced phrase source.
- Which confusable forms justify an explicit mechanism.
- Which findings belong in current operator, catalog, mixfix, source, terms,
  declaration, invocation, cast, literal, or future owners.

### Initial stopping guidance

Stop when the work has:

- dispositioned every known phrase spelling and operation family;
- established natural and explicit phrase source;
- established keyword/identifier/phrase recognition and future-keyword safety;
- established phrase fixity, arity, precedence, associativity, and grouping;
- established declarations, receivers, visibility, forward recognition, imports,
  and ambiguity behavior;
- established or rejected the `bare{...}` model with representative expression,
  statement, flow-header, invalid-boundary, and confusable-form examples;
- established or rejected local phrase fencing with literal coordination;
- established protected, eager, short-circuit, mixfix-component, and custom phrase
  behavior;
- constrained casting, literal, numeric, pointer, lifetime, allocation,
  reflection, generic, unsafe, and implementation consequences;
- identified costs, diagnostics, formatting, and source-stability pressure;
- preserved every deferred consequence in a live owner or indexed raw
  destination;
- identified plausible lasting owners and their boundaries; and
- gathered enough information for a pre-promotion documentation-fit dry run.

Do not promote findings, update current phrase examples broadly, archive this
work item, or begin work item `010` without the separately required discussion,
alignment, and authorization.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs ownership,
  focused reading, numbered work, promotion, and archival.
- [Zax language vision](../../language/vision.md) - provides foundational
  language direction.
- [Zax language principles](../../language/principles.md) - provides clear
  intent, readability, source stability, cost, and optimization constraints.
- [Zax source structure](../../language/source-structure.md) - provides
  contextual-keyword, whitespace, continuation, layout, and diagnostic rules.
- [Zax declarations and bindings](../../language/declarations-and-bindings.md) -
  provides names, visibility, namespaces, and declaration boundaries.
- [Zax function invocation](../../language/function-invocation.md) - provides
  shared callable mapping, defaults, results, selection, and ambiguity.
- [Zax qualifiers](../../language/qualifiers.md) - provides receiver and operand
  qualification truth.
- [Zax operators](../../language/operators.md) - provides the shared operator
  model and selection.
- [Zax operator catalog](../../language/operator-catalog.md) - provides current
  phrase-operation concepts, precedence obligations, and deferred forms.
- [Zax mixfix operators](../../language/mixfix-operators.md) - provides phrase
  components, tree matching, holes, and barriers.
- [Zax language-design terms](../../language/terms.md) - provides current intent,
  protection, and operator vocabulary.
- [Raw bare-source input](../raw/bare-source.md) - preserves the complete
  keyword-neutral/confusable-form candidate and examples.
- [Raw literal input](../raw/literal-operators.md) - preserves doubled-quote
  phrase fencing and literal collision pressure.
- Focused [legacy basics operator phrases](../../basics.md#historical-operator-phrase-proposals) -
  preserves the broad built-in phrase inventory.
- [Legacy forward declarations](../../forward.md) - preserves custom pre/post/
  binary phrase and literal forward examples.

### Consequence-driven

- Read focused [casting](../../casting.md) when `as`, `unsafe as`, conversion,
  or endian phrases constrain recognition or protected signatures.
- Read [raw function composition and chaining](../raw/function-composition-and-chaining.md)
  when phrase call/chaining/capture behavior becomes necessary.
- Read focused [compiler directives](../../compiler-directives.md),
  [meta-functions](../../meta-functions.md), or [meta-types](../../meta-types.md)
  when layout, reflection, host/target, or compile-time phrase behavior constrains
  the phrase model.
- Read focused [pointers](../../pointers.md), [memory allocation](../../memory-allocation.md),
  [custom allocators](../../custom-allocators.md), or
  [raw lifetime input](../raw/lifetimes.md) when pointer, lifetime, overhead, or
  allocator phrases constrain recognition.
- Read [raw indexing and slicing](../raw/indexing-and-slicing.md) when phrase
  forms interact with index, range, or splice grammar.
- Read [raw analysis controls](../raw/analysis-controls.md) or
  [raw safety boundaries](../raw/safety.md) when `bare{...}` interacts with
  semantic assertions, lint control, or categorized unsafe source.
- Expand beyond this list whenever discussion or a material finding establishes
  a concrete need. State why the additional reading is relevant and keep it
  focused.

Material not listed in this reading scope is neither required nor prohibited. It
remains available when discussion or a concrete consequence makes it relevant.

### Audit-only

- `project/archive/`, including work items `001` through `008`.
- [Historical operator-overloading path](../../operator.md).

Do not read archived work item `008` while conducting ordinary work on `009`.
Its applicable findings have been promoted into current owners and incorporated
into the fixed initiating input above. Read it only when the language maintainer
explicitly requests a targeted audit or provenance investigation.

## Working record

Not started. The agent creating this work item stops here without analyzing its
content.
