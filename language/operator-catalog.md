# Zax operator catalog

| Field | Value |
| --- | --- |
| Status | Current conceptual design |
| Audience | Human developers and tooling looking up recognized operator source forms |
| Applies To | Exact forms, fixity, precedence, association, reservation, and domain routing; not type-specific result semantics or a formal grammar |
| Implementation State | Not established by this repository |
| Owns | The closed symbolic and circumfix catalogs; exact language-defined phrase forms, including transfer-stance restatement, protected reset, cursor protocol, array capacity queries, iterable count, and outer-cast forms; implicit literal-source adjacency, explicit merge `<\|>`, value join `<+>`, and array compound join `<+>=`; array range/splice components; reserved allocation-initializer tokens; precedence and association; form reservation; compact protected-domain availability; generated immediate-underlying and enum forms; call/index/splice recognition; and deferred/unavailable forms |
| Does Not Own | Complete transfer semantics ([transfer stances](transfer-stances.md)); shared operator/callable selection ([operators](operators.md), [function invocation](function-invocation.md)); phrase use and presentation ([operator phrases](operator-phrases.md)); literal declarations, payloads, merge, and required compile-time execution ([literal source and operators](literal-source-and-operators.md)); string/character join domains ([strings and characters](strings-and-characters.md)); complete [iteration and cursor behavior](iteration.md); uncommitted integer behavior ([integer literals and realization](integer-literals.md)); or cohesive type-specific behavior such as [structural shapes and compatibility](structural-shapes-and-compatibility.md), [composition](composition.md), [optional values](optional-values.md), [integer operations](integer-operator-catalog.md), [fixed-point scalars](fixed-point-scalars.md), [floating-point scalars](floating-point-scalars.md), [identity types](identity-types.md), [enums](enums.md), and [endianness](endianness.md) |
| Source / Provenance | Legacy [basics](../basics.md) operator evidence, refined against current operator, phrase, mixfix, integer, identity, and endian design |

## How to use this catalog

This catalog answers:

- Is this source form recognized?
- What fixity does it have?
- How tightly does it bind?
- How does it associate?
- Is the form open, protected for a domain, reserved, custom-only, or
  unavailable?
- Which concept owner defines its selected type-specific behavior?

It does not say that every overload has the behavior suggested by a familiar
symbol:

```zax
myCustomLeft +! myCustomRight
```

A permitted custom overload may return any declared result shape. Protected
integer behavior for `+!` is defined separately by the
[integer operator catalog](integer-operator-catalog.md).

## Operator forms

Zax recognizes:

| Family | Representative forms |
| --- | --- |
| Arithmetic | `+`, `+?`, `+%`, `+\|`, `+!`, `delta`, `distance`, `full precision product` |
| Comparison | `==`, `!=`, `<`, `<=`, `>`, `>=`, `<=>` |
| Logical | `!`, `&&`, `\|\|`, `^^`, `logical nand`, `logical nor`, `logical xnor` |
| Bitwise | `~`, `&`, `\|`, `^`, `&~`, phrases, counts, masks, shifts, and rotations |
| Optional | `?value`, `value.`, `reset value`, `last value`, `move value` |
| Conversion/admission | `as`, `narrowing as`, `from`, `optional from`, `narrowing from`, `unchecked from`, `unsafe from`, `outer cast`, `tracked outer cast`, `unsafe outer cast` |
| Structural compatibility | `as shape`, `as flattened shape`, `as layout`, `as flattened layout`, safe/unsafe coercive `as`, `anchor`, `unsafe cast` |
| Structural mapping | `>-`, `-<`, `-<>-`, with result-routing and `reshape` integration |
| Transfer stance | `value as copy`, `value as deep`, `value as move`, `value as last` |
| Literal source/value | Source merge `<\|>` and value join `<+>` |
| Mutation | Compounds including array `<+>=`, increment/decrement, `~=`, and exact phrase mutations |
| Circumfix | `\|value\|`, `\|?value\|`, `\|!value\|`, `\|\|value\|\|` |
| Delimited/multi-part | Call, index, splice, and [mixfix](mixfix-operators.md) forms |
| Allocation initializer | `@`, `@!`, `@<`, `@!<`, with optional attached `@{...}` policy |

A type may overload a recognized form where its operand domain remains open.
Declarations cannot invent arbitrary punctuation or assign another precedence.

## Precedence and grouping

Higher levels bind first:

