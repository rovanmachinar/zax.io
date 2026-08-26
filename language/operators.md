# Zax operators

| Field | Value |
| --- | --- |
| Status | Current conceptual design |
| Audience | Human developers reading, writing, or evaluating Zax |
| Applies To | Programmer-facing operator behavior accepted so far; not a formal grammar or specification |
| Implementation State | Not established by this repository |
| Owns | The fact that operator results may be arbitrary types; direct pre-unary `?` and `!` selection, direct ambiguity, and opposite-operator fallback; explicit typed result selection through an anonymous typed declaration; ordinary overloadable eager `&&` and `\|\|`; the protected language-provided exact `Boolean`/`Boolean` `&&` and `\|\|` operations and their left-to-right short-circuit behavior; branch-path operator-node classification and compile-time validity of every path; the optional-presence operation `?value`; operator costs, diagnostics, and source stability at the depth accepted so far |
| Does Not Own | Complete operator declaration syntax, operator catalogs, lookup domains, priority, ranking and precedence, literal operators, word and compound-word operators, conversion operators, or general generated operators ([legacy operator overloading](../operator.md) and future operator work); shared fixed-arity callable viability and narrow result-context selection ([function invocation](function-invocation.md)); condition placement, selected-arm order, and branch convergence ([core flow control](core-flow-control.md)); complete optional representation, construction, reset, and unwrapping ([legacy optional types](../optional.md)); proof that conditionally live storage holds a value before access ([construction, replacement, and destruction](construction-and-destruction.md)); the general boundary between required language support and optional libraries ([language vision](vision.md#required-language-support-is-not-an-optional-library)); compiler, linker, runtime, or helper implementation |
| Source / Provenance | Legacy [operator overloading](../operator.md), [Nothing type instances](../nothing.md), and [optional types](../optional.md) evidence |

## Mental model

An operator is a callable selected by its symbol and operands.

This document is the current conceptual owner for accepted programmer-visible
operator behavior. It begins with the semantics accepted so far and grows when
future reviewed operator behavior belongs cohesively here. It is a cumulative
owner rather than the remainder of another concern, and unreviewed operator
material stays deferred because it has not been reviewed, not because it falls
outside flow control.

```zax
myType : MyType = ?extractedType
```

An operator produces a value like any other callable. Its result type is whatever
the selected overload returns; there is no operator-specific truthiness rule.

## Operator results may be arbitrary types

A directly selected `?` or `!` overload may return any type, not just `Boolean`:

```zax
myType : MyType = ?extractedType
```

Where a `Boolean` is required, the already selected result is validated. A
non-`Boolean` result is an error; the context does not silently choose a
different, result-only overload merely because `Boolean` is wanted:

```zax
// error if the selected ? overload returns MyType rather than Boolean
if ?extractType()
  useResult()
```

Which contexts require a `Boolean` is owned by
[core flow control](core-flow-control.md). This document owns what the operator
itself produces.

## Direct selection, ambiguity, and opposite-operator fallback

Pre-unary `?` and `!` are ordinary overloadable operators. Selection happens
normally first, and a condition never supplies an expected `Boolean` result to
that selection:

1. resolve the directly written operator under ordinary selection;
2. a condition does not provide an expected `Boolean` result to that selection;
3. one applicable direct overload is used even when its result later fails a
   `Boolean` requirement;
4. ambiguous direct selection is an error and does not activate fallback;
5. only the absence of any applicable direct overload activates opposite-operator
   fallback;
6. resolve the opposite operator normally, without filtering its candidates to
   `Boolean` results;
7. fallback is available only when that opposite selection is unambiguous and
   returns exactly `Boolean`; and
8. an explicit anonymous typed declaration may supply the result context that an
   ordinary condition does not.

So `?value` may become `!!value` only when no applicable direct `?` exists and
the normally resolved `!value` is unambiguous and returns exactly `Boolean`, and
`!value` may become `!?value` under the mirrored conditions.

The following representative declarations use legacy candidate operator syntax
only to make the overload sets concrete; complete declaration grammar remains
future operator work:

```zax
A :: type {
  operator pre unary '?' final : (result : Boolean)() = { ... }
}

B :: type {
  operator pre unary '?' final : (result : Boolean)() = { ... }
  operator pre unary '!' final : (result : Boolean)() = { ... }
}

C :: type {
  operator pre unary '?' final : (result : Boolean)() = { ... }
  operator pre unary '?' final : (result : WhateverType)() = { ... }
  operator pre unary '!' final : (result : Boolean)() = { ... }
}

D :: type {
  operator pre unary '?' final : (result : Boolean)() = { ... }
  operator pre unary '?' final : (result : WhateverType)() = { ... }
}

E :: type {
  operator pre unary '!' final : (result : Boolean)() = { ... }
}
```

```zax
a : A
b : B
c : C
d : D
e : E

if ?a               // direct Boolean ?
  print(a)

if !a               // fallback through the unique Boolean ?: !?a
  print(a)

if ?b               // direct Boolean ?
  print(b)

if !b               // direct Boolean !
  print(b)

if ?c               // error: direct ? is ambiguous; a condition cannot choose Boolean
  print(c)

if : Boolean = ?c   // explicit typed result selection chooses the Boolean ?
  print(c)

if !c               // direct Boolean !
  print(c)

if ?d               // error: direct ? is ambiguous
  print(d)

if : Boolean = ?d   // explicit typed result selection chooses the Boolean ?
  print(d)

if !d               // error: fallback cannot filter ambiguous ? candidates
  print(d)

if ?e               // fallback through the unique Boolean !: !!e
  print(e)

if !e               // direct Boolean !
  print(e)
```

An ambiguous direct set is never rescued by the opposite operator, and an
ambiguous opposite set is never narrowed by the `Boolean` a condition wants. The
anonymous typed declaration `: Boolean = ?c` is the accepted way to ask for the
`Boolean`-returning overload explicitly. Which contexts require a `Boolean` is
owned by [core flow control](core-flow-control.md); the narrow expected-result
boundary that makes the typed form work is owned by
[function invocation](function-invocation.md#narrow-expected-result-selection).

A type may define both Boolean-returning complement forms to avoid the extra
negation:

```zax
if !_
  return
```

Independently maintaining both complement operations is a possible lint concern
rather than a language error. Exact lint and suppression syntax remains future
analysis work.

## `&&` and `||`

### Ordinary eager overloads

`&&` and `||` are not globally `Boolean`-only. For operand shapes other than
exact `Boolean`/`Boolean`, they are ordinary overloadable operators selected the
usual way, and they may return any type:

```zax
A :: type {
  operator binary '||' final : (result : Boolean)(rhs : A) = { ... }
  operator binary '&&' final : (result : Boolean)(rhs : A) = { ... }
  operator pre unary '?' final : (result : Boolean)() = { ... }
}

B :: type {
  operator binary '||' final : (result : FooBarType)(rhs : B) = { ... }
  operator binary '&&' final : (result : FooBarType)(rhs : B) = { ... }
}
```

```zax
a1 : A
a2 : A
a3 : A
b1 : B
b2 : B

// error unless a Boolean || A overload also exists:
// A || A returns Boolean, so the next node is Boolean || A
result1 := a1 || a2 || a3
result3 := b1 || b2       // eager ordinary overload; result is FooBarType
result4 := b1 && b2       // eager ordinary overload; result is FooBarType
```

For those operand shapes both operands are evaluated left to right before the
ordinary operator call. There is no short-circuiting, because an arbitrary
overload's result cannot be predicted from the left operand alone.

### Protected exact `Boolean`/`Boolean` operations

The exact language-provided `Boolean || Boolean` and `Boolean && Boolean`
operations are protected:

- they evaluate left to right;
- they short-circuit runtime evaluation of the right operand;
- source must still resolve and validate the right operand at compile time; and
- a user declaration attempting to replace either exact operation is a compile
  error.

```zax
// error: the protected Boolean || Boolean operation cannot be replaced
operator binary '||' final :
  (result : Boolean)(lhs : Boolean, rhs : Boolean) = {
  return asmCompare(lhs, rhs)
}
```

Because the right operand of a protected `&&` runs only after the left operand
produces `true`, short-circuit order is a source-visible guarantee that
conditions and proofs may rely on:

```zax
if ready && ?resource
  use(resource)
```

### Branch-dependent short-circuiting

A conditional expression may give one `&&` or `||` node different resolved
behavior on different paths:

1. statically resolve every possible conditional arm and operator path;
2. classify each resolved operator node on each path;
3. an exact `Boolean`/`Boolean` node uses protected short-circuiting;
4. any other valid operand pair uses ordinary eager overload behavior;
5. left-to-right runtime order and selected-arm-only evaluation are preserved;
6. runtime skips a right operand only on a path classified as `Boolean`
   short-circuit;
7. every possible path remains subject to compile-time validation; and
8. every path must converge by the complete operation boundary.

```zax
result2 := (?a1) || (?a2) // both sides resolve Boolean; right may short-circuit
```

```zax
result5 :=
  (?a1) ||
  (c < d ?? (?a2) ;; e) ||
  (?a3) ||
  (g < h ?? (?a3) ;; f)
```

That expression is resolved as a branch graph. Some paths may use protected
`Boolean` short-circuit nodes while others require ordinary eager overloads. No
path may escape the complete expression with an unresolved result shape.

Short-circuiting skips runtime evaluation, not compile-time resolution.
Determining the type of a later expression never requires evaluating it early at
runtime, and compile-time resolution never implies early runtime evaluation.

These chained forms group and progress left to right. Precedence relative to
other operators remains future operator work.
Selected-arm order and convergence are owned by
[core flow control](core-flow-control.md#conditional-expression-and-branch-convergence),
and which callable each arm selects is owned by
[function invocation](function-invocation.md#branch-specific-selection-under-a-conditional-expression).

## Optional presence operation

`?value` used as a presence operation reports whether an optional holds a live
value and produces `Boolean` at the depth flow requires. The general proof form
tests presence and then dereferences in the proven body:

```zax
if ?optionalValue
  use(optionalValue.)
```

Postfix optional dereference resolves before a pre-unary operator, so `?value.`
means `?(value.)`. A compound proof such as `?optionalValue && ?optionalValue.`
is safe only when the right expression resolves unambiguously to exactly
`Boolean`, because only then is the node a protected short-circuit node. If the
underlying `?` returns another type the node is an ordinary eager
`Boolean && WhateverType` overload that evaluates the dereference regardless, and
if its direct selection is ambiguous the source is an error. Make the intent
explicit when the underlying value's `?` operation is genuinely needed:

```zax
if ?optionalValue && (: Boolean = ?optionalValue.)
  use(optionalValue.)
```

The typed right operand makes both `Boolean` short-circuit eligibility and result
selection explicit.

An arbitrary user-defined Boolean-returning `?` does not by itself prove that
storage is live; recognizing an optional-presence contract and proving that
conditionally live storage holds a value before access are owned by
[construction, replacement, and destruction](construction-and-destruction.md#conditionally-live-storage-and-access-proof).
Complete optional representation and state transitions remain
[legacy optional](../optional.md) input until a focused review.

## Language-provided protected operations

The exact `Boolean && Boolean` and `Boolean || Boolean` operations are provided by
the language and cannot be replaced by user declarations. They are available
without requiring an optional library API.

This section owns only that operator-specific guarantee. The general distinction
between required language support and optional libraries is defined by the
[language vision](vision.md#required-language-support-is-not-an-optional-library).
How a compiler or toolchain realizes required support is outside operator
semantics and is not promised here.

## Shared callable selection

Operators share the fixed-arity callable viability, narrow result-context
selection, and partial-order preference owned by
[function invocation](function-invocation.md). This document does not restate
those rules or introduce a second selection model. An operator used as an
expression is subject to the same once-only source evaluation and the same
branch-specific selection under a conditional expression.

## Costs

- A directly selected operator overload pays its ordinary call cost.
- A synthesized `?`/`!` Boolean fallback adds one negation that a direct overload
  may avoid.
- Protected `Boolean` `&&` and `||` pay only for the operands they actually
  evaluate.
- An eager `&&` or `||` overload pays for both operands and the call.
- Defining both complement operators may avoid the fallback negation, but can
  increase generated code size and maintenance cost and may raise a lint.

## Diagnostics

Operator diagnostics at this depth distinguish:

- a directly selected `?` or `!` result that is non-`Boolean` where a `Boolean`
  is required;
- an ambiguous direct `?` or `!` selection, which is an error rather than a
  fallback trigger;
- no applicable direct operator and no unambiguous exactly-`Boolean` opposite
  operator for a fallback;
- a user declaration of a protected exact `Boolean`/`Boolean` `&&` or `||`
  operation;
- a branch path whose operator node cannot resolve or converge; and
- a presence operation used as proof without a recognized optional-presence
  contract.

Complete operator-selection diagnostics are owned by
[function invocation](function-invocation.md); exact identifiers remain future
diagnostics design.

## Source stability

- Adding or changing an operator overload may change a selected result type and,
  through it, whether a condition still type-checks.
- Adding a second `?` or `!` overload can turn a previously unambiguous direct
  selection into an ambiguity error.
- Adding a direct `?` or `!` overload suppresses a previously synthesized
  opposite-operator fallback.
- Defining both Boolean complement forms stabilizes direct selection but creates
  the maintenance lint above.
- Changing an operand's type may move a `&&` or `||` node between the protected
  short-circuit operation and an ordinary eager overload, which changes whether
  the right operand runs.
- Protected operations cannot be replaced, so another source declaration cannot
  change their behavior.

## Boundaries and maturity

This document is a cumulative but currently bounded owner. It accepts only the
operator behavior reviewed so far and does not yet accept:

- complete operator declaration syntax or the operator catalog;
- operator lookup domains, priority, ranking, precedence, or associativity;
- literal operators, word operators, or compound-word operators;
- conversion operators or `as`-style boundaries; or
- general generated or compiler-provided operators beyond those established here.

That material remains [legacy operator overloading](../operator.md) input and
future focused work. Later operator design must preserve the arbitrary-result
fact, direct-before-fallback selection, ambiguity as an error rather than a
fallback trigger, the unambiguous exactly-`Boolean` fallback constraint, ordinary
eager `&&`/`||` overloads, and the protected `Boolean` short-circuit operations
established here.

For Zax's accepted foundational direction, see the
[language vision](vision.md).
