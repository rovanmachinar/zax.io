# Zax source structure

| Field | Value |
| --- | --- |
| Status | Current conceptual design |
| Audience | Human developers reading, writing, or evaluating Zax |
| Applies To | Programmer-facing source structure; not a formal grammar or specification |
| Implementation State | Not established by this repository |
| Owns | Statement-level newlines, explicit and construct-open continuation, effective statements and bodies, semicolon composition, comment and physical-line trivia retained through composition and layout validation, exact two-space structural indentation and physical-tab rejection, symbolic-operator whitespace and adjacency, longest recognized symbolic tokens, grouped separate unary applications, application of general tokenization/comment/continuation mechanics to operator phrases and their literal boundary, declaration-colon and mixfix-component-list continuation, the boundary between structural operands and expression continuation, `;;` and `??` separator whitespace, flow-header continuation and the explicit `\` alignment escape hatch, brace layout, `else` attachment and layout, body boundaries and the empty-header-block intent error, contextual keyword recognition, mandatory layout validation, diagnostic categories, and comment forms and attachment |
| Does Not Own | Declaration behavior ([declarations and bindings](declarations-and-bindings.md)); flow semantics ([core flow control](core-flow-control.md)); or operator/phrase interpretation ([operators](operators.md), [operator phrases](operator-phrases.md)) |

## Mental model

Zax uses significant statement-level newlines without making indentation define
the program's structure.

At statement level, an uncontinued newline ends a simple statement. Ordinary
statements therefore need no terminator:

```zax
doA()
doB()
```

Newlines may remain inside an open expression delimiter such as `(` or `[`.
The exact multiline behavior of each expression form belongs with that form,
but ordinary delimited layout does not require an explicit continuation on
every physical line.

Outside an open construct, `\` explicitly continues one statement across a
physical newline. Zax does not infer continuation merely because an operator is
missing an operand.

Grammar and explicit tokens determine structure. Indentation must truthfully
present that structure, but indentation does not create blocks or decide which
body owns a statement.

## Explicit continuation

`\` suppresses an otherwise significant physical newline:

```zax
value = calculate() + \
    more()
```

The continuation marker must be the last non-comment code token on its line. An
ordinary `//` comment may follow it:

```zax
value = calculate() + \ // Explain the continued expression.
    more()
```

A `///` documentation comment may not follow `\` because the continued
construct is not yet complete.

### Continuation indentation

Every continued physical line must be indented beyond the structural
indentation of the statement being continued:

```zax
value = calculate() + \
    more()
```

The exact expression alignment may vary:

```zax
value = calculate() + \
                   more()
```

This is an error because the line visually presents itself as a new statement
at the surrounding level:

```zax
value = calculate() + \
more() // error: continuation is not hanging-indented
```

At a nested level, the continuation must be deeper than the continued statement,
not merely deeper than the enclosing control header:

```zax
if condition
  value = calculate() + \
    more()
```

### Continued and continuation-only lines

A continued line may contribute source tokens and continue again:

```zax
value = 1 + 2 \
    + 3 + \
    4
```

It may also consist only of an appropriately indented `\`. Such a line visibly
carries the continuation across another physical newline:

```zax
value = 1 + 2 + \
    \
    3
```

A continuation-only line may have an ordinary `//` comment after `\`.

A blank or comment-only line does not carry continuation:

```zax
value = 1 + 2 + \

    3 // error: blank line broke the continuation
```

```zax
value = 1 + 2 + \
    // error: comment-only line does not carry continuation
    3
```

The joined physical lines must eventually form one valid statement. A
continuation at end of file is an error. The source before `\` need not be
incomplete in isolation when the joined statement is valid:

```zax
value = 1 + 2 \
    + 3
```

## Symbolic operators, whitespace, and adjacency

The [operator catalog](operator-catalog.md) fixes every recognized symbolic
form. Whitespace presents its fixity:

```zax
!value       // pre-unary: attached to the operand on its right
value++      // post-unary: attached to the operand on its left
left + right // binary: whitespace on both sides
```

A newline counts as binary-operator whitespace only when another accepted rule
already continues the expression. Missing an operand does not itself continue:

```zax
value := first +
  second // error: `+` does not continue the statement
```

Use explicit continuation:

```zax
value := first + \
  second
```

### Longest token and separate unary applications

At an operator position, the longest recognized symbolic token wins:

```zax
a +! b // checked/reporting addition form
a + !b // binary + whose right operand begins with pre-unary !
```

Separate adjacent unary applications require grouping:

```zax
--value    // one pre-decrement token
-(-value)  // two pre-unary negations
!(!value)  // two pre-unary logical negations
```

`!!value` cannot mean two independent `!` applications. This leaves that
contiguous spelling available for future allocation without reinterpreting
existing valid source.

The rule applies to ordinary symbolic unary operations. Call, index, projection,
and other grammar-recognized postfix forms retain their own chaining rules.

### Circumfix attachment

A circumfix opener attaches to the enclosed expression on its right, and its
closer attaches on the left:

```zax
|value|
||vector||
```

An ungrouped recognized opener/closer pair is one operation, not independent
pre/post unary applications. Nested magnitude uses grouping:

```zax
|(|value|)|
```

Exact circumfix forms and availability are defined by the
[operator catalog](operator-catalog.md#circumfix-forms).

## Operator phrase source integration

The complete programmer model for phrase words, interpretation, grouping,
fencing, and phrase-specific presentation is owned by
[Zax operator phrases](operator-phrases.md). This section records how the general
source machinery owned here applies to that feature.

### Phrase words and selected presentation

A phrase word is an ASCII `[a-z][a-z0-9]*` token. After phrase selection, each
multiword component must contain exactly one ASCII space between words and remain
on one physical line:

```zax
chickens cluck loudly    // valid when this phrase is uniquely selected
chickens cluck  loudly   // error: phrase-whitespace intent
chickens cluck \
    loudly               // error: selected phrase spans physical lines
```

Comments, doubled spaces, and explicit continuation remain source trivia while
candidate trees are formed. They do not remove a competing phrase reading.
Only after one interpretation survives does this document's physical-line and
trivia validation report a phrase-presentation error. An uncontinued newline is
different because it already ends the logical statement.

The complete ambiguity and presentation examples are in
[operator phrases](operator-phrases.md#presentation-confirms-a-selection-it-never-makes-one).

### Phrase fences and literal coordination

An attached single quote starts a literal payload; a whitespace-separated single
quote starts an exact phrase fence:

```zax
"ordinary text" // ordinary unprefixed literal
h'DEADBEEF'     // attached prefixed literal
foo 'bar'       // whitespace-separated phrase fence
```

Every fenced word follows the phrase-word token rule. The fence permits no
escapes or leading/trailing spaces. A whitespace-separated single-quoted payload
is a phrase fence rather than an unprefixed literal, and `''` is an invalid empty
fence.

The phrase feature owns what fencing does to candidate interpretations; see
[exact phrase fencing](operator-phrases.md#exact-phrase-fencing). Complete
literal realization remains future literal work and may not reinterpret this
attachment boundary.

### Contextual keywords and enclosure boundaries

The contextual-keyword machinery owned here considers keyword roles only where
their constructs are grammatically permitted. Phrase words may use the same
spellings; complete phrase interpretation and ambiguity behavior are defined by
[operator phrases](operator-phrases.md#keyword-words-in-phrase-roles).

A phrase component cannot cross a hard statement boundary. It also cannot span a
source enclosure whose payload must independently form a complete expression or
effective statement. General statement, continuation, delimiter, and enclosure
recognition happen before the phrase owner applies candidate pruning.

### Formatter and source-reflection impact

A formatter must retain comments, spaces, physical lines, and fences until phrase
selection completes. It may then normalize one selected phrase to its exact
single-space presentation, but it may not add or remove a fence or turn ambiguity
into another meaning.

Source reflection and documentation must retain enough physical trivia to
reproduce an explicit phrase fence even though the fence creates no final
expression-tree node. Complete source-reflection representation remains future
work.

## Comma-list continuation

A comma already recognized as a separator in a comma-list implicitly continues
that list across the following physical newline:

```zax
return first: number:,
  second: text: = produce()
```

This is a narrow list rule, not general continuation for incomplete
expressions:

```zax
value := first +
  second // error: `+` does not continue the statement
```

The continuation line must be hanging-indented beyond the structural indentation
of the statement:

```zax
return first: number:,
second: text: = produce() // error: continuation is not hanging-indented
```

A blank or comment-only physical line breaks comma-list continuation. A
continuation-only `\` line may carry the list across one additional newline:

```zax
return \
  first: number:,
  \
  second: text: = produce()
```

### One continuation reason per newline

Each continued physical newline has one sufficient reason:

1. an open delimiter permits the newline;
2. a recognized comma-list separator continues it; or
3. a recognized construct such as a declaration colon or mixfix component list
   opens continuation; or
4. `\` explicitly suppresses an otherwise significant newline.

An explicit `\` is an error when another rule already continues that same
newline:

```zax
return \
  first: number:, \ // error: the comma already continues the list
  second: text: = produce()
```

```zax
consume(
  first, \ // error: the open `(` already permits the newline
  second
)
```

The explicit marker is not harmless defensive punctuation. Redundancy makes
source intent contradictory.

When a list begins on the line after an otherwise complete construct such as a
bare `return`, the first newline still needs explicit continuation:

```zax
return \
  first: number:,
  second: text: = produce()
```

Once the list exists, each comma carries its following newline.

### Grouping and list contexts

Grouping parentheses require their contents to form one expression. They do not
preserve or introduce multiple-result mapping:

```zax
return (produce()) // one expression
```

Call argument delimiters, construction packets, and naked return-result lists
are distinct syntactic contexts that may accept their applicable mapping forms.
Complete expression-versus-result-mapping behavior is defined by
[Zax function invocation](function-invocation.md#expression-mode-and-result-mapping-mode).

## Statements, blocks, and bodies

Zax distinguishes these source forms:

- A **simple statement** performs one ordinary statement-level operation.
- A **composed statement** combines complete statements with `;`.
- A **block statement** contains statements within `{` and `}`.
- A **flow-control body** consumes one effective statement, which may be simple,
  composed, or a block.

```zax
if condition
  doA()
```

```zax
if condition
  doA();
  doB()
```

```zax
if condition {
  doA()
  doB()
}
```

A flow-control clause establishes a scope around its complete body regardless
of which source form supplies that body. Names introduced inside the body do not
escape after the clause. Operands of a composed body share the clause scope, so
a name introduced by an earlier operand may be used by a later operand.

The exact relationship between a clause scope and a braced block's scope,
including destruction and scope-exit behavior, is owned by
[core flow control](core-flow-control.md) and
[construction, replacement, and destruction](construction-and-destruction.md).
Braces do not make body-local names escape.

## Semicolon composition

`;` is a statement-composition operator, not a statement terminator. It requires
a complete statement on both sides and produces one effective statement.

`;` attaches to the complete statement on its left and requires whitespace on its
right. Whitespace before `;`, or no whitespace after it, is a deliberate intent
error:

```zax
foo(); bar()

foo();
bar()

// foo() ; bar() // error: whitespace before ;
// foo();bar()   // error: no whitespace after ;
```

The doubled `;;` header-section separator and the `??` conditional-expression
separator are distinct tokens with their own surrounding-whitespace rule; see
[Header sections and separators](#header-sections-and-separators).

### Horizontal composition

Several statements may be composed on one physical line:

```zax
doA(); doB(); doC()
```

The final statement has no trailing semicolon because there is no further
operand.

This differs intentionally from C in a one-statement body:

```zax
if condition
  doA(); doB()
```

Both calls belong to the Zax `if`. In the visually similar C form, only the
first call would be conditional.

### Vertical composition

Vertical composition extends a source position that otherwise accepts one
statement:

```zax
if condition
  doA();
  doB();
  doC()
```

Each right operand must have structural indentation consistent with the same
body. Leading and trailing semicolons are errors because an operand is missing:

```zax
; doA()       // error: missing left operand
doA();        // error: missing right operand
```

A block already accepts vertically separated sibling statements, so vertical
composition there is redundant and is an error:

```zax
if condition {
  doA();
  doB() // error: vertical composition is redundant
}
```

Horizontal composition remains legal where a newline is not separating the
statements:

```zax
if condition {
  doA(); doB()
}
```

Comment trivia may appear between vertical operands, including while `;` is still
waiting for its right operand. No `\` is required:

```zax
if condition
  doA(); // `;` establishes that another operand follows.
  // Comment trivia may occur before that operand.
  doB()
```

A physically blank line may not separate them because it visually contradicts
their composition.

The source processor must therefore preserve comment and physical-line trivia
long enough to validate composition, blank-line boundaries, documentation
attachment, and layout intent. It need not expose comments as ordinary semantic
tokens, but it may not erase the relevant positions before those checks run.

### Structural operands versus expression continuation

Composed operands are new statements and stay at exactly one structural level:

```zax
if true
  a();
  b();
  c()
```

Progressively indenting sibling operands is an error:

```zax
if true
  a();
    b(); // error: composed operands must share one structural level
      c()
```

A body that begins four spaces deeper is also an error:

```zax
if true
    a(); // error: the body must begin exactly two spaces deeper
    b();
    c()
```

Deeper alignment remains legal for tokens continuing the same expression once
continuation is established, even when those expressions are then composed:

```zax
if true
  a := 1 + 2 + 3 + 4 \
    + 5 + 6;
  b := 1 + 2 + \
       3 + 4;
  calc(a, b)
```

The lasting rule is:

> Parsed continuation may use deeper alignment for tokens that continue the same
> expression. It does not relax exact structural indentation for a new body
> statement, composed operand, header section, or sibling declaration.

### Nested control statements

A semicolon after an unbraced inner body composes at the inner level:

```zax
if outer
  if inner
    doA();
    doB()
```

To compose the complete inner control statement with another statement in the
outer body, braces expose the inner statement's end:

```zax
if outer
  if inner {
    doA()
  };
  doB()
```

Without those braces, syntax and indentation disagree:

```zax
if outer
  if inner
    doA();
  doB() // error: semicolon composition and indentation disagree
```

```zax
if outer
  if inner
    doA()
  doB() // error: outer body membership is not expressed by syntax
```

## Mandatory layout validation

Indentation does not determine Zax grammar, but it is significant to source
validity.

After parsing explicit source structure, Zax validates that statement-start
indentation presents that structure truthfully. Contradictory indentation is a
mandatory error; it is not an optional lint and cannot be disabled.

Structural indentation uses ASCII spaces only, and one structural level is
exactly two spaces:

```zax
if condition
  doConditionalWork()
```

- statement-start indentation must use the exact two-space structural level;
- physical tab characters are deliberate intent errors; and
- continuation or expression alignment may be deeper only where the parsed source
  already establishes continuation.

```zax
if condition
  doA()
  doB() // error: indentation claims unexpressed body membership
```

Removing a required semicolon therefore produces an error rather than silently
moving `doB()` outside the body. Adding a semicolon while dedenting its intended
right operand is also an error:

```zax
if condition
  doA();
doB() // error: composition and indentation disagree
```

Structural indentation and expression alignment are different concerns.
Statement-start lines must occur at the indentation expected for their parsed
position. Lines already known to continue a statement may align for readability
within the hanging-indentation requirement.

A formatter may normalize valid source. It must not silently rescue source whose
layout contradicts its structure.

## Header sections and separators

`;;` separates construct-specific flow-header sections, and `??` separates the
arms of a conditional expression. Both require whitespace on both sides, and
neither is `;` statement composition:

```zax
if i := 0 ;; i < 100 ;; ++i {
  body()
}

e := a > b ?? c ;; d
```

Their flow-control and conditional-expression meaning is owned by
[core flow control](core-flow-control.md); this document owns only their token
spacing and their relationship to statement composition.

### Header continuation

Top-level operands and sections continued across a flow header use one common
two-space continuation level rather than aligning under a variable-length keyword
or label prefix:

```zax
if i := 0;
  j := 1;
  k := 2 ;;
  i + j + k == 7 ;;
  ++i; --j; ++k {
  body()
}
```

Progressively indented sibling sections contradict the header structure:

```zax
if i := 0 ;;
    i < 100 ;;
      ++i {
  body()
}
```

### Explicit continuation as the alignment escape hatch

An alignment-sensitive programmer may move the whole header to the common
continuation level with `\`:

```zax
if \
  firstInitializer();
  secondInitializer();
  thirdInitializer() ;;
  condition ;;
  postOperation() {
  body()
}
```

The same form remains stable under a long label:

```zax
if shadowable extraordinarily_long_label: \
  firstInitializer();
  secondInitializer() ;;
  condition {
  body()
}
```

A label does not itself continue the physical line, so this explicit `\` is
necessary and legal. By contrast, a trailing `;` or `;;` already establishes
continuation across the following newline, so `\` after `;` or `;;` is redundant
continuation and therefore a deliberate intent error.

## Declaration-header continuation

A declaration colon that requires a type or callable prototype opens a narrow
construct-specific continuation:

```zax
value :
  MyType = source

operator pre unary '--' final :
  ()() = {
  }
```

This is not a general rule that incomplete syntax continues. A flow-label,
target, result-routing, or other contextual colon does not inherit declaration
continuation:

```zax
if retry: // error: the flow label does not continue the header
  initialize() ;;
  condition {
}
```

That header requires explicit continuation:

```zax
if retry: \
  initialize() ;;
  condition {
}
```

Initializer and assignment `=` do not implicitly continue:

```zax
value : MyType =
  source // error: `=` does not continue the declaration
```

Explicit continuation may state that intent:

```zax
value : MyType = \
  source
```

### Mixfix component-list continuation

`operator mixfix` opens a grammar-delimited component list:

```zax
operator mixfix
  index 1
  binary '='
final : (
  result : MyResult
)(
  indexValue : MyIndex,
  rhs : MyValue
) writable = {
}
```

The grammar recognizes component specifications and the declaration
qualifier/prototype that ends the list. Indentation validates that structure but
does not create it. Component lines use one consistent hanging indentation.

An explicit `\` is redundant while the open mixfix component list or an open
prototype delimiter already permits the newline.

Exact mixfix declaration semantics are defined by
[mixfix operators](mixfix-operators.md#declaration-model).

## Braces and body boundaries

A `{` used as a scope opener has whitespace on both sides, with a newline
counting as whitespace. A braced body begins on the final physical header line,
and a multiline body-closing `}` aligns with its flow keyword:

```zax
if condition {
  doA()
  doB()
}
```

A body-opening `{` separated onto the next physical line, or a closing `}` at the
wrong structural level, is a deliberate intent error. An unbraced body ends after
its one effective statement, so a following dedented statement runs
independently. An empty braced body is legal:

```zax
if condition { }
```

### Empty-header-block intent

An exact empty block used as an initializer or post header is a deliberate intent
error, because it reads more like a missing or misplaced body than meaningful
header work:

```zax
while i := 0 ;; i < 100 ;; { } {
  body()
}
```

The rule is syntactic rather than an attempt to prove whether arbitrary nested
statements have effects. A nonempty but apparently effectless header operation
may instead become lint material.

### Blocks inside composed initializers

A block in an initializer keeps its ordinary nested lexical scope, so names it
introduces do not escape to the condition or body. Binding visibility across a
flow header is owned by
[declarations and bindings](declarations-and-bindings.md#flow-control-initialization).

```zax
if {
  temporary := inspectEnvironment()
  recordInspection(temporary)
} ;; condition {
  body()
}
```

When a multiline block is one operand of vertically composed header
initialization, explicit continuation places the block and its sibling operands
at the common level:

```zax
if \
  {
    temporary := inspectEnvironment()
    recordInspection(temporary)
  };
  resource := getResource() ;;
  validated(resource) ;;
  discard(resource) {
  use(resource)
}
```

The compact horizontal form is also legal:

```zax
if {
  temporary := inspectEnvironment()
  recordInspection(temporary)
}; resource := getResource() ;;
  validated(resource) ;;
  discard(resource) {
  use(resource)
}
```

This vertical form is a deliberate layout-intent error even though its tokens can
be parsed, because the block closes at the flow-keyword level while `;` claims an
indented right operand as its structural sibling:

```zax
if {
  temporary := inspectEnvironment()
  recordInspection(temporary)
};
  resource := getResource() ;;
  validated(resource) ;;
  discard(resource) {
  use(resource)
}
```

Use explicit continuation to move both operands to the header-continuation level,
or put the right operand horizontally after `};`. Moving `}` onto the last inner
statement line is not a workaround; a multiline block's closing delimiter returns
to the level where that block statement began.

## `else` attachment and layout

An `else` clause must present its attachment to the `if` it completes.

An unbraced clause aligns exactly with its owning `if`, follows the complete true
body with no physically blank line between them, and begins its own unbraced body
exactly two spaces deeper:

```zax
if condition
  trueBody()
