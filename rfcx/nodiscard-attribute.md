# NoDiscard Attribute for Functions 

FFlags: LuwuNoDiscard

## Summary

This RFC proposes a `@nodiscard` attribute, which may be applied to a function, that enforces the usage of a function's return value. 

If the return value is not used, a lint should be raised during analysis time. 

This feature aims to improve code quality by ensuring that developers do not inadvertently ignore important return values, which can lead to bugs and unintended behavior.

## Motivation

In many programming scenarios, functions return values that are critical for the correct operation of the program. Developers sometimes forget to use these return values, leading to potential issues such as unhandled errors or missed data. 

Consider the following code, which makes a network request making use of a user-implemented `Future` async primitive, which rely on having the `:await` method called on them to execute.

```luau
function fetchData(): Future<>
    return Future.new(function()
        net.request({
            url = "https://jsonplaceholder.typicode.com/todos",
            method = "POST"
            body = {
                userId = 1,
                title = "Hello, world!",
                completed = false,
            }
        })
    end)
end

-- Here, the developer forgets to use the returned `Future` by calling
-- `:await` on it, which leads to the network request never being made
fetchData()
```

The `@nodiscard` attribute will serve as a reminder and enforce the requirement to utilize the return value of a function. This will support use cases where functions return status codes, error messages, or important data that must be processed. The expected outcome is a reduction in bugs related to ignored return values and an overall improvement in code reliability.

## Design

The `@nodiscard` attribute can be applied to any function declaration. When a function with this attribute is called, the return value must be assigned to a variable or used in an expression. If the return value is not used, a `NoDiscard` lint warning will be generated.

The syntax for using the `@nodiscard` attribute is straightforward. It can be placed directly before the function declaration, just as any other function attributes:

```luau
@nodiscard
function fetchData(): Future<>
    -- ...
end

fetchData()                     -- Error; NoDiscard: Unusued return value must be used
fetchData():await()             -- Valid; Used in a chained function call
someOtherFunction(fetchData())  -- Valid; Used in an expression as function argument
local _ = fetchData()           -- Valid; Assigned to a variable
```

## Drawbacks

Introducing the `@nodiscard` attribute may lead to several drawbacks:

1. **Misuse of attribute**: Developers may mislabel functions which do not necessarily require return values to be used as `@nodiscard`, leading to false positives.
2. **Increased verbosity**: The presence of the `@nodiscard` attribute may lead to more verbose code, as developers will need to ensure they are using return values, potentially leading to additional variable assignments.
3. **Learning curve**: New developers may find the requirement to use return values cumbersome, especially if they are accustomed to languages that do not enforce such rules.

## Alternatives

Luwu could consider a user-defined attribute system, where developers can implement their own attributes, such as `@nodiscard`. However, it is more complex to implement such a feature into Luwu.