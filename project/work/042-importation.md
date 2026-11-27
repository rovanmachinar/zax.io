# 042: Importation

| Field | Value |
| --- | --- |
| Status | Active working material / non-normative / not started |
| Work Item | `042` |
| Created | 2026-11-26 |
| Owns | The bounded review of how importing works in a module: the one defining import statement per instance and how the rest of the module reaches it, and whether importation as a whole, including the import body and the settings it may hold, still holds together |
| Does Not Own | Package acquisition, versioning, and import-cycle identity (build and dependency work); exact export and re-export option syntax (export and visibility work); the meaning of `[<safe-verification>]` levels, which stays with safety and analysis; current module behavior, which stays with its owners until a promotion is authorized; the decision itself, which belongs to the language maintainer |

## Non-authority notice

This file is a collaborative working record. Existing documentation, candidate
syntax, examples, and later aligned findings remain non-authoritative until a
separately discussed, aligned, and explicitly authorized promotion incorporates
them into their lasting owners.

## Fixed initiating input

This section records the aligned information known when work item `042` was
created. It is intentionally incomplete and must not be rewritten as work
develops.

### Initiating concern

Inside a module, each imported instance is defined by exactly one import
statement, together with its body, and every other reference to that instance
in the module goes through that statement. Current documentation implies parts
of this but does not state or resolve it. To settle:

- how files earlier and later in module order reach the instance;
- whether `forward module` remains useful, or every reference must follow the
  import in source order;
- what other files see of a direct `:: import`, which has no forward form;
- where an `import` may appear: module root, namespace bodies, function
  bodies, type bodies.

The maintainer widened the item to a general review of importation, to make
sure it still holds together, including the `[<safe-verification>]` setting
that may now appear in an import body.

### Starting input

- The maintainer's candidate: when an import completes,
  `Module.<imported-name>` becomes bound to that import. The maintainer is
  open to this being a good idea or a bad one.
- The example recorded when the concern was found:

  ```zax
  // file: first.zax, earlier in module order
  shape : Geometry.Point            // reachable only through a forward, if at all

  // file: second.zax
  Geometry :: import Module.MyGeometryModule {   // the one defining import
  }

  // file: third.zax, later in module order
  origin : Geometry.Point           // the same instance, through the binding
  ```

- The current minimum statement in namespaces and modules: each instance has
  one defining import statement through which the rest of the module reaches
  it, and "exactly how earlier files, forwards, and direct exposure reach an
  instance remains future module work".
- The import body now holds two kinds of content: injected declarations,
  which become part of the imported module, and an unattached
  `[<safe-verification=...>]`, which is a setting on the instance and is not
  injected. The setting is the one exception to "directive state never
  crosses an import", justified because it changes only what the compiler
  reports.
- The maintainer is unsure how important `forward module` is here; it may be
  that an import must come before any other reference to it.

### Starting boundaries

- Generative imports stay: importing the same source twice creates two
  instances, each with its own defining statement and its own settings.
- Package acquisition, version selection, locking, and the identity used for
  import-cycle detection remain build and dependency work.
- Exact `private`, selective export, alias export, and re-export syntax
  remain export and visibility work, except where a finding here constrains
  them.
- What the `safe-verification` levels mean stays with safety and analysis;
  this item reviews only where and how the setting is written in an import.

### Initial stopping guidance

Per "The first deliverable is a set of decisions" in
[documentation architecture](../documentation.md#the-first-deliverable-is-a-set-of-decisions):
first identify the issues importation raises, and present a workable set of
decisions the maintainer can act on, each with concrete Zax examples, the
realistic options, what each costs, and a recommendation. Supporting evidence
and tensions sit underneath the decisions.

Creating this work item does not authorize analysis. A later assignment
begins it.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs focused reading,
  authority, promotion, deferral, closure, and the writing rules.
- [Zax namespaces and modules](../../language/namespaces-and-modules.md):
  "Start with a named import", "Every module has its own root", "Files
  contribute to one ordered module", "Named import, direct exposure, and
  collisions", "Generative imports and injection" with its subsections,
  "Visibility and export baseline", "Directive state stops at an import",
  "Namespace and module aliases", and "Forward module and namespace roots" -
  the current owner under review.
- [Zax declarations and bindings](../../language/declarations-and-bindings.md):
  "Forward anchors" and "Pending path suffixes" - how forwards complete, which
  the import question depends on.

### Consequence-driven

Read only the smallest relevant sections when the concern reaches them:

- [Zax compile-time code](../../language/compile-time-code.md), the
  `wellformed{...}` fallback for an injected import and the `Compilation`
  module queries, when injection or module identity is reached.
- [Zax partials](../../language/partials.md), including a partial through an
  import, when the import body's injected content is reached.
- [Zax generics](../../language/generics.md), "Module instances", when
  instance identity is reached.
- [Zax compiler directives](../../language/compiler-directives.md), "What
  bounds a range" and the catalog entry for `safe-verification`, and
  [Zax safety and analysis](../../language/safety-and-analysis.md), "How
  loudly findings are reported", when the import-body setting is reached.
- [Export and visibility directives](../raw/export-and-visibility-directives.md),
  when re-export or exposure of an import binding is reached.
- [Build execution and dependencies](../raw/build-and-dependencies.md), when
  import-cycle identity, acquisition, or versioning is reached.

### Audit-only

- Archived numbered work, only when a provenance question cannot be answered
  from current owners or raw input. The one-defining-import finding and the
  import-body setting came from
  `project/archive/work/041-compiler-verified-unsafe.md`; the original module
  design came from
  `project/archive/work/024-namespaces-imports-aliases-and-forwarding.md`.

## Working record

Not started.
