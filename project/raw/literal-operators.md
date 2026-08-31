# Raw input: literals and literal operators

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work defining literal realization, prefixes, payloads, or literal operators |
| Applies To | Preserved literal syntax and operator pressure from operator review |
| Owns | Literal realization questions, representative source, activation pressure, and retirement criteria |
| Does Not Own | Accepted literal semantics or current source/integer behavior |
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

A typed slot may directly realize a compiler-known integer result when that
result has not already committed to a concrete integer type:

```zax
myByte : Byte = compileTimeResult()
```

Out-of-range realization is a compile-time error. An ordinary result already
typed as `Integer` does not become directly realizable as `Byte` merely because
its value is constant. Literal and compile-time work must preserve that boundary
and make the minimum signed value expressible without first realizing an
unrepresentable positive value in the same type.

The current integer-side boundary is owned by
[Zax integers](../../language/integers.md#initialization-and-compile-time-realization).
This input retains literal grammar, prefixes, payload, and unrealized-result
mechanics.

Legacy literal-operator evidence includes:

```zax
value : Byte = roman'IX'
```

That form raises separate questions about prefix recognition, quote delimiters,
payload escaping, expected result selection, compile-time execution, and whether
custom literal operators belong to a closed or bounded prefix catalog. It remains
plausible precisely because its quote is *attached* to a prefix.

Quote characters remain literal delimiters rather than generic pre/post unary or
circumfix operators.

## Phrase-fence coordination

Operator-phrase review settled the single-quote question, and literal work
inherits the result rather than reopening it.

Accepted today by
[Zax operator phrases](../../language/operator-phrases.md#exact-phrase-fencing)
and applied to general tokenization by
[Zax source structure](../../language/source-structure.md#phrase-fences-and-literal-coordination):

```zax
"ordinary text" // ordinary unprefixed literal
h'DEADBEEF'     // attached prefixed literal
name'payload'   // attached custom/prefixed literal
foo 'bar'       // separated phrase fence
```

A whitespace-separated single-quoted payload is a phrase fence rather than a
literal:

```zax
'ordinary text' // phrase fence or phrase diagnostic, not a literal
''              // error: empty phrase fence, not an empty literal
```

Future literal work inherits a stable source boundary: an attached single quote
introduces a literal payload, while a whitespace-separated single quote
introduces a phrase fence. It may not restore standalone single-quoted literals,
implicit juxtaposition, or another form that silently reinterprets that
distinction.

### What literal work must still decide

- default width, signedness, and encoding for unprefixed double-quoted literals
  now that the single-quote alternative is gone;
- whether every custom literal must carry a prefix, since an unprefixed single
  quote is unavailable;
- how a literal prefix is recognized and whether the catalog is closed, bounded,
  or open;
- payload escaping and quote embedding without the old
  single-quote/double-quote symmetry;
- adjacent literal tokens and merged literal sequences;
- empty literals; and
- tokenization beside identifiers, and Markdown/documentation presentation.

### Future juxtaposition constraint

Future literal, concatenation, juxtaposition, or implicit-call syntax must not
reinterpret a whitespace-separated single-quoted phrase fence as an expression
operand:

```zax
foo 'bar'
// must remain an exact phrase component, never `foo` applied to a literal
```

That constraint holds regardless of which juxtaposition mechanism is eventually
adopted. Compare the deferred
[bare-source enclosure candidate](bare-source.md), which addresses keyword roles
rather than phrase extent.

## Activation and retirement

Activate this input for literal parsing, literal prefixes, custom literals,
compile-time literal execution, expected-result literal work, or future
juxtaposition and concatenation syntax. Move accepted behavior into its language
owner and retire this file when every preserved form is promoted, rejected, or
moved.
