# 039: Pointer stances

| Field | Value |
| --- | --- |
| Status | Historical / non-normative / audit-only; closed 2026-11-22 |
| Work Item | `039` |
| Created | 2026-11-16 |
| Owns | The bounded review of whether, and how, a stance can describe what may happen to a value reached through a pointer, an optional, or an array, separately from the stance of the pointer, optional, or array itself |
| Does Not Own | Current transfer-stance, pointer, optional, array, qualifier, or invocation behavior, which remain with their owners until a promotion is authorized; the decision itself, which belongs to the language maintainer |

## Non-authority notice

This file is a collaborative working record. Existing documentation, candidate
syntax, examples, and later aligned findings remain non-authoritative until a
separately discussed, aligned, and explicitly authorized promotion incorporates
them into their lasting owners.

## Fixed initiating input

This section records the aligned information known when work item `039` was
created. It is intentionally incomplete and must not be rewritten as work
develops.

### Initiating concern

A transfer stance applies to one whole value. A pointer's whole value is the
pointer itself, its address and any ownership it holds, so nothing in a
pointer type says what may happen to the value it points to. Should a stance
be able to describe the pointed-to value separately from the pointer, through
any nesting of `*` and `&`, and if so, how? The same question applies to an
optional and the value it contains, and to an array and its elements.

### Motivating pressure

- **A loan cannot travel through a pointer.** Making a raw pointer from a loan
  is an error outside `unsafe`, because the pointer cannot carry the loan's
  promise that the whole value is not copied or taken. A data-oriented
  language that works through pointers, and interoperates with C, needs to
  lend through a pointer without `unsafe`.
- **`_` already does it.** The receiver `_` is pointer-shaped and carries a
  loan in a function with no receiver stance, so the language already has a
  pointer-shaped path whose pointed-to value has a stance. Pointer types cannot
  express the same thing.
- **Optionals and arrays have the same shape.** The maintainer is equally
  concerned with optionals and arrays. Each holds or reaches a second value:
  the optional's contained value, the array's elements. Whether that inner
  value needs its own stance, separate from the optional or array, belongs to
  the same design.

### Why the whole-value model has been outgrown

This explanation was requested by the maintainer as part of the initiating
input. It is the agent's framing for 039 to test, not an aligned conclusion.

**The original model.** A stance was something written at a use, such as
`as move`: a one-time instruction for one complete value at one call.
`qualifiers.md` recorded the consequence: transfer stance is not a fourth
per-layer qualifier axis, because `copy`, `deep`, `move`, and `last` "describe
how one complete source or receiver is offered to a consumer". A one-time
instruction has no reason to be part of a type, so a slot for it on each layer
made no sense.

**What changed in work item 038.** Stances became lasting properties of
places and paths, not only one-time instructions:

- a stance declared on a variable is a requirement on every later transfer
  from it;
- `loan` is the stance of a reference, enforced on everything made from it;
- a reference's stance already describes the value it refers to, not the
  reference: `& loan`, `& move`, and `& last` say what may happen to the
  referent;
- members follow their own declared stance, independent of the value that
  contains them;
- a reference or receiver passes the whole value on with at most the stance it
  was given;
- a `self` result carries the caller's variable with the stance written on the
  call;
- `_` is pointer-shaped and carries a loan.

So a reference already has a stance on its pointed-to layer, while a pointer
has a stance only on its own layer. Members already have stances independent
of the containing value. The rule "a stance applies only to a whole value" now
describes the stance written at a use, but not the stances the language
actually tracks.

**Why independent stances may be better.** A pointer, an optional, and an
array each involve two things that can be treated differently: the container
or pointer itself, and the value inside or pointed to. One stance cannot say
both "you may take ownership of this pointer" and "you may only borrow what it
points to", or "you may move this array" and "you may not take its elements".
Giving the pointed-to or contained value its own stance could:

- let a loan travel through a pointer, an optional, or an array without
  `unsafe`;
- explain `_` as a pointer whose pointed-to value is on loan, instead of a
  special case;
- make references and pointers uniform, if a reference's stance is simply the
  stance of its pointed-to layer;
- help with allocation, for example lending an allocation without passing its
  ownership.

The costs to weigh include syntax on each layer, defaults that keep existing
pointer code working, generic code over types that may or may not be
pointers, combinations with ownership kinds that make no sense, interop where
nothing can be enforced, and diagnostics that say which layer's stance a use
violates.

### Starting input

- [Raw pointer stances](../raw/pointer-stances.md): the concern, the
  motivating examples, the history of the per-layer decision, the agent's
  candidate direction (a stance on the pointed-to layer, such as
  `MyValue loan *`, unrestricted when unmarked), the questions the work must
  answer, and the captures from the 038 review: pointer allocation, array
  elements, optional values, and who frees a `copy` allocating result.
- The maintainer has no preferred solution. The agent is asked to produce a
  range of options, from keeping the current rule to independent stances on
  every layer, each with examples, advice, a recommendation, and the
  feasibility pressure it places on an eventual implementation.

### Starting boundaries

- In scope: stances on the pointed-to layer of raw, `unique`, `strong`, and
  `weak` pointers; how stances attach through nested `*` and `&`; whether a
  reference's stance is the same thing; the receiver `_`; optional contained
  values; array elements; conversions that add or drop such a stance;
  diagnostics; generic code.
- The meanings of the five stances (`loan copy move last deep`), and the 038
  rules for references, receivers, members, results, and `self` results, are
  current design. A conflict with them is recorded as a finding for
  discussion, not resolved by silently changing them.
- Pointer ownership kinds, allocation, and arenas are reviewed only as far as
  stances touch them.
- Until 039 decides otherwise, making a raw pointer from a loan, including
  passing a loan `_` to a raw pointer parameter, remains an error outside
  `unsafe`.

### Initial stopping guidance

First present the candidate programmer model, the options with code and a
recommendation, the most important uncertainty, and the decisions the
maintainer must make. Do not design every combination of layers in full
before the direction is aligned.

Creating this work item does not authorize analysis. A later assignment
begins it.

Do not promote findings, change current owners, archive this work item, or
create work item `040` without the separately required discussion, alignment,
and authorization.

## Reading scope

> **A session resuming this work item** starts with "Next session: how to
> resume the promotion" under "Promotion progress" near the end of this file.
> It replaces the original analysis reading below.

### Required

- [Documentation architecture](../documentation.md) - governs focused reading,
  authority, promotion, deferral, closure, and the writing rules.
- [Raw pointer stances](../raw/pointer-stances.md) - the preserved concern,
  examples, candidate direction, and questions.
- [Zax transfer stances](../../language/transfer-stances.md), especially
  "Loan", "References pass on at most their stance", "Receiver stance",
  "Passing the receiver on", and "Pointer ownership transitions" - the current
  model this work extends.
- [Zax qualifiers](../../language/qualifiers.md), how qualifiers attach to
  each level of indirection - the existing per-layer precedent.
- [Zax pointers, allocation, and arenas](../../language/pointers-and-arenas.md),
  "Pointer ownership roles", the raw-pointer sections, and
  `OpaqueReferenceObserver` - what pointers mean today.

### Consequence-driven

Read only the smallest relevant sections when the concern reaches them:

- [Zax optional values](../../language/optional-values.md), when contained
  values are reached.
- [Zax arrays and slices](../../language/arrays-and-slices.md), when elements
  or slices are reached.
- [Zax lifetimes and references](../../language/lifetimes-and-references.md),
  when reference stances or returned references are compared with pointers.
- [Zax Nothing instances](../../language/nothing-instances.md), when the
  pointer shape of `_` is reached.
- [Zax generics](../../language/generics.md), when type parameters that may be
  pointers are reached.
- [Zax function invocation](../../language/function-invocation.md), when
  parameter selection or results through pointers are reached.
- [Raw mutability-indexed type families](../raw/mutability-indexed-type-families.md),
  when per-layer qualifiers and type-declared qualifier allowances interact.

### Audit-only

- Archived numbered work, only when a provenance question cannot be answered
  from current owners or raw input. The transfer-stances reviews are
  `013-transfer-stances-and-source-state.md` and
  `038-terminal-transfer-intent.md`; the reference and pointer review is
  `014-reference-origins-and-lifetime-strategies.md`.

## Working record

**How this record is organized** (restructured 2026-11-18 because the
previous layout had become unreviewable):

- **Part A, the current proposal.** Only what the maintainer and agent have
  aligned. It reads top to bottom. It is working material, not teaching
  text, and it is not accepted language design until a separately
  authorized promotion.
- **Part B, open questions.** Each has context, an example, the decision
  needed, and a recommendation. When one is answered, it moves into part A.
- **Part C, deferred work.** Each item has an example and what brings it
  back.
- **Part D, captures for other owners.** Items that belong outside 039.
- **Part E, consequences for promotion.** Which current documents would
  change.
- **Part F, decision log.** One line per decision or rejected idea, so
  rejected ideas never sit beside current ones.
- **Part G, the maintainer's input at assignment**, preserved.

The earlier initial reconstruction and the round-by-round discussion record
were removed in the restructure. They remain in the staged version and in
Git history for any audit.

All syntax in this record is **candidate syntax**.

### Part A: the current proposal

#### A1. What a stance means

A stance belongs to a value, at one layer of a type. It is written in the
same place as that layer's qualifiers, last among that layer's words. It
says **what may be built from that value**. It does not say how the value
itself was built.

| Stance | What may be built from the value |
| --- | --- |
| `loan` | nothing; the value can only be used where it is, or lent |
| `copy` | a copy, or an independent (`deep`) copy |
| `move` | only a new value that takes the contents (`move`) |
| `last` | only a terminal transfer (`last`) |
| `deep` | only an independent copy (`deep`) |

```zax
myValue : MyValue copy     // copies may be built from myValue
report : MyReport move     // anything built from report must take its contents
pinned : MyDevice loan     // nothing may ever be built from pinned
archive : MyArchive deep   // only independent copies may be built from archive
```

`copy` includes `deep`, because a deep copy is still a copy that leaves the
original unchanged. `deep` does not include `copy`: a `deep` value refuses an
ordinary copy, which might share resources.

A value declared `loan` is **pinned**. It can be used and lent, but never
copied, moved, or finished into another value. Pinning applies to the whole
value, not its members (A5).

#### A2. What an owner may write, and what a borrower may write

Who holds a value decides which stances may be written on it.

**An owner** holds its own value: a local or member you declared, a
by-value parameter (the function owns it), or the value an owning pointer
points to (A11). The owner of a `copy` value may also write `as move` and
`as last`, and the owner of a `move` value may also write `as last`, because
the value is the owner's to give up:

| Owner's value stance | The owner may write |
| --- | --- |
| `loan` | nothing (lending only) |
| `copy` | `as copy`, `as deep`, `as move`, `as last` |
| `move` | `as move`, `as last` |
| `last` | `as last` |
| `deep` | `as deep` |

**A borrower** reaches a value someone else owns: a reference, a raw
pointer that holds only an address, or the receiver `_`. A borrower may
write only the stance it was granted:

| Borrower's stance | The borrower may write |
| --- | --- |
| `loan` | nothing (lending only) |
| `copy` | `as copy`, `as deep` |
| `move` | `as move` |
| `last` | `as last` |
| `deep` | `as deep` |

Lending (`as loan`) is always allowed, for owners and borrowers alike.

**Why a `copy` borrower can't write `as move` or `as last`.** The caller
relies on the stance it wrote to know what state its variable is in
afterward:

```zax
remember final : ()(input : MyMessage copy &) = {
  stash(input as last)       // error: input was granted copy
}

message : MyMessage
remember(message)            // the caller expects message unchanged
message.send()               // would find a finished message if stash could take it
```

**Why a `move` borrower can't copy.** The value it reaches might be declared
`move`, which forbids copies, and the borrower can't see that declaration:

```zax
drain final : ()(input : MyMessage move &) = {
  keep(input as move)        // fine
  log(input)                 // error if log takes MyMessage by value: input was granted move
}

report : MyMessage move      // its owner said: never copy report
drain(report as move)        // if drain could copy input, report would be copied after all
```

So "you may keep copying, but you can't take it over" is a `copy` borrower:

```zax
looksLikeANormalCopy final : ()(value : MyValue copy &) = {
  keep(value)                // fine: copies
  keep(value as last)        // error: not granted
}
```

The same rules apply to receivers. In a version declared `last`, `_` may be
written only `as last` or lent:

```zax
MyDocument :: type {
  publish final : ()() last = {
    _.inspect()                // fine: no stance written, and _ (granted last) forbids a copy,
                               // so _ is lent to inspect's version with no receiver stance
    (_ as copy).snapshot()     // error: _ was granted last
  }
}
```

**A receiver is a parameter bound implicitly.** It follows the same rules as
an explicit parameter. The one difference is that a version with no receiver
stance is `loan` (A6). A reference parameter, by contrast, must write its
stance.

**With no stance written on a call, the receiver is treated like an argument
with no stance written:** it is copied where the value allows a copy, and
otherwise lent. That is what explicit arguments already do:

```zax
show final : ()(value : MyReport readonly loan &) = {
}

remember final : ()(value : MyReport readonly copy &) = {
}

plain : MyReport            // copy
report : MyReport move

remember(plain)             // no stance means copy: the copy & parameter binds
show(plain)                 // lending is always allowed
show(report)                // lending: fine, even though report forbids copies
remember(report)            // error: report forbids copies
```

The receiver gets the same rule:

```zax
MyCounter :: type {
  foo final : ()() = {
  }

  foo final : ()() copy = {
  }
}

plain1 : MyCounter           // copy
plain2 : MyCounter copy
locked : MyCounter move

plain1.foo()                 // the same as (plain1 as copy).foo(): the copy version
plain2.foo()                 // the same
locked.foo()                 // locked forbids copies, so it is lent: the version with no receiver stance
```

If a type has only a `copy` version, a call with no stance on a value that
allows copies uses it. On a value that forbids copies, that call is an error.

**A call on the type lends the Nothing instance (2026-11-22).** In a call
such as `MyType.inspect()`, the receiver is the type's Nothing instance, a
shared stand-in that is nobody's value. It is pinned: nothing is ever built
from it. So a call on the type uses the version with no receiver stance, and a
type with only a `copy` version can't be called on the type itself. Calls on
instances are unchanged:

```zax
// candidate syntax
MyResource :: type {
  inspect final once : ()() = { }
  inspect final once : ()() copy = { }
}

MyResource.inspect()          // the version with no receiver stance: the Nothing instance is lent
resource : MyResource
resource.inspect()            // the copy version: a copy value with no stance written
```

