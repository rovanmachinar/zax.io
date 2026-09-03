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
| [Bare source enclosure](bare-source.md) | Keyword-neutral source, confusable-form intent, transparent source enclosures, or source reflection is reviewed | Move accepted behavior into source, terminology, flow, and operator owners, then retire |
| [Build execution and dependencies](build-and-dependencies.md) | Build/dependency ownership, required language support and automatic toolchain/helper linkage, or implementation contracts are designed | Consume through a subject-specific work item |
| [Compile-time execution](compile-time-execution.md) | Compile-time execution, native/compiler-host/target execution context, constant availability, or compile-time failure is reviewed | Move accepted behavior into compile-time, invocation, diagnostics, and applicable domain owners, then retire |
| [CPU provider model](cpu-provider-model.md) | Native/software-emulated integer profiles, optimized maxima, target/compiler-host selection, machine-word relationships, or profile stability is reviewed | Move accepted behavior into target-profile, integer, build, reflection, and compatibility owners, then retire |
| [Cross-cutting audit](cross-cutting-audit.md) | A concern crosses several owners or a repository-wide sweep is needed | Disposition each entry individually as its owner consumes it; stays live across numbered-work archival and retires only when every entry is resolved |
| [Enum types](enum-types.md) | Enum members, validity, operation inheritance, safe creation, generic enum generation, or non-underlying enum conversion is reviewed | Move accepted behavior into an enum owner and the applicable operator, casting, numeric, and reflection owners, then retire |
| [Legacy feature catalog](feature-catalog.md) | Existing language topics are inventoried or routed into accepted owners | Disposition each feature claim, then retire the catalog |
| [Function composition and chaining](function-composition-and-chaining.md) | Function composition, chaining, captures, lambda/generated callable types, optional callable construction/combinators, callable generics, or reflection is reviewed | Move accepted behavior into function, invocation, operator, optional, selection, generic, or reflection owners, then retire |
| [Indexing and slicing](indexing-and-slicing.md) | Arrays, indexing, slicing, ranges, bit places, proxies, or indexed mixfix declarations are reviewed | Move accepted behavior into array, index, source, qualifier, lifetime, and operator owners, then retire |
| [Assembly and foreign interoperability](interop.md) | Assembly, FFI, ABI, or binding behavior is designed | Consume through a subject-specific work item |
| [Lifetime strategies](lifetimes.md) | Pointer/lifetime policy, reference origin, independent pointer/reference binding replacement or rebinding, replacement aliasing, pointer preservation, move/copy, or `last` is reviewed | Consume through a subject-specific work item |
| [Literals and literal operators](literal-operators.md) | Prefixed/custom literal parsing, declarations, payloads, typed/bit-pattern results, literal-only type sugar, juxtaposition, or compile-time literal execution is reviewed | Move accepted behavior into literal, numeric, source, compile-time, or result-type owners, then retire |
| [Mutability-indexed type families](mutability-indexed-type-families.md) | Type definitions, qualifier-indexed representation, structural compatibility, reflection, generics, or mutable/immutable conversion are reviewed | Evaluate distinct variants against separately named types and composition, then move accepted behavior or explicitly reject and retire the input |
| [Numeric type families](numeric-type-families.md) | Custom integer-family participation, signedness, fixed-point, unusual or unbounded width, count/associated types, reversal, extraction/deposit, multiword numeric behavior, alias, float, representation, or generic numeric behavior is reviewed | Move accepted behavior into numeric, type, literal, cast, generic, or operator owners, then retire |
| [Owned composition and exposed identity surfaces](owned-composition.md) | `own`, identity-aware exposure, signature substitution, same-storage identity views, representation casts, composition sugar, or filtering is reviewed | Move accepted behavior into composition, type, identity, declaration, operator, layout, lifetime, casting, and reflection owners, then retire |
| [Partial type extensions](partial-types.md) | Partial/open or sealed types, intrinsic/identity authority, extension methods, stored-member additions, external phrase/mixfix extension, or external completion is reviewed | Move accepted behavior into type, import, layout, identity, and operator owners, then retire |
| [Reflection](reflection.md) | Reflection, type or declaration metadata, source reflection, or reflection-dependent tooling is reviewed | Move accepted behavior into a reflection owner and the domain owners whose facts it exposes, then retire |
| [Safety boundaries](safety.md) | Public guarantees, construction lifecycle diagnostics, manual lifecycle safety, panic behavior, or unsafe boundaries are designed | Consume through a subject-specific work item |
| [Selection and matching](selection.md) | `switch`, `case`, `default`, or pattern-like selection is reviewed | Consume through a future selection work item, preserving the `case next` consequence, then retire |
| [Language similarity guide](similarity.md) | Enough accepted Zax concepts exist to compare semantics accurately | Consume through a future numbered work item, then move or retire based on that work's documentation fit dry run |
| [Structural typing](structural-typing.md) | Structural identity, equivalence, storage/shape compatibility, layout, conversion, truncating transfer, or subtyping is reviewed | Treat the preserved agent-authored proposal as candidate input, then retire it |
| [Type parameters and generics](type-parameters-and-generics.md) | Generics, type parameters, constraints, computed type results, type-receiver identity, or generated type families are reviewed | Move accepted behavior into generic, declaration, invocation, type, and reflection owners, then retire |
