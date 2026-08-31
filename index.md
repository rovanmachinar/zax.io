# Zax programming language

Zax is a general-purpose programming language intended to provide high-level
capabilities without hiding the lower-level behavior, costs, and tradeoffs
producing them.

> **Project status:** Zax is in conceptual design. The current conceptual-design
> pages below own the accepted concerns assigned to them. The root topic pages
> remain legacy design notes being reviewed; none of this material is a formal
> specification or implementation promise.

## Start here

- Read the [Zax language vision](language/vision.md) for its purpose, audience,
  foundational direction, goals, advantages, non-goals, and accepted tradeoffs.
- Read [Zax language principles](language/principles.md) for the reusable
  principles that guide source, APIs, and future language decisions.
- Read [Zax source structure](language/source-structure.md) for statement-level
  newlines, continuation, bodies, semicolon composition, mandatory layout, and
  comments.
- Read [Zax declarations and bindings](language/declarations-and-bindings.md) for
  declaration, initialization, binding visibility, qualifiers, and assignment
  boundaries.
- Read [Zax identity types](language/identity-types.md) for transparent aliases,
  distinct identities, admission, projection, exposed and opaque surfaces, and
  declared identity bridges.
- Read [Zax qualifiers](language/qualifiers.md) for place replacement, value
  mutability, access capabilities, defaults, indirection, and unsafe escape
  behavior.
- Read [Zax function invocation](language/function-invocation.md) for ordinary
  calls, arguments, defaults, results, result routing, and callable selection.
- Read [Zax construction, replacement, and destruction](language/construction-and-destruction.md)
  for constructors, construction packets, generated lifecycle operations,
  same-storage replacement, and destruction.
- Read [Zax core flow control](language/core-flow-control.md) for exact-`Boolean`
  conditions, `if`/`else`, loops, explicit `scope`, flow labels, and
  `break`/`continue`/`next`/`return`.
- Read [Zax operators](language/operators.md) for the shared operator mental
  model, declarations, receiver discovery, selection, expected results,
  evaluation, protected domains, and fallback.
- Read [Zax operator phrases](language/operator-phrases.md) for word-spelled
  operations, receiver-owned declarations, candidate interpretations, grouping,
  fencing, physical presentation, and phrase-specific diagnostics.
- Use the [Zax operator catalog](language/operator-catalog.md) to look up exact
  symbolic/circumfix forms, precedence, built-in arithmetic, logical, bitwise,
  shift, mutation, result, and availability behavior.
- Read [Zax integers](language/integers.md) for exact and target-selected integer
  types, representation, conversion, arithmetic policy, counts, sizes, and
  portability.
- Use the [Zax integer operator catalog](language/integer-operator-catalog.md)
  to look up complete protected integer availability, results, policies,
  failures, counts, shifts, conversion, `delta`, and `distance`.
- Read [Zax mixfix operators](language/mixfix-operators.md) for direct
  multi-component tree operations, holes, receiver anchors, matching,
  decomposition, and protected barriers.
- Read [Zax endianness](language/endianness.md) for endian semantic enum types,
  receiver-correct encoding, decoding, raw adoption, the supported endian
  operation surface, and native versus compiler-host and target byte order.
- Use [Zax language-design terms](language/terms.md) for cross-cutting
  descriptive vocabulary used by current conceptual design.
- Use the grouped design notes below when exploring a specific proposed concept.

## Current conceptual design

- [Language vision](language/vision.md)
- [Language principles](language/principles.md)
- [Source structure](language/source-structure.md)
- [Declarations and bindings](language/declarations-and-bindings.md)
- [Identity types](language/identity-types.md)
- [Qualifiers](language/qualifiers.md)
- [Function invocation](language/function-invocation.md)
- [Construction, replacement, and destruction](language/construction-and-destruction.md)
- [Core flow control](language/core-flow-control.md)
- [Operators](language/operators.md)
- [Operator phrases](language/operator-phrases.md)
- [Operator catalog](language/operator-catalog.md)
- [Integers](language/integers.md)
- [Integer operator catalog](language/integer-operator-catalog.md)
- [Mixfix operators](language/mixfix-operators.md)
- [Endianness](language/endianness.md)
- [Language-design terms](language/terms.md)

## Legacy language-design notes

These pages preserve substantial prior design work. Their detailed syntax and
semantics remain non-authoritative until individually reviewed and promoted.

### Foundations and flow

- [Basics](basics.md) — legacy foundations; current finite integer design is
  owned by [Zax integers](language/integers.md)
- [Type definition](type-definition.md)
- [Enums](enums.md) — legacy enum input; current generated enum operations are
  in the [operator catalog](language/operator-catalog.md#generated-enum-forms)
  and endian behavior is owned by [Zax endianness](language/endianness.md)
- [Arrays](arrays.md)
- [Functions](functions.md)
- [Variadic functions](variadic.md)
- [Flow control](flow-control.md)
- [Scope](scope.md)

### Types and data modeling

- [Alias](alias.md) — legacy alias input; current transparent aliases and
  explicit identities are owned by
  [Zax identity types](language/identity-types.md)
- [Composition](composition.md)
- [Optional types](optional.md)
- [Nothing type instances](nothing.md)
- [Partial types](partial.md)
- [Casting](casting.md) — legacy general casting input; current integer
  conversion is owned by [Zax integers](language/integers.md)

### Memory and lifetime

- [Pointers and references](pointers.md)
- [Memory allocation](memory-allocation.md)
- [Custom allocators](custom-allocators.md)
- [Handle and hint pointers](handle-hint.md)
- [Strong and weak pointers](strong-weak.md)
- [Constructors and destructors](ctor-dtor.md)
- [Discard operator](discard.md)

### Build-time programming and modules

- [Compiler directives](compiler-directives.md)
- [Meta-functions](meta-functions.md)
- [Meta-types](meta-types.md)
- [Namespacing and module imports](namespacing.md)
- [Forward declarations and hints](forward.md)

### Execution, errors, and concurrency

- [Except error handling](except.md)
- [Lazy functions](lazy.md)
- [Concurrency](concurrency.md)
- [Context type](context.md)
- [Compiler warnings and errors](warnings-errors.md)

## Questions

- [Frequently asked questions](faq.md)
