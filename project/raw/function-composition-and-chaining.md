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
- ordinary capture defaults to `copy`; when the captured name is a reference,
  this copies the referent into the callable's capture path rather than storing
  another reference;
- a by-value capture owns its captured storage and may adopt a declaration
  stance;
- an explicit reference capture behaves like an alias, retains the original
  fixed target, requires lifetime proof for every invocation, and does not
  silently inherit `move` or `last` from the source declaration;
- destructive use through a reference capture requires an explicit
  `as move` or `as last` restatement;
- repeated invocation must not accidentally repeat a one-time destructive
  transfer; and
- forwarding through generated callable types must preserve the exact accepted
  stance and affected object.

Exact capture syntax, once-only callable contracts, lifetime, allocation, and
reflection remain future lambda/callable work.

Preserve both legacy explicit reference-capture shapes:

```zax
// Legacy illustrative syntax.
callback := [&source, alternate : & = otherSource] {
  use(source)
  use(alternate)
}
```

`&source` keeps the source name. `alternate : & = otherSource` proposed a named
reference capture. Neither spelling is accepted yet.

Legacy `scope` input also proposed callable-like capture:

```zax
// Legacy illustrative syntax.
scope [myInput, &myOutput] {
  myOutput = transform(myInput)
}
```

Future work must decide whether this remains an independently useful restricted
scope form or is better expressed by ordinary declarations, a lambda, or a
direct call. It must preserve the current capture rule: ordinary capture copies
the source value, while explicit reference capture borrows one fixed target and
requires lifetime proof. If callable-like restricted scope remains, names not
listed in its capture list are not visible inside that scope.

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