| Level | Forms | Grouping |
| --- | --- | --- |
| Literal source formation | Adjacent same-declaration payload segments and optional `<\|>` merge | Forms one payload before expression parsing |
| Enclosed/grouped | Parentheses and recognized circumfix forms | Delimiter-owned |
| Postfix/delimited | Call, index, member/dereference, post `++`/`--` families | Left-to-right chaining |
| Prefix | `?`, `!`, `~`, `~=`, `#...`, pre `++`/`--`, unary arithmetic policies | Separate applications require grouping |
| Multiplicative | `*`, `/`, `%`, `full precision product`, and accepted policy variants | Left |
| Additive | `+`, `-`, accepted policy variants, `delta`, `distance` | Left |
| Literal join | `<+>` | Left for join chains; group when mixed with another value-operator level pending final placement |
| Shift/rotate/composition | `<<`, `>>`, `>>>`, `<<<`, `<<%`, `>>%`, modulo-count phrases, multiword operations | Left |
| Ordinary phrase | Conversion/admission forms, reserved language phrases, newly declared phrases | Left |
| Relational | `<`, `<=`, `>`, `>=`, `<=>` | Ungrouped chaining is an intent error |
| Equality | `==`, `!=` | Ungrouped chaining is an intent error |
| Bitwise AND | `&`, `&~`, `bitwise nand` | Left |
| Bitwise XOR | `^`, `bitwise xnor` | Left |
| Bitwise OR | `\|`, `bitwise nor`, `bitwise or not` | Left |
| Logical AND | `&&`, `logical nand`, `logical and not` | Left |
| Logical XOR | `^^`, `logical xnor` | Left |
| Logical OR | `\|\|`, `logical nor`, `logical or not` | Left |
| Conditional | `condition ?? trueValue ;; falseValue` | Right-nesting |
| Assignment/compound | `=`, compounds including `<+>=`, exact phrase mutations | Right |
| Swap | `<<>>` | Left |

Every newly declared phrase uses ordinary phrase precedence. A language-defined
phrase may instead use its fixed family level, as `delta`, `distance`, logical,
bitwise, shift, and mutation phrases do.

Precedence belongs to the source form, never to the receiver type.

### Grouping-required comparisons

Ungrouped relational and equality chains are rejected:

```zax
myA < myB < myC   // error: grouping required
myA == myB == myC // error: grouping required
```

Explicit grouping may be valid when the intermediate result supports the next
operation.

## Fixity and adjacency

Symbolic whitespace presents fixity:

```zax
!myValue
myValue++
myLeft + myRight
```

The longest recognized token wins. Separate unary applications require grouping:

```zax
--myValue
-(-myValue)
!(!myValue)
```

A phrase component uses pre-unary, post-unary, or binary fixity:

```zax
type of myValue
myBits trailing set bit position
myLeft logical nand myRight
```

Recognition forms every structurally complete candidate rather than taking the
longest word sequence. Phrase use, grouping, fencing, and presentation are
defined by [Zax operator phrases](operator-phrases.md).

## Results and overloads

An operator may declare zero, one, or several results. A nested expression
requires one usable value:

```zax
mySum:, myCarry: = myCustomLeft + myCustomRight

myCombined := myCustomLeft + myCustomRight
// error when the selected operation has two mandatory results
```

Several results do not become an implicit tuple or anonymous structure. Shared
result routing belongs to [Zax function invocation](function-invocation.md).

## Protected domains

User code cannot claim a signature whose every operand belongs to a closed
intrinsic family, whether or not Zax currently supplies that exact operation.

| Family | Current routing |
| --- | --- |
| `Boolean` | Protected truth, equality, logical, assignment, and related forms are summarized below |
| Exact integers | Protected availability and results are in the [integer operator catalog](integer-operator-catalog.md) |
| Exposed integer identities | Receive the applicable identity-adjusted protected integer surface |
| Pointer-representation integers | Follow the applicable integer identity; pointer objects remain separate |
| Floating point | Protected format-specific behavior is owned by [floating-point scalars](floating-point-scalars.md) |
| Fixed-point | Protected coefficient/scale behavior is owned by [fixed-point scalars](fixed-point-scalars.md) |
| Unbounded numeric types | Domain reserved pending focused numeric review |
| Other closed intrinsic families | Reserved where focused type work has not established behavior |
| Extensible library families | Library-owned; recognition does not by itself close a type such as `String` |

Aliases retain protection only when they retain the underlying identity. Missing
required protected support means the toolchain is incomplete, not that a program
failed to link an optional library.

## Boolean forms

### Symbolic forms

| Operation | Form | Exact Boolean evaluation |
| --- | --- | --- |
| Identity/status | `?myValue` | Return the Boolean value |
| Negation | `!myValue` | Return its opposite |
| AND | `myLeft && myRight` | Skip right when left is false |
| OR | `myLeft \|\| myRight` | Skip right when left is true |
| XOR | `myLeft ^^ myRight` | Evaluate both |

