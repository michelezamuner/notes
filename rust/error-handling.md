# Error handling

During development it's best to let the program fail as fast as possible to quickly point out where unrecoverable errors, which are actually bugs of the program that needs to be fixed instead of handled, are located. In order to do this, Rust provides the `panic!()` macro, which causes a sudden termination of the program, while displaying the given error message:

```rust
fn main() {
  panic!("crash and burn");
}
```

many built-in functions and operators causes panics when unrecoverable error situations happen, like trying to use an out-of-bounds index on an array.

```rust
fn main() {
  let v = vec![1, 2, 3];

  v[99];
}
```

To handle recoverable errors, on the other hand, Rust provides no such mechanisms as exceptions catching, like other languages do, instead it relies on explicit returning objects that can be errors, but with some perks. To model a value that might not be there because an error happened, Rust provides the `Result` enum:

```rust
enum Result<T, E> {
  Ok(T),
  Err(E),
}
```

When we call a function that might produce a recoverable error, the function does not return the requested value, but actually an instance of `Result`:

```rust
let f: Result<std::fs::File, std::io::Error> = std::fs::File::open("hello.txt");
```

If the call to `open()` is successful, `f` has type `Ok<std::fs::File>`, while if instead the call produced an error, `f` has type `Err<std::io::Error>`. What we can do with this is a match for example:

```rust
let f = match f {
  Ok(file) => file,
  Err(error) => {
    // log error
  }
}
```

During development we might want to extract the `Result` value in a quicker way than building a match, and just panicking in case of error, for quick debugging:

```rust
let f = File::open("hello.txt").unwrap();
```

here if `open()` returns an instance of `Ok`, its inner value is returned, while if it's an instance of `Err`, the system will panic with a standard error message.

```rust
let f = File::open("hello.txt").expect("Failed to open hello.txt");
```

here, `expect()` works exactly as `unwrap()`, with the only difference that the error message printed during the panic is the one we wrote.

A common pattern in Rust programs is to immediately return the `Err` instance to the calling function, in case an error happened, and instead proceeding with the computation if a valid value was produced:

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
  let mut f = File::open("hello.txt")?;
  let mut s = String::new();
  f.read_to_string(&mut s)?;
  Ok(s)
}
```

here the `File::open()` call returns a `Result`: with the `?` operator, if `Result` is actually an `Ok`, it's unwrapped and its value is returned, so that it's assigned to `f`, while if `Result` is an `Err`, it's not unwrapped but it's immediately returned to the caller; the same thing happens for `read_to_string`, which will induce `read_username_from_file()` to return early with an `Err` in case `read_to_string` fails; then, since our `read_username_from_file` function must return a `Result` as well, once we have produced the successful value `s`, we need to wrap it into an `Ok` in order to return it.

Additionally, the `?` operator calls the `from` function of `Err`, which converts the type of error returned into the type declared by the function, as long as the error type that we want to return implements the `From` trait.
