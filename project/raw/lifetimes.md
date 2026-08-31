# Raw input: lifetime strategies

| Field | Value |
| --- | --- |
| Status | Raw placeholder / non-authoritative |
| Audience | A future numbered work item refining pointer and lifetime concepts |
| Applies To | Intentional lifetime-policy plurality, its costs, and displaced pointer/reference binding replacement |
| Owns | Preservation of aligned direction, displaced binding-replacement syntax, and required future decisions |
| Does Not Own | Accepted lifetime semantics or current qualifier/construction behavior |
| Source / Provenance | Work items `001`, `005`, and `006`; Zax purpose, construction/replacement aliasing, invocation lifetime pressure, and the corrected declaration-side `final`/`varying` model |

## Aligned direction

Multiple lifetime strategies are intentional because they provide different
guarantees and carry different runtime, memory, synchronization, and compile-time
costs.

Policies may support meaningful upgrades or downgrades where their guarantee
boundaries permit them. Complexity requires strong documentation and tooling but
is not by itself a reason to collapse the strategies into one model.

## Construction-derived replacement pressure

[Zax construction, replacement, and destruction](../../language/construction-and-destruction.md#self-aliasing-and-interior-aliases)
establishes that a replacement right-hand operand may alias the destination or
one of its members:

```zax
value = value
value = viewOf(value)
value = makeReplacementUsing(value)
```

Future lifetime work must distinguish:

- an independent by-value snapshot;
- a reference whose exact pointee lifetime is retained until its final use;
- exact self-aliasing handled by the selected operation;
- an interior alias to a member whose lifetime ends;
- a place-tracking reference that may observe the replacement lifetime; and
- a raw pointer whose address remains usable only under programmer
  responsibility.

Same-address reconstruction does not by itself preserve the old pointee
lifetime. Conversely, retaining the exact member lifetime may preserve a pointer
or reference into that member.

Move, copy, and `last` must also identify:

- when source authority transfers;
- who destroys the source shell;
- which resources remain with the old destination;
- which resources move from the right-hand operand; and
- how every resource receives exactly one final disposition.

One lifetime strategy may reject replacement while a stable reference exists.
Another may permit the programmer to proceed through a narrow unsafe permission.
The final policy set must make the difference and cost visible.

## Displaced pointer and reference binding replacement

Qualifier and declaration owners now use declaration-side `final`/`varying` to
express whether *this* declaration may replace the whole value it reaches. See
[Zax qualifiers](../../language/qualifiers.md#type-side-truth-versus-declaration-side-permission).

```zax
myType varying : MyType varying

myAliasType final : MyType varying & = myType
```

`myAliasType` cannot initiate replacement of the referent, while its type use
still records that `myType` may.

That resolves a real conflict, but it displaces a question the previous model
answered implicitly. Earlier owners used name-side `final` on a pointer or
reference declaration to describe the *binding's own* replacement place, so a
name-side-final writable reference could still reconstruct a varying referent.
Under the current model, name-side `final` restricts replacement through the
declaration instead, and the binding-place meaning has no spelling.

Future pointer, reference, or lifetime work must recover it and decide:

- how a program independently replaces or rebinds a pointer or reference binding
  while leaving the referent untouched;
- whether rebinding is spelled as a qualifier, an operator, or a distinct
  operation;
- how the binding's own place stance is expressed at each indirection level;
- whether a reference may be rebound at all, or only a pointer;
- how rebinding interacts with declaration-side replacement permission on the
  same declaration; and
- how the two concepts are diagnosed distinctly so a programmer is never told
  "final" when the other meaning was intended.

Silently retaining both meanings for one word is not acceptable. Until this is
decided, current documentation states the declaration-side meaning and defers the
binding-replacement syntax by subject.

## Future decisions

A future owner must define:

- the final policy set;
- guarantees and failure modes;
- representation and runtime cost;
- allowed conversions;
- upgrade and downgrade behavior;
- thread and async implications; and
- a readable selection guide;
- reference-origin and interior-alias tracking;
- replacement while place-tracking or stable references exist; and
- pointer behavior when an address remains but the intended pointee lifetime
  changes.

## Pointer-representation and memory-domain pressure

Current pointer-representation integer capacities are owned by
[Zax integers](../../language/integers.md#pointer-representation-integer-roles). This
section retains actual pointer validity, transfer, provenance, and lifetime
behavior.

CPU profiles provide ordinary, near, and far pointer-capacity domains satisfying:

```text
Near pointer capacity <= ordinary pointer capacity <= Far pointer capacity
Near.TypeSize.maximum <= TypeSize.maximum <= Far.TypeSize.maximum
Near.IndexSize.maximum <= IndexSize.maximum <= Far.IndexSize.maximum
```

When near or far representations mirror ordinary pointers, their public intent
identities remain distinct rather than becoming transparent aliases on only some
targets.

Pointer-representation integers are freely admitting numeric capacity types.
Numeric transfer uses exact, optional, or narrowing admission according to the
range relationship. Actual pointer-object transfer remains separate because
numeric fit does not establish segment, tag, provenance, permission, arena, or
lifetime validity.

`pointerA delta pointerB` must not overflow for a semantically valid difference,
but does not make arbitrary pointers comparable. Future pointer work must decide
comparability, failure, and the associated `PointerDelta` result identity.
Unsigned pointer distance remains related pressure with the same validity
boundary.

## Same-storage identity-reference pressure

A future identity admission may return `MyIdentity &` viewing existing
underlying storage rather than constructing another value. Lifetime work must
decide:

- whether the source reference outlives every identity view;
- whether a temporary source may be viewed;
- whether underlying and identity references may coexist;
- how moves, replacement, and destruction invalidate either view;
- whether a returned view preserves its exact origin; and
- how readonly/writable and final/varying permissions project.

No independent identity lifetime or destructor is created for a reference view.

## Invocation-derived lifetime pressure

[Zax function invocation](../../language/function-invocation.md) establishes the
synchronous call boundary:

- explicit arguments bind at their source evaluation point;
- an argument temporary bound to a reference survives through the selected call;
- nested producer results survive through their transfer into an outer call; and
- that temporary extension does not automatically make a returned reference
  valid after completion.

Future lifetime and ownership work must define:

- exact destruction order among caller argument and result temporaries;
- reference origin through nested result routing;
- when a returned reference aliases argument or result temporary storage;
- moved-from source state after copy, move, or `last`;
- source-shell destruction and resource disposition;
- when a lifetime strategy may extend, reject, or unsafely permit an escape; and
- how an async call broadens the synchronous completion boundary.

## Activation and retirement

Activate this input before promoting pointer or lifetime documentation, or when
independent pointer/reference binding replacement or rebinding syntax is needed.
Consume its findings through that work and retire or archive this placeholder
afterward.
