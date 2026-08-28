# Raw input: function composition and chaining

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work defining function composition, capture, chaining, generated callable types, or generic callable reflection |
| Applies To | Preserved `>>`, `|>`, call-component, capture, and reflection pressure |
| Owns | Representative source, generated-signature concepts, costs, boundaries, activation pressure, and retirement criteria |
| Does Not Own | Accepted composition/chaining syntax, callable representation, capture, generics, reflection, or lowering |
| Source / Provenance | Legacy `functions.md` and operator review |

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

## Activation and retirement

Activate this input for composition, chaining, captures, lambdas, partial
application, callable generics, or callable reflection. Move accepted behavior
into function, invocation, operator, generic, or reflection owners and retire
this file when fully dispositioned.
