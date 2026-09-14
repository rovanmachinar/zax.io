# Maintainer notes: structural shapes and compatibility

| Field | Value |
| --- | --- |
| Status | Raw maintainer-preparation notes / non-authoritative |
| Audience | The Zax language maintainer and the agent assigned work item `021` |
| Applies To | Questions, examples, and design ideas to prepare before structural-shape review begins |
| Owns | The maintainer's evolving input for work item `021` |
| Does Not Own | Accepted structural semantics, the fixed initiating input, or current language behavior |

## Purpose

These notes prepare the maintainer's intended primary input for work item `021`.
The language maintainer may expand, revise, or reorganize them before assigning
that work in a new session.

The examples and questions below establish a starting point. They do not imply
that any answer is already accepted.

The information contained is meant to be a refresh on the legacy documentation and more representative of newer thinking on these subjects. However, there might be some thoughts in the legacy materials that still apply or contain good insights, and there might be concerns or considerations not handled in this newer thinking.

Finally, I've coined some phrases, operators and keywords, but that doesn't mean I'm am married to them. Suggested alternatives be considered if a reviewer finds a concern with the current selected name (or merely one that would be better fitting or more standard).

## Maintainer's context

Zax has strict type requirements. If a `Car` instance is expected as a parameter then exactly that must be supplied, not `LooksLikeACar` instance.

This is good for detecting problems so that a type shape that looks-like it might be compatible (e.g. has all the same names inside the shape, but it has a different type name) wouldn't be accidentally passed into a function.

That said there are times that requiring an exact shape is actually more of a hindrance than a help. For example, something that calculates vectors coordinates might not care if the structure was called `Vector` or `MyVector` or whatever, so long as it was "compatible". Likewise there are times when a programmer just wants to define a simple anonymous input type inline to the function call and doesn't want to have to lookup the name of the struct, the namespace, and then include it all just to pass a `Point`.

This is where "compatible" comes in... If a structure can be defined intentionally as compatible or to be compatible, the programmer can use duck-like typing where appropriate. This is up programmer to decide when to use it, they just need rules and syntax to make it happen.

That said, there are constraints. Zax is a compile language, and that means that a function expecting a type shape isn't just wanting to have variables in random places. Variables have storage placement. `x` followed by `y` in memory is not the same as `y` followed by `x`. The physical layout must match or the callee can't work.

Likewise things like alignment matter. A `Boolean` followed by a `Integer` could be tightly packed or contain padding. If they aren't the same, they can't function.

There said `x` vs `myX` and `y` vs `myY` despite having different names, might be a trivial difference and unworthy of an error.

That's what this work is to determine. What can be compatible, and how can it be made compatible.

## Constructor as an argument

Consider:
```zax
Point :: type {
  x : I32
  y : I32
}

func final : ()(point : Point &) = { ... }

func([{ .x = 4, .y = 5 }])                    // implicit construction
func(point: : Point = [{  .x = 4, .y = 5 }])  // same as above, anonymous name with explicit type and with more decoration
```

This should be perfectly legal that a constructor is passed in as an argument. In this particular case a temporary of `Point` could be constructed as given the `last` stance. Nested constructors should be allowed.

However, this is not "compatibility". This is creating an actual `Point` because the function being called takes a `Point` and the compiler can recognize that the constructor arguments can implicitly create a temporary and the intent to do so is clear.

## Type Decomposition, Recomposition, Transformation and Reshaping

### Type decomposing via `>-` operator

Consider:
```zax
Point :: type {
  myX : I32
  myY : I32
}

getXY final : (resultX : I32, resultY : I32)() { ... }
func final : ()(x : I32, y : I32) = { ... }
getPoint final : (result : Point)() = { ... }

func(resultX: x:, resultY: y: = getXY())        // map results from getXY() into func's inputs
func(myX: x:, myY: y: = getPoint())             // error: getPoint returns `Point` not `myX` or `myY`

myPoint := getPoint()

func(myX: x:, myY: y: = myPoint)                // error: myPoint is a single value, it can't be mapped

// solution? Decomposition

func(myX: x:, myY: y: >- getPoint())            // the result of `getPoint` is decomposed in place allowing the mapping
func(myX: x:, myY: y: >- myPoint)               // `myPoint` is decomposed in place allowing the mapping

myX: newX :, myY: newY : >- getPoint()          // can decompose and declare new variable from them
```

