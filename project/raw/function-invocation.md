# Raw input: function invocation

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work defining ordinary function-call syntax, argument mapping, parameter binding, overload selection, and temporary lifetime |
| Applies To | Construction-derived constraints that should inform ordinary invocation without importing stored-member construction syntax |
| Owns | Preservation of examples, aligned construction behavior, function-facing pressures, activation conditions, and retirement criteria |
| Does Not Own | Accepted function-call syntax, complete parameter defaults, overload ranking, result-context selection, move/copy semantics, or implementation |
| Source / Provenance | Work item `005`, construction-packet and argument-evaluation review |

## Reading posture

[Zax construction, replacement, and destruction](../../language/construction-and-destruction.md#construction-packets)
defines current construction-packet behavior. Future ordinary function calls
should reuse the applicable evaluation and binding model without inheriting
stored-member entries or necessarily using packet punctuation.

This file is not required for ordinary construction use.

## Construction behavior to preserve

Construction packets establish:

- strict left-to-right argument-expression evaluation;
- immediate parameter or input-slot binding during each entry;
- copied inputs capturing the source value at their evaluation point;
- references binding at evaluation while observing later referent changes;
- move and `last` effects occurring at their evaluation point;
- declarations being usable as argument expressions;
- named arguments establishing a positional cursor for following positional
  arguments;
- candidate-specific mapping and ordinary ambiguity diagnostics; and
- temporary lifetime long enough to complete the selected lifecycle call.

General function work should start from those constraints rather than choosing
different invisible evaluation behavior.

## Evaluation and binding example

Current construction syntax:

```zax
source : Integer = 1

value : Example = [{
    : source,
    action: changeSourceToTwo(source)
}]
```

If the first input copies, it captures `1` before `changeSourceToTwo` runs. If it
binds a reference, the call body later observes `2`.

Future function-call syntax must express an equivalent call clearly. The
following is illustrative only:

```zax
consume(
    : source,
    action: changeSourceToTwo(source)
)
```

The future review may choose different punctuation. It should not change the
left-to-right evaluation and immediate binding model without concrete pressure.

## Declaration-expression example

An argument expression may introduce a named temporary:

```zax
[{
    : temporary : String,
    option: otherTemporary : Options
}]
```

Future function syntax must preserve the distinction between:

- the invocation marker or argument label; and
- the declaration expression that produces the input value.

Formatting may add parentheses when nesting is difficult to read.

## Named and positional mapping

Construction establishes a cursor model:

1. positional input binds the current parameter and advances;
2. named input binds that parameter and resets the cursor after it; and
3. an unresolved equal candidate set is an error.

Ordinary calls have no `.member = expression` entry. They therefore do not need
the construction rule that a member entry invalidates the cursor.

Overload contracts can remain advanced and closely related. Adding or changing a
function may make another candidate a better match without producing an
ambiguity. Zax diagnoses unresolved selection but cannot prevent every logic
mistake caused by poorly distinguished overloads.

## Temporary lifetime

A temporary bound to a reference parameter must survive through completion of
the call. By-value, move, and `last` inputs require destruction and transfer
points that preserve their observable evaluation-time behavior.

Compilers may elide storage or operations only when construction, destruction,
side effects, and selected results remain unchanged.

Complete full-expression, result, capture, closure, and async-call lifetime
rules remain future function and lifetime work.

## Defaults and omitted arguments

Constructors and replacement constructors use the eventual ordinary parameter
default and omission model. Future function work must define:

- default-expression declaration syntax;
- when a default expression evaluates;
- how it participates in left-to-right ordering;
- interaction with named and following positional inputs;
- candidate viability when defaults differ; and
- source compatibility when an overload changes.

## What this input does not decide

This file does not establish:

- ordinary function-call punctuation;
- whether calls use leading `:` markers;
- complete named-argument syntax;
- default-argument syntax;
- arbitrary overload ranking;
- result-context selection;
- move/copy/`last` contracts;
- temporary representation; or
- compiler lowering.

## Activation and retirement

Activate this input when focused work begins on ordinary function invocation,
parameters, defaults, overloads, temporaries, or result mapping.

That work must:

1. preserve or explicitly revise each construction-derived constraint;
2. provide concrete named, positional, declaration-expression, reference, copy,
   move, and temporary examples;
3. separate accepted general rules from construction-only behavior;
4. move accepted behavior into the future function and lifetime owners; and
5. remove this file from the raw index and retire it when no unresolved material
   remains.
