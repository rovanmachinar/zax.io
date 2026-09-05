
# [Zax Programming Language](index.md)

## Functions

Accepted function-value declaration, recursive-binding, qualifier, and
assignment boundaries are defined by
[Zax declarations and bindings](language/declarations-and-bindings.md).

Current qualifier behavior for parameters, results, captures, and
[receiver operands](language/terms.md#receiver-operand) is defined by
[Zax qualifiers](language/qualifiers.md).

Current ordinary calls, argument/default binding, result slots and routing,
fixed-arity callable selection, compatible visible prototypes, and synchronous
completion are defined by
[Zax function invocation](language/function-invocation.md). The remaining
`copy`/`deep`/`move`/`last` model is defined by
[Zax transfer stances](language/transfer-stances.md). The remaining material
below is legacy input for captures, reassignment, raw function pointers,
composition, `mutator`, `once`, variadic, and related future function behavior.


### Functions value capturing

````zax
print final : ()(...) = {
    // ...
}

changeValue final : (result : Boolean)() = {
    // ...
}

a : Integer = 1
b : Integer = 2

func : ()() = [a, b] {
    // The value of `a` and `b` are captured by-value and `1` and `2` will
    // always be displayed (unless the captured `a` or `b` is changed
    // locally)
    print(a, b)

    // If the function `changeValue()` returns `true` then the captured
    // `a` will become `42` but the global `a` will retain its original value.
    if changeValue()
        a = 42
}

// No need to specify the captured values as they are already contained
// within the captured function.
func()
````


#### Function declaration and definition with capturing value reassignment

````zax
print final : ()(...) = {
    // ...
}

// Declare and define a variable and function type with no return results, an
// `Integer` input argument, with the ability to capture variables and
// assign the function to point to nothing.
magicNumber : ()(value : Integer)
`
// variable representing the current message to display
messageToDisplay : String = "You've found the number"

// assign the `magicNumber` to code to execute and capture the value of
// `messageToDisplay` for later usage
magicNumber = [messageToDisplay] {
    if value == 1001
        print("Horray -=> ", messageToDisplay)
}

// reassign the `messageToDisplay` to an entirely new value
messageToDisplay = "Move along, nothing to see here people"

// execute the `magicNumber`'s code which will display the value of the message
// as it was originally captured and not the new value
magicNumber(1001)   // will display "Horray -=> You've found the number"
````


### Functions can capture reference

Functions can capture by reference (or by other type) by declaring a new variable that refers to the original variable.

````zax
print final : ()(...) = {
    // ...
}

assert final : ()(okay : Boolean) = {
    // ...
}

changeValue : (result : Boolean)() = {
    // ...
}

a : Integer = 1
b : Integer = 2

// Both `a` and `b` are captured by reference. The variable `a` maintains the
// `a` variable name whereas `b` is captured as a reference into a new variable
// named `altB`.
func : ()() = [&a, altB : & = b] {
    // The value of `a` and `b` are captured by reference and will always
    // display and reference the contents of the global scope `a` or `altB`.
    print(a, altB)

    // If the function `changeValue()` returns `true` then the captured
    // `a` will become `42` which is a reference of the global value.
    if changeValue()
        a = 42
}

func()
assert(a == 1 || a == 42)

a = 5

func()
assert(a == 5 || a == 42)
````


### Function input capturing and chaining

The `>>` composition and capture forms and `|>` chaining material in this
section remain legacy evidence. Current operator work fixes `>>` parsing but does
not accept the composition signatures, generated callable types, capture
representation, or chaining behavior shown below. Those questions are preserved
in [raw function composition and chaining input](project/raw/function-composition-and-chaining.md).

#### Functions input / output composition

If a function's output arguments matches the input arguments of another function they can be bound together as a newly created function using the function composition operator (`>>`). Each of the output arguments from the first function must match the input arguments from the second function. Functions can be composed together in chains of two or more functions where each input is chained to another output.

````zax
func1 final : (result : Integer)(input : String) = {
    // ...
}
func2 final : (result : String)(input : Integer) = {
    // ...
}

// func3 takes a `String` and returns a `String`
func3 final := func1 >> func2

// func4 takes a `String` an returns an `Integer`
func4 final := func3 >> func1

// bind four functions together in a longer chain
func5 final := func1 >> func2 >> func1 >> func2

// calling func4() actually calls func1(func2(func1()))
value : Integer = func4("5")
````


#### Function argument capturing and composition

A function can captured values to chain into a new function. When value capturing occurs, the function is not actually called immediately. Instead captured values become input arguments passed into an existing function and the result becomes a new function which input arguments previously captured. This allows a newly captured and created function to be called later.

````zax
func final : (result : String)(input1 : Integer, input2 : String) = {
    // ...
}

// capture values for a later function invocation
// (don't actually call the function now)
funcLater1 := [2, "hello"] >> func

// the above statement is functionally equivalent to this statement
funcLater2 := [a := 2, b := "hello"] { func(a, b) }

// call the previously captured function
result := funcLater1()

// call the previously captured function again
result := funcLater1()
````


#### Function input argument capturing and composition

A function's input arguments can be captured and automatically become input arguments to function call resulting in creating a new function which has one less (or many lessor) required input arguments than the original function. If the name of an captured variable matches the name of one of a function's input arguments then the input argument is automatically selected as that argument's input for a function. If a captured name does not match any input argument's variable name and no re-assigned captured name is given then the captured value is attempted to be matched positionally to a functions input arguments (excluding any positions which already have definitive match).

````zax
print final : ()(...) = {
    // ...
}

func1 final : ()(name : String, age : Integer) = {
    print("name: ", name, "age: ", age)
}

myNameIs final := "Slim"

func2 final := [name = myNameIs] >> func1

// will print "name: Slim age: 47"
func2(47)

// will print "name: Slim age: 48"
func2(48)


name final := "Shady"

// capture the `name` variable which matches the functions input variable `name`
func3 final := [name] >> func1

// will print "name: Shady age: 47"
func3(47)

// capture the `name` variable, and positionally assign to the functions input 
// arguments (excluding name which is already matched)
func4 final := [name, 48] >> func1

// will print "name: Shady age: 48"
func4()
````


#### Function invocation chaining

The invocation chaining operator (`|>`) can be used to feed the result of one function as the input to the next function. Input arguments are attempted to be matched by name first. After any output arguments matching to input arguments are removed then the renaming input and output arguments are matched positional. Finally remaining input arguments that do not match chained arguments will be taken from a function's invocation input argument list.

An initial input value (which is not a function call) can be chained as the first input argument into a chain of a function invocations.

The invocation chaining operator (`|>`) should not be confused with the pipe operator (`|`) or the shift right operator (`>>`) from other languages. On many languages those operators would take the output of one function and keep track of the result in internal state and then apply the state to the next function invoked. Often on those languages output argument of a chained function call is the type used to start the chain in the first place. This is how `std::cout` works in C++. The result of evaluating `std::cout << value` is the `std::cout&` object itself (i.e. via `return *this` from the `<<` operator). Whereas in Zax, the chaining operator (`|>`) literally takes the output of one function and feeds it as the input to the next function without needing an intermediate state object.

````zax
double : (result : Integer)(input : Integer) = {
    return input * 2
}

square : (result : Integer)(input : Integer) = {
    return input * input
}

half : (result : Integer)(input : Integer) = {
    return input / 2
}

squareAndAdd : (result : Integer)(input : Integer, add : Integer) = {
    return input * input + add
}

// the result of this function is "50", ((5 * 2) * (10)) / 2
result := 5 |> double() |> square() |> half()

// the result of this function is "38", (((4 * 2) * (8)) + 12) / 2
result = 4 |> double() |> squareAndAdd(12) |> half()
````


### Functions can be reassigned to new code

````zax
print final : ()(...) = {
    // ...
}

save final : ()(...) = {
}

func : ()(myValue : Integer) = {
    print(myValue)
}

// will call print(42)
func(42)

func = {
    // the variable `myValue` is known by the declared type of `func` and thus
    // `myValue` exists within this code block
    save(myValue)
}

// will call save(42)
func(42)
````


### Functions marked as `final` cannot be reassigned

````zax
func final : ()() = {
    // do something
}

// call `func`
func()

// ERROR: `func` is declared as a final value and cannot be reassigned to a
// new value in the future.
func = {
    // do something else
}
````


### Function pointers without ability to capture

Functions will allocate space needed for capture values but function pointers do not have additional space to capture any values. Function pointers are more space efficient when stored inside types with the tradeoff of being the inability to capture values.

````zax
print final : ()(...) = {
    // ...
}

// define and declare a function that return no results and takes no input
// arguments and has no ability to capture data and assign the function to
// point to nothing (since the function is only a function pointer without
// the storage capacity of captured values)
funcWithNoCapture : ()() *

// reassign the function to new executable code
funcWithNoCapture = {
    // ...
}

// execute the declared function
funcWithNoCapture()

myMessage : String = "Try to capture me!"

// ERROR: The function variable named `funcWithNoCapture` is not capable of
// capturing data and thus will error if a capture was attempted
funcWithNoCapture = [myMessage] {
    // ...
}
````


#### More errors when attempting to capture with capture disabled

````zax
print final : ()(...) = {
    // ...
}

value := 42

// ERROR: The function is a raw function pointer and cannot capture any values.
func : ()()* = [value] {
    print(value)
}

// ERROR: not possible to call the function since the capture of value was
// not done
func()

// ERROR: Even though no values are captured an error will occur as the declared
// type for `func` cannot capture values
func = [] {
    // do something else...
}

func()
````


### `mutator` functions

Function variables declared with `mutator` become [mutator methods](https://en.wikipedia.org/wiki/Mutator_method). Accessing `mutator` variable name as a value will call a method which matches a function prototype returning the type requested with no input arguments. Assigning a value to the `mutator` name with a value will call a method which matches a function prototype of accepting the assigned type as an input argument and optionally returning a result type.

Functions that return a value marked as `mutator` are especially useful when a data variable is calculated rather than being a natural data type. As `mutator` functions add overhead in each function call, care must be done to ensure `mutator` functions are not overly accessed in CPU cycle sensitive code. Likewise, a value change in a seemingly unrelated variable on a type might impact a `mutator` which might not be as obvious given a function is being called in a hidden way.

Function variables marked as `mutator` can be polymorphic. The compiler will attempt to match input or output arguments by type to the best matching `mutator` function.

The example below creates a `mutator` that returns a value:

````zax
// length is a calculated value type and not a true independent value
length mutator final : (output : Integer)() = {
    // ...
    return output
}

// the `length` function is called which returns a value to a function
roomSize := 5 + length

// ERROR: `length` has no `mutator` that accepts an `Integer` value
length = 10
````

The example below creates a `mutator` that returns a value and a `mutator` that accepts a value:

````zax
// length is a calculated value type and not a true independent value
length mutator final : (output : Integer)() = {
    // ...
    return output
}
length mutator final : ()(input : Integer) = {
    // ...
}

// the `length` function is called which returns an `Integer` value
roomSize := 5 + length

// the `length` function is called which accepts an `Integer` value
length = 10
````

The example below creates two polymorphic `mutator` function that both accept a value:

````zax
calculateHistoricalAge final : (output : Integer)() = {
    // ...
}

person mutator final : ()(input : Integer) = {
    // ...
}
person mutator final : ()(input : String) = {
    // ...
}

// the `person` function is called that accepts an `Integer` value
person = calculateHistoricalAge()

// the `person` function is called that accepts a `String` value
person = "Socrates Johnson"
````


#### Assign or replace a function `mutator` implementation

If a `mutator` function (not marked as `final`) accepts a value is instead assigned a function pointer to a function that matches its own function prototype then that function will be replaced rather than attempting to match another `mutator` function. Priority of `mutator` functions are always given to matching function prototypes rather than attempting to find a `mutator` function which accepts a function pointer as its function's input or output argument.

Likewise `mutator` functions (not marked as `final`) that return a value can instead be assigned a function pointer to a function that matches its own function prototype. This assignment will cause the definition of the `mutator` function to be replaced.

The example below replaces a `mutator` function that accepts an input value:

````zax
length mutator : ()(input : Integer) = {
    // ... do something ...
}

// the seemingly complex declaration below does the following:
// * declare an anonymous value
// * associate the type of the anonymous value to a function prototype that
//   matching the original `length` `mutator` function
// * assign the function `length` to a new replacement function
length = # : ()(input : Integer) = {
    // ... do something else ...
}

// the second function definition is called and not the original definition
length = 10
````

The example below replaces a `mutator` function that returns an output value:

````zax
length mutator : (output : Integer)() = {
    // ... do something ...
    return output
}

// the seemingly complex declaration below does the following:
// * declare an anonymous value
// * associate the type of the anonymous value to a function prototype that
//   matching the original `length` `mutator`
// * assign the function `length` to a new replacement function
length = # : (output : Integer)() = {
    // ... do something else ...
    return output
}

// the second function definition is called and not the first definition
value := length
````


### Declare and immediately call functions

Anonymous functions can be called immediately upon construction by proceeding a closing curly brace (`}`) with an open bracket (`(`), arguments and a close bracket (`)`). Once the function's declaration completes the function is executed. Anonymous functions using the discard operator (`#`) do not need to use the `final` keyword as the function is implicitly final since no value captures the function pointer.

