# 041: Compiler-verified unsafe

| Field | Value |
| --- | --- |
| Status | Historical / non-normative / audit-only; completed 2026-11-26 and archived |
| Work Item | `041` |
| Created | 2026-11-24 |
| Owns | The bounded review of a directive that names one or more compilers which must prove an `unsafe` site safe, while the site keeps its `unsafe` for every other compiler |
| Does Not Own | Which facts any compiler or the specification actually proves (specific tracking or analysis rules); threading and exclusive access; general language-contract selection beyond where it meets the directive; current safety, directive, and array behavior, which stay with their owners until a promotion is authorized; the decision itself, which belongs to the language maintainer |

## Non-authority notice

This file is a collaborative working record. Existing documentation, candidate
syntax, examples, and later aligned findings remain non-authoritative until a
separately discussed, aligned, and explicitly authorized promotion incorporates
them into their lasting owners.

## Fixed initiating input

This section records the aligned information known when work item `041` was
created. It is intentionally incomplete and must not be rewritten as work
develops.

### Initiating concern

Zax requires `unsafe` wherever the language's listed proofs do not cover a
site, even when a particular compiler can prove that site safe, so that every
compiler accepts the same programs. A directive could let source name one or
more compilers that must actually prove the site safe, while the `unsafe`
stays in place for every other compiler. Current teaching already shows such a
directive, written `[<safe=…>]`, as if it exists and marked as still being
designed (arrays and slices, "What the compiler can prove"). What should the
directive be, what does it claim, and how does it behave?

### Starting input

- `project/raw/analysis-controls.md`, "Compiler-verified unsafe tags": the
  intended behavior (compilers that don't implement the named verifier ignore
  the tag and accept the `unsafe`; compilers that do must prove the site and
  reject it when the proof fails; reviewers can tell verified sites from
  unchecked ones; a later contract that adopts the proof makes both the tag
  and the `unsafe` redundant), multiple verifiers, the naming candidates and
  their polarity concerns (`safe=`, `proved=`, `proved-safe=`,
  `verified-safe=`), and its open questions (exact spelling, sharing the
  `[<...>]` form with `[<panic=...>]`, unknown verifiers silently ignored or
  linted, a strict build option requiring listed verifiers, interaction with
  compiler agreement).
- `project/raw/compiler-agreement-principle.md`: acceptance depends only on
  rules the specification states; a stronger compiler may warn but never
  accept or reject differently. The maintainer's recorded concern and the tag
  as its answer.
- The maintainer's direction from work item 040: use the directive in
  teaching as if it exists, because it will, subject to change when this work
  completes. Specific tracking rules are not ready, because the language should
  be settled first, threading especially.

### Starting boundaries

- What any compiler proves, and any specific tracking or analysis rule, is
  out of scope. This item decides the directive, not the analyses it names.
- Threading and exclusive access are out of scope; they have their own
  capture in `project/raw/safety.md`.
- The compiler-agreement direction is the starting position. A conflict with
  it is recorded as a finding for discussion, not resolved by silently
  changing it.
- `intent<...>` acknowledgements are a separate mechanism: they confirm
  defined but suspicious behavior and supply no unsafe fact.

### Initial stopping guidance

