# 020: `own` and composition

| Field | Value |
| --- | --- |
| Status | Active working material / non-normative |
| Work Item | `020` |
| Created | 2026-09-10 |
| Owns | The bounded review defined by the fixed initiating input |
| Does Not Own | Class inheritance, a complete interface or trait system, generalized structural subtyping, complete generic or partial-type behavior, reflection, callable capture, or compiler lowering |

## Non-authority notice

This file is a collaborative working record. Existing statements, new
observations, candidate wording, and aligned findings remain non-authoritative
until a separately discussed, aligned, and explicitly authorized promotion
incorporates them into their lasting owners.

## Fixed initiating input

This section records the information aligned when work item `020` was created.
It is intentionally incomplete and must not be rewritten as work develops.

### Initiating concern

Recover and refine a coherent, data-oriented model for `own` and composition
without introducing class inheritance or hiding the contained relationship.

The language maintainer supplied a
[refreshed `own` and composition mental model](../raw/owned-composition-new-mental-model.md)
as the intended primary conceptual starting point for this work. Treat that
material as the refreshed candidate model to understand, test, and refine. Its
role as the initiating model does not make every detail accepted language
design; promotion still requires ordinary discussion, alignment, documentation
fit, and explicit authorization.

Earlier composition material is superseded as a mental model. Read it as
supplemental evidence to identify concrete requirements, examples, failure
cases, or consequences absent from the refreshed model. Do not preserve a
conflicting legacy rule merely because it was previously documented, and do not
silently merge old behavior into the refreshed model.

The review should establish whether and how a separately stored member can
participate in its containing value's programmer-visible surface while
preserving explicit composition, identity, lifetime, and cost.

### Motivating pressure

Zax's foundational direction favors composition over inheritance, explicit data
shape and representation, structural relationships, and visible cost. Legacy
material proposes `own` as a way to expose contained state and behavior through
an outer value, but it mixes several potentially distinct ideas:

- ordinary named containment;
- member and operation exposure;
- implicit selection of one contained value;
- ambiguity resolution across several contained values;
- identity and representation relationships;
- interface-like substitution;
- outer and inner conversion; and
- local lexical member exposure.

Those ideas must not be accepted or rejected as one package. The refreshed
mental model supplies the primary reconstruction. Older material may reveal
additional pressure, but it no longer supplies the default model.

### Known assumptions

- Zax favors explicit composition and does not gain class inheritance through
  `own`.
- A contained value has ordinary construction, lifetime, qualification,
  transfer, replacement, and destruction behavior unless this review aligns a
  concrete additional rule.
- Current identity-type behavior remains owned by
  `language/identity-types.md`; composition must not silently erase a distinct
  identity boundary.
- Current function selection, result routing, operator resolution, and
  declaration lookup remain in force unless a concrete composition consequence
  requires an aligned integration change.
- The refreshed mental model is the primary candidate input. Legacy composition
  material is supplemental and superseded.
- No compiler implementation exists in this repository.

### Known inclusions

- The ordinary named-containment baseline and the exact additional purpose, if
  any, served by `own`.
- The stored-place and identity relationship between an outer value and each
  owned member.
- Whether the contained member remains directly and explicitly nameable.
- Which stored members, functions, operators, phrases, type-level operations,
  or other declarations may participate in an exposed surface.
- Visibility, filtering, suppression, and owner authority.
- Precedence, ambiguity, and explicit disambiguation when an outer value or
  several owned members provide matching names or operations.
- Qualification, access capability, transfer stance, source-state, and
  mutability consequences.
- Construction, replacement, lifetime, destruction, and disposal consequences.
- Function-input, function-result, operator-resolution, and conversion pressure
  created by an exposed surface.
- Interaction with current transparent aliases and distinct identity types.
- Multiple owned members, nested composition, and source evolution.
- Cost visibility, diagnostics, source stability, and formatting.
- Complete disposition of the refreshed model, the legacy composition page,
  the older indexed owned-composition input, and the surviving
  `using value own` member-exposure evidence.
- Lasting documentation ownership and legacy/raw disposition.

### Known starting boundaries

- Class inheritance, virtual base classes, vtables, or compiler-hidden object
  hierarchy.
- A complete generalized interface, trait, or runtime-polymorphism facility.
- General structural equivalence, layout compatibility, subtyping, or
  truncating structural transfer beyond direct consequences of the reviewed
  composition relationship.
- Complete type-parameter, generic-specialization, partial/open-type, or
  reflection behavior.
- General callable capture, closure representation, or function composition.
- General local lexical member injection. The legacy `using value own` proposal
  receives a disposition but does not reopen resource lifetime or grant
  implicit local projection.
- Compiler lowering, ABI implementation, or optimizer strategy.

These boundaries do not erase consequences. Preserve concrete pressure in a
live owner or indexed raw destination rather than leaving it only in the
eventual archive of this work item.

### Intentionally unresolved framing

- Whether `own` denotes a semantic relationship, surface-generation sugar, an
  explicit forwarding declaration, or some combination.
- Whether the named contained place always remains directly accessible.
- Whether exposure includes data, behavior, type-level declarations, or only an
  explicitly selected subset.
- Whether an outer value can satisfy a parameter or result expecting an owned
  member's type, and whether doing so is projection, conversion, or selection.
