# Raw input: lifetime strategies

| Field | Value |
| --- | --- |
| Status | Raw placeholder / non-authoritative |
| Audience | A future numbered work item refining pointer and lifetime concepts |
| Applies To | Intentional lifetime-policy plurality and its costs |
| Owns | Preservation of aligned direction and required future decisions |
| Does Not Own | Accepted policy names, representations, or conversion rules |
| Source / Provenance | Work items `001`, `005`, and `006`; Zax purpose, construction/replacement aliasing, and invocation lifetime pressure |

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

Activate this input before promoting pointer or lifetime documentation. Consume
its findings through that work and retire or archive this placeholder afterward.
