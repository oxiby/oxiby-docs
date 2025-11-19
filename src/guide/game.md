# Writing tic-tac-toe in Oxiby

We've learned enough now that we can write some actually useful programs.
We're going to write a version of the game tic-tac-toe that you can play with a friend in your terminal.

We'll start by looking at the complete program.
Then we'll break it down bit by bit to understand how it works and how it uses all the things we've learned in this language guide (as well as a few new things).

Paste the code below into an Oxiby file and run it with `obc run` to try the game.

```oxiby
// File: examples/chapter_15_game/tic_tac_toe.ob

use std.io read_line

enum Player {
    X,
    O,
}

enum Winner {
    Player(Player),
    Draw,
    NoneYet,
}

struct Game {
    player: Player,
    rows: List<List<Option<Player>>>,

    fn winner(self) -> Winner {
        for row in (0..2) {
            match (self.rows[row][0], self.rows[row][1], self.rows[row][2]) {
                (Some(a), Some(b), Some(c)) -> {
                    if a == b && b == c {
                        return Winner.Player(a)
                    }
                },
                _ -> (),
            }
        }

        for column in (0..2) {
            match (self.rows[0][column], self.rows[1][column], self.rows[2][column]) {
                (Some(a), Some(b), Some(c)) -> {
                    if a == b && b == c {
                        return Winner.Player(a)
                    }
                },
                _ -> (),
            }
        }

        match (self.rows[0][0], self.rows[1][1], self.rows[2][2]) {
            (Some(a), Some(b), Some(c)) -> {
                if a == b && b == c {
                    return Winner.Player(a)
                }
            },
            _ -> (),
        }

        match (self.rows[0][2], self.rows[1][1], self.rows[2][0]) {
            (Some(a), Some(b), Some(c)) -> {
                if a == b && b == c {
                    return Winner.Player(a)
                }
            },
            _ -> (),
        }

        for row in self.rows {
            for column in row {
                match column {
                    Some(player) -> (),
                    None -> return Winner.NoneYet,
                }
            }
        }

        Winner.Draw
    }

    fn choose(self, cell: Integer, player: Player) -> Boolean {
        if !(1..9).contains(cell) {
            return false
        }

        match self.rows[(cell - 1) / 3][(cell - 1) % 3] {
            Some(_) -> false,
            None -> {
                self.rows[(cell - 1) / 3][(cell - 1) % 3] = Some(player)
                true
            }
        }
    }
}

fn main() {
    let game = Game {
        player: Player.X,
        rows: List.times(3, fn () { List.times(3, fn () { None }) }),
    }

    loop {
        print_line("Game board:")

        for row in game.rows {
            for cell in row {
                match cell {
                    Some(player) -> print(player),
                    None -> print("."),
                }
            }

            print_line("")
        }

        print_line("")

        match game.winner() {
            Winner.Player(player) -> {
                print_line("The winner is #{player}!")
                break
            },
            Winner.Draw -> {
                print_line("It's a draw!")
                break
            },
            Winner.NoneYet -> (),
        }

        loop {
            print("It's #{game.player}'s turn. Enter the cell to play (1-9): ")

            match read_line().map(fn (s) { s.to_i() }) {
                Ok(cell) -> {
                    if game.choose(cell, game.player) {
                        game.player = match game.player {
                            Player.X -> Player.O,
                            Player.O -> Player.X,
                        }

                        break
                    }
                },
                _ -> (),
            }

            print_line("")
            print_line("You entered an invalid cell number.")
        }

        print_line("")
    }
}
```

## How to play

In case you're not familiar with tic-tac-toe, here's a short description of how it works, and how we interface with our command line version of the game.

```
Game board:
...
...
...

It's X's turn. Enter the cell to play (1-9):
```

At the start of the game, we're presented with a three-by-three grid of cells represented by dots, and a prompt for the first player's turn.
Two players, X and O, will take turns placing their marks on the grid.
A cell is selected by choosing a number from 1 to 9, with each number corresponding to its position on the grid when read left to right, top to bottom:

```
123
456
789
```

The goal is to be the first player to get three of their marks in a line, either horizontally, vertically, or diagonally.

