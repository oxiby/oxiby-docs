# Modeling data with enums

Enums (enumerations) are the second option for creating our own types in Oxiby.
We've already had some experience with enums with the `Result` and `Option` types.
An enum is a single type that can have multiple variants.
Each variant can contain different data, but each variant is considered the same type.
We might also hear the same concept described as a **sum type** in other languages.

```oxiby
enum TrafficLight {
    Red,
    Yellow,
    Green,
}
```

An enum is defined with the `enum` keyword, followed by a type name beginning with a capital letter, and then a comma-separated list of variant names, each beginning with a capital letter.

Constructing an enum inside a function looks like accessing a field on a struct:

```oxiby
fn main() {
    let red = TrafficLight.Red
}
```

`TrafficLight` is an example of an enum with **unit variants**.
This is the same idea as the **unit structs** that we learned about in the last chapter:
Types that hold no data but can have methods.

Like structs, enum variants can also have named fields:

```oxiby
enum Book {
    Finished {
        title: String,
    },
    BlankPages,
}
```

In this example, the `Finished` variant has a named field, while `BlankPages` is a unit variant with no fields.
Since structs are usually used in the form with named fields, an enum variant with named fields is called a **struct variant**.

For cases with data but where named fields aren't desired, we can also create **tuple variants**, just like tuple structs:

```oxiby
enum Quadrilateral {
    Rectangle(Integer, Integer),
    Square(Integer),
}
```

In this example, the `Rectangle` variant is a tuple with two integers representing the height and width of the quadrilateral, while the `Square` variant is a tuple with an Integer representing the length of all four sides.
Note that a trailing comma in the single-element tuple is not necessary.
That's only needed when creating a tuple value.

## Methods

Like structs, enums can have instance methods and static methods:

```oxiby
// File: examples/chapter_11_enums/vehicle.ob

enum Vehicle {
    Car,
    Truck,

    fn car() -> Self {
        Self.Car
    }

    fn truck() -> Self {
        Self.Truck
    }

    fn start(self) -> String {
        match self {
            Self.Car -> "Started the car.",
            Self.Truck -> "Started the truck.",
        }
    }
}

fn main() {
    let car = Vehicle.Car

    print_line(car.start())
}
```

These work in exactly the same way as structs, except that each method needs to do some extra work to determine which variant it's being called on.
Remember that `Self` is an alias for the type the method belongs to.

> **Warning: The `Self` type alias is not yet available.
> The version of the above program found in the examples directory uses `Vehicle` instead to account for this.**

## Pattern matching

We introduced the `match` expression in the chapter about `Result`, but just as a refresher:
When we have an enum value, we use `match` to determine which variant it is, as above in the `Vehicle.start` method.
If a variant contains data, `match` is also used to extract that data into local variables.
This works for both forms of variants that hold data:

```oxiby
// File: examples/chapter_11_enums/pattern_matching.ob

enum E {
    StructVariant {
        a: String,
        b: Integer,
    },
    TupleVariant(String),
    UnitVariant,
}

fn main() {
    let e = [
        E.StructVariant {
            a: "a",
            b: 1,
        },
        E.TupleVariant("t"),
        E.UnitVariant,
    ].sample()

    match e {
        E.StructVariant { a, b } -> print_line("Struct variant with data #{a} and #{b}"),
        E.TupleVariant(x) -> print_line("Tuple variant with data #{x}"),
        E.UnitVariant -> print_line("Unit variant with no data"),
    }
}
```

In this example, we create a list containing three enums, one for each of the variants of `E`.
We then use the `sample` method on `List` to pick a random element from the list.
Each time the program is run, `e` might be a different variant.

The `match` expression is used to handle every case of what `e` might be.
The named fields of the struct variant are matched and destructured using curly braces, matching the syntax for struct definition and initialization.
The same is true for the tuple variant, which is matched and destructured using parens.
Finally, the unit variant, which doesn't hold any data, is just matched by its name.
You can see in each match arm that any variables bound by that arm are used in the expression after the arm's arrow.

These types of patterns are actually not unique to enum variants.
They work exactly the same way for the three forms of structs.
That's because enums are really just a way of combining multiple structs into a single type.
