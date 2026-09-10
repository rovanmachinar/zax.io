
As input into the next package I'm going to provide some maintainer notes to include. These are be part of the `raw/` input captured in relation to `own` and composition.

The notes here are intended to supplement and supersede the legacy notes as this is the latest intended model compared to how the legacy materials describe composition.

---

# simple `own` variable name member sugar

## Basic sugar

Consider:
```
MyType :: type {
  foo : Integer
  bar : String
}

func final : ()() {
  myType own : MyType

  myType.foo = 1    // these two statements are identical in behavior
  foo = 1           // this "hides" and makes `myType.` invisible

  myType.bar = "hello"  // this is the same scenario -- they are the same
  bar = "hello"         // the implicit version of `myType.`
}
```

This is sugar.

## shadowable

### shadowing problem

This can cause shadowing though.

Consider:
```
MyType :: type {
  foo : Integer
  bar : String

  a : Float
}

func final : ()() {
  foo : String          // intentionally changed type
  bar : Integer

  myType own : MyType

  myType.foo = 1        // these two statements could be identical in behavior...
  foo = 1               // but, which foo? error or success?

  myType.bar = "hello"  // same problem here
  bar = "hello"         // which bar? error or success?

  a = 1.3               // `a` is clear and available
}
```

### shadowing solution 1

Does `shadowable` change things:
```
MyType :: type {
  foo : Integer
  bar : String

  a : Float
}

func final : ()() {
  foo shadowable : String   // adding shadowable
  bar : Integer

  myType own : MyType       // but neither `foo` nor `bar` are hidden

  myType.foo = 1            // these two statements could be identical in behavior...
  foo = "no abstract"       // but they are not because shadowable isn't overridden by `own`

  myType.bar = "hello"      // same problem here
  bar = 3                   // using original declared `bar`

  a = 1.3                   // this remains available
}
```

`shadowable` forward declares nested variables as being allowed to be hidden. It does not allow `own` to replace variables declared at the same scope.



### shadowing solution 2

What if it was `myType own` was declared first? Does that change things?

Consider:
```
MyType :: type {
  foo : Integer
  bar : String
}

func final : ()() {
  myType own shadowable : MyType    // `myType` is `shadowable` and but members `foo` and `bar` not `shadowable`

  myType.foo = 1                    // these two statements are identical in behavior...
  foo = 1                           // using the `myType.foo`

  myType.bar = "hello"              // these two statements are identical in behavior...
  bar = "hello"                     // using the `myType.bar`

  foo shadowable : String           // error: `myType.foo` occupy this scope's `foo` name (declaring `shadowable` here does not help)
  bar : Integer                     // error: `myType.bar` occupy this scope's `bar` name
}
```

`shadowable` works only in the nested forward direction. It gives permission to be shadowed after itself exactly one time only at a deeper flow depth. `own` only takes priority over values within the promoted container if the `own` was declared first. With `myType` being declared `shadowable` then the `myType` variable itself is `shadowable` by nested flow control values, but it would have no impact on the `foo` or `bar` contained.


### shadowing solution 3

What `shadowable` actually does...

Consider:
```
func final : ()() {
  foo shadowable : String
  bar : Integer

  while bar < 10 ;; ++bar {
    foo : Float = 0.3               // this is okay! `foo` was declared `shadowable`
    bar : String                    // error: `bar` was not declared `shadowable`
  }
}
```

`shadowable` works only in the nested forward direction. It gives permission to be shadowed after itself exactly one time only at a deeper flow depth. `own` only takes priority over values within the promoted container if the `own` was declared first. With `myType` being declared `shadowable` then the `myType` variable itself is `shadowable` by nested flow control values, but it would have no impact on the `foo` or `bar` contained.

## Conclusion

`own` on a variable injects the member declarations as if they existed in the "outer" container, whatever that container might be, a scope, or even a type. `own` will not abstract variables that exist within that "outer" container. `shadowable` only applies to forward nested values.


# simple but not simple type `own` sugar

## Simple case

