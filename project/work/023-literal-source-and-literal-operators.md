# 023: Literal source and literal operators

| Field | Value |
| --- | --- |
| Status | Active working material / non-normative / awaiting assignment |
| Work Item | `023` |
| Created | 2026-09-15 |
| Owns | The bounded review defined by the fixed initiating input |
| Does Not Own | Complete generics, a complete text/Unicode type system, runtime string libraries, exhaustive numeric operations, compiler parser implementation, or formal grammar |

## Non-authority notice

This file is a collaborative working record. Existing statements, new
observations, candidate wording, and aligned findings remain non-authoritative
until a separately discussed, aligned, and explicitly authorized promotion
incorporates them into their lasting owners.

## Fixed initiating input

This section records the information aligned when work item `023` was created.
It is intentionally incomplete and must not be rewritten as work develops.

### Initiating concern

Establish a coherent programmer-facing model for Zax literal source and literal
operators, including ordinary numeric and string forms, attached
prefixed/custom literals, compile-time parsing, one concrete result selection,
joining, diagnostics, and interaction with expression typing.

The [literal-operator maintainer notes](../raw/literals-maintainer-notes.md) are
primary maintainer-supplied input that refreshes several legacy literal
concepts. Read and disposition their complete contents rather than treating
older root pages as the current proposal. The indexed
[literal-operator raw input](../raw/literal-operators.md) preserves current
token-boundary constraints and deferred questions discovered by prior work.

### Motivating pressure

Current integer design defines uncommitted integer source and realization.
Current fixed-point and floating-point design defines typed real-number
realization and decimal `e`/`E` power-of-ten exponents. Source structure and
operator phrases distinguish:

```zax
"ordinary text" // ordinary unprefixed literal
h'DEADBEEF'     // attached prefixed literal
foo 'bar'       // separated operator-phrase fence
```

The language still lacks one current explanation of:

- which remaining literal tokens and payloads exist;
- how a literal operator is declared and found;
- when and how its code executes during compilation;
- how one concrete result type is known;
- how a numeric payload can request enough width without always selecting an
  extreme maximum type;
- how mathematical-value and raw-bit interpretations differ;
- which ordinary and prefixed string forms exist; and
- when adjacent literal results join.

These questions cross parsing, type selection, generics, compile-time execution,
strings, and operators. The literal model must settle their shared boundary
without attempting to complete every adjacent subsystem.

### Known assumptions

- A resolved expression result has one concrete static type before it
  participates in its surrounding expression.
- A resolved prefixed literal does not remain an uncommitted integer merely so
  later context can reinterpret it.
- Payload-dependent numeric width remains legitimate generic pressure: a future
  literal/type factory may derive one concrete result from compile-time payload
  magnitude before resolution completes.
- Range failure does not retry unrelated literal overloads or search wider
  result types after one candidate has already been selected.
- Literal operators are compile-time facilities; exact execution requirements,
  sandboxing, resource limits, and failure paths must remain visible.
- An attached single quote introduces a literal payload. A whitespace-separated
  single quote is an exact operator-phrase fence, not an unprefixed literal.
- Current decimal real source accepts `e`/`E` with an optionally signed
  power-of-ten exponent.
- Typed integer, fixed-point, and floating realization semantics remain owned by
  their current scalar documents.

### Known inclusions

- Ordinary unprefixed integer, real-number, and string source as needed to give
  readers one literal taxonomy.
- Prefix recognition, attachment, namespace/import visibility, ambiguity, and
  owner/type relationship.
- Single- and double-quoted payload boundaries, escaping or absence of escaping,
  invalid source characters, multiline/continuation behavior, and empty
  payloads.
- Literal declaration shape, input, compile-time execution, result identity,
  diagnostics, and source stability.
- Built-in and custom numeric radix literals, including mathematical magnitude
  versus typed value versus raw bit-pattern interpretation.
- Payload-dependent concrete numeric result pressure and its exact handoff to
  future generic/type-factory work.
- Decimal point and accepted decimal exponent forms.
- Evaluation of binary, hexadecimal, octal, or other-base fractional/exponent
  forms when they provide a coherent programmer use case.
- Ordinary and prefixed string/character forms, result identities, supported
  encoding pressure, and conversion/join consequences far enough to assign one
  live owner to every material deferral.
- Adjacent literal merging or joining, including when prefixes, grouping,
  separators, or differing result types prevent one unified payload.
- Interaction with candidate selection, contextual completion, constants,
  arrays, source reflection, formatting, and documentation.
- Complete disposition of the literal maintainer notes, literal-operator raw
  input, and applicable legacy literal material.

### Exponent review boundary

Work item `023` does not presume that every numeric base or encoded payload
needs an exponent form.

Evaluate an exponent only when it gives the represented form a clear,
nonredundant use:

- decimal `e`/`E` remains the established power-of-ten form;
- binary or hexadecimal floating-style payloads may justify a `p`/`P`
  power-of-two exponent comparable to established programmer expectations;
- arbitrary-base payloads must not receive an exponent merely for catalog
  symmetry; and
- a prefix-specific parser may reject fractional or exponent syntax when its
  intended result is an integer or raw bit pattern.

Illustrative candidates such as `h'1.8p10'` or `b'1.1p-3'` are questions for
review, not accepted syntax.

### Known starting boundaries

- Complete generic declaration, type-factory, associated-type, inference, and
  specialization syntax.
- Complete compile-time execution scheduling, sandboxing, caching, and resource
  policy.