else
  falseBody()
```

A braced clause keeps `else` on the physical line that closes the preceding body,
separates `}` and `else` with whitespace, opens the `else` body on the `else`
line, and closes at the owning `if`'s level:

```zax
if condition {
  trueBody()
} else {
  falseBody()
}
```

A newline between `}` and `else` is a deliberate attachment-intent error:

```zax
if condition {
  trueBody()
}
else { // error: else is visually detached from the completed if
  falseBody()
}
```

A blank line before an unbraced `else`, or an `else` at the wrong structural
level, is the same class of error:

```zax
if condition
  trueBody()

else // error: a blank line detaches else from its if
  falseBody()
```

```zax
if condition
  trueBody()
  else // error: else must align with its owning if
    falseBody()
```

`else if` places `else` and `if` on one physical line separated by whitespace, and
every clause of the chain aligns with the owning `if`:

```zax
if firstCondition
  firstBody()
else if secondCondition
  secondBody()
else
  finalBody()
```

Comment trivia may precede or follow a clause without weakening the attachment
rule, because a comment line is not a blank line:

```zax
if condition
  trueBody()
// Explain why the alternative exists.
else
  falseBody()
```

Which clause runs, what a flow label names, and when a post operation runs are
owned by [core flow control](core-flow-control.md).

## Contextual keyword recognition

A spelling has keyword status only where that keyword's construct is
grammatically permitted. Outside those positions, that spelling is not a keyword:

```zax
next          // transfer keyword
next outer:   // transfer keyword with a target
next()        // next has no keyword status here