Consider:
```
MyType :: type {
  foo : Integer
  bar : String
}

MyContainer :: type {
  myType own : MyType

  a : Integer
  b : String
}

func final : ()() = {
  myContainer own : MyContainer

  myContainer.myType.foo = 1
  myContainer.foo = 1               // identical to above statement
  myContainer.myType.bar = "hello"
  myContainer.bar = "hello"         // identical to above statement
  myContainer.a = 2
  myContainer.b = "goodbye"

  a = 1                             // because it maps to `myContainer.a` due to `myContainer ow`n`
  foo = 1                           // because it maps to `myContainer.myType.foo` due to `myContainer own` and `myType own`
}
```

This is `own` based composition and value name introduction. Nothing strange about this at all!


## `own` and priority within a type

Consider:
```
MyType :: type {
  foo : Integer
  bar : String

  a : String
  b : Float
  c : String
}

MyContainer :: type {
  myType own : MyType           // only `MyContainer.foo`, `MyContainer.bar`, `MyContainer.c` binds to `MyContainer.myType.*`

  a shadowable : Integer        // allowed (and priority)!
  b : String                    // allowed (and priority)!

  anotherType own : :: Type {
    a : String                  // allowed as `MyContainer.a` was declared `shadowable`, but does not bind to `MyContainer.a` as it's lower priority
    // b : Integer              // would cause error if uncommented: `MyContainer.b` was not declared `shadowable`
  }

  yetAnotherType own : MyType   // completely okay, but none of the values become bound to `MyContainer` container
                                // because of `myType` exists first, and because of direct `MyContainer.a` and `MyContainer.b` variables

  func1 : ()() = {
    _.foo = 1                   // because of `myType.foo`
    _.bar = "bagel"             // because of `myType.bar`

    _.a = 1                     // always mapped to `MyContainer.a`, as `MyContainer.myType.a` has less priority over `MyContainer.a`, `shadowable` doesn't change things
    _.b = "hello"               // always mapped to `MyContainer.b`
    _.c = "there"               // bound to `MyContainer.myType.c`
  }

  func2 : ()() = {
    a : String = "hello"        // allowed! The `_.` disambiguates;
    b : Integer = 0             // allowed! The `_.` disambiguates;

    _.foo = 1                   // because of `MyContainer.myType.foo`
    _.bar = "bagel"             // because of `MyContainer.myType.bar`

    _.a = 5                     // sees `MyContainer.a`
    _.b = "banana"              // sees `MyContainer.b`
    _.c = "coconut"             // sees `MyContainer.myType.c`
  }
}

func final : ()() = {
  myContainer : MyContainer

  myContainer.myType.a = "hello"            // this is normal `MyContainer.myType.a`
  myContainer.anotherType.a = "hello"       // this is normal `MyContainer.anotherType.a`
  myContainer.yetAnotherType.a = "hello"    // this is normal `MyContainer.yetAnotherType.a`
  myContainer.a = 1                         // the `MyContainer.a` is really `MyContainer.a`
  myContainer.b = "hello"                   // the `MyContainer.b` is really `MyContainer.c`
  myContainer.c = "goodbye"                 // the `MyContainer.c` is really `MyContainer.myType.c`
}
```

Ordering is important in functions, but not inside types. The reason is types are fully defined before functions are processed within the types or outside the types. Functions are processed in the order inside they are defined thus `own` can cause values to occupy a function's scope which doesn't happen in `type`s.

`shadowable` only affects anonymous inner type declarations. Functions are never ambiguous because `_.` disambiguates already. `shadowable` doesn't apply to `types`s declared outside a container `type`, and they are an "allow gate" for a variable to be used. They don't change a binding.



## `abstract` case

`abstract` forces a container to declare certain variables, and their types. These `abstract` values are not part of the `type` storage. They exist as abstract concepts but are not real declarations and they never contain a default value. If an `abstract` variable is not specified (anonymous) then any type can be used in the overridden container value. `abstract` only takes effect if the `own` is used by the container.

Consider:
```
MyType :: type {
  foo : Integer
  bar : String

  a abstract : String
  b abstract : Float
  c abstract : String
  d abstract : Integer
  e abstract :
}

MyOtherType :: type {
  text : String
  number : Integer

  a abstract : Integer
  b abstract :
}

