# Modeling data with structs

Our programs so far have dealt entirely with types native to Oxiby.
These have mostly been scalar types like `String` and `Boolean`, but we've also seen compound types like `List` and `HashMap`.
If we want to create our own types to model data in our programs, there are two options available.
The first are __structs__, which is short for __structures__.
We might also hear the same concept described as __records__, __objects__, or __product types__ in other languages.
A struct is a single type that is made up of multiple pieces of data.
Each of these constituent pieces has a name associated with it. Here's an example:

```oxiby
struct Person {
    name: String,
    age: Integer,
}
```

A struct is defined with the `struct` keyword, followed by a type name beginning with a capital letter, and then a comma-separated list of fields inside curly braces.
A field is a name beginning with a lower case letter, a colon, and the type of the field.

Constructing a struct inside a function uses very similar syntax, but specifies the values instead of the types:

```oxiby
fn alice() -> Person {
    Person {
        name: "Alice",
        age: 42,
    }
}
```

Fields can be accessed with the `.` operator for reading and writing:

```oxiby
let person = Person {
    name: "Alice",
    age: 42,
}

//  Prints "The person's name is Alice."
print_line("The person's name is #{person.name}.")

person.name = "Ainsley"

//  Prints "The person has changed their name to Ainsley."
print_line("The person has changed their name to #{person.name}.")
```

## Instance methods

There are two benefits to using structs rather than using individual variables to hold each piece of data.
The first is that it's easier to think about larger concepts like `Person` if all the data is combined, matching the way we think about it.
The second is that it allows us to attach behavior to the data in the form of methods.
Let's write some code similar to the previous example, but with a `Person` knowing how to print a sentence with its own name.

```oxiby
// File: examples/chapter_10_structs/instance_methods.ob

struct Person {
    name: String,
    age: Integer,

    fn print_name(self) {
        print_line("My name is #{self.name}.")
    }
}

fn main() {
    let person = Person {
        name: "Alice",
        age: 42,
    }

    //  Prints "My name is Alice."
    person.print_name()
}
```

A method is created by writing a function inside the definition of the struct, after all of the fields.
The `print_name` method has one parameter named `self`.
Notice that unlike usual function parameters, `self` is not followed by a colon and a type.
It's a special parameter that refers to an instance of the method's type.
Inside the method, we use `self` to access properties of the `Person` value associated with a particular method call.
The `self` parameter always comes first in the parameter list.

Recall that when we first learned about methods, we learned that the following two forms are equivalent:

```oxiby
function(value)
value.function()
```

That's exactly what's happening when we call `person.print_name()` at the end of the `main` function.
We don't pass any arguments to `print_name`, but inside the body of the `print_name` method, `person` is bound to `self`.

Methods that take a `self` parameter are called **instance methods** because they operate on an _instance_ of a type, i.e. a value.

## Static methods

Methods don't have to have a `self` parameter.
If they don't have one, they are considered **static methods**.
Unlike instance methods, the output of a static method is only determined by the arguments given to it directly.
As such, it's more like a regular function than a method.
The reason to use a static method rather than a free function (one not associated with a type) is to make it clear that the function's behavior has something to do with that type.

We'll commonly see static functions used for **constructors**.
These are functions that initialize a new value of the type:

```oxiby
# File: examples/chapter_10_structs/static_methods.ob

struct ShoppingList {
    map: HashMap<String, Integer>,

    fn produce() {
        Self {
            map: ["apple": 3, "banana": 1, "carrot": 2],
        }
    }

    fn print_contents(self) {
        for (food, quantity) in self.map {
            print_line("Quantity of #{food} to buy: #{quantity}")
        }
    }
}

fn main() {
    let shopping_list = ShoppingList.new()
    shopping_list.print_contents()
}
```

The static method `produce` constructs a new `ShoppingList` with a few predetermined food items to buy.
Using this constructor function, we can easily create a new `ShoppingList` without having to know anything about `ShoppingList`'s fields.
Both instance and static are called using the `.` operator.
The difference is that instance methods are called via _values_ while static methods are called via the _type_ and are not associated with any particular value.

We might have noticed that the `produce` function returns the type `Self`.
This is an alias that always refers to whatever type the associated function or method belongs to.
In this case, writing `Self` is the same as writing `ShoppingList`.
It's a nice shortcut that we can use to avoid writing the full name of the type over and over.
It also means that if we decide to rename the type, we only have to change its name in one place within its definition.

In Oxiby, static function names don't have any special meaning, so we don't have to name constructors anything specific like `new`, as we do in other programming languages.

> **Warning: The `Self` type alias is not yet available.
> The version of the above program found in the examples directory uses `ShoppingList` to account for this.**

> **Warning: Currently, a static function named `new` _does_ have a special meaning and cannot be used for constructors.**

## Tuple structs

There are two other kinds of structs we can create.

The first is called a __tuple struct__.
The difference from the main form of struct is that the fields are not named.
They are defined like this:

```oxiby
struct Name(String, String)
```

And constructed like this:

```oxiby
let name = Name("Alice", "Henderson")
```

Tuples are a core data type we haven't talked about yet.
They are like lists, in that they are a sequence of ordered values, but they have a fixed length (whereas lists can change size) and they need not all be the same type.
The following are all examples of tuples:

```oxiby
// Type: (String, String)
("Alice", "Henderson")

// Type: (Integer, String, String, String)
(123, "Main Street", "Anytown", "New Mexico")

// Type: () - an empty tuple, pronounced "unit"
()

// Type: (String)
// The comma is needed to distinguish from a parenthesized expression
("Oxiby",)
```

These "raw" tuple types do not have names.
A tuple struct is simply a way to represent such a type with a specific name, and to give it methods.

Just like structs with named fields, tuple structs can have both instance methods and static methods:

```oxiby
struct Name(String, String) {
    fn new(first_name: String, last_name: String) {
        Self(first_name, last_name)
    }

    fn full_name(self) -> String {
        "#{self.0} #{self.1}"
    }
}
```

This also illustrates how to access the fields of a tuple struct:
Use the `.` operator followed by a number, referring to the offset from the start of the tuple's values, just like a list.
The same syntax is used for regular tuples.

Tuple structs can be useful to "wrap" another type and give it new functionality, a technique called a **newtype**:

```oxiby
struct ExclamationString(String) {
    fn exclaim(self) -> String {
        "#{self.0}!"
    }
}
```
They're also convenient when the meaning of the fields are obvious and writing out names for them wouldn't provide much value.

> **Warning: The `Self` type alias is not yet available as shown above.**

## Unit structs

The other kind of struct is the unit struct, which holds no data at all.
They are defined like this:

```oxiby
struct Empty
```

They are constructed simply using the the type name:

```oxiby
let empty = Empty
```

Like the other kinds of structs, we can define methods on unit structs.
Since they have no data to work with, the benefit of doing this is mostly for organizing functions that relate to the same concept:

```oxiby
struct First {
    fn first_letter() -> String {
        "a"
    }

    fn first_positive_integer -> Integer {
        1
    }
}
