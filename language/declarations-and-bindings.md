# Zax declarations and bindings

| Field | Value |
| --- | --- |
| Status | Current conceptual design |
| Audience | Human developers reading, writing, or evaluating Zax |
| Applies To | Programmer-facing declaration, binding, initialization, name-resolution, and assignment boundaries; not a formal grammar or specification |
| Implementation State | Not established by this repository |
| Owns | Value declaration forms; default, direct, inferred, and explicitly bypassed initialization; binding visibility; redeclaration and shadow permission; one lexical identifier namespace; qualified-path resolution through incomplete declarations; explicit instance-member lookup; declaration-facing qualifier axes and attachment; the declaration-versus-assignment boundary; the general non-value definition family; named type self-reference and `forward` at the depth required by declarations; declaration diagnostics and formatting |
| Does Not Own | Complete inference, function and capture semantics, operator resolution, replacement-constructor internals, constructors and lifetimes, move/copy and ownership, complete [qualifier behavior](qualifiers.md), anonymous recursive type syntax, flow-control grammar, import/module behavior, type identity and layout, formal grammar, diagnostic identifiers, or compiler and tooling implementation |

## Mental model

Zax makes name introduction visible.

```zax
x = 1
y := 1
```

`x = 1` uses an existing destination. It does not create `x`.

`y := 1` declares `y`, infers its type from `1`, and initializes it.

This distinction separates three concepts:

- A **declaration** introduces a name and determines what kind of entity it
  denotes.
- **Initialization** establishes the first state of a declared value.
- **Assignment** invokes an operator using an already declared destination.

An unresolved assignment target is therefore an error, not an implicit local
declaration.

## Core value forms

| Form | Meaning |
| --- | --- |
| `name : Type` | Declare `name` with an explicit type and perform the type's default initialization. |
| `name : Type = value` | Declare `name` with an explicit type and initialize or construct it directly from `value`. |
| `name := value` | Declare `name`, infer its type from `value`, and initialize it. |
| `name = value` | Invoke an assignment operator using an already declared destination. |
| `name : Type = unsafe ???` | Declare live storage while explicitly and unsafely bypassing ordinary value initialization and construction. |

### Inferred declarations

`:=` is the adjacency of declaration and initialization:

```zax
x := 0
y : = 0
```

Both forms declare an inferred `Integer` and initialize it from `0`. A formatter
uses the compact `:=` spelling consistently.

The compiler may optimize storage reservation and initialization when doing so
preserves behavior. That does not collapse the programmer-visible distinction
between introducing a name and initializing its value.

A declaration must provide either an explicit type or an initializer from which
the type can be inferred:

```zax
i: // error: no explicit or inferable type
```

Inference uses the initializer and its immediate context. It does not scan later
statements to discover intended use:

```zax
value := makeValue()
```

Some type shape may be explicit while the remaining type is inferred:

```zax
reference : & = original
pointer : * = original
```

The complete inference algorithm and exact set of partially explicit qualifiers
remain later design.

### Direct initialization

An initialized typed declaration performs direct initialization:

```zax
item : Item = source
```

It does not default-initialize `item` and then invoke later assignment. That
alternative could introduce an extra constructor, destructor, allocation, or
other visible operation.

Initializer and constructor selection remain type and construction concerns.
The declaration guarantees only that `item` is introduced and initialized
directly from the supplied source.

## Default initialization

Every ordinary value declaration initializes its value:

```zax
count : Integer
item : Item
```

Default initialization may initialize contained values, execute constructors,
allocate storage, or perform other visible work. Constructors and destructors do
not report errors through return results, and Zax does not use exceptions.
Initialization may nevertheless encounter a panic such as allocation failure or
stack exhaustion.

A declaration followed by assignment performs two operations:

```zax
x : Integer; x = 1
```

This default-initializes `x` and then invokes assignment. It is equivalent in
those respects to:

```zax
x : Integer
x = 1
```

It is not an initialize-later form.

### Explicitly uninitialized storage

`unsafe ???` is an explicit escape from ordinary initialization:

```zax
item : Item = unsafe ???
```

This form:

1. reserves storage;
2. establishes a live variable of the declared type;
3. skips ordinary value initialization and constructor execution;
4. leaves its representation indeterminate;
5. schedules the ordinary destructor at scope exit; and
6. makes the programmer responsible for establishing every invariant required
   by later use and destruction.

It supports low-level initialization:

```zax
if condition {
    item : Item = unsafe ???
    initializeThroughAssembly(item)
} // Item's destructor runs
```

