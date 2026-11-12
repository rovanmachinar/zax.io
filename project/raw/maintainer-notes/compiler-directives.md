
# compiler-directives.md

These are the maintainer's notes on compiler directives that remain useful after
the compile-time, directive, and generics reviews. Every other note was consumed
by those reviews. The Generators note below carries direction for source
generation, and takes priority over legacy directive material where they differ.

## Generators

For the generation stuff, I think that should be separated out. I do want generators, and they do need to execute at compile time but I think I'd like to do that separate so we should leave that part in the original document for now.

## Generating declarations from a type pack

This section is not a maintainer note. It records source-generation pressure
found while reviewing variadics, kept here because this file is the indexed
source-generation input.

A [variadic](../../../language/variadics.md) function can walk a list of types
with `each`. The body is checked once per type, but it produces runtime code
only:

```zax
describeType$ final : (UseType : type)()() = {
}

describeTypes$ final : (Items ... : type)()() = {
  each Item : in Items {
    describeType$(Item)()   // a runtime call per type, inside one function body
  }
}

describeTypes$(Integer, String)()
```

A library may instead want to generate declarations from such a list: one
accessor, overload, or member per type. No current form does that:

```zax
// Illustrative only: generating a declaration per type is not established.
MyVisitor$(Kinds ... : type) :: type {
  each Kind : in Kinds {
    visit final : ()(value : Kind readonly &) = {
    }
  }
}
```

Future generator work must decide whether walking a type pack can produce
declarations, how generated names avoid collisions, when the generated
declarations become visible to lookup, and how diagnostics point back to the
generating source.

**Activates when** source-generation work begins.

**Constraint on current design.** `each` over a type pack produces runtime
bodies only; it never generates declarations.
