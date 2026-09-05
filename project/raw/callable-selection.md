# Raw input: callable selection

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work refining overload, generic-specialization, and callable-prototype selection |
| Applies To | Callable candidate comparison beyond the currently accepted fixed-arity partial-order baseline |
| Owns | Naturally unselectable declaration pressure, cross-axis comparison, exact-prototype resolution, generic-specialization comparison, generated-versus-declared ambiguity, source compatibility, activation pressure, and retirement criteria |
| Does Not Own | Accepted current invocation behavior; `switch`/`case` or pattern-like selection; generic syntax; or runtime dispatch implementation |
| Source / Provenance | Transfer-stance review of value/reference, qualification, stance, result, generated-family, and exact-prototype selection |

## Why this input exists

[Zax function invocation](../../language/function-invocation.md) establishes a
partial-order model:

- candidates compare slot by slot;
- one candidate dominates only when it is no worse everywhere and better
  somewhere;
- incomparable candidates produce ambiguity;
- and declaration, import, source, or discovery order never breaks a tie.

Transfer-stance review adds concrete cases that need a later complete preference
design without weakening that model.

## Legal declarations may be naturally unselectable

Distinct declarations remain legal even when every currently obvious natural
call is ambiguous:

```zax
process final : ()(
  input : Message move
) = {
  // Copy in a private value, then process it under move stance.
}

process final : ()(
  input : Message mutable readonly & move
) = {
  // Alias caller storage through readonly access.
}
```

Some source may not distinguish these contracts. The compiler need not prove at
declaration time that each declaration will eventually be naturally selectable.
A use reports ambiguity when several viable candidates remain nondominated.

Exact canonical duplicate signatures remain redeclaration errors.

A private declaration outside its permitted caller context is ineligible before
preference. It does not become an unavailable best candidate or block a public
declaration. **Unavailable** applies only to an eligible visible shape that
cannot be invoked, such as a bodyless, forbidden, or unsupported generated
declaration.

## Exact-prototype resolution

A typed function alias or resolved function value can select one complete
prototype:

```zax
myProcess : ExactProcessPrototype = process
myProcess(message)
```

This is intentional contract selection. It is not merely a workaround for
incomplete overload rules.

Future work must define:

- exact prototype spelling;
- when generic arguments or qualifiers are inferred;
- visibility and availability;
- whether aliases preserve labels and defaults;
- interaction with generated declarations;
- and diagnostics when no declaration has that exact shape.

Future `own` work has a complete prototype when deciding which callable surface
is promoted and should use the same exact-signature discipline.

## Cross-axis comparison

A callable slot may differ in several independent dimensions:

- base type and conversion;
- mutable/immutable value shape;
- final/varying place shape;
- writable/readonly access;
- value versus reference or pointer binding;
- offered and accepted transfer stance;
- source effect;
- and result compatibility.

Transfer fallback contributes one comparison:

```text
last -> move -> copy
move -> copy
deep -> no fallback
copy -> no fallback
```

It does not automatically dominate every other axis.

Examples requiring future comparison work include:

- exact `last` by value versus fallback `move` by writable reference;
- exact `move` through readonly reference versus fallback `copy` by value;
- better stance match versus better type conversion;
- better qualification match versus better expected result;
- and explicit receiver stance versus unstanced receiver fallback.

When no candidate dominates across every applicable slot and dimension,
ambiguity remains the correct outcome.

## Value versus reference transfer

For otherwise equal `move` or `last` candidates, a mutable/writable reference is
better than a by-value form when the source directly supplies that authority:

```zax
process(message as move)
```

The reference candidate acts on caller-owned storage without inserting a
copy-in barrier.

A readonly restatement can make that candidate nonviable:

```zax
process((message as readonly) as move)
```

If a readonly-reference `move` candidate also exists, the readonly reference and
by-value candidates may remain incomparable.

Future work must preserve the distinction:

- value form means `copy` in and process independent storage;
- reference form means operate on caller-visible storage;
- cheaper execution alone does not justify changing ownership semantics.

## Receiver comparison

Receiver stance is one independent callable slot.

- An exact receiver stance admits only that stance.
- An unstanced receiver declaration is a fallback for every stance.
- Exact receiver stance is better than fallback when other comparisons permit
  dominance.
- Receiver matching does not use the source stance fallback ladder.

Future work must integrate receiver comparison with parameter and result slots
without making receiver stance a lexicographic winner over every other
dimension.

A directly constructed compiler-managed unnamed by-value temporary inherently
offers `last` as its receiver/source stance. This is a stable source category,
not liveness analysis of a named declaration.

## Result-mapping intent

An omitted owned by-value result remains implicit `copy`. At its structurally
final mapping boundary, current invocation behavior requires explicit intent
when offering `last` could materially change the accepted consumer or source
outcome.

Future selection work must preserve:

- producer result stance as public prototype information;
- caller-side visibility of the complete destination hole;
- per-result-slot comparison;
- no terminal inference for reference results;
- and no intent error when `last` changes neither accepted contract nor source
  outcome.

Generic and generated result families must expose enough exact metadata for that
mapping decision without treating every temporary reference or pointer pointee
as terminal.

## Generic specializations

Future explicit qualifier generics may produce exact demanded specializations:

```zax
// Illustrative only.
process$(Mutability, PlaceStance) final : (
  result : Message Mutability readonly PlaceStance &
)(
  input : Message Mutability readonly PlaceStance &
) = {
}
```

Selection work must decide:

- when qualifier or type arguments are inferred;
- how constraints affect viability;
- whether a concrete declaration is better than an inferred specialization;
- how several valid specializations compare;
- when an invalid specialization remains latent;
- and how diagnostics expose inferred metadata.

Ordinary qualifier omission never creates genericity.

## Generated and declared candidates

Generated lifecycle declarations participate like exact declarations. A
programmer declaration that fails to match a generated shape may coexist with
it and create ambiguity.

Future selection work must preserve:

- exact generated signature identity;
- demand-driven generation;
- body/default/existing/bodyless/forbidden availability;
- unavailable-best behavior;
- concrete specialization identity;
- and diagnostics showing the generated and programmer-declared differences.

## Source compatibility

Adding or removing a declaration, generic specialization, generated shape, or
preference relation may:

- make previously unique source ambiguous;
- make an unavailable declaration uniquely best;
- or make previously invalid source valid.

It must never silently reinterpret a tie by declaration, import, source, module,
or discovery order.

Future revisions may improve the comparison model while Zax remains conceptual
design. Once implementations, users, and compatibility commitments accumulate,
preference changes require an appropriately higher threshold and migration
guidance.

## What this input does not decide

This file does not establish:

- a complete preference table;
- a generic syntax or constraint system;
- exact callable-prototype syntax;
- runtime dispatch;
- template or specialization implementation;
- formal diagnostic identifiers;
- or `switch`/`case` and pattern-matching behavior.

## Activation and retirement

Activate this input when callable preference, overload resolution, generic
specialization, exact function aliases, generated declaration comparison, or
`own` surface selection is reviewed.

That work must:

1. preserve partial-order dominance and loud ambiguity;
2. define each comparison dimension and its interaction with results;
3. test value/reference, qualifier, stance, receiver, generic, and generated
   cases together;
4. move accepted behavior into invocation, declaration, generic, composition,
   and reflection owners;
5. update source-compatibility guidance; and
6. remove this file from the raw index and retire it when no unresolved callable
   selection input remains.
