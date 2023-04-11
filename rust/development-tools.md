# Development tools

## Installation and update

It's common when developing with a programming language to have the need to keep language binaries (compilers, runtimes, build tools, etc.)up to date with time. Unlike many traditional languages that either require manual intervention from the developer to update these binaries, or provide non-standard third-party tools to do this automatically (think of `nvm`, `n`, `rvm` etc.), Rust comes pre-packaged with a default version manager: `rustup`.

To download the latest version of `rustup`, head over to [https://rustup.rs](https://rustup.rs), and follow the instructions there. This will download an installation script, and if you follow its instructions you'll have both `rustup` and the latest version of Rust installed in your system. To check that you have the Rust compiler correctly installed:

```shell
$ rustc --version
rustc 1.59.0 (9d1b2106e 2022-02-23)
```

You can then use `rustup` from time to time to either check if there are new versions of Rust available:

```shell
$ rustup check
stable-x86_64-apple-darwin - Update available : 1.52.1 (9bc8c42bb 2021-05-09) -> 1.59.0 (9d1b2106e 2022-02-23)
rustup - Update available : 1.24.2 -> 1.24.3
```

and to update your local Rust tools:

```shell
rustup update
```

You can use `rustup` to uninstall all your local Rust tools (including `rustup` itself) as well:

```shell
rustup self uninstall
```

`rustup` also comes with a copy of the latest Rust documentation, that you can open locally:

```shell
rustup doc
```

this command will open the HTML version of the documentation in your default browser. You can also directly look for a specific topic:

```shell
rustup doc std::result
```

this won't work with any keyword, check `rustup doc --help` for more information.

## Project creation

Rust provides a complete build tool and package manager named `cargo`. We can scaffold a basic project with:

```shell
$ cargo new hello_cargo
    Created binary (application) `hello_cargo` package
```

This will create a new project directory `hello_cargo` containing a default `src/main.rs` source file with an "Hello, world!" program, the `Cargo.toml` configuration file, and initializing the repository with Git:

```shell
$ tree -a hello_cargo
hello_cargo/
├── .git
│   ├── ...
├── .gitignore
├── Cargo.toml
└── src
    └── main.rs

1 directory, 2 files
```

The `Cargo.toml` file contains basic information necessary to define, and possibly export, the project:

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
```

## Building

To build a simple Rust program made of just a `main.rs` file, and produce an executable we can directly use the `rustc` compiler:

```shell
$ rustc main.rs
$ ./main
"Hello, world!"
```

However, if we are managing our project with Cargo, we can just call:

```shell
$ cargo build
   Compiling hello_cargo v0.1.0 (/path/to/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 2.75s
```

The resulting binary will be placed in the `target/debug` folder of the project:

```shell
$ target/debug/hello_cargo
Hello, world!
```

Alternatively, we can also compile and run the project with a single command:

```shell
$ cargo run
   Compiling hello_cargo v0.1.0 (/path/to/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 1.65s
     Running `target/debug/hello_cargo`
Hello, world!
```

With this command, if the project is already built, Cargo will just run it instead.

If we're still developing, we can just check if the program compiles without wasting time actually building it, with:

```shell
$ cargo check
    Checking hello_cargo v0.1.0 (/path/to/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.27s
```

When the development is complete and we want to build our project with all optimizations needed for production use, we can call:

```shell
$ cargo build --release
   Compiling hello_cargo v0.1.0 (/path/to/hello_cargo)
    Finished release [optimized] target(s) in 0.81s
```

which will produce the final executable file inside `target/release` instead.

## Handling dependencies

To add an external crate to our Rust project we first need to edit the `Cargo.toml` file, adding the crate name in the `dependencies` section:

```toml
...
[dependencies]
rand = "0.3.14"
```

The crate version is interpreted using Semantic Versioning, and in this case writing `0.3.14` is equivalent to writing `^0.3.14`.

An external crate is just another Rust project that is built as a library instead of a binary (i.e. it has no `main` function defined).

Now, to actually download and build this new dependency we just need to run `cargo build` again:

```shell
$ cargo build
   Updating registry `https://github.com/rust-lang/crates.io-index`
 Downloading rand v0.3.14
 Downloading libc v0.2.14
   Compiling libc v0.2.14
   Compiling rand v0.3.14
   Compiling guessing_game v0.1.0 (/path/to/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 1.50 secs
```

External crates are fetched from `https://crates.io/`: Cargo maintains a local copy of the index of packages of `crates.io`, called the *registry*. Before doing any dependency update, Cargo updates the registry in order to have all the latest versions available, and then downloads the crates that we still don't have, at the right versions. Since external crates might in turn depend on other crates, those will need to be downloaded as well. After downloading all needed crates, Cargo compiles them, and then compiles our project, linking all external dependencies in the right places.

Once project dependencies, and all their sub-dependencies, have been selected and downloaded, Cargo writes their exact versions in the `Cargo.lock` file in order to have reproducible builds.

If we just saved the versions written in Semantic Versioning, Cargo would have to select again the proper versions of all dependencies, but could also download newer versions of the same dependencies when run at a later time on a different environment, because newer versions might have become available in the meanwhile, which might contain changes that make our project behave differently.

Again, if we run `cargo build` when the dependencies have already been downloaded, and no changes to `Cargo.toml` has been done in the meanwhile (and no changes to the code either), the command will just print the `Finished ...` line, since all the build would've already been done.

We can update existing dependencies using:

```shell
cargo update
```

This will update the registry, check which dependencies have a newer version available, according to the semantic versioning rules, download and compile it; the `Cargo.lock` file is updated as well. If we want to update to a version that is not accepted by our current versioning rule, we need to change it in the `Cargo.toml` file. For example, if our dependency has the version rule `0.3.14`, which means `^0.3.14` only versions `0.3.*` would be selected for download; thus, if we wanted for example version `0.4.0`, we'd have to update the dependency definition in `Cargo.toml` to `0.4.0`, and run `cargo build`. Finally, we can target a single dependency to be updated with:

```shell
cargo update rand
```