continue          // transfer keyword
continue outer:   // transfer keyword with a target
continue()        // continue has no keyword status here
```

The rule is about position rather than about a reserved-word list, so a
concept owner that shows a likely confusion should link here instead of
restating the rule. The term itself is defined by
[language-design terms](terms.md#contextual-keyword).

## Comment lexical modes

Comment delimiters do not cross lexical modes. At ordinary source level, the
recognized form determines how its contents are scanned:

- `///` begins documentation through the end of its physical line.
- `//` begins an ordinary comment through the end of its physical line.
- `/*` followed immediately by one or more `#` characters begins a raw block
  comment.
- An otherwise ordinary `/*` begins a nestable block comment.

The more specific documentation and raw forms are recognized before their
ordinary prefixes.

### Line comments

Once `//` or `///` begins, apparent comment delimiters later on that physical
line are payload:

```zax
/// Documentation text /* remains documentation through this */ text.
// Ordinary comment text /* does not open a block comment.
```

An apparent block opener cannot affect later lines:

```zax
/// The following /* is documentation payload.
doSomething()
*/ // error: no block comment was opened
```

### Ordinary block comments

Ordinary block comments nest:

```zax
/*
  copiedCode()
  /* An existing nested comment. */
  moreCopiedCode()
*/
```

Inside an ordinary block comment, `//` and `///` are inert text. Another
ordinary `/*` increases nesting depth, and `*/` decreases it, even when those
sequences appear in copied text that would otherwise be a string or another
source form.

