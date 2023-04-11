# Debugging

The easiest way to debug a Rust program is to print the values of some variables on the standard output:

```rust
let a = 5;
println!("{}", a); // prints: 5
```

To print composite types we need to use the debug formatter:

```rust
let a = (1, 2);
println!("{:?}", a); // prints: (1, 2)
```

For custom structs we need to derive the `Debug` trait in order to use the debug formatter:

```rust
#[derive(Debug)]
struct Rectangle {
  width: i32,
  length: i32,
}

let r = Rectangle { width: 30, length: 50 };

println("{:?}", r); // prints: Rectangle { width: 30, height: 50 }
```

For a prettier display of bigger data structures we can use another formatter:

```rust
println("{:#?}", r);
// prints:
// Rectangle {
//   width: 30
//   height: 50
// }
}
```
