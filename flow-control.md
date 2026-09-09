
# [Zax Programming Language](index.md)

## Flow Control

Core synchronous flow control is now owned by
[Zax core flow control](language/core-flow-control.md): exact-`Boolean`
conditions, `if`/`else` and header initialization, the conditional (ternary)
expression, `while`, `until`, `redo while`, `redo until`, `forever`, explicit
`scope`, flow labels, and `break`/`continue`/`next`/`return`. The operator
behavior those conditions rely on is owned by
[Zax operators](language/operators.md).

The sections below are the **legacy remainder** of this page. They preserve
prior design evidence for constructs that core flow control deliberately
deferred: `switch`/`case` selection, the `using` resource statement, and value
polymorphism using `if`. This evidence is non-authoritative until its own
focused review. Future selection work must revisit the legacy `case continue`
form under the accepted `next`/`continue` distinction.


### `switch`

#### The `switch`, `case` and `default` flow control

A `switch` statement can be used to test a variable against a set of values. A `switch` statement will compare against a value list and execute zero or more statements based on values tested. A `switch` value is compared against each `case` value for equality and if `default` is present then any `switch` value that does not match an existing `case` causes a `default` code block to be executed.

````zax
doSomething final : ()() = {
    // ...
}

coinFlip final : (result : Boolean)() = {
    // ... return `true` or `false` randomly ...
}

func final : ()(value : Integer) = {
    switch value {
        case 1 {
            if coinFlip()
                break
            doSomething()

            // unlike other languages, a `break` is not required between
            // `case` statements as the code logic will not flow through from
            // one case to another automatically
        }
        case 2
            // single statement is executed then the `switch` exits
            doSomething()
        case 3
        case 4 {
            // if `value` is `3` or `4` two statements are executed and then
            // the `switch` exits
            doSomething()
            doSomething()
        }
        case 5
        case 6
            doSomething()
            // ERROR: this code will not compile as multiple statements are
            // present thus requiring using a `{}` scope or the `;` operator
            doSomething()
        case 7
        case 8
            // the `;` operator causes both statements to be joined as a
            // single statement at the same scope
            doSomething();
            doSomething()
        case 9
        case 10
        default {
            // values not `1` through `8` will execute this default scenario
            doSomething()
            doSomething()
        }
    }
}
````


#### Using `switch` to compare against runtime values

A `switch` statement can be used to compare against other runtime values. Values should not be computed inside a `case` statement but values can be used as a comparisons.

````zax
doSomething final : ()() = {
    // ...
}

coinFlip final : (result : Boolean)() = {
    // ... return `true` or `false` randomly ...
}

uniqueRandomNumber final : (result : Integer)() = {
    // .... return a random number never returned before ...
}

func final : ()(value : Integer) = {

    a := uniqueRandomNumber()
    b := uniqueRandomNumber()
    c := uniqueRandomNumber()
    d := uniqueRandomNumber()

    switch value {
        case a {
            if coinFlip()
                break
            doSomething()

            // a `break` is not required between `case` statements as the code
            // logic will not flow through from one case to another
            // automatically
        }
        case b
            // single statement is executed then the `switch` exits
            doSomething()
        case c
        case d {
            // if `value` is `c` or `d` two statements are executed and then
            // the `switch` exits
            doSomething()
            doSomething()
        }
    }
}
````


#### Using `switch` with complex types

A `switch` statement can be used to compare against complex types. Complex types must have comparison operator support.

The example below uses String types (but any type could be used):

````zax
doSomething final : ()() = {
    // ...
}

coinFlip final : (result : Boolean)() = {
    // ... return `true` or `false` randomly ...
}

uniqueFruit final : (result : String)() = {
    // .... return a unique random fruit name ...
}

func final : ()(fruit : String) = {

    a := uniqueFruit()
    b := uniqueFruit()
    c := uniqueFruit()

    realD := uniqueFruit()

    d : String * = realD

    switch fruit {
        case a {
            if coinFlip()
                break
            doSomething()
        }
        case b
            // single statement is executed then the `switch` exits
            doSomething()
        case c
        case d. {
            // if `fruit` is `c` or `d` two statements are executed and then
            // the `switch` exits
            doSomething()
            doSomething()
        }
    }
}
````


