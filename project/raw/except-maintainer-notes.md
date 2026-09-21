
# except-maintainer-notes.md

Old legacy `except` in `except.md` has some good concepts but some muddy early thinking and is in need of corrective work. It can be mined for ideas, but it must not be treated as authoritative over this `except-maintainer-notes.md` file. This file attempts to remedy the mistakes in the original legacy work.


## `except` keyword mental model

The `except` keyword is not related to C++ style exception handling. The `except` keyword helps capture and route exceptional failure cases using normal flow control methods rather than attempting stack unwind or forcing a programmer to deal with potential mid-setup failure flow controls. Even though some of the concepts used in `except` resemble exceptions they are all based entirely on traditional flow controls, thus C++ style exceptions and catch handling are not part of Zax; yet Zax has first class modeling around error conditions.

### `except` as a capture

Consider:
```
myFunc final : (result : Integer, failure : MyError)(input : String) = {
  //...
}

myResult := myFunc("hello") except failure
```

In the above example `myResult` will contain a value, but the result called `failure` is separately captured by name in the `except` clause. Both values are returned, but one of the values likely has a default value and the other is likely set. In the above example, it's difficult to know what exactly happened normal or via a failure case. Also nothing prevented `myFailure := myFunc("hello") except result` since the except captures and extracts out the value by name from the normal call flow. The function does return two results, the except just pulls one or more out of the normal result pattern.

#### `capture` mapping

Consider:
```
myFunc final : (result : Integer, failure : MyError)(input : String) = {
  //...
}

myResult := myFunc("hello") except failure: myFailure:

if ?myFailure
   myFailure.print()
```

The except captures the value, and introduces it to the scope. It specifically takes it outside the expression and allows the value to live. `source:` `destination:` style name mapping applies, if the functional result name isn't the desired introduction name into the scope.

Consider:
```
myFunc final : (result : Integer, failure : MyError)(input : String) = {
  //...
}

newResult := 5 * (myFunc("hello") except failure: myFailure:)

if ?myFailure
   myFailure.print()
```

Even as an expression, the `myFailure` is introduced into the scope. This is a new variable, and it continues to outside of the expression using the type returned in the function call. Once the variable is resolved, it can be used in the expression elsewhere:

Consider:
```
myFunc final : (result : Integer, failure : MyError)(input : String) = {
  //...
}

newResult := 5 * (myFunc("hello") except failure: myFailure:) + myFailure.count()

if ?myFailure
   myFailure.print()
```

As you can observe, it can be used for scenarios that aren't exceptions to pull out multiple return results into their own captured values but that's not the intended use case.

#### Multiple except

Consider:
```
myFunc final : (
  result : Integer,
  failure : MyError,
  anotherFailure : MyOtherError
)(
  input : String
) = {
  //...
}

myResult := myFunc("hello") except failure: myFailure: except anotherFailure

if ?myFailure
   myFailure.print()

if ?anotherFailure
   anotherFailure.print()
```

Each `except` is listed one by one with its own `except` clause. While it might seem that `,` would be more appropriate, it is not the accepted form for `except` due to the way `catch` works later.


### `except` result tagging and `catch`

Putting an `except` tag on a return result requires the result to support a `?` or `!` check which must return exactly a `Boolean` result.

The old legacy model used `as` with a `Boolean` and that is no longer the model to follow. Deem that model obsoleted.

But the `?` Boolean result is not typical where `?` means "good condition" if `true`. Here a `?` returning a `Boolean` of `true` indicates there is an except condition in the return result. This allows both optionals and pointers to be used for `except` and have meaningful intent. Thus `MyError?` or `MyError * unique` could be used as return results, and when they hold values as present (not vacant or absent) that indicates the `except` condition did indeed occur. As a backup `!` which returns `Boolean` is also a fallback for a type if `?` is not supported. In that case a `!` returning `false` indicates that an `except` condition happened.

Consider:
```
myFunc final : (
  result : Integer,
  failure except : MyError?     // `except` is on the variable, not the type
)(
  input : String
) = {
  //...
}

myResult := myFunc("hello") catch failure {
  //...
  failure.print()
}
```

In the above example, the `catch` clause is executed if the `?failure` resolves to `true`, in which case the body of the catch is executed.

NOTE: order of execution is not specified yet; a recommendation is desired. Should the return results bind first, or the `catch` be tested and called first?

#### multiple `catch` clauses

Consider:
```
myFunc final : (
  result : Integer,
  failure except : MyError?     // `except` is on the variable, not the type
  anotherFailure except : Integer?
)(
  input : String
) = {
  //...
}

myResult := myFunc("hello") catch failure {
  //...
  failure.print()
} catch anotherFailure: myFailureValue : HttpFailureCode {
  if myFailureValue.isGlobalFailure()
    abandonRetry()
  anotherFailure.print()    // error: `anotherFailure` was not introduced into this scope!
}
```

The order of the `catch` evaluation is based on the order of the declaration. If the `catch` issues a `break` or `continue` or `return` flow control inside, that is completely legal. However, if the order is decided that the result returned by the original function that had a `catch` then those values cannot be assumed to be constructed thus a scope must exist one scope outer (or greater) to resume normal flow.

#### except to end function call early

Consider:
```
myFunc final : (
  result : Integer,
  failure except : MyError?
  anotherFailure except : MyCustomHttpError
)(
  input : String
) = {
  if input == "general-failure" {
    preparedError : MyError = "Service is down. Try again later.".
    except failure: preparedError
  }
  if input == "not-found"
    except anotherFailure: : Integer = 404

  return randomNumber()
}
```

The `except failure:` maps the right hand side value into that slot.

The `except anotherFailure: : Integer = 404` causes the `Integer` to be passed into the constructor of `MyCustomHttpError`.

In the cases where `except` doesn't happen the default constructors are applied to the `except` values. This must be true so that the `catch` clause can test the instances returned from the function and execute the `catch` blocks.

`return` does not include `except` result values on the `return` result

#### `catch` with `!`

The original idea is that `catch` with `!` declaration would flip to `Boolean` test to determine `catch`, thus a `true` is tested as `false` and vice versa. This would work for `?value` tests that returned the wrong `Boolean` state to be compatible with normal `except`. I'm just not sure the syntax recommendation if we are to support this idea.


### `except` diagnostic

All except errors must be caught unless they are flagged to be discarded. There are cases where valid return results are returned from a function and an `except` occurs where the proper value can be used and the `except` is indeed discarded. The same discard mechanism applies here.


## Other considerations

The discard `#` logic in the old legacy is highly suspect. I suggest treating that with caution.