Consider:
```
StrangePoint :: type {
  x : Integer
  y : Integer
  foo : Integer
  bar : Integer
}

func final : ()(
  what : Integer
  ever : Integer
  x : Integer
  y : Integer
) = { ... }

myPoint : StrangePoint

func(>- myPoint)      // decompose into func (some, or in this case, no mappings can be specified)
```

This illustrates how mapping works for parameters that don't have defined explicit mappings. The member names are attempted to be match to the function inputs. If they match, they are consumed. The remaining members are positionally matched to `func` until `func` has been satisfied. If `func` were polymorphic a `func` would have to have a clear winning selection. Any `func` which could not be satisfied of all its parameters would be rejected. Any `func` whose arguments being passed to it are not compatible thus become rejected. If all the members form an exact name match to a particular `func` with no remaining members where no other function also matches the same way, then an obvious winner is made. Concepts like "longest match" or "more matching names" is not a reason to select one `func` over another. If an obvious winner can't be made then an selection ambiguity error must result. I will need recommendation as to what the rules should be for "obvious" or "best" match, but it should never be relaxed enough to ever have to guess.

Important key learning to promote into philosophy if it's not there already:
> Zax doesn't guess intent. Intent must be made clear.

This helps, but it's still not duck-like...


### Type recomposition via `-<` operator

Consider:
````zax
print final : ()(...) = {
    // ...
}

func final : (
    output1 : Integer,
    output2 : String,
    output3 : Float,
    output4 : Rune
)() = {
    // ...
    return output1, output2, output3, output4
}

// creates a new anonymous type and fills the results with the remaining
// arguments in the function's return arguments
value1 :, remaining : -< func()

// print the first result
print(value1)

// print the other resulting values
print(remaining.output2)
print(remaining.output3)
print(remaining.output4)

// ERROR: the `output1` was already extracted as an output argument thus will
// not be present as a result of using the combine operator (`-<`)
print(remaining.output1)
````

The `-<` works in the opposite direction of decomposition. Instead of taking a structure and decomposing the members away from the structure, Recomposition takes results and composes the inputs into the type, of which a type can be anonymous or a named type.

The `=` is replaced by the `-<` to indicate the recomposition. The result mapping takes place as normal. Whatever results are consumed as part of the result mapping become ineligible for recomposition. However, the final variable positionally before the `-<` accepts the remaining return results into a composed structure.

If the type of the recomposed result is specified, the results are by name mapped into the type. Meaning an attempt is made to map each result into type's shape where the shape's name matches the result name. Effectively this delivers a `[{ .result1 = result1, .result2 = result2 }]` style constructor package to the type. If the type is incompatible with receiving that pseudo style constructor then it would be incompatible to use with recomposition. The ordering of initialization of the members should follow the rules as if the programmer has specified the members in a constructor package by member name in the result order. Constructor rules don't change just because of recomposition.

If the type of the recomposed result is not specified (anonymous), a shape is dynamically constructed based on the results that remail where each becomes a member inside the type and is mapped to the result. If the return result was a reference, the member becomes a reference. The order of this new shape is based on the order of the remaining results as defined in the callee.

If no results remain, an existing type is constructed as if it were delivered `[{}]`; if an anonymous type was used, the shape of the anonymous type would contain no members.

No attempt should be made to access two different recompositions from a single function invocation.

Another form is an assignment based operation. Consider:
````zax
myType :: type {
  x : Integer
  y : Integer
}

func final : (
    x : Integer,
    y : Integer
)() = {
    // ...
    return x, y
}

myType -< func()
````

In the above example, the recomposition become an assignment operator. Each result is mapped to a member by the order of the result are listed and `myType.*` is assigned one-by-one to the results.

This is still not duck types, but it's a step in the direction because an anonymous type can be formed, and this type has a shape.

### Transformation operator `-<>-`

This is a depose and recomposition in a simple step.

