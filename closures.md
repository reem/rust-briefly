# Closures

Closures are anonymous functions with an associated environment. When a closure
is created, certain data is "captured" within its environment and can then be
accessed by the closure when it is called. This is in contrast to normal `fn`s,
which cannot close over any environment and can only access their arguments and
global state or symbols in their body.

## The Fn Family

The implementation of closures in Rust is based on making the function call
operator, `()`, overloadable. Just like with other operator overloading, the
function call operator is tied to a set of traits, `std::ops::{Fn, FnMut,
FnOnce}`. Here is a very slightly simplified version of their definition:

```rust
pub trait FnOnce<Args> {
    type Output;

    fn call_once(self, Args) -> Self::Output;
}

pub trait FnMut<Args> {
    type Output;

    fn call_mut(&mut self, Args) -> Self::Output;
}

pub trait Fn<Args> {
    type Output;

    fn call(&self, Args) -> Self::Output;
}
```

Just like with other operator overloading trait families like `Deref` and
`DerefMut`, the only difference between these traits is the type of the
receiver, `self`. This is just so we can provide every ownership mode for
the function call operator.

If you implement one of these traits, you can be called with the function call
operator, and for all intents and purposes, are a closure.

_Closures in Rust are just instances of types which implement one of the traits
in the Fn Family._

## Manually Implementing Closures

The `Fn` traits are the only thing we need to create a fully functional
closure. All we have to do is define a new type which will hold the captured
environment and implement one of the traits for it:

```rust
struct CountingClosure {
    current: usize
}

// We implement FnMut, since we will need mutable access to our environment.
impl FnMut<()> for CountingClosure { // Args = () means no arguments
    type Output = usize;

    fn call_mut(&mut self, _: ()) -> usize {
        self.current += 1;
        self.current
    }
}

let count = CountingClosure { current: 0 };
count(); count(); count();
assert_eq!(count(), 4);
```

## Closure Literals

A closure literal is just an expressive shorthand for what we just wrote out
ourselves.

The arguments go between a pair of `|`s, followed by a block which contains the
body of the closure. Unlike when declaring `fn`s, annotating the argument and
return types of the closure is optional. Here's the equivalent of the above:

```rust
let mut counter = 0;
let mut count = || {
    counter += 1;
    counter
};
count(); count(); count();
assert_eq!(count(), 4);
```

The closure literal expands to a new type, an implementation of `FnOnce` and
`FnMut` for that type, and an instantiation of the type with the captured
environment. Note that Rust infers the "kind" of the closure, and only emits
implementations of `FnOnce` and `FnMut` in this case.

There is one subtle difference between this example and what we wrote
explicitly above - `count` contains a mutable reference to `counter`, capturing
by-reference, rather then by-value. To truly make the example equivalent, we
must capture by-value, which we can do by using the `move` keyword when we
create the closure. This way, `counter` is moved into `count`:

```rust
let mut counter = 0;
let mut count = move || {
    counter += 1;
    counter
};
count(); count(); count();
assert_eq!(count(), 4);
```