This reverses the earlier "otherwise a `copy` version" fallback for calls on
the type. It keeps one receiver rule ("no stance means `copy` where the value
allows it") with the Nothing instance as a value that doesn't allow it.

**`move`, `last`, and `deep` versions are chosen only when the call writes
the stance.** The variable's declared stance only limits what may be
written:

```zax
MyDocument :: type {
  doSomething final : ()() = {
  }
  // lends _

  doSomething final : ()() last = {
  }
  // may finish _
}

doSomethingWith final : ()(document : MyDocument loan &) = {
}

doSomethingWithLast final : ()(document : MyDocument last &) = {
}

document : MyDocument last

doSomethingWith(document)              // explicit parameter, no stance written: lending works
document.doSomething()                 // receiver, no stance written: the same; lends document

doSomethingWithLast(document)          // error: a last & parameter needs as last written
doSomethingWithLast(document as last)  // fine: document counts as finished
(document as last).doSomething()       // fine: the last version; document counts as finished
```

Two other options were rejected:

- Letting the declared stance choose the version, so that
  `document.doSomething()` would finish `document`. That would hide the
  transfer from a reader.
- Choosing the `last` version without counting `document` as finished. The
  version may finish `_` or hand it on, and the next line could then use a
  finished value.

**A version may serve any call whose written stance allows what that version
takes,** along `copy`, `move`, `last`, provided the value's own stance allows
the version's stance. The version with no receiver stance always qualifies.
The closest version to the written stance wins. This is the same rule as for
parameters, where `as last` may be written to a `copy &` parameter (A4):

```zax
doc : MyDocument                 // copy: its owner may write every stance
(doc as last).publish()          // the move version, if there is no last version:
                                 // it takes no more than last, and it is closer than
                                 // the version with no receiver stance
fixed : MyDocument last
(fixed as last).publish()        // the version with no receiver stance: fixed doesn't allow a move
```

A `last` version is never used for a call written `as move`: the caller
expects a usable value afterward.

**`self` results carry the written stance along a chain** (today's rule). The
variable counts as finished only once the whole expression is done:

```zax
MyFoo :: type {
  process final : (result self : MyFoo &)() last = {
    return _
  }

  doSomething final : (result self : MyFoo &)() last = {
    return _
  }

  myOhMy final : (result : MyValue)() last = {
    // the end of the chain: may take everything from _
  }
}

foo : MyFoo
takeTheLast := (foo as last).process().doSomething().myOhMy()
// each call gets foo written as last and picks its last version; foo counts
// as finished once the whole expression is done
```

A version that returns `self` must leave `_` usable for the next call in the
chain.

#### A3. Passing a value to a reference parameter

A value can be passed to a reference parameter when the parameter's stance
is one its holder may write (A2). From a value you own:

| Source you own | to `loan &` | `copy &` | `move &` | `last &` | `deep &` |
| --- | --- | --- | --- | --- | --- |
| `MyType loan` | yes | no | no | no | no |
| `MyType copy` | yes | yes | yes | yes | yes |
| `MyType move` | yes | no | yes | yes | no |
| `MyType last` | yes | no | no | yes | no |
| `MyType deep` | yes | no | no | no | yes |

From a reference parameter you hold, passed on to another function:

| Your reference | to `loan &` | `copy &` | `move &` | `last &` | `deep &` |
| --- | --- | --- | --- | --- | --- |
| `MyType loan &` | yes | no | no | no | no |
| `MyType copy &` | yes | yes | no | no | yes |
| `MyType move &` | yes | no | yes | no | no |
| `MyType last &` | yes | no | no | yes | no |
| `MyType deep &` | yes | no | no | no | yes |

The two tables differ only in the `copy &` and `move &` rows, because the
second holder is a borrower.

#### A4. Writing a stance at a use

`as` does not change any stance. It is the caller saying what it allows to
happen to its own variable in this one use, so a reader can see the
variable's state afterward:

- The stance written must be one the caller may write on that variable (A2).
- No stance written means `copy`.
- A reference parameter that may change the caller's value needs its stance
  written: `as move` for `move &`, `as last` for `last &`. A `deep` value
  needs `as deep`.

```zax
func2 final : ()(value : MyType move &) = {
}

myType : MyType move
func2(myType)                // error: a reader assumes myType is unchanged
func2(myType as move)        // fine: shows that myType may be emptied
```

**A reference stance is a permission, not a promise.** The function may or
may not use it, so the caller plans for the most the parameter allows:

| Parameter | The caller assumes afterward |
| --- | --- |
| `loan &` or `copy &` | nothing was moved out and it is not finished; through writable access its contents may have changed |
| `move &` | still usable, possibly emptied |
| `last &` | finished: it may only be destroyed or replaced |

```zax
func final : ()(value : MyType last &) = {
  // may finish value, or may not
}

myType : MyType last
func(myType as last)         // fine: myType counts as finished, whatever func did
func(myType as last)         // error: the caller already said it was done with myType
```

The second call is an error because the caller gave `myType` up, not
because `myType` is known to be empty. This prevents a double terminal
transfer. The destructor at the end of scope handles both states, as
destructors already must.

A call may write a stance that would allow the function to take the whole
value, even when the parameter only asks to copy it:

```zax
notTakingOver final : ()(value : MyValue copy &) = {
}

myValue : MyValue
notTakingOver(myValue as last)
// notTakingOver can still only copy myValue, because its parameter says copy &.
// myValue counts as finished afterward, because the call said as last.
```

If `notTakingOver` later becomes `last &`, this call still matches, and
nothing the caller assumed becomes false.

**Lending.** Any call may lend a value by writing `as loan`. A lent value
can only be passed to a reference parameter, or used as the receiver of a
function that declares no receiver stance. A by-value parameter can't
accept it, because the parameter would have to be built from the value, and
lending allows nothing to be built.

#### A5. Members

Members follow their own stance. Whether a member can be moved or finished
depends on writable access, not on owning the whole value:

```zax
refresh final : ()(input : MyMessage copy mutable writable &) = {
  keep(input as last)                 // error: the caller granted copy for the whole message
  keepPayload(input.payload as move)  // fine: writable access already allows emptying payload
}

inspectOnly final : ()(input : MyMessage copy readonly &) = {
  keepPayload(input.payload)          // fine: copies the member
  keepPayload(input.payload as move)  // error: readonly access can't change payload
}
```

The limit on a borrower exists to protect what the caller tracks: its whole
variable. A caller that grants writable access has already accepted that
members may change in any way, including being emptied by assignment.

Pinning applies to the whole value, not its members:

```zax
MyTank :: type {
  fuel : MyFuel
}

pinned : MyTank loan
keep(pinned)                  // error: nothing may be built from pinned
keep(pinned.fuel as move)     // fine: fuel follows its own stance
```

#### A6. When a stance may be left unwritten

| Declaration | Unwritten stance |
| --- | --- |
| A value you own: `value : MyType` | `copy` (its owner may write every stance) |
| The value an owning pointer points to: `MyType * unique` | `copy` |
| A local reference or raw pointer bound from one visible value | taken from that value; see the table below |
| The receiver `_` in a version with no receiver stance | `loan` |
| A `self` result | taken from the call; a `self` result cannot declare a stance |
| A **stand-alone borrower**: a reference or raw-pointer parameter, a reference result, a member, a type alias | **must be written** |

**No receiver stance is the same as `loan`.** Wherever a function has a
receiver, writing no receiver stance and writing `loan` are the same
declaration. That covers every function bound to an instance, including
operators and constructors. Declaring the same function both ways is a
duplicate, which is an error:

```zax
MyDocument :: type {
  describe final : (result : String)() = {
  }

  describe final : (result : String)() loan = {
  }
  // error: the same function as the one above
}
```

A function with no receiver at all, such as a free function, has no
receiver stance to write, so the rule does not apply. For constructors, this
is why A9 treats "no receiver stance" and `loan` as one row.

**Stand-alone borrowers must write their stance**, because there is no
single value to take it from. Neither default worked: `loan` would make
`MyValue &` differ from `MyValue`, and `copy` would make `MyValue &` reject
every value that forbids copying. Requiring it now can be relaxed later by
adding a default, and every program still compiles. Changing a default later
would silently change meaning.

```zax
show final : ()(value : MyValue &) = {                 // error: write a stance, for example MyValue loan &
}

show final : ()(value : MyValue readonly loan &) = {   // fine
}

current final : (result : MyDocument readonly loan &)() = {   // a reference result: stance required
}

printAll$ final : (T : type)()(items : T readonly loan &) = {  // generic: stance required
}

MyNode :: type {
  parent : MyNode loan *                               // a member: stance required
}
```

The accepted cost: nearly every read-only reference parameter is written
`readonly loan &`.

**A borrower bound from one visible value** takes a stance from it only
when that stance leaves the owner's value unchanged. A stance that would let
the borrower change the owner's value must be written by the owner at the
binding:

| Owner's stance | An unmarked borrower bound to it gets | To let the borrower change the owner's value |
| --- | --- | --- |
| `loan` | `loan` | not possible |
| `copy` | `copy` | write `as move` or `as last` |
| `deep` | `deep` | not possible |
| `move` | `loan` | write `as move` |
| `last` | `loan` | write `as last` |

The owner learns its variable's state from the stances it writes. If an
unmarked borrower could pick up `move` or `last`, a raw pointer carrying that
permission could be passed to a function that empties the owner, with no
`as` anywhere near the owner.

```zax
MyBuffer :: type transfer copy move {
  bytes : Byte[0..]
}

consume final : ()(buffer : MyBuffer) = {
}

owner : MyBuffer move
alias : MyBuffer & = owner                // owner is move, so alias gets loan
mover : MyBuffer move & = owner as move   // the owner grants move when binding

consume(alias as move)              // error: alias was granted loan
consume(mover as move)              // fine: owner's contents leave; owner is now moved-from
consume(mover as last)              // error: mover isn't the owner
consume(owner as last)              // fine: the owner of a move value may write as last

plain : MyBuffer
view : MyBuffer & = plain           // plain is copy, so view gets copy
consume(view)                       // fine: copies; plain is unchanged
consume(view as move)               // error: view isn't the owner
```

Loop bindings follow the same table:
`each item : MyValue readonly & from values` gets `copy` when the elements
are `copy`.

Inheriting a stance is a deliberate exception to today's rule that inferred
declarations take the type only. It applies only to borrowers.

#### A7. By-value parameters and `via`

A by-value parameter is a new value the function owns. It has two stances:

- **The parameter's stance** says what the function may build from the
  parameter inside its body, exactly like a local's stance. It may be
  `loan`, which pins the parameter for the length of the call.
- **The `via` stance** says the least stance the call must write. The
  default is `via copy`, which accepts any stance. `via loan` is invalid,
  because lending builds nothing.

**`via` is only for something that is built:** by-value parameters,
including pointer values passed by value, and by-value results (A8). A
reference parameter, a reference result, and the receiver are never built,
so `via` on them is an error. A reference's stance before `&` already
requires the call to write that stance, which is the job `via` does for a
by-value parameter:

```zax
take final : ()(item : MyItem copy via move) = {
}
// by value: the call must write as move or as last

hold final : ()(item : MyItem move &) = {
}
// reference: move & already requires as move; there is no via

broken final : ()(item : MyItem loan & via move) = {
}
// error: a reference isn't built, so via has nothing to constrain
```

The stance the call writes still chooses which constructor builds the
parameter, with the usual fallback (A10).

```zax
myType1 : MyType copy
myType2 : MyType move
myType3 : MyType last

func1 final : ()(value : MyType copy via move) = {
  passByValue(value)              // fine: the function owns a copy value
  passByValue(value as last)      // fine
}

func2 final : ()(value : MyType move via copy) = {
  passByValue(value)              // error: the parameter's stance is move
  passByValue(value as move)      // fine
}

func3 final : ()(value : MyType copy via last) = {
}

func1(myType1)              // error: via move needs as move or as last
func1(myType1 as move)      // fine
func1(myType2)              // error: needs move, and myType2 forbids copy
func1(myType3 as last)      // fine

func2(myType1)              // fine: copied into place
func2(myType2)              // error: myType2 forbids copy
func2(myType2 as move)      // fine
func2(myType3 as last)      // fine

func3(myType3)              // error: no stance means copy, which via last and myType3 both refuse
func3(myType2 as last)      // fine
```

Three declarations decide such a call:

- the parameter's `via` stance says which stances the call must write;
- the stance declared on the caller's variable says which stances the call
  is allowed to write;
- the parameter's own stance says what the function may build from the
  parameter inside its body.

**Stances describe places, not lineage.** `func3(myType2 as last)` is fine
even though `myType2` was declared `move` and `func3`'s body may copy.
`myType2`'s stance restricted that variable. Once its owner hands the
contents over, the value belongs to `func3`. A restriction that must follow
every value of a kind, wherever it ends up, belongs in the type's or
identity's `transfer` list.

**Uses for `via`:**

```zax
MyArchive :: type transfer copy deep {
  history : MyHistory * strong      // copy shares history; deep makes its own
}

sendToWorker final : ()(archive : MyArchive via deep) = {
  // archive shares nothing with the caller, so it may cross threads
}

sendToWorker(local)                 // error: via deep needs as deep
sendToWorker(local as deep)         // fine

push final : ()(item : MyBigBuffer via move) = {
  // refuses silent copies of an expensive value
}

registerDevice final : ()(device : MyDevice loan) = {
  configure(device)                 // fine if configure takes MyDevice loan &
  devices.add(device)               // error if add takes MyDevice by value: device is pinned
}
```

**Overloading on `via` is allowed.** The call uses the most specific `via`
it satisfies: a call written `as last` prefers `via last` over `via copy`,
and a call with no stance uses `via copy`.

**A borrowing pointer parameter's `= @` default borrows from a temporary
owner (2026-11-22).** `value : MyValue * = @` is an error, because `@` can't
go into a borrowing pointer. The meaning it had (if the caller omits the
argument, allocate one that lives through the call, and free it afterward)
needs no new rule:

```zax
useValue final : ()(value : MyValue loan * = (: MyValue * last = @).) = {
}
// the temporary owner lives through the call and frees the allocation afterward

useMade final : ()(value : MyValue loan * = makeValue().) = {
}
// the same with a function's owning result
```

A temporary lives through the call it is part of, and making a pointer from
`p.` copies the address (A11). Making the parameter owning instead
(`value : MyValue * last = @`) was rejected: it would quietly turn a
borrowing parameter into an owning one.

**`via` is only needed where one author writes the stance and another
declares the destination.** A lambda capture (`[[ source as last: captured ]]`),
a loop source (`each … in source as last`), and a label mapping each write
both in one place, so they need no `via`. A lambda's own parameters are
ordinary parameters.

**The keyword.** Composition also uses `via`, as in `= via fuel.refill`,
always after `=` in a function definition. The parameter use always follows
a type, so readers can tell them apart. `using` was considered and rejected:
`using (makeDocument()) { … }` already means keeping a value alive and
disposing of it at the end of a block, so a parameter written
`copy using move` could be read as enrolled in that disposal. It also reads
as if the body uses `move`, which is the very meaning `via` exists to keep
separate. `MyItem copy via move` reads as "a `copy` item that arrives via a
move".

#### A8. Results

A result is a value the function owns and fills in. When the caller uses the
result to start a new variable, the two are the same value. So **a result
has one stance, with the same meaning inside the function and for the
caller.**

```zax
makeMessage final : (result : MyMessage move)() = {
  result.+++(...)          // fill it in
  cache.store(result)      // error: result is move, so nothing may be copied from it
}

makeFree final : (result : MyMessage)() = {
  result.+++(...)
  cache.store(result)      // fine: an unmarked result is copy
}
```

**`via` on a result** names the least stance the function's own `return`
must write when it builds `result`. That is the same meaning as on a
parameter, where the call builds the value. The function may also build
`result` directly with `result.+++()`.

```zax
giveMeResult final : (result : MyType loan via copy)() = {
  builtUpValue : MyType move
  doStuff(builtUpValue)
  return builtUpValue as last    // via copy accepts last
}

myResult : MyType loan = giveMeResult()   // the same value: nothing is built from result

makeBig final : (result : MyBigBuffer via move)() = {
  return cached                  // error: no stance means copy; the return must write as move or as last
  return cached as move          // fine
}
```

**The caller's variable may declare its own stance**, because stances
describe places, not lineage. The only exception is a `loan` result: nothing
may be built from it, so the caller's variable must be that same value,
declared `loan`. For a `loan` result, building it straight into the
caller's variable becomes mandatory; today it is optional.

```zax
stored : MyMessage = makeMessage()           // fine: stored is an ordinary copy variable
pinned : MyMessage loan = makeMessage()      // fine
kept : MyType = giveMeResult()               // error: nothing may be built from a loan result
```

**A result the caller doesn't name is a temporary, and every temporary is
treated as written `as last`.** That is today's rule for unnamed values; it
comes from the result being a temporary, not from anything about results.
The implicit stance is still limited by the result's own stance:

| Result's stance | The unnamed result is treated as written |
| --- | --- |
| `copy` (or none), `move`, `last` | `as last` |
| `deep` | `as deep` |
| `loan` | nothing can be built; it can only be received as the same value |

This still matters even though new variables are built in place:

```zax
makePayload final : (result : MyPayload)() = {
}

existing : MyPayload
existing = makePayload()          // assignment: as last lets the move or last assignment run

consume final : ()(input : MyPayload loan &) = {
}

consume final : ()(input : MyPayload last &) = {
}

consume(makePayload())            // as last chooses the last & version

convert : MyOtherType = makePayload()   // a converting constructor may take the payload's resources
```

A caller may still write a stance on the call: `makePayload() as copy` is
fine for a `copy` result and an error for a `move` result.

`except` results follow the same rules as ordinary results. A reference
`except` result is a stand-alone borrower and writes its stance, as in
`failure except : MyError readonly loan &`.

**A compatible prototype may narrow a result's stance, never widen it, and
never change it to or from `loan` (2026-11-22).** A visible prototype that
reuses a minted body:

- may narrow: callers may do less with a value the body built for them;
- may not widen: turning a `MyValue * last` result into `MyValue * copy`
  would make an owning pointer copyable, so the allocation could be freed
  twice;
- may not change to or from `loan`: a `loan` result must be built directly
  into the caller's variable, and the prototype can't change how the body
  builds it.

This also removes the old `peekValue` use, which widened `last` to `copy`.

#### A9. Constructor receiver stances

**The stance declared on a new value is the receiver stance given to its
constructor**, just as `as move` written on a call is the receiver stance
given to a function:

```zax
myValue : MyValue loan    // the constructor's receiver stance is loan
myValue : MyValue copy    // the constructor's receiver stance is copy
myValue : MyValue move    // the constructor's receiver stance is move
myValue : MyValue last    // the constructor's receiver stance is last
```

A constructor may declare a receiver stance of its own. It is a requirement
on the value it builds: "nothing may be built from the new value except
what I allow". So the declared stance of the new value decides which
constructors may run:

| The new value is declared | Constructors that may build it, identified by their declared receiver stance |
| --- | --- |
| `value : MyValue loan` | any |
| `value : MyValue copy` | none (or `loan`), `copy` |
| `value : MyValue move` | none (or `loan`), `copy`, `move` |
| `value : MyValue last` | none (or `loan`), `copy`, `move`, `last` |
| `value : MyValue deep` | none (or `loan`), `copy`, `deep` |

(the source is chosen separately, by the written stance)

The new value keeps the stance it was declared with. The table only says
which constructors may be chosen to build it. A constructor whose receiver
stance is `move` needs "this value can never be copied". A value declared
`move` or `last` keeps that promise, and so does a pinned `loan` value,
which can't even be moved. A value declared `copy` breaks it.

Two consequences:

- **A `loan` value can be built by any constructor.** Nothing may ever be
  built from it, so it meets every requirement.
- **A `copy` receiver stance adds no requirement.** `copy` allows every
  stance, so a constructor declared `copy` behaves like one with no receiver
  stance.

This lets a constructor refuse to build a value whose stance would let it be
copied later. Built-in raw pointers follow what their constructors would
declare, with one exception. Their owner may not write `as move` or
`as last` on a raw `copy` or `loan` pointer, a rule constructors can't
express (A11):

```zax
// illustrative: a raw pointer modelled as an ordinary type
MyRawPointer :: type {
  +++ final : ()(source : MyRawPointer copy &) = {
    // copies an address: builds a borrower; no requirement on the new pointer
  }

  +++ final : ()(source : MyRawPointer last &) move = {
    // takes over responsibility for freeing the allocation;
    // needs: the new pointer can never be copied
  }

  +++ final : ()(source : MyRawPointer last &) = forbidden
  // taking over the allocation with no requirement on the new pointer is forbidden
}

p1 : MyValue * last = @
p2 : MyValue * move = @
p3 : MyValue * loan = @
p4 : MyValue * copy = @
```

`@` is a value whose stance is `last` (A11), so the borrower constructor,
whose source is `copy &`, can never take it (A3). The other two compete:

| Line | Candidates that may build it | Chosen | Result |
| --- | --- | --- | --- |
| `p1` | `move` receiver; forbidden one | `move` receiver: closer to `last` | fine |
| `p2` | `move` receiver; forbidden one | `move` receiver: exact | fine |
| `p3` | `move` receiver; forbidden one | forbidden one: exact for `loan` | error |
| `p4` | forbidden one only | forbidden one | error |

"No receiver stance" and `loan` are the same in the table above, so the one
forbidden shape covers `loan` pointers. A `loan` raw pointer is a pinned
borrower, so it must never receive a responsibility. If it could, the stance
would no longer tell the compiler whether to free anything, and it would have
to track how each pointer was initialized. The same pattern applies to a
`move &` source, for ownership handed on with `as move`.

**Generated constructors and receiver stances.** Today's generation rules
stay as they are:

- only `copy` is generated automatically, as a fixed set of shapes, and only
  when something uses them;
- a declaration replaces a generated shape only when it is exactly that shape.

Two points are added:

- **Generated constructors never have a receiver stance.** The compiler
  generates only constructors whose source is `copy` and that declare no
  receiver stance, so they may build a value declared with any stance. No
  receiver stance is the same as `loan` (A6). So a declared constructor with
  a `copy` source and an explicit `loan` receiver stance is exactly the
  generated shape: it replaces the generated constructor rather than
  competing with it.

  This also means writing `loan` on a constructor does not by itself limit it
  to pinned values: `loan` means no requirement, not "pinned only". To allow
  only pinned values, a type keeps the no-receiver-stance constructor and
  forbids the other receiver stances (`copy = forbidden` covers `copy`, `move`,
  and `last`; `deep = forbidden` covers `deep`). For the receiver measure in
  the selection rule, "no receiver stance" is the exact match for a value
  declared `loan`.
- **The receiver stance is part of a constructor's shape.** A constructor
  declared with a receiver stance is never the same shape as a generated one.
  It doesn't replace the generated constructor; it is a more specific
  candidate that wins selection for the values its receiver stance targets.

| You declare | Effect on the generated shape |
| --- | --- |
| exactly a generated shape (same source, qualifiers, and stance, no receiver stance) | replaces it: your body, `= default`, `= existing`, or `= forbidden` |
| the same shape plus a receiver stance | nothing: both exist and compete in selection |
| a different source, qualifiers, or stance | nothing: both exist; an unresolved conflict at a use asks you to resolve it |

```zax
MyBuffer :: type transfer copy move {
  bytes : Byte[0..]

  +++ final : ()(source : MyBuffer readonly copy &) move = {
    // copies into a value that can never be copied again
  }
}

source : MyBuffer
a : MyBuffer = source          // a copy value: the declared constructor can't build it,
                               // so the generated copy constructor runs
b : MyBuffer move = source     // a move value: both may build it; the declared one is exact
                               // on the receiver, so it wins
```

**Denials don't need to suppress generation; they win selection.** A
forbidden shape with a receiver stance is chosen, and fails, exactly where it
is the better match:

```zax
MyRecord :: type transfer copy {
  +++ final : ()(source : MyRecord readonly copy &) move = forbidden
  // copying into a move value is an error; ordinary copies still use the generated constructor
}

c : MyRecord = record          // fine: the generated copy constructor
d : MyRecord move = record     // error: the forbidden shape is exact on the receiver and wins
```

The raw pointer's `(source : MyRawPointer last &) = forbidden` works the same
way. `last` constructors are never generated, so there is nothing to
suppress; the forbidden shape exists only to win selection for `copy` and
`loan` destinations.

**Functions and constructors agree.** In both, an operation may take no
more than what was stated:

```zax
MyDocument :: type {
  publish final : ()() = {
  }

  publish final : ()() move = {
  }
}

doc : MyDocument
(doc as last).publish()       // the move version: a move takes no more than last (A2)

MyHandle :: type {
  +++ final : ()(source : MyHandle readonly copy &) move = {
    // needs: the new handle can never be copied
  }
}

h1 : MyHandle move = other    // this constructor runs: exact
h2 : MyHandle last = other    // this constructor runs too: a last handle can never be copied either
h3 : MyHandle = other         // it can't run: the generated copy constructor builds h3
```

**Where the stance comes from differs, for a reason.** For a function, the
receiver is an existing value that the caller owns. Choosing a `last`
version can finish it, so the caller must write the stance where a reader
sees it: `(foo as last).foo()`. For a constructor, the receiver is the new
value, which doesn't exist until the constructor runs. Nothing existing is
given up through it, and its declaration is the only place a stance can be
written. So the declaration supplies the receiver stance, with no `as`:

```zax
fresh : MyBuffer last = source as copy
// receiver stance last: from fresh's declaration, the only place it can be written
// source stance copy: written on source, because source is an existing value being used
```

The rule behind both: whatever could take from an existing value needs a
stance written where that value is used. A constructor's source still needs
`as last` for the constructor to take from it.

**Choosing a constructor uses two inputs.** Building a value from a source
of the same type picks a constructor by:

| What is declared or written | Picks the constructor's |
| --- | --- |
| the stance the call writes (at least the parameter's `via`) | source stance, with the usual fallback |
| the new value's declared stance | receiver stance, by the table above |

```zax
func1 final : ()(value : MyValue copy via last) = {
}
// direct match: +++ final : ()(source : MyValue readonly last &) copy

func2 final : ()(value : MyValue move via copy) = {
}
// direct match for func2(myValue as copy):
//   +++ final : ()(source : MyValue readonly copy &) move
```

**Selection: the receiver is just another input.** It is bound implicitly,
but it competes on equal terms with the parameters, for constructors and
ordinary functions alike. The compiler:

1. leaves out every candidate whose source can't accept the stance written
   on the source;
2. keeps every constructor whose receiver stance can't build the new value,
   counting it as the worst possible match on the receiver; if such a
   constructor is chosen, the line is an error;
3. chooses a candidate only if it is at least as good as every other on
   every input, and better on at least one;
4. otherwise reports the call as ambiguous.

(Corrected 2026-11-21. The earlier step 1, "removes every candidate that
isn't compatible", contradicted A10's rule that a constructor which exists
but refuses the destination makes the line an error: `loose : MyToken =
makeToken()` would have quietly run the generated copy constructor. Keeping
the refusing constructor as the worst receiver match reproduces every
example in A9 to A11 as written, and turns the `MyToken` case into an
ambiguity error that names the refusing constructor.)

**How "better" is measured on the receiver.** An exact receiver match is
best. Otherwise, a receiver stance closer to the declared stance along
`loan`, `copy`, `move`, `last` is better. `deep` matches only exactly, or
through "no receiver stance". This is the measure for the receiver input
only; it gives the receiver no priority over the other inputs. Without it,
`p1` in the pointer example would be ambiguous.

This is the rule Zax already uses when one overload matches the type
exactly and another matches the stance exactly. No input takes priority,
because any fixed priority would silently run an operation the type's
author may not have intended.

```zax
MyValue :: type {
  +++ final : ()(source : MyValue mutable writable move &) = {
  }
  // A: exact source for as move; no receiver stance

  +++ final : ()(source : MyValue readonly copy &) move = {
  }
  // B: source reached only by fallback; exact receiver stance
}

func2(myValue as move)   // error: ambiguous; A is better on the source, B on the receiver
```

The fix is a candidate that is exact on both inputs, which then wins
outright:

```zax
  +++ final : ()(source : MyValue mutable writable move &) move = {
  }
  // with this added, func2(myValue as move) uses it
```

**Building a value declared `deep` from an ordinary copy.** A value's
declared stance governs what is built *from* it later, not how it was built.
So `shared : MyArchive deep = original` is written as `copy`. The type
decides what happens, through two different kinds of `deep` constructor:

- a `deep` **source** constructor (`source : MyArchive readonly deep &`)
  runs when the initializer writes `as deep`;
- a `deep` **receiver** constructor (`… copy &) deep`) runs when a value
  declared `deep` is built from an ordinary copy.

```zax
MyArchive :: type transfer copy deep {
  history : MyHistory * strong

  +++ final : ()(source : MyArchive readonly copy &) = {
    // ordinary copy: shares history
  }

  +++ final : ()(source : MyArchive readonly deep &) = {
    // runs for as deep
  }

  +++ final : ()(source : MyArchive readonly copy &) deep = {
    // runs when a deep-declared value is built from a copy: makes its own history
  }
}

shared : MyArchive deep = original
// written as copy. The first and third constructors are both exact on the source,
// and the third is also exact on the receiver, so the third wins: shared is independent

independent : MyArchive deep = original as deep   // the deep source constructor runs
```

Without a `deep`-receiver constructor, this is not an error. The ordinary
copy constructor declares no receiver stance, so it may build a value
declared `deep`, and `shared` then shares `history` with `original`. That
is the type author's choice. No special rule or warning is needed at the
declaration.

**Temporaries.** A temporary plays one of three roles when an operation is
chosen. Using an ordinary type:

```zax
MyBuffer :: type transfer copy move last {
  bytes : Byte[0..]

  +++ final : ()(source : MyBuffer readonly copy &) = {
  }

  +++ final : ()(source : MyBuffer mutable writable move &) = {
  }

  +++ final : ()(source : MyBuffer mutable writable last &) = {
  }
}

makeBuffer final : (result : MyBuffer)() = {
  scratch : MyBuffer
  fill(scratch)
  return scratch as last
}
```

| The temporary is | Its role | The stance involved |
| --- | --- | --- |
| being built | receiver | its own declared stance (unmarked: `copy`) |
| the source for a new value | source | written as the strongest stance its stance allows |
| the value a function is called on | receiver of that call | written as the strongest stance its stance allows |

1. **Being built.** `return scratch as last` builds `result`: the written
   stance `last` picks the `last` constructor, and `result`'s declared
   stance, unmarked `copy`, is the receiver stance.
2. **The source for a new value.** The result is written `as last`. The new
   value's declared stance is the receiver stance. Elision may skip running
   the chosen constructor, but the choice must still succeed:

   ```zax
   a : MyBuffer = makeBuffer()           // the last constructor is chosen; elision skips running it
   b : MyBuffer move = makeBuffer()      // the same; every constructor here accepts a move value
   other : MyOtherBuffer = makeBuffer()  // no elision: a converting constructor taking MyBuffer last & runs
   existing = makeBuffer()               // assignment: MyBuffer's last assignment runs
   ```

   With the pointer constructors above, the same rule makes
   `c : MyValue * copy = makeOwner()` an error even when elided, because the
   taking-over constructor can't build a `copy` pointer.
3. **The value a function is called on.** `makeBuffer().append(moreBytes)`
   writes the temporary `as last` on the receiver, so `append`'s `last`
   version is used if there is one, otherwise the version with no receiver
   stance. The temporary is destroyed at the end of the statement.

**Every temporary's stance is declared somewhere, or is the unmarked
`copy`.** A function or operator result takes the result's declaration. An
expression such as `(: MyValue)` takes its own, so `5 + (: MyValue)` builds a
`copy` operand. A lambda expression is unmarked. A temporary built by an
implicit conversion to bind a reference, such as a `MyPacket` made from a
`MyMessage`, is a value the call expression owns with no stance written, so
it is `copy`. A stance carried by an alias or generic, as in `(: T)` with
`T` an alias declared `move`, is declared, but not visible at that line. No
temporary's stance is ever guessed.

**The stance and the `transfer` list are independent filters.** The stance
says which stances may be written; the `transfer` list says which operations
exist. An unmarked value of a type that allows only `last` is still `copy`,
and only `as last` actually works, which is exactly what a temporary is
written with:

```zax
MyTicket :: type transfer last {
  id : Integer
}

makeTicket final : (result : MyTicket)() = {
}

keep final : ()(ticket : MyTicket) = {
}

keep(makeTicket())        // fine: the temporary is written as last, and the last operation runs

ticket : MyTicket         // unmarked: copy
keep(ticket)              // error: written as copy, and MyTicket has no copy
keep(ticket as move)      // error: MyTicket has no move, and move can't fall back upward to last
keep(ticket as last)      // fine
```

The unmarked `copy` never claims that copying works. It means "this variable
adds no restriction of its own"; the type decides the rest.

#### A10. Fallback and forbidden operations

**Leaving a stance out of a type's `transfer` list means "not defined".** A
type can't define an operation for an unlisted stance. Callers may still
write that stance, and fallback passes over the missing operation to a
weaker one that exists (`last` to `move` to `copy`; `deep` never falls
back):

```zax
MyPoint :: type transfer copy {
  x : Integer
}

point : MyPoint move
moved : MyPoint = point as move    // fine: MyPoint has no move, so copy runs
```

**Fallback passes over operations that don't exist, never over ones that
do.** An operation that exists but refuses the destination, through its
receiver stance or `= forbidden`, makes the line an error. For raw pointers:

```zax
valueLastPtr : MyValue * last = @
borrow : MyValue * copy = valueLastPtr as last
// error: valueLastPtr's taking-over constructor needs a move or last destination.
// To borrow the value it points to, write valueLastPtr.
```

Without this rule, `borrow` would build a borrower *and* finish
`valueLastPtr`, because the call wrote `as last`. With `valueMovePtr as move`,
the owner would be tracked as possibly empty although nothing moved. Both
are confusing, so they are errors.

**Forbidden declarations take part in selection exactly like any other
candidate.** A forbidden declaration blocks every selection it would win,
not only its own exact shape:

| Where the forbidden candidate stands | Result |
| --- | --- |
| it is clearly the best candidate | it is chosen, and the call is an error |
| another candidate is clearly better | the better one is chosen; the forbidden one doesn't matter |
| it ties with another candidate (each is better on a different input) | the call is ambiguous: an error that names the forbidden candidate |
| fallback reaches it | fallback stops there and goes no further |

The table works the same way on both inputs, the receiver and the source.

*On the receiver.* The value is declared `last`; the `move`-receiver
constructor is forbidden:

```zax
MyValue :: type transfer copy move last {
  +++ final : ()(source : MyValue readonly copy &) = {
  }
  // no receiver stance

  +++ final : ()(source : MyValue readonly copy &) move = forbidden
}

a : MyValue last = other   // error: both may build a last value; the forbidden one is closer
                           // to last than "no receiver stance", so it is chosen
b : MyValue = other        // fine: a move constructor can't build a copy value, so only
                           // the one with no receiver stance qualifies
c : MyValue loan = other   // fine: "no receiver stance" is the exact match for loan
```

*On the source.* `as last` is written, there is no `last` source
constructor, and the `move` source constructor is forbidden:

```zax
MyValue :: type transfer copy move last {
  +++ final : ()(source : MyValue mutable writable move &) = forbidden
}

x : MyValue = other as last   // error: last falls back to move, which exists and is forbidden;
                              // fallback stops there and does not continue to copy
y : MyValue = other           // fine: the generated copy constructor
```

*A tie stays an error.* A forbidden candidate is never removed before the
comparison, so it can't hand a tie to the other candidate:

```zax
MyValue :: type transfer copy move {
  +++ final : ()(source : MyValue mutable writable move &) = {
  }
  // A: exact on the source for as move; no receiver stance

  +++ final : ()(source : MyValue readonly copy &) move = forbidden
  // B: source reached only by fallback; exact on the receiver for a move value
}

v : MyValue move = other as move
// error: ambiguous. A is better on the source, B on the receiver.
// The compiler does not pick A just because B is forbidden
```

If a tied forbidden candidate quietly dropped out, forbidding something would
change which other constructor runs, and the author would never learn that
the two declarations overlap. The fix is a declaration that is exact on both
inputs, or removing one of the two.

*Forbidding only one stance.* To forbid `move` values while letting `last`
values be built normally, add an exact match for `last`. It then wins
instead of the forbidden one:

```zax
+++ final : ()(source : MyValue readonly copy &) last = existing
// a last value is built exactly like the version with no receiver stance
```

With that added, `a : MyValue last = other` works, and only values declared
`move` reach the forbidden constructor.

**Bodyless declarations are retired (2026-11-21).** The earlier design had
a fifth declaration form, a lifecycle or operator signature with no body,
meant to reserve a shape for "composition or later completion". Nothing in
the language can complete it: composition uses `abstract` and `fulfill`,
partials can't supply the body of an existing declaration, and callable
`forward` requirements are later work. Without completion it only blocked
selection, which `= forbidden` does, and it carried one extra rule that
conflicted with the table above: a better generated shape next to a bodyless
declaration was an error rather than running. A forgotten body silently
became one. So:

- a lifecycle or operator declaration ends in a body, `= default`,
  `= existing`, or `= forbidden`; ending in none of them is an error;
- the "declare now, implement later" idea, with its no-silent-neighbor
  generation rule and example, is captured in
  `project/raw/callable-selection.md` for a future completion feature.

**Leaving a stance out is all it means.** The operation doesn't exist, and
the fallback rules apply as normal. A type can't write any signature for an
unlisted stance, not even `= forbidden`. `= forbidden` works only on
signatures of listed stances, where it is a deliberate, narrow choice about
an operation that otherwise exists:

```zax
MyPoint :: type transfer copy {
  x : Integer

  +++ final : ()(source : MyPoint mutable writable & move) = forbidden
  // error: move is not in MyPoint's transfer list, so move doesn't exist and
  // as move falls back to copy. A type can't forbid an operation it doesn't define.
}

MyMixed :: type transfer copy move {
  +++ final : ()(source : MyMixed readonly & copy) = forbidden
  // fine: copy is listed; copying from a readonly source is removed
}
```

The error message should explain this, because a programmer who tries it
believes they are making the type stricter.

**A declared stance that can never be used is an error at the declaration**
(today's rule). For a type with a `transfer` list, no later code can add a
missing operation, so the compiler reports the declaration itself:

```zax
MyDeepOnly :: type transfer deep {
}

shared : MyDeepOnly move   // error: the owner may write only as move or as last,
                           // and MyDeepOnly has neither
other : MyDeepOnly         // fine: copy includes deep, so other as deep works
```

Today's documents use `shared : MyDeepOnly copy` as the example of this
error. That held only under 038's rule, where `deep` was outside `copy`.
Under A1, `copy` includes `deep`, so that declaration is satisfiable.

An identity that leaves out a stance its underlying type defines follows the
same rule: the stance doesn't exist for the identity, and fallback applies.

Making a stance's use an error instead of a fallback (`forbid`) is deferred
(part C).

#### A11. Pointers

A pointer involves two values: the pointer, and the value it points to. Each
has its own stance, in its own layer's position:

```zax
view : MyValue loan * copy      // nothing may be built from the value;
                                // the pointer itself may be copied
```

**A pointer is an owner of what it points to when it holds something
besides an address:**

| Pointer | Holds besides the address | May the pointer itself be written `as move` or `as last`? | Owner of what it points to? |
| --- | --- | --- | --- |
| raw `copy` | nothing | no; only `as copy`, or lending | no |
| raw `loan` | nothing | no; only lending (a pinned borrower) | no |
| raw `move` or `last` | responsibility for freeing the allocation | yes | yes |
| `unique` | sole ownership | yes | yes |
| `strong` | a share of ownership | yes: hands over that share | yes |
| `weak` | a weak registration | yes: hands over the registration | only after acquiring a `strong` pointer |

```zax
borrowed : MyValue * copy = myValue           // a borrower
promoted : MyValue * move = borrowed as move  // error: borrowed holds nothing to hand over
```

**Allocation.** `@` produces a value whose stance is `last`: only a terminal
transfer may be built from it. **A raw pointer holds a responsibility exactly
when its stance is `move` or `last`**, so only such a pointer, or an owning
kind, can receive `@`. The compiler frees a raw pointer exactly when its
stance is `move` or `last` and it is not empty, judged by stance alone with no
tracking. This follows from the forbidden constructor in A9, not from a
special pointer rule.

```zax
owner : MyValue * last = @           // fine
plain : MyValue * = @                // error: a raw copy pointer can't hold a responsibility
held : MyValue * loan = @            // error: a raw loan pointer can't hold one either
stray : MyValue * move = myValue     // error: myValue is not an allocation this pointer can free
taken : MyValue * move = owner as last   // fine: responsibility moves to taken
```

**Every allocation has an owner.** The allocation forms are `@`, which
panics on failure, and `@!`, which gives a vacant pointer on failure. The
open-ended forms `@<` and `@!<` are removed, together with the "open-ended"
state and its special rules. Assigning `@` to an undecorated raw pointer,
which today makes an open-ended allocation, is an error:

```zax
ordinary : MyValue *
ordinary = @                  // error: a raw copy pointer can't hold a responsibility;
                              // declare it MyValue * last
```

What the open-ended forms did is done this way:

| Need | How |
| --- | --- |
| free an allocation early, by hand | `owner : MyValue * last = @`, then `reset owner`; the automatic cleanup later finds it vacant |
| hand an allocation to foreign code that frees it | the foreign function's Zax declaration takes `MyValue * last`, and the call writes `as last` |
| deliberately abandon an allocation without freeing it | `unsafe vacate owner` |
| a result the caller manages | `result : MyValue * last = @`; the caller owns it and may `reset` it whenever it likes |

**Ownership passes between owner kinds without `unsafe`.** An owned
allocation may move into a new raw `move` or `last` pointer, just as a raw
owner may already become `unique` with `as last`:

```zax
unique1 : MyValue * unique = @
raw1 : MyValue * last = unique1 as last      // fine: raw1 now owns and frees it; unique1 is empty

shared1 : MyValue * strong = @
raw2 : MyValue * last = shared1 as last      // succeeds only if shared1 is the only owner, no
                                             // weak pointer remains, and shared1 targets the
                                             // allocation root; otherwise raw2 is empty

borrow : MyValue * = unique1 as last         // error: a raw copy pointer can't take ownership
```

Once the value is no longer `unique`, its "no raw borrowers" guarantee ends,
and `raw1` may hand out raw borrowers.

**An owning raw pointer can never point inside an allocation.** Like a
blockless `unique`, an owning raw pointer holds only one address and no
control block, so it can only free an allocation if it points at the
allocation's start:

- claiming a raw `move` or `last` pointer from an interior `strong` or
  `unique shareable` pointer produces a vacant result (or a compile error
  when statically proved interior), matching a failed `unique` claim;
- applying `unsafe cast as last` to an interior address is the programmer's
  unsafe assertion.

**`reset` and `vacate` on raw pointers:**

- **`reset` frees, so it is an error on a raw `copy` or `loan` pointer,**
  which holds nothing to free. On a raw `move` or `last` owner, `reset`
  applies the allocation's disposition and leaves the pointer vacant.
- **`vacate` on a raw `copy` or `loan` pointer is always safe,** with no
  `unsafe`: a borrower holds no responsibility to lose.
- **`unsafe vacate` on a raw `move` or `last` owner** drops responsibility
  without freeing (the first half of the vacate-then-restore sequence below,
  or deliberate abandonment). `unsafe vacate` stays an error on `unique`,
  `unique shareable`, `strong`, and `weak`.
- **To free an allocation reached only through a borrower,** restore
  ownership through the one escape (`(view unsafe cast as last) as last`)
  and `reset` the owning pointer or let it go out of scope. There is no
  separate `unsafe reset view` route.

**The one built-in raw-pointer rule.** The owner of a raw `copy` or `loan`
pointer may not write `as move` or `as last` on the pointer itself, because
it holds nothing to hand over. This is the one exception to A2's owner rule,
and it exists only for raw pointers.

Two different rules are involved, and only the second is special:

- **Any borrower** may write only the stance it was granted on the value it
  *reaches* (A2). This applies to the value behind a reference, behind a raw
  pointer, or `_`:

  ```zax
  view : MyMessage copy * = message
  keep(view. as move)            // error: view only borrows the message (general rule)
  ```

- **A raw `copy` or `loan` pointer** is a value you own. Normally the owner
  of a `copy` value may write `as move` or `as last` (A2). For the pointer
  itself, it may not:

  ```zax
  copyPtr : MyMessage * = message
  other : MyMessage * last = copyPtr as last   // error: copyPtr is yours, but it holds nothing to hand over

  count : Integer
  total := count as last         // fine: you own count; Integer has no last, so it falls back to copy
  ```

**Why constructors can't express this rule.** Constructor selection sees
only the stance written at the call, not how the source variable was
declared. On a reference parameter, the stance before `&` is what was
granted (A12). So these two lines look identical to every constructor:

```zax
copyPtr : MyValue * = myValue        // a borrower: declared copy
lastPtr : MyValue * last = @         // an owner: declared last

a : MyValue * last = copyPtr as last     // the constructor sees: a MyValue * written as last
b : MyValue * last = lastPtr as last     // the constructor sees: a MyValue * written as last
```

Any constructor shape that matched `a`, forbidden or not, would also match
`b`. Telling them apart would need a second stance on every reference
(rejected: one word per layer), or making a raw pointer's stance part of its
type (just as special, and less visible). The built-in rule is the simplest
to state and to diagnose.

Raw pointers need this because their type has a constructor that treats a
`last` source as "hand over responsibility for freeing", while whether a
given raw pointer actually holds that responsibility depends only on its
declared stance. For most types, writing `as last` on a value you own is
harmless: the type has a `last` operation, or fallback runs a weaker one.

**A limitation, not a pattern to copy.** A user-defined type whose ownership
depended on its declared stance rather than its type could not express this
rule. Such a design should be two types.

**`unsafe cast as last` is the escape from this rule.** It changes only the
stance (A14), and writing it asserts what the compiler can't check: the
pointer identifies an allocation root, and nothing else holds
responsibility for it. This lets an owner create a borrower, give up
ownership, and later have that ownership taken back through the borrower:

```zax
owner : MyValue * last = @
view : MyValue * = owner.                    // a borrower
unsafe vacate owner                          // owner gives up responsibility without freeing
restored : MyValue * last = (view unsafe cast as last) as last
// the cast gives view itself with the stance last; writing as last hands it
// over, so restored takes over ownership and will free the allocation.
// view counts as finished afterward, so it can't be used by accident as a
// borrower of an allocation someone now owns
```

If the specification later defines cases the compiler can prove, those
cases can stop requiring `unsafe` without breaking any program (part D).

**Building a raw `copy` or `loan` pointer**, which holds only an address:

```zax
movePtr : MyValue * move = @
lastPtr : MyValue * last = @
copyPtr : MyValue * copy = movePtr.     // a plain borrower
pinned  : MyValue * loan = copyPtr      // a pinned borrower

a : MyValue * loan = copyPtr            // fine: copies a borrower's address
b : MyValue * loan = pinned             // error: nothing may be built from a loan pointer
c : MyValue * loan = pinned.            // fine: borrows the value pinned points to
d : MyValue * loan = movePtr            // error: movePtr is move, so it can't be copied
e : MyValue * loan = movePtr as move    // error: hands ownership to a pointer that can't hold it
f : MyValue * loan = movePtr.           // fine: a pinned, non-owning borrower; movePtr is untouched
g : MyValue * loan = lastPtr.           // fine: the same for a last owner
h : MyValue * loan = lastPtr as last    // error: hands ownership to a pointer that can't hold it
i : MyValue * loan = @                  // error: the same
```

| Source | Into a raw `copy` or `loan` pointer |
| --- | --- |
| a raw `copy` pointer | fine: copies the address |
| a raw `loan` pointer | error; write `pinned.` |
| a `move` or `last` owner, no stance written | error; write `owner.` |
| a `move` or `last` owner, written `as move` or `as last` | error: forbidden hand-off to a pointer that holds no responsibility |
| `@` | error: the same |
| `x.` from any pointer, or a value you can name | fine: borrows the value |

The borrower is made from `movePtr.` and never from `movePtr as move`.
Writing `as move` would make the compiler treat `movePtr` as possibly empty
afterward, when nothing could have moved.

**Owners of what a pointer points to** may write what A2 allows for that
value's stance. That includes `strong` pointers: another owner of the same
allocation may then see a moved-from or finished value, which is accepted.
A pinned value stays pinned even under an owning pointer, so the pointer can
move while the value never does:

```zax
func1 final : ()(value : MyValue copy * unique) = {
  callByValueMove(value. as move)    // fine: owner
}

install final : ()(device : MyDevice loan * unique) = {
  registry.adopt(device as move)     // fine: ownership of the pointer moves
  relocate(device. as move)          // error: the device is pinned
}
```

**A borrowing pointer's stance is set when it is formed.** Its holder isn't
the owner, so later it can only narrow:

```zax
myValue : MyValue
view : MyValue copy * = myValue            // formed from the owner
lent : MyValue loan * = view               // fine: narrows
taker : MyValue move * = view              // error: view can't write as move
taker2 : MyValue move * = myValue as move  // fine: the owner grants move when forming it
```

**Borrowing the value an owning pointer points to.** Write `p.`. That names
the value, not the pointer, so the new borrower is formed from the value,
just as from a local. Nothing is built from the pointer.

Making a pointer from `p.` copies `p`'s address as it is:

- if `p` points at a value, the new pointer points at the same value;
- if `p` is vacant (points at nothing), the new pointer is vacant too.

This matters because dereferencing is unchecked in Zax. `p.` on a vacant
pointer reaches the type's Nothing instance, a stand-in value. Without this
rule, the new pointer could point at that stand-in and look like a real,
non-vacant pointer, and under a trapping Nothing policy merely writing `p.`
might trap. So making a pointer from `p.` is not a dereference: it touches no
memory, and a trapping Nothing policy cannot fire. In normal code you would
test `?p` first; the rule only makes the unchecked form behave sensibly.

```zax
owner : MyType * last = @
view : MyType * = owner.        // points at the same value owner points at

reset owner                     // owner is now vacant
borrowed : MyType * = owner.    // vacant too: not a pointer to the Nothing instance; no access, no trap
use(owner.x)                    // a member read: traps under a trapping Nothing policy, as today
```

**`unique` allows no raw borrowers.** Nothing unchecked may point at a
`unique`'s value or into its allocation. References and slices are checked
by lifetime analysis, so they are allowed:

```zax
owner : MyValue * unique = @
view : MyValue readonly loan & = owner.   // fine: checked
raw : MyValue * = owner.                  // error outside unsafe
```

That is what separates `unique` from an owning raw pointer: both own their
value alone, but only the raw owner allows raw borrowers, and keeping those
valid is the programmer's job.

**Making a raw pointer from a reference whose origin the function can't see
needs `unsafe`.** This covers a reference parameter, a reference result, and
`_`. The function can't know whether the reference reaches a `unique`'s
value:

```zax
makePtrFromRef final : (result : MyValue loan *)(input : MyValue readonly loan &) = {
  return input        // error outside unsafe
}
```

Raw pointers from visible values stay allowed: a local, a member of a local,
or the value behind a raw owner.

**A pointer made from an integer is a borrower (2026-11-22).** An address
carries no responsibility for freeing anything, so converting an integer to
a pointer gives a borrowing raw pointer. Taking ownership of the allocation
at that address also needs the stance cast (A14):

```zax
// candidate syntax
myAddress : UPointer = myPointer as UPointer
view : MyType * = myAddress unsafe cast MyType *                   // a borrowing raw pointer
owned : MyType * last = myAddress unsafe cast MyType * last        // error: a type cast can't change a stance
owned : MyType * last = (view unsafe cast as last) as last         // asserts view is an unowned allocation root
```

This was first written in the casting owner during promotion and recorded
here afterwards.

**What happened to 039's first motivation.** "Lend through a pointer without
`unsafe`" is answered by: lend with `&`. Raw pointers are for lifetimes the
compiler can't see: links such as parent pointers and graphs, addresses used
as data, foreign code, hardware addresses, owning allocations, and frequent
repointing. Optional references (`MyValue loan & ?`) cover "present or
absent". The original `inspectMapped(_)` example takes
`MyMappedValue readonly loan &`.

**`deep` on a pointer layer is not adopted.** Copying a `strong` pointer
deeply would allocate, and a copy has no way to name the arena. The current
rule that `deep` never clones what a pointer points to stays.

#### A12. Pointers through several layers

Each layer that owns its value keeps owning it, however you reach that
layer. What changes is what you may write through the chain:

- you count as an owner of a value only when every layer between you and it
  is one you own;
- once your path passes through a borrower, you are a borrower of everything
  beyond it.

**One stance per layer.** In `MyType copy * last *`, the word after the first
`*` is both the owning pointer's stance and what the outer borrower was
granted for that pointer. On a borrowed layer, the stance written is the
borrower's grant, which may name less than the owner declared, exactly as
`alias : MyBuffer loan & = owner` sees a `move` value as `loan`.

Four different lines, each doing something different:

```zax
owner : MyType copy * last = @

view : MyType copy * loan * = owner              // 1. grant loan (the default):
                                                 //    a pointer to owner that may only look.
                                                 //    Nothing moves; owner stays usable.
taker : MyType copy * last * = owner as last     // 2. grant last: a pointer to owner that may
                                                 //    take ownership later. Nothing moves yet,
                                                 //    but owner counts as finished from here.
owned : MyType copy * last = (taker.) as last    // 3. take: builds a new owning pointer from the
                                                 //    owning pointer reached through taker.
                                                 //    Ownership moves now.
value : MyType * = (view.).                      // 4. borrow the MyType through the chain:
                                                 //    no as needed; nobody is finished.
```

Line 2 is allowed while `valueLastPtr as last` into a `copy` pointer is not
(A10) because the destinations differ. Line 2 builds a pointer *to* the
owning pointer, so the owning pointer is lent. The forbidden line would
build a `copy` pointer *from* the owning pointer, copying its address.

After line 2, if the permission is used, `owner` is empty and its cleanup
finds nothing to free. If it is never used, `owner` still frees the
allocation at the end of its scope. `taker` must not outlive `owner`, as
with any raw borrower. So write `as last` in a chain only when you really
mean to let someone take ownership. To reach the value, use lines 1 and 4.

Several dereferences are written with brackets: `((x.).).`.

#### A13. Slices, array joins, and optionals

**Slices are borrowers, checked like references.** A slice into a
`unique`-owned array is allowed:

```zax
owner : MyMessage[0..] * unique = @
window : MyMessage readonly loan [] = owner.     // fine: a slice is checked
raw : MyMessage * = owner.[0]                    // error outside unsafe
```

**Array joins need no new rule.** Each operand of `<+>` is an ordinary
source:

```zax
left : MyMessage[0..] move
right : MyMessage[0..]

joined1 := left <+> right            // error: no stance means copy; left is move
joined2 := left as move <+> right    // fine
joined3 := left as last <+> right    // fine: left may also hand over its storage

window : MyMessage readonly copy [] = right
joined4 := window as last <+> right  // error: a slice is a borrower granted copy
```

Storage reuse with `as last`, and a slice being unable to hand over storage,
now follow from A2.

**Optionals have two stances.** The stance before `?` belongs to the
contained value. The stance after `?` belongs to the optional itself.
Copying or moving the whole optional also copies or moves the contained
value, so the optional's own stance may only name stances the contained
value's stance also allows:

| Contained value's stance | Allowed stances for the optional itself |
| --- | --- |
| `copy` | any |
| `move` | `loan`, `move`, `last` |
| `last` | `loan`, `last` |
| `deep` | `loan`, `deep` |
| `loan` | `loan` only |

```zax
slot : MyPayload move ? loan     // a pinned slot whose contents may be moved out

keep(last slot)                  // fine: takes the contained value; slot becomes empty
other := slot as move            // error: the slot itself is pinned
slot .= makePayload()            // fine: builds a new contained value in place

broken : MyPayload loan ? move   // error: moving the optional would move a pinned value
```

#### A14. Erasure and casts

An `OpaqueObserver` or `OpaqueReferenceObserver` records the type, the
stance, and the qualifiers of what was put in. Recovering a value must state
them explicitly. It may state the same stance, or any stance whose permitted
operations are all inside what was put in, but never one that permits
something that wasn't put in:

```zax
view : MyValue copy * = myValue
erased : OpaqueObserver = view
back : MyValue copy * = erased       // fine: the same stance
lent : MyValue loan * = erased       // fine: loan permits nothing, so it fits inside copy
taker : MyValue move * = erased      // error: copy was put in; move was not
```

Qualifiers follow their usual rules: recovery may turn `writable` into
`readonly`, never the reverse, and the recorded `immutable` or `final` truth
must match.

**Implementation pressure.** The stance and qualifiers belong to the path,
not the type, so they can't live in the type's reflection metadata. The same
`MyValue` can be erased once as `loan` and once as `copy`. The expected
approach is for the compiler to build a record for each erasure and recovery
site, holding the type, stance, and qualifiers. When both sides are visible,
the check is entirely at compile time. A truly opaque recovery checks the
type identity, then that the requested permissions fit inside the stored
ones. That is a small mask test, not an `==` comparison. A failed stance or
qualifier check behaves the same as a type mismatch does today:
`OpaqueObserver` produces a vacant pointer, while `OpaqueReferenceObserver`
panics because a reference cannot be vacant.

**Two unsafe casts, one change each.** A cast that changed the type and
the stance together would hide mistakes: a typo in the type would go
unnoticed while the stance changed. So each cast does only one thing:

- **`x unsafe cast T` never changes a stance.** It keeps today's meaning: it
  may change the type, the qualifiers, and the pointer kind, the last under
  the ordinary transition rules. It keeps every stance:
  `view unsafe cast MyOtherValue *` gives `MyOtherValue loan *`. Writing a
  stance in `T` that differs from `x`'s is an error. In
  `myCar as last unsafe cast Truck * unique`, the `as last` is a stance
  written for the transfer, not one the cast changes, so that form still
  works.
- **`x unsafe cast as S` changes only the stance,** on the outermost layer.
  The type stays the same.

```zax
view : MyValue * = owner.
restored : MyValue * last = (view unsafe cast as last) as last   // fine: only the stance changes
wrong : FooVar * last = (view unsafe cast as last) as last       // error: the cast gives MyValue * last,
                                                                 // which isn't a FooVar * last
mixed := view unsafe cast FooVar * last                     // error: a type cast can't change a stance
retyped : MyOther * = view unsafe cast MyOther *            // fine: type only
trusted := myMutable unsafe cast MyType immutable &         // fine, as today: qualifiers change
```

**`unsafe cast as S` is the general escape from stance rules.** It may
replace any stance, not only make a raw pointer an owner (A11). Without it,
nothing could ever widen a stance, even where the programmer knows more than
the types say: container internals, or interop where a foreign API's real
behavior is known. Writing `unsafe` means the programmer takes
responsibility for what the stance protected:

```zax
remember final : ()(input : MyMessage copy &) = {
  keep(input as last)                          // error: input was granted copy (A2)
  keep((input unsafe cast as last) as last)    // compiles: the function claims last on the
                                               // caller's value. The caller still believes it
                                               // is unchanged; keeping that true is now the
                                               // programmer's responsibility
}
```

**Inner layers are reached through `.`,** one layer per cast. `lent.` names
the pointed-to value; casting it and making a pointer from it uses the
address-preserving rule (A11):

```zax
lent : MyValue loan * = myValue
widened : MyValue copy * = (lent.) unsafe cast as copy
// the MyValue that lent points to, seen as copy instead of loan;
// the new pointer has lent's address, vacant included
```

A reference has no layer of its own, so its outermost stance is the
referred-to value's. A short form that restates nothing else was preferred
over a full target type.

**The cast gives the same value with a new stance, not a new value.** So
transferring it still needs the stance written, as in
`(view unsafe cast as last) as last`. The cast forces the value into a
stance it wasn't declared with. Four consequences, each following from rules
already in part A:

1. **It changes the stance, not which operations exist.** The type's
   `transfer` list still decides what exists, and fallback still applies
   (A10):

   ```zax
   MyPoint :: type transfer copy {
     x : Integer
   }

   pinned : MyPoint loan
   moved := (pinned unsafe cast as move) as move   // compiles; MyPoint has no move, so copy runs
   deeper := (pinned unsafe cast as deep) as deep  // error: MyPoint has no deep, and deep never falls back
   ```

2. **A cast that only narrows is legal but pointless.** Narrowing is already
   allowed by ordinary binding. A compiler may warn that the `unsafe` is
   unnecessary, but can't reject it (part D):

   ```zax
   plain : MyValue copy * = myValue
   lent : MyValue loan * = plain                       // ordinary narrowing: no unsafe
   lent2 : MyValue loan * = plain unsafe cast as loan  // legal; may draw a warning
   ```

3. **It changes stances, never qualifiers.** Making a `readonly` path
   writable stays with the qualifier mechanisms (`unsafe pliable` and
   qualifier casts). Changing qualifiers would be a different operation:

   ```zax
   reader : MyValue readonly copy & = myValue
   taken := (reader unsafe cast as last) as last   // the stance becomes last, but the path is still
                                                   // readonly, so a last operation that must change
                                                   // the value can't use it
   ```

4. **A pinned value can be moved under `unsafe`.** Other code may rely on a
   pinned value's address staying fixed. Having written `unsafe`, the
   programmer gets what they asked for:

   ```zax
   device : MyDevice loan
   relocated := (device unsafe cast as move) as move  // compiles; pointers to device may now be stale
   ```

#### A15. Spelling

- The stance goes **before `&`**, because a reference has no layer of its
  own. Only `MyValue loan &` is written; `MyValue & loan` is not an
  alternative.
- The stance is **last on its layer**, after qualifiers and pointer kind
  words. `via` follows it:

  ```zax
  source : MyArchive readonly copy &           // today: MyArchive readonly & copy
  value : MyValue loan * unique writable move  // stance of the value, then the pointer layer
  item : MyItem copy via move                  // parameter stance, then via
  ```

- **A stance that lands after `&` through an alias or a generic** applies to
  the nearest layer that can carry one, the referred-to value:

  ```zax
  DeepView :: alias type MyPayload readonly deep &
  CopyView :: alias type DeepView copy       // CopyView is MyPayload readonly copy &

  keepOne$ final : (T : type)()(item : T move) = {
  }
  keepOne(myRef)                             // with T = MyValue &, item is MyValue move &
  ```

### Part B: open questions

None at present. The last two (forbidding unlisted stances, and narrowing
on recovery) were settled on 2026-11-18; see A10, A14, and part C.

### Part C: deferred work

**To a follow-up work item.** None of these changes the pointer model.

- **A stance on each array element.** Today elements are governed by their
  type. A stance before `[` could apply to each element:
  `messages : MyMessage move [4]`, so that `keep(messages[0])` is an error.
- **Slice parameters** are stand-alone borrowers and would need an element
  stance: `values : MyMessage readonly loan []`.
- **Optionals' `move source` and `last source` forms** may be explained as
  the optional's owner writing a stance on the contained value.
- **Optional references** as parameters or members, such as
  `MyValue loan & ?`, would need a stance as stand-alone borrowers. The
  unmarked optional-reference examples in `language/optional-values.md`
  were left as they are on purpose during 039's step 2, for this work item
  to settle: the type-composition examples around line 1055
  (`MyValue readonly & ?`, `MyValue readonly & * writable`), the composed
  forms around line 1070 (`MyValue & * & * &`, `MyValue & ? * & ?`), the
  `view : MyValue readonly & ?` declaration around line 1081, and the local
  `inner : MyValue? & = nested.` around line 631.
- **Lambda reference captures** would follow the inheritance table in A6.

**Deferred elsewhere,** each with what brings it back:

- **Making a stance's use an error instead of a fallback (`forbid`).** The
  maintainer proposed `transfer copy forbid move`. Allowing `= forbidden` on
  an unlisted stance's signature would have the same effect. Both were
  deferred because fallback is relied on in three places, and forbidding
  `move` would break all three:

  ```zax
  // candidate syntax, not adopted
  MyPoint :: type transfer copy forbid move {
    x : Integer
  }

  makePoint final : (result : MyPoint)() = {
  }

  existing : MyPoint
  existing = makePoint()          // error: the unnamed result is treated as written as last,
                                  // which falls back through move, which is forbidden

  points : MyPoint[0..]
  points.append(makePoint())      // error for the same reason; arrays also move
                                  // elements when they grow or relocate

  keepAll$ final : (T : type)()(item : T via move) = {
  }
  keepAll(point as move)          // error: generic code that moves any T breaks for MyPoint
  ```

  No use case outweighs that. A type that must never be duplicated leaves
  `copy` out of its list. A type that wants to discourage `as move` because
  it only copies is better served by a warning, which never changes what
  compiles. Reactivates: a concrete type that needs `as move` to be an
  error, with answers for temporaries, arrays, and generic code. Constraint
  if it returns: a declared stance that `forbid` makes unusable is an error
  at the declaration (A10), as in `point : MyPoint move`.

- **Foreign-function declarations that take `&`** and pass an address at the
  boundary, with trust placed in the declaration. Reactivates: interop
  design. Until then, lending to C needs `unsafe`.
- **Replacing a pinned `loan` variable in place with `.=`.** Reactivates:
  replacement design, or the first example that needs it.
- **Owning raw pointers compared with blockless `unique`.** Pointing inside
  an allocation is settled: neither has a control block, so neither can point
  inside an allocation (A11). Reactivates if other overlap or divergence
  between the two needs them to merge or separate further.
- **Raw borrows made before a raw-to-`unique` conversion** may still exist.
  Accepted for now. Reactivates if `unique`'s guarantee must be complete.
- **`deep` on owning pointer layers.** Reactivates if a use can name an
  allocation arena.
### Part D: captures for other owners

**Candidate principle: what compilers must agree on.** Raised by the
maintainer while discussing raw-pointer safety; agreed in direction.
Candidate owner: `language/principles.md`, after separate discussion.

> Whether a program is accepted depends only on rules the specification
> states. A compiler's extra analysis may produce warnings, but never accept
> or reject differently.

The specification defines the cases that count as safe. A compiler that can
prove more still requires `unsafe` outside those cases, and may say the
marker looks unnecessary. Adding safe cases later keeps existing programs
compiling. Adding required errors later breaks them. The maintainer's
long-standing concern is recorded with it: a capable compiler will sometimes
demand `unsafe` for code it knows is safe.

**Documentation lessons.** Destination: `project/documentation.md`, after
separate discussion and authorization.

- **Every change leaves the edited documentation clearer than it was**, even
  when the surrounding text is poor. New or rewritten material meets the
  teaching standard, not the level of the text around it.
- **Teaching promotion is written by the agent that holds the discussion**,
  not delegated to a subagent, because the subtleties do not survive a
  handoff. Delegation is limited to mechanical changes, such as spelling
  migrations.
- **Working-record wording is not teaching wording.** Promotion re-authors
  from a reader's point of view. The maintainer declined to approve even the
  revised wording in this record as sufficient for teaching.
- **Do not hold back retiring files to keep links in any work item working,
  including the current one.** Git history exists for that. When a work item
  closes, the repository should be in its proper final state. (Maintainer,
  2026-11-22, after this session held back retiring
  `project/raw/pointer-stances.md` because 039 still linked to it.)
  **Applied to `project/documentation.md` on 2026-11-22** with the
  maintainer's go-ahead: a new "Retiring files never waits for work-item
  links" section under closure, a raw-file note that the link check covers
  promoted material only, and the live-link validation item now excluding
  links inside `project/work/` as well as `project/archive/`. The rule now
  lives only in `project/documentation.md`: in both operating prompts,
  `project/handoff.md` and `project/rehydrate.md`, the old sentence ("check
  every live reading path") is replaced by a pointer to
  `project/documentation.md`, and both moved to operating guidance
  revision 9. The other part D lessons were applied at closure (2026-11-22):
  "every change leaves the edited text clearer" in "Teaching debt"; "teaching
  promotion is written by the agent that holds the discussion" in
  "Promotion"; "the current proposal and open questions stay separate from
  history, and an unreviewable record is restructured" under the
  numbered-work file shape. "Working-record wording is not teaching wording"
  was already covered by "Promotion re-authors for a different reader", so no
  edit was needed.
- **A working record shows the current proposal and the open questions
  separately from history**, and is restructured as soon as it stops being
  reviewable, rather than added to. This record had to be restructured after
  the maintainer could no longer review it.

### Part E: consequences for promotion

The current documents a promotion would change:

- **Every owner that writes `& S`.** About 150 occurrences across 14 owners
  become `S &`, including every lifecycle source such as
  `source : MyArchive readonly copy &`. Read-only reference parameters gain
  `loan`.
- **Transfer stances.** Most sections that define declared stances, bodies,
  references, receivers, results, and pointer ownership. In particular:
  - "One stance on one complete value";
  - "Where `loan` may appear";
  - "Declared stances by location" and its subsections;
  - "Stances inside a body";
  - "References pass on at most their stance";
  - "Results and forwarding": the `peekValue` example goes;
  - "Results with no declared stance" shrinks to the rule for unnamed values;
  - "Written `copy` versus no stance";
  - "Pointer ownership transitions";
  - the composition wrapper table.
- **Construction and destruction.**
  - Constructor receiver stances (A9), including how they relate to
    generated constructors and the selection rule.
  - The sentence "A constructor has no receiver stance, because the value it
    builds does not exist yet" (in "Generated `copy` construction and
    assignment") is replaced. What it was trying to say still holds: the
    compiler generates only constructors with no receiver stance and a `copy`
    source. A declared receiver stance is a requirement on the value being
    built.
  - The fallback rule (A10). `forbid` is deferred (part C).
- **Declarations and bindings.** The declaration-error example
  `MyCopiedDocument :: alias type MyDeepOnlyDocument copy` (in
  `language/declarations-and-bindings.md`) is replaced, because `copy`
  includes `deep` (A10). Corrected 2026-11-21: this note previously placed
  the example in construction and destruction, which never contained it.
  `language/transfer-stances.md` already uses the corrected `MyDeepOnly move`
  example.
- **Qualifiers.** A stance attaches to a layer like a qualifier but is not
  one. Canonical ordering gains the stance, last on its layer, then `via`.
- **Pointers and arenas.** Raw ownership through `move` and `last`, with `@`.
  In particular:
  - `scoped : MyValue * = @` becomes an error in many examples;
  - the `copy` allocating result goes;
  - the raw pointer into a `unique` array (`pointerTo(container.items[0])`)
    becomes an error outside `unsafe`;
  - borrowing with `p.`;
  - pinned values under owning pointers.
- **Nothing instances.** A carve-out: making a pointer from `p.` is not a
  dereference and cannot trap.
- **Lifetimes and references.** Elision becomes mandatory for `loan` results.
- **Function invocation.** `via` on parameters and results, overload
  selection by `via`, and the selection rule that treats the receiver as one
  more input (A9).
- **Transfer stances, "Receiver stance" section.** A receiver becomes a
  parameter bound implicitly (A2):
  - with no stance written, it is copied where allowed and otherwise lent;
  - a version serves any call whose written stance allows what it takes,
    and the closest wins;
  - "versions with a receiver stance do not fall back to each other" goes,
    except that a `last` version never serves `as move`;
  - the separate rule "if the type has no version without a receiver
    stance, use the `copy` version" becomes a consequence, not a rule.

  The current guarantee "adding a `copy` version to a type never changes
  what existing calls with no stance do" is lost, deliberately: such a call
  on a value that allows copies now chooses the new `copy` version. That
  version can't change the caller's value.
- **Declarations and bindings.** Borrowers bound from one visible value take
  their stance from it (A6).
- **Optional values.** The two stances of an optional (A13).
- **Conversions and casts.** `unsafe cast` keeps every stance and rejects a
  target that writes a different one. The new `unsafe cast as S` changes only
  the outermost stance. The sentence about managed pointers ("the ownership
  role, stance, and lifetime rules are those of the ordinary transition")
  gains this.

### Part F: decision log

| Date | Decision or rejected idea | Reason |
| --- | --- | --- |
| 2026-11-16 | A stance means what may be built from a value | The maintainer's definition; `loan` fits it exactly |
| 2026-11-16 | Rejected: a stance means exactly its own set for everyone | An owner's `move` local could then never be finished with `as last` |
| 2026-11-16 | Rejected: a separate limit carried by each borrowing marker | The limit is the stance applied to a borrower (A2) |
| 2026-11-16 | Rejected: an inner layer's stance as a reference-parameter-style minimum and maximum | Replaced by owners and borrowers (A2) |
| 2026-11-16 | `copy` includes `deep` | A deep copy leaves the original unchanged, like any copy |
| 2026-11-17 | Owners may write `as move` and `as last` on a `copy` value, and `as last` on a `move` value; borrowers only their grant | The caller tracks its variable by the stance it wrote |
| 2026-11-17 | A by-value parameter's stance governs the body; `via` is the caller's minimum | The caller chooses how the value is built; the type protects itself |
| 2026-11-17 | Rejected: stances follow lineage | The `transfer` list already covers restrictions on every value of a kind |
| 2026-11-17 | Stand-alone borrowers must write a stance | Neither default worked; a requirement can be relaxed later without breaking programs |
| 2026-11-17 | Rejected: an unmarked local borrower inherits `move` or `last` | The permission could travel to a function with no `as` near the owner |
| 2026-11-17 | Raw pointer ownership through `move` and `last` stances, not `= @` alone | A pointer owns when it holds a responsibility |
| 2026-11-17 | Rejected: a constructor accepting `move &` and building a `copy` borrower | The owner would be tracked as possibly empty although nothing moved |
| 2026-11-17 | `unique` allows no raw borrowers; raw pointers from references of unseen origin need `unsafe` | Otherwise `unique`'s guarantee could be bypassed through a reference |
| 2026-11-17 | `strong` may hold `last` | The value can be rebuilt in place; others already see values they can't track |
| 2026-11-17 | Rejected: `deep` on pointer layers | A copy can't name the arena |
| 2026-11-17 | Rejected: binding a reference to a `loan` result | It hides who destroys the value; use a `loan` local |
| 2026-11-17 | A result has one stance inside and outside; `via` on a result constrains the `return` | The function's result and the caller's variable are the same value |
| 2026-11-17 | Rejected: "callers receive a copy" as a result form | No use case remains |
| 2026-11-17 | Rejected: a hand-off stance on results separate from the result's stance | Same reason as above |
| 2026-11-18 | Fallback passes over missing operations, never existing ones that refuse | Keeps "take ownership" from silently becoming "borrow and finish the owner" |
| 2026-11-18 | Pinning applies to the whole value, not its members | Pinning fixes identity, not contents |
| 2026-11-18 | Reference results and generic reference parameters must write a stance; `self` results can't | Same reason as stand-alone borrowers |
| 2026-11-18 | Erasure records and requires explicit stance and qualifiers; `unsafe cast` keeps the stance | You can't extract what you didn't put in |
| 2026-11-18 | Arrays, slice parameters, optional forms, optional references, and lambda captures deferred to a follow-up | None changes the pointer model |
| 2026-11-18 | Leaving a stance out of a `transfer` list means it doesn't exist; fallback applies; `= forbidden` only on listed stances | That is all leaving it out means |
| 2026-11-18 | Deferred: `forbid` in the `transfer` list, and `= forbidden` on unlisted stances | Both break temporaries, array growth, and generic code that rely on fallback |
| 2026-11-18 | A declared stance a type can never satisfy is an error at the declaration | Today's rule; a `transfer` list is closed, so the error is known early |
| 2026-11-18 | Recovery from an observer may state any compatible stance, never a wider one; the compiler builds a record for each erasure and recovery site | Stances belong to the path, not the type |
| 2026-11-18 | A new value's declared stance is the receiver stance given to its constructor; a constructor's receiver stance is a requirement the new value's stance must meet | A constructor's receiver stance protects the value it builds |
| 2026-11-18 | The receiver is just another input: a candidate wins only if at least as good on every input and better on one; otherwise ambiguous | Any fixed priority would silently run an operation the author may not have intended |
| 2026-11-18 | Rejected: "the closest receiver match is chosen", and "match the source first" | Replaced by the ambiguity rule |
| 2026-11-18 | A temporary is chosen for as if the new value were built from it; elision may skip running the constructor but the choice must succeed | Keeps elision from bypassing a constructor's requirement |
| 2026-11-18 | Every temporary's stance is declared or the unmarked `copy`; the stance and the `transfer` list are independent filters | No temporary's stance is guessed |
| 2026-11-18 | Corrected: `MyDeepOnly copy` is satisfiable, because `copy` includes `deep` | Today's example relied on 038's rule |
| 2026-11-19 | No receiver stance is the same as `loan`, for every function bound to an instance, including constructors; declaring both forms is a duplicate | One meaning for an unwritten receiver stance; not applicable where there is no receiver |
| 2026-11-19 | A receiver is a parameter bound implicitly; with no stance written on a call it is copied where allowed, otherwise lent | The same rule as an argument with no stance written |
| 2026-11-19 | `move`, `last`, and `deep` versions are chosen only when the call writes the stance | The caller authorizes what happens to its value where a reader can see it |
| 2026-11-19 | Rejected: the declared stance chooses the version (option 1) | Hides the transfer from the reader |
| 2026-11-19 | Rejected: choosing the `last` version without counting the value as finished (option 3) | The next line could use a finished value; `self` chains already give the desired behavior |
| 2026-11-19 | A version may serve any call whose written stance allows what it takes; closest wins; a `last` version never serves `as move` | The same as parameters; replaces "versions do not fall back to each other" |
| 2026-11-19 | Accepted loss: adding a `copy` version can change which version an existing call with no stance uses | A `copy` version can't change the caller's value |
| 2026-11-19 | `via` only on what is built (by-value parameters and results); an error on references and receivers | A reference's stance already requires the call to write it |
| 2026-11-19 | `via` kept as the keyword; `using` rejected | `using` already means keep-alive and disposal, and reads as the body's stance |
| 2026-11-19 | Forbidden and bodyless candidates take part in selection like any other: chosen means error, a tie means ambiguous, fallback stops at them | Forbidding something must never silently change which other operation runs (bodyless retired 2026-11-21; see below) |
| 2026-11-19 | Rejected: dropping a forbidden candidate from a tie | It would hide overlapping declarations from the author |
| 2026-11-19 | A constructor's receiver stance comes from the new value's declaration, with no `as` | The receiver is the new value; nothing existing is given up through it |
| 2026-11-19 | `@<` and `@!<` removed; every allocation has an owner; `@` into a raw `copy` pointer is an error | Keeps cleanup decided by stance alone; every former use has a replacement |
| 2026-11-19 | An owned allocation may move into a raw `move` or `last` pointer without `unsafe` | Ownership simply passes between owner kinds |
| 2026-11-19 | `unsafe` may make a non-owning pointer an owner | Lets ownership be given up through `unsafe vacate` and later taken back |
| 2026-11-19 | Rejected: `unsafe adopt`; then rejected: `unsafe cast` to a full type changing type and stance together | `adopt` names a posture, not an operation; a combined cast hides type mistakes |
| 2026-11-19 | `x unsafe cast T` never changes a stance (it may still change type, qualifiers, and pointer kind, as today); `x unsafe cast as S` changes only the outermost stance | One stance change per cast; corrected by dry run 2 (E1) to match the casting owner |
| 2026-11-19 | `unsafe cast as S` is a general escape: it may widen any stance, including a borrower's grant; inner layers are reached through `.` | Otherwise no escape is possible; `unsafe` marks the responsibility |
| 2026-11-19 | `unsafe cast as S` gives the same value with a new stance, so a transfer still writes the stance; it never makes operations exist, never changes qualifiers, and narrowing with it is legal but pointless | It forces a stance, nothing more |
| 2026-11-19 | Rejected: limiting `unsafe cast as` to the raw-pointer ownership escape | Leaves no escape for low-level code that knows more than the types say |
| 2026-11-19 | The one built-in raw-pointer rule: the owner of a raw `copy` or `loan` pointer may not write `as move` or `as last` on it | Constructors can't see a source's declared stance; a second stance per reference or stance-as-type was rejected |
| 2026-11-19 | A user type whose ownership depends on its declared stance can't express this rule; such a design should be two types | Recorded so raw pointers are not taken as a pattern |
| 2026-11-19 | Dry run 1 FAIL resolved: D3 terms (owner, borrower, pinned; "owning raw pointer"), D4 homes (040 for follow-ups, one raw file for long-term deferrals, retire `raw/pointer-stances.md`), D5 (part D outside this promotion), D6 (three-step promotion plus a cold-reader review) | Accepted as recommended |
| 2026-11-19 | Generated constructors never have a receiver stance; a receiver stance is part of a constructor's shape, so a declared one never replaces a generated one and competes as a more specific candidate | Generation stays exact-shape; receiver stances only add more specific matches |
| 2026-11-19 | `@` produces a value whose stance is `last` | Otherwise an unmarked allocation could bind to a borrower constructor and leak |
| 2026-11-19 | A raw pointer holds a responsibility exactly when its stance is `move` or `last`; cleanup is decided by stance alone | A `loan` raw pointer is a pinned borrower; letting it own would force the compiler to track initialization |
| 2026-11-19 | Rejected: a pinned raw owner (`* loan = @`) | Same reason; a `* last` owner that is never handed on behaves the same |
| 2026-11-19 | The raw pointer forbids taking over with no receiver requirement (`= forbidden`), instead of a built-in pointer exception | Uses existing rules; pointers stay what their constructors would declare |
| 2026-11-19 | Receiver match quality: exact best, otherwise closer along `loan`, `copy`, `move`, `last` | The ambiguity rule needs a per-input measure |
| 2026-11-19 | Raw borrowers from owners only through `owner.`, for `move` and `last` owners alike | `as move` would leave the owner tracked as possibly empty when nothing moved |
| 2026-11-18 | A value declared `deep` but initialized by an ordinary copy is built by whatever constructor the type provides; a `deep`-receiver constructor makes such values independent; no warning | The type decides what copying means and has the tool to treat `deep`-declared values differently |
| 2026-11-18 | Rejected: the first A9 table (a `copy` constructor excluded `loan` and `deep` values; `loan` values were buildable only by unmarked constructors) | A pinned value meets every requirement, and `copy` adds none |
| 2026-11-21 | `reset` is an error on raw `copy`/`loan` borrowers (no `unsafe reset` route); plain `vacate` on a borrower is always safe; `unsafe vacate` is allowed on raw `move`/`last` owners and stays an error on managed pointers | Freeing stays decided by stance alone; one escape (`unsafe cast as last`) handles reclaiming and freeing |
| 2026-11-21 | Observer runtime stance/qualifier mismatch matches type mismatch: `OpaqueObserver` gives a vacant pointer; `OpaqueReferenceObserver` panics | Corrected wording in A14; a reference cannot be vacant, while a raw pointer can |
| 2026-11-21 | An owning raw pointer can never point inside an allocation (same as blockless `unique`); interior claims yield vacant (or error if proved interior) | No control block to record the allocation root; narrows the part C deferral |
| 2026-11-21 | A constructor whose receiver stance can't build the new value stays in selection as the worst receiver match; if chosen, the line is an error (A9 step 2) | The earlier "remove incompatible candidates" step contradicted A10 and let `as last` quietly run `copy` |
| 2026-11-22 | A borrowing pointer parameter's `= @` default is written as a borrow from a temporary owner, `= (: MyValue * last = @).` (A7) | Keeps the borrowing meaning with no new rule; making the parameter owning was rejected |
| 2026-11-22 | A compatible prototype may narrow a result's stance, never widen it, never change to or from `loan` (A8) | Widening could make an owning pointer copyable; a `loan` result's construction is fixed by the body |
| 2026-11-22 | A call on the type lends the Nothing instance, which is pinned: the version with no receiver stance is used, and a type with only a `copy` version can't be called on the type (A2) | Copying a shared stand-in has no meaning; keeps one receiver rule |
| 2026-11-22 | A pointer made from an integer is a borrowing raw pointer; owning the allocation at that address also needs `unsafe cast as last` (A11) | An address carries no responsibility; recorded after it was written into the casting owner |
| 2026-11-22 | Current owners describe delayed presence contracts neutrally ("callable contracts or pointer types") | An owner must not rule out, or link to, a direction recorded in raw input |
| 2026-11-21 | Bodyless declarations retired; a lifecycle or operator declaration must end in a body, `= default`, `= existing`, or `= forbidden` | Nothing could complete them; `= forbidden` already blocks; their extra rule conflicted with the selection table; a forgotten body became one silently. The completion idea is captured in `project/raw/callable-selection.md` |

### Part G: the maintainer's input at assignment

The maintainer supplied this when assigning 039. It is kept because the
fixed initiating input must not change. Several of its positions were later
revised in discussion; the decision log (part F) records how.

**Why stances reached pointers.** Adding `loan` changed what a stance slot
says. The other stances say how an instance was transferred, and `loan` adds
"this instance cannot be transferred at all". "Transferred" needs to be
precise. It does not mean that nobody can be handed access to an instance. It
means: *how can we create a new instance from an existing one using
constructors, assignment, and replacement, if at all.* That question formed
the stances.

**The bizarre case.** Applying a never-transferred stance to a pointer today
gives:

```zax
func final : ()(foo : FooIsNeverTransferredType * unique loan) = {
  // ...
}
```

That cannot work: `unique` can only be passed via `move` or `last`, yet `foo`
must never be transferred.

```zax
func final : ()(foo : FooIsNeverTransferredType * unique move) = {
  // ...
}
```

Now the pointer can be moved, but so can `foo`.

**Unclear signals.** When pointers are handed around, the type does not say:
am I adopting ownership of the raw pointer, and what am I allowed to do with
it?

**References have no layer stance.** The reference itself is always loaned;
there is no other choice. So a stance on the reference layer is meaningless,
just as qualifiers on the reference layer are:

```zax
Foo readonly & writable     // not a thing
Foo writable & readonly     // not a thing

Foo readonly * writable     // yes, this makes sense
Foo writable * readonly     // yes, this makes sense
```

Today the stance goes at the end, `Foo & loan`, which means exactly the same
as `Foo loan &`. But `Foo loan *` and `Foo * loan` would not mean the same
thing. Every current example puts the stance at the end, so it applies to the
core type, and nothing is ever said about the pointer itself. Adding stances
to pointers clarifies and corrects this.

**`loan` on a by-value type is nonsensical:**

```zax
func final : ()(myType : MyType loan) = {
  // ...
}
```

To create `myType`, `MyType` needs permission to be constructed, and `loan`
grants none. This is sensible:

```zax
func final : ()(myType : MyType & loan) = {
  // ...
}
```

It is sensible because the reference allows the loan stance to be handed off,
not because the reference itself is a loan. So it should really be:

```zax
func final : ()(myType : MyType loan &) = {
  // ...
}
```

The type is `loan`, not the reference. The reference is always copied, with an
implicit `copy` stance, and is always `readonly`, `immutable`, and `final`.
There is no reason to declare any stance or qualifier on the reference itself.

**A reservation about defaults.**

```zax
func final : ()(myType : MyType) = {        // type defaults to copy, because that's the minimal to make it work
  // ...
}

func final : ()(myType : MyType &) = {      // type defaults to loan, because that's the lowest promise we can grant
  // ...
}
```

The reference changes the default mostly because it lowers the lowest usable
stance: by value, `loan` is impossible.

**Pointers are their own thing.** They are not always `readonly`,
`immutable`, and `final`:

```zax
func final : ()(myType : MyType loan * copy) = {    // the type can be loaned, but the pointer is copied
  // ...
}
```

An outer pointer cannot be `loan`, because a pointer must be copied to be
passed into a function.

**Raw pointer to a pointer:**

```zax
funcPtrBad final : ()(myType : MyType loan * loan) = {          // error: this is not legal because a pointer cannot be loaned by-value
  // ...
}

funcPtrRef final : ()(myType : MyType loan * loan &) = {        // works, but effectively you are being loaned via a reference where the reference is copy
  // ...
}

funcPtrCopy final : ()(myType : MyType loan * copy) = {         // point is copy, this doesn't take ownership
  // ...
}

funcPtrDefault final : ()(myType : MyType loan *) = {           // point is implicitly copy by default, this doesn't take ownership
  // ...
}

funcPtrPtr final : ()(myType : MyType loan * loan * copy) = {    // the type can be loaned, but the pointer is copied
  // ...
  funcPtrBad(myType.)       // no, pointer can't be loaned
  funcPtrRef(myType.)       // works, pointer is loaned
  funcPtrCopy(myType.)      // no, pointer cannot be copied
  funcPtrDefault(myType.)   // no, pointer cannot be copied
}
```

A `copy` pointer cannot accept ownership. A `move` or `last` pointer would.

**`unique`:** a unique pointer cannot have a `loan` stance, at least not
without a reference. A unique pointer can never have a `copy` stance.

**`strong` and `weak`:** a shared pointer cannot have a `loan` stance, at
least not without a reference. It can have `copy`, `move`, or `last`. It could
also have `deep` if the pointed-to instance is also `deep`, since a full
independent copy could be made.

**Not yet examined by the maintainer at assignment:** whether this logic
holds together; how it affects every place a transfer happens; whether there
is a problem point that must be resolved; how it applies to optional and
array layers; and whether it means anything for variants (the maintainer
suspects not).

## Dispositions and promotion dry run

### Promotion progress

Promotion was authorized on 2026-11-19 and is being done in the reviewable
steps of D6.

**Done:**

- `language/transfer-stances.md` re-authored to part A, following the
  teaching plan's order. The sections "Same spelling, different location",
  "Stances inside a body", and "References pass on at most their stance" are
  replaced by "Owners and borrowers" and "Passing a value to a reference".
  Results, temporaries, receivers, `via`, pinned values, optionals, and
  pointers are rewritten to part A.
- Inbound links from ten other owners that pointed at retired section
  anchors in transfer stances now point at the new sections. Only the links
  changed in those files.
- `language/pointers-and-arenas.md` rewritten to A11 to A14 (2026-11-21),
  reviewed and staged by the maintainer.
- `language/construction-and-destruction.md` rewritten to A9 and A10
  (2026-11-21), including the new "Constructor receiver stances" section.
- **Bodyless declarations retired (2026-11-21, A10).** Decided during the
  construction review. Impact, all applied:
  - construction and destruction: the lifecycle forms table goes from five
    forms to four, the "Bodyless declarations" section is removed, a
    missing-body error with an example is added, and every list and
    diagnostic drops "bodyless";
  - function invocation (three places), operators, operator phrases, mixfix
    operators: "bodyless" removed from the unavailable-best lists; the
    unavailable-best rule itself is unchanged;
  - composition: the bodyless direct-declaration sentence now uses
    `= forbidden`;
  - transfer stances: "or no body" removed from generated operations;
  - declarations and bindings: "not a bodyless callable" becomes "not a
    callable";
  - `project/raw/reflection.md`: "bodyless" removed;
  - `project/raw/callable-selection.md`: "bodyless" removed, and a new
    "Declaring a shape now and implementing it later" section captures the
    retired form, its intended rules, an example, and what would bring it
    back.
  - Unaffected: "bodyless" in `switch.md` and `source-structure.md` means a
    `case` with no body, a different concept. Archived work is historical.
- `language/function-invocation.md` rewritten to A2, A7, and A8
  (2026-11-22): `via` on by-value parameters with a per-stance table (`deep`
  worked out), one stance per result, unnamed results as temporaries (section
  renamed "Results the caller doesn't name"), owning raw results only, the
  receiver as one more input, the pinned Nothing receiver for calls on the
  type, the temporary-owner `= @` parameter default, narrowing-only result
  stances in compatible prototypes, neutral wording for presence contracts,
  and respelled references. The coined word "plain" is retired here and in
  the transfer-stances table.
- `language/casting.md` rewritten to A14 (2026-11-22): "A type cast never
  changes a stance", the managed-pointer sentence (type, qualifiers, and
  pointer kind may change; never a stance; `myCar as last unsafe cast …`
  explained), the maintainer's example that a type cast can't make a raw
  owner, a new "Changing a stance with `unsafe cast as`" section with its
  four consequences and inner layers through `.`, the integer-to-pointer
  note, costs, and diagnostics.
- **Pinned-Nothing consistency fixes applied with casts (2026-11-22):**
  `language/nothing-instances.md` ("Receiver stance is statically known",
  diagnostics distinctions, diagnostics, and source-stability list),
  `language/transfer-stances.md` ("Receiverless `once` calls"),
  `language/composition.md` (`once` exposure), and
  `project/raw/callable-selection.md` (two bullets).
- **Consequences still to apply:**
  - `language/nothing-instances.md`, `demonstrateMapping` example: a `copy`
    receiver version guarded by `?_` is now callable only on instances,
    where `?_` is always true, and passing `_` to a raw pointer parameter
    needs `unsafe` (A11). The example teaches `_` as pointer-shaped and
    passable to pointer parameters, the idea 039 retired with "lend with
    `&`". The step 3 Nothing-instances edit must:
    - rebuild the example around a reference parameter
      (`MyMappedValue readonly loan &`), with any `?_` guard in a version
      that has no receiver stance, where calls on the type can reach it;
    - check the rest of `nothing-instances.md` for other passages that treat
      `_` as a raw pointer or make a pointer from it (more are expected);
  - `language/declarations-and-bindings.md` still teaches the old result
    rule just above the paragraph whose link was repointed (step 3);
  - (fixed in batch B, 2026-11-22) remaining "unmarked means a loan"
    examples, to fix during batch B or step 3. Where the example is about a loan, write `loan` explicitly;
    otherwise correct the comment to say the reference takes its stance from
    the value it is bound to:
    - `language/lifetimes-and-references.md` around line 806,
      `loanedView : Document readonly & = source   // unmarked: a loan`, and
      the `borrowed` comment at line 809 that depends on it;
    - `language/declarations-and-bindings.md` around line 1509,
      `inspect(resource)  // fine: an unmarked reference is a loan`, where
      `inspect`'s parameter (line 1504) also needs `loan`, and the sentence
      at line 1516 ("an unmarked reference or receiver");
    - `language/declarations-and-bindings.md` around line 1582,
      `alias : Buffer mutable writable & = owner   // unmarked: a loan`
      (`owner` is `move`, so `alias` does get `loan`; the comment should say
      why);
  - reference `except` results need a written stance in `except.md` and
    lifetimes (step 2).
- **Step 2, batch A progress** (respelling and stances; authorized
  2026-11-22). Working rules from the maintainer: writing `loan &` on a
  formerly unmarked reference parameter or result keeps its old meaning, so
  bodies matter only where an example copied through it or a comment
  describes it; unmarked locals bound to a `copy` value are now `copy`, so
  "unmarked: a loan" comments are corrected or the local writes `loan`;
  `deep`, `move`, and `last` by-value parameters become `via`. Record
  progress here after each few files.
  - done: composition (also `Coordinates * last = @`, the wrapper table
    and the `via` sentence), structural shapes, except (reference `except`
    payloads now `readonly loan &`), partials, generics
    (`myPointer : Boolean * last = @`), literal source, qualifiers
    (parameters and the `ReadView` alias only; the stance-ordering concept
    edit stays in step 3), enums, compile-time code, variants, variadics
    (`via last` and `via move` on `sendAll$`, `sendPair`, `encode`, and the
    item-by-item rule), intent acknowledgements (`member : Member * last =
    @`, `via` wording, the temporaries sentence), type definitions,
    optional values (one parameter), operators (one reference result),
    iteration (loop reference variables now take the element's stance;
    "unmarked, it is a loan" corrected), identity types, arrays and slices
    (array and slice reference parameters written `loan &`), lambdas (the
    `loanedView` example now writes `loan`). **Batch A is complete.**
  - batch B progress (allocation ownership, `@<` removal, and respelling
    in the remaining owners): done so far: operator catalog, operators
    (`reset` and `vacate` text), source structure, safety and analysis (the
    raw allocation, reset, and vacate passage; the unseen-origin rule stays
    in step 3), lifetimes and references (the scheduled and open-ended
    passage, every reference parameter and result, the `loanedView`
    example now written `loan`, reference captures taking their stance from
    the captured variable, and the `last &` comment), declarations and
    bindings (the allocation section, the parameter-default example, the
    assignment example, every reference parameter, the stance-position
    sentence, and the two "unmarked means a loan" examples carried forward
    above), terms (the allocation-policy sentence; "Declaration-attached raw
    allocation" removed; "Open-ended raw allocation" replaced by "Owning raw
    pointer"; the Loan entry's sentence about unmarked references and its
    example), Nothing instances (`reset` and `vacate` passage and one
    diagnostic), and one sentence in intent acknowledgements. **Batch B is
    complete.** A repository-wide search of live documents finds no `@<`,
    `@!<`, scheduled or open-ended raw wording, `& S` spelling, or links to
    the retired pointer sections. Left for step 3 in these files: the
    concept edits already listed (the old result rule and the inference
    exception in declarations and bindings, the Declared stance and other
    terms entries, `loan` result construction in lifetimes, the unseen-origin
    rule in safety, and the `demonstrateMapping` rebuild).
  - the root legacy page `compiler-directives.md` (line 166): the one line
    `value1 : Integer * = @` corrected to `value1 : Integer * last = @`, an
    allocation the declaration owns; nothing else on the page changed
    (authorized and applied 2026-11-22).
- **Step 3 progress** (authorized 2026-11-22; record after each few files):
  - qualifiers: stance described as "what may be built", attached last on
    its layer; canonical type-use order gains the stance and `via`, with an
    example and the alias rule; the `loan` sentence covers pinned values;
  - declarations and bindings: the `makeItem` paragraph (temporaries), the
    alias stance paragraph, the `MyDeepOnlyDocument` example (now `move` is
    the error and `copy` is fine), the inference exception for references
    bound to one value, and the result rule;
  - lifetimes and references: a `loan` result must be built directly into
    the caller's variable, with an example, in "Result elision joins life
    paths";
  - optional values: the two stances of an optional, with the table and the
    `slot` example, replacing "Transfer stance attaches once to the complete
    optional source"; the transfer-stance reference table points to it;
  - arrays and slices: slices as checked borrowers (allowed into a `unique`
    array where a raw element pointer is not), `slice as last` as an error,
    and joins as ordinary sources with the `left move` example;
  - safety and analysis: `unique` allows no raw borrowers, and raw pointers
    from references of unseen origin need `unsafe`, with examples;
  - Nothing instances: the `demonstrateMapping` rebuild around a
    `readonly loan &` parameter with the `?_` guard in a version with no
    receiver stance; `_` no longer described as pointer-shaped; a new
    "Making a pointer from `p.` is not a dereference" section; stances on
    raw-pointer members and parameters. Function invocation's `_` paragraph
    was rewritten to match. (These, and raw-pointer stances in composition,
    declarations and bindings, and type definitions, came from an earlier
    attempt of this session whose output was lost on a retry; the maintainer
    confirmed no other editor.)
  - raw-pointer stance search: borrowing raw-pointer parameters, results,
    members, and type aliases must write the stance of the value they point
    to; a repository-wide search finds none left, apart from a local bound
    to a value and two intentional error examples;
  - terms: new "Owner and borrower" and "Pinned value" entries; Declared
    stance (with `via`), Receiver stance, Transfer stance (with the
    what-may-be-built table and `via`), and Vacant pointer updated;
  - `index.md`: the transfer-stances and pointers router lines.
  - raw files: created `project/raw/pointer-ownership-deferrals.md` (part
    C's "Deferred elsewhere" items, each with an example, why it can wait,
    and what reactivates it) and `project/raw/compiler-agreement-principle.md`
    (part D's candidate principle, its reasoning, the maintainer's concern,
    an example, and its relation to the compiler-verified `unsafe` tag);
    both indexed in `project/raw/README.md`. `project/raw/pointer-stances.md`
    is retired and its index entry removed (2026-11-22). Its findings are
    promoted, and its deferrals live in the pointer and stance deferrals file.
    The links to it in this file's fixed initiating input and reading scope
    are left as they are; Git history recovers the file.
  - **Step 3 is complete.**

### Closure (2026-11-22)

- **Validation.** A full link and anchor check of the live tree found no
  broken links, excluding numbered work as `project/documentation.md` now
  allows. A cold-reader review of every materially changed owner, by an
  agent that had not read this file, reported 30 findings.
- **Cold-read findings fixed in promoted material:**
  - fallback: a source parameter's qualifiers that rule out the source count
    as "can't accept", so fallback continues; refusing (forbidden, or a
    receiver stance that can't build the value) is an error (transfer
    stances; casting's readonly `unsafe cast as last` example now says copy
    runs);
  - the written-stance rule applies to *borrowing* raw-pointer parameters,
    results, members, and aliases; owning pointers' pointed-to values are
    `copy` when unmarked (pointers, transfer stances, terms);
  - terms: `loan` is never a `via` stance or in a `transfer` list (a value
    declared `loan` is pinned); a call must write `as move`, `as last`, or
    `as deep` for those reference stances, while `loan &` and `copy &` need
    none; the owner rule names the borrowing-raw-pointer exception (also in
    transfer stances);
  - construction: a `loan` or absent receiver stance places no requirement;
    the raw-pointer paragraph now says what the receiver stance does; the
    `= existing` example became `= default`;
  - transfer stances: a `loan` receiver stance replaces a generated
    constructor; `result.+++(...)` examples made concrete; "today" wording
    removed; one `func1` comment corrected;
  - pointers: the several-layers example reordered so borrowing comes before
    the hand-off; `makePtrFromRef` returns `readonly loan *` (also in safety
    and analysis, and in the analysis-controls raw example);
    `pointerTo(...)` replaced by taking the address directly, with the
    `unique` contrast explained; the Sprite example explains publishing;
  - casting: `(myCar as last) unsafe cast …` parenthesized and explained;
  - function invocation: the `widened` example's comment; the presence
    paragraph shortened;
  - optional values: the section renamed "Stances on an optional and its
    contained value" and its list rewritten in plain terms.
- **Cold-read findings outside 039's changes, given live homes:** five
  teaching passages in the teaching-debt log ("Older passages found by the
  work-item 039 cold read", plus the qualifiers "stance" collision added to
  the existing entry); two pre-existing contradictions (qualifier defaults;
  `as` destination restriction) in the cross-cutting audit as "Owners that
  contradict themselves".
- **Teaching observations recorded in this file:** none, so none to
  dispose of.
- **Part D:** the compiler-agreement principle is a raw input; all
  documentation lessons are applied to `project/documentation.md`.
- **Raw inputs:** `pointer-stances.md` retired; `present-pointers.md`,
  `pointer-ownership-deferrals.md`, and `compiler-agreement-principle.md`
  created and indexed; analysis-controls and its registry updated.
- **Archived** to `project/archive/work/039-pointer-stances.md`; work item
  040 created from the proposal below, with the maintainer's go-ahead
  ("Please finish with 039 and prepare 040").

### Proposed initiating input for work item 040 (created at closure)

Creating 040 needs the maintainer's separate go-ahead at closure. Proposed
slug: `040-element-and-contained-stances`.

**Initiating concern.** 039 gave pointers a stance on the value they point to
and gave optionals two stances. Arrays, slices, optional references, and
lambda reference captures still reach a second value without a way to state
that value's stance. Should each be able to, and how?

**Starting input** (part C's follow-ups):

- **A stance on each array element.** Elements are governed by their type
  today. A stance before `[` could apply to each element, as in
  `messages : MyMessage move [4]`, so that `keep(messages[0])` is an error.
- **Slice parameters.** A slice parameter is a borrower that writes no stance
  for the elements it shows: `values : MyMessage readonly loan []`.
- **Optionals' `move source` and `last source` forms**, which may be
  explained as the optional's owner writing a stance on the contained value.
- **Optional references** as parameters, results, or members, such as
  `MyValue loan & ?`, which would need a stance as borrowers that stand alone.
  The unmarked examples in `language/optional-values.md` left on purpose
  during 039 (around lines 631, 1055, 1070, and 1081) are listed in part C.
- **Lambda reference captures**, which should follow the inheritance table in
  A6; the exact capture spelling is unsettled.

**Boundaries.** The 039 model is current design: owners and borrowers, `via`,
pinned values, one stance per layer written last on it, and the optional's two
stances. A conflict with it is recorded for discussion, not resolved by
changing it. Non-vacant (`present`) pointers are out of scope: they are a
separate concern with their own raw input and trigger. **Stated overlap:** that
raw input's sixth open question asks whether `present` should extend to
optionals; if 040 settles optional references, it should note the answer's
effect on that question rather than design `present`.

**Reading scope.** Required: `project/documentation.md`;
`language/transfer-stances.md` ("What a stance says", "Owners and borrowers",
"Declaring stances", "Optional values", "Arrays and slices");
`language/arrays-and-slices.md` (slices and array parameters);
`language/optional-values.md` ("Move and last source adapters", optional
references, the two stances); `language/lambdas-and-callable-composition.md`
(captures). Consequence-driven: `language/qualifiers.md` (ordering),
`language/function-invocation.md` (parameter selection),
`language/iteration.md` (loop bindings), `language/generics.md`.

**Initial stopping guidance.** First present the candidate model for each item,
with code and a recommendation, the most important uncertainty, and the
decisions needed. Creating 040 does not authorize analysis.
  - left as they are on purpose: `self` results (they take the caller's
    stance); local references bound to a visible value; the
    optional-reference type examples in optional values (optional
    references are a 040 follow-up); the `active size of Integer[5] &` type
    query.
- **Maintainer adjustments to the remaining steps (2026-11-22):**
  - step 2: `basics.md` gets only a minimal correction of its one sentence
    so the allocation family is `@` and `@!`, without retrofitting the rest
    of the legacy page (authorized and applied 2026-11-22);
  - step 3: `via` gets no `terms.md` entry (D3: a keyword, not a term); it
    is mentioned inside the Declared stance and Transfer stance entries;
  - step 3: `project/raw/present-pointers.md` stays its own raw input with
    its own trigger; 040 gets only part C's follow-ups, and any concrete
    overlap is stated rather than bundled;
  - closure: before retiring `project/raw/pointer-stances.md`, confirm that
    no live document links to it; before archiving, dispose of every
    teaching observation recorded here (resolved by promotion, entered in
    the teaching-debt log, or kept as a blocker); then propose 040's
    initiating input and stop.
- **Captures made during promotion review (2026-11-21),** each authorized by
  the maintainer and indexed:
  - `project/raw/present-pointers.md`: `present` (non-vacant) pointers,
    with six open questions for a follow-up work item;
  - `project/raw/analysis-controls.md`: the `[<safe=x-…>]` compiler-verified
    unsafe tag and its naming concerns, and the requirement that an error
    name the exact `unsafe` form when an escape exists;
  - `project/raw/analysis-control-registry.md`: provisional categories
    `raw-from-reference` and `unique-raw-borrow`.

**Decisions made while writing,** all following from part A, for the
maintainer to confirm:

- A `deep &` reference no longer makes `deep` the default inside the
  function. A use of a `deep` value with no stance is an error, as in A1's
  `archive` example, so `as deep` is written.
- Old examples with by-value `move` parameters (`sendMessage`, `sendHandle`,
  `sendAll`, `encode`, `sendPair`) now use `via move` or `via last`, because a
  by-value parameter's stance now governs only the body.
- In the composition wrapper table, a pinned by-value outer parameter still
  passes its member on as `last`, because members follow their own stance
  (A5).
- `slice as last` is described as an error, because a slice is a borrower
  not given `last` (A13).

**Remaining:**

1. Step 1 is complete.
2. Step 2: the mechanical respelling and the `@<` removal across the other
   owners.
3. Step 3: terms, routers, raw files, and work item 040.
4. A cold-reader review of the changed owners.

Review of the re-authored transfer stances (2026-11-20) corrected four
points, which also apply to the remaining owners:

- "no stance" never means "unchanged": it means nothing was moved out and
  the value isn't finished, while writable access may still change contents;
- "built from" is defined early (a copy, a value taking the contents, or a
  terminal transfer; reading is not building);
- the reason an unmarked alias of a `move` value gets `loan` is shown with
  an example;
- examples must not carry unexplained lines from older documents.

### Next session: how to resume the promotion

Promotion is authorized (2026-11-19). This session prepares and executes
the remaining steps one owner at a time. It does not re-open part A; a
material problem found while writing is raised with the maintainer before
any edit.

**1. Orient (read before anything else):**

- `project/documentation.md`, especially the writing rules ("Write the way
  you would explain it to a colleague", example obligations, positive-first
  teaching, and "Promotion re-authors for a different reader").
- In this file: part A completely; part E; part F only as needed; both dry
  runs (map, files affected, structure, teaching plans); this "Promotion
  progress" section.
- `language/transfer-stances.md` completely. It is the re-authored model
  owner and the reference for vocabulary, example style, and spelling. The
  other owners hand off to it rather than restating it.
- Part D's documentation lessons apply throughout. In particular, teaching
  text is written by this session, not delegated.

**2. Prepare (read the target sections, then propose, before editing).** For
each remaining owner, read the sections listed below in full. Then report to
the maintainer, in chat, a short plan for that owner:

- what each section teaches today;
- what part A changes in it;
- the planned shape: what is rewritten, what is kept and respelled, and
  which new section is added;
- what a reader is likely to get wrong;
- any conflict found.

Wait for the maintainer's review of the plan, then edit that owner, then stop
for review before the next owner. Order and sections:

1. **`language/pointers-and-arenas.md`** (A11, A12, A14, D1, D2). Read:
   - "Choose ownership separately from allocation";
   - "Allocate through a declaration", including "Direct allocation
     assignment" and "Allocation forms";
   - "Adopt an existing allocation";
   - "Pointer instances and pointees";
   - all of "Pointer ownership roles", including the raw-pointer
     subsections and "Allocation roots and records", which has the
     `pointerTo(container.items[0])` example;
   - "Resetting a pointer" and "Vacating a raw pointer";
   - "Ownership transitions";
   - the three opaque owner and observer sections;
   - "Costs", "Diagnostics", and "Source stability".

   Expect to:
   - replace "scheduled" and "open-ended" raw allocations with owning raw
     pointers (`move`/`last`);
   - remove `@<` and `@!<`;
   - add stances on pointed-to values, borrowing through `p.` (vacant stays
     vacant, no trap), the one built-in raw-pointer rule with
     `unsafe cast as last` as its escape, `unique` allowing no raw
     borrowers, and pointers through several layers.
2. **`language/construction-and-destruction.md`** (A9, A10). Read:
   - "Declared and generated lifecycle operations" through "Generated `copy`
     construction and assignment";
   - "Lifecycle sources are never transferred whole".

   Expect to:
   - add a section on constructor receiver stances (the declared stance is
     the receiver stance; generation; selection with the receiver as one
     more input; forbidden and bodyless candidates; temporaries; `deep`
     receiver constructors);
   - replace the sentence "A constructor has no receiver stance";
   - replace the `MyDeepOnly copy` example;
   - respell sources (`readonly copy &`).
3. **`language/function-invocation.md`** (A2, A7, A8, A9). Read:
   - "Receiver operands";
   - "Transfer-aware value and reference parameters";
   - "Visible defaults and compatible implementations";
   - "Result slots" through "Opt-in result initialization", including
     "Receiver-origin `self` results" and "Results with no declared
     stance";
   - "Candidate selection" through "A last use needs a stated stance".

   Expect to:
   - add `via` on parameters and results;
   - make the receiver a parameter bound implicitly (replacing the receiver
     bullets and "adding a `copy` version never changes" text);
   - give results one stance;
   - remove `@<` result rows;
   - respell.
4. **`language/casting.md`** (A14). Read "Unchecked reinterpretation with
   `unsafe cast`" through "Pointer ownership roles", and "Diagnostics".
   Expect to:
   - state that a type cast never changes a stance;
   - add `unsafe cast as S` as the general stance escape, with its four
     consequences and inner layers reached through `.`.

**3. Step 2 (mechanical, may be delegated with review).** Respell `& S` as
`S &` and add `loan` to read-only reference parameters across the remaining
owners (dry run 1's "Spelling migration" and "Removing `@<` and `@!<`"
lists). Before delegating, check each file for **non-mechanical** cases,
which this session handles itself:

- by-value parameters declared `move` or `last` to require a caller stance,
  which become `via move` or `via last`;
- raw `= @` declarations, which become `* last = @`;
- unmarked reference parameters whose body copies, which need `copy &`
  rather than `loan &`.

Every delegated change is read by this session afterwards.

**4. Step 3.**

- `terms.md`:
  - new entries for owner, borrower, and pinned;
  - update Declared stance, Loan, Receiver stance, Transfer stance, and
    Vacant pointer;
  - retire "Open-ended raw allocation".
- Targeted edits per dry run 1's list:
  - qualifiers (ordering);
  - declarations and bindings (inherited borrower stances, alias overlays,
    the inference exception);
  - lifetimes and references (elision for `loan` results);
  - optional values (two stances);
  - Nothing instances (the `p.` carve-out);
  - safety and analysis;
  - arrays and slices.
- `index.md` router wording, if needed.
- Raw files (D4, D5):
  - create `project/raw/pointer-ownership-deferrals.md` and a raw file for
    the compiler-agreement principle, with their examples from part C and
    part D;
  - update `project/raw/README.md`;
  - retire `project/raw/pointer-stances.md`.
- Propose the initiating input and reading scope for work item 040 (part
  C's follow-ups). Creating it needs the maintainer's explicit go-ahead at
  closure.
- The documentation lessons in part D are a separate, separately authorized
  change to `project/documentation.md`, not part of this promotion.

**5. Validate and close.**

- Check every live link and anchor.
- Run a cold-reader review by an agent that has not read this file, of
  every owner materially changed.
- Then follow closure in `project/documentation.md`:
  - mark this file historical;
  - archive it;
  - update `project/README.md`;
  - create 040 only after authorization.

**Working rules for that session:**

- Report staged and unstaged state at the start and after each owner.
  Preserve the maintainer's staged boundary.
- Lead every changed section with source and a plain explanation.
- Avoid comparisons such as "more" or "less"; name the actual stances.
- Don't cite this work item from any language document.

### Dry run 2 (2026-11-19): PASS after E1

**Resolution (2026-11-19).** The maintainer accepted the E1 recommendation,
and A14 now states it. With E1 resolved, dry run 2 passes:

- every aligned finding in part A has a lasting owner (the dry run 1 map
  plus the additions below);
- every affected file is listed;
- deferred items have destinations (D4, D5);
- the structure needs no new owners;
- the promotion change set and teaching plan are recorded (D6 and both
  teaching plans).

Passing does not authorize promotion. Promotion needs separate
authorization.

The original finding follows.

Dry run 2 checked what changed since dry run 1:

- the D1 to D6 decisions;
- the built-in raw-pointer rule;
- the two unsafe casts and the general escape;
- the `forbidden` and selection clarifications.

All of these fit the ownership map and file list from dry run 1, with the
additions below. One item conflicts with a current owner and needs
alignment.

#### E1. What a type cast may change

A14 says `x unsafe cast T` "changes only the type" and that writing a
different stance in `T` is an error. The casting document lets a type cast
change more than the type it points to:

```zax
// today's design (casting, "Qualification" and "Pointer ownership roles")
trusted := myMutable unsafe cast MyType immutable &               // changes qualifiers
observer : Truck * weak = myCar unsafe cast Truck * weak          // changes the pointer kind
exclusive : Truck * unique = myCar as last unsafe cast Truck * unique
```

The concern behind A14 was a type mistake hiding inside a *stance* change,
not qualifiers or pointer kinds.

*Recommendation:* `x unsafe cast T` keeps today's meaning: it may change
the type, the qualifiers, and the pointer kind, the last by the ordinary
transition rules. It never changes a stance. A stance in `T` that differs
from `x`'s is an error. `x unsafe cast as S` changes only a stance. So:

```zax
// candidate syntax
raw : MyValue * = owner.
retyped : MyOther * = raw unsafe cast MyOther *                 // fine: type only
owned : MyValue * last = raw unsafe cast MyValue * last         // error: a type cast can't change a stance;
                                                                // write (raw unsafe cast as last) as last
trusted := myMutable unsafe cast MyType immutable &             // fine, as today: qualifiers
```

The phrase `myCar as last unsafe cast Truck * unique` is unaffected. Its
`as last` is a stance written for the transfer, not a stance changed by the
cast.

Once E1 is aligned and recorded in A14, nothing else blocks promotion.

#### Additions to the dry run 1 map

- **Conversions and casts.** Owns both unsafe casts, with A14's examples
  and the E1 wording. Its "View the storage" rule already says `unsafe cast`
  produces a view, which matches "the same value with a new stance".
- **Operator phrases and operator catalog.** The new phrase
  `unsafe cast as S` is added. The catalog's allocation rows drop `@<` and
  `@!<` (D1).
- **Pointers and arenas.** `unsafe vacate` on a raw `move` or `last` owner
  matches today's "Vacating a raw pointer". `unsafe vacate` on `unique`
  stays an error, as today. The built-in raw-pointer rule and the
  `(view unsafe cast as last) as last` sequence go in the new "Owning raw
  pointers" section.
- **Terms.** New entries for owner, borrower, and pinned (D3). The
  "Open-ended raw allocation" entry retires (D1). Scheduled-allocation
  wording becomes "owning raw pointer".

#### Additions to the teaching plan

**Conversions and casts.** The reader arrives wanting to treat memory or a
value as something it wasn't declared as. What changes: a stance has its own
cast; a type cast never touches stances; and neither cast creates operations
the type lacks. They might wrongly try:

- changing type and stance in one cast;
- expecting `unsafe cast as move` to give a type a `move` it doesn't have;
- expecting a stance cast to make a `readonly` path writable.

**Pointers and arenas** (in addition to dry run 1's plan). The reader needs
to see the one built-in rule (the owner of a raw `copy` pointer may not
write `as last` on it), why constructors can't express it, and the
`unsafe cast as last` escape, in that order.

### Dry run 1 (2026-11-19): FAIL

**Resolution (2026-11-19).** All six items were decided by the maintainer:

- **D1:** `@<` and `@!<` are removed, and every allocation has an owner
  (A11). `unsafe` may make a non-owning pointer an owner (A11).
- **D2:** an owned allocation may move into a raw `move` or `last` pointer
  without `unsafe` (A11).
- **D3 to D6:** the recommendations below were accepted as written.

**Dry run 2 is required** before promotion, to confirm these against the
map below. The files affected by D1 are added to the list under "Files
affected".

The original findings follow.

Part A is coherent on its own. Checking it against the current language
documents found **two design gaps** (D1, D2) and **four decisions** (D3 to D6)
that need alignment before promotion can begin. Everything else is mapped
below, so a second dry run should only need to confirm those six items.

Teachability drove the structure proposal: the main owner, transfer stances,
needs re-authoring around the new model, not patching (D6).

#### Blocking items

**D1. Manual (open-ended) allocations are not covered by part A.** Today
there is a deliberate manual path. `@<` and `@!<` allocate with no cleanup
owner, into an ordinary raw pointer, and the programmer frees it with `reset`,
which needs proof or `unsafe`. Today's rules also treat a plain `@` *assigned*
to an undecorated raw pointer as open-ended:

```zax
// today's design
manual : MyValue * = @<      // no cleanup owner; manual protocol
reset manual                 // safe with proof, otherwise unsafe

ordinary : MyValue *
ordinary = @                 // today: legal, open-ended
```

Part A makes `@` produce a value whose stance is `last`, and a raw pointer
hold a responsibility only when its stance is `move` or `last`. That
contradicts `ordinary = @`, and part A says nothing about `@<`.

*Recommendation:*

- `@<` and `@!<` produce a value with stance `copy`. That is the explicit
  manual path, allowed into raw `copy` pointers, and `reset` keeps today's
  proof-or-`unsafe` rule.
- `@` into a raw `copy` pointer is an error whose message points at `@<`, or
  at declaring the pointer `move` or `last`.

This keeps automatic cleanup decided by stance alone, and makes manual
allocation always visible as `@<`.

**D2. "Opening" an owned allocation into a raw pointer.** Today
`raw : MyValue * = uniqueOwner as last` is allowed when another owner is
proved, or under `unsafe`. Part A's forbidden constructor blocks this
completely, and `unsafe` cannot restore a forbidden operation.

*Recommendation:* hand an owned allocation to the manual path only through an
explicit unsafe form, which keeps the forbidden constructor intact. For
example:

```zax
// illustrative syntax only
raw : MyValue * = unsafe open (uniqueOwner as last)
// the allocation becomes manual: as if made with @<
```

This needs a decision on whether to keep the capability at all, and on its
spelling.

**D3. Which discussion words become Zax terms.** Part A relies on words
that the writing rules say must be defined before teaching, or replaced by
plain description.

*Recommendation:*

- Make **owner** and **borrower** defined terms. They carry the central rule
  (A2).
- Make **pinned** a term for a value declared `loan`.
- Do not make "grant" or "stand-alone borrower" terms. Say "the stance the
  reference was given" and "a reference or raw-pointer parameter, result,
  member, or type alias" in place.
- Replace "scheduled raw allocation" with "owning raw pointer" (`* move` or
  `* last`). Keep "open-ended" for `@<`.
- `via` is a keyword, not a term.

**D4. Homes for deferred items** (needed before 039 closes).

*Recommendation:*

- The part C follow-ups (array element stances, slice parameters, optional
  `move source` and `last source`, optional references, lambda reference
  captures) become the initiating input for work item 040.
- The long-term deferrals (foreign-function declarations, `.=` on pinned
  values, raw owners compared with `unique`, borrows before a raw-to-`unique`
  conversion, `deep` on pointer layers, `forbid`) go into one new indexed raw
  file, `project/raw/pointer-ownership-deferrals.md`, with their examples.
- `project/raw/pointer-stances.md` retires, and its index entry is removed.

**D5. Part D captures.**

*Recommendation:* both stay out of the 039 promotion.

- The compiler-agreement principle becomes a new indexed raw file for
  separate discussion, destined for `language/principles.md`.
- The four documentation lessons are proposed as a small separate change to
  `project/documentation.md`, discussed and authorized on their own.

**D6. Promotion shape.**

*Recommendation:* one authorized change set, applied in three reviewable
steps:

1. Re-author transfer stances, and rewrite the affected sections of pointers
   and arenas, construction and destruction, and function invocation. This is
   all written by the agent that holds the discussion.
2. The mechanical `& S` to `S &` respelling, and `loan` added to read-only
   reference parameters, across the other owners. This is the only part
   suitable for delegation.
3. Terms, routers, the raw-file retirement, and the 040 creation (D4).

After that, a cold-reader review by an agent that has not read this record,
as `project/documentation.md` requires.

#### Ownership map

| Part A section | Lasting owner |
| --- | --- |
| A1 to A5 (what a stance means, owners and borrowers, passing, `as`, members) | transfer stances |
| A6 (defaults, required stances, inheritance) | transfer stances; inheritance for local bindings also in declarations and bindings |
| A7 (by-value parameters, `via`) | transfer stances; selection in function invocation |
| A8 (results) | transfer stances; result construction in function invocation; mandatory elision for `loan` results in lifetimes and references |
| A9 (constructor receiver stances, generation, selection, temporaries) | construction and destruction; the selection rule in function invocation |
| A10 (fallback, forbidden, declaration errors) | construction and destruction; fallback summary stays in transfer stances |
| A11, A12 (pointers, chains, `@`, `p.`, `unique`) | pointers and arenas; `p.` and trapping in Nothing instances; raw pointers from references in safety and analysis |
| A13 (slices, joins, optionals) | arrays and slices; optional values |
| A14 (erasure, casts) | pointers and arenas (observers); casting for `unsafe cast` |
| A15 (spelling) | qualifiers (ordering); every owner that writes stances |

#### Files affected

**Re-authored or substantially rewritten:**

- `language/transfer-stances.md`: re-authored (see the teaching plan).
- `language/pointers-and-arenas.md`. These sections change:
  - "Raw pointers";
  - "Scheduled raw allocations", which becomes owning raw pointers;
  - "Open-ended raw allocations" (D1);
  - "Returning a scheduled raw allocation";
  - "Direct allocation assignment";
  - "Ownership transitions";
  - the `pointerTo(container.items[0])` example under `unique`;
  - the observers;
  - new sections on stances on pointed-to values and on borrowing through
    `p.`.
- `language/construction-and-destruction.md`. These change:
  - "Declaring which transfers a type allows";
  - "Generated `copy` construction and assignment", including the sentence
    "A constructor has no receiver stance";
  - the `MyDeepOnly copy` example;
  - "Demand-time generation";
  - a new section on constructor receiver stances and selection.
- `language/function-invocation.md`. These change:
  - "Transfer-aware value and reference parameters";
  - "Partial-order preference": the receiver bullets, the stance table, and
    "adding a `copy` version never changes";
  - "Results with no declared stance";
  - "Opt-in result initialization";
  - "Receiver-origin `self` results";
  - `via`.

**Targeted edits:**

- `language/qualifiers.md`: stance on layers, and canonical ordering.
- `language/declarations-and-bindings.md`: inherited borrower stances, alias
  overlays landing on the nearest layer, the inference exception, and the
  `= @` examples.
- `language/lifetimes-and-references.md`: unmarked-loan examples, and
  elision for `loan` results.
- `language/optional-values.md`: two stances, and the "Transfer stance
  reference" table.
- `language/nothing-instances.md`: the `p.` carve-out, and `_` described as
  a parameter bound implicitly.
- `language/safety-and-analysis.md`: raw allocation adoption (D1, D2), and
  raw pointers from references of unseen origin.
- `language/arrays-and-slices.md`: slices as checked borrowers; joins
  unchanged.
- `language/terms.md` (D3):
  - entries for Declared stance, Loan, Receiver stance, Transfer stance,
    Open-ended raw allocation, and Vacant pointer;
  - new entries for owner, borrower, and pinned.

**Removing `@<` and `@!<` (D1):**

- `pointers-and-arenas.md`: the allocation-forms table, "Direct allocation
  assignment", "Open-ended raw allocations", reset and vacate, costs, and
  diagnostics;
- `function-invocation.md`: the `@<` and `@!<` result rows and examples;
- `operator-catalog.md`, `operators.md`, `source-structure.md`: the reserved
  tokens;
- `declarations-and-bindings.md`, `lifetimes-and-references.md`,
  `safety-and-analysis.md`: examples and the open-ended rules;
- `terms.md`: the "Open-ended raw allocation" entry retires.

**Spelling migration and small mentions:**

- partials, lambdas and callable composition, iteration, composition (the
  wrapper table also changes), type definitions, identity types, generics;
- `using.md` and `switch.md` (receiver-stance mentions);
- `source-structure.md`, `operator-catalog.md`, and `operators.md` (`= @`
  examples).

**Routers and project files:**

- `index.md`: check the router wording for transfer stances and pointers.
- `project/raw/pointer-stances.md`: retired.
- `project/raw/README.md`: index updated.
- New raw files and work item 040 (D4, D5).
- `project/README.md`: the active-work pointer, at closure.

#### Structure proposal

No new owner documents or directories. Transfer stances stays the single
owner of the stance model. Pointers and arenas keeps pointer ownership,
construction and destruction keeps constructor selection and generation, and
function invocation keeps overload selection. Each owner links to transfer
stances for the model rather than restating it. Raw and deferred material
stays in `project/raw/` and work item 040, never linked from language owners.

#### Teaching plan

**Transfer stances (re-authored).** The reader arrives wanting to know what
`as move` or `loan &` does. They know ordinary copying and references from
other languages. What changes for them: a stance says what may be built from
a value, and whether you own the value decides what you may write. What they
might wrongly try:

- `MyValue & loan`;
- moving through an alias of a `move` value;
- copying through a `move &` parameter;
- expecting `x : T deep = y` to make a deep copy;
- `via` on a reference.

Proposed order, leading with source each time:

1. Why stances exist (kept).
2. What a stance says: the five stances as "what may be built from this
   value"; `copy` includes `deep`; pinned values.
3. Writing a stance at a use: no stance means copy where allowed, otherwise
   lend; reading a call's effect on your variable.
4. Owners and borrowers: what each may write; the `remember` and `drain`
   examples; "a reference stance is a permission".
5. Passing to references: the two tables, which differ only in the borrower
   rows.
6. Declaring stances: locals and pinned values; references (written on
   parameters, inherited on locals); receivers as parameters bound
   implicitly; by-value parameters and `via`; results; members.
7. Temporaries and the implicit `as last`.
8. Fallback, and what leaving a stance out of a type means (summary; the
   full rule in construction and destruction).
9. Pointers, optionals, arrays, and variants: short sections that hand off to
   their owners.
10. Costs, diagnostics, and source stability.

**Pointers and arenas.** The reader arrives wanting to allocate or hand out
pointers. What changes: a raw pointer owns exactly when it is `move` or
`last`; you borrow through `p.`; `unique` allows no raw borrowers. They might
wrongly try `p : MyValue * = @`, `= movePtr as move` to borrow, or a raw
pointer into a `unique`.

**Construction and destruction.** The reader arrives writing constructors.
What changes: a declared stance is the constructor's receiver stance;
generated constructors have none; forbidden and bodyless declarations
compete in selection. They might wrongly expect forbidding to suppress
generation, or a tied forbidden candidate to step aside.

**Function invocation.** The reader arrives wondering which overload runs.
What changes: the receiver is one more input; `via`; how `as last` reaches
`move` versions. They might wrongly expect the variable's declared stance to
pick a version.

#### Next steps

1. Decide D1 to D6.
2. Record the decisions in part A and part C.
3. Run dry run 2. It should only need to confirm D1 to D6 against this map.
4. Promotion, after separate authorization.

**Risk: context.** This session is long. If promotion is to be written by the
agent that holds the discussion, it is best started soon after dry run 2.
Otherwise part A, as reviewed, is the state a fresh session would promote
from.