```
MyType :: type {
  x : Integer
  y : Integer
  foo : String
  bar : String
}

MyOtherType :: type {
  bar : String
  foo : String
  x : Integer
  y : Integer
  whatever : String
}

myType : MyType
myOtherType : MyOtherType

myType -<>- myOtherType    // perform assignment
```

This tells the compiler to decompose `myOtherType` and recompose it into `myOtherType`. The types that match by name become transferred from `myOtherType` into `myType` by one-by-one assignment.

With the above example, consider:
```
myTransformed : myOtherType -<>- MyType    // deliver as [{ .* = ... }] constructor package
```

Advice needed as it is undecided: What happens where not enough arguments are needed for assignment/construction or too many are delivered. My leaning is that it works so long as it's compatible. "compatible" is not yet defined.


### Understanding the symbols

The symbols have visual meaning. 

For decomposition `>-`:
The `>` delivers an "open set" of values on the lhs (left hand side) taken from a packed shape `-` on the rhs (right hand side).

For recomposition `-<`:
The `-` delivers a packed shape on the lhs (left hand side) taken from an "open set" of values from the rhs (right hand side).

For transformation `-<>-`:

The left `-` delivers a packed shape from an "open set" `<` of values taken from a newly derived "open set" `>` of values taken from a packed shape from the right hand side.


### Decomposition and Recomposition and Transformation overloading

Not supported.


### Anonymous Types

Consider:
```
myType : :: type {
  x : Integer
  y : Integer
}
```

The above type is an anonymous type. It does not have a name and it's dynamically created to have the shape defined. The compiler does not assign a name, although it might internally construct a handle representing this type for tracking purposes but this is never exposed. Reflection on such a type will provide the type a unique type id, like all distinct types have a compiler assigned unique id, but it lacks a name because it doesn't have one.

Input parameters and return results can have anonymous types.

Consider:
```
func final : (
  result : :: type {
    x : Integer
    y : Integer
  }
)(
  result : :: type {
    start : Point
    end : Point
  }
) = { ... }

result := func( ... problem - how pass a strict type when this type is local the function??? ...)

print(result.x, result.y)     // the anonymous shape of the result is fine because a result can assume a type
```

Results are easily handled already because they can adopt the type as returned. 


### Putting it together (where type shape names overlap)

Transformation allows one type to become transformed into another's shape. This is directly useful when the shapes are not exactly alike. There is cost to performing the transform an `-<>-` correctly symbolizes the cost of such a transformation. Still, it's useful.

```
MyStrangePoint :: type {
  name : String
  x : Integer
  y : Integer
}

Point :: type {
  x : Integer
  y : Integer
}

myStrangePoint : MyStrangePoint
myPoint : Point

func final : (result : Point)(input : Point) = {...}

anotherStrangePoint : MyStrangePoint -<>- func(-<>- myStrangePoint)

anotherStrangePoint -<>- func(input: -<>- myStrangePoint)   // similar to above, except assignment `=`, and labelled input mapping
```

This is an illustration where the input argument is prefixed by `-<>-` is transformed into the input shape. The `input` becomes the lhs and the `myStrangePoint` becomes the rhs.

A `-<>-` always performs a copy operation of some kind. If the input argument were a reference, the `-<>-` operation would create a temporary based on the `input` type, construct it using `[{...}]` semantics from the argument supplied, and apply the `last` stance on this temporary to allow for some form of optimized transform.

A similar process happens for referenced results, although it's not necessary to create a temporary because the storage shape is already supplied in the lhs.

This process only works if the input and the outputs are transformation compatible, but it does not require the shapes be the same, but the transferred shapes only map names within the shape that match.

### Shape remapping

Consider:
```
MyReshaper :: type reshape {
  x: myX:     // from: to:
  y: myY:     // matching how input parameters and result parameter matching work
}

MyPoint :: type {
  myX : Integer
  myY : Integer
}

Point :: type {
  x : Integer
  y : Integer
}

myPoint : MyPoint
point : Point

myPoint -<>- point    // error: these cannot be transformed - the names don't match

myPoint -<>- : MyReshaper -<>- point  // success!

myPoint -<>- MyReshaper -<>- point    // error: an actual instance of the reshaper is required (although compiler will attempt to optimize the intermediate step away)

myReshape : MyReshaper                // error: a reshaper requires a transformation source

myGoodReshape : MyReshaper -<>- point // success! The shape is based on the remapping but the member typing is based on the rhs
```

