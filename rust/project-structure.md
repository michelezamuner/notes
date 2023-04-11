# Project structure

## Packages and crates

The highest level of project organization in Rust is made of *packages*. A package is made of a set of sub-components providing a set of related functionality: these are called *crates* and can represent either binary applications, or libraries.

Every package must contain a `Cargo.toml` file describing how to build the package's crates. A package can contain either zero or one library crate, and any number of binary crates, but it must contain at least one crate of either type.

To tell if a crate is binary or library, Cargo look at the crate root. If the crate root is a file `src/main.rs`, then that one is a binary crate; if the crate root is a file `src/lib.rs`, then that's a library crate.

The `src/main.rs` file represents the starting point for the execution of the binary program, and it must define one `main` function taking no argument and returning no value:

```rust
fn main() {
  // ...
}
```

If needed, the `main` function can also return a `Result`:

```rust
fn main() -> Result<(), Box<dyn Error>> {
  // ...
}
```

The crate root should link together all other files of the crate, so that when the crate root is compiled, automatically all crate's files are compiled as well.

To define other binary crates in addition to the default one, we can put them inside `src/bin`.

## Modules

A *module* is a way to namespace code, and to control its visibility. A module is defined with the `mod` keyword:

```rust
// src/lib.rs
mod front_of_house {
  mod hosting {
    fn add_to_waitlist() {}

    fn seat_at_table() {}
  }

  mod serving {
    fn take_order() {}

    fn serve_order() {}

    fn take_payment() {}
  }
}
```

Here we created a library crate containing a top-level module named `front_of_house`. Modules can then contain global-scope definitions, including other modules.

Global-scope definitions are identified by their *path*. The *absolute path* identifies a definition with relation to the crate root, while a relative path identifies a definition with relation to the module where that path is used:

```rust
// absolute path
crate::front_of_house::hosting::add_to_waitlist();
// relative path
front_of_house::hosting::add_to_waitlist();
```

thus, if we are in the top-level module of a crate, the absolute path using `crate` is equivalent to the relative path.

We can refer to the parent module by using the `super` keyword:

```rust
fn serve_order() { }

mod back_of_house {
  fn fix_incorrect_order() {
    cook_order();
    super::serve_order();
  }

  fn cook_order() { }
}
```

## Visibility

All definitions inside a module are private by default, meaning that they can't be used outside of that module. In particular, every module encapsulates its implementation, so other modules cannot see its inner definitions. However, a module is defined within the context of another module, so it can see the other definitions of the outer modules.

In order to expose a definition to the outer modules, we can use the `pub` keyword:

```rust
// src/lib.rs
mod front_of_house {
  pub mod hosting {
    fn add_to_waitlist() {}
  }
}
```

this way the module `hosting` has become public, so we can refer to it from the outside with `front_of_house::hosting`: however, the inner function `add_to_waitlist` is still private, so trying to write `front_of_house::hosting::add_to_waitlist()` would produce an error. In order for this to work we need to make also the function public:

```rust
// src/lib.rs
mod front_of_house {
  pub mod hosting {
    pub fn add_to_waitlist() {}
  }
}
```

When we declare a struct as `pub`, its fields will still remain private, unless we declare them as `pub` as well:

```rust
mod back_of_house {
  pub struct Breakfast {
    pub toast: String,
    seasonal_fruit: String,
  }
}

fn test_construct () {
  let breakfast = back_of_house::Breakfast {
    toast: String::from("toast"),
    // error: field `seasonal_fruit` of struct `Breakfast` is private
    seasonal_fruit: String::from("fruit"),
  }
}

fn test_read (breakfast: back_of_house::Breakfast) {
  let toast = breakfast.toast;
  // error: field `seasonal_fruit` of struct `Breakfast` is private
  let seasonal_fruit = breakfast.seasonal_fruit;
}
```

here the `toast` field, which has been declared as `pub`, will be visible from the outside.

The visibility is always related only to the module, and not the struct. This means that a struct field that is not public is still visible to all the module where that struct is defined, which means also to all the code of that module outside that struct; on the other hand, making struct fields public does not mean making them visible outside that struct, but outside the module. Thus, if we only need to access a struct's fields from inside the same module where the struct is defined, we don't need to make its fields public.

## Importing modules

In order to avoid having to repeat a long absolute or relative path every time we need to refer to an identifier, we can declare that identifier once with a `use` declaration:

```rust
mod front_of_house {
  pub mod hosting {
    pub fn add_to_waitlist() {}
  }
}

use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
  hosting::add_to_waitlist();
}
```

Here, with `use crate::front_of_house::hosting` we're importing a module by its absolute path. If we wanted to use a relative path, instead, we'd have to use the `self` keyword:

```rust
use self::front_of_house::hosting;
```

If we're trying to import two types with the same name into the same scope, we can use the `as` keyword to assign aliases to them:

```rust
use std::fmt::Result;
use std::io::Result as IoResult;

fn function1() -> Result {
  // ...
}

fn function2() -> IoResult<()> {
  // ...
}
```

After we imported a name into a scope, that name would by default be private to that new scope. If we want to export that name to the outer scope as well, we can use `pub use`:

```rust
mod app {
  pub use crate::front_of_house::hosting;
}

use app::hosting;
```

When we want to import multiple types that are defined under the same module, we can write:

```rust
use std::io;
use std::cmp::Ordering;
```

To save lines, we can group together imports that refer to the same module using nested paths:

```rust
use std::{io, cmp::Ordering};
```

We can also import all public elements of a module with the *glob operator*:

```rust
use std::collections::*;
```

## Modules and files

When a module is defined in a different file, we need to declare it into the crate root, in order for it to be findable:

```rust
// src/lib.rs
mod front_of_house;

pub use front_of_house::hosting;
```

Here we put the definition of `front_of_house` in a file `src/front_of_house.rs`. It's important that the file is name exactly as the module, in order for the compiler to be able to find it. This file might contain a reference to another module as well:

```rust
// src/front_of_house.rs
pub mod hosting;
```

In order for this to work, the `hosting` module must be defined inside a file `src/front_of_house/hosting.rs`, where the parent module `front_of_house` has become a directory.

## Project crate

When we define the library crate for the project, that crate automatically takes the name of the project itself. For example, if we have a project called `proj`:

```
proj
|-- Cargo.lock
|-- Cargo.toml
|-- src
|   |-- lib.rs
|   |-- main.rs
```

and the library crate contains a definition like:

```rust
// src/lib.rs
pub fn run() {
  println!("Run");
}
```

then we can access this function by using the name of the project to refer to the library crate root from the main application root:

```rust
// src/main.rs
use proj::run;

fn main() {
  run();
}
```

exactly like we would do if we were referring to a third party crate.

If we've also defined a module called `proj` related to the application crate:

```rust
// src/proj.rs
pub fn exec() {
  println!("exec");
}
```

we need to distinguish the `proj` crate  from the `proj` module:

```rust
// src/main.rs
mod proj;

use ::proj::run;
use crate::proj::exec;

fn main() {
  run();
  exec();
}
```

here we first need to import the `proj` module in the crate root with `mod proj`; then we have two modules called `proj`: one which is a child of the application crate, and the other that is a child of the library crate. To identify the `proj` module belonging to the application crate, we use `use crate::proj::exec`, because `crate::` identifies the current crate, which in this case is the application crate, since we're inside `main.rs`; to identify the `proj` module of the library crate, instead, we cannot just write `use proj::exec` like we did before, because this would conflict with the `mod proj` import, but we need to use a global scope identifier `::` in `use ::proj::run`, to identify that we are referring to the `proj` module that is in the global scope, thus not under the current crate.

## Nested submodules and folders

When we write:

```rust
// lib.rs
use self::mod1;
```

the Rust compiler will do the following:

1. check if there is a `mod1.rs` file in the current directory, and in that case load it
2. check if there is a `mod1` directory containing a `mod.rs` file, and in that case load it

this means that we can create complex submodules composed of many files, just by grouping them under a directory with the module's name:

```
|-- lib.rs
|-- mod1
|   |-- mod.rs
|   |-- mod2.rs
```

So if we add:

```rust
// mod1/mod.rs
pub mod mod2
```

then we can access `mod2` from the crate root as well:

```rust
// lib.rs
use self::mod1;
use self::mod1::mod2;
```

Another way to achieve the same result is to have a `mod1.rs` file at the same level as a `mod1` directory, replacing the `mod.rs`:

```
|-- lib.rs
|-- mod1
|   |-- mod2.rs
|-- mod1.rs
```

where:

```rust
// mod1.rs
pub mod mod2;
```
