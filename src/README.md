# Introduction

**Oxiby** is a statically typed programming language inspired primarily by [Rust](https://www.rust-lang.org/).
Oxiby programs compile into readable [Ruby](https://www.ruby-lang.org/) source code, which can then be executed by the Ruby interpreter.

The language was motivated by the question:

**What if a developer wants the high-level features of Rust, but does not need its low-level features, and maximal performance is not a requirement?**

Oxiby is free software.
You can find its source code for reading and modification in its repository on GitHub: [https://github.com/oxiby/oxiby](https://github.com/oxiby/oxiby)

## A quick taste of Oxiby

Here's what Oxiby looks like:

```oxiby
fn greet(name: String) -> String {
  "Hello, #{name}!"
}

fn main() {
  print_line(greet("Oxiby"))
}
```

Given a file named `intro.ob` with this source code, the Oxiby compiler produces the following Ruby:

```ruby
# frozen_string_literal: true

require_relative "std/all"

module Intro
  def self.greet(name)
    "Hello, #{name}!"
  end

  def self.main
    ::Std::Io.print_line(greet("Oxiby"))
  end

  main
end
```

## Features

Oxiby offers the following features for productive programming:

* Static typing with type inference
* Parametric polymorphism (generic programming)
* Ad-hoc polymorphism (operator overloading)
* Algebraic data types (product and sum types)
* Type classes (traits)
* Pattern matching

## Audience

The language guide is not written for an audience presumed to have experience programming.
As such, many fundamental programming concepts are explained alongside their implementation in Oxiby.

## Caveats

Oxiby is an experimental language and is not intended for use in serious projects.

Although it compiles to Ruby, it is not intended to be interoperable with existing Ruby programs.

The language is in early development and many features are not yet implemented.
As such, some of the behavior described in this book is aspirational.
The plan is to implement the language as described in the book, so anything described here will be supported eventually.
That said, all syntactically valid Oxiby source code should compile into equivalent Ruby, but the compiler will allow programs that shouldn't pass type checking, and Oxiby code can "escape" into Ruby not exposed by Oxiby.

All the example programs shown in the language guide will compile and produce the expected behavior when run, with certain exceptions as noted.

Throughout the guide, if a featuring being described is notably absent or differs from the current implementation, it will be denoted with a warning like this:

> **Warning: This feature is not yet implemented.**
