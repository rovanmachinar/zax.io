# 003: Declarations, bindings, and assignment

| Field | Value |
| --- | --- |
| Status | Active working material / non-normative |
| Work Item | `003` |
| Created | 2026-08-20 |
| Owns | The bounded review defined by the fixed initiating input |
| Does Not Own | Accepted declaration syntax, formal grammar, or compiler behavior |

## Non-authority notice

This file is a collaborative working record. Existing statements, new
observations, candidate wording, and aligned findings remain non-authoritative
until a separately discussed, aligned, and explicitly authorized promotion
incorporates them into their lasting owner documents.

## Fixed initiating input

This section records the information known and aligned when work item `003` was
created. It is intentionally incomplete rather than exhaustive and must not be
rewritten as the work develops.

### Initiating concern

Develop a coherent human-developer-facing model for how Zax introduces names and
distinguishes declaration, initialization, assignment, and build-time or
type-level binding.

The review must determine the intended roles and relationships of:

- `name : Type`;
- `name : Type = value`;
- `name := value`;
- assignment with `=`;
- declarations using `::`;
- declarations without an immediate value;
- scope entry and visibility of a newly declared name;
- repeated declaration, shadowing, and accidental assignment to the wrong name;
- declaration statements within the accepted statement and body model;
- documentation attachment to declarations; and
- diagnostics and formatting that make declaration intent visible.

Use the review to determine whether the proposed punctuation earns its
unfamiliarity and whether its forms remain understandable in ordinary code.

### Motivating pressure

Declarations occur throughout the legacy corpus, but their punctuation and
meaning remain unreviewed. Accepted source structure now explains how statements
and bodies are formed without defining what declaration statements mean.

Several later concerns cannot be reviewed coherently until declaration and
assignment are distinct:

- flow-control initialization;
- parameters and function results;
- type and member declarations;
- inference;
- build-time bindings;
- shadowing and scope;
- structural typing; and
- initialization and lifetime.

Promoting other concept pages first would risk treating legacy declaration
punctuation as accepted merely because examples use it.

### Known assumptions

- The current [language vision](../../language/vision.md) is accepted
  foundational direction.
- [Zax source structure](../../language/source-structure.md) is accepted
  conceptual design.
- Statement-level newlines, semicolon composition, blocks, clause scope, layout
  validation, and documentation attachment retain their accepted meanings.
- Existing declaration syntax is legacy design input rather than authority.
- The review should establish a programmer-facing model before formal grammar
  or compiler behavior.
- Familiarity, readability, visible intent, and explicit cost remain evaluation
  pressures.
- A declaration model may expose consequences for types, lifetimes, functions,
  and flow control without prematurely designing those entire concerns.

### Known inclusions

- The distinction among declaration, initialization, and later assignment.
- The conceptual role of `:`, `:=`, `::`, and `=`.
- Explicitly typed versus inferred declarations.
- Declarations with and without initial values.
- When a declared name becomes visible.
- The relationship between a declaration and its enclosing source scope.
- Whether declarations are only statements or may occur in other syntactic
  positions.
- Shadowing, redeclaration, and mistaken assignment at the depth needed to make
  the model safe and readable.
- Declaration formatting and mandatory layout consequences.
- Leading and trailing documentation attachment to declarations.
- Immediate diagnostic expectations.
- Candidate lasting owner and reading-path boundaries.

### Known starting boundaries

- Exact structural-type identity, equivalence, layout, conversion, and
  subtyping.
- The complete type-inference algorithm.
- Detailed function parameter, result, overload, and invocation behavior.
- Multiple-value declarations and destructuring unless they expose a
  contradiction in the singular model.
- Detailed mutability, `final`, constant, and build-time execution semantics.
- Allocation, construction, destruction, ownership, and lifetime strategies.
- Module export, import, visibility, and namespace resolution.
- Detailed flow-control header semantics, including acceptance or rejection of
  `;;`.
- Formal grammar, parser data structures, lowering, or implementation.

These areas are not part of the initiating question and need not be explored to
begin. They do not prohibit following, recording, or dispositioning material
consequences.

### Intentionally unresolved framing

- Whether `:` is best understood as type ascription, declaration, or part of a
  larger declaration form.
- Whether `:=` provides enough readability or inference value to justify a
  distinct operator.
- Whether `::` represents one coherent category or currently combines unrelated
  concepts.
- Whether declaration without initialization is ordinary, restricted, or
  explicitly unsafe.
- Whether `=` has the same role during initialization and later mutation.
- Whether a declaration's scope begins before or after its initializer is
  evaluated.
- Whether shadowing is prohibited, restricted, or allowed with mandatory
  explicitness.
- Whether declarations can appear wherever statements appear or only in
  selected statement positions.
- Whether type and value declarations belong in one lasting owner or eventually
  separate after sharing a common binding model.

### Initial stopping guidance

Stop when the work has:

- established an aligned programmer-facing model for declaration,
  initialization, and assignment;
- dispositioned `:`, `:=`, `::`, and `=` at the necessary conceptual depth;
- established when a declared name becomes visible;
- established the relationship with source scopes and statements;
- dispositioned declaration without initialization;
- addressed shadowing and redeclaration enough to prevent hidden ambiguity;
- identified immediate formatting and diagnostic consequences;
- captured constraints on functions, types, flow headers, lifetimes, and
  structural typing without designing them prematurely;
- identified one plausible lasting owner and its boundaries; and
- gathered enough information for a pre-promotion documentation fit dry run.

Do not promote findings, update language examples broadly, archive this work
item, or begin work item `004` without the separately required discussion,
alignment, and authorization.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs ownership,
  focused reading, numbered work, and promotion.
- [Language vision](../../language/vision.md) - provides the accepted
  readability, familiarity, visibility, and explicit-cost principles.
- [Zax source structure](../../language/source-structure.md) - provides the
  accepted statement, body, scope-visible, layout, and documentation framework
  within which declarations must fit.
- [Basics: operators](../../basics.md#operators) - preserves the focused legacy
  punctuation roles proposed for assignment and explicit declaration.
- [Basics: type declaration](../../basics.md#type-declaration) - contains the
  remaining declaration forms and examples, including inferred declaration,
  that initiate this review.

### Consequence-driven

- Read a focused section of [scope](../../scope.md) when declaration visibility,
  shadowing, or scope entry requires legacy evidence.
- Read focused [type-definition](../../type-definition.md) material only when
  `::` or member declaration cannot be evaluated from the basics material.
- Read focused function material only when parameter or result declarations
  expose a constraint on the general declaration model.
- Read focused flow-control initialization examples only when deciding whether
  declarations can occupy special header positions.
- Read namespacing or module material only when file-level declarations differ
  materially from local declarations.
- Read mutability, construction, lifetime, or allocation material only when
  declaration without initialization cannot be dispositioned without it.
- Read the [raw structural-typing input](../raw/structural-typing.md) only if
  declaration shape crosses into structural identity or equivalence.
- Propose an update or new raw input when a material finding must be preserved
  for future work. Discussion, alignment, and explicit edit authorization still
  apply.
- Inspect focused Git history only when the origin or former purpose of an
  unusual declaration choice becomes consequential.
- Expand beyond this list whenever discussion or a material finding establishes
  a concrete need. State why the additional reading is relevant and keep it
  focused.

Material not listed in this reading scope is neither required nor prohibited. It
remains available when discussion or a concrete consequence makes it relevant.

### Audit-only

- `project/archive/`, including work items `001` and `002`.

Audit-only material requires direct language-maintainer instruction or a
specific audit or recovery need.

## Working record

Not started. The agent creating this work item stops here without analyzing its
content.
