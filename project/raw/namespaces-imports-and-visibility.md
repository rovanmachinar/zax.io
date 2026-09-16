# Raw input: namespaces, imports, and visibility

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work defining namespaces, imports, aliases, exports, visibility, and declaration forwarding |
| Applies To | Name exposure and declaration identity across lexical, type, namespace, module, and import boundaries |
| Owns | Preserved literal-operator alias/import pressure, exact-alias identity, collision behavior, visibility, forwarding, representative source, activation pressure, and retirement criteria |
| Does Not Own | Accepted module/import syntax; current qualified-path lookup ([declarations and bindings](../../language/declarations-and-bindings.md)); or current literal payload/result behavior ([literal source and operators](../../language/literal-source-and-operators.md)) |
| Source / Provenance | Literal-operator review together with legacy namespace, import, alias, and forward-declaration material |

## Why this input exists

Current declaration design establishes one lexical identifier namespace and
deterministic qualified paths. Current literal design permits namespace- or
type-owned literal declarations and future shorter aliases, but it does not
finish module import/export or literal forwarding.

Literal merge creates an exact declaration-identity requirement:

```zax
MyNamespace.x'first' <|> x'second'
// Valid only when x independently resolves to the exact MyNamespace.x
// literal-operator declaration.
```

The qualified side must not choose the unqualified side. If `x` is ambiguous,
the merge is ambiguous. If it resolves to another parser with the same result
type, merge fails because equal result identity is not equal declaration
identity.

## Preserved questions

Future work must decide:

- exact namespace and module declaration syntax;
- how a type- or namespace-owned literal operator is exported;
- how an exact short alias to an operator declaration is written;
- whether an alias preserves every visibility and availability property;
- import selection, re-export, renaming, collision, shadowing, and diagnostics;
- private literal declarations and caller eligibility;
- whether an exact alias and its target share reflected declaration identity;
- how incomplete namespaces and pending path suffixes interact with operator
  aliases;
- the final role, if any, of `forward operator literal`; and
- source-stability effects when imports or exports add another visible prefix.

No source, declaration, import, module, or search order may break an ambiguity
tie. A qualified prefix remains the stable repair.

## Representative failures

```zax
x'payload'
// error when two visible declarations expose x

MyNamespace.x'first' <|> x'second'
// error when x independently resolves elsewhere

myValue : MyType = parse'payload'
// MyType does not cause lookup of MyType.parse
```

`<+>` between exact aliases of one literal declaration still requires:

```zax
intent<same-prefix-literal-join>{
  MyNamespace.x'first' <+> x'second'
}
```

The alias affects identity, not the distinction between one parser invocation
and two.

## What this input does not decide

This file does not accept the legacy module loader, import directives, export
directives, first-import-wins behavior, or forward syntax. It does not decide
general external operator extension, generic alias identity, partial types, or
runtime dynamic lookup.

## Activation and retirement

Activate this input when namespace/module syntax, import/export, operator
aliases, declaration visibility, shadowing across modules, or literal forwarding
is reviewed.

Move accepted behavior into module, declaration, source, literal, reflection,
and diagnostic owners. Retire this file when every preserved question has an
accepted owner or explicit narrower deferral.
