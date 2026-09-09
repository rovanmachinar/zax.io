# Raw input: generalized pattern matching

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work reviewing patterns, destructuring, guards, or selected payload binding |
| Applies To | Selection shapes that cannot be expressed as current Boolean case tests and ordinary proven access |
| Owns | Preservation of generalized-pattern pressure, recursive destructuring, variant payload binding, guarded patterns, binding convergence, and interaction with direct body entry |
| Does Not Own | Current runtime `switch` behavior, accepted variant syntax, or current optional access |
| Source / Provenance | Former raw selection input consumed by runtime-selection review; work item `018`; raw variants/unions and optional matching pressure |

## Reading posture

Current [switch, case, and default](../../language/switch.md)
defines `case` alternatives as Boolean operations over one retained selector.
Those alternatives introduce no payload bindings and are not generalized
patterns.

This file preserves only the pattern-like remainder. It accepts no syntax or
behavior and is not required for ordinary selection reading.

## Why a distinct future form is needed

Current cases can:

- compare through implicit equality;
- apply direct binary or post-unary tests;
- apply an operandless pre-unary test;
- establish recognized path facts such as optional presence; and
- use ordinary nested selection and postfix access.

They cannot:

- destructure a selected value;
- declare names in a case header;
- bind a variant payload;
- recursively match nested shapes;
- attach a guard that resumes later patterns when false; or
- guarantee one binding shape across structurally different alternatives.

Future pattern syntax must remain visibly distinct from current Boolean case
tests. It must not silently reinterpret existing source such as:

```zax
case expectedValue
case < upperBound
case ?
```

## Optional payload-binding pressure

Current optional selection crosses one layer at a time and uses ordinary proven
access:

```zax
switch nested {
  case ? {
    switch nested. {
      case ?
        use(nested..)
      default
        handleInnerAbsent()
    }
  }
  default
    handleOuterAbsent()
}
```

A future pattern feature may seek a flatter payload-binding form. The following
is illustrative and unaccepted:

```zax
switch nested {
  case present(present(value : MyValue &)) // illustrative future pattern
    use(value)
  case present(absent) // illustrative future pattern
    handleInnerAbsent()
  case absent // illustrative future pattern
    handleOuterAbsent()
}
```

Any accepted form must preserve:

- outer absence, outer-present/inner-absent, and both-present as three distinct
  states;
- one-layer-at-a-time presence proof;
- the exact boxed qualifications and lifetime;
- invalidation when reset, packet construction, wrapper replacement, or another
  operation ends that boxed lifetime; and
- no treatment of a construction packet as an anonymous value.

## Variant and union pressure

Managed variants need active-alternative testing and may need payload binding.
The declaration, storage, replacement, and lifetime model remains in
[raw variants and unions](variants-and-unions.md).

Illustrative pattern source:

```zax
switch event {
  case MessageEvent(message : Message &) // illustrative future pattern
    handle(message)
  case ClosedEvent // illustrative future pattern
    handleClosed()
}
```

Future work must decide how active-alternative proof, payload qualifications,
replacement, destruction, exhaustive coverage, and unnamed or unknown
alternatives interact. Current enum member cases do not provide a variant model.

Unmanaged unions additionally require explicit active-lifetime or unsafe
responsibility. Pattern syntax cannot make several overlaid resident instances
simultaneously live.

## Guards and resumption

A pattern guard would need to decide whether guard failure resumes with later
patterns and which facts survive:

```zax
switch event {
  case MessageEvent(message : Message &) if acceptable(message)
    handle(message)
  default
    reject(event)
}
```

This source is illustrative and unaccepted. Future work must establish:

- whether the guard is exactly `Boolean`;
- when payload bindings become live;
- whether a false guard destroys them before later testing;
- which effects and evaluation order are observable;
- how guards affect overlap and exhaustiveness; and
- whether direct entry can target a guarded body.

## Binding agreement and direct entry

Comma alternatives currently share one body because they introduce no bindings.
Pattern alternatives that bind values would need one usable body environment:

```zax
case Left(value : MyValue &), Right(value : MyValue &) // illustrative
  use(value)
```

Future work must define whether every alternative must establish the same names,
types, qualifications, and lifetimes or whether another explicit convergence
form is required.

Current `goto case_label:` enters a body without running its tests. A future
pattern-bound body therefore cannot be an ordinary direct-entry target unless
every required binding and proof already exists on that incoming path. Possible
designs include rejecting such a target, requiring a separate unbound body
entry, or defining another explicit mechanism. This file chooses none.

## Value-producing selection boundary

A future selection expression may interact with patterns but is independently
motivated. It must define direct result construction, exact path convergence,
exhaustiveness, and transfer restrictions. The current statement switch does
not imply one result slot or a value-bearing `break`.

The current runtime-selection owner retains that boundary. Move it here only if
future pattern and value-selection work become one cohesive feature.

## Activation and retirement

Activate this input when a concrete use case requires:

- destructuring rather than Boolean testing;
- a selected payload binding;
- guarded patterns;
- recursive shape matching;
- variant active-alternative binding; or
- pattern-aware value-producing selection.

Future work must begin with current runtime selection, optional proof, variant
lifetime, construction, qualifier, and direct-entry constraints. Move accepted
behavior into a dedicated pattern owner and applicable domain owners, then retire
this file.
