# Ownership

In Rust, memory is managed through an ownership system, which is a set of rules checked at compile time. None of the ownership features slow down your program while it's running.

To understand ownership, let's first take a look at Rust's scoping rules and move semantics.

## Scoping rules

In Rust, like most other programming languages, variables are valid only within a certain scope. In Rust scopes are usually denoted by curly brackets `{}`. Common scopes include function bodies and `if`, `else`, and `match` branches.

> In Rust, "variables" are often called "bindings". This is because "variables" in Rust aren't very variable - they don't change that often since they're immutable by default. Instead, we often think about names being "bound" to data, hence the name "binding". We'll use both "variable" and "binding" interchangeably though.

Let's say we have a `mascot` variable that's a string, defined within a scope:

```rust
// `mascot` is not valid and cannot be used here, because it's not yet declared.
{
    let mascot = String::from("ferris");   // `mascot` is valid from this point forward.
    // do stuff with `mascot`.
}
// this scope is now over, so `mascot` is no longer valid and cannot be used.
```

If we try to use `mascot` beyond its scope, we'll get an error like this:

```rust
{
    let mascot = String::from("ferris");
}
println!("{}", mascot);
```

```
    error[E0425]: cannot find value `mascot` in this scope
     --> src/main.rs:5:20
      |
    5 |     println!("{}", mascot);
      |                    ^^^^^^ not found in this scope
```

The variable is valid from the point at which it's declared until the end of that scope.

## Ownership and dropping

Rust adds a twist to the idea of scopes: whenever an object goes out of scope, it is "dropped". Dropping a variable means releasing any resources tied to it. For variables of files, this means closing the file, and for variables that have allocated memory associated with them, this means freeing that memory.

In Rust, bindings that have things "associated" with them that they will free when the binding is dropped are said to "own" those things.

In our example above, the `mascot` variable owns the String data associated with it. The `String` itself owns the heap allocated memory which holds the characters of that string. At the end of the scope, `mascot` is "dropped", the `String` it owns is dropped, and finally the memory that `String` owns is freed.

```rust
{
    let mascot = String::from("ferris");
    // mascot dropped here. The string data memory will be freed here.
}
```

## Move semantics

Sometimes though we don't want the things associated with a variable to be dropped at the end of scope. Instead, we want to transfer ownership of an item from one binding to another.

The simplest example of this is when declaring a new binding:

```rust
{
    let mascot = String::from("ferris");
    // transfer ownership of mascot to the variable ferris.
    let ferris = mascot;
    // ferris dropped here. The string data memory will be freed here.
}
```

A key thing to understand is that once ownership is transferred, the old variable is no longer valid. In our example above, after we transfer ownership of the `String` from `mascot` to `ferris`, we can no longer use the `mascot` variable.

In Rust, "transferring ownership" is known as "moving". In other words, in the example above, the `String` value has been moved from `mascot` to `ferris`.

If we try to use `mascot` after the `String` has been moved from `mascot` to `ferris`, the compiler will not compile our code:

```rust
{
    let mascot = String::from("ferris");
    let ferris = mascot;
    println!("{}", mascot) // We'll try to use mascot after we've moved ownership of the string data from mascot to ferris.
}
```

```
error[E0382]: borrow of moved value: `mascot`
 --> src/main.rs:4:20
  |
2 |     let mascot = String::from("ferris");
  |         ------ move occurs because `mascot` has type `String`, which does not implement the `Copy` trait
3 |     let ferris = mascot;
  |                  ------ value moved here
4 |     println!("{}", mascot);
  |                    ^^^^^^ value borrowed here after move
```

This is known as a "use after move" compile error.

It bares repeating: in Rust, only one thing can ever own a piece of data at a time.

## Ownership in functions

Let's take a look at an example of a string being passed to a function as a argument. Passing something as argument to function *moves* that thing into the function.

```rust
fn process(input: String) {}

fn caller() {
    let s = String::from("Hello, world!");
    process(s); // Ownership of the string in `s` moved into `process`
    process(s); // Error! ownership already moved.
}
```

The compiler complains that the value `s` was *moved*.

```
    error[E0382]: use of moved value: `s`
     --> src/main.rs:6:13
      |
    4 |     let s = String::from("Hello, world!");
      |         - move occurs because `s` has type `String`, which does not implement the `Copy` trait
    5 |     process(s); // Transfers ownership of `s` to `process`
      |             - value moved here
    6 |     process(s); // Error! ownership already transferred.
      |             ^ value used here after move
```

As you can see in the preceding snippet, the first call to `process` transfers ownership of the variable `s`. The compiler tracks ownership, so the second call to `process` results in an error. After resources are moved, the previous owner can no longer be used.

This pattern has a profound impact on the way Rust code is written. It's central to the promise of memory safety that Rust proposes.

In other programming languages, the `String` value of the `s` variable can be implicitly copied before being passed to our function. But in Rust, this does not happen.

In Rust, ownership transfer (that is, moving) is the default behavior.

## Copying instead of moving

You might have noticed that in the (rather informative) compiler error messages above, the `Copy` trait was mentioned. We haven't talked about traits yet, but values that implement the `Copy` trait, don't get moved but are rather copied.

Let's take a look at a value that implements the `Copy` trait: `u32`. The following code which mirrors are broken code from above, compiles without issue.

```rust
fn process(input: u32) {}

fn caller() {
    let n = 1u32;
    process(n); // Ownership of the number in `n` copied into `process`
    process(n); // `n` can be used again because it wasn't moved, it was copied.
}
```

Simple types like numbers *copy* types - they implement the `Copy` trait, meaning they are not moved but copied. This is the case for most simple types. Copying numbers is very inexpensive, and so it makes sense for them to be copied. Copying strings or vectors or other complex types can be quite expensive and so they do not implement the `Copy` trait and are moved.

## Copying types that don't implement Copy

One way to work around the errors we saw above is by *explicitly* copying types before they are moved: known as cloning in Rust. A call to `.clone` will duplicate the memory and produce a new value. The new value is moved meaning the old value can still be used.

```rust
fn process(s: String) {}

fn main() {
    let s = String::from("Hello, world!");
    process(s.clone()); // Passing another value, cloned from `s`.
    process(s); // s was never moved and so it can still be used.
}
```

This approach can be useful but can make your code slower as every call to `clone` is making a full copy of the data. This often means performing memory allocations or some other expensive operation. We can avoid this if we "borrow" values using *references*, the topic of our next unit.
