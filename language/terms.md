# Zax language-design terms

| Field | Value |
| --- | --- |
| Status | Current conceptual terminology |
| Audience | Human developers reading, writing, discussing, or documenting Zax |
| Applies To | Cross-cutting human-language terms used by current conceptual design |
| Implementation State | Not applicable |
| Owns | Concise meanings and usage boundaries for cross-cutting descriptive terms |
| Does Not Own | Keyword catalogs or the complete behavior owned by the concept documents linked here |

## Purpose

This document defines the human-language terms used to discuss Zax consistently.
It is not a list of reserved words or operators. A term may describe a concept
without being available as source syntax.

Behavior remains with its concept owner. This page gives readers enough meaning
to follow documentation and links to the relevant owner when the distinction
affects programming behavior.

## Active execution environment

The **active execution environment** is the environment whose execution
semantics currently govern an operation. It may be the generated target or the
compiler host during applicable build-time execution. An unqualified
environment-relative scalar name selects against this environment. See
[Zax integers](integers.md#native-representation-and-software-emulation).

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

## Compiler host

The **compiler host** is the environment running the compiler and applicable
build-time code. It is distinct from the generated target even when both happen
to use the same CPU model or representation. See
[Zax integers](integers.md#native-representation-and-software-emulation).

## Circumfix operator

A **circumfix operator** is one complete operation with a language-recognized
opening component, closing component, and enclosed operand:

```zax
|value|
```

The opening and closing symbols are not independent pre/post unary operations.
Recognized forms and built-in availability are defined by the
[operator catalog](operator-catalog.md#circumfix-forms).

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

## Keyword-role conflict

A **keyword-role conflict** occurs when a word is grammatically eligible as a
non-operator keyword while the programmer intends it as an identifier or
operator-phrase component.

This is a lexical-role question rather than unsafe behavior or operator
overload ambiguity. Words inside a declared or fenced operator phrase carry
phrase roles instead of keyword roles; see
[Zax operator phrases](operator-phrases.md#keyword-words-in-phrase-roles).

## Current instance

The **current instance** is the instance whose type-defined function body is
executing.

Use this term when discussing the body independent of invocation or overload
selection. At a call or operator-selection boundary, use **receiver operand**.
Do not use "receiver object"; Zax is not object-oriented.

## Deliberate intent or layout error

A **deliberate intent or layout error** is the umbrella category for a bounded,
deliberately selected source pattern whose presentation conflicts with parsed
structure or whose natural form is gated because it strongly resembles a likely
mistake.

It is distinct from ordinary syntax rejection and semantic error. Source-layout
cases are owned by [Zax source structure](source-structure.md); flow-shape cases
by [Zax core flow control](core-flow-control.md).

## Confusable-form intent error

A **confusable-form intent error** rejects natural source for a coherent
recognized form because it overwhelmingly resembles a damaged neighboring form
and cannot be clarified through ordinary whitespace or grouping.

```zax
||value| // candidate asymmetric circumfix resembles a missing final |
```

Unlike an operator-attachment error, the intended form needs an explicit future
interpretation mechanism rather than ordinary punctuation repair.

## Layout-intent error

A **layout-intent error** occurs when indentation, brace placement, clause
attachment, or physical-line position contradicts the parsed source structure.

The remedy is to correct presentation or explicit structure. It does not require
a keyword-neutral or confusable-form escape.

## Direct storage

**Direct storage** means that a value occupies the storage place declared for
it, rather than being reached through a pointer or reference.

For a directly stored value, the declared binding and value share one immediate
replacement boundary. Indirection introduces additional boundaries.

## Eager operator

An **eager operator** evaluates every runtime operand hole required by the
selected operation exactly once in visible source order before entering its
body.

This does not evaluate both arms of a conditional expression. Non-Boolean
overloads of symbolic logical operators are ordinary eager operations.

## Exact value

An **exact value** is the same mathematical or semantic value requested by an
operation, without rounding, truncation, wrapping, saturation, substitution, or
discarded information.

An exact-width integer instead describes a type whose logical width is fixed.
An exact conversion preserves the source value. An exact arithmetic result is
the mathematical result without another policy transformation.

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

## Identity boundary

An **identity boundary** separates types that are not interchangeable even when
their representation is the same. Assignment, argument passing, return, and
operator matching cross it only through declared projection, admission, or a
direct bridge. See
[Zax identity types](identity-types.md#identity-boundary-and-underlying-type).

## Identity type

An **identity type** is a distinct outer type based on an existing underlying
type. Its declaration explicitly chooses admission and an exposed or opaque
underlying surface. See [Zax identity types](identity-types.md).

## Lifecycle operation

A **lifecycle operation** establishes, transitions, or ends a value or member
lifetime.

Ordinary `+++`, contextual `replacement +++`, `---`, and the compiler-owned
generated replacement skeleton participate in lifecycle operations. An
arbitrary operator is not a lifecycle operation merely because it uses `=`.
See [Zax construction, replacement, and destruction](construction-and-destruction.md).

## Mixfix operator

A **mixfix operator** is one complete overloadable operation selected from an
expression-tree pattern containing several recognized operator components and
operand holes.

```zax
a[b] = c
```

A direct mixfix may consume the index and assignment nodes as one operation.
Complete behavior is defined by [Zax mixfix operators](mixfix-operators.md).

## Mixfix-consumption barrier

A **mixfix-consumption barrier** is a resolved source node that a user mixfix may
accept as a completed operand-hole value but may not consume as one of its key
components.

Exact Boolean short-circuit operations are barriers because consumption would
erase their conditional runtime evaluation. Compiler-owned lifecycle and flow
boundaries may be barriers or noncomponents for their own reasons. This differs
from signature protection.

## Native endianness

**Native endianness** is the active byte order of the environment whose execution
semantics are currently in effect.

It is distinct from **compiler-host endianness**, the byte order of the
environment running the compiler, from **target endianness**, the byte order
selected for generated target behavior, and from the absolute orders **big
endian** and **little endian**. See [Zax endianness](endianness.md).

## Native representation

A scalar type has a **native representation** when the applicable CPU provider
classifies that width as a native scalar representation. This does not promise
that every operation has a direct instruction or one uniform cost. See
[Zax integers](integers.md#native-representation-and-software-emulation).

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

## Operand hole

An **operand hole** is one expression position in a mixfix tree skeleton. Source
at that position resolves as a complete ordinary expression and supplies one
mixfix input.

See [Zax mixfix operators](mixfix-operators.md#trees-components-and-holes).

## Operator component

An **operator component** is one recognized pre-unary, post-unary, binary,
circumfix, call, index, or phrase form fixed inside a mixfix tree skeleton.

Consumed components do not execute independently when the mixfix is selected.
See [Zax mixfix operators](mixfix-operators.md).

## Operator form

An **operator form** is the general category for a language-recognized symbolic,
phrase, circumfix, call/index, or mixfix operation when the exact source category
does not matter.

## Operator phrase

An **operator phrase** is an operator form written as one or more exact words
rather than a punctuation-only symbol:

```zax
ready := left logical nand right
converted := source as DestinationType
```

A phrase is not a second dispatch mechanism; it uses the shared callable and
operator model. See [Zax operator phrases](operator-phrases.md),
[Zax operators](operators.md#operator-phrases), and the
[operator catalog](operator-catalog.md#operator-forms).

## Phrase word

A **phrase word** is one canonical word in an operator phrase. It matches
`[a-z][a-z0-9]*`: an ASCII lower-case letter followed by ASCII lower-case letters
or digits.

The complete word and presentation rules are owned by
[Zax operator phrases](operator-phrases.md#exact-finite-forms).

## Phrase component

A **phrase component** is one contiguous exact word sequence participating as an
operator or as one component of a
[mixfix](mixfix-operators.md#phrase-components) skeleton.

A component carries exactly one fixity: pre-unary, post-unary, or binary.

## Phrase fence

A **phrase fence** is explicit local source requiring one exact phrase component:

```zax
foo 'bar'
```

It eliminates identifier, keyword, literal, and differently worded phrase
interpretations for the fenced words. It does not group, add a precedence
boundary, or select an implementation. See
[Zax operator phrases](operator-phrases.md#exact-phrase-fencing).

## Reserved phrase form

A **reserved phrase form** is an exact operator phrase that user code cannot
declare. Reserving a form holds it for the language even when some of its domain
behavior remains future work.

The current reserved set is listed by the
[operator catalog](operator-catalog.md#reserved-phrase-forms).

## Operator-attachment intent error

An **operator-attachment intent error** occurs when whitespace, adjacency, or
grouping presents another fixity or number of operator applications than the
programmer intends.

```zax
!!value   // cannot mean two independent ! applications
!(!value) // corrected through grouping
```

It is repaired through ordinary source presentation, unlike a confusable-form
intent error.

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

An **identity projection** is a separate qualified use: an explicit conversion
from an identity value to its immediate underlying type. It does not mean member
access and need not create an access path. Documentation uses the qualified term
when that distinction matters; see
[Zax identity types](identity-types.md#identity-projection).

## Protected intrinsic signature

A **protected intrinsic signature** is an exact operator signature whose every
operand belongs to a closed intrinsic family. User declarations cannot replace
it or claim a currently unavailable all-intrinsic combination.

Protection preserves predictable primitive behavior and future language
evolution. A mixed signature containing a custom operand remains extensible.
See [Zax operators](operators.md#protected-intrinsic-domains).

## Receiver anchor

A **receiver anchor** is the operand hole whose resolved type owns and supplies
type-defined discovery for a mixfix declaration.

Other holes affect viability but do not contribute type-defined mixfix
declarations. See [Zax mixfix operators](mixfix-operators.md#receiver-anchor).

## Receiver operand

The **receiver operand** is the implicit operand representing the instance on
which a type-defined function or operator acts.

Use the complete term where context permits. "Receiver" alone has other
meanings, and "receiver object" incorrectly suggests an object-oriented model.

All three qualifier axes may constrain a receiver operand. See
[Zax qualifiers](qualifiers.md#receiver-operands).

## Redundant-structure intent error

A **redundant-structure intent error** occurs when source supplies two competing
mechanisms for one structural role, such as explicit `\` where an open delimiter
or recognized separator already continues the newline.

The remedy is to remove the redundant marker. This is distinct from harmless
explicit source that merely repeats information.

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

## Sealed type

A **sealed type** cannot receive ordinary externally added members or
operations. A separately authorized partial mechanism may still permit narrowly
classified additions without changing stored shape. See
[Zax integers](integers.md#exact-intrinsic-family).

## Short-circuit operator

A **short-circuit operator** has language-defined evaluation in which an earlier
operand result may determine that a later runtime operand is not evaluated.

The skipped operand still resolves and validates at compile time. Short-circuit
behavior is not general laziness: the language fixes which operand may be
skipped, and a user overload does not receive an arbitrary unevaluated
expression.

## Software emulation

**Software emulation** implements a scalar operation through several native
operations or automatically supplied helper behavior when the CPU does not
natively represent or directly operate on the requested width.

A **software fallback** is a type-selection result that uses software emulation
because no native candidate satisfies the request. See
[Zax integers](integers.md#native-representation-and-software-emulation).

## Symbolic operator

A **symbolic operator** is an operator whose recognized source form is written
primarily with punctuation, such as `+`, `&~`, or `<<%`.

The language owns the closed symbolic catalog and precedence. Declarations may
overload permitted forms but cannot invent arbitrary punctuation. See the
[operator catalog](operator-catalog.md).

## Type argument

A **type argument** is the concrete type identity supplied to a
[type parameter slot](#type-parameter-slot):

```zax
converted := source as DestinationType
```

A type argument has no runtime storage, lifetime, or evaluation. See
[Zax function invocation](function-invocation.md#type-parameter-slots).

## Target environment

The **target environment** is the environment selected for generated program
behavior. It is distinct from the compiler host and may be named explicitly by
environment-relative type paths. See
[Zax integers](integers.md#native-representation-and-software-emulation).

## Transparent alias

A **transparent alias** is another source name for the same canonical type
identity. It creates no conversion or overload domain. See
[Zax identity types](identity-types.md#transparent-aliases).

## Type parameter slot

A **type parameter slot** is a prototype input completed by one concrete type
identity rather than a runtime value:

```zax
DestinationType : type
```

It occupies its ordered slot for viability, preference, and diagnostics like any
other parameter. See
[Zax declarations and bindings](declarations-and-bindings.md#type-parameter-slots-and-type-arguments).

## Type-receiver operator

A **type-receiver operator** is an operation discovered through a concrete type
identity rather than through an instance:

```zax
BackingType := EnumType underlying type
```

It is declared with `operator type`, has no receiver instance, and is not
inherently compile-time. See
[Zax declarations and bindings](declarations-and-bindings.md#type-receiver-operators).

## Type use

A **type use** is an occurrence of a type in a declaration or expression
context, as distinct from the type's definition.

A type use may carry contextual qualification without changing the underlying
type definition.

## Underlying type and value

An identity type's **underlying type** is the existing type on which its
representation begins. The **underlying value** is the value used to create the
identity or returned by identity projection. See
[Zax identity types](identity-types.md#identity-boundary-and-underlying-type).

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
