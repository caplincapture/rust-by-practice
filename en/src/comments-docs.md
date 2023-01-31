# Comments and Docs
Every program requires comments:


## Comments
- Regular comments which are ignored by the compiler:
  - `// Line comment, which goes to the end of the line`
  - `/* Block comment, which goes to the end of the closing delimiter */`


### Examples
```rust
fn main() {
    // This is an example of a line comment
    // There are two slashes at the beginning of the line
    // And nothing written inside these will be read by the compiler

    // println!("Hello, world!");

    // Run it. See? Now try deleting the two slashes, and run it again.

    /* 
     * This is another type of comment, a block comment. In general,
     * line comments are the recommended comment style. But
     * block comments are extremely useful for temporarily disabling
     * chunks of code. /* Block comments can be /* nested, */ */
     * so it takes only a few keystrokes to comment out everything
     * in this main() function. /*/*/* Try it yourself! */*/*/
     */

    /*
    Note: The previous column of `*` was entirely for style. There's
    no actual need for it.
    */
}
```

### Exercises
1. 🌟🌟
```rust,editable

/* Make it work, only using comments! */
fn main() {
    //todo!();
    //unimplemented!();

    assert_eq!(6, 5 + 1)
}
```


## Doc Comments
- Doc comments which are parsed into HTML and supported `Markdown`
  - `/// Generate library docs for the following item`
  - `//! Generate library docs for the eclosing item`

Before starting, we need to create a new package for practice: `cargo new --lib doc-comments`.


### Line doc comments `///` 
Add docs for function `add_one`
```rust
// in lib.rs

/// Add one to the given value and return the value
///
/// # Examples
///
/// ```
/// let arg = 5;
/// let answer = my_crate::add_one(arg);
///
/// assert_eq!(6, answer);
/// ```
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```

### Cargo doc
We can use `cargo doc --open` to generate html files and open them in the browser.

### Block doc comments `/** ... */`
Add docs for function `add_two`:
```rust
/** Add two to the given value and return a new value

# Examples

let arg = 5;
let answer = my_crate::add_two(arg);

assert_eq!(7, answer);

*/
pub fn add_two(x: i32) -> i32 {
    x + 2
}
```

### Doc comments for crate and module
We can also add doc comments for our crates and modules.

Firstly, let's add some doc comments for our library crate:

> Note: We mush place crates and module comments at the top of the crate root or module file.
```rust
//! # Doc comments
//! 
//! A library for showing how to use doc comments

// in lib.rs
pub mod compute;
```

You can also use block comments to achieve this:
```rust
/*! # Doc comments

 A library for showing how to use doc comments */
```

Next, create a new module file `src/compute.rs`, and add following comments to it:
```rust
//! //! Do some complicated arithmetic that you can't do by yourself

// in compute.rs
```

Then run `cargo doc --open` and see the results.


### Doc tests
The doc comments of `add_one` and `add_tow` contain two example code blocks.

The examples can not only demonstrate how to use your library, but also running as test with `cargo test` command.

2. 🌟🌟 But there are errors in the two examples, please fix them, and running with `cargo test` to get following result: 
```shell
running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests doc-comments

running 2 tests
test src/lib.rs - add_one (line 11) ... ok
test src/lib.rs - add_two (line 26) ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.55s
```

3. 🌟🌟 Sometimes we expect an example to be panic, add following code to `src/compute.rs` and make the `cargo test` passed.

> You can only modify the comments, DON'T modify `fn div`

```rust
// in src/compute.rs

/// # Panics
///
/// The function panics if the second argument is zero.
///
/// ```rust,should_panic
/// // panics on division by zero
/// doc_comments::compute::div(10, 0);
/// ```
pub fn div(a: i32, b: i32) -> i32 {
    if b == 0 {
        panic!("Divide-by-zero error");
    }

    a / b
}
```

4. 🌟🌟 Sometimes we want to hide the doc comments, but keep the doc tests.

Add following code to `src/compute.rs` ,

```rust
// in src/compute.rs

/// ```
/// # fn try_main() -> Result<(), String> {
/// let res = doc_comments::compute::try_div(10, 0)?;
/// # Ok(()) // returning from try_main
/// # }
/// # fn main() { 
/// #    try_main().unwrap();
/// #
/// # }
/// ```
pub fn try_div(a: i32, b: i32) -> Result<i32, String> {
    if b == 0 {
        Err(String::from("Divide-by-zero"))
    } else {
        Ok(a / b)
    }
}
```

and modify this code to achieve two goals:

- The doc comments must not be presented in html files generated by `cargo doc --open`
- run the tests, you should see results as below:

```shell
running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests doc-comments

running 4 tests
test src/compute.rs - compute::div (line 7) ... ok
test src/lib.rs - add_two (line 27) ... ok
test src/lib.rs - add_one (line 11) ... ok
test src/compute.rs - compute::try_div (line 20) ... ok

test result: ok. 4 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.51s
```

### Code navigation
Rust provide a very powerful feature for us, that is code navigation in doc comments.

Add following code to `src/lib.rs`:
```rust
// in lib.rs

/// Add one to the given value and return a [`Option`] type
pub fn add_three(x: i32) -> Option<i32> {
    Some(x + 3)
}
```

Besides jump into the standard library, you can also jump to another module in the package.

```rust
// in lib.rs

mod a {
    /// Add four to the given value and return a [`Option`] type
    /// [`crate::MySpecialFormatter`]
    pub fn add_four(x: i32) -> Option<i32> {
        Some(x + 4)
    }
}

struct MySpecialFormatter;
```

### Doc attributes
Below are a few examples of the most common `#[doc]` attributes used with `rustdoc`.

### `inline`

Used to inline docs, instead of linking out to separate page.

```rust,ignore
#[doc(inline)]
pub use bar::Bar;

/// bar docs
mod bar {
    /// the docs for Bar
    pub struct Bar;
}
```

### `no_inline`

Used to prevent linking out to separate page or anywhere.

```rust,ignore
// Example from libcore/prelude
#[doc(no_inline)]
pub use crate::mem::drop;
```

### `hidden`

Using this tells `rustdoc` not to include this in documentation:

```rust,editable,ignore
// Example from the futures-rs library
#[doc(hidden)]
pub use self::async_await::*;
```

For documentation, `rustdoc` is widely used by the community. It's what is used to generate the [std library docs](https://doc.rust-lang.org/std/).


### Full Code
The full code of package `doc-comments` is [here](https://github.com/sunface/rust-by-practice/tree/master/practices/doc-comments).