#### Using `switch` with alternative operators

A `switch` statement can be used with other binary operators that return a `Boolean` value. When binary operators are used, evaluation ordering of binary conditions occur in the same order as `case` conditions.

````zax
doSomething final : ()() = {
    // ...
}

coinFlip final : (result : Boolean)() = {
    // ... return `true` or `false` randomly ...
}

uniqueRandomNumber final : (result : Integer)() = {
    // .... return a random number never returned before ...
}

func final : ()(value : Integer) = {
    switch value {
        case < a {
            if coinFlip()
                break
            doSomething()

            // a `break` is not required between `case` statements as the code
            // logic will not flow through from one `case` to another
            // automatically
        }
        case b
            // single statement is executed then the `switch` exits
            doSomething()
        case > c
        case < d {
            // if `value` is `c` or `d` two statements are executed and then
            // the `switch` exits
            doSomething()
            doSomething()
        }
    }
}
````


#### `switch` statement and condition

A `switch` statement can contain a statement followed by a variable to test which must be separated by a sub statement separator (`;;`) operator. If a value is declared in an initialization statement then that value's scope only exists within the context of a `switch` control flow. This scenario can be useful to capture a computed value, test the computed value, and later access the previously computed value.

````zax
doSomething final : ()() = {
    // ...
}

coinFlip final : (result : Boolean)() = {
    // ... return `true` or `false` randomly ...
}

uniqueRandomNumber final  : (result : Integer)() = {
    // .... return a random number never returned before ...
}

func final : ()(value : Integer) = {

    a := uniqueRandomNumber()
    b := uniqueRandomNumber()
    c := uniqueRandomNumber()
    d := uniqueRandomNumber()

    switch value := uniqueRandomNumber() ;; value {
        case a {
            // use captured value
            if value < 0 && coinFlip()
                break
            doSomething()

            // a `break` is not required between `case` statements as the code
            // logic will not flow through from one `case` to another
            // automatically
        }
        case b
            // single statement is executed then the `switch` exits
            doSomething()
        case c
        case d {
            // if `value` is `c` or `d` two statements are executed and then
            // the `switch` exits
            doSomething()
            doSomething()
        }
    }
}
````

#### `switch` statement and `case continue`

A `case continue` statement will cause the next case clause to be processed as if the following `case` (or `default`) condition were true. Effectively this allows for one `case`'s statement to fallthrough to another `case`'s statement.

````zax
doSomething final : ()(value : Integer) = {
    // ...
}

coinFlip final : (result : Boolean)() = {
    // ... return `true` or `false` randomly ...
}

func final : ()(value : Integer) = {
    switch value {
        case 1 {
            if coinFlip()        // `if coinFlip()` is `true`
                case continue    // will cause the next `case 2` to `continue`
                                 // processing as if it were a true thus causing
                                 // `doSomething(2)` to execute but not
                                 // `doSomething(1)`

            doSomething(1)

            // unlike other languages, a `break` is not required between
            // `case` statements as the code logic will not flow through from
            // one case to another automatically
        }
        case 2
            // single statement is executed then the `switch` exits
            doSomething(2)
        case 3
        case 4 {
            // if `value` is `3` or `4` two statements are executed and then
            // the `switch` exits
            doSomething(3)
            doSomething(4)
        }
        case 5
        case 6
            doSomething(5)
            // ERROR: this code will not compile as multiple statements are
            // present thus requiring using a `{}` scope or the `;` operator
            doSomething(6)
        case 7
        case 8
            // the `;` operator causes both statements to be joined as a
            // single statement at the same scope
            doSomething(7);
            doSomething(8)
        case 9
        case 10
        default {
            // values not `1` through `8` will execute this default scenario
            doSomething(9)
            doSomething(10)
        }
    }
}
````


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
