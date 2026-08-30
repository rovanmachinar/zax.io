# Raw input: legacy feature catalog

| Field | Value |
| --- | --- |
| Status | Raw legacy inventory / non-authoritative |
| Audience | Future numbered work inventorying or refining Zax concepts |
| Applies To | Feature claims formerly presented on the website entry point |
| Owns | Preservation of the legacy catalog for later disposition |
| Does Not Own | Accepted feature commitments, semantics, or documentation routes |
| Source / Provenance | Legacy `index.md` feature catalog |

## Reading posture

This catalog preserves proposed capabilities without accepting them. Each item
must be evaluated through its applicable concept owner or future numbered work.

Do not use this file as onboarding or as evidence that a feature is committed.

## Preserved catalog

### Low-level features

- Raw memory access
- Raw memory casting
- Bitwise operations
- Sized types and alignment
- Manual memory allocation and deallocation
- Memory clear and memory copy
- Custom allocators
- Collective allocation and deallocation
- Raw pointers and references
- Uninitialized memory type control
- Direct type casting
- Allocation versus initialization control
- Union types
- Structure-of-arrays and array-of-structures support

### Type and flow-control features

- Familiar flow controls
  - `if`, `else`
  - `switch`, `case`, `default`
  - `while`, `until`
  - `redo while`, `redo until`
  - `each in`
  - `each from`
- Scope control and logic grouping with `break` and `continue`
- Arrays
- Explicitly sized integers, floating points, enums, booleans, and strings
- Enums with underlying types
- Locally defined variables
- Locally defined types

### High-level features

- Sized arrays, including strings
- `unique`, `own`, `strong`, `weak`, `handle`, `hint`, `discard`, and `collect`
  memory policies
- Default type initialization
- Optional function arguments
- Multiple function results
- Polymorphic types and functions
- Mutable and immutable types
- Operator overloading
- Constructors and destructors
- Type default-value initialization, including allocation
- Lambda functions with capture
- Member functions
- Namespace import control
- Source-repository module imports
- Shared module compilation control
- Source preservation through asset and module management
- Library and compiler separation
- Type composition with inner and outer casting
- Safe conversion using `as`
- Selective runtime type information
- End-of-scope `defer` execution
- Private and non-exported types
- Type and variable aliases
- Partial types
- Anonymous types
- `nothing` type instances
- Value polymorphism for functions

### Asynchronous programming

- Asynchronous function calling
- Parallel and sequential allocators
- Lazy type evaluation
- Promises
- Tasks
- Coroutines

### Metaprogramming

- Enum metadata
- Compile-time `if` and `else`
- Custom compile-time literals
- Variadic functions
- Explicit meta-function parameters
- Compile-time reflection
- Compile-time execution
- Compile-time generation
- Compile-time checking
- Compile-time type metadata and traits
- Language-integrated build control

### Other proposed features

- Usage deprecation
- No header files
- Runtime panics with selective disabling
- Harmonized warning-suppression mechanisms across compilers

## Recorded dispositions

Focused construction, invocation, and core-flow work has dispositioned these
catalog entries:

