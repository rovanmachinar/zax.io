# 035: Compile-time code and compiler directives

| Field | Value |
| --- | --- |
| Status | Active working material / non-normative / awaiting assignment |
| Work Item | `035` |
| Created | 2026-09-26 |
| Owns | The bounded review of Zax compile-time code, including compiler directives, as defined below |
| Does Not Own | Generics; async-related directives; the decision itself, which belongs to the language maintainer |

## Non-authority notice

This file is a collaborative working record. Existing documentation, candidate
syntax, examples, and later aligned findings remain non-authoritative until a
separately discussed, aligned, and explicitly authorized promotion incorporates
them into their lasting owners.

## Fixed initiating input

This section records the aligned information known when work item `035` was
created. It is intentionally incomplete and must not be rewritten as work
develops.

### Initiating concern

What is Zax compile-time code, and how do compiler directives work as part of
it?

Compile-time code and compiler directives are intrinsically linked, so this
item treats them as one concern rather than reviewing directives separately.

### Motivating pressure

- **Generics depend on it.** Generic work needs a settled model of compile-time
  evaluation first.
- **Directives appear without a general model.** Current owners already use
  directive forms such as `[<nothing-instance-default=trap>]` and the
  compiler-directive enclosure, but no owner defines what a directive is, when
  it runs, or how directives are categorized.
- **A large unreviewed legacy page.** The root `compiler-directives.md` covers
  many directives, including `compile`, `compilation`, `compiles`, `execute`,
  `requires`, `concept`, `panic`, `inline`, `export`, literal directives, and
  qualifier-default directives.
- **Open compile-time questions from other owners.** These include detecting
  whether a hook point has fulfillments, `size of` before a type's storage
  closes, required compile-time execution for literals, and native versus
  compiler-host versus target execution.

### Starting input

The maintainer's notes are
[`project/raw/maintainer-notes/compiler-directives.md`](../raw/maintainer-notes/compiler-directives.md).
What those notes describe takes priority over the legacy material wherever they
differ. The maintainer fills the notes before this item is assigned.

The legacy root [`compiler-directives.md`](../../compiler-directives.md) and the
raw [compile-time execution input](../raw/compile-time-execution.md) are
evidence. They are not authoritative over the concept or the maintainer's notes.

### Starting boundaries

- Do not design generics. The maintainer's generics notes are future input and
  are not read in this item.
- Async-related directives, such as `synchronous` and `asynchronous`, are
  outside this item unless a concrete compile-time consequence makes them
  necessary.
- Panic categories, lint suppression, and unsafe controls stay with the
  [analysis-control input](../raw/analysis-controls.md) except where directive
  syntax touches them.

### Initial stopping guidance

Establish the foundational model first: what runs at compile time, how native,
compiler-host, and target execution relate, and the general form and categories
of directives. Then give each legacy directive a disposition (promote, defer, or
reject) rather than designing every directive in full.

Creating and routing this work item does not authorize analysis. A later
assignment begins it.

Do not promote findings, change current owners, archive this work item, or
create work item `036` without the separately required discussion, alignment,
and authorization.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs focused reading,
  authority, promotion, deferral, and closure.
- [Maintainer notes on compiler directives](../raw/maintainer-notes/compiler-directives.md) -
  the primary input and the maintainer's current direction.
- [Raw compile-time execution](../raw/compile-time-execution.md) - preserved
  compile-time questions, host and target context, constant availability, and
  deferred partial pressure.
- Legacy [`compiler-directives.md`](../../compiler-directives.md), its opening
  "Compiler Directives" section and "Official and extended directives" - the
  legacy framing of what directives are.
- [Zax source structure](../../language/source-structure.md#compiler-directive-enclosure),
  compiler-directive enclosure, and the
  [compiler-directive enclosure](../../language/terms.md#compiler-directive-enclosure)
  term - the current directive syntax boundary.
- [Zax namespaces and modules](../../language/namespaces-and-modules.md#nothing-policy-defaults),
  Nothing-policy defaults - a current directive whose scope-bound attachment
  any general model must preserve.

### Consequence-driven

Read only the smallest relevant sections when the concern reaches them:

- The remaining sections of the legacy
  [`compiler-directives.md`](../../compiler-directives.md), one directive at a
  time as each is dispositioned.
- Legacy [`meta-functions.md`](../../meta-functions.md) and
  [`meta-types.md`](../../meta-types.md) when compile-time functions or type
  computation are reached.
- Raw inputs on [export and visibility directives](../raw/export-and-visibility-directives.md),
  [analysis controls](../raw/analysis-controls.md),
  [the CPU provider model](../raw/cpu-provider-model.md), and
  [global and once lifetimes](../raw/global-and-once-lifetimes.md) when their
  concerns are reached.
- [Zax safety and analysis](../../language/safety-and-analysis.md#language-contracts-and-compiler-analysis)
  for language-contract selection.
- [Zax integer literals](../../language/integer-literals.md#concrete-results-and-compile-time-execution)
  and [Zax literal source](../../language/literal-source-and-operators.md#required-compile-time-execution)
  for existing required compile-time behavior.
- [Zax partials](../../language/partials.md) for `size of` before storage closes
  and hook-attachment detection.
- [Raw type parameters and generics](../raw/type-parameters-and-generics.md)
  only where the compile-time boundary itself is at issue.

### Audit-only

- Archived numbered work, only when a provenance question cannot be answered
  from current owners or raw input.

## Working record

Not started.
