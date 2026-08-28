# Raw input: literals and literal operators

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work defining literal realization, prefixes, payloads, or literal operators |
| Applies To | Preserved literal syntax and operator pressure from operator review |
| Owns | Literal realization questions, representative source, activation pressure, and retirement criteria |
| Does Not Own | Accepted literal grammar, types, parsing, compile-time execution, or operator behavior |
| Source / Provenance | Legacy `basics.md` and `operator.md` literal evidence |

## Preserved requirements

Pre-unary `+` requests unsigned integer-literal realization:

```zax
a := +55
b := +(55)
```

Grouping must not change the selected type, width, value, or failure. Future
literal work must decide:

- default width and signedness;
- widening when a value exceeds the default integer;
- arbitrary-width compile-time values;
- explicit base, width, and encoding prefixes;
- when realization fails at compile time; and
- how expected type context participates.

Legacy literal-operator evidence includes:

```zax
value : Byte = roman'IX'
```

That form raises separate questions about prefix recognition, quote delimiters,
payload escaping, expected result selection, compile-time execution, and whether
custom literal operators belong to a closed or bounded phrase/prefix catalog.

Quote characters remain literal delimiters rather than generic pre/post unary or
circumfix operators.

## Phrase-fence collision pressure

Future operator-phrase work should evaluate doubled single quotes as a compact
phrase fence:

```zax
value''fast''
''run''value
x ''runs fast'' y
x + a'normal c-style\t-string' + y
```

The candidate could make phrase extent and pre/post attachment visible while
preventing keyword interpretation. Literal work must test:

- empty single-quoted literals;
- adjacent literal tokens;
- quote escaping and payload modes;
- literal prefixes such as `a'...'`;
- tokenization beside identifiers;
- Markdown and documentation presentation; and
- whether another phrase fence avoids collision more cleanly.

No doubled-quote phrase syntax is accepted by this input.
Future phrase work should compare it with the complete
[bare-source enclosure candidate](bare-source.md).

## Activation and retirement

Activate this input for literal parsing, literal prefixes, custom literals,
compile-time literal execution, expected-result literal work, or a phrase fence
that uses quote characters. Move accepted behavior into its language owner and
retire this file when every preserved form is promoted, rejected, or moved.
