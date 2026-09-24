# 032: Construction with `.=`

| Field | Value |
| --- | --- |
| Status | Active working material / non-normative / awaiting assignment |
| Work Item | `032` |
| Created | 2026-09-24 |
| Owns | The bounded review of whether `.=` should express construction beyond result slots, as defined below |
| Does Not Own | A general redesign of assignment, replacement, lifecycle, or declaration syntax beyond the consequences of the question below; the decision itself, which belongs to the language maintainer |

## Non-authority notice

This file is a collaborative working record. Existing documentation, candidate
syntax, examples, and later aligned findings remain non-authoritative until a
separately discussed, aligned, and explicitly authorized promotion incorporates
them into their lasting owners.

## Fixed initiating input

This section records the aligned information known when work item `032` was
created. It is intentionally incomplete and must not be rewritten as work
develops.

### Initiating concern

Should `.=` mean **construction and replacement construction** at every
construction site, rather than only for function result slots?

Current teaching already constructs result slots with `.=`:

```zax
Celsius :: type {
  degrees : Binary64

  operator binary 'as' final : (
    result : DestinationType
  )(
    DestinationType : type
  ) readonly = {
    result .= [{
      .degrees = degrees * 9.0 / 5.0 + 32.0
    }]
  }
}
```

That is accepted, promoted design. It must be understood as real design
pressure, not a local curiosity. Promoted documentation now teaches readers
that `.=` constructs a result, so they will reasonably ask why other
construction sites use `=`:

```zax
// Illustrative only: would declarations construct with `.=`?
myType : MyType .= value
```

### Motivating pressure

- Ordinary `=` never constructs: `result.degrees = x` before construction is an
  error.
- A result initializer such as `= :` is part of the **prototype**. It obligates
  every caller and every other implementation of that prototype to
  pre-construct the result, so it is the wrong tool for a need local to one
  body.
- `result.+++()` is correct but verbose.
- `.=` already makes a place hold a newly constructed value. It replaces a live
  place, constructs an optional or variant payload, and now performs first
  construction of a result slot. It could become the single visible spelling
  for "construct here".

### Starting constraints

These are accepted today:

- ordinary `=` never constructs;
- first-construction `.=` reads exactly like a declaration initializer (`x .= src`
  means what `x : T = src` means, including `[{}]` constructing a present
  optional);
- `.=` replacement permissions apply only to live places, so first construction
  of a `final` result needs none;
- a path either completes a result itself or returns a value, never both;
- `.=` produces access to the newly constructed value.

### Required output: implications with representative examples

The maintainer will decide. The review must **not** deliver only its own
judgment. It must:

1. **Show the implications of moving construction from `=` to `.=`.** For each
   class of construction scenario that would change, give one representative
   before-and-after example: a representative class, not every example in the
   documentation. The classes include at least:
   - declarations and inferred declarations;
   - constructor members (`_.member .= [{ ... }]` as the readable form of
     `_.member.+++(...)`);
   - result initializers in prototypes;
   - multiple results and result mappings or routing;
   - construction packets;
   - optional and variant payloads;
   - allocation initializers;
   - captures;
   - any other construction site the review finds.
2. **Show what does not change**, where confusion is likely: ordinary
   assignment, in-lifetime mutation, and replacement.
3. **Trace the consequences** for teaching, diagnostics, source stability, and
   existing examples.
4. **Give the agent's opinion** with its reasoning, including narrower
   alternatives, such as constructor members only, or results and members but
   not declarations, and any other ideas that emerge.

The maintainer can then judge from both the opinion and the examples whether to
adopt a broad change, a narrower change, another idea, or no change, leaving
`.=` local to results and replacement.

### Initial stopping guidance

Creating and routing this work item does not authorize analysis. A later
assignment begins it.

Do not promote findings, change current owners, archive this work item, or
create work item `033` without the separately required discussion, alignment,
and authorization.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs focused reading,
  authority, promotion, deferral, and closure.
- [Zax function invocation](../../language/function-invocation.md), the result
  slots and opt-in result initialization sections - the current `.=`
  first-construction rule and the prototype obligation of `= :`.
- [Zax construction, replacement, and destruction](../../language/construction-and-destruction.md),
  the ordinary construction, construction packets, reconstructive replacement,
  and wrapper-owned `.=` sections - constructor-member construction and every
  current meaning of `.=`.
- [Zax declarations and bindings](../../language/declarations-and-bindings.md),
  the declaration and initialization sections - the largest class of `=`
  construction sites.

### Consequence-driven

Read only the smallest relevant sections when a scenario class reaches them:

- [Zax optional values](../../language/optional-values.md) and
  [Zax variants](../../language/variants.md) for payload construction.
- [Zax operator catalog](../../language/operator-catalog.md) for `=` and `.=`
  forms and precedence.
- [Zax pointers, allocation, and arenas](../../language/pointers-and-arenas.md)
  for allocation initializers.
- [Zax lambdas and callable composition](../../language/lambdas-and-callable-composition.md)
  for captures.
- [Zax structural shapes and compatibility](../../language/structural-shapes-and-compatibility.md)
  for result mapping and transformation construction.
- [Zax qualifiers](../../language/qualifiers.md) for replacement permission.
- [Zax conversions and casts](../../language/casting.md) as the first taught use
  of result `.=`.

### Audit-only

- [Archived `031`](../archive/work/031-casting.md), for how the result-slot
  rule was reached, only when a provenance question cannot be answered from
  current owners.

## Working record

Not started.
