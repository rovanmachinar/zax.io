# Zax core flow control

| Field | Value |
| --- | --- |
| Status | Current conceptual design |
| Audience | Human developers reading, writing, or evaluating Zax |
| Applies To | Programmer-facing synchronous flow control; not a formal grammar or specification |
| Implementation State | Not established by this repository |
| Owns | The exact-`Boolean` condition contract; clause selection; effective-body execution; conditional and loop header schemas, phase order, and `;;` section roles; `if`/`else` clause forms, chaining, and normal-completion post operations; `while`, `until`, `redo while`, `redo until`, `forever`, and explicit `scope`; `break`, `continue`, `next`, and `return` as flow transfers, target eligibility, and barriers; flow-label spelling, placement, and reference; the conditional expression's shared condition, selected-arm, and convergence model; flow-facing costs, diagnostics, formatting, and source stability |
| Does Not Own | How expressions produce values, including operator selection ([operators](operators.md)), exact logical forms ([operator catalog](operator-catalog.md)), and mixfix paths ([mixfix operators](mixfix-operators.md)); token spacing, continuation, brace layout, `else` source layout, and body boundaries ([source structure](source-structure.md)); binding visibility, the ordinary identifier namespace, and shadow permission ([declarations and bindings](declarations-and-bindings.md)); scope-exit destruction order and proof of a live value before access through conditionally live storage ([construction, replacement, and destruction](construction-and-destruction.md)); whole-function `return` result completion and branch-specific callable selection ([function invocation](function-invocation.md)); complete iteration, selection, resource management, error propagation, value polymorphism, async flow, formal grammar, or compiler behavior |
| Source / Provenance | Legacy [flow control](../flow-control.md) and [scope](../scope.md) evidence |

## Mental model

A core flow statement selects and runs a body under a predictable set of visible
phases. It has:

1. a complete flow introducer such as `if`, `while`, or `redo until`;
2. an optional flow label;
3. construct-specific initializer, condition, and post sections;
4. exactly one effective body statement; and
5. defined normal-completion and abrupt-transfer behavior.

Ordinary `if` reads as expected:

```zax
if input < 0
  return true
```

Ordinary `if`/`else` reads the same way:

```zax
if input < 0
  handleNegative()
else
  handleOther()
```

Ordinary loops also read as expected:

```zax
while starting <= 100 {
  ++starting
}
```

Header sections make the phases explicit. The initializer runs once, the
condition produces exactly `Boolean`, the selected body runs, and a post
operation runs after normal completion while header bindings are still alive:

```zax
while i := 0 ;; i < 100 ;; ++i {
  useIndex(i)
}
```

Loops add a repetition point, and transfers choose whether the post operation
runs:

```zax
while outer: i := 0 ;; i < 100 ;; ++i {
  if i == 5
    next outer:

  if i == 50 {
    i += 10
    continue outer:
  }
}
```

`next` runs the target's post operation before proceeding. `continue` skips it.
Every transfer destroys the scopes it leaves before arriving at its target.

The remainder of this document layers that model: conditions, effective bodies
and header schemas, `if`/`else` and post, loops and explicit `scope`, labels and
transfer targets, unwinding and completion, the conditional expression, then
analysis, costs, diagnostics, and source stability.

## Conditions

### The exact `Boolean` contract

Every core condition must resolve to exactly `Boolean`. Zax does not apply a
condition-specific truthiness conversion to pointers, optionals, errors, or
arbitrary values.

```zax
if booleanValue
  useBoolean()

if ?pointer
  use(pointer)

if !pointer
  handleNothing()
```

A value that is not already `Boolean` is a condition error rather than an
implicit conversion:

```zax
// if pointer       // error: pointer is not Boolean
// if optionalValue // error: MyType? is not Boolean
```

Each condition test is evaluated once at its defined phase and in visible source
order. Its completed side effects are observable before the selected body runs.

### Truth-producing operations

Conditions consume `Boolean` values; [operators](operators.md) owns how those
values are produced. The parts a condition depends on are:

- `?` and `!` are ordinary pre-unary operators resolved under ordinary selection.
  A condition supplies no expected `Boolean` result to that selection: it
  validates the already selected result and errors if it is not `Boolean`.