The low-level operation must leave `item` in a state accepted by every later
operation and by its destructor. Failure to do so is unsafe programmer behavior.

The compiler should diagnose obvious misuse when practical. Complete
definite-initialization analysis, low-level initialization contracts, relocation,
return-value optimization, partial construction, and destruction-path analysis
remain later safety and lifetime work.

## Scope entry and redeclaration

An ordinary value binding enters scope only after initialization completes. The
new name does not participate in lookup while its type and initializer are being
evaluated:

```zax
x := use(x) // error if no outer x exists
```

If an outer `x` exists and permits shadowing, the right-hand `x` resolves to that
outer binding. The new inner `x` becomes visible afterward.

Same-scope redeclaration is always an error:

```zax
x : Integer
x := 1 // error: x is already declared in this scope
```

Semicolon composition does not introduce another scope:

```zax
x := 1; x := 2 // error: same-scope redeclaration
```

Earlier operands may introduce names for later operands:

```zax
x := makeValue(); y := use(x)
```

The source formation, body, and clause-scope rules that make this possible are
defined by [Zax source structure](source-structure.md).

### Deferred function bodies

A function body is delayed code rather than an expression executed while the
function value is initialized. It may refer to its own completed binding:

```zax
factorial final : (result : Integer)(n : Integer) = {
    return n <= 1 ?? 1 ;; n * factorial(n - 1)
}
```

This is a recursive binding for a delayed body, not permission for an ordinary
initializer to self-reference. Ordinary surrounding captures are discovered
lexically and materialized when the function value is constructed. A function's
self-reference cannot be an ordinary by-value copy of a function value that does
not yet exist.

Exact capture representation, recursive-function `forward` requirements,
mutually recursive functions, and behavior after reassigning a non-`final`
recursive function remain later function design.

### Default function values

A function value declared without another initializer still receives default
initialization:

```zax
callback : ()()
```

Its default value is the function type's `Nothing` state. The compiler should
diagnose an invocation it can prove still targets that default state. An
otherwise unhandled invocation panics rather than manufacturing arbitrary
results or executing undefined code.

Future explicit behavior may permit selected function types to request a no-op
or default-result `Nothing` implementation. Exact directives and complete
`Nothing` semantics remain later work.

## Shadowing

Nested shadowing is prohibited by default. The outer declaration that may be
hidden must grant permission:

```zax
value shadowable := 1

if condition {
    value := 2
}
```

`shadowable` belongs to the declaration that is willing to be hidden. It is not
a marker placed on the inner declaration after the conflict is found.

The newly introduced inner declaration is not automatically `shadowable`:

```zax
value shadowable := 1

if outer {
    value := 2

    if inner {
        value := 3 // error: the middle value is not shadowable
    }
}
```

Permission can be propagated deliberately:

```zax
value shadowable := 1

if outer {
    value shadowable := 2

    if inner {
        value := 3
    }
}
```

The rule applies to ordinary lexical declarations that can be shadowed,
including values, types, aliases, imports, and namespaces. A forward declaration
and its completion are one declaration rather than shadowing. Overload groups
and labels require category-specific rules in their future owners.

Once a name is shadowed, ordinary lookup no longer reaches it. Zax does not
provide a general parent-scope or `..` lookup operator. A programmer who needs
both entities can establish an alias before shadowing.

## Instance members

Instance members are not injected into an instance function's ordinary lexical
lookup. Access through the current instance uses `_` explicitly:

```zax
value := 0

value = 1
_.value = 2
_.foobar = 3
foobar = 4 // error: no local foobar is declared
```

The same principle applies to instance member functions. Static or shared
members use their appropriate qualified form.

Explicit access prevents copied code, a newly added member, or a forgotten local
declaration from silently changing name resolution.

## Identifier namespace and paths

Each lexical scope has one ordinary identifier namespace. This prevents
context-dependent type-versus-value lookup and makes collisions visible.

Structured cases remain distinguishable:

- declarations may form one overload set when overload rules permit it;
- a forward declaration and its completion are one declaration;
- members occupy their containing entity's member scope;
- operators are not ordinary identifiers; and
- labels may require control-flow-specific treatment.

### Naming intent

Type-like and value-like declarations use mandatory capitalization and casing
intent:

```zax
myType :: type { } // error: type-like declaration uses a value-like name
MyValue := 0       // error: value declaration uses a type-like name
```

The compiler enforces mechanically knowable intent. Lints may advise on acronyms,
discouraged but legal shapes, or other conventions. Editors and spelling tools
may help with dictionary and domain terminology that the compiler cannot decide.

### Qualified paths

