
# nothing-maintainer-notes.md

This is refreshed thinking related to `Nothing` or more importantly the concept of "nothing" instances. This is meant to supersede the legacy material describing "nothing" or `Nothing`. That said, the current legacy material still has references and may have information and ideas suitable to mine.

## concern

`null` as a type is used in many languages for pointers to indicate "absence of value", and when accessed typically crash. When used in pointer math they can cause string pointer offsets that may or may not crash.

Not all CPUs will crash though. The ability to capture a problem is based on the CPU privilege of the code, and in kernel operations or on CPUs without privileges a `null` may be a valid access.

Zax approaching things differently.

## receiverless type functions

Type functions use `_` to indicate their "this" pointer, i.e. their own receiver. But not all functions on a type have receivers. `unbound` functions on a type do not contain a receiver, and thus `_` is inaccessible. They are basically global functions namespaced within a type.

But `once` functions are different. They have access to a `_` which allow access to the function directly via a type, or directly from a receiver.

```
MyType :: type {
  value : Integer

  myOnceFunc once final : ()() = {
    if ?_ {
      print("I have a receiver!")
      ++_.value
    } else {
      print("No receiver here!")
    }
  }

  myUnboundFunc final : ()() unbound = {
    // _.value is not accessible
  }

  myNormalFunc final : ()() = {
    ++_.value
  }

  myReplaceableFunc : ()() unbound

  myOtherReplaceable : ()() bound = {
    --_.value
  }
}

myType : MyType

myType.myOnceFunc()         // I have a receiver!
MyType.myOnceFunc()         // No receiver here!

myType.myUnboundFunc()      // error: only usable from the type
MyType.myUnboundFunc()      // allowed, no receiver present

myType.myNormalFunc()       // allowed, and receiver is bound
MyType.myNormalFunc()       // error: missing receiver

myType.myReplaceableFunc()  // panic!

myType.myReplaceableFunc = { ... }
myType.myReplaceableFunc()  // this is okay now

myType.myOtherReplaceable() // okay, --value

myType.myOtherReplaceable = myNormalFunc

myType.myOtherReplaceable() // okay, now ++value
```

These are the basic forms of functions, but this has implications.

### `varying` functions

The `final` functions know their value, and the function is explicitly defined. Calling them is always valid within their compiler allowed contexts, and they will not panic for lack of existence (they can of course panic for unrelated reasons).

But `myType.myReplaceableFunc` could panic if called. This is not a crash, but an explicit panic.

