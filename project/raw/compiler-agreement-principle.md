# Raw input: what compilers must agree on

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative; agreed in direction, not yet discussed as a principle |
| Audience | Future work on `language/principles.md`, safety contracts, and diagnostics |
| Applies To | Whether a program's acceptance may depend on how much a particular compiler can prove |
| Owns | The candidate principle, its reasoning, the maintainer's recorded concern, and an example |
| Does Not Own | Current safety, verification, and contract evolution ([safety and analysis](../../language/safety-and-analysis.md)); the principles catalog ([principles](../../language/principles.md)) |
| Source / Provenance | Work item `039` (pointer stances), part D, raised by the maintainer while discussing raw-pointer safety; wording revised in work item `041` after verification of claims was designed |

## The candidate principle

Tentative wording:

> Whether source is valid Zax depends only on rules the specification states,
> and compile-time checks such as `wellformed` answer by those rules alone.
> Beyond them, a compiler may report a `safe<...>` claim it proves false,
> which is always an error, and may report claims it can prove, at a level the
> build chooses. Neither changes what the program means.

The specification defines the cases that count as safe. A compiler that can
prove more still requires the `safe<...>` claim outside those cases:

```zax
safe<distinct-element>{
  dispatch(inbox[2 * i] as last)
  dispatch(inbox[2 * i + 1] as last)
}
// every compiler requires the claim, even one that can see the two indexes
// always differ; that compiler may report that it can verify the claim, and
// a [<verify=x-acme>] directive asks it to keep checking
```

## Why

- Adding safe cases to the specification later keeps existing programs
  compiling.
- Adding required errors later breaks them.
- If acceptance depended on each compiler's cleverness, a program could compile
  with one compiler and fail with another under the same language contract, or
  a `wellformed` check could choose a different branch.
- A claim a compiler proves false describes code with undefined behavior, so
  reporting it as an error protects the program without making acceptance
  depend on cleverness for valid code.

## The maintainer's concern, recorded with it

A capable compiler will sometimes demand a claim for code it knows is safe.
That cost is accepted for portability. Asking a named compiler to verify the
claim, described by
[safety and analysis](../../language/safety-and-analysis.md#asking-a-compiler-to-verify-a-claim),
lets that compiler keep checking the site without changing whether other
compilers accept it.

## What this input does not decide

- The principle's wording in `language/principles.md`.
- How it interacts with explicitly selected extension contracts, which
  [safety and analysis](../../language/safety-and-analysis.md) already allows
  to make stronger proofs canonical for the source that selects them.

## Activation and retirement

Activate when the principles catalog, safety contracts, or diagnostics policy
is next reviewed. Retire this file once the principle is accepted into
`language/principles.md` or explicitly rejected, and remove it from
[the raw index](README.md).
