# 037: Variadic arguments

| Field | Value |
| --- | --- |
| Status | Historical / non-normative / audit-only; closed 2026-11-12 |
| Work Item | `037` |
| Created | 2026-11-04 |
| Owns | The bounded review of variable-length arguments and variadic compile-time parameters, as defined below |
| Does Not Own | Current generic behavior, which is design owned by `language/generics.md`; source generation; async; the decision itself, which belongs to the language maintainer |

## Non-authority notice

This file is a collaborative working record. Existing documentation, candidate
syntax, examples, and later aligned findings remain non-authoritative until a
separately discussed, aligned, and explicitly authorized promotion incorporates
them into their lasting owners.

## Fixed initiating input

This section records the aligned information known when work item `037` was
created. It is intentionally incomplete and must not be rewritten as work
develops.

### Initiating concern

How should variable-length arguments work in Zax? In particular: how a
declaration accepts a variable number of arguments, how callers supply them,
how the body receives, counts, and walks them, how they interact with
required, named, and defaulted parameters and with callable selection, how
they are forwarded with transfer stances, and how variadic compile-time
parameters work: a variable number of `$` parameters, and variadic inputs
whose every argument is a compile-time value.

### Motivating pressure

- **Current owners defer variadics.** Function invocation, mixfix operators,
  lambdas and callable composition, and operator phrases list variadics as
  future work.
- **Generics is now current design.** `language/generics.md` supplies
  compile-time inputs, `[<constant>]`, processing once per distinct set of
  compile-time inputs, worked-out types that keep qualifiers and references,
  instantiation identity, constraints and `requires`, and specialization
  ranking. Variadics should build on that model rather than invent a parallel
  one.
- **Legacy input is unreviewed.** The root `variadic.md` page covers walking
  `...` arguments with `each`, required arguments before a variadic tail,
  `count of`, variadic forwarding with transfer stance, and variadic types.
  The raw feature catalog preserves the legacy `print([<compile>] ...)` form,
  in which every argument had to be a compile-time value.

### Starting input

There are no maintainer notes for this item. The maintainer asked the agent to
give advice on how variadics should work, given the findings so far, and to
propose a recommended model in the working record for review. That proposal is
candidate analysis, not aligned design.

Legacy and raw material is evidence. It is not authoritative over current
design or over the maintainer's direction in discussion.

### Starting boundaries

- In scope: declaring, supplying, receiving, counting, walking, forwarding,
  and selecting among variadic callables; variadic compile-time inputs; and
  variadic generic parameters, meaning a variable number of `$` parameters.
- Variadic composition, variadic mixfix holes, and variadic operator phrases
  are reviewed far enough to record constraints, and are otherwise captured
  for later items unless a concrete consequence makes them necessary.
- Source generation and async remain outside this item.
- Generics and compile-time code are current design. A conflict with either is
  recorded as a finding for discussion, not resolved by silently changing it.

### Initial stopping guidance

Establish the foundation in order: first the runtime variadic model
(declaration, supply, receiving, counting, walking, forwarding, selection),
then compile-time variadic inputs, then variadic generic parameters. Give each
legacy and raw variadic evidence item a disposition (promote, defer, or
reject) rather than designing every variadic feature in full.

Creating and routing this work item does not authorize analysis. A later
assignment begins it.

Do not promote findings, change current owners, archive this work item, or
create work item `038` without the separately required discussion, alignment,
and authorization.

## Reading scope

### Required

- [Documentation architecture](../documentation.md) - governs focused reading,
  authority, promotion, deferral, and closure.
- Legacy [`variadic.md`](../../variadic.md) - the primary legacy evidence for
  variadic declaration, `each` traversal, required arguments, `count of`,
  forwarding, and variadic types.
