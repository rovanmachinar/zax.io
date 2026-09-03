# Raw input: selection and matching

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work reviewing `switch`, `case`, and pattern-like selection |
| Applies To | Selection constructs whose complete design was deferred by core flow control |
| Owns | Preservation of the selection concern, its legacy provenance, the aligned `case next` consequence, current-flow constraints, activation pressure, and retirement criteria |
| Does Not Own | Accepted selection semantics or current flow/operator ownership |
| Source / Provenance | Work items `007` and `012`; legacy [flow control](../../flow-control.md) `switch`, `case`, and value-polymorphism sections; optional per-layer matching pressure |

## Reading posture

Core flow control established the shared condition, clause, scope, and exit model
and explicitly deferred complete selection. This file preserves the selection
concern so it survives numbered-work archival. It accepts nothing.

It is not required for ordinary flow-control reading. Read or update it when a
focused selection work item begins or a concrete consequence establishes its
relevance.

## Preserved selection concern

Legacy [flow control](../../flow-control.md) proposes `switch`, `case`, and
`default`, including matching against runtime values, complex types, and
alternative operators. That evidence remains legacy provenance and is not
accepted here.

Selection pressure to review includes:

- `switch`, `case`, and `default` structure and body selection;
- matching against runtime values and complex or user-defined types;
- alternative comparison operators in a `case`;
- exhaustiveness, overlap, and fallthrough behavior; and
- how a selection construct participates in labels and transfer targets.

## Aligned `case next` consequence

Legacy selection uses `case continue`. Core flow control distinguishes `next`
(runs the target's post operation) from `continue` (skips it) and makes `next`
the re-entry transfer for re-enterable constructs.

The aligned consequence is that legacy `case continue` should be reviewed as
`case next` when selection is designed. This is a captured consequence, not an
accepted spelling. Selection work must decide the actual transfer behavior of a
`case` body and whether a `case` is a transfer target at all.

## What core flow already constrains

Selection design must preserve the already-accepted
[core flow control](../../language/core-flow-control.md) model:

- conditions and any Boolean guards resolve to exactly `Boolean`;
- a clause consumes one effective body statement and owns a scope around it;
- transfers destroy the scopes they leave in reverse construction order;
- `next` runs a target post operation and `continue` skips it;
- construction and result completeness must hold on every normal path; and
- bare transfers may not silently skip a labeled eligible target.

Selection may add its own matching and dispatch behavior but may not contradict
these constraints.

## Optional matching pressure

[Zax optional values](../../language/optional-values.md) gives every optional
layer independent absence/presence and boxed lifetime:

```zax
nested : MyType? ?
```

Future selection work must decide how source matches:

- outer absence;
- outer presence with inner absence;
- both layers present;
- a boxed binding under its declared qualifications.

Matching must preserve per-layer proof. A boxed binding is valid only while the
exact matched lifetime remains active; reset, packet construction, complete
wrapper replacement, or terminal transfer invalidates it. Selection syntax must
not flatten nested optionals or make a bare construction packet into an
anonymous value.

Exhaustiveness should count semantically distinct optional states rather than
representation niches. Exact binding syntax and interaction with guards remain
unaccepted.

Present/absent combinators such as illustrative
`fold(optionalValue, onPresent, onAbsent)` create the same selected-branch
pressure in expression form. Future selection work must coordinate with
function-composition input so only one callback executes, both possible result
paths converge, and a present callback receives a valid boxed lifetime without
turning wrapper method syntax into implicit postfix access.

## Activation and retirement

Activate this input when a focused selection or pattern-matching work item
begins.

That work must:

1. disposition the `switch`/`case`/`default` concern and the `case next`
   consequence;
2. decide selection's interaction with labels, transfer targets, and post
   operations under the core flow model;
3. move accepted behavior into the applicable current owner; and
4. remove this file from the raw index and retire it when no unresolved selection
   material remains.
