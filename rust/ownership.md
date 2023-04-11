# Ownership

While a program runs, usually memory needs to be allocated for various reasons, for example to hold function arguments and return values, or to store local variables.

In many cases, when the program execution as reached a certain point, the data allocated cannot be retrieved any longer, and then the memory used to store it needs to be freed, in order to be available again to be used by other parts of the program.

Many languages do this by using a garbage collector, that periodically scans the memory to see which is not being used any more, and marks it as free. This solution is very useful for the programmer, because it allows them not to worry about this problem at all, but applies an important performance burden to the program's execution.

The most common alternative that can be used to avoid the performance cost, is manual memory management, which means that the programmer must take the responsibility of manually allocating and freeing memory every time.

Rust takes a third approach, that is a bit of a mix of the former two. Memory is allocated and freed automatically, but this is done following a set of rules that allow to know at compile time exactly when memory can be freed, and thus avoiding having to periodically run a garbage collector to scan memory.

Additionally, with ownership rules it's easier to work more with the stack than with the heap, improving performances even more.

When a dynamic type is created, for example with `String::from("Hello, world!")`, the operating system is asked to find space on the heap to store this value, according to the actual size it needs to have at runtime.

However, at a certain point the variable holding this dynamic type will not longer be needed, for example because it's gone out of scope and it's not usable any more. When this happens, the memory reserved to it must be freed to allow other data to be stored there.

This proved to be a hard problem to solve, because of these reasons:

- if we forget to free allocated memory, we'll waste memory
- if we free memory too early, we'll have an invalid variable that will make the program crash when trying to access it
- if we free memory twice, we'll have a bug as well

This means that we need to make sure to have exactly one free operation following every allocate operation.

## Variable de-allocation

To ensure memory is always freed, Rust by default automatically frees variables when they go out of scope, i.e. when the execution reaches the end of the code block where the variable has been defined:

```rust
{
  // variable 's' is allocated here
  let s = String::from("hello");
}
// variable 's' is gone out of scope here,
// so it's memory will be released
```

In particular, there's a function `String::drop()` that is called when a string is freed, which will perform all operations that are needed to properly free the memory that was allocated by the string.