Other valid operand shapes are ordinary eager overloads and may return another
result shape.

### Exact logical phrases

| Operation | Exact form | Level | Exact Boolean evaluation |
| --- | --- | --- | --- |
| NAND | `logical nand` | Logical AND | Skip right when left is false |
| AND-NOT | `logical and not` | Logical AND | Skip right when left is false |
| NOR | `logical nor` | Logical OR | Skip right when left is true |
| OR-NOT | `logical or not` | Logical OR | Skip right when left is true |
| XNOR | `logical xnor` | Logical XOR | Eager |

Exact Boolean AND/OR-derived forms short-circuit and form mixfix-consumption
barriers. Exact Boolean XNOR is eager. Permitted custom implementations are
ordinary eager operations at the form's fixed level.

Exact phrase mutation forms use assignment precedence:

```zax
myFlag logical and assign myRight
myFlag logical nand assign myRight
myFlag logical and not assign myRight
myFlag logical or assign myRight
myFlag logical nor assign myRight
myFlag logical or not assign myRight
myFlag logical xor assign myRight
myFlag logical xnor assign myRight
```

### Complement fallback

Fallback exists only for:

```text
?  <-> !
== <-> !=
<  <-> >=
<= <-> >
```

Direct selection happens first. Only absence of an applicable direct operation
attempts the counterpart. The counterpart must resolve directly and return
exactly `Boolean`; the language then applies protected Boolean negation.

Fallback never swaps operands, combines comparisons, recurses, or bypasses an
ambiguous/unavailable direct candidate.

## Integer form menu

This section records exact integer-related forms and their levels. Complete
protected behavior is in the
[integer operator catalog](integer-operator-catalog.md).

Plain arithmetic, mathematical shifts, and width-invariant binary bitwise forms
can be used before number literals have selected a width:

```zax
myByte : U8 = 200 + 55
myShifted := 1 << 3
myBits := 55 ^ 77
```

Complement is available before commitment only when value and sign intent leave
one result at every fitting width. Zero-fill shift, rotation, counts, mutation,
and policy/reporting forms need a concrete integer type:

```zax
myComplement := ~1 // error: signed and unsigned results differ
myRotated := 1 <<% 2 // error: no concrete width
```

[Zax integer literals and realization](integer-literals.md) defines that
boundary. This catalog continues to own the exact forms and precedence.

### Arithmetic and mutation forms

```text
Required:          +   -   *   /   %
Optional:         +?  -?  *?  /?  %?
Wrapping:         +%  -%  *%
Saturating:       +|  -|  *|
Combined report:  +!  -!  *!
Wrapped report:   +!% -!% *!%
Saturated report: +!| -!| *!|
```

Compound forms append `=` where defined.

Pre/post increment and decrement recognize:

```text
Increment: ++  ++?  ++%  ++|  ++!  ++!%  ++!|
Decrement: --  --?  --%  --|  --!  --!%  --!|
```

### Signedness, exact difference, and distance

For a concrete integer, pre-unary `+` and its policy variants request the
signedness counterpart. For an uncommitted integer, plain pre-unary `+` toggles
sign intent before realization; policy variants require commitment.

`delta` and `distance` are exact language-defined binary phrases at additive
precedence.

### Fixed-point full-precision product

`full precision product` is an exact language-defined binary phrase at
multiplicative precedence:

```zax
myFull := myLeft full precision product myRight
```

