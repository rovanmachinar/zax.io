# Zax raw future-work input

| Field | Value |
| --- | --- |
| Status | Raw routing guidance / non-authoritative |
| Audience | The Zax language maintainer and agents assigned to targeted future work |
| Applies To | Evidence-backed future inputs without an established lasting owner |
| Owns | The index and reading posture for `project/raw/` |
| Does Not Own | Accepted Zax language design or the final documentation structure |

This is a temporary, indexed holding area. Its existence does not decide that
long-term Zax documentation should be organized by maturity.

This directory is not required during ordinary onboarding. Read or update a
specific raw file when a current work item names it or a concrete consequence
establishes its relevance. Do not scan the directory merely as general
background.

| Raw input | Future pressure | Intended disposition |
| --- | --- | --- |
| [Analysis controls](analysis-controls.md) | Unsafe semantic permissions/assertions, static-analysis contracts, language/compiler/community provenance, or lint suppression is designed | Move semantic guarantees and source controls into safety, diagnostics, tooling, or contract owners, then retire the input |
| [Asynchronous execution](async.md) | Async language documentation or lowering/runtime design begins | Consume through a subject-specific work item |
| [Build execution and dependencies](build-and-dependencies.md) | Build/dependency ownership or implementation contracts are designed | Consume through a subject-specific work item |
| [Legacy feature catalog](feature-catalog.md) | Existing language topics are inventoried or routed into accepted owners | Disposition each feature claim, then retire the catalog |
| [Function invocation](function-invocation.md) | Ordinary call syntax, argument mapping, parameter binding, defaults, overloads, or temporary lifetime is designed | Preserve or explicitly revise construction-derived constraints through function and lifetime owners, then retire the input |
| [Assembly and foreign interoperability](interop.md) | Assembly, FFI, ABI, or binding behavior is designed | Consume through a subject-specific work item |
| [Lifetime strategies](lifetimes.md) | Pointer/lifetime policy, reference origin, replacement aliasing, pointer preservation, move/copy, or `last` is reviewed | Consume through a subject-specific work item |
| [Mutability-indexed type families](mutability-indexed-type-families.md) | Type definitions, qualifier-indexed representation, structural compatibility, reflection, generics, or mutable/immutable conversion are reviewed | Evaluate distinct variants against separately named types and composition, then move accepted behavior or explicitly reject and retire the input |
| [Safety boundaries](safety.md) | Public guarantees, construction lifecycle diagnostics, manual lifecycle safety, panic behavior, or unsafe boundaries are designed | Consume through a subject-specific work item |
| [Language similarity guide](similarity.md) | Enough accepted Zax concepts exist to compare semantics accurately | Consume through a future numbered work item, then move or retire based on that work's documentation fit dry run |
| [Structural typing](structural-typing.md) | Structural identity, equivalence, layout, conversion, or subtyping is reviewed | Treat the preserved agent-authored proposal as candidate input, then retire it |
