# Raw input: analysis controls

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work defining unsafe controls, static-analysis contracts, diagnostics, lints, or language-version behavior |
| Applies To | Remaining future syntax and category questions for `safe<...>` claims, `unsafe` operations, runtime-check controls, and lints |
| Owns | Remaining enclosure grammar questions, deferred verification questions, analysis provenance, remaining runtime-check and lint syntax, activation pressure, and retirement criteria |
| Does Not Own | The centralized provisional category inventory ([analysis-control registry](analysis-control-registry.md)); current `safe<...>` claims, `unsafe` operations, categories, verification, redundant-claim, and lint boundaries ([safety and analysis](../../language/safety-and-analysis.md)) |
| Source / Provenance | Work items `005`, `006`, `007`, `012`, `015`, `032`, `034`, `036`, `039`, and `041`; construction/lifecycle, invocation/result, core-flow, optional proof/alias, panic-category, intent-naming, compile-time error, generic-extension, and compiler-verified claim pressure |

## Reading posture

[Zax construction, replacement, and destruction](../../language/construction-and-destruction.md)
identifies places where compiler analysis may reject a valid low-level program.
This file preserves the future source-control and portability questions without
making provisional spellings current language syntax.

Current general prove/assert/permit/reject behavior and language-contract
evolution are owned by
[Zax safety and analysis](../../language/safety-and-analysis.md). The examples
below preserve candidate categories and source forms, not a competing current
definition.

It is not required for ordinary construction or qualifier use.

Provisional names, unnamed registry obligations, live occurrences, and legacy
panic identifiers are centralized in the
[raw analysis-control registry](analysis-control-registry.md). This file retains
the reasoning and examples that give those entries meaning; it is not a second
registry.

## Semantic controls are not a broad mode

