
# lambda-maintainer-notes.md

This raw notes documentation is meant to superseded what exists in the legacy documentation. If information conflicts with legacy documentation, then assume this version is the likely correct version. The old legacy material is maintained

## General concept

Lambdas are functions that can capture values and present those captured values as accessible to the lambda function as if they were an ordinary input parameters. Once a lambda is captured they exist as if they were normal callable functions, except they are different and those differences matter.

Consider:
```
foo : Integer = 42
bar : String = "apples"

// capture happens here
myLambda = [[foo, bar]] ()() {
  print(foo, bar)
}

print("You have:")
myLambda()

foo = 7
bar = "bananas"

myLambda()
```

In this example, the `You have:` is displayed first then the `42` `apples`. Changing `foo` to `7` and `bar` to `bananas` does not affect the follow up call to `myLambda()` as the lambda has captured a copy of the original values. The lambda doesn't just create a pointer or reference to the original values (although that's possible to do). The variables `foo` and `bar` are reintroduced into `myLambda` but these are not the same variables as the global `foo` and `bar`. They only look like they are the same.

## Capture variable mapping

### simple mapping by labels

Consider:
```
foo : Integer = 42
bar : String = "apples"

// capture happens here
myLambda = [[foo: myFoo:, bar: myBar:]] ()() {
  print(myFoo, myBar)
}
```

Variable mapping works like function invocation and return results. The `source: destination:` labels can remap any input to an output.

### mapping and introducing types

This approach can also be used to capture other ways:

Consider:
```
foo : Integer = 42
bar : String = "apples"

// capture happens here
myLambda = [[foo: myFoo : MyType, bar: myBar : &]] ()() {
  print(myFoo, myBar)
}

myLambda()

foo = 7
bar = "bananas"

myLambda()
```

In the above example, `foo` is now represented by `myFoo` which is of type `MyType`, which means it accepted the Integer `foo` as a constructor and created a `MyType` from it. `bar` was captured as a reference in `myBar`. Which means, the second call to `myLambda` will show `bananas` whereas the `MyType` will retain its state from when it constructed and displayed `myFoo`.


### mapping and introducing re-used type names

Consider:
```
foo : Integer = 42
bar : String = "apples"

// capture happens here
myLambda = [[foo: myFoo:, bar: : &]] ()() {
  print(myFoo, bar)
}

myLambda()

foo = 7
bar = "bananas"

myLambda()
```

In the above scenario, the `bar` replacement name was left blank, and thus the compiler knows the programmer doesn't intend to rename bar inside the lambda. It still applies the `&` to `bar`, and thus `bananas` will get displayed in the second call, whereas `myFoo` will retain `7`.


### mapping and calling a function

Consider:
```
myLambda = [[foo: myFoo:, bar: = produceFooBar()]] ()() {
  print(myFoo, bar)
}

myLambda()
```

The compiler can map a function call result into the lambda directly using the same mapping approach where instead of taking the values from the surrounding scope, the results are supplied from the function call(s). More than one function can be called, and a mixture of invoked functions and surrounding scope values can be used.

The recommendation is that surrounding scope captured variables come after any function call result to avoid confusing the compiler as to which variables belong to what result and which variables are intended to be from surrounding scope. Function call results can also be done positionally using positional cursors, but the names that are introduced into the lambda either come from the original function or are directly defined as part of the capture declaration.

Consider:
```
myLambda = [[: myFoo:,: : = produceFooBar()]] ()() {
  print(myFoo, bar) // assuming `produceFooBar : (foo : Integer, bar : String)()` was the prototype
}

myLambda()
```

The `: :` might look strange, but this is the same way a function selection happens by position `:` and then reuses the name in `:`; (verify this is the correct way that function invocation works)

If two names end up the same from multiple function calls, then an error is issued.

## lambda internals

The compiler documents how lambda work because they have two properties that force implementation details to become known:
- the compiler creates an anonymous `type` from each unique capture set
- the compiler need storage to keep track of the captured data
- the compiler needs storage to use lambdas for use as functional callbacks
- the compiler needs to know how to perform transfer stances against these captures

### storage in a callable

Consider:
```
MyType :: type {
  value : Integer

  reportValueCallback : ()(value : Integer)

  reportValue final : ()() = {
    if ?reportValueCallback
      _.reportValueCallback(value)
  }
}

myType : MyType

myType.value = 7

fruit := "apples"
presentation := "sliced"

myType.reportValueCallback = [[fruit: myFruit:, presentation:]] ()(value : Integer) {      // error: cannot bind to myType.reportValue
  print(value, myFruit, presentation)
}

myType.reportValue()    // indirectly call myType.reportValueCallback()
```

The issue is that `reportValueCallback` only has room for a single pointer to a callback function. That function is `unbound` and doesn't have a receiver.

### callable with a bound receiver

If the callback type was `reportValueCallback : ()(value : Integer) bound` then a function with a receiver could be captured.

