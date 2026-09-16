
# [Zax Programming Language](index.md)

## Casting

Current qualifier-preservation, ordinary promise-strengthening, and explicit
unsafe-weakening constraints are defined by
[Zax qualifiers](language/qualifiers.md). The complete cast catalog and the
remaining nonstructural conversion proposals on this page remain legacy input.

> **Disposition.** `as` and `unsafe as` are now current open language-defined
> [operator phrases](language/operator-phrases.md) whose exact forms are in the
> [operator catalog](language/operator-catalog.md#conversion-and-admission-forms). Their left
> value supplies receiver discovery and their right operand is a complete
> [type argument](language/function-invocation.md#type-parameter-slots) rather
> than a discarded value parameter.
>
> Safe structural recasting, compatibility postures, source anchors, coercive
> layout conversion, and view-shaped `unsafe cast` are now current in
> [Zax structural shapes and compatibility](language/structural-shapes-and-compatibility.md).
> Current string and character identities, literal-time validation, and
> compile-time directional joins are owned by
> [Zax strings and characters](language/strings-and-characters.md). Runtime
> string admission, conversion, and concatenation remain indexed future input.
> Nothing else on this page is current design except where a current owner is
> linked.
>
> Current `as copy`, `as deep`, `as move`, and `as last` source restatement is
> defined by [Zax transfer stances](language/transfer-stances.md). Those forms do
> not perform a cast or transfer by themselves.
>
> **Integer conversion disposition.** Fundamental integer conversion is now
> current conceptual design in
> [Zax integers](language/integers.md#integer-identity-types).
> Guaranteed `as`, optional `as Destination?`, `narrowing as`, identity
> admission, and projection replace the legacy claim that ordinary `as` performs
> every potentially panicking intrinsic conversion and `unsafe as` performs
> ordinary truncation.
>
> **Fixed and floating conversion disposition.** Current exact, optional,
> rounded, representation, and coercive boundaries are owned by
> [fixed-point scalars](language/fixed-point-scalars.md),
> [binary floating-point scalars](language/floating-point-scalars.md), and
> [endianness](language/endianness.md). General non-numeric casting remains
> future work.
>
> Custom `as` declarations use a type parameter slot directly. Unprefixed
> literals use double quotes, and every single-quoted literal payload carries its
> own attached prefix.
>
> **Enum conversion evidence.** The enum side of `as` and `unsafe as` is
> now belongs to [Zax enums](language/enums.md#generated-boundary-operations).
> `enumValue as UnderlyingType` projects to the exact immediate backing type.
> Direct `as` to another integer type is rejected; the programmer projects to
> the backing type and then uses ordinary integer conversion. The legacy
> `enumValue unsafe as OtherInteger` shortcut is superseded. Raw backing
> adoption uses `EnumType unsafe from rawValue`.

### String-conversion disposition

The legacy conversion examples formerly here mixed byte strings, ASCII,
validated UTF, native wide strings, code pages, panicking casts, and lossy
conversion without stable identities or failure contracts.

Their useful compile-time behavior is now taught by
[Zax strings and characters](language/strings-and-characters.md), including
explicit left-owned `<+>` admission and transcoding. Their runtime validation,
optional/reporting/lossy conversion, allocation, mutation, and foreign-view
pressure is preserved as indexed future string-runtime and encoding input.
The superseded examples are removed rather than retained as competing teaching.


### Pointer casting using `unsafe as`

Any type can be converted from one pointer type to another pointer type using the `unsafe as` operator. A compiler will not perform any type checking on pointer `type` conversions to check if they are compatible. If a pointer `type` is cast to an incompatible pointer `type` and accessed then undefined behaviors can result.

An `Unknown *` can be used to hold a generic pointer to anything by casting with an `unsafe as` operator.

````zax
MyType :: type {
    // ...
}

// create an instance of `MyType`
myType : MyType

// take a pointer to the type using implicit pointer cast
myTypePointerOriginal : MyType * = myType

// using `unsafe as` convert to an `Unknown *`
unknown := myTypePointerOriginal unsafe as Unknown *

// using `unsafe as` convert from an `Unknown *`
myTypePointerCopy := unknown unsafe as MyType *

// the original pointer and the copied pointer match identically
assert(myTypePointerOriginal == myTypePointerCopy)
````


### Casting a by-value type into a pointer

When a type's instance is cast as a pointer, an address of the instance is taken. However, in many cases manual casing to a pointer type is unnecessary. In Zax, a value type will automatically convert to a pointer `type` implicitly for the same `type` without any conversion being required. For greater explicitness, the `as` operator can convert from a value to a pointer to the same type safely without introducing undefined behaviors.

A dangling pointer is when a pointer to a type's instance is maintained past the lifetime of a type's instance. Keeping a copy of a pointer is not safe. A pointer should be used within the scope of obtaining a pointer and no longer. Copying pointers across asynchronous functions is not safe. The underlying memory could be disposed prior to attempting to access the pointer to a type's instance.

The `unsafe as` operator will forcefully convert any value type into a pointer of any other type. This type of conversion is not recommended as it can lead to undefined behaviors.

Examples of pointer casting:

````zax
MyType :: type {
    value1 : Integer
    value2 : String
}

AnotherType :: type {
    value1 : Float
    value2 : WideString
}

func final : ()(input : MyType *) = {
    // ...
}

myType : MyType
anotherType : AnotherType

myTypePointer1 := myType as MyType *    // allowed
myTypePointer2 := myType as *           // allowed - type is deduced
myTypePointer2 : MyType * = myType      // allowed - implicit casting

func(myType)                            // allowed - implicit casting


// ERROR: cannot convert from myType to AnotherType * as the types do not match
myOtherTypePointer := myType as AnotherType *

// UNDEFINED BEHAVIOR: casting a pointer to one type into a pointer of another
// can lead to undefined behaviors if the pointers are accessed
myOtherTypePointer := myType unsafe as AnotherType *

// ERROR: cannot convert from `AnotherType` to `MyType *`
func(anotherType)

// ERROR: cannot convert from `AnotherType *` to `MyType *`
func(anotherType unsafe as AnotherType *)

// UNDEFINED BEHAVIOR: casting a pointer to one type into a pointer of another
// can lead to undefined behaviors if the pointers are accessed
func(anotherType unsafe as MyType *)
````

Example of a dangling pointer:

````zax
MyType :: type {
    value1 : Integer
    value2 : String
}

func : (result : MyType *)() = {
    myType : MyType

    // WARNING: `dangling-reference-or-pointer` is found which will cause
    // undefined behaviors if the pointer is accessed
    return myType
}

// hold onto the pointer beyond the lifetime of the instance where it points
danglingPointer := func()

// undefined behavior since the pointer points to memory for a `MyType` instance
// that is already disposed
danglingPointer.value1 = 5
danglingPointer.value2 = "hello"
````


### Casting a pointer to a by-reference or by-value type

A pointer cannot be implicitly converted back to a by-reference type (or to a by-value type). A compiler does not allow this kind of casting to occur because a pointer may point to nothing and a programmer should check for `Nothing` (or decide they don't need to check). If a pointer is not checked if that the pointer is pointing to a valid type's instance then a panic may occur at runtime if the pointer actually points to `Nothing`.

An `as` operator can be used as one method to convert a pointer to a by-value or by-reference type, or alternatively the dot (`.`) operator can convert a pointer to a by-reference type.

The `unsafe as` operator will forcefully convert any pointer type into a value reference of any other type but this is not recommended as it can lead to undefined behaviors.

Examples of pointer casting:

````zax
MyType :: type {
    value1 : Integer
    value2 : String
}

AnotherType :: type {
    value1 : Float
    value2 : WideString
}

funcByValue final : ()(input : MyType) = {
    // ...
}

funcByRef final : ()(input : MyType &) = {
    // ...
}

myType : MyType
anotherType : AnotherType

myTypePointer := myType as MyType *     // allowed

myTypeRef1 := myType as MyType &        // allowed - implicit casting
myTypeRef2 := myType as &               // allowed - deduced reference type
myTypeRef3 : MyType & = myType          // allowed - implicit casting 
myTypeRef4 : & = myType                 // allowed - implicit casting with
                                        // deduced reference type
funcByValue(myType)                     // allowed - copy
funcByRef(myType)                       // allowed


myTypeRef5 := myTypePointer as MyType & // allowed - implicit casting but would
                                        // runtime panic if myTypePointer was
                                        // pointing to `Nothing`
myTypeRef6 := myTypePointer as &        // allowed - deduced reference type but
                                        // would runtime panic if myTypePointer
                                        // was pointing to `Nothing`

funcByValue(myTypePointer as MyType &)  // allowed - copy but would runtime
                                        // panic if myTypePointer was pointing
                                        // to `Nothing`
funcByRef(myTypePointer as MyType &)    // allowed - implicit casting but would
                                        // runtime panic if myTypePointer was
                                        // pointing to `Nothing`


// ERROR: cannot implicitly convert from a pointer type to a reference type
myTypeRef7 : MyType & = myTypePointer 
myTypeRef8 : & = myTypePointer

// ERROR: cannot implicitly convert from a pointer type to a by-value type or
// a reference type
funcByValue(myTypePointer)
funcByRef(myTypePointer)


if myTypePointer {
    // safe because the pointer was checked if it points to something
    // valid and this code executes and the conversion is performed
    checkedType := myTypePointerToNothing as MyType &
}

// may runtime panic as `myTypePointer` was not checked if it is valid
// (although in this context it most certainly is a valid pointer)
myTypeRefA := myTypePointer. as MyType &// allowed - already a reference
myTypeRefB := myTypePointer. as &       // allowed - already a reference

funcByValue(myTypePointer. as MyType &) // allowed - copy
funcByRef(myTypePointer. as &)          // allowed - already a reference

myTypeRefC : MyType & = myTypePointer.  // allowed - already a reference
myTypeRefD : & = myTypePointer.         // allowed - already a reference
myTypeRefE := myTypePointer.            // allowed - copy with
                                        // deduced type

funcByValue(myTypePointer.)             // allowed - copy
funcByRef(myTypePointer.)               // allowed - already a reference


myTypeCopy1 := myType as MyType         // allowed - copy 
myTypeCopy2 : MyType = myType           // allowed - copy 
myTypeCopy3 := myType                   // allowed - copy with deduced type 

funcByValue(myType as MyType)           // allowed - copy of a copy
funcByRef(myType as MyType)             // allowed - reference to a copy

funcByValue(myType)                     // allowed - copy
funcByRef(myType)                       // allowed


myTypeCopy4 := myTypePointer as MyType  // allowed - implicit copy casting 

// ERROR: cannot implicitly convert from a pointer type to a value copy
myTypeCopy4 : MyType = myTypePointer


funcByValue(myTypePointer as MyType)    // allowed - copy of a copy
funcByRef(myTypePointer as MyType)      // allowed - reference of a copy

// ERROR: cannot implicitly convert from a pointer type to a value copy
funcByValue(myTypePointer)
// ERROR: cannot implicitly convert from a pointer type to a reference
funcByRef(myTypePointer)


// accessing `myTypePointer` may runtime panic if it points to `Nothing`
myTypeCopy5 := myTypePointer. as MyType // allowed - copy 
myTypeCopy6 : MyType = myTypePointer.   // allowed - copy 

funcByValue(myTypePointer. as MyType)   // allowed - copy of a copy
funcByRef(myTypePointer. as MyType)     // allowed - reference to a copy

funcByValue(myTypePointer.)             // allowed - copy
funcByRef(myTypePointer.)               // allowed reference to type



myTypePointerToNothing : MyType *       // points to nothing

if myTypePointerToNothing {
    // safe because the pointer was checked if it points to something
    // valid (this code will not execute)
    checkedType := myTypePointerToNothing as MyType &
}

myTypePanic1 := myTypePointerToNothing as MyType &  // PANIC AT RUNTIME
myTypePanic2 := myTypePointerToNothing as &         // PANIC AT RUNTIME
myTypePanic3 := myTypePointerToNothing as MyType    // PANIC AT RUNTIME

funcByValue(myTypePointerToNothing as MyType &)     // PANIC AT RUNTIME
funcByRef(myTypePointerToNothing as MyType &)       // PANIC AT RUNTIME

funcByValue(myTypePointerToNothing as &)            // PANIC AT RUNTIME
funcByRef(myTypePointerToNothing as &)              // PANIC AT RUNTIME

funcByValue(myTypePointerToNothing as MyType)       // PANIC AT RUNTIME
funcByRef(myTypePointerToNothing as MyType)         // PANIC AT RUNTIME

myTypePanic4 := myTypePointerToNothing. as MyType & // PANIC AT RUNTIME
myTypePanic5 := myTypePointerToNothing. as &        // PANIC AT RUNTIME
myTypePanic6 := myTypePointerToNothing. as MyType   // PANIC AT RUNTIME


// ERROR: cannot implicitly convert from a pointer type to a reference type
myTypePanicA : MyType & = myTypePointerToNothing
myTypePanicB : & = myTypePointerToNothing
myTypePanicC : MyType = myTypePointerToNothing


// ERROR: cannot implicitly convert from a pointer type to a reference type
funcByValue(myTypePointerToNothing)
funcByRef(myTypePointerToNothing)


myTypePanicD : MyType & = myTypePointerToNothing.   // PANIC AT RUNTIME
myTypePanicE : & = myTypePointerToNothing.          // PANIC AT RUNTIME
myTypePanicF : MyType = myTypePointerToNothing.     // PANIC AT RUNTIME

funcByValue(myTypePointerToNothing.)                // PANIC AT RUNTIME
funcByRef(myTypePointerToNothing.)                  // PANIC AT RUNTIME
````


### Structural shape and layout conversion

The former structural `as` and `unsafe as` proposal on this page has been
consumed and superseded.

Current programmer-facing behavior is defined by
[Zax structural shapes and compatibility](language/structural-shapes-and-compatibility.md):

- distinct identity remains separate from direct and flattened shape;
- safe low-overhead recasting uses explicit compatibility postures, optional
  source anchors, and protected `as shape`/`as layout` forms;
- noncontiguous mapping and reordering use `-<>-` and `reshape`;
- checked-layout semantic coercion uses local safe or unsafe coercive `as`
  according to representation validity and writable-invariant risk; and
- unchecked reinterpretation uses view-shaped `unsafe cast`.

The old positional-prefix matching, private-member bypass, broad structural
`unsafe as`, and unqualified by-value raw cast rules no longer remain as
candidate alternatives here.


### `as` operator overloading

The exact phrase form `as` is current design; see the
[operator catalog](language/operator-catalog.md#conversion-and-admission-forms). Protected
conversion domains, result selection, and generated/disabled behavior must still
be reconsidered under the current [operator model](language/operators.md) and
future casting work.

A conversion declares a
[type parameter slot](language/declarations-and-bindings.md#type-parameter-slots-and-type-arguments).
A type argument has no runtime storage, lifetime, or evaluation, so no discarded
runtime parameter is required.

Types can implement an `as` operator to support custom conversion from one type to another. This type of conversion can only be done by-value as by-reference would not be logical (as a new instance is needed for an unrelated destination type to exist as a reference).

````zax
IncompatibleType :: forward type

MyType :: type {
    category final once : String

    age : Integer
    name : String
    height : Float

    operator binary 'as' final : (
        result : DestinationType
    )(
        DestinationType : type
    ) readonly = {
        result.name = name
        return result
    }
}

IncompatibleType :: type {
    name : String
    height : Float
}

myType : MyType

incompatibleType1 := myType as IncompatibleType

// ERROR: no `as` operator that can convert to the destination type by reference
byRefValue1 := myType as IncompatibleType &
````


#### Disable `as` operators

Legacy material proposes explicitly enabling generated conversions, disabling
one conversion with a bodyless final declaration, and disabling an otherwise
open family through a catch-all meta declaration. The exact constraint,
generated/default, forbidden, and reference-result syntax has not been reviewed
against the current operator and type-parameter model.

Future casting work must decide which of those controls survive and then provide
current examples. This page does not retain obsolete discarded-value parameter
forms as candidate syntax.


### Casting as `default`

> `as default` is now a current
> [reserved phrase form](language/operator-catalog.md#reserved-phrase-forms) with
> two shapes: a default-qualified concrete type identity, and a value compatible
> with the expression's default type. Its complete qualifier-default, transfer,
> generic, and type-result behavior remains future work.

The remaining transfer question is whether value-shaped `as default` resets an
explicit declaration or use-site stance to implicit `copy`:

```zax
source : MyType move
defaulted := source as default
```

This source is unresolved evidence, not an accepted transfer-reset contract.
Future `as default` work must reconcile it with current declaration stance,
one-use `as <stance>` restatement, qualifier defaults, generic metadata, and
result typing. It must not revive the superseded `shallow`/`lease` equivalence or
silently perform transfer.