- A complete text, Unicode, locale, normalization, code-page, formatting, or
  runtime string-conversion model.
- Exhaustive numeric families or operations beyond literal formation.
- General implicit conversion or result-type lookahead.
- Arbitrary expression parsing inside literal payloads unless a concrete
  literal-operator use requires it.
- Source reflection and formatter implementation.
- Compiler tokenization/parsing algorithms and formal grammar.

These boundaries do not erase consequences. Preserve every material pressure in
a current owner or indexed raw destination rather than leaving it only in the
eventual archive of this work item.

### Intentionally unresolved framing

- Whether ordinary string source performs escapes, preserves source text
  directly, or uses another explicitly taught policy.
- Which built-in prefixes exist and which are aliases.
- Whether custom literal prefixes form an open, bounded, or otherwise governed
  catalog.
- Whether prefixed payloads may use both quote kinds and how delimiter
  characters are represented inside them.
- Whether adjacent equal-prefix tokens are parsed once as one payload or produce
  concrete values that then join.
- Which string, character, byte-array, or encoded-text identities each built-in
  form returns.
- Whether a numeric prefix denotes mathematical radix, one concrete type, a raw
  bit pattern, or separately named forms for those meanings.
- How a literal declaration can compute one bounded concrete result type from
  payload magnitude without reopening the result after selection.
- Whether binary/hexadecimal `p`/`P` exponents are useful, and whether any other
  encoded numbers warrant exponent syntax.
- Exact literal diagnostics, source limits, and malformed-token recovery.

### Initial stopping guidance

Stop when the work has:

- established a teachable literal taxonomy and token/attachment boundaries;
- described ordinary and prefixed numeric and string use with representative
  valid and invalid source;
- defined literal declaration, discovery, compile-time execution, and one
  concrete result boundary at the depth needed by programmers;
- dispositioned payload-dependent result types without prematurely completing
  generics;
- decided each materially proposed exponent form by use rather than symmetry;
- defined joining/merging behavior and its costs;
- identified supported built-in forms and assigned complete string/text
  consequences to current or indexed future owners;
- reconciled source structure, phrase fencing, scalar realization, operators,
  diagnostics, reflection, and source stability;
- dispositioned every useful item in the primary literal notes, focused raw
  input, and reached legacy material; and
- completed the required documentation-fit dry run.

Do not promote findings, archive this work item, begin work item `024`, or
design complete generics, compile-time execution, text/Unicode semantics,
runtime string libraries, or compiler parsing without the separately required
discussion, alignment, and authorization.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs focused reading,
  decision maturity, promotion, deferral, and closure.
- [Literal-operator maintainer notes](../raw/literals-maintainer-notes.md) -
  supplies the maintainer's refreshed literal, string, numeric-prefix,
  compile-time, joining, and result-pressure input.
- [Raw literal operators](../raw/literal-operators.md) - preserves current
  attachment/phrase-fence constraints and deferred prefix, payload, result, and
  ordinary-real questions.
- [Source structure](../../language/source-structure.md) - owns current token
  attachment, phrase-fence, string-delimiter, continuation, and layout
  boundaries.
- [Integer literals and realization](../../language/integer-literals.md) - owns
  uncommitted integers, decimal exponent token boundaries, candidate holes, and
  the rule that range failure does not retry another type.
- [Fixed-point scalars](../../language/fixed-point-scalars.md) - owns typed
  fixed-point real realization, rounding, range, and representation.
- [Binary floating-point scalars](../../language/floating-point-scalars.md) -
  owns typed floating real realization, format rounding, range, and
  representation.
- [Operator phrases](../../language/operator-phrases.md) - owns attached literal
  versus whitespace-separated single-quote phrase fencing.

### Consequence-driven

- [Raw type parameters and generics](../raw/type-parameters-and-generics.md)
  when payload-dependent concrete result identity, bounded size selection, or
  a literal/type factory becomes material.
- [Raw compile-time execution](../raw/compile-time-execution.md) when required
  literal execution, host/target semantics, resource failure, or static
  availability becomes concrete.
- [Function invocation](../../language/function-invocation.md),
  [declarations and bindings](../../language/declarations-and-bindings.md),
  [operators](../../language/operators.md), and the
  [operator catalog](../../language/operator-catalog.md) when declaration,
  result, discovery, ambiguity, or exact operator form crosses those owners.
- [Raw reflection](../raw/reflection.md) when source spelling, payload,
  generated result identity, or literal provenance requires metadata.
- [Raw numeric type families](../raw/numeric-type-families.md) when a
  representation-operation or nonstandard numeric family becomes necessary.
- Relevant legacy literal, string, character, array, encoding, namespacing, or
  casting material only when a concrete question cannot be resolved from the
  primary notes and current owners.

### Audit-only

- Archived work only when a concrete provenance or regression question cannot
  be answered from current owners and live raw input.

## Initial working posture

Begin with programmer source: an ordinary integer, an ordinary decimal real, an
ordinary string, a small radix payload, a payload requiring a much wider result,
and adjacent literal tokens. Explain what each token produces before designing
the declaration mechanism that supplies it.

Treat the maintainer notes as refreshed evidence rather than accepted behavior.
Preserve current phrase-fence and scalar-realization boundaries. Evaluate
non-decimal exponents only when they make a concrete encoding easier to express;
do not add one form per radix merely to complete a matrix.

## Working record

Awaiting assignment. Creating and routing work item `023` does not authorize
analysis or begin the literal review.
