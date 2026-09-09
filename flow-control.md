# [Zax Programming Language](index.md)

## Flow Control

Current synchronous flow control is owned by
[Zax core flow control](language/core-flow-control.md): exact-`Boolean`
conditions, `if`/`else`, header initialization and posts, the conditional
expression, `while`, `until`, `redo while`, `redo until`, `forever`, explicit
`scope`, flow labels, and `break`/`continue`/`next`/`goto`/`return`.

Current `switch`, `case`, and `default` behavior is owned by
[Zax switch, case, and default](language/switch.md). Its ordered
first-match model, case operators, clause posts, transfers, optional proof, enum
coverage, and diagnostics supersede the former switch material on this page.

The sections below are the **legacy remainder** of this page. They preserve prior
design evidence for the unresolved `using` resource statement and value
polymorphism using `if`. This evidence remains non-authoritative until its own
focused review.

### `using` statement

A `using` statement is akin to a shortened `if` statement where a condition is
not specified and always assumed to be `true`. This allows a temporary resource
to be declared and used within a `using` scope. Callable-like restricted capture
is preserved separately in
[raw function-composition input](project/raw/function-composition-and-chaining.md).
If a value is declared in an initialization statement, that value's scope exists
only within the `using` control flow.

````zax
print final : ()(...) = {
    // ...
}

MyType :: type {
    value1 : Integer
    value2 : String
}

func final : (myType : MyType)() = {
    // ...
    return myType
}

using value := func()
    doSomething(value)

using value := func() {
    print(value.value1)
    print(value.value2)
}

using value own := func() {
    print(value1)
    print(value2)
}

using value own := func()
    doSomething(value);
    print(value2)
````


### Value polymorphism using `if`

An `if` statement can also be used in a function declaration to indicate that a function supports value polymorphism. Which function to call is based on a pre-condition check for a given `if` statement. The compiler will execute each test based on the order of appearance in code if no specific order has a bias. If no match is found (and if present) then a undecorated version of a function will be executed. A compiler may decide to reorder tests if the reordering will have no net resulting impact on the code flow. Care should be taken to not have overlapping pre-conditions if code order cannot be preserved or guaranteed. The `[[likely]]` and `[[unlikely]]` compiler directives can be used as a hint to a compiler which execution path is more likely to be followed (thus tests can be reordered appropriately).

If some value polymorphic functions are declared using an `if` statement then a single polymorphic version function using the same types can be declared as a catch-all if none of the other conditions succeed (i.e. the logical equivalent of a `switch` `default` statement). If no function was found a panic may be issued.

Only functions marked as `final` support value polymorphism. A conditional check on a function cannot be replaced and any assignment of a changeable functions would be ambiguous to which value polymorphic version should be replaced. However, a function without any value polymorphism `if` condition can be `varying` allowing the function to be reassigned to a new function implementation that will assume to replace only a default non-conditional version (i.e. a version that does not contain value polymorphism).

````zax
random final : (value : Integer)() = {
    // ... return a positive or negative integer
}

func final : ()(value : Integer) if { return value > 0 } = {
    // ...
}

func final : ()(value : Integer) = {
    // ...
}

forever {
    // each time the function is called a different function may be invoked
    func(random())
}
````

Another example computing factorial:

````zax
assert final : ()() = {
    // ...
}

factorial final : (r : Integer)(n : Integer) if { return n > 1 } = {
    return n * factorial(n - 1)
}

factorial final : (r : Integer)(n : Integer) = {
    return 1
}

assert(120 == factorial(5))
````

An example of a children's game of FizzBuzz using value polymorphism:

````zax
print final : ()(...) = {
    // ...
}

toString final : (result : String)(value : Integer) = {
    // ...
}

next final : (s: String)(i : Integer) if [[unlikely]] { return i % 15 == 0 } = {
    return "FizzBuzz"
}

next final : (s: String)(i : Integer) if [[likely]] { return i % 3 == 0 } = {
    return "FizzBuzz"
}

next final : (s: String)(i : Integer) if { return i % 5 == 0 } = {
    return "Buzz"
}

// next is not marked as `final` and can be replaced with an alternative
// implementation
next : (s: String)(i : Integer) [[likely]] = {
    return toString(i)
}

// displays: 1, 2, Fizz, 4, Buzz, Fizz, 7, 8, Fizz, Buzz, ... 14, FizzBuz, ...
while i := 1 ;; i < 100 ;; ++i {
    print(next())
}
````


#### Value polymorphism using `if` on nothing instances

A [nothing instance](nothing.md) can filter between normal function calls and functions that are called on a nothing instance. By checking if a self pointer  (`_`) is valid inside an `if` condition of a value polymorphic function, code can decide if a nothing version of a function or a normal version function should be called.

````zax
MyType :: type {
    +++ final : ()(:Nothing) = {
        // instance to a nothing type
    }

    doSomething final : ()( : Integer) if [[unlikely]] { return !_ } = {
        // do nothing -- inside nothing instance of MyType
    }
    doSomething final : ()(value : Integer) = {
        // ...
        // do something -- normal instance of MyType
        // ...
    }
}

myType1 : MyType *       // points to Nothing
myType2 : MyType * = @   // declaration-bound allocation

myType1.doSomething()   // does nothing
myType2.doSomething()   // does something
````
