# Raw input: analysis controls

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work defining unsafe controls, static-analysis contracts, diagnostics, lints, or language-version behavior |
| Applies To | Source-local semantic permissions and assertions, analysis provenance, contract-version evolution, and the boundary between unsafe controls and lint suppression |
| Owns | Preservation of construction-derived requirements, concrete examples, the contract-dependent severity of redundant controls, unresolved syntax, activation pressure, and retirement criteria |
| Does Not Own | Accepted unsafe syntax, safety guarantees, diagnostic identifiers, lint syntax, language contracts, compiler extensions, or implementation |
| Source / Provenance | Work items `005`, `006`, and `007`; construction/lifecycle, invocation/result, and core-flow analysis pressure |

## Reading posture

[Zax construction, replacement, and destruction](../../language/construction-and-destruction.md)
identifies places where compiler analysis may reject a valid low-level program.
This file preserves the future source-control and portability questions without
making provisional spellings current language syntax.

It is not required for ordinary construction or qualifier use.

## Semantic controls are not broad unsafe mode

A future semantic control must:

- use the literal `unsafe`;
- identify the exact unsafe category;
- name the resolved member, binding, expression, or operation where applicable;
- apply at the narrowest useful source location;
- affect only the stated analysis or generated behavior;
- permit attached `///` documentation explaining the claim; and
- have no required runtime check.

Compiler debugging modes may instrument a claim and panic when misuse is
detected. Such instrumentation is tooling, not a language guarantee.

## Permission regions

Some operations are valid only when the programmer accepts responsibility for a
specific analysis boundary.

Illustrative syntax:

```zax
/// The callback consumes the view synchronously and stores nothing.
unsafe <lifetime-escape> {
  myCaptureBeyondLifetime(myType.view())
}
```

The angle-bracketed category is placeholder notation, not accepted syntax.

An unsafe grouping block should not silently create a lexical scope. Either the
future construct is non-scoping or it uses the established descope mechanism:

```zax
unsafe <lifetime-escape> [[descope]] {
  x : Integer
  myCaptureBeyondLifetime(myType.view())
}

x = 5
```

## Lifecycle-state assertions

An opaque operation may establish a member state that ordinary analysis cannot
observe:

```zax
opaqueInitialize(_.prop2)

/// opaqueInitialize establishes prop2 on every normal path.
unsafe <construction-path-complete> _.prop2
```

The future mechanism must distinguish:

- suppressing a generated lifecycle call;
- asserting that an opaque operation established or ended a lifetime;
- asserting completion on every normal path; and
- asserting that apparently overlapping calls cannot affect one lifetime twice.

A warning-suppression mechanism is insufficient because these claims may change
generated code or the compiler's semantic state.

## Known construction-derived categories

| Provisional identifier | Claim or permission |
| --- | --- |
| `manual-member-construction` | Suppress generated member construction because another operation establishes the member lifetime |
| `manual-member-destruction` | Suppress generated member destruction because another operation ends or transfers the member lifetime |
| `construction-path-complete` | Treat a member as constructed on every applicable normal path despite incomplete proof |
| `destruction-path-complete` | Treat a member as destroyed or dispositioned on every applicable path despite incomplete proof |
| `construction-at-most-once` | Trust that apparently overlapping paths cannot construct one member lifetime twice |
| `destruction-at-most-once` | Trust that apparently overlapping paths cannot end one member lifetime twice |
| `partial-instance-access` | Permit bounded access to a current instance that is not fully constructed |
| `partial-instance-escape` | Permit an incomplete current instance or access path to escape or become externally observable |
| `lifetime-escape` | Permit an access path to cross a boundary whose lifetime cannot be proved sufficiently bounded |
| `replacement-alias` | Permit replacement despite unresolved possible aliasing between the destination and a right-hand operand |
| `terminal-reconstruction` | Permit reconstruction of a member during enclosing destruction |

Invocation and result work adds future proof categories without establishing
their final identifiers:

- completing every result slot on every normal path;
- constructing each result slot at most once per lifetime;
- preventing use after move or `last`;
- preventing a returned reference from escaping a temporary;
- consuming each source-result and destination slot at most once; and
- preserving diagnostic provenance for the contract or analysis that rejected a
  call.

These names identify future-work concerns. They are not accepted keywords.

`unsafe ???` already supplies the unsafe acknowledgment for delayed
construction. Calling `+++` later does not need a second unsafe category merely
because initialization was bypassed.

An unsafe assertion may override incomplete proof or assert the result of an
opaque operation. It cannot make a known-ended lifetime live again.

## Flow-derived proof pressure

Core flow control adds cross-cutting proof pressure without accepting any
override syntax:

- **Optional presence and dereference.** Access through conditionally live
  storage, including a postfix optional dereference, requires proof that a live
  value exists on that path; the proof may come from a preceding presence test,
  construction, earlier control flow, or a recognized presence contract, and need
  not be immediately adjacent. A `&&` proof is only a proof when the right operand
  is unambiguously exactly `Boolean` and the node is therefore the protected
  short-circuit operation.
