# Raw input: literals and literal operators

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work defining prefixed/custom literals, payloads, literal declarations, or literal-only type sugar |
| Applies To | Preserved prefixed literal syntax and behavior pressure after unprefixed integer realization was promoted |
| Owns | Prefix/result questions, payload and merged-literal evidence, representative source, activation pressure, and retirement criteria |
| Does Not Own | Current unprefixed integer realization ([integer literals and realization](../../language/integer-literals.md)) or accepted source attachment behavior ([source structure](../../language/source-structure.md)) |
| Source / Provenance | Legacy `basics.md`, `casting.md`, and `operator.md` literal evidence together with integer-literal review |

## Current boundary inherited by future literal work

This file no longer decides how ordinary source such as `55` gets an integer
type. That behavior is current in
[Zax integer literals and realization](../../language/integer-literals.md).

This file retains prefixed forms such as:

```zax
// Illustrative future prefix.
myBits := b'100001'
```

When future work defines `b`, its declaration must produce one concrete result
type. The surrounding expression cannot later reinterpret the payload as a
different type.

Future prefixed/custom literal work inherits these constraints:

- a resolved prefixed literal has one concrete result type;
- its result does not remain an uncommitted integer for later
  destination-driven reinterpretation;
- a function used to implement a literal also has its declared concrete result
  type;
- an attached single quote introduces the payload;
- backticks are not literal delimiters; and
- a prefix cannot cause expression-result lookahead or speculative receiver
  discovery.

The prefix may identify a type-owned literal declaration, but exact declaration
syntax, namespaces, imports, ownership, shadowing, and ambiguity remain future
work.

## Preserved prefix and payload evidence

Legacy literal-operator evidence includes:

```zax
value : Byte = roman'IX'
```

That form raises separate questions about prefix recognition, result type,
payload escaping, compile-time execution, and whether custom literal operators
belong to a closed or bounded prefix catalog. It remains plausible precisely
because its quote is *attached* to a prefix.

Quote characters remain literal delimiters rather than generic pre/post unary or
circumfix operators.

Legacy numeric candidates include:

```zax
binary := b'1011101'
octal := o'12345670'
duodecimal := d'1234567890AB'
hexadecimal := h'ABC123'
typedBinary := b8'10011011'
```

These examples preserve pressure, not accepted prefix names or result types.
Future work must decide whether a form denotes a mathematical radix, a concrete
numeric type, a bit pattern, or another result family. In particular,
interpreting a width-specific signed payload as a positive magnitude differs
from interpreting it as a complete signed bit pattern.

Legacy merged-literal evidence also combines numeric payloads with character
sequences:

```zax
// Illustrative legacy candidates; merging is not accepted behavior.
binarySequence := "m" b'1011101'
wideSequence := w'value "' h'16F' w'"'
```

Future work must not infer this merging merely because both component prefixes
resolve. It must define evaluation order, result ownership, character-range
validation, encoding, and failure explicitly.

### Legacy encoding and transformation candidates

The legacy prefix catalog also proposed:

- `a` and `ascii` for ASCII strings with different escape policies;
- `c` for character values with C-style escapes;
- `w` and `unicode` for wide-character strings;
- `utf8` for UTF-8 payloads without escape processing;
- `b64` for base-64 decoding into bytes/string storage; and
- `xml` for XML-entity decoding.

Those names and meanings are not accepted. They preserve the broader pressure
that a prefix may determine result type, encoding, escape policy, validation,
and compile-time transformation together.

Representative legacy candidates:

```zax
// Illustrative future syntax and behavior.
myCharacter := c'\n'
myAscii := ascii'payload without escapes'
myUtf8 := utf8'payload retained as written'
myBase64 := b64'VGhlIHF1aWNrIGJyb3duIGZveC4='
myXml := xml'John&#39;s Fish &amp; Chips'
```

Legacy material allowed either attached single-quoted or prefixed
double-quoted payloads so one quote kind could contain the other:

```zax
// Illustrative future syntax.
mySingleQuote := c"'"
myDoubleQuote := c'"'
```

Future work must decide whether both payload delimiters survive, how escapes
interact with each, and whether result-type ownership is clear without expected
result lookahead.

Legacy continuation repeated the prefix on every payload and proposed merging
adjacent results:

```zax
// Illustrative future syntax and merging.
myText := utf8'first part ' \
          utf8'second part'
```

No payload inherits a prefix from an earlier token. Continuation, adjacency,
merging, and conversion of numeric payloads into character codes remain
independent decisions.

## Literal-only typed suffix pressure

A narrow source candidate would commit one bare numeric literal without
introducing general expression ascription:

```zax
// Illustrative future syntax.
myA := 55 : U8
myB := (55 : U8)

myC := (55 + 88 : U8) // intended error: not a bare literal
myD := (+55 : U8)     // intended error: not a bare literal
```

The form would be sugar for `(: U8 = 55)` only when the operand is one bare
numeric literal. Future work must decide whether this can be expressed without
conflicting with declaration colon, postfix parsing, whitespace, grouping, or
unary precedence. This input does not accept the syntax.

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

- concrete result types for language-provided prefixes such as `b`, `b8`, `o`,
  `d`, and `h`;
- whether every custom literal must carry a prefix, since an unprefixed single
  quote is unavailable;
- how a literal prefix is recognized and whether the catalog is closed, bounded,
  or open;
- literal declaration syntax, owner/type relationship, namespace, imports,
  shadowing, and ambiguity;
- mathematical-radix versus typed-value versus bit-pattern interpretation;
- digit alphabets, case, separators, grouping, and malformed payloads;
- compile-time execution requirements and diagnostics;
- payload escaping and quote embedding without the old
  single-quote/double-quote symmetry;
- adjacent literal tokens and merged literal sequences;
- empty literals; and
- tokenization beside identifiers and Markdown/documentation presentation; and
- whether the literal-only typed suffix is useful and grammatically viable.

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

Activate this input for prefixed/custom literal parsing, literal declarations,
payloads, compile-time literal execution, typed/bit-pattern literal behavior,
the literal-only type suffix, or future juxtaposition and concatenation syntax.
Move accepted behavior into literal, source, numeric, compile-time, or
applicable result-type owners and retire this file when every preserved form is
promoted, rejected, or moved.
