# Zax operator catalog

| Field | Value |
| --- | --- |
| Status | Current conceptual design |
| Audience | Human developers writing, reading, reviewing, or tooling Zax expressions |
| Applies To | Exact operator forms, precedence, built-in behavior, and availability accepted so far; not a formal grammar or specification |
| Implementation State | Not established by this repository |
| Owns | The closed symbolic and circumfix catalogs; the exact operator-phrase catalog; operator precedence and associativity; built-in logical, arithmetic, bitwise, shift, count, set-bit mask/position, reduction, mutation, and swap behavior; the compact generated enum and endian operator reference; protected intrinsic availability; Boolean fallback pairs; reserved, unavailable, and deferred forms |
| Does Not Own | Shared discovery, candidate-tree formation, and callable selection ([operators](operators.md)); operator-phrase use, declaration, interpretation, fencing, and presentation ([operator phrases](operator-phrases.md)); mixfix tree matching ([mixfix operators](mixfix-operators.md)); general source tokenization, spacing, comments, continuation, and layout ([source structure](source-structure.md)); complete call and result mapping ([function invocation](function-invocation.md)); endian semantic enum teaching ([endianness](endianness.md)); complete numeric, literal, floating-point, fixed-point, enum, indexing, allocation, pointer, reflection, or panic design |
| Source / Provenance | Legacy [basics](../basics.md) evidence and dispositioned operator-overloading material, refined against current language owners |

## Mental model

Zax recognizes a closed set of symbolic and circumfix operator forms, an exact
catalog of language-defined operator phrases, and an open facility for
receiver-owned custom phrases.

```zax
result := left + right * scale
```

The catalog fixes the tree before overload selection:

```text
left + (right * scale)
```

A type may overload a recognized form when the operand domain is open to it. A
declaration cannot invent arbitrary punctuation or assign a different precedence
to an existing form.

The common families are:

| Family | Representative forms |
| --- | --- |
| Arithmetic | `+`, `+?`, `+%`, `+\|`, `+!` |
| Comparison | `==`, `!=`, `<`, `<=`, `>`, `>=`, `<=>` |
| Logical | `!`, `&&`, `\|\|`, `^^`, `logical nand`, `logical nor`, `logical xnor` |
| Bitwise | `~`, `&`, `\|`, `^`, `&~`, `bitwise nand`, shifts, rotates, counts, masks, reductions |
| Mutation | Compounds, increment/decrement policies, `~=`, `<<>>`, exact phrase mutations |
| Circumfix | `\|value\|`, `\|?value\|`, `\|!value\|`, `\|\|value\|\|` |
| Delimited and multi-part | Call, index, and [mixfix](mixfix-operators.md) forms |

An **operator phrase** is spelled with exact ASCII lower-case alphanumeric words.
Its cohesive use and presentation model is owned by
[operator phrases](operator-phrases.md), and its shared selection machinery by
[operators](operators.md). This catalog owns the exact forms and precedence.

## Expression-tree formation

### Precedence and grouping

Higher levels bind before lower levels:

| Level | Forms | Grouping |
| --- | --- | --- |
| Enclosed/grouped | Parentheses and recognized circumfix forms | Delimiter-owned |
| Postfix/delimited | Call, index, member/dereference, post `++`/`--` families | Left-to-right chaining |
| Prefix | `?`, `!`, `~`, `~=`, `#...`, pre `++`/`--` families, unary arithmetic policies | Separate applications require grouping |
| Multiplicative | `*`, `/`, `%` and accepted policy variants | Left |
| Additive | `+`, `-` and accepted policy variants | Left |
| Shift/rotate/composition | `<<`, `>>`, `>>>`, `<<<`, `<<%`, `>>%`, modulo-count shift phrases, multiword shift/rotate/funnel operations | Left |
| Ordinary phrase | `as`, `unsafe as`, reserved language phrases, and every newly declared phrase form | Left |
| Relational | `<`, `<=`, `>`, `>=`, `<=>` | Ungrouped chaining is an intent error |
| Equality | `==`, `!=` | Ungrouped chaining is an intent error |
| Bitwise AND | `&`, `&~`, `bitwise nand` | Left |
| Bitwise XOR | `^`, `bitwise xnor` | Left |
| Bitwise OR | `\|`, `bitwise nor`, `bitwise or not` | Left |
| Logical AND | `&&`, `logical nand`, `logical and not` | Left |
| Logical XOR | `^^`, `logical xnor` | Left |
| Logical OR | `\|\|`, `logical nor`, `logical or not` | Left |
| Conditional expression | `condition ?? trueValue ;; falseValue` | Right-nesting |
| Assignment and compounds | `=`, arithmetic/bitwise/shift compounds, and exact phrase mutations | Right |
| Swap | `<<>>` | Left |

