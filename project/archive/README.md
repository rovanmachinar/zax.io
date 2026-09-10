# Zax project archive

| Field | Value |
| --- | --- |
| Status | Historical routing guidance |
| Audience | Humans and agents performing a targeted audit |
| Applies To | Completed project working material |
| Owns | Archive navigation and non-authority warning |
| Does Not Own | Current project guidance or accepted Zax language design |

Archived files are retained for discovery and targeted audits. They are
historical, non-normative, and excluded from ordinary onboarding and design
reading.

Do not inspect this directory by default. Read an archived file only when the
language maintainer directly requests it or a concrete audit or recovery need
identifies it. Current documents must not depend on archived material for
meaning.

Completed numbered work mirrors its active path:

```text
project/work/00n-<concept>.md
    ->
project/archive/work/00n-<concept>.md
```

## Archived work

| Work item | Concern | Completed outcome |
| --- | --- | --- |
| [001](work/001-concept.md) | Zax purpose and design principles | Promoted to the current language vision and indexed raw future-work inputs |
| [002](work/002-source-structure.md) | Source and statement structure | Promoted to the current source-structure owner and reconciled affected legacy routes, examples, and warning classifications |
| [003](work/003-declarations-and-bindings.md) | Declarations, bindings, and assignment | Promoted to the current declarations-and-bindings owner and reconciled qualifier terminology, lookup, shadowing, and affected legacy examples |
| [004](work/004-qualifiers-and-mutability.md) | Qualifiers, mutability, and access capabilities | Promoted to the qualifier, declaration, and terminology owners; reconciled unsafe spelling, replacement, alias, recursive-name, and affected legacy behavior; indexed constructor and type-family future work |
| [005](work/005-construction-destruction-and-replacement.md) | Construction, destruction, and replacement | Promoted to the current construction owner; reconciled immutable replacement eligibility, lifecycle tracking, construction packets, results, legacy routes, and indexed function, analysis, lifetime, safety, async, and structural future work |
| [006](work/006-function-invocation.md) | Function invocation, argument binding, and results | Promoted to the current function-invocation and reusable-principles owners; reconciled declarations, qualifiers, construction, source structure, terminology, teaching guidance, legacy routes, and indexed future lifetime, safety, analysis, async, and structural work |
| [007](work/007-core-flow-control.md) | Core flow control, clause headers, labels, and exits | Promoted to the current core-flow and operators owners; reconciled source layout, declarations, invocation, lifecycle, terminology, foundational support, project guidance, raw selection/analysis/safety/build/audit inputs, navigation, and legacy flow/operator/optional/error/scope material |
| [008](work/008-operator-catalog-and-resolution.md) | Operator catalog, declarations, expression resolution, and mixfix | Promoted to the current operator, catalog, and mixfix owners; reconciled source layout, declarations, qualifiers, invocation, lifecycle, flow, terminology, navigation, legacy operator material, and indexed bare-source, literal, numeric, composition, indexing, partial-type, safety, analysis, build, structural, and audit inputs |
| [009](work/009-operator-phrases.md) | Operator phrases and keyword-neutral source | Promoted to the current operator-phrase, operator, catalog, mixfix, source, declaration, invocation, qualifier, lifecycle, flow, terminology, and endianness owners; reconciled phrase fencing and literal source, enum conversions, integer bit operations, legacy material, navigation, and indexed enum, generic, compile-time, reflection, bare-source, partial-type, numeric, structural, and lifetime future input |
| [010](work/010-integer-types.md) | Fundamental integer types and finite bit domains | Promoted to the current integer, identity-type, integer-operator, general operator, declaration, endian, terminology, documentation, and navigation owners; consumed legacy integer/alias/casting material; indexed CPU-provider, owned-composition, partial, generic, reflection, literal, range, safety, analysis, pointer, structural, and numeric future input |
| [011](work/011-integer-literals.md) | Integer literals, uncommitted evaluation, realization, and contextual completion | Promoted to the current integer-literal, integer, operator, declaration, invocation, flow, construction, identity, endian, terminology, and documentation owners; consumed legacy literal evidence; preserved prefixed/custom literal, compile-time, numeric, partial, owned-composition, enum, optional, generic, reflection, and CPU-profile future input |
| [012](work/012-optional-values.md) | Optional values, construction, absence, and transfer | Promoted to the current optional-value owner and integrated construction, declaration, qualifier, operator, flow, invocation, numeric, identity, terminology, source, and navigation behavior; deleted exhausted legacy optional input; preserved transfer, lifetime, analysis, safety, callable, selection, generic, async, reflection, interop, and legacy-audit future pressure |
| [013](work/013-transfer-stances-and-source-state.md) | Transfer stances, source state, and lifetime completion | Promoted the copy/deep/move/last model, intent acknowledgements, transfer-aware invocation/construction/qualifier/optional/operator behavior, teaching-debt process, and legacy disposition; preserved callable selection, lifetime, safety, analysis, async, cross-thread, generic, reflection, `own`, mutability-family, and capture pressure |
| [014](work/014-reference-origins-and-lifetime-strategies.md) | Reference origins, life paths, pointer ownership, and lifetime strategies | Promoted the life-path and instance-place model, fixed references, pointer ownership and anchoring, arenas and allocation disposition, general safety analysis, invocation lifetime ordering, raw-pointer proof boundaries, and legacy cleanup; preserved allocation syntax, arena mechanics, `Nothing`, variants/unions, async, collection, affinity, and callable-origin pressure |
| [015](work/015-allocation-operations-and-arena-boundaries.md) | Allocation operations and arena boundaries | Promoted declaration-bound allocation, arena policy and execution context, scheduled/open raw disposition, pointer reset and adoption, allocation records and roots, control-block transitions, process-wide collection, minted callable result contracts, and affected legacy/raw disposition |
| [016](work/016-enum-types-and-values.md) | Enum types and values | Promoted strict, relaxed, flags, backing, member, admission, operation, string, safety, and endian enum behavior; integrated narrow type-callable `once` functions and case-conflicting-name acknowledgement; preserved direct `each` and later concept-dependent iterator pressure |
| [017](work/017-each-iteration-and-compiler-known-traversal.md) | Each iteration and compiler-known traversal | Promoted direct `each in`, cursor-driven `each from`, entry bindings, cursor acquisition and progression, active-`each` access, coordinated erasure, mutation responsibilities, keyword escape, and affected flow, source, enum, operator, legacy, and raw integration |
| [018](work/018-switch-cases-and-value-selection.md) | Switch, cases, and value selection | Promoted ordered `switch`/`case`/`default`, selector capture, case operations, posts, labels, `goto`, transfer-only cases, positional defaults, optional proof, enum coverage, intent diagnostics, and affected flow, source, lifecycle, operator, legacy, and raw integration |
| [019](work/019-using-and-scoped-resource-lifetimes.md) | `using` and scoped resource lifetimes | Promoted structural disposal through `using`, source-first result routing, anonymous and discard destinations, optional resource participation, grouped intent acknowledgement, caller-visible destruction order, elision consequences, and affected lifecycle, invocation, declaration, flow, source, legacy, and raw integration |
