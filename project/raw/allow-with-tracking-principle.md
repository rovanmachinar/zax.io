# Raw input: allow with tracking rather than forbid

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative; the maintainer considers it a principle worthy of promotion |
| Audience | Future work on `language/principles.md`, safety contracts, and diagnostics |
| Applies To | Whether Zax forbids a risky construct outright or allows it under specification-defined tracking |
| Owns | The candidate principle, the maintainer's framing, the cases that already follow it, and its boundary |
| Does Not Own | The principles catalog ([principles](../../language/principles.md)); current safety and contract evolution ([safety and analysis](../../language/safety-and-analysis.md)); what compilers must agree on ([compiler agreement](compiler-agreement-principle.md)) |
| Source / Provenance | Work item `040` (element and contained stances), raised by the maintainer while discussing finished members and source tracking |

## The candidate principle

> Prefer allowing a construct with specification-defined tracking and a clear
> diagnostic over forbidding it outright, provided the tracking is
> conservative wherever it cannot see.

## The maintainer's framing

There is always tension between "we simply don't allow it" and "we allow it,
but it could bite you in later updates". Taken to the extreme, the first would
allow only functions with no side effects at all: close to perfect, but very
restricted, and forcing one programming pattern. The other extreme allows
anything and reports nothing. The middle ground lets an API author cause some
problems, but those problems are trackable, and the language tries to keep the
author safe and honest.

## Cases that already follow it

- **Finishing array elements in place.** Writing `last` on an element that
  stays in its array is allowed; the array becomes partially finished, and a
  later use the compiler cannot prove safe is the error:

  ```zax
  consumeAll(messages[2..<5] as last)  // allowed
  messages[0].send()                   // fine: proven outside the finished range
  dispatch(messages[j] as last)        // error unless j is proven different
  ```

- **Members finished through a borrower,** under the direction recorded in
  [usage and provenance tracking](usage-and-provenance-tracking.md): allowed,
  with the error on a later use that source tracking finds.
- **Raw pointers made from references,** under the same direction: allowed when
  source tracking proves no path reaches a `unique` source.

## Boundary

The tracking must be the specification's, not each compiler's, as
[what compilers must agree on](compiler-agreement-principle.md) requires.
Where tracking stops, such as at a compiled, exported ABI, the language is
conservative: the construct becomes an error there, or needs `unsafe`.

## Activation and retirement

Activate when the principles catalog is next reviewed, together with
[what compilers must agree on](compiler-agreement-principle.md). Retire this
file once the principle is accepted into `language/principles.md` or
explicitly rejected, and remove it from [the raw index](README.md).
