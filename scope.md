
# [Zax Programming Language](index.md)

## Scope

Scopes can be used to help flow control by breaking or continuing out of a nested inner code flow to an outer code flow.

Accepted block formation, flow-body extent, and the source-visible clause scope
are defined by [Zax source structure](language/source-structure.md). The detailed
binding visibility, same-scope redeclaration, shadow permission, explicit member
access, and qualified lookup model are defined by
[Zax declarations and bindings](language/declarations-and-bindings.md).

Core unnamed, named, and anonymous scope exits, `break`/`continue`/`next`
targeting, and flow labels are now owned by
[Zax core flow control](language/core-flow-control.md); scope-exit destruction is
owned by
[Zax construction, replacement, and destruction](language/construction-and-destruction.md).

The remaining section below is the **legacy remainder** of this page: unreviewed
callable-like `scope` variable capture. It is non-authoritative until its own
focused review.


### `scope` variable capture

Similar to how [functions](functions.md#functions-can-capture-reference) can capture variables, scopes can capture variable too. Capturing variables in scopes follow all the same capture rules as would function scopes. This methodology can be useful for ensuring only captured variables are accessed and preventing other local scope variables from being visible inside the scope. This `scope` forms creates a light weight function isolation pattern and has all of the flow control rules of a normal `scope`.

````zax
func final : ()() = {
    myValue1 : Integer
    myValue2 : String

    // ...

    scope [myValue1] {
        
        // ...

        myValue1 *= 3       // will compile

        // ERROR: `myValue2` is not captured inside this scope
        ++myValue2

        // ...
    }
}
````


#### Using `scope` capture as return values

A `scope` is not a function, but it can be treated as a lightweight function that can both accept inputs and return outputs. Return results can be declared and captured and inputs can be captured by-value (or by reference).

The example below demonstrates a `scope` being treated as a function that accepts `myValue1` as input passed by-value and `output` is treated as a return result by being declared outside the `scope` and captured by reference.

````zax
func final : ()() = {
    myValue1 : Integer
    myValue2 : String

    // ...

    output : Integer; scope [&output, myValue1] {
        
        // ...

        myValue1 *= 3       // will compile

        // treat captured value by reference as 
        output = myValue1

        // ERROR: `myValue2` is not captured inside this scope
        ++myValue2
    }
}
````
