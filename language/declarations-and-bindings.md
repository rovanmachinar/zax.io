# Zax declarations and bindings

| Field | Value |
| --- | --- |
| Status | Current conceptual design |
| Audience | Human developers reading, writing, or evaluating Zax |
| Applies To | Programmer-facing declaration, binding, initialization, name-resolution, and assignment boundaries; not a formal grammar or specification |
| Implementation State | Not established by this repository |
| Owns | Value declaration forms; default, direct, inferred, and explicitly bypassed initialization; binding visibility; redeclaration and shadow permission; one lexical identifier namespace; qualified-path resolution through incomplete declarations; explicit instance-member lookup; declaration-facing qualifier axes and attachment, including declaration-side replacement permission; operator-phrase declaration ownership, type-parameter slots, and type-receiver operators; bounded private member eligibility; the declaration-versus-assignment boundary; the general non-value definition family and identity-declaration integration; named type self-reference and `forward` at the depth required by declarations; declaration diagnostics and formatting |
| Does Not Own | Function invocation/result routing ([function invocation](function-invocation.md)); source token/layout behavior ([source structure](source-structure.md)); qualifier semantics ([qualifiers](qualifiers.md)); or transparent alias/identity semantics ([identity types](identity-types.md)) |

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

Initializer and constructor selection are defined by
[Zax construction, replacement, and destruction](construction-and-destruction.md).
The declaration guarantees only that `item` is introduced and initialized
directly from the supplied source.

## Default initialization

Every ordinary value declaration initializes its value:

```zax
count : Integer
item : Item
```

Default initialization may initialize contained values, execute constructors,
allocate storage, or perform other visible work. Ordinary constructors and
destructors do not report errors through return results, and Zax does not use
exceptions. A replacement constructor may return additional results while still
being required to complete the destination. Initialization may nevertheless
encounter a panic such as allocation failure or stack exhaustion.

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

