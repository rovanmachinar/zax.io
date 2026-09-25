
This file represents my updated thinking on the `partial` question and how to solve some big complexities it introduces to limit the pandora's box it opens if not carefully done. The legacy partial should be considered non canonical with this partial attempting to replace how it's considered. The legacy material remains in case it offers any good insights or learnings to be applied but it is not authoritative over the concept or this document.

# partial explained

Partial reopens a non sealed type and allows additional variable declaration stored within a type, additional functions (varying or final), and other sub types / enums / etc.

One of the biggest "wins" is the advantage would be being able to augment a receiver that that cannot be expanded to support other types, for example
```
a : MyType
b : Integer

myResult1 := a + b      // okay, MyType can be extended to support an `Integer` with a `+` operator with a rhs of `Integer`
MyResult2 := b + a      // the `Integer` receiver is a built-in type, not possible to add an operator `+` which a rhs of `MyType`
```

By declaring a partial, additional functions can be added:
```
MyPartialInteger :: partial Integer {
  operator binary '+' final : (result : ResultType)(rhs : MyType) = {
    //...
  }
}
```

By creating a partial, this allows operators that act as if they were declared directly on the original `type`. In the example above, by creating a partial for `Integer`, the `+` operator could be added to support `MyType` as a rhs to the `Integer` receiver.

## additional design pressures

The `project/raw/partial-types.md` file contains some design pressures, but I wanted to further illustrate the important.

Beyond just adding a few "helper" functions to make using external receivers more naturally, there's an absolute hard need for partial support related to the `___` context instance.

