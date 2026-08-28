# Raw input: indexing and slicing

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work defining indexing, slicing, ranges, bit places, proxies, or multi-index operations |
| Applies To | `operator index`, direct indexed mixfix, slicing, bounds, and projection pressure |
| Owns | Representative source, direct-operation requirements, unsettled syntax, activation pressure, and retirement criteria |
| Does Not Own | Accepted array/index/range syntax, bounds policy, references, proxy types, or implementation |
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

## Activation and retirement

Activate this input for arrays, indexing, slicing, ranges, bit places, bounds,
proxies, or indexed mixfix declarations. Move accepted behavior into array,
index, source, qualifier, lifetime, and operator owners, then retire this file.