MyContainer :: type {
  myType own : MyType
  myOtherType : MyOtherType         // `abstract` does not take effect

  a shadowable : String             // `shadowable` has no effect on `abstract` but the type matches so this is proper
  b : Float                         // good
  c : Integer                       // error: `c` from `myType.c` forces a `String` and `Integer` is not allowed
  d : String                        // error: `d` from `myType.d` forces an `Integer` and `String` is not allowed
                                    // error: where is `e` ???
}

MyOtherContainer :: type {
  myType own : MyType
  myOtherType own : MyOtherType     // both `abstract` take effect
                                    // error: immediate conflict between `myType.a` and `myOtherType.a` because the required types do not match

  a shadowable : String             // `shadowable` has no effect on `abstract` but the type matches so this is proper
  b : Float                         // good (`myType` forces `Float` but `myOtherType` doesn't specify)
  c : String                        // good
  d : Integer                       // good
  e : String                        // good - any type is allowed
}
```


# own position matters in type

## has a and upcast and downcast

As Zax is compositional and not inherited, Zax supports a "has a" relationships, and Zax does not support any "is a" relationships.

Consider:
```
A :: type {
  a : Integer
}

B :: type {
  b : Integer
}

C :: type {
  foo : String
}

MyManaged :: type managed {
  text : String
  number : Integer
}

D :: type {
  a own : A     // "has a" relationship, and `D.a` points to `D.a` (the `A` type) not `D.a.a`
  b own : B     // "has a" relationship, and `D.b` points to `D.b` (the `B` type) not `D.b.b`
  c own : C     // "has a" relationship, and `D.c` points to `D.c` (the `C` type), and `D.foo` points to `D.c.foo`

  bar : C       // "has a" relationship, and `D.bar` is a `C` type this `D.bar.foo` is the valid perspective (no `own` is declared)

  myManaged : MyManaged     // "has a" relationship, which has RTTI information
}