**Ordinary phrase precedence** is the single language-wide level shared by every
newly introduced phrase form. It sits between shift/rotate/composition and
relational operations, and same-level ordinary binary phrase chains are
left-associative:

```zax
result = left combines with right + adjustment
// result = (left combines with (right + adjustment))

ready && value matches pattern
// ready && (value matches pattern)

converted compares with source == expected
// (converted compares with source) == expected

a phrase1 b phrase2 c
// (a phrase1 b) phrase2 c
```

Association defines the tree. Runtime evaluation order remains the separate rule
that eager runtime operands evaluate in visible source order.

A language-defined phrase form may instead occupy the globally fixed level of its
family, as the logical, bitwise, shift, and mutation rows above show. Every
permitted implementation of that exact form inherits its level; precedence never
depends on receiver or operand type. A custom implementation of a language-defined
form therefore keeps the form-specific level, while a newly declared form uses
ordinary phrase precedence.

A low-precedence unary phrase consumes the complete higher-precedence expression
on its operand side, while a type-use hole remains category-bounded:

```zax
type of left + right
// type of (left + right)

left + right is constant
// (left + right) is constant

size of MyType + overhead
// (size of MyType) + overhead
```

Parentheses deliberately create another tree:

```zax
a[b] = (c + d) * e
a[b] = c + d * e
```

That difference may select another ordinary overload or
[mixfix](mixfix-operators.md).

### Grouping-required comparisons

Ungrouped relational and equality chains are rejected as suspicious intent:

```zax
a < b < c   // error: grouping required
a == b == c // error: grouping required
```

Explicit grouping may be valid when the intermediate result supports the next
operation:

```zax
(a < b) < c
```

This syntax rule does not promise that custom comparisons are transitive or form
an equivalence relation.

### Symbolic fixity and adjacency

Symbolic whitespace presents fixity:

```zax
!value       // pre-unary
value++      // post-unary
left + right // binary
```

The longest recognized token wins. Two separate unary applications require
grouping:

```zax
--value    // one pre-decrement token
-(-value)  // two negations
!(!value)  // two logical negations
```

See [source structure](source-structure.md) for the exact whitespace,
continuation, and layout rules.

### Phrase fixity and adjacency

A phrase component uses pre-unary, post-unary, or binary fixity:

```zax
type of value           // pre-unary
bits trailing set bit position // post-unary
left logical nand right // binary
```