Every active thread, and async path following that thread (details on that point are TBD so don't go down that rabbit hole for now) receives a `___` instance. That instance will contain various build in defaults, like an arena for dynamic allocation. However, the context is intentionally not sealed. It can add not only additional functions but additional variable can be stored in the instance.

That allows a programmer to add their own information to each content instance they can then access, including any per-thread context storage.

## The complex details

There are some complexities though to both adding non storage variables/types to an existing type, but even more complexity with variables

### Conflicting names

As a partial can add named operations, if any other partial adds a function name then ambiguities arise. There are mitigations but that doesn't imply that ambiguities can't exist.

```
MyPartialA :: partial MyType {
  func final : ()() = { ... }
}

MyPartialB :: partial MyType {
  func final : ()() = { ... }
}

myType : MyType

myType.func()       // which func is called?? ambiguity error
```

### Conflicting names breaks existing functions

During compilation, a function on an existing library might call a `myType.foobar()` function; which during compilation was known and thus a linkage to that function was completed because it's clearly known what functions exist. Later a partial type can become described and what was linked is now ambiguous.

That creates stress on the compiler. The compiler has no way to know "should I wait to link what I believe is complete now, on the off chance some partial comes along later?" That seems unreasonable. The "obvious" solution might seem to be to declare a forward on the type, but the trouble is that can't be done for some types, especially the built-in scalar types.


### Addition variable storage

Operations like `size of` cannot complete while a `type` remains unsealed, but that's not the only problem.


### Hooking into constructions / assignment / other operators

Constructor, destructors, `operator binary '='`, `+++ replacement`, all expect to setup and complete their operations on a `type`. The moment a partial is defined that adds additional variable to be stored, those variables need to live with the existing variables. That implies they need to be constructed, or destructed, copied, factored into assignment, play a role during replacement. The trouble is that an existing type cannot know what doesn't yet exist. Even if the compiler delays finalizing a result for the sake of `size of`, there's no way for the compiler to guess how construction could happen. Without some kind of facility, the compiler at best could only "default" apply operations, and given that operators can become domain based overloads, that is not an appropriate solution.


### Order concerns

As partials can become known as they are compiled throughout modules the order they exist can become layered and expanded over time. If ordering matters then this can become an issues.


### External modules

An unsealed `type` can be expanded across modules. Keeping one module from seeing another's data is likely a good policy, unless exported too; for example additions to the `___` context should be module specific.


## Possible solutions

### `forward` and visibility of partial

Zax compilation order is known. Normally order shouldn't matter, but I would suggest that partial is one of those cases that order does matter. A type should be able to connect to what it knows at the time it compiles. Thus if a partial introduced later than a type declares without an explicit `forward` on the partial should exclude the partial from consideration. Once a `forward` on a partial is resolve, the partial is considered. Once a partial is defined it becomes part of consideration for future compilation against the type.

If this is coherent will partially be based on the way `forward` works. If forward only applies to the current "nesting", or "deeper nesting" like this:
```
namespace A {
  Module.C.D.MyType :: forward type

  namespace B {

    myTypeA : MyType    // okay
  }

  myTypeB : MyType
}

namespace A {
  namespace B {
    myTypeC : MyType    // error: localized nesting did not contain the forward
  }
}
```

I think that policy can work. I don't think `forward` is defined yet in such a way that makes that explicitly an error or not, but being an error would align with my intent. Basically, unknown partials would not be considered if they were not already defined or explicitly forwarded within view of the current nesting.

Thus:
```
namespace A {
  Module.E.F.MyPartialType :: forward partial Module.A.B.MyType

  namespace B {
    myTypeA : MyType

    myTypeA.myPartialDefinedFunc()  // okay: Module.E.F.MyPartialType is now "visible" here
  }

  myTypeB : MyType

  myTypeB.myPartialDefinedFunc()    // Module.E.F.MyPartialType is now "visible" here
}

namespace A {
  namespace B {
    myTypeC : MyType

    myTypeC.myPartialDefinedFunc()  // error: Module.E.F.MyPartialType is not "visible" here
  }
}
```

Why is this better? Because it limits the "search" scope to exact areas that a compiler will see so it doesn't endlessly wait for more future versions, and it doesn't override what an existing version will see. It does allow some override if you inject a partial into an imported namespace, but then again, you asked for that to happen in that scenario. It also narrows the chances of ambiguity where there shouldn't have been any based on the code that was written. It limits the splash damage from partials, and it allows libraries to intentionally "hide" their extensions added into types.

Without this visibility mechanism, it's very likely we'll have to create `forward` on individual functions within receivers and `operator` definitions within receivers thus I think this warrants strong consideration for adoption.


### `seal` and partial

I think it's important a type be able to seal itself in multiple ways:
```
MyType :: type seal storage callable once {
  // ...
}

MyPartialA :: partial MyType {
  foo : Integer                 // error: sealed to adding new type storage because of `seal storage`
}

MyPartialB :: partial MyType {
  func final : ()() = { ... }   // error: sealed to adding new functions because of `seal callable`
}

MyPartialC :: partial MyType {
  bar once : MyOtherType        // error: sealed to adding new `once` because of `seal once`
}
```

Conceptually a simple `seal` declaration will be everything but a `seal` with one category will seal only that aspect.

### `abstract` and hooks

The concern about hooking into construction, destruction, and assignment and other areas I think can be handled with `abstract`

Consider:
```
MyType :: forward type

MyHookContract :: type {
  construct abstract : ()(state : MyType &)
  destruct abstract : ()()
  assign abstract : ()(state : MyType readonly &)
  oneMoreHook abstract : ()(value : Integer)
}

MyType :: type {
  hooks partial : MyHookContract

  // ... other values here ...

  +++ final : ()() = {
    // ... setup internal state ...

    _.hooks.construct(_)
  }

  --- final : ()() = {
    _.hooks.destruct()
  }

  operator binary '=' : (result self : MyType &)(rhs : MyType readonly &) = {
    _.a = rhs.a
    //... more stuff ...

    _.hooks.assign(_)

    return _
  }

  func : ()() = {
    //... do stuff ...
    _.hooks.oneMoreHook(5)
  }
}

MyPartialType :: partial MyType {
  hookContract own : MyHookContract

  construct private final : ()(state : MyType &) = { ... }
  destruct private final : ()()
  assign private final : ()(state : MyType readonly &)
  oneMoreHook private final : ()(value : Integer)
}
```

The idea is that a partial can only become hooked into an existing type by explicit calls which the compiler will create linkages between partials based on their presence and implementation of the hooks. If no partials exist, these stubs are compiled out as if they were never called. If they are "hooked" each hook of a partial is called BUT the order, by contract, is not important. There is a defined ordering (likely based on partial introduction order), but the order should never be relied upon by implementations by policy.

Other rules are:
- no hook can return a result
- the contract needs to be implemented on the partial explicitly for a hook to be called, accidental matches do not count
- the hook implementation may be marked `private` since it's mean to be auto-hooked by the compiler to the partial implementation

### `size of` problem

The size of operation will have to be delayed until a finalized "sealing" phase of the compilation. I suspect we may want to allow a type to become sealed by intent earlier than "the end of all files compiled". Otherwise compile time functions that use `size of` will also need to be delayed. This could become a real problem if compile time functions are waiting on `size of` to complete where they go onto define other `partial` types based on the result of `size of`. It's very likely we'll have to deny `partial` on types we must seal to complete a compile time operation. `size of` operation should be highly discouraged, and subject to restrictions.

## Context

The `___` doesn't (yet) have a `type`. This needs a `type` name within a `namespace` because it's a system type, and to attach `partial` requires a type, not an instance. I welcome input on what to call it and a `namespace`. We can define a name but add design pressure for later on the `namespace` because we do so for other types like `OpaqueObserver` and related family.


# Other considerations

Better Syntax and spellings suggestions are very much welcomed. These are only the suggested starting points.