func final : ()() = {
  d : D

  // upcasting is completely safe... the compiler does pointer math to obtain the correct pointer to a contained member

  aPtr : A * = d.a      // take the address of value d.a
  bPtr : B * = d.b      // take the address of value d.b
  cPtr : C * = d.c      // take the address of value d.c
  barPtr : C * = d.bar  // take the address of value d.bar
  myManagedPtr : MyManaged * = d.myManaged   // take the address of value d.myManaged (RTTI is not involved / needed here)

  altA : A
  altB : B
  altC : C
  altMyManaged : MyManaged

  altAPtr : A * = altA      // take the address of `altA`
  altBPtr : B * = altB      // take the address of `altB`
  altCPtr : * = altC        // take the address of `altC` (using a singular * without the type when the type wasn't a pointer is sufficient, this is no different than `altAPtr`/`altBPtr`)
  altBarPtr : C * = altC    // take teh address of `altC` (again)
  myAltManagedPtr : MyManaged * = altMyManaged  // take the address of altMyManaged

  // downcasting is NOT safe (except when it can be made safe)
  doDowncast final : ()(
    a1 : A *,
    a2 : A *,
    b1 : B *,
    b2 : B *,
    c1 : C *,
    c2 : C *,
    bar1 : C *,
    bar2 : C *,
    inManaged1 : MyManaged *,
    inManaged2 : MyManaged *
  ) = {
    dA1Upcast = a1 unsafe outer cast D.a    // we happen know that `a1` comes from `D.a` so it's technically safe
                                            // but the compiler can't know this because its the caller that passed in the correct value
    dA1Upcast.a.a = 5                       // okay (but only because we KNEW it was okay)!

    dA2Upcast = a2 unsafe outer cast D.a    // we know this does NOT comes from `D.a`
                                            // but the compiler can't know this because its the caller that passed in the value
                                            // so the compiler assumes the programmer knows what they are doing because of `unsafe`
    dA2Upcast.a.a = 5                       // undefined behavior!

    dB1Upcast = b1 unsafe outer cast D.b    // we happen know that `b1` comes from `D.b` so it's technically safe but the compiler can't know this!
    dB1Upcast.a.a = 5                       // okay (but only because we KNEW it was okay)!

    dB2Upcast = b2 unsafe outer cast D.b    // we know this does NOT comes from `D.b` but the compiler can't know this!
    dB2Upcast.a.a = 5                       // undefined behavior!

    dC1Upcast = c1 unsafe outer cast D.c    // we happen know that `c1` comes from `D.c` so it's technically safe but the compiler can't know this!
    dC1Upcast.b.b = 6                       // okay (but only because we KNEW it was okay)!

    dC2Upcast = c2 unsafe outer cast D.c    // we know this does NOT comes from `D.c` but the compiler can't know this!
    dC2Upcast.b.b = 6                       // undefined behavior!

    dBar1Upcast = bar1 unsafe outer cast D.bar  // we happen know that `bar1` comes from `D.bar` so it's technically safe but the compiler can't know this!
    dBar1Upcast.b.b = 6                         // okay (but only because we KNEW it was okay)!

    dBar2Upcast = bar2 unsafe outer cast D.bar  // we know this does NOT comes from `D.bar` but the compiler can't know this!
    dBar2Upcast.b.b = 6                         // undefined behavior!

    dBar3Upcast = bar2 unsafe outer cast D.c    // we know this does NOT comes from `D.c` but the compiler can't know this and
                                                // the compiler will not error because `bar2` and `D.c` are both of `C` type
    dBar3Upcast.b.b = 6                         // undefined behavior!

    dManaged1Upcast = inManaged1 unsafe outer cast D.myManaged  // we happen know that `inManaged1` comes from `D.myManaged`
                                                                // so it's technically safe but the compiler can't know this!
                                                                // this is an acknowledgeable intent error (category needed) because there is a "safe" alternative
                                                                // but unsafe is faster because it's pointer math and not RTTI so it's still allow with explicit intent
    dManaged1Upcast.b.b = 6                                     // okay (but only because we KNEW it was okay)!

    dManaged2Upcast = inManaged2 unsafe outer cast D.myManaged  // we know this does NOT comes from `D.myManaged` but the compiler can't know this!
    dManaged2Upcast.b.b = 6                                     // undefined behavior!
                                                                // this is an intent error because there is a safe conversion but the programmer opted out of using it
                                                                // but the programmer can still decide to override and demonstrate intent to do this without RTTI

    dManaged3Upcast = inManaged1 outer cast D.myManaged         // we happen know that `inManaged1` comes from `D.myManaged`
                                                                // so it's technically safe but the compiler can't know this!
                                                                // the compiler will allow without `unsafe` because the compiler sees RTTI information
                                                                // it can use to make this conversion "safe"
    if ?dManaged3Upcast                                         // continue if not `Nothing`
      dManaged3Upcast.b.b = 6                                   // okay -- and the compiler made sure it was okay

    dManaged4Upcast = inManaged2 outer cast D.myManaged         // we happen know that `inManaged1` does NOT come from `D.myManaged`
                                                                // so it's not safe but the compiler can't know this!
                                                                // the compiler will allow without `unsafe` because the compiler sees RTTI information
                                                                // it can use to verify this conversion "safe"
    if ?dManaged4Upcast                                         // continue if not `Nothing`
      dManaged4Upcast.b.b = 6                                   // will NOT happen here because `Nothing` would be returned

    other = c1 unsafe outer cast D.a                            // error: `c1` cannot be a `D.a` type because it's a `C` type not an `A` type (compiler enforced)
  }

  doDowncast(
    aPtr,
    altAPtr,
    bPtr,
    altBPtr,
    cPtr,
    altCPtr,
    barPtr,
    altBarPtr
    myManagedPtr,
    myAltManagedPtr
  )
}