A reshaper does not define it's own storage. It's reason for existing is to aid in transformation. This reshaper type takes member names from the rhs and reshapes them to the lhs. This can be used as an intermediate shape to get two incompatible shapes to align. Names that do not have a mapping (lack a target or lack a source) are not part of the reshaping, but when the reshape is used as an intermediate shape, names on either side that are not remapped attempt a direct remapping between each other as if they "bypassed" the reshape. But ONLY if the names were excluded from the reshape type.

A reshape type cannot be an input argument (at least outside a generic perhaps, but that can be raw input design pressure for a generic). The reason is that the reshape would have to take on the type member information from the argument thus the concrete implementation would have to adjust per argument. That can't happen outside of generics.

A reshape type can be used a result type because the shape can be based on the original result and only one concrete shape can immerge. That would create a concrete version of the result based on the reshape.

Undecided:
- additional non-consumed members from the reshape can be included in the result shape so long they don't conflict with the new names in the reshape result. Should they?
- if a function returns two types as a result, can each type have it's own reshape? I don't see why not... but need syntax on that...

## Compatible Shapes

### Types of Compatibility

#### directly compatibility

Consider:
```zax
Point :: type {
  x : I32
  y : I32
}

Coordinates :: type {
  x : I32
  y : I32
}
```

While "transformation" can be done to convert these shapes, they are actually not only memory compatible, they are name compatible too, thus they are said to be "directly compatible". Why? They have exactly the same declared storage. Each member is the same name with the same type and neither has more or less members than the other. In every way they are the same except with a different name. Non storage types, like a `final` function do not occupy actual space in the structure.

Keep in mind that directly compatible does not mean they are logic compatible. If the `x` was in pixels in one scenario and the other is in kilometers, they are not obviously the same value. This is the entire reason why Zax enforces strict typing: to prevent logic incompatible types from being seen as the same.


#### Slices Compatibility

Consider:
```zax
Point2D :: type {
  x : I32
  y : I32
}

Point3D :: type {
  x : I32
  y : I32
  z : I32
}

myPoint2D : Point2D
myPoint3D : Point3D

handle2DPoint(myPoint2D)    // okay
handle2DPoint(myPoint3D)    // error -- but why? why can't this be made compatible?

handle3DPoint(myPoint2D)    // error: a `Point2D` lacks all the members needed for a Point3D
handle3DPoint(myPoint3D)    // okay
```

In this example, these types are almost memory and name compatible. Except one type has more members than the others, otherwise they would be entirely the same. The `Point3D` is said to be "slice compatible" with a `Point2D` because the additional members could be sliced away to reveal a "directly compatible" type. To be clear a `Point3D` is not "directly compatible" but via slicing it can be made to be "directly compatible".

However, a `Point2D` cannot be made "directly compatible" because it lacks the additional member required to be a `Point3D`. There's no way for `Point2D` to fake that extra member of guess what it could be thus it is not "slice compatible", since there's no way to slice extra members away from a `Point2D` to become a `Point3D`.


#### Memory Compatibility

Consider:
```zax
Point :: type {
  x : I32
  y : I32
}

Coordinates :: type {
  myX : I32
  myY : I32
}
```

These types do not have "directly compatibility" and "slice compatibility" does not help to make them have "directly compatibility". However, it's fairly clear to a reader that these are in fact meant to be conceptually the same, while the name differ, the types do not; thus they are said to be "memory compatible". Except, the compiler can't know they are the same. Maybe the types use different units? One could be defined in pixels and the other in meters, which would not be "logic compatible". There's no way a compiler can automatically say they are the same.

Other compiler directives like alignment can affect "memory compatibility".

Consider:
```
MyType :: type {
  flag : Boolean
  value : Integer
}
```

Two types without compiler specified individual alignment rules will be memory compatible if the only difference is the name of a member. But a type with a different compiler alignment direct may not be memory compatible, or worse only memory compatible on some systems but not others.

But caution is warranted. Memory compatible is not logic compatible. The compiler has no way to determine logic compatible.


#### Coerced Memory Compatibility

