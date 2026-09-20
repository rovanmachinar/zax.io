# 028: Types, unions, and variants

| Field | Value |
| --- | --- |
| Status | Active working material / non-normative / awaiting assignment |
| Work Item | `028` |
| Created | 2026-09-20 |
| Owns | The bounded review defined by the fixed initiating input |
| Does Not Own | Complete generics, pattern matching beyond concrete variant pressure, FFI/ABI contracts, reflection schemas, compiler implementation, or runtime layout except where a programmer-visible type/union/variant decision constrains them |

## Non-authority notice

This file is a collaborative working record. Existing statements, new
observations, candidate wording, and aligned findings remain non-authoritative
until a separately discussed, aligned, and explicitly authorized promotion
incorporates them into their lasting owners.

## Fixed initiating input

This section records the aligned information known when work item `028` was
created. It is intentionally incomplete and must not be rewritten as work
develops.

### Initiating concern

Establish a coherent programmer-facing model for defining and using Zax types,
including ordinary stored/member behavior, unmanaged overlapping `union`
storage, managed `Variant` alternatives, and the type-declaration integration of
`outer tracked`.

Zax discusses type behavior throughout declarations, construction, lifetime,
qualification, composition, structural compatibility, identity, enums,
optionals, and pointers without one dedicated current language entry teaching
how a type definition works as a whole. The root `type-definition.md` page
contains substantial legacy evidence but is not authoritative language design.

The maintainer-supplied
[types, unions, and variants notes](../raw/types-unions-variant-maintainer-notes.md)
are the latest primary input for this work item. They may add to, replace,
supersede, or reject conflicting legacy evidence.

Current `language/` owners remain current for the concerns they already own.
Root legacy pages and other legacy examples are always input to mine for useful
intent, not authority merely because current review has not yet replaced them.
When refreshed maintainer notes and legacy material conflict, treat the
maintainer notes as the likely current direction and bring any conflict with a
current `language/` owner to explicit review.

### Motivating pressure

The work must give a programmer one teachable model for:

- what `Name :: type { ... }` introduces;
- which declarations contribute stored shape, behavior, metadata, or no
  per-instance storage;
- named and anonymous type identity;
- incomplete self-reference, forwards, recursive layout, and completion;
- member construction, lifetime, replacement, destruction, and generated
  operations;
- qualification and access across a type and its members;
- physical containment, composition-facing paths, and structural shape;
- unmanaged overlapping storage and its active-lifetime/safety obligations;
- managed variant alternatives and their active-path behavior;
- selection, access proof, transfer, replacement, and destruction of
  alternatives; and
- `outer tracked` as a type capability with hidden placement and lifecycle cost.

Without a cohesive owner, programmers must reconstruct basic type meaning from
several specialized documents, and union/variant work can accidentally invent a
second lifecycle, qualification, or composition model.

### Known inclusions

- Ordinary named type declaration and body organization.
- Stored members, instance functions, unbound/type-owned functions, nested
  declarations, and other non-storage body contributions.
- Type identity, aliases, incomplete self-names, forwards, recursion, size, and
  layout pressure.
- Default/direct construction, generated lifecycle operations, replacement,
  destruction, and member order.
- Qualification, transfer, reference origin, member-place stability, and
  composition boundaries.
- Anonymous types where concrete evidence is sufficient.
- Unmanaged `union` storage, overlapping lenses, active resident-instance
  obligations, unsafe boundaries, alignment, and destruction.
- Managed `Variant` alternatives, discriminant/presence state, active
  alternative paths, construction, access, transfer, replacement, and
  destruction.
- Variant interaction with `switch`, exhaustiveness, and future pattern
  matching to the depth needed for a coherent current model.
- `outer tracked` declaration, hidden representation, placement relationship,
  replacement/copy/destruction effects, and outer-cast integration.
- Costs, diagnostics, source stability, terminology, navigation, legacy
  disposition, and documentation ownership.
- Complete disposition of the maintainer notes, reached raw input, and useful
  legacy `type-definition.md` material.

### Known starting boundaries

- Complete type-parameter/generic syntax, constraints, and specialization.
- A general pattern-matching language beyond concrete managed-variant pressure.
- Complete FFI, ABI, stable layout contracts, foreign unions, and calling
  conventions.
- General reflection APIs and metadata schema.
- Complete partial/open type extension authority.
- Compiler data structures, lowering, tagging algorithms, niche optimization,
  and code generation except where programmer-visible behavior or cost
  constrains them.

These boundaries do not erase consequences. Preserve every material pressure in
a current owner or indexed raw destination rather than leaving it only in the
eventual archive of this work item.

### Intentionally unresolved framing

- Whether ordinary type teaching and union/variant teaching belong in one
  lasting owner or a cohesive type owner with specialized linked owners.
- Exact `union` and `Variant` declaration/source forms.
- Whether unmanaged union access ever has a safe active-member path or always
  requires explicit unsafe responsibility.
- Variant default state, alternative naming, recursive alternatives, and
  selection/binding source.
- Which lifecycle operations are generated for unions and variants.
- How anonymous type identity interacts with named type definitions.
- Exact placement and spelling of `outer tracked` among type-declaration
  properties.
- Which accepted facts are conceptual language guarantees and which remain
  future formal layout, reflection, ABI, or implementation contracts.

### Initial stopping guidance

Stop when the work has:

- established a cold-reader-friendly ordinary type-definition model;
- defined representative ordinary, recursive, composed, outer-tracked, union,
  and managed-variant source;
