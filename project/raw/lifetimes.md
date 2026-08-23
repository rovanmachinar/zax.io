# Raw input: lifetime strategies

| Field | Value |
| --- | --- |
| Status | Raw placeholder / non-authoritative |
| Audience | A future numbered work item refining pointer and lifetime concepts |
| Applies To | Intentional lifetime-policy plurality and its costs |
| Owns | Preservation of aligned direction and required future decisions |
| Does Not Own | Accepted policy names, representations, or conversion rules |
| Source / Provenance | Work item `001`, Zax purpose and design principles |

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

## Activation and retirement

Activate this input before promoting pointer or lifetime documentation. Consume
its findings through that work and retire or archive this placeholder afterward.
