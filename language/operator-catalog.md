# Zax operator catalog

| Field | Value |
| --- | --- |
| Status | Current conceptual design |
| Audience | Human developers writing, reading, reviewing, or tooling Zax expressions |
| Applies To | Exact operator forms, precedence, built-in behavior, and availability accepted so far; not a formal grammar or specification |
| Implementation State | Not established by this repository |
| Owns | The closed symbolic and circumfix catalogs; operator precedence and associativity; built-in logical, arithmetic, bitwise, shift, count, reduction, mutation, and swap behavior; protected intrinsic availability; Boolean fallback pairs; reserved, unavailable, and deferred forms |
| Does Not Own | Shared discovery and callable selection ([operators](operators.md)); mixfix tree matching ([mixfix operators](mixfix-operators.md)); source tokenization, spacing, and continuation ([source structure](source-structure.md)); complete call and result mapping ([function invocation](function-invocation.md)); complete numeric, literal, floating-point, fixed-point, enum, indexing, phrase, allocation, pointer, or panic design |
| Source / Provenance | Legacy [basics](../basics.md) evidence and dispositioned operator-overloading material, refined against current language owners |

## Mental model

Zax recognizes a closed set of symbolic and circumfix operator forms.

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
| Logical | `!`, `&&`, `\|\|`, `^^`; extended phrase operations |
| Bitwise | `~`, `&`, `\|`, `^`, `&~`, shifts, rotates, counts, reductions |
| Mutation | Compounds, increment/decrement policies, `~=`, `<<>>` |
| Circumfix | `\|value\|`, `\|?value\|`, `\|!value\|`, `\|\|value\|\|` |
| Delimited and multi-part | Call, index, and [mixfix](mixfix-operators.md) forms |

Exact phrase spellings remain future operator-phrase work. Illustrative phrases
in this document identify accepted operation concepts without making their words
canonical source.

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
| Shift/rotate/composition | `<<`, `>>`, `>>>`, `<<<`, `<<%`, `>>%`, future modulo-count phrases | Left |
| Relational | `<`, `<=`, `>`, `>=`, `<=>` | Ungrouped chaining is an intent error |
| Equality | `==`, `!=` | Ungrouped chaining is an intent error |
| Bitwise AND | `&`, `&~`; future NAND phrase | Left |
| Bitwise XOR | `^`; future XNOR phrase | Left |
| Bitwise OR | `\|`; future NOR/OR-NOT phrases | Left |
| Logical AND | `&&`; future NAND/AND-NOT phrases | Left |
| Logical XOR | `^^`; future XNOR phrase | Left |
| Logical OR | `\|\|`; future NOR/OR-NOT phrases | Left |
| Conditional expression | `condition ?? trueValue ;; falseValue` | Right-nesting |
| Assignment and compounds | `=` and arithmetic/bitwise/shift compounds | Right |
| Swap | `<<>>` | Left |

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

### Extended logical operations

Zax also accepts these operation concepts:

```zax
// Illustrative phrase syntax; exact words are not established.
a logical nand b
a logical and not b
a logical nor b
a logical or not b
a logical xnor b
```

For exact Boolean operands:

| Operation | Evaluation |
| --- | --- |
| NAND | Skip RHS when LHS is false; result true |
| AND-NOT | Skip RHS when LHS is false |
| NOR | Skip RHS when LHS is true; result false |
| OR-NOT | Skip RHS when LHS is true |
| XNOR | Eager |

Direct phrase mutation concepts are also accepted:

```zax
// Illustrative phrase syntax.
flag logical and assign rhs
flag logical nand assign rhs
flag logical and not assign rhs
flag logical or assign rhs
flag logical nor assign rhs
flag logical or not assign rhs
flag logical xor assign rhs
flag logical xnor assign rhs
```

The receiver evaluates once. Exact Boolean AND/OR-derived operations retain their
short-circuit behavior; XOR/XNOR are eager. Exact phrase syntax, declaration,
precedence, and formatting remain operator-phrase work.

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

Extended operations use future phrases:

```zax
// Illustrative phrase syntax.
a bitwise nand b
a bitwise nor b
a bitwise xnor b
a bitwise or not b
```

Their direct mutation counterparts likewise use phrase concepts rather than
duplicate symbolic aliases.

### Counts and reductions

| Operation | Form | Built-in result |
| --- | --- | --- |
| Population count | `#bits` | Unsigned count |
| Leading zeros | `#<bits` | Unsigned count |
| Trailing zeros | `#>bits` | Unsigned count |
| Leading ones | `#~<bits` | Unsigned count |
| Trailing ones | `#~>bits` | Unsigned count |
| Reduction AND | `#&bits` | `Boolean` |
| Reduction OR | `#\|bits` | `Boolean` |
| Reduction XOR/parity | `#^bits` | `Boolean` |
| Reduction NAND | `#~&bits` | `Boolean` |
| Reduction NOR | `#~\|bits` | `Boolean` |
| Reduction XNOR | `#~^bits` | `Boolean` |

Reductions have no in-place forms because they change value shape. Counts and
reductions require a finite defined logical bit extent; hidden capacity or
alignment padding is not part of it.

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

Modulo-count shifts are future phrase operations:

```zax
// Illustrative phrase syntax.
value shift left modulo count
value shift right modulo count
value logical shift right modulo count
```

They reduce the count modulo the logical extent and then discard outgoing bits;
they are not rotations.

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
operation is not writable as current source. A saturating-magnitude phrase
remains an alternative for future operator-phrase work.

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
- `as`, `unsafe as`, endian, layout, reflection, lifetime, and allocator phrases
  remain phrase and domain-specific work.
- `<=>` is recognized for custom types; built-in signatures await an ordering
  result type.
- Floating bitwise and integer-policy operations remain reserved and unavailable.
- `~&`, `~|`, `~^`, `|~` and logical punctuation such as `!&&` are not canonical
  aliases for the accepted phrase-operation concepts.
- `()` and `[]` are call/index forms rather than generic post-unary operators.
- Quote marks are literal delimiters rather than generic circumfix declarations.

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
- unavailable protected signatures; and
- zero/multiple results used where one value is required.

Diagnostics distinguish unrecognized form, unsupported fixity, no candidate,
nonviability, ambiguity, unavailable best, protected-signature conflict,
grouping-required comparison, confusable-form intent, invalid signed shift count,
and policy-specific failure.

## Source stability

- Adding an overload may select another body or create ambiguity.
- Adding a direct complement suppresses prior fallback.
- Adding a direct mixfix may replace component decomposition.
- Changing an operand type may move a logical operation between protected
  short-circuit and eager custom behavior.
- Adding a language-defined phrase or built-in signature may turn previously
  unavailable source into valid source.
- Changing precedence or tokenization would reinterpret existing source and is
  therefore not a routine library evolution.

Exact spellings, fixed precedence, explicit policy suffixes, protected
intrinsic domains, and ambiguity errors preserve the intended stability.

## Boundaries and maturity

This catalog is current conceptual design, not formal grammar, a conformance
contract, or an implementation mapping.

The exact phrase catalog, literal operators, complete numeric type families,
floating-point behavior, fixed-point and arbitrary-width types, enum operation
inheritance, indexing/slicing, allocation, pointer/reference operations,
composition/chaining, bare-source/confusable-form syntax, and panic recovery
remain focused future work.
