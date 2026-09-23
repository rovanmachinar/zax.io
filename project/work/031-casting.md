# 031: Casting

| Field | Value |
| --- | --- |
| Status | Active working material / non-normative / awaiting assignment |
| Work Item | `031` |
| Created | 2026-09-23 |
| Owns | The bounded legacy mining and disposition review defined below |
| Does Not Own | A general redesign of integers, identity, structural compatibility, composition, strings, or pointer ownership beyond concrete pressure recovered from legacy casting and the maintainer notes |

## Non-authority notice

This file is a collaborative working record. Existing legacy text, recovered
observations, candidate teaching, and later aligned findings remain
non-authoritative until a separately discussed, aligned, and explicitly
authorized promotion incorporates them into their lasting owners.

## Fixed initiating input

This section records the aligned information known when work item `031` was
created. It is intentionally incomplete and must not be rewritten as work
develops.

### Initiating concern

Mine legacy [`casting.md`](../../casting.md) by value so that page can receive
a complete disposition.

Casting here includes pointer casts and `as` operator casting. It is not limited
to either one. Current documentation already teaches substantial `as`,
conversion, admission, and recast behavior. The review distinguishes teaching
that is already current from legacy material that is still missing, and from
legacy material the maintainer notes mark defunct or superseded.

The maintainer-supplied
[casting notes](../raw/casting-maintainer-notes.md) are the primary input for
how casting should work. They may add to, replace, supersede, or reject
conflicting legacy evidence, and they may introduce new casting ideas. The
notes are empty at the creation of this work item. The maintainer will write
them before the mining review is assigned. Emptiness is not a finding and is
not permission to treat legacy `casting.md` as the preferred model.

For each distinct concept, rule, source form, example, or design pressure:

- determine whether current documentation already teaches its useful meaning;
- identify worthwhile material that current owners do not yet preserve;
- identify conflict with, or supersession by, current conceptual design or the
  maintainer notes;
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

- Treat `casting.md` as legacy evidence, not accepted design.
- Treat the maintainer notes, once written, as the preferred direction unless
  concrete semantic, cross-feature, teachability, or feasibility pressure
  exposes a flaw. Legacy precedence by itself is not such a flaw.
- Treat current `language/` owners as current for the concerns they own.
- Do not preserve legacy syntax merely because it exists.
- Do not promote rejected or superseded concepts merely to deny them in current
  teaching.
- When useful intent survives but its old syntax or mechanism does not,
  separate the intent from that obsolete vehicle.
- Distinguish duplicate current teaching from genuinely missing language
  design.
- Follow material consequences far enough to expose conflicts among pointer
  casts, `as`, identity, structure, and qualification without redesigning
  unrelated language areas.
- If no unique useful material remains after the audit, recommend deleting
  `casting.md`.

### Initial stopping guidance

Do not begin the mining review while
[casting notes](../raw/casting-maintainer-notes.md) are still empty. Stop and
wait for the maintainer to write them.

Creating and routing this work item does not authorize analysis. The session
that creates `031` stops before that review. A later assignment begins it.

Do not promote findings, delete `casting.md`, archive this work item, or create
work item `032` without the separately required discussion, alignment, and
authorization.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs focused reading,
  authority, legacy consumption, promotion, deferral, and closure.
- [Casting maintainer notes](../raw/casting-maintainer-notes.md) - primary
  account of current casting intent, including what is defunct, superseded,
  still valid, or new. Read this before mining the legacy page. If it is still
  empty, stop.
- [Legacy casting](../../casting.md) - immutable source corpus to mine and
  disposition after the notes are present.

### Consequence-driven

Read only the smallest relevant sections after a concrete note or legacy claim
crosses one of these boundaries:

- [Zax integers](../../language/integers.md) and the
  [integer operator catalog](../../language/integer-operator-catalog.md) when
  integer `as`, narrowing, optional conversion, or admission is implicated.
- [Zax identity types](../../language/identity-types.md) when identity
  admission or projection is implicated.
- [Zax structural shapes and compatibility](../../language/structural-shapes-and-compatibility.md)
  when structural recast, same-storage views, or `unsafe cast` is implicated.
- [Zax composition](../../language/composition.md) when outer cast is
  implicated.
- [Strings and characters](../../language/strings-and-characters.md) when
  string-identity conversion is implicated.
- [Operator catalog](../../language/operator-catalog.md) when the spelled `as`
  forms are implicated.
- [Pointers, allocation, and arenas](../../language/pointers-and-arenas.md) and
  [raw pointer and arena mechanics](../raw/pointer-and-arena-mechanics.md) when
  a pointer cast is concrete.
- [Zax qualifiers](../../language/qualifiers.md) when a cast changes access,
  mutability, or replacement authority.

### Audit-only

- Archived numbered work only when a concrete provenance or regression question
  cannot be answered from the maintainer notes, `casting.md`, and the current
  owners named above.

## Working record

Awaiting assignment. The maintainer will write the casting account in
[casting notes](../raw/casting-maintainer-notes.md) before this review begins.
Creating and routing work item `031` does not authorize analysis.
