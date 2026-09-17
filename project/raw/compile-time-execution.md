# Raw input: compile-time execution

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work defining compile-time execution, execution context, constant availability, or capability-dependent static selection |
| Applies To | Directed/inferred compile-time execution, host/target context, target-format realization, and static support questions |
| Owns | Preserved compile-time execution, target-format realization, capability-dependent static selection, representative source, activation pressure, and retirement questions |
| Does Not Own | Accepted compile-time semantics or current integer/fixed/floating/literal/reflection behavior |
| Source / Provenance | Legacy `compiler-directives.md` and `meta-functions.md` evidence together with operator-phrase review of type receivers, `is constant`, and native execution context |

## Why this input exists

Current documentation says only that compile-time execution is *directed and
inferred* under the ordinary compile-time function model, that a type-receiver
operation is not inherently compile-time, and that a compile-time language-level
panic becomes a compiler diagnostic. Nothing else about compile-time execution is
accepted, yet several accepted concepts depend on an execution context existing.

This file keeps those dependencies visible.

## Current literal boundary

[Zax literal source and literal operators](../../language/literal-source-and-operators.md)
now requires literal operators and literal-result `<+>` joins to execute during
compilation.

The current local contract already fixes:

- one concrete generic result specialization before a literal body runs;
- active execution context, with target/compiler-host facts selected explicitly;
- host-locale-independent source and numeric parsing;
- deterministic results from payload, declaration, explicit static inputs,
  selected profile/environment, and versioned language/library data;
- no mutable compile-time state that makes identical literal results depend on
  source order;
- no compiler allocation leaking into the produced value;
- deliberate rejection and language panic becoming source diagnostics;
- resource failure not becoming candidate fallback; and
- no retry of a runtime or weaker `<+>` declaration after selected
  compile-time unavailability.

This input retains the general mechanism: exact availability syntax, literal
rejection APIs and payload offsets, scheduling, capabilities, sandboxing,
caching, allocation policy, recursion/termination limits, filesystem/network
access, and work budgets.

## Exact uncommitted real evaluation

Current literal design calculates the initial uncommitted-real operation family
as exact finite rationals and rounds once at commitment. Decimal and binary-power
source, including `1.0 / 3.0`, does not use a hidden compiler-host floating
format.

Compile-time work must preserve:

- exact grouping, sign, `+`, `-`, `*`, rational `/`, equality, and ordering;
- zero-divisor diagnostics;
- exact scaled source without expanding repeating decimal digits;
- destination-aware algorithms only when they prove the same correctly rounded
  result as exact evaluation followed by one realization;
- capacity/work exhaustion as a compiler diagnostic rather than hidden
  intermediate rounding; and
- a concrete commitment requirement before square root, transcendental, or
  representation-dependent operations.

Future work must define practical capacity guarantees, algorithm-independent
resource diagnostics, caching, and how a programmer establishes an earlier
concrete boundary to control cost.

Generated fixed/floating mathematical constants apply the same requirement:
`T.pi`, `T.e`, and peers are rounded directly from their definitions into `T`,
not copied from compiler-host constants or derived from another rounded type.
Floating `inf`, `ninf`, `nan`, `snan`, and `nzero` are exact type-owned format
values whose availability follows the concrete format.

## Directed and inferred execution

Legacy material directs execution with a directive option set:

```zax
main final [[execute=target]] : ()() = {
}

double final [[execute=dual]] : (result : Integer)(value : Integer) = {
    return value * 2
}

four := double(2) // inferred: every input is a compile-time constant
```

The legacy options are `generate`, `delegate`, `dual`, `host`, and `target`.
Future work must decide:

- which options survive and what they mean;
- how inference decides that a call executes at compile time;
- whether the programmer can require, forbid, or merely prefer an execution
  context;
- how execution interacts with resolution order and forward declarations; and
- what happens when a directed compile-time execution cannot be satisfied.