- An ambiguous direct `?` or `!` selection is an error. It does not activate the
  opposite-operator fallback, and the wanted `Boolean` never narrows the
  candidate set.
- Only when no direct overload applies may the opposite operator supply a
  fallback (`?value` via `!(!value)`, `!value` via `!(?value)`), and only when
  that opposite selection is unambiguous and returns exactly `Boolean`.
- An anonymous typed declaration such as `if : Boolean = ?value` supplies result
  context that an ordinary condition does not.
- The exact language-provided Boolean logical family is protected and cannot be
  user-replaced. Its members differ only in the logical operation and whether
  that operation can decide its result before evaluating the right operand:

  | Exact Boolean form | Evaluation |
  | --- | --- |
  | `&&` | Short-circuit when the left operand is false |
  | `\|\|` | Short-circuit when the left operand is true |
  | `^^` | Eager |
  | `logical nand`, `logical and not` | Preserve the corresponding AND short-circuit decision |
  | `logical nor`, `logical or not` | Preserve the corresponding OR short-circuit decision |
  | `logical xnor` | Eager |

  Every exact form evaluates required operands from left to right and still
  requires every possible operand path to be valid at compile time. Their exact
  forms and precedence are listed in the
  [operator catalog](operator-catalog.md#boolean-operations).
- For operand shapes other than an exact protected Boolean signature, the
  symbolic and phrase forms are ordinary eager overloadable operations, may
  return any type, and evaluate both operands left to right.

Short-circuiting skips runtime evaluation, not compile-time resolution. A
condition therefore may rely on protected short-circuit order for proof, but only
where the node really is exact `Boolean`/`Boolean`.

Operator discovery and selection are defined by [operators](operators.md);
exact forms and precedence by the
[operator catalog](operator-catalog.md#boolean-operations).

### Optional presence and dereference in conditions

Postfix optional dereference resolves before a pre-unary operator, so `?value.`
means `?(value.)` and does not test presence before dereferencing. Dereferencing
an optional requires static proof that it holds a live value. The general form
tests presence and dereferences in the proven body:

```zax
if ?optionalValue
  use(optionalValue.)
```

A compound proof is safe only when its right expression is unambiguously exactly
`Boolean`, because only then is the node a protected short-circuit operation:

```zax
if ?optionalValue && (: Boolean = ?optionalValue.)
  use(optionalValue.)
```

Without that typed right operand, an underlying `?` returning some other type
makes the node an ordinary eager overload that evaluates the dereference anyway,
and an ambiguous direct selection is an error:

```zax
// if ?optionalValue && ?optionalValue. // error unless ?optionalValue. is exactly Boolean
```

The proof need not be immediately adjacent. Construction, earlier control flow,
and other recognized presence contracts may also establish it. The
programmer-visible proof obligation is owned by
[construction, replacement, and destruction](construction-and-destruction.md#conditionally-live-storage-and-access-proof);
the presence operation itself is owned by [operators](operators.md).

## Effective bodies and header boundaries

Each clause consumes one effective body statement: simple, `;`-composed, or
braced. The clause owns a scope around that body regardless of its spelling.

```zax
if input < 0
  return true

if input < 0 { return true }

if input < 0
  ++input;
  value := input * 3;
  return true
```

The condition or header and the body must have a clear boundary: an indented
following statement, or a body-opening `{` on the final physical header line. A
body attached with neither is an error:

```zax
// if input < 0 return true // error: no body boundary
```

An unbraced body ends after its one effective statement. A correctly dedented
statement runs independently without requiring a blank line:

```zax
if condition
  doConditionalWork()
doIndependentWork()
```

An empty braced body is legal:

```zax
if condition { }
```

Complete effective-statement, composition, brace-layout, and body-boundary rules
belong to [source structure](source-structure.md). This document depends on them
but does not redefine them.

## Header schemas, ordering, and binding scope

`;;` separates construct-specific header sections. It is a header-section
separator, not doubled statement composition, and it requires whitespace on both
sides.

Conditional constructs (`if`, `while`, `until`, `redo while`, `redo until`) use:

```text
condition
initializer ;; condition
initializer ;; condition ;; post
;; condition
;; condition ;; post
```

Only the initializer position may be empty. One `;;` always means initializer
plus condition, so this source treats `condition` as the initializer and requires
`postOperation()` to produce `Boolean`:

```zax
if condition ;; postOperation()
  body() // error unless postOperation() returns Boolean
```

Even when `postOperation()` does return `Boolean` and the source is therefore
valid, it still does not mean condition plus post: the second section is the
condition. Write the no-initializer condition-plus-post form explicitly:

```zax
if ;; condition ;; postOperation()
  body()
```

Conditionless `forever` and `scope` have no condition to supply and use:

```text
body
initializer body
;; post body
initializer ;; post body
```

```zax
forever i := 0 {
  body()
}

forever ;; ++i {
  body()
}

scope i := 0 ;; ++i {
  body()
}
```

The keyword determines the header schema. A conditional loop's second section is
a condition; a conditionless construct's second section is a post operation.
This is construct-specific meaning, not an ambiguity.

The initializer accepts any effective statement and runs once. A binding it
introduces becomes visible after its own initializer completes and remains
visible to later initializer operands, the condition, the true and false
clauses, the post operation, and the complete body. It is destroyed when the
complete flow statement exits. Bindings introduced inside a nested initializer
block do not escape that block. Binding visibility and the nested-block boundary
are owned by [declarations and bindings](declarations-and-bindings.md).

Calls in initializer, condition, and post sections are ordinary synchronous call
sites. Earlier completed effects are visible to callbacks and reentrant code; no
code observes a partially established binding.

## `if`, `else`, and normal-completion post operations

`else` binds to the nearest open `if`. A label on `if` names the complete
`if`/`else if`/`else` chain; individual `else` clauses do not have independent
flow labels.

### `else` clause forms

An unbraced `else` follows the complete true body and takes one effective body
statement of its own:

```zax
if condition
  trueBody()
else
  falseBody()
```

A braced chain keeps `else` on the physical line that closes the preceding body:

```zax
if condition {
  trueBody()
} else {
  falseBody()
}
```

`else if` is an `else` whose effective body is another `if`, so a chain reads as
one selection:

```zax
if firstCondition
  firstBody()
else if secondCondition
  secondBody()
else
  finalBody()
```

Exactly one clause of a chain runs. A chain with no matching clause and no final
`else` runs no body, and the `if`'s post operation still runs. One flow label on
the leading `if` names the complete chain:

```zax
if pick_one: firstCondition
  firstBody()
else if secondCondition
  break pick_one:
else
  finalBody()
```

`else` must remain visually attached to the `if` it completes: it aligns with its
owning `if`, follows the complete true body with no physically blank line
between them, and shares a physical line with a preceding `}`. Those source
rules and their counterexamples are owned by
[source structure](source-structure.md#else-attachment-and-layout).

### Execution and post operations

```zax
if bad_stuff: \
  resource := getResource();
  inspectMetadata(resource) ;;
  validated(resource) ;;
  discard(resource) {
  if !resourceInspectionPassed(resource)
    break bad_stuff:

  use(resource)
} else {
  tryToFix(resource)

  if stillBad(resource)
    break bad_stuff:
}
```

Execution is:

1. run the initializer once;
2. establish its bindings;
3. evaluate the condition once;
4. select the true body or the corresponding false or `else` body;
5. run that body;
6. destroy clause-body locals on normal completion;
7. run the post operation exactly once while header bindings remain alive;
8. destroy header bindings; and
9. leave the complete `if`.

A post operation runs after the selected body completes normally. It also runs
when the condition is false and no `else` body exists. It does not run when
`break`, `continue`, `next`, `return`, or a panic leaves the complete `if`; a
targeted `break bad_stuff:` above deliberately skips it. Panic is fatal rather
than an ordinary alternative flow path.

An `if` post operation is ordinary normal-completion work, not a destructor,
`finally`, `defer`, or guaranteed cleanup hook. Deliberately bypassing it is
part of the labeled-exit model.

## Loops and explicit `scope`

`while` and `until` test before first entry. `redo while` and `redo until`
differ only by skipping that first test: the body executes once, then ordinary
post-then-test cycling begins.

```zax
while starting <= 100 {
  ++starting
}

until starting > 100 {
  ++starting
}

redo while starting < ending {
  doSomething()
}

redo until starting == ending {
  doSomething()
}

forever {
  if done
    break
  ++i
}
```

The common progression model is:

| Construct | Initial entry | Normal fallthrough or `next` | `continue` |
| --- | --- | --- | --- |
| `while` / `until` | initializer, test, selected body | post, test, selected body | test, selected body |
| `redo while` / `redo until` | initializer, body without first test | post, test, selected body | test, selected body |
| `forever` | initializer, body | post, body | body |
| explicit `scope` | initializer, body | post, then exit normally; `next` instead re-enters | re-enter body |

For `while`, continuation selects the body when the condition is true. For
`until`, continuation selects the body while the condition is false.

`next` and `continue` are deliberately different:

```zax
while i := 0 ;; i < 100 ;; ++i {
  if i == 5
    next // post increments i, then the loop tests 6

  if i == 50 {
    i += 10
    continue // skips post, then the loop tests 60
  }
}
```

Normal loop fallthrough behaves like `next`: it runs post before the next test
or unconditional entry. `break` skips post and exits. `continue` skips post and
goes to the next test, or directly to the next body entry when the construct has
no condition.

An explicit `scope` is re-enterable; an arbitrary `{ ... }` block is not. Normal
`scope` completion runs post and exits. `next` targeting the scope runs post and
re-enters. `continue` targeting it skips post and re-enters. `break` skips post
and exits. A `forever` body differs because normal completion implicitly repeats
after post.

Comparing `forever` with an always-repeat decision and `scope` with a
never-repeat decision is useful in design reasoning, but neither construct
evaluates a hidden condition, operator, or reflectable expression.

## Flow labels and transfer targets

### Label spelling and placement

A flow label:

- starts with a lower-case ASCII letter;
- may then use lower-case ASCII letters, digits, and `_`, including consecutive
  interior underscores;
- may not begin or end with `_`; and
- is immediately followed by `:`.

```zax
outer:
my___label:
loop2:
```

The label follows the complete flow introducer:

```zax
if check_resource: condition
  body()

while outer: condition
  body()

redo while retry: condition
  body()

scope process_work: {
  body()
}
```

For `redo while` and `redo until`, the two keywords form one introducer, and a
label follows that complete introducer. The optional `shadowable` marker, when
present, sits between the introducer and the label; see
[Label namespace and shadow permission](#label-namespace-and-shadow-permission).

### Label namespace and shadow permission

Flow labels are a separate, explicitly shaped name category from ordinary
bindings, so the same spelling may name both without lookup ambiguity:

```zax
while outer: outer := 0 ;; outer < 100 ;; ++outer {
  ++outer
  break outer:
}
```

Expression `outer` names the binding; target `outer:` names the label. Label
shadowing is checked only against labels, and ordinary binding shadowing only
against ordinary bindings. The namespace boundary is owned by
[declarations and bindings](declarations-and-bindings.md#flow-labels-and-the-ordinary-namespace).

Overlapping duplicate labels are errors unless the outer label grants one level
of `shadowable` permission. `shadowable` follows the complete flow introducer and
precedes the label it applies to:

```zax
forever shadowable young: {
  forever young: {
    next young: // inner target

    // forever young: { } // error: middle label is not shadowable
  }

  next young: // outer target is visible again
}
```

`shadowable`:

- follows the complete flow introducer;
- precedes a required label;
- has whitespace on both sides;
- belongs to the outer label willing to be hidden;
- grants permission only to the next overlapping declaration; and
- is not inherited by that inner declaration.

An inner label must itself say `shadowable` to permit another nested reuse. While
the inner label is active, references to that spelling cannot reach the hidden
outer label.

### Bare and explicit target selection

For a particular transfer keyword:

1. scan outward through constructs ineligible for that keyword;
2. find the nearest eligible target;
3. if it is unlabeled, a bare transfer targets it;
4. if it is labeled, a bare transfer is an error and may not silently skip past
   it; and
5. an explicit `keyword label:` may cross nearer constructs and selects that
   visible eligible target.

| Construct | Bare `break` | Bare `continue` / `next` | Explicit `break label:` | Explicit `continue label:` / `next label:` |
| --- | --- | --- | --- | --- |
| Loop | Yes when nearest and unlabeled | Yes when nearest and unlabeled | Yes | Yes |
| Explicit `scope` | Yes when nearest and unlabeled | Yes when nearest and unlabeled | Yes | Yes |
| Labeled `if` | No implicit target | No | Yes | No |
| Unlabeled `if` | Transparent | Transparent | Not nameable | Not nameable |
| Ordinary block | Transparent | Transparent | No | No |

`continue if_label:` and `next if_label:` are errors: re-evaluating an `if` would
turn selection into iteration, and treating them as exits would make them
misleading synonyms for `break if_label:`.

```zax
scope process_work: i := 0 ;; ++i {
  while moreToDo() {
    if lessImportantStuffDone()
      next // targets the inner unlabeled while
  }
}
```

With no inner loop, the labeled scope is the nearest eligible target and requires
explicit naming:

```zax
scope process_work: i := 0 ;; ++i {
  if lessImportantStuffDone()
    next // error: process_work: is labeled
}
```

An unlabeled explicit scope is an ordinary bare target:

```zax
while i := 0 ;; i < 100 ;; ++i {
  scope j := 0 ;; ++j {
    if lessImportantStuffDone()
      next // targets the explicit scope
  }
}
```

Target references retain `:`:

```zax
break outer:
continue outer:
next outer:
```

Omitting the colon in a target position is an error. `continue()` and `next()`
remain ordinary call expressions: a spelling has keyword status only where its
keyword construct is grammatically permitted, so these keywords are transfers
only when written standalone or followed by a flow label. The general rule is
owned by
[source structure](source-structure.md#contextual-keyword-recognition).

## Unwinding, destruction, and completion

Before a transfer arrives at its target, every body or nested scope it leaves is
destroyed in reverse construction order. Target-header bindings remain alive when
`next` or `continue` re-enters that target and are destroyed only when the
complete target flow statement exits.

On normal body completion:

1. destroy body-local bindings;
2. run post when the construct requires it;
3. test, re-enter, or exit according to the construct; and
4. destroy header bindings when the complete construct exits.

`break` and `return` skip ordinary post operations but still perform applicable
scope destruction. Construction and result completeness must hold on every normal
path produced by branches, loops, and transfers. The automatic local, body, and
header lifetime ordering is owned by
[construction, replacement, and destruction](construction-and-destruction.md#scope-exit-destruction-and-flow-transfers).

### `return` as a flow exit

`return` exits the complete function rather than selecting a flow label. It
unwinds every applicable scope and must satisfy the complete declared result
shape. A value-bearing return supplies every result position; a bare return or
fallthrough is valid only when every result slot is already complete; and
`return #` occupies one result position while preserving or default-completing
that slot. Complete return and result behavior, including `return #`, is owned by
[function invocation](function-invocation.md#return-and-completion). Core flow
treats `return` as an exit that unwinds scopes and never runs enclosing post
operations.

## Conditional expression and branch convergence

The conditional expression uses the shared exact-`Boolean` condition contract,
evaluates its condition once, evaluates only the selected arm, and directly
constructs or supplies its selected result:

```zax
e := a > b ?? c ;; d
```

`??` requires whitespace on both sides. A conditional expression inside a
`;;`-sectioned header requires parentheses for human intent even when a parser
could pair the tokens:

```zax
while i := 0 ;; (a < b ?? c ;; d) ;; ++i {
  body()
}
```

The selected path establishes the destination directly, without
default-constructing and then assigning over it. Surrounding operators and calls
may resolve separately for each selected arm:

```zax
e := f + (condition ?? c ;; d)
```

This may select `f + c` or `f + d`, including different overload bodies, while
preserving source evaluation:

1. evaluate `f` exactly once in its original position;
2. evaluate `condition` once;
3. evaluate only `c` or `d`; and
4. invoke the operation selected for that path.

The distribution never duplicates evaluation of `f`. Branch-specific callable
selection under a conditional expression is owned by
[function invocation](function-invocation.md#branch-specific-selection-under-a-conditional-expression);
this document owns only the shared condition, single-arm evaluation, and
convergence requirement.

A path may also select a direct mixfix while another path uses ordinary
component operations:

```zax
result := (condition ?? a * b ;; fallback) + c
```

The true path may select a `*`, `+` mixfix and the false path ordinary `+`.
The condition and common operands still evaluate once, only the selected arm
runs, every path is validated, and the results must converge. Complete tree
matching is defined by
[mixfix operators](mixfix-operators.md#branch-specific-mixfix-selection).

Every runtime path must converge to one statically usable result shape before
the complete operation completes:

```zax
whatAmI : KnownType = (a < b ?? c ;; d) // each path constructs KnownType
```

An inferred binding may not retain several possible types:

```zax
whatAmI := (a < b ?? c ;; d) // error if c and d remain different types
```

Different overloads or constructors may provide convergence when they ultimately
produce the same type. Boolean-producing paths may converge inside a condition:

```zax
if ?(a < b ?? c ;; d)
  doSomething()
```

A typed optional destination is another convergence form:

```zax
myType : MyType? = (a < b ?? : MyType = a ;; : MyType?)

if ?myType
  use(myType.)
```

On the true path the anonymous `MyType` is constructed from `a` and then
constructs the destination optional. On the false path the anonymous type-default
`MyType?` supplies an empty optional. Both paths converge to the single
destination type `MyType?`. The spelling uses the current anonymous typed
declaration form owned by
[function invocation](function-invocation.md#anonymous-typed-declarations);
complete optional construction and reset design remains
[legacy optional](../optional.md) input and future optional work.

Statement composition does not extend convergence into a later statement:

```zax
if condition
  e : Type = f + (condition ?? c ;; d);
  doSomething(e)
```

The declaration must converge before `;`; the later call cannot help resolve it.
Allowing an ordinary stored binding to retain branch-dependent runtime types
would cross into deferred value-polymorphic representation and dispatch, which is
future work. `??` is itself sufficient source acknowledgement; no extra marker is
required merely because branch paths select different overloads.

## Static analysis, completion, and callbacks

Flow analysis is cross-cutting rather than owned solely by loops or lifecycle:

- access through conditionally live storage, such as an optional dereference,
  requires proof that a live value exists on that path;
- every normal path must complete required result and instance state;
- one lifetime may not be constructed or destroyed twice;
- use before construction and use after destruction are errors;
- obvious unreachable code and accidental nontermination are diagnosable; and
- conservative proof failures may require a future narrow semantic assertion.

```zax
// while i : Integer ;; i < 100 { } // candidate accidental-nontermination diagnostic
```

The empty body above never changes `i`. Complete termination proof is not
required here; a future analysis owner separates mandatory language validity,
conservative proof overrides, advisory diagnostics, and ordinary lints. Which of
these an implementation must prove follows the selected static-analysis contract
rather than one compiler's current cleverness, and the contract model itself
remains future analysis and diagnostics work.

Every branch path of a conditional expression is statically validated even though
only the selected arm evaluates, and a protected `Boolean` short-circuit skips
runtime evaluation without skipping compile-time resolution. Compile-time
resolution never implies early runtime evaluation.

Initializers, conditions, post operations, truth-producing operators, and
conditional arms may invoke arbitrary synchronous code. Each call completes
before the next visible phase. Reentrant code may observe completed earlier
effects but never a partially established binding. Async suspension and
cancellation remain separate future design.

## Costs

- Header initialization pays ordinary construction and call cost once.
- Conditions pay their visible evaluation cost at each defined test.
- `redo` always pays one body execution before its first test.
- `next` pays post cost; `continue` deliberately skips it.
- `break`, `continue`, `next`, and `return` pay destruction cost for every scope
  they unwind.
- A normal-completion post operation may perform substantial ordinary work and is
  not hidden cleanup.
- A conditional expression evaluates one arm and directly constructs its result.
- Branch-dependent ternary resolution may select different callable bodies and
  code paths.
- One branch may select one mixfix operation while another invokes ordinary
  component operators.
- A synthesized `?` or `!` Boolean fallback adds a negation that a direct
  overload may avoid.

## Diagnostics

Flow diagnostics fall into the shared categories defined by
[language-design terms](terms.md): ordinary syntax rejection, semantic errors,
and deliberate intent or layout errors.

Representative semantic errors include:

- a condition that does not resolve to exactly `Boolean`;
- a directly selected `?` or `!` result that is non-`Boolean` in a condition;
- an ambiguous direct `?` or `!` selection in a condition, which a condition may
  not resolve by preferring a `Boolean` candidate;
- an access through conditionally live storage without sufficient proof or
  semantic assertion;
- a composed statement occupying a condition and therefore producing no
  `Boolean`;
- branch-dependent expression paths that fail to converge;
- a transfer label naming a construct ineligible for that keyword;
- label shadowing without permission; and
- an incomplete result or instance on a normal completion path.

Representative deliberate intent or layout errors include:

- a bare transfer stopped by a labeled eligible target;
- a target reference without trailing `:`;
- an `else` detached from the `if` it completes; and
- a ternary in a `;;` header without disambiguating parentheses.

The intent-and-layout category is intentionally bounded. It does not authorize
speculative checks for every malformed spelling a compiler might hypothetically
explain. Source spacing, continuation, and brace-layout diagnostics are owned by
[source structure](source-structure.md); exact diagnostic identifiers remain
future diagnostics design.

## Formatting

Canonical flow formatting:

- writes structural indentation as exactly two spaces per level;
- keeps a body-opening `{` on the final physical header line;
- aligns a multiline body-closing `}` with its flow keyword;
- keeps `;;` and `??` surrounded by whitespace;
- keeps `else` aligned with its owning `if`, with no blank line before it and no
  newline between a preceding `}` and `else`;
- uses explicit `\` continuation to move a whole header to the common
  continuation level rather than aligning under a variable-length label; and
- preserves explicit target `:` references and labels.

Exact source spacing and continuation rules are owned by
[source structure](source-structure.md).

## Source stability

The aligned rules deliberately protect against:

- changing a value's type silently changing condition truthiness;
- adding or removing `;;` silently turning a body into a post operation;
- inserting a labeled eligible target silently redirecting a bare transfer;
- adding a local binding changing label resolution, or vice versa; and
- changing one ternary arm silently materializing a runtime-dependent stored
  type.

Some changes remain intentionally source-visible:

- renaming a flow label breaks explicit target references;
- adding or changing an overload may alter a ternary branch's selected operation;
- adding `else` or a post operation changes normal flow work;
- adding a nested unlabeled loop or explicit `scope` may change a bare target;
  and
- changing construction or result shape changes path-completion obligations.

## Boundaries and maturity

This document is current conceptual design, not a formal grammar, implementation
mapping, compatibility contract, or conformance specification. It teaches the
programmer model and depends on the current owners linked above rather than
duplicating their rules.

The following remain explicit future work and are not established here:

- complete `each` iteration protocols, ranges, arrays, and iterator
  customization;
- complete `switch`, `case`, patterns, exhaustiveness, and fallthrough; future
  selection work must reconsider legacy `case continue` under the accepted
  `next`/`continue` distinction;
- complete `using` and resource-management semantics; core `scope` here is an
  explicit flow target, not a resource construct;
- runtime value-polymorphic declarations and stored branch-dependent types;
- `except`, `catch`, and specialized error-result propagation, which must
  preserve ordinary exit and result-completion rules ([except](../except.md));
- complete function capture and closure representation, including callable-like
  scope capture;
- complete pointer and reference lifetime strategies and ownership;
- async suspension, cancellation, executors, and concurrent flow;
- `[[likely]]`, `[[unlikely]]`, and other reachability or likelihood directives;
  and
- formal reachability, definite-initialization, and termination algorithms,
  diagnostic identifiers, and compiler lowering.

For Zax's accepted foundational direction, see the
[language vision](vision.md).
