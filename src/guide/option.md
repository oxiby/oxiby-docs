# Accounting for missing data with Option

Many programming languages have a construct that represents the absence of data, often called null or nil.
This is used as a placeholder for things that contain data under certain conditions, but don't always.
Oxiby doesn't have a dedicated type for this.
Instead, it uses an enum named `Option` to represent a value that might be present or might not.
Let's look at the definition of the type.

```oxiby
enum Option<t> {
    Some(t),
    None,
}
```

We learned about enums previously when discussing the `Result` type.
Recall that `Result` is a similar enum that represents either the success or failure of a computation.
In `Result`'s case, both enum variants, `Ok` and `Err` contain a value that gives more information: either the value that was produced by a successful computation, or details about an error that occurred.

`Option` is similar in that it has two variants, but the second case, `None`, does not hold any additional data, because the variant represents a lack of data entirely.

We still haven't talked in detail about generics, but for now, note that `Option` can hold any type of value inside it.
The `<t>` is used to indicate this.
Here are some examples of constructing `Option`s:

```oxiby
let some_string = Option.Some("Oxiby")
let some_integer = Option.Some(123)
let none = Option.None
```

Like `Result`, because `Option` is so widely used, its variants are available to use directly, so we should write this instead:

```oxiby
let some_string = Some("Oxiby")
let some_integer = Some(123)
let none = None
```

In the last chapter, we asked what indexing into a hash map evaluated to:

```oxiby
let shopping_list = ["apple": 3, "banana": 1, "carrot": 2]
shopping_list["carrot"]
```

The answer is `Some(2)`, a value of type `Option<Integer>`.
We might think it should just be `Integer`, since we can clearly see that we associated `"carrot"` with `2` when creating the hash map on the previous line.
But if that was the case, what would the indexing operation return for a key that had no value associated with it?

```oxiby
shopping_list["beans"] // ???
```

A function always returns the same type, so we can't return two different types under two different circumstances.
But this is exactly what `Option` is for:
Telling us when a value of a certain type might be there and might not.
Using an enum, we're able to return one of two variants which are both values of the same type.

With `Option` we can safely handle hash map indexing by `match`ing on an optional value:

```oxiby
fn food_count(shopping_list: HashMap<String, Integer>, food: String) {
    match shopping_list[food] {
        Some(quantity) => print_line("Buy #{quantity} #{food}."),
        None => print_line("Do not buy any #{food}"),
    }
}
```

This function takes a hash map of strings to integers, like our shopping list, and a string to use to index into the hash map.
We perform the index and then match on the returned `Option`.
If there was a value associated with key, we print both the key and the value.
If there was no value associated with the key, we print a fallback message with only the key.

Remember that `match` expressions must cover all cases exhaustively, so if we leave out the `None` match arm, our program won't compile.
This enforcement reduces bugs in our programs because we can't simply forget to check for the case of missing data.

> **Warning: Non-exhaustive matches are not yet enforced at compile time.
> Failure to account for all possibilities might result in a runtime crash.**

## A note about lists

Recall that when we learned about lists, we learned that indexing into a list uses the same square bracket syntax as indexing into a hash map, only with an integer offset rather than a key name:

```oxiby
let fruits = ["apple", "banana", "carrot"]
fruits[0] // "apple"
```

It's worth noting that currently, this operation does _not_ return an `Option` like it does for hash maps, even though it's the same idea, conceptually.
Instead, accessing an array index that is out of bounds (e.g. `fruits[3]` above) will cause an error when the program actually runs.
This is primarily for convenience, since manual bounds checking on every list index would add a lot of extra code when working with nested lists.
However, this is likely to change to a safer construct in a future revision of Oxiby.
