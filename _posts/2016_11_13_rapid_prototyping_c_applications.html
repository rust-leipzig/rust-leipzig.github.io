---
layout:     post
title:      Rapid prototyping C applications with Rust
date:       2016-11-13
summary:    With some small overhead it is possible to use the capabilities of Cargo and Rust for C based programs.
categories: cargo
---

[Cargo](https://github.com/rust-lang/cargo) is a gorgeous tool when it comes to building, packaging and shipping your
own Rust applications. Some of you guys are living in a C/C++ world where developing Rust applications is just a hobby.
A reason could be that Rust is currently not that highly accepted in the world of _applications in production_. This is
sad, but even within your day to day development it could be possible to use Rust for a lots of tools or prototyping
scenarios!

One example could be to use the possibilities of Rust and Cargo to do prototyping of C based applications. So how could
we do that?

For the first try we simply build a new cargo project:
```
> cargo new --bin rapidc
     Created binary (application) `rapidc` project
```

The target is now to come as fast as possible back into the C world. For this we add the two dependencies `libc` and
`gcc`, as well as a custom build script `build.rs` to our `Cargo.toml` file:

```toml
[package]
name = "rapidc"
version = "0.1.0"
build = "build.rs"

[dependencies]
libc = "0"

[build-dependencies]
gcc = "0"
```

What else? For sure we need an entry point for our C application, which we name `src/main.c` and looks like:

```c
#include <stdio.h>

int main_c(int argc, char *argv[])
{
    printf("Found %d args:\n", argc);
    for (int i = 0; i < argc; ++i) {
        printf("\t- %s\n", argv[i]);
    }
    return 0;
}
```

This program does not do really special at all, but as a demonstration we print out all available command line
arguments. The interesting part happens within the Rust entry function in `main.rs`:

```rust
extern crate libc;

use std::env;
use std::ffi::CString;
use libc::{c_int, c_char};

extern "C" {
    fn main_c(argc: c_int, argv: *const *const c_char) -> c_int;
}

fn main() {
    // Get the current args and map them to a vector of zero terminated c strings
    let args: Vec<CString> = env::args().filter_map(|arg| CString::new(arg).ok()).collect();

    // Convert these c strings to raw pointers
    let c_args: Vec<*const c_char> = args.iter().map(|arg| arg.as_ptr()).collect();

    // Call the main function within the created c library
    unsafe {
        main_c(c_args.len() as c_int, c_args.as_ptr());
    };
}
```

The Foreign Function Interface (FFI) of Rust is pretty straight forward and Cargo even supports us in writing those
applications. We simply need to declare our C function within the scope of `extern "C" { ... }`. Since we cannot use the
basic C types within Rust we have to use the ones of the `libc` crate (like `c_int` or `c_char`). The `main` function
creates a Vector of C pointers to our command line arguments and passes them within the `unsafe {...}` block to our C
entry function.

Do we forgot something? Yes, what about the `build.rs` script? How can we tell Cargo to first build our C library
and then link it to the Rust application? The build script looks like:

```rust
extern crate gcc;

fn main() {
    gcc::Config::new().file("src/main.c").include("src").compile("libmain.a");
}
```

Pretty easy right? The `gcc` crate helps us to easily build the `main.c` to the library `libmain.a`. Now we are able to
try out our work by executing:

```
> cargo run
cargo run
    Finished debug [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/rapidc`
Found 1 args:
        - target/debug/rapidc
```

Or what about:

```
> cargo run arg1 arg2 arg3
    Finished debug [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/rapidc arg1 arg2 arg3`
Found 4 args:
        - target/debug/rapidc
        - arg1
        - arg2
        - arg3
```

Great, everything works as expected, ...but stop! What happened here? We did not tell Cargo to link against a library
called `libmain.a`. How should Cargo know what to do with the library? That is one of the very important features of
such a great build environment: Strong defaults will lead into a better world. Cargo will simply assume that you call
your library `libmain.a` for the file `main.c`.

If we want to link another library to our C application we can simply use the `link` directive:
```rust
#[link(name = "pthread")]
extern "C" {
    fn main_c(argc: c_int, argv: *const *const c_char) -> c_int;
}
```

It is pretty brilliant: On one hand we have the full power of Rust and Cargo via the language itself within the
`build.rs` script. This means we have the full flexibility like in CMake by the usage of the complete programming
language set of Rust. On the other hand the strong defaults of Cargo will make it very easy to build release versions,
debug our code via `gdb` or `lldb`, package the application and write benchmarks or unit tests for your C functions.
This is all possible without the need of a cruel Makefile or a bloated CMake environment. How great is that?! 😀

Currently I am working on some [Cargo port called 'Craft'](https://github.com/saschagrunert/craft) which should combine
both worlds out of the box. Replacing the compiler facade is a lots of work and Cargo is even evolving very fast which
makes me think about writing just an cargo extension to do faster prototyping in C. What do you think?

The full source code of the small example project is available [on GitHub](https://github.com/saschagrunert/rapidc).