A qualified path resolves deterministically:

```zax
value : Node.Node
```

The first component is resolved lexically once. Each remaining component is
resolved only inside the entity selected by the preceding component. A missing
later component does not make lookup backtrack to another outer candidate for
the first component.

Aliases and stable roots preserve access when another name would be hidden:

```zax
OuterNode :: alias type Node
value : OuterNode.Node
```

```zax
value : Module.Node.Node
```

Imports can similarly use aliases. A path does not excuse a collision in its
root name.

### Pending path suffixes

A suffix may remain pending when an already resolved prefix names an incomplete
declaration:

```zax
MyType :: forward type

value : MyType.SubType
```

`MyType` resolves immediately to that forwarded declaration and remains fixed.
If completion of `MyType` later discovers `SubType`, the pending suffix resolves
inside the completed declaration. If `SubType` never appears, the path is
unresolved when the dependency graph must be finalized. The compiler does not
retry another outer `MyType`.

An explicit nested forward may categorize a needed member earlier:

```zax
MyType :: forward type
MyType.SubType :: forward type
```

It is not required merely because the containing type was incomplete when the
path was encountered. Operations requiring completed size, layout, overload
sets, or initialization behavior remain pending until enough information is
available.

This is dependency-directed later resolution, not speculative rebinding.

## Qualifier axes

Complete programmer-facing qualifier behavior is defined by
[Zax qualifiers](qualifiers.md). This section establishes the declaration-facing
axes and attachment boundaries.

Zax distinguishes three independent concerns:

| Pair | Governs |
| --- | --- |
| `final` / `varying` | Whether a storage place may be replaced |
| `mutable` / `immutable` | Whether the underlying value may ever change |
| `writable` / `readonly` | Whether the current access path may perform an otherwise permitted content mutation or place replacement |

In short:

- mutable means the underlying value is permitted to change;
- immutable means the underlying value is guaranteed not to change;
- writable means this access path may change the value; and
- readonly means this access path may observe but not change the value.

Mutable means someone may change the value. Writable means this access may
change it. Readonly means this access may not change it. Immutable means no
access may change it.

`readonly` identifies the access-path promise without conflating it with
immutable state or compile-time constant evaluation.

### `final` and `varying`

`final` and `varying` can qualify a declared variable and a place exposed through
a type use. They apply to the immediately qualified storage place:

A directly stored value and its variable occupy one replacement boundary.
Compatible qualifiers on that boundary combine idempotently:

```zax
foo final : Foo final = makeFoo()
foo varying : Foo varying = makeFoo()
```

Conflicting explicit qualifiers are errors:

```zax
foo final : Foo varying = makeFoo() // error
foo varying : Foo final = makeFoo() // error
```

An omitted qualifier provides the default only when no explicit qualifier
establishes another stance:

```zax
foo : Foo         // varying by default
foo : Foo final   // final
foo final : Foo   // final
```

Compatible repetition is legal. This supports aliases, generic substitution,
generated declarations, and deliberate explicitness. A linter may report visible
redundancy.

A `final` place is nonreplaceable, not immutable:

```zax
foo final : Bar = makeBar()

foo = other       // ordinary replacement is unavailable
foo.member = 200  // legal when Bar and this access are mutable and writable
```

Place truth survives references and captures:

```zax
source final : Foo = makeFoo()
stableView : Foo final & = source
```

Normal replacement through `stableView` remains unavailable because the
referenced place is final. A qualifier on the reference variable itself cannot
strip that referent qualification.

An alias to a varying place preserves that stance:

```zax
source varying : Foo = makeFoo()
trackingView : Foo varying & = source
```

A final view of that same referent would be false:

```zax
falseView : Foo final & = source // error: source's place is varying
```

Reference bindings and their referents have independent places:

```zax
view final : Foo varying & = source
```

The name-side `final` applies to `view`'s reference binding. The type-side
`varying` describes the referent.

An immutable reference tracking a varying place must spell `varying`
explicitly. Omission must not silently introduce a path that may observe
successive immutable lifetimes.

Exact reference rebinding, assignment, and qualifier syntax remain later
reference design. An ordinary alias preserves the source place's actual stance.

Copying by value creates a new place with its own declaration stance.

### `mutable`, `immutable`, `writable`, and `readonly`

A mutable value can be viewed through readonly access:

```zax
value : Foo mutable

writer : Foo writable & = value
reader : Foo readonly & = value
```

`reader` cannot modify the value, but `writer` or another permitted alias may.

An immutable value can provide readonly access:

```zax
value : Foo immutable
reader : Foo readonly & = value
```

