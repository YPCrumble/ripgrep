termcolor
=========
A simple cross platform library for writing colored text to a terminal. This
library writes colored text either using standard ANSI escape sequences or
by interacting with the Windows console. Several convenient abstractions
are provided for use in single-threaded or multi-threaded command line
applications.

[![Windows build status](https://ci.appveyor.com/api/projects/status/github/BurntSushi/ripgrep?svg=true)](https://ci.appveyor.com/project/BurntSushi/ripgrep)
[![](https://img.shields.io/crates/v/wincolor.svg)](https://crates.io/crates/wincolor)

[![Linux build status](https://api.travis-ci.org/BurntSushi/ripgrep.png)](https://travis-ci.org/BurntSushi/ripgrep)
[![Windows build status](https://ci.appveyor.com/api/projects/status/github/BurntSushi/ripgrep?svg=true)](https://ci.appveyor.com/project/BurntSushi/ripgrep)
[![](https://img.shields.io/crates/v/termcolor.svg)](https://crates.io/crates/termcolor)

Dual-licensed under MIT or the [UNLICENSE](http://unlicense.org).

### Documentation

[https://docs.rs/termcolor](https://docs.rs/termcolor)

### Usage

Add this to your `Cargo.toml`:

```toml
[dependencies]
termcolor = "0.1"
```

and this to your crate root:

```rust
extern crate termcolor;
```

### Organization

The `WriteColor` trait extends the `io::Write` trait with methods for setting
colors or resetting them.

`Stdout` and `StdoutLock` both satisfy `WriteColor` and are analogous to
`std::io::Stdout` and `std::io::StdoutLock`.

`Buffer` is an in memory buffer that supports colored text. In a parallel
program, each thread might write to its own buffer. A buffer can be printed
to stdout using a `BufferWriter`. The advantage of this design is that
each thread can work in parallel on a buffer without having to synchronize
access to global resources such as the Windows console. Moreover, this design
also prevents interleaving of buffer output.

`Ansi` and `NoColor` both satisfy `WriteColor` for arbitrary implementors of
`io::Write`. These types are useful when you know exactly what you need. An
analogous type for the Windows console is not provided since it cannot exist.

### Example: using `Stdout`

The `Stdout` type in this crate works similarly to `std::io::Stdout`, except
it is augmented with methods for coloring by the `WriteColor` trait. For
example, to write some green text:

```rust
use std::io::Write;
use termcolor::{Color, ColorChoice, ColorSpec, Stdout, WriteColor};

let mut stdout = Stdout::new(ColorChoice::Always);
try!(stdout.set_color(ColorSpec::new().set_fg(Some(Color::Green))));
try!(writeln!(&mut stdout, "green text!"));
```

### Example: using `BufferWriter`

A `BufferWriter` can create buffers and write buffers to stdout. It does *not*
implement `io::Write` or `WriteColor` itself. Instead, `Buffer` implements
`io::Write` and `io::WriteColor`.

This example shows how to print some green text to stdout.

```rust
use std::io::Write;
use termcolor::{BufferWriter, Color, ColorChoice, ColorSpec, WriteColor};

let mut bufwtr = BufferWriter::stdout(ColorChoice::Always);
let mut buffer = bufwtr.buffer();
try!(buffer.set_color(ColorSpec::new().set_fg(Some(Color::Green))));
try!(writeln!(&mut buffer, "green text!"));
try!(bufwtr.print(&buffer));
```
