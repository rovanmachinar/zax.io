# Raw input: bare source enclosure

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work defining operator phrases, keyword-neutral source, confusable-form intent, or transparent source enclosures |
| Applies To | The complete candidate `bare{...}` model and its source, scope, continuation, operator, and mixfix consequences |
| Owns | Preservation of the candidate mental model, representative valid/invalid source, constraints, alternatives, activation pressure, and retirement criteria |
| Does Not Own | Accepted `bare` syntax, keyword behavior, phrase fencing, operator grammar, source layout, scope, or implementation |
| Source / Provenance | Operator-catalog promotion review and follow-up source-intent analysis |

## Why this input exists

Some intent problems can be repaired with ordinary whitespace, grouping, or
layout:

```zax
!(!value)
```

Other coherent forms cannot retain their intended source through those remedies.
The candidate asymmetric saturating-magnitude circumfix is:

```zax
||value|
```

Natural source strongly resembles a norm expression missing its final `|`.
Future work needs a general way to acknowledge that form without making it
unsafe or permanently prohibiting it.

Operator phrases create related lexical pressure when words could otherwise be
keywords:

```zax
run value fast
```

## Candidate mental model

`bare{...}` is a non-scoping, keyword-neutral, tree-transparent source
enclosure.

Its payload must independently form exactly one complete:

- expression; or
- effective statement.

After that completeness check, the enclosure:

- creates no runtime operation;
- creates no scope or destruction boundary;
- creates no final precedence boundary;
- creates no expression-tree node or mixfix hole;
- neutralizes non-operator keyword roles;
- permits recognized confusable-form operators;
- and leaves grammar, operator selection, qualification, protection, lifetime,
  and diagnostics otherwise active.

This is candidate behavior, not current Zax syntax.

## Exact contextual recognition

The candidate uses one contiguous opener:

```zax
bare{ ... }
```

Bare `bare` is not a keyword. This is not the same form:

```zax
bare { ... }
```

Adjacency distinguishes the transparent enclosure from a keyword followed by an
ordinary block.

## Expressions and final tree formation

The payload is independently complete, but its provisional internal parse is not
preserved as a subtree:

```zax
a = b + c * d
a = bare{ b + c } * d
```

Both form:

```zax
a = b + (c * d)
```

Ordinary parentheses remain the grouping mechanism:

```zax
a = (b + c) * d
```

The enclosure is also transparent to wider mixfix matching:

```zax
a = a + bare{ b * c / d }
```

A mixfix may match components inside and outside the former bare boundary after
payload validation.

The payload cannot borrow missing syntax from outside:

```zax
c = bare{ a + b + } d // error: incomplete payload
```

The candidate processing model is:

1. recognize `bare{...}` and determine whether the surrounding position requires
   an expression or effective statement;
2. parse the payload independently with non-operator keyword roles neutralized
   and recognized confusable forms enabled;
3. require one complete unit of that category;
4. retain lexical-role and confusable-form acknowledgement on its tokens;
5. remove the enclosure as a precedence, tree, hole, scope, and runtime boundary;
6. form the complete surrounding tree normally; and
7. perform ordinary and mixfix selection without weakening semantic rules.

## One effective statement

Several operations are legal only when `;` composes them into one effective
statement:

```zax
bare{
  myFunc();
  myOtherFunc()
}
```

This remains invalid even where the surrounding source accepts a statement list:

```zax
bare{
  myFunc()
  myOtherFunc()
}
// error: two uncomposed statements
```

Inline and multiline forms have identical semantics:

```zax
bare{ myFunc(); myOtherFunc() }
```

The multiline opener permits physical layout but does not turn newline into
semicolon composition.

## Flow-header initialization

The candidate may contain one composed initializer:

```zax
if bare{
  i : Integer = 0;
  k : Integer = 1
} ;; condition {
}
```

The bindings belong to the ordinary flow-header scope. The bare closer does not
end their lifetimes.

These remain invalid:

```zax
if bare{
  i : Integer = 0
  k : Integer = 1
} ;; condition {
}
// error: two uncomposed initializer statements
```

```zax
if bare{
  i : Integer = 0;
  k : Integer = 1;
} ;; condition {
}
// error: trailing ; has no right operand
```