No access may modify the underlying value. An immutable value cannot provide a
writable view.

The baseline unqualified access default is writable and may be changed for an
applicable source context. Explicit `writable` remains useful where a positive
capability must be stated, selected, reflected, or restored. Defaults fill only
unresolved axes and never override qualifications supplied by a source,
referent, or resolved type. Exact default-directive syntax is later design.

### Capability conversions

An ordinary conversion may remove permissions or request weaker access:

- immutable values may provide readonly access;
- writable access may be viewed as readonly;
- readonly access may not become writable; and
- mutable or readonly values do not thereby become immutable.

Final/varying is preserved for an alias of the same place. A new by-value
destination, constructed result, copy, or move destination resolves its own
independent place stance.

The complete cast lattice, unsafe conversions, reference projection, and capture
projection remain later design. They must preserve the constraints established
by [Zax qualifiers](qualifiers.md).

### Move, copy, and future capabilities

Move and copy are contextual transfer operations rather than persistent
qualifications of an entire value. Their availability depends on the complete
source, destination, qualifier, policy, and overload context. Ordinary reference
passing aliases a value and is neither a move nor a copy.

This design does not classify broader capabilities such as sendability,
synchronization safety, pinning, known layout, copying, or interior freezing.
Future reviews may represent them as qualifiers, selectable operations,
reflection, concepts, compiler-recognized guarantees, type-definition contracts,
or combinations. The declaration model neither requires nor precludes those
choices.

## Assignment and overload selection

An overloadable operator is an ordinary selectable function. User-defined
operators may have domain-specific effects and results that are unusual outside
their domain.

For:

```zax
consume(a = 1)
```

the compiler:

1. resolves `a`;
2. selects a viable `=` candidate for the qualified operands;
3. invokes it; and
4. maps its result to `consume` if the result shape is accepted.

No operator can introduce an unresolved operand as a declaration.

Generated operators participate in ordinary candidate selection with qualifier
requirements. The compiler-recognized reconstructive `=` scenario requires both
a varying destination and a writable path. It is unavailable for a final place
or through readonly access.

