# Zax intent acknowledgements

| Field | Value |
| --- | --- |
| Status | Current conceptual design |
| Audience | Human developers reading, writing, reviewing, or tooling Zax source |
| Applies To | Explicit acknowledgement of defined but suspicious-looking source; not a formal grammar or diagnostic specification |
| Implementation State | Not established by this repository |
| Owns | The intent-acknowledgement mental model; acknowledgeable versus non-acknowledgeable intent errors; `intent<category>{...}` use; the current category registry; intent-specific costs, diagnostics, and source-stability requirements |
| Does Not Own | Layout and enclosure mechanics ([source structure](source-structure.md)); transfer semantics ([transfer stances](transfer-stances.md)); domain behavior gated by a category; the complete intent/unsafe/lint distinction, unsafe assertions and permissions ([safety and analysis](safety-and-analysis.md)); or exact diagnostic identifiers |
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

The category names the exact concern being acknowledged. Each category defines
the smallest complete contextual source unit it may enclose. The payload remains
ordinary Zax source:

- type and qualifier checking still applies;
- callable and operator selection still applies;
- lifetime and alias requirements still apply;
- layout and source structure still apply;
- and unrelated diagnostics remain active.

Most categories enclose a complete expression, effective statement, or
declaration. A category may instead permit another complete contextual unit,
such as one complete switch clause. No acknowledgement can borrow a required
operand, separator, declaration piece, or body from outside its enclosure.

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

