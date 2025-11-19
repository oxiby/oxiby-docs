# Organizing code with modules

For simple programs like the ones we've written so far, there's no trouble fitting everything in one file, but for larger programs, it's useful to split code up across multiple files to help with readability and organization.

In Oxiby, a module is a single file containing Oxiby source code.
Modules define types, functions, and a few more things we haven't learned about yet.
Each of these things is called an **item**.
An item can be imported from one module into another with the `use` keyword.
This is how modules share code with each other.

Importing items from another module looks like this:

```oxiby
use database.tables Person, load_people_from_json
```

The syntax is the keyword `use`, followed by the path to the module (as you would see in the file system, except with the slashes replaced with dots and the ".ob" suffix omitted), and a comma-separated list of items defined in that module to be imported into the current one.
You can have as many imports in a module as you need.

The path of a module is written relative to the main module of the program (the one you provide to the `obc build` command as the `INPUT` argument).
Modules beginning with the path `std` are reserved for Oxiby's standard library.

There can't be two items in a module with the same name, so if there's a name conflict, imports must be renamed with the `->` operator:

```oxiby
use library_a Name -> AName
use library_b Name -> BName
```

There's a special "module" called `self` that allows us to create aliases to items defined in the current module.
This is handy for using enum variants without naming the enum:

```oxiby
use self Vehicle.Car

enum Vehicle {
    Car,
}

fn main() {
    // This works as it always does
    let car = Vehicle.Car

    // Because of the `self` import, this also works
    let car = Car
}
```

In addition to reducing file size and grouping code for related functionality, modules also allow for hiding implementation details from other modules.

> **Warning: The `self` module is not yet implemented.**

## Encapsulation and visibility

By default, items are private to the module they are defined in, meaning that they cannot be imported into other modules.
Struct and enum fields are also private by default, meaning they cannot be accessed directly with the `.` operator from other modules.

Items and fields can be made public using the `pub` keyword, allowing us to selectively expose functionality to serve as the module's "public interface" while keeping any internal bookkeeping or support code hidden.
This idea is called **encapsulation** and is a commonly used technique across many programming languages.

Let's look at an example of how this can be useful to protect the data in our programs.
Back in the chapter on structs, we had a program that looked like this:

```oxiby
struct Person {
    name: String,
    age: Integer,
}

fn main() {
    let person = Person {
        name: "Alice",
        age: 42,
    }

    print_line!("The person's name is #{person.name}.")

    person.name = "Ainsley"

    print_line!("The person has changed their name to #{person.name}.")
}
```

In this program, the `main` function has direct access to `Person`'s fields, because `Person` is defined in the same module.
We create a `Person` and then change its name afterwards, simply by assigning a new value to its `name` field.
We reach inside the struct and change its internal state, and there is nothing the struct can do to stop us.

When our data is very simple this might not be a problem and might be the most convenient way to work.
But if we want to maintain the integrity of the data in the struct, we might want to protect it.
Consider the following example:

```oxiby
// File: examples/chapter_12_modules/exposed_shopping_list.ob

struct ShoppingList {
    map: HashMap<String, Integer>,
}

fn main() {
    let shopping_list = ShoppingList {
        map: HashMap.new(),
    }

    shopping_list.map["apple"] = -1

    print_line("Number of apples to buy: #{shopping_list.map["apple"]}")
}
```

In the chapter on hash maps, we used a hash map to represent a shopping list.
In this example, we use a dedicated struct called `ShoppingList` to represent it.
Internally, our struct uses the same hash map as before to keep track of the actual data.
In our main function, we create a new shopping list, and then set the quantity of apples to buy to -1.
But that doesn't make any sense.
How can you buy a negative number of apples?
Our struct should protect access to its internal data to make sure the user doesn't change it to something that doesn't make sense.

If we define the struct in its own module, we can use encapsulation to prevent this problem:

```oxiby
// File: examples/chapter_12_modules/shopping_list.ob

pub struct ShoppingList {
    map: HashMap<String, Integer>,

    pub fn new() -> Self {
        Self {
            map: HashMap.new(),
        }
    }

    pub fn add_food(self, food: String, quantity: Integer) -> Result<(), String> {
        if quantity < 1 {
            return Err("Quantity must be at least 1.")
        }

        self.map[food] = quantity

        Ok(())
    }
}
```

