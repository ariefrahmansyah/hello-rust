# Use the derive trait

You might have noticed that our custom types are a little difficult to use in practice. This simple `Point` struct can't be compared to other `Point` instances or displayed in the terminal. Because of this difficulty, we might want to use the `derive` attribute to allow new items to automatically be generated for the struct.

## Downside of generic types

Take a look at the following code example:

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p1 = Point { x: 1, y: 2 };
    let p2 = Point { x: 4, y: -3 };

    if p1 == p2 { // can't compare two Point values!
        println!("equal!");
    } else {
        println!("not equal!");
    }

    println!("{}", p1); // can't print using the '{}' format specifier!
    println!("{:?}", p1); //  can't print using the '{:?}' format specifier!
}
```

The preceding code will fail for three reasons. See this output:

```
    error[E0277]: `Point` doesn't implement `std::fmt::Display`
      --> src/main.rs:10:20
       |
    10 |     println!("{}", p1);
       |                    ^^ `Point` cannot be formatted with the default formatter
       |
       = help: the trait `std::fmt::Display` is not implemented for `Point`
       = note: in format strings you may be able to use `{:?}` (or {:#?} for pretty-print) instead
       = note: required by `std::fmt::Display::fmt`
       = note: this error originates in a macro (in Nightly builds, run with -Z macro-backtrace for more info)

    error[E0277]: `Point` doesn't implement `Debug`
      --> src/main.rs:11:22
       |
    11 |     println!("{:?}", p1);
       |                      ^^ `Point` cannot be formatted using `{:?}`
       |
       = help: the trait `Debug` is not implemented for `Point`
       = note: add `#[derive(Debug)]` or manually implement `Debug`
       = note: required by `std::fmt::Debug::fmt`
       = note: this error originates in a macro (in Nightly builds, run with -Z macro-backtrace for more info)

    error[E0369]: binary operation `==` cannot be applied to type `Point`
      --> src/main.rs:13:11
       |
    13 |     if p1 == p2 {
       |        -- ^^ -- Point
       |        |
       |        Point
       |
       = note: an implementation of `std::cmp::PartialEq` might be missing for `Point`

    error: aborting due to 3 previous errors#+end_example
```

This code fails to compile because our `Point` type doesn't implement the following traits:

- The `Debug` trait, which allows a type to be formatted by using the `{:?}` format specifier, is used in a programmer-facing, debugging context.
- The `Display` trait, which allows a type to be formatted by using the `{}` format specifier, is similar to `Debug`. But `Display` is better suited for user-facing output.
- The `PartialEq` trait, which allows implementors to be compared for equality.

## Use derive

Luckily, the `Debug` and `PartialEq` traits can be automatically implemented for us by the Rust compiler by using the `#[derive(Trait)]` attribute, if each of its fields implements the trait:

```rust
#[derive(Debug, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}
```

Our code will still fail to compile because Rust's standard library doesn't provide automatic implementation for the `Display` trait, because it's meant for end users. But if we comment out that line, our code now produces this output:

```
    Point { x: 1, y: 2 }
    not equal!
```

Nevertheless, we can implement the `Display` trait for our type by ourselves:

```rust
use std::fmt;

impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
    write!(f, "({}, {})", self.x, self.y)
    }
}
```

Our code will then compile just fine:

```
    (1, 2)
    Point { x: 1, y: 2 }
    not equal!
```
