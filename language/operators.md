# Zax operators

| Field | Value |
| --- | --- |
| Status | Current conceptual design |
| Audience | Human developers reading, writing, defining, or evaluating Zax operators |
| Applies To | Programmer-facing operator model and selection; not a formal grammar or specification |
| Implementation State | Not established by this repository |
| Owns | The operator mental model; symbolic, phrase, circumfix, call/index, and mixfix categories; ordinary operator declarations; global and receiver operands; candidate discovery; application of shared callable viability, expected-result, preference, ambiguity, and unavailable-best rules; once-only evaluation; protected intrinsic domains; direct-before-fallback and optional-presence behavior; operator costs, diagnostics, source stability, and summary menu |
| Does Not Own | Exact forms, precedence, built-in family behavior, and availability ([operator catalog](operator-catalog.md)); mixfix tree matching ([mixfix operators](mixfix-operators.md)); source tokenization, whitespace, and continuation ([source structure](source-structure.md)); shared result routing and callable preference ([function invocation](function-invocation.md)); exact conditions and convergence ([core flow control](core-flow-control.md)); lifecycle behavior ([construction, replacement, and destruction](construction-and-destruction.md)); complete phrase, literal, numeric, floating-point, indexing, allocation, pointer, generic, or panic design |
| Source / Provenance | Legacy [basics](../basics.md), [Nothing](../nothing.md), and [optional](../optional.md) evidence together with dispositioned operator-overloading material |

## Mental model

An operator is a callable operation selected from a language-recognized source
form and qualified operands.

```zax
result := left + right
```

Source syntax first forms one fixed expression tree. Operator discovery and
selection then choose one declaration for each ordinary node, or one direct
[mixfix](mixfix-operators.md) declaration for a matching multi-component tree.

Zax permits custom types to define domain-specific results and effects. It does
not let a declaration invent arbitrary punctuation, change precedence, break
protected built-in behavior, or resolve ambiguity by source order.

## Operator forms

Zax distinguishes:

- **symbolic operators**, such as `+`, `&~`, and `<<%`;
- **operator phrases**, language-recognized contextual words whose exact catalog
  remains future phrase work;
- **circumfix operators**, such as `|value|`, with recognized opening and closing
  components;
- **postfix delimited call and index forms**, `callable(...)` and
  `container[...]`; and
- **mixfix operators**, one operation selected from a tree containing several
  recognized components and operand holes.

The complete current symbolic/circumfix menu and precedence table are in the
[operator catalog](operator-catalog.md). Mixfix has its own
[programmer model](mixfix-operators.md).

## Declarations and receiver operands

Ordinary declarations state their form explicitly:

```zax
operator pre unary '-' final :
  (result : Vector)(value : Vector) = {
  return negate(value)
}

operator binary '+' final :
  (result : Vector)(lhs : Vector, rhs : Vector) = {
  return add(lhs, rhs)
}
```

A type-defined operation uses `_` as its receiver operand:

```zax
Vector :: type {
  operator pre unary '-' final :
    (result : Vector)() readonly = {
    return negate(_)
  }

  operator binary '+' final :
    (result : Vector)(rhs : Vector) readonly = {
    return add(_, rhs)
  }
}
```

| Type-defined form | Receiver | Explicit inputs |
| --- | --- | --- |
| Pre/post unary | Sole operand | None |
| Binary | Left operand | Right operand only |
| Circumfix | Enclosed operand | None |
| Call/index | Callee/base operand | Declared call/index inputs |
| Mixfix | Declared receiver-anchor hole | Every other hole |

Representative declarations and uses:

### Pre-unary

```zax
negated := -vector
```

The type-defined `operator pre unary '-'` above receives `vector` as `_`.

### Post-unary

```zax
Counter :: type {
  operator post unary '++' final :
    (previous : Counter)() writable = {
    // Update `_`, then return its previous value.
  }
}

previous := counter++
```

### Binary

```zax
sum := left + right
```

The left `Vector` supplies the receiver-owned `+` declarations; `right` binds the
selected declaration's explicit parameter.

### Circumfix

