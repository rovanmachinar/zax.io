# Raw input: what compilers must agree on

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative; agreed in direction, not yet discussed as a principle |
| Audience | Future work on `language/principles.md`, safety contracts, and diagnostics |
| Applies To | Whether a program's acceptance may depend on how much a particular compiler can prove |
| Owns | The candidate principle, its reasoning, the maintainer's recorded concern, an example, and how it relates to compiler-verified `unsafe` tags |
| Does Not Own | Current safety and contract evolution ([safety and analysis](../../language/safety-and-analysis.md)); the principles catalog ([principles](../../language/principles.md)); the verifier-tag idea itself ([analysis controls](analysis-controls.md#compiler-verified-unsafe-tags)) |
| Source / Provenance | Work item `039` (pointer stances), part D, raised by the maintainer while discussing raw-pointer safety |

## The candidate principle

> Whether a program is accepted depends only on rules the specification
> states. A compiler's extra analysis may produce warnings, but never accept or
> reject differently.

The specification defines the cases that count as safe. A compiler that can
prove more still requires `unsafe` outside those cases, and may say the marker
looks unnecessary:

```zax
owner : MyValue * last = @
view : MyValue * = owner.
unsafe vacate owner
restored : MyValue * last = (view unsafe cast as last) as last
// every compiler requires the unsafe cast here, even one that can see that
// view is the only remaining address of the allocation; it may warn that
// the unsafe looks unnecessary, but it may not accept the line without it
```

## Why

- Adding safe cases to the specification later keeps existing programs
  compiling.
- Adding required errors later breaks them.
- If acceptance depended on each compiler's cleverness, a program could compile
  with one compiler and fail with another under the same language contract.

## The maintainer's concern, recorded with it

A capable compiler will sometimes demand `unsafe` for code it knows is safe.
That cost is accepted for portability. The compiler-verified `unsafe` tag
idea, `[<safe=x-my-compiler>]`, is a related way to let a named compiler check
such a site without changing whether other compilers accept it; see
[analysis controls](analysis-controls.md#compiler-verified-unsafe-tags).

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
