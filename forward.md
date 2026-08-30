
# [Zax Programming Language](index.md)

## Forward

Accepted forward-completion, named self-reference, recursive-layout, and pending qualified-path
constraints are defined by
[Zax declarations and bindings](language/declarations-and-bindings.md). This
page remains legacy input for the complete forward-declaration catalog,
dependency behavior, and generated forward material.

> **Operator phrases.** An operator phrase is declared by its operator
> implementation. Zax has no separate phrase-form declaration and no phrase
> `forward` form. See
> [Zax operator phrases](language/operator-phrases.md#declaring-a-phrase),
> [Zax declarations and bindings](language/declarations-and-bindings.md#operator-phrase-declarations-and-type-parameters)
> and [Zax operators](language/operators.md#operator-phrases).
>
> `roman :: forward operator literal` remains unresolved literal evidence
> preserved in
> [raw literal input](project/raw/literal-operators.md).
>
> The variable, type, enum, namespace, and module forward material on this page is
> unaffected and remains ordinary legacy input.

A named type does not require `forward` merely to refer to its own incomplete
name inside its body. `forward` remains relevant when a different name is needed
before its declaration, for mutual or separately completed definitions, and
when a suffix must remain pending below an incomplete forwarded prefix.

### Basic forwarding concept

A compiler will attempt to resolve dependencies in late-binding processes but the programmer may have to give hints to the compiler so that all the symbols are recognizable as their appropriate type. Giving hints will assist the compiler to better recognize symbols and categorize these symbols appropriately during the compilation process.

A `forward` keyword is used to assist in symbol recognition. A symbol name is declared, followed by the `forward` declaration followed by a declaration of how to interpret the symbol. Any defined symbols will only be visible available within the context and exporting rules apply as normal.

A `forward` does not define qualifiers for a variable or type. All `forward` definitions are temporary until the real definition becomes available. A `forward-symbol-type-mismatch` error will occur of a forwarded symbol's type does not match a later defined symbol's type.

Ultimately all forward referenced symbols must become resolved or a compiler will issue an error.

````zax
// forward of variable names
variableName :: forward variable

// forward of type names
TypeName :: forward type

// forward of a value within a forwarded `type`
TypeName.value :: forward variable

// forward of enum type and an enum value
EnumType :: forward enum

// order matters as `EnumType` needs to be forwarded prior to `Value`
EnumType.Value :: forward enum value

// Literal forwarding remains raw literal evidence.
roman :: forward operator literal

// forward declare a namespace
ImportedNamespace :: forward namespace

// forward declare a type within a namespace
ImportedNamespace.TypeName :: forward type

// forward declare a variable into a known namespace
Module.globalValue :: forward variable

// ...

// entirely composed statements based exclusively on forwarded symbols

run to a store with variableName

variableName connect with Module.globalValue

variableName jump farther

variableName = roman'XIII'

variableName = EnumType.Value as Integer

// defining a new variable based on a forward declared type which is treated as
// a meta-type
unseenVariable := 3.5 : TypeName$(Float)

anotherUnseenVariable : ImportedNamespace.TypeName = "hello there!"

// usage of a forward declared pre-unary operator based on the `type` for
// unseenVariable
shake up unseenVariable
````
