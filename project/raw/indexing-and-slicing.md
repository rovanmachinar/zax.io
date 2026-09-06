# Raw input: indexing and slicing

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work defining indexing, slicing, ranges, bit places, proxies, or multi-index operations |
| Applies To | `operator index`, direct indexed mixfix, slicing, bounds, and projection pressure |
| Owns | Representative source, direct-operation requirements, unsettled syntax, activation pressure, and retirement criteria |
| Does Not Own | Accepted indexing semantics or current integer/lifetime behavior |
| Source / Provenance | Legacy `arrays.md` evidence and operator review |

## Direct indexed operations

Index is a postfix delimited form:

```zax
value := container[index]
```

A custom direct definition derives index count from its prototype. In a flattened
mixfix, `index N` identifies the number of index holes.

Direct indexed mutation may be one operation:

```zax
container[index] = value
container[index] ^= mask
```

It need not produce or capture an intermediate proxy. Receiver, index, and RHS
evaluate once.

If no direct mixfix applies, ordinary decomposition is possible only when indexed
read genuinely returns an assignable place or real proxy:

```zax
proxy := container[index]
proxy = value
```

Future work must decide whether any index result supports such capture.

## Multiple indices and slicing

```zax
matrix[row, column]
```

may use a two-index form. Exact comma mapping, bounds, result shape, and custom
declaration syntax remain unsettled.

Preserve direct splice pressure:

```zax
// Illustrative future syntax.
container[low ..< high] = replacement
```

The operation may expose receiver, low, high, and replacement holes without a
range value or proxy. Future work must decide range syntax, inclusive/exclusive
bounds, omission, evaluation, overlap, aliases, result, and failure.

### Range punctuation candidates

Preserve:

```zax
1..=5
0..<myCount
```

as candidate closed and half-open range forms. `..=` is visually inspired by an
English en dash; `..<` states that the upper endpoint is excluded. Exact syntax
is not accepted.

Bare `..` receives no two-ended range meaning here. Leaving it unassigned avoids
conflict with the strong Rust expectation that `..` is half-open and preserves
it for future open-ended/rest investigation.

Future work must decide:

- whether `..=` includes both endpoints and `..<` excludes the upper endpoint;
- longest-token recognition;
- tokenization beside postfix `.` dereference;
- required grouping around adjacent dereference/range punctuation;
- empty and reversed ranges; and
- whether a range is a value, syntax consumed directly by indexing, or both.

Optional-pointer/dereference adjacency is parser pressure, but this input does
not establish its final grouped spelling.

## Index-size and endpoint pressure

Current `IndexSize`/`TypeSize` capacity and identity relationships are owned by
[Zax integers](../../language/integers.md#counts-sizes-and-indexes). This section
retains container-specific indexing, endpoint, and bounds behavior.

For each ordinary, near, or far memory domain, `IndexSize` and `TypeSize` are
different intent identities with equal numeric capacity and representation:

```text
IndexSize.maximum = TypeSize.maximum
IndexSize representation = TypeSize representation
```

Their declared equal-range relationship permits protected exact `as` bridges in
both directions without making the identities interchangeable.

For a collection of length `L`, valid element positions are `0..L-1` and valid
slice/splice endpoints are `0..L`. Since `IndexSize` already represents `L`, no
separate splice-index type or extra one-past capacity is required.

Future indexing work must decide:

- whether every indexable container uses the profile `IndexSize` or exposes an
  associated narrower index type;
- how element counts, positions, and endpoints are distinguished by consuming
  operations rather than by representation;
- how bounds failure reports;
- how checked multiplication converts element count and element size into
  `TypeSize`; and
- how ordinary, near, and far domains constrain container availability.

## Bit places

Integer bit indexing may expose direct read/write/mutation:

```zax
isSet : Boolean = bits[index]
bits[index] = true
bits[index] = false
bits[index] ^= true
```

The complete operation may map to specialized behavior without making an
individual bit an ordinary addressable `Boolean &`. A lasting bit proxy would
need an explicit type, lifetime, qualification, and cost model.

## Element-place stability

[Zax lifetimes and references](../../language/lifetimes-and-references.md#arrays-and-collections)
requires every dynamic collection operation to expose which element places
continue and which end or relocate.

Future array and indexing work must decide:

- whether an in-capacity append preserves every existing element place;
- whether insertion or removal preserves places before or after the affected
  range;
- whether contiguous growth relocates every element;
- whether segmented growth relocates only one segment;
- how slices and iterators report their stability guarantee;
- whether any owning element view exists;
- and which operations invalidate raw pointers, references, or anchored
  relationships.

Storage capacity and unchanged logical index do not by themselves preserve one
element place. These guarantees are programmer-visible API behavior rather than
hidden implementation choices.

## Activation and retirement

Activate this input for arrays, indexing, slicing, ranges, bit places, bounds,
proxies, or indexed mixfix declarations. Move accepted behavior into array,
index, source, qualifier, lifetime, and operator owners, then retire this file.
