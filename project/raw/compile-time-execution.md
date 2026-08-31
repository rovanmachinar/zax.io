# Raw input: compile-time execution

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work defining compile-time execution, execution context, or constant availability |
| Applies To | Directed and inferred compile-time execution and the host/target context questions it raises |
| Owns | Preserved compile-time execution questions, representative source, activation pressure, and retirement criteria |
| Does Not Own | Accepted compile-time semantics or current integer/literal/reflection behavior |
| Source / Provenance | Legacy `compiler-directives.md` and `meta-functions.md` evidence together with operator-phrase review of type receivers, `is constant`, and native execution context |

## Why this input exists

Current documentation says only that compile-time execution is *directed and
inferred* under the ordinary compile-time function model, that a type-receiver
operation is not inherently compile-time, and that a compile-time language-level
panic becomes a compiler diagnostic. Nothing else about compile-time execution is
accepted, yet several accepted concepts depend on an execution context existing.

This file keeps those dependencies visible.

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

## Deferred integer realization

Current programmer-visible integer realization boundaries are owned by
[Zax integers](../../language/integers.md#initialization-and-compile-time-realization).
This section retains the future compile-time result mechanism.

A compile-time operation may produce an integer whose concrete realization
remains open until a typed slot requests a width and signedness:

```zax
myValue := compileTimeResult()
myByte : Byte = compileTimeResult()
```

The typed declaration directly realizes `myByte` when the value fits and reports
a compile-time error otherwise. This is not implicit conversion from a hidden
preselected `Integer`.

An ordinary expression already typed as `Integer` remains `Integer` even when
the compiler proves it constant. Constancy alone does not grant another
conversion surface. Future work must define the compiler-known unrealized
integer category, result-polymorphic alternative, or other mechanism preserving
that distinction.

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
- compile-time evaluation of an already concrete integer must not silently
  change its type-conversion rules.

## Activation and retirement

Activate this input when compile-time execution, execution context, host/target
distinction, or constant availability is reviewed. Move accepted behavior into
compile-time, invocation, diagnostics, and the applicable domain owners, then
retire this file once every preserved question has an accepted owner or explicit
deferral.