```oxiby
// File: examples/chapter_12_modules/main.ob

use shopping_list ShoppingList

fn main() {
    let shopping_list = ShoppingList.new()

    match shopping_list.add_food("apple", -1) {
        Ok(_) -> print_line("Added apples to our shopping list."),
        Err(error) -> print_line("Couldn't add apples to our shopping list: #{error}"),
    }
}
```

A few things have changed here.

### Visibility

`ShopingList` and the code that uses `ShoppingList` are now defined in two separate modules.
`ShoppingList` is in its own module, in a file called `shopping_list.ob`, while our `main` function is in a separate module, in a file called `main.ob`.
In the `main` module, we import the struct with the `use` keyword.
Note that the definition of `ShoppingList` in the `shopping_list` module is now prefixed with `pub`.
This is required for it to be imported into the `main` module.
Once we import `ShoppingList`, it's available for use in the `main` module, but only via its **public interface**.

Now that we're using the struct in a different module from where it was defined, we can no longer access its fields directly by default.
We can only access them if they have **public visibility**, indicated with the keyword `pub`.
In this case, the `map` field in the struct is not prefixed with `pub`, so it's private.
In order to read or write data from the struct, we need to use its public methods.

Because we can no longer access `map` directly from outside the struct, you'll notice that the syntax for creating the shopping list at the beginning of the `main` function has changed:

```oxiby
let shopping_list = ShoppingList.new()
```

Instead of using the struct initialization syntax to specify the initial value for the struct's fields, we call the constructor function `new`, which returns a new `ShoppingList` with default values.
Of course, it was necessary to define this constructor function, and we've done so inside the struct's body back in the `shopping_list` module, making it public with `pub`.
Since `ShoppingList` can access its own fields inside its own functions, we're able to specify an initial value for `map` inside `new`.

### The `add_food` method

```oxiby
pub fn add_food(self, food: String, quantity: Integer) -> Result<(), String> {
    if quantity < 1 {
        return Err("Quantity must be at least 1.")
    }

    self.map[food] = quantity

    Ok(())
}
```

Setting the quantity of a certain food to buy is now achieved by calling the `add_food` method rather than modifying the internal `map` field directly.
The `add_food` method checks the quantity it's given to make sure it's a positive value before changing the state in `map`.
If it's not, it returns an error back to the calling code in the `main` function.
But if the quantity supplied was good, `add_food` returns `()`, the empty tuple called **unit** that we learned about previously.
There's no meaningful value to return for the success case, but we have to return something inside the `Result.Ok` variant to signal to the caller that the operation succeeded.

This means that back in `main`, we must check to see whether or not our addition succeeded:

```oxiby
match shopping_list.add_food("apple", -1) {
    Ok(_) -> print_line("Added apples to our shopping list."),
    Err(error) -> print_line("Couldn't add apples to our shopping list: #{error}"),
}
```

When we match on the result, we use the pattern `Ok(_)`.
The underscore matches any value but does not bind it to a variable.
We use this because we aren't going to be doing anything with the unit inside the result—we just want to know when the `add_food` call was successful.

Just like `new`, `add_food` is marked with `pub` so we can use it from `main.ob`.
Of course, not all methods need to be marked with `pub`.
It's often useful to have methods with private visibility, which can only be called by code in the defining module.
This allows us to be selective in how we expose our struct's behavior to outside code.
Private methods can be used for implementation details that outside code doesn't need to know about.

When writing our own types, it's a good idea to use the default, private visibility when possible.
Only make fields and methods public with the `pub` keyword when you need to.
A field should be private if it could be made invalid as it could with our shopping list.

> **Warning: Currently, importing from a module does not automatically build that module.
> Each dependent module must be built individually with `obc build`.**

> **Warning: Currently, a static function named `new` has special meaning and cannot be used for constructors.
> The version of the above program found in the examples directory uses `create` instead to account for this.**

> **Warning: The `Self` type alias is not yet available.
> The version of the above program found in the examples directory uses `ShoppingList` instead to account for this.**

> **Warning: Item and field visibility is not yet enforced, so code that should be private is currently accessible from other modules.**

> **Warning: Currently, modules cannot be in nested directories.
> Outside of the automatically generated `std` modules, all user modules must live in the same directory.**
