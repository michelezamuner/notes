# Generics and traits

## Generics

Generics are used to refer to types in the code without knowing exactly what concrete type will be used when the code will be called.

Generic types are conventionally identified with letters such as `T`, `U`, etc. Since these identifiers are no different from usual types identifiers, like `String`, we need a way to tell the compiler that these names refer to generic types that need to be inferred by looking at the code, instead of concrete types; in order to do this we put a list of the generic types that we're using inside angular brackets, like `<T, U>`.

### Generics with functions and enums

Generics can be used in functions definitions:

```rust
fn largest<T>(list: &[T]) -> T {
  // ...
}
```

here we defined a function `largest` that works with a generic type `T`, in the sense that it takes a slice of `T`'s and returns a `T`, whatever that `T` might be; thus, we can call this function with any possible type:

```rust
let number_list = vec![34, 50, 25, 100, 65];
let result = largest(&number_list);

let char_list = vec!['y', 'm', 'a', 'q'];
let result = largest(&char_list);
```

Generics can be used with enums:

```rust
enum Option<T> {
  Some(T),
  None
}
```

here we're defining the `Option` enum, that works with a generic type `T`, where `T` is the type of the value contained by the `Some` variant.

### Generics with structs

Generics can be used with structs:

```rust
struct Point<T> {
  x: T,
  y: T,
}

impl<T> Point<T> {
  fn x(&self) -> &T {
    &self.x
  }
}
```

We can implement methods only for a specific concrete type for a generic struct:

```rust
impl Point<f32> {
  fn distance_from_origin(&self) -> f32 {
    (self.x.powi(2) + self.y.powi(2)).sqrt()
  }
}
```

here this method `distance_from_origin()` will only exist for instances of `Point<T>`, where `T` is `i32`, and not in all other cases. Notice how we didn't need to write `impl<i32>`, since we aren't using any generic type name, and thus we didn't need to tell the compiler that a certain identifier is a generic type.

Parametric types used in the struct definition can be different than those used in the method definition:

```rust
impl<T, U> Point<T, U> {
  fn mixup<V, W> (self, other: Point<V, W>) -> Point<T, W> {
    // ...
  }
}
```

### Monomorphization

When code containing generics is compiled, the compiler looks through all the occurrences of the generic parameters to understand what are the types those parameters are actually turned into, and creates duplicated versions of the same code, replacing the generic parameters with the actual types:

```rust
fn largest<T>(list: &[T]) -> T { /* ... */}

let list1 = &[1, 2, 3];
let result1 = largest(list1);

let list2 = &["a", "b", "c"];
let result2 = largest(list2);
```

here the compiler sees that there are only two usages of the generic function `largest<T>`: one where `T` is `i32`, and another where `T` is `&str`; during compilation then, the generic function definition `largest<T>` is replaced with two concrete definitions, one for each concrete type this function has actually been used with:

```rust
fn largest_i32(list: &[i32]) -> i32 { /* ... */ }
fn largest_str(list: &[&str]) -> &str { /* ... */ }
```

This process is called *monomorphization* and allows to use generics with no runtime penalties, since generics are fully resolved at compile time.

## Traits

A trait defines an interface of features that can be shared among multiple types:

```rust
pub trait Summary {
  fn summarize(&self) -> String;
}
```

since a trait only describes the interface of a functionality, we don't add an implementation for it, in order to allow every different type to provide its own specific implementation.

A trait needs to be implemented by a concrete type:

```rust
// ...
impl Summary for NewsArticle {
  fn summarize(&self) -> String {
    format!("{}, by {} ({})", self.headline, self.author, self.location)
  }
}

// ...
impl Summary for Tweet {
  fn summarize(&self) -> String {
    format!("{}: {}", self.username, self.content)
  }
}
```

Traits implementations can be defined only if either the trait, or the type, are local to the current crate: this means that it's possible to implement external traits, like `Display`, for custom types, and it's possible to implement custom traits for external types, like `Vec<T>`. This is important in order to prevent two crates to implement the same traits for the same types, at which point the compiler wouldn't know which implementation to choose.

