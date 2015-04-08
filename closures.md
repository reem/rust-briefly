# Closures

Closures are anonymous functions with an associated scoped environment. When a closure
is created, certain data is "captured" within its externalscope and can then be
accessed by the closure when it is called. This is in contrast to normal `fn`s,
which cannot close over any scope and can only access their arguments and
global state or symbols in their own scope/body.

## The Fn Family

The implementation of closures in Rust is based on making the function call
operator, `()`, overloadable. As with any other operator overloading, the
function call operator is tied to a set of traits, `std::ops::{Fn, FnMut,
FnOnce}`. Here is a simplified version of their definition:

```rust
pub trait FnOnce<Args> {
    type Output;

    fn call_once(self, Args) -> self::Output;
}

pub trait FnMut<Args> {
    type Output;

    fn call_mut(&mut self, Args) -> self::Output;
}

pub trait Fn<Args> {
    type Output;

    fn call(&self, Args) -> self::Output;
}
```

As with any operator overloading trait families, like `Deref` and
`DerefMut`, the only difference between these traits is the type of the
receiver, `self`. This is so that we can provide every ownership mode for
the function call operator. <This needs further explanation...>

If you implement one of these traits, you can be called with the function call
operator, and for all intents and purposes, are a closure. <i.e. implementing
these traits is synonymous to overloading the `()` operator.>

_Closures in Rust are just instances of types which implement one of the traits
in the Fn Family._

## Manually Implementing Closures

<Specify: We're going to implement a toy closure!>

The `Fn` traits are the only thing we need to create a fully functional <punny>
closure. All we have to do is define a new type which will hold the captured
environment and implement one of the traits for it <it?>:

```rust
struct CountingClosure {
    current: usize
}

// A quirk of the traits is that we must specify at least one argument. This is
// not a limitation for the real syntax sugar in rust, which hides this from
// the user.
//
// We implement FnMut, since we will need mutable access to our environment.
impl FnMut<()> for CountingClosure {
    type Output = usize;

    fn call_mut(&mut self) -> usize {
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
body of the closure <Show, don't tell.>. Unlike when declaring `fn`s, annotating the argument and
return types of the closure is optional <Reverse the two statements in this sentence>.
Here's the equivalent of <s>the above</s> our toy closure:

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
environment. Note that Rust infers the "kind" of the closure <i.e. the traits that must
be implemented>, and only emits implementations of `FnOnce` and `FnMut` in this case.

<This is a new point, and may deserve its own heading, as it is more advanced.>

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