Each of the functions below is executed after declaration:

````zax
// declare a function that takes zero arguments and returns no results
# : ()() = {
    // ...
}()

// declare a function that takes one argument and returns no results
# : ()(input : Integer) = {
    // ...
}(5)


newValue := # : (output : Integer)() = {
    // ...
    return output
}(10)
````

### Functions marked as `final`

If a function should never have its associated code changed then that function should be declared as `final`. Functions that are not marked `final` require additional storage capacity to accommodate a function pointer and may incur calling overhead to access the function via a function table instead of optimizing the call directly to code. Functions marked as `final` cannot have their code segment reassigned once declared.

````zax
MyType :: type {
    value1 := 0
    value2 := 0
    value3 := 0

    bucket final : (output : Integer)() = {
        return (value1 + value2 + value3) % 17
    }
}

myType : MyType
myType.value1 = 1
myType.value2 = 55
myType.value3 = 1001

// call the function that exists within the type
bucket := myType.bucket()

// ERROR: The function `bucket` is declared as `final` and thus cannot be
// reassigned to a new code segment.
myType.bucket = {
    // do something else
}
````

### Functions marked as `once`

Functions marked as `once` will only have a single function definition for all instances of a type and allow a function on a type to be called without specifying a type instance. A `once` function can be called by calling a function directly off a type instead of an instance of a type. Marking the function as `final` will ensure that a function cannot be reassigned to a new function definition.

