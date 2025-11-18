# Sharing behavior with closures

Program logic in Oxiby lives in functions, but there's another kind of function we haven't seen yet.
It's called a __closure__, or alternatively an __anonymous function__.
These functions are anonymous, because unlike the ones we've seen so far, they don't have names.
They're called closures because they "close over" their environment, which we'll see in a moment.

Closures are values, just like string, integers, booleans, etc.
They can be assigned to variables or passed as arguments to other functions.
Closures are written and used like this:

```oxiby
// File: examples/chapter_09_closures/closure.ob

fn main() {
    let closure = fn () {
        print_line("Hello from a closure!")
    }

    // Prints "Hello from a closure!"
    closure()
}
```

As we can see, closures really are just functions without names.
This closure has no parameters and returns `()` (i.e. the default return value for functions when none is specified).
Instead of being defined at the top level of the file, it's defined _inside_ the named function `main`, and it's assigned to a local variable.
Calling the closure is done by writing whatever local variable name it's bound to, followed by parenthesis, just like a regular function.
We might wonder why closures are useful if they are defined and used more or less like regular functions.
There are two main reasons.

> **Warning: Calling a closure like a function doesn't work yet.
> Instead, you must invoke the `call` method on the closure, i.e. `closure.call()`.
> The version of the above program found in the examples directory uses the currently working form.**

## Closures as function parameters

Since closures are values, they can be passed as arguments to functions.
A great example is the `map` method on the `Result` type.
Let's say we want to read in a line of text from the standard input and transform the string created into the number of characters that were in the string.

```oxiby
// File: examples/chapter_09_closures/map.ob

use std.io read_line

fn main() {
    let length = read_line().map(fn (s) {
        s.length()
    }).unwrap_or(0)

    print_line("The input length was #{length}.")
}
```

Recall that `read_line` returns a `Result` that contains either a string or an error, depending on whether the I/O succeeded or failed.
The `map` method takes a closure, which itself takes the result's `Ok` type as an argument and must return a value.
In this case, the `Ok` type is `String`.
We pass `map` a closure accepting one argument, `s`, and then return `s.length()` from the closure.
Finally, we use the `unwrap_or` method like we did previously to specify a default value if no input was read.
Previously, the default value was a string, matching the original result's `Ok` type.
But in this case, the closure we provide to `map` transforms the `String` into an `Integer`, so `unwrap_or` must provide a default integer.

Internally, `map` checks which variant the result is.
If it's `Ok` and contains a string, it will call our closure and pass it that string as an argument.
It will then return a _new_ result with the return value of our closure as its `Ok` type.
If the original result was the `Err` variant, it will simply return itself and our closure won't be called.
This allows us to specify a transformation for the `Ok` value without knowing in advance whether or not `read_line` succeeded.

We might also notice that the type of the `s` parameter of the closure is not specified as it would be with a function.
Because closures are usually used "inline" like this, and there's context from the surrounding code about how they're being used, it's okay to leave the variable types out.
They will be figured out by the compiler.
If we want to, we can specify the type of `s` using the same syntax as a regular function, `s: String`.

## Environment capture

The other reason closures are useful is that they __capture their environment__.
What this means is that they "remember" local variables that were in scope at the location they were defined, even if they are called in a different location.
To demonstrate:

```oxiby
// File: examples/chapter_09_closures/capture.ob

fn make_greeting() -> Fn() -> String {
    let name = "Oxiby"

    fn () -> String {
        "Hello, #{name}!"
    }
}

fn main() {
    let greeting = make_greeting()

    // Prints "Hello, Oxiby!"
    print_line(greeting())
}
```

The key insight here is that the body of the closure is _not executed_ until it is called on the last line of `main`.
`make_greeting` is a function that _returns another function_, and this is reflected in its signature with the multiple return types chained with `->`.
Specifically, `make_greeting` returns the type `Fn() -> String`, which is how we write the type of a closure that takes no arguments and returns a string.

We store the closure returned by `make_greeting` in a local variable just called `greeting`.
When we call the closure on the last line, it can still access `name`, a local variable that was defined at the point that the closure was defined, back in the `make_greeting` function, even though that function has already returned by the time we execute the closure.

This property of closures can be very handy when passing them as arguments to functions.

> **Warning: Calling a closure like a function doesn't work yet.
> Instead, you must invoke the `call` method on the closure, i.e. `closure.call()`.
> The version of the above program found in the examples directory uses the currently working form.**