Give it a shot, either with a friend, or playing against yourself, controlling both X and O.
Try to find ways to break the game: entering input outside the 1 to 9 range, entering a blank line as input, playing a cell that's already taken, etc.

## Breaking down the program

### Imports

```oxiby
use std.io read_line
```

The program starts with the one function we need from the standard library that is not imported by default: `read_line` from `std.io`.
We've seen this several times before.
It's what we use to read text the user enters on the command line into our program.

### Data types

```oxiby
enum Player {
    X,
    O,
}

enum Winner {
    Player(Player),
    Draw,
    NoneYet,
}

struct Game {
    player: Player,
    rows: List<List<Option<Player>>>,

    // ...
}
```

We model the data for tic-tac-toe with three custom data types.

`Player` is an enum representing the two players, X and Y.
We'll use this to keep track of whose turn it is, the marks placed on the board, and who the winner is.
This enum has two unit variants that hold no data, because the name of the variant contains enough information for our needs.

`Winner` is another enum representing the state of the win condition.
At any point in the game, there are four possible cases:

* Player X won
* Player O won
* Neither player won and there are no more open cells
* Neither player won and there are still open cells

Again, we represent these cases with enum variants, collapsing the first two into a single variant that holds a `Player`, telling us which of the two players won.

`Game` is a struct that represents the state of the overall game.
It has two fields, `player`, which tracks whose turn it is, and `rows`, which tracks the grid and the contents of each cell.
The type of the `rows` field might look a little unwieldy, but it's not too complicated.
It's a list of lists.
Each element in the outer list is another list.
This represents our rows.
Each element in the inner list is an optional player.
This represents the columns in a row and whether or not a player has left a mark in the corresponding cell.

### Creating the initial state

The `Game` type has two methods, but let's ignore them for now and take a look at the main loop that runs the game.
The `main` function begins by initializing the state for a new game:

```oxiby
let game = Game {
    player: Player.X,
    rows: List.times(3, fn () { List.times(3, fn () { None }) }),
}
```

We create a `game` with the initial player being X.
We initialize the grid with a fancy expression that creates our nested lists.
This might be a little tricky to read, so let's add some whitespace and comments to help see what's happening:

```oxiby
// Create outer list (rows)
List.times(
    3, // Create three rows
    fn () {
        // Create inner list (columns/cells)
        List.times(
            3, // Create three columns/cells
            fn () {
                // Each cell is empty to start
                None
            }
        )
    }
)
```

`List.times` is a static method on `List` from the standard library that allows us to create a list of a known size and to populate each of its elements with a known value.
Here's its signature:

```oxiby
fn times(n: Integer, f: f) -> List<t> where f: Fn() -> t
```

The function takes an integer, which is the number of elements the new list should have, and a closure.
The closure returns a value that should be used to populate each element in the new list.
Since we're creating a grid, represented as rows of columns, we create a list of three elements to represent the three rows.
The value of each element in a row is another list of three elements, hence the inner `List.times` call.
The value of the elements in the inner list (our columns/cells) begins as `None`, the variant of `Option<Player>` indicating that neither player has left their mark in that cell yet.

### Printing the board

```oxiby
loop {
    print_line("Game board:")

    for row in game.rows {
        for cell in row {
            match cell {
                Some(player) -> print(player),
                None -> print("."),
            }
        }

        print_line("")
    }

    // ...
}
```

The main game loop is an actual `loop`!
The loop will continue until we explicitly break out of it.
In game terms, this means that the players will keep making moves until one of them wins, or until there's a draw.

Each loop of the game begins by printing a visualization of the grid.
We do this with a nested loop, iterating through the rows and the cells within those rows.
For each cell, we check whether or not there's a player mark.
If there is, we print that mark (X or O).
If the cell is still empty, we print a dot.
After each row, we print a blank line so that each row of cells will appear on its own line in the output.

### Checking the win condition

```oxiby
match game.winner() {
    Winner.Player(player) -> {
        print_line("The winner is #{player}!")
        break
    },
    Winner.Draw -> {
        print_line("It's a draw!")
        break
    },
    Winner.NoneYet -> (),
}
```

