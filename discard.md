
# [Zax Programming Language](index.md)

## Discard Operator

Current function-result acknowledgement, discardable results, omitted inputs,
and result routing are defined by
[Zax function invocation](language/function-invocation.md). This page remains
legacy input for local, type, and memory-policy uses of `#`.

### discard operator on previously declared variables

The discard operator (`#`) can be applied to values after declaration which will force the compiler to treat the type as unimportant and not complain if the type was not referenced.

````zax
func : ()(input : Integer) = {
    // ...
}

// ...

// replace func with a new implementation that ignores the input variable
func = {
    # input
    // ...
}
````


### discard operator on local variables

Variables that are declared but never used must be marked as discard or the compiler will generate a `variable-declared-but-not-used` warning. The compiler enforces that all declared named variables are referred to or marked with a discard operator.

````zax
print final : ()(...) = {
    // ...
}

func final : (result : Integer)() = {
    return 42
}

// Allowed since the result is captured and used elsewhere
value := func()
print(value)

// WARNING: If this variable was captured as a result but never used again
// then the compiler will issue an compiler warning as all results must have
// defined usage elsewhere.
valueNeverUsed := func()

// Allowed since the result is captured as per requirement of the
// called function using the discard operator to indicate that the value is
// knowingly being tossed out
# := func()

// ... insert code that never uses ignoredValue ...
````



### discard operator on types

Types declared with the discard operator may be constructed without ever being further referenced. This directive allows a variable of the type to be declared without further referencing the type elsewhere thus suppressing all `variable-declared-but-not-used` warnings for this type.

````zax
MyMutex :: type {
    // ...
}

MyLock # :: type {
    // ...

    +++ final : ()(lock : MyLock &) = {
        // ...
    }
}

mutex : MyMutex

// normally `lock` being unreferenced would issue an error but the type is a
// simple RAII type and thus does not need further usage beyond merely
// constructing the value
lock : MyLock = mutex

// ... code which never referenced `lock` variable ...
````