```

The compiler will perform pointer math to obtain pointers to individual members. But the compiler cannot know if a particular pointer to a type that happens to also be in a container actually comes from a container. So converting from a contained value to a container is inherently not safe, except the `managed` keyword can be used to add extra overhead "RTTI (Run Time Type Information)" so that it can make these safe conversions possible. RTTI conversions require extra storage, and "safe" RTTI conversions are slower than straight `unsafe` math conversions because extra "is this actually the correct conversion" checks happen.


## final functions and containers

functions within a type declared `final` and `own` do NOT automatically become part of a container type. They require no instance storage so they do not become contained within the `own`. This is also true of `once` declared values as they do not occupy instance storage (as they become effectively part of global storage).

`final` functions cannot be declared `abstract`. They require an immediate definition. Non-final functions can be declared `abstract` and they also occupy no instance space but they signal to the container with `own` of that type that a function must be defined. A container of an `own` container containing an `own` type with `abstract` will not be forced to implement the `abstract` in the contain of the container. `abstract` is only forced to the immediate surrounding container and only if `own` is declared.

Functions with `copy` or `&` style results and parameters based on their own type do not change their type magically just because they are declared `own` within a container. This is important.

Consider:
```
A :: type {
  a : Integer

  funcARefRef final : (result : A &)(input : A &) = { ... }
  funcACopyRef final : (result : A)(input : A &) = { ... }
  funcARefCopy final : (result : A &)(input : A) = { ... }
  funcACopyCopy final : (result : A)(input : A) = { ... }
}

B :: type {
  b : Integer

  funcBRefRef final : (result : B &)(input : B &) = { ... }
  funcBCopyRef final : (result : B)(input : B &) = { ... }
  funcBRefCopy final : (result : B &)(input : B) = { ... }
  funcBCopyCopy final : (result : B)(input : B) = { ... }
}

C :: type {
  a own : A         // "has a" relationship (`a` is promoted to `C.a` but no `final` function is promoted from `A`)
  b own : B         // "has a" relationship (`b` is promoted to `C.b` but no `final` function is promoted from `A`)

  altA own : A      // "has a" relationship (`own` does not promote any `A` values because of `C.a` and `own` exists first)
  altB : B          // "has a" relationship (`own` does not promote any `B` values because it was not declared as `own`)

  myAAARefRef final : (result self : A &)(input : A &) = C.a.funcARefRef        // use the `C.a` version
  myAAAAltRefRef final : (result self : A &)(input : A &) = C.altA.funcARefRef   // use the `C.altA` version

  myBBBRefRef final : (result self : B &)(input : B &) = C.b.funcBRefRef        // use the `C.b` version
  myBBBAltRefRef final : (result self : B &)(input : B &) = C.altB.funcBRefRef   // use the `C.altB` version


  // intent is required because these are `downcast` scenarios, and this is still potentially very `unsafe`
  // but this acknowledges that the `&` results are NOT from a different `A` but in-fact are related to containerized `A`;
  // the programmer does not require `unsafe` acknowledgement as this is a fairly common scenario where the compiler
  // can examine the return results and then cause an `unsafe` static analysis plus `unsafe` acknowledgement
  // if the downcast looks suspicious
  intent<container-downcast-return-results>{
    // legal -- prototype is intent compatible - use the `C.a` version, and return is auto downcast as a `C.a` -> `C`
    myACARefRef final : (result self : C &)(input : A &) = C.a.funcARefRef

    // legal -- prototype is intent compatible - use the `C.altA` version, and return is auto downcast as a `C.altA` -> `C`
    myACAAltRefRef final : (result self : C &)(input : A &) = C.altA.funcARefRef
  }

  myAACRefRef final : (result self : A &)(input : C &) = C.a.funcARefRef            // error -- prototype is not compatible - use `C.a` as input or `C.altA` as input?
  myAACAltRefRef final : (result self : A &)(input : C &) = C.altA.funcARefRef      // error -- prototype is not compatible - use `C.a` as input or `C.altA` as input?

  // while the `A` can be `intent` downcast to `C` the input is still ambiguous and `intent` cannot repair the ambiguity
  intent<container-downcast-return-results>{
    myACCRefRef final : (result self : C &)(input : C &) = C.a.funcARefRef          // error -- prototype is not compatible - use `C.a` as input or `C.altA` as input?
    myACCAltRefRef final : (result self : C &)(input : C &) = C.altA.funcARefRef    // error -- prototype is not compatible - use `C.a` as input or `C.altA` as input?
  }



  intent<container-downcast-return-result>{
    // legal -- prototype is intent compatible - use the `C.b` version, and return is auto downcast as a `C.b` -> `C`
    myBCBRefRef final : (result self : C &)(input : B &) = C.b.funcBRefRef

    // legal -- prototype is intent compatible - use the `C.altB` version, and return is auto downcast as a `C.altB` -> `C`
    myBCBAltRefRef final : (result self : C &)(input : B &) = C.altB.funcBRefRef
  }

  myBBCRefRef final : (result self : B &)(input : C &) = C.a.funcARefRef        // okay -- prototype is compatible - use `C.b` as input as `C.b` is `own` (and thus selected)
  myBBCAltRefRef final : (result self : B &)(input : C &) = C.altA.funcARefRef  // okay -- prototype is compatible - use `C.b` as input as `C.b` is `own` (and thus selected)

  // combine and convert `B` -> `C` automatically, return result requires `intent` for downcast
  intent<container-downcast-return-result>{
    // legal -- prototype is intent compatible - use the `C.b` version, and return is auto downcast as a `C.b` -> `C`
    myBCCRefRef final : (result self : C &)(input : B &) = C.b.funcBRefRef          // okay -- use `C.b` as input as `C.b` is `own` (and thus selected)

    // legal -- prototype is intent compatible - use the `C.altB` version, and return is auto downcast as a `C.altB` -> `C`
    myBCCAltRefRef final : (result self : C &)(input : B &) = C.altB.funcBRefRef    // okay -- use `C.b` as input as `C.b` is `own` (and thus selected)
  }
}

