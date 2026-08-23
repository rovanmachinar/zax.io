# Zax language-design terms

| Field | Value |
| --- | --- |
| Status | Current conceptual terminology |
| Audience | Human developers reading, writing, discussing, or documenting Zax |
| Applies To | Cross-cutting human-language terms used by current conceptual design |
| Implementation State | Not applicable |
| Owns | Concise meanings and usage boundaries for cross-cutting descriptive terms |
| Does Not Own | Keyword catalogs, complete behavior of the concepts named here, formal grammar, compiler terminology, implementation data structures, or definitions already owned by a language concept document |

## Purpose

This document defines the human-language terms used to discuss Zax consistently.
It is not a list of reserved words or operators. A term may describe a concept
without being available as source syntax.

Behavior remains with its concept owner. This page gives readers enough meaning
to follow documentation and links to the relevant owner when the distinction
affects programming behavior.

## Access path

An **access path** is one particular route through which code observes or acts
on a value. A name, reference, pointer dereference, member projection, parameter,
or capture may provide an access path.

Different paths to the same value may have different capabilities. See
[Zax qualifiers](qualifiers.md).

## Binding

A **binding** associates a source-level name with a declaration. The binding
becomes visible at the declaration boundary even while its initialization is
still being resolved.

A binding is not the same as the value it names or the storage place holding
that value. See [Zax declarations and bindings](declarations-and-bindings.md).

## Construction packet

A **construction packet** is the `[{ ... }]` construct that supplies positional
constructor arguments, named constructor arguments, and direct stored-member
initializers.

Packet order controls input evaluation and binding. It does not reorder member
construction. See
[Zax construction, replacement, and destruction](construction-and-destruction.md#construction-packets).

## Current instance

The **current instance** is the instance whose type-defined function body is
executing.

Use this term when discussing the body independent of invocation or overload
selection. At a call or operator-selection boundary, use **receiver operand**.
Do not use "receiver object"; Zax is not object-oriented.

## Direct storage

**Direct storage** means that a value occupies the storage place declared for
it, rather than being reached through a pointer or reference.

For a directly stored value, the declared binding and value share one immediate
replacement boundary. Indirection introduces additional boundaries.

## Indirection level

An **indirection level** is one pointer or reference layer between an access path
and a value or another pointer/reference value.

Each level may have its own place and access qualifications. See
[Zax qualifiers](qualifiers.md).

## Lifecycle operation

A **lifecycle operation** establishes, transitions, or ends a value or member
lifetime.

Ordinary `+++`, contextual `replacement +++`, `---`, and the compiler-owned
generated replacement skeleton participate in lifecycle operations. An
arbitrary operator is not a lifecycle operation merely because it uses `=`.
See [Zax construction, replacement, and destruction](construction-and-destruction.md).

## Place

A **place** is storage that can hold a value. Replacing a place changes which
value that storage holds.

Place replacement is distinct from changing the contents of the value already
stored there. See [Zax qualifiers](qualifiers.md).

## Pointee

A **pointee** is the place or value targeted by a pointer.

The binding storing a pointer and its pointee are distinct qualification layers.
A pointer may target Nothing; a reference must have a valid referent.

## Projection

A **projection** creates an access path to part of a value, such as member access
through a container.

A projection derives capabilities from the path used to reach the containing
value. Complete projection behavior belongs with the relevant language concept.

## Receiver operand

The **receiver operand** is the implicit operand representing the instance on
which a type-defined function or operator acts.

Use the complete term where context permits. "Receiver" alone has other
meanings, and "receiver object" incorrectly suggests an object-oriented model.

All three qualifier axes may constrain a receiver operand. See
[Zax qualifiers](qualifiers.md#receiver-operands).

## Reconstructive replacement

**Reconstructive replacement** is the compiler-recognized generated `=`
transition that ends one immutable value lifetime and establishes another in the
same storage.

The compiler owns the complete lifecycle skeleton. It selects a replacement
constructor when a viable customization exists; otherwise it uses the generated
fallback of enclosing `---` followed by ordinary `+++`.

It requires an immutable value in a varying place through a writable access
path. See
[Zax construction, replacement, and destruction](construction-and-destruction.md#reconstructive-replacement)
and [Zax qualifiers](qualifiers.md#reconstructive-replacement).

## Replacement constructor

A **replacement constructor** is an optional type-defined customization selected
within the compiler-owned reconstructive-replacement operation.

It is written with contextual `replacement +++`. When selected, it runs instead
of the generated fallback's enclosing `---` followed by ordinary `+++` and may
recycle the previous representation and resources while establishing the
complete replacement instance. Complete behavior is defined by
[Zax construction, replacement, and destruction](construction-and-destruction.md#custom-replacement).
The qualification boundary is defined by
[Zax qualifiers](qualifiers.md#reconstructive-replacement).

## Referent

A **referent** is the place or value reached through a reference.

The binding storing a reference and its referent are distinct qualification
layers.

## Type use

A **type use** is an occurrence of a type in a declaration or expression
context, as distinct from the type's definition.

A type use may carry contextual qualification without changing the underlying
type definition.

## Value

A **value** is data together with the language-level meaning of its type. A value
may occupy a place, be copied into a new place, be moved, or be observed through
multiple access paths.

Value mutation, place replacement, and access capability are separate concerns.
See [Zax qualifiers](qualifiers.md).

## Value lifetime

A **value lifetime** is the period during which one particular value exists in a
place.

Construction establishes a value lifetime. Destruction or reconstructive
replacement ends it. Contained member lifetimes may begin or end at different
points from the complete enclosing lifetime. See
[Zax construction, replacement, and destruction](construction-and-destruction.md#mental-model).