That means even though `myType.myReplaceableFunc` is not specified, it is NOT `null`. The default constructor sets up a panic function for `myType.myReplaceableFunc`, and this is not a `null` value. The panic version accepts the prototype but never returns because the panic effectively is a crash (and it's extremely difficult to imagine how this panic could be recovered). There is no stack unwind and there is no exception.

This is one of the reasons why Zax does not have `null`. `null` has an expectation that a CPU will catch and throw an exception. Zax doesn't promise the CPU will catch, and Zax doesn't have exceptions to throw and Zax doesn't have stack unwinding (at least in terms of exceptions, normal unwinding of course it supports).

### Nothing type instance

Zax has a concept of a nothing instance. When a `once` function is called on a type, the receiver can be absent. The `if ?_` can be used to determine if the receiver is present or not. That does not check for a `null` value. Instead that checks `if _ == <MyType-nothing-instance>`. That value is not directly exposed. It is a legitimate instance, but not a known fixed value like `null`.

Also, trivial types on a nothing instance can be read where `_` is "nothing" and `_.value` returns the default trivial initialized value of 0.

Unfortunately, nothing instances can't protect against writes, but it's a contract violation to write to a nothing instance, at least for the instances provided by the compiler (custom nothing instances described later can allow writing).

The compiler may add debug checking code to verify that writes don't occur but a normal runtime is not required to check for a nothing instance before a write.

This has implications though.

#### Overlapping trivial nothing instances

```
A :: type {
  foo : Integer
}

B :: type {
  bar : Float
}

C :: type {
  x : I64
  y : U64
  z : MyTrivialTypeContainer

  func final : ()() = { ... }

  sharedValue once : Integer
}
```

For `A`, `B` and `C`, those instances can all overlap to the same memory pointer assigned by the compiler. This the physical underlying pointer for nothing instances for `A._` and `B._` and `C._` may be the same. They are not required to be the same, but the compiler is allowed to share their storage.

Why? because the contract says that writing is not legal thus the trivial values can never present a bad value.

A `final` function, `bound` or `unbound` does not contribute storage, nor do `once` values. They are not factors for the nothing instances to consider.


#### Non-trivial types nothing instances

Consider:
```
MyType :: type {
  foo : Integer

  callbackOne : ()()
  callbackTwo : (result : MyValue)(input : InputType)

  callMeOne once final : ()() { _.callbackOne() }
  callMeTwo once final : (result : MyValue)(input : InputType) {
    return _.callbackTwo(input)
  }
}

MyOtherType :: type {
  var : Integer
  myArray : Integer[5]

  callbackThree : ()()

  callMeThree once final : ()() { _.callbackThree() }
  callMeFour once final : (value : Integer)() { return _.myArray[0] }
}

// ...

// all three of these must panic
MyType.callMeOne()
result1 := MyType.callMeTwo(myInputType)

MyOtherType.callMeThree()
```

In the above examples, the "nothing" instances must contain proper information according to the contract. The trivial values must be readable and return their trivial information where `0` values correctly cause a representation, or when accessed they must panic.

The means `callbackOne`, `callbackTwo`, `callMeThree` all must panic when called, and not crash because they are `null`.

For `callMeFour`, either that function will panic or return 0, depending if the array has a representative value when filled with trivial `0`s or it has no representation. That is a future decision for array detailed work because it's unclear if it's possible to be trivially initialized. It's actually highly unlikely to be trivial thus access is likely to panic.

The greater point is that the compiler cannot just trivially overlay those "nothing" instances for those types in the same memory space where trivially initialized values exist.

But the requirement is to panic with a "nothing accessed panic" category when access happens, so many "nothing" instances can overlap if the overlapped memory would all produce the same behavior. There is no requirement that input parameters, stack, or return results recover and panic in a nothing instance is fairly severe and almost certainly non recoverable. No effect is made in the Zax language to attempt to rescue this scenario.

The non-writable requirement for a "nothing" instance also means the compiler doesn't have to worry about a programmer setting one of those callbacks to a proper non-panic implementation. Thus the compiler is free to attempt to optimize type instance overlap without that consideration if it so chooses. There's no requirement it does attempt that but it is desirable.

#### Requesting a special nothing instance

The legacy material did a constructor hack involving a special `Nothing` type to signal that a particular type requires a nothing instance.

That doesn't seem like the right direction. Instead of special signal to the compiler is needed.

There are a few requirements:
- when requested, the compiler dedicates global storage for the nothing instance
- the nothing instance is initialized as order requires for instance creation
- the programmer can implement a special constructor to setup the nothing instance
- the programmer doesn't need to check inside every `+++` constructor if "this instance" is the nothing instance
- a nothing instance has no constructor arguments
- adding a nothing instance's constructor does not cause a constructor with no parameters to implicitly occur

Consider these approaches:

A.
```
MyType :: type nothing {
  +++ final : ()() = { ... }    // not called for the "nothing" instance

  // how does the "nothing" instance constructor get declared??
}
```

B.
```
MyType :: type {
  +++ final : ()() = { ... }    // not called for the "nothing" instance

  +++ final : ()() = { ... }    // how does this become the "nothing" constructor?
}
```

C.
```
MyType :: type {
  +++ final : ()(input : Integer) = { ... } // not called for the "nothing" instance

  +++ final : ()() = { ... }    // how does this not become a default initializer constructor, but a nothing constructor?
}
```

Proposed solution:

D.
```
MyType :: type {
  +++ final : ()(input : Integer) = { ... } // not called for the "nothing" instance

  +++ final once : ()() = { ... }    // `once` on a constructor doesn't normally make sense but does here
}
```

`once` can be the signal that this type requires a nothing instance and that function is the initializer of the "nothing" instance, and `_` is accessible just like it would be with a `once` type method.

#### Nothing instance special

When a nothing instance is request, that nothing instance has its own unique global storage unshared with any other instance. The contract requirement to not write to that instance is removed. No debug checks are performed to "check" if the nothing instance was accessed or written. Panics do not happen when accessing types as the contained types are real and concrete (unless, for example, a callback was unset and called on that nothing instance). This the nothing instance can act like a special shared instance.

#### Accessing the nothing instance type

A nothing instance can be accessed through the type instead of a receiver.

```
MyType :: type {
  value : Integer

  func once final : (result : Integer)() = { _.value }

  +++ once final : ()() = { ... }   // create a separate nothing instance
}

MyType.value            // error: receiver is required

myType : MyType

myType.value = 1        // allowed

value := MyType.func()  // accesses the nothing instance via the once function
```


## Nothing reset

```
MyType :: type {
  next : MyType *   // points to the "nothing" instance, not nothing
}

myType : MyType

myPointer : MyType * = myType   // take a pointer of the type

if ?myPointer {
  print("obviously valid pointer")
}

myPointer = : MyPointer *       // this does reset, ungraceful

reset myPointer                 // could be a phrase operator

reset myPointer.                // but if it was a phrase operator on the receiver, the intent is easily confused

myPointer = #                   // possible but a bit of a drift from intended discard meaning, might have implications for calling defaults

myPointer = null                // NO!!! teaches the wrong concept!

myPointer = MyType._            // not accessible, so no

myPointer = nothing             // maybe, but kind of feels like null
```

Something is needed, and of those, `#` is likely the best of the worst.


## Pointer checking

Pointers can be checked for nothing by `?pointer` or `!pointer` to check if it exists, or doesn't exist. These operators on pointers don't check for `null`. Instead they check for the nothing instance pointer for that type.


## casting

The `cast as` only performs safe casting operations, ones which the compiler recognizes as being safe. Thus a nothing instance cast will never cause a strange pointer to something very wrong.

The `unsafe cast as` can force a pointer conversion. Including the conversion of a nothing instance to a pointer to some other type. This is acceptable `unsafe` undefined behavior. No attempt to convert the pointer is made thus:

```
myFooInstancePtr : Foo * = myBarInstancePtr unsafe cast as Foo *
```

If `myBarInstancePtr` pointed to nothing, the cast doesn't yield a pointer to `myFooInstancePtr`'s nothing instance (unless they coincidentally share the same underlying memory location). The pointer will point to a `Foo *` with undefined behavior. It is acceptable if they by coincidence happen to point to the same nothing instance, as that too would fall into the "undefined behavior" bucket. Undefined can also mean "accidental success".

## Other considerations

I think the concept that a `Nothing` type should be retired. "nothing" as a concept exists, but not as a concrete type. Ideally we won't have a `nothing` equivalent to `null`.
