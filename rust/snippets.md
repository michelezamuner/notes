# Snippets

## Finding the type of a variable

```rust
use std::any::type_name;

fn type_of<T>(_: T) -> &'static str {
  type_name::<T>()
}

fn main() {
  let v = [1, 2, 3, 4, 5];
  let a = 21;
  println!("{}", type_of(a));
  println!("{}", type_of(&a));
  println!("{}", type_of(v));
  println!("{}", type_of(&v[1..4]));
  println!("{}", type_of("abc"));
  println!("{}", type_of(String::from("abc")));
  println!("{}", type_of(main));
}
```
