# Zax operators

| Field | Value |
| --- | --- |
| Status | Current conceptual design |
| Audience | Human developers reading, writing, defining, or evaluating Zax operators |
| Applies To | Programmer-facing operator model and selection; not a formal grammar or specification |
| Implementation State | Not established by this repository |
| Owns | The operator mental model; the general operator form and fixity table; symbolic, phrase, circumfix, call/index, and mixfix categories; ordinary operator declarations; global and receiver operands; candidate-tree formation, structural completeness, and pruning; outward result flow and expected-result limits; candidate discovery; contextual/explicit operator completion and direct-before-contextual fallback; application of shared callable viability, expected-result, preference, ambiguity, and unavailable-best rules; private eligibility before preference; once-only evaluation; eager, protected, and short-circuit behavior; protected intrinsic domains; direct-before-fallback and optional presence/reset/transfer-source behavior; operator costs, diagnostics, source stability, and summary menu |
| Does Not Own | Phrase-specific behavior ([operator phrases](operator-phrases.md)); exact forms and domain reservation ([operator catalog](operator-catalog.md)); complete [optional behavior](optional-values.md); uncommitted integer evaluation and realization ([integer literals and realization](integer-literals.md)); protected integer behavior ([integer operator catalog](integer-operator-catalog.md)); mixfix matching ([mixfix operators](mixfix-operators.md)); or shared callable preference/result routing ([function invocation](function-invocation.md)) |
| Source / Provenance | Legacy [basics](../basics.md), [Nothing](../nothing.md), and retired optional evidence together with dispositioned operator-overloading material |

## Mental model

An operator is a callable operation selected from a language-recognized source
form and qualified operands.

```zax
result := left + right
```

Symbolic source syntax forms one fixed expression tree. Word-spelled source may
present several structurally complete readings, so the compiler forms every
complete candidate tree and requires exactly one to remain viable. Operator
discovery and selection then choose one declaration for each ordinary node, or
one direct [mixfix](mixfix-operators.md) declaration for a matching
multi-component tree.

Zax permits custom types to define domain-specific results and effects. It does
not let a declaration invent arbitrary punctuation, change precedence, break
protected built-in behavior, or resolve ambiguity by source order.

## Operator forms

Zax distinguishes:

- **symbolic operators**, such as `+`, `&~`, and `<<%`;
- **[operator phrases](operator-phrases.md)**, operators spelled as one or more
  exact words such as `logical nand` and `as`;
- **circumfix operators**, such as `|value|`, with recognized opening and closing
  components;
- **postfix delimited call and index forms**, `callable(...)` and
  `container[...]`; and
- **mixfix operators**, one operation selected from a tree containing several
  recognized components and operand holes.

**Fixity** states where an operation's operands occur relative to its recognized
components:

| Operator form or fixity | Source shape |
| --- | --- |
| Pre-unary | `operator operand` |
| Post-unary | `operand operator` |
| Binary | `left operator right` |
| Circumfix | `opening operand closing` |
| Call/index | Postfix delimited inputs |
| Mixfix | One complete operation consuming several fixed components and operand holes |

Symbolic and phrase spellings share this table. A phrase component uses only
pre-unary, post-unary, or binary fixity; circumfix, call, and index remain
separate operator forms rather than phrase-component fixities. A word-delimited
operation that surrounds or separates several holes is a
[mixfix](mixfix-operators.md) built from several phrase components rather than a
phrase circumfix.

