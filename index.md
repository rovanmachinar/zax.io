# Zax programming language

Zax is a general-purpose programming language intended to provide high-level
capabilities without hiding the lower-level behavior, costs, and tradeoffs
producing them.

> **Project status:** Zax is in conceptual design. The language vision is the
> current owner of its accepted foundational direction. The topic pages below
> are legacy design notes being reviewed; they are not yet a formal
> specification or implementation promise.

## Start here

- Read the [Zax language vision](language/vision.md) for its purpose, audience,
  principles, goals, advantages, non-goals, and accepted tradeoffs.
- Use the grouped design notes below when exploring a specific proposed concept.

## Legacy language-design notes

These pages preserve substantial prior design work. Their detailed syntax and
semantics remain non-authoritative until individually reviewed and promoted.

### Foundations and flow

- [Basics](basics.md)
- [Type definition](type-definition.md)
- [Enums](enums.md)
- [Arrays](arrays.md)
- [Functions](functions.md)
- [Variadic functions](variadic.md)
- [Flow control](flow-control.md)
- [Scope](scope.md)
- [Operator overloading](operator.md)

### Types and data modeling

- [Alias](alias.md)
- [Composition](composition.md)
- [Mutability](mutable.md)
- [Optional types](optional.md)
- [Nothing type instances](nothing.md)
- [Partial types](partial.md)
- [Casting](casting.md)

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
