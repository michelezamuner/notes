# Expressions

## Literals

Literals are a way to describe values directly in the source code, and they are expressions that are evaluated to the value they represent:

```rust
let a = 2;
let s = "hey";
```

Here both `2` and `"hey"` are literal expressions, which are evaluated to the value `2` of type `i32` and the value `"hey"` of type `&str`.

## Function calls and operators

A function call is an expression, which is evaluated to the function's return value:

```rust
produce_result() // evaluated to the function's return value
```

Likewise, using an operator produces an expression:

```rust
1 + 2 // evaluated to 3
```

Notice that there's no semicolon `;` in the examples above: this is because adding a semicolon turns an expression into a statement, preventing them from being evaluated.

## Code blocks

A code block is a set of statements included in curly braces, and it's an expression. The value of a code block is either the value of the last expression, or the unit value `()` if the block has no expression as the last line:

```rust
{
  let x = 3;
  x
}
```

in this case the block is evaluated to the value of `x`, since the block ends with an expression (notice the absence of a semicolon, making it an expression rather than a statement).

A block can end with a statement, in which case it will be evaluated to the unit value `()`:

```rust
{
  let x = 4;
  let y = 5;
}
```

The unit value `()` represent the absence of a value, and is the only value available for the unit type `()`.

## `if` expressions

With `if` expressions it's possible to execute a part of a program according to the value of a boolean expression:

```rust
if number < 5 {
  println!("condition was true");
}
```

in this case, if the condition is met, i.e. the boolean expression is evaluated to `true`, then the code block is executed, otherwise it's just ignored.

With `if/else` expressions we can choose to execute a code block only when the condition is *not* met:

```rust
if number < 5 {
  println!("condition was true");
} else {
  println!("condition was false");
}
```

With `if/else if` expressions we can create many branches of execution according to a set of different conditions:

```rust
if number % 4 == 0 {
  println!("number is divisible by 4");
} else if number % 3 == 0 {
  println!("number is divisible by 3");
} else if number % 2 == 0 {
  println!("number is divisible by 2");
} else {
  println!("number is not divisible by 4, 3, or 2");
}
```

Since all these branches are exclusives, at most one of them will be executed, after which the execution will resume to the code after the entire `if` block.

When an `if` block contains one global `else` branch, such that at least one branch will be executed, and all branches evaluate to the same type, then the `if` block can be evaluated to the value produced by the selected branch. This means that the `if` block can be assigned to a variable, for example:

```rust
let number = if condition { 5 } else { 6 };
```

If different branches produced values of different types, the compiler couldn't infer the type of the assigned variable:

```rust
// ERROR: `if` and `else` have incompatible types
let number = if condition { 5 } else { "hey" };
```

If it was possible for the execution not to enter any branch, the compiler wouldn't know what value to evaluate the expression to:

```rust
// ERROR: `if` may be missing an `else` clause
let number = if condition { 5 };
```

## Loops

The `loop` keyword can be used to repeat a block of code indefinitely:

```rust
loop {
  println!("again");
}
```

`loop` constructs are also expressions, because they can be evaluated to the value returned with the `break` keyword:

```rust
let result = loop {
  counter += 1;
  if counter == 10 {
    break counter * 2;
  }
};
```

### `while` loops

Something like this can be expressed more concisely with a `while` loop:

```rust
while number != 0 {
  println!("{}!", number);
  number = number - 1;
}
```

While `while` is still technically an expression, it's always evaluated to the unit type `()`, and it's not possible to produce a different value from it, not even by using `break`.

When we need to loop over an iterable data structure, we can use `for` instead:

```rust
let a = [10, 20, 30, 40, 50];
for element in a.iter() {
  println!("the value is: {}", element);
}
```

Like `while`, also `for` is an expression that will only produce the unit value `()`.

### `for` loops

In a `for` loop, the part before `in` is a pattern, and `in` performs pattern matching with the value after the `in`:

```rust
for (index, value) in v.iter().enumerate() {
  // ...
}
```

