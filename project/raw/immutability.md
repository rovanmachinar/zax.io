# Raw input: path-based immutability and a "forever" form

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative; direction set by the maintainer |
| Audience | A follow-up numbered work item on immutability |
| Applies To | What an `immutable` path promises, conversions to and from `immutable`, and a possible form for values that never change |
| Owns | The decisions already taught for path-based immutability and how safety depends on them, the open questions, the "forever" form and its options, and the maintainer's reasoning |
| Does Not Own | Current qualifier teaching ([qualifiers](../../language/qualifiers.md#ordinary-promise-strengthening)); `unsafe cast` ([conversions and casts](../../language/casting.md#qualification)); safety claims ([safety and analysis](../../language/safety-and-analysis.md)); split `mutable`/`immutable` representations ([mutability-indexed type families](mutability-indexed-type-families.md)) |
| Source / Provenance | Work item `041` (compiler-verified unsafe), classification of conversions to `immutable` |

## Path-based immutability

Current [qualifiers](../../language/qualifiers.md#ordinary-promise-strengthening)
teaching treats these decisions as true. Follow-up work is expected to refine
them for corner cases; if it revises or reverses one, it must also revisit the
safety classifications listed below that depend on it.

```zax
counter : Counter mutable writable = makeCounter()
snapshot : Counter immutable readonly & = counter   // allowed if analysis proves it
counter.increment()                                 // error: counter is written while snapshot is held
inspect(snapshot)

frozen : Counter immutable = makeCounter()
reader : Counter mutable readonly & = frozen        // allowed: same type, no marker
writer : Counter mutable writable & = frozen        // needs unsafe: breaks the promise to frozen's holders
```

Decisions made:

- **The default meaning of `immutable` is path-based.** While you hold an
  `immutable` path, the value cannot change. When you no longer hold it, the
  guarantee no longer applies.
- **`immutable` to `mutable readonly` is allowed** when both are the same
  type. The new holder is told something weaker and cannot write.
- **`mutable` to `immutable` is allowed when analysis proves the path keeps
  the promise.** `mutable readonly` to `immutable` follows the same analysis;
  being `readonly` does not help in this direction, because a writable path
  may exist elsewhere. If the specification proves it, no marker is needed;
  otherwise a `safe<no-mutation-while-held>` claim asserts it, and cleverer
  compilers may verify it.
- **`immutable` to `mutable writable`** breaks the promise to every holder, so
  it stays an `unsafe` operation (`unsafe pliable` or `unsafe cast`).
- **Split types**, whose `mutable` and `immutable` variants have different
  representations, allow no conversion.

The category name `no-mutation-while-held` was chosen during promotion and may
be revised.

Safety classifications that depend on these decisions:

- conversions to `immutable` are analysis or `safe<no-mutation-while-held>`
  sites rather than `unsafe cast`;
- only the conversion to `mutable writable` stays `unsafe`.

Open questions:

- **What "hold" covers.** Copies of the reference, member references, and
  tracked pointers formed from it all extend the time the promise must hold. A
  `* unsafe` pointer formed from it is outside the guarantee.
- **Where the error is reported.** Current teaching reports it at the
  conflicting write, pointing back to the `immutable` path, as the
  use-after-`last` direction in
  [usage and provenance tracking](usage-and-provenance-tracking.md) reports at
  the later use. The exact rule is open.
- **Construction and deep immutability.** Construction authority establishes
  `immutable` state, and immutability is deep over direct containment; both
  need re-reading against the path meaning.
- **Threading.** Data shared across threads as `immutable` must stay
  unchanged for as long as any thread holds it; this belongs with the
  threading capture in [safety boundaries](safety.md).

## A "forever" form

The maintainer wants a form whose value never changes, not only while a path
is held. Example: a function takes a handle from an `immutable` argument;
later it receives the same value again and assumes the handle is unchanged,
although it kept no reference in between. Under path meaning that assumption
is wrong.

```zax
// illustrative: what the function assumes under a forever meaning
remember final : ()(config : MyConfig immutable readonly &) = {
  cachedHandle = config.handle         // keeps the handle, not the reference
}

check final : ()(config : MyConfig immutable readonly &) = {
  use(cachedHandle)                    // assumes config.handle did not change since remember
}
```

Why path is the default: "forever" can be approximated by a naming convention
the compiler does not check, such as `myNeverChangingValue : MyType immutable`,
while a path-scoped promise could not be expressed at all without `unsafe`.

Options raised, none chosen:

- **Qualifying the qualifier**, such as `immutable path` and
  `immutable forever`. Functions taking one could never take the other, which
  splits calling conventions.
- **A compile-time assertion**, such as `verify<immutable-lifetime>{ value }`.
  Zax has no such form; it would stack awkwardly with `safe<...>` and
  `[<verify>]`.
- **A type-level choice**, such as `MyType :: type immutability forever {}`.
  Each type has one meaning, so there is no calling-convention split. The type
  author decides for all users, and generic code can assume only the weaker
  meaning unless constrained. `type mutable` and `type immutable` are reserved
  for split types, so the spelling must differ.
- **Whether an `identity` over an existing type can change the default.**

## Activation and retirement

Activate with the follow-up work item on immutability. Move accepted behavior
into qualifiers, casting, construction, and safety and analysis, then retire
this file and remove it from [the raw index](README.md).
