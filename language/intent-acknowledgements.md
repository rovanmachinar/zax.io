# Zax intent acknowledgements

| Field | Value |
| --- | --- |
| Status | Current conceptual design |
| Audience | Human developers reading, writing, reviewing, or tooling Zax source |
| Applies To | Explicit acknowledgement of defined but suspicious-looking source; not a formal grammar or diagnostic specification |
| Implementation State | Not established by this repository |
| Owns | The intent-acknowledgement mental model; `intent<category>{...}` use; distinction from unsafe responsibility and lint suppression; the current category registry; common costs, diagnostics, and source-stability requirements |
| Does Not Own | Layout and enclosure mechanics ([source structure](source-structure.md)); transfer semantics ([transfer stances](transfer-stances.md)); domain behavior gated by a category; unsafe proof assertions; or exact diagnostic identifiers |
| Source / Provenance | Transfer terminal-use review and prior confusable-form intent input |

## Defined source can still look mistaken

Some source is valid under its domain contract but strongly resembles a mistake.
Zax requires the programmer to acknowledge that intent rather than accepting the
source silently.

For example, ordinary use after terminal transfer is suspicious:

```zax
processed := record as last
record.releaseRemainingHandle()
// error: record is being used after accepted terminal transfer
```

If `releaseRemainingHandle` is defined for the record's terminal state, the
programmer can say that the use is deliberate:

```zax
processed := record as last

intent<terminal-source-reuse>{
  record.releaseRemainingHandle()
}
```

An **intent acknowledgement** says:

> The compiler understood this defined source correctly. I deliberately intend
> the suspicious interpretation named by this category.

It does not grant capability, change types, select another operation, or make an
invalid program valid.

## Source form

The conceptual source form is:

```zax
intent<category>{
  source
}
```

The category names the exact concern being acknowledged. The payload remains
ordinary Zax source:

- type and qualifier checking still applies;
- callable and operator selection still applies;
- lifetime and alias requirements still apply;
- layout and source structure still apply;
- and unrelated diagnostics remain active.

The acknowledgement adds no required runtime operation or check.

The braces create no scope or lifetime boundary:

```zax
intent<category>{
  value : Integer
}

value = 5
```

Complete tokenization, payload shape, layout, and source-tree behavior belong to
[Zax source structure](source-structure.md).

## Intent, unsafe, and lint suppression are different

These mechanisms answer different questions:

| Mechanism | Programmer statement |
| --- | --- |
| `intent<...>{...}` | "This defined but suspicious interpretation is deliberate." |
| `unsafe<...>{...}` | "This operation is valid, and I accept responsibility for a required property the compiler cannot prove." |
| Lint suppression | "Do not report this advisory diagnostic here." |

### Intent is not unsafe

```zax
intent<terminal-source-reuse>{
  record.releaseRemainingHandle()
}
```

This is appropriate only when the operation is actually defined for terminal
state. The acknowledgement does not revive an ended lifetime, grant writable
access, override immutability, or make duplicate ownership valid.

An unproved lifetime or alias property belongs to a future narrow unsafe
assertion instead:

```zax
unsafe<replacement-alias>{
  destination = sourceView
}
```

The unsafe syntax and category shown here remain subject to the separate safety
and analysis-control design.

### Intent is not a warning switch

Intent errors are hard source errors until acknowledged or rewritten. A linter
cannot change whether source is accepted, and lint suppression cannot provide
an intent acknowledgement.

## Current category registry

Category names describe the semantic situation being acknowledged rather than
copying a diagnostic message.

