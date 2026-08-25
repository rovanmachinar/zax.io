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
