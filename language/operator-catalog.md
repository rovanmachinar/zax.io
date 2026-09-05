# Zax operator catalog

| Field | Value |
| --- | --- |
| Status | Current conceptual design |
| Audience | Human developers and tooling looking up recognized operator source forms |
| Applies To | Exact forms, fixity, precedence, association, reservation, and domain routing; not type-specific result semantics or a formal grammar |
| Implementation State | Not established by this repository |
| Owns | The closed symbolic and circumfix catalogs; exact language-defined phrase forms, including transfer-stance restatement; precedence and association; form reservation; compact protected-domain availability; generated immediate-underlying and enum forms; call/index recognition; and deferred/unavailable forms |
| Does Not Own | Complete transfer semantics ([transfer stances](transfer-stances.md)); shared operator/callable selection ([operators](operators.md), [function invocation](function-invocation.md)); phrase use and presentation ([operator phrases](operator-phrases.md)); uncommitted integer behavior ([integer literals and realization](integer-literals.md)); or cohesive type-specific behavior such as [optional values](optional-values.md), [integer operations](integer-operator-catalog.md), [identity types](identity-types.md), and [endianness](endianness.md) |
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
| Arithmetic | `+`, `+?`, `+%`, `+\|`, `+!`, `delta`, `distance` |
| Comparison | `==`, `!=`, `<`, `<=`, `>`, `>=`, `<=>` |
| Logical | `!`, `&&`, `\|\|`, `^^`, `logical nand`, `logical nor`, `logical xnor` |
| Bitwise | `~`, `&`, `\|`, `^`, `&~`, phrases, counts, masks, shifts, and rotations |
| Optional | `?value`, `value.`, `reset value`, `last value`, `move value` |
| Conversion/admission | `as`, `narrowing as`, `from`, `optional from`, `narrowing from`, `unchecked from`, `unsafe from` |
| Transfer stance | `value as copy`, `value as deep`, `value as move`, `value as last` |
| Mutation | Compounds, increment/decrement, `~=`, and exact phrase mutations |
| Circumfix | `\|value\|`, `\|?value\|`, `\|!value\|`, `\|\|value\|\|` |
| Delimited/multi-part | Call, index, and [mixfix](mixfix-operators.md) forms |

A type may overload a recognized form where its operand domain remains open.
Declarations cannot invent arbitrary punctuation or assign another precedence.

## Precedence and grouping

Higher levels bind first:

| Level | Forms | Grouping |
| --- | --- | --- |
| Enclosed/grouped | Parentheses and recognized circumfix forms | Delimiter-owned |
| Postfix/delimited | Call, index, member/dereference, post `++`/`--` families | Left-to-right chaining |
| Prefix | `?`, `!`, `~`, `~=`, `#...`, pre `++`/`--`, unary arithmetic policies | Separate applications require grouping |
| Multiplicative | `*`, `/`, `%` and accepted policy variants | Left |
| Additive | `+`, `-`, accepted policy variants, `delta`, `distance` | Left |
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
| Assignment/compound | `=`, compounds, exact phrase mutations | Right |
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
| Floating point | Domain reserved pending focused float review |
| Fixed-point and unbounded numeric types | Domain reserved pending focused numeric review |
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

## Optional forms

These exact forms are protected for optional operands:

| Form | Fixity/level | Result role |
| --- | --- | --- |
| `?value` | Symbolic prefix | Return exactly `Boolean` presence |
| `value.` | Grammar-recognized postfix access | Produce boxed access after static presence proof |
| `reset value` | Pre-unary phrase at ordinary phrase level | Leave the same wrapper absent and return its reference |
| `last value` | Pre-unary phrase at ordinary phrase level | Produce the same optional type, offer `last`, and schedule optional payload cleanup at consumer completion |
| `move value` | Pre-unary phrase at ordinary phrase level | Produce the same optional type and offer `move` without scheduling wrapper absence |

Protected optional forms are distinct from generic post-unary stance
restatement. Complete optional behavior and source consequences are defined by
[Zax optional values](optional-values.md).

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

An identity original-definition body additionally receives a private final
post-unary place form:

| Operation | Exact form | Behavior owner |
| --- | --- | --- |
| Private immediate underlying place | `myIdentity underlying place` | Identity type model |

These forms are regenerated for each outer boundary and are not mechanically
forwarded by identity exposure. `underlying place` is not part of the current
generated enum surface; a future enum definition body may motivate an
equivalent private capability after enum validity and mutation are reviewed.

Every enum additionally receives:

| Operation | Exact form | Behavior owner |
| --- | --- | --- |
| Semantic decode | `myEnum as UnderlyingType` | Enum and applicable semantic owner |
| Raw adoption | `MyEnum unsafe from myRaw` | Enum and safety model |

The exact underlying type argument is required for semantic decode.

Whether raw adoption should ultimately be classified as unchecked or unsafe
remains future enum/safety review.

## Endianness reference

Endian type teaching belongs to [Zax endianness](endianness.md). Its compact
form surface is:

| Concern | Form |
| --- | --- |
| Native value in | `BigEndianU32 from myNative` |
| Other endian in | `LittleEndianU32 from myBig` |
| Raw storage in | `BigEndianU32 unsafe from myRaw` |
| Semantic value out | `myBig as U32` |
| Raw storage out | `myBig underlying value` |

Available protected forms include same-type lifecycle/swap, equality,
complement, Boolean bitwise forms and mutations, population count, and
reductions. Complete availability/results remain with the endian owner.

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
[Zax intent acknowledgements](intent-acknowledgements.md). `bare{...}` remains
future keyword-neutral source.

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

## Call, index, and mixfix

Call and index are recognized postfix delimited forms:

```zax
myCallable(myArguments)
myContainer[myIndexes]
```

They are not generic post-unary delimiter operators.

A mixfix component uses `call N` or `index N` to partition flattened inputs.
Complete mixfix matching belongs to [Zax mixfix operators](mixfix-operators.md).

## Deferred and unavailable forms

- `@`, `@@`, and `@!` remain allocation evidence.
- `|>` remains function-chaining evidence.
- `->` and `<-` remain result-shape transformation evidence.
- Literal prefixes/quote behavior remain literal work.
- Exact multiword, reversal, and masked extraction/deposit words remain numeric
  work.
- Layout, lifetime, allocator, and complete reflection phrases remain their
  concept owners' work.
- `<=>` remains recognized for custom types; protected integer behavior awaits
  an ordering result type.
- Floating bitwise/integer-policy operations remain unavailable.
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
[integer operator catalog](integer-operator-catalog.md). Complete literal,
floating/fixed-point/unbounded numeric, enum, pointer, indexing, allocation,
reflection, panic-recovery, and build-option syntax remain focused future work.