| Category | Meaning acknowledged | Complete behavior owner |
| --- | --- | --- |
| `implicit-stance-at-terminal-use` | Keep implicit transfer stance at a proven terminal opportunity where explicit stance could materially change the accepted contract | [Transfer stances](transfer-stances.md#terminal-opportunity-must-be-explicit) |
| `terminal-source-reuse` | Perform a defined operation after accepted `last` transfer | [Transfer stances](transfer-stances.md#last) |
| `asymmetric-saturating-magnitude` | Parse `\|\|value\|` as asymmetric saturating magnitude rather than malformed norm | [Integer operator catalog](integer-operator-catalog.md#magnitude) |

### Implicit stance at terminal use

```zax
intent<implicit-stance-at-terminal-use>{
  forward(resource)
}
```

This confirms that ordinary implicit transfer behavior should remain in effect
even though analysis found a material terminal opportunity. It does not offer
`last`.

The clearer ordinary repair is often to state the desired stance:

```zax
forward(resource as last)
forward(resource as move)
forward(resource as copy)
forward(resource as deep)
```

The same category applies when an owned by-value result reaches its structurally
final consumer and its producer left stance implicit:

```zax
intent<implicit-stance-at-terminal-use>{
  consume(makePayload())
}
```

This keeps the implicit `copy` result behavior. It does not offer `last`.

### Terminal-source reuse

```zax
consume(resource as last)

intent<terminal-source-reuse>{
  resource.releaseRemainingHandle()
}
```

The selected `last` consumer has already established terminal source state.
Acknowledgement permits a later operation only when the type's terminal contract
defines that operation.

### Asymmetric saturating magnitude

```zax
saturated :=
  intent<asymmetric-saturating-magnitude>{ ||value| }
```

Natural `||value|` strongly resembles a norm expression missing its final `|`.
The category confirms the asymmetric saturating-magnitude interpretation.

`bare{...}` is not the intent mechanism. Its remaining future role is
keyword-neutral source.

## Choosing category boundaries

A category should be:

- narrow enough to identify one suspicious semantic situation;
- reusable wherever that same situation appears;
- independent from incidental diagnostic wording;
- distinct from unsafe proof responsibility;
- and discoverable in this registry.

One acknowledgement may name only categories applicable to its payload.
Unrelated or inapplicable categories are errors rather than universal
suppression.

Future categories require review of:

- the valid source they acknowledge;
- why ordinary source is rejected;
- the behavior owner;
- interaction with unsafe and lints;
- source-reflection and formatting requirements;
- and compatibility consequences.

## Costs and tooling

Intent acknowledgement has no required runtime cost. It creates source and
tooling obligations:

- formatters must retain the category and enclosure;
- source reflection must preserve the acknowledgement;
- diagnostics should state which category is required and why;
- editors should link or route the category to this registry and its domain
  owner;
- code review and search tools should make acknowledgements easy to audit;
- and build or language-version changes must not silently reinterpret one
  category as another.

## Diagnostics

Diagnostics should distinguish:

- defined suspicious source missing an acknowledgement;
- unknown category;
- category not applicable to the enclosed source;
- acknowledgement that cannot make a known-invalid operation valid;
- unsafe responsibility incorrectly expressed as intent;
- lint suppression incorrectly used for a semantic error;
- and malformed acknowledgement source.

The diagnostic should identify the suspicious interpretation, a positive
explicit rewrite when one exists, and the applicable category.

## Source stability

- Adding a newly recognized intent error may make previously accepted source
  require explicit acknowledgement, but it must not silently change behavior.
- Once present, an acknowledgement retains its category through formatting and
  source-preserving tooling.
- Category meaning cannot change according to compiler optimization or analysis
  quality.
- Removing an acknowledgement is a source-semantic edit, not cosmetic cleanup.
- Exact future diagnostic identifiers may evolve without changing category
  meaning.

## Boundaries and maturity

This document is current conceptual design, not a formal grammar, diagnostic
catalog, conformance contract, or implementation mapping.

Future work remains for:

- exact enclosure grammar and category-list syntax;
- declaration-level or expression-only acknowledgement forms;
- source reflection;
- category namespaces and extension policy;
- interaction with generated source and build contracts;
- redundant acknowledgement policy as analysis improves;
- unsafe category syntax;
- lint declaration and suppression syntax;
- and formal diagnostic identifiers.
