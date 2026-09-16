# 024: Namespaces, imports, aliases, and forwarding

| Field | Value |
| --- | --- |
| Status | Active working material / non-normative / awaiting assignment |
| Work Item | `024` |
| Created | 2026-09-16 |
| Owns | The bounded review defined by the fixed initiating input |
| Does Not Own | Complete package acquisition, dependency solving, build caching, generic declarations, partial-type extension, compile-time execution, reflection, global/module initialization, dynamic loading, or implementation architecture |

## Non-authority notice

This file is a collaborative working record. Existing statements, new
observations, candidate wording, and aligned findings remain non-authoritative
until a separately discussed, aligned, and explicitly authorized promotion
incorporates them into their lasting owners.

## Fixed initiating input

This section records the information aligned when work item `024` was created.
It is intentionally incomplete and must not be rewritten as work develops.

### Initiating concern

Establish a coherent programmer-facing model for how Zax declarations become
nameable, available, and stable across lexical scopes, types, namespaces,
modules, imports, aliases, visibility boundaries, and incomplete/forward
declarations.

The
[namespace, aliasing, and forwarding maintainer notes](../raw/namespace-aliasing-forwarding-maintainer.md)
are primary maintainer-supplied input. The language maintainer will populate
that file before work item `024` begins. Read and disposition its complete
contents as the latest design evidence. It may add to, replace, supersede, or
obsolete any portion of the legacy corpus; retain legacy pages as evidence for
useful material the refreshed notes do not cover.

The indexed
[namespaces, imports, and visibility input](../raw/namespaces-imports-and-visibility.md)
preserves exact literal-operator alias identity, import-order independence,
collision, visibility, and forwarding pressure discovered by literal work.

### Motivating pressure

Current declaration design already establishes:

- one ordinary identifier namespace per lexical scope;
- deterministic nearest-scope root lookup;
- qualified suffix resolution only inside the selected root;
- pending suffixes below incomplete declarations;
- explicit shadow permission;
- transparent type aliases as another name for one identity; and
- no source, declaration, import, or module order tie-break for current operator
  selection.

Several later concepts require a complete availability model:

```zax
MyNamespace.x'first' x'second'
// Both literal prefixes must independently identify one exact declaration.

MyType :: forward type
MyAlias :: alias type ExistingType
```

Literal merge needs exact aliases to preserve declaration identity. Optional
platform declarations such as `Legacy.WChar` need future static source to ask
whether a declaration is available without first failing ordinary name
resolution. Generics, partial types, compile-time generation, function aliases,
and eventual reflection all depend on stable answers to which declaration a
name denotes and who may expose or complete it.

This naming/module boundary should be established before compile-time source
selection, generics, partial extensions, and reflection build larger mechanisms
on top of it.

### Known assumptions

- The first component of a qualified path resolves once through ordinary lexical
  lookup. Failure of a later component never backtracks to another root.
- Source, declaration, import, module, or generated order does not silently
  resolve ambiguity.
- A transparent type alias preserves its target identity; an explicit identity
  declaration creates another identity.
- A reference binding or same-place declaration alias is not automatically a
  namespace/type/callable alias.
- An exact literal-operator alias preserves declaration identity strongly enough
  for qualified and unqualified segments to merge.
- Visibility determines eligibility before preference; inaccessible
  declarations do not compete as worse public candidates.
- A forward declaration categorizes an incomplete declaration and must
  eventually match one completion. It is not an implementation body, implicit
  import, or source-order preference.
- Platform-optional declarations create compile-time availability pressure but
  do not authorize reflection or conditional-compilation syntax here.
- Namespace and module lifetime/global initialization consequences remain
  visible without being completed by this item.

### Known inclusions

- Lexical, type-member, namespace, module, and imported name boundaries.
- Namespace identity versus module identity and file/package organization only
  where programmers need the distinction to predict lookup and visibility.
- Qualified paths, stable roots, pending suffixes, shadowing, and collisions.
- Import into a named namespace versus deliberate exposure into a surrounding
  namespace.
- Export/re-export, private/public eligibility, selective exposure, and
  diagnostics.
- Exact aliases for types, namespaces/modules, functions/callables, operators,
  and literal operators at the depth required to define declaration identity
  and lookup.
- The boundary among transparent aliases, compatible visible prototypes,
  references/same-place aliases, imported names, and new identities.
- Forward declarations for types, namespaces/modules, values, callables,
  operators, and literal operators when each category has a demonstrated need.
- Completion matching, dependency cycles, pending resolution, and errors when a
  declaration never completes.
- Literal merge through qualified/unqualified aliases.
- Operator/callable discovery and ambiguity consequences.
- Platform-optional declarations and the exact handoff to future static
  source-resolution/capability work.
- Source stability, formatting, diagnostics, tooling, and eventual reflection
  consequences.
- Complete disposition of the maintainer notes, focused raw input, and reached
  legacy alias/forward/namespace material.

### Known starting boundaries

- Package fetching, repository URLs, dependency-version solving, lockfiles,
  caches, generated files, and build-system execution.
- Complete module initialization, destruction order, startup concurrency, and
  global/`once` lifetime scheduling.
- Complete compile-time execution, speculative source-enclosure syntax, and
  static capability queries.
- Generic declaration, deduction, specialization, constraint, and alias-family
  syntax.
- Partial/open type authority, external member/operation extension, and
  cross-module layout evolution.
