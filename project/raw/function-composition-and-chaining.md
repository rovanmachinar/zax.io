# Raw input: function composition and chaining

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work defining function composition, capture, chaining, generated callable types, or generic callable reflection |
| Applies To | Preserved `>>`, `|>`, call-component, capture, optional callable construction/combinators, and reflection pressure |
| Owns | Representative source, generated-signature concepts, optional callable/combinator pressure, costs, boundaries, activation pressure, and retirement criteria |
| Does Not Own | Accepted composition semantics or current invocation/operator ownership |
| Source / Provenance | Legacy `functions.md`, operator review, and work item `012` optional callable/combinator pressure |

## Composition and capture pressure

Legacy composition joins compatible callable results and inputs:

```zax
combined := first >> second
```

Value capture may create a new callable without invoking it immediately:

```zax
bound := 55 >> process
```

The generated result may have an unnameable concrete type containing capture
state while remaining compatible with a visible callable prototype. Future work
must define:

- which `>>` operand/result shapes are language-generated;
- output-to-input mapping;
- capture evaluation and lifetime;
- allocation, copy, move, and indirect-call costs;
- chaining versus composition;
- overload and generated-candidate preference;
- protected all-callable combinations;
- lambda and closure type identity;
- and reflection.

[Zax transfer stances](../../language/transfer-stances.md) now constrains
capture:

- constructing the callable is the transfer consumer; later invocation is not
  the capture transfer point;
- a by-value capture owns its captured storage and may adopt a declaration
  stance;
- a reference capture behaves like an alias and does not silently inherit `move`
  or `last` from the source declaration;
- destructive use through a reference capture requires an explicit
  `as move` or `as last` restatement;
- repeated invocation must not accidentally repeat a one-time destructive
  transfer; and
- forwarding through generated callable types must preserve the exact accepted
  stance and affected object.

Exact capture syntax, once-only callable contracts, lifetime, allocation, and
reflection remain future lambda/callable work.

`>>` retains the operator catalog's shift-level precedence for every use. Types
cannot assign composition another precedence.

## Invocation chaining

`|>` remains a separate chaining candidate:

```zax
value |> process
```

It may use the same general generated-candidate machinery without sharing
composition semantics.

## Call-form mixfix and reflection

A custom callable may use a `call N` mixfix component:

```zax
result := callable(5, "apple") + adjustment
```

Future work must integrate call labels, defaults, variadics, result forwarding,
and generated lambda types.

Preserve generic/reflection pressure:

```zax
myReflectionAnalyzer << myFunc
```

At least one custom operand keeps this outside an all-intrinsic signature, but
complete generic and reflection behavior remains unsettled.

## Optional callable construction pressure

[Zax optional values](../../language/optional-values.md) requires a construction
packet to establish a boxed function value in an existing optional:

```zax
callback : MyCallback?
callback = [{ doSomethingFunc }]
```

A future inline callable value may contain a braced body:

```zax
callback = [{ { doSomething() } }]
```

That source is illustrative, not accepted. Function-value work must decide
whether the inner braces construct a callable, create a scope, or require a
prior named value. It must preserve `[{}]` as the explicit zero-entry packet and
`[{ expression }]` as one packet input rather than confusing callable-body and
packet boundaries.

## Optional combinator pressure

Optional values may benefit from present/absent callable composition:

```zax
fold(optionalValue, onPresent, onAbsent)
```

Possible `on some`, `on none`, mapping, and folding operations must decide:

- only the selected callback executes;
- how a present callback receives boxed qualifications and lifetime proof;
- how outermost `copy`/`deep`/`move`/`last` stance reaches the callback;
- result convergence between present and absent callbacks;
- capture, temporary, allocation, and indirect-call costs;
- async callback behavior;
- protected language operation versus library API ownership.

`optionalValue.onSome(...)` is not neutral candidate syntax because postfix `.`
already crosses into the boxed value. Exact source must respect current optional
access and operator-phrase rules.

This concrete pressure does not establish a general monad abstraction. If later
work finds a reusable model spanning optionals, results, collections, async
values, and composition laws, it may justify a separately indexed owner.

## Activation and retirement

Activate this input for composition, chaining, captures, lambdas, partial
application, callable generics, callable reflection, optional callable
construction, or optional combinators. Move accepted behavior into function,
invocation, operator, optional, generic, selection, or reflection owners and
retire this file when fully dispositioned.
