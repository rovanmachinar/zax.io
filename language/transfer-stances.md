# Zax transfer stances

| Field | Value |
| --- | --- |
| Status | Current conceptual design |
| Audience | Human developers reading, writing, or evaluating Zax |
| Applies To | Programmer-visible `loan`, `copy`, `move`, `last`, and `deep`, and when a stance must be written; not a formal specification |
| Implementation State | Not established by this repository |
| Owns | The transfer-stance mental model; the five stances, their order, and fallback; `loan` and what stays on loan; source state after a transfer; when a stance must be written, including at a last use, in generic bodies, and per item of an expanded pack; what a declared stance means in each location; stances inside a body; receiver stance, passing the receiver on, and `self` results; member and binding stances; result stances; common costs, diagnostics, and source stability |
| Does Not Own | Exact callable selection and parameter binding ([function invocation](function-invocation.md)); a type's `transfer` list, generated and defaulted operations, and lifecycle sources ([construction, replacement, and destruction](construction-and-destruction.md)); identity `transfer` lists ([identity types](identity-types.md)); inferred declarations ([declarations and bindings](declarations-and-bindings.md)); cohesive [exceptional result flow](except.md); composition publication and wrapper eligibility ([Zax composition](composition.md)); qualifier axes ([qualifiers](qualifiers.md)); optional wrapper cleanup ([optional values](optional-values.md)); complete [variant transfer](variants.md#copy-deep-move-and-last) or [union transfer](unions.md#whole-union-operations); [reference lifetime](lifetimes-and-references.md); or [pointer ownership and allocation](pointers-and-arenas.md) |
| Source / Provenance | Legacy function, pointer, casting, and constructor input, reconciled with current invocation, construction, qualifier, optional, operator, and documentation design |

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
**transfer stance**. The stance says how much of a value may leave the place
that owns it:

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
ways, and stances do not try to stop that.

### Reading the examples

The examples use a few Zax forms that other documents teach in full:

| Form | Meaning | Explained in |
| --- | --- | --- |
| `name final : (results)(inputs) = { }` | A function: its result slots, then its inputs, then its body | [function invocation](function-invocation.md#callable-prototypes-and-visible-contracts) |
| `+++` | A constructor declared inside a type | [construction, replacement, and destruction](construction-and-destruction.md) |
| `---` | A destructor | [construction, replacement, and destruction](construction-and-destruction.md) |
| `place .= value` | Reconstructive replacement: end the old value in place and build a new one | [construction, replacement, and destruction](construction-and-destruction.md) |
| `_` | The receiver: the instance a type's function was called on | [function invocation](function-invocation.md#receiver-operands) |
| `Type &` | A reference to an existing place rather than a new value | [lifetimes and references](lifetimes-and-references.md) |
| `readonly`, `mutable writable` | Qualifiers saying whether a path may change the value; after a function's inputs, as in `()() readonly`, they apply to the receiver | [qualifiers](qualifiers.md) |
| `+++ replacement` | A replacement constructor: what `.=` runs to give an existing variable a new value | [construction, replacement, and destruction](construction-and-destruction.md#custom-replacement) |
| `(: Source)`, `[{ … }]` | A new value built in place: `(: Source)` builds a default `Source`; `[{ … }]` lists the constructor's inputs | [construction, replacement, and destruction](construction-and-destruction.md#construction-packets) |
| `name$ final : (Items ... : type)()(items ... : Items)` | A generic function: `$` marks it, `Items ... : type` is a list of types chosen per call, and `items ... : Items` receives one argument of each | [generics](generics.md), [variadics](variadics.md) |
| `result # : Type` | A result the caller may ignore; without `#`, ignoring a result is an error | [function invocation](function-invocation.md#required-and-discardable-results) |
| `once` | A function that can be called on the type itself, as in `MyType.inspect()`, without an instance | [Nothing instances](nothing-instances.md) |

## Five stances: how much of the value leaves its owner

Zax has five stances. They differ in how much of the value leaves its owner,
and in what the owner still holds afterward:

| Stance | What leaves the owner | What the owner holds afterward |
| --- | --- | --- |
| `loan` | nothing: the consumer uses the value where it is | the same value, possibly changed if the reference allows changes |
| `copy` | a duplicate | its value, unchanged |
| `move` | the contents | a usable value, emptied or reset as the type's `move` defines |
| `last` | everything | a finished value that can only be destroyed or replaced |
| `deep` | a duplicate that shares nothing with the original | its value, unchanged |

`loan`, `copy`, `move`, and `last` are ordered by how much leaves the owner:
`loan < copy < move < last`. `deep` is not part of that order. Like `copy`, it
leaves the owner unchanged, but it also promises that the duplicate shares
nothing with the original.

When stances are listed together, they are written in this order:
`loan copy move last deep`. A type's `transfer` list uses the same order but
never includes `loan`.

Three habits cover most code:

- **No stance means `copy`.** A reader who sees `send(message)` assumes that
  `send` received a copy and `message` is unchanged. So the compiler never adds
  `move` or `last` for you; you write them.
- **A reference only borrows unless it says otherwise.** A reference with no
  stance is a `loan`: the function can use the value where it is, but cannot
  copy it or take anything from it.
- **Write `as move` when you will use the variable again, and `as last` when
  you are done with it.** This is for variables that hold their own value.
  Through a reference to someone else's value, you can pass the value on with
  at most the stance you were given, as
  [references pass on at most their stance](#references-pass-on-at-most-their-stance)
  explains.

```zax
Message :: type transfer copy move {
  text : String

  +++ final : ()(source : Message mutable writable & move) = default
}

show final : ()(message : Message readonly &) = {
}
// unmarked reference: a loan

keep final : ()(message : Message) = {
}
// by value: builds keep's own Message

message : Message
show(message)           // lends message; nothing leaves it
keep(message)           // copies message
keep(message as move)   // Message's move constructor runs; message is empty but usable
keep(message as last)   // message is finished
```

`Message :: type transfer copy move` says that `Message` allows only the `copy`
and `move` operations. Callers can still write `as last`: `Message` has no
`last` operation, so its `move` constructor runs instead, as
[fallback](#fallback) explains. `= default` asks the compiler to write the
`move` constructor by moving each member. Both are explained in
[construction, replacement, and destruction](construction-and-destruction.md#declaring-which-transfers-a-type-allows).

## One stance on one complete value

A stance applies to the whole value you write it on. It does not apply
separately to each pointer, reference, or member inside that value:

```zax
source : MyValue

moved := source as move
// MyValue's move constructor receives the whole source; that constructor
// decides what happens to each member
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

A member can declare its own stance, and that stance controls transfers from
that member, as [members and bindings](#members-and-bindings) explains. A
stance written on the containing value does not pass down to its members.

A stance can also be written on an expansion of several arguments, such as
`from items as last`. It then applies to each item separately, as
[a stance on several items](#a-stance-on-several-items) explains.

## Copy

`copy` is what you get when you write no stance:

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

  +++ final : ()(source : MyFile readonly & deep) = default
  // error: MyFileId has no deep, so a default deep cannot be built
}
```

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
[`last` without `move`](#last-without-move) shows. A function that receives a
`& move` reference may also change the caller's members through ordinary
writable access, as
[references pass on at most their stance](#references-pass-on-at-most-their-stance)
explains; this guarantee is about what the type's own `move` leaves.

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

  +++ final : ()(source : DatabaseCommand mutable writable & move) = {
    // takes source.results and copies source.query; opens a connection of
    // its own, so source keeps its connection and stays fully usable
  }

  +++ final : ()(source : DatabaseCommand mutable writable & last) = {
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

runBatch final : ()(queries : Query[]) = {
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

A `loan` lets a function use a value where it is. The value as a whole stays
with its owner: the function cannot copy the whole value, move it, or take it.
Whether the function may change the value is a separate question, answered by
`readonly` or `mutable writable`. A `mutable writable` loan may change the
caller's value in place, including its members, but still cannot keep or copy
the whole value.

Most functions only read or change their reference inputs, so a reference with
no stance is a loan:

```zax
retain final : ()(input : MyType) = {
}

display final : ()(input : MyType readonly &) = {
  print(input.label)      // fine: reading a member does not transfer input
  retain(input)             // error: input is on loan; retain would copy it
}

fill final : ()(buffer : MyBuffer mutable writable &) = {
  buffer.append(data)     // fine: a mutable loan may be changed in place
}
```

Writing `input : MyType readonly & loan` means the same thing. A reference that
needs to copy the value, or take from it, must declare `& copy`, `& move`,
`& last`, or `& deep`:

```zax
retainCopy final : ()(input : MyType readonly & copy) = {
  retain(input)             // fine: this reference may copy
}
```

The same holds for receivers: a receiver with no stance is a loan, and a
function that copies or takes its receiver must declare a receiver stance, as
[receiver stance](#receiver-stance) shows.

### Writing `as loan`

`as loan` restricts a call to consumers that take a loan, so a reader sees that
the call takes nothing from the value:

```zax
display(value as loan)
```

It matters when a function has two overloads, one taking a loan and one taking
a value, and the call writes no stance. When the compiler compares stances,
`copy` normally ranks above `loan`, as [fallback](#fallback) shows, so you
might expect the by-value overload to win. It does not: you must write
`as loan` or `as copy`. The two overloads can behave quite differently. A
mutable loan lets the function change your variable, which a by-value copy
cannot, and a by-value function may keep the value it receives:

```zax
update final : ()(input : MyType mutable writable &) = {
}

update final : ()(input : MyType) = {
}

update(value)           // error: both fit; write `as loan` or `as copy`
update(value as loan)   // the loan overload
update(value as copy)   // the by-value overload
```

Complete selection is taught by
[function invocation](function-invocation.md#a-loan-or-a-copy-must-be-stated).

### Everything derived from a loan stays on loan

A loan would be easy to get around if you could make a new reference from it
that allows copying. So anything made from a loan that can reach the whole
value is also a loan:

- a reference initialized from it must be a `loan`;
- a lambda that captures it by reference also holds a loan, and a lambda
  cannot capture a copy of it;
- converting it to the underlying type of an
  [identity type](identity-types.md) gives another loan, so a conversion that
  would copy is an error;
- passing `_` on from a loan receiver passes on a loan; and
- making a raw pointer from it is an error outside `unsafe`, because a raw
  pointer cannot carry the loan.

```zax
display final : ()(input : MyType readonly &) = {
  other : MyType readonly & copy = input   // error: a reference that may copy, bound from a loan
  later := [[ input ]] ()() {              // error: [[ input ]] captures a copy of input
    show(input)
  }
  kept := [[ input: view : & ]] ()() {     // fine: captures a reference, which is still a loan
    retain(view)                           // error: view is on loan; retain would copy it
  }
}
```

`[[ input ]]` captures a copy by default; `[[ input: view : & ]]` captures a
reference named `view`. Captures are taught in
[lambdas and callable composition](lambdas-and-callable-composition.md).

### Where `loan` may appear

- **References, receivers, and reference results only.** A by-value parameter
  always builds a new value, so `input : MyType loan` by value is an error, as
  is `loan` on a by-value local or member.
- **Never in a `transfer` list.** `loan` is not an operation a type defines;
  every type can be lent.
- **Not as the same-type source of a constructor, assignment, or
  replacement.** A constructor that builds a value from a loaned value of the
  same type would just be the copy constructor under another name.
- **As a function result.** A `loan` result lets the caller use the value, as
  its qualifiers allow, but not copy or keep it. The usual lifetime rules for
  returned references apply:

  ```zax
  current final : (result : MyDocument readonly & loan)() = {
  }
  ```

### What a loan does not promise

A loan stops the value being copied, moved, or taken as a whole. It does not
protect the value's members. Each member follows its own declared stance:
through a `readonly` loan a member can be copied, and through a writable loan
it can also be moved or taken, just as it could be changed:

```zax
keepPayload final : ()(payload : Payload) = {
}

refresh final : ()(input : Message mutable writable &) = {   // a loan
  keepPayload(input)                    // error: the whole message is on loan
  keepPayload(input.payload as move)    // fine: payload follows its own stance,
                                        // and the loan is writable
}

inspectOnly final : ()(input : Message readonly &) = {
  keepPayload(input.payload)            // fine: copies the member
  keepPayload(input.payload as move)    // error: readonly access cannot change payload
}
```

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
takes less instead. For each stance you can write, the table lists what the
compiler can use, best match first:

| You write | What can receive it, best match first |
| --- | --- |
| Implicit, or `as copy` | `copy`, then `loan` |
| `as move` | `move`, then `copy`, then `loan` |
| `as last` | `last`, then `move`, then `copy`, then `loan` |
| `as deep` | `deep`, or a `loan` consumer that builds nothing; otherwise error |
| `as loan` | `loan` only |

```zax
next : Message = message as last    // Message has no last constructor:
                                    // its move constructor builds next
```

A reference parameter builds nothing, so there is nothing to fall back
between. Its declared stance is what the function may do to the caller's
value, and a call must write at least that stance:

```zax
hold final : ()(input : Message mutable writable & last) = {
}

hold(message as move)   // error: hold requires `as last`; nothing falls back
hold(message as last)   // fine
```

Two more rules apply to choosing a function. A by-value parameter declared
`move` or `last` accepts only calls that write at least that stance, as
[by-value parameters](#by-value-parameters) explains. And when a function has
both a loan overload and a by-value overload, a call with no stance must choose
with `as loan` or `as copy`, as [writing `as loan`](#writing-as-loan) shows.

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
    source : Ledger readonly & copy
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
    source : Ledger readonly & copy
  ) = {
  }

  +++ final : ()(
    source : Ledger mutable writable & move
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

No stance means `copy`. Two separate rules decide when that is not enough.

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

  +++ final : ()(source : Message mutable writable & move) = default
}

MyOwnedFile :: type transfer move {
  handle : Integer

  +++ final : ()(source : MyOwnedFile mutable writable & move) = default
}

relay final : ()(input : Message) = {
}

sendMessage final : ()(input : Message move) = {
}

hold final : ()(input : Message mutable writable & last) = {
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
sendMessage(message)        // error: sendMessage requires at least move
hold(message)               // error: hold's reference requires last

archive(file as last)       // fine
sendMessage(message as move)
hold(message as last)
```

The compiler never adds `move` or `last` to make these calls work, because a
reader who sees no stance assumes the function took a copy and `message` is
unchanged. This applies even when the type has nothing to take:

```zax
bump final : ()(value : Integer mutable writable & last) = {
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
locals and parameters. A function's result with no declared stance needs no
check, because it is always passed on as `last`, as
[unnamed values](#unnamed-values-are-already-treated-as-last) explains. A
reference only points at a value someone else holds, so the last use of a reference is not the last use of
that value. The rule applies where the value itself is held:

```zax
forwardMessage final : ()(input : Message readonly & copy) = {
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
as if its operation were already written, even when only `copy` exists today,
as [reserved stances](#reserved-stances-count-as-implemented) shows. So "nothing
could differ" holds only for a type such as `Integer`, which has no `move` or
`last` and reserves neither.

Here is the first case. With no stance, the call works, but `as last` would
call the other overload:

```zax
store final : ()(value : Integer readonly & copy) = {
}

store final : ()(value : Integer mutable writable & last) = {
}

store(total)   // selects the & copy store; at a last use: error, because
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

sendHandle final : ()(input : MyHandle move) = {
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
no stance, to a plain by-value parameter or into a new variable, stops
compiling until a stance is written. Declaring `transfer copy move` from the
start avoids that.

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

### Unnamed values are already treated as `last`

A value with no name cannot be used again after the expression that uses it,
so it is treated as if written `as last`. That covers a value built directly in
an expression, such as `(: Source)`, and a function's by-value result that
declares no stance:

```zax
fromTemporary := (: Source).get()   // the unnamed Source is treated as `as last`
item := makeItem()                  // makeItem declares no result stance:
                                    // item is built from the result with last
```

"No stance means `copy`" is about variables you can use again. An unnamed
value has no name, so nobody can use it again and nothing is hidden from the
reader. A function that wants its callers to receive a copy declares
`result : Item copy`, and a caller can write a stance on the call, as in
`makeItem() as copy`.
[Receiver stance](#compiler-created-temporary-receivers) shows the receiver
case in full, and [results with no declared stance](#results-with-no-declared-stance)
shows results.

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

### A stance on several items

A function that accepts any number of arguments receives them as a
[pack](variadics.md) and can pass them on with `from items`. The pack is not
one value. Each item is its own complete value, so a stance written on the
expansion applies to each item separately, and both rules above apply to each
item:

```zax
sendAll$ final : (Items ... : type)()(items ... : Items last) = {
}
// each item's parameter is declared last, as `input : Message last` is for one value

forwardAll$ final : (Items ... : type)()(items ... : Items) = {
  sendAll(from items)
  // error at any use, for every item type including Integer: sendAll
  // requires last, and no stance means copy
}

forwardLast$ final : (Items ... : type)()(items ... : Items) = {
  sendAll(from items as last)   // last applies to each item separately
}
```

`items ... : Items last` writes the stance after the pack's type parameter, the
same way as for a single parameter.
[Variadics](variadics.md#a-stance-applies-to-each-item) uses the same
position. The exact spelling is not final yet.

A function that forwards a pack therefore states its intent, just as it would
for a single value. Each item also keeps its own type, including whether it is
a reference, so an item that is a reference stays a reference.

Sometimes each item must go through a call before it is passed on. Here each
message is encoded into a packet, and the packets are sent together:

```zax
encode final : (packet : Packet)(message : Message last) = {
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
sendPair final : ()(header : Header move, body : Body) = {
}

forwardPair$ final : (Items ... : type)()(items ... : Items)
  requires Items count == 2 = {
  sendPair(items[0] as move, items[1] as copy)
}
```

[Passing items on](variadics.md#passing-items-on) teaches the complete pack
model.

## Declared stances by location

A stance written in a declaration means something different depending on where
it is written. A parameter's stance is a contract with the caller about how the
value arrives. A local's stance is its author restricting how that variable may
later be used. A result's stance decides how the caller's destination is built
from it.

| Location | Example | What the declared stance means |
| --- | --- | --- |
| By-value parameter | `input : Message move` | A call must write at least this stance to use this function |
| Reference parameter | `input : Message mutable writable & last` | What the function may do to the caller's value; a call must write at least this stance |
| Reference with no stance | `input : Message readonly &` | `loan` |
| Receiver | `publish final : ()() deep` | The stance a call must write on the receiver to use this function; none means `loan` |
| Source of a constructor, assignment, or replacement | `source : Message readonly & copy` | Which operation this is: here, the copy constructor |
| Local, alias, or member | `report : Report move` | Every transfer from it must be written with this stance or a stronger one |
| Type alias | `CopyView :: alias type DeepView copy` | The same requirement, for every declaration that uses the alias |
| Result | `result : Message move` | The stance the caller's destination is built with; with none, a by-value result is passed on as `last` |
| Type or identity `transfer` list | `MyOwnedFile :: type transfer move` | Every transfer the type allows |

As a reminder, a **transfer** is any use that builds or assigns a new value
from a place: passing it to a by-value parameter, initializing a variable,
assigning, storing it in an array, capturing it in a lambda, or returning it.
Passing it to a `loan` reference, or calling a function with a `loan`
receiver on it, is not a transfer.

### By-value parameters

A by-value parameter's stance says the least a call must write to use the
function. A by-value parameter with no stance is called a **plain**
parameter; it behaves exactly like one declared `copy`. A call with no stance,
or with `as copy`, can use only plain and `copy` parameters. A call with
`as move` prefers a `move` parameter and can also use a plain one. A call with
`as last` prefers `last`, then `move`, then plain:

```zax
emit final : ()(input : Message) = {
}

emit final : ()(input : Message move) = {
}

emit final : ()(input : Message last) = {
}

emit(message)           // plain emit
emit(message as move)   // move emit
emit(message as last)   // last emit
```

With only the `move` overload, `emit(message)` is an error at any use: the
function asked for at least `move`. A by-value parameter declared `deep`,
such as `input : Message deep`, can be used only by a call written `as deep`,
because `deep` is outside the `copy`, `move`, `last` order and nothing else
counts as stronger.

The parameter's stance only decides which function a call can use. It does
not decide how the parameter is built: the stance the caller writes chooses
which constructor builds it, as
[by value and by reference](#by-value-and-by-reference) explains. Inside the
function, uses of the parameter with no stance are copies again.

### Reference parameters

A reference parameter refers to the caller's own value. Its stance says what
the function may do to that value, and the function's body depends on it. So a
call must write at least that stance, every time and for every type:

```zax
hold final : ()(input : Message mutable writable & last) = {
}

remember final : ()(input : Message readonly & copy) = {
}

show final : ()(input : Message readonly &) = {
}

// each line below is a separate alternative
hold(message)            // error: hold requires `as last`
hold(message as last)    // fine
remember(message)        // fine: no stance means copy
show(message)            // fine: a loan accepts any call
```

A reference parameter with no stance is a `loan`. Declare `& copy` only when
the function copies the value through the reference.

### Receivers

A receiver's stance says which stance a call must write on the receiver to use
that function. A receiver with no stance is a `loan`, and the version with no
receiver stance is used when no version with a matching receiver stance
exists.
[Receiver stance](#receiver-stance) covers the details, including temporaries
and calls on a type rather than an instance.

```zax
Document :: type {
  describe final : (result : String)() = {
  }
  // no receiver stance: _ is on loan

  snapshot final : (result : Document)() copy = {
    return _ as copy
  }
  // copies its receiver, so declares a copy receiver
}
```

### Lifecycle sources

A constructor, assignment, or replacement that builds a value from another
value of the same type declares that source with `& copy`, `& move`,
`& last`, or `& deep`. Here the stance says which operation this is:
`& copy` declares the copy constructor, which runs when a value is copied, and
`& move` declares the move constructor, which runs for `as move`:

```zax
Archive :: type transfer copy move last deep {
  history : History

  +++ final : ()(source : Archive readonly & copy) = {
    // copy construction
  }

  +++ final : ()(source : Archive mutable writable & move) = {
    // may take resources from source, leaving it fully usable
  }

  +++ final : ()(source : Archive mutable writable & last) = {
    // may take everything; source only needs to survive destruction
  }

  +++ final : ()(source : Archive readonly & deep) = {
    // independent construction
  }
}
```

A `move` or `last` source is usually `mutable writable`, because taking a
resource means changing the source so that it no longer owns that resource.
A `readonly` `move` or `last` source is allowed when the operation really does
not change the source, for example when the resource it takes lives somewhere
outside the value.

Inside these operations, the source as a whole is never transferred; doing so
would call the same operation again. The operation works on the source's
members instead: a copy constructor copies them, and a move constructor may
take them. The other parameters of a constructor are ordinary parameters. Many
of them are only read, so `& loan` is often the right stance for them. The full
rules are in
[construction, replacement, and destruction](construction-and-destruction.md#lifecycle-sources-are-never-transferred-whole).

### Locals, aliases, and members

A stance declared on a local variable, a local reference, or a member is a
requirement: every transfer from that place must be written with the declared
stance or a stronger one.

```zax
byValue final : ()(input : MyType) = {
}

inspectCopying final : ()(input : MyType readonly & copy) = {
  byValue(input)   // copies the referent
}

display final : ()(input : MyType readonly &) = {
  byValue(input)   // error: input is on loan
}

myType : MyType move

// each line below is a separate alternative
byValue(myType)              // error: myType must not be copied
byValue(myType as move)      // fine
byValue(myType as last)      // fine: stronger than move
byValue(myType as copy)      // error: myType was declared move
display(myType)              // fine: display takes a loan; nothing is transferred
myType.describe()            // fine: describe's receiver has no stance, so it only lends myType
inspectCopying(myType)       // error: & copy could copy a move-restricted value
```

`input` inside `inspectCopying` is a reference, so passing it on is never the
last use of the caller's value, and the intent rule does not apply there.

The reason for the requirement is the same as for the necessity rule: a reader
who sees no stance assumes a copy, and the author of `myType` has said it must
not be copied. Lending is still allowed. Otherwise a variable declared `move`
could not be passed to most functions, and most of its methods could not be
called.

The requirement controls only which stance you may write. What actually runs
still follows the fallback rules. If `MyPoint` has only `copy`,
`point : MyPoint move` is still allowed: writing `as move` runs `copy`.

A function that wants the restriction on its input rebinds it, because a
parameter's stance describes only how the value arrives:

```zax
process final : ()(input : MyType move) = {
  restricted : MyType mutable writable & move = input as move
  // `as move` is needed: input starts from copy in the body. Binding a
  // reference moves nothing; restricted refers to input itself.
  // each call below is a separate alternative
  something(restricted)          // error if something takes MyType by value
  something(restricted as move)  // fine
  something(input as copy)       // fine if MyType has copy: input itself is unrestricted
}
```

The remaining rules for declared places:

- **A place declared `deep`** requires every transfer to be `as deep`. `deep`
  is outside the chain, so nothing counts as stronger.
- **A place's restriction does not extend to its members.** A member is
  governed by its own declared stance, as
  [members and bindings](#members-and-bindings) shows.
- **A declared stance never settles an overload choice.** A place declared
  `copy` still needs `as loan` or `as copy` at a call where both a loan and a
  by-value function fit. The reader of the call should not need to look up a
  declaration.
- **`loan` on a by-value local or member is an error.** Lending is not a
  transfer, so a `loan` requirement would have nothing to govern.
- **A declared stance the type can never satisfy is an error.** If the type's
  `transfer` list rules out every stance the declaration would accept, the
  declaration itself is an error. A type without a `transfer` list can still
  add operations later, so for such a type the error is reported where a
  transfer is attempted.

  ```zax
  DeepOnly :: type transfer deep {
  }

  shared : DeepOnly copy   // error: copy, move, and last are all unavailable on DeepOnly
  point : MyPoint move     // fine: MyPoint has copy; `as move` falls back to it
  ```

The bodies of `display` and `inspectCopying` show why the restriction cannot leak
through a call. A restricted place may be lent but not bound to a reference
that could copy it, and nothing derived from a loan can be copied as a whole.

The restriction covers only declared places. Values stored in array elements,
or in members that declare no stance, are governed by their type. A guarantee
for every value of a kind, wherever it is stored, comes from a `transfer` list
on the type or an identity.

### Same spelling, different location

The same declaration text means different things on a parameter and on a local
reference. On a parameter, `& move` says what the function may do to the
caller's value: take its contents. Inside the function, uses with no stance
are still copies, and the function may write any stance up to `move` on
`input`, but nothing stronger:

```zax
drain final : ()(input : Message mutable writable & move) = {
  relay(input)              // fine: copies the referent
  relay(input as move)      // fine: takes from the caller's message
  relay(input as last)      // error: the caller allowed only move
}
```

On a local reference, the same spelling is a requirement on every transfer from
that reference:

```zax
message : Message
taker : Message mutable writable & move = message as move
relay(taker)                // error: taker requires `as move` or `as last`
relay(taker as move)        // fine
```

### Type aliases

A type alias can carry a stance, so a project can name a kind of variable
once instead of repeating the stance on every declaration. Every declaration
that uses the alias gets the stance as a requirement, as if it had written the
stance itself:

```zax
OwnedPayload :: alias type Payload move
// a Payload that must be moved or finished, never copied

send final : ()(payload : Payload) = {
}

draft : OwnedPayload
send(draft)            // error: draft was declared through OwnedPayload, which requires move
send(draft as move)    // fine
```

An alias can also be built from another alias, replacing its stance:

```zax
DeepView :: alias type Payload readonly & deep
CopyView :: alias type DeepView copy
```

`CopyView` is still a readonly reference to a `Payload`, but its stance is
`copy` instead of the `deep` it would otherwise get from `DeepView`. Declaring
or using an alias copies nothing; it only changes what declarations that use
it require. The order in which alias properties combine is explained in
[declarations and bindings](declarations-and-bindings.md#exact-aliases-and-property-overlays).

### Results

A result's declared stance says how the caller's destination is built from it.
It is part of the function's signature, and it is not a restriction on the
caller:

```zax
makeMessage final : (result : Message move)() = {
}

stored := makeMessage()   // stored is built from the result with move
copied := stored          // fine: stored is unrestricted
show(stored)              // stored is used again, so the copy above was not its last use
```

Inside the function that produces it, `result` behaves like a parameter: a use
with no stance is a copy, and anything stronger must be written. [Results and forwarding](#results-and-forwarding)
covers result mapping, forwarding, and `self` results.

### Written `copy` versus no stance

Writing `copy` in a declaration does different things by location:

- **On a result**, `result : Payload copy` is the function's author saying
  that callers receive a copy. With no stance, the result would be passed on
  as `last`, as
  [results with no declared stance](#results-with-no-declared-stance) shows.
- **On a local, alias, or member**, a declared `copy` is a requirement, like any
  declared stance. It does not state intent at a use, so it does not remove a
  last-use intent error:

  ```zax
  useMessage2 final : ()() = {
    message : Message copy
    relay(message)   // error: last use; `as last` would run Message's move
                     // constructor, so write the stance here
  }
  ```

### Inferred declarations take the type only

An inferred declaration takes its initializer's type, never a declared stance.
In the example above, `stored` is a `Message` with no declared stance. Because
a declared stance is a requirement, inheriting one through inference would
restrict a variable whose declaration shows no stance. To restrict `stored`,
write the stance: `stored : Message move = makeMessage()`. Complete inference belongs to
[declarations and bindings](declarations-and-bindings.md).

### Transfer lists on types and identities

A type can list, in its definition, every transfer it allows:

```zax
MyOwnedFile :: type transfer move {
  handle : Integer
}

inspectFile final : ()(file : MyOwnedFile readonly &) = {
}

useFile final : ()() = {
  file : MyOwnedFile
  inspectFile(file)                    // fine: a loan duplicates nothing
  other : MyOwnedFile = file           // error: MyOwnedFile has no copy
  owner : MyOwnedFile = file as last   // fine: falls back to move; file is finished
}
```

What this means for code that uses the type:

- an operation for a stance missing from the list can never exist: writing it
  in the type, or adding it from a partial, is an error;
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

## Stances inside a body

A parameter's declared stance says how the value arrives. Inside the function,
a use with no stance gets a default, and each default is the safest choice
given what the caller allowed. You can always write a different stance
explicitly.

| Inside the body of | Whole value | Members without their own stance |
| --- | --- | --- |
| By-value parameter, any stance | `copy` | `copy` |
| `result` slot | `copy` | `copy` |
| Reference with no stance or `& loan`, or receiver with no stance | no transfer | `copy`; through writable access, `move` or `last` when written |
| `& copy` reference, or `copy` receiver | `copy` | `copy` |
| `& move` or `& last` reference, or `move` or `last` receiver | `copy` | `copy` |
| `& deep` reference, or `deep` receiver | `deep` | `deep` |
| Source of a constructor, assignment, or replacement | never transferred as a whole | depends on the operation, as described below |

A member that declares its own stance follows its declaration instead of this
table: the declared stance is a requirement on every transfer from that
member, as [members and bindings](#members-and-bindings) explains.

The safest choice is not always `copy`:

- **A `move` or `last` reference or receiver** defaults to `copy`. Nothing is
  taken unless the function writes `as move` or `as last`, so a helper call in
  the middle of the function cannot take the value by accident.
- **A `deep` reference or receiver** defaults to `deep`. The caller asked for
  any duplicate to be independent, and an ordinary copy might share the
  caller's resources. Writing `as copy` explicitly is allowed, so a function
  that chooses to share shows it.
- **A by-value parameter**, including a `deep` one, is the function's own
  value, so copying it later cannot affect the caller.
- **A loan** never transfers the whole value. Its members follow their own
  stance: they may be copied, and through writable access also moved or taken
  when the function writes that stance.

```zax
dispatch final : ()(input : MyType move) = {
  something(input)           // copies, because no stance is written; MyType must have copy here
  something(input as move)   // the transfer is visible where it happens
}

prepareForSend final : ()(input : Message mutable writable & move) = {
  log(input)                         // copies or lends; input is untouched
  sendPayload(input.payload as move) // the payload is taken deliberately here
}

publishArchive final : ()(input : Archive readonly & deep) = {
  stored : Archive = input            // deep by default
  scratch : Archive = input as copy   // allowed: the callee visibly chose to share
  archiveHistory(input.history)       // deep: an unstanced member through a deep reference
}
```

If `log(input)` were passed `move` automatically, it could call a version of
`log` that takes the message, and `input` would be empty before `sendPayload`
runs. For the same reason, you cannot make another reference that may take
with `last` from `input` without writing the stance:

```zax
handOver final : ()(input : Message mutable writable & last) = {
  stored : Message mutable writable & last = input
  // error: a use of input with no stance is a copy; the reference requires last

  explicit : Message mutable writable & last = input as last
}
```

`explicit` refers to the same message as `input` and may take it with `last`.
The usual lifetime checks for references still apply to it.

Inside a constructor, assignment, or replacement, the source as a whole is
never transferred. Its members are copied from a `& copy` source, deep-copied
from a `& deep` source, and copied from a `& move` or `& last` source unless
the operation writes `as move` or `as last` to take them. See
[lifecycle sources are never transferred whole](construction-and-destruction.md#lifecycle-sources-are-never-transferred-whole).

### References pass on at most their stance

A reference or receiver reaches someone else's value, and the stance the caller
wrote is what the caller expects to happen to that value. So a function can
pass the whole value on with the stance it was given, or a weaker one, but
never a stronger one:

```zax
keep final : ()(message : Message) = {
}

drain final : ()(input : Message mutable writable & move) = {
  keep(input as last)      // error: the caller allowed only move
  keep(input as move)      // fine
}

message : Message
drain(message as move)
message.send()             // fine: after move, message is still a usable message
```

If `drain` could write `input as last`, the caller's `as move` would promise
nothing: `message` could be finished when `drain` returns. A by-value
parameter, a local, or a result is different. It is the function's own value,
so the function may pass it on with any stance.

Passing on less than you were given is always allowed, and each use chooses
its own stance:

```zax
process final : ()(source : Message mutable writable & move) = {
  show(source)                                // lends it
  view : Message readonly & copy = source     // a reference that may only copy
  snapshot := source                          // copies: no stance means copy
  keep(source as move)                        // still allowed afterward
}
```

The limit is on the whole value. Members follow their own declared stance.
Through a writable reference, a function may already change a member's
contents however it likes, including emptying it, so it may also take a
member with `last`:

```zax
keepPayload final : ()(payload : Payload) = {
}

drainPayload final : ()(input : Message mutable writable & move) = {
  keepPayload(input.payload as last)   // fine: payload follows its own stance
}
```

At the call site, the compiler cannot see that `payload` was left finished,
just as it cannot see that a function emptied a member through ordinary
writable access. `as move` promises a usable `Message`; what is left in its
members is up to the function and the type. Destructors and replacement
constructors already have to cope with whatever `last` leaves behind.

## By value and by reference

A by-value parameter gives the function its own value. A reference parameter
gives it access to the caller's value. A function may have either, or
overloads with both.

For a by-value parameter, three separate things happen when the function is
called:

1. **Choosing the function.** A call must write at least the parameter's
   declared stance, as [by-value parameters](#by-value-parameters) shows.
2. **Building the parameter.** The stance the caller wrote chooses which of
   the type's constructors builds the parameter, with the usual fallback. A
   `move` parameter called with `as last` still runs the `last` constructor
   if the type has one.
3. **Inside the function.** A use of the parameter with no stance is a copy,
   whatever the parameter's declared stance.

The type's constructor builds the parameter, and that constructor receives a
reference to the caller's variable. That is how a `move` or `last` constructor
takes the contents out of the caller's variable. By-value parameters are built
exactly the way local variables are:

```zax
MyMoveOnlyType :: type transfer move {
  handle : Integer

  +++ final : ()(source : MyMoveOnlyType mutable writable & move) = {
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
| `Message` | plain | copy runs | move runs | move runs (no `last`) | Yes: `last` would move instead of copy |
| `Message` | `move` | error: requires at least `move` | move runs | move runs | Already required at any use |
| `Integer` | plain | copy runs | copy runs | copy runs | No |
| `Integer` | `move` | error: requires at least `move` | copy runs | copy runs | Already required at any use |
| `MyOwnedFile` | plain | error: no copy | move runs | move runs | Already required at any use |
| `MyOwnedFile` | `move` | error: requires at least `move` | move runs | move runs | Already required at any use |

The caller's variable ends in the state left by the operation that ran, except
that `as last` always leaves it finished.

A reference parameter refers directly to the caller's variable. Nothing is
built, and the function may change or take from the caller's value as far as
the reference's stance allows:

```zax
prepareForSend final : ()(input : Message mutable writable & move) = {
  sendPayload(input.payload as move)   // takes from the caller's message
}
```

A by-value `deep` parameter is built as a complete independent copy before the
function starts. A readonly `& deep` reference is often better: nothing is
copied up front, and the function makes independent copies of just the parts
it needs:

```zax
prepareTemplate final : ()(input : Archive readonly & deep) = {
  archiveHistory(input.history)   // deep: the body default for a deep reference
}
```

### Choosing value or reference

Suppose a function has two overloads, one taking a `mutable writable & move`
reference and one taking a by-value `move` parameter, and the caller's variable
is mutable and writable. A call with `as move` could use either. The reference
overload is chosen, because it does not build another value:

```zax
prepareForSend final : ()(input : Message mutable writable & move) = {
}

prepareForSend final : ()(input : Message move) = {
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

send2 final : ()(input : Packet last) = {
}
// a Message converts to a Packet

send2(message as last)
// error: ambiguous; the first matches the type exactly, the second matches
// the stance exactly, and neither is better at both
```

The complete overload rules are in
[function invocation](function-invocation.md#transfer-aware-value-and-reference-parameters).

## Receiver stance

A function defined in a type has a hidden input: the instance it is called on,
written before the dot in `draft.publish()`. That instance is the **receiver**.
A type may want different versions of a function depending on whether the
receiver is lent, copied, moved, or finished, so a function can declare a
receiver stance after its signature:

```zax
Document :: type {
  publish final : ()() = {
    // no receiver stance: _ is on loan; used when no other version matches
  }

  publish final : ()() deep = {
    // exact deep receiver
  }
}

draft : Document
draft.publish()               // unmarked publish
(draft as deep).publish()     // exact deep publish
```

The version with no receiver stance takes the receiver as a `loan`, so it
accepts any call. Three rules decide which version runs:

- **The receiver's stance comes from the call, never from the variable's
  declaration.** A variable declared `document : Document deep` still needs
  `(document as deep).publish()` to call the `deep` version.
- **A call that writes a stance** uses the version with exactly that receiver
  stance, otherwise the version with no receiver stance. In the example above,
  `(draft as deep).publish()` uses the `deep` version. Versions with a
  receiver stance do not fall back to each other: a `move` version is not
  used for a call written `as last`, and a `last` version is not used for
  `as move`.
- **A call that writes no stance** uses the version with no receiver stance,
  which lends the receiver. If the type has no such version, it uses the
  `copy` version; otherwise the call is an error. A stronger version is never
  chosen without its stance written.

Calling a function on a value therefore lends the value unless the call says
otherwise:

```zax
Report :: type {
  render final : (result : String)() = {
  }

  render final : (result : String)() copy = {
  }

  drain final : ()() move = {
  }
}

report : Report
report.render()                  // the version with no receiver stance: lends report
(report as copy).render()        // the copy version, written on purpose

restricted : Report move
// each call below is a separate alternative
restricted.render()              // fine: lends restricted
(restricted as copy).render()    // error: restricted may not be copied
restricted.drain()               // error: drain requires `as move`
(restricted as move).drain()     // fine
```

Adding a `copy` version to a type never changes what existing calls with no
stance do, because they still use the version with no receiver stance. When a
type has only a `copy` version, a call with no stance uses it, and the
variable's declared stance must allow copying:

```zax
Counter :: type {
  snapshot final : (result : Counter)() copy = {
    return _ as copy
  }
}

counter : Counter
counter.snapshot()               // fine: the copy version
locked : Counter move
locked.snapshot()                // error: the copy version would copy locked,
                                 // which is declared move
```

Parameters have one rule that receivers do not: when a function has both a
loan overload and a by-value overload, a call with no stance must choose with
`as loan` or `as copy`, as [writing `as loan`](#writing-as-loan) shows. A
by-value overload gets a separate value, so the choice matters. Every receiver
version works on the caller's own object, so receivers lend by default
instead.

At a last use, the intent rule applies to receivers just as to arguments: if
`as last` would call a different version, a call with no stance must state
one.

A function's receiver stance, parameter stances, and result stances are
separate; each describes its own input or output.

### `_` is the receiver path

Inside the function, `_` stands for the receiver exactly as the caller wrote
it, with the same permissions. Passing `_` on passes the receiver with the
receiver's stance. From a version with no receiver stance, that is a loan:

```zax
keepShape final : ()(shape : Shape) = {
}

showShape final : ()(shape : Shape readonly &) = {
}

Shape :: type {
  describe final : ()() readonly = {
    showShape(_)    // fine: lends the receiver on
    keepShape(_)    // error: _ is on loan; keepShape would copy it
  }
}
```

The defaults for each receiver stance are in
[stances inside a body](#stances-inside-a-body). Inside a `move` or `last`
version, a call that uses `_` with no stance copies or lends it; taking from
`_` must be written:

```zax
Document :: type {
  publish final : ()() last = {
    _.inspect()                      // no stance: copies or lends _
    (_ as last).finishPublishing()   // calls finishPublishing's last version on _
  }
}
```

A version's receiver stance also limits what it may pass on: `_ as last` can
be written only in a `last` version, and `_ as move` only in a `move` or
`last` version, as
[references pass on at most their stance](#references-pass-on-at-most-their-stance)
explains.

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
myself : Task mutable writable & last = _ as last
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

### Receiverless `once` calls

A `once` function can be called on the type itself, as in
`MyResource.inspect()`, with no instance. There is then no variable to write a
stance on, so it is a call with no stance and follows the rules above. The
receiver is the type's Nothing instance, a stand-in value that every type has
for calls like this:

```zax
MyResource :: type {
  inspect final once : ()() = {
  }

  consume final once : ()() last = {
  }
}

MyResource.inspect()  // the version with no receiver stance
MyResource.consume()  // error: a call on the type writes no stance, and consume requires last
```

A call on an instance uses the stance written on that instance:

```zax
(resource as last).consume()
```

The compiler decides the receiver's stance from how the call is written, not
at run time, and an alias or stored function value keeps the same decision.
Calls on a type and Nothing instances are explained in
[Zax Nothing instances](nothing-instances.md#receiver-stance-is-statically-known).

### Compiler-created temporary receivers

A temporary value built directly in an expression has no name, so nothing can
use it after the call it is passed to. It is therefore treated as if written
`as last`, even though no stance is written:

```zax
Source :: type {
  get final : (
    result : Payload copy
  )() copy = {
    // keeps a named receiver unchanged and returns a copy
  }

  get final : (
    result : Payload last
  )() last = {
    // reuse resources from a receiver whose temporary lifetime is ending
  }
}

fromTemporary := (: Source).get()

source : Source
fromNamed := source.get()
source.reset()   // source is used again, so the get above was not its last use
```

The unnamed `(: Source)` is treated as `as last`, so the `last` version of `get`
runs, and it may move the temporary's resources into its result. The named
`source` has no stance written, so the `copy` version runs.

You can still write a different stance on a temporary:

```zax
fromTemporaryCopy := ((: Source) as copy).get()
```

## Members and bindings

A member's own declared stance is a requirement, just like a local variable's.
A member with no stance follows the defaults in
[stances inside a body](#stances-inside-a-body), based on how you reached it.
A stance on the containing variable does not pass down to its members:

```zax
Package :: type {
  label : Label move
  weight : Integer
}

package : Package move
total := package.weight            // fine: weight declares no stance
// each call below is a separate alternative
ship(package.label)                // error: label's own declaration requires move;
                                   // write `as move` or `as last`
ship(package.label as move)        // fine
```

Array elements are governed by their type. A declared stance on the array
variable restricts transfers of the array, not copies of its elements.

### References you declare

A reference you declare is a new reference with its own stance. Unmarked, that
stance is `loan`, as for parameters and receivers:

```zax
consumeBuffer final : ()(buffer : Buffer) = {
}

owner : Buffer move

alias : Buffer mutable writable & = owner              // unmarked: a loan
consumeBuffer(alias)                                   // error: alias is on loan
taker : Buffer mutable writable & move = owner as move
consumeBuffer(taker as move)                           // fine: Buffer's move constructor runs
```

Binding a restricted place to a reference that could copy it is an error,
because the restriction would otherwise leak. Typed loop bindings are declared
references too:

```zax
each value : MyValue readonly & from values {   // declared reference: a loan
  showValue(value)   // showValue takes a loan
}
```

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

inspectChoice final : ()(message : MyOwnerChoice readonly &) = {
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
may take the `payloadSize` region with `last`. If the
caller picks a different anchor, the guarantee does not apply. Compatible
shapes and anchors are explained in
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
hand-written body can always pass `source.fuel` itself. The generated wrapper writes a stance when it passes
the member on, chosen by how the outer parameter is declared:

| Outer parameter | Stance the wrapper writes for the member |
| --- | --- |
| By value with no declared stance, `move`, or `last` | `last` |
| By value, declared `deep` | `deep` |
| By reference | the stance the outer reference parameter declares |

A by-value `source` belongs to the wrapper, and passing its member on is its
last use, so the wrapper writes `last` and the member's operation may reuse
its resources. The usual fallback applies, and nothing is free: if `source` is
a plain parameter and the call writes no stance, building `source` copies the
`Tank`, and the member's operation may copy `fuel` again.

An outer `deep` parameter passes its member on as `deep`, never as `last`. An
outer `& move` reference passes its member on as `move`, never as anything
stronger.

When the wrapper returns, it destroys what is left of `source`, including the
finished member. When the wrapper can be generated, and how results are
passed back, are explained in
[Zax composition](composition.md#forwarding-values-stances-and-results).
If the type's destructor cannot handle a finished member, the wrapper cannot
be generated, and the type must write the wrapper's body by hand.

## Results and forwarding

A result's declared stance says how the caller's destination is built from
it. Inside the function, `result` behaves like a parameter: a use with no
stance is a copy:

```zax
makeMessage final : (
  result : Message move
)(
  template : Message readonly & copy
) = {
  result.+++(template as copy)   // runs Message's copy constructor to build result
  show(result)           // show takes a loan: nothing leaves result
  keep(result)           // keep takes Message by value: copies, since no stance is written
}

stored := makeMessage(template)   // stored is built from the result with move
```

A function's body is written once, against its own declared result stance. You
can give the same body a second name with a different result stance, and
callers of that name receive the result with the new stance. The body does
not change. For example, a function can allocate its result and hand ownership
of the allocation to the caller:

```zax
makeValue final : (
  result : MyValue * last = @
)() = {
  // = @ allocates the result before the body runs; last hands the
  // allocation to the caller
}

peekValue final : (
  result : MyValue * copy = @
)() = makeValue
// the same body; a caller of peekValue borrows the pointer by default, and
// the allocation is freed after the call unless the caller takes it over
```

A caller of `peekValue` may use the pointer only while the allocation is still
alive, and the compiler rejects a caller that keeps it longer. A caller that
writes `peekValue() as last` takes ownership after all. When a second
name like this is allowed is explained in
[compatible visible prototypes](function-invocation.md#compatible-visible-prototypes).

When a function returns another function's result, two declarations are
involved:

- the inner function's result stance decides how the outer function's result
  is built;
- the outer function's result stance decides what its own caller receives.

Result construction, mapping, and completion are defined by
[function invocation](function-invocation.md).

A failure result works the same way. Here the failure value is a `unique`
pointer, which cannot be copied, and the failure result is declared `last`:

```zax
openResource final : (
  resource : MyResource * unique last,
  failure except : MyOpenError * unique last
)(
  name : String readonly &
) = {
  if cannotOpen(name)
    except failure: makeOpenError(name) as last

  return makeResource(name) as last
}

resource := openResource(name) catch failure {
  error : MyOpenError readonly & = failure.   // failure. is the value the pointer refers to
  report(error)
  return
}
```

The `catch` block receives the failure value with the `last` stance that
`openResource` declares. With no stance on the failure result it would still
receive it with `last`, because a failure result with no stance is passed on
as `last`, like any [result with no declared stance](#results-with-no-declared-stance).
Declaring the failure result `copy` would be an error here, because a
`unique` pointer cannot be copied. A function that passes a failure on to
its own caller with `except` decides, with its own failure result, how its
caller receives it. Failures are explained in full in
[Zax exceptional result flow](except.md#transfer-ownership-and-references).

### `self` results carry the caller's variable

A function declared with a `self` result returns its receiver, so calls can be
chained, as in a builder:

```zax
Builder :: type {
  name : String
  size : Integer

  setName final : (result self : Builder &)(name : String readonly &) = {
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
is the caller's own variable. This is why `result self : Builder &` is not a
loan, although other references written with no stance are: a `self` result
takes its stance from the call.

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

### Results with no declared stance

A by-value result holds its own value only until the caller uses it, and
nothing can use it again after that. So a result whose declaration writes no
stance is passed on as `last`:

```zax
consumePayload final : ()(input : Payload readonly &) = {
}

consumePayload final : ()(input : Payload mutable writable & last) = {
}

makePayload final : (result : Payload)() = {
}

consumePayload(makePayload())
// the result is passed on as last, so the `& last` consumePayload is used
```

A function that wants its callers to receive a copy says so:

```zax
makePayload final : (result : Payload copy)() = {
}

consumePayload(makePayload())   // the result is passed on as copy: the loan
                                // consumePayload is used
```

A caller can also write a stance for one call, which replaces the result's:

```zax
consumePayload(makePayload() as copy)
```

A result with no stance never needs a stance written at the call, because it
is always passed on as `last`. A discarded result is destroyed like any
unused value.

Reference results are different. A returned reference only refers to a value
someone else holds, so a reference result with no stance is a loan, like any
other reference with no stance. Pointer results follow their own ownership
rules: a raw pointer result that allocates with `= @`, like `makeValue`'s in
[results and forwarding](#results-and-forwarding), hands the allocation to
its caller unless it declares `copy`. See
[Zax pointers, allocation, and arenas](pointers-and-arenas.md).

## Pointer ownership transitions

Pointers use the same stances. The destination's pointer type says what kind
of ownership you want to end up with:

```zax
shared : MyValue * strong = prepared as last
```

Here `prepared` must be a `MyValue * unique shareable`: a single owner whose
allocation was made so that it can later be shared. The transfer turns its
unique ownership into shared ownership and leaves `prepared` empty. Like any
variable after `as last`, `prepared` then counts as finished.

A raw pointer can become a `unique` owner only with `as last`. Without `last`,
the destination would only borrow the pointer, and a `unique` pointer cannot
borrow. When a raw pointer is declared with `= @`, the declaration, not the
pointer type, is responsible for freeing the allocation:

```zax
scheduled : MyValue * = @
// each line below is a separate alternative
owner1 : MyValue * unique = scheduled            // error: unique cannot borrow; write `as last`
owner2 : MyValue * unique = scheduled as last    // owner2 takes over; scheduled is empty
```

After the transfer, `scheduled` is empty. Its cleanup still runs at the end of
its scope, but finds nothing to free. `owner2` now owns the allocation. A raw
pointer declared without `= @` also needs `as last`, and in addition the
compiler must be able to prove that it is the only owner of the start of the
allocation; otherwise the transfer needs `unsafe`.

The same rule applies to parameters:

```zax
keepTarget final : ()(target : MyValue * unique) = {
}

borrowTarget final : ()(target : MyValue *) = {
}

borrowTarget(scheduled)         // borrows the address
// each line below is a separate alternative
keepTarget(scheduled)           // error at any use: write `as last`
keepTarget(scheduled as last)   // keepTarget owns the allocation; scheduled is vacant
```

A raw pointer declared with `= @` can also take over an allocation from a
`unique` pointer or from another such raw pointer:

```zax
scheduled = uniqueOwner as last
scheduled = otherScheduled as last
```

It first frees the allocation it already had. The source is left empty.

A raw pointer declared without `= @` cannot take over ownership, because
nothing would free the allocation:

```zax
raw : MyValue * = uniqueOwner as last
// error unless the compiler can prove another owner frees it, or the code uses unsafe
```

Going the other way, from shared to unique, works only sometimes:

```zax
owner : MyValue * unique = shared as last
```

It succeeds only when `shared` is the only strong pointer to the allocation and
no weak pointer remains. Otherwise `owner` is empty, and `shared` keeps its
ownership until it is destroyed.

Turning a weak pointer into a strong one is a `copy`:

```zax
owner : MyValue * strong = observer
```

This adds `owner` as another strong owner of the same allocation; it does not
copy the value pointed to. If the allocation is already gone, `owner` is
empty.

Making a raw pointer from a `loan` is an error outside `unsafe`, because a raw
pointer cannot carry the loan. An observer built from a loan reference is
also a loan, like everything else made from a loan.

Pointer kinds, shared ownership, arenas, and failures are explained in
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
writing that stance, like any other transfer. `bound strong` is separate from
the captures: it says how the lambda holds the object its function is called
on, here as another strong owner.

A loan cannot be captured as a copy, because copying from a loan is not
allowed. If the lambda captures it by reference instead, the lambda holds a
loan too:

```zax
view : Document readonly & = source        // unmarked: a loan
callback := [[ view ]] ()() {              // error if this captures a copy of view
  showDocument(view)
}
```

The spelling of a reference capture belongs to
[Zax lambdas and callable composition](lambdas-and-callable-composition.md).

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

An optional value accepts the ordinary stances, and also two forms written
before it, `last source` and `move source`:

```zax
source as last // the ordinary last stance on the optional value
last source    // optional-specific: take the contained value, then empty source
move source    // optional-specific: move from the contained value; source keeps it
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

Copying an array that owns its elements creates another array with its own
storage, and copies each element with that element's `copy`. `deep` does the
same, but every element must have a `deep` operation; it never falls back to
`copy`.

An array whose elements are stored inside it cannot hand its storage to another
array, so `move` and `last` move the elements one by one. An array can instead
get its storage from a separate storage provider, such as an arena, as in
`Integer[50] storage provider`. Such an array, even a fixed-length one, can
hand its whole block of storage to the new array, when the new array can use
storage from that provider. The new array then takes the elements without
moving them one by one.
[Storage providers](arrays-and-slices.md) are explained with arrays.

After `move`, the original must still be a usable, empty array. After `last`,
it only needs to be safe to destroy, so less has to be left behind.

Elements are governed by their type, not by a stance declared on the array
variable. An element type with a `transfer move` list makes every element
move-only wherever the array is stored.

Copying or moving a slice only copies or moves the slice's description of
which elements it shows; the elements themselves are not copied or moved. Only
the array that owns the storage can hand it over; `slice as last` cannot hand
over storage the slice does not own.

You can write stances on the arrays you join:

```zax
combined := left <+> right
reused := left as last <+> right   // the result may reuse left's storage
```

When the result's storage provider is not specified, a `copy` or `deep` join
may use the same provider and ask it for new storage. A `move` or `last` join
may take over `left`'s storage, leaving `left` as `move` or `last` normally
does.

The join operation decides whether storage can be reused, which elements are
copied or moved, and how large the result is. Arrays are explained in full in
[Zax arrays and slices](arrays-and-slices.md#copy-deep-move-and-last).

## Stances do not change qualifiers

A stance is not a qualifier, and `loan` is a reference's stance, not a
qualifier either. Qualifiers make three separate promises, and stances do not
change them:

| Axis | Meaning |
| --- | --- |
| `mutable` / `immutable` | `immutable` promises that the current value's contents do not change through ordinary safe behavior; `mutable` makes no such promise |
| `final` / `varying` | `final` promises that the place is never given a new value; `varying` allows a new value to replace the old one |
| `writable` / `readonly` | `readonly` promises that this access path performs no change; `writable` may perform an otherwise permitted change |

For example, this parameter:

```zax
drainBuffer final : ()(source : Buffer mutable readonly final & move) = {
}
```

describes:

- a mutable current `Buffer` value;
- readonly access through this path;
- a final referenced place; and
- permission for `drainBuffer` to move from the caller's `Buffer`, so a call
  must write `as move` or `as last`.

Because the access is `readonly`, `drainBuffer` can move from the `Buffer`
only in a way that does not change it, for example when the resource it takes
lives outside the value.

`readonly` does not cancel the other two promises. Each function declaration
describes one exact combination of qualifiers:

```zax
consume final : ()(
  source : Buffer mutable readonly final & copy
) = {
}

consume final : ()(
  source : Buffer mutable readonly varying & copy
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
  type supports;
- declaring one of those exact operations yourself, whether with a body or
  with `= default`, `= existing`, `= forbidden`, or no body, replaces only that
  one generated operation;
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

  +++ final : ()(source : MyFolder mutable writable & move) = default
  // file moves with MyOwnedFile's move; name moves with String's move
}
```

Without that declaration, a structure holding a move-only member cannot be
transferred: its generated copy is unavailable and no `move` exists.

If you write an operation meant to replace a generated one but get its
signature slightly wrong, both exist, and a call may become ambiguous. The
error message should show the exact generated signature you missed.

Generated operations, `= default`, assignment, replacement, and destruction
are explained in
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

Each type documents what its own operations cost.

## Diagnostics

Error messages should name the rule and the declaration that caused them. They
cover:

- an unstated stance where the use needs `move` or `last`: a type without
  `copy`, a by-value parameter requiring at least `move`, or a reference
  parameter that requires a stronger stance than the call wrote;
- a last use where `as last` would call another function or run another
  operation, counting stances the type's `transfer` list includes as written,
  including for each item of an expanded pack and at the end of a chain of
  `self` results;
- a last use inside a generic body, naming the generic body as the place to
  fix and the instantiating call;
- a transfer from a restricted local, alias, or member written with a weaker
  stance, or bound to a reference that could copy it;
- a declared stance the type's `transfer` list can never satisfy;
- a transfer, copy capture, or copy-capable reference from a `loan`, and a raw
  pointer formed from one outside `unsafe`;
- `loan` on a by-value parameter, local, or member;
- a reference or receiver passed on whole with a stronger stance than it was
  given, such as `input as last` from a `& move` reference;
- `_`, or a reference made from it, passed on with `move` or `last` without
  `intent<receiver-transfer>`;
- an unstated call that both a `loan` and a by-value function fit;
- unavailable `deep` behavior with no fallback;
- use of a finished source without `intent<terminal-source-reuse>`;
- a call that is ambiguous between a by-value and a reference overload, and
  any mismatch in qualifiers or stance, saying exactly which one;
- a variable or result used again after it was finished or passed on;
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

- A use of a variable with no stance is a `copy`, or a loan when it goes to a
  reference. The only other exception is inside a function whose `deep`
  reference or receiver makes `deep` the default, and that only makes copies
  more independent. The compiler never adds `move` or `last` to a variable:
  only unnamed values, which nothing can use again, are treated as `as last`:
  a value built directly in an expression, and a by-value result with no
  declared stance. A variable's declared stance is never
  applied to a use on your behalf.
- If a later compiler recognizes more last uses, it reports more errors; it
  never starts calling different functions.
- `as last` leaves the variable finished for every type, so your code stays
  correct if a type later adds a real `last` operation.
- A type's `transfer` list reserves stances: code that relies on a reserved
  `move` or `last` keeps compiling once the operation is written. Adding `move`
  to a type without a `transfer` list can make existing code that uses the
  type stop compiling until stances are written at its last uses.
- Adding another version of a function may make a call ambiguous, and the
  compiler reports that. The new version never wins because of where it was
  declared or imported.
- Results, including failure results, are passed on as `last` only when their
  declaration writes no stance; a declared stance is always used as written.
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

- how overloads are compared when stance, type, and qualifiers favor different
  candidates, and how generic specializations are compared;
- qualifier-generic syntax and constraints;
- detailed pointer provenance and casting beyond the current ownership model;
- which last uses analysis proves across loops and other control flow beyond
  straight-line code and branches;
- async suspension and cancellation;
- cross-thread value preparation and whether it is a stance, result capability,
  or separate contract;
- types whose mutable and immutable forms have different layouts;
- formal diagnostics and analysis contracts;
- complete general swap mechanics; and
- formal grammar, reflection, ABI, and implementation.