````zax
MyType :: type {
    value : Integer

    func1 final once : ()(value : Integer) readonly = {
        // do something...
    }

    func2 final once : ()(value : Integer) = {
        // check if called from type instance or from a type by checking if `_`
        // points to nothing
        if _
            _.value = value   // only set value if called with a type instance
    }

}

myType : MyType

// OKAY: all instances share the same definition of `func1` so calling the
// instance with the `type` name and not an instance of a `type` is legal
MyType.func1(42)

// OKAY: all instances share the same definition of `func2` so calling the
// instance with the `type` name and not an instance of a `type` is legal
MyType.func2(42)

// OKAY: all instances share the same definition of `func2`, but calling
// the function from an instance will pass in the instance pointer to the
// shared `func2` implementation
myType.func2(42)
````


### Split and combine argument operators

#### Splitting type into a function call

The split operator (`<-') can take a type and create an argument list for a function to satisfy the functions argument list. For any type variable names matching input argument names (which are not previously matched input arguments) will be automatically filled as arguments to a function. Any additional type values unmatched will be ignored. Any unfulfilled arguments will need to be filled as per standard input argument passing rules.

````zax
print final : ()(...) = {
    // ...
}

func final : ()(
    age : Integer,
    name : String,
    weight : Float,
    defaultSmiley : Rune
) = {
    // ...
}

MyType :: type {
    name : String
    famousQuote : String
    age : Integer
    weight : Float
}

myType : MyType

func(42, <- myType, r'😀')

// print the first result
print(value)

print(myType.name)
print(myType.famousQuote)
print(myType.age)
print(myType.weight)
````

#### Splitting type into a function call

The argument split (`<-`) operator with multi-value operator and named declarations `[{` `}]` can be used to fill input arguments passed into a function with a newly declared anonymous type. Each argument is matched with an input argument. All input arguments considered satisfied by the split (`<-`) operator are no longer needing to be passed values and standard rules for remaining input arguments apply.


````zax
print final : ()(...) = {
    // ...
}

func final : ()(
    age : Integer,
    name : String,
    weight : Float,
    defaultSmiley : Rune
) = {
    // ...
}

func(42, <- [{ .name = "Boothby", .age = 61, .weight = 120 }], r'😀')
````


#### Multiple argument operator combining into an argument type

Using the multiple argument operator `[{` `}]` with named declaration can cause multiple arguments to become initialized into a type's values without using the split (`->`) or combined (`<-`) operators.

````zax
print final : ()(...) = {
    // ...
}

MyType : type {
    age : Integer,
    name : String,
    weight : Float,
    defaultSmiley : Rune
}

func final : ()(
    value : MyType
) = {
    // ...
}

func([{ .name = "Boothby", .age = 61, .weight = 120 }])
````


#### Combining argument results into an anonymous type

The argument combine operator (`->`) can be used to define and declare a new anonymous type whose values contain results from remaining output arguments returned from a function.

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
value1 :, remaining : -> = func()

// print the first result
print(value1)

// print the other resulting values
print(remaining.output2)
print(remaining.output3)
print(remaining.output4)

// ERROR: the `output1` was already extracted as an output argument thus will
// not be present as a result of using the combine operator (`->`)
print(remaining.output1)
````


#### Combining argument results into an existing type

The argument combine operator (`->`) can be used to assign values returned from function directly into an existing type. For the names of return arguments unfulfilled in existing return results, the return argument names are matched to the names in the type declared and any matching names are treated as if they were removed from the return result list as they are fulfilled. Any non matching names can be declared as additional arguments need to be captured as per standard argument returning rules. If none of the names match when using the combine operator (`->`), an attempt is made to apply an automatic `as` operator from the remaining returned types (as if they were a combined type) up to the final value present in the destination type. If the types are deemed compatible (as per `as` casting rules) those arguments are considered fulfilled and treated as if they were removed from the returned argument list. If neither method results in any matches then the compiler will issue an error. Any unmatched values present in the result results will need to be fulfilled as per standard argument returning rules.

````zax
print final : ()(...) = {
    // ...
}

func final : (
    age : Integer,
    name : String,
    weight : Float,
    defaultSmiley : Rune
)() = {
    // ...
}

MyType :: type {
    name : String
    famousQuote : String
    age : Integer
    weight : Float
}

// uses existing `MyType` type and fills the results with the arguments
// as returned from the function
value1 :, myType : MyType ->, defaultSmiley: = func()

// print the first result
print(value)

print(myType.name)
print(myType.famousQuote)
print(myType.age)
print(myType.weight)
````
