# Statements

Any expression can be turned into a valid statement by adding a semicolon `;` at the end; this doesn't mean that the statement would be semantically meaningful though:

```rust
1 + 2; // this is successfully compiled as a statement
```

## Functions

Functions definitions in Rust must start with the keyword `fn`, followed by the identifier for the function, a set of parenthesis and the body of the function between curly braces, containing a sequence of statements:

```rust
fn some_function() {
  // ...
}
```

To call a function we use the identifier of the function, followed by a set of parenthesis:

```rust
some_function();
```

A function can be called either before or after its definition:

```rust
fn main() {
  some_function();
}

fn some_function() {
  // ...
}
```

Function calls must always be placed in local scope.

### Function arguments

Functions can be defined to have arguments, as a comma-separated list of variables declarations inside parenthesis:

```rust
fn some_function(x: i32, y: i32) {
  // ...
}
```

at that point, all arguments are available as local variables inside the function's body. We can call a function with arguments by passing them parameters accordingly:

```rust
some_function(5, 6);
```

Although in most cases the compiler could infer the type of an argument from the type of the parameters, by design Rust requires to explicitly state arguments types in the function definition to avoid having to declare them in random place around the code, when the compiler actually cannot infer them, which would make the code much harder to understand.

Functions arguments are actually patterns, and passing a value to a function argument actually matches the value to that pattern:

```rust
fn print_coordinates(&(x, y): &(i32, i32)) {
  println!("Current location: ({}, {})", x, y);
}
```

here the value that is passed to `print_coordinates()` is matched against the pattern `(x, y)`, meaning that the given tuple is automatically destructured and its values are bound to the local variables `x` and `y`, which are then used in the `println!()` macro.

Since the function can't do anything meaningful with a value that doesn't match, function arguments must be irrefutable patterns, meaning patterns that always match.

### Nested functions

Since function definitions are statements, we can put a function definition inside the body of another function definition:

```rust
fn main() {
  fn some_function() {
    // ...
  }
  some_function();
}
```

### Returning values

The body of a function can interrupt its execution before the last line by using the keyword `return`:

```rust
fn some_function() {
  let x = 5;
  return;
  let x = 6; // unreachable statement
}
```

Since the control of the execution returns to the calling code when the `return` keyword is reached, any other statement placed after `return` is unreachable, meaning that it cannot be executed.

A function can be defined so that it returns a value; in this case we must specify the return type by adding `->` followed by the return type after the arguments list:

```rust
fn some_function() -> i32 {
  // ...
}
```

When a return type is declared, the body of the function must abide by it, and return a value of the declared type, either as the last expression, or with the `return` keyword:

```rust
fn some_function() -> i32 {
  2
}
```

or

```rust
fn some_function() -> i32 {
  return 2;
}
```

The value thus returned will be the value the function will be evaluated to when it's called:

```rust
fn some_function() -> i32 {
  2
}

fn main() {
  let x = some_function(); // evaluated to 2
}
```

The `return` keyword works only within functions bodies: it cannot be used to break early out of a generic code block expression.

### Function pointers

The name of a function is also a pointer to that same function, meaning that we can pass a function to another function:

```rust
fn add_one(x: i32) -> i32 {
  x + 1
}

fn do_twice(f: fn(i32) -> i32, arg: i32) -> i32 {
  f(arg) + f(arg)
}

do_twice(add_one, 5);
```

here the type of the `add_one()` function is written as `fn(i32) -> i32`.

## Assignment statements

An *assignment statement* is used to bind a value to an identifier. For example we can assign values to *variables*:

```rust
let x = 5;
```

Variables in Rust can only be declared in a local scope, i.e. inside the body of a function, and not in the global scope:

```rust
// main.rs
let x = 6; // compiler error: expected item, found keyword `let`

fn main() {
  let x = 5;
}
```

In a `let` statement the part before the `=` sign is actually a pattern, and `=` performs pattern matching with the value to the right of it:

```rust
let (x, y, z) = (1, 2, 3);
```

Since an assignment can't do anything meaningful with a value that doesn't match, assignments must contain irrefutable patterns, meaning patterns that always match.

### Mutability

Variables are immutable by default, meaning that we cannot assign a different value to a variable that have already been assigned.

```rust
let x = 5;
x = 6; // compiler error: cannot assign twice to immutable variable
```

In order to declare that a variable can be reassigned, we write:

```rust
let mut x = 5;
x = 6;
```

### Constants

In Rust we can also define *constants*, which are always evaluated at compile time, and thus must always have a type annotation and a value that can be evaluated at compile time; additionally, constants can be defined in the global scope:

```rust
// main.rs
const MAX_POINTS: u32 = 100_000;

fn main() {
  const MY_CONST: String = String::from("value"); // compiler error: calls in constants are limited to constant functions, tuple structs and tuple variants
}
```

Constant values are created when the execution reaches the constant definition statement, and live up to the termination of the program, but are only accessible from the scope they're defined in.

### Shadowing

Rust supports variables *shadowing*, meaning that we can have multiple definitions of a variable with the same name in the same scope, which can be useful to allow making some changes to a value without having to come up with different names each time:

```rust
let mut guess = String::new();
// ...
let guess: u32 = guess.trim().parse().expect("Please type a number!");
```

in this case we changed the type of `guess` from `mut String` to `u32`: notice that the variable has also become immutable. However, we can use shadowing also with variables of the same type:

```rust
let x = 5;
let x = x + 1;
let x = x * 2;
```

this way, we can make controlled changes to the value of a variable without making it mutable.

### Assigning variables to variables

Variables can be assigned to other variables:

```rust
let s1 = "hello";
let s2 = s1;
```

For stack-allocated values this means that the value of the first variable is copied on the stack, and then bound to the second variable.

With heap-allocated values the same thing happens, but with an important implication:

```rust
let s1 = String::from("hello");
let s2 = s1;
```

Here `s1` contains a fixed-size data structure allocated on the stack, that contains a pointer to the actual bytes of the string on the heap, along with some other information:

```
s1: {
  ptr: 0xa6f3093c312
  len: 5
  capacity: 5
}
```

When we assign `s1` to `s2`, only this data structure on the stack is copied, exactly as it happens with scalar types, but no copy of the heap memory is ever performed. This means that Rust by default never performs deep copies of data structures.
