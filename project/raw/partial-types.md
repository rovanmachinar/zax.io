# Raw input: partial type extensions

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work defining partial, open, or externally extended types |
| Applies To | Ownership, coherence, import, layout, phrase-extension, and mixfix-extension pressure |
| Owns | Preserved partial-type risks and the possible owner-authorized phrase/mixfix route |
| Does Not Own | Accepted partial syntax, extension authority, layout, imports, phrase behavior, or mixfix behavior |
| Source / Provenance | Legacy `partial.md` evidence and operator/operator-phrase review |

## Mixfix ownership pressure

Global mixfix declarations are prohibited. A mixfix belongs to its receiver type
because an imported declaration can replace several visible component operations
with one operation.

A future partial mechanism may permit an authorized completion:

```zax
// Illustrative future syntax.
MyExtension :: partial MyType {
  operator mixfix
    index 1
    binary '='
  ...
}
```

## Phrase extension pressure

Custom operator-phrase implementations are receiver-owned for the same reason,
and an imported extension may not add a natural phrase to a type it does not own.
See [Zax operator phrases](../../language/operator-phrases.md#receiver-ownership).

That rule creates a concrete, currently unavailable capability. A global symbolic
declaration can support a custom value appearing only on the right of an
intrinsic left operand; a phrase cannot:

```zax
intrinsicValue combines with customValue
// error: no receiver type can own this phrase implementation
```

The accepted workaround reframes the wording around the extensible receiver:

```zax
customValue accepts then combines intrinsicValue
```

Intrinsic types cannot currently be extended merely to recover the unavailable
left-receiver form. A future owner-authorized partial mechanism is the natural
place to revisit that, for example:

```zax
// Illustrative future syntax.
MyIntrinsicExtension :: partial U32 {
  operator binary 'combines with' final : (
    result : MyType
  )(
    rhs : MyType
  ) = {
  }
}
```

## Protected signatures must survive any extension

Whatever partial mechanism is adopted, it must permanently preserve
[protected intrinsic signatures](../../language/operators.md#protected-intrinsic-domains)
and the protected language-provided phrase signatures. An extension may not claim
a signature whose every operand belongs to a closed intrinsic family, and it may
not replace a reserved phrase form.

## Adopted-versus-reserved phrase conflict

A partial extension that supplies a phrase form creates a version-evolution
hazard that ordinary member extension does not: a later Zax version may adopt and
reserve the same exact word sequence.

Future work must decide what happens when:

- the language later reserves a form an extension already supplies;
- the language later defines the form as open, so the extension becomes an
  ordinary permitted implementation at a form-specific precedence level;
- two independent extensions supply the same exact form to the same type; and
- an extension's form changes the precedence of existing valid source because a
  language-defined form is not at ordinary phrase precedence.

Adding, removing, lengthening, reserving, or repricing a phrase form is already a
source-compatibility event. Allowing external declarations multiplies the surface
on which that event can occur.

## Future work must decide

- who may extend a type;
- whether the original owner opts in;
- whether partial pieces may add ordinary, phrase, circumfix, call/index, or
  mixfix operations;
- whether intrinsic types may be extended at all;
- import and visibility behavior, including whether an extension's private
  members participate in the owning type's private context;
- conflicts among partial pieces;
- source compatibility when an added mixfix replaces decomposition or an added
  phrase creates ambiguity;
- construction/destruction interception;
- stored-member and layout changes; and
- reflection and build-order independence.

Import or declaration order must not resolve conflicts.

## Activation and retirement

Activate this input for partial types, extension methods, open types, mixins,
external phrase or mixfix extensions, or external type completion. Move accepted
behavior into type, module/import, layout, and operator owners, then retire this
file.
