# Raw input: iteration protocols

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work defining iterator values, iterable inputs, and `each` source |
| Applies To | General iteration protocols and the concrete iterator surfaces required by current enum behavior |
| Owns | Preserved iteration questions, enum iterator requirements, representative pressure, activation, and retirement |
| Does Not Own | Accepted iteration syntax or current enum semantics |
| Source / Provenance | Legacy `each in` / `each from` input and enum promotion requirements |

## Why this input exists

Current [enum behavior](../../language/enums.md#deferred-enum-iteration)
requires iterator-shaped results without deciding the general iterator protocol.
Legacy flow material also preserves `each in` and `each from` iteration forms.

The language needs one coherent iteration design rather than enum-specific
collection machinery.

## Enum member iteration

`EnumType.members()` conceptually returns an iterator over every declared member
in source order.

Each entry must expose at least:

- the declared ASCII member name;
- the enum value; and
- declaration order.

The iterator preserves duplicate-valued aliases. It does not collapse
declarations merely because their values compare equal.

```zax
members := Color.members()
// The exact iterator result type and consumption syntax are deferred.
```

This capability supports reflection-adjacent inspection and programmer-defined
flags formatting without choosing a general reflection model.

## Case-insensitive matching

`EnumType.matchingValuesIgnoringCase(source)` conceptually returns an iterator
over distinct values whose declared names match `source` under ASCII case
folding.

- Equal values are yielded once.
- Values appear in the order of their first matching declaration.
- No match yields an empty iterator.
- An acknowledged case conflict may yield several distinct values.

This operation is different from `members()`: it is query-specific and
deduplicates values, while member iteration preserves every declaration.

## Iterable string input for flags

Flags conversion conceptually accepts an iterable of strings:

```text
Permission.fromStrings(iterable of String) -> Permission?
Permission.fromStringsIgnoringCase(iterable of String) -> Permission?
```

Future iteration work must define:

- which values satisfy the iterable input;
- how elements are obtained and in what order;
- temporary and element lifetimes;
- how iteration reports failure;
- whether iteration can be compile-time, runtime, or either;
- whether an iterator can be consumed once or restarted; and
- how early conversion failure closes or disposes the iterator.

Enum behavior already requires atomic conversion: an unknown or ambiguous
element returns absence rather than a partially accumulated flags value.

## General `each` pressure

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
each member in Color.members() {
  inspect(member)
}
```

## Activation and retirement

Activate this input when iterator values, iterable parameters, `each`, ranges,
or generated enum iteration is reviewed.

That work must return to [Zax enums](../../language/enums.md) and replace its
deferred iterator descriptions with concrete declarations and examples. Move
general behavior into an iteration owner, local behavior into the applicable
domain owners, and retire this file when every preserved concern has a current
owner or explicit disposition.