- [Zax function invocation](../../language/function-invocation.md), especially
  [positional and named inputs](../../language/function-invocation.md#positional-and-named-inputs),
  [omitted inputs and defaults](../../language/function-invocation.md#omitted-inputs-and-defaults),
  [candidate selection](../../language/function-invocation.md#candidate-selection),
  and its boundaries - the fixed-arity model variadics must extend.
- [Zax generics](../../language/generics.md), especially
  [two forms of generic](../../language/generics.md#two-forms-of-generic),
  [supplying compile-time arguments](../../language/generics.md#supplying-compile-time-arguments),
  [when two instantiations are the same](../../language/generics.md#when-two-instantiations-are-the-same),
  and [specializations](../../language/generics.md#specializations) - the
  compile-time model variadic inputs and variadic `$` parameters build on.
- The variadic-functions entry in the
  [raw feature catalog](../raw/feature-catalog.md) - the preserved
  compile-time variadic pressure.

### Consequence-driven

Read only the smallest relevant sections when the concern reaches them:

- [Zax iteration](../../language/iteration.md), when variadic arguments are
  walked with `each`.
- [Zax arrays and slices](../../language/arrays-and-slices.md), when variadic
  arguments are received as an array or slice.
- [Zax transfer stances](../../language/transfer-stances.md), when variadic
  arguments are forwarded or consumed.
- [Zax lambdas and callable composition](../../language/lambdas-and-callable-composition.md)
  and [raw function composition and chaining](../raw/function-composition-and-chaining.md),
  when variadic callables are composed, partially applied, or stored.
- [Zax mixfix operators](../../language/mixfix-operators.md) and
  [Zax operator phrases](../../language/operator-phrases.md), when variadic
  holes are reached.
- [Zax compile-time code](../../language/compile-time-code.md), when
  compile-time variadic inputs or per-input processing are reached.
- [Raw callable selection](../raw/callable-selection.md), when variadic and
  fixed-arity candidates compete.
- [Raw type parameters and generics](../raw/type-parameters-and-generics.md),
  when variadic generic parameters meet deferred generic aliases or families.

### Audit-only

- Archived numbered work, only when a provenance question cannot be answered
  from current owners or raw input. The generics review is
  `036-generics.md`.

## Working record

### Session handoff (2026-11-11): read this first

*Historical. This item closed on 2026-11-12; see "Closure" at the end of this
file.*

**Where the work stands.** Design is aligned, the pre-promotion dry run passed
(dry run 3 re-check), and the findings have been promoted into current owners.
The promoted prose has also had a readability rewrite, reviewed for meaning
drift. Nothing is committed. The next phase is **maintainer review of the
promoted documentation**.

**What the next session's job is.** The language maintainer will supply
feedback in the new chat. That feedback may include:

- proposed edits to the promoted pages;
- teaching and wording problems;
- findings that refine or change the model.

The session reads enough now to act on that feedback, then waits for it. It
does not hunt for problems or start revisions on its own.

For each piece of feedback, it:

1. understands it against the aligned findings and the promoted text;
2. discusses it and traces consequences across owners;
3. records any aligned refinement of the model in this working record;
4. applies only explicitly authorized edits, keeping every affected owner
   coherent.

A model change that contradicts an aligned finding is recorded here as a
revision of that finding, with the maintainer's reasoning, before owners are
edited.

**Version-control state to preserve.** The maintainer has staged part of the
promotion:

- staged: `basics.md`, `index.md`, `language/arrays-and-slices.md`,
  `language/compiler-directives.md`, the deletion of `variadic.md`, this file,
  `project/README.md`, and `project/archive/work/036-generics.md`;
- unstaged, on top of those staged versions: the later edits to
  `language/arrays-and-slices.md` and `language/compiler-directives.md`;
- unstaged: the remaining promoted `language/` and `project/raw/` edits;
- untracked: `language/variadics.md`.

Check `git status` at the start and report it. Never stage, unstage,
commit, or discard without explicit authorization.

**Reading for the next session.** Read in this order. Expand only when
feedback makes more material relevant.

1. `project/documentation.md`, which governs teaching, promotion, and
   closure.
2. In this file:
   - this handoff;
   - the review entry point and its model in one screen;
   - the aligned findings A1 to A34, with the dry run 2 and dry run 3
     resolutions, which refine A15, A16, A30, and A31;
   - "Rejected alternatives";
   - "Deferrals";
   - the "Promotion" and "Post-promotion review and resolutions" sections.
   Earlier dry-run text is history; its resolutions are authoritative over
   its open questions.
3. `language/variadics.md` in full: the teaching home.
4. The promoted sections of the other owners, located with
   `git diff HEAD -- <file>`:
   - `language/function-invocation.md`: the routing rule, incidental
     labels, inputs around a pack, result packs, variadic candidates, and
     function values;
   - `language/lambdas-and-callable-composition.md`: lambdas with packs,
     partial application, composition and reshape, and chains;
   - `language/generics.md`: variadic `$` parameters, generic lambdas, and
     every-element constraints;
   - `language/structural-shapes-and-compatibility.md`: `>-`, `[N]`, and
     callable reshape;
   - smaller additions in `declarations-and-bindings.md`, `except.md`,
     `transfer-stances.md`, `iteration.md`, `arrays-and-slices.md`,
     `source-structure.md`, `compiler-directives.md`, `operator-catalog.md`,
     `operators.md`, `terms.md`, `using.md`, and `mixfix-operators.md`.

**Teaching standard for any revision.** Revise in each owner's own voice. A
section opens with the reader's goal, then an example, then the plain
outcome, then the exact rule. Keep examples valid and self-contained. Never
cite this work item or finding numbers in current owners. Keep headings
stable, because owners link to their anchors. After substantial prose edits,
a separate agent should review for meaning drift and readability before the
work is reported done.

**Still open before closure.** None of these is started:

- the maintainer's review and any revisions it produces;
- teaching-debt disposition (no observations recorded so far);
- final validation;
- archival of this file;
- the proposed initiating input and reading scope for item 038, whose
  creation needs authorization.

### State of this record (updated 2026-11-05)

The initial reconstruction (2026-11-04) has been replaced by the model aligned
in discussion with the language maintainer. Findings marked **aligned** below
remain non-authoritative until a separately authorized promotion. Rejected
alternatives are kept briefly so promotion does not reinvent them.
Outstanding items are listed separately.

The maintainer confirmed that tracing consequences into other current owners,
notably immediate `|>` chaining and `reshape`, is necessary to resolve
variadics and belongs to this work item.

Examples use the aligned working spellings. Names such as `write`,
`describe`, `isScalar`, `formatMatches`, and `Compilation.error` are
placeholder helpers.

Reading performed beyond the required set, with its trigger:

- `language/iteration.md`, direct `in` traversal and stored instance members:
  walking mixed-type arguments needs a heterogeneous `each`, and one exists.
- `language/arrays-and-slices.md`, array expressions, contributions, iterable
  `count`: runtime lists, contextual `from`, and the `count` phrase.
- `language/structural-shapes-and-compatibility.md`, `>-` decomposition and
  `reshape`: spreading values into calls, and remapping carried values.
- `language/transfer-stances.md`, one stance on one complete value: what a
  stance on an expansion applies to.
- `language/lambdas-and-callable-composition.md`, partial application,
  composition, immediate chaining: the `|>` mapping surface.
- `language/except.md`, nested calls and pipelines: failure partway through a
  mapped pack.
- `basics.md`, the remaining legacy operator list for `count of`, `...`, and
  `$...`.
- Grep-only: `language/compile-time-code.md` has no compile-time loop form.

### Review entry point

#### The model in one screen

A variadic parameter absorbs any number of arguments into a **pack**: a group
of separately bound parameters whose count is known while compiling. The
function is processed once per distinct count (and per distinct type list when
types vary). A pack is not a value.

```zax
// Stated element type: no $.
sum final : (result : Integer)(values ... : Integer) = {
  total := 0
  each value : in values {
    total += value
  }
  return total
}

// Mixed element types: a variadic $ type parameter names them.
print$ final : (Items ... : type)()(items ... : Items) = {
  each item : in items {
    write(item)                    // checked once per item type
  }
}

sum(1, 2, 3)                       // sum processed for a count of 3
print("total:", 3, 4.5)            // print$(String, Integer, <real type>)
```

Using a pack inside the body:

```zax
forward$ final : (Items ... : type)()(items ... : Items) = {
  n := items count                           // compile-time count
  first := items[0]                          // compile-time index
  sendAll(from items as copy)                // expand as is
  sendAll(from each item : in items |> describe())   // map per item into a new pack
}
```

Why it is shaped this way: the legacy page called `...` a runtime list, but
its own example selected a different overload per argument inside the body.
That requires each argument's type while compiling, which makes every
variadic a compile-time-shaped feature. A same-typed list is not special; it
merely happens to be uniform.

#### Where to look next

- [Outstanding items](#outstanding-items) lists what still needs discussion.
- [Aligned findings](#aligned-findings) is ordered as a programmer meets the
  feature: declare, supply, receive, map, the `|>` and `reshape`
  consequences, select, constraints, compile-time inputs, and callables
  (function values, result packs, local packs, lambdas).

### Aligned findings

#### Declaring

##### A1. Every variadic parameter is a pack; a uniform list is not special

A pack is a group of separately bound parameters with a compile-time count,
whether its element types are stated, shared, or each worked out. A
same-typed pack is not a slice, array, or runtime list.

Maintainer rationale: if `...` became a slice whenever its types were
uniform, `...` would turn into an optimization scenario under certain
combinations of arguments.

Consequences:

- The count is a compile-time input. A variadic function is processed once
  per distinct count, and per distinct type list when types vary; compile
  points run once per processing.
- A variadic function must be `final`, for the reason generic functions are:
  a replaceable slot would need one entry per processing.
- Each element is its own parameter, so `values ... : String readonly &` is
  several reference parameters; no array of references is needed.

| Kind | Element types | `$` list | Processed once per |
| --- | --- | --- | --- |
| Stated type | Stated | None | Distinct count |
| Shared worked-out type | One type worked out from all elements | Ordinary `$` parameter | Distinct element type and count |
| Mixed types | Worked out per element | Variadic `$` type parameter | Distinct type list |

##### A2. A list known only at runtime is an array or slice

Runtime-length lists never enter packs. An API that takes one declares an
array or slice parameter. Building an array from a pack is explicit.

```zax
sumAll final : (result : Integer)(values : Integer[]) = {
}

sum(from runtimeSlice)       // error: a pack's count must be known while compiling
sumAll(runtimeSlice)         // the runtime-list API

sum final : (result : Integer)(values ... : Integer) = {
  return sumAll([ from values ])   // building an array from the pack is explicit
}
```

##### A3. `$` appears only where element types can be named or supplied

The `...` marker is itself the visible sign of per-count processing,
following the precedent of `[<constant>]`, which also causes per-input
processing without a `$` list.

```zax
sum final : (result : Integer)(values ... : Integer) = { }
largest$ final : (UseType : type)(result : UseType)(values ... : UseType) = { }
print$ final : (Items ... : type)()(items ... : Items) = { }
```

##### A4. Declaration spelling and placement

- The working spelling is `name ... : Type`, putting `...` in the
  per-parameter slot that `[<constant>]` uses. A variadic `$` parameter is
  written `Items ... : type`.
- Packs are named. The anonymous legacy `...` and `$...` are retired, so a
  bare `...` is never an expression and existing `{ ... }` documentation
  placeholders stay unambiguous.
- At most one variadic input parameter per prototype.
- A variadic `$` parameter absorbs every following positional `$(...)`
  argument; a later `$` parameter needs a label. Several variadic `$`
  parameters in one list are deferred.

```zax
zip$ final : (Lefts ... : type, Rights ... : type)()(
  lefts ... : Lefts,
  rights ... : Rights
) = { }
// error: at most one variadic input; zip-like APIs are deferred
```

#### Supplying

##### A5. Fixed parameters, defaults, and labels around a pack

Parameters before the pack map as today, including the rule that a positional
input never skips a defaulted parameter. The positional cursor stays on the
pack while positional inputs keep arriving, so a parameter after the pack is
reachable only by label. A label on the pack places the cursor on the pack,
which keeps absorbing.

```zax
connect final : ()(
  host : Host,
  attempts : Integer = 3,
  extras ... : Option,
  where : SourceLocation = Compilation.caller
) = {
}

connect(host)                           // attempts defaulted; extras empty
connect(host, 5, a, b)                  // 5 maps attempts; a and b join extras
connect(host, :, a, b)                  // attempts defaulted explicitly
connect(host, extras: a, b)             // the label starts the pack; it keeps absorbing
connect(host, a, b)                     // a maps attempts, then fails by type
connect(host, 5, a, where: otherPlace)  // a later parameter only by label
```

##### A6. Expanding as is with `from`

`from pack` contributes the items as separate positional inputs through the
ordinary cursor, wherever they land: into a pack or into ordinary parameters.
Each processing knows the count, so mapping and selection are checked per
processing. A fixed-length array expands the same way. Runtime-length sources
are errors (A2). `>-` remains the separate name-based structural spread.

```zax
sendTwo final : ()(header : Header, body : Body) = { }

forward$ final : (Items ... : type)()(items ... : Items) = {
  sendTwo(from items as copy)    // fills two ordinary parameters when the count is 2
}

forward(myHeader, myBody)        // fine
forward(myHeader)                // error for this use: sendTwo needs two inputs

coords : Integer[2] = [ 3, 4 ]
plot(from coords)                // plot(x : Integer, y : Integer)
plot(>- point)                   // members map by name: a different operation
```

#### Receiving

##### A7. A pack is not a value

A pack is several bindings, not one value. It cannot be assigned or stored as a
whole in one place. A bare pack name is valid only where a pack is expected:
`each ... in` and `from each ... in` sources, `count`, compile-time indexing,
and a lambda capture list (A27). Anywhere it contributes values, it is written
`from pack`.

```zax
saved := items                    // error: a pack is not a value
send(items)                       // error: write send(from items)
```

Several bindings may still be produced and held as packs: a function may
return a result pack (A25), a local declaration may bind a pack (A26), and a
lambda may capture one (A27). Each remains several bindings. Building a
structure from a pack is not viable without names for its items.

##### A8. Walking a pack with `each`

Packs become a direct `each in` source family that uses the stored-member
rules: the body is checked once per item type, runs in argument order, and an
explicit `copy` must be viable for every item. Type packs can be walked as
well. Per-item checking is a typing rule, not a mandated unrolling strategy.

```zax
print$ final : (Items ... : type)()(items ... : Items) = {
  each item : in items {
    write(item)
  }
  each item : copy in items {
    keep(item)                    // copy must be viable for every item
  }
}

describe$ final : (Items ... : type)()(items ... : Items) = {
  each Item : in Items {
    write(target size of Item)
  }
}
```

##### A9. Counting a pack

`items count` and `Items count` use the existing post-unary `count` phrase,
whose meaning is the exact number of entries traversal produces. For a pack
that number is a compile-time `IndexSize`. The legacy `count of` is retired.
Because the mapping form binds each item to its own separate name (A11),
`items count` always counts the pack, and `item count` is one item's own
count.

```zax
buffer : Byte[Items count]
average final : (result : Float)(values ... : Float)
  requires values count >= 1 = { }
```

##### A10. Compile-time indexing

`items[N]` and `Items[N]` select one item when `N` is a compile-time value.
A runtime index is an error because items may differ in type. An index past
the count is an error for that processing.

```zax
labelled$ final : (Items ... : type)()(items ... : Items)
  requires Items count >= 1 && isText(Items[0]) = {
  title := items[0]
  // items[i]  // error: i is a runtime value
}
```

#### Mapping a pack

##### A11. `from each item : in pack |> stages` maps per item into a new pack

`from` says that several values are contributed at this position, as it does
for `from items` and `[ from row ]`. `each item : in pack` is the ordinary
`each` binding header, and the binding name is required, as it is for the
`each` statement. The `|>` chain runs once per item, and the results form a
new pack at that position. The mapping ends at the enclosing argument's comma
or closing parenthesis.

Maintainer rationale: a binding-less `each items |> f()` did not read well,
and `each item : in items |> f()` alone would look like a loop rather than a
spread list. Leading `from` marks contribution; the `each` header names the
item.

```zax
callme(from items)                                   // the items as they are
callme(foo(from items))                              // foo receives the whole pack, once
callme(from each item : in items |> foo())           // foo per item; results form a new pack
callme(from each item : in items |> foo(), extra)    // the mapped pack, then an ordinary input
render(from each doc : in docs |> parse() |> check(doc))   // multi-stage; later stages may use doc
```

**Placing the item.** In the first stage, the carried value is the item
itself:

- If the first stage writes the item's name, the item goes only where it is
  written, and nothing is carried implicitly into that stage. Labels mean
  exactly what they mean in any call.
- If it does not, the item is one unlabeled value carried as the stage's first
  positional input, under the routing rule (A15).
- Later stages carry the previous stage's results as usual. The item's name
  stays visible in every stage of that mapping, like an `each` body binding.

```zax
funcD final : ()(width : Integer, value : Integer, height : Integer) = { }
format final : (result : String)(value : Integer, width : Integer) = { }

callme(from each item : in items |> funcD(width, item, height))         // placed where written
callme(from each item : in items |> funcD(width, value: item, height))  // a label, as in any call
callme(from each item : in items |> format(width))                      // not written: the first positional input
```

The suppression rule belongs only to a `from each` binding. An ordinary chain
such as `x |> f(x)` still passes `x` twice, because there `x` is an ordinary
variable rather than the carried value.

**Where it is valid.** Wherever several values are accepted: an argument list
or an array expression, the same places contextual `from` works. In an array
expression every item must produce one common element type:

```zax
widths$ final : (Items ... : type)(result : IndexSize[Items count])(items ... : Items) = {
  return [ from each item : in items |> measureWidth() ]   // one width per item
}

[ from each item : in items |> describe() ]   // error if describe returns different types per item
```

Calling a whole statement once per item needs no expression form. It is the
existing `each` statement, and an expression form that would repeat the
enclosing call per item is an error:

```zax
each item : in items {
  callme(foo(item))
}
```

A stance may be written at the use or on the source; on the source it applies
to each item (A13):

```zax
sendAll(from each item : in items |> consume(item as last))
sendAll(from each item : in items as last |> consume())
```

The binding header accepts the same forms as the `each` statement, for
example `from each item : copy in items |> keep()`.

##### A12. Evaluation order and pack size

- Item by item: `parse(doc0)`, `check(doc0)`, `parse(doc1)`, `check(doc1)`.
- The whole mapping occupies one argument position, so caller source order
  holds: `header()`, then every mapped item, then `footer()`.
- Each resulting value binds immediately, like any input.
- Intermediate stages may pass several results between them. Each item's
  **final** stage must produce exactly one value, so the new pack has the same
  count as the source pack. Growing a pack is future work.

```zax
split final : (head : String, tail : String)(text : String) = { }
join final : (result : String)(first : String, second : String) = { }

combine(from each item : in items |> split() |> join())   // fine: one value per item at the end
combine(from each item : in items |> split())             // error: two values per item would grow the pack
```

##### A13. Transfer stance on an expansion applies to each item

A stance restated on `from pack`, or on the source of `from each`, applies once
to each complete item. Mixed stances use separate inputs with compile-time indexes.
Each element keeps its worked-out shape, so reference items stay references.
The existing terminal-opportunity intent error applies per item, so a
forwarder states its intent; Zax keeps no silent perfect forwarding.

```zax
sendAll$ final : (Items ... : type)()(items ... : Items last) = {
}
// Each item may be taken by terminal transfer.

forwardAll$ final : (Items ... : type)()(items ... : Items) = {
  sendAll(from items)
  // error: this is each item's final use, and sendAll could take ownership;
  // state copy, move, last, or deep intent
}

forwardLast$ final : (Items ... : type)()(items ... : Items) = {
  sendAll(from items as last)    // last applies to each item separately
}
```

Different stances for different items need the items written as separate
inputs:

```zax
sendPair final : ()(header : Header move, body : Body) = {
}

forwardPair$ final : (Items ... : type)()(items ... : Items)
  requires Items count == 2 = {
  sendPair(items[0] as move, items[1] as copy)
}
```

`Items last` attaches a declared stance to a worked-out type using the
existing parameter-stance form; the exact spelling is illustrative.

##### A14. An exceptional outcome on one item stops the rest

The mapped calls are made by the function containing the expression, before
the outer callable starts, exactly as if each call were written out as its own
argument. A handler attached to the outer invocation therefore covers them,
under the existing rule that a handler covers the invocation's receiver,
arguments, and nested calls. There is no unwinding; the handler's flow
control provides the exit path.

```zax
parse final : (tree : Tree, failure except : ParseError)(text : String readonly &) = { }
render final : (page : Page)(head : Header, trees ... : Tree, foot : Footer) = { }

buildPage final : (page : Page, failure except : ParseError)(docs ... : String) = {
  page := render(header(), from each doc : in docs |> parse(), footer()) catch failure {
    report(failure)
    return
  }
  return page
}
// With three docs this behaves as:
//   render(header(), parse(doc0), parse(doc1), parse(doc2), footer()) catch failure { ... }
```

If `parse(doc1)` selects `failure`, completed values are not rolled back and
are cleaned up as already-bound arguments; later items, `footer()`, and
`render` do not run; the handler runs. `except failure` forwards in the same
way.

- Failures are not collected. Zax returns one outcome, and an aggregate would
  need the erased error object the exceptional-flow owner rules out. A
  programmer who wants processing to continue handles the outcome inside the
  stage, for example with a helper that substitutes a value, or uses an
  `each` statement.
- With mixed types, each item's `parse` instantiation may carry a different
  payload type; the existing rule checks the handler once per source path.
- The handler cannot tell which item failed unless the payload says so. That
  is left to payload design.

#### Results into inputs, `|>`, and `reshape` (cross-owner consequences)

These findings change how results are forwarded into inputs, which is owned by
`language/function-invocation.md`, the immediate-chaining surface owned by
`language/lambdas-and-callable-composition.md`, and the `reshape` surface
owned by `language/structural-shapes-and-compatibility.md`. They apply to all
code, not only packs.

##### A15. One routing rule wherever results meet inputs

A forwarded **result sequence** is the several results of one call used
directly as inputs. Wherever result sequences meet inputs, routing is:

1. **Implicit matches have mapping priority.** Each labeled result in a
   forwarded result sequence binds the parameter with exactly that label,
   unless the result's declared label or the parameter's label is
   `[<label=incidental>]` (A30). A label given by a reshape is written intent
   and binds the parameter it names, even an incidental one. Those
   parameters are consumed. "First" is mapping priority only: evaluation
   stays in strict caller source order.
2. **Then everything else, in source order, under ordinary call rules.**
   Unmatched results sit at the position where their call was written,
   alongside the written inputs. Positional inputs follow the cursor, and an
   explicit label moves the cursor as in any call. The cursor skips only
   parameters consumed by implicit matches. A parameter supplied by an
   explicit label is not skipped, so `f(b: 1, a: 2, x)` is still a
   duplicate, as today.

An implicit match can therefore displace a positional input silently:
`f(5, g())`, where `g` offers a result labeled `a`, sends `5` to the next
remaining parameter. Against an explicit label, an implicit match is a
duplicate instead.

The rule applies to a nested call, a `|>` stage, `>|>` composition (which has
no written inputs), `return g()` forwarding into result slots, and
`[{ g() }]` forwarding into constructor inputs. A chain is therefore the same
call: `f(g(), rest)` means exactly `g() |> f(rest)`.

Maintainer rationale: a programmer expects labels to match first and the
remaining items to fill positionally. The current positional-only forwarding
rule is replaced; Zax has no compatibility obligation that requires keeping
it.

```zax
produce final : (text : String, number : Integer)() = { }
produceMore final : (enabled : Boolean, count : Integer)() = { }
consume final : ()(
  flag : Boolean,
  number : Integer,
  text : String,
  enabled : Boolean = false,
  count : Integer = 0
) = { }

consume(produce(), true)                    // number, text by label; true → flag
consume(true, produce())                    // the same: matched parameters drop out
consume(true, produce(), produceMore())     // every result matches by label
produce() |> consume(true)                  // the same call as consume(produce(), true)
consume(produce(), true, enabled: true, 5)  // true → flag; enabled by label; 5 follows the cursor → count
```

Because a chain is the same call, a positional input after a label behaves
the same in both forms:

```zax
f final : ()(a : Integer, b : Integer, c : Integer, d : Integer) = { }

f(x, c: 1, y)       // x→a, c, then y→d
x |> f(c: 1, y)     // the same: x→a, c, then y→d
```

**Collisions make a candidate not viable.** Two sequences offering one label,
or an implicit match against an explicit label, supply one parameter twice.
Under the existing viability rule, a candidate is viable only when its inputs
map without duplication. If another candidate remains viable, ordinary
selection picks it. If none does, the diagnostic shows the collision. A
discardable colliding result still collides.

```zax
consume(produce(), produce())          // error: number and text offered twice
consume(produce(), number: 7, true)    // error: number from produce() and from the label

consumeAny final : ()(number : Integer, text : String, flag : Boolean) = { }
consumeAny final : ()(label : String, count : Integer, a : Integer, b : Boolean) = { }
produceOther final : (number : Integer, flag : Boolean)() = { }

consumeAny(produce(), produceOther())
// the first candidate collides on number and is not viable; the second matches
// no labels and takes text, number, number, flag by position, so it is selected
```

**Fixing an accidental match.** Say where each result goes with a postfix
reshape (A16), or label the call's inputs:

```zax
split final : (head : String, tail : String)(text : String) = { }
joinReversed final : (result : String)(tail : String, head : String) = { }

joinReversed(split(myText))            // by label: head → head, tail → tail

joinReversed(split(myText) reshape {   // to route split's first result into tail
  head: tail:
  tail: head:
})
```

To route a sequence purely by position, use the existing explicit mapping
form, `: = producer()`. It sets up source and destination cursors and does not
match labels:

```zax
joinReversed(: = split(myText))        // positional: head → tail, tail → head
```

Renaming to a label that matches no parameter also leaves a result
positional, but only while no such parameter exists, so it is a workaround,
not the taught form.

**A single result is positional.** One result is one expression value. It binds
at its position, and its label does not route it. If that label names a
different, still-unfilled parameter of the selected candidate, the call is an
intent error. The decision uses source shape, not whether types would fit, and
is checked on the selected candidate, so it never shifts selection to another
overload.

```zax
getCount final : (count : Integer)() = { }
report final : ()(total : Integer, count : Integer) = { }

report(getCount(), myTotal)                         // intent error: count would reach total by position
getCount() |> report(myTotal)                       // the same intent error
report((getCount()), myTotal)                       // still the intent error: parentheses do not change meaning
```

The fixes, in the order teaching should present them:

```zax
report(myTotal, getCount())                         // 1. reorder: position and label agree
report(count: getCount(), total: myTotal)           // 2. label the call's inputs
report(: getCount(), myTotal)                       // 3. explicit positional intent: "by position, on purpose"
report(getCount() reshape count: total:, myTotal)   // 4. reshape to the intended parameter
report(myTotal, count: = getCount())                // 5. an explicit routing group
report((: Integer = getCount()), myTotal)           // 6. an anonymous typed destination drops the label
```

Two more ways also avoid the error:

- the producer or consumer marks the label `[<label=incidental>]` (A30), so it never
  routes implicitly;
- renaming to a label that matches nothing (`reshape count: value:`), which is
  a workaround for the reason given above.

The error fires only when a result's label names a different parameter of the
selected candidate. A common label such as `value` will sometimes trigger it,
for example `pad(getValue(), width)` against `pad(width, value)`. That is
accepted: it catches a real swap.

**Variable names never route.** Labels exist only on result slots while they
are forwarded. Once a result is bound to a name, the name is the caller's
private choice. The same holds for a `from each` binding name (A11).

```zax
total := getMeATotal()
count := getMeACount()

report(total, count)                  // positional
report(count, total)                  // positional, not an error: count → total
report(count: count, total: total)    // by name, when wanted
report(count:, total:)                // not a shorthand: empty labels omit both inputs
```

**Destination names never route either.** Results go into destinations only
through existing forms, none of which match labels implicitly:

```zax
produce final : (text : String, number : Integer)() = { }

number:, text: = produce()                     // new same-named bindings, by explicit selection
// or, as an alternative in another scope:
number : Integer, text : String = produce()    // typed capture, by position: a swap fails to type-check

existingNumber : Integer
existingText : String
existingText, existingNumber = produce()       // assignment to existing places, by result order
text: existingText, number: existingNumber = produce()   // assignment, by label
```

Several bare new names, `number, text := produce()`, are not a form. They
would mix introducing names with routing, and `language/using.md` already
forbids introducing names positionally from several results so that
reordering results cannot silently change what a name denotes. The existing
short capture remains: `value := measure()` binds a new name to the first
result, and every later result must be discardable.

Positional assignment to existing places is an existing form, which current
text calls an existing positional destination. It is kept: the places already have declared types,
so most swaps fail to type-check. A same-typed swap stays silent, the same
accepted cost as `report(count, total)`. A linter may warn on name swaps, but
the language does not treat private names as routing evidence.

**Principle behind every error case:** a language error arises only when two
**meaningful declared labels** (not `[<label=incidental>]`) disagree, such as
a result label against a parameter label, or when one parameter is supplied
twice. A private name, whether a
variable or a destination, never routes and never causes an error by itself.
This is why `report(getCount(), myTotal)` is an intent error while
`report(count, total)` is not.

**Explicit routing moves the cursor; implicit matching does not.** An implicit
match consumes its parameter with nothing written at a position, so later
positional inputs fill whatever remains. An explicit label, whether
`count: value` or a routing group `count: = producer()`, is a written
instruction and moves the cursor as in any call:

```zax
getCount final : (count : Integer)() = { }
report final : ()(total : Integer, count : Integer) = { }

report(count: = getCount(), myTotal)          // error: the cursor moves past count; myTotal has no parameter
report(myTotal, count: = getCount())          // total by position, then count by name
report(count: getCount(), total: myTotal)     // the simpler form for a single result
```

Teaching presents `count: getCount()` for a single result and the routing group
for selecting one result from several:

```zax
produceDetailed final : (text # : String, number : Integer)() = { }
consumeNumber final : ()(number : Integer, flag : Boolean) = { }

consumeNumber(number: = produceDetailed(), flag: true)   // selects number; text is discardable
```

**Renaming results** uses existing forms only:

```zax
text: caption:, number: count: = produce()      // source selectors introduce new names
text: caption, number: count = produce()        // later: source selectors assign into those places
number:, text: message : String = produce()     // a typed destination introducing its own name

consumeCaption final : ()(caption : String, number : Integer) = { }
consumeCaption(produce() reshape text: caption:)   // postfix reshape relabels for matching
```

The producer's own result declaration chooses the label in the first place.

**Leftovers.** An unconsumed result must be discardable. An item of an input
pack has no discard marking, so an unconsumed one is always an error;
result-pack items follow their declared discardability (A25).

```zax
measure final : (width : Integer, height # : Integer)(shape : Shape) = { }
area final : (result : Integer)(width : Integer) = { }

area(measure(myShape))           // width by label; height is discardable, so it may go unused
myShape |> measure() |> area()   // the same
```

**Effect on mapped items.** A `from each` item is one unlabeled value carried
as the first positional input unless the first stage writes its name (A11):

```zax
format final : (result : String)(value : Integer, width : Integer) = { }
pad final : (result : String)(width : Integer, value : Integer) = { }

render(from each item : in items |> format(width))               // format(item, width)
render(from each item : in items |> pad(width: width))           // error: item → width, then width twice
render(from each item : in items |> pad(width: width, value: item))   // placed where written
```

##### A16. Reshape on an invocation, and `[N]` positional source paths

In a `|>` chain, a reshape is applied postfix to a stage's invocation, using
the existing invocation-level `reshape`. A reshape is not a separate `|>`
stage. Every chain stage after the source is an invocation, so the postfix form always has
something to attach to. Postfix `reshape` must bind more tightly than `|>`;
the operator catalog confirms that at promotion.

```zax
produce() reshape MyReshape |> consume()
```

Retained `>|>` composition keeps its reshape stage
(`parse >|> ParseToRender >|> render`) and callable reshape
(`renamedParse := parse reshape FailureNames`), because a retained
composition has no invocation to attach to. A reshape stage is defined as
exactly a relabel of the callable on its left: `f >|> R >|> g` means
`(f reshape R) >|> g`. One reshape therefore has one meaning on every callable
surface. A structural reshape used with `-<>-` keeps its own rule, in which
explicit entries consume both endpoints. Entries apply together, so swaps still work; a relabel that
duplicates a remaining label is an error:

```zax
produce final : (a : Integer, b : Integer)() = { }
consume final : ()(b : Integer, c : Integer) = { }

AToB :: reshape { a: b: }
pipeline := produce >|> AToB >|> consume        // error: two results labeled b

AToBAndBToC :: reshape {
  a: b:
  b: c:
}
pipeline := produce >|> AToBAndBToC >|> consume // a → b, b → c
```

This replaces the current stage rule that explicit entries "consume both
endpoints" before label matching.

A reshape source path may be `[N]`, meaning the Nth ordinary result in result
order, which lets a reshape name results that have no label, such as
result-pack items. `[N]` counts results whether or not they also have labels.
A reshape may name one result by either route; naming the same result both
ways is a duplicate-mapping error. `[N]` matches the compile-time index
spelling for packs (A10).

```zax
measure final : (width # : Integer, height : Integer)(shape : Shape) = { }
scale final : (result : Shape)(size : Integer) = { }
// width is discardable, so it may go unused below

SecondAsSize :: reshape {
  [1]: size:
}

measureThenScale := measure >|> SecondAsSize >|> scale         // retained: height feeds size
resized := myShape |> measure() reshape SecondAsSize |> scale()  // immediate: postfix on the invocation
resized2 := scale(measure(myShape) reshape [1]: size:)           // the same as a nested call
```

A plain value or a whole pack at the start of a chain has no invocation to
reshape, and needs none: write the call directly.

```zax
funcD final : ()(width : Integer, value : Integer, height : Integer) = { }

funcD(width, x, height)                       // a plain value is just placed

reportLast final : ()(first : Integer, second : Integer, last : Integer) = { }
relayThree$ final : (Items ... : type)()(items ... : Items)
  requires Items count == 3 = {
  reportLast(items[0], items[1], last: items[2])   // a pack: index and label directly
}
```

`[N]` counts only the ordinary success results of the invocation or callable
being reshaped. An exceptional
outcome is an alternative to the success results, not a position in a
sequence, so outcome reshape always names exceptional outcomes by label:

```zax
parse(source) reshape failure: parseFailure:   // exceptional outcomes: by label only
```

`[N]` applies only to results on a callable surface. A
structural reshape used with `-<>-` maps members by name only; members are
never addressed by position:

```zax
DeviceToRender :: reshape {
  [0]: x:          // error: structural members are mapped by name, not position
  deviceY: y:
}
```

Structured values always have member names, and positional member mapping
would silently change meaning when a type's members are reordered.

#### Selection

##### A17. Selection with packs

- **Empty pack as an omitted default.** An empty pack counts as one omitted
  callable-parameter default under the existing "fewer defaults" rule.

  ```zax
  show$ final : (First : type)()(first : First) = { }
  show$ final : (First : type, Rest ... : type)()(first : First, rest ... : Rest) = { }

  show(1)   // the first wins: the second needs an empty pack
  ```

- **Fixed beats absorbed.** An argument bound to an ordinary parameter beats
  one absorbed into a pack when the matches are otherwise equal. A fixed
  declaration accepts a strict subset of the pack's call shapes.

  ```zax
  max final : (result : Integer)(first : Integer, second : Integer) = { }
  max final : (result : Integer)(values ... : Integer) = { }

  max(1, 2)   // the two-parameter max wins
  ```

- **A same-typed pack without `$` is non-generic.** It beats an equally good
  `$` candidate, and a call written with `$(...)` never reaches it. Precedent:
  a `[<constant>]` function is processed per input yet is an ordinary
  non-generic candidate.

  ```zax
  sum final : (result : Integer)(values ... : Integer) = { }
  sum$ final : (Items ... : isNumeric)(result : Integer)(items ... : Items) = { }

  sum(1, 2)   // the no-$ sum wins
  ```

- **Self-recursion is not diagnosed.** Recursion through a pack may be
  intentional and may stop in ways a compiler cannot see. A programmer who
  wants to exclude a count uses an ordinary constraint:

  ```zax
  list$ final : (Items ... : type)()(items ... : Items)
    requires Items count != 1 = { }
  ```

#### Constraints

##### A18. "Every element" short form

A constraint function written in place of `type` on a variadic `$` parameter
requires every element type to satisfy it. It counts as one ranking term, so
the existing term rule is unchanged; two different "every" checks on one pack
are different terms.

```zax
total$ final : (Items ... : isScalar)(result : Integer)(items ... : Items) = { }

total(1, 2.5, myByte)   // fine
total(1, "two")         // not viable: element 2, String, fails isScalar
```

##### A19. `requires` over packs

`requires` covers counts, positions, and whole-pack checks. An ordinary
compile-time function may take a type pack and walk it; such a call is one
opaque term. A same-typed pack with no `$` list may still have a `requires`
clause on its count, because the count is a compile-time input.

```zax
pair$ final : (Items ... : isScalar)()(items ... : Items)
  requires Items count == 2 = { }

allSameWidth final : (result : Boolean)(Types ... : type) = {
  // walk Types with each, comparing target size of each
}

packed$ final : (Items ... : type)()(items ... : Items)
  requires allSameWidth(Items) = { }
```

##### A20. Empty packs satisfy "every"; failures name the element

`total()` satisfies `Items ... : isScalar`. A declaration needing at least one
element states `requires Items count >= 1`. A failed "every" constraint
reports the element position and type.

#### Compile-time inputs and variadic `$` parameters

##### A21. Compile-time values in a pack

`[<constant>]` on a variadic parameter requires every item to be a
compile-time value; the function is processed once per distinct list of
values. Mixing one `[<constant>]` input with a runtime pack is the main use.

```zax
printf$ final : (Args ... : type)()(
  format [<constant>] : String readonly,
  args ... : Args
) = {
  if compile !formatMatches(format, Args) {
    Compilation.error("printf: format does not match the arguments")
  }
}

tags final : ()(names ... [<constant>] : String) = { }
tags("alpha", "beta")   // processed once for this exact list
```

##### A22. Value packs in a `$` list and identity

Two packs are the same argument when they have the same length and each item
compares equal under the existing value-argument rule. An empty pack is an
argument too. A pack is never worked out from an expected result.

```zax
MyMask$(bits ... : Integer) :: type { }

a : MyMask$(1, 4, 9)
b : MyMask$(1, 4, 9)   // same type as a
c : MyMask$()          // a different type
```

#### Callables

##### A23. Function values select one processing

A variadic callable has no single function value. A fully typed function
value selects one processing by mapping its parameters onto the pack, as a
typed function value already selects a generic instantiation. This is
supported. Label differences use existing compatible-prototype adaptation.
Binding may also need compiler-generated adaptation, such as ABI glue between
the processing's calling convention and the fixed prototype; whether or not it
does, the assignment is supported, and that glue is a cost tools should be
able to show.

```zax
pickSum : (result : Integer)(first : Integer, second : Integer) = sum
```

##### A24. `operator call` may take a pack

A custom callable type's `operator call` follows the same prototype rules.
Operator `$` parameters are always worked out, so no `$(...)` supply is
needed.

##### A25. Result packs

A function may return a variable number of results as a result pack. It is a
variable number of result slots fixed per processing, consistent with several
results never becoming a tuple; existing routing, acknowledgement, elision,
and `return #` rules apply slot by slot.

```zax
passThrough$ final : (Items ... : type)(results ... : Items)(items ... : Items) = {
  return from items as last                          // or: return from each item : in items |> normalize()
}

sendAll(passThrough(a, b, c))                        // forwarding results into a call already exists
x : Integer, y : Integer, z : Integer = passThrough(1, 2, 3)   // typed capture: the caller knows the count here
```

`[N]` names unlabeled result-pack items, and a reshape changes their routing:

```zax
show final : ()(highlight : String, others ... : String) = { }

show(passThrough("a", "b", "c") reshape [1]: highlight:)
// "b" binds to highlight by label; "a" and "c" fill the remaining others pack, in order.
// Without the reshape, "a" would fill highlight.
```

- At most one result pack per prototype; a result after it is reachable only
  by label.
- Result-pack types come from inputs or from `$(...)`, never from the
  expected result.
- Acknowledgement and discardability are declared once and apply to each
  item.
- A body may construct items one by one, each checked per item type:

  ```zax
  makeDefaults$ final : (Items ... : type)(results ... : Items)() = {
    each result : in results {
      result .= [{}]            // a construction packet; each item's type is its destination
    }
  }

  first : Integer, second : String = makeDefaults$(Integer, String)()
  ```

- The explicit `as last` in `passThrough` follows A13 for the final use of
  each item. Whether terminal use when mapping into results strictly requires
  the stance, or whether by-value parameters with no declared stance avoid the
  intent error, is checked against the transfer-stances owner during the dry
  run.

##### A26. Local packs

A local declaration may bind a pack: several locals, exactly as a pack
parameter is several parameters. A generic caller needs this to hold a result
pack whose count it cannot write out. A local pack follows A7: it is never
assigned or stored as a whole.

```zax
relay$ final : (Items ... : type)()(items ... : Items) = {
  outs ... := passThrough(from items as copy)   // several locals; items stay usable
  // Not the several-bare-names form rejected in A15: pack items are anonymous
  // and reached only by index, so no per-item name can be silently redirected.
  log(from outs)
  sendAll(from outs as last)
}
```

##### A27. Lambdas may capture a pack

The enclosing function is processed per type list, so a lambda's generated
receiver gets one member per captured item with every type known. Inside the
lambda the capture is again a pack. A stance applies per item, as on
expansion. The lambda's own prototype is unaffected.

```zax
deferSend$ final : (Items ... : type)(callback : ()() bound unique)(items ... : Items) = {
  return [[ items ]] ()() {        // a pack capture; each item copied in
    sendAll(from items as last)
  }
}
```

##### A28. Variadic lambdas in `final` storage

A lambda may declare a pack parameter. As with declared functions (A1), it
must be held in `final` storage, where its call operation is processed per
count. A `varying` slot or callback parameter has one fixed prototype, which
selects one processing (A23).

```zax
total := [[]] (result : Integer)(values ... : Integer) {   // final storage
  // ...
}
total(1, 2, 3)

handler : (result : Integer)(a : Integer, b : Integer) varying = total
// selects total's two-item processing
```

A same-typed variadic lambda needs no `$`, following A3.

Dry-run check: A28 and A29 assume that an inferred declaration such as
`total := [[]] ...` gives `final` storage. Current owners use `name := [[...]]`
throughout, but the default is not confirmed. If it is `varying`, these
examples need an explicit `final` and its correct spelling.

##### A29. Generic lambdas: `$` on the binding name, in `final` storage

A mixed-type variadic lambda needs a variadic `$` parameter, which makes it a
generic lambda. As for every generic, `$` goes on the name: the lambda's
binding name carries `$`, which says the lambda's first parenthesized list
holds compile-time parameters. Callers use the ordinary generic call forms.

```zax
show$ := [[ prefix ]] (Items ... : type)()(items ... : Items) {
  each item : in items {
    write(prefix, item)
  }
}

show("a", 2, 3.0)                  // worked out: show$(String, Integer, <real type>)
show$(String, Integer)("a", 2)     // stated
```

Rationale, for generics and lambdas teaching: `$` marks a name because a
generic is a family of instantiations reached through its name, as with
`add$`, `MyPair$`, and `'*'$`. An unnamed callable is used once, so it has one
processing and needs no family.

Consequences:

- A generic lambda uses the same mechanism as A28: one generated receiver and
  a call operation processed per distinct set of compile-time inputs.
  Constraints and `requires` work unchanged. Specialization does not apply,
  because a lambda is one expression with no overloads to rank. This resolves
  the generic part of the lambdas owner's current deferral.
- A `$` binding must be `final` storage; a `varying` `$` binding is an error.
- An inline lambda (passed, returned, or invoked immediately) cannot have a `$`
  list. It meets a fixed prototype or a single use, so it takes concrete types:
  from its destination through prototype inheritance, written directly, or
  from an enclosing generic's `$` parameters.

```zax
([[]] ()(a : Integer, b : Integer, c : Integer) { })(1, 2, 3)   // single use: concrete

relay$ final : (Items ... : type)()(items ... : Items) = {
  ([[]] ()(values ... : Items) { })(from items as copy)   // concrete per processing of relay$
}
```

#### Routing refinements from dry run 2

##### A30. `[<label=incidental>]`: a label that carries no routing meaning

The language cannot tell whether a label carries meaning. `result`, `output`,
and similar conventional labels usually do not. `[<label=incidental>]`,
written in the per-name slot that `[<constant>]` uses, says the label is
incidental. It is the `label` directive with the value `incidental`; naming
the label says the property belongs to the label, not to the value. It may
be used on results and on parameters. The directive catalog currently defines
a slot only after a parameter's name, so promotion adds the slot after a
result's name.

- An incidental **result** label never matches a parameter implicitly and
  never triggers the single-result intent error.
- An incidental **parameter** label is never claimed by a producer's own
  label. A single result whose label names an incidental parameter raises no
  intent error, because no two meaningful labels disagree. A `>-` member whose
  name matches an incidental parameter does not bind it either; the
  parameter must be supplied another way.

  ```zax
  plot2 final : ()(x : Integer, y [<label=incidental>] : Integer) = { }
  plot2(>- point)              // error: y is incidental, so the member y does not bind it
  plot2(>- point, point.y)     // fine: y supplied by position
  ```
- Explicit use still works in both cases: `name: value`, routing groups, and
  reshape labels. A reshape label is written intent, so it binds even an
  incidental parameter.

The name says what is true of the label; positional routing is the
consequence.

```zax
sumOf final : (result [<label=incidental>] : Integer)(values ... : Integer) = { }
record final : ()(total : Integer, result : Integer) = { }

record(sumOf(1, 2), myResult)            // fine only because sumOf's result is incidental
record(myTotal, result: = sumOf(1, 2))   // explicit selection of the incidental label still works

getCount final : (count : Integer)() = { }
getTotal final : (total : Integer)() = { }
store2 final : ()(count : Integer, total [<label=incidental>] : Integer) = { }

store2(getTotal(), 5)                               // no intent error: total is incidental on store2
store2(getCount() reshape count: total:, myCount)   // the reshape label binds the incidental total
```

Because of this directive, operators need no special rule (A31). The
single-result intent error fires only when a label names a different
parameter, so most existing `(result : ...)` declarations need no marking.
Only the declaration writes the directive; callers write nothing extra.

The marking belongs to the **visible prototype**, like the labels themselves.
A compatible implementation installed in a `varying` slot may have different
labels or markings, because compatibility is positional over ordered slots.
Callers always route by the prototype they call through, so installing
another implementation never changes routing at a call site:

```zax
MyHandler :: type {
  onResize : ()(width : Integer, height : Integer)   // the slot's visible prototype
}

resizeImpl final : ()(w : Integer, h : Integer) = { }

handler : MyHandler
handler.onResize = resizeImpl          // compatible: same types and order; labels differ

measure final : (width : Integer, height : Integer)(shape : Shape) = { }
handler.onResize(measure(myShape))     // routes by the slot's labels, width and height
```

Adding or removing `[<label=incidental>]` on a visible prototype is a
relabel-style change, not a type change. Like any label change, it can change
routing at call sites written against that prototype.

##### A31. Operators route like any callable

An operator's results follow the same rules as a function's. An operator in
argument position is one expression value when it has one result, so the
single-result intent error applies as for a call. `[<label=incidental>]` on the
operator's result label removes it. A multi-result operator in argument
position needs an explicit routing group, as today. Only a bare invocation can
produce a forwarded result sequence; an operator expression is one expression
node.

Built-in operators declare no result labels. Their results are treated as
incidental, stated once in the operator catalog, so ordinary arithmetic never
triggers the intent error:

```zax
record final : ()(total : Integer, result : Integer) = { }
record(a + b, myResult)   // fine: built-in Integer + has an incidental result
```

Programmer-declared operators follow their own declarations.

```zax
MyAmount :: type {
  operator binary '+' final : (result [<label=incidental>] : MyAmount)(other : MyAmount) readonly = { }
}

store final : ()(total : MyAmount, result : MyAmount) = { }
store(left + right, myTotal)   // fine: result is incidental, so the sum routes by position
```

##### A32. Bare `:` in a routing group skips implicitly consumed parameters

The destination cursor of a routing group is the call's cursor, so it skips
parameters consumed by implicit matches, including matches from a sequence
written later in the same call:

```zax
consume final : ()(a : Integer, b : Integer, c : String) = { }
produceA final : (a : Integer, note # : String)() = { }
g final : (x : Integer)() = { }

consume(x: : = g(), produceA())
// produceA claims a by label; g.x goes to b; produceA's note falls by position to c
// (a discardable result is still mapped while a destination remains)
```

Teaching must state this action at a distance: a later argument can change
where a group's bare `:` lands.

##### A33. `>-` decomposition binds by name without moving the cursor

Decomposed members bind parameters by name, like implicit matches, and do not
move the cursor. Teaching contrasts members with results: members that match
nothing are ignored, while results that match nothing fall to positions and
must be discardable if left over.

```zax
Point :: type {
  x : Integer
  y : Integer
  description : String
}

plot final : ()(x : Integer, y : Integer, label : String = "point", detail : String = "") = { }

point : Point
plot(>- point, "caption")
// x and y by name; "caption" → label; description is ignored;
// detail keeps its default

plot(>- point, "caption", detail: point.description)
// the same, plus description supplied to detail explicitly
```

An unmatched member never falls to a position, even when a parameter of a
compatible type remains. Without the default on `detail`, the first call is
an error, because nothing fills `detail`.

##### A34. The prototype written in partial application maps by position

In partial application, a written prototype renames the target's remaining
inputs in order. That is prototype adaptation, like compatible visible
prototypes. It is not result routing, so the routing rule does not apply.

Orientation: in partial application, a capture entry such as
`number: number:` captures the local `number` now and supplies it as the
target's `number` input on every call. The new callable's implementation is
the target itself, so no body is written.

```zax
printTriple final : ()(number : Integer, label : String, text : String = "") unbound = { }

number := 42
f := [[ number: number: ]] ()(text : String) >|> printTriple
f("hello")   // printTriple(42, "hello", ""): f's text is printTriple's next remaining input, label
```

To keep the target's own names, omit the written prototype and inherit the
remaining prototype, which the lambdas owner already allows.

### Rejected alternatives

| Alternative | Why rejected |
| --- | --- |
| Single-type tail received as a slice over a caller-built temporary array | Made `...` an optimization scenario dependent on the argument mix; hid a temporary, copies on forwarding, and possible allocation (A1, A2) |
| Anonymous `...` and `$...` | Unlabelable, needs special spellings, collides with `{ ... }` placeholders (A4) |
| Bare pack name expanding automatically | Hides the positional jump and makes `f(g(items))` ambiguous between passing the pack and mapping (A7) |
| C++-style trailing `...` patterns | The same name means the pack or one item depending on a later `...`, so `items count` and `items[0]` change meaning (A9, A11) |
| `each item : in items -> expression` | Adds a lambda-like surface without being a lambda |
| Named stage `item -> expression`, and a short-form lambda stage | Superseded by `from each item : in pack |>` (A11); a lambda would also need capture rules |
| Binding-less `each items |> f()` | Did not read well and gave the item no name (A11) |
| `each item : in items |> f()` without `from` | Reads as a loop rather than a contributed list (A11) |
| `$` after a lambda's capture list, `[[ captures ]]$` | Would allow inline generic lambdas, which have no use the existing forms do not cover, and would leave `show$(...)` callers using a name never declared with `$` (A29) |
| The item's binding name acting as a label | Silent routing by accidental name match; explicit placement by writing the name was preferred (A11) |
| Label-moves-cursor placement for the carried value only | Arbitrary; replaced by the single routing rule (A15) |
| A `|>` stage rule filling remaining parameters, carried values first | Diverged from ordinary calls; replaced when nested forwarding adopted labels-first routing, making a chain the same call (A15) |
| Positional-only nested forwarding | Made `f(g())` and `g() |> f()` route differently (A15) |
| A single result routing by its label | Would let a lone call result leave its written position silently; replaced by positional binding plus an intent error (A15) |
| Variable names acting as labels | A local name is not a contract; renaming it would silently reroute (A15) |
| Several bare new names from several results, `number, text := produce()` | Mixes introducing names with routing; `using` already forbids positional name introduction from several results (A15) |
| Collisions as hard errors regardless of other candidates | Contradicts the existing viability rule; a colliding candidate is simply not viable (A15) |
| A special rule exempting operator results from the intent error | Operators are callables; `[<label=incidental>]` handles conventional labels uniformly (A30, A31) |
| Bare `[<incidental>]` | Placed between name and type, it reads as describing the value rather than the label (A30) |
| `[<positional>]`, `[<match=explicit>]`, `[<anonymous>]` as the directive name | The first two name the side effect rather than the property of the name; the name still exists, so it is not anonymous (A30) |
| Reshape stage entries consuming both endpoints before label matching | Gave one reshape two meanings; a stage is now a relabel of its left callable (A16) |
| A written partial-application prototype routing by label | It is a rename list over the remaining inputs, like compatible visible prototypes (A34) |
| Reshape as a separate `|>` stage | Postfix `reshape` on the stage invocation already exists and always applies (A16) |
| A placeholder token for the carried value | `...`, `_`, `$`, `:`, and `#` already have meanings |
| A pack item carrying its pack's name as a label | Plural name for one item, pack-only, duplicates under `from` (A16) |
| Running every item and collecting failures | One outcome per call; no erased aggregate error (A14) |
| A self-recursion warning | Intentional recursion is common and undetectable (A17) |

### Outstanding items

None. Resolved items:

- O1, `[N]` in structural reshapes: disallowed (A16).
- O2, work-file status metadata: fixed.
- O3, a mapped pack in an array expression: allowed (A11).

### Deferrals

Each deferral names its reactivating pressure, any constraint it places on
this work, and its candidate destination.

| Consequence | Reactivates when | Constraint on 037 | Candidate destination |
| --- | --- | --- | --- |
| Tuple-like generic types with a member per pack element | A library needs a heterogeneous value | Packs never silently become that value | Raw generics input |
| Growing a pack: several values per item at the end of a mapping (see below) | A real flattening use case | Final stage yields exactly one value per item | Lambdas `|>` boundaries |
| Short-form lambdas | A concrete need for a compact stored lambda; per-item work is now covered by `from each` (A11) | None; a future lambda can become a chain stage | Lambdas owner boundaries |
| Zipping two packs | A zip-like API | At most one variadic input; one pack per mapping | Raw generics input |
| Several variadic `$` parameters in one list | A zip-like API needing a delimiting `$(...)` form | Same as above | Raw generics input |
| Mixfix `call N` with a pack | Mixfix needs a variable call component | Variadic call components stay disallowed | Mixfix owner boundaries |
| Variadic operator phrases and holes | A phrase needs variable operands | None | Operator-phrases boundaries |
| Folding across a mixed pack | Accumulating mixed numeric types | None | Raw generics input |
| Identifying which item failed | A diagnostics or payload need | Payload design only | Exceptional-flow boundaries |
| C-style untyped varargs for foreign functions | FFI work | Untyped varargs never enter ordinary Zax calls | Future FFI owner (no raw file yet) |
| Generating declarations by walking a type pack | Source-generation work | Type-pack `each` produces runtime bodies only | Source-generation input |

Promotion must create or update each destination; none of these may survive
only in this file.

#### Flattening detail the destination must preserve

`func(from each item : in items |> produceTwo())` would be predictable (item by item:
`a0, b0, a1, b1`) and its count still known while compiling. It stays an error
because:

- **Silent shifts.** Adding a result to a producer grows every pack built from
  it and shifts later positions. Because expansion can fill ordinary
  parameters, the shift can still type-check. Under the one-value rule the
  same change fails loudly.

  ```zax
  produceTwo final : (low : Integer, high : Integer)(item : Integer) = { }
  setRange final : ()(min0 : Integer, max0 : Integer, min1 : Integer, max1 : Integer) = { }

  setRange(from each value : in pair |> produceTwo())
  // If produceTwo later adds `mid`, positions shift; against a pack or
  // defaulted trailing parameters this could route mid into max0 silently.
  ```

- **Discardable results become ambiguous.** Nothing consumes flattened results
  by name, so it is unclear whether a discardable result joins the pack or is
  dropped, and adding a discardable result would change the pack's size. This
  consequence decided the deferral.
- **Labels are lost and results interleave.** `low0, high0, low1, high1` gives
  a callee no way to take all lows then all highs without manual reordering.
- **The count depends on result shape per type.** Mixed packs may select
  overloads with different result counts, so the size is not visible at the
  call site.

In favor: flattening treats several results as several values, consistent
with results never becoming a tuple. Relaxing the error later is
source-compatible, so deferral costs nothing.

Workaround, maintainer-identified: a producer returns one structured value, so
labels survive as member names and the pack keeps its size.

```zax
MyRange :: type {
  low : Integer
  high : Integer
}

produceRange final : (range : MyRange)(item : Integer) = { }

setRanges(from each item : in items |> produceRange())   // one MyRange per item
```

If flattening is reopened, an explicit opt-in that states the per-item result
shape at the call site would address the silent-shift hazard.

### Likely documentation-owner boundaries

A candidate for the dry run, not a decision:

- A cohesive owner, plausibly `language/variadics.md`, so that readers do not
  assemble one feature from five owners.
- `language/function-invocation.md`: the cursor at a pack, labels into a pack,
  label-only later parameters, `from` in argument lists, selection rules
  (A17), function values (A23), result packs (A25), and the boundaries list.
- `language/declarations-and-bindings.md`: local pack declarations (A26).
- `language/generics.md`: variadic `$` parameters, the count as a
  compile-time input, pack identity, "every element" terms, `requires` without
  a `$` list, and the new diagnostic.
- `language/iteration.md`: packs as a direct `in` source family, and the
  `from each binding : in pack |>` expression form.
- `language/function-invocation.md` also owns the single routing rule (A15):
  labels first, then ordinary call rules, at every forwarding site; collisions;
  the single-result intent error; variable names never routing.
- `language/lambdas-and-callable-composition.md`: chains as the same call under
  A15, postfix reshape in chains (A16), the one-value final-stage rule, pack capture (A27),
  variadic lambdas and generic lambdas with `$` on the binding name in `final`
  storage (A28, A29), and boundary updates, including removing the
  generic-lambda deferral.
- `language/generics.md` also gains the generic-lambda form under its two
  forms of generic, with the "`$` marks a name" rationale.
- `language/structural-shapes-and-compatibility.md`: `[N]` source paths and
  their callable-surface-only scope.
- `language/except.md`: a mapped pack under nested-call handling (A14).
- `language/arrays-and-slices.md`: `from` packs and `from each ... |>` in
  array expressions; runtime lists are arrays or slices.
- `language/transfer-stances.md`: a stance on an expansion.
- `language/operator-catalog.md`, `language/terms.md`: `count` on packs; the
  term "pack".
- Legacy retirement: `variadic.md`, the `basics.md` variadic operator list,
  the `index.md` route, and the raw feature-catalog entry.

### Legacy and raw evidence dispositions

| Item | Disposition |
| --- | --- |
| `each value: in ...` | Promote as `each` over a named pack (A8) |
| Per-element overload selection in the body | Promote; per-item checking (A8) |
| Infinite-recursion caveat | Replace with an ordinary `requires` example; no warning (A17) |
| Required arguments before the group | Promote and extend with label-only later parameters (A5) |
| `count of ...` | Replace with post-unary `count` (A9) |
| Forwarding `sendAll(...)` | Replace with `from` expansion and per-item stance (A6, A13) |
| Automatic `last` on forwarding | Already superseded; keep superseded |
| Unresolved forwarding question list | Answered by A7, A11, A13, A25, and A27 |
| `$...` types | Supersede with a named variadic `$` type parameter (A3, A4) |
| `[<compile>] ...` (raw catalog) | Promote as `[<constant>]` on a variadic parameter (A21) |
| `basics.md` `count of`, `...`, `$...` | Retire at promotion |
| `# : [] = [{ 1, 2, 3}]` argument in the legacy `list(...)` example | Discard; not recoverable syntax |

## Dispositions and promotion dry run

### Dry run 1 (2026-11-10): FAIL

The result is **FAIL** on one alignment question (F1). Everything else has an
owner, a destination, and a teaching plan. Two corrections and one capture
come from checking current owners (C1 to C3). After F1 is decided, the dry run
can be re-checked against this record without repeating the reading.

#### F1. A `|>` stage and a nested call route labeled results differently

Current owners already contain both rules:

- `language/function-invocation.md`, *Forwarding into another call*: "Simple
  forwarding is positional. Result labels do not automatically remap
  parameter names."
- `language/lambdas-and-callable-composition.md`, *Chain calls immediately*:
  `|>` uses "the same explicit routing principles as composition", and
  composition matches exact equal labels before positions. A15 makes that
  concrete.

The consequence, not discussed before:

```zax
produce final : (text : String, number : Integer)() = { }
consume final : ()(number : Integer, text : String) = { }

consume(produce())        // positional: text→number, a type error
produce() |> consume()    // labels match first: number→number, text→text; works
```

The two spellings look interchangeable but route differently whenever a
result label matches a parameter in another position. This is wider than
variadics, and promotion would teach it in two owners, so it needs
maintainer confirmation.

Options:

- **(a) Accept and teach it (agent recommendation).** A chain routes like
  composition: labels first, then position. A nested call routes by position
  only. This keeps `|>` and `>|>` identical, which is already aligned, and
  changes neither current owner's rule. Teaching states it in both places
  with this example.
- (b) Make `|>` positional only, with labels used only through an explicit
  reshape. `f(g())` and `g() |> f()` would then agree, but `|>` would diverge
  from `>|>`, contradicting A15's aligned basis.
- (c) Make nested forwarding label-aware. This changes a current
  function-invocation rule and its source-stability notes, and is outside
  this item's pressure.

**Resolution (2026-11-10):** the maintainer chose a generalization of (c): one
routing rule wherever results meet inputs, so `f(g(), rest)` and
`g() |> f(rest)` are the same call. Recorded as the revised A15 and A16. F1 is
closed; see the re-check below.

#### Corrections and captures found against current owners

- **C1. Storage default (correction, no new decision).**
  `language/declarations-and-bindings.md`, *`final` and `varying`*, says an
  omitted stance is varying by default (`foo : Foo // varying by default`),
  and `x := 0` is the compact form of `x : = 0`. The A28 and A29 examples
  therefore declare varying storage. Promotion writes the declaration-side
  qualifier with the existing grammar:

  ```zax
  total final := [[]] (result : Integer)(values ... : Integer) { }
  show$ final := [[ prefix ]] (Items ... : type)()(items ... : Items) { }
  ```

  A `$` binding without `final` is an error under A29. A non-`$` variadic
  lambda without `final` is likewise an error under A28.

- **C2. Returning a parameter as its final use (capture).**
  `language/transfer-stances.md` defines terminal opportunity for arguments
  and for result slots used as sources, considering each result slot
  separately. It does not say whether `return param` as a parameter's final
  use is an intent-requiring terminal opportunity. This gap predates
  variadics. Promotion keeps the explicit `return from items as last` in
  examples, which is valid under any reading, and adds the open question to
  the transfer-stances boundaries:

  ```zax
  relay final : (result : Message)(input : Message) = {
    return input   // final use of input: intent required, as for arguments?
  }
  ```

- **C3. Deferral destinations confirmed.** FFI varargs go to
  `project/raw/interop.md`. Declaration generation from a type pack goes to
  `project/raw/maintainer-notes/compiler-directives.md`, the indexed
  source-generation input.

#### Ownership map

| Concern | Owner | Teaching weight |
| --- | --- | --- |
| The pack model: declaring, supplying, receiving, counting, indexing, expanding, mapping, result and local packs, variadic lambdas, costs, diagnostics | **New** `language/variadics.md` | Primary teaching home |
| Cursor at a pack, label-only later parameters, labels into a pack, `from` in argument lists | `function-invocation.md`, positional and named inputs | Short local rule plus link |
| Empty pack as omitted default; fixed beats absorbed; no-`$` pack is non-generic | `function-invocation.md`, candidate selection | Rules live here; variadics teaches by example |
| Result packs in result slots and routing | `function-invocation.md`, result slots and forwarding | Short local rule plus link |
| Function values selecting one processing, with adaptation glue | `function-invocation.md`, compatible visible prototypes | One paragraph |
| Variadic `$` parameters, count as compile-time input, pack identity, every-element terms, `requires` without `$`, diagnostics, generic lambdas with `$` on the binding name | `generics.md` | Rules live here; variadics links |
| Packs as a direct `in` source family | `iteration.md`, direct `in` traversal | New subsection beside stored members |
| `from each binding : in pack |> stages` | `variadics.md` teaches; `iteration.md` names it as the expression use of an `each` header | Short |
| The single routing rule (A15): labels first, then ordinary call rules; collisions; single-result intent error; variable names never route; leftovers | `function-invocation.md`, forwarding results | Rule lives here; applies to all code |
| Chains as the same call, postfix reshape in chains, one-value final stage | `lambdas-and-callable-composition.md`, chain calls immediately | Rewritten section; links the routing rule |
| Pack capture; variadic and generic lambdas in `final` storage | `lambdas-and-callable-composition.md` | New subsections |
| `[N]` source paths and their callable-surface scope | `structural-shapes-and-compatibility.md`, reusable mapping with `reshape` | New subsection |
| A mapped pack under nested-call handling | `except.md`, pipelines | One example |
| `from pack` and `from each` in array expressions; runtime lists are arrays | `arrays-and-slices.md`, one or several elements | Short |
| Stance on an expansion, per item | `transfer-stances.md` | Short; plus C2 boundary |
| Local pack declarations | `declarations-and-bindings.md`, inferred declarations or multiple results | Short |
| The `...` declaration token and spacing; `from each` header presentation | `source-structure.md` | Reference entries |
| Pack `count` | `operator-catalog.md`, the `count` row | One row extension |
| "Pack", "variadic parameter"; count as compile-time input | `terms.md` | Two entries plus one edit |

#### Proposed structure

- Create `language/variadics.md`. Readers would otherwise assemble one
  feature from about ten owners, which fails the cohesive-teaching test even
  though each rule has an owner. It teaches the model end to end and hands
  exact rules to their owners by link without restating them.
- Retire root `variadic.md`. Every useful passage has a disposition in the
  table above.
- `index.md`: add Variadics to the current-owner list after Generics; remove
  the legacy route.
- No directory changes. Raw inputs stay indexed in `project/raw/README.md`;
  only descriptions of touched raw files change.
- Ordinary reading reaches the owner from `index.md` and from links in
  invocation, generics, iteration, and lambdas. Raw and deferred material is
  reachable only from project routes.

#### Teaching approach

The working record is dense, organized for review, and full of finding
numbers, rejected alternatives, and discovery order. Promotion rebuilds the
teaching rather than transplanting it:

- **One home teaches; other owners hand off.** `variadics.md` carries the
  mental model and most examples. In an existing owner, a variadic addition
  states only what changes for a reader of that page, with one local example
  and a link. Existing sections are revised in their own voice and order, not
  appended with record text.
- **Self-contained examples.** Every promoted example declares what it uses,
  places pack names inside the function that has them, marks known errors
  inline, and uses `My...` names where they distinguish programmer
  declarations. Working-record fragments are rebuilt, not copied.
- **No process vocabulary.** No finding numbers, "aligned", maintainer
  rationale, or rejected alternatives. A counterexample stays only where a
  reader would plausibly write it: `send(items)`, a runtime slice into a
  pack, `[0]:` in a structural reshape, the single-result intent error, and
  `report(count:, total:)` read as a by-name shorthand.
- **Words before terms.** "A group of arguments the function receives
  separately" before "pack"; "the function is compiled for each number of
  arguments" before "processed per count".
- **Cold-reader review after promotion,** by a separate agent that has not
  read this record, as validation requires.

Outline for `language/variadics.md`:

1. Accept any number of arguments: `sum` and `print`, compiled per count or
   type list, with cost stated up front.
2. Declaring: `name ... : Type`; the three kinds; when `$` is needed; `final`;
   one per prototype; later parameters by label.
3. Calling: defaults and labels around a pack; `from` with packs and
   fixed-length arrays; runtime lists belong in array parameters.
4. Inside the body: a pack is not a value; `each`, `count`, and
   compile-time indexing.
5. Passing items on: `from items`, stance per item, the terminal-use intent
   error.
6. Transforming each item: `from each item : in items |> ...`; placing the
   item; arrays; order; one value per item; failure stops the rest.
7. Returning and holding packs: result packs and local packs.
8. Checking packs: every-element constraints, counts, empty packs.
9. Choosing between overloads: the three selection rules by example.
10. Compile-time values in packs and value packs in `$` lists.
11. Lambdas and function values: capture, variadic and generic lambdas in
    `final` storage, selecting one processing.
12. Costs, diagnostics, source stability, and boundaries.

#### Section teaching plans

| Owner section | Why the reader is here | What they already know | What changes for them | What they might wrongly try |
| --- | --- | --- | --- | --- |
| `variadics.md` opening | Wants a function taking any number of arguments | Ordinary calls; maybe C varargs or C++ packs | Every variadic is compiled per count or type list; a same-typed list is not a runtime array | Expecting a runtime list, or a `$` on `sum` |
| `variadics.md` inside the body | Needs to use the arguments | `each` over arrays and members | A pack is several bindings; `count` and `[N]` are compile-time | `saved := items`; `items[i]` with a runtime `i` |
| `variadics.md` transforming each item | Wants to call something per item and pass the results on | `each` statements; `|>` chains | `from each item : in items |> ...` builds a new pack; writing `item` places it | A plain `each` statement to build arguments; expecting two results per item to flatten |
| `function-invocation.md` positional inputs | Reading cursor rules | The cursor and named inputs | The cursor stays on a pack; later parameters need labels; a label can start a pack | Passing a later parameter positionally |
| `function-invocation.md` candidate selection | Overloads with a variadic version | Partial order, fewer defaults | Three tie rules with packs | Expecting `max(1, 2)` to be ambiguous, or the `$` version to win |
| `function-invocation.md` forwarding | `f(g())` with several results | Positional forwarding | Labels match first, then ordinary call rules; `f(g(), rest)` is `g() |> f(rest)`; a lone result stays positional with an intent error when its label points elsewhere; variable names never route | Expecting variable names to route; writing `report(count:, total:)` as a by-name shorthand |
| `generics.md` `$` list and constraints | Writing a mixed-type variadic or constraining one | `$` lists, short-form constraints, terms | Variadic `$` parameters; every-element constraints; empty packs pass | Writing `isScalar` and expecting it to reject an empty call |
| `generics.md` generic lambdas | Wants a reusable generic lambda | `$` goes on the name | `$` goes on the binding name; `final` is required; inline lambdas take concrete types | `$` after the capture list; a `varying` `$` binding |
| `iteration.md` direct `in` | Walking something | Arrays, enums, members | Packs are a fourth source family under the member rules | Expecting element places of one type |
| `lambdas-and-callable-composition.md` chaining | Using `|>` with arguments | Empty-`()` chain examples only | A chain is the same call as the nested form; reshape applies postfix to a stage's invocation; one value per item at the end of a mapping | Expecting a chain to route differently from the nested call; writing a reshape as its own stage |
| `lambdas-and-callable-composition.md` lambdas | Capturing or declaring a variadic lambda | Capture lists, storage kinds | Pack capture; `final` requirement; `varying` slots select one processing | Declaring `total := ...` without `final` |
| `structural-shapes-and-compatibility.md` reshape | Remapping results | Name-based entries | `[N]` for unlabeled results; never for structural members or exceptional outcomes | `[0]: x:` in a structural reshape |
| `except.md` pipelines | A failing stage | Handlers cover nested calls | A mapped pack behaves as if each call were written out | Expecting the other items to keep running |
| `arrays-and-slices.md` contributions | Building an array | `[ from row ]` | `from items` and `from each` contribute; same-type rule applies | Expecting mixed results to make an array |
| `transfer-stances.md` | Forwarding packs | One stance on one value | A stance on an expansion applies per item | Expecting silent perfect forwarding |
| `declarations-and-bindings.md` | Holding results of unknown count | `:=` and multiple results | `outs ... :=` declares several locals | Treating `outs` as one value |
| `source-structure.md`, `operator-catalog.md`, `terms.md` | Reference lookups | Token and phrase catalogs | `...` token and spacing; pack `count`; two glossary entries | Writing `values...` without spacing (formatter canonicalizes) |

#### Exact promotion change set

New:

- `language/variadics.md`, following the outline above.

Modified current owners:

- `language/function-invocation.md`: positional-cursor and label rules at a
  pack; candidate-selection rules; result packs in result slots and
  forwarding; the single routing rule replacing "simple forwarding is
  positional" in forwarding into another call, from a return, and into
  construction, with collisions, the single-result intent error, and
  variable names never routing; source-stability entries for label renames
  and new-result collisions; function
  values selecting one processing; the default-timing note no longer listing
  variadics; boundaries list without "variadics"; Does Not Own routes to
  variadics.
- `language/generics.md`: variadic `$` parameters and the count as a
  compile-time input in "processed once per input"; pack identity under
  comparing arguments; every-element short form under constraints; `requires`
  on a declaration with no `$` list; generic lambdas with `$` on the binding
  name and the "`$` marks a name" rationale; two diagnostics; boundaries.
- `language/iteration.md`: packs as a direct source family; the `from each`
  expression use of a header; source-family list in the opening.
- `language/lambdas-and-callable-composition.md`: the chaining section
  rewritten so a chain is the same call under the routing rule, with postfix
  reshape on stage invocations and the one-value final stage; pack capture;
  variadic and generic lambdas in `final`
  storage; boundaries without the generic-lambda line, with short-form
  lambdas and flattening named; Does Not Own updated.
- `language/structural-shapes-and-compatibility.md`: `[N]` source paths in
  `>|>` stages and postfix invocation reshape; structural and exceptional
  exclusions.
- `language/except.md`: the mapped-pack example under pipelines; "which item
  failed" named under boundaries.
- `language/arrays-and-slices.md`: `from` packs and `from each` in array
  expressions; runtime lists versus packs.
- `language/transfer-stances.md`: stance on an expansion; the C2 boundary.
- `language/declarations-and-bindings.md`: local pack declarations.
- `language/source-structure.md`: the `...` declaration token and spacing;
  `from each` header presentation.
- `language/operator-catalog.md`: the `count` row covers packs as a
  compile-time value.
- `language/terms.md`: new "Pack" and "Variadic parameter" entries; the
  "Compile-time input" entry includes a variadic count.
- `language/mixfix-operators.md`: "variadics" in the future-integration list
  becomes "variadic call components".

Routers and legacy:

- `index.md`: add Variadics; remove the `variadic.md` route.
- `variadic.md`: delete.
- `basics.md`: remove the remaining variadic operator list (`count of`, `...`,
  `$...`) and its framing; remove the `$` legacy line, now owned by generics.

Raw inputs:

- `project/raw/feature-catalog.md`: the Variadic functions row routes to
  `language/variadics.md`; the Polymorphic functions row stops listing
  variadics as future.
- `project/raw/type-parameters-and-generics.md`: a new section with examples
  for tuple-like generic types, zipping two packs, several variadic `$`
  parameters, and folding across mixed packs.
- `project/raw/function-composition-and-chaining.md`: flattening detail with
  examples and the struct workaround; short-form lambda pressure.
- `project/raw/interop.md`: C-style untyped varargs pressure.
- `project/raw/maintainer-notes/compiler-directives.md`: generating
  declarations from a type pack.
- `project/raw/README.md`: description updates for the touched raw files.

Unchanged but checked: `language/operator-phrases.md` (variadic holes remain
future and are still described correctly), `language/principles.md`, and
`README.md`.

Not changed by promotion: `project/README.md`, which changes only at closure.

#### Archive effects

None during promotion. At closure this file moves to
`project/archive/work/037-variadic-arguments.md`. No current owner may cite
it.

### Dry run 1 re-check (2026-11-10): PASS

F1 is resolved by the revised A15 and A16. The ownership map, teaching plans,
and change set above were updated in place for that resolution. The re-check
found:

- **Every aligned finding has one owner.** The routing rule's primary owner is
  `function-invocation.md`, forwarding results, because it governs nested
  calls, returns, and construction packets, not only chains. The lambdas
  owner teaches chains as the same call and links it.
- **No new alignment question.** The revised rule replaces a current
  function-invocation sentence and adds source-stability entries. The
  maintainer explicitly chose that change, so it is integration, not an open
  design question.
- **Examples still to rebuild at promotion:**
  - A28 and A29 with declaration-side `final` (C1);
  - `return from items as last` kept explicit (C2);
  - every working-record fragment rewritten as self-contained source.
- **Deferrals.** The new "unlabel" deferral has a live destination in the
  structural-shapes boundaries. Every other destination is unchanged.
- **Postfix `reshape` precedence against `|>`.** This is the remaining
  factual confirmation, made against the operator catalog during promotion.
  If postfix `reshape` did not bind more tightly, promotion stops and
  reports rather than inventing precedence.

The promotion change set is the one listed above. This result does not
authorize promotion.

**Superseded by dry run 2.** The claim of "no new alignment question" did not
hold once the existing matching teaching was read.

### Dry run 2 (2026-11-10): FAIL

Scope: the sections of current owners that already teach matching, cursors,
result routing, composition, reshape, and multiple results, read against A15
and A16. A delegated reader without the design discussion performed the
targeted reading. The agent verified the load-bearing claims against the
source: the undefined `a, b := g()` form, the `using` precedent, the existing
`: = producer()` form, and the viability rule.

#### Open alignment questions

- **R1. Introducing names from several results.** No owner defines
  `a, b := g()`. Every current destination form either selects results by
  label (`number:, text: existing = produce()`) or uses typed captures.
  `language/using.md` forbids introducing names positionally from several
  results, "so result reordering or API evolution" cannot silently change
  what a name denotes. A15 and A25 rely on the undefined form, and A15
  accepted the very risk that `using` rules out.
- **R2. Collisions: an error, or a candidate that is not viable?** Viability
  already says a candidate is viable only when inputs map "without
  duplication", so a collision would silently select another overload. It
  must also be decided whether a discardable colliding result still
  collides.
- **R3. `consume(: = produce())`.** This existing explicit mapping sets up
  cursors. Is it positional, and so the deferred "unlabel" form? Does
  explicit positional intent, `report(: getCount(), myTotal)`, suppress the
  single-result intent error? Do parentheses?
- **R4. Operators in argument position.** Should an operator's result label
  raise the single-result intent error? `result` is the conventional label
  in about 100 declarations. Is a multi-result operator a forwarded
  sequence?
- **R5. Bare `:` inside a routing group.** Does it skip a parameter claimed
  implicitly by a sequence written later in the same call?
- **R6. `>-` decomposition.** Do decomposed members move the cursor? Teaching
  must contrast them with results: unmatched members are ignored, while
  unmatched results fall to positions.
- **R7. Reshape as a `>|>` stage versus postfix reshape.** Explicit entries
  consume both endpoints in the stage form, while the postfix form relabels
  and rejects duplicates. Should `f >|> R >|> g` be defined as
  `(f reshape R) >|> g`?
- **R8. The prototype written in partial application.** Does it map onto the
  target positionally, as prototype adaptation, or by labels under the
  composition rule?

#### Corrections to findings

- A15 example `consume(produce(), enabled: true, 5)` never fills `flag`, so
  it is an error as written.
- `measure` declares no discardable result, so the leftover examples in A15
  and A16 (`area(measure(...))`, `SecondAsSize` with `scale`) are invalid as
  declared. Declare `#` where intended.
- "Implicit matches first" must say mapping priority, not evaluation order.
- The "only declared labels disagree" principle overstates the rule.
  Collisions are duplicate supply, not disagreement.
- State that explicitly labeled parameters are not skipped by the cursor:
  `f(b: 1, a: 2, x)` remains a duplicate.
- State the asymmetry: an implicit match silently displaces a positional
  input (`f(5, g())`) but is an error against a written label.
- List the single-result fixes once. "Carried values" in A16 needs a
  definition or replacement.
- Remove process vocabulary such as "no prior behavior to preserve" and
  "earlier rejection" from text intended for promotion.
- The single-result intent error may fire often with common labels such as
  `value`. That needs explicit acceptance or a mitigation.

#### Change-set gaps

Sections missing from the change set:

- `function-invocation.md`: positional cursor (skip consumed parameters);
  expression mode ("consecutive input parameters" becomes false; R3);
  operator result shapes (R4); the routing-group opening; two cursors (R5);
  cursor exhaustion (an example that cannot be evaluated); forwarding into
  construction (contrast with packing members); viability (R2); compatible
  visible prototypes (relabeling now changes routing); diagnostics; the
  formatting rule on parentheses; boundaries (chaining, and the "anonymous/
  private parameter and result labels" line now pointing to
  `[<label=incidental>]` and stating what remains future); fuller
  source-stability entries.
- `lambdas-and-callable-composition.md`: "Compatible callables compose
  positionally" is wrong under A15; remap with `reshape` restates the
  universal algorithm; partial application (R8).
- `structural-shapes-and-compatibility.md`: mapping callable results to
  inputs (R7); outcome reshape gains a routing role; `>-` contrast (R6);
  source stability for `[N]`.
- `declarations-and-bindings.md`: multiple results (R1).
- `terms.md`: the reshape entry, and one consistent term for a forwarded
  result sequence (the owner already says "bare result sequence").
- `using.md`: optionally, one clause stating that resource lists never match
  implicitly.

Re-run after R1 to R8 are aligned and the corrections are applied.

#### Resolutions (2026-11-10)

All aligned with the maintainer and recorded in the findings:

- **R1:** several bare new names are not a form. Typed capture and selection
  by label introduce names. Positional assignment to existing places and the
  short capture `value := measure()` remain (A15). The A25 examples use typed
  captures.
- **R2:** a colliding candidate is not viable; ordinary selection proceeds
  (A15).
- **R3:** `: = producer()` is explicit positional mapping and closes the
  "unlabel" deferral. `: getCount()` is a fix for the single-result intent
  error; parentheses are not (A15).
- **R4:** operators route like any callable. `[<label=incidental>]` on results and
  parameters marks names that carry no routing meaning (A30, A31).
- **R5:** a bare `:` in a routing group skips implicitly consumed parameters
  (A32).
- **R6:** `>-` members bind by name without moving the cursor (A33).
- **R7:** a reshape stage in `>|>` is a relabel of its left callable (A16).
- **R8:** a written partial-application prototype maps by position (A34).

Every listed correction has been applied. The principle is reworded to
include supplying one parameter twice, and use of common labels such as
`value` is accepted.

#### Change set additions

The "Change-set gaps" listed above join the promotion change set, together
with:

- `language/compiler-directives.md`: `[<label=incidental>]` in the directive
  catalog, with its meaning on results and parameters.
- `language/function-invocation.md`: `[<label=incidental>]` in the routing rule,
  the bare positional assignment example, and the short-capture boundary.
- `language/lambdas-and-callable-composition.md`: reshape stage as a relabel;
  partial application stated as positional adaptation.
- `language/structural-shapes-and-compatibility.md`: the stage-relabel
  definition replacing "explicit mappings consume both endpoints" in
  callable mapping.

The "unlabel" deferral is closed, so it no longer needs a destination.

### Dry run 3 (2026-11-10): FAIL

The same delegated reader re-checked the revised findings and change set.
R1 to R8 are captured correctly, and every dry run 2 item is resolved or in
the change set. Three narrow questions remain open, all about
`[<label=incidental>]`:

- **S1. Does a reshape label bind an incidental parameter?** A15 rule 1 says
  no implicit match happens when either label is incidental, but A30 says
  reshape entries may still name it.

  ```zax
  store2 final : ()(count : Integer, total [<label=incidental>] : Integer) = { }
  store2(getCount() reshape count: total:, myCount)
  ```

- **S2. Does an incidental parameter suppress the single-result intent
  error?** Same question for `>-` members.

  ```zax
  getTotal final : (total : Integer)() = { }
  store2(getTotal(), 5)
  ```

- **S3. What result labels do built-in operators carry?** The operator
  catalog declares none, so A31 is undefined for built-ins.

  ```zax
  record final : ()(total : Integer, result : Integer) = { }
  record(a + b, myResult)
  ```

Corrections found, to apply with the S1 to S3 update:

- **N1.** The `consumeAny` example is invalid: its second candidate is not
  viable either.
- **N2.** A30's main example shows nothing, because `total` already
  matches positionally.
- **N3.** The S1 contradiction.
- **N4.** "One meaning in every position" overstates A16, since structural
  `-<>-` keeps its rule. `except.md` restates the replaced callable rule.
- **N5.** A31 should explain why multi-result operators still need a group:
  only a bare invocation is a mapping-capable producer.
- **N6.** The directive grammar defines a slot after a parameter's name only.
  The catalog must add a result-name slot, and `label` is the directive name
  with value `incidental`, not a key.
- **N7.** A26's local pack needs one sentence explaining why it is not the
  form R1 rejects.
- **N8.** "Renaming results" omits `text: caption:, number: count: = produce()`,
  and the quoted phrase "bare positional destinations" means something else in
  current text.
- **N9.** A34's capture uses an undeclared `number`.

Leftovers to fix:

- "carried values" still appears in A16;
- process wording remains in one rejected row;
- the anonymous typed destination is missing from the single-result fixes;
- the stale "unlabel" references in the change-set lists;
- teaching plans and ownership rows for A30 to A34 are missing.

Change set additions:

- `compiler-directives.md`: slot list, source stability, and diagnostics;
- `operator-catalog.md`: built-in operator result labels (S3);
- `except.md`: the replaced reshape rule;
- `terms.md`: an "Incidental label" entry;
- optionally, one link line in `operators.md`.

After S1 to S3 are decided and the corrections applied, a re-check is
expected to pass as integration only.

#### Resolutions (2026-11-10)

- **S1:** a reshape label is written intent and binds even an incidental
  parameter (A15 rule 1, A30).
- **S2:** an incidental parameter raises no single-result intent error; the
  same holds for `>-` members (A30).
- **S3:** built-in operator results are incidental, stated in the operator
  catalog (A31).

Corrections N1 to N9 and the listed leftovers are applied. A33 now shows both
the ignored-member case and the explicit-parameter case. A34 declares
`number` and explains the partial-application form.

#### Ownership and teaching additions for A30 to A34

| Finding | Owner | Teaching weight |
| --- | --- | --- |
| A30 `[<label=incidental>]` meaning | `function-invocation.md`, the routing rule | Rule lives here; one example |
| A30 directive entry and slots | `compiler-directives.md`, directive catalog and "Where a directive goes" | Reference row; result-name slot added |
| A31 operators, built-in results incidental | `operators.md` result shapes (link) and `operator-catalog.md` (statement) | One sentence each |
| A32 bare `:` skipping implicit matches | `function-invocation.md`, two cursors | Short, with the action-at-a-distance note |
| A33 `>-` members by name without moving the cursor | `structural-shapes-and-compatibility.md`, decompose with `>-` | Both examples |
| A34 partial application by position | `lambdas-and-callable-composition.md`, partial application | One sentence plus example |

| Section | Why the reader is here | What they already know | What changes | What they might wrongly try |
| --- | --- | --- | --- | --- |
| Routing rule, incidental labels | A conventional label like `result` keeps causing intent errors | The routing rule | Marking a label incidental stops implicit use; explicit use and reshape labels still work | Marking every label incidental, losing useful routing |
| Directive catalog | Looking up `label` | Other directives | Value `incidental`; slots after parameter and result names; part of the visible prototype | Placing it after the type |
| Operators | Passing `a + b` as an argument | Operators as callables | Built-in results are incidental; declared operators follow their declarations | Expecting a multi-result operator to forward like a call |
| Two cursors | Mixing a routing group with a bare sequence | Two independent cursors | A bare `:` skips implicitly claimed parameters, even ones claimed later | Assuming `:` always means the next written position |
| Decompose `>-` | Spreading a struct into a call | Name-based decomposition | Members don't move the cursor; unmatched members are ignored | Expecting an unmatched member to fill a remaining parameter |
| Partial application | Building a callable from a target | Capture lists | The written prototype renames remaining inputs in order | Expecting the written names to route by label |

#### Change set additions (dry run 3)

- `language/compiler-directives.md`: catalog row for `label` with value
  `incidental`, its kind (semantic, part of the visible prototype), the
  result-name slot in "Where a directive goes", source stability (adding or
  removing the marking reroutes calls), and diagnostics for misplacement.
- `language/operator-catalog.md`: built-in operator results are incidental.
- `language/except.md`: replace the restated callable reshape rule
  ("explicit entries consume their source and destination labels") with the
  relabel definition.
- `language/terms.md`: an "Incidental label" entry.
- `language/operators.md` (optional): one link line in result shapes.
- `language/using.md` (optional): one clause that resource lists never match
  implicitly.

### Dry run 3 re-check (2026-11-10): PASS

The same delegated reader confirmed:

- S1 to S3, N1 to N9, and every listed leftover are resolved;
- no alignment question remains open.

Its last corrections are applied:

- A15's discardable `text` in the `consumeNumber` example;
- valid renaming examples;
- the redeclaration marked as an alternative;
- `g` and `point` declared;
- A16's pack example given an enclosing function;
- the `>-` incidental clause stated with an example;
- the principle limited to meaningful labels;
- one teaching-plan wording fix.

Remaining work is integration, made against current owners during
promotion: examples rebuilt as self-contained teaching, C1's `final`
declarations, and the postfix-`reshape` precedence confirmation.

This result does not authorize promotion.

### Promotion (2026-11-10)

The maintainer authorized promotion, accepting changes beyond the original
scope.

Four agents with fresh contexts promoted disjoint file groups against the
change set:

- invocation cluster;
- composition cluster;
- generics, iteration, and arrays;
- the new `language/variadics.md`, routers, legacy pages, glossary, and raw
  inputs.

Validation:

- **Links and anchors.** A script check of every relative link and heading
  anchor in the live tree, excluding `project/archive/` and `project/work/`,
  found no problems, before and after fixes.
- **Leakage.** No numbered-work, finding-number, or process vocabulary
  appears in `language/`.
- **Cold-reader review.** A separate agent that had not read this record
  reviewed the promoted sections as a direct reader. Its findings were fixed:
  - invalid examples: `fill`, discarded results, undeclared `parse`
    outcomes, an inline variadic lambda without storage, colliding consumer
    shapes;
  - dependency order in `variadics.md`;
  - a misplaced subsection in `function-invocation.md`;
  - reshape-stage wording;
  - element-position wording, now zero-based throughout;
  - the "processing" vocabulary claim.
- **Dismissed.** Its comments on Source / Provenance and Supersedes rows and
  "Later work must preserve" lists follow established owner metadata and
  boundary style.
- **Postfix `reshape` precedence.** The operator catalog was silent. It now
  states that postfix `reshape` binds more tightly than `|>`, consistent with
  existing examples.
- **Example adjustments.**
  - `foot: footer()` in the pack-mapping failure example, because a
    parameter after a pack is reachable only by label.
  - `allSameWidth(from Items)`, because a pack contributes values only through
    `from`.

Open questions raised during promotion. Nothing was decided; the promoted
text avoids asserting an answer:

- **P1.** The order of `...`, `#`, and a directive within one name slot,
  for example where `#` goes on a discardable result pack.
- **P2.** Whether a mapping's final stage with one ordinary result plus a
  discardable result counts as one value per item.
- **P3.** Confirmation of `allSameWidth(from Items)` as the spelling for
  passing a type pack to a compile-time function.

Remaining before closure:

- Decide P1 to P3, and apply them if needed.
- Teaching-debt disposition. No observations were recorded.
- Final validation by the maintainer.
- Archival of this file.
- Preparation of item 038.

A pre-existing outcome-reshape fragment in structural shapes, with `parse`
and `source` undeclared, was left untouched as outside the promoted
change.

#### Post-promotion review and resolutions (2026-11-10)

The promoting agent reviewed every promoted diff and the new
`language/variadics.md` against the findings. No substantive defect was
found, and one sentence in `variadics.md` was clarified.

Maintainer decisions, applied to current owners:

- **Spacing.** One space on each side of `...` is required. Other spacing is
  an intent error (`source-structure.md`), not a formatter repair.
- **Postfix `reshape` precedence.** It binds more tightly than `|>`; the
  operator-catalog text stays.
- **P1.** In a name slot the order is: name, then `...`, then `#`, then
  directives (`source-structure.md`, `compiler-directives.md`).
- **P2.** A per-item chain's last stage contributes its first result. Later
  results must be discardable and go unused, as in short capture
  (`value := measure()`). A discardable result therefore never joins a pack.
  This is applied in `variadics.md` (with the `measure` example), in the
  lambdas chain section and diagnostics, and in the raw flattening input's
  example comments.
- **P3.** `allSameWidth(from Items)` is confirmed: a type pack's types are
  passed to a compile-time function, which is legal in a compile-time
  context.
- Examples that bind an unread result name follow the existing fragment
  style and are accepted.

Remaining before closure: maintainer review, teaching-debt disposition (none
recorded), archival of this file, and preparation of item 038.

### Maintainer review of the promoted documentation (2026-11-11)

#### Wording revisions applied

The maintainer reviewed the promoted prose and authorized wording fixes,
applied to `arrays-and-slices.md`, `declarations-and-bindings.md`,
`except.md`, `function-invocation.md`, `generics.md`,
`lambdas-and-callable-composition.md`, `operator-catalog.md`, and
`variadics.md`, plus two older passages in the same spirit
(`compile-time-code.md` "the source says so"; the lambdas `(input : ...)`
layout line). None changes meaning, except as noted:

- The undefined term "short capture" is replaced by plain descriptions in
  all three owners that used it.
- `number, text := produce()` is now explained by how destination entries
  are read: each entry stands alone, and the `:` of `:=` belongs only to
  `text`. So `number` is an assignment into an existing place, and `text`
  would take an inferred type by position alone. That is the rule
  broken: an inferred new name must select its result by label. This
  replaces the earlier "not a form" framing.
- A pack is not a value "even when it holds a single item", because the body
  is checked for every count.
- "A variadic function can return as many results as it received inputs" was
  wrong (`makeDefaults$` has no inputs); it now says a result pack returns a
  variable number of results, fixed per processing.

#### Answered and aligned

- `mixedCall(giveMeY(), 0, 1)` with a one-result `giveMeY` labeled `y` is the
  single-result intent error. It neither fills `x` and `z` nor reports a duplicate.
- Discard and input matching: with no label matches, a discardable result is
  still mapped by position while a parameter remains.
- No source form can place `...` next to `(` or `)`, so the one-space rule
  stands. Constraint on future work: reintroducing anonymous packs or a
  postfix expansion operator would reopen it.
- **Variable names that match a different parameter: linter only.** The
  language rule from A15 is unchanged: a name the caller chose never routes
  and never causes an error by itself.
- **Several input packs separated by labels: viable (direction aligned, not
  yet designed in full).** A second pack is a parameter after a pack, which is
  already reachable only by label, and a label already starts a pack. The same
  holds for variadic `$` parameters in a `$` list. This revises A4's
  one-variadic-input limit and the "several variadic `$` parameters" and zip
  deferrals once the result-pack question below is settled.

  ```zax
  tagged$ final : (Tags ... : type, Values ... : type)()(tags ... : Tags, values ... : Values) = { }

  tagged("a", "b", values: 1, 2.5)   // tags takes "a" and "b"; the label switches to values
  makeDefaults$(Firsts: Integer, String, Seconds: MyValue, Float)()
  // illustrative: a variant of makeDefaults$ declaring two type packs, Firsts and Seconds,
  // supplied by label; a $(...) label names the $ parameter, so it is capitalized
  // like Integer$(LogicalBits: 57)
  ```

- **No lockstep walking of two packs.** The maintainer does not expect Zax to
  walk two packs together. Several packs are for passing distinct lists, each
  used on its own.

- **R1, applied: explicitly labeled parameters count as consumed.** This
  revises A15's "a parameter supplied by an explicit label is not skipped, so
  `f(b: 1, a: 2, x)` is still a duplicate" and the dry run 2 correction that
  stated it. Every parameter supplied by name, whether by a label match, a
  `>-` member, or an explicit label, is consumed. After a named input, the next
  positional input fills the first open parameter after it. A duplicate arises
  only when a label names an already-filled parameter. The cursor does not
  wrap back, so `report(count: = getCount(), myTotal)` is still an error.
  Maintainer rationale: one notion of "consumed" is simpler and matches the
  expected model. Applied to rule 6, the routing rule's second step, and the
  `place` example in function invocation.

  ```zax
  place final : ()(x : Integer, y : Integer, z : Integer) = { }

  place(y: 1, x: 2, 3)   // after x:, y is already supplied, so 3 fills z
  place(1, x: 2)         // error: x is supplied twice, by position and then by label
  ```

- **R2, partly aligned: result packs carry a pack-level label.** The pack's
  declared name labels the pack as a whole, not each item; per-item labels
  remain rejected. Aligned:
  - a pack label matches only an input pack, never an ordinary parameter,
    even when the count is 1;
  - a whole pack can be captured into a local pack by label, with a spelling
    still to be chosen;
  - a reshape relabels a whole pack, and `[N]` still names individual items.

  Consequence: a prototype may have several result packs, with counts
  supplied by labeled `$(...)`. That answers `combiner`, whose count could
  otherwise come only from its body.

  ```zax
  makeBoth$ final : (Firsts ... : type, Seconds ... : type)(firsts ... : Firsts, seconds ... : Seconds)() = { }
  store$ final : (Firsts ... : type, Seconds ... : type)()(firsts ... : Firsts, seconds ... : Seconds) = { }

  store(makeBoth$(Firsts: Integer, String, Seconds: Float)())
  // firsts binds firsts as a group; seconds binds seconds as a group
  ```

- **R3, aligned: `[<label=significant>]`.** A significant label must be
  matched by name. `essential` was the better word but reads as "required, not
  defaulted". The neutral default is spelled `[<label=descriptive>]`. A
  significant parameter is satisfied by an explicit `name:` label, a forwarded
  result with that label, a bare variable with that name, a `>-` member with
  that name, or member access ending in that name. Explicit positional
  `: whatever` does not satisfy it. A matching variable name validates but
  never routes. Parameters only. Significant result labels are deferred until
  a producer needs them, because they would forbid positional captures,
  typed positional captures, and `value := f()`. Owner changes (directive
  catalog, function invocation) await promotion authorization.

#### Open, under discussion

- **R2a. Labeled return values.** No owner defines a return that labels
  values, such as `return number: 42, text: "answer"`. Only positional
  `return 42, "answer"` and routing groups (`return text: out:, number: n: =
  produce()`) exist. The maintainer's example
  `return firsts: 1, "foobar", seconds: 5.0, "hello", "goodbye"` would need
  that form, with the call cursor model applied to result slots, where a label
  starts a pack that keeps absorbing. It is coherent, because returns already
  follow the same routing rule. Separately, the example's `makeValues` has no
  `$` on its name, and nothing supplies `Firsts` or `Seconds`, so its counts
  would come from the body, which remains disallowed. A body that knows its
  exact values has a fixed result count and declares ordinary results.
- **R4. Growing packs.** Hidden size is dropped as a concern. The maintainer
  sees flattening as producing a long, label-free list, and raises a second
  option: each item's several results become one anonymous structured value,
  so the pack keeps its size and labels survive as members. Agent view: the
  structured option is better, but should be explicit. Function invocation
  states that several results stay several values until `-<` explicitly
  creates one, and an implicit struct per item would change the pack's element
  type whenever a producer gains a result. An explicit per-item recomposition
  in the chain would be safe, because a receiver using `.x` and `.y` keeps
  working when `z` is added. No spelling has been chosen, and a recomposition
  stage in a `|>` chain does not exist yet.
- **R5. Taking every item by reference: a real gap, correcting the earlier
  candidate.** Type layers stack: `MyValue & * &` is valid, and
  `MyValue & &` is rejected. So `items ... : Items readonly &` is
  well-defined only if working out matches the declared shape. Read naively,
  as "append `&` to the worked-out type", a reference argument produces a
  reference to a reference:

  ```zax
  inspectAll$ final : (Items ... : type)()(items ... : Items readonly &) = { }

  myFlag : Boolean = true
  myView : Boolean & = myFlag
  myPointer : Boolean * = @

  inspectAll(myFlag, myView, myPointer)
  // myFlag: Items[0] = Boolean, item is Boolean readonly &
  // myView: appending gives Boolean & readonly & (error: a reference to a reference);
  //         matching the shape gives Items[1] = Boolean, item is Boolean readonly &
  // myPointer: Items[2] = Boolean *, item is Boolean * readonly &, a readonly reference to the pointer
  ```

  Pointers are not a problem: a reference to a pointer is an ordinary layer.
  The question is the reference case. `generics.md` says a worked-out type
  "is exactly the argument's type" for a plain `UseType` parameter, and
  already works out through a written shape (`bits : MyBits$(UseType)
  readonly &`). It does not say whether a stated `&` matches an argument's
  reference. Candidate rule: a stated `&` in the parameter matches the
  argument's reference layer when there is one, and binds a reference to the
  argument's place otherwise. Owner: generics (working out from arguments),
  with qualifiers for the layer rule.

#### Decisions and edits (2026-11-11, later)

- **R3 applied.** `function-invocation.md` gains a "Significant labels"
  section with the three markings (`descriptive` the default, `incidental`,
  `significant`), plus a diagnostics entry and a source-stability entry. The
  "put together" principle now reads: a name the caller chose never routes,
  and causes an error only when a significant parameter receives a value that
  doesn't show its name. Also updated: the `label` row and source-stability
  entry in `compiler-directives.md`, and a new `terms.md` entry. Edges not
  yet decided, so the owners say nothing about them: `significant` on a pack
  parameter; whether a forwarded result whose label is incidental satisfies
  a significant parameter; and `: caption`, explicit positional intent with
  the matching name. Only `: whatever` was ruled out.
- **R5 applied as canonical.** `generics.md`, under working out from
  arguments: a written `&` after a type parameter matches the argument's own
  reference when there is one, and otherwise binds a reference to the
  argument's place, so a reference to a reference never forms. A pointer is
  an ordinary value, so `&` adds a reference to it. `variadics.md` shows
  `items ... : Items readonly &` and links to the rule.
- **R2a aligned: labeled return values are wanted.** The proposed form mirrors
  named call inputs, `return label: value`, with the call's cursor and
  pack-absorbing rules applied to result slots. It stays distinct from a
  return routing group, which ends in `= producer()`. Spelling and owner
  changes await authorization, together with R2 and several input packs.
- **R4 aligned: one structured value per item.** The mechanism is under
  discussion.

#### Significant-label edges: aligned and applied (2026-11-12)

These revise the edges R3 left undecided.

- **Which forms show a name.** Only fixed built-in constructs that pass the
  value along unchanged: a bare variable, member access (the member's name),
  grouping, a leading `:` (neutral), a stance restatement such as
  `as copy`, a `>-` member, and an argument declaration with that name. A
  call result shows a name only through its own declared label. Any operator,
  conversion (`as MyType`), or other call makes a new value without one, so
  `caption + "!"` needs `caption: caption + "!"`.
  - *Rejected:* carrying a name through operators when exactly one operand
    has one. Maintainer reasoning: `sqrt(area)` no longer means an area;
    marking `sqrt`'s result incidental would carry the name through despite
    the change in meaning, and a meaningful label would replace it anyway.
    Fixed built-in constructs make the line clear rather than arbitrary.
    Adding the carried-name form later stays compatible, because it only makes
    more calls valid.
- **Incidental result labels** do not satisfy a significant parameter.
- **Significant packs** are checked as a whole: started by an explicit label,
  a forwarded result pack with that label, or `from` a source pack with that
  name. An empty pack is fine.

Applied in the "Significant labels" section of `function-invocation.md` and
its diagnostics, and in the `terms.md` entry.

#### Pack labels, labeled returns, and `-<` per item: aligned and applied

Authorized as one change set after `-<` was settled.

- **Several packs.** A prototype may declare several input packs and several
  result packs, and a `$` list several variadic `$` parameters. Each one after
  the first starts at its label and absorbs until the next label;
  `$(...)` labels use the `$` parameter names (`Firsts:`). This revises A4 and
  retires the zip and several-`$` deferrals. Two packs are never walked
  together: maintainer direction, not a deferral.
- **Pack-level labels.** A result pack's declared name labels the whole pack.
  It binds a same-labeled input pack as a group, never an ordinary parameter,
  even at count 1. `reshape results: values:` relabels the whole pack, and
  `[N]` still names items. Local capture by label:
  `firsts: lefts ... :, seconds: rights ... : = makeBoth$(...)()`. Result
  pack counts never come from the body.
- **Labeled return values.** `return label: value` follows the call's cursor
  rules over result slots, and a label on a result pack starts it. It is
  distinct from a return routing group, which ends in `= producer()`.
- **Destination-less `-<`.** `-< producer()` makes one anonymous value, and
  `MyType -< producer()` makes one `MyType`, in any expression. As a chain
  stage it binds more tightly than `|>`, and the item is carried into the
  stage's invocation as usual. Every labeled result becomes a member,
  discardable ones included, so a producer gaining a result adds a member
  rather than shifting positions. A result pack cannot become a member:
  route it out first. Resolves function invocation's future-work item
  "explicit promotion of several results into one structural value". A
  per-item chain whose last stage offers several values is still an error,
  and the diagnostic suggests `-<`. Spreading several values per item into a
  pack stays deferred in the raw function-composition input, now without the
  discardable and hidden-size reasons.

Owners changed: `variadics.md`, `function-invocation.md`, `generics.md`,
`structural-shapes-and-compatibility.md`, `declarations-and-bindings.md`,
`lambdas-and-callable-composition.md`, `operator-catalog.md`, `terms.md`,
`project/raw/type-parameters-and-generics.md`,
`project/raw/function-composition-and-chaining.md`, and
`project/raw/README.md`.

#### `-<` precedence wording (2026-11-12)

The operator catalog said the structural mapping forms take their grouping
from declaration, call, and result-mapping grammar "rather than inventing
ordinary arithmetic-expression precedence", which sat oddly beside the new
expression form. Maintainer direction: state it plainly as precedence. The
catalog now names the one expression form, `-< <producer>`, which applies to
the following invocation and has greater precedence than `|>`. Its other
mentions in the catalog, structural shapes, and variadics say "has greater
precedence than `|>`". The postfix-`reshape` wording ("binds more tightly")
was left as is.

#### Cold-reader review (2026-11-12)

A separate agent with no access to this record reviewed the changed owner
sections. Fixes the maintainer authorized as obvious, now applied:

- the inferred-name rule limited to destination lists taking several results;
- "every routing error comes from one of two situations" made three
  label-related situations;
- cursor rule 6: forwarded labels and `>-` members claim their parameters
  before positional placement, while explicit labels claim in source order;
- the `sqrt(area)` sentence covers both operators and calls;
- the duplicate `tagged$` in generics renamed to `annotate$` and moved after
  the label rule it builds on;
- `descriptive` introduced before the marking table, a link added for
  argument declarations, and the `terms.md` list marked as examples;
- the routed-entries-first clause added to both `produceReport$` examples;
- overload rule order aligned between invocation and variadics;
- the `store(getTotal(), 5)` comment states that the call is positional;
- `...` placeholders in chain prose replaced;
- smaller teaching fixes: the "both cases" link text, `compareForms$`,
  `describeSizes$`, the `checkSource` comment, the redeclared `parse` renamed
  `parseTree`, and the section pointer for `DevicePoint`.

Dismissed: the `variadics.md` provenance row, which is standard metadata, and
the spreading hypothetical in "One value per item", which teaches a reason.
Undeclared placeholder names stay unless an example is unclear without a
prototype (maintainer direction).

Resolved afterwards, with the maintainer:

- **Stances at a last forwarded use.** The intent error applies whenever the
  receiver could take an item over, including an ordinary by-value parameter,
  not only one declared `last`. `variadics.md` now says so, and the generic
  `collect$` example in arrays states `as copy`. Maintainer direction: plain
  types such as `Integer` should not need a stance, but Zax has no definition
  of such types yet, so `Integer`-only examples stay unannotated without
  claiming a rule. Captured in the new raw input
  `project/raw/terminal-transfer-intent.md`, indexed in the raw README: a
  plain-data definition, an explicit type-level signal, and how generic
  bodies are checked.
- **Built-in operator labels.** These now read "a built-in operator's result
  label is incidental", matching `operators.md`, in both places in function
  invocation and in the operator catalog. A label that exists but may never be
  used stays under invocation's existing future work on anonymous or private
  labels.

#### Clarified in discussion: typed declarations inside a call

`consume(value : Integer, text : String = produce())` is not a typed capture.
The commas separate two arguments, and each is a bare declaration expression
at the top level of an argument, which is an intent error because it may be a
mistyped label. With `:` or grouping, each would be its own argument
declaration expression. `text : String = produce()` would then capture only
`produce`'s first result, and every later result would have to be
discardable. A typed capture of several results is a statement-level form.
In a call, `consume(produce())` follows the forwarded-sequence rule: labels
first, then positions. `consume(: = produce())` goes purely by position.

#### Documentation lessons, applied

Added to the example obligations in `project/documentation.md`:

- no `...` in a parameter or result position;
- no coined term used before it is defined;
- say the concrete thing: no stock phrasings such as "says so", and no unclear
  "it" in a rule.

### Closure (2026-11-12)

#### Post-promotion dry-run update: PASS

The maintainer review added design after the dry run 3 re-check passed. Each
addition has one lasting owner, and the readers' route reaches it from
`variadics.md` or function invocation:

| Decision | Owner | Also integrated in |
| --- | --- | --- |
| Explicitly labeled parameters count as consumed; the cursor never moves back | Function invocation (input mapping rule 6, forwarding) | — |
| Several input packs, result packs, and variadic `$` parameters, separated by labels; two packs never walked together | Variadics | Function invocation, generics, terms, raw generics |
| A result pack's name labels the whole pack, matches only an input pack, and can be relabeled by a reshape | Function invocation (result packs) | Variadics, structural shapes, declarations (capture by label) |
| Labeled return values | Function invocation (forwarding from a return) | Variadics |
| Destination-less `-<` as an expression and a chain stage; every labeled result becomes a member; result packs cannot be members | Structural shapes | Variadics, lambdas, operator catalog |
| Label markings `descriptive`, `incidental`, `significant`; which forms show a name | Function invocation | Compiler directives, terms |
| A written `&` after a type parameter matches an existing reference | Generics (working out from arguments) | Variadics |
| A built-in operator's result label is incidental | Function invocation | Operator catalog, operators |
| Example rules for `...`, coined terms, and stock phrasing | Documentation architecture | — |

Deferred material and its live homes:

- tuple-like types over a pack, folding across mixed packs, and compile-time
  iteration over pack indexes: raw type parameters and generics;
- spreading several values per item into a pack, and short-form lambdas: raw
  function composition and chaining, and the lambdas boundaries;
- when a last use needs a stance, and a plain-data definition: raw terminal
  transfer intent (new);
- significant result labels, and anonymous or private labels: function
  invocation future work;
- identifying which item of a mapped pack failed: except boundaries and the
  variadics boundaries;
- untyped foreign variable arguments: raw interop;
- generating declarations by walking a type pack: raw compiler-directive
  maintainer notes;
- variadic mixfix call components and variadic operator phrases: those owners'
  boundaries and the variadics boundaries.

Rejected alternatives that could look plausible to a later reader are kept in
"Rejected alternatives" above and in this review's sections: names carried
through operators, `essential`, and the alternative label vocabularies.

#### Teaching debt

No observations were recorded. The cold-reader review's findings were fixed or
dismissed with reasons above. Nothing is entered in the teaching-debt log.

#### Validation

- Relative links and heading anchors across the live tree, excluding the
  archive and this file: no problems.
- No numbered-work, finding, or process vocabulary appears in `language/`.
- Staged boundary preserved. The maintainer staged the promotion and most
  review edits. The closing edits stay unstaged on top: the final
  arrays, function-invocation, and raw generics changes, and the project and
  archive routers. The archive move of this file shows as a staged addition at
  the old path, deleted in the working tree, plus an untracked file at the
  archive path; the new item 038 is also untracked.

#### Successor

Work item `038`, when a terminal use needs a stated stance and what counts as
plain data (`project/work/038-terminal-transfer-intent.md`).
