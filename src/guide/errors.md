# Handling errors with `Result` and `match`

In the last chapter, we saw this construct:

```oxiby
read_line().unwrap_or("Oxiby")
```

As explained, this calls the fallible function `read_line` and ensures that we always end up with a string by providing a default string to use in the case of error. But what is an error, exactly?

Here's the signature of `read_line`:

```oxiby
fn read_line() -> Result<String, String>
```

`read_line` returns a value of the type `Result`.
`Result` is a __generic type__, which means it can take different forms depending on the types it's used with.
It's also a compound type called an __enum__ (short for __enumeration__), which is one of the two user-definable types in Oxiby.
We'll explore generics more in a future chapter.
For now, let's focus on what an enum is and why it's useful here.

## Enums

An enum is a type that can have multiple **variants**.
Each variant is used to construct a value of the same type, but each can hold different data.
Ignoring generics for now, we can imagine `read_line`'s return type as being defined like this:

```oxiby
enum Result {
    Ok(String),
    Err(String),
}
```

This means that a value of type `Result` has two possible variants: `Ok`, which holds a string, and `Err`, which also holds a string.

But if both variants hold a string, why is an enum necessary?
Why not just use a string?
The reason is because the variant itself implies different meaning for its contained string.
`Ok`'s string is the input we receive from the standard input.
`Err`'s string is a message with details about why the attempt to read a line from the standard input failed.

An enum is constructed by calling one of its variants like a function.
We write the name of the enum and the name of the variant separated by the `.` operator, followed by a value of the type it holds in parentheses, like this:

```oxiby
Result.Ok("It worked!")
```

Because `Result` is such a commonly used type in Oxiby, its variants are also available without the enum prefix, so we should write:

```oxiby
Ok("It worked!")
```

## The `match` expression

Sometimes enums will provide methods like `unwrap_or` that handle specific use cases, but in the general case, we need a way to know which variant an enum value is in order to do anything useful with it.
The mechanism for this is the `match` expression:

```oxiby
// File: examples/chapter_05_errors/hello_world.ob

use std.io read_line

fn main() {
    print("> ")

    match read_line() {
        Ok(line) -> print_line(line),
        Err(reason) -> print_line("Error: #{reason}"),
    }
}
```

This will have two possible outcomes:

1. If `read_line` returns `Result.Ok` with the line of user input we wanted, we print it.
2. If it returns `Result.Err` with an error message, we print the error message, prefixed with `"Error: "`.

Try running the above program.
When typing some text and pressing return, the typed text is printed right back.
When pressing `Ctrl + D` to close the standard input, the program prints:

```
Error: no input was received
```

Let's remove the specifics of this `match` example to understand the form of a `match` expression:

```oxiby
match expression {
    pattern -> expression,
    pattern -> expression,
}
```

`match` takes an expression to compare against, then a pair of curly braces.
Inside the curly braces are one or more **match arms**.
Each arm is a pattern, followed by an arrow, followed by an expression.
When a pattern matches, the corresponding expression on the right side of the arrow is evaluated.
Oxiby will walk through the patterns in the order they're defined, selecting the first pattern to match.

Patterns can contain variables, like `line` and `reason` in the `read_line` example above.
We can think of these as *conditional* variable assignments.
The strings inside each respective variant are bound to our variables, but only if the result matches the structure of the pattern.
In this way, patterns can be used to "unpack" enums, extracting the data held inside.
This unpacking is called **destructuring**.

The entire `match` expression ultimately evaluates to whichever match arm was selected, and so like conditional expressions, the value that `match` produces can be assigned to a variable or returned from a function:

```oxiby
fn read_line_wrapper() -> String {
    match read_line {
        Ok(line) -> line,
        Err(reason) -> reason,
    }
}
```

`match` matches exhaustively.
This means that the compiler requires that every possible case be covered.
For example, if we were to leave out the `Err` pattern in the example above, our Oxiby program would not compile, because we did not tell it what to do with a `Result` in the case that it's an `Err`:

```oxiby
match read_line {
    Ok(line) -> line,
    // Oops, we forgot to check for `Err`!
}
```

> **Warning: Non-exhaustive matches are not yet enforced at compile time.
> Failure to account for all possibilities might result in a runtime crash.**