A trait that does not require any method to be implemented is known as a *marker trait*, meaning that it's only used to "mark" a type with some meta information:

```rust
pub trait MyMarker {}
```

### Default implementations

Traits accept default implementations for some of all of their methods:

```rust
pub trait Summary {
  fn summarize(&self) -> String {
    String::from("(Read more...)")
  }
}
```

which will automatically be available to all implementations. Default methods can still be overriden by specific implementations though. Default methods can call other non-default methods of the trait inside their body, since when the code will be called on a concrete type, those methods will have an actual implementation:

```rust
pub trait Summary {
  fn summarize_author(&self) -> String;

  fn summarize(&self) -> String {
    format!("(Read more from {}...)", self.summarize_author())
  }
}
```

### Syntactic sugar for traits

Traits can be used to specify that the argument of a function must be of a type that implements a certain trait:

```rust
pub fn notify(item: impl Summary) {
  println!("Breaking news! {}", item.summarize());
}
```

This is actually just syntactic sugar for this more general form:

```rust
pub fn notify<T: Summary>(item: T) {
  println!("Breaking news! {}", item.summarize());
}
```

If a type must implement more than one trait:

```rust
pub fn notify(item: impl Summary + Display) {
  // ...
}
```

or

```rust
pub fn notify<T: Summarize + Display>(item: T) {
  // ...
}
```

When there are many trait bounds to be specified:

```rust
fn some_function<T: Display + Clone, U: Clone + Debug>(t: T, u: U) -> i32 {
  // ...
}
```

we can use this alternative syntax instead:

```rust
fn some_function<T, U>(t: T, u: U) -> i32 {
  where T: Display + Clone,
    U: Clone + Debug
  // ...
}
```

### Trait bounds

We can use trait bounds to implement certain methods only for certain variations of a generic type:

```rust
use std::fmt::Display;

struct Pair<T> {
  x: T,
  y: T,
}

impl<T> Pair<T> {
  fn new(x: T, y: T) -> Self {
    Self {
      x,
      y,
    }
  }
}

impl<T: Display + PartialOrd> Pair<T> {
  fn cmp_display(&self) {
    // ...
  }
}
```

here the method `cmp_display()` is implemented only for those concrete types `Pair<T>` where `T` implements `Display` and `PartialOrd`. All other concrete types of `T` won't have the `cmp_display()` method available.

### Blanked implementation

We can implement a trait for another abstract type, called *blanked implementation*, as long as it implements another trait:

```rust
impl<T: Display> ToString for T {
  // ...
}
```

this example is part of the standard library, and means that we can call `to_string()` on any type that implements `Display`, without needing to repeat this implementation for every single type.

### Trait objects

When we want to allow a struct to have a known number of fields of unknown types, we can use generics:

```rust
struct S<T, U> {
  f1: T,
  f2: U
}
```

however, what if we have an unknown number of types, for example because the struct contains a collection of objects of different types?

```rust
struct S<T> {
  f: Vec<T>,
}
```

if we define the struct like this, all objects of the field `f` will have the exact same type, which is whatever type will be used in place of `T`. We can then use a trait, and declare that all objects of the collection will implement that trait, albeit being of different types:

```rust
trait T {
  fn f();
}

struct S {
  // error: the trait `T` cannot be made into an object
  f: Vec<Box<T>>,
}
```

since `T` is a trait and not a generic, it won't be replaced at compile time with a concrete type, and thus the compiler won't know how much space objects of type `T` will take on the stack, and thus it won't know how much memory to reserve for objects of the new struct `S`; for this reason, we store the objects on the heap, and handle a vector of references.

We still have a compilation error though, which is related to the fact that, although the size of the objects is known (they are references), the compiler doesn't know what methods the actual types will support, because we will know which concrete types of objects will be in the collection only at runtime.

When the type of a variable is known at compile time, Rust can use *static dispatch*, meaning that the connection between the variable and the implementation to use when a method is called on that variable is known at compile time, and thus Rust can store the address of the right function along with the object in the final executable.

