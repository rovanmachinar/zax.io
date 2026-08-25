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

Focused construction work has dispositioned these catalog entries:

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

These dispositions do not accept the remaining catalog items.

## Activation and retirement

Use this file when inventorying existing concepts or checking whether legacy
feature claims have received dispositions. Retire it after every retained item
has an accepted owner, explicit deferral, or discarded status.
