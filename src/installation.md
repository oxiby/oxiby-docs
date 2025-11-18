# Installation

Executable commands shown on this page should be run in a terminal emulator on macOS or Linux.
This process has not been tested on Windows, but will likely work the same way, especially when using Windows Subsystem for Linux.

## Prerequisites

Oxiby requires:

* Git to clone the compiler's source code
* Ruby (version 3.4 or later) to run the programs produced by the Oxiby compiler
* Rust (version 1.65 or later) to build the compiler itself
  * Cargo, the build tool for Rust, which will be installed alongside Rust

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
