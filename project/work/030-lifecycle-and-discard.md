# 030: Legacy lifecycle and discard audit

| Field | Value |
| --- | --- |
| Status | Active working material / non-normative / awaiting assignment |
| Work Item | `030` |
| Created | 2026-09-22 |
| Owns | The bounded legacy mining and disposition review defined below |
| Does Not Own | A general redesign of construction, destruction, result routing, ownership, or resource disposal beyond concrete pressure recovered from the two initiating legacy pages |

## Non-authority notice

This file is a collaborative working record. Existing legacy text, recovered
observations, candidate teaching, and later aligned findings remain
non-authoritative until a separately discussed, aligned, and explicitly
authorized promotion incorporates them into their lasting owners.

## Fixed initiating input

This section records the aligned information known when work item `030` was
created. It is intentionally incomplete and must not be rewritten as work
develops.

### Initiating concern

Mine legacy [`ctor-dtor.md`](../../ctor-dtor.md) and
[`discard.md`](../../discard.md) by value so both pages can receive a complete
disposition.

For each distinct concept, rule, source form, example, or design pressure:

- determine whether current documentation already teaches its useful meaning;
- identify worthwhile material that current owners do not yet preserve;
- identify conflict with, or supersession by, current conceptual design;
- explain ambiguity that prevents a responsible disposition;
- recommend the lasting owner or explicit deferral for material worth keeping;
  and
- recommend whether the legacy page can be deleted after all useful input is
  promoted, deferred, rejected, or superseded.

The review must do more than recommend deletion. Any concept worth preserving
must be made understandable in plain language, supported by representative Zax
source where source can clarify it, and shaped like material that could later be
promoted for human developers.

### Review posture

- Treat both root pages as legacy evidence, not accepted design.
- Treat current `language/` owners as current for the concerns they own.
- Do not preserve legacy syntax merely because it exists.
- Do not promote rejected or superseded concepts merely to deny them in current
  teaching.
- When useful intent survives but its old syntax or mechanism does not,
  separate the intent from that obsolete vehicle.
- Distinguish duplicate current teaching from genuinely missing language
  design.
- Follow material consequences far enough to expose conflicts, costs, cleanup,
  lifetime, transfer, or source-stability effects without redesigning unrelated
  language areas.
- If no unique useful material remains after the audit, recommend deleting the
  applicable legacy page.

### Required output

The working record must provide:

1. a reviewable inventory of the useful claims and examples in each legacy
   page;
2. the current owner, if any, that already preserves each useful idea;
3. teachable candidate material for every uncovered idea worth retaining;
4. concrete conflicts with current design and a recommended disposition;
5. explicit deferrals with a live destination and activation pressure;
6. a whole-file recommendation for `ctor-dtor.md`;
7. a whole-file recommendation for `discard.md`; and
8. the exact promotion/deletion set if the work reaches a documentation-fit dry
   run.

The inventory supports review but must not replace understandable findings.

### Initial stopping guidance

Stop when:

- every materially useful passage and representative example from both legacy
  pages has one disposition;
- any uncovered candidate is explained well enough for maintainer review;
- every conflict identifies the current rule or owner that decides it;
- no future-relevant concept survives only in this eventual archive;
- page-retirement recommendations are evidence-based; and
- any aligned promotion has completed the required documentation-fit dry run.

Do not promote findings, delete either initiating legacy page, archive this work
item, or create work item `031` without the separately required discussion,
alignment, and authorization.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs focused reading,
  authority, legacy consumption, promotion, deferral, and closure.
- [Legacy constructors and destructors](../../ctor-dtor.md) - first immutable
  source corpus to mine and disposition.
- [Legacy discard](../../discard.md) - second immutable source corpus to mine
  and disposition.
- [Construction, replacement, and destruction](../../language/construction-and-destruction.md) -
  current owner for constructors, destructors, lifecycle state, replacement,
  and scope-exit destruction.
- [Declarations and bindings](../../language/declarations-and-bindings.md) -
  current owner for anonymous declarations, explicit discard names,
  initialization, and declaration/result contexts.
- [Function invocation](../../language/function-invocation.md) - current owner
  for result acknowledgement, source/destination discard, `return #`, mapping,
  completion, and elision.
- [Zax `using`](../../language/using.md) - current owner for anonymous resource
  enrollment, disposal suppression through `#`, destruction, and resource-exit
  ordering.

### Consequence-driven

- [Exceptional result flow](../../language/except.md) when legacy discard or
  lifecycle material implies omission, forwarding, or failure outcomes.
- [Lifetimes and references](../../language/lifetimes-and-references.md) when a
  claim changes instance paths, reference validity, escape, or destruction.
- [Transfer stances](../../language/transfer-stances.md) when copy, move,
  terminal use, or source post-state becomes material.
- [Pointers, allocation, and arenas](../../language/pointers-and-arenas.md) when
  legacy ownership, allocation, reset, or deallocation behavior is concrete.
- [Optional values](../../language/optional-values.md) and
  [variants](../../language/variants.md) when conditional contained lifetime or
  reset behavior is implicated.
- [Qualifiers](../../language/qualifiers.md) when construction/destruction or
  discard claims change mutation, access, or replacement authority.
- [Core flow control](../../language/core-flow-control.md) when scope exit,
  transfer, or post behavior matters.
- [Source structure](../../language/source-structure.md) when punctuation,
  attachment, continuation, or declaration-versus-routing intent needs
  disposition.
- [Safety and analysis](../../language/safety-and-analysis.md) when legacy
  material requires proof, unsafe responsibility, panic, or known-invalid
  classification.

Read only the smallest relevant sections after a concrete legacy claim crosses
one of these boundaries.

### Audit-only

- Archived numbered work only when a concrete provenance or regression question
  cannot be answered from current owners and the two initiating legacy pages.

## Working record

Awaiting assignment. Creating and routing work item `030` does not authorize
analysis. A new session or an explicitly reassigned session begins the mining
review.
