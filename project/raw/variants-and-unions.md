# Raw input: variants and unions

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work defining managed alternatives or unmanaged overlay storage |
| Applies To | Active-alternative lifetime, conditional access, overlay representation, and manual lifecycle pressure |
| Owns | Preservation of the managed-variant versus unmanaged-union distinction, lifecycle constraints, reference consequences, activation pressure, and retirement criteria |
| Does Not Own | Accepted variant/union syntax or current optional/reference behavior |
| Source / Provenance | Work item `014`; optional, instance-place, replacement, and low-level storage review |

## Candidate distinction

Candidate terminology:

- **variant**: managed storage with exactly one live alternative path;
- **union**: unmanaged overlay storage viewed through different type lenses.

These names and their source forms are not accepted.

No reliable representative declaration syntax exists yet. Future work should not
invent one until type declaration, selection, access, and lifecycle requirements
are reviewed together.

## Managed alternative pressure

A managed variant behaves like an optional generalized to several alternatives:

- exactly one alternative path is present;
- every other alternative is absent;
- changing alternatives ends one path and begins another;
- references into the old alternative do not follow the new alternative;
- access requires proof of the active alternative; and
- the complete variant path bounds every alternative path.

Future work must define:

- construction and default state;
- active-alternative testing and binding;
- replacement and transfer;
- destruction order;
- recursive alternatives;
- representation and discriminant cost;
- exhaustive selection;
- qualification of wrapper versus active value;
- and diagnostics for inactive access.

## Unmanaged overlay pressure

An unmanaged union would expose the same storage through several type lenses.
It may be suitable only when:

- every view has trivial lifecycle requirements; or
- the programmer explicitly performs and tracks manual `+++` and `---`
  operations under unsafe responsibility.

Future work must define:

- which resident instance is considered live;
- whether any safe access exists without an active-member tag;
- alignment and size;
- aliasing and provenance;
- initialization and destruction;
- reference validity when the chosen lens changes;
- interaction with immutable and varying places;
- and whether a managed wrapper can safely encapsulate unsafe overlay use.

Shared address and bytes do not prove that two typed resident instances are
simultaneously live.

## Current constraints

Future design must preserve:

- [life paths and instance places](../../language/lifetimes-and-references.md);
- complete replacement renewing member resident instances;
- optional-style conditional paths for managed alternatives;
- narrow unsafe responsibility rather than broad unsafe mode; and
- no reference rebinding when the active alternative changes.

## Activation and retirement

Activate this input when variant, union, tagged-alternative, active-member,
overlay-memory, or exhaustive-alternative selection is reviewed.

Move accepted behavior into a dedicated type owner and the applicable
construction, selection, lifetime, qualifier, safety, and layout owners, then
retire this file.
