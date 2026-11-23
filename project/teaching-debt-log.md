# Zax teaching-debt log

| Field | Value |
| --- | --- |
| Status | Live project observation queue / non-authoritative |
| Audience | The language maintainer, promotion agents dispositioning firsthand observations, and agents assigned selected teaching refresh entries |
| Applies To | Unresolved material teaching problems in current `language/` owners after promotion disposition |
| Owns | The durable queue of concrete current-owner reader-friction observations |
| Does Not Own | Language meaning, refresh methods, ordinary concept-work reading, raw/legacy/archive debt, style preferences, or correctness audits |

## Reading posture

Do not read this log during ordinary onboarding or merely to discover what
problems a concept agent is expected to notice.

The log is not:

- a checklist;
- a test answer key;
- a prompt to search current owners for defects;
- or permission to expand an assigned concern.

Read only:

- selected entries named by a teaching-debt work item;
- a relevant owner section when promotion must add, update, or deduplicate an
  independently noticed concern;
- or the queue when the language maintainer is choosing a bounded refresh.

Ordinary concept work records only concrete firsthand friction encountered while
following required focused reading. The complete recognition, promotion, and
archival lifecycle is defined by
[documentation architecture](documentation.md#teaching-debt). Suggested refresh
methods are in [teaching-debt guidance](teaching-debt.md).

## Entry requirements

Each entry identifies:

- one current `language/` owner and section;
- the reader task that was materially blocked;
- concrete wording, ordering, examples, or missing consequences causing the
  friction;
- why the concern is more than stylistic preference;
- when the concern should be revisited;
- and provenance.

Possible solutions may be recorded when already clear but are not required.

Organize entries by current owner path. Cross-owner patterns may be summarized
only after several concrete entries demonstrate them.

Resolved entries are removed. Partially resolved entries are narrowed. This file
is not a historical archive.

## Entries

### Cross-owner pattern: specification-like vocabulary in older teaching

Maintainer direction, 2026-11-14: the language maintainer suspects that the
whole `language/` corpus needs a teaching review, beyond the concrete entries
below. Work item 038 found the same kinds of friction in every owner it
touched, in older text it did not change: abstract nouns acting as subjects
("authority", "disposition", "provenance", "truth", "contract"), rules stated
without source, and terms used before they are taught. The entries below are
the concrete evidence found so far; they are not a complete list.

- **Revisit when:** the maintainer chooses a teaching-refresh work item. A
  corpus-wide refresh would likely need its own scoping, owner by owner,
  rather than one change set.
- **Provenance:** work item 038, dry run 6 and the cold reads after its
  follow-up promotion.

### Older passages found by the work-item 038 cold reads

The cold readers listed these as poor teaching in unchanged text near the
sections 038 changed. Each is one concrete passage; none was rewritten,
because it lies outside the promotion. All have the same reader problem:
undefined design vocabulary where a programmer needs a plain statement of
what happens. Revisit with the owner's teaching refresh.

- **`language/construction-and-destruction.md`, "Custom replacement" and
  "Destructor declarations and sequence":** "has transitional construction
  authority", "renewing every member resident instance", "enters terminal
  destruction authority".
- **`language/declarations-and-bindings.md`, "Capability conversions" and
  "`final` and `varying`":** uses "stance" for place qualifiers ("omitted
  type-side stance", "resolves its own independent place stance"), which
  collides with transfer stances. `language/qualifiers.md` does the same
  ("the place's final stance", the heading "Inspecting the stance", "Opposing
  stances applied to the same entity"); the 039 cold read flagged it as the
  most confusing collision now that stances are central. Say "place
  qualification" or "final/varying".
- **`language/optional-values.md`, the reset, move, and last table near the
  top:** "Complete consumer of `last value` | Remaining payload state
  dispositioned".
- **`language/terms.md`, "Declaration-attached raw allocation":** "An accepted
  terminal transfer leaves the source pointer vacant".
- **`language/identity-types.md`, "Representation and qualifications":**
  "whole-root same-storage compatible reference can retain varying
  replacement authority".
- **`language/structural-shapes-and-compatibility.md`, "Whole-root varying
  views require full compatibility":** written as a dense specification.
- **`language/switch.md`:** "clause-local reference-shaped payload access".
- **`language/arrays-and-slices.md`, element moves:** "as required by the
  selected element contract".
- **`language/pointers-and-arenas.md`, "Adopt an existing allocation":** the
  bullets on "blockless" and "detached dormant block", and "The producer's
  declared result stance and disposition authority".
- **`language/operators.md` and `language/variants.md`, near their stance
  sections:** "Candidate-tree formation", "exact independent-copy contract",
  "terminal but destruction-valid payload".
- **`language/construction-and-destruction.md`, "Opting in to default move,
  last, and deep":** the claim that "An `Integer` that counts packets still in
  flight … may need different handling" has no example.

- **Provenance:** work item 038 cold reads, 2026-11-14.

### Older passages found by the work-item 039 cold read

Unchanged text near the sections 039 rewrote. Each is one concrete passage
that a cold reader could not follow; none was rewritten, because it lies
outside the promotion. Revisit with the owner's teaching refresh.

- **`language/function-invocation.md`, "Minted implementation model", the
  composition `via` part:** "selected-contract proof", "`outer tracked`
  capability", "thin adapter", "exact family anchor", and "data `via`" are
  used without definition. A one-paragraph handoff to composition would serve
  the reader better.
- **`language/function-invocation.md`, "Visible defaults and compatible
  implementations", opening:** "not layered under `configuredTimeout()`" and
  "a cost of the visible contract", with `callback : … = implementation`
  unexplained. Say which defaults apply on each call path.
- **`language/function-invocation.md`, "Type and instance calls to `once`
  functions":** "`once bound`" where the examples write `final once`, and
  "missing target trap capability has the undefined boundary defined by the
  Nothing owner" reads as specification.
- **`language/pointers-and-arenas.md`, "Shareable unique ownership":** "The
  legacy pointer qualifier `own` is superseded by `unique shareable` … does
  not revive the old pointer family" is decision history. Say only that `own`
  is a composition word, not a pointer kind.
- **`language/transfer-stances.md`, "Stances and qualifiers":**
  `drainBuffer(source : Buffer mutable readonly final move &)` "can move from
  the `Buffer` only in a way that does not change it", which a reader can't
  reconcile with moving. Show concretely what a readonly move takes, or use a
  writable example.

- **Provenance:** work item 039 cold read, 2026-11-22.

### `language/pointers-and-arenas.md`

#### Raw pointers

- **Blocked reader task:** understanding when a raw pointer is safe to use.
- **Evidence:** "The raw pointer remains non-owning; the containing lifecycle
  is the independently proved owner responsible for ending the nested path."
  and "losing or repointing that pointer may leave no authority able to end
  the path." The section explains safety through "lifecycle", "path",
  "provenance", and "authority" with one declaration and no example of a safe
  versus unsafe use.
- **Why material:** a reader cannot tell which raw-pointer uses compile
  without `unsafe`, which is the decision the section exists to support.
- **Revisit when:** a teaching refresh of pointers, or any change to raw
  pointer safety rules.
- **Possible direction:** show one safe member-pointer use, one that escapes
  and needs `unsafe`, and say who frees the pointee in each.
- **Provenance:** work item 038, section 10 step 5 and dry run 6.

#### Allocation disposition

- **Blocked reader task:** choosing when an allocation is destroyed and its
  storage recovered.
- **Evidence:** "Allocation independently chooses one disposition and whether
  the allocation may participate in cycle collection." The table rows say
  "When ordinary ownership or scheduled disposition closes". "Disposition" is
  used as the concept's name without a plain explanation or example.
- **Why material:** the choice changes when destructors run, which a
  programmer must predict.
- **Revisit when:** a teaching refresh of pointers or arenas.
- **Provenance:** work item 038, section 10 step 5.

### `language/qualifiers.md`

#### Type-side truth versus declaration-side permission

- **Blocked reader task:** understanding why `final` and `varying` can appear
  in two positions and what each controls.
- **Evidence:** the heading and tables use "truth", "replacement authority",
  and "Replace the entire value lifetime in place"; the section asks the
  reader to "Teach it as **capability versus permission**", which is an
  instruction to an author, not teaching.
- **Why material:** the two positions are a Zax-specific idea a newcomer will
  not expect; the abstract wording makes the distinction harder to see than
  one paired example would.
- **Revisit when:** a teaching refresh of qualifiers.
- **Possible direction:** lead with two declarations that differ only in the
  position of `final`, and show which assignments each allows.
- **Provenance:** work item 038, section 10 step 5.
