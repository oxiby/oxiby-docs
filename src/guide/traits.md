# Defining common behavior with traits

In the last chapter, we tried to define a generic function that adds two values together:

```oxiby
// File: examples/chapter_13_generics/add.ob

fn add(a: t, b: t) -> t {
    a + b
}
```

This doesn't work because when we have a generic type like `t` that could be _any_ type, we can't assume anything about what it can do.
We can add numbers together with `+` but that isn't true for all types.
For example, what would it mean to add two boolean values together?

If we want to use specific operators or methods with our generic types, we need to **constrain** the type parameters with **traits**.

## Traits

A **trait** is a set of functions that multiple types can have in common.
In other languages you might hear the same concept described as a **type class**.
Traits are also similar to the concept of **interfaces** in other languages.
It's a way to define common behavior across types.

Here's an example of a trait from the standard library:

```oxiby
trait ToString {
    fn to_string(self) -> String
}
```

A trait is defined with the keyword `trait` followed by a name in capital letters (like a type name, but generally phrased as a verb) and then function signatures inside curly braces.
Note that the function `to_string` has no body.
That's because it's up to the types that **implement** the trait to define how the function behaves for that type.

Let's implement the `ToString` trait for the `Person` type we've used previously:

```oxiby
// File: examples/chapter_14_traits/person_to_string.ob

struct Person {
    name: String,
    age: Integer,
}

impl ToString for Person {
    fn to_string(self) -> String {
        "#{self.name} is #{self.age} years old."
    }
}

fn main() {
    let person = Person {
        name: "Alice",
        age: 42,
    }

    // Prints "Alice is 42 years old."
    print_line(person)
}
```

`Person` is the same as we've seen before: a struct with a string name and an integer age.
We **implement** the `ToString` trait for `Person` with the `impl` keyword followed by the trait name, the `for` keyword, and then the type we're implementing the trait for, `Person`.
Inside the curly braces, we define each function from the trait, but this time we provide a function body that determines how to create a `String` given a `Person`.

In the `main` function, we create a person and then pass it to the `print_line` function.
Notice that we don't actually call the `to_string` method anywhere.
Why not?
What was the point of implementing the trait if we don't use it?
To understand that, we need to look at the signature of the `print_line` function:

```oxiby
fn print_line(s: s) where s: ToString
```

There's also something new after the return type:
There's some new syntax we haven't seen before after the parameters.
The `where` keyword followed by some type shenanigans.
This is called a "where clause" and it lists **constraints** on the types in the signature.

We know that a type starting with a lowercase letter is actually a type parameter, and that the parameter `s: s` means that the function accepts a value of some type `s` that will be bound to the variable `s`.
Don't be confused about the repetition of the letter here.
It's just how this particular function is defined.
The parameter could just as easily be something like `string: t` or any other combination of variable names and it would work the same way.

But what is the meaning of the `s: ToString` in the where clause?
This means that the type parameter `s` cannot be a value of _any_ type.
It must be a value of a type that implements the `ToString` trait.
We say that `s` is **constrained** to a type that implements `ToString` or that "`s` must be `ToString`."

Because `print_line` doesn't know anything about `s` other than that it implements `ToString`, it can't do much with it other than call the `to_string` method.
But that's okay, because that's all `print_line` needs.
In its implementation, it calls `to_string` on the given argument, and prints the resulting string to the standard output.

> **Warning: All user-defined types can be printed with `print_line` regardless of whether or not they implement `ToString`.
> Implementing `ToString` will override the default formatting.**

## Operators as traits

Let's finally bring this back around to our `add` function:

```oxiby
// File: examples/chapter_13_generics/add.ob

fn add(a: t, b: t) -> t {
    a + b
}
```

We know now that we can't use methods unless we know that the types involved implement those methods.
We can't add two `t`s together if we don't know that they support addition.
In Oxiby, operators are implemented as traits.
If we specify that `t` must be `Add`, our function will work as we orignally expect:

```oxiby
use std.ops Add

fn add(a: t, b: t) -> t where t: Add {
    a + b
}
```

