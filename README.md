# Learning Rust

Exercises based on https://doc.rust-lang.org/book/ 

## How to run the examples

Evaluate code snippets inline using [neovim](https://neovim.io/) with the [mdeval plugin](https://github.com/gpanders/vim-medieval).

Using FeMaco creates an editing floating window with `rust-tools` LSP attached and Treesitter attached.

## Topics

Learning topics with random notes and observations.

### Traits

Traits are very similar to `C# interfaces` but much more versatile and powerful. For example, it's possible to use `dependency injection` techniques from OOP.

```rust
#[derive(Debug)]
struct StdoutLogger {
    log: String,
    stamp: String,
}
fn main() {
    let structer = StdoutLogger {
        log: String::from("aaa"),
        stamp: String::from("2022-10-28"),
    };
    println!("Structre: {:?}", structer)
}
```

*Results:* `Structre: StdoutLogger { log: "aaa", stamp: "2022-10-28" }`

### ? Operator

? Will return the error value from the current function for the caller to handle.

### Struct Constructors

The `new` constructor should never fail. Use other methods if you expect them to fail.

```rust
struct Config {
    query: String,
    file_path: String,
}

impl Config {
    fn new(args: &[String]) -> Config {
        Config {
            query: args[0].clone(),
            file_path: args[1].clone(),
        }
    }
}

fn main() {
    let args = vec![String::from("arg1"), String::from("arg2")];
    let config = Config::new(&args);

    print!("query: {}, file_path: {}", config.query, config.file_path);
}
```

*Results:* `query: arg1, file_path: arg2`

### Reading input

Reading command line input arguments

```bash
cat <<EOF - | cargo run arg1 arg2
use std::env; 

fn main() {
    let args:Vec<String> = env::args().collect();
    dbg!(args);
}
```

*Results:*
```
/tmp/mdeval//READMEmd_52_60.sh: line 7: warning: here-document at line 1 delimited by end-of-file (wanted `EOF')
   Compiling rust-exercises v0.1.0 (/home/decoder/dev/learning-repos/rust-exercises)
    Finished dev [unoptimized + debuginfo] target(s) in 0.37s
     Running `target/debug/rust-exercises arg1 arg2`
Hello World!
```

### Higher Order Functions

This is pretty similar to `F#` syntax:

```rust
fn is_odd(n: u32) -> bool {
    n % 2 == 1
}

fn main() {
    let upper = 1000;
    let sum_of_squared_odd_numbers: u32 =
        (0..10).map(|n| n * n)                           // Numbers from 0 to 10
             .take_while(|&n_squared| n_squared < upper) // Below upper limit
             .filter(|&n_squared| is_odd(n_squared))     // That are odd
             .sum();                                     // Sum them
    println!("functional style: {}", sum_of_squared_odd_numbers);
}

```

*Results:* `functional style: 165`


### Statements & Expressions

> If you add a semicolon to the end of an expression, you turn it into a statement, and it will then not [return a value](https://doc.rust-lang.org/book/ch03-03-how-functions-work.html#:~:text=if%20you%20add%20a%20semicolon%20to%20the%20end%20of%20an%20expression%2C%20you%20turn%20it%20into%20a%20statement%2C%20and%20it%20will%20then%20not%20return%20a%20value)

```rust
fn main() {
    let x = return_five();

    println!("The value of x is: {x}");
}

// The below function would evaluate to 5
fn return_five() -> i32 {
    5
}
```

*Results:* `The value of x is: 5`

```rust
fn main() {
    let x = return_five();

    println!("The value of x is: {x}");
}

// This errors out as the semicolon creates a statement and there is no return value
fn return_five() -> i32 {
    5;
}
```

*Results:*
```
error[E0308]: mismatched types
 --> temp.rs:8:21
  |
8 | fn return_five() -> i32 {
  |    -----------      ^^^ expected `i32`, found `()`
  |    |
  |    implicitly returns `()` as its body has no tail or `return` expression
9 |     5;
  |      - help: remove this semicolon

error: aborting due to previous error

For more information about this error, try `rustc --explain E0308`.
```

### Closures

```rust
#[derive(Debug, PartialEq, Copy, Clone)]
enum ShirtColor {
    Red,
    Blue,
}

struct Inventory {
    shirts: Vec<ShirtColor>,
}

impl Inventory {
    fn giveaway(&self, user_preference: Option<ShirtColor>) -> ShirtColor {
        user_preference.unwrap_or_else(|| self.most_stocked())
    }

    fn most_stocked(&self) -> ShirtColor {
        let mut num_red = 0;
        let mut num_blue = 0;

        for color in &self.shirts {
            match color {
                ShirtColor::Red => num_red += 1,
                ShirtColor::Blue => num_blue += 1,
            }
        }
        if num_red > num_blue {
            ShirtColor::Red
        } else {
            ShirtColor::Blue
        }
    }
}

fn main() {
    let store = Inventory {
        shirts: vec![ShirtColor::Blue, ShirtColor::Red, ShirtColor::Blue, ShirtColor::Red, ShirtColor::Red, ShirtColor::Red],
    };

    let user_pref1 = Some(ShirtColor::Red);
    let giveaway1 = store.giveaway(user_pref1);
    println!(
        "The user with preference {:?} gets {:?}",
        user_pref1, giveaway1
    );

    let user_pref2 = None;
    let giveaway2 = store.giveaway(user_pref2);
    println!(
        "The user with preference {:?} gets {:?}",
        user_pref2, giveaway2
    );
}
```

*Results:*
```
The user with preference Some(Red) gets Red
The user with preference None gets Red
```

## Links & Resources

- [Rust in examples](https://doc.rust-lang.org/rust-by-example/index.html)

## Reference

<a id="1" href="https://doc.rust-lang.org/book/ch03-03-how-functions-work.html#:~:text=if%20you%20add%20a%20semicolon%20to%20the%20end%20of%20an%20expression%2C%20you%20turn%20it%20into%20a%20statement%2C%20and%20it%20will%20then%20not%20return%20a%20value">[1]</a>