Per "The first deliverable is a set of decisions" in
[documentation architecture](../documentation.md#the-first-deliverable-is-a-set-of-decisions):
first identify the issues this directive is likely to raise, and present a
workable set of decisions the maintainer can act on, each with concrete Zax
examples, the realistic options, what each costs, and a recommendation.
Supporting evidence and tensions sit underneath the decisions. Do not design
every combination in full before the direction is aligned.

Creating this work item does not authorize analysis. A later assignment
begins it.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs focused reading,
  authority, promotion, deferral, closure, and the writing rules.
- [Analysis controls](../raw/analysis-controls.md): "Language contracts and
  compiler analysis", "Contract-dependent redundant controls",
  "Compiler-verified unsafe tags", and "Diagnostics must name the required
  `unsafe` form" - the existing design input for the directive.
- [What compilers must agree on](../raw/compiler-agreement-principle.md) - the
  principle the directive must fit.
- [Zax safety and analysis](../../language/safety-and-analysis.md): "Safety is
  a selected contract", "Four analysis outcomes", and "Language contracts and
  compiler analysis" - the current owner the directive extends.
- [Zax compiler directives](../../language/compiler-directives.md): directive
  families, grammar, `x-` names, and attachment - the form the directive takes.
- [Zax arrays and slices](../../language/arrays-and-slices.md), "What the
  compiler can prove" - the one current page that already shows the
  directive.

### Consequence-driven

Read only the smallest relevant sections when the concern reaches them:

- [Allow with tracking rather than forbid](../raw/allow-with-tracking-principle.md),
  when the directive's role in the wider safety approach is reached.
- [Analysis-control registry](../raw/analysis-control-registry.md), when
  `unsafe` category names are reached.
- [Zax intent acknowledgements](../../language/intent-acknowledgements.md),
  when the directive must be distinguished from `intent<...>`.
- [Zax source structure](../../language/source-structure.md), "Compiler-directive
  enclosure", when placement or spelling is reached.
- [Zax pointers, allocation, and arenas](../../language/pointers-and-arenas.md),
  when the raw-pointer-from-reference example is reached.
- [Usage and provenance tracking](../raw/usage-and-provenance-tracking.md),
  only to keep its tracking rules outside this item's boundary.

### Audit-only

- Archived numbered work, only when a provenance question cannot be answered
  from current owners or raw input. The compiler-agreement principle came from
  `project/archive/work/039-pointer-stances.md`; the directive's current use
  in teaching came from `project/archive/work/040-element-and-contained-stances.md`.

## Working record

### How to read this record

Findings marked **Aligned** were discussed and aligned with the maintainer.
They remain non-authoritative until an authorized promotion. Anything not
marked aligned is labeled as tentative, open, deferred, or captured.

**Current state:** every decision is aligned, including the questions raised
by the first dry run. The second dry run passed (see
[dispositions](#dispositions-and-promotion-dry-run)). Promotion needs its own
authorization.

The record is organized as:

1. [Scope as it has developed](#scope-as-it-has-developed)
2. [Aligned model](#aligned-model): the programmer view
3. [Decisions](#decisions): each decision and its aligned outcome
4. [Repairs to existing material](#repairs-to-existing-material)
5. [Assumptions 041 relies on](#assumptions-041-relies-on)
6. Superseded candidates, evidence, risks, deferred and captured findings,
   promotion notes, and promotion scope

All category names in examples are illustrative unless a table marks them
aligned.

**Note on wording.** Much of this record's wording is not suitable for
teaching; the maintainer confirmed that the concepts are correct but the
language is not teachable. Promotion re-authors everything for programmers, as
the [documentation architecture](../documentation.md#promotion-re-authors-for-a-different-reader)
already requires. One example the maintainer gave: "the findings still reach
you" makes a finding the actor; promoted text says something like "the
programmer still learns the outcome".

### Scope as it has developed

The initiating concern was one directive. Following its consequences, and at
the maintainer's direction, `041` also covers:

- **The `safe<...>` / `unsafe` split.** It resolved the directive's naming
  polarity and attachment. Promotion updates every affected occurrence.
- **Repair of unclear or misclassified safety categories and operator forms
  that `041` encounters.** The split is only coherent if every form falls on
  the right side of it.

`041` does **not** design pointer tracking, path-based immutability, or a
"forever" immutability form. It treats them as true in outline
([assumptions](#assumptions-041-relies-on)), records how safety depends on
them, and hands them to raw input and follow-up work items.

### Aligned model

#### Two markers: `safe<...>` and `unsafe`

**Aligned.**

```zax
// illustrative category name
safe<distinct-element>{                       // "this is safe; here is the fact"
  dispatch(inbox[2 * i] as last)
  dispatch(inbox[2 * i + 1] as last)
}

header : PacketHeader * = myBuffer unsafe cast PacketHeader *   // an unchecked operation
```

- `safe<category>{...}` asserts a fact the compiler could not establish, about
  state the language tracks. The programmer is vouching that the enclosed code
  is safe. A false claim has undefined consequences.
- `unsafe` remains for unchecked operations and for claims about things the
  language does not track: `unsafe cast`, `unsafe cast as`, `unsafe vacate`,
  `unsafe pliable`, `unsafe as coercive`, `unsafe from`, `union unsafe`, the
  assumed `* unsafe`, and `unsafe<category>{...}` enclosures for the few
  categories that are not tracked facts, such as `opaque-construction`.
- No third keyword is added.
- The word `safe<` stays despite the false-claim concern: it is the
  programmer asserting safety, which is what the word says.
- A search for `safe<` also finds `unsafe<`. That is a feature: the search
  returns every point of programmer responsibility, and false matches are
  easy to filter.
- Only `safe<...>` sites can be verified by a named compiler.

When a language contract requires the proof, the `safe<...>` marker is
redundant and the programmer deletes it, leaving the plain code underneath.

#### Asking a compiler to verify a `safe<...>` site

**Aligned.**

```zax
[<verify=x-acme>]
safe<distinct-element>{
  dispatch(inbox[2 * i + 1] as last)
}

[<verify=x-acme>][<verify=x-other>]     // one directive per compiler
safe<distinct-element>{
  dispatch(inbox[j] as last)
}
```

- The directive is `[<verify=...>]`, on the line directly above the
  `safe<...>` enclosure.
- Several compilers are named by repeating the directive, which leaves room
  for compiler-specific options later.
- An enclosure that names several categories is verified for all of them.
- Verifier names are always `x-` names, chosen so they never collide: a
  registered extension specification (`x-extension-spec-name`, in the style of
  an IANA registry) or a specific compiler (`x-specific-compiler-name`).
- `[<verify=x-acme>]` concerns only the `x-acme` compiler. Other compilers
  ignore it. A misspelled name is not caught; that is accepted.

#### What a compiler does at a `safe<...>` site

**Aligned** (the maintainer's three rules):

| Situation at a `safe<...>` site | Result |
| --- | --- |
| The specification's rules prove the site safe | Error: remove the marker |
| Any compiler proves, with certainty, that the claim is false, even using analysis beyond the specification | Error: fix the code. This runs at every noise level, including `off` |
| A compiler proves the site safe only by going beyond the specification, and there is no `[<verify>]` for it | Finding "add `[<verify=x-acme>]`", reported at the `safe-verification` level |
| `[<verify=x-acme>]` is present and `x-acme` still proves the site | Nothing reported |
| `[<verify=x-acme>]` is present and `x-acme` can no longer prove the site, without proving it false | Finding "remove the tag or fix the code", reported at the `safe-verification` level |

- "Proves false" must be a certainty, not a suspicion.
- The `safe-verification` level defaults to `error` for your own code and
  `lint` for imported code. The maintainer wants a project's own code to
  compile cleanly, so that a library does not produce lint findings for the
  people who use it.
- A stale `[<verify>]` tag inside imported code therefore produces lint in the
  importer's build and an error in the library's own build.
- A proven-false claim inside imported code fails the importer's build even at
  `off`, because the code has undefined behavior. The importer's remedies are
  to avoid that code or to patch the library.

#### Inside `wellformed{...}`, only the specification decides

**Aligned.**

```zax
if compile wellformed{ pairwiseDispatch$(MyMessage)(inbox) } {
  pairwiseDispatch$(MyMessage)(inbox)   // body has [<verify=x-acme>] on a safe<...> site
} else {
  serialDispatch(inbox)
}
```

- The answer of `wellformed{...}` follows the specification's rules
  exclusively, so the compiler used can never change it. The same holds for
  `if compile`, `switch compile`, constraints, overload and specialization
  selection, and reflection.
- Findings that come from a cleverer compiler's analysis inside a
  `wellformed` payload, both "this claim is false" and "add the tag", are
  reported at a separate `wellformed` level, which defaults to `lint`.
- Code that is also compiled outside the payload, as `pairwiseDispatch` is
  in the selected branch above, is judged there under the ordinary rules.
  The `wellformed` level matters for code that exists only as a probe, which
  should not break a build.

#### How loudly findings are reported: `[<safe-verification=...>]`

**Aligned.**

```zax
C :: import Module.C {
  [<safe-verification=off>]                    // for this whole instance of C
}

D :: import Module.D {
  [<safe-verification=error, wellformed=error>]
}

E :: import Module.E                           // nothing written: lint, and lint inside wellformed
```

- Levels are `off`, `lint`, and `error`, with a separate `wellformed=` level.
- **Inside an import body**, the setting is unattached and applies to the
  whole imported instance. Imports in a module are written once per instance;
  every other reference to that instance goes through the defining import
  statement, which is where its setting lives. The same source imported twice
  is two instances, each with its own setting.
- **In ordinary code**, the setting can be used as a range, like panic
  controls, to tune particular regions; this is expected to be rare.
- **Who wins:** a module's own `safe-verification` directives apply when that
  module is the code being built. When it is imported, the importer's setting
  applies to the whole instance. This is the opposite of panic controls, and
  safe for the same reason it is allowed at all: the setting only changes what
  is reported, never what the program does. Promoted teaching must give that
  reason.
- **The setting stops at that instance.** B's choice for C does not reach D
  if C imports D. Re-exported and injected instances keep the setting of the
  import that created them.
- **A finding is judged where the `safe<...>` is written**, under the
  setting of the instance whose source contains it. The diagnostic names the
  instantiation that exposed it:

  ```zax
  // module MyLibrary
  pairwiseDispatch$ final : (UseType : type)()(items : UseType[0..]) = {
    [<verify=x-acme>]
    safe<distinct-element>{
      dispatch(items[2 * i + 1] as last)
    }
  }

  // module MyApp
  pairwiseDispatch$(MyMessage)(inbox)
  // x-acme can't prove the site for MyMessage:
  //   reported at MyLibrary's safe<...> site, under MyLibrary's instance setting,
  //   "while instantiating pairwiseDispatch$(MyMessage) from MyApp"
  ```

- **`off`** silences the "add the tag" and "remove the tag" findings, so the
  compiler need not search for proofs of safety. The "claim is false" check
  still runs and is still an error.
- **The project's own source** gets its starting level from a compiler or
  build option, defaulting to `error`.

#### Classifications

**Aligned.**

| Form or category | Marker | Reason |
| --- | --- | --- |
| `distinct-element`, `optional-presence`, `replacement-alias` | `safe<...>` | Distinctness, presence, and alias facts |
| `lifetime-escape` | `safe<...>` | The reference's target outlives the use, where the compiler loses track of where the instance came from through complex paths. Also covers returned references from opaque origins and the construction owner's future reference-use permission |
| `unique-raw-borrow` (absorbs `raw-from-reference`) | `safe<...>` for the uncertain case | See below. Also covers a raw pointer made from `_` |
| `valid-representation` (renamed from `union-lens-validity`) | `safe<...>` | These bits are a valid value of this type. Covers union lenses, a reference into a union lens across later writes, and unchecked floating-point patterns such as `U80` read as `X87Extended80`. A compiler tracing where the bits came from may prove it |
| `pointer-presence` (replaces `possible-nothing-write` and `possible-nothing-read-trap`) | `safe<...>` | The pointer is not vacant here. Also covers an opaque call inside a result that promises presence. See [repairs](#nothing-instances-possible-vacancy) |
| `exact-origin`: outer-cast and `via` origin (today `unsafe outer cast`, `unsafe via`) | The plain form inside `safe<...>` | A compiler may prove the origin in context, and the plain forms are already what is written when the origin is proven |
| `opaque-type`: opaque-owner recovery (today `unsafe transfer (opaque as last)`) | The plain recovery inside `safe<...>` | Same three forms as outer cast: proven by a preceding `is type` test, checked recovery that reports failure, or a claim |
| `partial-instance-access`, `partial-instance-escape` | `safe<...>` | A specification or cleverer compiler can see that nothing touches an unconstructed member |
| `destruction-path-complete` | `safe<...>` | If the compiler cannot verify that every path ended the member, it cannot insert its own destruction, so it must report an error unless the programmer vouches |
| `destruction-at-most-once` | `safe<...>` | Another `---` the compiler cannot prove is safe needs the claim |
| `terminal-reconstruction` | `safe<...>` | A lifecycle-sequence fact during destruction; also covers the construction owner's "advanced terminal code" sentence |
| `not-finished` | `safe<...>` | Use after `as last` the compiler cannot rule out, from the use-after-`last` tracking direction |
| `disjoint-storage` | `safe<...>` | A structural transform whose source and destination may overlap and whose source cannot be copied aside |
| Vendor extension categories (`safe<x-...>`) | `safe<...>` | The vendor compiler knows and enforces the fact, whether that means reporting a problem other compilers miss or confirming code the programmer wants checked. An unknown extension category may be ignored only when the compiler proves the code valid without it |
| `opaque-construction` | `unsafe<...>{...}` | Only a lower layer can vouch for it. If an ABI could later make the assertion, true on every compiler, a compiler would say "remove `unsafe`, it's safe" |
| `manual-member-destruction` | `unsafe<...>{...}` | The mirror of `opaque-construction`: unseen code ends the member, and the marker suppresses generated destruction |
| Opaque raw-pointer facts (allocation root, alignment, sole disposition) | No separate category | They are what `unsafe cast` and `unsafe cast as last` already claim |
| Single consumption of source results and destination slots | No category | Routing is written in the call, so the compiler can always check it; a violation is an error |
| `unsafe cast` to another type, or integer to pointer | `unsafe` | Unchecked reinterpretation |
| `unsafe cast as last` | `unsafe` | Brings an untracked pointer back under tracking; the maintainer does not want that tracked |
| `unsafe vacate` | `unsafe` | Breaks the owning pointer's dispose-once promise and can leak |
| `unsafe as coercive` | `unsafe` | Destination validity of arbitrary storage and the later restoration obligation are not tracked state |
| `unsafe pliable` | `unsafe` | Proving it would need knowledge of what every alias relies on |
| `unsafe from` | `unsafe` | Unchanged |
| `union unsafe` | Unchanged | A type qualifier; access uses `safe<valid-representation>` |
| A conversion from `immutable` to `mutable writable` | `unsafe` | Breaks the promise to every holder that the value does not change |
| Conversions to `immutable` from `mutable` or `mutable readonly` | Analysis, or `safe<...>` | Follows [path-based immutability](#path-based-immutability), taught as true |
| `immutable` to `mutable readonly`, same type | No marker | The new holder is told something weaker and cannot write |
| `replacement-member-successor` | `intent<reference-tracks-replacement>` | Not a safety question; see [repairs](#the-renewed-member-reference) |
| Accessing a destroyed instance | Never allowed; inspect the storage through a `* unsafe` borrow | See [repairs](#the-destroyed-instance-example) |
| Raw pressures still unnamed in the registry | Side of the split recorded; names stay unassigned | As the registry requires |

**`unique-raw-borrow`.** A `unique` allocation allows no raw borrowers. The
category absorbs `raw-from-reference`, because both describe the same
uncertain case:

```zax
owner : MyValue * unique = @
raw : MyValue * = owner.               // error: a proved violation; no safe<...> can fix it
escape : MyValue * unsafe = owner.     // the only way: an untracked borrow, fully on you

makePtr final : (result : MyValue readonly loan *)(input : MyValue readonly loan &) = {
  safe<unique-raw-borrow>{
    return input                       // input might come from a unique; the compiler can't tell
  }
}
```

The compiler can be certain the pointer is fine when the instance behind the
reference is never inside a `unique` allocation at all. In between lies a
grey area. For example, a `unique` might enter a type whose `if` logic
prevents it from ever reaching the function that returns the reference; the
programmer is certain, but the compiler cannot follow the logic. A cleverer
compiler may verify such a site; one that proves a `unique` origin reports an
error.

#### A check counts only if the compiler can follow it

**Aligned.** Wherever a check removes the need for `safe<...>`, such as an
`is type` test before opaque recovery or a `?ptr` test before access, the
compiler must be able to recognize the check:

```zax
if opaque is type MyType {
  typed : MyType * unique = opaque as last         // fine: the test is recognized
}

isWanted := fooBarChecking(opaque is type MyType, springSummerAndFallCheck) || goodToGo
if isWanted {
  typed : MyType * unique = opaque as last         // checked recovery: the compiler can't follow the test
}
```

The specification defines which check shapes count, so every compiler agrees
on what compiles. A cleverer compiler may recognize more; that falls under the
rule for a site proven safe beyond the specification and never changes what
compiles. When a check is too tangled, simplify it or write `safe<...>`.

#### Carrying a presence claim forward

**Aligned.** Every possibly vacant access through a tracked `*` needs proof or
`safe<pointer-presence>`, whatever the type's Nothing policy, including reads
of prepared members. The error lands on the operation that dereferences, such
as `myPointer.`, not inside a callee that receives the resulting reference.

- **Within a function**, a claim or a recognized `?ptr` test establishes
  non-vacancy for later uses on the same path, until something could change
  the pointer: a reset, a reassignment, or a call that can write it.
- **Across calls**, tracking follows source. If every call path into a callee
  passes a non-vacant pointer, the callee's accesses are safe, including when
  the callee is exported from its module. When the callee is compiled into a
  fixed exported ABI, its callers can no longer be tracked, so a possibly
  vacant access inside it is an error.
- A pointer type that cannot be vacant, which would carry the fact in the
  type itself, is the subject of the raw
  [present pointers](../raw/present-pointers.md) input.

### Decisions

Each decision below is **aligned**. The model above states the outcomes; this
section keeps the reasoning and the alternatives that were weighed.

#### Decision 1: the classification test

- **`safe<...>`**: a fact about state the language already tracks, where
  analysis could close the gap. Such a marker becomes redundant when analysis
  of the Zax source proves the fact.
- **`unsafe`**: a claim about something the language deliberately does not
  track, or a move back into tracking from that state. Such a marker can
  become redundant only when a new language-recognized declaration elsewhere
  takes over the vouching, true on every compiler, as in the ABI scenario for
  `opaque-construction`.

The concept is aligned; its wording must be re-authored for teaching.

#### Decision 2: classifications and the operator form

Outcomes are in the [classification table](#classifications). The operator
form for a fact-claiming operation is the plain operation inside a
`safe<...>` enclosure, not a `safe ...` operator spelling, which would read
like a checked operation:

```zax
// today
member : Container & = memberReference unsafe outer cast Container.member

// aligned form
safe<exact-origin>{
  member : Container & = memberReference outer cast Container.member
}
```

When a contract later requires the origin proof, the programmer deletes the
enclosure and the plain `outer cast` underneath is already correct. The same
applies to `unsafe via`.

#### Decision 3: the per-site directive's name

`[<verify=x-acme>]`. Before the split, `verify` next to `unsafe` read as
"verify that it is unsafe". Next to `safe<...>` it reads "verify that it is
safe". Repeating it once per compiler fits the directive grammar, which reads
every item after the first as `key=value` or a flag, so `[<verify=x-a, x-b>]`
would have read `x-b` as a flag.

#### Decision 4: attachment

On the line directly above the `safe<...>` enclosure. A slot inside the
enclosure was considered and not chosen. Declaration-level and range forms
are deferred.

#### Decision 5: the directive kind

[Compiler directives](../../language/compiler-directives.md#what-a-directive-may-promise)
labels each directive with a kind, so a reader knows whether a compiler may
ignore it and whether it changes meaning:

```zax
[<inline>]        // hint: a compiler may ignore it; meaning never changes
[<never>]         // unchecked promise: you state a fact; breaking it is undefined
[<deprecate>]     // semantic control: changes how the program is checked
```

`[<verify=...>]` fits none of these: a compiler that implements the named
verifier must not ignore it, yet it changes no meaning and promises nothing.
Outcome: a fourth kind, **verification request**, which also covers
`[<safe-verification=...>]`.

#### Decision 6: the noise-level setting

Outcomes are in [how loudly findings are reported](#how-loudly-findings-are-reported-safe-verification).
Earlier placement on the line above the import statement was replaced by
placement inside the import body. Separate control of individual findings or
verifiers is deferred and can be added later without breaking source.

#### Decision 7: what a compiler does at a `safe<...>` site

Outcomes are in [what a compiler does](#what-a-compiler-does-at-a-safe-site)
and [inside `wellformed`](#inside-wellformed-only-the-specification-decides).
This decision reversed an earlier aligned point: both findings had been lint
by default everywhere; your own code now defaults to `error`. It also
replaced an earlier aligned wording, in which a verification error inside a
`wellformed` payload escaped the check and failed the build, with the
separate `wellformed` level.

The "proved false" question from the current owner is resolved: any
compiler's certain proof of falsity is an error, and inside `wellformed` the
specification alone decides the answer.

#### Decision 8: verifier names and availability

Outcomes are in [asking a compiler to verify](#asking-a-compiler-to-verify-a-safe-site)
and the `off` rule. A build option requiring listed verifiers to be available
was rejected: `x-acme` concerns only the `x-acme` compiler, and an uncaught
misspelling is accepted. Who maintains the registry of extension names is
deferred.

Contract interactions follow the table in
[what a compiler does](#what-a-compiler-does-at-a-safe-site): a later or
selected extension contract that requires the proof makes the `safe<...>` a
redundant-marker error, and the `[<verify>]` above it then covers nothing and
is removed with it.

Generic bodies: a site is verified for every instantiation the build produces,
and a finding is judged where the marker is written and names the
instantiation. A site executed at compile time is verified the same way.

#### Decision 9: the renewed-member reference

`intent<reference-tracks-replacement>`, parallel to the existing
`intent<inner-pointer-tracks-replacement>` for interior pointers. The name is
treated as accepted and may be revisited at review. See
[repairs](#the-renewed-member-reference).

### Repairs to existing material

These are promotion obligations. Each was found while classifying.

#### The renewed-member reference

```zax
record varying : MyRecord immutable writable varying
countView : Integer readonly & = record.count

record .= makeReplacement()
inspect(countView) // error: the referenced member was renewed

intent<reference-tracks-replacement>{
  inspect(countView)   // observes the new count, as intended
}
```

The code is valid, but the reference now observes a different lifetime than
the one it was formed from; the acknowledgement confirms the programmer meant
that. Today the
[lifetimes owner](../../language/lifetimes-and-references.md#direct-member-references-cross-a-renewal-boundary)
teaches the behavior without naming any category, and the name
`replacement-member-successor` appears only in the safety owner's one-line
example and the raw registry, neither of which shows a fix. Promotion names
the intent category where the lifetimes owner teaches the behavior, with this
example; replaces the safety owner's "defined unsafe permission" example with
a real `unsafe` operation; and updates the construction owner's
cross-reference, the intent registry, and the raw analysis-control registry.

#### The destroyed-instance example

The safety owner's [false-claims example](../../language/safety-and-analysis.md#false-unsafe-claims)
uses a placeholder category `lifetime` to show that a marker cannot revive a
destroyed instance. The central rule stays: no marker makes access to a
destroyed instance valid. Promotion rewrites the example and shows the proper
escape hatch for inspecting what was left behind, without encouraging it:

```zax
secret : MySecretBuffer = loadSecret()
probe : MySecretBuffer * unsafe = secret   // untracked borrow, taken while secret is alive
secret.---()                               // the destructor clears the buffer
checkCleared(probe.)                       // compiles: untracked, so the check is on you
inspect(secret)                            // error: no instance exists
```

This is appropriate, for example, in a unit test where the buffer lives in an
arena, so the storage stays valid after destruction.

#### Nothing instances: possible vacancy

Both raw categories become one `safe<pointer-presence>`, written at the
handoff of a possibly vacant pointer:

```zax
writeCount final : ()(counter : MyCounter *) = {
  counter.count = 1           // writing through a pointer is not unsafe in itself
}

counter : MyCounter * = findCounter(id)   // vacant when id is unknown
safe<pointer-presence>{
  writeCount(counter)                     // the programmer vouches counter is not vacant
}
```

- An access the compiler proves reaches Nothing backing is an error.
- Any other possibly vacant read or write through a tracked `*` needs proof or
  `safe<pointer-presence>`, as described under
  [carrying a presence claim forward](#carrying-a-presence-claim-forward).
  This replaces an earlier aligned rule that exempted reads of prepared
  members.
- The rule depends only on what is visible statically, not on the target.

Repairs in the [Nothing owner](../../language/nothing-instances.md):

- Add this example; the owner has none for the possible-write handoff.
- State the rule for members with no prepared value plainly, and list which
  member forms are prepared and which are not, in one place. Arrays are the
  current unprepared case; future array work may prepare a dynamic array as
  empty.
- **Correct a misworded sentence (aligned).** "Invalid writes are not checked
  on every store" says the trapping policy "requires a panic for dereference
  or access even when the target lacks a cheap hardware trap". The intended
  meaning: the trapping policy relies on hardware traps that panic. Without
  hardware support, Zax cannot stop reads or writes of Nothing backing. What
  Zax can still do on such a target is check at function entry: a non-`once`
  function called with a Nothing receiver panics, like a hidden assertion that
  `_` is valid. That check needs an independently selectable panic category,
  added to the raw registry's unassigned list.

The owner already has examples for each policy (`= default`, `= trap`,
custom), the scoped `nothing-instance-default`, prepared member values, and
the trapping array; those meet the maintainer's example requirement.

#### Other example corrections

- The pointers owner's `unsafe<unique-raw-borrow>{ raw2 : MyValue * = owner. }`
  example compiles today. Under the aligned model it is an error, and
  `* unsafe` is the escape.
- The pointers owner's `raw-from-reference` example becomes
  `safe<unique-raw-borrow>`.
- The pointers owner's `unsafe transfer (opaque as last)` becomes the plain
  recovery inside `safe<opaque-type>{...}`.
- The floating-point owner's "unsafe assertion that a pattern is valid"
  becomes `safe<valid-representation>`.

### Example catalog for promotion

**Aligned obligation.** Every category needs an example showing why the
situation arises and how it is fixed. Promotion places these as a list after
the safety owner's category table, because examples cannot sit inside a table,
and may also use them in domain owners. The examples below are the working
versions; promotion re-authors their wording. Names in comments such as
`calibrate` are placeholders.

**`partial-instance-access`**

```zax
MyEngine :: type {
  sensor : MySensor
  log : MyLog

  +++ final : ()() = {
    _.sensor .= makeSensor()
    calibrate(_)         // error: _ is incomplete; log is not built yet
    _.log .= makeLog()
  }
}

calibrate final : ()(engine : MyEngine readonly loan &) = {   // in another module
  engine.sensor.zero()
}
```

Better: pass only what is ready, `calibrate(_.sensor)`. When the whole
instance must be passed: `safe<partial-instance-access>{ calibrate(_) }`.

**`partial-instance-escape`**

```zax
+++ final : ()() = {
  _.sensor .= makeSensor()
  registry.add(_)        // error: publishes an incomplete instance
  _.log .= makeLog()
}
```

Better: register after every member exists. If registration must come early:
`safe<partial-instance-escape>{ registry.add(_) }`.

**`destruction-path-complete`**

```zax
--- final : ()() = {
  if _.pooled {
    pool.release(_.buffer as last)     // buffer finished here
  }
  if !_.pooled {
    _.buffer.---()                     // or destroyed here
  }
}                                      // error: buffer may still be alive at exit
```

Better: `if … else`. Otherwise wrap the second `if` in
`safe<destruction-path-complete>{...}`.

**`destruction-at-most-once`**

```zax
--- final : ()() = {
  if isPrimary(_) { _.buffer.---() }
  if isMirror(_)  { _.buffer.---() }   // error: buffer may be destroyed twice
}
```

Better: `else if`. Otherwise wrap the second line in
`safe<destruction-at-most-once>{...}`.

**`manual-member-destruction`**

```zax
--- final : ()() = {
  unsafe<manual-member-destruction>{
    foreignRelease(_.handle)           // the C library ends handle; don't destroy it again
  }
}
```

**`terminal-reconstruction`**

```zax
--- final : ()() = {
  _.conn.---()
  reopenFlushAndClose(_)               // a helper rebuilds conn, flushes, and destroys it again
}                                      // error: the compiler can't confirm conn ended exactly once
```

Better: write the steps inline. Otherwise wrap the helper call in
`safe<terminal-reconstruction>{...}`.

**`lifetime-escape`**

```zax
pick final : (result : MyValue readonly &)(a : MyValue readonly &, b : MyValue readonly &) = {
  return chooser(a, b)                 // error: the compiler loses track of where the result came from
}
```

The programmer knows `chooser` returns one of its arguments:
`safe<lifetime-escape>{ return chooser(a, b) }`.

**`unique-raw-borrow`**: see [the model above](#classifications), including
the raw pointer from `_`:

```zax
MyValue :: type {
  selfPtr final : (result : MyValue loan *)() = {
    safe<unique-raw-borrow>{
      return _                         // _ might live inside a unique allocation
    }
  }
}
```

**`pointer-presence`**: the `writeCount` example under
[repairs](#nothing-instances-possible-vacancy), and:

```zax
acquire final : (result : MyValue * last = @)() = {   // = @ promises a present result
  reset result
  result = foreignAcquire()            // error: foreignAcquire might return vacant
}
```

Better: `if !?result { result = @ }`. Otherwise
`safe<pointer-presence>{ result = foreignAcquire() }`.

**`exact-origin`**

```zax
safe<exact-origin>{
  member : Container & = memberReference outer cast Container.member
}
```

**`opaque-type`**

```zax
typed : MyType * unique = opaque as last           // checked: vacant on mismatch

if opaque is type MyType {
  sure : MyType * unique = opaque as last          // proven by the test
}

safe<opaque-type>{
  vouched : MyType * unique = opaque as last       // you vouch opaque holds a MyType
}
```

**`valid-representation`**

```zax
value.raw = readRawCode()
safe<valid-representation>{
  use(value.code)                       // you vouch the raw bits name an admitted code
}

codeView : MyCode readonly & = value.code
update(value)                           // might write value.raw through another path
safe<valid-representation>{
  use(codeView)                         // you vouch the lens still holds a valid code
}

raw : U80 = readSavedRegister()
safe<valid-representation>{
  value := raw as X87Extended80         // illustrative: you vouch the pattern is valid
}
```

**`not-finished`**

```zax
refresh(message)                       // might finish message.payload through its writable parameter
message.payload.send()                 // error: payload may have been finished
```

The programmer knows `refresh` replaces the payload before returning:
`safe<not-finished>{ message.payload.send() }`.

**`disjoint-storage`**

```zax
// illustrative: GridTranspose maps each cell to its transposed position;
// MyGrid's cells allow neither copy nor move
adjust final : ()(target : MyGrid writable loan &, source : MyGrid readonly loan &) = {
  target -<>- GridTranspose -<>- source   // error: target and source may be the same grid,
                                         // and the cells can't be copied aside first
}
```

Better: pass grids that are visibly distinct, or make the cells copyable.
Otherwise wrap the transform in `safe<disjoint-storage>{...}`.

**Vendor extension category**

```zax
safe<x-acme-ring-index>{               // a fact only the acme compiler defines
  buffer[head] = item
}
```

**`distinct-element`, `optional-presence`, `replacement-alias`**: the existing
examples in arrays and slices, optional values, and intent acknowledgements,
converted to `safe<...>`.

### Assumptions 041 relies on

**Aligned.** These concepts are treated as true for `041`, and current owners
teach them as true, with code written accordingly. Each is also captured as
raw input that records the decisions made, so that follow-up work which
refines, revises, or reverses one also revisits the dependent `safe<...>` /
`unsafe` classifications. The maintainer expects refinement for corner cases
rather than reversal.

#### Pointer tracking and `* unsafe`

```zax
// illustrative
node : MyNode = makeNode()
link : MyNode * = node             // tracked borrow, like a slice
use(link.)                         // checked: node is still alive here

parent : MyNode * unsafe = link    // fine: leaving tracking is one-way
use(parent.)                       // the programmer's responsibility; the unsafe is in the type

back : MyNode * = parent           // error: an untracked pointer can't re-enter tracking
```

- A plain `*` borrow is tracked like a slice. The compiler checks that the
  lifetime is valid at each use, where use means accessing what the pointer
  points to.
- `* unsafe` marks a pointer that is not tracked. `unsafe` becomes a
  dimension of the pointer layer. A `*` converts to `* unsafe`, never the
  reverse.
- Where the compiler cannot tell whether a use of a tracked `*` is safe,
  `safe<...>{...}` asserts it.
- A pointer to a local value is formed with `= value`. The postfix `.` form
  applies to a pointer and yields a pointer to what it points to; writing it
  on a value that is not a pointer is likely a compile-time error.

How `041` depends on it: tracked-pointer gaps are `safe<...>` sites;
`* unsafe` is on the `unsafe` side of the split and is the escape for a
proved `unique` violation and for inspecting destroyed storage. Origin
tracking for `unique-raw-borrow` is a separate kind of tracking from borrow
lifetime tracking.

Questions for the follow-up work:

- Tracking a `*` stored in a member, such as a graph edge.
- Whether tracked `*` falls back to a runtime check where proof fails, as
  slices do with their invalidated-slice check, or only to `safe<...>`.
- Whether re-entering tracking is never allowed, or allowed only through an
  `unsafe` cast, as `cast as last` re-enters ownership tracking.
- What `* last unsafe` or `* move unsafe` would mean.
- What happens to the uses the pointers owner now assigns to raw pointers:
  parent links, graphs, addresses stored as data, foreign code, hardware
  addresses.
- The trigger rules for `safe<pointer-presence>`.

Today, as far as this review found, `*` has ownership tracking for owning raw
pointers and a raw provenance direction in
[usage and provenance tracking](../raw/usage-and-provenance-tracking.md), but
no borrow-lifetime tracking; the pointers owner says raw pointers "are for
lifetimes the compiler cannot track".

Raw destination: extend
[usage and provenance tracking](../raw/usage-and-provenance-tracking.md),
unless the dry run proposes a separate file.

#### Path-based immutability

```zax
counter : Counter mutable writable = makeCounter()
snapshot : Counter immutable readonly & = counter   // allowed if analysis proves it
counter.increment()                                 // error: counter is written while snapshot is alive
inspect(snapshot)

frozen : Counter immutable = makeCounter()
reader : Counter mutable readonly & = frozen        // allowed: same type, no marker
writer : Counter mutable writable & = frozen        // unsafe: breaks the promise to frozen's holders
```

- **The default meaning of `immutable` is path-based.** While you hold an
  `immutable` path, the value cannot change. When you no longer hold it, the
  guarantee no longer applies.
- **`immutable` to `mutable readonly` is allowed** when both are the same
  type.
- **`mutable` to `immutable` is allowed when analysis proves the path keeps
  the promise.** `mutable readonly` to `immutable` follows the same analysis;
  being `readonly` does not help in this direction. If the specification
  proves it, no marker is needed; otherwise `safe<...>` asserts it, and
  cleverer compilers may verify it.
- **Split types** allow no conversion.

How `041` depends on it: conversions to `immutable` become analysis or
`safe<...>` sites rather than `unsafe cast`; only the conversion to
`mutable writable` stays `unsafe`.

Questions for the follow-up work:

- What "hold" covers: copies of the reference, member references, and tracked
  pointers derived from it. A `* unsafe` derived from it is outside the
  guarantee.
- Where the error is reported. Tentative: at the conflicting write, pointing
  back to the immutable path.
- Re-reading construction authority, deep immutability, and future threading
  rules against the path meaning.
- The current owner's contrary statements:
  [qualifiers](../../language/qualifiers.md#ordinary-promise-strengthening)
  calls `immutable` "a guarantee about one value lifetime", says "a scoped
  non-mutating view that later permits mutation again is readonly, not
  temporarily immutable", and lists "`immutable` to `mutable`" as a weakening
  without separating the readonly case.

#### A "forever" immutability form

The maintainer wants a form whose value never changes, not only while a path
is held. The handle example shows why: a function takes a handle from an
`immutable` argument; later it receives the same value again and assumes the
handle is unchanged, although it retained no reference in between. Under path
meaning that assumption is wrong.

Options raised, none chosen:

- Qualifying the qualifier (`immutable path`, `immutable forever`): splits
  calling conventions.
- A compile-time assertion such as `verify<immutable-lifetime>{ value }`: a
  form Zax lacks.
- A type-level choice (`MyType :: type immutability forever {}`): one meaning
  per type, so no split; the type author decides for all users, and generic
  code assumes the weaker meaning unless constrained. `type mutable` and
  `type immutable` are reserved for split types.
- Whether an `identity` over an existing type can change the default.

Path is the default because "forever" can be approximated by a naming
convention, while a path-scoped promise cannot be expressed at all without
`unsafe`.

Raw destination: a new raw file for immutability, created in the promotion
change set.

### Superseded and rejected candidates

- A failed verification as a language error for the named compiler (the raw
  input as written): it could change program meaning per compiler.
- A separate verification pass invisible to compile-time queries: replaced by
  the aligned model.
- A verification error inside `wellformed` that escapes and fails the build:
  replaced by the separate `wellformed` level.
- Warning only, with no way to fail the build.
- Requiring the tag by default once a compiler proves a site safe (the
  maintainer's first framing): replaced by the `safe-verification` level.
- Lint by default for a project's own code: replaced by `error`.
- `[<safe=...>]`, `[<proved=...>]`, `[<proved-safe=...>]`,
  `[<verified-safe=...>]`, `[<prove-safe=...>]`, `[<expect-safe=...>]`: not
  chosen; `verify` reads correctly next to `safe<...>`.
- Placing `[<safe-verification>]` on the line above the import statement:
  replaced by placement inside the import body.
- `verification` and `unsafe-proof` as the setting's name.
- Classification test "vouching for safety versus accepting risk": both
  markers involve both.
- Classification test "could a contract ever make the marker redundant":
  superseded by the ABI scenario.
- `unique-raw-borrow` disappearing under pointer tracking: origin tracking is
  separate from borrow tracking.
- `unsafe vacate` as `intent<...>`.
- `safe<...>` for `cast as last` and for coercive views.
- `unsafe<usage-after-destruction>{...}`: replaced by the `* unsafe` borrow,
  which keeps the central rule.
- A required-verifier build option.
- `immutable` only as a guarantee for a value's whole lifetime: replaced by
  the path default, with "forever" captured separately.
- Showing `* unsafe` and path immutability in current owners with a "still
  being designed" marker, and keeping conversions to `immutable` out of
  current teaching (dry run 1 proposal): replaced by teaching both as true.
- Exempting reads of prepared members from `safe<pointer-presence>`:
  replaced by the tracked-`*` rule.
- `partial-instance-escape`, opaque-owner recovery, and floating-point
  pattern validity as `unsafe` (dry run 1 proposals): moved to `safe<...>`.
- A category for single consumption of source results and destination
  slots: not needed.
- The name `union-lens-validity`: renamed `valid-representation` to fit every
  use.

### Evidence and legacy tensions

- [Arrays and slices, "What the compiler can prove"](../../language/arrays-and-slices.md#what-the-compiler-can-prove)
  teaches `[<safe=…>]` "naming that compiler". Promotion updates it to
  `safe<...>` and `[<verify=...>]`.
- [Analysis controls, "Compiler-verified unsafe tags"](../raw/analysis-controls.md#compiler-verified-unsafe-tags)
  supplied the original behavior and examples.
- [What compilers must agree on](../raw/compiler-agreement-principle.md)
  supplied the principle the aligned model satisfies.
- [Safety and analysis](../../language/safety-and-analysis.md) distinguishes
  an assertion from a permission, which the split refines. Its "Analysis
  provenance" section covers an `x-` category a vendor adds, which promoted
  teaching distinguishes from asking a vendor to verify a standard category.
- [Unions](../../language/unions.md) lists the lens facts every compiler
  tracks.
- [Structural shapes, coercive reference views](../../language/structural-shapes-and-compatibility.md#coercive-reference-views)
  defines the restoration obligation that keeps unsafe coercion `unsafe`.
- [Nothing instances](../../language/nothing-instances.md) defines each type's
  policy through its `+++ final once` constructor (`= default`, `= trap`, or a
  custom body), a scoped `nothing-instance-default` attached to a namespace or
  module opening (not a range), and built-in types that always trap.

### Corner cases and risks

- **Verifier changes over time.** A new release of `x-acme` that proves less
  turns passing sites into findings; under the `error` default for a
  project's own code, a compiler upgrade can fail a build until tags are
  removed or the level is lowered.
- **Imported undefined behavior.** A proven-false claim in a library fails
  the importer's build at every level.
- **Debug instrumentation.** Verification changes no generated code.
- **Threading.** Some claims depend on other threads. What can be proved
  belongs to the threading capture in `project/raw/safety.md`; path
  immutability also needs a threading reading.

### Deferred and captured findings

- **`import` does not state how one import serves its module.** Owner:
  [namespaces and modules](../../language/namespaces-and-modules.md).
  Maintainer finding: inside a module, each imported instance is defined by
  exactly one import statement, including its injection body, and every other
  reference to that instance in the module goes through it. The current owner
  implies parts of this but does not state or resolve it.

  ```zax
  // file: first.zax, earlier in module order
  shape : Geometry.Point            // reachable only through a forward, if at all

  // file: second.zax
  Geometry :: import Module.MyGeometryModule {   // the one defining import
    // injection body: written once, here
  }

  // file: third.zax, later in module order
  origin : Geometry.Point           // the same instance, through the binding
  ```

  To settle: that an import binding is a declaration later files use; that
  the import and its injection are written once and processed at that point
  in source order; whether `forward module` remains useful or references must
  follow the import; what other files see of a direct `:: import`; where
  `import` may appear; and that a `[<safe-verification>]` inside an import body
  is a setting on the instance. Constraint on promotion: the promotion that
  places the setting in the import body must at least state that each
  instance has one defining import statement through which the rest of the
  module reaches it. Activation: the next review of `import`, modules, or
  build and dependency design.
- **Compiler-agreement wording.** Tentative proposal for the raw principle,
  reflecting the aligned rules:

  > Whether source is valid Zax depends only on rules the specification
  > states, and compile-time checks such as `wellformed` answer by those rules
  > alone. Beyond them, a compiler may report a `safe<...>` claim it proves
  > false, which is always an error, and may report sites it can prove safe,
  > at a level the build chooses. Neither changes what the program means.

  The principle stays in [its raw file](../raw/compiler-agreement-principle.md)
  unless its promotion is separately aligned.
- **A buffer-clear operation that cannot be optimized away.** Clearing writes
  in a destructor, such as zeroing a key, can be removed as dead stores
  because nothing reads them afterwards. A test that inspects the storage
  through a `* unsafe` probe may keep the writes alive and pass while
  production code loses them. The maintainer expects an intrinsic for this
  rather than reduced optimization. Not `041`'s concern; destination decided
  at the dry run.
- **Registry of extension names.** Who maintains the IANA-style registry of
  `x-` extension specification names.
- **Declaration-level and range forms of `[<verify>]`.** Activation: real code
  with many verified sites in one function.
- **Separate control of individual findings or verifiers** in
  `[<safe-verification>]`.
- **Whether source reflection exposes `[<verify>]`** well enough for tooling
  to list verified sites.
- **Official verifier names.** A plain value could later name a language
  contract ("would this be provable under contract N"). No pressure yet.

### Follow-up work items to prepare at closure

- **Path-based immutability**, with the conversion rules above, and the
  forever form as part of it or as raw input.
- **Pointer tracking and `* unsafe`**, including the trigger rules for
  `safe<pointer-presence>`.

Each starts from its raw input and carries the obligation that revising or
reversing a decision revisits the `safe<...>` / `unsafe` classifications that
depend on it.

### Promotion scope and likely owners

- [Safety and analysis](../../language/safety-and-analysis.md): the
  `safe<...>` / `unsafe` model, the classification test, verification, the
  three rules, `wellformed` behavior, noise levels, diagnostics, costs, the
  rule "use the literal `unsafe`", and the replaced examples.
- [Compiler directives](../../language/compiler-directives.md): the
  verification-request kind, catalog entries for `verify` and
  `safe-verification`, the enclosure family table (`safe<...>{ }` joins
  `intent<...>{ }` and `unsafe<...>{ }`), positions, ranges, and diagnostics.
- [Namespaces and modules](../../language/namespaces-and-modules.md): the
  setting inside the import body, why it may cross an import, and the minimum
  import statement above.
- [Compile-time code](../../language/compile-time-code.md#warnings-and-intent):
  `wellformed` answering by the specification alone, with the separate
  `wellformed` level.
- Every current `unsafe<...>{...}` occurrence, about 15 across
  [optional values](../../language/optional-values.md),
  [construction and destruction](../../language/construction-and-destruction.md),
  [pointers](../../language/pointers-and-arenas.md),
  [safety and analysis](../../language/safety-and-analysis.md),
  [declarations and bindings](../../language/declarations-and-bindings.md),
  [intent acknowledgements](../../language/intent-acknowledgements.md), and
  [unions](../../language/unions.md), reclassified by the test.
- Operator owners for the outer-cast form: [composition](../../language/composition.md),
  [function invocation](../../language/function-invocation.md),
  [lifetimes and references](../../language/lifetimes-and-references.md),
  [operator catalog](../../language/operator-catalog.md), and
  [terms](../../language/terms.md).
- [Lifetimes and references](../../language/lifetimes-and-references.md#direct-member-references-cross-a-renewal-boundary):
  the renewed-member intent.
- [Nothing instances](../../language/nothing-instances.md): the
  `pointer-presence` example and rule, the prepared-member listing, and the
  trapping contradiction.
- [Arrays and slices](../../language/arrays-and-slices.md#what-the-compiler-can-prove):
  the updated directive reference.
- Raw: [analysis controls](../raw/analysis-controls.md) (retire the
  verifier-tag section and its line in "What this input does not decide";
  move deferred remainders), the
  [analysis-control registry](../raw/analysis-control-registry.md)
  (reclassified categories), the
  [compiler-agreement principle](../raw/compiler-agreement-principle.md),
  [usage and provenance tracking](../raw/usage-and-provenance-tracking.md)
  (the pointer-tracking assumption), a new immutability raw file, and the
  [raw index](../raw/README.md).
- Whether the qualifiers and pointers owners receive interim wording before
  their follow-up work is a dry-run question: current teaching must not
  contradict the promoted `safe<...>` text, but those owners' redesigns belong
  to the follow-up items.

## Dispositions and promotion dry run

### Dry run 2: result PASS

Run on 2026-11-26, after the questions from dry run 1 were aligned. Read-only
survey of every live occurrence the promotion touches, excluding
`project/archive/` and `project/work/`, plus the pointers type-erased
ownership section, the construction destruction sections, the function
invocation routing and result-presence sections, and the structural overlap
section, which the new classifications reach.

**PASS.** Every aligned finding has one lasting owner; the owner structure
absorbs them without new owner documents; deferred material has raw
destinations; and the change set below is exact. The pointers and qualifiers
owners now receive minimal edits so that no current owner contradicts
another. This result does not authorize promotion.

### Dry run 1: result FAIL, and how its questions were resolved

Run on 2026-11-25. It failed on four questions, all since aligned:

1. **Presenting the assumed concepts.** Resolved: current owners teach
   `* unsafe`, tracked `*`, and path-based immutability as true, with code
   written accordingly, and no "being designed" marker. The pointers and
   qualifiers owners get minimal consistency edits; detailed rules go to the
   follow-up work items, and raw input records the decisions.
2. **Classifying the sites not yet examined.** Resolved by the
   [classification table](#classifications) and the
   [example catalog](#example-catalog-for-promotion). Changes from the
   proposals: `partial-instance-escape`, opaque-owner recovery, and
   floating-point pattern validity moved to `safe<...>`; single consumption
   needs no category; union-reference escape and floating-point validity
   joined `valid-representation`.
3. **The trapping contradiction.** Resolved as a misworded sentence; see
   [repairs](#nothing-instances-possible-vacancy). The tracked-`*` rule makes
   every possibly vacant access need proof or `safe<pointer-presence>`.
4. **The interior-pointer intent entry.** Resolved: promote
   `inner-pointer-tracks-replacement` and `reference-tracks-replacement`
   together into the accepted intent registry.

The detailed proposals from dry run 1 are superseded by the sections named
above and are not repeated here.

#### Choices made without a blocking question

These stood unless the maintainer objected; none was objected to:

- **Outer-cast category name:** `exact-origin`, matching the composition
  owner's term "exact-origin proof".
- **Presence category for optionals:** `optional-presence` stays separate
  from `pointer-presence`, because optional presence is a boxed-lifetime fact
  and pointer presence is non-vacancy.
- **Category catalog location:** the safety owner gains a short table of
  current categories, each linking to its domain owner. The raw registry keeps
  provisional, unassigned, and panic material and marks promoted entries as
  accepted domain entries.
- **Duplicate paragraph:** the safety owner's "Array bounds, slices, and
  disabled checks" repeats one paragraph verbatim ("Array and storage
  providers may cooperate…"). Remove the duplicate in the same change set.
- **Vision and index wording:** [vision](../../language/vision.md) says
  "Exact unsafe syntax, category identifiers, and formal guarantees remain
  future design work" and "raw-pointer operations require proof or unsafe
  responsibility". Proposed minimal updates: "proof, a `safe<...>` claim, or
  `unsafe` responsibility", and "Exact category identifiers and formal
  guarantees remain future design work". The [index](../../index.md) router's
  unions and safety lines gain `safe<...>` wording.
- **Legacy pages unchanged:** `warnings-errors.md` (`unsafe-outer-of-ambiguous`,
  legacy `unsafe as`), `basics.md`, `faq.md`, and `concurrency.md` mention
  `unsafe` only in ways the promotion does not contradict, or as legacy
  evidence; they are not retrofitted.
- **Principles unchanged:** [principles](../../language/principles.md)
  mentions "checks and unsafe boundaries" and "unsafe-control" generically;
  still true.

### Ownership map

| Aligned finding | Lasting owner |
| --- | --- |
| `safe<...>` / `unsafe` split, classification test, false claims, no valid interpretation | [Safety and analysis](../../language/safety-and-analysis.md) |
| `[<verify=...>]`, the three rules, verification diagnostics | Safety and analysis (meaning); [compiler directives](../../language/compiler-directives.md) (directive kind, catalog, positions) |
| `[<safe-verification=...>]` levels, defaults, range use, `wellformed=` level, who wins | Safety and analysis (meaning); compiler directives (catalog, range rules) |
| The setting inside an import body; one defining import per instance | [Namespaces and modules](../../language/namespaces-and-modules.md) |
| `wellformed` answers by the specification alone | [Compile-time code](../../language/compile-time-code.md#warnings-and-intent) |
| Enclosure family includes `safe<...>{ }` | Compiler directives (family table); [source structure](../../language/source-structure.md) is unchanged because it already covers `word<category>{...}` enclosures generically |
| Category catalog | Safety and analysis (table); domain owners keep each rule |
| `exact-origin` and the removal of `unsafe outer cast` / `unsafe via` | [Composition](../../language/composition.md); [operator catalog](../../language/operator-catalog.md) forms |
| `unique-raw-borrow` merge and model | [Pointers](../../language/pointers-and-arenas.md); safety "Raw pointers" summary |
| `pointer-presence` and prepared-member listing | [Nothing instances](../../language/nothing-instances.md); safety summary |
| `reference-tracks-replacement` | [Lifetimes and references](../../language/lifetimes-and-references.md); [intent registry](../../language/intent-acknowledgements.md#current-category-registry) |
| Destroyed-instance example | Safety and analysis "False unsafe claims" (renamed) |
| Path-based immutability and its conversion rules | [Qualifiers](../../language/qualifiers.md) (minimal edits, taught as true); decisions, open questions, and dependent classifications in new raw `project/raw/immutability.md` |
| The forever immutability form | New raw `project/raw/immutability.md` |
| Tracked `*` and `* unsafe` | [Pointers](../../language/pointers-and-arenas.md) (minimal edits, taught as true); decisions, open questions, and dependent classifications in [usage and provenance tracking](../raw/usage-and-provenance-tracking.md) (extended) |
| Presence-claim carry-forward, including the ABI rule | Safety and analysis; Nothing instances summary |
| Checks the compiler must be able to follow | Safety and analysis |
| Example for every category | Safety and analysis (list after the category table); domain owners where useful |
| Receiver check at function entry for trapping types | Nothing instances (behavior); [raw registry](../raw/analysis-control-registry.md) (unassigned panic category) |
| Compiler-agreement wording | [Raw principle](../raw/compiler-agreement-principle.md) (stays raw) |
| `import` statement gap (minimum statement only) | Namespaces and modules |
| Buffer-clear intrinsic, extension-name registry, declaration-level `[<verify>]`, per-finding control, reflection exposure, official verifier names | [Analysis controls](../raw/analysis-controls.md) (raw) |
| Diagnostic naming the required marker | [Analysis controls](../raw/analysis-controls.md) "Diagnostics must name the required `unsafe` form", updated to name `safe<...>` or `unsafe` |

### Structure proposal

No new owner documents and no moves. One new raw file,
`project/raw/immutability.md`, indexed in [the raw index](../raw/README.md).
The raw analysis-controls file loses its verifier-tag section but keeps other
unresolved material, so it does not retire. The raw registry stays, with
promoted entries re-labeled. Focused reading reaches the new behavior through
safety and analysis, which current routers already list. Raw material stays
out of ordinary reading paths.

### Exact proposed change set

Language owners:

- [`language/safety-and-analysis.md`](../../language/safety-and-analysis.md):
  - Metadata `Owns`: add `safe<...>` claims, verification, noise levels.
  - "Safety is a selected contract": central rule names `safe<...>` claims.
  - "Four analysis outcomes": "Unproved but valid" example becomes
    `safe<lifetime-escape>`; "Defined unsafe permission" example becomes a
    real `unsafe` operation (`unsafe cast`), and the section explains that
    `unsafe` covers what the language does not track.
  - New section after "Four analysis outcomes": **"Vouching for safety:
    `safe<...>`"** teaching the split and the classification test, with the
    category table, followed by the
    [example catalog](#example-catalog-for-promotion) as a list, and a
    subsection on checks the compiler must be able to follow.
  - "Unsafe is narrow": the rule "use the literal `unsafe`" becomes "use the
    literal `safe` or `unsafe`".
  - "False unsafe claims": renamed "False claims"; the `unsafe<lifetime>`
    example becomes the destroyed-instance example with the `* unsafe` borrow.
  - "Diagnosable unchecked Nothing access": replace the two provisional
    categories with `safe<pointer-presence>`, the tracked-`*` rule, and
    presence-claim carry-forward, including the ABI rule.
  - "Intent, unsafe, and linting are different": table gains a
    `safe<...>` row and the unsafe rows are rewritten.
  - "Domain rules remain with their owners": composition bullet names
    `exact-origin` instead of `unsafe via` and `unsafe outer cast`.
  - "Unsafe union lens validity": renamed for `safe<valid-representation>`.
  - "Replacement and aliases": the "defined unsafe successor boundary" bullet
    points to the intent.
  - "Returned references": `safe<lifetime-escape>`.
  - "Raw pointers": tracked `*` and `* unsafe`; the `unique` rules follow the
    merged model.
  - "Opaque owner and observer recovery": `unsafe transfer` becomes the plain
    recovery inside `safe<opaque-type>{...}`.
  - "Array bounds…": remove the duplicate paragraph.
  - "Language contracts and compiler analysis": the source-validity table and
    composition example use `safe<exact-origin>`; "A compiler may prove more"
    is rewritten around the three rules.
  - New subsection: **"Asking a compiler to verify a claim"** with
    `[<verify>]`, the three rules, `wellformed`, and noise levels.
  - "Analysis provenance": `safe<x-...>` / `unsafe<x-...>` wording.
  - "Diagnostics": add verification findings and their levels.
  - "Boundaries and maturity": exact category names stay deferred; syntax no
    longer is.
- [`language/compiler-directives.md`](../../language/compiler-directives.md):
  family table adds `safe<...>{ }`; "What a directive may promise" adds the
  verification-request kind; catalog adds `verify` and `safe-verification`;
  "Directive ranges" lists `safe-verification` among range directives, with
  its levels in place of `always`/`never`; diagnostics add a `[<verify>]` with
  no `safe<...>` below it.
- [`language/namespaces-and-modules.md`](../../language/namespaces-and-modules.md):
  "Generative imports and injection" states that each instance has one
  defining import statement through which the rest of the module reaches it,
  and adds the `[<safe-verification>]` setting inside an import body, which is
  a setting on the instance rather than injected source; "Directive state
  stops at an import" gains the exception and its reason.
- [`language/compile-time-code.md`](../../language/compile-time-code.md):
  "Warnings and intent" adds that `wellformed` answers by the specification
  alone and that cleverer-compiler findings use the `wellformed` level.
- [`language/composition.md`](../../language/composition.md): metadata,
  "result" rules (line ~1022), the `Car` example (`assertedPeerOwner` inside
  `safe<exact-origin>{...}` with plain `via`), "Proved, tracked, and unsafe
  outer casting" (renamed), the `inspectContainer` example, managed-pointer
  paragraph, the source-validity table's third column, the `via` three-way
  list, diagnostics, and formatting.
- [`language/function-invocation.md`](../../language/function-invocation.md):
  the `_` raw-pointer sentence; the presence-assertion sentence; the
  `assertedOwner` example.
- [`language/operator-catalog.md`](../../language/operator-catalog.md):
  remove `unsafe outer cast` from the navigation row and outer-cast forms;
  show plain `outer cast` inside `safe<exact-origin>`.
- [`language/terms.md`](../../language/terms.md): outer-cast entry; new
  entries "Safety claim" (`safe<...>{...}`) and "Verification request",
  parallel to "Intent acknowledgement".
- [`language/lifetimes-and-references.md`](../../language/lifetimes-and-references.md):
  renewal paragraph names `intent<reference-tracks-replacement>` with the
  example; outer-cast paragraph drops `unsafe outer cast`.
- [`language/intent-acknowledgements.md`](../../language/intent-acknowledgements.md):
  "Intent, unsafe, and lint suppression are different" and "Intent is not
  unsafe" use `safe<replacement-alias>`; registry gains
  `reference-tracks-replacement` and `inner-pointer-tracks-replacement`.
- [`language/pointers-and-arenas.md`](../../language/pointers-and-arenas.md):
  - "Raw pointers": replace "Raw pointers are for lifetimes the compiler
    cannot track" with the tracked-borrow rule and `* unsafe` as the untracked
    form, with one example; move the listed untracked uses (parent links,
    graphs, addresses stored as data, foreign code, hardware addresses) under
    `* unsafe`; a `*` formed from a local value uses `= value`.
  - The `raw-from-reference` example becomes `safe<unique-raw-borrow>`; the
    `unique-raw-borrow` example becomes a proved error with the `* unsafe`
    escape; the line-730 sentence; diagnostics lines for `unique` and unseen
    origin.
  - "Safe and unsafe owner recovery": `unsafe transfer (opaque as last)`
    becomes the plain recovery inside `safe<opaque-type>{...}`; heading
    renamed.
  - "Boundaries and maturity": stored-pointer tracking, the runtime-fallback
    question, re-entering tracking, and `* last unsafe` listed as future work
    by subject.
- [`language/qualifiers.md`](../../language/qualifiers.md):
  - "Ordinary promise strengthening" and "Unsafe casts and new values":
    path-based `immutable`; `immutable` to `mutable readonly` allowed for the
    same type; `mutable` and `mutable readonly` to `immutable` by analysis or
    `safe<...>`; `immutable` to `mutable writable` stays `unsafe`; remove "not
    temporarily immutable" and the "guarantee about one value lifetime"
    wording.
  - "Boundaries and maturity": what holding a path covers, where the error is
    reported, and a forever form listed as future work by subject.
- [`language/floating-point-scalars.md`](../../language/floating-point-scalars.md):
  "unsafe assertion that a pattern is valid" and the `U80` line become
  `safe<valid-representation>`.
- [`language/structural-shapes-and-compatibility.md`](../../language/structural-shapes-and-compatibility.md):
  "Overlap is handled statically" replaces "unless future analysis-control
  work supplies an explicit non-overlap contract" with
  `safe<disjoint-storage>` and a short example.
- [`language/optional-values.md`](../../language/optional-values.md): three
  examples and two prose references become `safe<...>`.
- [`language/construction-and-destruction.md`](../../language/construction-and-destruction.md):
  `opaque-construction` stays `unsafe<...>`; "A future narrow unsafe
  permission may allow a use…" names `safe<lifetime-escape>`; "Advanced
  terminal code may need a narrow future unsafe permission…" names
  `safe<terminal-reconstruction>`; destructor sections gain short examples
  for `destruction-path-complete`, `destruction-at-most-once`, and
  `manual-member-destruction`, and constructor material for the two
  partial-instance categories, where they fit; the closing distinction
  sentence uses the split.
- [`language/declarations-and-bindings.md`](../../language/declarations-and-bindings.md):
  unchanged example (`opaque-construction` stays `unsafe`); no edit needed.
- [`language/unions.md`](../../language/unions.md): `safe<valid-representation>`
  example and sentence, plus the lens-reference example.
- [`language/nothing-instances.md`](../../language/nothing-instances.md):
  "Possible write flow…" rewritten with the `pointer-presence` example and
  the tracked-`*` rule; a prepared-member list near "What compiler
  preparation provides"; the misworded trapping sentence corrected, with the
  receiver check at function entry.
- [`language/function-invocation.md`](../../language/function-invocation.md)
  (in addition to the items above): the presence-assertion sentence names
  `safe<pointer-presence>` with the `acquire` example.
- [`language/arrays-and-slices.md`](../../language/arrays-and-slices.md):
  "What the compiler can prove" uses `safe<distinct-element>` and
  `[<verify=…>]`; line 98's "would need `unsafe`" becomes the `* unsafe`
  wording.
- [`language/identity-types.md`](../../language/identity-types.md): "an
  unchecked or unsafe assertion" becomes "an unchecked operation or a
  `safe<...>` claim".
- [`language/vision.md`](../../language/vision.md) and
  [`index.md`](../../index.md): the minimal wording above.

Raw and project files:

- [`project/raw/analysis-controls.md`](../raw/analysis-controls.md): retire
  "Compiler-verified unsafe tags"; update "Permission regions", category
  tables, and examples to the split; record the deferred items listed in the
  ownership map; update "Diagnostics must name the required `unsafe` form";
  update "What this input does not decide".
- [`project/raw/analysis-control-registry.md`](../raw/analysis-control-registry.md):
  re-label promoted entries; rename `union-lens-validity` to
  `valid-representation`; merge `raw-from-reference` into
  `unique-raw-borrow`; replace the Nothing categories with `pointer-presence`;
  add `exact-origin`, `opaque-type`, `not-finished`, and `disjoint-storage`;
  classify the construction-derived categories; move
  `replacement-member-successor` to the intent registry; remove single
  consumption from unassigned pressure; record the side of the split for the
  remaining unassigned pressure; add the receiver check at function entry to
  unassigned panic categories; update the occurrence audit and the
  `inner-pointer-tracks-replacement` note.
- [`project/raw/compiler-agreement-principle.md`](../raw/compiler-agreement-principle.md):
  the tentative wording and the `[<verify>]` reference.
- [`project/raw/usage-and-provenance-tracking.md`](../raw/usage-and-provenance-tracking.md):
  add the decisions made for tracked `*` and `* unsafe` (now taught by the
  pointers owner), the presence-claim carry-forward and ABI rule, the open
  questions, and which classifications depend on them; `unsafe<...>` becomes
  `safe<...>`.
- New `project/raw/immutability.md`: the decisions made for path-based
  immutability (now taught by the qualifiers owner), the forever form and its
  options, the open questions, and which classifications depend on them.
- [`project/raw/safety.md`](../raw/safety.md): outer-provenance passage and
  Nothing categories updated; `opaque-construction` unchanged.
- [`project/raw/callable-selection.md`](../raw/callable-selection.md): the
  `unsafe via` sentence.
- [`project/raw/README.md`](../raw/README.md): index the new immutability
  file; update the analysis-controls and usage-tracking triggers.

Not changed: `language/source-structure.md`, `language/casting.md`
(`unsafe cast` and `unsafe cast as` stay), `language/enums.md`
(`unsafe from` stays), `language/declarations-and-bindings.md`
(`opaque-construction` stays), `language/principles.md`,
`project/raw/reflection.md` (it mentions only plain and tracked forms),
`project/raw/present-pointers.md` (referenced, not changed), and legacy root
pages other than `index.md`.

### Teaching plan for changed owner sections

- **Safety, "Vouching for safety: `safe<...>`"** (new). Reader: has just
  seen proven-safe and unproved-but-valid code. Knows: the compiler requires
  proof. Changes: two markers instead of one; which to use. Might wrongly
  try: `safe<...>` around an unchecked cast, or `unsafe` around a lifetime
  fact. Lead with one `safe<distinct-element>` and one `unsafe cast`, then the
  test in plain words, then the table.
- **Safety, "Asking a compiler to verify a claim"** (new). Reader: maintains
  code with `safe<...>` and uses a capable compiler. Knows: markers, contracts.
  Changes: `[<verify>]`, what each compiler does, defaults. Might wrongly
  expect: the directive to remove the marker, or a clever compiler to change
  which code compiles. Lead with the two findings in source, then the rules
  table, then `wellformed`, then levels and imports.
- **Safety, "False claims"**. Reader: wants to know the limits of a marker.
  Might wrongly try: a marker to read a destroyed value. Show the error, then
  the `* unsafe` escape, as a proper escape hatch rather than an encouraged
  pattern.
- **Safety, the category examples.** Reader: met an error naming a category,
  or wants to know when one applies. Changes: one example per category, after
  the table. Might wrongly reach for a marker when restructuring works; each
  example shows the fix without a marker first where one exists.
- **Safety, checks the compiler must follow.** Reader: wrote a check but still
  gets an error. Changes: only recognized check shapes count. Show the
  recognized `is type` test beside the tangled one.
- **Pointers, tracked `*` and `* unsafe`.** Reader: forms a pointer to a
  local or stores one. Knows: references are checked. Changes: plain `*` is
  checked at each use; `* unsafe` is the untracked form, one-way. Might
  wrongly try converting `* unsafe` back to `*`. Short example, then the list
  of untracked uses, then a boundary sentence naming future work.
- **Qualifiers, path-based `immutable`.** Reader: wants a value that does not
  change while they hold it. Knows: `readonly` versus `immutable`. Changes:
  `immutable` is a promise while the path is held; conversions in each
  direction. Might wrongly expect `readonly` to make `mutable` into
  `immutable`. Lead with the `snapshot` example and the conflicting write,
  then the conversion list.
- **Compiler directives, kinds and catalog.** Reader: meets `[<verify>]` in
  code. Changes: a fourth kind. Might wrongly assume it is a hint a compiler
  may ignore. One sentence per kind with the existing examples.
- **Namespaces and modules, import body setting.** Reader: importing a noisy
  library. Changes: one line in the import body. Might wrongly expect the
  setting to reach the library's own imports, or the library's directives to
  override it. Show `C :: import Module.C { [<safe-verification=off>] }` and
  the reason it may cross an import.
- **Compile-time code, `wellformed`.** Reader: writing a probe. Changes: the
  answer never depends on the compiler. One paragraph after the deprecation
  rule.
- **Composition, outer casting.** Reader: casting to a container without a
  proof. Changes: no `unsafe outer cast`; plain form inside
  `safe<exact-origin>`. Might wrongly write `safe outer cast`. Show today's
  table with the third column rewritten.
- **Pointers, `unique` and raw borrowers.** Reader: wants a raw pointer near a
  `unique`. Changes: certain violation is an error; uncertain origin uses
  `safe<unique-raw-borrow>`; `* unsafe` is the escape. Might wrongly wrap a
  proved violation in `safe<...>`.
- **Nothing instances, possible vacancy.** Reader: passes a pointer that might
  be vacant to a writer. Changes: `safe<pointer-presence>` at the handoff;
  which reads need it. Lead with the `writeCount` example; then the list of
  prepared and unprepared members.
- **Lifetimes, renewal.** Reader: holds a member reference across
  replacement. Changes: an intent, not an unsafe permission. Show the error,
  then the acknowledged form.
- **Intent acknowledgements, the distinction sections.** Reader: deciding
  between intent and a safety marker. Changes: `safe<...>` replaces "unsafe
  assertion". One example swap.
- **Arrays, "What the compiler can prove".** Reader: finishing elements
  outside the listed cases. Changes: `safe<distinct-element>` and
  `[<verify>]`. Replace the paragraph's spelling and link to the new safety
  section.

### Validation checks for the promotion

- Every `unsafe<` occurrence in live owners is reclassified or intentionally
  kept (`opaque-construction`, `manual-member-destruction`).
- No live owner mentions `unsafe outer cast`, `unsafe via`,
  `unsafe transfer`, `raw-from-reference`, `union-lens-validity`,
  `possible-nothing-write`, `possible-nothing-read-trap`,
  `replacement-member-successor`, or `[<safe=`.
- The pointers and qualifiers owners no longer say raw pointers are untracked
  or that `immutable` cannot be temporary.
- Every category in the safety table has an example.
- No current owner cites numbered work.
- Links and anchors resolve; renamed headings ("False claims", the outer-cast
  section) have their inbound links updated.
- A cold-reader agent reviews the new safety sections and the composition,
  pointers, and Nothing changes, with the colleague-voice guidance.

### Promotion record

Promotion applied on 2026-11-26 after authorization. The work file was staged
by the maintainer before promotion; this record and all owner and raw edits
are unstaged.

#### Revisions to the dry-run change set found during promotion

Each follows from an aligned finding; none adds new design.

- **`language/casting.md` changed** (listed as unchanged). Its "Qualification"
  section said adding `immutable` always needs `unsafe cast` because
  `immutable` holds for a value's whole lifetime. Rewritten for path-based
  immutability, with a matching diagnostics bullet.
- **`language/operators.md` and the operator catalog's opaque row changed.**
  Both listed `unsafe transfer (value as last)`, which the dry run missed.
  Rewritten to the plain recovery inside `safe<opaque-type>`.
- **Category name `no-mutation-while-held` chosen during promotion** for a
  conversion to `immutable` that analysis cannot prove; no name had been
  aligned. Added to the safety category table with an example. Open to the
  maintainer's revision.
- **Construction's "future narrow unsafe permission" sentence** about
  replacement aliasing names `safe<replacement-alias>`, not
  `safe<lifetime-escape>` as the change set said; the surrounding text is
  about replacement aliasing.
- **`not-finished` example changed.** The catalog example
  (`refresh(message)` then `message.payload.send()`) relies on tracking across
  calls, which is still raw direction; transfer stances says a member
  finished inside a called function is left to that function and the type.
  The promoted example uses uncertainty within one function, and says
  cross-call tracking is future work.
- **`destruction-path-complete` example corrected.** The catalog example used
  `pool.release(_.buffer as last)` on one path, but finishing a member with
  `as last` does not end it: its destructor still runs. The promoted example
  destroys explicitly on two complementary paths. The construction owner's
  sentence about `as last` was corrected the same way.
- **Unchecked dereference scoped to `* unsafe`.** The Nothing owner ("Pointer
  dereference is unchecked", "Invalid writes are not checked on every
  store") and the safety owner ("Diagnosable unchecked Nothing access") showed
  plain `*` dereference as unchecked, contradicting the aligned tracked-`*`
  presence rule. Their examples now use `* unsafe`, and both say that whether
  owning and managed pointers follow the tracked rule is future work. That
  question is recorded in the pointer raw input.
- **A proven-vacant access in the Nothing owner** (`use(owner.x)` after
  `reset owner`) was shown as compiling; it is now marked as an error, which
  the owner's own rule already required.
- **Audit wording corrected.** "Searching for `safe<` finds every place of
  responsibility" was false for `unsafe` operations; the safety owner now says
  to search for both `safe<` and `unsafe`.

#### Cold-reader review

A cold-reader agent reviewed the changed sections. Resolved in promotion:

- the dereference contradiction, the proven-vacant example, the
  destruction-path example, and the search claim (above);
- whether caller tracking is portable: the pointers owner now says tracking
  follows source, consistent with the safety owner, and "fixed exported
  interface" is explained;
- the destroyed-storage escape now says `* unsafe` leaves accesses unchecked
  rather than making them valid, and why arena storage can still be read;
- the meaning of `= @` in the `pointer-presence` example and where the fix
  goes;
- how the own-code `error` default relates to every compiler accepting the
  same programs, and the default `wellformed` level;
- the two-category enclosure example now says which fact each category
  covers;
- the misleading `member` name in the `exact-origin` example;
- the postfix `.` description, a pronoun in the pointers owner, "code acts"
  wording in composition, an unfinished comparison in qualifiers, the
  `wellformed=` level and range override in namespaces, and the
  interior-pointer parallel in lifetimes, which is written at formation, not
  use;
- why `once` functions are excluded from the receiver check.

**Open questions for the maintainer**, recorded in
[usage and provenance tracking](../raw/usage-and-provenance-tracking.md):

- **Why `unique` forbids tracked raw borrowers.** The old reason was that raw
  pointers were untracked. With tracked `*`, a reader asks why a slice may
  borrow from a `unique` and a tracked raw pointer may not. Current teaching
  states the rule without a reason.
- **Whether owning and managed pointers follow the presence rule.**

#### Teaching observations outside the promoted change

The cold reader also reported friction in text this promotion did not
materially change. Per the documentation architecture, these go to the
[teaching-debt log](../teaching-debt-log.md) at archival unless resolved
first:

- discussion vocabulary such as "authority", "authorized", and "canonical" in
  safety (opaque recovery, intent), construction ("terminal destruction
  authority"), qualifiers ("same-place authority conversion", "exclusive
  authority"), and pointers ("ownership/capability profile",
  "type-witness check");
- safety: "`value is type MyType` is a safe nonexecuting proof operation" and
  other dense phrasing in the opaque recovery and contract sections; the
  double `as last` in `(view unsafe cast as last) as last` is unexplained
  where first shown;
- safety: `deliberatelyTracked : Container & ? = intent<...>{ ... }` uses an
  enclosure as an initializer, which reads against "enclosures create no
  scope and no value";
- composition: whether plain `outer cast` compiles today given a
  "conceptual proof model"; "undefined behavior" versus "cannot corrupt
  ownership accounting" for a false claim; the tracked form "produces absence
  or vacancy" in composition versus "absence" in safety; the redundant-tracking
  diagnostic shown only in prose;
- Nothing: "There is no separate `write trap` policy" denies a concept the
  text never introduces; "accessing an unprepared member panics" versus the
  trap capability caveat;
- qualifiers: `copy final : Payload = source` uses the stance word `copy` as
  a variable name;
- pointers: the `p.` table row omits `unique`, though `unique` examples use
  `owner.`;
- lifetimes: "renewal boundary" and "renews every member resident instance"
  are undefined where used;
- construction: "Lifecycle analysis should distinguish them" is design voice.

#### Post-promotion review

The maintainer's read-through led to two safety-owner fixes (examples for
`distinct-element`, `optional-presence`, and `replacement-alias` in place of
"shown by their owners"; "establishes the fact" replaced by the fact it
establishes, also in the Nothing owner) and three captures: example coverage
for every category in `project/raw/cross-cutting-audit.md`, per-category proof
rules in `project/raw/analysis-controls.md`, and opaque recovery revalidation
in `project/raw/pointer-and-arena-mechanics.md`. The maintainer accepted the
promoted changes, including the category name `no-mutation-while-held`.

### Closure

**Closure, 2026-11-26.** Authorized by the maintainer. Every material finding
has a live destination:

- promoted design in the current owners listed in the promotion record;
- the `import` concerns in work item `042`, which the maintainer chose as the
  next item and widened to a general review of importation, including the
  `[<safe-verification>]` setting in import bodies;
- path-based immutability and the forever form in
  `project/raw/immutability.md`;
- tracked pointers, `* unsafe`, presence carry-forward, and the two open
  pointer questions in `project/raw/usage-and-provenance-tracking.md`;
- deferred verification questions, the buffer-clear intrinsic, and
  per-category proof rules in `project/raw/analysis-controls.md`;
- category history in `project/raw/analysis-control-registry.md`;
- the compiler-agreement wording in
  `project/raw/compiler-agreement-principle.md`;
- example coverage in `project/raw/cross-cutting-audit.md`;
- opaque recovery in `project/raw/pointer-and-arena-mechanics.md`;
- the teaching observations above in `project/teaching-debt-log.md`.

Pointer tracking and immutability remain queued as follow-up work items; their
raw inputs carry the activation pressure and the obligation to revisit
dependent safety classifications.