### Raw block comments

A raw block comment closes only with a contiguous run of exactly the same number
of `#` characters followed immediately by `*/`:

```zax
/*#
Arbitrary content containing /*, */, "*/", or malformed source.
#*/
```

The fence can be lengthened when its closing sequence occurs in the content:

```zax
/*###
Content containing #*/ and ##*/ remains opaque.
###*/
```

Raw contents recognize no nested comments, strings, escapes, documentation, or
other source forms. A raw comment does not itself nest.

A raw comment may appear inside an ordinary block comment at any nesting depth.
The raw opener is recognized before another ordinary nested opener. Everything
through the exact raw closer is ignored, after which scanning resumes at the
same ordinary nesting depth:

```zax
/*
  /*
    /*##
    Everything here is raw, including /*, */, ///, and /*# #*/.
    ##*/
  */
*/
```

## Documentation comments

`///` identifies documentation metadata. Zax recognizes, groups, preserves, and
attaches it deterministically without prescribing XML, Markdown, or another
payload language.

### Grouping and payload

Adjacent full-line `///` comments at the same structural indentation form one
documentation block. Line boundaries are preserved. An empty documentation line
is represented by `///` and remains part of the block:

```zax
/// First paragraph.
///
/// Second paragraph.
```

A physically blank line ends the block.

