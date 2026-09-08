# Raw input: iteration and traversal

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work defining `each`, compiler-known traversal, and eventually iterator and iterable protocols |
| Applies To | Direct traversal source and the future abstraction mechanisms that may generalize it |
| Owns | Preserved `each` questions, enum traversal requirements, future concept-dependent iterator pressure, representative source, activation, and retirement |
| Does Not Own | Accepted iteration syntax or current enum semantics |
| Source / Provenance | Legacy `each in` / `each from` input and enum promotion requirements |

## Why this input exists

Current [enum behavior](../../language/enums.md#deferred-enum-traversal)
requires member and case-insensitive-match traversal. Zax cannot yet promise
first-class iterator values because the concepts and constraints needed to
express a general iterator protocol are not current.

Legacy flow material preserves `each in` and `each from` iteration forms. An
initial iteration design can use those forms over compiler-known sources without
silently designing concepts.

The language ultimately needs one coherent iteration design rather than
enum-specific collection machinery, but direct `each` behavior and first-class
iterator abstraction may be developed in separate stages.

## Enum member traversal

Enum traversal must visit every declared member in source order.

Each entry must expose at least:

- the declared ASCII member name;
- the enum value; and
- declaration order.

Traversal preserves duplicate-valued aliases. It does not collapse
declarations merely because their values compare equal.

```zax
// Illustrative legacy pressure; exact `each` syntax is not established.
each member : in Color {
  inspect(member)
}
```

This capability supports reflection-adjacent inspection and programmer-defined
flags formatting without choosing a general reflection model or creating a
first-class iterator value.

## Case-insensitive matching

Case-insensitive match traversal must visit the distinct values whose declared
names match one input string under ASCII case folding.

- Equal values are visited once.
- Values appear in the order of their first matching declaration.
- No match executes the traversal body.
- An acknowledged case conflict may yield several distinct values.

This traversal is query-specific and deduplicates values, while member traversal
preserves every declaration.

## Several string inputs for flags

Flags conversion needs a future way to consume several strings:

```text
Permission.fromStrings(sequence of String) -> Permission?
Permission.fromStringsIgnoringCase(sequence of String) -> Permission?
```

No general iterable parameter can be declared until concepts or another
constraint mechanism can express its requirements. Earlier work may support a
concrete array or another compiler-known source without treating that local
facility as the general protocol.

Future work must define:

- which concrete sources are initially accepted;
- how elements are obtained and in what order;
- temporary and element lifetimes;
- how iteration reports failure;
- whether iteration can be compile-time, runtime, or either;
- how early conversion failure cleans up the source; and
- when concepts are sufficient to expose a general iterable input.

Enum behavior already requires atomic conversion: an unknown or ambiguous
element returns absence rather than a partially accumulated flags value.

## Direct `each` pressure

Legacy source proposes `each in` and `each from`. Exact syntax and semantics are
not current. Future work must determine:

- the iterable/iterator boundary;
- forward, reverse, keyed, and range iteration;
- binding and element-reference behavior;
- termination and mutation during iteration;
- cleanup on normal completion and control transfer;
- costs and allocation;
- integration with arrays, slices, variadics, generated ranges, and enums; and
- whether `each in` and `each from` remain distinct.

Illustrative future source:

```zax
// Exact `each` syntax is not established.
each member : in Color {
  inspect(member)
}
```

## First-class iterator pressure

True iterator values remain dependent on concepts or an equivalent constraint
system able to state:

- how a source exposes an element type and next-step operation;
- whether iteration is single-pass or restartable;
- reference and element-lifetime relationships;
- cleanup after completion, early exit, or failure;
- generic viability and overload selection; and
- whether an iterator can itself be copied, moved, stored, or returned.

Direct `each` work must not invent these contracts implicitly. It should leave a
clear future handoff from compiler-known traversal to concepts-based protocols.

## Activation and retirement

Activate this input first for `each` and compiler-known traversal. Activate its
first-class iterator portion only when concepts or another sufficient constraint
model is current.

That work must return to [Zax enums](../../language/enums.md) and replace its
deferred traversal description with concrete declarations and examples. Move
direct traversal into its current owner, retain concept-dependent iterator
pressure in an indexed live destination, and retire this file only when every
preserved concern has a current owner or explicit disposition.
