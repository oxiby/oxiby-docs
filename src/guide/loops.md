# Iterating through collections

Our program currently greets whomever we specify on the command line.
What if we want to be able to greet multiple people at once?
We can achieve that with lists and loops.

```oxiby
// File: examples/chapter_06_loops/hello_world.ob

use std.io read_line

fn greet() -> List<String> {
    print("> ")

    let names = read_line().unwrap_or("Oxiby").trim_end().split(" ")
    let greetings = []

    for name in names {
        greetings.push("Hello, #{name}!")
    }

    greetings
}

fn main() {
    let greetings = greet()

    for greeting in greetings {
        print_line(greeting)
    }
}
```

Run the program and enter multiple names separated by spaces when the program pauses for input:

```
$ obc run hello_world.ob
> Oxiby Rust Ruby
Hello, Oxiby!
Hello, Rust!
Hello, Ruby!
```

Let's take a look at the new ideas introduced here.

## Lists

Lists are a core type that represent an ordered sequence of elements.
Every element in a list must be the same type, so we can't have a list containing integers and strings, for example.
A list type is written with the word `List` followed by the type of the element in the list in angle brackets.
Previously, our `greet` function returned a string, but now it returns a list of strings:

```oxiby
fn greet() -> List<String>
```

A list is created with a pair of square brackets.
In our program, we initialize the variable `greetings` as an empty list, which we will populate with elements later:

```oxiby
let greetings = []
```

Here's an example of creating a list with some initial elements:

```oxiby
let fruits = ["apple", "banana", "carrot"]
```

To access a specific item in a list, we use square brackets to perform an **indexing** operation:

```oxiby
fruits[0]
```

The number in the square brackets is a numeric offset from the beginning of the list, so 0 would be the first element of the list, 1 the second, and so on.

In our program, we're creating the list of names to iterate over using the `split` method on the `String` type:

```oxiby
read_line().unwrap_or("Oxiby").trim_end().split(" ")
```

Remember that `read_line().unwrap_or("Oxiby").trim_end()` returns a string.
If the string the user provided contains multiple words separated by some delimiter, like `"Rust Ruby Oxiby"`, `split` will break these words apart into a list of multiple strings.
We pass `" "` to `split` to specify a single empty space as the delimiter between words.
If there are no spaces in the input, `split` will return a list containing only one element, the original string.

## `for` loops

```oxiby
for name in names {
    greetings.push("Hello, #{name}!")
}
```

`for` loops allow us to iterate through the elements in a list and run some code for each element.
These loops use the syntax `for ITEM in ITEMS`, where `ITEMS` is a list and `ITEM` is a variable name we choose.
A block of code in curly braces follows.
The block will be executed once for each item in the list.
Each time the block is executed, the next item in the list will be bound to the variable name we chose.
After the block has been executed for each item in the list, the loop ends and the program proceeds.

In our program, we iterate through all the names the user provided to us on the command line and add them to our `greetings` list by using the `push` method on the `List` type, which "pushes" the element onto the end of the list, after any elements that are already in it.

Back in our `main` function, we take the list of strings returned by our `greet` function and use another `for` loop to print each one back to the user:

```oxiby
for greeting in greetings {
    print_line(greeting)
}
```
What if we want to skip elements in a list or stop iterating a list before we reach the end?
That can be achieved with the `continue` and `break` keywords, respectively:

```oxiby
// File: examples/chapter_06_loops/continue_break.ob

fn main() {
    let fruits = ["apple", "banana", "carrot"]

    for fruit in fruits {
        if fruit == "apple" {
            continue
        }

        print_line(fruit)

        if fruit == "banana" {
            break
        }
    }
}
```

`continue` means "immediately proceed to the next iteration of the loop".
`break` means "immediately stop iterating the loop".
In this example, when the loop iterates on "apple", we continue to the next iteration before printing anything.
When the loop iterates on "banana", we print it and then break out of the loop before proceeding to "carrot".
As a result, "banana" is the only one of the three fruits that is printed.

## `while` loops

`for` loops are the right choice when we need to iterate through a list, but there are times when we might want to execute a block of code multiple times without a list.
There's a more general kind of loop called a `while` loop that we would use in this case.

```oxiby
// File: examples/chapter_06_loops/while_loop.ob

fn main() {
    let i = 1

    while i <= 10 {
        print_line(i)

        i = i + 1
    }
}
```

A `while` loop takes a conditional expression (an expression that evaluates to a boolean value) and a block of code.
It starts by evaluating the condition.
If it's `true`, the code in the block is executed and then the condition is evaluated again.
If it's `false`, the loop ends and the program continues.

In this example, we print the integers from 1 to 10.
We start with a variable `i` bound to 1, then check that `i` is less than or equal to 10.
If it is, we print it and increment the value of `i` by 1.
The condition will then run again against the new value of `i`.
This continues until `i` reaches 11, at which point our loop ends without printing the number.

## Indefinite loops

Notice that the `while` loop always evaluates the condition before the block executes, even the first time.
If we want to always execute the code in the block at least once without checking a condition, we can use `loop` and break out of it explicitly.
The same logic as our `while` loop example can be expressed this way:

```oxiby
// File: examples/chapter_06_loops/indefinite_loop.ob

fn main() {
    let i = 1

    loop {
        print_line(i)

        i = i + 1

        if i > 10 {
            break
        }
    }
}
```

Be careful with terminating conditions when using loops.
If the condition never evaluates to `true`, the loop will be infinite and the program will hang!