Everything after `///` is opaque payload:

```zax
/// A URL such as https://zax.io and /* delimiters */ remain documentation.
```

An apparent `//`, `/*`, or `*/` is not stripped or interpreted by Zax.
Documentation markers inside ordinary or raw block comments have no
documentation meaning and require no attachment.

### Leading attachment

Every documentation block has a deterministic target. A leading block attaches
forward to the following construct at the same structural indentation:

```zax
/// Documentation for doSomething.
doSomething()
```

When the target introduces one or more bindings, documentation attaches to the
complete declaration construct rather than to one individual name. Declaration
meaning and multi-binding behavior are defined by
[Zax declarations and bindings](declarations-and-bindings.md).

Documentation is prefix metadata rather than a statement. It does not consume a
body position:

```zax
if condition
  /// Documentation for the body statement.
  doSomething()
```

Misaligned documentation is an error:

```zax
if condition
/// error: documentation does not match the body indentation
  doSomething()
```

Ordinary comment trivia may intervene without becoming documentation:

```zax
/// Documentation for doSomething.
// An ordinary source note.
doSomething()
```

A physically blank line breaks forward attachment.

### File attachment

A top-level block separated from the first code construct by a blank line
attaches to the source file as preamble documentation. Top-level documentation
after the last code construct attaches to the file as epilogue documentation.
Several separately grouped file-level blocks may attach in source order.

