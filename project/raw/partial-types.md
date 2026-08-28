# Raw input: partial type extensions

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work defining partial, open, or externally extended types |
| Applies To | Ownership, coherence, import, layout, and mixfix-extension pressure |
| Owns | Preserved partial-type risks and the possible owner-authorized mixfix route |
| Does Not Own | Accepted partial syntax, extension authority, layout, imports, or mixfix behavior |
| Source / Provenance | Legacy `partial.md` evidence and operator review |

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

Future work must decide:

- who may extend a type;
- whether the original owner opts in;
- whether partial pieces may add ordinary, circumfix, call/index, or mixfix
  operations;
- import and visibility behavior;
- conflicts among partial pieces;
- source compatibility when an added mixfix replaces decomposition;
- construction/destruction interception;
- stored-member and layout changes; and
- reflection and build-order independence.

Import or declaration order must not resolve conflicts.

## Activation and retirement

Activate this input for partial types, extension methods, open types, mixins, or
external type completion. Move accepted behavior into type, module/import,
layout, and operator owners, then retire this file.
