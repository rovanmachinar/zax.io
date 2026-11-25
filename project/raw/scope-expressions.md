# Raw input: scopes that produce a value

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative; an idea from the maintainer to work through later |
| Audience | A future numbered work item on flow control |
| Applies To | Letting `scope` and other constructs that `break` can target produce a value, so a block can be used as an expression |
| Owns | The idea, the maintainer's direction on the keyword, representative examples, and the questions to settle |
| Does Not Own | Current flow control ([core flow control](../../language/core-flow-control.md)); the conditional expression and branch convergence; `outward` declarations ([declarations and bindings](../../language/declarations-and-bindings.md)) |
| Source / Provenance | Work item `040` (element and contained stances), raised by the maintainer alongside a documentation review |

## The idea

Zax has no way for a block to produce a value. `return` exits the function, as
it should; `break`, `continue`, `next`, and `goto` control flow; `scope`
declares a re-enterable block. Rust lets a block end with an expression that
becomes its value, which depends on `;` separating statements; that is not
Zax's approach. The maintainer would like a value-producing form instead:

```zax
// illustrative
value := 5 + scope {
  if cached
    break cache.value              // leaves the scope with a result
  break compute()
}
```

The result is not a declared property of the scope; it makes the scope act as
an expression, so it can be used anywhere an expression can.

## Direction on the keyword

`break` is well suited, because it already targets a scope and exits it. The
maintainer is open to another word, but whatever is chosen replaces `break` in
this role; there should not be two words for leaving a construct. `break` is a
C-legacy flow word, but not an awful one.

The form applies to every construct `break` can target, not only `scope`, with
the usual label rules:

```zax
// illustrative
limit := scope pick: {
  each candidate : in candidates {
    if candidate.fits()
      break pick: candidate.size   // targets the labelled scope from inside a loop
  }
  break pick: defaultSize
}
```

## Questions to settle

- **Every exit produces a result.** Normal completion, `break` without a
  value, and early exits must each be resolved, and the results must converge
  to one type; the conditional expression's convergence model is the natural
  basis.
- **Re-entry.** `next` and `continue` re-enter a `scope`, so an expression
  scope could retry until it produces a value; post operations must be placed
  relative to the result.
- **Lifetimes.** The result is built before the scope's locals are destroyed,
  so it cannot refer to them.
- **Several results and exceptional outcomes.** Zax functions have several
  results and named exceptional outcomes; decide whether a scope can produce
  more than one value, and how `except` and `return` cross it.
- **Relation to existing features.** The conditional expression
  (`condition ?? a ;; b`) and `outward` declarations already cover some of
  the same needs; the design should say when to use which.

## Legacy note

`basics.md` writes a scope label without the colon that the current label rule
requires (`scope my_scope {` rather than `scope my_scope: {`). Fix it when
`basics.md` is next dispositioned.

## Activation and retirement

Activate with a future flow-control work item, or when examples repeatedly
need `outward` or a helper function only to compute one value. Move accepted
behavior into core flow control and the affected owners, then retire this file
and remove it from [the raw index](README.md).
