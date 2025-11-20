# Associating values with hash maps

Lists are useful for representing ordered collections of values, but aren't the right tool when we want to associate values with one another.
For example, if we had a shopping list, where each type of food was associated with the quantity of that food to buy, it would be awkward to represent as a list.
The right tool is a hash map.

```oxiby
let shopping_list = ["apple": 3, "banana": 1, "carrot": 2]
```

As we can see, a hash map looks very much like a list, but instead of each item being a single expression, each item is two expressions separated by a colon.
The expression on the left is called the **key** and the expression on the right is called the **value**.
For this reason, we'll sometimes hear hash maps referred to as key-value pairs.
Hash maps may also be referred to as dictionaries, since they are structured like a dictionary, which has words associated with definitions.

The reason it's called a hash map is because it uses a [hash function](https://en.wikipedia.org/wiki/Hash_function) on each key to determine how to uniquely identify a key-value pair.
Any type can be a value in a key-value pair, but only hashable types can be used as keys.
All the basic data types, `Boolean`, `Integer`, `Float`, and `String` are hashable.

## Iterating through key-value pairs

```oxiby
for (food, quantity) in shopping_list {
    print_line("Number of #{food} to buy: #{quantity}")
}
```

Just like a list, we use a `for` loop to iterate through the key-value pairs in a hash map.
The only difference is the variable we use to bind each pair is compound.
Instead of a single variable name like `item`, we have two variables in parenthesis, separated by a comma: `(key, value)`.

The reason we're able to do this is that the expression that comes after `for` is not just a variable name, but a pattern.
Remember that patterns allow us to **destructure** compound values into their constituent parts.
Don't worry about that for now.
We'll explore patterns further in a future chapter.
For now, just remember that when using a `for` loop with a hash map, we must bind both the key and the value.

## Inserting values

To insert a new value into a hash map, use the indexing operator plus an assignment:

```oxiby
shopping_list["dragonfruit"] = 1
```

If there was already a value associated with the key, it will be replaced with the one we're assigning.

## Retrieving a value by key

To access a specific value in a hash map, use the indexing operator, just as with lists, but with a key instead of an integer offset:

```oxiby
shopping_list["carrot"]
```

What does the above expression evaluate to?
We might have guessed 2, based on the initial value of `shopping_list` at the start of this chapter.
But it's a little more complicated than that.
We'll find out why in the next chapter.