`unsafe ???` may also appear in a stored-member declaration. It explicitly
bypasses, rather than performs, that member's ordinary initialization. The
member satisfies the containing constructor's initialization obligation through
explicit unsafe responsibility: the compiler neither default-initializes it nor
requires an explicit member `+++`. A later explicit `+++` remains legal delayed
construction.
Complete member and delayed-construction behavior is defined by
[Zax construction, replacement, and destruction](construction-and-destruction.md#manual-and-delayed-construction).

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

Call-boundary behavior for that state is defined by
[Zax function invocation](function-invocation.md#callable-prototypes-and-visible-contracts).

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
and its completion are one declaration rather than shadowing. Overload groups use
category-specific rules in their future owners, and flow labels are a separate
category described under
[flow labels and the ordinary namespace](#flow-labels-and-the-ordinary-namespace).

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
- flow labels are a separate, explicitly shaped category that does not share
  ordinary-identifier lookup.

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

`final` and `varying` appear in two distinct positions, and they answer two
different questions:

| Position | Question answered |
| --- | --- |
| Type-use side | Whether the underlying place is actually `final` or `varying` |
| Declaration-name side | Whether this declaration may exercise whole-value replacement authority |

This parallels the distinction between a mutable value and a readonly access
path. The complete capability-versus-permission model is owned by
[Zax qualifiers](qualifiers.md#type-side-truth-versus-declaration-side-permission).

For new direct storage, an omitted type-side stance resolves from the
declaration-name side:

```zax
foo final : Foo         // final storage
foo varying : Foo       // varying storage
foo : Foo               // varying by default
foo : Foo final         // final
```

A declaration-side `final` may restrict replacement through a varying place, but
a declaration cannot claim more authority than the place actually provides:

```zax
foo final : Foo varying   // legal: restrict this declaration's replacement
foo varying : Foo varying // legal: retain replacement authority
foo final : Foo final     // legal
foo varying : Foo final   // error: access cannot exceed underlying capability
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
referenced place is final. A type-use qualifier cannot misreport a referent's
actual stance:

```zax
source varying : Foo = makeFoo()
trackingView : Foo varying & = source

falseView : Foo final & = source // error: source's place is varying
```

A same-place alias may still narrow its own replacement authority through the
declaration-name side:

```zax
restricted final : Foo varying & = source
```

`source` retains replacement access. `restricted` cannot initiate replacement,
but its type use still truthfully records that another path may replace the
referent.

Declaration-side replacement permission must survive aliases, argument mapping,
results, and captures. A declaration-final path cannot regain replacement
authority merely by being supplied to a callable whose referent type remains
varying.

An immutable reference tracking a varying place must spell `varying`
explicitly. Omission must not silently introduce a path that may observe
successive immutable lifetimes.

The reserved `is final` query reports the resolved type-use or referent-place
truth, not this declaration's replacement permission.

Independent replacement or rebinding of a pointer or reference *binding* itself
is a separate concern from replacement through the declaration, and its exact
syntax remains later pointer and reference design.

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

Type-side final/varying truth is preserved for an alias of the same place, while
the declaration-name side may narrow that alias's own replacement permission. A
new by-value destination, constructed result, copy, or move destination resolves
its own independent place stance.

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

## Operator phrase declarations and type parameters

An [operator phrase](operator-phrases.md) is declared exactly like a symbolic
operator, with its exact words quoted and its fixity stated:

```zax
Chicken :: type {
  operator post unary 'cluck loudly' final : (
    result : Chicken
  )() readonly = {
    // `_` is the receiver operand.
  }
}
```

The operator declaration is both the phrase-form declaration and its
implementation. Zax has no separate source-level phrase-form declaration and no
phrase `forward` form, so there is nothing to introduce ahead of the body.

Custom phrase implementations are receiver-owned. There are no global custom
phrase declarations, and a module may not add a natural phrase to a type it does
not own. Operand holes come from fixity and the callable prototype, never from
substitution inside the quoted words.

The complete phrase feature, including exact finite word sequences, source
interpretation, fencing, and the receiver-oriented workaround for a phrase that
no type can own, is taught by
[Zax operator phrases](operator-phrases.md).

### Type parameter slots and type arguments

A prototype may declare a **type parameter slot** completed by a concrete type
identity:

```zax
Source :: type {
  operator binary 'as' final : (
    result : DestinationType
  )(
    DestinationType : type
  ) readonly = {
  }
}

converted := source as DestinationType
```

A **type argument** has no runtime storage or lifetime and is not evaluated at
runtime. It may determine a value result type, as it does for `as`.

Type category and unused intent remain independent concerns:

```zax
# : Value                 // runtime value is supplied; no body binding
value # : Value           // binding exists; unused use is intentional
DestinationType : type    // concrete type argument; no runtime value
```

A value receiver may accept a type argument without that argument contributing
receiver discovery:

```zax
MyReceiverType :: type {
  operator binary 'for' final : (
    result : ResultType
  )(
    SubjectType : type
  ) = {
  }
}

schema : MyReceiverType
result := schema for SomeType
```

### Type-receiver operators

`operator type` declares an operation whose receiver is a concrete type identity
rather than an instance:

```zax
MyType :: type {
  operator type pre unary 'custom type info for' final : (
    result : MyCustomTypeInfo
  )() = {
    // `_` has the Nothing state because no MyType instance exists.
  }
}

info := custom type info for MyType
```

The concrete type identity supplies discovery and has no runtime storage or
lifetime. `_` points to the `Nothing` instance state because there is no receiver
instance.

A type-receiver operation is not inherently compile-time. It may execute at
runtime and return a runtime value:

```zax
instance := factory create MyType
```

Compile-time execution remains directed and inferred under the ordinary
compile-time function model; a phrase adds no special execution rule.

For a non-generic declaration, the enclosing type name identifies the receiver
type inside its body. Generic instantiations, aliases, and type-receiver
qualifications remain future generic and reflection work.

### Bounded private eligibility

A private type member is eligible only from the owning type's permitted private
context. Outside that context it is *ineligible* rather than merely a worse
candidate, so it is removed before callable preference and cannot defeat, tie, or
block a public operation:

```zax
Chicken :: type {
  weigh private final : (result : Grams)() readonly = {
  }

  weigh final : (result : Kilograms)() readonly = {
  }
}
```

Inside `Chicken`, both declarations are eligible and compete under ordinary
preference: visibility is not match quality among eligible candidates. Outside
`Chicken`, only the public declaration is eligible. Private-only declarations
therefore never change the meaning or validity of source whose caller can see
only public operations.

The visibility modifier follows the declared form rather than preceding
`operator`. Because word-spelled source may present several structurally complete
readings, this rule is what lets one phrase source line be ambiguous inside a
type and unambiguous outside it; that worked scenario is taught by
[operator phrases](operator-phrases.md#visibility-and-private-phrase-eligibility).

Complete visibility behavior, including nested types, friendship, modules, and
reflection, remains future visibility work.

## Assignment and overload selection

Operators occupy language-recognized operator categories rather than the
ordinary identifier namespace. Their declarations may be global or type-defined
as permitted by [Zax operators](operators.md), but an arbitrary identifier or
punctuation sequence does not become an operator through ordinary name
declaration.

An overloadable operator is a callable operation. User-defined operators may have
domain-specific effects and result shapes that are unusual outside their domain.
Exact forms and precedence are defined by the
[operator catalog](operator-catalog.md); multi-component tree declarations are
defined by [mixfix operators](mixfix-operators.md).

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

Protected ordinary intrinsic assignment evaluates the existing destination once,
updates its mutable value through writable access, and returns writable access to
that destination:

```zax
updated : Integer writable & = destination = source
```

This supports right-associated assignment chains when each selected result binds
to the next destination. Custom `=` overloads may return another result shape.

Generated operators participate in ordinary candidate selection with qualifier
requirements. The compiler-recognized reconstructive `=` scenario requires an
immutable value in a type-side varying destination, declaration-side varying
replacement permission, and a writable path. It is unavailable for a mutable
value, a final place, a declaration-side final path, or readonly access.

The reconstructive candidate has a compiler-owned lifetime skeleton and may
select a contextual [`replacement +++` constructor](construction-and-destruction.md#custom-replacement).
User-defined code does not replace that skeleton with an ordinary `=` body.
Complete fallback, result, resource-retention, and alias behavior is defined by
[Zax construction, replacement, and destruction](construction-and-destruction.md#reconstructive-replacement).

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

Replacement-constructor result forwarding is defined by the construction owner.
Exact results for other built-in operators, expression value categories,
overload ranking, conversion, reconstructive-candidate priority, and generated
operator sets remain later operator design.

## Non-value definitions

`::` introduces a family of non-value definition forms:

```zax
Point :: type { }
Fruit :: enum { }
FriendlyName :: alias type ExistingType
MyCount :: identity admit expose type U32
MyHandle :: identity restricted opaque type Integer
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

`alias type` introduces another name for one identity. `identity ... type`
introduces a new identity represented by an existing type. Identity declarations
write one admission keyword (`admit` or `restricted`) and one surface keyword
(`expose` or `opaque`) before `type`; neither axis has an omission default.

Complete projection, admission, identity bridges, representation relationships,
and exposed/opaque behavior are defined by
[Zax identity types](identity-types.md).

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
there becomes visible after its own initializer completes and remains visible to
later initializer operands, the condition, the applicable clause bodies, the
corresponding false or `else` clauses, and the post operation, then leaves scope
after the complete flow statement:

```zax
if result := tryValue() ;; result.isValid() ;; recordAttempt(result)
  use(result)
```

`result` is visible to the condition, the body, and the post operation
`recordAttempt(result)`, and is destroyed when the complete `if` exits.

An explicit block inside a composed initializer keeps its own nested scope, so a
binding it introduces is destroyed at the block's `}` and does not escape to the
condition, body, or post. A sibling initializer operand introduced outside the
block belongs to the enclosing flow-header scope:

```zax
if {
  token := hello()
  goodbye(token)
}; value := compounding() ;; value < 1 {
  use(value)
  token = 5 // error: token belonged to the completed inner block
}
```

Here `token` belongs to the completed inner block, while `value` is a header
binding visible through the condition and body. The flow-header schema, `;;`
section roles, and phase execution are owned by
[core flow control](core-flow-control.md); token spacing and mandatory layout are
owned by [source structure](source-structure.md).

### Flow labels and the ordinary namespace

A flow label is a separate, explicitly shaped name category from the ordinary
identifier namespace. Because the two categories do not share lookup, the same
spelling may name both a binding and a label without ambiguity:

```zax
while outer: outer := 0 ;; outer < 100 ;; ++outer {
  ++outer
  break outer:
}
```

Ordinary binding shadowing is checked only against ordinary bindings, and label
shadowing only against labels. The one-level `shadowable` permission model
described above applies to each category independently: an outer label may grant
one nested reuse of its spelling, and the inner label must itself say
`shadowable` to permit a further reuse. Complete flow-label spelling, placement,
target eligibility, and `:` reference are owned by
[core flow control](core-flow-control.md#flow-labels-and-transfer-targets).


### Stored members

A stored-member initializer runs once per actual containing instance. A storage
qualifier such as `once` may change how many storage instances exist and when the
one instance is initialized. It does not make an ordinary member initializer run
once merely because it appears once in a type definition.

### Default parameters

An input parameter may declare a default expression:

```zax
connect final : ()(
  host : Host,
  attempts : Integer = 3
) = {
}
```

The parameter is a declaration context. Exactly one source establishes it for a
call:

- an explicit argument; or
- its declared default after all explicit value-producing arguments complete.

The new parameter name becomes visible to later parameter defaults only after
its own binding completes. It is not visible in its own default.

Complete omission syntax, default evaluation order, and call-contract ownership
are defined by
[Zax function invocation](function-invocation.md#omitted-inputs-and-defaults).

### Multiple results

Results are specialized declaration contexts and ordered output obligations. A
result declaration does not default-construct a body local merely because it
names a type:

```zax
make final : (
  result : Item
)() = {
  return source
}
```

`return source` directly constructs the result slot. A result initializer opts
into construction before body entry:

```zax
make final : (
  result : Item = :
)() = {
  result.name = "example"
}
```

At a result-routing site, adjacent `name:` selects the result label and may
introduce a same-named inferred binding:

```zax
number:, text: = produce()
```

The labels must exist in the selected callable prototype. Declaration spacing
expresses a different intent:

```zax
number :, text: = produce() // error: incomplete declaration-like intent
```

A complete typed capture introduces ordinary declarations:

```zax
number : Integer, text : String = produce()
```

New declarations and existing destinations may coexist:

```zax
number:, existingText = produce()
```

The first result initializes a new binding. The next result performs ordinary
assignment into `existingText`. Earlier effects remain observable if a later
destination panics.

Duplicate introduced names remain errors. Ordinary declaration visibility,
same-scope redeclaration, and declaration-versus-assignment rules continue to
apply. Complete result labels, routing, omission, construction order, and
completion are defined by
[Zax function invocation](function-invocation.md#result-labels-and-acknowledgement).

## Documentation attachment

Documentation attaches to the complete declaration construct:

```zax
/// Describes x.
x := makeValue()
```

```zax
/// Describes the produced pair.
number:, text: = produce()
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
- a declaration form used in a source position that does not accept it;
- declaration-like spacing that conflicts with label intent;
- duplicate bindings introduced by a result-routing construct; and
- use of an unconstructed result slot as a live value;
- an identity declaration missing either its admission or surface keyword; and
- conflicting `admit`/`restricted` or `expose`/`opaque` intent.

Exact identifiers, wording, and presentation remain later diagnostics design.

## Formatting

Formatting should make declaration intent visible:

```zax
inferred := value
explicit : Type = value
stableValue final : Type = value
anonymous final : :: type { }
MyCount :: identity admit expose type U32
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
- function invocation may use declarations as inputs and result destinations
  without changing when a declared binding becomes visible;
- result slots may delay construction as a specialized output obligation without
  creating an initialize-later form for ordinary local declarations;
- named non-value definitions may expose incomplete self-names without making
  ordinary value initializers self-referential;
- constructors and lifetime policies must preserve default, direct, and explicit
  `unsafe ???` distinctions, including stored-member and delayed construction;
- move, copy, ownership, and qualifier design must preserve independent binding,
  value, and access capabilities;
- operator design must not permit operator overloads to introduce unresolved
  names;
- flow-control design must preserve header-binding scope, nested block scope, and
  the separate flow-label category described here;
- module and name-resolution design must preserve one lexical identifier
  namespace, fixed path roots, and pending suffix resolution; and
- structural typing must decide explicitly whether member names, qualifiers,
  defaults, inferred types, and recursive forms participate in identity,
  equivalence, layout, conversion, and reflection; and
- future generic, composition, and partial work must preserve the explicit
  identity-declaration integration owned here and the behavior owned by
  [Zax identity types](identity-types.md).
