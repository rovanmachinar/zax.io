# Raw input: when a terminal use needs a stated stance

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work on transfer stances, type definitions, generics, and diagnostics |
| Applies To | The boundary of the terminal-opportunity intent error, and how a type shows whether taking it over can change anything |
| Owns | The preserved question of which types need a stated stance at a provable last use, the plain-data question, and their activation and retirement |
| Does Not Own | Current transfer-stance meaning, the existing intent rule, or any type classification; those belong to [transfer stances](../../language/transfer-stances.md) and [function invocation](../../language/function-invocation.md#terminal-opportunity-requires-intent) |
| Source / Provenance | Work item `037` maintainer review of per-item stances on expanded packs |

## The concern

Current design makes an unstated stance at a provable last use an intent error
when offering `last` could materially change the accepted contract. It needs
no stance when `last` would accept the same contract and produce the same
source post-state, as for a receiver that can only copy:

```zax
inspect final : ()(value : Record readonly & copy) = {
}

forward final : ()(value : Resource) = {
}

inspect(record)      // fine: no consumer could take record over
forward(resource)    // error at a last use: state copy, move, last, or deep intent
```

The rule decides by what the receiver could do. It does not say whether the
type itself can make a difference. A plain value such as an `Integer` has
nothing to hand over, so copying it and finishing it should be
indistinguishable, and the maintainer's direction is that the compiler should
not require a stance for such types:

```zax
sumAll final : (result : Integer)(values : Integer[]) = {
}

sum final : (result : Integer)(values ... : Integer) = {
  return sumAll([ from values ])   // intended: no stance needed for Integer items
}
```

Generic code cannot know in advance. The same body may be fine for one
processing and need a stance for another:

```zax
collect$ final : (Items ... : type)()(items ... : Items) = {
  all := [ from items as copy ]   // current examples state a stance for generic items
}

collect(1, 2)            // Integer items: a stance changes nothing
collect(myResource)      // a resource-owning item: the stance matters
```

## What future work must decide

- A clear, teachable definition of the types for which taking a value over
  can never change anything, the role "plain old data" plays elsewhere. It
  must cover scalars, fixed arrays and structures of such types, references,
  pointers, optionals, and types with programmer-declared copy or
  destruction behavior.
- Whether a type can state that property, or its opposite ("this type needs a
  stance"), explicitly, and whether it is inferred from members when not
  stated.
- How generic code is checked: per processing, which makes the same source
  valid for some calls and an intent error for others, or once against the
  most demanding possible type, which requires a stance in every generic body.
- How diagnostics explain the decision, so a programmer can tell why one call
  needed a stance and a similar one did not.

## What this does not decide

It does not change the current intent rule, add a type classification, or make
any existing example valid or invalid.

## Activation and retirement

**Activates when** transfer stances, type definitions, or generics are
reviewed for copy, move, and destruction behavior, or when a programmer-facing
example needs to rely on a plain-data exemption.

**Constraint on current work.** Current owner examples state a stance whenever
generic items are passed on for the last time, and leave plain `Integer`
examples unannotated without claiming a rule for them.

Move the accepted definition and any type-level signal into the transfer-stance,
type-definition, generics, and diagnostics owners, then retire this file.