- Complete callable compatibility, function values, lambda capture, chaining,
  and dynamic dispatch.
- Reflection metadata shape, declaration enumeration, and runtime type
  information.
- Dynamic module loading or runtime name lookup.
- Compiler symbol tables, dependency-graph algorithms, mangling, object formats,
  and linker implementation.

These boundaries do not erase consequences. Preserve every material pressure in
a current owner or indexed raw destination rather than leaving it only in the
eventual archive of this work item.

### Intentionally unresolved framing

- Whether a namespace and module are distinct declaration kinds or one
  declaration with different source/build roles.
- Exact import/export/re-export and selective-exposure syntax.
- Which alias categories share one declaration form and which need structured
  type/callable/operator forms.
- Whether an exact callable/operator alias reuses one minted implementation or
  creates a new visible declaration contract.
- Which declaration categories benefit from explicit `forward`, and which can
  rely on dependency-directed pending resolution.
- Whether namespace/module cycles are always errors or only become errors when
  initialization, layout, or completion dependencies cannot be ordered.
- How an optional declaration is named inside a future non-failing static source
  query.
- Which module/name changes are source, binary, build-cache, or import-contract
  compatibility events.

### Initial stopping guidance

Stop when the work has:

- established a teachable mental model for scopes, namespaces, modules, imports,
  aliases, visibility, and forwarding;
- defined deterministic name and qualified-path resolution without
  order-dependent fallback;
- separated alias identity from reference binding, visible-prototype adaptation,
  and new identity declaration;
- defined import/export/re-export and collision behavior at programmer depth;
- defined useful forward categories, completion matching, pending resolution,
  and cycle diagnostics;
- reconciled literal/operator/callable alias identity and discovery;
- dispositioned optional declaration availability without prematurely designing
  compile-time execution or reflection;
- assigned module lifetime, build/package, generic, partial-type, callable, and
  implementation consequences to current or indexed future owners;
- dispositioned all useful refreshed and reached legacy material; and
- completed the required documentation-fit dry run.

Do not promote findings, archive this work item, begin work item `025`, or
design complete build/dependency management, generics, partial types,
compile-time execution, reflection, function values, or module initialization
without the separately required discussion, alignment, and authorization.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs focused reading,
  decision maturity, promotion, deferral, and closure.
- [Namespace, aliasing, and forwarding maintainer notes](../raw/namespace-aliasing-forwarding-maintainer.md) -
  will contain the maintainer's refreshed primary input before this work item is
  assigned; read and disposition it completely.
- [Raw namespaces, imports, and visibility](../raw/namespaces-imports-and-visibility.md) -
  preserves exact literal-alias, collision, qualification, visibility, and
  forwarding pressure already discovered.
- [Declarations and bindings](../../language/declarations-and-bindings.md) -
  owns current lexical namespace, qualified paths, pending suffixes, shadowing,
  visibility eligibility, forwarding, callable aliases, and declaration forms.
- [Identity types](../../language/identity-types.md) - owns transparent aliases
  versus distinct identity declarations.
- [Literal source and operators](../../language/literal-source-and-operators.md) -
  supplies exact alias identity pressure from qualified/unqualified literal
  merge.
- [Legacy aliases](../../alias.md), [forward declarations](../../forward.md), and
  [namespaces/imports](../../namespacing.md) - preserve the principal legacy
  syntax, use cases, and unresolved module/name input to disposition by value.

### Consequence-driven

- [Function invocation](../../language/function-invocation.md),
  [operators](../../language/operators.md), and the
  [operator catalog](../../language/operator-catalog.md) when callable/operator
  alias identity, visible contracts, lookup, or ambiguity becomes material.
- [Raw callable selection](../raw/callable-selection.md) when an exact alias,
  compatible visible prototype, minted implementation, or preference boundary
  cannot be separated locally.
- [Raw build and dependencies](../raw/build-and-dependencies.md) when module
  identity crosses into package acquisition, versioning, caching, source lists,
  generated files, or build execution.
- [Raw partial types](../raw/partial-types.md) when module ownership, external
  completion, or extension authority becomes concrete.
- [Raw compile-time execution](../raw/compile-time-execution.md) when optional
  declaration availability or a non-failing source-resolution enclosure becomes
  concrete.
- [Raw global and once lifetimes](../raw/global-and-once-lifetimes.md) when a
  namespace/module boundary constrains global construction, dependency ordering,
  destruction, or concurrency.
- [Raw type parameters and generics](../raw/type-parameters-and-generics.md) when
  generic aliases, specialization identity, or type-dependent availability
  becomes material.
- Relevant legacy module, compiler-directive, meta-type, meta-function, partial,
  or function material only when a concrete question cannot be resolved from
  the primary notes, required legacy pages, and current owners.

### Audit-only

- Archived work only when a concrete provenance or regression question cannot
  be answered from current owners and live raw input.

## Initial working posture

Begin with programmer source that introduces a namespace, imports or exposes one
name, creates an exact alias, resolves a qualified path, and forwards one
declaration. Explain which declaration each name denotes before designing module
loading, dependency graphs, or compile-time availability.

Treat the maintainer notes as the latest primary evidence once populated. Legacy
pages remain available for useful unrefreshed cases but do not override the
maintainer notes merely because they contain more text.

## Working record

Awaiting assignment and completion of the maintainer-supplied primary notes.
Creating and routing work item `024` does not authorize analysis.