However, in these cases where the type of a variable is known only at runtime, the compiler can only know that this variable will support a method `f()`, because this method is declared by the trait `T` of the variable, but it cannot know what will be the actual implementation of that method, and thus cannot store any function address along with each object. In this case Rust needs to resort to *dynamic dispatch*, which means that the compiler needs to create a *virtual table* that connects every concrete type implementing `T` with the address of the particular implementation of `f()` provided by that concrete type: when at runtime the function `f()` is called on an object of a concrete type implementing `T`, Rust checks in the virtual table what is the actual implementation of `f()` that needs to be called for that specific concrete type.

In order to tell Rust to use dynamic dispatch, then, we need to use a *trait object*:

```rust
struct S {
  f: Vec<Box<dyn T>>
}
```

here the `dyn` keyword informs Rust that the type of `Box<T>` is a trait object, meaning a type implementing that trait, for which dynamic dispatch needs to be used.

This way, at runtime we can add to the vector `f` pointers to objects of different types, as long as they all implement the trait `T`.

When we define a trait object, none of its methods can return `Self`, meaning the concrete type implementing that method: this is needed of course because when a trait object is used, the compiler doesn't know what the concrete type will be, and the concrete type can be different in different usages of the trait object.

In the same way, trait objects cannot have generic parameters: this is needed because generics are turned into concrete types during compilation, and these concrete types become part of the concrete type that will be used in place of the trait object at runtime, and again there can be different concrete types with different generics in place of the trait object.

For example, the `Clone` trait cannot be made into a trait object:

```rust
pub struct Screen {
  // error: the trait `Clone` cannot be made into an object
  pub components: Vec<Box<dyn Clone>>;
}
```

### Traits and generics

We can use generics with traits:

```rust
pub trait Iterator<T> {
  fn next(&mut self) -> Option<T>;
}
```

however this is usually avoided, because if we create many specific implementations of the trait, when the trait is used in the code we won't know what the specific implementation will be, and so we would specify it every time:

```rust
impl Iterator<String> for Counter {
  // ...
}

impl Iterator<i32> for Counter {
  // ...
}

fn get_counter() -> Counter {
  // ...
}

// is this Counter with String, or Counter with i32?
let counter = get_counter();

let next_string = counter.next<String>();
let next_i32 = counter.next<i32>();
```

### Associated types

To avoid the previous problem with using generics with traits, we can use *associated types* to refer to a type that is not know during the definition of the trait: this that will then be replaced with a concrete type by the implementors of the trait, much like generics, but for traits

```rust
pub trait Iterator {
  type Item;

  fn next(&mut self) -> Option<Self::Item>;
}

impl Iterator for Counter {
  type Item = u32;

  fn next(&mut self) -> Option<Self::Item> {
    // ...
  }
}
```

### Method call disambiguation

We can define different traits having methods with the same name, and have the same struct implement them, without conflicts:

```rust
trait Pilot { fn fly(&self); }
trait Wizard { fn fly(&self); }

struct Human;

impl Pilot for Human {
  fn fly(&self) { /* ... */ }
}

impl Wizard for Human {
  fn fly(&self) { /* ... */ }
}

impl Human {
  fn fly(&self) { /* ... */ }
}

let person = Human;
person.fly();
```

here when we call `person.fly()`, the implementation specific to `Human` will be selected, and not one of those specific for the two traits. If we want to call one of the other implementations, instead, we have to refer to it explicitly:

```rust
Pilot::fly(&person);
Wizard::fly(&person);
```

When we do the same thing for associated functions, we must always specify what implementation we're referring to, because the compiler wouldn't be able to use the type of `self` to understand what implementation to select:

```rust
trait Animal { fn baby_name() -> String }

struct Dog;

impl Dog {
  fn baby_name() -> String { /* ... */ }
}

impl Animal for Dog {
  fn baby_name() -> String { /* ... */ }
}

Dog::baby_name();
<Dog as Animal>::baby_name();
```

In general, we can use this syntax also with methods:

```rust
Human::fly(&person);
<Human as Pilot>::fly(&person);
<Human as Wizard>::fly(&person);
```
