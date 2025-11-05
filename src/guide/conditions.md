# Controlling flow with conditions

Our program always says "hello" to the person we're greeting.
What if we want to change our greeting depending on who is greeted?
We can do that with conditional expressions.

```oxiby
// File: examples/chapter_03_conditions/hello_world.ob

fn greet(name: String) -> String {
    let salutation = if name == "Oxiby" {
        "Greetings"
    } else {
        "Hello"
    }

    "#{salutation}, #{name}!"
}

fn main() {
    let greeting = greet("Oxiby")

    print_line(greeting)
}
```


Use the Oxiby compiler to run the program and see its ouput:

```
$ obc run hello_world.ob
Greetings, Oxiby!
```

This update to our program introduces two important tools for programming in Oxiby.

## Operations

```oxiby
name == "Oxiby"
```

Operations are expressions for common mathematical operations like addition, subtraction, and equality.
In our program we're using the equality comparison to check if the string bound to the variable `name` is `"Oxiby"`.

Operations are sort of like function calls, but use __infix__, __prefix__, or __postfix__ notation.

Equality uses `==` with infix notation, where the two operands appear on either side of the operation.
Conceptually, we can think of these two forms as equivalent:

```oxiby
name == "Oxiby"
equal(name, "Oxiby")
```

Comparison operations like equality evaluate to boolean values, i.e. either `true` or `false`.

An example of prefix notation is the unary operator `!` which inverts a boolean value.
The following two lines both evaluate to `true`:

```oxiby
true
!false
```

An example of postfix notation is using square brackets to index a list.
(Don't worry about what indexing a list means yet.)

```oxiby
list[0]
```

## Conditional expressions

Conditional expressions allow our logic to __branch__ and do different things based on a comparison.

```oxiby
if name == "Oxiby" {
    "Greetings"
} else {
    "Hello"
}
```

This checks whether `"Oxiby"` is bound to the variable `name`.
If it is, the code inside the first set of curly braces is evaluated, producing `"Greetings"`.
If it isn't, the code inside the second set of curly braces is evaluated, producing `"Hello"`.

In our example program, we're assinging the conditional expression to a variable.

```oxiby
let salutation = if name == "Oxiby" {
    "Greetings"
} else {
    "Hello"
}
```

This illustrates that the entire conditional expression itself evaluates to a value: the last expression of the block that was chosen.
Assigning this value to a variable is not required.
We'll often seen conditionals used for their side effects.
For example, our entire program could be rewritten inside `main` like this:

```oxiby
fn main() {
    let name = "Oxiby"

    if name == "Oxiby" {
        print_line("Hello, me!")
    } else {
        print_line("Hello, #{name}!")
    }
}
```

Conditionals can also be chained as many times as needed if there are more than two logical branches:

```oxiby
if country == "Norway" {
    "Greetings to the Norwegians!"
} else if country == "Egypt" {
    "Greetings to the Egyptians!"
} else {
    "Greetings to everyone!"
}
```

Finally, the `else` branch can be omitted if there is nothing to do when the condition doesn't match:

```oxiby
if name == "Oxiby" {
    "It's me!"
}
```