It applies to like fixed-point identities and returns the associated widened
type able to represent the complete coefficient product and doubled fractional
precision. Complete result and arithmetic behavior is defined by
[fixed-point scalars](fixed-point-scalars.md#full-precision-product).

Fixed-point `%` uses the ordinary multiplicative remainder form with like
fixed-point operands. Its quotient truncates toward zero, its result retains the
operand identity, and a zero divisor fails. Complete behavior is defined by
[fixed-point scalars](fixed-point-scalars.md#remainder).

### Bitwise forms

```text
~   ~=   &   |   ^   &~
&=  |=   ^=  &~=
```

Exact bitwise phrases use their family levels:

```text
bitwise nand
bitwise xnor
bitwise nor
bitwise or not
```

Their direct mutation phrases append `assign`.

### Counts, masks, positions, and reductions

```text
#    #<   #>
#~<  #~>
#&   #|   #^
#~&  #~|  #~^
```

Exact post-unary position phrases:

```text
leading set bit position
trailing set bit position
```

### Shifts and rotations

```text
<<   >>   >>>   <<%   >>%
```

Compounds append `=`.

Exact modulo-count phrases use shift/rotate/composition precedence:

```text
shift left modulo
shift right modulo
logical shift right modulo
```

Their mutation forms append `assign`.

`<<<` and `<<<=` remain recognized custom-only forms with no protected basic
integer operation.

### Multiword and arrangement concepts

The following concepts are reserved for future numeric work:

- multiword logical/arithmetic/zero-fill shifts;
- multiword rotations;
- funnel shifts;
- writable multi-place forms;
- bit reversal;
- byte reversal;
- masked extraction; and
- masked deposit.

Exact multiword/reversal/extraction words are not established.

## Conversion and admission forms

These open language-defined forms use ordinary phrase precedence:

```zax
myConverted := mySource as DestinationType
myNarrow := mySource narrowing as DestinationType

myValue := MyIdentity from mySource
myOptional := MyIdentity optional from mySource
myNarrowIdentity := MyIdentity narrowing from mySource
myUnchecked := MyIdentity unchecked from mySource
myUnsafe := MyIdentity unsafe from mySource
```

`as`, `narrowing as`, and `unsafe as` use the left value for receiver discovery.
The right operand is a complete type argument.

Plain `as` may declare that type argument as a contextual anchor when the left
side is an uncommitted integer:

```zax
myByte := 55 as U8
```

This follows the shared contextual-completion mechanism rather than a
literal-specific rewrite. Protected optional and narrowing integer forms require
an already concrete left receiver.

`from`, `optional from`, `narrowing from`, `unchecked from`, and `unsafe from`
use the destination type identity as receiver.

Integer-specific behavior belongs to the
[integer operator catalog](integer-operator-catalog.md). General identity
admission/projection belongs to [Zax identity types](identity-types.md).

User-defined words do not independently grant unsafe authority.

### Structural compatibility forms

These exact safe forms are protected:

| Relationship | Exact source pattern | Deciding distinction |
| --- | --- | --- |
| Direct shape | `<source> as shape <DestinationType>` | Names and direct boundaries participate |
| Flattened shape | `<source> as flattened shape <DestinationType>` | Eligible containment flattens; leaf names participate |
| Direct layout | `<source> as layout <DestinationType>` | Names do not participate; direct boundaries remain |
| Flattened layout | `<source> as flattened layout <DestinationType>` | Eligible boundaries and names do not participate |

Posture restatement supplies the destination from context:

| Offered posture | Exact source pattern |
| --- | --- |
| Direct shape | `<source> as compatible shape` |
| Flattened shape | `<source> as compatible flattened shape` |
| Direct layout | `<source> as compatible layout` |
| Flattened layout | `<source> as compatible flattened layout` |

Every safe form preserves complete qualifications.

An optional trailing source anchor selects where one contiguous resident region
begins:

| Context | Exact source pattern |
| --- | --- |
| Written destination | `<source> as layout <DestinationType> anchor <source-path>` |
| Destination supplied by context | `<source> anchor <source-path>` |

`anchor` is not the numeric contextual type anchor, mixfix receiver anchor, or
pointer ownership anchor.

Coercive forms remain local and produce reference views:

| Relationship | Exact source pattern |
| --- | --- |
| Safe direct layout coercion | `<source> as coercive layout <DestinationType> &` |
| Safe flattened layout coercion | `<source> as coercive flattened layout <DestinationType> &` |
| Unsafe direct layout coercion | `<source> unsafe as coercive layout <DestinationType> &` |
| Unsafe flattened layout coercion | `<source> unsafe as coercive flattened layout <DestinationType> &` |
| Safe anchored coercion | `<source> as coercive layout <DestinationType> & anchor <source-path>` |
| Unsafe anchored coercion | `<source> unsafe as coercive layout <DestinationType> & anchor <source-path>` |

Safe coercion proves source-to-destination validity and, for writable views,
that destination writes preserve source validity. `unsafe` accepts an unproved
validity precondition or a restoration obligation after writes. Neither form is
a declarable posture or grants qualification authority. Complete behavior
belongs to
[structural shapes and compatibility](structural-shapes-and-compatibility.md#coercive-reference-views).

View-shaped raw casting is separate:

| Source role | Exact source pattern |
| --- | --- |
| Pointer | `<pointer> unsafe cast <DestinationType> *` |
| Value place | `<value> unsafe cast <DestinationType> &` |
| Reference | `<reference> unsafe cast <DestinationType> &` |

There is no bare by-value `<value> unsafe cast <DestinationType>` form.

The mapping-bound protected forms are:

| Operation | Exact source pattern |
| --- | --- |
| Decompose one source | `<destination mappings> >- <source>` |
| Recompose results | `<destination> -< <producer>` |
| Update an existing destination | `<destination> -<>- <source>` |
| Update through a reshape | `<destination> -<>- <reshape> -<>- <source>` |
| Construct through automatic transformation | `<destination context> = -<>- <source>` |
| Construct through a reshape | `<destination context> = -<>- <reshape> -<>- <source>` |
| Transform one named result | `<source-result label>: -<>- [<reshape> -<>-] <destination>` |
| Transform one positional result | `: -<>- [<reshape> -<>-] <destination declaration>` |

Multiple-result routing keeps its ordinary final
`= <producer>` boundary. Transformation belongs only to entries that contain
`-<>-`; it is never enabled for the complete result sequence.

These forms receive their grouping from declaration, call, and result-mapping
grammar rather than inventing ordinary arithmetic-expression precedence.

Complete semantics, visual mnemonic, costs, and failures belong to
[Zax structural shapes and compatibility](structural-shapes-and-compatibility.md).

### Outer-cast forms

These exact binary phrase forms use ordinary phrase precedence and are protected
composition operations:

```zax
proved : Container & =
  knownMemberReference outer cast Container.member
checked : Container & ? =
  uncertainMemberReference tracked outer cast Container.member
asserted : Container & =
  assertedMemberReference unsafe outer cast Container.member
```

The right operand is an exact resident stored-member path, not a type. User code
cannot overload any form. Plain `outer cast` requires selected-contract static
proof that every origin reaching this site is the exact path and produces a
non-optional result without tracking. `tracked outer cast` explicitly uses the
`outer tracked` placement capability and returns an optional result. If the
selected contract also proves that tracked operation's exact origin, retaining
it requires `intent<redundant-outer-tracking>`; compiler-private proof does not.
`unsafe outer cast` trusts the programmer's provenance assertion instead.
Complete behavior belongs to
[Zax composition](composition.md#outer-casting-to-an-immediate-container).

## Transfer-stance forms

These exact forms are reserved post-unary phrases at ordinary phrase precedence:

| Form | Result role |
| --- | --- |
| `value as copy` | Preserve the complete value shape and offer `copy` |
| `value as deep` | Preserve the complete value shape and offer `deep` |
| `value as move` | Preserve the complete value shape and offer `move` |
| `value as last` | Preserve the complete value shape and offer `last` |

The form itself performs no transfer or user-code invocation. A selected
consumer accepts the stance and owns any source effect.

Post-unary stance restatement is distinct from binary conversion:

```zax
converted := source as DestinationType
moved := source as move
```

Phrase recognition needs no exact phrase fence. Complete stance meaning and
fallback are defined by [Zax transfer stances](transfer-stances.md).

Repeated restatement groups from the left at ordinary phrase precedence:

```zax
result := source as move as copy
// (source as move) as copy; the consumer sees `copy`.
```

This is legal but normally pointless: no consumer observes the intermediate
`move`, so `as copy` replaces it without any transfer occurring.

## Optional and pointer lifecycle forms

These exact forms are protected for optional operands:

| Form | Fixity/level | Result role |
| --- | --- | --- |
| `?value` | Symbolic prefix | Return exactly `Boolean` presence |
| `value.` | Grammar-recognized postfix access | Produce boxed access after static presence proof |
| `reset value` | Pre-unary phrase at ordinary phrase level | Leave an optional absent, or release a pointer relationship and leave the pointer at `Nothing` |
| `last value` | Pre-unary phrase at ordinary phrase level | Produce the same optional type, offer `last`, and schedule optional payload cleanup at consumer completion |
| `move value` | Pre-unary phrase at ordinary phrase level | Produce the same optional type and offer `move` without scheduling wrapper absence |

Protected optional forms are distinct from generic post-unary stance
restatement. Complete optional behavior and source consequences are defined by
[Zax optional values](optional-values.md). Pointer `reset` is defined by
[Zax pointers, allocation, and arenas](pointers-and-arenas.md#resetting-a-pointer).

`T? ?` is two optional type layers and requires the separating space. Compact
`T??` contains the conditional-expression token. `[{}]` is the canonical
zero-entry construction packet; neither form is an independently overloadable
operator. Their source boundaries are owned by
[Zax source structure](source-structure.md#optional-layers-and-empty-construction-packets).

## Reserved phrase forms

A reserved phrase form cannot be declared by user code.

| Exact form | Receiver/shape | Broad behavior | Deferred refinement |
| --- | --- | --- | --- |
| `as default` | Post-unary type identity or value | Default-qualified type identity or compatible default-type value | Complete qualifier defaults, transfer, generics |
| `type of` | Pre-unary expression | Concrete selected static type without execution | Anonymous/qualified identity, reflection |
| `size of` | Pre-unary type receiver | Byte size in active execution environment | Complete layout/context |
| `alignment of` | Pre-unary type receiver | Required alignment in active environment | Complete layout/context |
| `offset of` | Binary member path and containing type | Byte offset to member path | Member-designator grammar/layout |
| `is constant` | Post-unary expression | Whether expression is compile-time available | Constant/execution model |
| `is final` | Post-unary type-use query | Underlying place final/varying truth | Qualifier reflection |
| `is immutable` | Post-unary value/type-use query | Value-lifetime mutability truth | Qualifier reflection |
| `is readonly` | Post-unary access/type-use query | Access capability truth | Qualifier reflection |

A type-information operation is reserved in concept but has no exact words yet.
It returns immutable, readonly, final metadata about a concrete type identity.

## Generated underlying and enum forms

Enums and identities receive immediate-boundary type/value forms:

| Operation | Exact form | Behavior owner |
| --- | --- | --- |
| Immediate underlying type | `MyType underlying type` | Enum or identity type model |
| Immediate underlying value | `myValue underlying value` | Enum or identity type model |

An identity or enum original-definition body additionally receives a private
final post-unary place form:

| Operation | Exact form | Behavior owner |
| --- | --- | --- |
| Private immediate underlying place | `myValue underlying place` | Identity or enum type model |

These forms are regenerated for each outer boundary and are not mechanically
forwarded by identity exposure. `underlying place` remains private to the
original defining authority and requires an instance receiver. Complete enum
owner behavior is defined by [Zax enums](enums.md).

Every enum additionally receives:

| Operation | Exact form | Behavior owner |
| --- | --- | --- |
| Backing projection | `myEnum as UnderlyingType` | Enum owner |
| Raw adoption | `MyEnum unsafe from myRaw` | Enum and safety model |

The exact underlying type argument is required for backing projection. Raw adoption
is a defined unsafe permission that bypasses the enum's ordinary admission
policy and preserves the exact backing representation.

Every enum also receives overridable comparison defaults using the comparison
forms cataloged above. A `flags` enum receives overridable defaults for `|`,
`&`, `^`, `&~`, and their direct mutation forms. Exact availability and
type-specific behavior are defined by [Zax enums](enums.md).

## Iteration protocol phrases

`each from` recognizes these receiver-owned phrase forms:

| Phrase words | Use shape | Protocol role |
| --- | --- | --- |
| `iterate` | `iterate <source>` | Produce an initial cursor or optional cursor |
| `value at` | `value at <cursor>` | Produce the current value, reference, or proxy |
| `advance` | `advance <cursor>` | Mutate to a successor and return `Boolean`, or return a replacement cursor or optional cursor |
| `erase and advance` | `erase and advance <cursor>` | Erase current and mutate to or return a successor |
| `erase` | `erase <cursor>` | Erase current without producing a successor |

These are language-recognized protocol forms that a source or cursor type may
declare on its own receiver. They are not global extension points and do not
create a generic iterator constraint by themselves. Complete acquisition,
result-shape, progression, erasure, and availability behavior is owned by
[Zax iteration](iteration.md#cursor-driven-from-traversal).

An iterable may additionally declare:

| Phrase words | Use shape | Protocol role |
| --- | --- | --- |
| `count` | `<source> count` | Return the exact non-consuming `IndexSize` count for the corresponding traversal |

The `count` operation is optional. A provided operation returns a concrete
count and promises exact agreement with traversal. Array construction may use
it for reservation; complete behavior is in
[Zax arrays and slices](arrays-and-slices.md#iterable-contributions).

## Array capacity forms

Intrinsic arrays provide exact pre-unary query phrases:

```zax
actual := capacity of values
suggested := suggested capacity of values

minimum := minimum length of MyArrayType
maximum := maximum length of MyArrayType
```

`capacity of` returns concrete `IndexSize`. `suggested capacity of` returns
`IndexSize?`, absent when no suggestion is present.

`minimum length of` receives an array type and returns concrete `IndexSize`.
`maximum length of` returns `IndexSize?`, absent for an open maximum. Current
value length uses the ordinary `array.length()` callable.

Mutating reserve, trim, and suggest operations are ordinary array callables
rather than phrase operators.
Complete behavior belongs to
[Zax arrays and slices](arrays-and-slices.md#length-capacity-and-suggestion-are-different).

## Endianness reference

Endian type teaching belongs to [Zax endianness](endianness.md). Endianness is
an intrinsic scalar property rather than an enum boundary. Its compact
conversion surface is:

| Concern | Form |
| --- | --- |
| Numeric conversion | `myLittle as Scalars.Integers.Big.U32` |
| Active-environment conversion | `myBig as U32` |
| Byte-preserving local view | `myLittle as coercive layout Scalars.Integers.Big.U32 &` |
| Representation extraction/adoption | Concept accepted; exact words remain unsettled |

Each concrete-endian specialization receives its scalar family's complete
applicable protected surface. Mixed-endian operands require explicit numeric
conversion.

## Circumfix forms

The closed circumfix catalog begins with:

| Opening/closing | General operation |
| --- | --- |
| `\|` ... `\|` | Required magnitude |
| `\|%` ... `\|` | Wrapping magnitude |
| `\|?` ... `\|` | Optional magnitude |
| `\|!` ... `\|` | Combined magnitude report |
| `\|\|` ... `\|\|` | Norm-like operation |
| `\|\|` ... `\|` | Intent-gated saturating magnitude |

Protected integer behavior is in the
[integer operator catalog](integer-operator-catalog.md).

Natural unacknowledged source is a confusable-form intent error:

```zax
||myValue| // error: explicit intent acknowledgement is required
```

The operation is available when an intent acknowledgement confirms the
asymmetric interpretation:

```zax
intent<asymmetric-saturating-magnitude>{ ||myValue| }
```

The category and its defined-but-suspicious source contract are owned by
[Zax intent acknowledgements](intent-acknowledgements.md). `bare{...}` is the
separate strict keyword-neutral source enclosure defined by
[Zax source structure](source-structure.md#keyword-neutral-bare-source); it does
not acknowledge this form.

`|myValue||` has no assigned operation.

## Assignment and swap forms

Assignment uses:

```zax
myDestination = mySource
```

It is right-associative. Declaration initialization remains separate syntax.
The generated same-type form returns readonly `copy` access to its destination
receiver, allowing:

```zax
myFirst = mySecond = myThird
```

Custom assignment may declare another result shape. Exact generated signatures
and reconstructive replacement belong to
[construction, replacement, and destruction](construction-and-destruction.md#generated-copy-construction-and-assignment).

Swap uses:

```zax
myLeft <<>> myRight
```

It is left-associative.

Protected integer behavior belongs to the
[integer operator catalog](integer-operator-catalog.md). Lifecycle-specific
assignment/replacement remains with its lifecycle owner. Optional `T? = T?`
replaces the complete wrapper lifetime, while `T? = [{...}]` is a distinct
packet-construction boundary; see
[Zax optional values](optional-values.md#construction-wrapper-replacement-and-boxed-assignment).

## Allocation initializers

`@`, `@!`, `@<`, and `@!<` are exact reserved declaration-bound allocation
tokens:

| Form | Request failure | Raw destination behavior |
| --- | --- | --- |
| `@` | Panic | Declaration-attached disposition |
| `@!` | Produce `Nothing` | Declaration-attached disposition on success |
| `@<` | Panic | Open-ended |
| `@!<` | Produce `Nothing` | Open-ended on success |

They are not ordinary open unary operators and require either a declaration
initializer or an existing typed pointer destination:

```zax
owner : MyValue * unique = @
ownerInArena : MyValue * unique = @{ myArena }

scheduled : MyValue * = @
reset scheduled
scheduled = @{ anotherArena }

ordinary : MyValue *
ordinary = @

missing = @ // error: no typed destination
```

An attached `@{...}` enclosure supplies allocation policy. A following
construction packet remains a separate source component:

```zax
owner : MyValue * strong =
  @{ myArena } [{ endpoint }]
```

A trailing ordinary expression instead supplies an existing allocation whose
resident instance is preserved:

```zax
prepared : MyValue * unique shareable =
  @ originalValue as last
```

The operation allocates only metadata required by the destination role. A
construction packet and trailing source are distinct forms.

Complete behavior is defined by
[Zax pointers, allocation, and arenas](pointers-and-arenas.md#allocate-through-a-declaration).

## Call, index, splice, and mixfix

Call and index are recognized postfix delimited forms:

```zax
myCallable(myArguments)
myContainer[myIndex]
```

They are not generic post-unary delimiter operators.

Each index bracket contains one index. Multidimensional access chains brackets:

```zax
myMatrix[row][column]
```

A bracket containing a range forms splice source:

```zax
myContainer[start..<end]
myContainer[start..end]
```

`..` is inclusive and `..<` is half-open. They are recognized in splice and
other specifically assigned range contexts rather than receiving a general
binary-expression precedence. `start..<` is malformed.

A custom splice declaration uses `operator splice 1`. A mixfix component uses
`call N` for an N-input call or `index 1`/`splice 1` for one bracket component.
Several adjacent brackets appear as several index/splice components.
Complete mixfix matching belongs to [Zax mixfix operators](mixfix-operators.md).
Endpoint normalization, empty ranges, intrinsic bounds, custom results, and
slice behavior belong to [Zax arrays and slices](arrays-and-slices.md#slicing).

## Literal merge and join

Adjacent segments resolving to one literal declaration implicitly merge.
`<|>` is the optional protected, non-overloadable explicit form:

```zax
myImplicit := x"first '" x' second"'
myPayload := x"first '" <|> x' second"'
```

Both combine payload before expression parsing and therefore bind before every
value operator. Every segment must independently resolve to the same literal
declaration. Implicit merge requires ordinary `\` across a physical line;
trailing `<|>` establishes symbolic continuation. Complete source, grouping,
alias, continuation, and empty-operand rules are defined by
[literal source and operators](literal-source-and-operators.md#merge-literal-source).

`<+>` is an open overloadable binary operator. Join chains associate left:

```zax
myText := utf8'first' <+> c'\n' <+> utf8'second'
```

It associates left and uses ordinary left-receiver discovery and callable
selection. Literal use requires the selected implementation to execute during
compilation. String/character result identities and directional admission are
defined by [strings and characters](strings-and-characters.md#compile-time-joining).

`<|>` always forms source before `<+>`. The final placement of `<+>` among
unrelated value-operator levels remains future catalog work; group an expression
that mixes them rather than relying on an unstated precedence.

When both immediate literal operands resolve to one declaration, `<+>` requires
`intent<same-prefix-literal-join>`.

Intrinsic arrays define ordinary runtime `<+>` and its assignment-level
compound `<+>=`:

```zax
combined := left <+> right
left <+>= right
```

Array-specific result and mutation behavior belongs to
[Zax arrays and slices](arrays-and-slices.md#joining-arrays).

## Deferred and unavailable forms

- Legacy `@@` parallel-allocation meaning is superseded. Arena capabilities
  express concurrency requirements.
- `|>` remains function-chaining evidence.
- Legacy result split/combine `<-` and `->` are superseded by the protected
  `>-`, `-<`, and `-<>-` structural mapping family.
- Runtime `<+>` behavior outside current intrinsic arrays remains future
  string/operator work.
- Exact multiword, reversal, and masked extraction/deposit words remain numeric
  work.
- Layout, lifetime, allocator, and complete reflection phrases remain their
  concept owners' work.
- `<=>` remains recognized for custom types; protected integer behavior awaits
  an ordering result type.
- Ordinary floating bitwise operations remain unavailable; use the
  representation/coercion surface defined by the floating owner.
- `~&`, `~|`, `~^`, `|~`, and logical punctuation such as `!&&` are not
  aliases for the exact phrase forms.
- `()` and `[]` are call/index forms.
- Quote marks are literal delimiters; a standalone single-quoted
  whitespace-separated form is a phrase fence.

## Costs and diagnostics

Tooling needs to expose:

- selected declaration and discovery domain;
- protected, generated, declared, phrase, circumfix, or mixfix origin;
- eager versus short-circuit behavior;
- operand evaluation and immediate binding;
- copies, moves, references, temporaries, and mutation;
- selected type-specific policy/result owner;
- fallback negation;
- branch-specific selection;
- zero/multiple result misuse; and
- unavailable protected signatures.

Diagnostics distinguish:

- unrecognized/reserved form;
- unsupported fixity;
- incomplete source;
- tree or implementation ambiguity;
- unavailable best candidate;
- protected-signature conflict;
- grouping-required comparison;
- confusable source;
- invalid shift count; and
- type-specific failure routed to the applicable concept owner.

## Source stability

- Adding an overload may select another body or create ambiguity.
- Adding a direct complement suppresses prior fallback.
- Adding a direct mixfix may replace decomposition.
- Adding/removing/lengthening/reserving a phrase may change validity.
- Adding a protected signature may make unavailable source valid.
- Changing precedence or tokenization may reinterpret source.

Source, declaration, import, or module order never silently resolves these
effects.

## Boundaries and maturity

This catalog is current conceptual design, not formal grammar, a type-specific
numeric reference, a conformance contract, or an implementation mapping.

Complete integer behavior is in the
[integer operator catalog](integer-operator-catalog.md). Fixed and floating
semantics are current in [fixed-point scalars](fixed-point-scalars.md) and
[floating-point scalars](floating-point-scalars.md); exact wording for several
advanced operations remains future catalog work. Literal source and declarations
are current in [literal source and operators](literal-source-and-operators.md).
Runtime joining outside arrays, unbounded numeric, enum, pointer, non-array
indexing, allocation, reflection, panic-recovery, and build-option syntax remain
focused future work.
