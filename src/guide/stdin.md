# Reading from standard input

While our greeting using conditionals could do different things based on who we were greeting, the same branch of code was executed every time because the input to the program never changed.
Let's change the program to greet whomever is named by the user.

```oxiby
// File: examples/chapter_04_stdin/hello_world.ob

use std.io read_line

fn greet() -> String {
    print("> ")

    let name = read_line().unwrap_or("Oxiby").trim_end()

    "Hello, #{name}!"
}

fn main() {
    let greeting = greet()

    print_line(greeting)
}
```

Run the program as usual:

```
$ obc run hello_world.ob
```

Notice that only a `>` appears but no greeting is output yet.
The program waits for input from us before it can proceed.
Type your name and hit return to let the program continue and greet you.

```plain
$ obc run hello_world.ob
> Al Dente
Hello, Al Dente!
```

A few new concepts are introduced here.

## Imports

```oxiby
use path.to.module function_to_import
```

Functions can be imported from other modules.
A module is just a file containing Oxiby source code.
A function is imported with the keyword `use`, followed by the path to a module, followed by the name of the function to import.

Modules beginning with the path segment `std` are the standard library, a special collection of modules provided by Oxiby.

## The `print` function

`print` is a function just like the `print_line` function we've been using, but it doesn't put a line break at the end of the given string.
We use this to indicate a prompt for user input, allowing the user to type on the same line that the `>` appears.

## Reading from standard input

```oxiby
let input = read_line()
```

A line of text can be read from the standard input with the `read_line` function from the `std.io` module.
This allows our program to change behavior each time it's run depending on different inputs.

`read_line` blocks execution of the program until it receives a line from the standard input or until the standard input is closed.

## Methods

```oxiby
value.method()
```

A method is a function that "belongs" to a type.
Up until now, we had only called "free" functions, which look like this:

```oxiby
example()
```

But with a method, the function is called in the context of a specific value.
This is done by by writing the value, followed by a dot, followed by the usual function call syntax.
The method receives the value as its first argument.
We can think of these two forms as equivalent:

```oxiby
function(value)
value.function()
```

The benefit of methods is that they allow us to chain together sequences of function calls in a natural way.

In our program, the function `read_line` returns a value, and we use the `.` operator to call the `unwrap_or` method on that value.
Similarly, the `unwrap_or` method returns a value, and again we use the `.` operator to call the `trim_end` method on that value.

## Fallible functions

```oxiby
read_line.unwrap_or("default")
```

Reading from the standard input involves an I/O operation which might fail, so we need to handle the case where we don't get a string back from the `read_line` call.
This is achieved by using the `unwrap_or` method to provide a default value to use in the case of error.

We can see this in action by running the program and pressing `Ctrl + D` (holding down the control key while pressing the D key), rather than typing a name.
This closes the standard input and will cause the program to return the default value we provided to `unwrap_or`.

`unwrap_or` uses a "functional" style to transform the potential error into this default value.
We don't deal with the value that `read_line` returns directly.
Next, we'll learn how to handle errors using an "imperative" style that will help us understand what `read_line` actually returns.
