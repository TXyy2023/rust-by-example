# `abort` and `unwind`

The previous section introduced `panic!`. When a panic occurs, Rust can either
unwind the stack or abort the process:

- `unwind` walks back up the stack, running destructors for local values along
  the way.
- `abort` terminates the process without running those destructors.

To observe the difference, the following example gives a value a [`Drop`]
implementation that prints a message when it is cleaned up:

```rust,editable,should_panic
struct Lemonade;

impl Drop for Lemonade {
    fn drop(&mut self) {
        println!("Dropping the lemonade.");
    }
}

fn main() {
    let _lemonade = Lemonade;
    panic!("AAAaaaaa!!!!");
}
```

Save this example as `lemonade.rs`, then compile and run it with each strategy:

```console
rustc lemonade.rs -C panic=unwind -o lemonade
./lemonade

rustc lemonade.rs -C panic=abort -o lemonade
./lemonade
```

Both runs report the panic and terminate, but only the `unwind` version prints
`Dropping the lemonade.`. The `abort` version skips that cleanup. With `unwind`,
a panic can also be caught using [`std::panic::catch_unwind`]; with `abort`, it
cannot be caught.

Different code paths can also be conditionally compiled based on the panic
setting, as the following examples show. These examples only inspect the
setting; they do not trigger a panic themselves.

Building on the prior lemonade example, we explicitly use the panic strategy to exercise different lines of code.

```rust,editable,mdbook-runnable
fn drink(beverage: &str) {
    // You shouldn't drink too much sugary beverages.
    if beverage == "lemonade" {
        if cfg!(panic = "abort") {
            println!("This is not your party. Run!!!!");
        } else {
            println!("Spit it out!!!!");
        }
    } else {
        println!("Some refreshing {} is all I need.", beverage);
    }
}

fn main() {
    drink("water");
    drink("lemonade");
}
```

Here is another example focusing on rewriting `drink()` and explicitly use the `unwind` keyword.

```rust,editable
#[cfg(panic = "unwind")]
fn ah() {
    println!("Spit it out!!!!");
}

#[cfg(not(panic = "unwind"))]
fn ah() {
    println!("This is not your party. Run!!!!");
}

fn drink(beverage: &str) {
    if beverage == "lemonade" {
        ah();
    } else {
        println!("Some refreshing {} is all I need.", beverage);
    }
}

fn main() {
    drink("water");
    drink("lemonade");
}
```

[`Drop`]: ../trait/drop.md
[`std::panic::catch_unwind`]: https://doc.rust-lang.org/std/panic/fn.catch_unwind.html
