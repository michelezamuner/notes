# Collections

The built-in composite types, array and struct, are allocated on the stack. However, Rust provides a set of standard collections that, in order to work with a dynamic amount of data, are allocated on the heap.

## Vector

Vectors have type `Vec<T>`, and store multiple values of the same type to contiguous memory locations on the heap. Vectors are allocated on the heap, and thus can have variable size, but their elements must have a size known at compile time, and thus will be allocated on the stack, in order to be able to calculate how much memory needs to be allocated to store the whole vector.

To create a new vector:

```rust
let v = vec![1, 2, 3];
```

Here we used the `vec!` macro to create a new vector of type `Vec<i32>`, where the type of the elements has been inferred from the initialization values.

To create an empty vector, instead, we need to annotate the type, since we don't have values to infer it from:

```rust
let v: Vec<i32> = Vec::new();
```

Vectors support the `push()` method to add elements to the back:

```rust
let mut v = Vec::new();

v.push(5);
v.push(6);
```

Since we're immediately adding hardcoded values to the vector, the compiler can infer the type of the elements without requiring us to annotate the type.

Vectors allow to read elements using the indexing operator, or the `get` method:

```rust
let v = vec![1, 2, 3, 4, 5];

let third = v[2];

match v.get(2) {
  Some(third) => // ...
  None => // ...
}
```

When we're reading vectors elements we need to face the problem that the index requested might be greater than the maximum number of elements contained in the vector. When using the indexing operator `[]`, if we try to access an out-of-bounds index, the program will panic and terminate. Instead, when using the `get` method we get an `Option` that will be `None` in case we tried to access a non-existing element.

When a vector is allocated, the operating system reserves a fixed amount of memory, known as the capacity, for the vector. When we add new elements to the vector, they are added to this memory, until the size of the vector grows up to fill the whole capacity. At this point there wouldn't be any memory left to contain any more element, so the operating system finds another area in memory that can hold a bigger capacity, and copies the whole vector into the new area.

This explains why the rule that we cannot have both a mutable and an immutable borrow in the same scope holds also for vectors:

```rust
let mut v = vec![1, 2, 3, 4, 5];

let first = &v[0];

// error: cannot borrow `v` as mutable because it is also borrowed as immutable
v.push(6);

println!("{}", first);
```

at first thought, the operation of adding an element to the back of the vector would not change the value of the first element, which is the one we immutably borrowed: however this operation might still require a reallocation in memory, which means that after the vector has been reallocated, the reference `&v[0]` would be pointing to deallocated memory.

## Strings

The only language-native string type in Rust, is the *string slice*. Like all slices, also string slices are specific views of an existing area of memory, in this case containing data that is interpreted as a string. String slices are created from string literals:

```rust
let s: &str = "Hello, world!";
```

These values are just hardcoded into the final executable, meaning that they're eventually pushed to the stack, and thus are fast and efficient, but exactly for this reason they are immutable, and can only represent text that is known at compile-time, meaning they cannot be used to handle user input, or text coming from a data source.

To represent a text whose size is not known at compile time, we can use the `String` type, which is part of the standard library:

```rust
let s = String::new();
```

this would actually create an immutable empty string, which is not very useful. To create a string out of a string slice:

```rust
let string = String::from("hello");
let also_string = "hello".to_string();
```

Here the argument of the factory method could come from any source, without the compiler knowing its size in advance, and thus the resulting text represented by this `String` can have any size.

Strings are intended to be collections of chars, so we can think of pushing specific characters to a mutable string:

```rust
let mut s = "hello".to_string();
s.push('!'); // "hello!"
```

Like with all collections, `push()` changes the original string in-place, appending a character to it, meaning that the size of the value represented by the `s` variable has changed at runtime.

To append a string to another string in place:

```rust
let mut s = String::from("hello");
s.push_str(", world!"); // "hello, world!"
```

To construct a string from a combination of values that can be converted to string:

```rust
let s = format!("{}, {}", "hello", "world"); // "hello, world!"
```

To append a slice to a string:

```rust
let s = "hello".to_string();
let s1 = s + ", world!"; // "hello, world!"
```

here what's really happening is that the `+` operator, which actually is an alias for a function like `fn add(self, s: &str) -> String`, takes ownership of the original `s`, modifies it in-place by appending the slice argument, and then returns ownership of the string to the caller. This means that, although we're still assigning the result to a new variable `s1`, no copy of the original string has ever been made.

Although many functions of `String` work in a similar way than some of `Vec`, like `push`, there are some important differences, because strings are not implemented like collections of chars, like one would think, but they're actually wrappers around `Vec<u8>`, so they're really collections of bytes. The reason for this is that strings need to support Unicode, where characters can be represented with different amount of bytes, which means that a collection of characters is not necessarily the same thing as a collection of bytes.

