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

## Call completion

**Call completion** is the synchronous boundary after the selected callable has
bound all parameters, executed its body, completed its declared results, and
mapped those results into the surrounding context.

An argument or result temporary required by the call remains live through the
part of this boundary that uses it. Async suspension requires a broader future
completion model. See [Zax function invocation](function-invocation.md).

## Callable

A **callable** is a function value, declared function, constructor, operator, or
other language-defined entity that has a prototype and may be selected and
invoked.

The callable category determines discovery and any category-specific behavior.
Ordinary argument binding, results, and fixed-arity selection are defined by
[Zax function invocation](function-invocation.md).

## Construction packet

A **construction packet** is the `[{ ... }]` construct that supplies positional
constructor arguments, named constructor arguments, and direct stored-member
initializers.

Packet order controls input evaluation and binding. It does not reorder member
construction. See
[Zax construction, replacement, and destruction](construction-and-destruction.md#construction-packets).

## Contextual keyword

A **contextual keyword** is a spelling that has keyword status only where that
keyword's construct is grammatically permitted. Outside those positions, the same
spelling is not a keyword and may name an ordinary entity.

The term describes recognition position, not a reserved-word list. See
[Zax source structure](source-structure.md#contextual-keyword-recognition).

## Current instance

The **current instance** is the instance whose type-defined function body is
executing.

Use this term when discussing the body independent of invocation or overload
selection. At a call or operator-selection boundary, use **receiver operand**.
Do not use "receiver object"; Zax is not object-oriented.

## Deliberate intent or layout error

A **deliberate intent or layout error** rejects a bounded, deliberately selected
near-miss pattern whose tokens may parse or resemble another valid form while
presenting contradictory programmer intent.

It is distinct from ordinary syntax rejection and semantic error. Source-layout
cases are owned by [Zax source structure](source-structure.md); flow-shape cases
by [Zax core flow control](core-flow-control.md).

## Direct storage

**Direct storage** means that a value occupies the storage place declared for
it, rather than being reached through a pointer or reference.

For a directly stored value, the declared binding and value share one immediate
replacement boundary. Indirection introduces additional boundaries.

## Flow label

A **flow label** is a separately shaped name category that names a complete flow
statement so that transfers eligible for that construct may target it.

A flow label is not an ordinary identifier and does not share ordinary-identifier
lookup. See
[Zax core flow control](core-flow-control.md#flow-labels-and-transfer-targets).

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

## Normal completion

**Normal completion** is relative to the construct being discussed. A body
normally completes when it reaches its end without an abrupt transfer or panic;
that completion runs any applicable post operation and satisfies required state.

`break`, `continue`, `next`, and `return` are abrupt relative to enclosing flow
bodies. An explicit `return` may still be a normal function exit, as opposed to
panic or non-completion, and must satisfy the function's result obligations. See
[Zax core flow control](core-flow-control.md),
[Zax function invocation](function-invocation.md), and
[Zax construction, replacement, and destruction](construction-and-destruction.md).

## Ordinary syntax rejection

**Ordinary syntax rejection** means source that matches no legal production. The
parser reports the legal token or construct expected at that point.

The language does not enumerate every unrelated thing a programmer could write,
and this is distinct from a semantic error or a deliberate intent or layout
error.

## Parameter slot

A **parameter slot** is one ordered input position declared by a callable
prototype.

Positional, named, omitted, or routed source input may bind the slot. The slot is
complete before the callable body begins. See
[Zax function invocation](function-invocation.md).

## Place

A **place** is storage that can hold a value. Replacing a place changes which
value that storage holds.

Place replacement is distinct from changing the contents of the value already
stored there. See [Zax qualifiers](qualifiers.md).

## Pointee

A **pointee** is the place or value targeted by a pointer.

The binding storing a pointer and its pointee are distinct qualification layers.
A pointer may target Nothing; a reference must have a valid referent.

## Post operation

A **post operation** is the construct-specific flow-header section that runs on
normal completion, and on a `next` transfer, while header bindings remain alive.

`break`, `continue`, and `return` skip it. It is ordinary work, not a destructor
or guaranteed cleanup hook. See [Zax core flow control](core-flow-control.md).

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

## Result-routing group

A **result-routing group** evaluates one producer, then maps or discards its
ordered results through explicit source-result and destination cursors.

The producer executes once. See
[Zax function invocation](function-invocation.md#result-routing-groups).

## Result shape

A **result shape** is the ordered sequence of zero or more result slots declared
by a callable prototype.

Several results are not implicitly one tuple or structural value. See
[Zax function invocation](function-invocation.md#result-slots).

## Result slot

A **result slot** is one ordered output position declared by a callable
prototype.

The slot begins as an output obligation and must contain one complete value on
every normal exit. A result initializer may opt into construction before body
entry. See [Zax function invocation](function-invocation.md#result-slots).

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

## Semantic error

A **semantic error** is source that parses successfully but violates a type,
target, name-resolution, lifetime, or completion rule.

It is distinct from ordinary syntax rejection and from a deliberate intent or
layout error. The violated rule and its behavior remain with the applicable
concept owner.

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