- How locally declared outer behavior interacts with exposed behavior.
- How ambiguity is diagnosed and explicitly resolved.
- Whether exposure is fixed when the outer type is completed or can change when
  a source type changes.
- Which relationships are representation-preserving and which require ordinary
  construction or transfer.
- Whether existing `own` spelling remains coherent after the model is tested
  across qualifiers, transfer, lifetime, identity, and operation selection.

### Initial stopping guidance

Stop when the work has:

- established a coherent ordinary-composition and `own` mental model;
- dispositioned every material claim in the refreshed and legacy inputs;
- defined storage, identity, exposure, lookup, ambiguity, access, transfer, and
  lifecycle consequences;
- integrated every accepted consequence with current invocation, operator,
  declaration, identity, qualifier, transfer, lifetime, and construction
  owners;
- preserved structural typing, generics, partial types, reflection, callable,
  casting, and implementation pressure without prematurely designing those
  complete facilities;
- identified exact lasting owners and legacy/raw changes; and
- completed the required documentation-fit dry run.

Do not promote findings, archive this work item, begin work item `021`, or
redesign generalized structural typing, interfaces, generics, partial types,
reflection, or callable capture without the separately required discussion,
alignment, and authorization.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs focused reading,
  decision maturity, promotion, deferral, and closure.
- [Refreshed `own` and composition mental model](../raw/owned-composition-new-mental-model.md)
  - supplies the maintainer's intended primary candidate model and must be read
  before reconstructing `own`.
- [Language vision](../../language/vision.md#data-oriented-organization-and-composition)
  - owns the accepted foundational preference for composition over inheritance.
- [Declarations and bindings](../../language/declarations-and-bindings.md) -
  owns stored declarations, lookup, visibility, shadowing, and initialization
  framing.
- [Qualifiers](../../language/qualifiers.md) - owns access capabilities,
  qualification, mutability, and place behavior.
- [Construction and destruction](../../language/construction-and-destruction.md)
  - owns construction packets, replacement, lifecycle tracking, and terminal
  destruction.
- [Transfer stances](../../language/transfer-stances.md) - owns copy, deep, move,
  last, and source-state behavior.
- [Lifetimes and references](../../language/lifetimes-and-references.md) - owns
  life paths, references, places, and storage-duration consequences.
- [Function invocation](../../language/function-invocation.md) - owns callable
  selection, input and result binding, complete prototypes, routing, and
  elision.
- [Identity types](../../language/identity-types.md) - owns transparent aliases,
  distinct identities, projection, admission, and current exposed/opaque
  behavior.
- Legacy [composition](../../composition.md) - supplies superseded public
  composition, `own`, ambiguity, interface, slicing, and conversion claims;
  inspect it for additional evidence, not as the current mental model.
- Older [owned-composition input](../raw/owned-composition.md) - supplies
  superseded identity-exposure, signature-transformation, filtering,
  same-storage-view, casting, and reflection pressure; retain only independently
  justified additional insight.

### Consequence-driven

Read these when a concrete question reaches their ownership boundary:

- [Operators](../../language/operators.md),
  [operator phrases](../../language/operator-phrases.md), and
  [mixfix operators](../../language/mixfix-operators.md) for exposed-operation
  declarations, lookup, fencing, and ambiguity.
- [Source structure](../../language/source-structure.md) and
  [terms](../../language/terms.md) for type-body placement, parsing, naming, and
  vocabulary.
- Legacy [type definitions](../../type-definition.md) and
  [casting](../../casting.md) for concrete type-body or conversion evidence not
  already preserved by the required inputs.
- Raw [callable selection](../raw/callable-selection.md) for exact-prototype or
  `own` surface-selection pressure.
- Raw [structural typing](../raw/structural-typing.md) for shape, equivalence,
  layout, conversion, or subtyping pressure.
- Raw [partial types](../raw/partial-types.md) for suppression, extension, or
  owner-authority pressure.
- Raw [reflection](../raw/reflection.md) for exposed-versus-local declaration
  identity and source-evolution visibility.
- Raw [mutability-indexed type families](../raw/mutability-indexed-type-families.md)
  for qualifier-specialized representation or surface pressure.
- Raw [function composition and chaining](../raw/function-composition-and-chaining.md)
  only when callable capture or generated-callable pressure cannot remain
  separate.
- Raw [cross-cutting audit](../raw/cross-cutting-audit.md) when an aligned
  finding creates or resolves a repository-wide audit entry.

### Audit-only

- Archived [work item `019`](../archive/work/019-using-and-scoped-resource-lifetimes.md)
  only when targeted provenance for the retired `using value own` spelling is
  necessary beyond the live raw inputs.
- Earlier archived work only when a concrete provenance or regression question
  cannot be answered from current owners and live raw input.

## Initial working posture

Begin with the refreshed mental model and reconstruct its programmer-visible
claims in plain language. Test each claim against ordinary composition and the
current type, identity, declaration, qualifier, transfer, lifetime,
construction, invocation, and operator models.

Use legacy material to challenge coverage, not to restore superseded framing.
For each additional legacy insight, record why the refreshed model does not
already cover it and whether it belongs in `020`, a current owner, another raw
destination, or retirement.

Do not promote candidate behavior while reconstructing it. Discussion,
alignment, documentation-fit dry run, and explicit promotion authorization
remain separate steps.