Consider:
```
A :: type {
  value1 : UChar
  value2 : Integer
}

B :: type {
  value1 : U8
  value2 : Integer
}
```

On many systems `A` and `B` could be thought of as being memory compatible. But they aren't... not exactly... Just as an alignment compiler directive might cause a type to be in/out of memory compatible, so can different types. If `UChar` is 8bits wide, then it would likely be memory compatible. Both are unsigned. Both have the same size. But they are not the same type. And worse, on some systems a `UChar` could be 16 or 32 bits. `UChar` requires a minimize size, not a maximum.

Thus types with defined alignment where the alignment could change compatibility across machines, or types that appear to be compatible, but aren't necessarily on all machines, or a `U8` and an `I8` might be size compatible, but direct memory compatible they could not be as there's a clear logical intent difference.

Thus these types of compatibilities are called "coerced memory compatibility". They can be forced to be made compatible. But that doesn't mean they are indeed compatible.

#### Not Compatible

Consider:
```
A :: type {
  value1 : U8
  value2 : U8
  value3 : Integer
}

B :: type {
  value1 : Integer
  value2 : Float
}
```

These types are not memory compatible or even "coerced memory compatibility". Forcing one type to be seen as the other via some kind of `unsafe cast` operation would cause undefined behavior, at least in the sense the compiler can't define what will happen. If the types are indeed compatible through some casting lens is entirely based the programmer understanding what is happening at the byte level on a particular target system. The compiler will not even attempt to see these types as similar, and if one is used as the other via an `unsafe cast` process, the compiler will allow it and treat one type as another but no outcome is predictable by the compiler.

#### Combined Compatibility with Slice Compatibility

