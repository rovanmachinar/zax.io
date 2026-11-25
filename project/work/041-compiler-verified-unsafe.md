# 041: Compiler-verified unsafe

| Field | Value |
| --- | --- |
| Status | Active working material / non-normative / not started |
| Work Item | `041` |
| Created | 2026-11-24 |
| Owns | The bounded review of a directive that names one or more compilers which must prove an `unsafe` site safe, while the site keeps its `unsafe` for every other compiler |
| Does Not Own | Which facts any compiler or the specification actually proves (specific tracking or analysis rules); threading and exclusive access; general language-contract selection beyond where it meets the directive; current safety, directive, and array behavior, which stay with their owners until a promotion is authorized; the decision itself, which belongs to the language maintainer |

## Non-authority notice

This file is a collaborative working record. Existing documentation, candidate
syntax, examples, and later aligned findings remain non-authoritative until a
separately discussed, aligned, and explicitly authorized promotion incorporates
them into their lasting owners.

## Fixed initiating input

This section records the aligned information known when work item `041` was
created. It is intentionally incomplete and must not be rewritten as work
develops.

### Initiating concern

Zax requires `unsafe` wherever the language's listed proofs do not cover a
site, even when a particular compiler can prove that site safe, so that every
compiler accepts the same programs. A directive could let source name one or
more compilers that must actually prove the site safe, while the `unsafe`
stays in place for every other compiler. Current teaching already shows such a
directive, written `[<safe=…>]`, as if it exists and marked as still being
designed (arrays and slices, "What the compiler can prove"). What should the
directive be, what does it claim, and how does it behave?

### Starting input

- `project/raw/analysis-controls.md`, "Compiler-verified unsafe tags": the
  intended behavior (compilers that don't implement the named verifier ignore
  the tag and accept the `unsafe`; compilers that do must prove the site and
  reject it when the proof fails; reviewers can tell verified sites from
  unchecked ones; a later contract that adopts the proof makes both the tag
  and the `unsafe` redundant), multiple verifiers, the naming candidates and
  their polarity concerns (`safe=`, `proved=`, `proved-safe=`,
  `verified-safe=`), and its open questions (exact spelling, sharing the
  `[<...>]` form with `[<panic=...>]`, unknown verifiers silently ignored or
  linted, a strict build option requiring listed verifiers, interaction with
  compiler agreement).
- `project/raw/compiler-agreement-principle.md`: acceptance depends only on
  rules the specification states; a stronger compiler may warn but never
  accept or reject differently. The maintainer's recorded concern and the tag
  as its answer.
- The maintainer's direction from work item 040: use the directive in
  teaching as if it exists, because it will, subject to change when this work
  completes. Specific tracking rules are not ready, because the language should
  be settled first, threading especially.

### Starting boundaries

- What any compiler proves, and any specific tracking or analysis rule, is
  out of scope. This item decides the directive, not the analyses it names.
- Threading and exclusive access are out of scope; they have their own
  capture in `project/raw/safety.md`.
- The compiler-agreement direction is the starting position. A conflict with
  it is recorded as a finding for discussion, not resolved by silently
  changing it.
- `intent<...>` acknowledgements are a separate mechanism: they confirm
  defined but suspicious behavior and supply no unsafe fact.

### Initial stopping guidance

Per "The first deliverable is a set of decisions" in
[documentation architecture](../documentation.md#the-first-deliverable-is-a-set-of-decisions):
first identify the issues this directive is likely to raise, and present a
workable set of decisions the maintainer can act on, each with concrete Zax
examples, the realistic options, what each costs, and a recommendation.
Supporting evidence and tensions sit underneath the decisions. Do not design
every combination in full before the direction is aligned.

Creating this work item does not authorize analysis. A later assignment
begins it.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs focused reading,
  authority, promotion, deferral, closure, and the writing rules.
- [Analysis controls](../raw/analysis-controls.md): "Language contracts and
  compiler analysis", "Contract-dependent redundant controls",
  "Compiler-verified unsafe tags", and "Diagnostics must name the required
  `unsafe` form" - the existing design input for the directive.
- [What compilers must agree on](../raw/compiler-agreement-principle.md) - the
  principle the directive must fit.
- [Zax safety and analysis](../../language/safety-and-analysis.md): "Safety is
  a selected contract", "Four analysis outcomes", and "Language contracts and
  compiler analysis" - the current owner the directive extends.
- [Zax compiler directives](../../language/compiler-directives.md): directive
  families, grammar, `x-` names, and attachment - the form the directive takes.
- [Zax arrays and slices](../../language/arrays-and-slices.md), "What the
  compiler can prove" - the one current page that already shows the
  directive.

### Consequence-driven

Read only the smallest relevant sections when the concern reaches them:

- [Allow with tracking rather than forbid](../raw/allow-with-tracking-principle.md),
  when the directive's role in the wider safety approach is reached.
- [Analysis-control registry](../raw/analysis-control-registry.md), when
  `unsafe` category names are reached.
- [Zax intent acknowledgements](../../language/intent-acknowledgements.md),
  when the directive must be distinguished from `intent<...>`.
- [Zax source structure](../../language/source-structure.md), "Compiler-directive
  enclosure", when placement or spelling is reached.
- [Zax pointers, allocation, and arenas](../../language/pointers-and-arenas.md),
  when the raw-pointer-from-reference example is reached.
- [Usage and provenance tracking](../raw/usage-and-provenance-tracking.md),
  only to keep its tracking rules outside this item's boundary.

### Audit-only

- Archived numbered work, only when a provenance question cannot be answered
  from current owners or raw input. The compiler-agreement principle came from
  `project/archive/work/039-pointer-stances.md`; the directive's current use
  in teaching came from `project/archive/work/040-element-and-contained-stances.md`.

## Working record

Not started.