- **Reachability and accidental nontermination.** Obvious unreachable code and an
  obviously non-terminating loop, such as one whose empty body never changes its
  condition, are diagnosable without a complete termination proof.
- **Loop-sensitive result completion.** Result and instance completeness must
  hold on every normal path a loop, branch, or transfer produces.
- **Branch-dependent construction.** A conditional expression's arms may
  construct or select differently and must converge to one statically usable
  result shape per complete operation.
- **Obsolete proof assertions.** A smarter compiler may make a previously required
  proof override redundant. Whether that redundancy is advisory or an error
  follows the selected static-analysis contract; see
  [Contract-dependent redundant controls](#contract-dependent-redundant-controls).

These are future analysis concerns. A semantic assertion may override incomplete
proof, while lint suppression may not change semantics; the two mechanisms remain
distinct as described below.

## Partial access and publication are distinct

A helper may use an incomplete current instance without retaining it:

```zax
+++ final : ()() = {
  _.first.+++()
  initializeRemaining(_)
}
```

Publishing the same instance is stronger:

```zax
+++ final : ()() = {
  registerGlobally(_)
}
```

The second call may retain an access path, notify subscribers, or reenter through
another path. A future permission for bounded access must not imply permission
for escape.

## Language contracts and compiler analysis

Future source may select a language-contract version whose required analysis
defines portable validity. Selection should be optional for small snippets and
may come from:

1. source;
2. project configuration;
3. build invocation or toolchain configuration; or
4. the compiler's stable default.

Omitting selection means accepting the active environment's contract, not
claiming compatibility with every historical and future version.

Analysis identifiers need visible provenance:

- language-contract identifiers;
- compiler or vendor experimental identifiers; and
- community identifiers implemented by multiple tools but not standardized.

An unknown standard identifier under a supported contract is an error. An
unknown lint extension may be ignored or reported according to policy. An
unknown unsafe semantic extension may be accepted only when the compiler
independently proves the code valid without it; otherwise the compiler reports
that the required extension is unsupported.

## Contract-dependent redundant controls

The mental model for any analysis the selected contract mandates is
prove-or-narrowly-assert:

> For every property mandatory under the selected language contract, Zax attempts
> to prove the operation valid. If proof cannot be established, the operation is
> rejected unless a narrow semantic assertion explicitly accepts responsibility
> for an unproved but valid boundary. An assertion cannot make a known violation
> valid.

Redundant-control validity is not absolute; it follows the selected contract:

| Situation | Severity of the now-redundant control |
| --- | --- |
| The contract does not require proof of that case | Implementation improvement may make the assertion redundant, but it is at most advisory |
| The selected contract requires that case to be recognized as safe | The redundant assertion may be a hard error |
| A compiler demands an assertion for a contract-mandated passing case | That compiler is nonconforming |
| Source adopts a later contract that newly mandates the proof | Source may need to remove a control that was valid under the earlier contract |

Source validity therefore follows the selected contract, not merely the executable
compiler's current cleverness. Compiler experimentation does not silently change
portable source validity, and a newly clever compiler does not by itself break
source whose contract did not change.

This rule is general. It applies to every static analysis, not only conditional
live-storage presence or one lifecycle feature.

Concrete example shapes that must survive into the future owner:

```zax
if ?optionalValue
  use(optionalValue.) // proven; no assertion is required under any contract
```

```zax
// unsafe <construction-path-complete> _.prop2
// Advisory when the contract does not mandate this proof;
// a hard error under a contract that mandates recognizing the case.
```

## Lint suppression is a separate mechanism

Lint suppression:

- does not change language semantics;
- does not change lifecycle code generation;
- cannot make invalid code valid;
- should not require `unsafe`;
- should identify a narrow rule and source subject; and
- should permit documentation of intentional exceptions.

```zax
MyType :: type {
  bar : Foo
  foo : Bar // A naming lint may be intentionally suppressed here.
}
```

Layout-efficiency lints must permit intentional ABI layout, readable grouping,
or insignificant small-type overhead.

Future work may give unsafe controls and lint controls related surface patterns.
It must preserve their different authority and failure consequences.

## What this input does not decide

This file does not establish:

- angle-bracket syntax;
- block, statement, expression, annotation, or directive form;
- the final category names;
- which analyses are mandatory in each contract;
- language-version syntax;
- extension namespaces;
- lint severity or suppression syntax;
- runtime instrumentation; or
- compiler analysis algorithms.

## Activation and retirement

Activate this input when focused work defines unsafe boundaries, analysis
contracts, diagnostics, lints, or source-version selection.

That work must:

1. disposition every provisional category and example;
2. separate language guarantees from compiler experiments;
3. define unsupported-extension behavior;
4. give semantic controls and lint controls distinct effects;
5. move accepted behavior into safety, diagnostic, tooling, or contract owners;
   and
6. remove this file from the raw index and retire it when no unresolved material
   remains.