Now `add` will work with any type, as long as that type is `Add`.
This is the basis for [operator overloading](https://en.wikipedia.org/wiki/Operator_overloading) in Oxiby, a form of [ad-hoc polymorphism](https://en.wikipedia.org/wiki/Ad_hoc_polymorphism) that allows us to use the built-in operators like `+` with our own types.

## Implementing traits

Let's use this knowledge to implement `Add` for the `ShoppingList` type we made previously.
To do that, we'll need to look at how `Add` is defined:

```oxiby
pub trait Add<rhs> where rhs = Self {
    type Output

    fn add(self, other: rhs) -> Self.Output
}
```

This uses more syntax we haven't seen before.
The where clause specifies a type parameter named `rhs` (short for "right-hand side") and it's followed by `= Self`.
This means that when a type implements `Add`, it doesn't need to specify the concrete type of `rhs` as would normally be required.
If it doesn't, `rhs` is assumed to be the same type as the type implementing the trait.
In our case, that will mean that `rhs` will be `ShoppingList`, because we're adding two shopping lists together.

The second part of the where clause specifies a type `Self.Output`.
This is called an **associated type**.
It's a generic type that the trait implementation must specify.
We'll see how it's different from a regular generic type like `rhs` in a moment.
For now, let's implement `Add` for `ShoppingList`, with a quick recap of how `ShoppingList` is defined:

```oxiby
// File: examples/chapter_14_traits/shopping_list_add.ob

use std.ops Add

struct ShoppingList {
    map: HashMap<String, Integer>,

    fn new() -> Self {
        Self {
            map: HashMap.new(),
        }
    }

}

impl Add for ShoppingList {
    type output = Self

    fn add(self, other: Self) -> Self.Output {
        let combined_map = HashMap.new()

        for (key, value) in self.map {
            combined_map[key] = value
        }

        for (key, value) in other.map {
            combined_map[key] = combined_map[key].unwrap_or(0) + value
        }

        ShoppingList {
            map: combined_map,
        }
    }
}

fn main() {
    let shopping_list_a = ShoppingList.new()
    shopping_list_a.map["apple"] = 3
    shopping_list_a.map["carrot"] = 1

    let shopping_list_b = ShoppingList.new()
    shopping_list_b.map["banana"] = 1
    shopping_list_b.map["carrot"] = 1

    let final_shopping_list = shopping_list_a + shopping_list_b

    print(final_shopping_list)
}
```

When implementing `Add` for `ShoppingList`, we specify `Self.Output`, the type that the `add` function will return, as `Self`.
As noted, because the type parameter `rhs` defaults to `Self`, which is the type we want in this case, we don't need to specify its concrete type in the implementation.
If it were required, the first line of the implementation would read `impl Add<ShoppingList> for ShoppingList` but it would mean the same thing.
Essentially this definition says, "When you add a `ShoppingList` to another `ShoppingList`, you get a `ShoppingList` back."

The implementation of the `add` method creates an empty hash map, copies over each key/value pair from `self` (which is the `ShoppingList` on the left-hand side of the addition operation), and then copies over each key/value pair from `other` (the `ShoppingList` on the right-hand side), taking care to add quantities to existing ones if there was already a value for a particular key in the hash map.
Finally, a new `ShoppingList` with the new hash map is constructed and returned.

> **Warning: The `Add` trait is not yet mapped to the `+` operator.
> The version of the above program found in the examples directory uses a method `add` on `ShoppingList` instead to account for this.**

> **Warning: Currently, a static function named `new` has special meaning and cannot be used for constructors.
> The version of the above program found in the examples directory uses `create` instead to account for this.**

> **Warning: The `Self` type alias is not yet available.
> The version of the above program found in the examples directory uses `ShoppingList` instead to account for this.**

## Multiple implementations of the same trait

Because `rhs` is a type parameter and we used the default type `Self` in our implementation, we can only add `ShoppingList`s to other `ShoppingList`s.
If we wanted to allow other types to be added to a `ShoppingList`, we'd need an additional implementation of the trait.
Perhaps a useful one would be `(String, Integer)`, a tuple of two values that match the key/value relationship inside `ShoppingList`.

```oxiby
// File: examples/chapter_14_traits/shopping_list_add_tuple.ob

use std.ops Add

struct ShoppingList {
    map: HashMap<String, Integer>,

    fn new() -> Self {
        Self {
            map: HashMap.new(),
        }
    }

}

impl Add<(String, Integer)> for ShoppingList {
    type output = Self

    fn add(self, other: (String, Integer)) -> Self.Output {
        let combined_map = HashMap.new()

        for (key, value) in self.map {
            combined_map[key] = value
        }

        combined_map[other.0] = combined_map[other.0].unwrap_or(0) + other.1

        ShoppingList {
            map: combined_map,
        }
    }
}

fn main() {
    let shopping_list = ShoppingList.new()
    shopping_list.map["apple"] = 3
    shopping_list.map["carrot"] = 1

    shopping_list = shopping_list + ("banana", 1) + ("carrot", 1)

    print_line(shopping_list)
}
```

This implementation of the trait is similar, but we specify `rhs` explicitly with `trait Add<(String, Integer)>`, because in this case, `rhs` is a `(String, Integer)` rather than a `ShoppingList`.
In the method body, we copy over just the one key/value pair represented by the tuple into the new hash map.

With these two versions of the trait implemented, we can now add to `ShoppingList`s in two different ways!

This illustrates the difference between the generic type `rhs` and the associated type `Self.Output`.
The concrete type of `rhs` for a given usage is determined by the calling code.
When the expression on the right-hand side of the `+` is another shopping list, `rhs = ShoppingList`.
When the expression on the right-hand side of the `+` is a tuple, `rhs = (String, Integer)`.

In contrast, the concrete type of `Self.Output` is determined by the trait implementation.
The calling code can't change the fact that when two `ShoppingList`s are added, a `Self.Output = ShoppingList` and a new `ShoppingList` is produced.
Likewise, the calling code can't change the fact that adding a `(String, Integer)` to a `ShoppingList` produces a new `ShoppingList` as well.

In our case, `Self.Output` is just `Self`, so adding something to a `ShoppingList` produces a new `ShoppingList`.
However, this `Output` associated type allows for the flexibility of the result of the operation being a different type than its operands.

> **Warning: The type checker is not implemented yet, so adding a second implementation of a trait for a type will overwrite the previous one.
As such, the two example programs in this chapter cannot currently be combined into one.**

> **Warning: The `Add` trait is not yet mapped to the `+` operator.
> The version of the above program found in the examples directory uses a method `add` on `ShoppingList` instead to account for this.**

> **Warning: Currently, a static function named `new` has special meaning and cannot be used for constructors.
> The version of the above program found in the examples directory uses `create` instead to account for this.**

> **Warning: The `Self` type alias is not yet available.
> The version of the above program found in the examples directory uses `ShoppingList` instead to account for this.**

