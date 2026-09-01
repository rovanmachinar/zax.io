
# [Zax Programming Language](index.md)

## Basics

### Basics of parsing

Accepted source reading and layout are defined by
[Zax source structure](language/source-structure.md). It owns statement-level
newlines, explicit continuation, simple and composed statements, block and body
formation, mandatory layout validation, and ordinary, raw, and documentation
comments.

Accepted value declarations, initialization, binding visibility, qualifier
axes, name resolution, and assignment boundaries are defined by
[Zax declarations and bindings](language/declarations-and-bindings.md).

The accepted rule for when a spelling counts as a keyword is defined by
[Zax source structure](language/source-structure.md#contextual-keyword-recognition),
and the term itself by
[Zax language-design terms](language/terms.md#contextual-keyword).

This legacy basics page no longer defines those concerns. The remaining sections
preserve unreviewed design input for keywords, detailed operators, types,
literals, and related concepts.


### Keywords

Keywords are simple words that a compiler will intrinsically understands. Keywords are either a single word or a compound set of words. When a compound set of words is used then all of the words must be present in the correct sequence of the keyword is not matched.

````zax
alias
alias keyword
alias operator
alias type
await
break
case
case continue
catch
channel
continue
collect
readonly
copy
deep
default
defer
discard
each / from
each / in
except
false
forever
forward enum
forward enum value
forward operator binary
forward operator literal
forward operator post unary
forward operator pre unary
forward namespace
forward type
forward union
forward variable
handle
hint
if
if / else
immutable
import
writable
last
lazy
lease
managed
move
mutable
mutator
once
operator binary
operator literal
operator pre unary
operator post unary
override
own
unsafe pliable
private
promise
raw
redo until
redo while
replacement +++
return
scope
shallow
switch
task
true
type
union
unique
unpliable
until
using
varying
weak
while
yield
yield suspend
````


#### Keyword disambiguation

The accepted rule is that a spelling has keyword status only where that keyword's
construct is grammatically permitted; outside those positions the spelling is not
a keyword. That rule is owned by
[Zax source structure](language/source-structure.md#contextual-keyword-recognition).
The remaining material on this page is unreviewed legacy evidence for the
underscore convention.

Keywords are reserved only within the context of where the keyword is allowed and legal. To disambiguate keywords from variables, variables can be postfixed with an underscore (`_`) but otherwise the underscore postfix (`_`) should never be used. Usage of keywords as variables is discouraged and usage of postfix underscores are also discouraged.

`replacement` is contextual only when immediately followed by `+++` where a
constructor declaration is legal. In every other context it remains an ordinary
identifier and requires no underscore disambiguation.

````zax
// legal name because `readonly` is a keyword in some contexts
readonly_ : Type readonly

// foobar_ is not a legal name because it contains an _ postfix on a name
// that is not a keyword
foobar_ : Type readonly
````


### Operators

> This flat list is retained only as legacy evidence. The current programmer
> model is [Zax operators](language/operators.md), exact current forms are in the
> [operator catalog](language/operator-catalog.md), and tree-pattern behavior is
> in [mixfix operators](language/mixfix-operators.md). Literal, phrase,
> allocation, pointer, and metaprogramming entries below remain future subjects;
> the list does not make them current or available.

#### Remaining literal-operator evidence

The promoted symbolic, circumfix, call, index, and mixfix forms were removed from
this legacy list. The single quote is now the
[exact phrase fence](language/operator-phrases.md#exact-phrase-fencing) when it
stands alone and separated by whitespace; a literal single quote must be attached
to a literal prefix. Quote-delimited literal prefixes and custom literal
operations remain future literal work:

````
"                  // ordinary unprefixed literal delimiter
prefix'            // attached prefixed-literal delimiter evidence
'                  // standalone single quote is a phrase fence, not a literal
````


#### Historical non-overloadable proposals

The following unresolved forms remain evidence for pointer, allocation,
composition, chaining, and result-shape work. Their legacy classifications are
not current guarantees:

````
*                   // post-unary pointer type declaration
&                   // post-unary reference type declaration, or
                    // pre-unary capture by reference operator
@                   // unary/binary standard allocator operator (allocate using
                    // the standard allocator and construct type)
@@                  // unary/binary parallel allocator operator (allocate using
                    // the parallel allocator and construct type)
@!                  // unary/binary synchronous allocator operator (allocate
                    // using the synchronous allocator and construct type)
.                   // post-unary dereference operator
.                   // binary namespace resolution operator
>>                  // binary function composition
|>                  // binary function invocation chaining
->                  // post-unary argument combine operator (combine remaining
                    // function result arguments into a single automatically
                    // defined type)
<-                  // pre-unary argument split operator (split type into
                    // multiple function arguments)
````


#### Remaining historical operator-phrase input

Operator phrases are current design and taught by
[Zax operator phrases](language/operator-phrases.md). Exact forms and precedence
are in the [operator catalog](language/operator-catalog.md#operator-forms).

The list below retains only unresolved pointer, lifetime, allocation,
function-representation, and variadic proposals. Their exact words are not
reserved because those domains have not established their prototypes.

````
outer of             // binary outer type instance of operator (convert from
                     // contained `type` pointer to container `type` pointer
                     // safely via a managed type's RTTI)
lifetime of          // shared lifetime operator (binds a raw pointer to an
                     // existing `strong` or `handle` pointer and safely checks
                     // if the pointer to a type points to memory within the
                     // allocated `strong` or `handle` pointer)
unsafe outer of      // binary unsafe outer type casting operator (convert from
                     // contained type pointer to container type pointer)
unsafe copy as       // binary unsafe `Unknown` copy casting of a function
                     // pointer (treat an `Unknown` pointer as pointing to an
                     // instance of a casted function `type` and make a copy of
                     // captured function contents)
unsafe lifetime of   // binary unsafe shared lifetime casting operator (converts
                     // a raw pointer to share a lifetime with an existing
                     // `strong` or `handle` pointer)
count of             // pre-unary count of a variadic expression
count of             // pre-unary count of a type
overhead count of    // pre-unary overhead count operator (returns the total
                     // reference count for a `handle` / `hint`, or
                     // `strong` / `weak` pointer)
overhead as          // pre-unary overhead operator (obtains a pointer to the
                     // overhead information for a pointer, `own`, `handle`,
                     // `hint`, `strong`, or `weak` pointer or optional type)
overhead size of     // pre-unary overhead sizing operator (return the number of
                     // bytes overhead is needed for this type i.e. typically
                     // the size of a control block)
allocator of         // pre-unary allocator operator (returns the allocator
                     // instance used to allocate an instance)
````

A compiler has both a host and a target for compilation. Any compile-time code that evaluates on a compiler's host system may have different value sizing and alignments than that of a compiler's target system. For example, a host may operate on a 64-bit system but target may compile for a 32-bit compilation.

A context sensitive version of sizing exist as follows (where the sizing and alignment is determined by code evaluating at compile-time or runtime context):

````
size of
alignment of
offset of
overhead size of
is constant
````

A `host` version of byte sizing operators exist as follows (whose meaning mirrors the version with a `host` prefix):

````
host size of
host alignment of
host offset of
host overhead size of
is host constant
````

A `target` version of byte sizing operators exist as follows (whose meaning mirrors the version with a `target` prefix):

````
target size of
target alignment of
target offset of
target overhead size of
is target constant
````

> The `host`/`target`/context distinction above is preserved future input. Current
> documentation defines only that `size of`, `alignment of`, `offset of`, and
> `is constant` answer in the *active execution context*, and separately defines
> **native** as the environment whose execution semantics are currently in
> effect. The contextual variants are routed to
> [raw compile-time execution input](project/raw/compile-time-execution.md).


#### Other expressions

The promoted declaration, discard, call/index, block, construction, directive,
current-instance, and lifecycle forms were removed from this legacy list. The
remaining generic/variadic spellings are future evidence:

````
$               // pre-unary templated argument declaration
...             // unary variadic values (array of optional variable arguments)
$...            // unary variadic types (array of types of variadic values)
````


### Compiler directives

````
abi                 // forces a function on a `type` to use a specific ABI
                    // calling convention
align               // align contained types to a zero modulus address boundary
asset               // copy asset to built bundle
asynchronous        // indicates a function not normally considered to operate
                    // asynchronously may be performed asynchronously
compilation         // controls if the compilation context is the host or the
                    // target for system specific intrinsic sizing and endian
                    // encoding
compile             // a value must be defined as a compile-time constant
compiles            // if a code block that follows compiles then a `true` is
                    // replaced otherwise a `false` is replaced
concept             // declare a function as a compile-time check for
                    // input/output argument type checks within a meta-function
deprecate           // declare API sections as being deprecated
error               // cause an error in compilation
execute             // evaluates code blocks at compile-time
export              // make type, variable, and other declarations visible to
                    // module importation
inline              // tells compiler to generate a function call as inline code
                    // rather than as a call to function
likely              // indicates a code path is more likely to be executed (for
                    // compiler and CPU optimization)
location            // the code URL location as a string literal
lock-free           // disable lock generation around `once` values
panic               // control panic behavior in code generation
reserve             // reserve non-accessible unused space in a type
source              // loads a related source file
void                // declared a contained value occupies a location within a
                    // `type` without allocating space for the contained value
resolve             // controls when a declaration should resolve
tab-stop            // sets the tab-stop for the source that follows
time                // time of compile as a string literal
unlikely            // indicates a code paths is less likely to execute (for
                    // compiler and CPU optimization)
warning             // display a warning or control compiler warning behaviors
````

#### Compiler literal directives

````
compiler                 // compiler related literals
file                     // indicates the source name being compiled
function                 // current function being compiled
line                     // source line being compiled
module                   // module related literals
````


### Recommended naming conventions

````zax
// variables are recommended in lower camelCase and type names are recommended
// in upper CamelCase
variableName : TypeName

// functions are recommended in lower camelCase
funcName : ()() = {
    // ...
}

// function prototypes are recommended in upper CamelCase
FunctionPrototype :: alias type ()()

// scope names are recommended in lower_case_with_underscores
scope my_scope {
}

// namespaces are recommended in upper CamelCase
variableName : MyModuleName.SubType

// enum names and enum values are recommended in upper CamelCase
Fruit :: enum {
    Apple,
    Banana
}

// false abbreviations are highly discouraged
notGood : WrdsNotKnwnAbbr
vecList : VecIsShortForVector
````


### Type declaration

````zax
:: import Module.System.Types

// Use Pascal style variable declaration where the variable name is
// specified followed by the type
variableName : TypeName

// Types are declared using the `type` keyword
TypeName :: type {
    variableName : Integer
}

// A `type` can be assumed based on a value rather than requiring explicit
// declaration (note: two unique operators are below; the `:` and the `=`
// are not the same operator)
assumedType := funcReturningType()

// types can be deduced based on automatic type deduction from a variable
// instead of an explicit `type` name
originValue : Integer
valueBorrowsOriginalValuesType : originalValue

// symbols that start with _ are reserved for compiler and toolchain generated
// symbols and may contain additional underscores where needed (thus are
// entirely reserved and must not be used by a programmer as a prefix)
_reservedVariableName
_ReservedTypeName

// symbols ending with _ are discouraged except in the case of disambiguating
// keywords and word operators from symbols (and a compiler may lint these
// names)
as_ // e.g. `as` is a keyword, but `as_` is not a keyword.

MyType :: type {
    m_no := 0   // NOT recommended as this Zax is a data oriented and not an
                // object or classification oriented language with members

    _no := 0    // NOT recommended (Zax has built in disambiguation)
    no_ := 0    // NOT recommended (Zax has built in disambiguation)

    // This pointer for the current `type`'s instance is reserved as a single
    // underscore `_` and `type` variables can be distinguished from arguments
    // by using `_.value` versus a locally declaration of `value`.

    value : Integer

    function final : ()() = {
        value : Integer

        _.value = 1    // `MyType`'s `value` is set to `1`
        value = 1      // local `value` is set to `1`
    }
}
````


### Intrinsic types

> **Current integer disposition.** Fundamental finite integer types, canonical
> names and namespaces, role identities, exact/optional/narrowing conversion,
> representation, counts, sizes, and portability are now owned by
> [Zax integers](language/integers.md). Transparent aliases and explicit
> identity declarations using `admit`/`restricted` and `expose`/`opaque` are
> owned by
> [Zax identity types](language/identity-types.md).
>
> The promoted integer catalog has been removed from this legacy page rather
> than retained as duplicate teaching. The remaining block preserves unresolved
> non-integer intrinsic input.

````zax
// import the module system types into the global `Module` namespace
:: import Module.System.Types

unknown : Unknown   // used as a generic pointer type to an `Unknown` type
nothing : Nothing   // used as a generic type of `Nothing`
void : Void         // an alias of the `Unknown` type
boolean : Boolean   // A value representing `true` or `false` literals

// aliased or meta-types floats mapping to fixed size type equivalents
float : Float           // fastest precision float
                        // (minimum 16 bits, 32/64 is typical)
half : Half             // half precision float (minimum 16 bit)
single : Single         // single precision float (minimum 32 bit)
double : Double         // double precision float (minimum 64 bit)
quadruple : Quadruple   // quadruple precision float (minimum 128 bit)

// fixed size floats
f16 : F16
f32 : F32
f64 : F64
f128 : F128

// advanced legacy float-family input
Float $(BitCount = Cpu.Integer.Optimal) :: type { /*... */ }

// strings have a built-in `length` `mutator` and are extended ASCII by default
stringA : String = "hello"
stringB := "type is implied"

// other string encodings are supported
utf8String : Utf8String = utf8'© Snowman Industries (☃)'
wideString : WideString = w'hello'
````


### Intrinsic system literals

Current unprefixed integer behavior is owned by
[Zax integer literals and realization](language/integer-literals.md). Attached
single-quote versus separated phrase-fence tokenization is owned by
[Zax source structure](language/source-structure.md#phrase-fences-and-literal-coordination).

The former prefix catalog, encoding/escape candidates, numeric-base payloads,
custom literal operators, compile-time transformations, and merged literal
examples remain unresolved future input. They are no longer presented here as
current language behavior.

### Intrinsic Namespaces

The language defines a default namespace named `Module`. The namespace is the root namespace for all types relative to any current namespace. See [namespacing](namespacing.md) for more details.


### Hello World

A simple hello world example is illustrated below:

````zax
:: import Module.System.Io

main final [[execute=target]] : ()() = {
    out.writeLine("Hello world!")
}

main()
````

An explanation of each step of the code:

````zax
// import a module named `Io` defined under namespace `Module.System.Standard`
// and define all types as being directly injected into the current namespace
:: import Module.System.Standard.Io

// define a function which accepts no input arguments and returns no results
// but only allow this function to be executed on the target system at
// runtime; without a `[[execute=target]]` the `main` function would execute
// immediately during the compilation process rather than on the compilation
// target system;
main final [[execute=target]] : ()() = {
    // using the `out` variable defined in `Module.System.Standard.Io` call a
    // function named `writeLine` that outputs the value to the standard out
    out.writeLine("Hello world!")
}

// call the function at global scope to cause the function to execute (although
// the code will only execute on a target system since the function can only
// execute on the compilation target system)
main()
````

A few key differences about `main` compared to other languages:
1. A main entry point need not be named `main` but can be called anything desired
1. Calling a `main` function is required as a linker will not automatically presume a `main` function must exist
1. Defining a `main` function using the `[[execute=target]]` is necessary otherwise the `main` function will execute at compile-time
1. Multiple `main` entry points are legal and they will be executed in the order found (unless they cannot be resolved immediately for host compile-time execution)
1. No entry point is required at all if a module is a library or code is exported in another fashion
1. Imported modules with their own entry points to perform compile-time code testing during a compilation process on a host system is normal and to be expected
1. Reading command line arguments are specific concept of a `Standard` application (as the concept of command line arguments are not a universal concept to all systems)
1. Returning a result to a command line executable is done via a `mutator` as part of a `Standard` application (as the concept of a single integer return result is not universal to all systems)
