# Raw input: remaining callable-adjacent pressure

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work defining optional combinators, restricted capture scopes, callable generics/reflection, growing a pack per item, short-form lambdas, or advanced call/mixfix integration |
| Applies To | Callable-adjacent material deliberately not resolved by current lambda, composition, and variadic design |
| Owns | Optional callable construction/combinator pressure; restricted `scope` capture evidence; advanced `operator call`/mixfix pressure; generic and reflection questions; growing a pack with several values per item at the end of a per-item chain; short-form lambda pressure; activation and retirement |
| Does Not Own | Current lambda, capture, callable-storage, composition, chaining, or per-item mapping behavior |
| Source / Provenance | Retired legacy function-composition material, optional review, operator review, work item `027` disposition, and work item `037` variadic deferrals |

## Current behavior moved

Current lambda expressions, capture construction, callable storage and receiver
lifetime, partial application, positional and reshape composition, immediate
chaining, reset, weak invocation, and costs are now owned by
[Zax lambdas and callable composition](../../language/lambdas-and-callable-composition.md).

Per-item chains over a pack, `from each item : in items |> stages`, are owned by
[Zax variadics](../../language/variadics.md#transforming-each-item).

This raw file must not be used to recover competing composition (now `>|>`),
`|>`, capture, or
receiver-reattachment behavior. The deferred pressure below on
[growing a pack](#growing-a-pack-per-item) and
[short-form lambdas](#short-form-lambdas) concerns forms current design rejects
or does not provide; it changes no current `|>` behavior.

## Restricted captured scope

Legacy input proposed a non-callable restricted scope:

```zax
// Legacy illustrative syntax.
scope [ myInput, &myOutput ] {
  myOutput = transform(myInput)
}
```

Future flow/scope work must decide whether this remains independently useful or
is better expressed by ordinary declarations, a lambda, or a direct call. If it
remains, names absent from the list are unavailable inside the scope, ordinary
capture copies, and explicit reference capture borrows one fixed place.

## Optional callable construction

An existing optional may construct a present callable through an ordinary
construction packet:

```zax
callback : MyCallback?
callback = [{ doSomethingFunc }]
```

Future optional/callable integration may permit an inline lambda as the one
packet input. Packet braces and lambda body braces must remain distinct, and
`[{}]` remains the zero-entry construction packet.

## Optional combinators

Optional values may benefit from operations such as:

```zax
fold(optionalValue, onPresent, onAbsent)
```

Future work must define:

- only the selected callback executes;
- how a present callback receives boxed qualifications and lifetime proof;
- how outer transfer stance reaches the callback;
- result convergence between present and absent callbacks;
- capture, allocation, temporary, and indirect-call costs;
- async callback behavior; and
- protected operation versus library ownership.

This pressure does not establish a general monad abstraction. A reusable model
would require concrete evidence across several value families.

## Call-form mixfix pressure

A custom callable may use a `call N` mixfix component. Future work must
integrate labels, defaults, variadics, result forwarding, and generated callable
types without creating an intermediate call result for a consumed component.

## Growing a pack per item

### Current rule

In a per-item chain, intermediate stages may pass several results along, but
each item's last stage must produce exactly one value, so the new pack has the
same count as the source pack:

```zax
split final : (head : String, tail : String)(text : String) = {
}

join final : (result : String)(first : String, second : String) = {
}

show final : ()(lines ... : String) = {
}

rejoin final : ()(lines ... : String) = {
  show(from each line : in lines |> split() |> join())   // fine: one value per item at the end
  show(from each line : in lines |> split())             // error: split's tail is not discardable
}
```

### What flattening would mean

Allowing the second call would be predictable: item by item, the results would
arrive as `head0, tail0, head1, tail1`, and the count would still be known while
compiling. In favor, flattening would treat several results as several values,
consistent with several results never becoming a tuple. Relaxing the error
later is source-compatible, so deferring it costs nothing.

It stays an error because of these hazards:

- **Silent shifts.** Adding a result to a producer would grow every pack built
  from it and shift every later position. Because an expansion can fill ordinary
  parameters, the shifted call could still type-check. Under the one-value rule,
  the same producer change fails loudly.

  ```zax
  produceTwo final : (low : Integer, high : Integer)(item : Integer) = {
  }

  setRange final : ()(min0 : Integer, max0 : Integer, min1 : Integer, max1 : Integer) = {
  }

  setBoth final : ()(pair ... : Integer)
    requires pair count == 2 = {
    setRange(from each value : in pair |> produceTwo())
    // error today. If flattening were allowed and produceTwo later added a
    // `mid` result, positions would shift; against a pack or defaulted
    // trailing parameters, mid could reach max0 silently.
  }
  ```

- **Labels are lost and results interleave.** `low0, high0, low1, high1` gives a
  callee no way to take all lows and then all highs. It cannot regroup them
  either, because no form walks a pack by computed index or walks two packs
  together.

  ```zax
  plotPoints final : ()(coordinates ... : Integer) = {   // reads x0, y0, x1, y1, ...
  }

  toPoint final : (x : Integer, y : Integer)(sample : Integer) = {
  }

  plotAll final : ()(samples ... : Integer) = {
    plotPoints(from each sample : in samples |> toPoint())
    // error today. Under flattening, a later z result would make the pack
    // x0, y0, z0, x1, ..., and plotPoints would still compile but misread it.
  }
  ```

What discardable results would do is already settled. The last stage
contributes its first result, and a discardable result never joins a pack.
Under flattening, every non-discardable result would join, and discardable ones
would not. A pack whose size depends on which overload each item's type
selects was considered and is not a concern.

### Combining instead of spreading

Current design keeps each item's parts together as one value. A `-<` stage
combines several results into one value per item, so labels survive as member
names and the pack keeps its size:

```zax
plotPointValues$ final : (Points ... : type)()(points ... : Points) = {
}

plotAllPoints final : ()(samples ... : Integer) = {
  plotPointValues(from each sample : in samples |> -< toPoint())   // one value with x and y per sample
}
```

A producer that returns one structured value does the same without `-<`:

```zax
MyRange :: type {
  low : Integer
  high : Integer
}

produceRange final : (range : MyRange)(item : Integer) = {
}

setRanges final : ()(ranges ... : MyRange) = {
}

markAll final : ()(items ... : Integer) = {
  setRanges(from each item : in items |> produceRange())   // one MyRange per item
}
```

### If it is reopened

An explicit opt-in that states the per-item result shape at the call site would
address the silent-shift hazard. No spelling is proposed. Reopening must also
settle how a receiver can use interleaved values.

**Activates when** a real use case needs several separate values per item
spread into a pack, where one combined value per item made with `-<` does not
serve.

**Constraint on current design.** Each item's last stage yields exactly one
value.

## Short-form lambdas

A compact stored lambda form remains possible future work. Per-item work no
longer needs one: a stored lambda, or any function, can already be a stage of a
per-item chain.

```zax
twice final := [[]] (result : Integer)(value : Integer) {
  return value * 2
}

collect final : ()(values ... : Integer) = {
}

twiceAll final : ()(values ... : Integer) = {
  collect(from each value : in values |> twice())   // the stored lambda is the stage
}
```

No short source form exists, and none is proposed here. A future form would
need its own capture rules.

**Activates when** there is a concrete need for a compact stored lambda.

**Constraint on current design.** None; a future short-form lambda can become a
chain stage like any callable.

## Generic and reflection pressure

Generic lambdas, with `$` on a `final` binding name, are current design in
[Zax generics](../../language/generics.md#generic-lambdas).

Future generic work must decide:

- specialization identity and body minting;
- type erasure and finite runtime callable surfaces; and
- interaction with exact callable prototypes.

Future reflection must distinguish:

- anonymous generated receiver identity;
- capture declarations and source presentation;
- fixed/varying and bound/unbound static storage capacity;
- the installed callable binding kind;
- visible prototype versus minted implementation;
- receiver ownership/observation mode;
- source availability and reset state; and
- generated composition stages and reshape mapping.

No exact reflection source form is accepted here.

## Activation and retirement

Activate this input for restricted scopes, optional callback construction or
combinators, callable generics beyond current generic lambdas, callable
reflection, growing a pack per item, short-form lambdas, or advanced call/mixfix
integration.

Move accepted behavior into its domain owners. Retire this file when every
remaining concern has been promoted, rejected, or moved to a narrower indexed
input.
