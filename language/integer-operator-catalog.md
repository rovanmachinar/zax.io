# Zax integer operator catalog

| Field | Value |
| --- | --- |
| Status | Current conceptual design |
| Audience | Human developers looking up protected behavior for Zax integer operations |
| Applies To | Built-in finite integer operations; not custom overload behavior, formal grammar, or a specification |
| Implementation State | Not established by this repository |
| Owns | Protected integer operand domains, availability, result identities, policy behavior, failure, mutation, conversion/admission, count/mask/shift behavior, `delta`, `distance`, and unchecked arithmetic contracts |
| Does Not Own | Uncommitted integer evaluation and realization ([integer literals and realization](integer-literals.md)); integer type choice and representation ([integers](integers.md)); exact forms/fixity/precedence ([operator catalog](operator-catalog.md)); shared discovery/selection ([operators](operators.md)); or general identity behavior ([identity types](identity-types.md)) |
| Source / Provenance | Integer behavior formerly embedded in the general operator catalog, reorganized after fundamental-integer review |

## How to use this catalog

[Zax integers](integers.md) explains which integer type to choose and how its
range, representation, identity, and storage behave.

This catalog answers what a protected operation does after its integer operands
and result domain are known.

The [general operator catalog](operator-catalog.md) remains authoritative for:

- recognized symbolic and phrase forms;
- fixity;
- precedence and association;
- reserved versus open forms; and
- source-tree formation.

Custom overloads may assign domain-specific behavior and result shapes where the
form is open. The rules below apply only to protected integer signatures.

### Number literals before a concrete type

Compare:

```zax
myMathematicalResult : U8 = 200 + 55
myTypedResult := (: U8 = 200) + 55
```

In the first declaration, `200 + 55` is calculated mathematically before the
final `255` is checked against `U8`. In the second, the left operand is already
`U8`, so the addition uses the concrete `U8` overflow rules in this catalog.

Plain negation, magnitude, arithmetic, division, remainder, mathematical
shifts, width-invariant binary bitwise operations, and comparison can operate
on not-yet-typed number literals as described by
[Zax integer literals and realization](integer-literals.md). Complement is
available there only when value and sign intent produce one result at every
fitting width.

This catalog describes operations after an exact or role-selected integer type
is known, including finite overflow, policy suffixes, optional/reporting
results, mutation, finite-width bitwise behavior, shifts, and associated result
types.

A typed operand can also determine the integer type for a number literal on its
other side:

```zax
myValue := 0 + (: U8 = 5)
```

The right operand is `U8`, so `0` is checked as though the programmer wrote
`(: U8 = 0)`. Ordinary concrete `U8 + U8` behavior then applies. This special
relationship belongs to the built-in integer operation; the right operand still
does not contribute custom declarations.

## Shared integer-operation rules

Unless a family below says otherwise:

- value operands use the same integer identity;
- each runtime operand evaluates once from left to right;
- the result retains the operand integer identity;
- no implicit widening, narrowing, signedness change, or role conversion occurs;
- an identity-preserving alias is the same operand type;
- a distinct integer role identity remains distinct even when its representation
  matches another role;
- compile-time execution uses the same value semantics; and
- a compile-time-known panic becomes a compiler diagnostic.

Dedicated operand roles are explicit exceptions. Shift counts use an accepted
unsigned count type, and an integer's associated bit-count type is always
directly accepted.

## Addition, subtraction, and multiplication

For operands of one integer identity `T`:

| Policy | Addition example | Result |
| --- | --- | --- |
| Required | `myLeft + myRight` | Exact `T`; panic when unrepresentable |
| Optional | `myLeft +? myRight` | `T?`; absent when unrepresentable |
| Wrapping | `myLeft +% myRight` | Modular `T` |
| Saturating | `myLeft +\| myRight` | `T` clamped to its nearest bound |
| Combined report | `myLeft +! myRight` | Wrapped value, saturated value, and overflow status |
| Wrapped report | `myLeft +!% myRight` | Wrapped `T` plus overflow status |
| Saturated report | `myLeft +!\| myRight` | Saturated `T` plus overflow status |

Subtraction and multiplication use the corresponding forms and behavior.

The specialized report contains:

```zax
// Conceptual anonymous result shape.
<anonymous> :: type {
  value : T
  overflowed : Boolean
}
```

The combined report contains:

```zax
// Conceptual anonymous result shape.
<anonymous> :: type {
  wrapped : T
  saturated : T
  overflowed : Boolean
}
```

`T` means the selected integer identity, not the default `Integer` role.

Through the protected presence/status behavior:

- `?report` means no overflow; and
- `!report` means overflow.

Exact anonymous structural identity remains future structural-type work.

## Signedness counterpart

Pre-unary `+` requests the equal-width opposite-signedness counterpart:

```zax
myUnsigned : U8 = 100
mySigned := +myUnsigned // I8 value 100
```

| Policy | Behavior |
| --- | --- |
| `+myValue` | Exact counterpart; panic when unrepresentable |
| `+?myValue` | Optional counterpart; absent when unrepresentable |
| `+%myValue` | Preserve the logical-width bit pattern |
| `+\|myValue` | Clamp to the counterpart's nearest bound |
| `+!myValue` | Combined counterpart report |

Exact intrinsic types derive the counterpart by changing signedness at equal
logical width.

Named role pairs such as `Small`/`USmall` and `FastI16`/`FastU16` return the
declared paired identity.

One-sided roles such as `Word` return the unnamed exact intrinsic counterpart
and leave their role identity.

For an uncommitted integer, pre-unary `+` toggles sign intent: unknown becomes
unsigned, unsigned becomes signed, and signed becomes unsigned. That
pre-commitment behavior is defined by
[Zax integer literals and realization](integer-literals.md).
For a concrete integer, the counterpart behavior in this section applies.

## Negation and magnitude

### Negation

Protected signed integers support:

```zax
-myValue
-%myValue
-|myValue
-?myValue
-!myValue
-!%myValue
-!|myValue
```

The minimum signed value has no positive value in the same type:

- required negation panics;
- wrapping negation retains the minimum value's bit pattern;
- saturating negation returns the signed maximum;
- optional negation is absent; and
- reporting forms return the selected policy value and overflow status.

Protected unsigned integers have no ordinary unary negation.

### Magnitude

| Policy | Form | Signed minimum behavior |
| --- | --- | --- |
| Required | `\|myValue\|` | Panic |
| Wrapping | `\|%myValue\|` | Retain minimum representation |
| Optional | `\|?myValue\|` | Absent |
| Combined report | `\|!myValue\|` | Report wrapped/saturated alternatives and overflow |
| Saturating | `intent<asymmetric-saturating-magnitude>{ \|\|myValue\| }` | Return the signed maximum |

Unsigned magnitude is the same value. Optional and reporting forms remain
available so generic numeric code can use one policy.

The norm-like `||myValue||` form has no protected integer operation.

For every signed value except the minimum, saturating magnitude is the ordinary
positive magnitude. For the minimum signed value, it returns the signed maximum.
For unsigned values, it returns the same value.

Natural `||myValue|` is an intent error because it resembles a malformed norm.
The registered intent category explicitly acknowledges the asymmetric form:

```zax
myMagnitude :=
  intent<asymmetric-saturating-magnitude>{ ||myValue| }
```

The acknowledgement contract is defined by
[Zax intent acknowledgements](intent-acknowledgements.md).

## Division and remainder

```zax
myQuotient := myDividend / myDivisor
myOptionalQuotient := myDividend /? myDivisor

myRemainder := myDividend % myDivisor
myOptionalRemainder := myDividend %? myDivisor
```

Required division panics when:

- the divisor is zero; or
- a signed minimum value is divided by `-1`.

Optional division is absent for either condition.

Successful signed division truncates toward zero.

Required remainder panics only when the divisor is zero. Signed minimum
remainder `-1` is zero. Any other nonzero signed remainder has the dividend's
sign.

No wrapping, saturating, or reporting division/remainder value forms are
currently defined.

## Compound arithmetic

Every compound evaluates the destination place and right operand once. It is a
direct operation, not a rewrite through the value operator and `=`.

| Policy | Addition form | Destination behavior |
| --- | --- | --- |
| Required | `+=` | Write exact result; panic leaves destination unchanged |
| Optional | `+?=` | Write on success; otherwise unchanged |
| Wrapping | `+%=` | Always write modular result |
| Saturating | `+\|=` | Always write clamped result |
| Combined report | `+!=` | Write exact result on success; otherwise unchanged; return report |
| Wrapped report | `+!%=` | Always write wrapped result and report overflow |
| Saturated report | `+!\|=` | Always write saturated result and report overflow |

Subtraction and multiplication use the same policy suffixes.

Ordinary successful compounds return writable access to the updated destination.
Specialized reporting forms return their report.

Division provides `/=` and `/?=`. Remainder provides `%=` and `%?=` with the
same failure conditions as their value operations.

## Increment and decrement

Both pre and post fixities support:

```text
++   ++?   ++%   ++|   ++!   ++!%   ++!|
--    --?   --%   --|   --!   --!%   --!|
```

Required increment/decrement panics at the representable boundary and leaves the
destination unchanged. Optional forms update only on success. Wrapping and
saturating forms always write their policy result.

Ordinary pre forms return writable access to the updated place:

```zax
myUpdated := ++myValue
```

Ordinary post forms return the previous value by `copy`:

```zax
myPrevious := myValue++
```