```zax
Vector :: type {
  // Illustrative declaration syntax.
  operator circumfix '|' '|' final :
    (result : Float)() readonly = {
    return calculateMagnitude(_)
  }
}

length := |vector|
```

The enclosed `vector` is the receiver operand. Exact recognized circumfix forms
are in the [operator catalog](operator-catalog.md#circumfix-operations).

### Call

```zax
Interpolator :: type {
  // Illustrative declaration syntax.
  operator call final :
    (result : Float)(position : Float) readonly = {
    return interpolate(_, position)
  }
}

curve : Interpolator
sample := curve(0.5)
```

A declared function is implicitly callable; a custom callable value uses the
overloadable complete call form.

### Index

```zax
Matrix :: type {
  // Illustrative declaration syntax.
  operator index final :
    (result : Float)(row : Index, column : Index) readonly = {
    return readElement(_, row, column)
  }
}

element := matrix[row, column]
```

Index is a postfix delimited form rather than a pair of independent bracket
operators. Exact indexing, slicing, proxy, and bounds behavior remains future
work.

Mixfix declarations and uses are taught by
[Zax mixfix operators](mixfix-operators.md).

The receiver's type and qualifications participate in selection. Complete
qualification behavior is defined by [qualifiers](qualifiers.md).

## Discovery

For an ordinary symbolic use, candidates come from:

1. protected or language-provided declarations for the exact form;
2. visible global declarations; and
3. type-defined declarations on the sole unary, enclosed circumfix, call/index
   base, or left binary receiver.

The right binary operand does not contribute declarations from its type. A
global declaration supports forms whose custom value appears on the right:

```zax
operator binary '>' final :
  (result : Boolean)(lhs : Integer, rhs : MyType readonly &) = {
  return lhs > rhs.value
}
```

Global mixfix declarations are not permitted. A mixfix belongs to its receiver
type; future partial-type work must define any owner-authorized extension.

Structural similarity does not widen discovery to every type with a compatible
shape. Future structural typing may affect viability after discovery without
turning member additions into distant lookup changes.

## Viability, preference, and availability

Operators apply the shared callable stages:

1. discovery;
2. operand mapping;
3. viability;
4. partial-order preference; and
5. availability.

A candidate is viable only when receiver, inputs, qualifications, transfers,
constraints, result context, and availability requirements fit.

Preference is not a score. Candidate A dominates B only when A is no worse for
every comparable receiver/input/result slot and strictly better somewhere.
Declaration origin supplies no preference: a type-defined and global candidate
that remain equal or incomparable are ambiguous.

A uniquely best bodyless, `forbidden`, unsupported generated/default, or
otherwise unavailable declaration reports an unavailable operation. Selection
does not silently fall through to a weaker candidate.

The complete shared relation is defined by
[function invocation](function-invocation.md#candidate-selection).

## Result shapes and expected context

An operator may declare zero, one, or several result slots:

```zax
sum:, carry: = customLeft + customRight
```

A nested expression requires exactly one usable value:

```zax
combined := customLeft + customRight
// error if the selected + has two mandatory results
```

Several results do not become an implicit tuple or structural value.

A direct, complete, explicitly typed declaration may provide narrow expected
result context:

```zax
integer : Integer = parseableLeft + parseableRight
```

An inferred declaration, outer overloaded call, later assignment, condition, or
unresolved surrounding operator does not invent that context. Result quality can
break a tie only after receiver and supplied-input comparisons are equal.

See [function invocation](function-invocation.md#narrow-expected-result-selection)
for the shared boundary.

## Evaluation order

Selection does not execute runtime operands. After selection, an ordinary binary
operation:

1. evaluates the receiver/left operand once;
2. immediately binds it according to the selected qualification and transfer;
3. evaluates the right operand once;
4. immediately binds it; and
5. invokes the operation.

```zax
result := makeLeft() + makeRight()
```

`makeLeft()` completes before `makeRight()`.

Pre/post unary, circumfix, call/index, and mixfix forms likewise evaluate every
written operand hole once in visible source order. Protected short-circuit forms
are the documented exception: they may skip a later runtime operand while still
requiring every possible path to resolve at compile time.

Optimization may elide work only when the selected operation, order, effects,
results, and lifetimes remain unchanged.

## Protected intrinsic domains

A **protected intrinsic signature** is an exact operator signature whose every
operand belongs to a closed intrinsic family. A user declaration cannot replace
it or claim a currently unsupported all-intrinsic combination.

```zax
// error: protected intrinsic signature
operator binary '+' final :
  (result : Strange)(lhs : Integer, rhs : Integer) = {
}
```

A mixed custom signature remains extensible:

```zax
operator binary '+' final :
  (result : MyType)(lhs : Integer, rhs : MyType) = {
}
```

This reserves fundamental Boolean, integer, pointer-sized numeric, floating, and
other closed domains for coherent language evolution. Extensible library types
such as `String` may expose ordinary library-owned overloads.

Signature protection differs from a
[mixfix-consumption barrier](terms.md#mixfix-consumption-barrier). A barrier is a
resolved source node that a user mixfix may accept as a completed hole value but
may not consume as a key component. Exact Boolean short-circuit operations are
barriers because consumption would erase their conditional RHS evaluation.

Exact Boolean XOR illustrates the distinction:

- `Boolean ^^ Boolean` is a protected intrinsic signature, so user code cannot
  replace it;
- it is eager, so a custom-receiver mixfix may consume that node without erasing
  short-circuit behavior.

Mixed custom logical shapes are eager and may participate in a mixfix key.

Missing required protected support indicates an incomplete toolchain rather than
an optional library the program forgot to link. The implementation may use
instructions, constant folding, emitted code, intrinsics, or automatically linked
helpers; operator semantics promise no lowering.

Exact family availability is listed in the
[operator catalog](operator-catalog.md#protected-intrinsic-domains).

## Boolean operations and fallback

Exact `Boolean && Boolean` and `Boolean || Boolean` are language-provided,
protected, left-to-right short-circuit operations:

```zax
if ready && ?resource
  use(resource)
```

Other valid `&&`/`||` shapes are ordinary eager overloads and may return
arbitrary result shapes. Exact Boolean `^^` is protected and eager.

Zax also accepts logical NAND, AND-NOT, NOR, OR-NOT, and XNOR operation concepts.
Their canonical source will be established by future operator-phrase work; the
[catalog](operator-catalog.md#extended-logical-operations) records their current
behavior.

### Direct-before-fallback

The closed complement fallback set is:

```text
?  <-> !
== <-> !=
<  <-> >=
<= <-> >
```

For a written operation:

1. resolve the direct candidate normally;
2. use one applicable direct candidate even if its result later fails a Boolean
   requirement;
3. report direct ambiguity or unavailable best;
4. only no applicable direct candidate attempts the paired operation;
5. resolve that counterpart directly without recursive fallback or Boolean
   filtering;
6. require its selected result to be exactly `Boolean`; and
7. apply one protected Boolean negation.

Fallback never swaps operands, combines comparisons, or changes receiver
discovery.

For example, a type may define only direct `!`:

```zax
Ready :: type {
  isReady : Boolean

  operator pre unary '!' final :
    (result : Boolean)() readonly = {
    return !_.isReady
  }
}

ready : Ready

if ?ready
  use(ready)
```

When no direct `?Ready` candidate applies, the compiler selects `!ready`
normally, requires its result to be exactly `Boolean`, and applies protected
Boolean negation. Conceptually:

```zax
if !(!ready)
  use(ready)
```

The grouping is required because adjacent independent unary applications are not
valid source.

Comparison fallback follows the same model:

```zax
Key :: type {
  operator binary '==' final :
    (result : Boolean)(rhs : Key) readonly = {
    return sameKey(_, rhs)
  }
}

if left != right
  handleDifference()
```

With no applicable direct `!=`, the operation may use:

```zax
if !(left == right)
  handleDifference()
```

Direct ambiguity, unavailable best, or a non-Boolean counterpart result reports
an error rather than activating fallback.

```zax
if : Boolean = ?value
  useValue(value)
```

The typed declaration supplies result context that an ordinary condition does
not.

### Optional presence operation

For an optional value, `?value` is the recognized presence operation and returns
`Boolean`:

```zax
if ?optionalValue
  use(optionalValue.)
```

Postfix optional dereference resolves before a pre-unary operator, so
`?optionalValue.` means `?(optionalValue.)`, not "test presence and then
dereference." Dereference requires proof that the optional contains a live value.

An arbitrary user-defined Boolean-returning `?` does not prove storage live. The
optional-presence contract must be recognized by the analyzer. The lifetime
obligation is owned by
[construction, replacement, and destruction](construction-and-destruction.md#conditionally-live-storage-and-access-proof).

## Branch-specific selection

A conditional expression may select another overload or mixfix on each path:

```zax
result := f + (condition ?? c ;; d)
```

`f` evaluates once. The condition evaluates once, only the selected arm runs,
and that path invokes its selected operation.

One path may use a direct mixfix while another decomposes into ordinary
operators. One logical node may be protected short-circuiting on one path and
eager on another. Every possible path is validated and must converge by the
complete operation boundary.

Condition, selected-arm, and convergence behavior are defined by
[core flow control](core-flow-control.md#conditional-expression-and-branch-convergence).
Mixfix path behavior is defined by
[mixfix operators](mixfix-operators.md#branch-specific-mixfix-selection).

## Assignment and mutation boundaries

Assignment uses an existing destination:

```zax
total = value  // error if total is unresolved
total := value // declaration and initialization
```

A user operator cannot introduce an unresolved caller name. Declaration
initialization remains language syntax rather than an overloadable `=` node.

Protected ordinary intrinsic assignment updates the existing mutable value and
returns writable access to the destination. Assignment is right-associative.

Compound operators are direct operations, not automatic rewrites through a
value operation and assignment:

```zax
counter += delta
counter = counter + delta
```

The second form may select two operations, create a temporary, and invoke
reconstructive replacement. The first evaluates its destination once and invokes
its own candidate.

Immutable varying replacement remains the compiler-owned lifecycle operation
defined by [construction, replacement, and destruction](construction-and-destruction.md#reconstructive-replacement).

## Costs

Programmers must be able to discover:

- selected declaration and discovery domain;
- protected, declared, generated, circumfix, phrase, or mixfix origin;
- eager versus short-circuit evaluation;
- fallback negation;
- copies, moves, references, temporaries, captures, and old-value snapshots;
- allocation and destruction;
- mutation versus replacement;
- arithmetic checks and policy results;
- branch-specific selection; and
- zero/multiple result handling.

The detailed per-form costs are recorded alongside their families in the
[operator catalog](operator-catalog.md).

## Diagnostics

Diagnostics should distinguish:

- unrecognized or reserved form;
- unsupported fixity or arity;
- no discovered candidate;
- operand, result, transfer, qualification, or constraint mismatch;
- equal or incomparable ambiguity;
- uniquely best unavailable operation;
- protected-signature conflict;
- exact Boolean short-circuit barrier;
- expected-result context versus no such context;
- direct selection versus fallback;
- zero/multiple results where one value is required;
- mixfix selection and structural overlap;
- grouping-required comparison;
- arithmetic policy failure;
- invalid shift count type;
- compile-time source panic; and
- source spacing or continuation error.

Exact diagnostic identifiers remain future diagnostics work.

## Source stability

- Adding an overload may select another body or create ambiguity.
- Adding a direct complement suppresses prior fallback.
- Adding a direct mixfix may replace component decomposition.
- Changing operand type may move a logical operation between protected
  short-circuit and eager custom behavior.
- Adding a language-defined phrase or built-in signature may make previously
  unavailable source valid.
- Changing precedence or tokenization would reinterpret existing source.

Source order, import order, and implementation convenience never resolve these
effects silently.

## Boundaries and maturity

This document is current conceptual design, not formal grammar, a compatibility
contract, or an implementation mapping.

See the [operator catalog](operator-catalog.md) for exact forms and
[mixfix operators](mixfix-operators.md) for tree-pattern operations. Phrase
syntax, literal operators, complete numeric families, call/index edge cases,
allocation, pointers, generics, and panic recovery remain future focused work.
