# Conventions

In this documentation, the following conventions are used.

## Terms

Important technical terms are indicated by bold text.

Examples:

* __Abstraction__ is the process of generalizing rules and concepts from specific examples.
* A field is optionally prefixed with a __visibility__.

## Emphasis

Emphasis is indicated by italics.

Example:

* Oxiby is a _lot_ of fun to use!

## Source code

Source code appears inline in prose with a monospace font.
Multi-line source code appears in blocks with a monospace font and syntax highlighting.

Examples:

* Inline code appears with `"a monospace font"`.

*   ```oxiby
    // Multi-line source code appears with
    fancy(syntax: "highlighting")
    ```

Multi-line code blocks can be easily copied to your system clipboard.
Hover over the block and click on the clipboard icon that appears in the upper right corner of the block.

## Example code

All of the example programs in the language guide are available in the `examples` directory of the documentation's Git repository, which you can find at [https://github.com/oxiby/oxiby-docs](https://github.com/oxiby/oxiby-docs).
Each multi-line code block corresponding to an example program will begin with a code comment showing the path to that file within the repository:

```oxiby
// File: examples/chapter_01_first/hello_world.ob
```

## Command line interaction

Commands that are intended to be run in a terminal emulator are shown in code blocks, with commands beginning with a `$` to indicate the command prompt.
You don't actually type the `$` when running the command.
A line beginning with a `>` indicates additional input you should type after executing the command that will be necessary for the program to proceed.
Lines in the same code block that do not begin with a `$` or `>` indicate the standard output or standard error of the previous command.

Example:

```
$ obc run hello_world.ob
> Oxiby
Hello, Oxiby!
```
