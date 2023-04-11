# Threads

To minimize the size of the runtime, the standard implementation of Rust threads provides a 1:1 threading model, where every thread in Rust is mapped to one operating system thread.

To create a new thread, we use the `std::thread::spawn()` function:

```rust
std::thread::spawn(|| {
  // ...
});
```

When the main thread ends, also any child threads are stopped as well, regardless of the status of their execution. With `thread::sleep()` we force a thread to stop its execution for a defined duration:

```rust
std::thread::spawn(|| {
  // ...
  std::thread::sleep(std::time::Duration::from_millis(1));
  // ...
});
```

since in this case `thread::sleep()` is called inside the closure representing the code of the new thread, the thread which is stopped is the nre thread, and not the main thread.

The function `thread::spawn()` returns a `JoinHandle`, which can be used to make a child thread join the main thread:

```rust
let handle = thread::spawn(|| { /* ... */ });
handle.join().unwrap();
```

here when we call `handle.join()` the main thread pauses its execution and waits for the thread to which `handle` belongs to finish its execution: thus, `handle.join()` is actually a blocking call.

Rust does not allow a child thread to borrow data from the main thread:

```rust
let v = vec![1, 2, 3];

// error: closure may outlive the current function, but it borrows `v`, which is owned by the current function
let handle = thread::spawn(|| println!("{:?}", v));

drop(v);

handle.join().unwrap();
```

here we're using `v` just inside `println!()`, which requires a reference, and thus the closure just borrows `v` from the calling context; however, since Rust doesn't know how long the thread will run, it cannot verify that `v` will still be available when the closure will be called: in particular, we called `drop(v)` just after having requested a new thread to be spawned, and since this procedure takes some time, there's a chance that `drop(v)` will be called before the thread is actually created, meaning that when the closure is finally executed, `v` would already be deallocated.

To fix this we have to force the closure to take ownership of the captured variables:

```rust
let handle = thread::spawn(move || println!("{:?}", v));
```

of course now we're not able to call `drop(v)` anymore, because `v` has been moved before this call.

`Rc` smart pointers cannot be shared between multiple threads:

```rust
let v = vec![1, 2, 3];
let r = Rc::new(v);
thread::spawn(|| {
  // error: `Rc<Vec<i32>>` cannot be shared between threads safely
  let r1 = Rc::clone(&r);
});
```

the problem here is that `Rc` has to handle the reference count, meaning that it needs to mutate itself when new clones of the same pointer target are created; this means that we are trying to share a mutable state between different threads, and this raises the risk of data races, because different threads might be trying to write to this state (clone the `Rc`) at the same time, with the consequence that the reference count might become wrong, and thus that the target value might end up never being deallocated.

To avoid this problem we need to use the `Arc<T>` smart pointer, which is an atomically reference counted pointer, meaning that the reference increment or decrement are done atomically, ensuring that once we start the procedure of modification of the counter, all other threads trying to modify the counter at the same time are forced to wait:

```rust
use std::sync::Arc;
use std::thread;

let v = vec![1, 2, 3];
let r = Arc::new(v);
thread::spawn(move || {
  let r1 = Arc::clone(&r);
});
```

here by using `Arc` instead of `Rc` we can have a thread-safe reference counted pointer; notice also how we needed to define a `move` closure, because to clone an `Arc` we just need a reference to the previous pointer, making the capture just a borrow, which we can't be sure will leave long enough to be used inside another thread.

Like `Rc`, not all types are safe to be moved or cloned into different threads. If a type is implemented so that it's safe to move it or clone it to a different thread, then it should implement the trait `Send`, which signals that the type is safe to be "sent" to different threads. Any type composed of only `Send` types, is automatically marked as `Send` as well.

Likewise, not all types are safe to be borrowed by different threads, meaning that it's not always safe to share a reference among different threads. If a type is implemented so that it's safe to share references of it with multiple threads, then it should implement the trait `Sync`, which signals that the type is safe to be "synchronized" with different threads. This means that a type `T` is `Sync` if and only if `&T` is `Send`. Again, any type composed only of `Sync` types, is automatically marked as `Sync` as well.

Since almost all basic Rust types are already `Send` and `Sync`, it's very unlikely that any type we might build will not end up being `Send` and `Sync` as well. In the rare cases we have a custom type that is not automatically `Send` and `Sync`, and we need to use it in multiple threads, we'd have to manually mark it as `Send` and `Sync`, but this is considered unsafe code, meaning that Rust cannot guarantee that operations on this type will be safe, and the responsibility of implementing safe code is left on the programmer.

When we define a custom trait, and we need to use it on types that will need to be shared between threads, we need to mark the trait as `Send` and `Sync`, in order to ensure that the type implementing this trait will be `Send` and `Sync` as well:

```rust
pub trait MyTrait: Send + Sync {
  // ...
}
```
