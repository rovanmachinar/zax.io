# Zax transfer stances

| Field | Value |
| --- | --- |
| Status | Current conceptual design |
| Audience | Human developers reading, writing, or evaluating Zax |
| Applies To | Programmer-visible `loan`, `copy`, `move`, `last`, and `deep`; who may write which stance; when a stance must be written; not a formal specification |
| Implementation State | Not established by this repository |
| Owns | The transfer-stance mental model; what each stance says may be built from a value; owners and borrowers and which stances each may write; pinned values; writing a stance at a use; fallback; source state after a transfer; when a stance must be written, including at a last use, in generic bodies, and per item of an expanded pack; declared stances on locals, members, references, receivers, by-value parameters (`via`), results, and type aliases; one stance per layer, including array elements and slices; an unmarked outer stance taking the inner stance; stances written inside reference types; finishing elements through a borrowed array; receivers as parameters bound implicitly; temporaries; member and binding stances; common costs, diagnostics, and source stability |
| Does Not Own | Exact callable selection ([function invocation](function-invocation.md)); a type's `transfer` list, generated and defaulted operations, constructor receiver stances, and lifecycle sources ([construction, replacement, and destruction](construction-and-destruction.md)); identity `transfer` lists ([identity types](identity-types.md)); inferred declarations ([declarations and bindings](declarations-and-bindings.md)); cohesive [exceptional result flow](except.md); composition publication and wrapper eligibility ([Zax composition](composition.md)); qualifier axes ([qualifiers](qualifiers.md)); `unsafe cast` ([conversions and casts](casting.md)); optional wrapper cleanup ([optional values](optional-values.md)); complete [variant transfer](variants.md#copy-deep-move-and-last) or [union transfer](unions.md#whole-union-operations); [reference lifetime](lifetimes-and-references.md); or [pointer ownership and allocation](pointers-and-arenas.md) |
| Source / Provenance | Legacy function, pointer, casting, and constructor input, reconciled with current invocation, construction, qualifier, optional, operator, pointer, and documentation design |

## Why transfer stance exists

Start with two ordinary lines:

```zax
a := b
c = d
```

You probably expect `a` to get its own copy of `b`, and `c` to get the value of
`d`. For simple values, that is all there is to it. For a value that owns
resources, such as a buffer, a file, or a database connection, these lines
leave several questions open:

- Does copying duplicate every resource, or do the two values share some?
- Does the new value need to be completely independent of the old one?
- May the new value take the resources away from the old one?
- Can the old value still be used afterward?
- Is this the last time the program uses the old value?
- What does it cost in allocation, copying, and destruction?

Zax answers these questions with one word written after `as`, called a
**transfer stance**:

```zax
ordinary := source              // copy: source keeps its value
independent := source as deep   // a copy that shares nothing with source
transferred := source as move   // the contents may move out; source stays usable
finished := source as last      // source is finished; it will only be destroyed
```

Writing a stance does not copy or move anything by itself. It tells the
compiler what you allow. The code that receives the value does the work: in
these lines, the constructor that builds each new variable. In this document,
whatever receives a value is called the **consumer**: a variable being
initialized, a function parameter, an assignment, or a function's result.

A **transfer** is any use that builds or assigns a new value from an existing
place: initializing a variable, passing an argument by value, assigning, or
returning a value. Lending a value through a reference is not a transfer.

Stances are promises about intended use, and the compiler helps you keep
them: it reports a call that copies a value you said must only be moved, or
takes from a value that was only lent. They are not a security barrier. Code
with writable access to a value can still change its members in ordinary
ways, and stances do not try to stop that. Where a program must go beyond
what its stances allow, `unsafe` provides an explicit escape, described in
[conversions and casts](casting.md).

### Reading the examples

The examples use a few Zax forms that other documents teach in full:

| Form | Meaning | Explained in |
| --- | --- | --- |
| `name final : (results)(inputs) = { }` | A function: its result slots, then its inputs, then its body | [function invocation](function-invocation.md#callable-prototypes-and-visible-contracts) |
| `+++` | A constructor declared inside a type | [construction, replacement, and destruction](construction-and-destruction.md) |
| `---` | A destructor | [construction, replacement, and destruction](construction-and-destruction.md) |
| `place .= value` | Reconstructive replacement: end the old value in place and build a new one | [construction, replacement, and destruction](construction-and-destruction.md) |
| `_` | The receiver: the instance a type's function was called on | [function invocation](function-invocation.md#receiver-operands) |
| `Type loan &` | A reference to an existing place rather than a new value; the word before `&` is the stance of the value it refers to | [lifetimes and references](lifetimes-and-references.md) |
| `readonly`, `mutable writable` | Qualifiers saying whether a path may change the value; after a function's inputs, as in `()() readonly`, they apply to the receiver | [qualifiers](qualifiers.md) |
| `+++ replacement` | A replacement constructor: what `.=` runs to give an existing variable a new value | [construction, replacement, and destruction](construction-and-destruction.md#custom-replacement) |
| `(: Source)`, `[{ … }]` | A new value built in place: `(: Source)` builds a default `Source`; `[{ … }]` lists the constructor's inputs | [construction, replacement, and destruction](construction-and-destruction.md#construction-packets) |
| `name$ final : (Items ... : type)()(items ... : Items)` | A generic function: `$` marks it, `Items ... : type` is a list of types chosen per call, and `items ... : Items` receives one argument of each | [generics](generics.md), [variadics](variadics.md) |
| `result # : Type` | A result the caller may ignore; without `#`, ignoring a result is an error | [function invocation](function-invocation.md#required-and-discardable-results) |
| `once` | A function that can be called on the type itself, as in `MyType.inspect()`, without an instance | [Nothing instances](nothing-instances.md) |

## What a stance says

A stance belongs to a value, and says **what may be built from that value**:

| Stance | What may be built from the value |
| --- | --- |
| `loan` | nothing; the value can only be used where it is, or lent |
| `copy` | a copy, or an independent copy (`deep`) |
| `move` | only a new value that takes its contents |
| `last` | only a terminal transfer, after which the value is finished |
| `deep` | only an independent copy |

"Built from a value" means one of three things: a copy of it, a new value
that takes its contents, or a terminal transfer that finishes it. Reading a
value to compute something new is not building from it.
`size := buffer.size()` makes a new `Integer` by reading `buffer`, and is
fine even when `buffer` is pinned.

A stance does not say how the value itself was built. It only restricts what
may later be built from it:

```zax
myValue : MyValue copy      // copies may be built from myValue
report : MyReport move      // anything built from report must take its contents
pinned : MyDevice loan      // nothing may ever be built from pinned
archive : MyArchive deep    // only independent copies may be built from archive
```

`copy` includes `deep`. A deep copy is still a copy that leaves the original
unchanged; it just shares nothing with it. The reverse does not hold: a value
declared `deep` refuses an ordinary copy, because an ordinary copy might share
resources with it.

A value declared `loan` is called **pinned**. It can be used and lent, but
nothing is ever copied, moved, or finished out of it. Its address stays the
same for as long as it exists. Pinning applies to the value as a whole; its
members follow their own stances, as [members](#members) explains.

When stances are listed together, they are written in the order
`loan copy move last deep`. A type's `transfer` list uses the same order but
never includes `loan`.

### Where a stance is written

A stance is written in the same place as a layer's qualifiers, last among that
layer's words. For a reference, the stance is written before `&`, because a
reference has no layer of its own: its stance is the stance of the value it
refers to.

```zax
source : MyArchive readonly copy &           // a reference: the stance is the referred-to value's
value : MyValue loan * unique                // a pointer: MyValue's stance before *, the pointer's after it
slot : MyPayload move? loan                  // an optional: the payload's stance before ?, the optional's after it
messages : MyMessage move[4] move            // an array: each element's stance before [, the array's after ]
window : MyMessage loan[]                    // a slice: only the elements' stance; a slice has none of its own
```

A pointer has two values, the pointer and what it points to, so it can carry
two stances. [Pointers](#pointers) summarizes what that means. An optional
and an array likewise carry one stance for what they hold and one for
themselves; a slice, like a reference, carries only the stance of what it
reaches. [Optional values](#optional-values) and
[arrays and slices](#arrays-and-slices) explain them.

When the outer stance of an optional or array is left out, it is the same as
the stance of what it holds: `MyMessage move[4]` is a `move` array, and
`MyPayload move?` is a `move` optional. That is always allowed, and it means a
stance arriving through an alias or a generic type parameter never makes a
declaration fail merely because the outer stance is unmarked.

## Writing a stance at a use

`as` is how a use says what it allows to happen to a value. It does not change
any stance; it states, for this one use, what may be built:

```zax
keep(report as move)        // this use may take report's contents
keep(myValue)               // no stance written
```

**No stance means `copy` where the value allows a copy, and otherwise the
value is lent.** A reader who sees no stance can rely on nothing having been
moved out of the value, and on the value not being finished. A function given
writable access may still change its contents. The compiler never adds `move`
or `last` for you.

```zax
show final : ()(value : MyReport readonly loan &) = {
}

keep final : ()(value : MyReport) = {
}

plain : MyReport            // no stance declared: copy
report : MyReport move

keep(plain)                 // copies plain
show(report)                // lends report, although it forbids copies
keep(report)                // error: keep must build a copy, and report forbids copies
keep(report as move)        // fine
```

You may write several stances in a row. Each one replaces the one before it,
so the last one counts:

```zax
result := source as move as copy
// `as copy` is written last, so result is built by copying
```

Two stance words need two `as` keywords:

```zax
result := source as move copy // error: write `as move` or `as copy`
```

A stance can also be written on an expansion of several arguments, such as
`from items as last`. It then applies to each item separately, as
[a stance on several items](#a-stance-on-several-items) explains.

## Owners and borrowers

Who holds a value decides which stances may be written on it.

An **owner** holds its own value: a local variable or member you declared, a
by-value parameter (the function owns it), or the value an owning pointer
points to. A **borrower** reaches a value someone else owns: a reference, a
raw pointer that holds only an address, or the receiver `_`.

The owner of a value decides how much of it to give up. The owner of a `copy`
value may also write `as move` or `as last` on it, and the owner of a `move`
value may also write `as last`. The one exception is a borrowing raw pointer:
its owner may not write `as move` or `as last` on the pointer itself, because
it holds nothing to hand over, as
[pointers, allocation, and arenas](pointers-and-arenas.md#owning-raw-pointers)
explains:

| Owner's value stance | The owner may write |
| --- | --- |
| `loan` | nothing; only lend it |
| `copy` | `as copy`, `as deep`, `as move`, `as last` |
| `move` | `as move`, `as last` |
| `last` | `as last` |
| `deep` | `as deep` |

A borrower may write only the stance it was given, and nothing else:

| Borrower's stance | The borrower may write |
| --- | --- |
| `loan` | nothing; only lend it |
| `copy` | `as copy`, `as deep` |
| `move` | `as move` |
| `last` | `as last` |
| `deep` | `as deep` |

Lending is always allowed, for owners and borrowers alike. The owner of a
`loan`, `last`, or `deep` value may write only that same stance, so a pinned
value stays pinned even for its owner.

```zax
report : MyReport move
keep(report as move)       // fine
keep(report as last)       // fine: the owner of a move value may also write as last
```

**Why a borrower can't take more than it was given.** The caller relies on
the stance it wrote to know what state its variable is in afterward:

```zax
remember final : ()(input : MyMessage copy &) = {
  stash(input as last)       // error: input was given copy
}

message : MyMessage
remember(message)            // the caller expects message unchanged
message.send()               // would find a finished message if stash could take it
```

**Why a borrower given `move` can't copy.** The value it reaches might be
declared `move`, which forbids copies, and the borrower can't see how that
value was declared:

```zax
drain final : ()(input : MyMessage move &) = {
  keep(input as move)        // fine
  log(input)                 // error if log takes MyMessage by value: input was given move
}

report : MyMessage move      // its owner said: never copy report
drain(report as move)        // if drain could copy input, report would be copied after all
```

So "you may keep copying, but you can't take it over" is simply a `copy`
reference:

```zax
looksLikeANormalCopy final : ()(value : MyValue copy &) = {
  keep(value)                // fine: copies
  keep(value as last)        // error: only copy was given
}
```

Inside a function, the receiver `_` is a borrower too. In a version declared
`last`, `_` may only be lent or written `as last`:

```zax
MyDocument :: type {
  publish final : ()() last = {
    _.inspect()                // no stance: _ forbids a copy here, so it is lent
    (_ as copy).snapshot()     // error: _ was given last
  }
}
```

## Passing a value to a reference

A value can be passed to a reference parameter when the parameter's stance is
one its holder may write. From a value you own:

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

A reference parameter that may change the caller's value needs its stance
written at the call, so a reader sees it: `as move` for `move &`, `as last`
for `last &`. A `deep` value needs `as deep`:

```zax
drain final : ()(value : MyType move &) = {
}

myType : MyType move
drain(myType)                // error: a reader assumes myType is unchanged
drain(myType as move)        // fine: shows that myType may be emptied
```

**A reference stance is a permission, not a promise.** The function may or
may not use it, so the caller plans for the most the parameter allows:

| Parameter | The caller assumes afterward |
| --- | --- |
| `loan &` or `copy &` | nothing was moved out and it is not finished; through writable access its contents may have changed |
| `move &` | still usable, possibly emptied |
| `last &` | finished: it may only be destroyed or replaced |

```zax
finish final : ()(value : MyType last &) = {
  // may finish value, or may not
}

myType : MyType last
finish(myType as last)       // fine: myType counts as finished, whatever finish did
finish(myType as last)       // error: myType was already given up
```

The second call is an error because the caller gave `myType` up, not because
`myType` is known to be empty. That prevents finishing a value twice. The
destructor at the end of the scope handles both states, as destructors
already must.

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

### Stances inside the referred-to type

The value a reference refers to may itself hold values: an array holds
elements, and an optional holds a payload. A stance written on those inner
values inside a reference type says what the function may do with them. The
caller's inner values must allow it, by the table above for passing a value
you own:

```zax
inspect final : ()(values : MyMessage loan[4] readonly loan &) = {
}
drain final : ()(values : MyMessage move[4] mutable writable loan &) = {
}
copyAll final : ()(values : MyMessage[4] readonly loan &) = {
}

messages : MyMessage move[4] move
inspect(messages)          // fine: move elements allow loan
drain(messages)            // fine: move elements allow move
copyAll(messages)          // error: unmarked elements are copy, and move elements forbid copies
```

An unmarked inner stance means `copy`, as it does everywhere else, so
`MyMessage[4] readonly loan &` accepts only arrays whose elements allow
copying. Naming `loan` for the inner values accepts any caller and only looks.
The same rule applies to the payload of an optional (`MyPayload move? loan &`)
and, for a pointer to a pointer, to the value the inner pointer points to. Arrays passed this way are covered
in full by
[element stances in an array reference](arrays-and-slices.md#element-stances-in-an-array-reference).

## Copy

`copy` is what you get when you write no stance on a value that allows it:

```zax
source : Document
snapshot := source
source.save()       // source is still used after the copy
```

`source` keeps its value. A copy is not necessarily independent of the
original. Depending on the type, copying may:

- duplicate the value's own storage;
- share backing storage with the original;
- increase a reference count;
- duplicate handles or references;
- allocate;
- share storage until one of the values changes (copy-on-write); or
- run code the type's author wrote.

Whatever it does, a copy must keep the type's rules about ownership,
qualifiers, and lifetimes. Sharing resources is allowed; breaking those rules
is not.

Writing `as copy` explicitly is useful when the compiler asks you to say what
you want at a variable's last use, as
[when a stance must be written](#when-a-stance-must-be-written) explains:

```zax
relay final : ()(input : Message) = {
}

relay(message as copy)   // a deliberate copy, even at message's last use
```

## Deep

Use `deep` when ordinary copying might share resources and you need a
duplicate that shares nothing with the original:

```zax
template : Project
editableProject := template as deep   // editableProject shares nothing with template
```

After a `deep` copy, changing or destroying one value should not affect
resources the other one needs. Each type decides exactly what "shares nothing"
means for it: a type with interned immutable data, an identity, cycles, or
external state may keep some of that shared. A member declared with its own
`copy` stance, such as `styles : StyleSet copy`, is copied rather than
deep-copied by a generated `deep` operation, so it stays shared. The type's
author decided that, and callers cannot override it.

A value declared `deep` allows only independent copies. A use with no stance
written is an error, because it would make an ordinary copy:

```zax
archive : MyArchive deep
keep(archive)              // error: no stance means copy, and archive allows only deep
keep(archive as deep)      // fine
```

`deep` never falls back to `copy`:

```zax
editableProject := template as deep
// error if Project defines no deep operation
```

Quietly making an ordinary copy would give you a value that might share
resources, which is exactly what you asked to avoid. You can still pass a value
`as deep` to a `loan` reference, because a loan builds no new value, so nothing
can end up shared.

The compiler cannot check every promise a programmer-written `deep` operation
makes. Defining a `deep` operation is still a promise to its callers that it
produces an independent value.

The basic value types have `deep`: the integer types, fixed-point and
floating-point numbers, `Boolean`, characters, and enums. For each of them a
copy already shares nothing with the original, so `deep` does the same as
`copy`. This lets a type built from them, and from other types with `deep`,
get its own `deep` with `= default`. Pointers, unions, and lambdas have no
`deep`: copying an address or raw bits does not make an independent value.

A number that names a resource, such as a handle ID, is the exception: a deep
copy of the number would duplicate the ID, not the resource. Give such an ID
its own [identity type](identity-types.md#transfer-lists-on-identities) whose
`transfer` list leaves out `deep`. A type that holds it then cannot get `deep`
by accident:

```zax
MyFileId :: identity admit expose transfer copy type U64   // no deep

MyFile :: type transfer copy deep {
  id : MyFileId

  +++ final : ()(source : MyFile readonly deep &) = default
  // error: MyFileId has no deep, so a default deep cannot be built
}
```

A value declared `deep` but initialized with an ordinary copy, as in
`shared : MyArchive deep = original`, is built by whatever constructor the
type provides for that case. A type can make such values independent with a
constructor that declares a `deep` receiver stance; see
[construction, replacement, and destruction](construction-and-destruction.md).

## Move

`move` lets the consumer take the contents of a value, while the original
variable stays usable:

```zax
buffer : MyBuffer
nextBuffer := buffer as move   // MyBuffer's move constructor builds nextBuffer
size := buffer.size()          // fine: buffer is empty, but still a valid buffer
```

When `MyBuffer`'s `move` constructor runs:

- `nextBuffer` is a complete, valid buffer;
- `buffer` is still a `MyBuffer`, and every ordinary operation on it still
  works, although it may now be empty;
- each resource `buffer` owned ends up owned exactly once: it moved to
  `nextBuffer`, stayed in `buffer`, or was released; if the constructor copies
  a resource instead, the copy is a new resource that `nextBuffer` owns; and
- `buffer` is destroyed later, at the end of its scope, as usual.

So after the type's `move` operation, the original value still behaves like a
normal value of its type. A type that cannot leave a fully usable value
behind should not define `move`; it can define `last` instead, as
[`last` without `move`](#last-without-move) shows. A function given a `move`
reference may also change the caller's members through ordinary writable
access, as [members](#members) explains; this guarantee is about what the
type's own `move` leaves.

`as move` permits taking the contents; it does not force it. A type's `move`
constructor may keep some resources in the original, or copy them, as long as
both values end up valid.

## Last

`last` means "I am done with this variable". It is useful for resources that
are expensive to duplicate, because the consumer can take them over instead.

Consider a command that owns a private database connection while it prepares a
request:

```zax
postDatabaseCommand final : ()() = {
  command : DatabaseCommand
  command.preparePrivateDatabase()

  submit(command as last)
}
```

If `submit` took `command` with `move`, it would have to leave `command` fully
usable, and for this type a usable command needs its own database connection.
With `last`, `submit` may take the connection itself, because nothing will use
`command` again before the function ends.

`as last` does not end `command` early:

- `command` still exists until the end of its scope;
- its destructor still runs there, exactly once;
- enough state remains for that destructor to run; and
- the destructor does not release anything `submit` took over.

After `as last`, only two things may happen to the variable: it is destroyed,
or it is replaced with `.=`. Any other use is an error:

```zax
postAndReport final : ()() = {
  command : DatabaseCommand
  command.preparePrivateDatabase()

  submit(command as last)
  printStatistics(command)
  // error: command is finished; submit may have taken what printStatistics needs
}
```

If the type documents an operation as safe on a finished value, you can call it
inside an [intent acknowledgement](intent-acknowledgements.md):

```zax
postAndRelease final : ()() = {
  command : DatabaseCommand
  command.preparePrivateDatabase()

  submit(command as last)

  intent<terminal-source-reuse>{
    command.releaseRemainingHandle()
  }
}
```

Acknowledgement names use the word "terminal" for a finished variable.
Zax has no declaration that marks which operations are safe on a finished
value. The acknowledgement records that you mean it, and the type's
documentation says which operations make sense. The type's own replacement
constructor and destructor need no acknowledgement, because handling a finished
value is part of their job.

**After `as last`, the variable is finished, whichever operation actually
ran.** If the type has no `last` operation and `move` or `copy` ran instead,
the variable still counts as finished. That way `as last` means the same thing
for every type, and your code stays correct if the type later adds a real
`last` operation. Write `as last` at a variable's last use even when its type
has no `last` operation.

### When `move` and `last` differ

After `move`, the original must still be a fully usable value. After `last`, it
only has to survive being destroyed or replaced. For many types that makes no
difference: an empty buffer, a null handle, or an empty collection is usable
and also safe to destroy. Such a type defines only `move`, and `as last` falls
back to it. Most types therefore declare `transfer copy move` or
`transfer move`. Leaving `last` out of the list does not stop callers writing
`as last`; it means `as last` always behaves like `as move` for that type.

A separate `last` operation is worth writing only when not having to leave a
usable value behind lets it take more, or skip work:

- **Private connection.** Every usable command needs its own connection, so
  `move` must keep one in the original or open a new one. `last` can take it.
- **Reusable backing allocation.** A buffer's `move` may promise that the
  original can be written to again right away, so `move` leaves some
  allocated storage behind. `last` can take the contents and the storage.
- **Registration.** A live service must stay registered, so `move` keeps or
  renews the registration. `last` can hand it over or cancel it.
- **Internal pointers.** After `move`, a structure that points into itself
  must repair those pointers so its methods keep working. After `last`, only
  destruction remains, so the repair can be skipped.

### `last` without `move`

Some types assume, in every method, that they hold their resource: a non-null
handle, a held lock, or a live registration. An empty but usable value would
break that assumption, so such a type cannot define `move`. It can define
`last`, because after `last` only its destructor and replacement constructor
ever see the released state:

```zax
MyLockGuard :: type transfer last {
  // every method assumes the lock is held
}

// each call below is a separate alternative
holdUntilDone(guard as last)   // hands the lock to another scope
holdUntilDone(guard as move)   // error: no move, and no copy to fall back to

MyShared :: type transfer copy last {
  // always points to something: copy adds a reference, last takes the
  // source's reference without adding one; move would need a null state
}
```

For a type like `MyShared`, `as move` falls back to `copy`: the caller said it
will use the variable again, and a copy keeps that promise. At a last use,
`as last` avoids the copy.

An unmarked variable of a type that allows only `last` is still `copy`: the
stance says which stances may be written, and the type's `transfer` list says
which operations exist. For such a variable only `as last` actually works:

```zax
MyTicket :: type transfer last {
  id : Integer
}

ticket : MyTicket         // unmarked: copy
keep(ticket)              // error: written as copy, and MyTicket has no copy
keep(ticket as move)      // error: MyTicket has no move, and move can't fall back to last
keep(ticket as last)      // fine
```

### Refreshing a value with `move` and `.=`

The difference between `move` and `last` pays off when a value is reused. A
command that submits several queries over one private connection can hand out
its results with `move`, refresh itself with `.=`, and give up the connection
only at the end:

```zax
DatabaseCommand :: type transfer copy move last {
  connection : PrivateConnection
  query : Query
  results : ResultSet

  +++ final : ()(source : DatabaseCommand mutable writable move &) = {
    // takes source.results and copies source.query; opens a connection of
    // its own, so source keeps its connection and stays fully usable
  }

  +++ final : ()(source : DatabaseCommand mutable writable last &) = {
    // takes source.results and source.connection; source only needs to
    // survive destruction or replacement
  }

  +++ replacement final : ()(nextQuery : Query) = {
    if _.connection.wasTaken() {
      _.connection .= PrivateConnection.open()
    } else {
      _.connection.resetSession()
    }
    _.query .= nextQuery as last   // the last use of nextQuery
  }
}

runBatch final : ()(queries : Query copy[]) = {
  command : DatabaseCommand
  command.prepare(queries[0])
  submit(command as move)        // the move constructor runs: results move out; the connection stays
  command .= [{ queries[1] }]    // refresh: the replacement constructor reuses the connection
  submit(command as last)        // the last constructor runs: it takes the connection too
}
// submit takes a DatabaseCommand by value
```

A replacement constructor must cope with every state the destructor copes
with, including whatever `last` leaves behind. After `as last`, `.=` runs the
replacement constructor as usual. How the type tells whether a resource is
still there is up to its author: `wasTaken()` above is the author's own
method. A type can instead hold the resource through something that can be
empty, such as `connection : PrivateConnection * unique`, and test whether it is
empty. The rules for replacement constructors are in
[construction, replacement, and destruction](construction-and-destruction.md).

## Loan

A `loan` lets a function use a value where it is. Nothing is built from the
value: the function cannot copy it, move it, or take it. Whether the function
may change the value is a separate question, answered by `readonly` or
`mutable writable`. A `mutable writable` loan may change the caller's value in
place, including its members, but still cannot keep or copy the whole value.

Most functions only read or change their reference inputs, so most reference
parameters are written `loan`:

```zax
retain final : ()(input : MyType) = {
}

display final : ()(input : MyType readonly loan &) = {
  print(input.label)        // fine: reading a member builds nothing
  retain(input)             // error: input is a loan; retain would copy it
}

fill final : ()(buffer : MyBuffer mutable writable loan &) = {
  buffer.append(data)       // fine: a mutable loan may be changed in place
}
```

A reference that needs to copy the value, or take from it, is written
`copy &`, `move &`, `last &`, or `deep &` instead:

```zax
retainCopy final : ()(input : MyType readonly copy &) = {
  retain(input)             // fine: this reference may copy
}
```

### Writing `as loan`

`as loan` restricts a use to consumers that build nothing, so a reader sees
that the call takes nothing from the value. Any value may be lent. A lent
value can only be passed to a reference parameter, or used as the receiver of
a function that declares no receiver stance:

```zax
display(value as loan)
```

It matters when a function has two overloads, one taking a loan and one taking
a value, and the call writes no stance. You might expect the by-value overload
to win. It does not: you must write `as loan` or `as copy`. The two overloads
can behave quite differently. A mutable loan lets the function change your
variable, which a by-value copy cannot, and a by-value function may keep the
value it receives:

```zax
update final : ()(input : MyType mutable writable loan &) = {
}

update final : ()(input : MyType) = {
}

update(value)           // error: both fit; write `as loan` or `as copy`
update(value as loan)   // the loan overload
update(value as copy)   // the by-value overload
```

A by-value parameter can't accept a lent value, because it would have to be
built from that value:

```zax
byValue final : ()(value : MyValue) = {
}

byValue(myValue as loan)     // error: a by-value parameter must be built, and a loan builds nothing
```

Complete selection is taught by
[function invocation](function-invocation.md#a-loan-or-a-copy-must-be-stated).

### Pinned values

A variable you own may be declared `loan`. It is then pinned: it can be used
and lent, but nothing is ever built from it, and it stays at the same address
for as long as it exists:

```zax
device : MyDevice loan = openDevice()
configure(device)                    // fine if configure takes MyDevice loan &
keep(device)                         // error: nothing may be built from device
keep(device as move)                 // error: the owner of a loan value may only lend it
```

A by-value parameter may be pinned too. The caller still decides how it is
built; inside the function, nothing is built from it:

```zax
registerDevice final : ()(device : MyDevice loan) = {
  configure(device)                  // fine if configure takes MyDevice loan &
  devices.add(device)                // error if add takes MyDevice by value: device is pinned
}

registerDevice(myDevice)             // fine: device is built by copy
registerDevice(myDevice as move)     // fine: device is built by move
```

A value the caller doesn't name can be pinned only by building it directly
into a pinned variable, as [results](#results) explains.

### Everything derived from a loan stays on loan

A loan would be easy to get around if you could make a new reference from it
that allows copying. So anything made from a loan that can reach the whole
value is also a loan:

- a reference initialized from it is a `loan`;
- a lambda that captures it by reference also holds a loan, and a lambda
  cannot capture a copy of it;
- converting it to the underlying type of an
  [identity type](identity-types.md) gives another loan, so a conversion that
  would copy is an error;
- passing `_` on from a loan receiver passes on a loan; and
- a raw pointer to its value carries the loan on the value it points to, as
  [pointers](#pointers) explains.

```zax
display final : ()(input : MyType readonly loan &) = {
  other : MyType readonly copy & = input   // error: a reference that may copy, bound from a loan
  later := [[ input ]] ()() {              // error: [[ input ]] captures a copy of input
    show(input)
  }
  kept := [[ input: view : & ]] ()() {     // fine: captures a reference, which is still a loan
    retain(view)                           // error: view is a loan; retain would copy it
  }
}
```

`[[ input ]]` captures a copy by default; `[[ input: view : & ]]` captures a
reference named `view`. Captures are taught in
[lambdas and callable composition](lambdas-and-callable-composition.md).

`loan` is never in a type's `transfer` list: it is not an operation a type
defines, and every type can be lent. Nor is it the stance of a constructor's
same-type source: a constructor that built a value from a lent value of the
same type would just be the copy constructor under another name.

### What a loan does not promise

A loan stops the value being copied, moved, or taken as a whole. It does not
protect the value's members, as [members](#members) explains: through a
writable loan, a member can be moved or taken, just as it could be changed.

So someone with a loan can copy the members one by one and build a new value
from them. If a kind of value must never be duplicated, give the type a
`transfer` list without `copy`, and keep its members private.

If you write `as last` and the call turns out to take a loan, your variable
still counts as finished, because `as last` said you were done with it. The
compiler does not warn about this; it reports any later use as usual.

## Fallback

Fallback happens when something is built or assigned: initializing a
variable, building a by-value parameter, assigning, or replacing. If the type
has no operation for exactly the stance you wrote, the compiler uses one that
takes less instead:

| You write | What can receive it, best match first |
| --- | --- |
| No stance, or `as copy` | `copy`, then `loan` |
| `as move` | `move`, then `copy`, then `loan` |
| `as last` | `last`, then `move`, then `copy`, then `loan` |
| `as deep` | `deep`, or a `loan` consumer that builds nothing; otherwise error |
| `as loan` | `loan` only |

```zax
next : Message = message as last    // Message has no last constructor:
                                    // its move constructor builds next
```

Fallback passes over an operation that can't accept the source, never over one
that can and then refuses. An operation can't accept the source when it doesn't
exist, or when its source parameter's qualifiers rule the source out, as a
`mutable writable` source parameter rules out a readonly path; fallback then
continues to a weaker operation. An operation that accepts the source but
refuses the use, because it is `= forbidden` or because its receiver stance
can't build the new value, makes the use an error instead of falling back
further. Leaving a stance out of a type's `transfer` list means the operation
doesn't exist, so fallback applies as normal. The full rules for forbidden operations, and for how a constructor is
chosen, are in
[construction, replacement, and destruction](construction-and-destruction.md).

What happens to the original afterward depends on the stance you wrote and the
operation that ran. A stance is **reserved** when the type lists it in its
`transfer` list but has not written that operation yet:

| You wrote, and what ran | The original afterward |
| --- | --- |
| Anything, and a loan accepted it | the same value, possibly changed through a mutable reference; finished if you wrote `as last` |
| `copy` or `deep` ran | unchanged |
| `as move`, and `move` ran | usable, emptied or reset as the type's `move` defines |
| `as move`, and `copy` ran | unchanged |
| `as last`, whatever ran | finished: it can only be destroyed or replaced with `.=` |

If a type reserves `move` but has not written it yet, `as move` runs `copy`,
and the original is unchanged. Do not rely on that: once the type's author
writes `move`, the same line moves. Because a value after `move` is still fully
usable, code that treats it as possibly emptied keeps working either way, as
[reserved stances](#reserved-stances-count-as-implemented) explains.

For example, `Ledger` initially has only a `copy` constructor:

```zax
Ledger :: type {
  +++ final : ()(
    source : Ledger readonly copy &
  ) = {
  }
}

source : Ledger
first : Ledger = source as move
// Ledger has no move constructor, so the copy constructor runs instead.
// source is unchanged.
```

If `Ledger` later defines a `move` constructor, the same line runs it:

```zax
Ledger :: type {
  +++ final : ()(
    source : Ledger readonly copy &
  ) = {
  }

  +++ final : ()(
    source : Ledger mutable writable move &
  ) = {
  }
}

source : Ledger
second : Ledger = source as move
// The move constructor runs now.
// source is moved-from.
```

Fallback only makes sense if a type's `copy`, `move`, and `last` operations
all do the same job and produce the same kind of result. They may differ in
cost, in how much they allocate, and in what they leave in the original, but a
type should not use different stances to do unrelated things.

## When a stance must be written

No stance means `copy`, or a loan where the value forbids a copy. Two
separate rules decide when that is not enough.

1. **Necessity, at any use.** If a use can only work with `move` or `last`,
   leaving the stance unstated is an error wherever it appears, not only at a
   last use.
2. **Intent, at a last use.** At a variable's last use, the implicit `copy`
   still works, but writing `as last` might change what happens. When it would,
   you state which one you mean.

These examples share one set of declarations. `Message` has `copy` and `move`;
`MyOwnedFile` has only `move`, like a single-owner file handle; `Integer` has
`copy` and `deep`, and no `move` or `last`:

```zax
Message :: type transfer copy move {
  text : String
  payload : Payload

  +++ final : ()(source : Message mutable writable move &) = default
}

MyOwnedFile :: type transfer move {
  handle : Integer

  +++ final : ()(source : MyOwnedFile mutable writable move &) = default
}

relay final : ()(input : Message) = {
}

sendMessage final : ()(input : Message via move) = {
}

hold final : ()(input : Message mutable writable last &) = {
}

archive final : ()(file : MyOwnedFile) = {
}

sendCount final : ()(input : Integer) = {
}
```

### Necessity, at any use

```zax
// each line below is a separate alternative
archive(file)               // error: MyOwnedFile has no copy; write `as move` or `as last`
sendMessage(message)        // error: sendMessage's parameter is via move
hold(message)               // error: hold's reference requires last

archive(file as last)       // fine
sendMessage(message as move)
hold(message as last)
```

The compiler never adds `move` or `last` to make these calls work, because a
reader who sees no stance assumes the function took a copy and `message` is
unchanged. This applies even when the type has nothing to take:

```zax
bump final : ()(value : Integer mutable writable last &) = {
}

count : Integer
bump(count)            // error: bump may take over count, so the call must say so
bump(count as last)    // fine
```

### Intent, at a last use

```zax
useMessage final : ()() = {
  message : Message
  count : Integer

  relay(message)      // fine: copies; message is used again below
  relay(message)      // error: last use of message; `as last` would run
                      // Message's move constructor instead of copying it
  sendCount(count)    // fine: Integer has no move or last, so nothing could differ
}
```

At the last use, `relay(message)` must state which stance it means:

```zax
// each line below is a separate alternative
relay(message as last)   // done with message: move runs, message is finished
relay(message as move)   // move runs; message stays usable
relay(message as copy)   // a deliberate copy
```

or keep the implicit behavior deliberately:

```zax
intent<implicit-stance-at-terminal-use>{
  relay(message)
}
```

Any explicit stance satisfies the rule, including `as copy`.

The intent rule applies only to variables that hold their own value: by-value
locals and parameters. A function's result needs no check, because a result
the caller doesn't name is a temporary, as
[temporaries](#temporaries) explains. A reference only points at a value
someone else holds, so the last use of a reference is not the last use of
that value. The rule applies where the value itself is held:

```zax
forwardMessage final : ()(input : Message readonly copy &) = {
  relay(input)    // fine: copies the referent; the caller's message lives on
}
```

### What decides whether a stance matters

At a last use, the compiler compares what happens with no stance to what would
happen with `as last`. You must write a stance when `as last` would:

- call a different function; or
- run a different constructor, assignment, or replacement of the type.

Both depend only on how the function's parameters are declared and on which
operations the type has. A stance listed in the type's `transfer` list counts
as if its operation were already written, even when only `copy` is written so far,
as [reserved stances](#reserved-stances-count-as-implemented) shows. So "nothing
could differ" holds only for a type such as `Integer`, which has no `move` or
`last` and reserves neither.

Here is the first case. With no stance, the call works, but `as last` would
call the other overload:

```zax
store final : ()(value : Integer readonly copy &) = {
}

store final : ()(value : Integer mutable writable last &) = {
}

store(total)   // selects the copy & store; at a last use: error, because
               // `as last` would call the other store
```

Even when the compiler can prove that a use is the last one, it never turns a
missing stance, or an explicit `as move`, into `as last` for you. Choosing a
different function changes what your program does, so it is not an
optimization the compiler may make quietly. If a later compiler version
recognizes more last uses, it reports more of these errors; it never starts
calling different functions. Destruction at the end of a scope does not count
as a use.

### Reserved stances count as implemented

If a type's `transfer` list includes a stance, the compiler treats that
operation as existing when it decides whether you must write a stance, even
before the type's author has written it:

```zax
MyHandle :: type transfer copy move last {
  id : Integer
  // only copy is implemented so far; move and last are reserved
}

logHandle final : ()(input : MyHandle) = {
}

sendHandle final : ()(input : MyHandle via move) = {
}

useHandle final : ()() = {
  handle : MyHandle
  logHandle(handle)
  // error at this last use: state a stance, because `as last` would run
  // the reserved move constructor
}

reuseHandle final : ()() = {
  handle : MyHandle
  sendHandle(handle as move)   // copy runs until move is implemented
  logHandle(handle as copy)    // fine: a moved-from MyHandle is still usable
}
```

Code written this way keeps compiling when the type's author adds the real
operations. A type without a `transfer` list reserves nothing. If such a type
later adds `move`, code that passes one of its values, at a last use and with
no stance, to a by-value parameter or into a new variable, stops compiling
until a stance is written. Declaring `transfer copy move` from the start
avoids that.

### Returning at a last use

A `return` builds the result from its source, so it follows the same rule:

```zax
relayBack final : (result : Message)(input : Message) = {
  return input   // error: last use; Message has move, so write
                 // `as copy`, `as move`, or `as last`
}

relayCount final : (result : Integer)(input : Integer) = {
  return input   // fine: Integer has no move or last
}
```

### Generic bodies are checked per instantiation

The intent rule is decided separately for each instantiation. A generic body
does not need a stance in advance merely because some other type might need
one:

```zax
sendEach$ final : (Items ... : type)()(items ... : Items) = {
}

forwardEach$ final : (Items ... : type)()(items ... : Items) = {
  sendEach(from items)
}

forwardEach(1, 2)        // Integer items: `as last` would still copy; fine
forwardEach(message)     // Message items: `as last` would run move; intent error
```

The caller cannot fix that line, so the diagnostic names the generic body as
the place to fix and shows the call that produced the instantiation:

```text
error: last use of `items` needs a stated stance
  in forwardEach$(Message), line 2: sendEach(from items)
  instantiated from: forwardEach(message), main.zax:14
  this line belongs to forwardEach$; it needs `as copy`, `as move`, or `as last`
```

A library can test representative instantiations with
`compile{ assert(wellformed{ forwardEach(myMessage) }) }`. A `requires` clause
can reject item types, but cannot make a stance unnecessary for a type it
accepts.

A generic reference parameter writes its stance like any other reference
parameter, as in `items : T readonly loan &`. A stance applied to a type
parameter lands on the nearest layer that can carry one: with `T` bound to
`MyValue &`, `T move` means `MyValue move &`; with `T` bound to
`MyValue * unique`, it applies to the pointer.

### A stance on several items

A function that accepts any number of arguments receives them as a
[pack](variadics.md) and can pass them on with `from items`. The pack is not
one value. Each item is its own complete value, so a stance written on the
expansion applies to each item separately, and both rules above apply to each
item:

```zax
sendAll$ final : (Items ... : type)()(items ... : Items via last) = {
}
// each item's parameter is via last, as `input : Message via last` is for one value

forwardAll$ final : (Items ... : type)()(items ... : Items) = {
  sendAll(from items)
  // error at any use, for every item type including Integer: sendAll
  // requires last, and no stance means copy
}

forwardLast$ final : (Items ... : type)()(items ... : Items) = {
  sendAll(from items as last)   // last applies to each item separately
}
```

`items ... : Items via last` writes the requirement after the pack's type
parameter, the same way as for a single parameter.
[Variadics](variadics.md#a-stance-applies-to-each-item) uses the same
position. The exact spelling is not final yet.

A function that forwards a pack therefore states its intent, just as it would
for a single value. Each item also keeps its own type, including whether it is
a reference, so an item that is a reference stays a reference.

Sometimes each item must go through a call before it is passed on. Here each
message is encoded into a packet, and the packets are sent together:

```zax
encode final : (packet : Packet)(message : Message via last) = {
}

sendPackets final : ()(packets ... : Packet) = {
}

sendMessages final : ()(messages ... : Message) = {
  sendPackets(from each message : in messages as last |> encode())
  // encode runs once per message; its packets are passed on as a new pack
}

sendMessagesAtUse final : ()(messages ... : Message) = {
  sendPackets(from each message : in messages |> encode(message as last))
}
// Both pass each message to encode with last, separately.
```

`from each message : in messages |> encode()` runs `encode` once for each item,
inside the argument list. It is not the `each` loop;
[transforming each item](variadics.md#transforming-each-item) teaches it. The
stance can be written in either of two places, with the same effect: after
`in messages`, where it applies to each item, or on `message` at the use
inside the call. When the call names `message`, as in
`encode(message as last)`, each item is passed only where it is written, so
it is still passed once.

A stance on the expansion is the same for every item. To give different items
different stances, write the items as separate inputs, selecting each one by
compile-time index:

```zax
sendPair final : ()(header : Header via move, body : Body) = {
}

forwardPair$ final : (Items ... : type)()(items ... : Items)
  requires Items count == 2 = {
  sendPair(items[0] as move, items[1] as copy)
}
```

[Passing items on](variadics.md#passing-items-on) teaches the complete pack
model.

## Declaring stances

A stance written in a declaration always says what may be built from that
value. Where it is written decides whose value it is, and whether it may be
left out:

| Declaration | Example | When no stance is written |
| --- | --- | --- |
| A local, member, or by-value parameter (you own it) | `report : Report move` | `copy`: its owner may write every stance |
| The value an owning pointer points to | `owner : MyValue * unique` | `copy` |
| Each element of an array, or an optional's payload | `messages : MyMessage move[4]` | `copy` |
| An array or optional itself | `messages : MyMessage move[4] move` | the stance of its elements or payload |
| A reference, raw pointer, or slice bound from one visible value | `alias : Buffer & = owner` | taken from that value; see [references you declare](#references-you-declare) |
| A reference, borrowing raw-pointer, or slice parameter; a reference or slice result; a member that is a reference, borrowing raw pointer, or slice; a type alias of a reference; or an optional reference | `show final : ()(value : MyValue readonly loan &)` | must be written |
| The receiver `_` | `describe final : (result : String)() = { }` | `loan` |
| A by-value parameter's `via` | `item : MyItem via move` | `via copy`: any stance the call writes |
| A result | `result : Message move` | `copy` |

### Locals and members

A stance declared on a local variable or member says what may be built from
it, and its owner may write the stances in the owner table:

```zax
byValue final : ()(input : MyType) = {
}

display final : ()(input : MyType readonly loan &) = {
}

myType : MyType move

// each line below is a separate alternative
byValue(myType)              // error: myType must not be copied
byValue(myType as move)      // fine
byValue(myType as last)      // fine: the owner of a move value may also write as last
byValue(myType as copy)      // error: myType was declared move
display(myType)              // fine: display takes a loan; nothing is built
myType.describe()            // fine: describe's receiver has no stance, so it lends myType
```

The stance controls only which stance you may write. What actually runs still
follows the fallback rules. If `MyPoint` has only `copy`,
`point : MyPoint move` is still allowed: writing `as move` runs `copy`.

A declared stance the type can never satisfy is an error. If the type's
`transfer` list rules out every stance the declaration would allow, the
declaration itself is an error. A type without a `transfer` list can still add
operations later, so for such a type the error is reported where a transfer
is attempted:

```zax
MyDeepOnly :: type transfer deep {
}

shared : MyDeepOnly move   // error: the owner may write only as move or as last,
                           // and MyDeepOnly has neither
other : MyDeepOnly         // fine: copy includes deep, so other as deep works
point : MyPoint move       // fine: MyPoint has copy; `as move` falls back to it
```

A declared stance never settles an overload choice. A variable declared `copy`
still needs `as loan` or `as copy` at a call where both a loan and a by-value
function fit. The reader of the call should not need to look up a declaration.

The restriction covers only declared places. Array elements and members that
declare no stance are `copy`, so only their type limits what may be built from
them. A guarantee for every value of a kind, wherever it is stored, comes from
a `transfer` list on the type or an identity.

**Stances describe places, not every value built from them.** When the owner
of a `move` variable hands its contents to a function, the new value belongs
to that function, which may treat it however its own declarations say:

```zax
func3 final : ()(value : MyType via last) = {
  keep(value)                // fine: the function owns value, a copy value
}

myType2 : MyType move
func3(myType2 as last)       // fine: myType2's stance restricted that variable, not the value afterward
```

### References you declare

A reference or raw pointer you bind to one visible value can take its stance
from that value, but only when the stance leaves the owner's value unchanged.
A stance that would let the borrower change the owner's value must be written
by the owner at the binding:

| Owner's stance | An unmarked borrower bound to it gets | To let the borrower change the owner's value |
| --- | --- | --- |
| `loan` | `loan` | not possible |
| `copy` | `copy` | write `as move` or `as last` |
| `deep` | `deep` | not possible |
| `move` | `loan` | write `as move` |
| `last` | `loan` | write `as last` |

```zax
consume final : ()(buffer : Buffer) = {
}

owner : Buffer move
alias : Buffer & = owner                // owner is move, so alias gets loan
mover : Buffer move & = owner as move   // the owner grants move when binding

consume(alias as move)              // error: alias was given loan
consume(mover as move)              // fine: owner's contents leave; owner is now moved-from
consume(mover as last)              // error: mover isn't the owner
consume(owner as last)              // fine: the owner of a move value may write as last

plain : Buffer
view : Buffer & = plain             // plain is copy, so view gets copy
consume(view)                       // fine: copies; plain is unchanged
consume(view as move)               // error: view isn't the owner
```

Why `alias` gets `loan` and not `move`: a `move` stance on `owner` says the
only thing that may be built from `owner` is a value that takes its contents.
If an alias bound with nothing written also got `move`, the alias could take
`owner`'s contents, and no line near `owner` would show it. With a raw
pointer, that alias can be passed far away:

```zax
// what inheriting move would allow
emptyIt final : ()(target : Buffer move * copy) = {
  keep((target.) as move)           // takes the buffer's contents
}

owner : Buffer move
view : Buffer * = owner              // if view inherited move...
emptyIt(view)                        // ...this would empty owner, and nothing at this line says so
owner.read()                         // owner would be unexpectedly empty
```

The owner learns its variable's state from the stances it writes. So an
unmarked borrower gets only a stance that leaves the owner's value unchanged:
`loan`, `copy`, or `deep`. For a `move` owner, that leaves only `loan`. To let
a borrower take the contents, the owner says so where it binds:
`mover : Buffer move & = owner as move`.

Typed loop bindings and slices are references you declare too, and follow the
same table:

```zax
each value : MyValue readonly & from values {
  // values' elements declare no stance, so they are copy, and value gets copy
  showValue(value)
}

messages : MyMessage move[8]
window := messages[1..<4]       // the elements are move, so window gets loan for each
```

A loop binding over elements declared `move` would likewise get `loan`,
unless the loop source writes `as move`.

### References that stand alone

A reference or borrowing raw-pointer parameter, a reference result, a member
that is a reference or borrowing raw pointer, and a type alias of a reference
have no single value to take a stance from. Their stance must be written; for a
raw pointer, that is the stance of the value it points to, before `*`. An
owning pointer, raw `move` or `last` included, owns what it points to, so its
pointed-to value is `copy` when unmarked:

```zax
show final : ()(value : MyValue &) = {                 // error: write a stance, for example MyValue loan &
}

show final : ()(value : MyValue readonly loan &) = {   // fine
}

current final : (result : MyDocument readonly loan &)() = {
}

MyNode :: type {
  parent : MyNode loan *
}
```

When code takes something by reference, it is usually trying to avoid a
duplicate, so `loan` is the common answer. A `self` result is the exception:
it takes its stance from the call, as [results](#results) explains.

Two more kinds of borrower stand alone in the same way:

- **A slice parameter, member, or result:** its declaration includes the
  stance it needs for each element, before `[]`, because a slice has no stance
  of its own: `values : Integer loan[]`. A reference to a slice includes it in
  the same place: `values : Integer loan[] &`.
- **An optional reference:** its declaration always includes a stance before
  `&`, even for a local with an initializer. A plain reference bound to one value can only
  ever refer to that value, so it can take that value's stance. An optional
  reference can later refer to a different value, which may allow different
  things, so its stance is never taken from the first one:

```zax
view : MyValue readonly loan & ?           // fine: absent now; any later reference is a loan
view .= [{ firstValue }]                   // refers to firstValue
view .= [{ secondValue }]                  // a new reference, to a different value, same stance

guess : MyValue & ? = firstValue           // error: write the stance before &
```

And for a slice parameter:

```zax
total final : (result : Integer)(values : Integer[]) = {       // error: write an element stance
}
total final : (result : Integer)(values : Integer loan[]) = {   // fine: total only reads
}
```

### Receiver stances

A function's receiver is a parameter bound implicitly. A version with no
receiver stance takes `_` as `loan`, and writing `loan` there declares the
same version: declaring both is a duplicate. [Receivers](#receivers) covers
how a call chooses a version.

### By-value parameters and `via`

A by-value parameter is a new value the function owns. It has two stances:

- **The parameter's stance** says what the function may build from the
  parameter inside its body, exactly like a local's stance. It may be `loan`,
  which pins the parameter for the length of the call.
- **The `via` stance** says the least stance a call must write. The default is
  `via copy`, which accepts any stance. `via loan` is an error, because lending
  builds nothing.

The stance the call writes still chooses which constructor builds the
parameter, with the usual fallback. Three declarations decide such a call:

- the parameter's `via` stance says which stances the call must write;
- the stance declared on the caller's variable says which stances the call is
  allowed to write;
- the parameter's own stance says what the function may build from the
  parameter inside its body.

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

func1(myType1)              // error: via move needs as move or as last
func1(myType1 as move)      // fine
func1(myType2)              // error: via move needs as move or as last
func1(myType3 as last)      // fine

func2(myType1)              // fine: copied into place
func2(myType2)              // error: myType2 forbids copy
func2(myType2 as move)      // fine
func2(myType3 as last)      // fine
```

`via` is useful when a function needs its argument to arrive a certain way:

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
```

`via` is only for something that is built: by-value parameters, including
pointers passed by value, and results. A reference is never built, so `via`
on a reference parameter, a reference result, or a receiver is an error; the
stance before `&` already requires the call to write it:

```zax
take final : ()(item : MyItem copy via move) = {
}
// by value: the call must write as move or as last

hold final : ()(item : MyItem move &) = {
}
// reference: move & already requires as move

broken final : ()(item : MyItem loan & via move) = {
}
// error: a reference isn't built, so via has nothing to constrain
```

Overloads may differ only in `via`. A call uses the most specific `via` it
satisfies: a call written `as last` prefers `via last` over `via copy`, and a
call with no stance uses `via copy`.

`via` is needed only where one author writes the stance and another declares
the value. A lambda capture such as `[[ source as last: captured ]]` and a
loop source such as `each … in source as last` write both in one place, so
they have no `via`. A lambda's own parameters are ordinary parameters.

Composition also uses the word `via`, as in `= via fuel.refill`, always after
`=` in a function definition; a parameter's `via` always follows a type.

### Type aliases

A type alias can carry a stance, so a project can name a kind of variable
once instead of repeating the stance on every declaration:

```zax
OwnedPayload :: alias type Payload move
// a Payload that must be moved or finished, never copied

send final : ()(payload : Payload) = {
}

draft : OwnedPayload
send(draft)            // error: draft was declared through OwnedPayload, which is move
send(draft as move)    // fine
```

An alias can also be built from another alias, replacing its stance. When the
alias is a reference, the stance applies to the referred-to value, the
nearest layer that can carry one:

```zax
DeepView :: alias type Payload readonly deep &
CopyView :: alias type DeepView copy        // CopyView is Payload readonly copy &
```

Declaring or using an alias copies nothing; it only changes what declarations
that use it allow. The order in which alias properties combine is explained in
[declarations and bindings](declarations-and-bindings.md#exact-aliases-and-property-overlays).

### Inferred declarations

An inferred declaration such as `stored := makeMessage()` takes its
initializer's type, never a declared stance, so `stored` is an ordinary `copy`
variable. To restrict it, write the stance:
`stored : Message move = makeMessage()`. The exception is a reference, raw
pointer, or slice bound to one visible value, which takes a stance from that
value as [references you declare](#references-you-declare) describes. Complete
inference belongs to
[declarations and bindings](declarations-and-bindings.md).

The same holds for the elements of a new array. Its elements are new places,
so they start unmarked, even when they were built by moving:

```zax
messages : MyMessage move[4]
moved := messages as move              // moved is a MyMessage[4]: its elements are copy
kept : MyMessage move[4] = messages as move   // write the stance to keep the restriction
```

A generic type parameter deduced from an argument works the same way: it gets
the argument's type, not the stance declared on the argument's variable.
`keepAll(messages as move)` deduces `MyMessage` for an element type, not
`MyMessage move`. A stance reaches a type parameter only when the type argument
is stated explicitly as an alias that carries one.

### Transfer lists on types and identities

A type can list, in its definition, every transfer it allows:

```zax
MyOwnedFile :: type transfer move {
  handle : Integer
}

inspectFile final : ()(file : MyOwnedFile readonly loan &) = {
}

useFile final : ()() = {
  file : MyOwnedFile
  inspectFile(file)                    // fine: a loan builds nothing
  other : MyOwnedFile = file           // error: MyOwnedFile has no copy
  owner : MyOwnedFile = file as last   // fine: falls back to move; file is finished
}
```

What this means for code that uses the type:

- a stance left out of the list doesn't exist for the type: writing an
  operation for it in the type, or adding one from a partial, is an error,
  and fallback passes over it;
- you can still lend the value, call its functions that take a loan, and use
  its other constructors;
- listing a stance allows the operation but does not create it: if a member
  cannot be copied, a type that lists `copy` still has no generated copy, and
  that is an error only where something tries to copy; and
- a stance that is listed but not yet written is reserved: the compiler treats
  it as existing when deciding whether you must write a stance.

The stances are written in the order `copy move last deep`, right after the
`type` keyword and before any `seal` clause. The full rules are in
[construction, replacement, and destruction](construction-and-destruction.md#declaring-which-transfers-a-type-allows).
An identity may declare its own list, which turns a copyable underlying type
into a move-only kind wherever its values are stored, as
[transfer lists on identities](identity-types.md#transfer-lists-on-identities)
explains.

## Members

A member follows its own declared stance, whoever reaches it. A stance on the
containing value does not pass down to its members:

```zax
Package :: type {
  label : Label move
  weight : Integer
}

package : Package move
total := package.weight            // fine: weight declares no stance
// each call below is a separate alternative
ship(package.label)                // error: label's own declaration is move;
                                   // write `as move` or `as last`
ship(package.label as move)        // fine
```

Whether a member can be moved or finished through a reference depends on
writable access, not on owning the whole value. The limit on a borrower
protects what the caller tracks, which is its whole variable. A caller that
grants writable access has already accepted that members may change in any
way, including being emptied by assignment:

```zax
keepPayload final : ()(payload : Payload) = {
}

refresh final : ()(input : Message mutable writable copy &) = {
  keepPayload(input)                     // error: only copy was given for the whole message
  keepPayload(input.payload as move)     // fine: writable access already allows emptying payload
}

inspectOnly final : ()(input : Message readonly copy &) = {
  keepPayload(input.payload)             // fine: copies the member
  keepPayload(input.payload as move)     // error: readonly access can't change payload
}
```

At the call site, the compiler cannot see that `payload` was left emptied,
just as it cannot see that a function emptied a member through ordinary
writable access. What is left in the members is up to the function and the
type. Destructors and replacement constructors already have to cope with
whatever `last` leaves behind.

Pinning applies to the whole value, not its members:

```zax
MyTank :: type {
  fuel : MyFuel
}

pinned : MyTank loan
keep(pinned)                  // error: nothing may be built from pinned
keep(pinned.fuel as move)     // fine: fuel follows its own stance
```

Array elements behave like members: each follows the element stance written
before `[`, and a stance on the array covers only transfers of the whole
array. One rule is stricter for elements. Through a writable reference to
someone else's array, a function may move an element out, but may not finish
it with `as last` unless the caller gave up the whole array:

```zax
refreshFirst final : ()(values : MyMessage[0..] mutable writable loan &) = {
  consume(values[0] as move)               // fine: the element stays usable
  consume(values[0] as last)               // error: a borrowed array allows move, not last
}
```

The difference is in who defines the code that later uses the finished
value. A finished member sits inside a value whose type defines how it is
copied, replaced, and destroyed, and those operations must already cope with
a member left finished, as described above. A finished element would sit
inside an array, and the language itself implements copying, iterating, and
destroying an array; those operations expect every element to be live. A member of an element is
still a member, and follows the member rule: the element's type must cope
with it.
[Finishing elements in place](arrays-and-slices.md#finishing-elements-in-place)
explains what a function does instead.

### Bindings the language creates

Some names are created for you, such as the name a variant `case` gives to the
payload, or an untyped loop variable. Such a name is just another way to write
the member or element it refers to. Using it behaves exactly like writing that
member out in full: the member's own stance applies, and so does any loan you
reached it through:

```zax
MyOwnerChoice :: variant {
  owner : MyPayload * unique
  unavailable : MyUnavailable
}

adopt final : ()(target : MyPayload * unique) = {
}

message : MyOwnerChoice

switch message {
  case owner {
    // each call below is a separate alternative
    adopt(owner as move)   // same as adopt(message.owner as move)
    adopt(owner)           // error: no stance means copy, and a unique pointer cannot be copied
  }
}

inspectChoice final : ()(message : MyOwnerChoice readonly loan &) = {
  switch message {
    case owner {
      adopt(owner as move) // error: message is a readonly loan, so message.owner cannot be changed
    }
  }
}
```

A stance on an iteration source, such as `in messages as last`, still means
that stance at each use.

### Compatible whole values and anchored regions

A value declared `compatible shape` may be passed where another type with the
same layout of members is expected, without converting it. Passing the whole
value this way keeps the stance you wrote: if you write `as move` on the
value, the function that receives it as the other type also gets `move`.

An anchor, written `value anchor .member`, selects one region in the middle
of a value, starting at that member, and leaves out the rest:

```zax
PayloadSummary :: type {
  payloadSize : U32
  payloadChecksum : U32
}

Packet :: type {
  header : Integer
  payloadSize : U32
  payloadChecksum : U32
  trailer : Integer
}

consumePayloadSummary final : ()(payload : PayloadSummary) = {
}

packet : Packet compatible shape
consumePayloadSummary(packet)                      // error: complete Packet is not PayloadSummary
consumePayloadSummary(packet.payloadSize)          // error: payloadSize is only U32
consumePayloadSummary(packet anchor .payloadSize)  // copies the region
```

The anchor starts at `payloadSize` and spans the sibling `payloadChecksum`
member because the consumer requires one complete `PayloadSummary`. No
`PayloadSummary` member exists inside `Packet`; `packet.payloadSize` alone
cannot supply it.

The region is treated like a member: with no stance written, it is copied,
and it does not take a stance from `packet`. Taking the middle region with
`as last` could leave `header` and `trailer` in a state where `packet` cannot
be destroyed correctly. So `(packet anchor .payloadSize) as last` is allowed
only when something guarantees that `packet` stays safe to destroy afterward.

A function can give that guarantee for one anchor of the value it returns, by
naming the anchor on its result:

```zax
makePacket final : (
  result :
    Packet last
    compatible layout
    anchor .payloadSize
)() = {
}
```

`compatible layout` lets callers view the result as another type with the same
layout, as `compatible shape` does for a variable. A caller of `makePacket`
may take the `payloadSize` region with `last`. If the caller picks a different
anchor, the guarantee does not apply. Compatible shapes and anchors are
explained in
[Zax structural shapes and compatibility](structural-shapes-and-compatibility.md#transfer-stance-and-anchored-regions).

### Explicit composition wrappers

A composition `via` or `= existing` declaration can take a whole outer value
and pass one of its members on to another operation:

```zax
Tank :: type {
  fuel preferred : Fuel

  refill final : ()(source : Tank) = via fuel.refill
}
// the generated body behaves like: _.fuel.refill(source.fuel as last)
```

`refill` on a `Tank` is written as "do `fuel.refill`": the compiler generates
the body. It can find `source.fuel` automatically because `fuel` is marked
`preferred`, which names it as the member to use when the wrapper needs one.
[Composition](composition.md) explains `preferred` and the related `own`. A
hand-written body can always pass `source.fuel` itself. The generated wrapper
writes a stance when it passes the member on, chosen by how the outer
parameter is declared:

| Outer parameter | Stance the wrapper writes for the member |
| --- | --- |
| By value, with any stance except `deep` | `last` |
| By value, declared `deep` | `deep` |
| By reference | the stance the outer reference was given |

A by-value `source` belongs to the wrapper, and passing its member on is its
last use, so the wrapper writes `last` and the member's operation may reuse
its resources. That holds even for a pinned `source`, because members follow
their own stance. The usual fallback applies, and nothing is free: if the
call writes no stance, building `source` copies the `Tank`, and the member's
operation may copy `fuel` again.

An outer `deep` parameter passes its member on as `deep`, never as `last`. An
outer `move` reference passes its member on as `move`, never as anything
else.

When the wrapper returns, it destroys what is left of `source`, including the
finished member. When the wrapper can be generated, and how results are
passed back, are explained in
[Zax composition](composition.md#forwarding-values-stances-and-results).
If the type's destructor cannot handle a finished member, the wrapper cannot
be generated, and the type must write the wrapper's body by hand.

## By value and by reference

A by-value parameter gives the function its own value. A reference parameter
gives it access to the caller's value. A function may have either, or
overloads with both.

For a by-value parameter, three separate things happen when the function is
called:

1. **Choosing the function.** A call must write at least the parameter's
   `via` stance, as [by-value parameters](#by-value-parameters-and-via)
   shows.
2. **Building the parameter.** The stance the caller wrote chooses which of
   the type's constructors builds the parameter, with the usual fallback. A
   `via move` parameter called with `as last` still runs the `last` constructor
   if the type has one.
3. **Inside the function.** The parameter's own stance says what the function
   may build from it.

The type's constructor builds the parameter, and that constructor receives a
reference to the caller's variable. That is how a `move` or `last` constructor
takes the contents out of the caller's variable. By-value parameters are built
exactly the way local variables are:

```zax
MyMoveOnlyType :: type transfer move {
  handle : Integer

  +++ final : ()(source : MyMoveOnlyType mutable writable move &) = {
    // source refers to the caller's variable; take the handle and leave
    // source empty
  }
}

take final : ()(input : MyMoveOnlyType) = {
}

myMoveOnlyType : MyMoveOnlyType
// each call below is a separate alternative
take(myMoveOnlyType as move)   // the move constructor builds input; myMoveOnlyType is empty
take(myMoveOnlyType)           // error: no copy constructor exists
```

With the declarations from
[when a stance must be written](#when-a-stance-must-be-written):

| Type | Parameter | Implicit | `as move` | `as last` | Last use needs a stance? |
| --- | --- | --- | --- | --- | --- |
| `Message` | no `via` | copy runs | move runs | move runs (no `last`) | Yes: `last` would move instead of copy |
| `Message` | `via move` | error: requires at least `move` | move runs | move runs | Already required at any use |
| `Integer` | no `via` | copy runs | copy runs | copy runs | No |
| `Integer` | `via move` | error: requires at least `move` | copy runs | copy runs | Already required at any use |
| `MyOwnedFile` | no `via` | error: no copy | move runs | move runs | Already required at any use |
| `MyOwnedFile` | `via move` | error: requires at least `move` | move runs | move runs | Already required at any use |

The caller's variable ends in the state left by the operation that ran, except
that `as last` always leaves it finished.

A reference parameter refers directly to the caller's variable. Nothing is
built, and the function may change or take from the caller's value as far as
the reference's stance allows:

```zax
prepareForSend final : ()(input : Message mutable writable move &) = {
  sendPayload(input.payload as move)   // takes from the caller's message
}
```

A by-value parameter declared `via deep` is built as a complete independent
copy before the function starts. A readonly `deep &` reference is often
better: nothing is copied up front, and the function makes independent copies
of just the parts it needs:

```zax
prepareTemplate final : ()(input : Archive readonly deep &) = {
  archiveHistory(input.history as deep)
}
```

### Choosing value or reference

Suppose a function has two overloads, one taking a `mutable writable move &`
reference and one taking a by-value `via move` parameter, and the caller's
variable is mutable and writable. A call with `as move` could use either. The
reference overload is chosen, because it does not build another value:

```zax
prepareForSend final : ()(input : Message mutable writable move &) = {
}

prepareForSend final : ()(input : Message via move) = {
}

prepareForSend(message as move)   // the reference form
```

`as` can also take a qualifier: `message as readonly` is `message` seen
through a readonly path. Writing it rules out the mutable reference overload.
The by-value overload is then used, and its parameter is built by copying,
because `Message`'s move constructor takes a `mutable writable` source and
cannot be used through a readonly path:

```zax
prepareForSend((message as readonly) as move)
```

Matching the stance is only one of the things the compiler compares when it
chooses between overloads, alongside the argument's type and qualifiers. A
better stance match never wins over a worse type match:

```zax
send2 final : ()(input : Message) = {
}

send2 final : ()(input : Packet via last) = {
}
// a Message converts to a Packet

send2(message as last)
// error: ambiguous; the first matches the type exactly, the second matches
// the stance exactly, and neither is better at both
```

The complete overload rules are in
[function invocation](function-invocation.md#transfer-aware-value-and-reference-parameters).

## Receivers

A function defined in a type has a hidden input: the instance it is called on,
written before the dot in `draft.publish()`. That instance is the
**receiver**, and it behaves like a parameter that is bound implicitly. A
type may want different versions of a function depending on whether the
receiver is lent, copied, moved, or finished, so a function can declare a
receiver stance after its signature:

```zax
Document :: type {
  publish final : ()() = {
    // no receiver stance: _ is a loan
  }

  publish final : ()() deep = {
    // _ was given deep
  }
}

draft : Document
draft.publish()               // the version with no receiver stance
(draft as deep).publish()     // the deep version
```

A call chooses a version the same way an argument chooses a parameter:

- **No stance written:** the receiver is treated as written `copy` where the
  value allows a copy, and otherwise it is lent. A version with no receiver
  stance always qualifies.
- **A stance written:** a version qualifies when the call's stance allows what
  that version takes, along `copy`, `move`, `last`, and the value's own stance
  allows the version's stance. A `last` version never serves a call written
  `as move`, because the caller expects a usable value afterward.
- **Among the versions that qualify, the closest to the written stance wins.**

```zax
MyCounter :: type {
  foo final : ()() = {
  }

  foo final : ()() copy = {
  }
}

plain : MyCounter            // copy
locked : MyCounter move

plain.foo()                  // the same as (plain as copy).foo(): the copy version
locked.foo()                 // locked forbids copies, so it is lent: the version with no receiver stance
```

If a type has only a `copy` version, a call with no stance on a value that
allows copies uses it. On a value that forbids copies, that call is an error.

**`move`, `last`, and `deep` versions are chosen only when the call writes the
stance.** The variable's declared stance only limits what may be written:

```zax
MyDocument :: type {
  doSomething final : ()() = {
  }
  // lends _

  doSomething final : ()() move = {
  }
  // may move from _
}

document : MyDocument
document.doSomething()                 // no stance written: the copy rule applies, and the
                                       // version with no receiver stance lends document
(document as move).doSomething()       // the move version
(document as last).doSomething()       // the move version: it takes no more than last, and it
                                       // is closer than the version with no receiver stance

fixed : MyDocument last
fixed.doSomething()                    // lends fixed
(fixed as last).doSomething()          // the version with no receiver stance: fixed doesn't allow a move
```

A call with no stance never finishes or empties the receiver, whatever the
variable was declared with. A reader of `document.doSomething()` can rely on
nothing having been moved out of `document`, and on it not being finished; a
version with writable access may still change its contents.

Adding a `copy` version to a type can change which version an existing call
with no stance uses, on a value that allows copies. A `copy` version cannot
change the caller's value, so the caller's code stays correct.

At a last use, the intent rule applies to receivers just as to arguments: if
`as last` would call a different version, a call with no stance must state
one.

A function's receiver stance, parameter stances, and result stances are
separate; each describes its own input or output.

### `_` is the receiver path

Inside the function, `_` stands for the receiver exactly as the caller wrote
it, with the same permissions. `_` is a borrower: it may be written with the
stance its version was given, or lent:

```zax
keepShape final : ()(shape : Shape) = {
}

showShape final : ()(shape : Shape readonly loan &) = {
}

Shape :: type {
  describe final : ()() readonly = {
    showShape(_)    // fine: lends the receiver on
    keepShape(_)    // error: _ is a loan; keepShape would copy it
  }
}
```

Inside a `move` or `last` version, a use of `_` with no stance lends it,
because `_` was not given `copy`; taking from `_` must be written:

```zax
Document :: type {
  publish final : ()() last = {
    _.inspect()                      // no stance: lends _
    (_ as last).finishPublishing()   // calls finishPublishing's last version on _
  }
}
```

### Passing the receiver on

A call on a receiver reads as the value doing something, not as the value
being given away. A reader of `(task as last).submit()` cannot tell that
`submit` hands `task` to a queue. So when a function passes the whole `_` on
with `move` or `last`, it states that the hand-off is deliberate, with an
[intent acknowledgement](intent-acknowledgements.md):

```zax
Task :: type {
  submit final : ()() last = {
    intent<receiver-transfer>{
      queue.push(_ as last)        // gives the task to the queue
    }
  }

  snapshot final : (result : Task)() copy = {
    return _ as copy               // copy leaves _ unchanged: no acknowledgement
  }
}
```

Without the acknowledgement, `queue.push(_ as last)` is an intent error. The
same applies through a reference made from `_`:

```zax
// inside a last version
myself : Task mutable writable last & = _ as last
myself.prepare()                   // a call on myself, not a hand-off: fine
intent<receiver-transfer>{
  adopt(myself as last)            // passes _ on
}
```

The acknowledgement is not needed for:

- **a call on `_`**, such as `(_ as last).finishPublishing()` above. If
  `finishPublishing` passes `_` on, its own body has the acknowledgement, so
  every hand-off is acknowledged once, where it happens;
- **members**, such as `send(_.buffer as last)`, which follow their own
  stance;
- **`copy` and `deep`**, which leave `_` unchanged.

Where `_` is a loan, passing it on with `move` or `last` is an error, and an
acknowledgement does not change that.

### `self` chains

A function declared with a `self` result returns its receiver, so calls can be
chained. The stance written at the start of the chain is carried along, and
the variable counts as finished only once the whole expression is done:

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
// each call gets foo written as last and picks its last version;
// foo counts as finished once the whole expression is done
```

A version that returns `self` must leave `_` usable for the next call in the
chain. [Results](#self-results-carry-the-callers-variable) covers `self`
results in full.

### Receiverless `once` calls

A `once` function can be called on the type itself, as in
`MyResource.inspect()`, with no instance. There is then no variable to write a
stance on, so it is a call with no stance. The receiver is the type's Nothing
instance, a stand-in value that every type has for calls like this. It is
nobody's value, so it is pinned: a call on the type lends it and uses the
version with no receiver stance, even when the type also has a `copy`
version:

```zax
MyResource :: type {
  inspect final once : ()() = {
  }

  consume final once : ()() last = {
  }
}

MyResource.inspect()  // the version with no receiver stance: the Nothing instance is lent
MyResource.consume()  // error: the Nothing instance is pinned, and consume requires last
```

A call on an instance uses the stance written on that instance:

```zax
(resource as last).consume()
```

The compiler decides the receiver's stance from how the call is written, not
at run time, and an alias or stored function value keeps the same decision.
Calls on a type and Nothing instances are explained in
[Zax Nothing instances](nothing-instances.md#receiver-stance-is-statically-known).

## Results

A result is a value the function owns and fills in. When the caller uses the
result to start a new variable, the two are the same value. So **a result has
one stance, with the same meaning inside the function and for the caller**.
The function may never do with `result` what the caller couldn't do with it:

```zax
makeMessage final : (result : Message move)(source : Message readonly copy &) = {
  result.+++(source as copy)   // fill it in
  cache.store(result)          // error: result is move, so nothing may be copied from it
}

makeFree final : (result : Message)(source : Message readonly copy &) = {
  result.+++(source as copy)
  cache.store(result)          // fine: an unmarked result is copy
}
// cache.store takes a Message by value
```

**`via` on a result** names the least stance the function's own `return` must
write when it builds `result`. That is the same meaning as on a parameter,
where the call builds the value. The function may also build `result`
directly with `result.+++()`:

```zax
giveMeResult final : (result : MyType loan via copy)() = {
  builtUpValue : MyType move
  doStuff(builtUpValue)
  return builtUpValue as last    // via copy accepts last
}

makeBig final : (result : MyBigBuffer via move)() = {
  return cached                  // error: no stance means copy; the return must write as move or as last
  return cached as move          // fine
}
```

**The caller's variable may declare its own stance,** because stances
describe places. The exception is a `loan` result: nothing may be built from
it, so the caller's variable must be that same value, declared `loan`, and
the result is built directly into it:

```zax
stored : Message = makeMessage()             // fine: stored is an ordinary copy variable
pinned : Message loan = makeMessage()        // fine
myResult : MyType loan = giveMeResult()      // fine: the same value; nothing is built from result
kept : MyType = giveMeResult()               // error: nothing may be built from a loan result
```

Building a result straight into the caller's variable is otherwise optional,
so a `loan` result is the one case where it is required. Chaining in one
expression, as in `giveMeResult().fooize().streamline()`, and a
[`using`](using.md) block also work with a `loan` result.

When a function returns another function's result, two declarations are
involved:

- the inner function's result stance decides how the outer function's result
  is built;
- the outer function's result stance decides what its own caller may build
  from it.

Result construction, mapping, and completion are defined by
[function invocation](function-invocation.md).

A failure result works the same way. Here the failure value is a `unique`
pointer, which cannot be copied:

```zax
openResource final : (
  resource : MyResource * unique last,
  failure except : MyOpenError * unique last
)(
  name : String readonly loan &
) = {
  if cannotOpen(name)
    except failure: makeOpenError(name) as last

  return makeResource(name) as last
}

resource := openResource(name) catch failure {
  error : MyOpenError readonly loan & = failure.   // failure. is the value the pointer refers to
  report(error)
  return
}
```

The `catch` block receives the failure value as a temporary, as
[temporaries](#temporaries) describes. Failures are explained in full in
[Zax exceptional result flow](except.md#transfer-ownership-and-references).

A reference result is a borrower and must write its stance, as in
`current final : (result : MyDocument readonly loan &)()`. The usual lifetime
rules for returned references apply.

### `self` results carry the caller's variable

A function declared with a `self` result returns its receiver, so calls can be
chained, as in a builder:

```zax
Builder :: type {
  name : String
  size : Integer

  setName final : (result self : Builder &)(name : String readonly loan &) = {
    _.name = name
    return _
  }

  setSize final : (result self : Builder &)(size : Integer) = {
    _.size = size
    return _
  }

  build final : (result : Product)() move = {
    // takes the parts out of the builder
  }
}

builder : Builder
product := (builder as move).setName(name).setSize(size).build()
```

The result of `setName` is `builder` itself, exactly as the caller wrote it:
the same variable, the stance written on it, here `as move`, and its
qualifiers. So the `move` written at the start of the chain reaches `build`,
the call at the end. `setName` and `setSize` only pass `builder` along; they
do not use up the `move`. A `readonly` function that returns `self`, called on
a writable variable, likewise hands back a writable path, because the result
is the caller's own variable. A `self` result therefore declares no stance:
it takes its stance from the call.

Because a `self` result is the variable, everything that applies to a use of
the variable applies at the end of the chain:

```zax
// Message.prepare returns self, and relay takes a Message by value;
// each line below is a separate alternative, at message's last use
relay(message)                       // error: state a stance
relay(message.prepare())             // the same error: relay still receives message
relay((message as last).prepare())   // fine
relay(message.prepare() as last)     // fine: the same stance, written later in the chain
```

The same holds for a variable's declared stance:

```zax
// refresh returns self marked #, and keep takes a MyValue by value
value : MyValue move

// each line below is a separate alternative
(value as copy).refresh()               // error: value was declared move
keep((value as loan).refresh())         // error: the result is value written as loan,
                                        // and a loan cannot be copied
keep((value as move).refresh())         // fine
keep((value as last).refresh() as move) // fine: the later stance replaces the earlier one,
                                        // so value ends moved-from, not finished
```

As with `source as move as copy`, the stance written last counts, so the
`as move` above replaces the `as last`, and `value` is not finished. When
`as last` is the stance that counts, it finishes the variable once the whole
expression is done, whichever calls ran. If nothing receives the chain's
result, the variable is still finished, and its destructor runs at the end of
its scope as usual.

The generated `=` also returns `self`, so a chain of assignments behaves
exactly like writing the destination again:

```zax
first = second = third                 // like second = third, then first = second
restricted : MyType move
// each line below is a separate alternative
first = restricted = third             // error: like first = restricted, which copies restricted
first = (restricted = third) as move   // fine
(first = second).describe()            // fine: describe's receiver has no stance, so it only lends first
```

How to declare a `self` result, and when its result may be discarded, are
explained in
[function invocation](function-invocation.md#receiver-origin-self-results).

## Temporaries

A value with no name cannot be used again after the expression that uses it,
so it is treated as written with the strongest stance its own stance allows.
That covers a value built directly in an expression, such as `(: Source)`,
and a function's result that the caller doesn't name:

| The temporary's stance | Treated as written |
| --- | --- |
| `copy` (or none), `move`, `last` | `as last` |
| `deep` | `as deep` |
| `loan` | nothing can be built from it; it can only be lent, or received as the same value |

```zax
makePayload final : (result : MyPayload)() = {
}

existing : MyPayload
existing = makePayload()          // the result is written as last, so the move or last
                                  // assignment can run instead of a copy

consume final : ()(input : MyPayload loan &) = {
}

consume final : ()(input : MyPayload last &) = {
}

consume(makePayload())            // written as last: the last & version is chosen

convert : MyOtherType = makePayload()   // a converting constructor may take the payload's resources

```

"No stance means `copy`" is about variables you can use again. A temporary has
no name, so nobody can use it again and nothing is hidden from the reader. A
caller can still write a stance on the call: `makePayload() as copy` is fine
for a `copy` result, and an error for a `move` result, which forbids copies.

A temporary's own stance is always declared somewhere, or is the unmarked
`copy`. A function or operator result takes its declaration. An expression
such as `(: MyValue)` takes its own. A temporary made by an implicit
conversion to bind a reference is a value the call expression owns, so it is
`copy`.

A temporary used as a receiver picks its version the same way:

```zax
MyBuffer :: type {
  bytes : Byte[0..]

  contents final : (result : Byte[0..])() copy = {
    return _.bytes as copy       // the buffer keeps its bytes
  }

  contents final : (result : Byte[0..])() last = {
    return _.bytes as last       // the buffer is ending, so its bytes are moved out
  }
}

makeBuffer final : (result : MyBuffer)() = {
}

fromTemporary := makeBuffer().contents()   // the temporary buffer is written as last:
                                           // the last version moves the bytes out
buffer : MyBuffer
fromNamed := buffer.contents()             // no stance on a copy variable: the copy version
buffer.append(moreBytes)                   // fine: buffer still has its bytes
```

## Pointers

A pointer involves two values: the pointer, and the value it points to. Each
has its own stance, in its own position:

```zax
view : MyValue loan * copy      // nothing may be built from the value;
                                // the pointer itself may be copied
```

The rules above apply to both. Whether a pointer is an owner or a borrower of
the value it points to depends on its kind:

- a `unique` pointer, a `strong` pointer, and a raw pointer declared `move` or
  `last` are owners of the value they point to;
- a raw `copy` or `loan` pointer holds only an address, so it is a borrower,
  and a `weak` pointer becomes an owner only by acquiring a `strong` one.

So the owner rules apply through owning pointers, and the borrower rules
apply through raw `copy` pointers:

```zax
func1 final : ()(value : MyValue copy * unique) = {
  callByValueMove(value. as move)    // fine: an owner
}

borrowOnly final : ()(value : MyValue copy * copy) = {
  keep(value.)                       // fine: copy was given
  keep(value. as last)               // error: a raw copy pointer is a borrower
}

install final : ()(device : MyDevice loan * unique) = {
  registry.adopt(device as move)     // fine: ownership of the pointer moves
  relocate(device. as move)          // error: the device is pinned
}
```

A pinned value owned by a movable pointer stays at the same address while the
pointer is handed around.

Allocation, how ownership passes between pointer kinds, the one rule that
applies only to raw pointers, borrowing through `p.`, and why a `unique`
pointer allows no raw borrowers are explained in
[Zax pointers, allocation, and arenas](pointers-and-arenas.md).

## Callable values and captures

Capturing a variable in a lambda builds the capture from it, so a capture is a
transfer. Moving or copying the lambda later is a separate transfer:

```zax
callback := [[ source as last: captured ]] ()() bound strong {
  use(captured)
}
```

`source as last` decides how `captured` is built, and that happens once, when
the lambda is created. Capturing a variable declared with a stance requires
writing a stance its owner may write, like any other transfer. `bound strong`
is separate from the captures: it says how the lambda holds the object its
function is called on, here as another strong owner.

A loan cannot be captured as a copy, because nothing may be built from a loan.
If the lambda captures it by reference instead, the lambda holds a loan too. A
reference capture is a reference bound to one visible variable, so with no
stance written it takes its stance from that variable, as
[references you declare](#references-you-declare) describes: `copy` from a
`copy` variable, and `loan` from a variable declared `move` or `last`.

```zax
view : Document readonly loan & = source
copied := [[ view ]] ()() {               // error: copying the document would build from a loan
  showDocument(view)
}
borrowed := [[ view: kept : & ]] ()() {   // fine: a reference capture of a loan is a loan
  showDocument(kept)
}
```

To let a lambda move the contents out of a `move` variable, write the stance
on the source and on the capture, as for any reference:

```zax
buffer : Buffer move
drainer := [[ buffer as move: mover : move & ]] ()() {
  consume(mover as move)
}
// while drainer exists, code using buffer must treat it as possibly moved-from
```

A captured `move &` may empty `buffer` during any later call of the lambda, so
from the capture onward, and for as long as `drainer` exists, code that uses
`buffer` must treat it as possibly moved-from. A capture written `as last`
with `last &` finishes `buffer` when it is captured.
Capture spelling belongs to
[Zax lambdas and callable composition](lambdas-and-callable-composition.md#explicit-reference-capture-borrows).

Copying or moving `callback` later never re-runs its captures:

- copying a lambda that only borrows the object it is bound to gives another
  lambda borrowing the same object;
- copying a lambda declared `bound strong` makes the copy another strong owner
  of the same object;
- copying a lambda declared `bound weak` gives another weak reference to it;
- a lambda declared `bound unique` cannot be copied;
- `move` and `last` hand the binding over to the new lambda, and the old
  lambda can no longer be called; and
- a lambda has no `deep` copy.

If the lambda's body moves or takes a captured value, that captured value
stays moved-from or finished for later calls. Calling the lambda again is
valid only if its body can cope with that, and the compiler reports calls it
can prove are invalid. Lambdas are explained in full in
[Zax lambdas and callable composition](lambdas-and-callable-composition.md#callable-transfer-reset-and-recursion).

## Optional values

An optional has two stances. The stance before `?` belongs to the contained
value; the stance after `?` belongs to the optional itself. Copying or moving
the whole optional also copies or moves the contained value, so the
optional's own stance may name only stances the contained value also allows:

| Contained value's stance | Allowed stances for the optional itself |
| --- | --- |
| `copy` | any |
| `move` | `loan`, `move`, `last` |
| `last` | `loan`, `last` |
| `deep` | `loan`, `deep` |
| `loan` | `loan` only |

```zax
slot : MyPayload move? loan      // a pinned slot whose contents may be moved out

keep(last slot)                  // fine: takes the contained value; slot becomes empty
other := slot as move            // error: the slot itself is pinned
slot .= makePayload()            // fine: builds a new contained value in place

broken : MyPayload loan? move    // error: moving the optional would move a pinned value
```

When the optional's own stance is left out, it is the contained value's
stance: `MyPayload move?` is a `move` optional.

An optional also accepts two forms written before it, `last source` and
`move source`. Each writes a stance on the contained value, so the contained
value's stance must allow it:

```zax
source as last // the ordinary last stance on the optional value
last source    // last on the contained value, then source becomes empty
move source    // move on the contained value; source keeps it, moved-from

frozen : MyPayload loan?
keep(move frozen)              // error: nothing may be built from a pinned contained value
```

When `last source` is used:

- the contained value, if there is one, is passed on with `last`;
- during that call, `source` counts as finished: the rest of the expression
  cannot use it;
- `source` and its contained value stay valid until the call ends;
- when the call ends, whatever is left of the contained value is destroyed,
  and `source` becomes an empty optional.

`move source` passes the contained value on with `move` and does not empty
`source`.

A function given a writable reference to someone else's optional may use
both forms: `move cache` leaves the contained value usable, and `last cache`
empties the optional, so nothing finished is left in it. Writing `last` on
the contained value directly, as `cache. as last`, is an error through a
reference, because it would leave a finished value inside the caller's
optional; this is the same rule as for
[array elements reached through a borrowed array](#members).

```zax
refill final : ()(cache : MyPayload? mutable writable loan &) = {
  keep(move cache)          // fine: the contained value stays usable
  keep(last cache)          // fine: the contained value leaves, and cache becomes empty
  if ?cache {
    keep(cache. as last)    // error: would leave a finished value in the caller's optional
  }
}
```

Optional values are explained in full in [Zax optional values](optional-values.md).

## Variants and unions

A stance written on a whole variant applies to its current payload. The
variant keeps the same alternative selected, or stays empty if it was empty:

- `copy` leaves the original unchanged;
- `move` leaves the original with the same alternative and a moved-from
  payload;
- `last` leaves the original with the same alternative and a finished payload
  that can still be destroyed; and
- `deep` needs the payload to have a `deep` operation.

`last` does not empty the variant by itself. The payload ends later, when the
variant is reset, assigned another alternative, replaced, or destroyed.
Variants are explained in full in
[Zax variants](variants.md#copy-deep-move-and-last).

A `case` name refers to the payload where it is, and creating it transfers
nothing. Using it is exactly like writing `choice.owner`, as
[bindings the language creates](#bindings-the-language-creates) shows.

A union owns no resources that could be moved out of it. Copying a union
copies all of its bits. `move` and `last` fall back to that copy, so the
union's bits are unchanged; after `as last` the variable still counts as
finished, as after any `as last`. A union has no `deep` copy, because it may hold raw
pointers or similar data, and copying their bits does not make an independent
value. See
[Zax unions](unions.md#whole-union-operations).

## Arrays and slices

An array has a stance for each element, written before `[`, and a stance for
itself, written after `]`. Elements behave like members: each follows its own
stance, and the array's owner may write stances on them. The array's stance
covers only transfers of the whole array, and the element stance limits it by
the same table as an optional's:

```zax
messages : MyMessage move[4] move    // each element: move; the array: move
consume(messages[0] as move)         // fine: the owner may move an element out
other := messages as move            // fine: moves the array, and each element with it
```

An element type with a `transfer move` list makes every element move-only
wherever the array is stored, whatever stance is written.

Copying an array creates another array with its own storage and copies each
element. `deep` does the same, but every element must have a `deep` operation;
it never falls back to `copy`. `move` moves every element out and leaves the
source at its declared minimum length, with the elements it keeps moved-from;
a fixed array, and an immutable one, keep all of them. An array whose storage comes from a storage
provider and whose minimum length is zero may instead hand its whole block
over on `move`, without moving any element. After `last`, the source only
needs to be safe to destroy, so a provider-backed array may always hand its
block over.

A slice has no stance of its own. Each element it shows is reached as if
through a `<stance> &` reference, and the stance written before `[]` is what
the slice was given for each element. A slice parameter's declaration must
include it, and `as` written on a slice applies to its elements:

```zax
drainSome final : ()(values : MyMessage mutable writable move[]) = {
  consume(values[0] as move)         // fine: the slice was given move for each element
  consume(values[0] as last)         // error: a slice given move may pass on only move
}
```

Writing `last` on elements that stay in their array leaves the array
partially finished: it no longer works as a whole, and only elements the
compiler can prove unfinished may still be used. `take` instead removes
elements from the array, or from a slice, and hands them on. Both are
explained, with element stances, slices, storage, and what `move` leaves, in
[Zax arrays and slices](arrays-and-slices.md#stances-on-elements-and-slices).

You can write stances on the arrays you join. Each operand follows the usual
rules:

```zax
left : MyMessage[0..] move
right : MyMessage[0..]

joined1 := left <+> right            // error: no stance means copy, and left is move
joined2 := left as move <+> right    // fine
joined3 := left as last <+> right    // fine: the result may reuse left's storage
```

A `move` or `last` join may take over `left`'s whole storage block, leaving
`left` as `move` or `last` normally does. Otherwise the result gets new
storage from the destination's storage clause, or from the `___` context's
default, never implicitly from `left`'s provider.

The join operation decides whether storage can be reused, which elements are
copied or moved, and how large the result is. Arrays are explained in full in
[Zax arrays and slices](arrays-and-slices.md#copy-deep-move-and-last).

## Stances and qualifiers

A stance is written in a qualifier's position, but it is not a qualifier.
Qualifiers make three separate promises, and stances do not change them:

| Axis | Meaning |
| --- | --- |
| `mutable` / `immutable` | `immutable` promises that the current value's contents do not change through ordinary safe behavior; `mutable` makes no such promise |
| `final` / `varying` | `final` promises that the place is never given a new value; `varying` allows a new value to replace the old one |
| `writable` / `readonly` | `readonly` promises that this access path performs no change; `writable` may perform an otherwise permitted change |

For example, this parameter:

```zax
drainBuffer final : ()(source : Buffer mutable readonly final move &) = {
}
```

describes:

- a mutable current `Buffer` value;
- readonly access through this path;
- a final referenced place; and
- permission for `drainBuffer` to move from the caller's `Buffer`, so a call
  must write `as move`.

Because the access is `readonly`, `drainBuffer` can move from the `Buffer`
only in a way that does not change it, for example when the resource it takes
lives outside the value.

`readonly` does not cancel the other two promises. Each function declaration
describes one exact combination of qualifiers:

```zax
consume final : ()(
  source : Buffer mutable readonly final copy &
) = {
}

consume final : ()(
  source : Buffer mutable readonly varying copy &
) = {
}
```

These are two different functions. Leaving a qualifier out fills in its
default; it does not make one function work for every combination.

A future feature may let one declaration generate the exact combinations a
program needs, but each generated version will still be its own function.

Qualifiers are explained in full in [Zax qualifiers](qualifiers.md).

## Generated operations

The compiler generates only `copy` automatically:

- it generates one `copy` operation for each combination of qualifiers the
  type supports, with no receiver stance;
- declaring one of those exact operations yourself, whether with a body or
  with `= default`, `= existing`, or `= forbidden`, replaces only that one
  generated operation;
- if a type's `transfer` list leaves out `copy`, nothing is generated, so a
  `transfer move` type has no copy;
- if a member cannot be copied, the generated copy does not exist, and that is
  an error only where something tries to copy;
- `move` and `last` fall back to `copy` when the type has no `move` or `last`;
  and
- without a `deep` operation, `as deep` is an error.

`move`, `last`, and `deep` are never generated automatically, because what a
value should leave behind can depend on what it means, not only on how it is
stored. A type asks for the member-by-member version with `= default`:

```zax
MyFolder :: type transfer move {
  file : MyOwnedFile
  name : String

  +++ final : ()(source : MyFolder mutable writable move &) = default
  // file moves with MyOwnedFile's move; name moves with String's move
}
```

Without that declaration, a structure holding a move-only member cannot be
transferred: its generated copy is unavailable and no `move` exists.

A constructor may also declare a receiver stance, so that a type can build a
value differently, or refuse to build it, depending on the stance the new
value is declared with. A constructor declared with a receiver stance other
than `loan` never replaces a generated one; it competes with it as a more
specific candidate. Writing `loan` is the same as writing no receiver stance,
so a constructor with an explicit `loan` receiver stance is exactly a
generated shape and replaces it.

If you write an operation meant to replace a generated one but get its
signature slightly wrong, both exist, and a call may become ambiguous. The
error message should show the exact generated signature you missed.

Generated operations, constructor receiver stances, `= default`, assignment,
replacement, and destruction are explained in
[construction, replacement, and destruction](construction-and-destruction.md#opting-in-to-default-move-last-and-deep).

## Assigning a value from itself

Using a value as the source of its own assignment is not automatically an
error:

```zax
value = value         // ordinary assignment
value .= value        // replace value with a new value built from itself
value .= value as move
value .= value as last
```

Whether this works is up to the operation. For a number it is harmless. For a
type that owns resources, the operation may need the source and the
destination to end up in different states, which is impossible when they are
the same variable.

Using a member of the destination as the source is harder still:

```zax
owner .= owner.payload as move
```

The replacement ends the old `owner`, including `owner.payload`, so it cannot
keep reading `owner.payload` afterward. A hand-written replacement can take the
member first and release the rest of the old value after.

The compiler reports the conflicts it can prove. The rules about where a
reference comes from and when it stops being valid are in
[lifetimes and references](lifetimes-and-references.md), and what the compiler
can prove, and what `unsafe` allows, is in
[safety and analysis](safety-and-analysis.md).

A successful transfer or replacement releases every resource exactly once:
each resource of the source, of the old destination, and of any temporary
ends up owned by exactly one value or is released exactly once.

## Costs

No stance guarantees that an operation is fast, allocates nothing, or actually
takes resources instead of copying them.

The costs to keep in mind:

- `loan` builds nothing, but a mutable loan may still change the caller's
  value;
- `copy` may share storage, allocate, or run code the type's author wrote;
- `deep` may walk the whole value and allocate at every level;
- a by-value parameter always builds a complete value, with whichever
  constructor the caller's stance chooses;
- a `move` or `last` reference parameter may change the caller's variable
  directly;
- when `move` or `last` falls back to `copy`, you pay for a copy, including
  for a reserved stance whose operation is not written yet;
- generated `copy` and `= default` operations run the same operation on every
  member;
- assignment and replacement release the old value, usually before building
  the new one;
- `last source` on an optional destroys the remaining contained value when the
  call ends;
- capturing values may allocate or make the lambda larger; and
- checking lifetimes, aliases, generic instantiations, and last uses takes
  compile time.

Stances themselves cost nothing at run time: they are checked by the compiler
and leave no trace in the program.

Each type documents what its own operations cost.

## Diagnostics

Error messages should name the rule and the declaration that caused them. They
cover:

- an unstated stance where the use needs `move` or `last`: a type without
  `copy`, a by-value parameter whose `via` requires more, or a reference
  parameter that requires a stance the call didn't write;
- a last use where `as last` would call another function or run another
  operation, counting stances the type's `transfer` list includes as written,
  including for each item of an expanded pack and at the end of a chain of
  `self` results;
- a last use inside a generic body, naming the generic body as the place to
  fix and the instantiating call;
- a stance written that the value's holder may not write: an owner writing a
  stance its declaration doesn't allow, or a borrower writing a stance it
  wasn't given, saying which;
- a reference bound with a stance its source doesn't allow;
- a reference, raw-pointer, or slice parameter, reference or slice result,
  member, type alias, or optional reference with no stance written;
- an array or optional stance that its element or contained-value stance
  does not allow, and a reference parameter whose inner stance the caller's
  value does not allow;
- `last` written on an element reached through a borrowed array, or on an
  optional's contained value reached through a reference;
- a declared stance the type's `transfer` list can never satisfy;
- `via` on a reference, `via loan`, or `loan` written into a by-value
  parameter at a call;
- `_`, or a reference made from it, passed on with `move` or `last` without
  `intent<receiver-transfer>`;
- an unstated call that both a `loan` and a by-value function fit;
- unavailable `deep` behavior with no fallback, and a value declared `deep`
  used with no stance;
- use of a finished source without `intent<terminal-source-reuse>`;
- a call that is ambiguous between candidates, and any mismatch in qualifiers
  or stance, saying exactly which one;
- a variable or result used again after it was finished or passed on;
- a `loan` result received by a variable that isn't `loan`;
- a failure value whose stance does not let it be passed to its handler or
  returned outward;
- a declaration meant to replace a generated one whose signature does not
  match, or a member that stops an operation being generated;
- assigning a value from itself where the operation does not allow it, or
  from one of its own members whose lifetime would end;
- an intent or unsafe acknowledgement that does not apply;
- a function, constructor, or replacement that can finish without every
  result or member being built;
- and a resource that would be released twice, or never.

When a generated operation fails because of a member, the message should show
the operation that was requested, which member blocked it, and why:

```text
error: MyBox has no copy: generated copy needs MyBox.file, and MyOwnedFile
       has no copy (transfer move)
```

## Source stability

- A use with no stance written is a `copy` where the value allows a copy, and
  otherwise a loan. The compiler never adds `move` or `last` to a variable:
  only temporaries, which nothing can use again, are treated as written with
  the strongest stance their own stance allows. A variable's declared stance
  is never applied to a use on your behalf.
- If a later compiler recognizes more last uses, it reports more errors; it
  never starts calling different functions.
- `as last` leaves the variable finished for every type, so your code stays
  correct if a type later adds a real `last` operation.
- A type's `transfer` list reserves stances: code that relies on a reserved
  `move` or `last` keeps compiling once the operation is written. Adding `move`
  to a type without a `transfer` list can make existing code that uses the
  type stop compiling until stances are written at its last uses.
- Reference and borrowing raw-pointer parameters, reference results, and
  members must write their stance. A later version may give them a default
  without changing what any existing program means.
- Adding another version of a function may make a call ambiguous, and the
  compiler reports that. The new version never wins because of where it was
  declared or imported. Adding a `copy` version to a type can change which
  version an existing call with no stance uses, but a `copy` version cannot
  change the caller's value.
- Generated operations exist only for the exact signatures they cover, and only
  when something uses them.
- Formatters and other tools keep every stance and intent acknowledgement, and
  write lists of stances in the standard order.
- Optimizations never change which operations run, what they do to the source,
  which callbacks run, the shape of results, or the order of destruction.

## Boundaries and maturity

This document is current conceptual design, not a formal grammar,
compatibility contract, conformance specification, or compiler mapping.

Focused future work remains for:

- how generic specializations are compared;
- qualifier-generic syntax and constraints;
- stances on each element of an array, slice parameters, and optional
  references;
- detailed pointer provenance beyond the current ownership model;
- which last uses analysis proves across loops and other control flow beyond
  straight-line code and branches;
- async suspension and cancellation;
- cross-thread value preparation and whether it is a stance, result capability,
  or separate contract;
- types whose mutable and immutable forms have different layouts;
- formal diagnostics and analysis contracts;
- complete general swap mechanics; and
- formal grammar, reflection, ABI, and implementation.