## Boundaries cannot be crossed

Joiners, separators, and partial operations cannot be isolated while obtaining
their required pieces outside:

```zax
a = b bare{ ; } c = d
while i : Integer = 0 bare{ ;; } condition {
}
while i : Integer = 1 bare{ + a } ;; condition {
}
```

Each payload is incomplete for the surrounding category.

`bare{...}` does not change semicolon composition, header-section roles,
continuation, operator fixity, or attachment.

## Keyword-neutral source

Inside:

```zax
a = bare{ if + while } + foo
```

`if` and `while` do not acquire their ordinary flow-control keyword roles. They
must resolve as identifiers, permitted operator-phrase components, or another
valid nonkeyword expression role.

This is not an ordinary `if` statement:

```zax
bare{
  if init : Integer ;; condition {
  }
}
```

The payload fails unless some other future nonkeyword grammar independently
makes it one valid effective statement.

Strings and comments retain their own lexical modes. Words inside a literal are
not reinterpreted merely because the literal occurs inside bare source.

## Confusable asymmetric circumfix

The operator catalog preserves asymmetric `||...|` as an intent-gated
saturating-magnitude form:

```zax
// Illustrative future syntax.
saturated := bare{ ||value| }
```

Natural source remains a confusable-form intent error:

```zax
saturated := ||value|
```

The likely alternatives are:

```zax
norm := ||value||
saturated := bare{ ||value| }
```

`bare{...}` cannot invent an unrecognized form. `|value||` has no assigned
operator meaning and remains invalid.

## Intent categories

Future work must preserve the distinctions among:

- layout-intent errors, repaired through physical presentation;
- operator-attachment intent errors, repaired through whitespace or grouping;
- redundant-structure intent errors, repaired by removing a competing marker;
- confusable-form intent errors, which require explicit interpretation; and
- keyword-role conflicts, which require a keyword-neutral interpretation.

The bare enclosure addresses only confusable forms and keyword roles. It does not
suppress the other diagnostics.

## What bare source does not permit

The candidate does not:

- recognize arbitrary symbolic operators;
- make an unavailable operation available;
- select one ambiguous overload;
- bypass protected intrinsic signatures or mixfix-consumption barriers;
- change result shape;
- grant writable, mutable, unsafe, pointer, or lifetime authority;
- disable indentation, continuation, or attachment validation;
- create block-style statement separation;
- or act as an unsafe region.

A future categorized unsafe enclosure may share a contiguous visual pattern but
has a separate purpose and contract.

## Phrase-fence alternative

A more local doubled-single-quote phrase fence remains a candidate:

```zax
value''fast''
''run''value
x ''runs fast'' y
```

It makes phrase extent and pre/post attachment visible without grouping the wider
expression, but may conflict with empty/adjacent literals, literal prefixes,
escaping, keyboards, and documentation formatting. Those concerns are preserved
in [raw literal input](literal-operators.md).

Future phrase work must decide whether:

- bare source is sufficient;
- a local phrase fence is also needed;
- another qualified phrase syntax is clearer; or
- the candidate should be replaced entirely.

## Open questions

- Is `bare{...}` the best spelling and terminology?
- Does the contiguous opener interact safely with comments and token spacing?
- How is multiline indentation categorized without implying scope?
- Which contextual keywords are neutralized?
- How do visible custom phrases participate without making imports reparse
  ordinary source?
- Does keyword-neutral parsing prefer identifiers, phrases, or diagnose
  incomparable interpretations?
- How are nested bare enclosures treated?
- Can documentation and formatting preserve the enclosure transparently?
- How does source reflection represent tokens whose enclosure is absent from the
  final expression tree?
- Which confusable forms, beyond asymmetric saturating magnitude, justify the
  mechanism?

## Activation and retirement

Activate this input for operator phrases, keyword-neutral source,
confusable-form intent, source reflection, transparent lexical enclosures, or
the next focused phrase/source work item.

That work must test the candidate against source structure, declarations, flow
headers, operator precedence, mixfix, literal lexing, comments, documentation,
formatting, and future unsafe syntax. Move accepted behavior into its lasting
owners and retire this file after every requirement and example is dispositioned.