Next, we check to see if the game should end.
We call the `winner` method on our `Game` type to inspect the state.
`winner` returns a value of our custom `Winner` type.
We `match` on this `Winner` to decide what to do:

1. If a player won, we print the name of the winner and then break out of the loop, ending the game.
1. If neither player won, we print that it's a draw and then break out of the loop, ending the game.
1. If neither player won, but there are still legal moves, there's nothing to do, so we give this match arm an empty tuple (unit) expression, which will do nothing and proceed to the next part of the loop.

### Determining the win condition

Here's the definition of the `winner` method, which determines the win condition, if any:

```oxiby
fn winner(self) -> Winner {
    for row in (0..2) {
        match (self.rows[row][0], self.rows[row][1], self.rows[row][2]) {
            (Some(a), Some(b), Some(c)) -> {
                if a == b && b == c {
                    return Winner.Player(a)
                }
            },
            _ -> (),
        }
    }

    for column in (0..2) {
        match (self.rows[0][column], self.rows[1][column], self.rows[2][column]) {
            (Some(a), Some(b), Some(c)) -> {
                if a == b && b == c {
                    return Winner.Player(a)
                }
            },
            _ -> (),
        }
    }

    match (self.rows[0][0], self.rows[1][1], self.rows[2][2]) {
        (Some(a), Some(b), Some(c)) -> {
            if a == b && b == c {
                return Winner.Player(a)
            }
        },
        _ -> (),
    }

    match (self.rows[0][2], self.rows[1][1], self.rows[2][0]) {
        (Some(a), Some(b), Some(c)) -> {
            if a == b && b == c {
                return Winner.Player(a)
            }
        },
        _ -> (),
    }

    for row in self.rows {
        for column in row {
            match column {
                Some(player) -> (),
                None -> return Winner.NoneYet,
            }
        }
    }

    Winner.Draw
}
```

This is a lot of code, but conceptually, it's straightforward:

1.  We check if any of the rows have each cell marked by the same player.
    If any does, we return that player as the winner.
1.  We check if any of the columns have each cell marked by the same player.
    If any does, we return that player as the winner.
1.  We check both possible diagonals (cells 1/5/9 and cells 3/5/7) to see if either is marked entirely by the same player.
    If either does, we return that player as the winner.

If we get past this point, it means no one has won yet, so it's either a draw or the game is still in progress.
Finally:

1.  We check if there are any empty cells.
    If there are, we return `Winner.NoneYet` to indicate the game should continue.
    If there aren't, we return `Winner.Draw` to indicate the game has ended without a winner, as there are no other possibilities.

Let's look at one of these checks in a little more detail to see how it's done:

```oxiby
for row in (0..2) {
    match (self.rows[row][0], self.rows[row][1], self.rows[row][2]) {
        (Some(a), Some(b), Some(c)) -> {
            if a == b && b == c {
                return Winner.Player(a)
            }
        },
        _ -> (),
    }
}
```

This loops through each row in the grid and checks for rows that have all of the same mark.
The expression `(0..2)` creates a value of type `Range` that is used to represent a range of values in a meaningful order.
The first number is the start of the range and the second number is the end of the range.
A range defined this way is _inclusive_ of the second number, so in this case, we're creating a range that covers the values `0`, `1`, and `2`.
If we wanted to create an _exclusive_ range, we'd write `(0..<2)`, which would cover only the values `0` and `1`.

Ranges can be iterated with a `for` loop.
We loop over the three integers, with each bound to the variable `row`.
In each iteration, we `match` against a tuple of each cell in that row.
`self.rows` is the list of rows, `[row]` takes the row we want to check from the list, and then the `[0]`, `[1]`, and `[2]` indexes get the value of the first, second, and third column of that row.
The type of each cell is `Option<Player>` so we're matching on a value of this type:

```oxiby
(Option<Player>, Option<Player>, Option<Player>)
```

There are a lot of combinations of possible values here, but the only one we care about is the one where all three are the same player, so we have a match arm for the case where all three are `Option.Some`.
If they're all `Some`, we check if the player is the same for each one, and if so, declare that player the winner by returning `Winner.Player`.
To make sure the match is exhaustive, we have one more match arm that matches on all other cases using the `_` wildcard.
That case just evaluates to an empty tuple (unit) and does nothing.

