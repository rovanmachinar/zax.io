# Zax source structure

| Field | Value |
| --- | --- |
| Status | Current conceptual design |
| Audience | Human developers reading, writing, or evaluating Zax |
| Applies To | Programmer-facing source structure; not a formal grammar or specification |
| Implementation State | Not established by this repository |
| Owns | Statement-level newlines, explicit continuation, effective statements and bodies, semicolon composition, mandatory layout validation, and comment forms and attachment |
| Does Not Own | [Declaration and binding behavior](declarations-and-bindings.md), [function invocation and result routing](function-invocation.md), expression precedence, the complete delimiter catalog, compiler-directive placement or attachment, detailed flow-control semantics, exact lifetime and scope-exit behavior, documentation payload languages, diagnostic identifiers, or compiler and tooling implementation |

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
3. `\` explicitly suppresses an otherwise significant newline.

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
including destruction and scope-exit behavior, belongs to later scope and
lifetime design. Braces do not make body-local names escape.

## Semicolon composition

`;` is a statement-composition operator, not a statement terminator. It requires
a complete statement on both sides and produces one effective statement.

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
body. Leading, doubled, and trailing semicolons are errors because an operand is
missing:

```zax
; doA()       // error: missing left operand
doA();; doB() // error: missing middle operand
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

Comment trivia may appear between vertical operands:

```zax
if condition
    doA(); // The semicolon is source before this comment.
    // Explain why the next operation belongs to the same body.
    doB()
```

A physically blank line may not separate them because it visually contradicts
their composition.

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

Exact diagnostic identifiers and presentation belong to later diagnostics
design. These source-validity conditions remain mandatory errors rather than
configurable warnings.

## Boundaries and maturity

This document owns the accepted programmer-facing source model, not a formal
grammar, parser algorithm, formatter implementation, or syntax-highlighting
architecture.

`;;` appears in legacy flow-control material as a proposed sub-statement
separator. It is not defined or accepted by this source-structure model and
must not be inferred from `;` composition.

Legacy material places compiler directives adjacent to, within, or around source
constructs, but their placement and attachment semantics remain later
compiler-directive design. A directive on one physical line and a related
construct on the next do not use `\` merely to express that relationship.
Explicit continuation retains its single purpose of continuing one statement
across physical lines.

Comma-list continuation is the one accepted statement-level implicit
continuation because the parsed comma already establishes a list requiring
another element. It does not establish a general rule that incomplete
expressions continue automatically.

[Declaration and binding behavior](declarations-and-bindings.md) is defined by
its current conceptual owner. Expression semantics, individual flow-control
behavior, scope-exit effects, literal continuation semantics, and documentation
payload interpretation remain later design work. Those later concepts must fit
the source structure established here.

For Zax's accepted foundational direction, see the
[language vision](vision.md).