Slice compatible can be also other forms of compatible, except directly compatible. That can be determined if slicing off the tail set of members from one type would make that type compatible with another type in some form (even if it's just "coerced memory compatible"). Thus slicing is a process to create a form of compatibility. In itself, it does not make compatibility alone.

### Conversion of Compatibility

#### Directly Compatibility Conversion

While an `as` operator can be used to convert one type into another via a type's `as` overload, this is not the same as being the same type.

To convert from one to the other a special compiler protected operator is needed:
```
myType become directly MyOtherType &
```

This will allow a memory compatible type to be seen as the other.


#### Sliced Compatibility Conversion

```
myPoint3D become directly sliced Point2D &
```

This will cause a type that is slice compatible and otherwise memory compatible to be treated the other. By reference will be a direct conversion and by value will cause a temporary to become constructed with the type being sliced.

Warning is needed. Slicing could cause problems. For example, if a member has a pointer to a sliced member, that slicing might cause issues. The compiler upon seeing unsafe conditions might force `unsafe become sliced` but that doesn't mean that logically the slicing make sense. Slicing off some values might cause other logic concerns beyond mere raw safety.

#### Memory Compatibility Conversion

```
myMemoryLayout become memory YourMemoryLayout &

myMemoryLayout become memory sliced YourMemoryLayout &
```

This will cause one layout to be seen as the other layout. Even though the names don't match, the types do so the compiler will allow this conversion. The compiler may enforce `unsafe` if the compiler determines that slicing or other rules breaks could cause concerns.


#### Coerced Memory Compatibility Conversion

```
myMemoryLayout unsafe become coerced memory YourMemoryLayout &

myMemoryLayout unsafe become coerced memory sliced YourMemoryLayout &
```

This will cause one layout to be seen as the other layout, even though they are not exactly the same type or alignment. To be clear, they must be current compiler target alignment compatible (thus other targets might fail), and the types must be size compatible and identity stripped compatible. The compiler requires `unsafe` because this is not something typical and acknowledgement must be done that this can cause issues on other targets.

The compiler will still error if the member types are not aligned on the same target, for example a `U8` and a 16 bit `UChar` are not alignment compatible even if they occupy the same reserved memory space because of alignment. Likewise types that are not identity stripped compatible are not allowed to be converted through this method.


#### Non Compatible Conversion

```
myMemoryLayout unsafe cast NonCompatibleType &
```

Unsafe case will not care of the types are compatible or not. The compiler won't care if the slicing is compatible. The compiler won't even care if a `Point2D` is seen as a `Point3D` after the `unsafe cast`. The programmer has requested this conversion and has accepted the risk in using types post conversion. Improper usage of an `unsafe cast` type will cause undefined behavior.


### Compatibility Stances

The conversions above allow direct conversion to a known type.

What remains to be solved is the original problem to solve:
```
func final : (
  result : :: type {
    x : Integer
    y : Integer
  }
)(
  result : :: type {
    start : Point
    end : Point
  }
) = { ... }

result := func( ... problem - how pass a strict type when this type is local the function??? ...)

print(result.x, result.y)     // the anonymous shape of the result is fine because a result can assume a type
```

Enter compatibility stances. Compatibility stances in many ways behave like `last` `copy` `move` and `deep` except they are a different axis and are greedily stripped as soon as they can be.

They do not affect selection directly, meaning that an input parameter declared as one compatibility stance vs another has no bearing if the argument passed in is allowed. Input parameter stances are entirely ignored. Result compatibility stances are not (but that's another consideration later).

What a type's compatibility stance dictates is it's willingness to be seen as compatible with another type. It's a self affirmation for the type instance, "I, `MyType` instance, am willing to be compatible with a stance of <compatibility stance>".

Thus:
```
MyPoint :: type {
  x : Integer
  y : Integer
}

myPoint : MyPoint
myDirectlyPoint : MyPoint directly

result1 := func(myPoint as directly)  // will allow `directly` compatible conversion to the input type
result2 := func(myDirectlyPoint)      // will allow `directly` compatible conversion to the input type

result3 := func(: {             // anonymous type instances are automatically granted a `directly` compatible stance
  x : Integer = 3
  y : Integer = 4
})
```

The stances are as follows:
- `directly` - the instance is treated as "directly compatible"
- `sliced directly`- the instance is treated as "directly compatible" with allowance for slicing
- `memory` - the instance is treated as "memory compatible"
- `sliced memory` - the instance is treated as "memory compatible" with allowance for slicing
- `coerced memory` - the instance is treated as "coerced memory compatible"
- `sliced coerced memory` - the instance is treated as "coerced memory compatible" with allowance for slicing

Use `as <compatible stance>` for conversion. `unsafe` stance versions of the compatibility stance might be required for some versions to function. Those should only be used when paired with a `as unsafe <compatibility stance>` and should not be declared on a type's direct instance due to the inherent risk.

These stances are stripped greedily. Any instance transfer strips the self affirmed compatibility stance, although the transferee is allowed to declare their own new compatibility stance.

## Starting examples and questions

These were questions asked by the a previous agent to be answered. With the tools above, the should hopefully all now have a solution or a position.

### Same members in a different order

```zax
Point :: type {
  x : I32
  y : I32
}

Coordinates :: type {
  y : I32
  x : I32
}
```

Questions:

- Are `Point` and `Coordinates` distinct type identities but structurally
  equivalent?

> They are distinct. While they are "memory compatible" they are clearly not logically compatible. This they should use a transformation to be compatible.

- Do stored-member names participate in structural shape?

> Yes, they do determine compatibility.

- Does declaration order participate?

> Yes, it does, as does alignment.

- Which member qualifications, defaults, lifecycle declarations, or other
  attributes participate?

> I need help deciding. For example `readonly immutable` is not necessarily compatible. Exact rule sets need to be defined.

- If the types have the same structural shape, does that permit assignment,
  require an explicit conversion, or merely describe a fact?

> It depends on the shape and differences.

- Can references share a same-storage view, or does that require separately
  proved layout, lifetime, qualification, and alias authority?

> Yes, they can have memory compatibility.


### A source with additional members

```zax
Position3D :: type {
  x : I32
  y : I32
  z : I32
}
```

Questions:

- Can `Position3D` satisfy a place where `Point` is expected?

> Yes, it can, with slicing.

- Is exact shape the only ordinary structural relationship?

> For direct conversions it's supremely important.

- Could a separate directional projection select `x` and `y` explicitly?

> Yes, via transformation.

- Would that projection copy a new value, discard `z`, or expose a
  same-storage prefix view?

As a reference `z` would be present, but sliced away from view (and if copied would be lost). By copy, it's lost from the copy.

- Which of those possibilities are compatibility facts, and which require a
  separately authorized conversion or transfer?

> The rules above illustrate what is possible.

- Must any lossy or truncating operation make its cost and discarded state
  visible?

> Yes, transforms are more costly and thus have more visible cost clearly associated to them.


### Stored shape versus composition surfaces

```zax
Engine :: type {
  rpm : Integer

  start final : ()() = {
  }
}

Car :: type {
  engine own expose : Engine
  wheels : Integer
}
```

Questions:

- Is `Car`'s physical instance shape composed only from `engine` and `wheels`?

> Yes, those are the members. Path binding does not affect shape compatibility. That said, bindings might be able to participate in transformations if appropriate rules can be defined to "what that means when done".

- Does the published `Car.rpm` path contribute another structural member, or
  remain only another path to `engine.rpm`?

> Only another path, but it might be able to participate if transformations (maybe).

- Do data `via` routes, preferred projections, exposed or adopted operations,
  fulfilled roles, or fences participate in storage shape, semantic shape,
  whole-type conformance, or reflection?

> `final` functions to not thus `via` with `final` declarations do not participate. Neither do `once` values.

- How should a structural model preserve the distinctions among stored
  declarations, access paths, generated declarations, and explicitly written
  declarations established by current composition?

> If they should participate as paths to transformations, I would need help defining the rules of how they are treated.

- Can `outer tracked` metadata affect storage or layout compatibility without
  changing semantic structural shape?

> It might, I would need help reasoning through the implications of a `tracked` type when it comes to compatibility. Certainly a `tracked` and non tracked version that would otherwise be directly compatible, are NOT directly compatible. But two `tracked` values with the same shape might be... maybe.

### Anonymous structural results

Current arithmetic reports create anonymous values conceptually shaped like:

```zax
<anonymous> :: type {
  wrapped : Integer
  saturated : Integer
  overflowed : Boolean
}
```

Questions:

- What exact type identity does an anonymous structural value have?

> A distinct and unique anonymous type. While the compiler may optimize implementations of compatible shapes, behavior wise they must remain and be treated as if each anonymous types was wholly and individually unique.

- Are separately produced anonymous values with the same members structurally
  equivalent?

> No. They might be compatible though.

- Can one initialize a named type with the same shape, and if so through what
  explicit or contextual operation?

> Yes, with the correct compatibility stance or pre-conversion.

- Is an anonymous value distinct from a callable's several result slots even
  when the labels and types happen to match?

> They are distinct but anonymous instances carry a `distinctly` compatibility stance to aid and loosen selection compatibility.

- Which layout, temporary-lifetime, and reflection facts must programmers be
  able to observe?

> All of them.


### Whole-type no-storage contracts

Composition currently provides member-level `abstract` roles. Future generic or
structural work may also need a contract describing the required shape of a
complete type without adding storage, inheritance identity, or runtime
dispatch.

Questions:

- Is a whole-type contract needed as part of the structural model?

> `abstract` is its own thing. Generics can consider using `abstract` to aid in compatibility declaration. This should be a design pressure and consideration for generics.

- Can it require stored values, callable prototypes, operators, or some bounded
  combination?

> `abstract` is its own thing. I suggest future raw `abstract` type contracts use this as design pressure / concerns / consideration, especially in relation to generics.

- Is conformance nominally declared, structurally matched, explicitly
  fulfilled, or composed from more than one of those relationships?

> Future `abstract` work.

- How do required versus `abstract optional` roles contribute?

> Future `abstract` work.

- How do exact versus `abstract relaxed` qualifier requirements contribute?

> Future `abstract` work.

- Does conformance permit use, conversion, or generic admission without making
  two type identities equivalent?

> Future `abstract` / generic work.

- How does contract evolution affect existing conforming types?

> Future `abstract` work.

## Distinctions the review should preserve

The work should distinguish rather than conflate:

1. type identity;
2. exact structural shape;
3. storage and layout compatibility;
4. conversion, projection, or truncating-transfer permission; and
5. whole-type conformance.

Equal shape must not silently imply equal identity, interchangeable storage,
implicit conversion, inheritance, or runtime dispatch. Conversely, a permitted
conversion must not by itself prove equal shape or layout.