- reconciled declaration, identity, stored shape, construction, lifetime,
  qualification, transfer, selection, safety, diagnostics, and cost behavior;
- separated managed alternatives from unmanaged overlapping storage;
- preserved generic, pattern, reflection, ABI/FFI, partial-type, and
  implementation deferrals in live owners;
- dispositioned every useful maintainer-note, raw, and reached legacy finding;
  and
- completed the required documentation-fit dry run.

Do not promote findings, archive this work item, begin work item `029`, or
redesign complete generics, pattern matching, FFI/ABI, reflection, partial
types, or compiler implementation without the separately required discussion,
alignment, and authorization.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs focused reading,
  decision maturity, promotion, deferral, and closure.
- [Types, unions, and variants maintainer notes](../raw/types-unions-variant-maintainer-notes.md) -
  latest maintainer-supplied primary input; read and disposition completely
  after assignment.
- [Legacy type definition](../../type-definition.md) - principal legacy type,
  member, anonymous-type, function-member, and union evidence to disposition by
  value rather than authority.
- [Raw variants and unions](../raw/variants-and-unions.md) - preserves the
  managed-alternative versus unmanaged-overlay distinction and reached
  lifecycle/composition constraints.
- [Declarations and bindings](../../language/declarations-and-bindings.md) -
  owns `type` declarations, stored/member declarations, identity, forwards,
  incomplete self-names, aliases, and declaration qualification.
- [Construction, replacement, and destruction](../../language/construction-and-destruction.md) -
  owns complete/member lifecycle, generated operations, replacement, destruction
  order, and manual lifecycle boundaries.
- [Lifetimes and references](../../language/lifetimes-and-references.md) - owns
  life paths, instance places, resident instances, member renewal, references,
  and conditional-path consequences.
- [Qualifiers](../../language/qualifiers.md) - owns place, value, access, member,
  indirection, and unsafe-pliability qualification.
- [Transfer stances](../../language/transfer-stances.md) - owns `copy`, `deep`,
  `move`, `last`, projection, source state, and generated-operation pressure.
- [Composition](../../language/composition.md) - owns physical containment,
  `own`/`preferred`/`expose`, semantic-indirection boundaries, `outer tracked`,
  and outer casting.
- [Structural shapes and compatibility](../../language/structural-shapes-and-compatibility.md) -
  owns stored shape, flattening, layout, compatible/coercive views, and
  transformation.
- [Identity types](../../language/identity-types.md) - owns nominal identity,
  transparent aliases, admission/projection, and underlying boundaries.
- [Enums](../../language/enums.md) - supplies a nearby integer-backed identity
  model and selection/exhaustiveness pressure without deciding variants.
- [Optional values](../../language/optional-values.md) - supplies the current
  managed absent/present wrapper and boxed-life-path comparison.
- [Switch, case, and default](../../language/switch.md) - owns current runtime
  value selection, exhaustiveness, and active proof flow.
- [Raw pattern matching](../raw/pattern-matching.md) - preserves future
  destructuring and alternative-binding pressure reached by managed variants.
- [Safety and analysis](../../language/safety-and-analysis.md) - owns proof,
  unsafe assertions/permissions, and known-invalid lifecycle boundaries.
- [Source structure](../../language/source-structure.md) - owns body, brace,
  delimiter, spacing, and declaration/expression presentation.
- [Language-design terms](../../language/terms.md) - owns cross-cutting type,
  place, lifetime, identity, and safety vocabulary.

### Consequence-driven

- [Pointers, allocation, and arenas](../../language/pointers-and-arenas.md) when
  `outer tracked`, allocated types, erased ownership, alignment, or provenance
  consequences become concrete.
- [Function invocation](../../language/function-invocation.md) and
  [lambdas/callable composition](../../language/lambdas-and-callable-composition.md)
  when callable members, generated behavior, variant callbacks, or result
  routing becomes concrete.
- [Raw type parameters and generics](../raw/type-parameters-and-generics.md) when
  generic type definitions, recursive specialization, generic variants, or
  constraints become concrete.
- [Raw partial types](../raw/partial-types.md) when extension authority, stored
  member additions, or type completion becomes concrete.
- [Raw reflection](../raw/reflection.md) when type/member/alternative metadata,
  anonymous identity, or source reflection becomes concrete.
- [Raw interoperability](../raw/interop.md) when foreign layout, C-style unions,
  stable ABI, or representation contracts become concrete.
- Relevant legacy constructor, casting, union, enum, optional, or composition
  material only when a concrete question cannot be resolved from the primary
  notes, current owners, and focused raw inputs.

### Audit-only

- Archived work only when a concrete provenance or regression question cannot
  be answered from current owners and live raw input.

## Initial working posture

Begin with programmer-visible examples of:

1. an ordinary named type with stored members and one instance function;
2. a self-referential type using finite indirection;
3. a type using composition without flattening lifecycle;
4. an `outer tracked` type and one checked outer relationship;
5. an unmanaged union with its active-lifetime responsibility; and
6. a managed variant whose active alternative changes safely.

For each, state identity, stored shape, live paths, construction/destruction,
qualification, access, transfer, hidden state, and failure cost before proposing
one general taxonomy.

Use the maintainer notes as the latest primary evidence. Preserve current owner
constraints and treat legacy material as input rather than authority.

## Working record

Awaiting assignment. Creating and routing work item `028` does not authorize
analysis. The maintainer will complete the primary notes before handoff.