This way we are sure that memory is always freed (and thus we're not wasting it), and that it's not freed when the variable can still be used.

However, this solution still doesn't prevent us from running into the double-free bug:

```rust
let s1 = String::from("hello");
let s2 = s1;
```

Here `s1` and `s2` end up pointing to the same memory on the heap, meaning that when they go out of scope the same memory will be freed twice.

## Move

To prevent the double-free bug from happening, in Rust when `s1` is assigned to `s2`, `s1` is decoupled from the memory it was pointing to, so that when it goes out of scope it won't need to be freed anymore

This leaves only `s2` with a valid pointer to the memory on the heap, meaning that only one free operation will need to be performed at scope exit.

In this case, we say that the ownership of the value is *moved* from `s1` to `s2`. When this happens, `s1` cannot be used any longer:

```rust
let s1 = String::from("hello");
let s2 = s1;

// borrow of moved value: `s1`
println!("{}", s1);
```

We might want to be able to keep using a variable after having assigned it to another one. In this case, though, we need to perform a deep copy of the value at the moment of the assignment:

```rust
let s1 = String::from("hello");
let s2 = s1.clone();

println!("{}", s1);
```

Here `s2` is not just a copy of the `s1` pointer, but it's a whole different pointer, referencing a different area on the heap, where the original value has been fully copied. This way, when going out of scope, both `s1` and `s2` can be independently freed, and thus `s1` can still be used even after `s2` has been declared.

Moving also happens when passing value to functions, and when returning from functions:

```rust
{
  let s = String::from("hello");
  take_ownership(s);
}
```

here `s` is moved to `take_ownership`, meaning that the value pointed to by `s` won't be freed when going out of the scope where `s` was defined, but when going out of the scope of `take_ownership`.

```rust
fn give_ownership() -> String {
  let s = String::from("string");

  s
}
```

here `s` is moved from `give_ownership` to the caller, meaning that the value pointed to by `s` won't be freed when going out of the scope of `give_ownership`, but when going out of the scope of the caller.

## Borrow

When we pass a value to a function, we're actually moving that value to a different variable, which is the function argument, and thus we cannot use it any longer after that.

```rust
fn calculate_length(s: String) -> usize {
  s.len()
}
let s = String::from("hello");
let len = calculate_length(s);
// here `s` cannot be used anymore
```

If we really wanted to keep the ownership of a value to the calling scope, we'd have to do a *borrow* instead of a move, my passing a *reference* of that value, instead of the value itself:

```rust
fn calculate_length(s: &String) -> usize {
  s.len()
}

let s = String::from("hello");
let len = calculate_length(&s);
// here `s` is still available to be used
```

A reference to a value, like `&s`, points to a value, but does not own it. When a reference is assigned to a different variable, for example during a function call, the new variable does not own the original value, and as such when the variable goes out of scope it cannot take responsibility in freeing the memory used to hold the value: this means that no memory is freed, and thus the value can still be used by the original variable it was assigned to.

While references cannot free the value they're pointing to, because they don't own it, they still run the risk of ending up pointing to a value that has been freed by someone else (whoever owning it), thus becoming dangling references.

To prevent this case, the Rust compiler also checks that references do not go out of scope before the value they're pointing to does:

```rust
fn dangle() -> &String {
  let s = String::from("hello");

  // error: this function's return type contains a borrowed value, but there is no value for it to be borrowed from
  &s
}
```

here when the function returns `s` goes out of scope, and thus is freed, but its reference `&s` is returned to the caller, which would then use it to reference a value that doesn't exist anymore.

A reference to a value is actually another value of a different type than the value pointed to by the reference:

```rust
let x = 5;
let y = &x;

assert_eq!(5, x);
// error: can't compare `{integer}` with `&{integer}`
assert_eq!(5, y);
```

here the error is due to the fact that while `x` is an integer, `y` is a reference to an integer, which is a completely different type. To make the comparison work, we need to get from the reference `y` the value pointed to, and this is called *dereferencing*:

```rust
assert_eq!(5, *y);
```

here we use the *deref operator* `*` applied to a reference `y` in order to get the referenced value, which is indeed of integer type.

## Mutable references

When a value is borrowed through a reference, it cannot be changed:

```rust
fn change(s: &String) {
  s.push_str(", world");
}

let s = String::from("hello");
// error: cannot borrow `*s` as mutable, as it is behind a `&` reference
change(&s);
```

This happens simply because references are immutable by default, as well as regular variables. We can make this code work with a *mutable reference*:

```rust
fn change(s: &mut String) {
  s.push_str(", world");
}

let mut s = String::from("hello");
change(&mut s);
```

In order to do this, first we need to declare the original value as mutable, and then we need to take a mutable reference out of it, with `&mut`.

Mutable references come at a cost: there can only be one mutable reference of the same value at the same time. For example:

```rust
let mut s = String::from("hello");

let r1 = &mut s;
// error: cannot borrow `s` as mutable more than once at a time
let r2 = &mut s;

println!("{}, {}", r1, r2);
```

This limit is important to prevent data races: when two or more references exist of the same value, and some of them try to change the original value at the same time (for example because they are running in different threads), then the value would be changed in unexpected ways, that would also likely be different with every execution, and thus very hard to debug.

A similar issue happens when we have both an immutable, and a mutable reference:

```rust
let mut s = String::from("hello");
let r1 = &s;
// error: cannot borrow `s` as mutable because it is also borrowed as immutable
let r2 = &mut s;

println!("{}, {}", r1, r2);
```

The problem here is that `r1` expects the value to never change, exactly because a reference is immutable, but if `r2` performs a change of the value at the same time in a different thread, then this precondition is not guaranteed anymore.

## Smart pointers

Smart pointers are objects containing a pointer to some memory (like references), and some additional metadata. Smart pointers can be used to allow some value to have multiple owners, by using reference counting, and cleaning up memory once the last owner was freed. `String` and `Vec` are actually smart pointers, because they contain a reference to some memory, plus some metadata, like the capacity.

A smart pointer is usually implemented with a `struct` implementing the `Deref` and `Drop` traits. The `Deref` trait allows a type to be used like a reference, i.e. to be dereferenced; the `Drop` trait allows to write some code to be called once an object is freed.

The `Box<T>` type is a smart pointer that allows to store some data on the heap. This can be useful for:

- using values whose size cannot be known at compile time, where the type size is instead required
- moving ownership of large amounts of data, preventing it from being copied
- owning a value when we don't care about its type, but only about the traits it implements

```rust
let b = Box::new(5);
println!("b = {}", b);
```

here the value `5` is actually allocated on the heap, but we can access it as if it was a regular value. When the variable `b` goes out of scope, both the `Box` pointer and the data pointed to will be deallocated.

We can dereference a `Box` like we do with references:

```rust
let x = 5;
let y = Box::new(x);
assert_eq!(5, *y);
```

and this works because `Box` implements the `Deref` trait. The `Deref` trait requires a type to implement the method `deref(&self) -> &T`. Since `deref()` returns a reference to the original value, this means that when we write `*y` and `y` is a `Deref`, Rust actually does `*(y.deref())`, so behind the scenes smart pointers always fall back to dereferencing built-in references. The reason for this is that in Rust references are used to borrow values, so when we use references we want to avoid moving the ownership.

Rust provides *deref coercion*, which means that if we have a function or method taking a reference, we can pass to it another kind of smart pointers (targeting the same type), and Rust will automatically call `deref()` on it to turn it into a native reference:

```rust
let m = Box::new(String::from("hello"));
hello(&m);

fn hello(name: &str) {
  // ...
}
```

here our function `hello()` is defined to take a string slice, but we can still pass a smart pointer to a string (in this case a `Box<String>`) instead. This works because when we do this Rust automatically calls `Box<String>::deref()`, which produces a `&String`; then, since `&String` implements `deref()` as well, Rust also calls `&String::deref()`, producing the final `&str` value that we needed. If deref coercion weren't there, we'd have to write something like:

```rust
hello(&(*m)[..]);
```

where `*m` turns the `Box<String>` into a `String`, and `&String[..]` takes a slice of the string, of the entire size of the string.

Deref coercion happens also for mutable references, because mutable references implement `DerefMut<T>`: this means that a `&mut T` reference where `T: DerefMut<Target=U>`  can automatically be converted to a `&mut U` reference.

Additionally, mutable references can automatically be converted to immutable references as well, thus a `&mut T` reference where `T: DerefMut<Target=U>` can automatically be converted to a `&U` immutable reference. The opposite is not possible because we can only have one mutable reference to a value at a time, and if we converted an immutable reference to a mutable one, there could be another mutable reference to that value around, that we don't know about.

The `Rc<T>` smart pointer represents reference counting, and it's a way to allow a value to have multiple owners. An `Rc<T>` pointer keeps track of the number of references that points to the same value, and frees the memory allocated for that value only when there's no reference to it left.

```rust
use std::rc::Rc;

let value = /* ... */;
let a = Rc::new(value);
let b = Rc::clone(&a);
let c = Rc::clone(&a);
```

the first reference we create to our target value is constructed by calling `Rc::new()`. However, all other references we want to make to the same value will need to be created with `Rc::clone(&a)`, or alternatively `a.clone()`, and passing a reference to the first reference to avoid transferring ownership. When we call `Rc::clone()` the reference count for that particular value is increased by one. We can get the number of references pointing to a value at a certain moment:

```rust
println!("{}", Rc::strong_count(&a));
```

this value increases every time `Rc::clone()` is called, and decreases every time a reference goes out of scope and gets freed.