myFuncRefRef final : ()() = {
  myC1 : C
  myC2 : C

  myA : A
  myB : B

  // myAAARefRef / myAAAAltRefRef

  { result : A & = myC1.myAAARefRef(myC2.a) }       // okay - underlying prototype is C.a.funcARefRef
  { result : A & = myC1.myAAARefRef(myC2.altA) }    // okay - underlying prototype is C.a.funcARefRef
  { result : A & = myC1.myAAARefRef(myA) }          // okay - underlying prototype is C.a.funcARefRef
  { result : A & = myC1.myAAAAltRefRef(myC2.a) }    // okay - underlying prototype is C.altA.funcARefRef
  { result : A & = myC1.myAAAAltRefRef(myC2.altA) } // okay - underlying prototype is C.altA.funcARefRef
  { result : A & = myC1.myAAAAltRefRef(myA) }       // okay - underlying prototype is C.altA.funcARefRef

  { result : A & = myC1.myAAARefRef(myC2) }         // error: underlying prototype is C.a.funcARefRef but should `myC2.a` be used or `myC2.altA` -- it's ambiguous
  { result : A & = myC1.myAAAAltRefRef(myC2) }      // error: underlying prototype is C.altA.funcARefRef but should `myC2.a` be used or `myC2.altA` -- it's ambiguous


  // myBBBRefRef / myBBBAltRefRef

  { result : B & = myC1.myBBBRefRef(myC2.b) }       // okay - underlying prototype is C.b.funcBRefRef
  { result : B & = myC1.myBBBRefRef(myC2.altB) }    // okay - underlying prototype is C.b.funcBRefRef
  { result : B & = myC1.myBBBRefRef(myB) }          // okay - underlying prototype is C.b.funcBRefRef
  { result : B & = myC1.myBBBAltRefRef(myC2.b) }    // okay - underlying prototype is C.altB.funcBRefRef
  { result : B & = myC1.myBBBAltRefRef(myC2.altB) } // okay - underlying prototype is C.altB.funcBRefRef
  { result : B & = myC1.myBBBAltRefRef(myB) }       // okay - underlying prototype is C.altB.funcBRefRef

  { result : B & = myC1.myBBBRefRef(myC2) }         // okay - underlying prototype is C.b.funcBRefRef and `myC2.b` should be used because `C.b` is marked `own`
  { result : B & = myC1.myBBBAltRefRef(myC2) }      // okay - underlying prototype is C.altB.funcBRefRef and `myC2.b` should be used because `C.b` is marked `own`


  // myACARefRef / myACAAltRefRef

  { result : C & = myC1.myACARefRef(myC2.a) }       // okay - underlying prototype is C.a.funcARefRef
  { result : C & = myC1.myACARefRef(myC2.altA) }    // okay - underlying prototype is C.a.funcARefRef
  { result : C & = myC1.myACARefRef(myA) }          // okay - underlying prototype is C.a.funcARefRef
  { result : C & = myC1.myACAAltRefRef(myC2.a) }    // okay - underlying prototype is C.altA.funcARefRef
  { result : C & = myC1.myACAAltRefRef(myC2.altA) } // okay - underlying prototype is C.altA.funcARefRef
  { result : C & = myC1.myACAAltRefRef(myA) }       // okay - underlying prototype is C.altA.funcARefRef

  { result : C & = myC1.myACARefRef(myC2) }         // error: underlying prototype is C.a.funcARefRef but should `myC2.a` be used or `myC2.altA` -- it's ambiguous
  { result : C & = myC1.myACAAltRefRef(myC2) }      // error: underlying prototype is C.altA.funcARefRef but should `myC2.a` be used or `myC2.altA` -- it's ambiguous


  // myBCBRefRef / myBCBAltRefRef

  { result : C & = myC1.myBCBRefRef(myC2.b) }       // okay - underlying prototype is C.b.funcBRefRef
  { result : C & = myC1.myBCBRefRef(myC2.altB) }    // okay - underlying prototype is C.b.funcBRefRef
  { result : C & = myC1.myBCBRefRef(myB) }          // okay - underlying prototype is C.b.funcBRefRef
  { result : C & = myC1.myBCBAltRefRef(myC2.b) }    // okay - underlying prototype is C.altB.funcBRefRef
  { result : C & = myC1.myBCBAltRefRef(myC2.altB) } // okay - underlying prototype is C.altB.funcBRefRef
  { result : C & = myC1.myBCBAltRefRef(myB) }       // okay - underlying prototype is C.altB.funcBRefRef

  { result : C & = myC1.myBCBRefRef(myC2) }         // okay - underlying prototype is C.b.funcBRefRef and `myC2.b` should be used because `C.b` is marked `own`
  { result : C & = myC1.myBCBAltRefRef(myC2) }      // okay - underlying prototype is C.altB.funcBRefRef and `myC2.b` should be used because `C.b` is marked `own`


  // myAACRefRef / myAACAltRefRef -- all error definitions so none of these are real but if they had have been...

  { result : A & = myC1.myAACRefRef(myC2.a) }       // error - expecting `C` as input
  { result : A & = myC1.myAACRefRef(myC2.altA) }    // error - expecting `C` as input
  { result : A & = myC1.myAACRefRef(myA) }          // error - expecting `C` as input
  { result : A & = myC1.myAACAltRefRef(myC2.a) }    // error - expecting `C` as input
  { result : A & = myC1.myAACAltRefRef(myC2.altA) } // error - expecting `C` as input
  { result : A & = myC1.myAACAltRefRef(myA) }       // error - expecting `C` as input

  { result : A & = myC1.myAACRefRef(myC2) }         // would have been okay, and underlying prototype would have been C.a.funcARefRef but still error because function is undefined
  { result : A & = myC1.myAACAltRefRef(myC2) }      // would have been okay, and underlying prototype would have been C.altA.funcARefRef but still error because function is undefined



  // myACCRefRef / myACCAltRefRef -- all error definitions so none of these are real but if they had have been...

  { result : C & = myC1.myACCRefRef(myC2.a) }       // error - expecting `C` as input
  { result : C & = myC1.myACCRefRef(myC2.altA) }    // error - expecting `C` as input
  { result : C & = myC1.myACCRefRef(myA) }          // error - expecting `C` as input
  { result : C & = myC1.myACCAltRefRef(myC2.a) }    // error - expecting `C` as input
  { result : C & = myC1.myACCAltRefRef(myC2.altA) } // error - expecting `C` as input
  { result : C & = myC1.myACCAltRefRef(myA) }       // error - expecting `C` as input

  { result : C & = myC1.myACCRefRef(myC2) }         // would have been okay, and underlying prototype would have been C.a.funcARefRef but still error because function is undefined
  { result : C & = myC1.myACCAltRefRef(myC2) }      // would have been okay, and underlying prototype would have been C.altA.funcARefRef but still error because function is undefined


  // myBBCRefRef / myBBCAltRefRef

  { result : B & = myC1.myBBCRefRef(myC2.b) }       // error - expecting `C` as input
  { result : B & = myC1.myBBCRefRef(myC2.altB) }    // error - expecting `C` as input
  { result : B & = myC1.myBBCRefRef(myB) }          // error - expecting `C` as input
  { result : B & = myC1.myBBCAltRefRef(myC2.b) }    // error - expecting `C` as input
  { result : B & = myC1.myBBCAltRefRef(myC2.altB) } // error - expecting `C` as input
  { result : B & = myC1.myBBCAltRefRef(myB) }       // error - expecting `C` as input

  { result : B & = myC1.myBBCRefRef(myC2) }         // okay - underlying prototype is C.b.funcBRefRef and `myC2.b` should be used because `C.b` is marked `own`
  { result : B & = myC1.myBBCAltRefRef(myC2) }      // okay - underlying prototype is C.altB.funcBRefRef and `myC2.b` should be used because `C.b` is marked `own`


  // myBCCRefRef / myBCCAltRefRef

  { result : C & = myC1.myBCCRefRef(myC2.b) }       // error - expecting `C` as input
  { result : C & = myC1.myBCCRefRef(myC2.altB) }    // error - expecting `C` as input
  { result : C & = myC1.myBCCRefRef(myB) }          // error - expecting `C` as input
  { result : C & = myC1.myBCCAltRefRef(myC2.b) }    // error - expecting `C` as input
  { result : C & = myC1.myBCCAltRefRef(myC2.altB) } // error - expecting `C` as input
  { result : C & = myC1.myBCCAltRefRef(myB) }       // error - expecting `C` as input

  { result : C & = myC1.myBCCRefRef(myC2) }         // okay - underlying prototype is C.b.funcBRefRef and `myC2.b` should be used because `C.b` is marked `own`
  { result : C & = myC1.myBCCAltRefRef(myC2) }      // okay - underlying prototype is C.altB.funcBRefRef and `myC2.b` should be used because `C.b` is marked `own`

}
```

`own` in a type serves to:
- promote members into a container (if container hasn't defined them)
- indicates intent to implement members containing `abstract`
- disambiguates if a single `own` will select one clear type value winner over another for selection

All the `copy` versions of return results cannot be auto downcast via `intent<container-downcast-return-results>{ ... }` as only references can be casted. `copy`s are inherently "sliced" from any container when returned thus obtaining access to an outer container is not possible.

However, input `copy` arguments can be compatible with downcast values passed into upcast `copy` prototypes. That's because the `copy` can slice out the value from the container. Ambiguity can still exist as to which value to slice, but that can be aided via `own` selection of a member value within a container.

An upcast value can never be passed as an argument to a downcast as a `copy` value because the value would not contain the full downcast set of values (and thus is inherently incompatible).

### reference `unsafe outer cast` and `outer cast`

On a pointer type, `unsafe outer cast` returns a calculated pointer from a pointer assuming that pointer is a member of the outer container. Concrete instances that use `unsafe outer cast` return by references to container types.

```
MyFooType :: type {
    a : Integer
}

MyBarType :: type {
    foo : MyFooType
    b : String

    makeMeABar once final : (result : BarType last)() = { return : MyBarType }
}

Bar bar1;
foo1 : & = bar1.foo

bar2 : & = foo1 unsafe outer cast MyBarType.foo

bar3 : & = ((MyBarType.makeMeABar()).foo) unsafe outer cast     // error: NO -- reference to a temporary inside an expression
```

### reference `unsafe outer cast` and `outer cast`

Similar to reference `unsafe outer cast`, `outer cast` return an optional reference to an concrete type if the RTTI indicates that conversion to the outer type is indeed possible.

```
MyFooType :: type managed {
    a : Integer
}

MyBarType :: type {
    foo : MyFooType
    b : String

    makeMeABar once final : (result : BarType last)() = { return : MyBarType }
}

Bar bar1;
foo1 : & = bar1.foo

bar2 : & ? = foo1 outer cast MyBarType.foo
if ?bar2
  use(bar2.)

bar3 : & = ((MyBarType.makeMeABar()).foo) outer cast     // error: NO -- optional reference to a temporary inside an expression
```
