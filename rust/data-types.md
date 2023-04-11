# Data types

## Type declaration

Every value in Rust has a *data type*, that must be known at compile time, in order for the type checker to verify the semantic correctness of the program.

Rust supports *type inference*, allowing the programmer to avoid explicitly specifying the type of a variable when it can be inferred by the compiler:

```rust
let x = 5; // inferred type: i32
```

However, sometimes type inference cannot automatically determine the type of a value, and thus the programmer is required to explicitly state it:

```rust
let guess: u32 = "42".parse().expect("Not a number!");
```

Data types can be separated between *scalar types*, representing single values, and *composite types*, representing data structures possibly composed of multiple values.

## Aliases

We can define aliases for types:

```rust
type Kilometers = i32;
```

allowing us to use the new name in place of the other one:

```rust
let y: Kilometers = 5;
```

This is particularly useful to avoid repetition:

```rust
type Thunk = Box<dyn Fn() + Send + 'static>;

let f: Thunk = Box::new(|| println!("hi"));
```

## Scalar types

Scalar types comprise values that represent single elements, and not collections or compounds of distinct elements.

For this reason, scalar values always have a fixed size, known at compile-time, and thus can always be allocated on the stack.

### Numeric types

*Integers* come with signed and unsigned variants; signed integers are implemented with two's complement representation:

- `i8`, `u8`: signed and unsigned 8-bit integers
- `i16`, `u16`: signed and unsigned 16-bit integers
- `i32`, `u32`: signed and unsigned 32-bit integers
- `i64`, `u64`: signed and unsigned 64-bit integers
- `i128`, `u128`: signed and unsigned 128-bit integers
- `isize`, `usize`: signed and unsigned integers of architecture-dependent size

Integers can be represented directly in the source code via literals, with these formats:

- decimal: es. `98_222`
- hexadecimal: es. `0xff`
- octal: es. `0o77`
- binary: es. `0b1111_0000`
- byte: es. `b'A'`

Types can be declared as part of the literal as well:

```rust
let a = 98_222_i32;
let b = 12_usize;
```

The default integer type that is chosen by type inference is `i32`. When integer overflow happens, and the application is compiled in debug mode, the runtime is interrupted by a panic, in order to signal the occurrence of the overflow; if the application is compiled in release mode, instead, the overflow is handled with two's complement wrapping.

*Floating-point types* in Rust are:

- `f32`: 32 bits single precision floating-point numbers
- `f64`: 64 bits double precision floating-point numbers

and can be represented with the usual literals:

```rust
let a = 1.2_f32;
let b = 0.01_f64;
```

Numeric types support the usual arithmetic operators, with infix notation. Like many other languages, Rust support operator overloading as well, meaning that the same operator, like `+`, can be used with multiple different numeric types.

Unlike other languages, in Rust there is no automatic conversion between types, so you cannot just sum together a `i32` and a `u64`, but you have to make an explicit conversion:

```rust
let a: i32 = 1;
let b: u64 = 1;
// error: no implementation for `i32 + u64`
a + b;
```

Unlike some other languages, in Rust we can call methods on number literals:

```rust
let a = 24.5_f32.round();
```

### Other scalar types

Rust has the `bool` type which has `true` and `false` as the only possible values.

The `char` type has a fixed length of 32 bits, and is used to represent Unicode characters, and its values can be represented with literals such as `'n'`, `'|'`, etc. or with Unicode codes such as `U+0000`, `U+D7FF`, etc.

## Fixed-size composite types

Unlike scalar types, composite types represent collections of multiple elements.

Rust provides some composite types that have nevertheless a fixed size, and can thus still be allocated on the stack.

### Tuples

The *tuple* type groups together different values of different types:

```rust
let tup = ('a', 1);
```

The type of a tuple is written as a comma-separated list of types:

```rust
let tup: (char, i32) = ('a', 1);
```

Since the type indexes the types of the inner elements, the size of a tuple has to be constant, meaning that we cannot add or remove elements from a tuple, otherwise its type would change.

Tuples support pattern matching to destructure themselves into their elements:

```rust
let (a, b, c) = (1, 'a', true);
println!("{}", a); // prints 1
```

Alternatively, we can directly access specific elements of a tuple:

```rust
let x: (i32, f64, u8) = (500, 6.4, 1);
let five_hundred = x.0; // 500
let six_point_four = x.1; // 6.4
let one = x.2; // 1
```

### Arrays

The *array* type represents a collection of a fixed number of elements, all of the same type:

```rust
let a: [i32; 5] = [1, 2, 3, 4, 5];
```

The type of an array is written as `[i32; 5]`, providing the type of its elements, and the number of elements, separated by a semicolon. The same syntax can be used to initialize an array with all equal elements:

```rust
let a: [i32, 5] = [3; 5]; // equivalent to  [3, 3, 3, 3, 3]
```

Arrays elements can be accessed with the indexing operator `[]`:

```rust
let a = [1, 2, 3, 4, 5];

let first = a[0];
let second = a[1];
```

Trying to use an index that is out of the array bounds will produce a runtime error:

```rust
let a = [1, 2, 3, 4, 5];
let index = get_index(); // for example index is 5
let error = a[index]; // runtime error: thread '<main>' panicked at 'index out of bounds: the len is 5 but the index is 5'
```

Since the actual index we're using might come from some dynamic data (user input, a database, etc.), the compiler cannot check if the call `a[index]` is valid or not. For this reason, we should always use arrays in production code very cautiously.

### Slices

A slice is a reference to a contiguous area of memory, of fixed size. For example we can create a slice out of an array:

```rust
let a = [1, 2, 3, 4, 5];

let slice_1:&[i32] = &a[..3]; // [1, 2, 3]
let slice_2:&[i32] = &a[3..]; // [4, 5]
let slice_3:&[i32] = &a[1..4]; // [2, 3, 4]
let slice_4:&[i32] = &a[..]; // [1, 2, 3, 4, 5]
```

in a way, slices are specific "views" of an array in memory. Since an array is allocated in the static memory (due to its fixed size), then array slices will just be selected views of that memory area.

We can cut slices from an existing reference as well:

```rust
let v = [1, 2, 3, 4, 5];
let vr = &v;
println!("{:?}", &vr[..2]); // [1, 2]
```

Like when indexing an array, also when we cut a slice we might accidentally use invalid range boundaries, for example trying to cut a slice that is bigger than the original array, and in this case the program would panic as well. So slices should be used with attention as well.

## Struct

Structs are a record-like data type:

```rust
struct User {
  username: String,
  email: String,
  sign_in_count: u64,
  active: bool,
}
```

### Creating structs

We can create an instance of a struct by specifying the actual values for its fields:

```rust
let user1 = User {
  email: String::from("someone@example.com"),
  username: String::from("someusername123"),
  active: true,
  sign_in_count: 1,
};
```

We can access struct's fields with the dot-notation:

```rust
println!("{}", user1.email);
```

We can change a struct's fields, provided it was defined as mutable:

```rust
let mut user1 = User {
  email: String::from("someone@example.com"),
  username: String::from("someusername123"),
  active: true,
  sign_in_count: 1,
};

user1.email = String::from("anotheremail@example.com");
```

It's not possible to have only certain fields mutable, and not the others.

If we have variables named the same as the struct fields, we can assign those variables to those fields with a shorthand notation:

```rust
fn build_user(email: String, username: String) -> User {
  User {
    email,
    username,
    active: true,
    sign_in_count: 1,
  }
}
```

We can conveniently create a new instance of a struct copying some fields of another existing instance:

```rust
let user2 = User {
  email: String::from("another@example.com"),
  username: String::from("anotherusername567"),
  ..user1
}
```

### Tuple structs and unit-like structs

We can create structs with unnamed fields, that look much like tuples, and are called *tuple structs*:

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);
```

Like enums, tuple structs also have initializers, which can be passed to functions in order to create new instances of a struct:

```rust
my_func(C);
```

We can define structs with no fields, named *unit-like structs*:

```rust
struct S;

let s = S{};
```

### Methods

Structs can have *methods*:

```rust
struct Rectangle {
  width: u32,
  height: u32,
}

impl Rectangle {
  fn area(&self) -> u32 {
    self.width * self.height
  }
}
```

where `self` refers to a specific instance of the struct, and thus doesn't need to be told what's its type.

Methods are called using the `.` operator on a struct instance:

```rust
let rect1 = Rectangle { width: 30, height: 50 };
rect1.area()
```

It's common for method to take only references to the invocation instance, like with `&self`, in order to allow the caller to keep using the instance. A method would take ownership of the instance with `self` for example when transforming the instance into something else, in which case we expect the caller to stop using the original instance, and start using the new value instead.

When calling `rect1.area`, Rust automatically manipulates the `rect1` variable in order for it to match the first argument of `area()`. In this case, since `area()` requires `&self`, meaning a reference to the instance, instead of the instance itself, Rust automatically uses `&rect1` instead of `rect1`, without requiring us to do anything. Had the method required `self`, while we were passing a reference like `&rect1`, Rust would've automatically dereferenced it with `*(&rect1)`, in order to get the instance itself, required by `self`.

We can define methods taking arguments (in addition to `self`):

```rust
impl Rectangle {
  fn can_hold(&self, other: &Rectangle) -> bool {
    // ...
  }
}
```

### Associated functions

Structs can also be bound to functions that are not meant to be called on the invocation instance, *associated functions*:

```rust
impl Rectangle {
  fn square(size: u32) -> Rectangle {
    Rectangle { width: size, height: size }
  }
}
```

Associated functions are often used as named constructors, and are called using the `::` operator on the struct name:

```rust
let sq = Rectangle::square(3);
```

## Enums

*Enums* allow to define a type by enumerating all its possible values:

```rust
enum IpAddrKind {
  V4,
  V6,
}
```

We can refer to enumerated values by using the `::` operator on the enum name:

```rust
let four = IpAddrKind::V4;
let six = IpAddrKind::v6;
```

These values will be of the type of the enumeration, so we can pass them to functions taking arguments of that type:

```rust
fn route(ip_kind: IpAddrKind) {
  // ...
}
```

Enum values can hold data inside them, of different types:

```rust
enum IpAddr {
  V4(u8, u8, u8, u8),
  V6(String),
}

let home: IpAddr::V4(127, 0, 0, 1);
let loopback: IpAddr::V6(String::from("::1"));
```

here the syntax `V6(String)` defines an *initializer*, meaning a way to initialize the enum variant. We can pass initializers where functions are required as arguments:

```rust
my_func(IpAddr::V4);
```

and this can be used inside `my_func()` to create a new instance of the `IpAddr::V4` variant from some given initialization value.

Enum can define methods as well:

```rust
enum Message {
  Quit,
  Move { x: i32, y: i32 },
  Write(String),
  ChangeColor(i32, i32, i32),
}
impl Message {
  fn call(&self) {
    // ...
  }
}
let m = Message::Write(String::from("hello"));
m.call();
```