The reconstructive candidate has a compiler-owned lifetime skeleton and may
select a contextual
[`replacement +++` constructor](qualifiers.md#reconstructive-replacement).
User-defined code does not replace that skeleton with an ordinary `=` body.

A domain-specific `=` candidate that accepts a final or readonly left operand
may remain selectable because the token itself is not assigned conventional
meaning in every scenario:

```zax
foo final : Bar = makeBar()
result := foo = other // requires a matching non-replacement candidate
```

Compound assignment, increment, decrement, and other operators likewise require
already declared operands and use ordinary selection.

Declaration initialization remains reserved:

```zax
foo : Bar = source
```

The `:` introduces `foo`; user code cannot overload that act. The initializer
selects construction or initialization behavior for the new `Bar`. Later
`foo = source` performs operator selection against an existing destination.

Exact built-in results, expression value categories, overload ranking,
conversion, reconstructive-candidate priority, and generated operator sets
remain later operator design.

## Non-value definitions

`::` introduces a family of non-value definition forms:

```zax
Point :: type { }
Fruit :: enum { }
FriendlyName :: alias type ExistingType
ModuleName :: import Module.Definition
TypeName :: forward type
```

The family does not imply one runtime behavior. In particular, `::` does not mean
"evaluated at build time." Build-time versus runtime evaluation depends on
functions, inputs, directives, and evaluation context. Build-time values use
ordinary `:` or `:=` bindings.

A named type becomes visible as an incomplete type before its body is resolved:

```zax
Node :: type {
    next : Node *
}
```

The self-name resolves and pointer representation is finite. Operations
requiring the completed size, layout, or member set remain pending until the
definition completes.

Functions and ordinary values use `:`, including values with anonymous types:

```zax
callback final : FunctionType = { }
value final : :: type {
    member : Integer
}
```

### `forward`

`forward` introduces a name before its complete declaration is otherwise
encountered. A named type does not need `forward` merely to refer to its own name
inside its body.

Forward declaration remains useful for out-of-order and mutually recursive
names:

```zax
OtherNode :: forward type

Node :: type {
    next : OtherNode *
}

OtherNode :: type {
    next : Node *
}
```

The later definition completes `OtherNode`.

Direct infinitely recursive layout remains an error:

```zax
Node :: type {
    next : Node // error: infinitely recursive layout
}
```

Mutually recursive named types can forward each required name.

### Anonymous recursive type syntax

Named types use their own incomplete names for self-reference. Anonymous
recursive type syntax is not established by this design.

Complete forward categories, anonymous recursive types, recursive-type identity,
and dependency algorithms remain later type and name-resolution work.

## Declaration contexts

The binding model applies beyond local statements. Parameters, results,
captures, stored members, type names, imports, and flow headers provide their own
declaration contexts. They share name, type, initialization, and qualifier
concepts without all becoming statements.

### Flow-control initialization

A flow-control initializer accepts any effective statement. A binding introduced
there is visible to the condition, applicable clause bodies, and corresponding
false or `else` clauses, then leaves scope after the complete flow statement:

```zax
if result := tryValue() ;; result.isValid()
    use(result)
```

An explicit block inside a composed initializer keeps its own nested scope:

```zax
if {
    token := hello()
    goodbye(token)
}; value := compounding() ;; value < 1 {
    use(value)
    token = 5 // error: token belonged to the completed inner block
}
```

Exact flow-header grammar, `;;`, and mandatory layout remain later flow-control
and source-structure work.

### Stored members

A stored-member initializer runs once per actual containing instance. A storage
qualifier such as `once` may change how many storage instances exist and when the
one instance is initialized. It does not make an ordinary member initializer run
once merely because it appears once in a type definition.

### Default parameters

Default parameter expressions remain later function design. The declaration
constraint is that an explicit argument or the default expression initializes
the parameter, not both. This document does not reserve `=` or another spelling
for that future behavior.

### Multiple results

Contextual multiple-result declarations may share one initializer:

```zax
first:, second: = produceTwo()
```

Each `name:` declares a binding. Result mapping must be unambiguous, and
duplicate names remain errors. Optional and discarded results, construction
order, and cleanup after panic remain function and lifetime work.

## Documentation attachment

Documentation attaches to the complete declaration construct:

```zax
/// Describes x.
x := makeValue()
```

```zax
/// Describes the produced pair.
first:, second: = produceTwo()
```

The second block documents the complete multi-binding declaration rather than
only its leftmost name. Documentation before a flow statement attaches to the
flow statement, not automatically to a declaration inside its header.

Grouping, placement, duplicate attachment, and other general attachment rules
are owned by [Zax source structure](source-structure.md).

## Diagnostics

Diagnostics should distinguish:

- assignment to an unresolved name;
- a declaration with neither an explicit nor inferable type;
- same-scope redeclaration;
- shadowing without permission from the hidden declaration;
- use of an ordinary binding before initialization completes;
- an implicit immutable reference to a varying place where explicit `varying`
  acknowledgement is required;
- a same-place alias whose final/varying stance conflicts with its referent;
- duplicate qualifier tokens at one syntactic point;
- conflicting explicit place qualifiers;
- a type mismatch or unavailable initializer;
- an unavailable operator candidate for the qualified operands;
- an attempt to regain replacement, writable, or mutability capabilities not
  supplied by the source;
- direct infinitely recursive type layout;
- use of an incomplete type where completed layout is required;
- ambiguous multi-result mapping;
- implicit instance-member access without `_.`; and
- a declaration form used in a source position that does not accept it.

Exact identifiers, wording, and presentation remain later diagnostics design.

## Formatting

Formatting should make declaration intent visible:

```zax
inferred := value
explicit : Type = value
stableValue final : Type = value
anonymous final : :: type { }
```

A formatter may canonicalize `: =` to `:=`. It must preserve the separation
among binding, value, access, and referent-place qualifiers and must not silently
resolve contradictory source intent. It may normalize qualifier ordering and
spacing but must not add or remove explicit qualifiers.

## Boundaries and maturity

This document is current conceptual design, not a formal grammar, compatibility
promise, implementation mapping, or conformance specification.

It establishes constraints that later work must preserve:

- complete inference may add rules without making later uses determine an
  earlier inferred declaration;
- functions and captures may refine recursive bindings without allowing
  executable ordinary self-initialization;
- named non-value definitions may expose incomplete self-names without making
  ordinary value initializers self-referential;
- constructors and lifetime policies must preserve default, direct, and explicit
  `unsafe ???` distinctions;
- move, copy, ownership, and qualifier design must preserve independent binding,
  value, and access capabilities;
- operator design must not permit operator overloads to introduce unresolved
  names;
- flow-control design must preserve header-binding scope and nested block scope;
- module and name-resolution design must preserve one lexical identifier
  namespace, fixed path roots, and pending suffix resolution; and
- structural typing must decide explicitly whether member names, qualifiers,
  defaults, inferred types, and recursive forms participate in identity,
  equivalence, layout, conversion, and reflection.