## Native, compiler-host, and target execution context

Current documentation defines **native endianness** as the byte order of the
environment whose execution semantics are currently in effect, and distinguishes
it from compiler-host and target byte order. See
[Zax endianness](../../language/endianness.md).

That definition presumes a general answer to a question compile-time work must
settle: *which environment's semantics are in effect during a given evaluation?*

Legacy material offers `host` and `target` prefixed variants:

```zax
size of
host size of
target size of

is constant
is host constant
is target constant
```

Future work must distinguish:

- actual compiler-host execution;
- target-context evaluation performed by the compiler;
- ordinary target runtime; and
- whether these are three contexts or two contexts with a switchable target
  model.

`native` must always mean the environment whose execution semantics are currently
active, never an unspecified build machine.

## Context-sensitive layout and constant queries

The reserved forms `size of`, `alignment of`, `offset of`, and `is constant`
return a result "in the active execution context". See the
[operator catalog](../../language/operator-catalog.md#reserved-phrase-forms).

Future work must decide:

- whether contextual variants are separate forms or one form plus a context;
- whether a host result and a target result may both be requested from one
  source location;
- how a mismatch between host and target layout is diagnosed; and
- how unusual widths and padding participate.

## `is constant`

Constancy is an expression-evaluation fact rather than part of type identity:

```zax
compileValue := 42
runtimeValue := readInteger()
// Both may have type Integer; only compileValue is constant.
```

Future work must decide what makes an expression "available as a compile-time
constant", whether availability depends on the requesting context, and how the
answer interacts with directed execution and resolution order.

## Type-receiver execution

A type-receiver operation is discovered through a concrete type identity and is
**not** inherently compile-time:

```zax
instance := factory create MyType
```

Future work must decide:

- when such an operation may execute at compile time;
- whether a type argument alone ever forces compile-time execution;
- what a compile-time type-receiver operation may observe; and
- whether the answer differs for language-provided versus user-declared
  type-receiver operations.

## Concrete results and constant inputs

Even when the compiler evaluates a function early, the call keeps the result
type written in its prototype:

```zax
compileTimeResult final : (result : Integer)() = {
  return 42
}

myValue := compileTimeResult() // concrete Integer
myByte : Byte = compileTimeResult() // error: result remains Integer
```

Current number-literal type selection is owned by
[Zax integer literals and realization](../../language/integer-literals.md).

A number literal can take its type from a selected function parameter or result:

```zax
makeByte final : (result : Byte)() = {
  return 42
}
```

Here `42` becomes a `Byte` while filling the declared result. The caller still
receives a concrete `Byte`; compile-time execution does not return a hidden
typeless number.

Constant-only inputs may make a selected operation eligible or attractive for
compile-time execution:

```zax
myValue := calculate(42)
```

They do not by themselves mandate it. Future work must distinguish required,
permitted, inferred, preferred, and optimization-only execution without
changing candidate selection or reopening concrete results.

## Target-format real realization

Current [fixed-point](../../language/fixed-point-scalars.md) and
[floating-point](../../language/floating-point-scalars.md) design parses typed
real source mathematically and realizes it according to the selected
destination. Compile-time execution must:

- use target format, endianness, rounding, range, and validity when the
  destination is target-selected;
- use compiler-host semantics only for an explicitly compiler-host-selected
  destination;
- avoid using host hardware arithmetic as an accidental oracle for target bits;
- preserve MBF40's historical rounding when that exact destination is selected;
  and
- respect provider-relative X87 behavior only in the environment whose profile
  declares it.

A compile-time-known real result remains its selected concrete scalar type. It
does not reopen as a typeless mathematical value at a later assignment.

## Capability-dependent static selection

An exact scalar type may exist while one environment lacks its numeric
operations. Future compile-time and generic work must let source inspect:

- complete core support;
- one exact operation's availability;
- software, guaranteed hardware, runtime-optional hardware, or trap-emulated
  classification;
- selected exact format and endianness; and
- language/profile size limits.

A statically discarded branch may name an unsupported operation without
demanding executable support. A selected branch that uses it receives an
availability diagnostic.

Current optional `Legacy.WChar`/`Legacy.WideString` adds declaration-resolution
pressure. Future compile-time/reflection design needs a static enclosed-source
or declaration-path query whose payload may mention an unavailable name without
producing an ordinary compilation error. A rejected payload:

- contributes no declarations or runtime behavior;
- does not execute ordinary side effects;
- reports `false` or another static unavailability result;
- preserves target versus compiler-host context; and
- remains symbol-aware rather than depending only on a fragile string path.

Exact source syntax remains open.

Generative import injection adds a concrete module use. An imported module may
ask whether its protected `Module` root already contains an injected dependency
or option and declare/import a fallback only when absent:

```zax
// Illustrative only; exact availability and conditional-source syntax is open.
if !compiles { Module.UseComplexMath } {
  UseComplexMath :: import Module.DefaultComplexMath
}
```

The query and selected branch become semantic inputs to that module instance.
Future work must preserve:

- symbol-aware lookup under the imported module's own root;
- target-module ownership for declarations selected by the branch;
- lexical identity of an injected alias from the importer;
- no declarations or dependency edges from a statically discarded branch;
- cycle detection for the selected fallback import;
- no ordinary unknown-name diagnostic merely for asking the question; and
- cache invalidation when injection changes the answer.

Current literal design permits a generic `uncommitted` scalar result slot.
Compile-time/generic processing selects one concrete result prototype before the
literal body is processed and invoked. A fixed suggestion such as `UInteger`
does not automatically widen after range failure.

Future bounded type probing may compute another suggested specialization from
payload magnitude. In particular, a numeric literal declaration may parse
`h'FF'` and a much longer payload into different exact magnitudes that need
different concrete result widths.

More generally, generic code may begin with an ordinary size and deliberately
test larger candidates until a supported conversion/type or declared limit is
reached. This must not become runtime exception fallback, source-order overload
search, reopening of a resolved prefixed literal, or mandatory construction of
one extreme maximum-width value.

## Compile-time availability and failure

Accepted today: a language-level panic during compile-time execution becomes a
compiler diagnostic with the operation, source location, relevant operands, and
compile-time evaluation path where practical. It is not an internal compiler
crash and does not imply a compile-time panic handler.

Future work must decide:

- what makes a function unavailable for compile-time execution;
- how an unavailable call is diagnosed versus deferred to runtime;
- whether compile-time execution may allocate, and under what policy;
- resource, recursion, and termination limits; and
- whether a failure is recoverable in any sense.

## Constraints this input places on current work

- current documentation must not treat a type-receiver operation as
  compile-time by default;
- `native` must not be used interchangeably with `host` or `target`; and
- contextual layout and constant queries must be preserved as context-dependent
  rather than completed here; and
- compile-time evaluation of an already concrete value must not silently change
  its type, conversion, identity, or operator-selection rules; and
- target-format real evaluation must not silently use compiler-host floating
  semantics; and
- unsupported scalar branches require a defined static-selection boundary
  rather than being accepted or rejected according to optimizer accident; and
- uncommitted integer realization must complete before an integer value crosses
  into runtime execution.

Current [enum members](../../language/enums.md#members) accept integer constants
resolved while the enum definition completes. Future compile-time execution may
expand how those constants are produced, but it must preserve definition-time
availability, exact backing representability, deterministic member order, and
compile-time diagnostics.

## Activation and retirement

Activate this input when compile-time execution, execution context, host/target
distinction, constant availability, target-format real realization,
capability-dependent static selection, or bounded type probing is reviewed.
Move accepted behavior into compile-time, invocation, diagnostics, generic, and
the applicable domain owners, then retire this file once every preserved
question has an accepted owner or explicit deferral.