For this reason, strings don't support the indexing operator `[]`, which is instead supported by the underlying `Vec<u8>` implementation: in fact, if we wrote `s[2]`, trying to get the third byte of the string `s`, and that byte happened to be the first of a multi-byte character, we couldn't return a whole character, like one would expect.

We might then think of forgetting the underlying `Vec<u8>` implementation, and just say that `s[2]` represents the third character, and not the third byte. Well, this wouldn't work as expected either, because the indexing operation by definition allows to access the elements of a collection in constant time, but in order to understand what's the third character we need to walk through all the bytes from the beginning of the string, because we don't know in advance how many bytes each valid character is made of, and this operation wouldn't clearly be performed consistently in constant time.

We can use the indexing operator on strings to cut slices, though:

```rust
let s = &hello[0..4];
```

however, in addition to be careful with passing a range that does not go out of the string bounds, in this case we also need to be sure to be cutting the string exactly at the boundaries of valid characters. This is because the slice operation works on single bytes, and the range we're using in the slice refers to the addresses of the actual bytes, so we might end up keeping into the slice only some bytes of a character, and not all. In this case, the program will panic exactly as if we tried to use an out-of-bounds index. So it's important to be careful when trying to cut string slices in production code.

Strings provide the `chars()` method to return precisely the characters they are composed of:

```rust
for c in "string".chars() {
  println!("{}", c);
}
```

Instead, the `bytes()` method returns the bytes the string is composed of:

```rust
for b in "string".bytes() {
  println!("{}", b);
}
```

the difference being of course that Unicode characters can be composed of more than one byte.

## Hash maps

Hash maps allow to associate values to keys, where both values and keys can be of any type, but all keys must have the same type, and all values must have the same type, thus the type of a hash map would be `HashMap<K,V>`:

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);
```

We can create an hash map from a vector of keys and a vector of values:

```rust
use std::collections::HashMap;

let teams = vec![String::from("Blue"), String::from("Yellow")];
let initial_scores = vec![10, 50];

let scores: HashMap<_,_> = teams.iter().zip(initial_scores.iter()).collect();
```

here we first create a vector of tuples, where each tuple contains one element of the `teams` vector and one element of the `initial_scores` vector; in order to perform complex manipulations on collections, we need to use iterators over those collections, and thus we first create an iterator for both vectors, and then call the `zip` method on the first one, passing the iterator to the second as argument, to merge the two vectors into a vector of tuples; the value returned by `zip` is a data structure representing this merge, rather than a collection, so we then need to call the `collect()` method in order to get a collection back: however, in order to let Rust know which specific collection we want this merge object to be turned into, we add the `HashMap<_,_>` type annotation, thus selecting the `HashMap` collection.

When we create an hash map with any method, we need to pass some objects to it as keys and values: if these objects implement the `Copy` trait, they will be copied by default, if they're owned values instead they will be moved:

```rust
use std::collections::HashMap;

let field_name = String::from("Favorite color");
let field_value = String::from("Blue");

let mut map = HashMap::new();
map.insert(field_name, field_value);
```

Since `String` are owned values, they will be moved into the map, meaning that after the last line we wont' be able to use `field_name` and `field_value` anymore.

Hash maps provide the `get()` method to access the values they store, given their respective keys:

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

let team_name = String::from("Blue");
let score:Option<i32> = scores.get(&team_name);
```

Since we could pass any value of the key type to `get()`, we don't know if that value will correspond to a key that actually exists in the map, so the result of this operation is a variant of the `Option` enum, which can be either `Some` or `None` meaning that we can get either something or nothing as a result.

Hash maps by default are iterated as tuples of one key and one value:

```rust
for (key, value) in &scores {
  println!("{}: {}", key, value);
}
```

If we insert a value into a hash map for a key that already exists, the value associated to that key is replaced:

```rust
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Blue"), 25);
```

here, after the second statement, the `"Blue"` key will be associated to the value `25`, and the original value `10` will be lost.

The `entry()` method returns a mutable references to an instance of the `Entry` enum telling if the key given to `entry()` is associated to a key, and providing some utility methods to change the original map:

```rust
scores.insert(String::from("blue"), 10);
scores.entry(String::from("Yellow")).or_insert(50);
scores.entry(String::from("Blue")).or_insert(50);
```

here we check if a given key exists, and inserting it with a value in case it doesn't. If a value already exists, `or_insert()` returns a mutable reference to it:

```rust
use std::collections::HashMap;

let text = "hello world wonderful world";

let mut map = HashMap::new();

for word in text.split_whitespace() {
  let count = map.entry(word).or_insert(0);
  *count += 1;
}
```