The requirements that every `safe<...>` claim and `unsafe` operation be narrow,
named, and documented are current design in
[safety and analysis](../../language/safety-and-analysis.md#markers-are-narrow).

## Intent acknowledgement is separate

[Zax intent acknowledgements](../../language/intent-acknowledgements.md) now
owns defined but suspicious source such as deliberate terminal-source reuse.
Intent confirms an interpretation the compiler already understands. It does not
accept proof responsibility and cannot make an invalid operation valid.

This raw input retains only remaining questions about claims and `unsafe`
operations that change what the compiler may trust about an unproved lifetime,
alias, construction, or other required property. Lint suppression remains
separate from both.

The maintainer intends a later collective review of category names. The
partial categories `grant-redirected-selection`, `injected-partial-exposure`,
`uninvoked-hook-role`, and `application-closed-storage` entered the current
intent registry on that understanding. They avoid a `partial-` prefix because
`partial-enum-selection` and `partial-variant-selection` already use "partial"
to mean incomplete.

## Permission regions

Current `safe<...>` claims, `unsafe` operations, the test that chooses between
them, and the current category list are owned by
[safety and analysis](../../language/safety-and-analysis.md#vouching-for-safety-safe).
This section keeps only the remaining grammar questions.

The enclosure is contiguous and non-scoping. Declarations retain their
surrounding scope:

```zax
safe<lifetime-escape>{
  x : Integer
  myCaptureBeyondLifetime(myType.view())
}

x = 5
```

One enclosure may name several exact categories without becoming a broad
mode:

```zax
safe<optional-presence, lifetime-escape, replacement-alias>{
  reset optionalValue
}
```

Future grammar must validate the contiguous opener, required whitespace after
each comma separator, documentation attachment, nesting, mixing `safe` and
`unsafe` categories, and source reflection.

## Lifecycle-state assertions

An opaque operation may establish a member state that ordinary analysis cannot
observe. Current construction design has one such assertion, made at the
establishing operation:

```zax
+++ final : ()() = {
  unsafe<opaque-construction>{ opaqueInitialize(_.prop2) }
}
```

The enclosure both takes `prop2` out of automatic construction and asserts that
the enclosed operation constructs it. Its category name is accepted for now and
may be revised. The syntax by which the enclosure names the places it
establishes is open: a call receiving two empty places must not silently
establish both.

For construction, the conservative first-construction rule has no override:
when the compiler cannot tell whether a place is constructed, the code must be
restructured.

## Construction-derived categories

The construction and destruction categories are now current design, with
examples, in
[safety and analysis](../../language/safety-and-analysis.md#current-categories)
and [construction and destruction](../../language/construction-and-destruction.md).
`opaque-construction` and `manual-member-destruction` use `unsafe<...>`; the
others use `safe<...>`.

`manual-member-construction`, `construction-path-complete`, and
`construction-at-most-once` are superseded. Every explicit construction step
suppresses automatic construction; opaque establishment
is `opaque-construction`; and unknown construction state is an error to
restructure rather than a fact to assert.

Invocation and result work still has proof concerns without category names:

- completing every result slot on every normal path;
- constructing each result slot at most once per lifetime; and
- preserving diagnostic provenance for the contract or analysis that rejected a
  call.

Operations on moved-from or finished values use `safe<not-finished>`, and a
reference escaping from an opaque origin uses `safe<lifetime-escape>`. Each
source result and destination slot being consumed at most once needs no
category: routing is written in the call, so the compiler always checks it.

Ordinary defined reuse after accepted `last` is an intent error and uses
`intent<terminal-source-reuse>{...}`. A safety claim is needed only when a
required valid lifetime, alias, or operation property cannot be proved.

A claim may cover incomplete proof, and an `unsafe<...>` category may assert
the result of an opaque operation. Neither can make a known-ended lifetime live
again.

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
- **Selection entry paths.** A runtime case body may be reached by a successful
  test, by a positional `default`, by `continue` after re-testing, or directly
  by `goto`; only facts valid on every incoming path are available in the body.
  Direct entry cannot inherit optional presence, pointer validity,
  initialization, or range facts from a bypassed test.
- **Selection reachability and cycles.** Ordered tests may prove a later clause
  unreachable, while backward `continue` or `goto` may create an intentional
  cycle. Future analysis must distinguish language-defined proof from assumptions
  about user-defined operator purity or algebra. A reachable incoming transfer
  makes an ordinarily shadowed body reachable; a self-edge inside otherwise dead
  source does not bootstrap reachability.
- **Selection segments.** Transfer-only cases are skipped by ordered searches.
  Positional defaults delimit test segments, and `continue` may start a later
  search. Proof, completeness, and reachability are evaluated for each reachable
  search entry rather than only initial switch entry.
- **Enum coverage.** Declared-member coverage and complete reachable-backing
  coverage are distinct. `default` may satisfy the latter without silently
  satisfying omitted declared members. Duplicate-valued aliases represent one
  distinct value, and arbitrary predicates contribute only when their covered
  domain is provable. Only reachable tests contribute member coverage: `goto`
  may make a body reachable while bypassing its test, and a case after a
  positional default does not count for a search that cannot reach it.
- **Obsolete claims.** A smarter compiler may be able to prove a previously
  required claim. What that compiler may report, and when the claim becomes an
  error, follows the selected contract; see
  [contract-dependent redundant controls](#contract-dependent-redundant-controls).

These are future analysis concerns. A `safe<...>` claim may cover incomplete
proof, while lint suppression may not change semantics; the two mechanisms
remain distinct as described below.

### Optional proof and invalidation pressure

[Zax optional values](../../language/optional-values.md) requires static proof of
the exact live boxed lifetime before postfix access. Proof may come from a
presence test, construction, earlier flow, or another recognized contract.

Reset, packet construction, complete-wrapper replacement, terminal transfer, or
another lifetime-ending operation invalidates earlier payload proof and aliases.
A `safe<...>` claim may accept responsibility when the compiler cannot prove
valid source. The claim adds no required runtime handle or check. If its presence,
alias, or lifetime claim is false, behavior is undefined; debug instrumentation
may detect the violation and panic without making that check a language
guarantee. It cannot legalize unconditional known absence, a known-ended
lifetime, direct boxed destruction, readonly mutation, immutable mutation, or
replacement of a final place.

Exact same-object source is not a universal error:

```zax
value = value
value foos value
```

A protected lifecycle operation may define alias-safe behavior, while an
arbitrary user phrase may intentionally accept the same object twice. Future
analysis should use operation contracts or lint policy rather than a blanket
semantic rejection.

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
unknown `safe<x-...>` or `unsafe<x-...>` extension category may be accepted only when the compiler
independently proves the code valid without it; otherwise the compiler reports
that the required extension is unsupported.

## Contract-dependent redundant controls

How a claim becomes redundant under the selected contract, what a cleverer
compiler may report, and when a claim is an error are current design in
[safety and analysis](../../language/safety-and-analysis.md#language-contracts-and-compiler-analysis).

### Deferred verification questions

Asking a compiler to verify a claim (`[<verify=...>]`), the reporting levels
(`[<safe-verification=...>]`), and their behavior inside `wellformed` are
current design in
[safety and analysis](../../language/safety-and-analysis.md#asking-a-compiler-to-verify-a-claim).
These questions remain:

- **Declaration-level and range forms of `[<verify>]`.** Today the directive
  attaches only to one `safe<...>` enclosure, which keeps it next to the
  claim it verifies:

  ```zax
  // possible future form, not current syntax
  [<verify=x-acme>]
  processBatch final : ()(inbox : MyMessage[0..] mutable writable loan &) = {
    // every safe<...> claim in this body would be verified by x-acme
  }
  ```

  Activation: real code with many verified claims in one function. Adding the
  form later does not break source.
- **Separate control of individual findings or verifiers** in
  `[<safe-verification>]`, such as silencing only "can be verified" findings
  or only one compiler's findings. Adding it later does not break source.
- **Source reflection of `[<verify>]`**, and whether it is enough for tooling
  to list verified claims.
- **Who maintains the registry** of `x-` extension specification names.
- **Official verifier names.** A plain, non-`x-` value could later name a
  language contract, asking "would this claim be provable under contract N"
  before upgrading. No pressure yet.
- **A buffer-clear operation that cannot be optimized away.** A destructor
  that zeroes a key may have its writes removed as dead stores, because
  nothing reads them afterwards. A test that inspects the storage through a
  `* unsafe` probe can keep the writes alive and pass, while production code
  loses them:

  ```zax
  --- final : ()() = {
    _.key.fill(0)          // may be removed: nothing reads key afterwards
  }
  ```

  The expected direction is an intrinsic whose writes always happen, rather
  than reduced optimization. Activation: security-sensitive library or
  runtime design.

The rule that a diagnostic names the exact `safe<...>` category or `unsafe`
operation that would apply, and says so when no marker can help, is current
design in
[safety and analysis](../../language/safety-and-analysis.md#diagnostics).

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

### Arithmetic-overflow analysis

Current integer arithmetic and the two unchecked build contracts are owned by
[the integer operator catalog](../../language/integer-operator-catalog.md#unchecked-build-contracts).
This section retains their future option syntax, placement, source-local
controls, and analysis-contract integration.

Bare fixed-width arithmetic has stable exact-result-or-panic semantics,
independent from debug or release optimization mode. A future analysis policy
may:

- diagnose definitely overflowing constant operations;
- report statically evident suspicious ranges;
- request stronger warnings when runtime safety is not proved; or
- elevate selected warnings according to project policy.

An explicit compiler/build policy may instead remove required runtime
representability checks by changing the selected contract:

> Every required arithmetic result is representable by its result type.

When the contract holds, the result is the exact mathematical value. When it is
violated, Zax makes no behavioral promise; the operation does not become
implicitly wrapping.

Future teaching must explain the practical choice before the contract details:

- keeping checks catches range failures through panic;
- disabling checks removes those panic branches and permits stronger loop,
  vectorization, and no-overflow optimization;
- the unchecked contract is useful when trusted validation or domain invariants
  already prove representability; and
- violating it has undefined consequences that tests may not expose.

The option is never defined by debug versus release mode. Project/build metadata
must make the chosen contract reproducible and auditable.

### General runtime-check contract pressure

Arithmetic overflow is one instance of a broader possible mechanism. Future
work should evaluate whether other cheaply checkable runtime failures use the
same contract shape:

```text
default contract -> perform the required check and panic on violation
unchecked contract -> omit the check; violation has undefined consequences
```

Candidate domains include bounds, nonzero divisors, explicit panic-on-absence
operations, allocation failure, and other checks whose runtime condition is
well-defined. Each domain still owns its safe default and exact failure.

### Panic registry pressure from intrinsic arrays

Every defined runtime panic condition needs a stable registry entry rather than
an ad hoc feature-local switch. Language-owned and programmer/library-owned
categories need collision-free provenance. Tooling and reproducible build
metadata must expose the effective enabled/disabled state of each category.
The centralized provisional inventory is the
[raw analysis-control registry](analysis-control-registry.md#panic-category-inventory).

Source-level panic-control attachment is now current design in
[safety and analysis](../../language/safety-and-analysis.md#controlling-panic-checks):
attached and range directives that name one category each, no wildcard disable,
no crossing of `import`, and a build-configured starting state that source
refines. Remaining future work is the category names, the build-option syntax
that sets a module's starting state, and how tooling reports the effective
state.

Current [arrays and slices](../../language/arrays-and-slices.md) require
provisional registry entries for:

- element index outside bounds;
- splice endpoint or order outside bounds;
- requested logical length outside its admitted range;
- initializer cardinality mismatch;
- invalidated slice use;
- foreign slice origin supplied to mutation;
- iterable exact-count promise mismatch;
- backing extent overflow; and
- required backing allocation failure.

Exact identifiers remain future work. Each condition is independently
selectable. Disabling one permits omission of its required runtime check and
makes violation undefined, but it never suppresses a statically mandated error
or the check required by an optional/reporting operation.

Slice invalidation additionally pressures the registry to expose check cost and
provenance. Current storage-strategy direction may let a slice retain a
stability kind/token and observed version while the array and provider
cooperate to advance versions after mapping/chunk changes. A statically proved
use may eliminate the check; a mismatch enters the existing invalidated-slice
category.

Exact token/version representation remains future array-storage work rather
than registry authority.

Categories are independently selectable rather than one all-or-nothing panic
mode. For allocation, ordinary `@` checks request failure and enters panic.
Disabling only that category may omit the check under a success guarantee; an
actual failure then has undefined consequences. `@!` remains checked because its
defined result must distinguish success from pointer vacancy.

Static proof obligations remain distinct. Optional presence, reference lifetime,
and alias validity may be impossible or prohibitively expensive to track at
runtime without changing representation. A narrow `safe<...>` claim may accept
responsibility without requiring a check; debug tooling may add best-effort
instrumentation.

Future contract work must decide which domains share syntax, how several
contracts compose, whether source/build configuration may strengthen or weaken
them, and how tooling exposes the accepted cost and undefined-behavior boundary.

The policy is independent from optimization level, shifts proof responsibility
to the programmer, is recorded in build metadata, and does not affect optional,
wrapping, saturating, reporting, or narrowing forms. Compile-time-proven
violation remains a diagnostic.

Required representability and nonzero-divisor contracts are independent. Future
compiler-directive work must decide whether each may be selected at:

- project/build level;
- module or namespace level;
- source-file level;
- lexical region;
- declaration;
- expression/operation; or
- a deliberately smaller set.

It must also decide inheritance, narrower overrides, project-level minimum
safety that forbids weakening, forced checks, whether a build may force
unchecked behavior over explicit source checks, imported/generated source,
conflict diagnostics, language-contract versions, and reproducible metadata.

The current recommendation is a project/build default with narrower explicit
source choices and an optional minimum-safety policy. A project may force checks
without changing contract-respecting results; it should not silently force
unchecked behavior over source that explicitly requests checks.

Future work must decide:

- the policy's exact name;
- exact supported selection levels and source syntax;
- how tooling reports code relying on the contract;
- how contract versions preserve source and build reproducibility.

Ordinary source does not require a programmer-supplied proof for every
calculation. A lint cannot change arithmetic semantics. Checks may always be
optimized away when the compiler proves the selected operation is preserved.

## Structural transform non-overlap pressure

[Zax structural transformation](../../language/structural-shapes-and-compatibility.md#overlap-is-handled-statically)
uses conservative intermediate capture when source and destination may overlap.
It emits no runtime range check merely to choose a cheaper transform.

A transform whose source cannot be copied, moved, or otherwise captured now
uses a `safe<disjoint-storage>` claim, which is current design in the
structural owner. Nothing further remains here.

## Nothing-instance diagnostics and controls

[Zax Nothing instances](../../language/nothing-instances.md) establishes the
semantic boundaries. Future analysis-control work must define exact,
independently selectable categories for:

- proved dereference or arithmetic from a vacant pointer;
- access that requests a trapping Nothing policy on a target with or without
  suitable hardware support;
- access to a member without a readable prepared representation;
- a proved write to compiler-provided Nothing backing;
- a runtime pointer-arithmetic check when ordinary target validity is required;
  and
- a proved call of an unavailable function value; and
- the check at entry to a function that is not `once` when it is called with a
  Nothing receiver on a target without a hardware trap.

Compiler analysis remains intentionally incomplete. A selected category may
control a required check where the language defines one; it must not convert a
static error into valid source or imply that every unchecked dereference checks
vacancy. Debug instrumentation may catch additional writes or invalid access,
but it is tooling rather than a language guarantee.

The default and trapping policies do not authorize universal software vacancy
checks. Hardware write protection is used when available; a requested read trap
also relies on target capability. On a target without the needed trap, an access
that relies on it has undefined behavior. Future controls and diagnostics must
expose this capability difference without pretending a debugger can stop every
operation at the faulting instruction.

### Possible vacancy

A possibly vacant access through a tracked pointer now needs proof or a
`safe<pointer-presence>` claim at the operation that dereferences it, as
[Nothing instances](../../language/nothing-instances.md#possible-vacancy-needs-a-presence-claim)
teaches. The earlier provisional `possible-nothing-write` and
`possible-nothing-read-trap` permissions are superseded by that claim.

Remaining future work: how far the rule's source tracking follows pointers
stored in members (part of the pointer-tracking follow-up in
[usage and provenance tracking](usage-and-provenance-tracking.md)), and
narrow responsibility for disabling an applicable check or asserting ordinary
target validity. A false claim has undefined consequences. No marker can
legalize a proved managed-pointer leak, and no control may collapse a vacant
pointer, absent optional, or unavailable function value into one universal
state.

## Compile-time error reporting and assertions

Generic and compile-time code needs a way to reject an unsupported case with
the author's own message. [Zax generics](../../language/generics.md) already
uses `Compilation.error("message")` in its examples for this, noting that the
exact spelling is not settled. That spelling must be reviewed here, and the
generics examples revised if it changes.

```zax
// Spelling under review.
formatTime final : (result : String)(
  format [<constant>] : String readonly,
  value : TimePoint
) = {
  if compile !looksLikeTimeFormat(format) {
    Compilation.error("formatTime: invalid time format")
  }
}
```

The sketch to settle:

- `Compilation.error("message")` reports an error written by the author, at the
  compile point, with the compile-time call chain. That evaluation stops and
  the build fails. If accepted, it joins the `Compilation` table in
  [compile-time code](../../language/compile-time-code.md#compilation-information).
- A separate `Compilation.panic` may be unnecessary: a panic during
  compile-time execution is already a compiler diagnostic.
  `Compilation.error` earns its place by carrying the author's message.
- A `Compilation.warning` needs care. `wellformed{...}` treats an unclassified
  language-defined warning as an error, so library warnings must not turn
  callers' `wellformed` checks into errors.
- `assert(condition, "message")` panics on failure and is governed by the
  existing panic-check controls, so it can be turned off for the shipped
  program.
- `compile assert(...)`, or `assert` inside `compile{...}`, runs the same check
  while compiling, where failure is a diagnostic. Compile-time assertions stay
  on; turning them off would only remove a guarantee.

## Extension lint candidate

Any module may add a specialization of a generic, or a helper a generic body
can find, even when it owns neither the generic nor the argument type. Every
use in the program must then agree, or the program is rejected (see
[extending generics](../../language/generics.md#extending-generics-from-other-modules)).

An optional, informational lint could flag such an extension, noting that
combining the module with other code can produce that disagreement error:

```zax
// Module Tweaks: owns neither MyBits$ nor ThirdPartyType
MyBits$(UseType : type) requires UseType == ThirdPartyType :: type { packed : Byte[2] }
// possible lint: extension of a generic and a type owned elsewhere
```

Undecided: whether to add it, and its name. It must never restrict
extension.

## `active` layout query in a compile-time decision

Every layout query names its environment (see
[layout queries name their environment](../../language/compile-time-code.md#layout-queries-name-their-environment)).
`active` means "the world running this code", which while compiling is the
build machine. Using it in a compile-time decision is legal, because the choice
is written, but a decision about the shipped program usually wants `target`:

```zax
if compile active size of MyEntry > 16 {   // legal; possible lint: decisions usually want `target`
  useLargeEntries()
}

MyBox$(UseType : type) requires active size of UseType <= 16 :: type { }   // same lint
```

An optional, informational lint could flag `active` layout queries and
`Compilation.active` layout descriptors evaluated in a constraint, `requires`
clause, compile-time parameter default, `if compile` or `switch compile`
condition, or a type-returning call. It must never make the source an error:
the maintainer accepted explicit `active` as a valid choice.

Undecided: whether to add it, its name, and whether it also follows a host
value computed with `host` or `active` into a later decision.

## Proof rules for each category

Every `safe<...>` category needs specification-level rules saying when the
language contract proves a site safe, so the claim is redundant, and when it
proves the claim false, so the code is an error. Without them, compilers
cannot agree on which claims are required, and the "proved by the contract"
and "proved only beyond the contract" cases in
[safety and analysis](../../language/safety-and-analysis.md#what-a-compiler-does-at-a-claim)
have no portable boundary.

The rules are algorithm-oriented and likely belong in their own documents
rather than in the teaching owners. Several categories probably share one
tracking mechanism, so the eventual structure may be one document per
mechanism, each listing the specific check for every category that uses it:

- **lifetime and origin tracking:** `lifetime-escape`, `unique-raw-borrow`,
  `exact-origin`, `not-finished`, `pointer-presence`, and tracked pointers;
- **path-merge state tracking**, as construction already uses for empty, live,
  and unknown member states: `destruction-path-complete`,
  `destruction-at-most-once`, `terminal-reconstruction`,
  `partial-instance-access`, and `partial-instance-escape`;
- **alias and write tracking:** `replacement-alias`, `disjoint-storage`, and
  `no-mutation-while-held`;
- **value and representation facts:** `distinct-element`,
  `valid-representation`, `opaque-type`, and `optional-presence`.

The kind of rule meant, for one category:

```zax
// destruction-path-complete: the contract proves the site when every path
// from destructor entry to exit destroys the member exactly once, with path
// conditions compared only in recognized shapes (if/else, else if, switch arms)
--- final : ()() = {
  if _.pooled {
    _.buffer.---()
  } else {
    _.buffer.---()        // proved: if/else covers both paths
  }
}
```

Each rule must also say which check shapes count as proof, matching the
current rule that a check counts only if the compiler can follow it, and must
stay stable across contract versions except where a later contract
deliberately recognizes more.

Activation: specification work on any category, or the first time two
compilers disagree about whether a claim is required. Constraint on current
work: teaching owners describe outcomes and examples, not the algorithms.

## What this input does not decide

This file does not establish:

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
