# Installation

## Prerequisites

Oxiby requires:

* Ruby (version 3.4 or later) to run the programs produced by the Oxiby compiler
* Rust (version 1.65 or later) to build the compiler itself

## Installation steps

1.  Install Ruby: [https://www.ruby-lang.org/en/downloads/](https://www.ruby-lang.org/en/downloads/)
1.  Install Rust: [https://rust-lang.org/tools/install/](https://rust-lang.org/tools/install/)
1.  Clone the Oxiby Git repository:

    ```
    $ git clone https://github.com/oxiby/oxiby.git
    ```

1.  From inside the newly cloned repository, execute:

    ```
    $ cargo install --path .
    ```

    Note the trailing `.` at the end of that command.

    This will build the compiler and place it in Cargo's `bin` directory, which you should add to your `PATH` environment variable as described in the installation instructions for Rust.
1.  The Oxiby compiler is now available. Try executing:

    ```
    $ obc --help
    ```