Consider:
```
MyType :: type {
  value : Integer

  reportValueCallback : ()(value : Integer) bound

  reportValue final : ()() = {
    if ?reportValueCallback
      _.reportValueCallback(value)
  }
}

MyCallbackType :: type {
  fruit : String
  presentation : String

  myCallbackFunc : ()(value : Integer) = {
    print(value, fruit, presentation)
  }
}

myType : MyType
myCallbackType : MyCallbackType

myType.value = 7

myCallbackType.fruit := "apples"
myCallbackType.presentation := "sliced"

myType.reportValueCallback = myCallbackType.myCallbackFunc      // okay: bound to the of myCallbackType and the function myCallbackFunc

myType.reportValue()    // indirectly call myType.reportValueCallback()
```

This works, but the issue is that the binding happens to a reference to the callable receiver and the receiver's function. The `bound` here reserves room for the receiver but it's insufficient to work for a lambda. The issue is that the lambda needs storage to survive so while using a lambda in a `bound` function is legal, only a reference to that lambda is taken. If the lambda falls out of scope with the reference still `bound` then calling into the lambda will have undefined behavior. The compiler will perform static analysis and error when it's 100% proven the lambda will destroy before the end of the captured reference lifetime. If the compiler is uncertain, it will require an `unsafe<possible-lambda-capture-beyond-lifetime>{ ... }` around the point where the lambda is assigned to the `bound` callable. That new category of `unsafe` needs to be captured into the unsafe catalog.

### callable keeping lambda alive

The question becomes - how does a lambda survive beyond the lifetime of the callable?

The answer is that the lambda needs to be allocated within an arena, consider:
```
MyType :: type {
  value : Integer

  aCallback : ()(value : Integer) bound unique
  bCallback : ()(value : Integer) bound strong
  cCallback : ()(value : Integer) bound strong atomic
  dCallback : ()(value : Integer) bound weak                // weak disallows results in the prototype

  reportValue final : ()() = {
    if ?aCallback
      _.aCallback(value)
    if ?bCallback
      _.bCallback(value)
    if ?cCallback
      _.cCallback(value)
    if ?dCallback               // checks if weak was assigned, does not check if weak will succeed, does not promote to strong, not truly needed, but doesn't hurt to check to avoid function call "setup"
      _.dCallback(value)        // will not actually call if temporary promotion to strong fails
  }

  clearCallbacks final : ()() = {
    reset aCallback             // resets to default state
    reset bCallback
    reset cCallback
    reset dCallback
  }
}

MyCallbackType :: type {
  fruit : String
  presentation : String

  myCallbackFunc : ()(value : Integer) = {
    print(value, fruit, presentation)
  }
}

myType : MyType
myCallbackType : MyCallbackType

myType.value = 7

myCallbackType.fruit := "apples"
myCallbackType.presentation := "sliced"

// a `unique` callable doesn't require the additional storage be used to house a `unique`, it makes room for it
// but the callback can still be bound regardless if the `unique` storage is used
myType.aCallback = myCallbackType.myCallbackFunc    // okay: bound to the of myCallbackType and done via a reference, no `unique` is captured

// create a strong, use the default arena
myType.bCallback = [[foo:, bar: = produceFooBar()]] ()(value : Integer) strong {
  print(value, foo, bar)
}

// create a strong, use a custom arena
myType.cCallback = [[foo:, bar: = produceFooBar()]] ()(value : Integer) strong atomic @{ myArena } {
  print(value, foo, bar)
}

// create a strong, use the default arena, keep track of it in a value, as storing directly into `weak` would cause the lifetime to destroy immediately
storeMyStrongCallback : ()(value : Integer) strong = [[foo:, bar: = produceFooBar()]] ()(value : Integer) strong {
  print(value, foo, bar)
}

myType.dCallback = storeMyStrongCallback    // weak pointer is bound to the `dCallback` while strong pointer is maintained by storeMyStrongCallback


myType.reportValue()    // all callbacks will be called

reset storeMyStrongCallback

myType.reportValue()    // all, except `dCallback` will be called

myType.clearCallbacks()

myType.reportValue()    // no callbacks will be called
```

### callable keeping lambda alive and type erasure with destruction

Lambdas don't have a known type as the compiler generates a `type` based on the captured values, which becomes an anonymous compiler type. The only way to reference a lambda is to capture into a `bound` callable, or allocate and track it into a `bound` and `unique` / `strong` / `strong atomic` / `weak` lifetime (where `weak` will not keep the lambda alive).

The reason why `bound` is insufficient is because there are three things that must be maintained (at additional storage cost):
- the function to call
- the receiver to call (i.e. the lambda or a bound type callable)
- the lifetime to maintain

A callable can maintain one, two or all three. At minimum, the function must be set with an `unbound` callable, or the function and the receiver must be called with a `bound` function. The additional lifetime is maintained with `unique` / `strong` / `strong atomic` / `weak`.

The callable doesn't know the actual underlying type involved. Yet, it's able to keep the type alive, and destroy the type after without knowing the type. This is because extra storage doesn't just track the lifetime, it tracks the destructor it needs to destroy the type.

