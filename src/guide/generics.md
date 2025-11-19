# Reusing code with generics

When we learned about the `Result` and `Option` types, we mentioned that these were **generic** types, but didn't fully explain what that means.
Now it's time to explore this feature in depth!

## Generic functions

We know that functions can have parameters—values they take as input that determine what they ultimately return.

```oxiby
fn identity(value: Integer) -> Integer {
    value
}
```

This simple `identity` function has a parameter for an integer, which it then returns right back to the caller.
If we wanted to use strings, we'd have to write a new function with the types changed:

```oxiby
fn string_identity(value: String) -> String {
    value
}
```

The key insight here is that the type of the parameter is not relevant to the logic inside the body of these functions.
No matter what the parameter is, the functions simply return it.

By making this function generic, we'll have a single function definition that works for _any_ type:

```oxiby
// File: examples/chapter_13_generics/identity.ob

fn identity(value: t) -> t {
    value
}

fn main() {
    let one = identity(1)
    let oxiby = identity("Oxiby")

    print_line(one)
    print_line(oxiby)
}
```

The function signature looks almost the same as before, but we're now saying that the parameter `value` is a type called `t`.
But `t` is not a **concrete type** like `Integer` or `String`.
It's a **type parameter**, in the same way that `value` is a **value parameter**.

Type names in Oxiby must start with a capital letter.
A name starting with a lowercase letter in a context where a type is expected indicates that the name refers to a type parameter rather than a concrete type.
Other than the requirement that it start with a lowercase letter, we can name a type parameter anything we want.
`t` (standing for "type") is a common type parameter name for situations where we know nothing at all about what the concrete type might be, but longer, more descriptive type parameter names are often easier to read and understand.

In our `main` function, we call `identity` twice, first with an integer, and second with a string.
We don't declare explicitly what concrete type we want to use when calling `identity`.
The Oxiby compiler uses a technique called **type inference** to figure it out based on the types of the arguments.
Sometimes there is not enough information for the compiler to figure out the concrete type.
We'll see an example of that in a moment.

## Generic types

Types we define can also be generic.
Here's a tuple struct that can hold any type inside it:

```oxiby
// File: examples/chapter_13_generics/container.ob

struct Container<t>(t)

fn main() {
    let contained_integer = Container(1)
    let contained_string = Container("Oxiby")

    print_line(contained_integer)
    print_line(contained_string)
}
```

The `t` is mentioned twice in the definition of the struct.
The first one, in angle brackets, is part of the name of the type, `Container<t>`.
The second one is the normal tuple struct syntax we've seen before, declaring `t` as the one and only unnamed field stored in the struct.

We've already seen examples of enums that can hold any type with `Result` and `Option`:

```oxiby
enum Result<t, e> {
    Ok(t),
    Err(e),
}

enum Option<t> {
    Some(t),
    None,
}
```

Here's an example that shows they are generic over the type they contain:

```oxiby
// File: examples/chapter_13_generics/no_type_annotation.ob

fn main() {
    let ok_integer = Ok(1)
    let ok_string = Ok("Oxiby")

    let err_integer = Err(-1)
    let err_string = Err("Uh oh!")

    let some_integer = Some(1)
    let some_string = Some("Oxiby")

    let none_integer = None
    let none_string = None
}
```

If we try to compile the above program, we'll get errors saying that the types of `none_integer` and `none_string` can't be determined.
`None` is the variant of `Option` where data is absent, but without context, it doesn't tell us what kind of data is absent.
In cases like this, we need to give the compiler a hint by adding type annotations to our variables:

```oxiby
// File: examples/chapter_13_generics/type_annotation.ob

fn main() {
    let none_integer: Option<Integer> = None
    let none_string: Option<String> = None
}
```

Type annotations are type names that come after the variable name, with the two separated by a colon.
The `<Integer>` syntax tells the compiler what the concrete type of the type parameter `t` is for this particular `None` value.

> **Warning: The type checker is not implemented yet, so the `no_type_annotation.ob` module will actually compile just fine.**

## What can we do with a `t`?

Generics are a powerful tool for code reuse.
With a single type parameter, we can do a lot.
A great example is the `unwrap_or` method on `Result` that we looked at way back in the chapter on reading from standard input.
Remember this?

```oxiby
let name = read_line().unwrap_or("Oxiby")
```

Now that we understand generics, let's see how `unwrap_or` is defined:

```oxiby
pub enum Result<t, e> {
    Ok(t),
    Err(e),

    pub fn unwrap_or(self, default: t) -> t {
        match self {
            Ok(t) -> t,
            Err(_) -> default,
        }
    }
}
```

`unwrap_or` takes a default value, which must be the same type as the result's `Ok` variant.
It checks which variant this particular result is using a `match` expression.
If it's the `Ok` variant, it returns the `t` inside.
If it's the `Err` variant, it returns the default `t` given as an argument.
The `_` in the `Err` match arm is a wildcard indicating that we don't care what the value inside the `Err` is.
`unwrap_or` is able to apply this same logic to any type `t` without having any idea what it actually is.

There are limits to what we can do with a completely unknown `t`, however.
Try compiling this example:

```oxiby
// File: examples/chapter_13_generics/add.ob

fn add(a: t, b: t) -> t {
    a + b
}
```

We might expect this to work fine, since all it does is add two `t`s together, but we'll get a compiler error saying that `t` cannot be added to `t`.
Why not?
How can we add two `t`s together?
We'll find out in the next chapter.

> **Warning: The type checker is not implemented yet, so the `add.ob` module will actually compile just fine.**