here we're matching the pattern `(index, value)` with every value of the iterator produced by `v.iter().enumerate()`; only when the value doesn't match the pattern any more, the loop is exited.

Since a `for` loop can't do anything meaningful with a value that doesn't match, `for` loops must use irrefutable patterns, meaning patterns that always match.

## Match

The `match` expression is used to branch execution according to how a given value matches a pattern:

```rust
let value: i32 = match coin {
  Coin::Penny => 1,
  Coin::Nickel => 5,
  Coin::Dime => 10,
  Coin::Quarter => 25,
};
```

Unlike `if`, with `match` the expression we are analyzing can be of any type; on each branch of `match` we have a pattern on the left, like `Coin::Penny`, and a block of code on the right, that will be executed if the given value matches that pattern.

In a `match` block, the first branch matching the pattern is executed, and the value produced by that branch is returned as the value of the entire `match` expression.

Patterns can be used to extract inner values from data structures, and use them in the code block:

```rust
let result: i32 = match x {
  None => 0,
  Some(i) => i,
};
```

This works because values like `Some(1)`, `Some(2)`, etc., all match the generic `Some(i)`; additionally, while doing the matching, the variable `i` is populated with the corresponding value, like `1`, `2`, etc.

`match` expressions require that the patterns we defined exhaustively cover all possible values that might be passed:

```rust
let result: i32 = match x {
  // error: non-exhaustive patterns: `None` not covered
  Some(i) => i,
}
```

here the compiler throws an error because `x` can be different than `Some(i)`: in particular it could also be `None`, but our match didn't cover that case.

We can use the `_` catch-all pattern to cover all remaining cases:

```rust
let some_u8_value = 0u8;
match some_u8_value {
  1 => println!("one"),
  3 => println!("three"),
  5 => println!("five"),
  7 => println!("seven"),
  _ => ()
}
```

here we explicitly listed only some of all the possible values that a `u8` can be: if `some_u8_value` comes with a value that is not one of those, it will then match the catch-all pattern `_`.

We can use the `_` pattern also nested inside data structures:

```rust
match (setting_value, new_setting_value) {
  (Some(_), Some(_)) => {
    // ...
  },
  _ => {
    // ...
  }
}
```

We can ignore the remaining parts of a value with `..`:

```rust
match origin {
  Point { x, .. } => {
    // ...
  }
}
```

or

```rust
match numbers {
  (first, .., last) => {
    // ...
  }
}
```

We can have multiple patterns in the same branch:

```rust
match x {
  1 | 2 => println!("one or two"),
  3 => println!("three"),
  _ => println!("anything"),
}
```

We can match a range of values:

```rust
match x {
  1 .. 5 => println!("one through five"),
  _ => println!("something else"),
}
```

We can define *match guards*:

```rust
match num {
  Some(x) if x < 5 => println!("less than five: {}", x),
  Some(x) => println!("{}", x),
  None => (),
}
```

match guards can also be applied to multiple patterns:

```rust
let y = true;
match x {
  4|5|6 if y == true => println!("yes"),
  _ => println!("no"),
}
```

We can use the `@` operator to capture a value while it's being matched to a pattern:

```rust
match msg {
  Message::Hello { id: id_variable @ 3..7 } => {
    println!("Found an id in range: {}", id_variable)
  },
  Message::Hello { id: 10..12 } => {
    println!("Found an id in another range")
  },
  Message::Hello { id } => {
    println!("Found some other id: {}", id)
  },
}
```

### `if let`

There are cases where we want to do something when a variable has a certain value, but do nothing in all the other cases:

```rust
let some_u8_value = Some(0u8);
match some_u8_value = {
  Some(3) => println!("three"),
  _ => (),
}
```

here we're only interested in the case when `some_u8_value` is `3`, so we use the `_` pattern to discard all other cases.

However, we can achieve the same result with a simpler syntax:

```rust
if let Some(3) = some_u8_value {
  println!("three");
}
```

An `if let` expression also allows an `else` branch, working pretty much like the `_` pattern:

