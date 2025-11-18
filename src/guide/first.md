# Writing your first Oxiby program

As is tradition, we'll begin learning Oxiby with the ["Hello, World!"](https://en.wikipedia.org/wiki/%22Hello,_World!%22_program) program.

Create an empty directory to work on the programs in this guide.
Inside this new directory, create a new file called `hello_world.ob`.
Enter the following text into the file:

```oxiby
// File: examples/chapter_01_first/hello_world.ob

// Welcome to Oxiby!
fn main() {
    print_line("Hello, World!")
}
```

Use the Oxiby compiler to run the program and see its output:

```
$ obc run hello_world.ob
Hello, World!
```

Remember this command because we'll use it a lot.

This simple program teaches us a few things about Oxiby.

## Comments

Two slashes (`//`) begin a comment. Any text afterwards on the same line will be ignored by the compiler and removed from the Ruby source code produced.

## String literals

```oxiby
"Hello, World!"
```

Strings, sequences of textual characters, are created by writing the text inside a pair of double quotes.

"Literal" just means that there is dedicated syntax in the language to construct a string.
There are other kinds of literals for different types in the language, such as integers (e.g. `123`) and [boolean](https://en.wikipedia.org/wiki/Boolean_data_type) values (i.e. `true` and `false`).

## Functions

```oxiby
fn example() {
    expression
}
```

Functions are declared with the keyword `fn`, followed by a function name, a pair of parentheses, and a function body inside curly braces.
The body is a sequence of zero or more expressions.
An expression is anything that can be reduced to a single value.

In our program, `main` has a body consisting of one expression.

## Calling functions

```oxiby
print_line("Hello, World!")
```

A function is called by writing its name followed by any function arguments in parentheses.
An argument is input to the function that changes how it behaves.

## Writing to stdout

The function `print_line` is used to write text to the standard output.

This is what causes "Hello, World!" to appear in the terminal when we run our program.

## The `main` function

When the program runs, if a function named `main` is defined, it will be automatically executed.