here every time the `word` key is found, a mutable reference to its value is returned, so the first time a reference to `0` is returned, and is then incremented by 1, meaning that the map is now containing the value `1` instead of `0`, related to that key.

## Iterators

Iterators are types implementing the `Iterator` trait, requiring them to implement a function `next(&mut self) -> Option<Self::Item>`.

An iterator is connected to a collection, since it's used to iterate over the items of that collection. The `next` method is used to produce all items of the collection, one by one: it returns an `Option` because when the collection has been fully iterated over, there won't be any more item to produce, and then `None` will be returned. The `next` method also borrows `self` mutably, because it needs to implement a cursor that is updated every time `next` is called, in order to keep track of how much we already iterated over the collection.

Iterators are automatically used in `for` loops:

```rust
for v in vec![1, 2, 3] {
  println!("{}", v);
}
```

here `for` produces an iterator from `vec![1, 2, 3]`, and then repeatedly calls `next()` on it to produce all the values of the vector. In particular, `for` takes ownership of the iterator thus produced, and makes it mutable: this is why we don't need to make mutable everything we pass to `for`.

We can create an iterator from a collection by calling the `iter()` method. In this case the values produced by calling `next()` on the iterator would be immutable references to the collection's items. If we instead want the iterator to take ownership over the values of the collection, we need to create it with `into_iter()`. Also, if we want `next()` to produce mutable references we need to create the iterator with `iter_mut()`.

The `Iterator` trait provides many default methods to perform typical useful operations on collections. Some of these methods actually use the values of the collection to produce some result: these are called *consuming adaptors*, because in order to use the collection's values, they need to take ownership of the iterator, by calling `next()`. For example the `sum()` method goes through all values of a collection to produce their sum:

```rust
let sum = vec![1, 2, 3].iter().sum();
```

Some other iterator methods don't need to look at the collection's values, because they just define operations to be performed on all collection's values, without actually performing them. These are called *iterator adaptors*, they don't call `next()` since they don't need to look at the actual values, so they don't take ownership of the iterator, and they just return other iterators. For example the `map()` method produces another iterator that applies a closure to all the values of the collection:

```rust
let iterator = vec![1, 2, 3].iter();
let map_plus_one = iterator.map(|x| x + 1);
let map_plus_two = iterator.map(|x| x + 2);
```

since `map()` does not move `iterator`, we can call it multiple times on the same iterator, producing different other iterators.

Since iterator adaptors just produce other iterators without even looking at the original values, they're pretty useless if used on their own: the typical pattern is then to chain some iterator adaptors to define the exact operation we want to perform on the original data, and then call a consuming adaptor to actually perform that operation and produce a meaningful result:

```rust
let result = vec![1, 2, 3].iter().map(|x| x + 1).sum();
```

here all methods before `sum()` are lazy, meaning that they don't perform any actual operations on the original values, and only when calling `sum()` the values are iterated over, applying all operations previously defined, and an actual result is produced. This also means that the closure passed to `map()` is only called when `sum()` is called.

An useful consuming adaptor is `collect()`, which produces a collection from an iterator:

```rust
let result: Vec<_> = vec![1, 2, 3].iter().map(|x| x + 1).collect();
```

here, `collect()` doesn't know what kind of collection we want to turn the iterator into, so we need to specify it by annotating the resulting type. Also, we only need to specify the type of the collection, and not the type of the values, which can be inferred by the last iterator, so the annotation `Vec<_>` is not required to annotate also the type of the items.

In some cases, `collect()` might be using the same values as the original collection:

```rust
// error: temporary value dropped while borrowed
let result: Vec<_> = vec![String::from("a"), String::from("ab"), String::from("abc")]
  .iter()
  .filter(|s| s.len() < 3)
  .collect();
println!("{:?}", result);
```

here `iter()` produces an iterator that immutably borrows the original strings, and `collect()` puts the filtered ones into the new collection `result`, which ends up containing references to the exact same values of the original collection; the error here is due to the fact that the original vector is not stored in a variable, and thus its contents will be dropped as soon as `collect()` is called: but then the references stored in `result` would be invalid. A possible solution here would be to store the original collection in a separate variable to outlive the call to `collect()`: however this would require to pass the original vector as a reference:

```rust
let original = vec![String::from("a"), String::from("ab"), String::from("abc")];
let result = produce_result(&original);

fn produce_result<'a>(original: &'a Vec<String>) -> Vec<&'a String> {
  original.iter()
    .filter(|s| s.len() < 3)
    .collect()
}
```

Alternatively, we could just create an iterator containing owned values:

```rust
let original = vec![String::from("a"), String::from("ab"), String::from("abc")];
let result = produce_result(original);

fn produce_result(original: Vec<String>) -> Vec<String> {
  original.into_iter()
    .filter(|s| s.len() < 3)
    .collect()
}
```