The complete current symbolic/circumfix menu, exact phrase forms, and the
precedence table are in the [operator catalog](operator-catalog.md). Phrases have
their own [programmer model](operator-phrases.md), and mixfix has
[its own](mixfix-operators.md).

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
| Pre/post unary, symbolic or phrase | Sole operand | None |
| Binary, symbolic or phrase | Left operand | Right operand only |
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
are in the [operator catalog](operator-catalog.md#circumfix-forms).

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

## Operator phrases

An **operator phrase** is an operator form written as one or more exact words:

```zax
loaf := baker bakes bread dough
converted := source as DestinationType
ready := left logical nand right
```

A phrase is not a second dispatch mechanism. Its receiver, parameter mapping,
viability, partial-order preference, result handling, availability, evaluation,
protection, and mixfix participation use the shared callable and operator model
described by the rest of this document, so a phrase adds spelling rather than
selection machinery.

The cohesive phrase feature is owned by
[Zax operator phrases](operator-phrases.md): exact finite word sequences, phrase
declarations, receiver ownership and the receiver-oriented workaround, phrase
candidate teaching and ambiguity, natural/grouped/fenced source, presentation
after selection, keyword roles, enclosure boundaries, private phrase eligibility,
and phrase costs and diagnostics.

Three consequences matter to the shared operator model and are applied
throughout this document:

- a phrase component uses only pre-unary, post-unary, or binary fixity;
- custom phrase implementations are receiver-owned, which narrows
  [discovery](#discovery); and
- word-spelled source may present several structurally complete readings, which
  is why [candidate-tree formation](#candidate-tree-formation-and-selection)
  exists.

Ordinary user-defined phrase operations are eager, and users cannot declare their
own evaluation strategy. The language protects the exact Boolean signatures whose
evaluation boundary must remain visible; exact Boolean `logical nand`,
`logical and not`, `logical nor`, and `logical or not` short-circuit and form
[mixfix-consumption barriers](mixfix-operators.md#protected-barriers), while
exact Boolean `logical xnor` is eager and consumable. Precedence belongs to the
form rather than the receiver. A **reserved phrase form** is an exact form user
code cannot declare at all; exact forms, precedence levels, and the reserved set
are in the [operator catalog](operator-catalog.md#operator-forms).

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
type; future partial-type work must define any owner-authorized extension. Custom
phrase implementations follow the same receiver-owned rule, so a phrase use
discovers only protected or language-provided declarations for the exact form and
type-defined declarations on its receiver operand. That narrowing, its rationale,
and the receiver-oriented wording workaround are taught by
[operator phrases](operator-phrases.md#receiver-ownership).

Structural similarity does not widen discovery to every type with a compatible
shape. Future structural typing may affect viability after discovery without
turning member additions into distant lookup changes.

### Contextual completion

A visible typed peer can sometimes supply the missing type for a number literal:

```zax
myValue := 0 + (: MyInteger = 5)
```

This is allowed only when `MyInteger` permits contextual construction from the
number and its `+` operator also permits contextual completion. The source then
behaves as though the programmer had written:

```zax
myValue := (: MyInteger = 0) + (: MyInteger = 5)
```

The construction is real and retains its source-order effects and costs.

Constructors and operators share one completion mode:

- `contextual` permits one bounded completion in a source context defined to
  support it;
- `explicit` requires the concrete value to be present before selection; and
- omission means `explicit`.

Exact final keyword placement remains future source integration. Conceptually,
both declarations must opt in:

```zax
// Illustrative completion-mode syntax.
+++ contextual final : ()(rhs : Integer) = {
}

operator binary '+' contextual final : (
  result : MyInteger
)(
  rhs : MyInteger
) = {
}
```

### Direct operations win before contextual fallback

Contextual completion is attempted only when ordinary selection has no viable
direct candidate. It never repairs:

- ambiguity among direct candidates;
- a range failure after a direct candidate is selected;
- a uniquely best but unavailable operation; or
- a source tree that remains structurally ambiguous.

The fallback obtains one type from one visible peer or declared type anchor,
tries one contextual construction, and then tests only the already identified
operator form on that type. It never chains constructors or conversions and
never searches unrelated types.

The right operand still contributes no declarations during ordinary discovery.
The contextual fallback is a later, explicitly permitted completion path, not a
general reversal of receiver ownership.

Only operator contexts currently opt in. A contextual constructor does not by
itself authorize implicit construction in ordinary calls, returns, or
conditional convergence.

The complete number-literal examples, sign/range checks, `as` behavior, and
stopping rules are in
[Zax integer literals and realization](integer-literals.md#when-a-visible-type-may-complete-an-operand).

## Candidate-tree formation and selection

A symbolic source form fixes one expression tree before selection. Word-spelled
source may present several structurally complete readings, so recognition forms
candidate trees and rejects incomplete combinations rather than preferring the
longest word sequence. A reading that leaves source unattached, or that places
two adjacent expressions with no joining operation, is not one expression and is
eliminated before parameter matching.

The observable model is:

1. establish the logical statement and hard source boundaries;
2. form every structurally complete phrase/operator/mixfix candidate tree;
3. eliminate incomplete holes, unattached source, unsupported fixities, and
   impossible receiver forms;
4. resolve inner operations independently;
5. require an inner expression node to produce exactly one usable result;
6. propagate that selected result's type, qualifications, result shape, and
   transfer behavior outward to the next receiver position;
7. apply ordinary parameter mapping, viability, preference, and availability
   within each candidate tree;
8. accept the source when exactly one complete tree remains viable;
9. diagnose an operator-attachment ambiguity when several complete trees remain
   viable; and
10. validate the selected tree's physical phrase presentation only after
    selection.

This is an observable dependency model, not a required compiler-pass
implementation. An implementation may prune inexpensive impossibilities early so
long as it produces the same result.

Resolution proceeds only through the finite candidate trees already formed from
source. A selected result cannot invent new phrase words, cause source to be
rescanned, or start a fixed-point rewrite.

Physical presentation confirms a selected interpretation and never selects it.
The phrase-specific presentation contract and worked examples are owned by
[operator phrases](operator-phrases.md#presentation-confirms-a-selection-it-never-makes-one);
general physical-line and trivia mechanics remain with
[source structure](source-structure.md#operator-phrase-source-integration).

Worked word-spelled examples, the distinction between overlapping words and real
ambiguity, and the grouping or fencing a programmer uses to state the intended
reading are taught by
[operator phrases](operator-phrases.md#how-word-spelled-source-is-interpreted).

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

### Outward result flow

A uniquely selected inner result may make an enclosing operation viable. Its
type, qualifications, result shape, and transfer behavior propagate outward to
the next receiver position.

Zero or multiple results cannot fill one expression position:

```zax
sum:, carry: = addWithCarry(a, b)

combined := addWithCarry(a, b) next operation
// error: no single intermediate receiver
```

There is no implicit "result of results" type. A future explicit combiner may
produce one structure value, after which that structure has an ordinary
compiler-known type and may become a receiver.

An inner result-only ambiguity does not flow backward from an enclosing
operation. The allowed direction is `selected inner result -> enclosing receiver
selection`, never `enclosing receiver requirement -> manufacture an inner result
choice`. The programmer states an intended narrow boundary with a complete typed
declaration.

Worked word-spelled examples of outward flow are taught by
[operator phrases](operator-phrases.md#results-flow-outward-never-inward).

### Narrow expected result

A direct, complete, explicitly typed declaration may provide narrow expected
result context:

```zax
integer : Integer = parseableLeft + parseableRight
```

An inferred declaration, outer overloaded call, later assignment, condition, or
unresolved surrounding operator does not invent that context. Result quality can
break a tie only after receiver and supplied-input comparisons are equal.

Expected-result context may choose an implementation within one already formed
tree. It may not choose between distinct operator extents or attachment trees.

See [function invocation](function-invocation.md#narrow-expected-result-selection)
for the shared boundary and
[operator phrases](operator-phrases.md#results-flow-outward-never-inward) for the
word-spelled example.

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
other closed domains for coherent language evolution. Exposed integer identities
receive the identity-adjusted protected surface defined by
[Zax integers](integers.md) and [Zax identity types](identity-types.md).
Extensible library types such as `String` may expose ordinary library-owned
overloads.

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
[operator catalog](operator-catalog.md#protected-domains). Complete protected
integer behavior is in the
[integer operator catalog](integer-operator-catalog.md).

## Boolean operations and fallback

Exact `Boolean && Boolean` and `Boolean || Boolean` are language-provided,
protected, left-to-right short-circuit operations:

```zax
if ready && ?resource
  use(resource)
```

Other valid `&&`/`||` shapes are ordinary eager overloads and may return
arbitrary result shapes. Exact Boolean `^^` is protected and eager.

Zax also supplies exact logical NAND, AND-NOT, NOR, OR-NOT, and XNOR phrase
operations. The
[catalog](operator-catalog.md#exact-logical-phrases) records their exact
forms, precedence, and evaluation.

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

### Optional reset and transfer-source operations

`reset value` is a protected pre-unary optional phrase. It destroys a present
boxed value, leaves the same wrapper absent, and returns a reference to that
wrapper:

```zax
reset optionalValue
(reset optionalValue) = [{}]
```

`last value` and `move value` are protected pre-unary optional source forms.
They preserve the optional type and attach distinct transfer stances. Producing
the stance has no effect until a constructor, assignment, parameter binding, or
another consumer accepts it:

```zax
source : MyType?

terminallyTransferred : MyType? = last source
```

The accepted consumer owns the source-state effect. A terminal payload transfer
leaves the source wrapper absent; a nonterminal move preserves source presence
and leaves a present boxed value live in its defined moved-from state.

Postfix optional access yields a reference under the boxed qualifications after
proof. It is not a runtime-checking `unwrap` operation. Safe source that lacks
proof is rejected rather than receiving an implicit runtime fallback.

A future narrow unsafe presence/lifetime assertion may permit access the compiler
cannot prove. It adds no required runtime check; if the asserted lifetime or
presence is false, the language provides no defined behavior. Debug
instrumentation may detect a violation and panic without making that check a
language guarantee. Complete optional state, construction, qualification,
nesting, and transfer behavior is defined by
[Zax optional values](optional-values.md).

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

Candidate-tree formation costs real compile time when several structurally
complete readings must be retained and pruned. The phrase-specific cost profile
and the design choices that bound it are recorded by
[operator phrases](operator-phrases.md#costs-diagnostics-formatting-and-source-stability).

Programmers must be able to discover:

- selected declaration and discovery domain;
- protected, declared, generated, circumfix, phrase, or mixfix origin;
- eager versus short-circuit evaluation;
- fallback negation;
- copies, moves, references, temporaries, captures, and old-value snapshots;
- allocation and destruction;
- mutation versus replacement;
- arithmetic checks and policy results;
- exact, optional, narrowing, unchecked, and unsafe conversion/admission;
- associated count, delta, and distance result identities;
- unchecked arithmetic and divisor contracts;
- branch-specific selection; and
- zero/multiple result handling.

The detailed per-form costs are recorded alongside their families in the
[operator catalog](operator-catalog.md) and applicable type-specific references
such as the [integer operator catalog](integer-operator-catalog.md).

## Diagnostics

Diagnostics should distinguish:

- unrecognized or reserved form;
- reserved-form declaration;
- unsupported fixity or arity;
- no discovered candidate;
- incomplete or unattached source;
- several viable candidate trees;
- ambiguity among implementations of one tree;
- an inaccessible declaration that is ineligible rather than merely worse;
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
- unproved exact integer conversion;
- failed or unavailable identity admission;
- unavailable maximum-width delta;
- unchecked-contract dependence;
- compile-time source panic; and
- source spacing, presentation, or continuation error.

Phrase-specific diagnostics are listed by
[operator phrases](operator-phrases.md#costs-diagnostics-formatting-and-source-stability).
Exact diagnostic identifiers remain future diagnostics work.

## Source stability

- Adding an overload may select another body or create ambiguity.
- Adding a direct complement suppresses prior fallback.
- Adding a direct mixfix may replace component decomposition.
- Changing operand type may move a logical operation between protected
  short-circuit and eager custom behavior.
- Adding, removing, lengthening, reserving, or repricing a
  [phrase form](operator-phrases.md#costs-diagnostics-formatting-and-source-stability)
  is a source-compatibility event that may make existing source ambiguous or
  invalid.
- Adding a language-defined phrase or built-in signature may make previously
  unavailable source valid.
- Adding a range fact or declared identity bridge may enable a conversion.
- Changing an integer role's associated type may change an operator result.
- Changing precedence or tokenization would reinterpret existing source.

Source order, import order, and implementation convenience never resolve these
effects silently.

## Boundaries and maturity

This document is current conceptual design, not formal grammar, a compatibility
contract, or an implementation mapping.

See the [operator catalog](operator-catalog.md) for exact forms,
[operator phrases](operator-phrases.md) for word-spelled operations, and
[mixfix operators](mixfix-operators.md) for tree-pattern operations. Literal
operators, floating/fixed-point/unbounded numeric families, call/index edge
cases, allocation, pointers, generics, reflection, build-contract syntax, and
panic recovery remain future focused work.