```rust
let mut count = 0;
if let Coin::Quarter(state) = coin {
  println!("State quarter from {:?}!", state);
} else {
  count += 1;
}
```

An `if let` expression can only work with refutable patterns: there must be a way to make the pattern fail, otherwise the condition would always be true, and it wouldn't make sense to use an `if`.

### `while let`

`while let` expressions work like `if let`, but for loops:

```rust
while let Some(top) = stack.pop() {
  // ...
}
```

here the  `while` loop keeps cycling until the pattern `Some(top)` doesn't match anymore with the given value, meaning when the `stack.pop()` call returns `None` because there are no values left on the stack.

A `while let` expression can only work with refutable patterns: there must be a way to make the pattern fail, otherwise the condition would always be true, and the loop would never end.

## Closures

A closure can be defined in Rust like this:

```rust
let expensive_closure = |num| {
  println!("calculating slowly...");
  thread::sleep(Duration::from_secs(2));
  num
};
```

With closure we're not required to annotate the types of the parameters, nor the type of the return value, the reason being that closures are meant to be immediately used inside a limited context, and not to represent official interfaces between a consumer and a provider; also, being always part of a specific context, the compiler is always able to infer the types of the parameters.

It's still possible to add type annotations to closures though:

```rust
let expensive_closure = |num: u32| -> u32 {
  // ...
}
```

If the body of the closure is a one-liner, we don't need to add curly braces:

```rust
let add_one = |x| x + 1;
```

### Captures

Closures are expressions that are evaluated to values, representing functions definitions. These values have types that implement at least one of these traits: `Fn`, `FnMut` or `FnOnce`. This is needed when we want to store a closure as a struct field:

```rust
struct Catcher<T>
  where T: Fn(u32) -> u32
{
  calculation: T,
  value: Option<u32>,
}
```

The difference between these traits is related to the values captured by the closures. Closures can capture variables belonging to the context where the closure has been defined:

```rust
let s = String::from("hello");
let f = || s;
```

Here the closure `|| s` is capturing the variable `s`, which belongs to the outer scope.

Capturing a variable is like passing it to a function: if the type of the variable doesn't implement `Copy`, it gets moved into the context of the closure when the closure is called. This means that we cannot call the closure twice:

```rust
// error: use of moved value: `f`
println!("{} {}", f(s), f(s));
```

In this case, the closure that we defined implements the `FnOnce` trait, which means exactly that we cannot call that closure twice, because it moves some captured value:

```rust
struct C<F> where F: FnOnce() -> String {
  pub f: F,
}

let v = String::from("hello");
let c = C{ f: || v };
```

We can still capture a value by borrowing it immutably; in this case the closure implements `Fn`:

```rust
struct C<'a, F> where F: Fn() -> &'a str {
  pub f: F,
}

let v = String::from("hello");
let c = C{ f: || &v };
```

Although `FnOnce` is meant to represent closures that cannot be called more than once, in Rust `Fn` is a subtrait of `FnOnce`, meaning that we can also use an `Fn` where a `FnOnce` is require instead:

```rust
struct C<'a, F> where F: FnOnce() -> &'a str {
  pub f: F,
}

let v = String::from("hello");
let c = C{f: || &v };
```

When we want to capture a variable with a mutable borrow, the closure implements `FnMut`:

```rust
struct C<F> where F: FnMut() -> () {
  pub f: F,
}

let mut v = vec![1, 2];
let mut c = C{ f: || { &v.push(2); } };
```

`FnMut` is subtrait of `FnOnce`, so we can pass a closure that has a mutable borrow where a `FnOnce` is required, too.

Sometimes the body of a closure might just require to borrow the captured value, and not move it:

```rust
let data = vec![1, 2, 3];
let closure = || println!("{:?}", data);
```

here `println!()` doesn't move data, so we're borrowing it immutably. If for some reason we want to move `data` inside the closure instead, we can do it by using the `move` keyword:

```rust
let closure = move || println!("{:?}", data);
```

here `data` is moved into the closure even if the body of the closure didn't require it.