### Reading the player's cell choices

The last part of our game is the logic that reads input from the current player and tries to mark the cell they choose.

```oxiby
loop {
    print("It's #{game.player}'s turn. Enter the cell to play (1-9): ")

    match read_line().map(fn (s) { s.to_i() }) {
        Ok(cell) -> {
            if game.choose(cell, game.player) {
                game.player = match game.player {
                    Player.X -> Player.O,
                    Player.O -> Player.X,
                }

                break
            }
        },
        _ -> (),
    }

    print_line("")
    print_line("You entered an invalid cell number.")
}
```

Note that this is a second `loop` _inside_ our main game `loop`.

We use `game.player` to prompt the player whose turn it is to choose a cell, then we read their choice.
Recall that `read_line` returns `Result<String, String>` so it might succeed and give us the input we want, or it might fail with some sort of I/O error.
By using `map`, we provide a closure that parses the input into an integer, but only in the case that `read_line` returns the `Result.Ok` variant.
By parsing the input into a number, we turn values like `"1"` into the integer `1`, so we can perform numeric operations on them.

We match on this mapped result and check for the `Ok` case.
If we got a valid integer back, we attempt to place their mark in the chosen cell by passing both the cell number and the current player to our `Game` type's `choose` method.
`choose` returns a boolean indicating whether or not the player's mark was successfully placed in the cell.
If it was, we use a `match` expression to reassign the value of `game.player` to the opposite of the current player, which effectively changes which player's turn it is on the next iteration of the main game loop.
Then we `break` out of the inner loop to start a new turn.
If `read_line` returned an `Err` or `game.choose` returned false, we fall through to the `print_line` statements at the bottom, letting the player know that whatever input they entered either couldn't be read or wasn't an available cell.
The inner loop then starts again, and continues the same process until the player chooses a valid cell.

> **Warning: `s.to_i` is just Ruby's `String#to_i` method exposed directly, which does loose conversions like parsing `"a"` as `0`.
> This program is written assuming the input will always successfully parse into a valid integer, but that isn't guaranteed.
> In the future, Oxiby will provide a safer technique for parsing integers from strings.**

### Choosing a cell

The last piece of code to look at is the definition of the `choose` method.
Again, this method's job is to try placing a player's mark in the given grid cell, and then returning a boolean value to indicate whether or not it succeeded.

```oxiby
fn choose(self, cell: Integer, player: Player) -> Boolean {
    if !(1..9).contains(cell) {
        return false
    }

    match self.rows[(cell - 1) / 3][(cell - 1) % 3] {
        Some(_) -> false,
        None -> {
            self.rows[(cell - 1) / 3][(cell - 1) % 3] = Some(player)
            true
        }
    }
}
```

The method begins with a conditional expression to check whether or not the integer entered by the player corresponds to a cell in the grid.
We do this by creating a range of all valid cell numbers with `(1..9)` and then calling the `contains` method on the range to check whether a specific element is part of that range.
If it isn't, we know the player's choice was out of bounds, and hence an invalid value, so we return `false`.

If the value is within the range, then we check whether or not there's already a mark in that cell by matching on the current value of the cell.
The fancy arithmetic inside the indexing operations are used to convert the 1 to 9 range that we accept as cell numbers to the corresponding offsets in our list representation of the grid.
`(cell - 1) / 3` changes our 1-based offsets to the 0-based offsets used by list indexing and then divides that by 3 so that we get the right index regardless of which row the player picked.
`(cell - 1) % 3` does somthing similar, but uses the `%` (modulo) operator to get the remainder of a division by three, which ends up being the correct list offset for the columns within each row.

If we find that there's already a mark in the cell, we return `false`.
We again use the `_` wildcard to match the player in the `Some` case, because we don't care which player has marked the cell.
We only care whether or not there is a mark in the cell.
If there's no mark, we set the value of that cell (using the same indexing expression) to the player who chose it.
This modifies our game state.
Then we return `true` to let the main loop know that the choice was valid and that we can proceed to the next turn.

And that's our game of tic-tac-toe!
