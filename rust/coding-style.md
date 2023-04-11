# Coding style

Rust provides a standard coding style that you can optionally use in your project. A tool named `rustfmt` is available to automatically format your source files according to this standard style.

## Source files

In a Rust project, all source files must be named with *snake_case*, for example `hello_world.rs` rather than `helloworld.rs` or `helloWorld.rs`.

## Constants

Constants names must be named in uppercase, with words separated by underscores:

```rust
const MAX_POINTS: u32 = 100_000;
```

## Functions

Rust's coding style for functions is *snake_case*:

```rust
fn some_function() {
  // ...
}
```
