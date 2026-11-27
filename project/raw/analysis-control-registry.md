# Raw registry: safety and panic analysis controls

| Field | Value |
| --- | --- |
| Status | Raw provisional registry / non-authoritative |
| Audience | Language maintainers and future work defining safety categories, runtime-check controls, or panic categories |
| Applies To | Discovery, maturity, and history of `safe<...>`, `unsafe<...>`, and panic category identifiers |
| Owns | The centralized provisional inventory of category names, unnamed category pressure, source occurrences, and disposition state |
| Does Not Own | The current category list and its teaching ([safety and analysis](../../language/safety-and-analysis.md#current-categories)), accepted panic-control syntax, domain behavior, intent acknowledgements, diagnostic identifiers, or compiler implementation |
| Source / Provenance | Current conceptual owners, `project/raw/analysis-controls.md`, reached raw safety input, legacy compiler-directive evidence, and work items `028`, `039`, `040`, and `041` |

## Purpose and reading posture

This file prevents analysis-control names and unnamed registry obligations from
surviving only in scattered examples. It is a discovery and disposition ledger,
not an accepted language registry.

Each behavior remains owned by its domain document. The future analysis-control
owner decides exact syntax, category kinds, extension provenance, selected
language-contract requirements, and promotion into an accepted registry.

The current accepted **intent acknowledgement** registry is separate and owned
by
[Zax intent acknowledgements](../../language/intent-acknowledgements.md#current-category-registry).
`intent<...>` confirms defined but suspicious behavior; it supplies no safety
fact and disables no runtime check. Do not duplicate its categories here.

The root legacy compiler-directive page formerly contained historical panic
names; they and their legacy meanings are preserved below under
[legacy panic identifiers](#legacy-panic-identifiers-requiring-disposition).
They are evidence to disposition by value, not the current registry and not
authority for syntax or behavior.

## Registry maturity

Entries use these local maturity labels:

- **Aligned candidate**: aligned in active numbered work but not promoted.
- **Accepted domain entry**: promoted behavior whose centralized
  analysis-control registry placement remains future work.
- **Provisional**: preserved evidence or a candidate name requiring focused
  review.
- **Illustrative**: appears in current conceptual prose only to demonstrate a
  mechanism; its spelling is not accepted.
- **Unassigned**: a domain requires a category, but no identifier is yet
  proposed.
- **Legacy**: historical evidence requiring value-based disposition.
- **Superseded**: replaced by a named mechanism; retained only so its evidence
  and replacement remain discoverable.

An occurrence in source does not raise its maturity.

## Safety-category inventory

Each category identifies one exact fact. `safe<...>` categories are facts about
state the language tracks; `unsafe<...>` categories concern things it does not
track. Categories do not disable a defined panic check merely by sharing a
failure condition. The current list, with an example for each, is taught by
[safety and analysis](../../language/safety-and-analysis.md#current-categories);
this ledger records maturity and history.

| Identifier | Marker | Maturity | Fact | Behavior owner |
| --- | --- | --- | --- | --- |
| `distinct-element` | `safe` | Accepted domain entry (name may be revised) | Two element accesses name different elements, or an element not finished earlier | [Arrays and slices](../../language/arrays-and-slices.md#what-the-compiler-can-prove) |
| `optional-presence` | `safe` | Accepted domain entry (name may be revised) | The optional's boxed value is present | [Optional values](../../language/optional-values.md#presence-proof-and-postfix-access) |
| `pointer-presence` | `safe` | Accepted domain entry (name may be revised) | This pointer is not vacant here; replaces the superseded `possible-nothing-write` and `possible-nothing-read-trap` | [Nothing instances](../../language/nothing-instances.md#possible-vacancy-needs-a-presence-claim) |
| `replacement-alias` | `safe` | Accepted domain entry (name may be revised) | A replacement's destination and right-hand operand do not alias in a way that matters | [Optional values](../../language/optional-values.md#presence-proof-and-postfix-access), [construction](../../language/construction-and-destruction.md) |
| `lifetime-escape` | `safe` | Accepted domain entry (name may be revised) | A reference's target outlives its use where the compiler lost track of its origin | [Lifetimes and references](../../language/lifetimes-and-references.md) |
| `unique-raw-borrow` | `safe` | Accepted domain entry (name may be revised) | A raw pointer does not come from inside a `unique` allocation; absorbs the superseded `raw-from-reference` | [Pointers, allocation, and arenas](../../language/pointers-and-arenas.md#raw-pointers-and-unique-allocations) |
| `valid-representation` | `safe` | Accepted domain entry (name may be revised) | These bits are a valid value of this type; renamed from `union-lens-validity` | [Unions](../../language/unions.md#unsafe-union), [floating-point scalars](../../language/floating-point-scalars.md) |
| `exact-origin` | `safe` | Accepted domain entry (name may be revised) | A member reference came from the named container member; replaces `unsafe outer cast` and `unsafe via` | [Composition](../../language/composition.md#proved-tracked-and-claimed-outer-casting) |
| `opaque-type` | `safe` | Accepted domain entry (name may be revised) | An erased owner or observer holds this type; replaces `unsafe transfer` | [Pointers, allocation, and arenas](../../language/pointers-and-arenas.md#safe-owner-recovery) |
| `partial-instance-access` | `safe` | Accepted domain entry (name may be revised) | Code given an incomplete instance touches only constructed members | [Construction and destruction](../../language/construction-and-destruction.md) |
| `partial-instance-escape` | `safe` | Accepted domain entry (name may be revised) | Code receiving an incomplete instance does not use it before it is complete | [Construction and destruction](../../language/construction-and-destruction.md) |
| `destruction-path-complete` | `safe` | Accepted domain entry (name may be revised) | Every destructor path ended the member | [Construction and destruction](../../language/construction-and-destruction.md#destructor-declarations-and-sequence) |
| `destruction-at-most-once` | `safe` | Accepted domain entry (name may be revised) | Apparently overlapping paths never end one member twice | [Construction and destruction](../../language/construction-and-destruction.md#destructor-declarations-and-sequence) |
| `terminal-reconstruction` | `safe` | Accepted domain entry (name may be revised) | A member rebuilt during destruction ends exactly once more | [Construction and destruction](../../language/construction-and-destruction.md#reconstruction-during-destruction) |
| `not-finished` | `safe` | Accepted domain entry (name may be revised) | A member or value was not finished by an `as last` the compiler cannot rule out | [Transfer stances](../../language/transfer-stances.md#members) |
| `disjoint-storage` | `safe` | Accepted domain entry (name may be revised) | A structural transform's source and destination do not overlap | [Structural shapes](../../language/structural-shapes-and-compatibility.md#overlap-is-handled-statically) |
| `no-mutation-while-held` | `safe` | Accepted domain entry (name may be revised) | Nothing writes a value while an `immutable` path to it is held | [Qualifiers](../../language/qualifiers.md#ordinary-promise-strengthening) |
| `opaque-construction` | `unsafe` | Accepted domain entry (name may be revised; place-naming syntax open) | An opaque operation constructs the named empty member or result slot; the member receives no automatic construction | [Construction and destruction](../../language/construction-and-destruction.md#construction-by-an-opaque-operation), [declarations and bindings](../../language/declarations-and-bindings.md#low-level-initialization-belongs-in-a-constructor) |
| `manual-member-destruction` | `unsafe` | Accepted domain entry (name may be revised) | An opaque operation ends the named member; its automatic destruction does not run | [Construction and destruction](../../language/construction-and-destruction.md#destructor-declarations-and-sequence) |
| `manual-member-construction` | — | Superseded (032) | Replaced by explicit construction control plus `opaque-construction` | — |
| `construction-path-complete` | — | Superseded (032) | Unknown construction state is an error to restructure, not a fact to assert | — |
| `construction-at-most-once` | — | Superseded (032) | Unknown construction state is an error to restructure, not a fact to assert | — |
| `union-lens-validity` | — | Superseded | Renamed `valid-representation` | — |
| `raw-from-reference` | — | Superseded | Merged into `unique-raw-borrow` | — |
| `possible-nothing-write`, `possible-nothing-read-trap` | — | Superseded | Replaced by `pointer-presence` | — |
| `replacement-member-successor` | — | Superseded | Not a safety question; became `intent<reference-tracks-replacement>` in the [intent registry](../../language/intent-acknowledgements.md#current-category-registry) | — |

### Operations that are not categories

`unsafe pliable`, `unsafe cast`, `unsafe cast as`, `unsafe as coercive`,
`unsafe from`, `unsafe vacate`, `union unsafe`, and `* unsafe` are domain
source forms, not category identifiers. Opaque raw-pointer facts such as
allocation root, alignment, and sole disposition are what `unsafe cast` and
`unsafe cast as last` already claim, so they need no separate category.

### Unassigned pressure

The following reached concerns still need focused review:

- optional or debug-only unchecked variant access if future work wants a local
  source form distinct from disabling its registered panic check; and
- the lasting list of array element cases every compiler must prove, which
  decides when `safe<distinct-element>` is needed; current teaching must not
  claim that a compiler may accept more without the claim.

Resolved since earlier audits: single consumption of source results and
destination slots needs no category, because routing is written in the call;
operations on moved-from or finished values use `not-finished`; returned
references from opaque origins use `lifetime-escape`; structural non-overlap
uses `disjoint-storage`; opaque raw-pointer facts belong to `unsafe cast`;
recovery of type-erased owners uses `opaque-type`; and union-reference escape
uses `valid-representation`.

Complete construction and at-most-once establishment of result slots and
explicitly controlled members need no category: the construction owner's
first-construction rule makes unknown state an error to restructure.

Do not assign identifiers merely to complete the table. A future category must
be placed on one side of the `safe<...>` / `unsafe` test and distinguished from
a known-invalid operation.

## Panic-category inventory

A panic category names one defined runtime failure whose required check can be
controlled independently. Unless its domain says otherwise:

- the default contract performs the check and panics on violation;
- static proof may eliminate the check;
- a statically proved violation remains an error;
- disabling the category promises that the condition cannot occur and permits
  check removal; and
- violation while disabled has undefined consequences.

Reporting or optional operations retain checks needed to produce their declared
results. Disabling a panic category does not turn required source into a
reporting form.

| Identifier | Maturity | Checked condition | Domain and disabled-check promise |
| --- | --- | --- | --- |
| `never-path-reached` | Accepted domain entry | A `[<never>]` branch, case, or point is reached | [Core flow control](../../language/core-flow-control.md#branch-hints-and-path-promises) and [switch](../../language/switch.md#case-hints-and-promises). Enabled by default; disabling promises the path is unreachable. |
| `always-path-missed` | Accepted domain entry | An `[<always>]` branch or case is not the one taken | [Core flow control](../../language/core-flow-control.md#branch-hints-and-path-promises) and [switch](../../language/switch.md#case-hints-and-promises). Enabled by default; disabling promises the marked path is always taken. |
| `inactive-variant-access` | Accepted domain entry | Direct access names a variant alternative that is not active because another name or absence is current | [Zax variants](../../language/variants.md#presence-and-named-access-answer-different-questions) and [safety and analysis](../../language/safety-and-analysis.md#checked-variant-access). Enabled by default; disabling promises the requested alternative is active and does not apply to `.=` or a case-routed binding. |

### Unassigned required panic categories

Current conceptual and raw owners require independently controllable categories
for these conditions but have not accepted exact identifiers:

- required allocation request failure;
- required fixed-width arithmetic result not representable;
- required division or remainder with a zero divisor;
- intrinsic array element index outside bounds;
- array splice endpoint or order outside bounds;
- requested array logical length outside its admitted range;
- array initializer cardinality mismatch;
- invalidated slice use;
- foreign slice origin supplied to mutation;
- iterable exact-count promise mismatch;
- array backing-extent overflow;
- required array backing allocation failure;
- pointer arithmetic requiring an ordinary target when the pointer reaches
  Nothing backing;
- access to a compiler-prepared member with no readable representation;
- access under a trapping Nothing policy;
- write to compiler-provided Nothing backing where a target-supported trap is
  part of the selected contract; and
- invocation of an unavailable function representation through a generated
  panic path; and
- a call of a function that is not `once` with a Nothing receiver, checked at
  function entry on a target without a hardware trap.

The array conditions are listed by
[arrays and slices](../../language/arrays-and-slices.md#required-reporting-and-unchecked-behavior)
and [raw analysis controls](analysis-controls.md#panic-registry-pressure-from-intrinsic-arrays).
Nothing-instance conditions are preserved by
[Nothing instances](../../language/nothing-instances.md) and
[raw analysis controls](analysis-controls.md#nothing-instance-diagnostics-and-controls).
Allocation, arithmetic, and zero-divisor contract pressure is preserved by
[raw safety](safety.md#panic-boundary) and
[raw analysis controls](analysis-controls.md#general-runtime-check-contract-pressure).

### Legacy panic identifiers requiring disposition

The legacy compiler-directive registry contained these names, with the legacy
default state in parentheses and its legacy meaning:

- `out-of-memory` (enabled): memory was requested but insufficient memory
  existed; allocation failure normally panicked rather than returning a vacant
  pointer;
- `intrinsic-type-cast-overflow` (enabled): an intrinsic value overflowed during
  `as` to a type with fewer bits;
- `string-conversion-contains-illegal-sequence` (enabled): a string literal
  conversion found an illegal character sequence;
- `not-all-pointers-destructed-during-allocator-cleanup`: allocator cleanup ran
  while instances it allocated were not all destructed;
- `impossible-switch-value`, `impossible-if-value`, and `impossible-code-flow`
  (enabled): a value or path marked impossible by `[<never>]` or `[<always>]`
  occurred. **Superseded** by `never-path-reached` and `always-path-missed`
  below;
- `lazy-already-complete` (enabled): a `lazy` function was called after its
  final `return`; and
- `value-polymorphic-function-not-found` (enabled): a value-polymorphic
  function was called but none of its precondition checks succeeded.

None is promoted by appearing here. Future analysis-control work must compare
each name and behavior with current allocation, scalar, string, pointer, flow,
callable, panic, and safety design, then retain, rename, split, reject, or
supersede it explicitly. The legacy `reference-from-pointer-to-nothing` and
`pointer-to-nothing-accessed` entries are already marked superseded by current
Nothing-instance direction and must not be revived as one collapsed condition.

## Occurrence audit

The initial live-source audit on 2026-09-20 searched:

- exact `unsafe<...>` examples;
- prose and tables naming unsafe categories;
- `panic category`, `panic-category`, registry, disablement, and directive
  wording;
- exact `intent<...>` examples; and
- the legacy panic registry for value-based disposition.

An audit on 2026-11-26, during the `safe<...>` / `unsafe` split, found
`safe<...>` and `unsafe<...>` examples in:

- `language/safety-and-analysis.md`;
- `language/arrays-and-slices.md`;
- `language/unions.md`;
- `language/optional-values.md`;
- `language/intent-acknowledgements.md`;
- `language/construction-and-destruction.md`;
- `language/declarations-and-bindings.md`;
- `language/pointers-and-arenas.md`;
- `language/composition.md`, `language/function-invocation.md`, and
  `language/operator-catalog.md` (`exact-origin`);
- `language/nothing-instances.md`, `language/qualifiers.md`,
  `language/casting.md`, `language/floating-point-scalars.md`, and
  `language/structural-shapes-and-compatibility.md`; and
- `project/raw/analysis-controls.md`.

Panic-category obligations without a current registry occur in:

- `language/variants.md`;
- `language/arrays-and-slices.md`;
- `language/construction-and-destruction.md`;
- `language/nothing-instances.md`;
- `language/pointers-and-arenas.md`;
- `language/safety-and-analysis.md`;
- `project/raw/analysis-controls.md`;
- `project/raw/array-storage-strategies.md`; and
- `project/raw/safety.md`.

The accepted intent registry covers current accepted intent names, including
`inner-pointer-tracks-replacement` (formerly
`anchored-pointer-tracks-replacement`) and `reference-tracks-replacement`,
which were promoted together.

Archived work preserves provenance but is not a live registry source. A future
audit may inspect it when a current category's provenance or regression cannot
be resolved from live owners and raw inputs.

## Addition and retirement rules

Add a registry entry when a current work item or live owner establishes a
concrete safety category or independently selectable runtime panic
condition. Record:

- the exact or provisional identifier;
- maturity;
- the fact claimed, its `safe` or `unsafe` marker, or the disabled-check
  meaning;
- behavior owner;
- false-claim or disabled-check consequence;
- representative live occurrences; and
- the pressure that must disposition it.

Do not add broad aliases for convenience, convert intent categories into safety
categories, or import a legacy panic name without current semantic review.

When accepted analysis-control owners exist, move each surviving entry into its
one lasting registry and domain owner. Retire this file after every provisional,
unassigned, and legacy entry has been promoted, renamed, rejected, superseded,
or moved to another indexed live destination.