| Catalog input | Disposition |
| --- | --- |
| Constructors and destructors | Current conceptual behavior is owned by [Zax construction, replacement, and destruction](../../language/construction-and-destruction.md) |
| Default type initialization | Declaration forms are owned by [Zax declarations and bindings](../../language/declarations-and-bindings.md); member and lifecycle behavior is owned by the construction owner |
| Allocation versus initialization control | Storage-versus-value lifecycle separation is owned by the construction owner; complete allocation policy remains legacy/future work |
| Uninitialized memory type control | `unsafe ???` declaration behavior is owned by declarations and bindings; construction interaction is owned by the construction owner |
| Optional function arguments | Current conceptual behavior is owned by [Zax function invocation](../../language/function-invocation.md) |
| Multiple function results | Current result slots, completion, acknowledgement, and routing are owned by the function-invocation owner |
| Polymorphic functions | Current fixed-arity callable viability and partial-order preference are owned by the function-invocation owner; generics, value polymorphism, variadics, and operator-specific lookup remain future work |
| Member functions | Current receiver-call behavior is owned by the function-invocation owner and receiver qualification by [Zax qualifiers](../../language/qualifiers.md); complete `once`, capture, and reassignment behavior remains future work |
| Result discard acknowledgement | Current caller acknowledgement and omission behavior is owned by the function-invocation owner; unrelated local, type, and memory-policy discard remains future input |
| Harmonized warning-suppression mechanisms across compilers | Preserved for future work in [raw analysis-control input](analysis-controls.md) |
| Familiar flow controls (`if`, `else`, `while`, `until`, `redo while`, `redo until`) | Current conceptual behavior, including `forever` and explicit `scope`, is owned by [Zax core flow control](../../language/core-flow-control.md) |
| Scope control and logic grouping with `break` and `continue` | Current `break`, `continue`, `next`, and flow-label targeting are owned by [Zax core flow control](../../language/core-flow-control.md) |
| `switch`, `case`, `default` | Deferred to future selection work; preserved with the `case next` consequence in [raw selection input](selection.md) |
| `each in`, `each from` | Iteration protocols remain future iteration work; core flow no longer depends on their details |
| Operator overloading | The shared model is owned by [Zax operators](../../language/operators.md), the cohesive word-spelled feature by [operator phrases](../../language/operator-phrases.md), exact forms by the [operator catalog](../../language/operator-catalog.md), and tree-pattern behavior by [mixfix operators](../../language/mixfix-operators.md); literal, numeric-family, composition, indexing, allocation, and pointer remainder is routed to focused live inputs |
| Bitwise operations | Current symbolic, phrase, shift, rotate, count, set-bit mask/position, reduction, and mutation forms are owned by the [operator catalog](../../language/operator-catalog.md); indexing/slicing and numeric-family remainder, including exact multiword, reversal, and masked extraction/deposit words, is routed to [indexing and slicing](indexing-and-slicing.md) and [numeric type families](numeric-type-families.md) |
| Explicitly sized integers, floating points, enums, booleans, and strings | Enum identity, validity, and operation questions are routed to [enum types](enum-types.md); remaining numeric representation questions stay with [numeric type families](numeric-type-families.md) |
| Enums with underlying types | The four generated enum operations are owned by the [operator catalog](../../language/operator-catalog.md); endian semantic enum behavior by [Zax endianness](../../language/endianness.md); complete enum behavior is routed to [enum types](enum-types.md) |
| Enum metadata | Routed to [reflection](reflection.md) for metadata shape and to [enum types](enum-types.md) for the enum-side questions |
| Polymorphic types | Type parameters, generic declarations, constraints, and generated type families are routed to [type parameters and generics](type-parameters-and-generics.md) |
| Explicit meta-function parameters | Type parameter slots and type arguments at the shared callable depth are owned by [Zax function invocation](../../language/function-invocation.md) and [Zax declarations and bindings](../../language/declarations-and-bindings.md); complete generic syntax is routed to [type parameters and generics](type-parameters-and-generics.md) |
| Compile-time execution | Routed to [compile-time execution](compile-time-execution.md); the accepted narrow statement is that execution is directed and inferred and that a type-receiver operation is not inherently compile-time |
| Compile-time `if` and `else`, compile-time generation, compile-time checking | Routed to [compile-time execution](compile-time-execution.md) |
| Compile-time reflection, compile-time type metadata and traits | Routed to [reflection](reflection.md); the currently reserved concepts are recorded by the [operator catalog](../../language/operator-catalog.md#reserved-phrase-forms) |
| Custom compile-time literals | Routed to [literals and literal operators](literal-operators.md), which now also carries the settled single-quote phrase-fence coordination |
| Selective runtime type information | Routed to [reflection](reflection.md) |
| Safe conversion using `as` | `as` and `unsafe as` are open language-defined phrase forms owned by the [operator catalog](../../language/operator-catalog.md#as-and-unsafe-as); complete conversion semantics remain legacy casting and future casting work |
| Anonymous types | Arithmetic-report, general anonymous structural, and storage/shape compatibility pressure is preserved in [raw structural typing](structural-typing.md) |
| Lambda functions with capture | Composition, chaining, generated callable, and capture pressure is preserved in [function composition and chaining](function-composition-and-chaining.md) |
| Partial types | Mixfix ownership, external phrase extension, and protected-signature pressure is preserved in [partial type extensions](partial-types.md) |
| Library and compiler separation | Required language support, including the protected `Boolean` operations, is distinguished from optional library APIs by the [Zax language vision](../../language/vision.md) and [Zax operators](../../language/operators.md); toolchain, helper-linkage, and incomplete-toolchain pressure is preserved in [raw build and dependency input](build-and-dependencies.md) |
| Value polymorphism for functions | Remains future value-polymorphism work; core flow constrains only the conditional expression's per-operation convergence |

These dispositions do not accept the remaining catalog items.

## Activation and retirement

Use this file when inventorying existing concepts or checking whether legacy
feature claims have received dispositions. Retire it after every retained item
has an accepted owner, explicit deferral, or discarded status.