An otherwise unattributed block in the middle of source is an error; it does not
silently become file documentation.

### Trailing attachment

An inline trailing block attaches backward to the complete effective statement
or construct at the structural indentation represented by that line:

```zax
doSomething() /// Documentation for doSomething.
```

Attachment does not bubble outward merely because completing an inner construct
also completes an enclosing construct:

```zax
if condition
  doSomething() /// Documentation for doSomething.
```

Leading documentation may independently attach to the enclosing `if`:

```zax
/// Documentation for the if.
if condition
  doSomething() /// Documentation for doSomething.
```

A trailing block after a braced body occurs at the outer structural level:

```zax
if condition {
  doSomething()
} /// Documentation for the if.
```

For a vertically composed body, trailing documentation on the final operand
attaches to the complete effective composed statement:

```zax
if condition
  doA();
  doB() /// Documentation for the composed body.
```

The target must be complete:

```zax
if condition /// error: the if is incomplete
  doSomething()

doA(); /// error: the composition operator requires a right operand
doB()
```

Documentation may appear where a multiline construct finishes:

```zax
value = 1 + 2 + \ // Ordinary commentary is legal here.
    3 /// Documentation for the complete assignment.
```

### Duplicate attachment

A non-file construct may receive one directly attached documentation block,
using either leading or trailing placement but not both:

```zax
/// error: doSomething also has trailing documentation
doSomething() /// Competing documentation for doSomething.
```

The rule applies independently at each structural level:

```zax
if condition
  /// Leading documentation for doSomething.
  doSomething() /// error: duplicate documentation for doSomething
```

Several adjacent `///` lines are one block. Distinct leading blocks separated by
ordinary comment trivia remain distinct; if both target the same construct,
they produce the same duplicate-attachment error. Use an empty `///` line for
separation within one documentation block.

## Programmer-visible diagnostics

Diagnostics should identify the local contradiction and distinguish, where
applicable:

- a missing `\`;
- a broken or unterminated continuation;
- redundant explicit continuation where an open delimiter or comma already
  continues the newline;
- a broken comma-list continuation;
- a missing semicolon operand;
- redundant vertical composition;
- missing braces around a nested control statement;
- unexpected or contradictory indentation;
- an unmatched comment delimiter;
- unattributed documentation; and
- duplicate documentation attachment.

Layout and separator diagnostics additionally distinguish:

- statement-start indentation that does not use the exact two-space structural
  level;
- physical tab characters;
- whitespace before `;` or missing whitespace after it;
- missing whitespace around `;;` or `??`;
- symbolic pre/post/binary whitespace that contradicts the recognized fixity;
- adjacent independent unary applications without grouping;
- an unknown contiguous symbolic token where no catalog form exists;
- a phrase-whitespace presentation error in a selected component;
- a selected phrase component spanning a physical line;
- an empty phrase fence, or a fence with an escape, leading space, or trailing
  space;
- an explicit `\` where a declaration colon or mixfix component list already
  continues the newline;
- an initializer `=` followed by an uncontinued newline;
- mixfix component lines at conflicting continuation levels;
- redundant `\` after `;` or `;;`;
- composed operands at progressively deeper structural levels;
- a body beginning more than one structural level deeper than its header;
- sibling header operands or sections at conflicting continuation levels;
- a body-opening `{` separated onto the next physical line;
- a scope-opening `{` without whitespace on both sides;
- a multiline closing `}` at the wrong structural level;
- an `else` separated from a preceding `}` by a newline, preceded by a blank
  line, or placed at a level other than its owning `if`;
- exact `{ }` used as an initializer or post header; and
- the parseable multiline initializer-block composition whose layout places its
  operands at conflicting levels.

Phrase-word, recognition, fencing, attachment, and other phrase-specific
diagnostics are listed by
[Zax operator phrases](operator-phrases.md#costs-diagnostics-formatting-and-source-stability).

Source diagnostics fall into ordinary syntax rejection (source that matches no
legal production), semantic errors (source that parses but violates a type,
name, or completion rule owned elsewhere), and deliberate intent or layout errors
(a bounded set of near-miss patterns whose tokens may parse but present
contradictory intent).

Intent diagnostics distinguish:

- a **layout-intent error**, where indentation, brace placement, clause
  attachment, or physical-line position contradicts parsed structure;
- an **operator-attachment intent error**, where whitespace, adjacency, or
  grouping presents another fixity or number of operations;
- a **redundant-structure intent error**, where two mechanisms claim one
  continuation or structural role; and
- a **confusable-form intent error**, where a coherent recognized form is gated
  because its natural spelling strongly resembles a damaged neighboring form and
  ordinary whitespace/grouping cannot preserve the intended source.

A keyword-role conflict is a related lexical interpretation question rather than
permission to ignore grammar. These terms are defined by
[language-design terms](terms.md).

Most rules in this document are deliberate intent or layout errors. The category
is intentionally bounded and does not require recognizing every malformed token
sequence.

Exact diagnostic identifiers and presentation belong to later diagnostics
design. These source-validity conditions remain mandatory errors rather than
configurable warnings.

## Boundaries and maturity

This document owns the accepted programmer-facing source model, not a formal
grammar, parser algorithm, formatter implementation, or syntax-highlighting
architecture.

`;;` and `??` are accepted header-section and conditional-expression separators.
Each requires whitespace on both sides and is distinct from `;` statement
composition; neither may be inferred from `;`. Their flow-control and
conditional-expression meaning is owned by
[core flow control](core-flow-control.md).

Contextual keyword recognition is accepted as a positional rule: a spelling is a
keyword only where its keyword construct is grammatically permitted. Words inside
a declared or fenced operator phrase carry phrase roles instead. The complete
keyword catalog and each construct's grammar remain with their own owners and
future work.

Legacy material places compiler directives adjacent to, within, or around source
constructs, but their placement and attachment semantics remain later
compiler-directive design. A directive on one physical line and a related
construct on the next do not use `\` merely to express that relationship.
Explicit continuation retains its single purpose of continuing one statement
across physical lines.

Future phrase/source work must evaluate a keyword-neutral, tree-transparent
`bare{...}` source enclosure. It is not current Zax syntax. The candidate uses
the contiguous opener `bare{`; bare `bare` is not a keyword, and `bare {` is not
the same form.

The candidate payload must independently form exactly one complete expression or
effective statement and cannot obtain a missing operand or separator from
outside. The enclosure creates no scope. After that completeness check, it is
transparent to final precedence and mixfix matching while preserving
keyword-neutral and confusable-form intent. Semicolon composition, continuation,
qualification, protection, and lifetime remain unchanged. Strict neutralization,
unavailable nesting, and the enclosure's interaction with reflection and
formatting remain with that future work; the phrase no-spanning rule above is
already stable regardless of the enclosure's final spelling.

Comma-list continuation is the ordinary list-level implicit continuation because
the parsed comma already requires another element. Declaration colon and mixfix
component-list continuation are separate grammar-open construct rules. None
establishes a general rule that incomplete expressions continue automatically.

[Declaration and binding behavior](declarations-and-bindings.md) is defined by
its current conceptual owner. Operator semantics, candidate-tree formation, and
precedence are defined by [operators](operators.md), the
[operator catalog](operator-catalog.md), and
[mixfix operators](mixfix-operators.md). Individual flow-control behavior,
scope-exit effects, literal realization, and documentation payload
interpretation remain with their applicable owners or future work.

For Zax's accepted foundational direction, see the
[language vision](vision.md).
