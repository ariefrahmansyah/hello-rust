# Data Types

Rust is a statically typed language, which means that the compiler must know exactly the data types for each variable in your code.

In most cases, the compiler can infer the type of some value, and you don't need to tell it explicitly in your code. Sometimes when many types are possible, you must inform the compiler what specific data type must be used. In these situations, you use type annotations.

For instance, suppose we're writing a program that needs to convert a string into a number and uses the .parse() method to do so.

```rust
let number: u32 = "42".parse().expect("Not a number!");
```

In this example, we tell the compiler that we want the number variable to be a 32-bit number by annotating that type (u32) right after the variable name.

We can experiment by removing that type annotation to provoke and observe a compiler error:

```rust
let number = "42".parse().expect("Not a number!");
```

The error:

```
    error[E0282]: type annotations needed
     --> src/main.rs:2:9
      |
    2 |     let number = "42".parse().expect("Not a number!");
      |         ^^^^^^ consider giving `number` a type
```

## Rust built-in data types

### Numbers

Integers in Rust can be identified by bit size and the signed property. Signed integers can represent positive and negative numbers. Unsigned integers can represent only positive numbers.

Length | Signed | Unsigned
-------|--------|---------
8-bit | i8 | u8
16-bit | i16 | u16
32-bit | i32 | u32
64-bit | i64 | u64
128-bit | i128 | u128
arch | isize | usize

Additionally, the `isize` and `usize` types depend on the kind of computer your program is running on: 64 bits if you're on a 64-bit architecture and 32 bits if you're on a 32-bit architecture. They're the default type assigned to integers whenever you don't specify one.

Rust's floating-point types are `f32` and `f64`, which are 32 bits and 64 bits in size, respectively.

The default type is `f64` because on modern CPUs it's roughly the same speed as `f32` but is capable of more precision.

```rust
let x = 2.0;      // f64, default type
let y: f32 = 3.0; // f32, via type annotation
```

All Rust's primitive number types support mathematical operations such as addition, subtraction, multiplication, and division.

```rust
fn main() {
    // Addition
    println!("1 + 2 = {}", 1u32 + 2);

    // Subtraction
    println!("1 - 2 = {}", 1i32 - 2);
    // ^ Try changing `1i32` to `1u32` to see why the type is important

    // Integer Division
    println!("9 / 2 = {}", 9u32 / 2);

    // Float Division
    println!("9 / 2 = {}", 9.0 / 2.0);

    // Multiplication
    println!("3 * 6 = {}", 3 * 6)
}
```

> We're using suffixes on the literal numbers to tell Rust which data type they will be (e.g., `1u32` is the number one as an unsigned 32-bit integer). If we don't provide these type annotations Rust tries to infer the type from context defaulting to `i32` (a signed 32-bit integer) when it's ambiguous.

### Booleans

Booleans in Rust are represented by the type `bool` and have two possible values: `true` or `false`. They're used widely in conditionals, such as `if` and `else` expressions. They come up as a result of comparison checks.

```rust
let is_bigger = 1 > 4;
println!("{}", is_bigger);  // prints "false"
```

### Character and strings

Rust has one character type and two string types. All of them are valid UTF-8 representations.

The `char` type is the most primitive type among them and is specified with single quotation marks:

```rust
let c = 'z';
let z = 'ℤ';
let heart_eyed_cat = '😻';
```

> Some languages treat their `char` types as 8-bit unsigned integers (the equivalent of Rust's `u8`). Rust's `char` types are utf-8 encoded unicode code points. This means they are **32 bits** wide.

The `str` type, also known as a *string slice*, is a view into string data. Most of the time, we refer to those types in referenced form by using the form `&str`. We'll cover references in the following modules. For now, you can think of `&str` as a pointer to an immutable string data. String literals are all of type `&str`.

Although string literals are convenient to use in introductory Rust examples, they aren't suitable for every situation in which we might want to use text. That's because not every string can be known at compile time. An example is when a user interacts with a program and sends text via a terminal.

For these situations, Rust has a second string type, `String`. This type is allocated on the heap. It can store an amount of text that's unknown to us at compile time.

> If you're coming from a garbage collected language, you might be wondering why Rust has two string types (spoiler alert: it actually has more!). As it turns out, strings are extremely complex data types. Most languages use their garbage collectors to gloss over this complexity, but Rust, being a system's language, exposes some of the inherent complexity of strings to you. With this added complexity comes a very fine grained amount of control over how memory is used in your program.

We won't get a full idea of the difference between `String` and `&str` until we learn about Rust's ownership and borrowing system. Until then, you can think of:
1. `String` data as string data that can change as your program runs,
2. while `&str` are immutable views into string data that do not changes as your program runs.

You can create a `String` from a string literal by using the `from` function, like so:

```rust
let mut hello = String::from("Hello, ");  // create a String from a string literal
hello.push('w');                          // push a character into our String
hello.push_str("orld!");                  // push a string literal into our String
println!("{}", hello)
```

### Tuples

A tuple is a grouping of values of different types collected into one compound. They have fixed length, meaning that after they're declared, they can't grow or shrink in size. The type of a tuple is defined by the sequence of each member's type.

Here's a tuple of length 3:

```rust
("hello", 5i32, 'c');
```

This tuple has the type signature `(&'static str, i32, char)`, where:

- `&'static str` is the type of the first element.
- `i32` is the type of the second element.
- `char` is the type of the third element.

Tuples elements can be accessed by position, which is known as *tuple indexing*. It looks like this:

```rust
fn main() {
  let tuple = ("hello", 5, 'c');

  assert_eq!(tuple.0, "hello");
  assert_eq!(tuple.1, 5);
  assert_eq!(tuple.2, 'c');
}
```

The `assert_eq!` macro verifies that two expressions are equal to each other.

Tuples are useful when you want to combine different types into a single value. For instance, functions can use tuples to return multiple values because tuples can hold any number of values.