Reporting forms return one transition report containing:

- previous `T`;
- wrapped `T`;
- saturated `T`; and
- `Boolean` overflow status.

Specialized reports contain the previous value, selected policy value, and
overflow status.

## Comparison

Same-identity integers support:

```zax
myLeft == myRight
myLeft != myRight
myLeft < myRight
myLeft <= myRight
myLeft > myRight
myLeft >= myRight
```

Each returns `Boolean`.

Ungrouped comparison chains remain a source intent error under the general
operator catalog.

The three-way comparison form `<=>` has no protected integer result until its
ordering result type is established.

## Bitwise operations

Every protected bitwise operation observes exactly the logical bit width.
Non-value storage bits never participate.

### Value operations

| Operation | Form | Result |
| --- | --- | --- |
| Complement | `~myValue` | Same integer identity |
| AND | `myLeft & myRight` | Same integer identity |
| OR | `myLeft \| myRight` | Same integer identity |
| XOR | `myLeft ^ myRight` | Same integer identity |
| AND-NOT | `myLeft &~ myRight` | Same integer identity |
| NAND | `myLeft bitwise nand myRight` | Same integer identity |
| XNOR | `myLeft bitwise xnor myRight` | Same integer identity |
| NOR | `myLeft bitwise nor myRight` | Same integer identity |
| OR-NOT | `myLeft bitwise or not myRight` | Same integer identity |

Direct mutation uses `~=`, `&=`, `|=`, `^=`, `&~=`, and the exact phrase
mutation forms recorded by the general catalog.

### Counts and reductions

| Operation | Form | Result |
| --- | --- | --- |
| Population count | `#myBits` | `T bit count type` |
| Leading-zero count | `#<myBits` | `T bit count type` |
| Trailing-zero count | `#>myBits` | `T bit count type` |
| Leading set-bit mask | `#~<myBits` | Same integer identity |
| Trailing set-bit mask | `#~>myBits` | Same integer identity |
| Reduction AND | `#&myBits` | `Boolean` |
| Reduction OR | `#\|myBits` | `Boolean` |
| Reduction XOR/parity | `#^myBits` | `Boolean` |
| Reduction NAND | `#~&myBits` | `Boolean` |
| Reduction NOR | `#~\|myBits` | `Boolean` |
| Reduction XNOR | `#~^myBits` | `Boolean` |

For zero, leading- and trailing-zero counts return the complete logical width.

Set-bit masks preserve one bit at its original position:

- leading mask preserves the most-significant set bit;
- trailing mask preserves the least-significant set bit; and
- zero produces zero.

Signed mask results retain the signed identity. Preserving a sign-bit mask is a
bit-pattern result, not arithmetic overflow.

Dedicated leading/trailing-one counts remain derivable:

```zax
myLeadingOnes := #<(~myBits)
myTrailingOnes := #>(~myBits)
```

### Set-bit positions

```zax
myLeadingPosition := myBits leading set bit position
myTrailingPosition := myBits trailing set bit position
```

The result is optional `T bit count type`.

Position zero is the least-significant bit. Zero has no set-bit position and
returns absent.

```zax
myPosition := myBits trailing set bit position

if ?myPosition
  myShifted := myValue << myPosition.
```

The associated count payload is directly accepted by the applicable protected
shift.

## Shifts and rotations

| Operation | Form |
| --- | --- |
| Left shift | `myValue << myCount` |
| Type-natural right shift | `myValue >> myCount` |
| Explicit zero-fill right shift | `myValue >>> myCount` |
| Rotate left | `myValue <<% myCount` |
| Rotate right | `myValue >>% myCount` |

The count is an accepted unsigned integer. `T bit count type` is always directly
accepted for `T`.

For logical width `W`:

- logical shifts by a count at least `W` return zero;
- signed type-natural right shift sign-fills and returns all sign bits for a
  count at least `W`;
- unsigned type-natural right shift and every `>>>` zero-fill;
- rotations use `count mod W`;
- outgoing bits are discarded; and
- the result never widens implicitly.

Compound forms append `=`.

The exact modulo-count shift phrases reduce the count modulo `W` before shifting:

```zax
myValue shift left modulo myCount
myValue shift right modulo myCount
myValue logical shift right modulo myCount
```

They are not rotations. Their exact mutation phrases are recorded by the general
operator catalog.

Multiword shift/rotate/funnel operations, reversal, and masked
extraction/deposit remain future numeric work.

## Integer conversion and admission

### Exact intrinsic conversion

Plain `as` may contextually give a number literal its written destination type:

```zax
myByte := 55 as U8
myError := 355 as U8 // error: 355 does not fit U8
```

Sign intent must permit construction of that destination. The optional and
narrowing forms require an already concrete receiver because otherwise their
apparent policy could not handle contextual construction failure:

```zax
myOptional := 55 as U8? // error: optional `as` requires a concrete receiver
myNarrow := 355 narrowing as U8 // error: narrowing requires a concrete receiver
```

Once the receiver is concrete, the ordinary policies apply:

```zax
myWide := myU16 as U32
myOptional := myU16 as U8?
myNarrow := myU16 narrowing as U8
```

- `as Destination` requires a declared type fact or compile-time-known value
  proving that the exact value fits.
- `as Destination?` returns present when the exact value fits and absent
  otherwise.
- `narrowing as Destination` performs defined modular conversion.
- `unsafe as` is not ordinary integer narrowing.

The narrowing rule is:

> Preserve the source integer modulo `2^W`, where `W` is the destination logical
> width, then interpret those bits under destination signedness.

Signed widening sign-extends. Unsigned widening zero-extends. Complete optional
result behavior is defined by
[Zax optional values](optional-values.md#construction-and-optional-conversion-are-different).

### Integer identity admission

```zax
myLong := Long from myI32
mySmall := Small optional from myU16
myNarrowSmall := Small narrowing from myU16
```

- `from` requires complete source-range containment;
- `optional from` returns absent when the exact value does not fit; and
- `narrowing from` applies modular conversion before creating the admitting
  integer identity.

Conversion availability follows declared family facts rather than accidental
representation equality in one CPU profile.

Restricted identity validation, unchecked admission, unsafe admission,
projection, and direct bridges are defined by
[Zax identity types](identity-types.md).

## Exact delta and distance

Both operations require the same integer identity and evaluate operands left to
right.

### `delta`

```zax
myDelta := myLeft delta myRight
```

`delta` returns `T delta type` and produces exact mathematical
`myLeft - myRight`.

For logical width `W`, the result type covers:

```text
-(2^W - 1) through +(2^W - 1)
```

The CPU profile may select the next suitable native width or an optimized
software-emulated width rather than exact `W + 1`.

`delta` is unavailable when no language-supported result type covers the range,
including for a maximum-width integer.

### `distance`

```zax
myDistance := myLeft distance myRight
```

`distance` returns unsigned `T distance type` and produces exact absolute
difference without intermediate overflow.

For logical width `W`, its result covers:

```text
0 through 2^W - 1
```

It remains available at the language maximum.

Neither operation has wrapping, saturating, optional, reporting, compound, or
mutation variants.

Pointer delta/distance require future pointer comparability and validity rules.

## Assignment and swap

Protected ordinary assignment:

```zax
myDestination = mySource
```

- requires one integer identity;
- evaluates the existing destination once;
- updates through writable access; and
- returns writable access to the destination.

Declaration initialization remains language syntax rather than assignment.

Swap:

```zax
myLeft <<>> myRight
```

- evaluates both writable places left to right;
- requires compatible integer identities;
- exchanges their values;
- returns zero results; and
- treats exact self-swap as a no-op.

## Unchecked build contracts

Required operations normally retain their panic checks in every optimization
mode.

### Required arithmetic representability

An explicit build setting may promise:

> Every required arithmetic result is representable by its result type.

It covers required addition/subtraction/multiplication value and compound forms,
increment/decrement, minimum signed negation/magnitude, and signed minimum
divided by `-1`.

When the promise holds, the operation produces its exact result. If it is
violated, the result and surrounding behavior are undefined. The operation does
not become implicitly wrapping.

### Nonzero divisors

A separate setting may promise:

> Every divisor supplied to required division or remainder is nonzero.

It covers `/`, `%`, `/=`, and `%=`. Violation has undefined behavior.

Both settings:

- are off by default;
- are independent from debug/release optimization mode;
- remain visible in build metadata and tooling;
- retain diagnostics for compile-time-known violations; and
- do not affect optional, wrapping, saturating, reporting, or narrowing forms.

Exact option/directive syntax remains future analysis-control work.

## Costs and diagnostics

Tooling should expose:

- selected integer identity and associated result type;
- required checks or unchecked build contracts;
- optional, wrapping, saturating, reporting, and narrowing policy;
- eager operand order;
- destination mutation and unchanged-on-failure behavior;
- copies, writable references, previous-value snapshots, and reports;
- native versus software-emulated operation;
- invalid count type;
- unavailable signed/unsigned form;
- unavailable maximum-width `delta`;
- unproved exact conversion; and
- failed identity admission.

Diagnostics lead with public/source type names and provide exact specialization
detail on demand.

## Boundaries and maturity

This catalog is current conceptual design, not formal grammar, a diagnostic
contract, or an implementation mapping.

Exact forms and precedence remain in the
[general operator catalog](operator-catalog.md). Multiword operations,
reversal, masked extraction/deposit, pointer difference validity, recoverable
panic, build-option syntax, and exact anonymous report identity remain future
work.