`intent<...>{...}` says: “This defined but suspicious interpretation is
deliberate.” It does not accept proof responsibility or suppress an advisory
lint. The complete distinction among intent, unsafe assertions, unsafe
permissions, and lint suppression is owned by
[Zax safety and analysis](safety-and-analysis.md#intent-unsafe-and-linting-are-different).

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

The complete distinction is defined by
[Zax safety and analysis](safety-and-analysis.md). Exact unsafe syntax and
categories remain future analysis-control design.

### Intent is not a warning switch

Intent errors are hard source errors until acknowledged or rewritten. A linter
cannot change whether source is accepted, and lint suppression cannot provide
an intent acknowledgement.

### Non-acknowledgeable intent errors

Not every diagnostic about apparent intent has a valid interpretation to
acknowledge.

- An **acknowledgement-required intent error** has one defined but
  suspicious-looking meaning. An applicable `intent<category>{...}` enclosure
  confirms that meaning.
- A **non-acknowledgeable intent error** recognizes a malformed, duplicate, or
  deliberately forbidden near miss so the compiler can provide a precise repair.
  It must be rewritten.

For example, redundant detached control-block placement has one meaning:

```zax
intent<redundant-control-placement>{
  owner : MyValue * strong =
    @{
      control: ControlBlockPlacement.Detached,
      controlArena: myControlArena
    }
}
```

By contrast, an empty allocation-policy enclosure has no accepted form:

```zax
owner : MyValue * unique = @{} // error: use @
```

Wrapping the latter in `intent<...>` cannot create a valid initializer.

A redundant switch-targeting tail `break` is another non-acknowledgeable intent
error:

```zax
switch value {
  case 1
    handleOne();
    break // error: normal case completion already exits the switch
}
```

It must be removed unless it skips a case/switch post or names an outer target.
An acknowledgement cannot preserve the misleading implication that ordinary
cases fall through. Complete behavior belongs to
[switch, case, and default](switch.md#redundant-tail-break).

A `default` reachable through no ordered search is also non-acknowledgeable. It
is not functioning as a fallback and must be rewritten as a transfer-only
labeled case.

## Current category registry

Category names describe the semantic situation being acknowledged rather than
copying a diagnostic message.

| Category | Meaning acknowledged | Complete behavior owner |
| --- | --- | --- |
| `implicit-stance-at-terminal-use` | Keep implicit transfer stance at a proven terminal opportunity where explicit stance could materially change the accepted contract | [Transfer stances](transfer-stances.md#terminal-opportunity-must-be-explicit) |
| `terminal-source-reuse` | Perform a defined operation after accepted `last` transfer | [Transfer stances](transfer-stances.md#last) |
| `asymmetric-saturating-magnitude` | Parse `\|\|value\|` as asymmetric saturating magnitude rather than malformed norm | [Integer operator catalog](integer-operator-catalog.md#magnitude) |
| `redundant-control-placement` | Deliberately restate detached placement even though `controlArena:` already implies it | [Pointers, allocation, and arenas](pointers-and-arenas.md#allocation-policy-enclosure) |
| `conditionally-unallocated-member` | Deliberately suppress a member's declared automatic allocation while permitting a normal constructor path to leave the pointer at `Nothing` | [Construction and destruction](construction-and-destruction.md#automatic-and-explicit-member-construction) |
| `case-conflicting-enum-member-names` | Declare ASCII case-equivalent enum member names with different enum values | [Enums](enums.md#ascii-case-insensitive-lookup) |
| `unreachable-selection-clause` | Deliberately retain one semantically proven unreachable complete `case` or `default` clause | [Switch, case, and default](switch.md#ordering-effects-and-overlap) |
| `empty-selection` | Deliberately retain a runtime switch containing no clauses | [Switch, case, and default](switch.md#empty-selection) |
| `outer-target-through-ineligible-label` | Deliberately select an eligible outer target through a nearer same-named label that is ineligible for the written transfer keyword | [Core flow control](core-flow-control.md#label-namespace-and-shadow-permission) |
| `partial-enum-selection` | Deliberately omit one or more distinct declared enum member values from explicit switch coverage | [Enums](enums.md#selection-with-enum-values) |

Anchored owning pointers also require intent acknowledgement when replacement
of their target or an enclosing direct place can renew the resident member
instance. That behavior is defined by
[pointers and arenas](pointers-and-arenas.md#replacement-intent). Its exact
category identifier remains unsettled and examples mark the spelling
provisional rather than adding it to this accepted registry prematurely.

### Case-conflicting enum member names

ASCII case-equivalent enum member names with different values strongly resemble
a naming mistake:

```zax
intent<case-conflicting-enum-member-names>{
  BoxKind :: enum U8 {
    BlueBox = 1
    Bluebox = 2
  }
}
```

The acknowledgement encloses the complete strict, relaxed, or flags enum
declaration. It permits both declarations but changes neither value and chooses
no preferred case-insensitive result. Exact lookup can distinguish the names;
unique case-insensitive lookup returns absence for the ambiguous folded name.
Complete conversion behavior is defined by
[Zax enums](enums.md#ascii-case-insensitive-lookup).

### Unreachable selection clause

A clause proven unreachable under the complete flow graph may be retained by
acknowledging that one complete contextual clause:

```zax
switch mode {
  case MyMode.Primary
    usePrimary()

  // PrimaryAlias has the same value as Primary under generated equality.
  intent<unreachable-selection-clause>{
    case MyMode.PrimaryAlias
      useAlias()
  }
}
```

The category neither changes case order nor makes the body reachable. Its
payload includes the clause's required body and occupies one complete clause
position in the switch. Enclosing only the test while borrowing a body from
outside remains invalid.

A reachable `goto` or `continue` edge can make an ordinarily shadowed clause body
reachable, in which case this category is inapplicable. Uncertain or
runtime-dependent overlap does not require acknowledgement.

### Empty selection

An empty switch has defined header and post behavior but strongly resembles
unfinished source:

```zax
intent<empty-selection>{
  switch initialize() ;; selectValue() ;; recordCompletion() { }
}
```

The acknowledgement preserves the empty selection. It does not remove
initializer, selector, post, or lifetime effects.

### Outer target through an ineligible label

A nearer same-named label can be ineligible for one transfer keyword while an
outer target remains eligible:

```zax
while shadowable retry: condition {
  switch value {
    case retry: 1
      handleOne()
    case 2 {
      intent<outer-target-through-ineligible-label>{
        next retry: // deliberately target the outer loop
      }
    }
  }
}
```

The acknowledgement confirms the eligible outer target. It cannot make the
nearer case a valid `next` target or reach an otherwise hidden or inactive
construct. Complete lookup belongs to
[core flow control](core-flow-control.md#label-namespace-and-shadow-permission);
case eligibility belongs to
[switch, case, and default](switch.md#labels-and-target-visibility).

### Partial enum selection

Enum switches audit explicit declared-member coverage separately from coverage of
unnamed or unsafe-admitted backing values. `default` can cover the latter without
silently covering an omitted declared member.

```zax
intent<partial-enum-selection>{
  switch status {
    case Status.Ready
      startWork()
    default
      handleOtherDeclaredOrUnnamedStatus()
  }
}
```

The acknowledgement deliberately permits omitted declared values such as
`Status.Waiting`. It does not make the switch exhaustive over unnamed values,
change case order, or alter the selected comparison operations. Adding a new
declared member continues to require review of an unacknowledged member-complete
switch.

Complete enum-domain rules belong to
[Zax enums](enums.md#selection-with-enum-values); per-search-entry behavior
belongs to [switch, case, and default](switch.md#enum-coverage-has-two-dimensions).

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

A scheduled raw result also requires this acknowledgement when its declaration
leaves stance implicit:

```zax
intent<implicit-stance-at-terminal-use>{
  makeValue final : (
    result : MyValue * = @
  )() = {
  }
}
```

The clearer ordinary declaration is `result : MyValue * last = @`. Explicit
`copy` or `move` instead states a deliberate borrowed outward contract and
remains subject to caller lifetime proof.

### Conditionally unallocated member

```zax
MyType :: type {
  member : Member * = @

  +++ final : ()(
    condition : Boolean,
    arena : MyArena &
  ) = {
    intent<conditionally-unallocated-member>{
      if condition
        _.member = @{ arena }
    }
  }
}
```

The direct member allocation suppresses automatic pointee allocation for this
constructor. The acknowledgement confirms that a normal path may deliberately
leave the valid scheduled pointer at `Nothing`; it does not permit an
indeterminate pointer representation.

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
