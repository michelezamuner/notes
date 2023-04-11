# Lifetimes

Lifetimes refer to the scope of validity of a reference, meaning the portion of execution during which a reference points to a valid area of memory. Rust is designed to control lifetimes in order of making sure at compile time that there can never be a dangling reference.

```rust
{
  let r;

  {
    let x = 5;
    r = &x;
  }

  // error
  println!("r: {}", r);
}
```

in this example, when execution exits the inner scope, the variable `x` is freed, and thus the reference `r` points to invalid memory: for this reason we cannot use it in the `println!()` macro, and we say that the variable `x` doesn't live long enough to allow `r`, which references it, to be used, since it lives longer. The *borrow checker* of the Rust compiler compares the lifetimes of `r`, the reference, and `x`, the subject of the reference, and notices that the subject of the reference has a shorter life than its reference, and thus rejects the program.

In some circumstances, however, it's impossible to infer the lifetime of a reference:

```rust
fn longest(x: &str, y: &str) -> &str {
  if x.len() > y.len() {
    x
  } else {
    y
  }
}
```

here the values to which `x` and `y` are pointing will generally have different lifetimes: however, since any of them can be returned by this function, it's not possible to determine at compile time what's the lifetime of the reference returned by the function. In order to fix this situation, we can tell the compiler that both `x` and `y` have the same lifetime:

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
  // ...
}
```

here we're using the lifetime annotation `'a` applied to references to specify that `x`, `y` and the function return value have the same lifetime, whatever it is. If we try to pass values to this function that don't respect these lifetime constraint, the borrow checker will reject the program:

```rust
fn main() {
  let string1 = String::from("long string is long");
  {
    let string2 = String::from("xyz");
    let result = longest(string1.as_str(), string2.as_str());
    println!("The longest string is {}", result);
  }
}
```

in this case `x` is valid until the end of `main`, `y` is valid until the end of the inner block, and the return value, stored in `result`, is also valid until the end of the inner block; thus, it exists a lifetime that makes the definition of `longest` work, which is the inner lifetime: in fact `x` has at least that lifetime (it has a bigger one in fact) and `y` and the return value have exactly that lifetime, thus the return value references something that is surely valid for its whole lifetime.

```rust
fn main() {
  let string1 = String::from("long string is long");
  let result;
  {
    let string2 = String::from("xyz");
    result = longest(string1.as_str(), string2.as_str());
  }
  // error
  println!("The longest string is {}", result);
}
```

here, if we take as a lifetime the intersection of the lifetimes of the given argument, that would be the lifetime of the inner block: however, the return value cannot have that lifetime, because it is used also after the inner block has exited.

We need to add lifetime annotations also when only one parameter is connected to the return value:

```rust
fn longest<'a>(x: &'a str, y: &str) -> &'a str {
  x
}
```

even in this very simple case, where it's obvious that the return value is exactly the first parameter, the compiler doesn't try to understand the body of the function, since in most cases it's too complicated, and just requires us to explicitly state the lifetime of the result value to the lifetimes of the input parameters.

Additionally, notice how we don't need to specify a lifetime for all parameters, as long as the lifetime of the return value is described in terms of the lifetime of some part of the input. If the lifetime of the return value wasn't the lifetime of one of the input parameter, the only alternative is that it would be the lifetime of a value created within the body of the function, and then that we would be returning a reference to a value that is bound to be freed as soon as the function call terminates.

When defining a struct containing references, we need to specify the lifetime of all references contained in the struct:

```rust
struct ImportantExcerpt<'a> {
  part: &'a str,
}
```

this means that an instance of `ImportantExcerpt` cannot live longer than the references it's containing, because the lifetime `'a` annotated on the `struct` means "the lifetime of the struct instance".

Additionally, lifetimes must be specified before types:

```rust
struct C<'a, F> {
  // ...
}
```

and not `struct C<F, 'a> {}`.

When using references in methods, we need to add lifetimes also after `impl`, and methods can refer both to lifetimes of the struct fields, or to lifetimes defined in the context of the method:

```rust
impl<'a> ImportantExcerpt<'a> {
  // ...
}
```

The static lifetime refers to those objects that live for the entire duration of the program. For example, string literals get assigned the static lifetime by default, since they are embedded in the binary, and thus they are available for the whole duration of the program:

```rust
let s: &'static str = "I have a static lifetime.";
```

## Lifetime elision

There are cases where the lifetime of references can be inferred by the compiler, where we can thus perform *lifetime elision*, meaning removing lifetime annotations. The compiler tries to infer lifetimes by trying to apply three *lifetime elision rules* to the code, and checking if at the end it was able to determine the lifetime of the output.

The first lifetime elision rule is that every input parameter of a function, or struct field, that is a reference, has a different lifetime. The second rule is that if there is only one input lifetime parameter, the same lifetime is applied to all output lifetimes. The third rule is that if one input lifetime is that of `&self` or `&mut self`, then the lifetime of `self` is assigned to all output lifetimes.

```rust
fn first_word(s: &str) -> &str
```

here the compiler applies the first rule, thus assigning a different lifetime to all input parameters:

```rust
fn first_word<'a>(s: &'a str) -> &str
```

then the compiler checks if the second rule can be applied here: in fact it does, since there is only one input parameter, so the compiler can infer that the lifetime of the output is the same as that of the input:

```rust
fn first_word<'a>(s: &'a str) -> &'a str
```

now the compiler was already able to infer the lifetime of the output, so we can stop applying elision rules, and proceed with the compilation as if the programmer had added all these lifetimes annotations by hand.

```rust
fn longest(x: &str, y: &str) -> &str
```

by applying the first rule here, we get:

```rust
fn longest<'a, 'b>(x: &'a str, y: &'b str) -> &str
```

here the second rule does not apply, because there are more than one input lifetime; the third rule does not apply as well, because none of the input parameters is `&self` or `&mut self`. However, by applying the elision rules the compiler isn't able to determine the lifetime of the output, it throws an error, requiring the programmer to specify lifetimes explicitly.

```rust
impl<'a> ImportantExcerpt<'a> {
  fn announce_and_return_part(&self, announcement: &str) -> &str {
    // ...
  }
}
```

here, by applying the first rule we get:

```rust
impl<'a> ImportantExcerpt<'a> {
  fn announce_and_return_part<'b, 'c>(&'b self, announcement: &'c str) -> &str {
    // ...
  }
}
```

then the compiler applies the third rule, stating that since one of the input references is `self`, then the output lifetime is the same of `self`:

```rust
impl<'a> ImportantExcerpt<'a> {
  fn announce_and_return_part<'b, 'c>(&'b self, announcement: &'c str) -> &'b str {
    // ...
  }
}
```

and we got to the lifetime of the output, so the programmer is not required to annotate the lifetimes.