Thus a callable may store
- pointer to function
- pointer to receiver when a `bound` with a `bound` `type` function, or a lambda is used
- storage for a generic `unique` / `strong` / `strong atomic` / `weak` when required
- storage for the destructor pointer when `unique` / `strong` / `strong atomic` / `weak` required

Thus these callables are fairly "fat" types. They incur real cost.

When a lambda is created it:
- creates a function pointer
- creates a receiver (where the captures are stored)
- optionally creates a `unique` / `strong` / `strong atomic` / `weak`
- creates a destructor so the types within the lambda can be destroyed

The lambda does not create a `copy` / `move` / `last` version of itself, although it supports those stances. This is because lambdas don't store their data in the callable, they store the data outside the callable so the storage of the data never moves location when a callable is assigned or move. Only the pointers transfer in the callable, not the data within; it's those pointers that respect the `copy` / `move` / `last` stance, and `deep` is not supported.

### capturing a receiver into a lambda

Consider:
```
MyType :: type {
  value : Integer

  myFunc final : ()() = {
    print(_.value)
  }
}

myType :: MyType

myUnbound : ()() unbound = MyType.myFunc    // compatible prototype? nope! error: missing the receiver
myBound : ()() bound =

```

## function composition

### function composition lambda

Lambdas support a special version to create function composition:
```

myFuncWith2Params final : ()(foo : Integer, bar : String) unbound = {
    print(foo, bar)
}

value : Integer = 42

myCompositionStorage := [[value: foo:]] ()(myBar : String) bound unique >> myFuncWith2Params   // map `value` into `myFuncWith2Params`'s `foo` parameter

myCompositionStorage("apples")  // 42 apples

callable : ()(myBar : String) bound unique = myCompositionStorage as last

callable("bananas")             // 42 bananas
```

This replaces the legacy version of function composition. The lambda definition `myCompositionStorage` binds to the `myFuncWith2Params` using an `unbound` version of the function. Had `myFuncWith2Params` been defined `bound unique`, then `myFuncWith2Params as last` would have been required. The `>>` signifies to bind to an existing callable, adopt the storage for the callable, and optionally bind one or more values as arguments to the callable. The `>>` can bind to `unbound`, `bound`, or containing storage with `bound unique` / `bound strong` / `bound strong atomic` / `bound weak` (although `weak` requires that no return results be present).


### keeping a receiver type instance alive with function composition

Consider:
```
MyType :: type {
  //... data ...

  func final : ()() = {
    //... function bound to `_`
  }
}

myType : MyType * strong = @

myType.func()   // works!

myCallableRef : ()() bound = myType.func   // yes, this binds BUT it does not keep the `myType` alive because it's only a reference


myCompositionStorage := [[myType: #]] ()() bound unique >> myType.func

callable : ()() bound unique = myCompositionStorage as last

callable()  // calls myType.func() and keeps myType.func() alive
```

In this version `callable` keeps `myType` alive because it captures the `myType` strong pointer. The `myType: #` is done so that `myType` is not introduced as a variable argument into the binding of `myType.func`. Without it, it would attempt to match the captured variable into the `myType.func` and bind it which would error because `myType.func` does not accept that binding. Using `myCompositionStorage` with `binding unique` allows the `myCompositionStorage` to maintain a longer lifetime because of its `unique` storage.

The function composition form of a lambda is able to capture and store `bound`, with `unique` / `strong` / `strong atomic` / `weak`. The type is not necessary to specify for the `>>` composition as the compiler knows what to choose based on the type passed into the `>>` argument. Thus the `bound` type on the definition of `myCompositionStorage` apply to itself, and have nothing to do with the `>>` definition. They can be entirely independently set.

Finally the `#` doesn't indicate "don't capture this" in a lambda for right hand side label mapping. It says "I don't want this mapped". On the left-side of a return result of a label mapping, it is a recognized discard operation saying "I don't want this captured".

## other concerns

### legacy definitions

Legacy defines `: ()() *` -- that would be a pointer to an `unbound` callable, there's no direct `: ()() *` definition of a callable.

Likewise `: ()() bound unique *` -- that would be a pointer to a callable with a `bound unique` type; not a direct definition of a callable.

The legacy material likely needs review for correctness in the new model as it likely (incorrectly) assumes `: ()() *` is a "thing", when it's not.


### spacing

We must decide which of these is the better "legal" approach:
```
[[amILegal]] ()() { ... }       // not as clearer to read, but stronger intent different from arrays
[[ amILegal ]] ()() { ... }     // clearer to read but arrays require spacing spacing too `[ [ 1, 2 ] ]` which is a `[1][2]` defined array
```

I prefer the extra space but I worry about array intent. We do have `[[` and `]]` showing intent, but is it enough? I'd like another opinion.

### confusion with compiler directive

Legacy compiler directives use `[[directive]]`. Those need a sweeping clean up to `[<directive>]` without accidentally changing lambdas. This is a more mechanical concern mostly but it has to be done with knowledge not to mix lambda and directives changes by accident.