Phrase recognition is not longest-word-sequence matching. The compiler forms
every structurally complete candidate tree and requires exactly one to remain
viable; see
[operators](operators.md#candidate-tree-formation-and-selection). Phrase use,
fencing, and physical presentation are taught by
[Zax operator phrases](operator-phrases.md).

Adjacent independent phrase applications require grouping, exactly as adjacent
symbolic unary applications do.

## Results and overloads

An operator may declare zero, one, or several result slots. A nested expression
node requires exactly one usable value:

```zax
sum:, carry: = customLeft + customRight

combined := customLeft + customRight
// error if the selected + has two mandatory results
```

Several results do not become an implicit tuple or anonymous structure. Shared
result routing and narrow expected-result selection are defined by
[function invocation](function-invocation.md).

Custom overloads may return arbitrary result shapes. Built-in operations use the
results documented below.

## Protected intrinsic domains

User code cannot claim a signature whose every operand belongs to a closed
intrinsic operator family, whether or not Zax currently supplies that exact
operation.

This **protected intrinsic signature** rule controls who may declare an exact
operation. It is separate from a
[mixfix-consumption barrier](terms.md#mixfix-consumption-barrier), which protects
an already resolved source node from being consumed into a user mixfix because
its evaluation or lifecycle boundary must remain visible.

```zax
// error: all operands are closed intrinsic values
operator binary '+' final :
  (result : Strange)(lhs : Integer, rhs : Integer) = { ... }

// permitted: one operand is custom
operator binary '+' final :
  (result : MyType)(lhs : Integer, rhs : MyType) = { ... }
```

Protection is recorded by family:

| Family | Current boundary |
| --- | --- |
| `Boolean` | Truth, equality, logical, assignment, and other accepted Boolean operations are protected |
| Fixed-width signed integers | Accepted arithmetic, comparison, bitwise, reduction/count, shift/rotate, magnitude, mutation, and swap operations are protected |
| Fixed-width unsigned integers | The applicable corresponding family is protected; signed-only operations remain unavailable |
| Pointer-sized numeric integers | Follow the applicable signed/unsigned integer family; pointers themselves do not |
| Floating point | Domain reserved; ordinary C++-familiar numeric behavior awaits focused float review; integer policies and bitwise forms are unavailable |
| Future fixed-point/arbitrary-width intrinsics | Domain reserved pending numeric and finite-bit-extent review |
| Other closed intrinsic families | Reserved and unavailable where focused type work has not established behavior |
| Extensible library families | Library-owned availability; `String` remains extensible rather than closed merely because the language recognizes its literals |

Aliases inherit a protected intrinsic domain only when they retain the
underlying type identity. Missing required protected support indicates an
incomplete toolchain, not an optional library that the program forgot to link.

## Boolean operations

### Symbolic logical core

| Operation | Form | Exact Boolean evaluation |
| --- | --- | --- |
| Identity/status | `?value` | Return the Boolean value directly |
| Negation | `!value` | Return its opposite |
| AND | `a && b` | Skip `b` when `a` is false |
| OR | `a \|\| b` | Skip `b` when `a` is true |
| XOR | `a ^^ b` | Evaluate both operands |

For operand shapes other than exact `Boolean`/`Boolean`, `&&`, `||`, and `^^`
are ordinary eager overloads and may return arbitrary result shapes.

### Exact logical phrase forms

The following exact binary phrase forms are language-defined:

| Operation | Exact form | Precedence | Exact Boolean evaluation |
| --- | --- | --- | --- |
| Logical NAND | `logical nand` | Logical AND | Skip RHS when LHS is false; result true |
| Logical AND-NOT | `logical and not` | Logical AND | Skip RHS when LHS is false |
| Logical NOR | `logical nor` | Logical OR | Skip RHS when LHS is true; result false |
| Logical OR-NOT | `logical or not` | Logical OR | Skip RHS when LHS is true |
| Logical XNOR | `logical xnor` | Logical XOR | Eager |

```zax
result := left logical nand right
result := left logical and not right
result := left logical nor right
result := left logical or not right
result := left logical xnor right
```

The exact Boolean AND/OR-derived operations short-circuit and are
[mixfix-consumption barriers](mixfix-operators.md#protected-barriers). Exact
Boolean `logical xnor` is eager and consumable. A permitted custom operand
implementation of the same form is an ordinary eager operation that keeps the
form's precedence.

The exact phrase mutation forms use assignment precedence and right association:

```zax
flag logical and assign rhs
flag logical nand assign rhs
flag logical and not assign rhs
flag logical or assign rhs
flag logical nor assign rhs
flag logical or not assign rhs
flag logical xor assign rhs
flag logical xnor assign rhs
```

The receiver place evaluates once. Exact Boolean AND/OR-derived mutations retain
their protected short-circuit behavior; XOR/XNOR mutations are eager.

### Complement fallback

Fallback exists only for these pairs:

| Written operation | Direct counterpart |
| --- | --- |
| `?value` | `!value`, then Boolean negation |
| `!value` | `?value`, then Boolean negation |
| `a == b` | `a != b`, then Boolean negation |
| `a != b` | `a == b`, then Boolean negation |
| `a < b` | `a >= b`, then Boolean negation |
| `a >= b` | `a < b`, then Boolean negation |
| `a <= b` | `a > b`, then Boolean negation |
| `a > b` | `a <= b`, then Boolean negation |

Direct selection happens first. Ambiguity and a uniquely best unavailable direct
operation are errors. Only no applicable direct operation attempts the
counterpart, which must resolve directly and return exactly `Boolean`. Fallback
never swaps operands or combines several comparisons.

## Integer arithmetic

### Addition, subtraction, and multiplication

The examples use addition; subtraction and multiplication follow the same
policy:

| Form | Built-in result |
| --- | --- |
| `a + b` | Mathematical result; panic when unrepresentable |
| `a +? b` | Optional result; empty when unrepresentable |
| `a +% b` | Wrapped/modular result |
| `a +\| b` | Saturated result |
| `a +! b` | Combined wrapped, saturated, and overflow report |
| `a +!% b` | Wrapped value plus overflow status |
| `a +!\| b` | Saturated value plus overflow status |

The specialized report shape is conceptually:

```zax
<anonymous> :: type {
  value : Integer
  overflowed : Boolean
}
```

The combined report is:

```zax
<anonymous> :: type {
  wrapped : Integer
  saturated : Integer
  overflowed : Boolean
}
```

These examples describe one anonymous structural result; exact anonymous type
syntax and identity remain future structural-typing work. `?report` means no
overflow, and `!report` means overflow through the accepted fallback.

### Unary signedness counterpart

Pre-unary `+` requests the equal-width opposite-signedness counterpart:

```zax
unsignedValue : U8 = 100
signedValue := +unsignedValue // I8(100)
```

Its policy forms are:

```zax
+value   // panic when counterpart cannot represent the value
+?value  // optional counterpart
+%value  // wrapped counterpart
+|value  // saturated counterpart
+!value  // combined report
```

Integer literal `+55` requests unsigned realization, and `+(55)` is equivalent.
Exact literal width and prefix rules remain future literal work.

### Unary negation

Fixed-width signed negation supports:

```zax
-value
-%value
-|value
-?value
-!value
-!%value
-!|value
```

Bare negation panics on the minimum signed value; wrapping retains its minimum
representation, saturation returns the signed maximum, and optional negation is
empty. Protected unsigned integers have no ordinary unary negation.

### Division and remainder

```zax
quotient := a / b
checkedQuotient : Integer? = a /? b

remainder := a % b
checkedRemainder : Integer? = a %? b
```

Bare division panics for zero and signed minimum divided by `-1`; optional
division is empty. Successful signed division truncates toward zero.

Bare remainder panics only for zero. Signed minimum remainder `-1` is zero. A
nonzero signed remainder has the dividend's sign.

No built-in wrapping, saturating, or reporting division/remainder forms are
currently recognized. A future reason-distinguishing division report may justify
another language addition.

### Compile-time arithmetic failure

Compile-time arithmetic has the same semantics as runtime arithmetic. A
language-level panic during compile-time execution becomes a compiler diagnostic
with the operation, source location, relevant operands, and compile-time
evaluation path where practical. It is not an internal compiler crash and does
not imply a compile-time panic handler.

## Compound arithmetic

Every compound evaluates the destination place and RHS once. It is a direct
operation, not fallback through a value operation and `=`.

| Policy | Addition forms | Mutation on failure/intervention |
| --- | --- | --- |
| Required | `+=` | Panic and leave destination unchanged |
| Optional | `+?=` | Update on success; otherwise unchanged and return empty optional access |
| Wrapped | `+%=` | Always write wrapped value |
| Saturated | `+\|=` | Always write saturated value |
| Combined report | `+!=` | Update exact result on success; otherwise unchanged; return combined report |
| Wrapped report | `+!%=` | Always write wrapped value and report overflow |
| Saturated report | `+!\|=` | Always write saturated value and report overflow |

Subtraction and multiplication use the same suffixes. Ordinary successful
compounds return writable access to the updated place. Specialized reporting
forms return their report.

Division supplies `/=` and `/?=`. Remainder supplies `%=` and `%?=`. Their
failure behavior mirrors the corresponding value operation.

## Increment and decrement

Both pre and post fixities recognize:

```text
++   ++?   ++%   ++|   ++!   ++!%   ++!|
--   --?   --%   --|   --!   --!%   --!|
```

Bare operations panic and leave the destination unchanged at the representable
boundary. Wrapping and saturating forms always write their policy result.
Optional forms update only on success.

Ordinary pre forms return writable access to the updated place. Ordinary post
forms return the previous value by copy:

```zax
updated := ++value
previous := value++
```

Reporting forms return one transition report:

```zax
<anonymous> :: type {
  previous : Integer
  wrapped : Integer
  saturated : Integer
  overflowed : Boolean
}
```

Specialized reporting forms use `previous`, the policy `value`, and
`overflowed`. Decrement mirrors increment.

## Bitwise operations

### Symbolic core

| Operation | Form |
| --- | --- |
| Complement | `~value` |
| In-place complement | `~=value` |
| AND | `a & b` |
| OR | `a \| b` |
| XOR | `a ^ b` |
| AND-NOT / clear | `a &~ b` |

The direct compounds are `&=`, `|=`, `^=`, and `&~=`.

### Exact bitwise phrase forms

| Operation | Exact form | Precedence |
| --- | --- | --- |
| Bitwise NAND | `bitwise nand` | Bitwise AND |
| Bitwise XNOR | `bitwise xnor` | Bitwise XOR |
| Bitwise NOR | `bitwise nor` | Bitwise OR |
| Bitwise OR-NOT | `bitwise or not` | Bitwise OR |

```zax
result := left bitwise nand right
result := left bitwise xnor right
result := left bitwise nor right
result := left bitwise or not right
```

Their exact direct mutation counterparts use assignment precedence and right
association:

```zax
value bitwise nand assign rhs
value bitwise xnor assign rhs
value bitwise nor assign rhs
value bitwise or not assign rhs
```

Ordinary AND, OR, XOR, and AND-NOT retain the concise symbolic forms and
compounds above. No redundant language-defined phrase mutations are introduced
for them.

### Counts and reductions

| Operation | Form | Built-in result |
| --- | --- | --- |
| Population count | `#bits` | Canonical unsigned bit count |
| Leading-zero count | `#<bits` | Canonical unsigned bit count |
| Trailing-zero count | `#>bits` | Canonical unsigned bit count |
| Leading set-bit mask | `#~<bits` | Same type and logical extent as the operand |
| Trailing set-bit mask | `#~>bits` | Same type and logical extent as the operand |
| Reduction AND | `#&bits` | `Boolean` |
| Reduction OR | `#\|bits` | `Boolean` |
| Reduction XOR/parity | `#^bits` | `Boolean` |
| Reduction NAND | `#~&bits` | `Boolean` |
| Reduction NOR | `#~\|bits` | `Boolean` |
| Reduction XNOR | `#~^bits` | `Boolean` |

Reductions have no in-place forms because they change value shape. Counts, masks,
and reductions require a finite defined logical bit extent; hidden capacity or
alignment padding is not part of it.

#### Set-bit masks

`#~<` and `#~>` produce masks rather than counts. Each preserves exactly one set
bit at its original position: `#~<bits` preserves the most-significant set bit,
`#~>bits` preserves the least-significant set bit, and zero produces zero. Mask
results retain the operand's signed type; preserving a signed sign-bit mask is a
bit-pattern operation that does not overflow or panic.

Dedicated leading and trailing *one* counts remain derivable rather than having
their own symbolic forms:

```zax
leadingOnes := #<(~bits)
trailingOnes := #>(~bits)
```

### Set-bit position phrases

| Operation | Exact form | Fixity | Result |
| --- | --- | --- | --- |
| Leading set-bit position | `leading set bit position` | Post-unary | Optional canonical unsigned bit count |
| Trailing set-bit position | `trailing set bit position` | Post-unary | Optional canonical unsigned bit count |

```zax
leadingPosition := bits leading set bit position
trailingPosition := bits trailing set bit position
```

Position zero is the least-significant bit. For nonzero `bits` of logical width
`W`:

```text
leading position  = W - 1 - leading-zero count
trailing position = trailing-zero count

leading mask  = 1 << leading position
trailing mask = 1 << trailing position
```

Zero has no set-bit position, so each phrase returns an optional value:

```zax
position := bits trailing set bit position

if ?position
  shifted := x << position.
```

The optional payload uses the same canonical unsigned bit-count type as the
zero-count forms. That type is directly viable as the count operand of every
applicable built-in shift without an explicit conversion. Its exact name and
representation remain future numeric work.

A contrasting example makes the three result families visible:

```zax
bits : U8 = 0b01010100

leadingZeros := #<bits  // 1
trailingZeros := #>bits // 2

leadingMask := #~<bits  // 0b01000000
trailingMask := #~>bits // 0b00000100

leadingPosition := bits leading set bit position   // present: 6
trailingPosition := bits trailing set bit position // present: 2
```

For zero:

```zax
bits : U8 = 0

leadingZeros := #<bits  // 8
trailingZeros := #>bits // 8
leadingMask := #~<bits  // 0
trailingMask := #~>bits // 0

leadingPosition := bits leading set bit position   // absent
trailingPosition := bits trailing set bit position // absent
```

The direct position forms remain useful despite derivability because they express
optional zero handling and permit target-specific optimization without promising
a particular instruction or lowering.

### Shifts and rotates

| Operation | Form |
| --- | --- |
| Left shift | `value << count` |
| Type-natural right shift | `value >> count` |
| Explicit zero-fill right shift | `value >>> count` |
| Rotate left | `value <<% count` |
| Rotate right | `value >>% count` |

Signed `>>` sign-fills; unsigned `>>` and every `>>>` zero-fill. The count is an
unsigned integer, although a nonnegative literal may realize directly against
that requirement.

For logical extent `W`, a shift count at least `W` produces zero for logical
shifts and all sign bits for signed arithmetic right shift. Rotate uses
`count mod W`. Bits shifted out by an ordinary shift are discarded; the result
never widens implicitly.

Compounds append `=`. `<<<` and `<<<=` remain recognized custom-only forms with
no protected basic-type operation.

### Exact modulo-count shift phrases

These exact binary phrase forms sit at shift/rotate/composition precedence:

```zax
result := value shift left modulo count
result := value shift right modulo count
result := value logical shift right modulo count
```

`shift right modulo` uses the type-natural signed/unsigned right-shift behavior,
and `logical shift right modulo` explicitly zero-fills. All three reduce the count
modulo the finite logical bit extent and then discard outgoing bits; they are not
rotations.

Their exact mutation counterparts use assignment precedence and right
association:

```zax
value shift left modulo assign count
value shift right modulo assign count
value logical shift right modulo assign count
```

### Multiword and bit-arrangement concepts

Zax accepts these operation concepts while their exact words, complete numeric
constraints, and some result shapes remain future numeric work:

- multiword logical left and right shift;
- multiword arithmetic right shift;
- multiword explicit zero-fill right shift;
- multiword rotate left and right;
- funnel shift left and right;
- writable multiword forms involving several component places;
- bit reversal;
- byte reversal;
- masked extraction; and
- masked deposit.

Multiword operations are [mixfix](mixfix-operators.md#multiword-bit-operations)
because they consume several value holes and phrase components. They use
shift/rotate/composition precedence and have language-provided basic-integer
forms. Illustrative source remains:

```zax
// Illustrative wording; exact multiword phrases are not established.
high shift left through low by count
high rotate left through low by count
result funnel right from high and low by count
```

Byte reversal preserves bit order within each byte; bit reversal does not. Masked
extraction packs every source bit selected by a mask into contiguous low-order
positions, and masked deposit scatters contiguous low-order input bits into
positions selected by a mask. All of these operations require a finite defined
logical bit extent.

## Operator phrase forms

Phrase forms fall into three kinds:

| Kind | Who may declare it | Precedence |
| --- | --- | --- |
| Open language-defined form | The language supplies protected signatures; user types may add implementations for their own receivers | The form's fixed level |
| Reserved phrase form | Language only; user code cannot declare it | The form's fixed level |
| Newly declared form | Any receiver type, for its own instances or type identity | Ordinary phrase precedence |

### `as` and `unsafe as`

`as` and `unsafe as` are open language-defined binary forms at ordinary phrase
precedence:

```zax
converted := source as DestinationType
converted := source unsafe as DestinationType
```

The left value supplies receiver discovery. The right operand is a complete type
argument, so the type use consumes as much as type-use grammar permits:

```zax
value as Payload is immutable
// (value as Payload) is immutable
```

Protected intrinsic conversion domains remain language-provided, and user-defined
words do not independently grant unsafe authority. Complete conversion semantics
remain future casting work.

### Reserved phrase forms

A **reserved phrase form** cannot be declared by user code. A form may be reserved
before every domain detail is complete; the theoretical prototype and broad
behavior below are recorded so later work does not have to rediscover the
operation. All of these sit at ordinary phrase precedence.

| Exact form | Fixity and theoretical prototype | Conceptual behavior | Deferred refinement |
| --- | --- | --- | --- |
| `as default` | Post-unary on a type identity or value expression | Produce the default-qualified type identity, or convert a value to a value compatible with that default type | Complete qualifier defaults, transfer, generics, and type-result behavior |
| `type of` | Pre-unary on one independently resolved expression | Return its concrete static type identity without executing or disambiguating it | Alias identity, qualified type identity, reflection representation |
| `size of` | Pre-unary type receiver | Return its byte size in the active execution context | Host/target context, unusual widths, padding |
| `alignment of` | Pre-unary type receiver | Return its required byte alignment in the active execution context | Host/target context and complete layout |
| `offset of` | Binary structural member path and containing type identity | Return the byte offset from the containing type entry point to that member path | Exact member-designator grammar, composition, layout |
| `is constant` | Post-unary expression query | Return whether that expression is available as a compile-time constant | Host/target context and compile-time execution |
| `is final` | Post-unary resolved type-use query | Return the underlying type-use/referent-place final stance, not one path's replacement restriction | Complete qualifier reflection |
| `is immutable` | Post-unary value/type-use query | Return the current value-lifetime mutability stance | Complete qualifier reflection |
| `is readonly` | Post-unary access/type-use query | Return the current access-path capability stance | Complete qualifier reflection |

`as default` has two reserved shapes:

```zax
MyType as default     // default-qualified concrete type identity
expression as default // value compatible with the expression's default type
```

The value form is conceptually equivalent to
`expression as ((type of expression) as default)`.

`type of` returns the selected expression's concrete static type identity. It
does not execute the expression to discover its type, choose among result-only
overloads, choose between phrase trees, or create a type for zero or multiple
results. Constancy is an expression-evaluation fact rather than part of type
identity, which is why `is constant` remains a direct expression query:

```zax
compileValue := 42
runtimeValue := readInteger()
// Both may have type Integer; only compileValue is constant.
```

`is final` reports the resolved type-use or referent-place truth rather than
whether one declaration may exercise replacement:

```zax
restricted final : MyType varying

restricted is final // false
```

That distinction is owned by [qualifiers](qualifiers.md#type-side-truth-versus-declaration-side-permission).

A pre-unary type-receiver **type-information operation** is also reserved in
concept: it returns immutable, readonly, final metadata about a concrete type
identity. Its exact words remain deferred, so no exact source spelling is
reserved for it yet. Illustrative wording such as `type information of` is not a
source reservation. The direct property list above is intentionally small;
declaration properties such as `private`, export status, deprecation, and
directives belong to future declaration reflection rather than an expanding
`is <property>` family.

## Generated enum operations

Every enum is an independent type backed by one fundamental intrinsic type, and
every enum receives four protected language-provided phrase operations:

| Operation | Exact form | Fixity | Behavior |
| --- | --- | --- | --- |
| Backing type | `EnumType underlying type` | Post-unary type receiver | Return the exact fundamental intrinsic type backing the enum, without use-site qualifiers and in its default-qualified form |
| Raw extraction | `enumValue underlying value` | Post-unary instance | Return one value of that exact underlying type containing the stored backing value unchanged, with no semantic conversion or validation |
| Semantic decode | `enumValue as UnderlyingType` | Binary, exact underlying type argument | Perform the enum's semantic conversion to its fundamental type |
| Raw adoption | `EnumType unsafe from rawValue` | Binary type receiver, exact underlying value | Establish the enum type from that backing value unchanged, with no membership, range, or semantic validation |

```zax
BackingType := EnumType underlying type
rawValue := enumValue underlying value
converted := enumValue as UnderlyingType
enumValue := EnumType unsafe from rawValue
```

For an ordinary restricted enum, semantic conversion and raw backing extraction
commonly produce the same value. A semantic enum may distinguish them; see
[endianness](endianness.md).

These four exact signatures are protected and cannot be replaced by an enum
declaration. `unsafe from` may create a value outside a restricted enum's
ordinarily named domain. Universal operations such as `type of`, `as default`,
and the direct qualifier predicates also apply to enum types and values but are
not enum-specific generated operations.

A future safe `EnumType from value` facility may validate, return an optional or
reporting result, or remain unavailable. Complete enum validity, members,
operation inheritance, and generic generation remain future enum work.

## Endianness reference

Zax conceptually supplies big-endian and little-endian semantic enum families
backed by the applicable fundamental integer types. Their teaching owner is
[Zax endianness](endianness.md); this catalog records only the compact operator
surface.

| Concern | Catalog fact |
| --- | --- |
| Entry into an endian domain | Destination type receiver: `BigEndianU32 from nativeValue`, `LittleEndianU32 from big` |
| Raw adoption | Baseline `EndianType unsafe from rawStorage` |
| Semantic exit | `endianValue as UnderlyingType` |
| Raw extraction | `endianValue underlying value` |
| Available operators | Same-type lifecycle and swap; `==`/`!=`; `~`; `&`, `\|`, `^`, `&~`; `bitwise nand`, `bitwise nor`, `bitwise xnor`, `bitwise or not`; their direct mutations; `#`; and the reduction family |
| Unavailable operators | Arithmetic, magnitude, ordering, increment/decrement, shifts, rotates, multiword operations, leading/trailing counts, masks, positions, reversal, and masked extraction/deposit |
| Mixed operand rule | An exact backing-type value is permitted only as the right operand of the equality, Boolean-bitwise, and mutation forms; the endian value must supply receiver discovery |

```zax
masked := bigValue & nativeMask
// same observable result as:
masked := bigValue & (BigEndianU32 from nativeMask)

nativeMask & bigValue
// unavailable by design: a fundamental left operand owns no enum-aware signature
```

Storage and shape compatibility never grant implicit transfer. Complete endian
teaching, native/compiler-host/target context, costs, and future operation
pressure belong to [Zax endianness](endianness.md).



## Circumfix operations

The closed circumfix catalog begins with:

| Opening/closing | Operation |
| --- | --- |
| `\|` ... `\|` | Required magnitude |
| `\|%` ... `\|` | Wrapped magnitude |
| `\|?` ... `\|` | Optional magnitude |
| `\|!` ... `\|` | Combined magnitude report |
| `\|\|` ... `\|\|` | Norm-like operation |
| `\|\|` ... `\|` | Intent-gated saturating magnitude; natural source is a confusable-form error |

```zax
magnitude := |value|
checked := |?value|
report := |!value|
length := ||customVector||
```

The minimum signed integer panics under required magnitude. Wrapped, optional,
and reporting forms expose the selected alternatives. Unsigned magnitude is the
same value; its optional/report forms remain available for generic code.

The norm form is recognized but unavailable for built-in integers. Custom types
may overload it. Nested magnitude requires `|(|value|)|`.

The asymmetric saturating-magnitude form is recognized conceptually but gated in
natural source:

```zax
||value| // confusable-form intent error: resembles a missing norm closer
```

Future bare-source work must evaluate this explicit use:

```zax
// Illustrative future syntax; bare{...} is not current Zax.
bare{ ||value| }
```

`bare{...}` would acknowledge the recognized asymmetric form without creating a
precedence or mixfix boundary. Until that source mechanism is accepted, the
operation is not writable as current source. Zax deliberately adds no redundant
saturating-magnitude phrase alias: the operation retains the asymmetric symbolic
family form and relies on the source-intent mechanism instead.

`|value||` has no assigned operation and remains unrecognized rather than gaining
meaning through the future bare-source mechanism.

## Assignment and swap

Protected ordinary intrinsic assignment updates the existing mutable value and
returns writable access to the destination:

```zax
updated : Integer writable & = destination = source
```

Immutable varying storage remains governed by compiler-owned reconstructive
replacement.

Swap uses:

```zax
left <<>> right
```

It evaluates both writable places left to right, exchanges compatible values,
and returns zero results for protected built-ins. Exact self-swap is a no-op.
Swap is left-associative so a custom mixfix may consume a chain even though
ordinary zero-result built-in swap cannot.

## Call, index, and mixfix forms

Call and index are grammar-recognized postfix delimited forms, not generic
post-unary delimiter operators:

```zax
callable(arguments)
container[indices]
```

Custom types use future `operator call` and `operator index` declarations.
Direct forms derive arity from their prototypes. A mixfix component uses
`call N` or `index N` to partition its flattened inputs.

See [function invocation](function-invocation.md) for shared call mapping and
[mixfix operators](mixfix-operators.md) for component behavior.

## Deferred and unavailable forms

- `@`, `@@`, and `@!` remain allocation evidence.
- `|>` remains function-chaining evidence.
- `->` and `<-` remain result-shape transformation evidence.
- Literal operator prefixes and quote behavior remain literal work.
- Exact multiword shift/rotate/funnel, reversal, and masked extraction/deposit
  words remain future numeric work; their operation concepts are recorded above.
- Layout, lifetime, allocator, and complete reflection phrases remain
  domain-specific work. Legacy spellings such as `count of`, `outer of`,
  `unsafe outer of`, `lifetime of`, `unsafe lifetime of`, `unsafe copy as`,
  `overhead count of`, `overhead as`, `overhead size of`, and `allocator of` are
  not reserved by this catalog.
- `<=>` is recognized for custom types; built-in signatures await an ordering
  result type.
- Floating bitwise and integer-policy operations remain reserved and unavailable.
- `~&`, `~|`, `~^`, `|~` and logical punctuation such as `!&&` are not canonical
  aliases for the exact phrase operations above.
- `()` and `[]` are call/index forms rather than generic post-unary operators.
- Quote marks are literal delimiters rather than generic circumfix declarations.
  A standalone single-quoted whitespace-separated form is a
  [phrase fence](operator-phrases.md#exact-phrase-fencing), not a literal.

## Costs and diagnostics

Programmers and tooling need to expose:

- the selected declaration and discovery domain;
- protected, generated, declared, phrase, circumfix, or mixfix origin;
- operand evaluation and immediate-binding order;
- short-circuit versus eager behavior;
- copies, moves, references, temporaries, and old-value snapshots;
- mutation versus reconstructive replacement;
- overflow policy, checks, and compile-time panic paths;
- fallback negation;
- branch-specific selections;
- finite-bit-extent requirements;
- endian conversion, transcoding, and raw-adoption cost;
- unavailable protected signatures; and
- zero/multiple results used where one value is required.

Diagnostics distinguish unrecognized form, unsupported fixity, no candidate,
nonviability, ambiguity, unavailable best, protected-signature conflict,
reserved-form declaration, grouping-required comparison, confusable-form intent,
invalid signed shift count, and policy-specific failure.

## Source stability

- Adding an overload may select another body or create ambiguity.
- Adding a direct complement suppresses prior fallback.
- Adding a direct mixfix may replace component decomposition.
- Changing an operand type may move a logical operation between protected
  short-circuit and eager custom behavior.
- Adding, removing, lengthening, reserving, or repricing a phrase form may turn
  previously unique source into a loud ambiguity, or previously unavailable
  source into valid source.
- Changing precedence or tokenization would reinterpret existing source and is
  therefore not a routine library evolution.

Exact spellings, fixed precedence, one ordinary phrase level, explicit policy
suffixes, protected intrinsic domains, and ambiguity errors preserve the intended
stability. No such change may silently reinterpret one viable tree through
source, declaration, import, module, or discovery order.

## Boundaries and maturity

This catalog is current conceptual design, not formal grammar, a conformance
contract, or an implementation mapping.

Literal operators, complete numeric type families, floating-point behavior,
fixed-point and arbitrary-width types, the canonical bit-count type name, exact
multiword and bit-arrangement words, complete enum behavior and operation
inheritance, complete reflection wording, indexing/slicing, allocation,
pointer/reference operations, composition/chaining, bare-source/confusable-form
syntax, and panic recovery remain focused future work.
