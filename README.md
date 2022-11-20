# Learning Rust

Exercises based on https://doc.rust-lang.org/book/ 

---

## How to run the examples

Evaluate code snippets inline using [neovim](https://neovim.io/) with the [mdeval plugin](https://github.com/gpanders/vim-medieval).

Using FeMaco creates an editing floating window with `rust-tools` LSP attached and Treesitter attached.

---

## Topics

Learning topics with random notes and observations.

---

### Algebraic data types

`Rust` has a powerful algebraic type system, it is easy to construct reach domains with it.

```rust
#![allow(unused)]
type CheckNumber = u32;
type CardNumber = String;

enum CardType {
    Visa,
    Mastercard,
}

#[derive(Debug)]
struct CreditInfo(CheckNumber, CardNumber);

#[derive(Debug)]
enum PaymentMethod {
    Cash,
    Check(CheckNumber),
    Card(CreditInfo),
}

type PaymentAmount = f32;
#[derive(Debug)]
enum Currency {
    Eur,
    Usd,
}

#[derive(Debug)]
struct Payment {
    amount: PaymentAmount,
    currency: Currency,
    method: PaymentMethod,
}

trait PrintDetails {
    fn payment_info(&self) -> String;
}

impl PrintDetails for Payment {
    fn payment_info(&self) -> String {
        let method = match &self.method {
            PaymentMethod::Cash => String::from("cash"),
            PaymentMethod::Check(c) => format!("a check with number {:?}", c),
            PaymentMethod::Card(c) => format!("a check {} with a credit card {}", c.0, c.1),
        };

        format!(
            "An amount of {}, was paid in {:?}, using {}",
            &self.amount, &self.currency, method
        )
    }
}

fn main() {
    let cc_payment = Payment {
        amount: 100.33,
        currency: Currency::Usd,
        method: PaymentMethod::Card(CreditInfo(122, String::from("452389798712097"))),
    };

    let check_payment = Payment {
        amount: 0.33,
        currency: Currency::Eur,
        method: PaymentMethod::Check(42),
    };

    println!("Payment details \n {}", check_payment.payment_info());
    println!("Payment details \n {}", cc_payment.payment_info());
}
```

*Results:*
```
Payment details 
 An amount of 0.33, was paid in Eur, using a check with number 42
Payment details 
 An amount of 100.33, was paid in Usd, using a check 122 with a credit card 452389798712097
```


---

### Ownership

Ownership rules: 

- Each value in Rust has an owner. 
- There can only be one owner at a time.
- When the owner goes out of scope, the value will be dropped.

Before I could understand the concept of borrowing, I had to understand the idea of `moving`.

> Moving is transferring the ownership of a `value` from one variable or function to another.

#### Out of Scope

```rust
fn main() {
    // Ownership

    let v = "this is valid for whole function";

    {
        let s = "this is valid only in this code block";
    }

    println!("Variable s: {}, variable v: {}", s, v); // This will not compile, variable s is out of scope
}
```

*Results:*
```
error[E0425]: cannot find value `s` in this scope
  --> temp.rs:10:48
   |
10 |     println!("Variable s: {}, variable v: {}", s, v); // This will not compile, variable s is out of scope
   |                                                ^ help: a local variable with a similar name exists: `v`

error: aborting due to previous error

For more information about this error, try `rustc --explain E0425`.
```

---

### Reference

Reference is like a pointer, but always points to the right value for the lifetime of the variable.
There can only be one mutable reference.

```rust
fn main() {
    let s1 = String::from("This is a sample string");
    let len = calculate_length(&s1);

    println!(r#"The length of {s1} is {len}"#);

}

fn calculate_length(s1: &str) -> usize {
    s1.len()
}
```

*Results:* `The length of This is a sample string is 23`

---

#### In Scope

```rust
fn main() {
    let v = "this is valid for whole function";

    {
        let s = "this is valid only in this code block";
        println!("{s}"); 
    }

    println!("{v}"); 
}
```

*Results:*
```
this is valid only in this code block
this is valid for whole function
```

---

### Raw strings

[Raw string](https://doc.rust-lang.org/reference/tokens.html#raw-string-literals) literals

> do not process any escapes

This is useful in regex expressions.

```rust
    let regex = Regex::new("opacity: (.*)").unwrap();
```

---

### Common Collections

Vectors - allow to store values only of the SAME type. Here is vector of `string slice`

```rust
fn main() {
    let cities: Vec<&str> = vec!["Frankfurt", "London", "New York"];
    for c in cities.iter() {
        println!("City: {}", c);
    }
}
```

*Results:*
```
City: Frankfurt
City: London
City: New York
```

---

#### Pattern matching on vector count

```rust
fn main() {
    let names: Vec<&str> = vec!["John", "Susanne", "Judy", "Reginald"];

    let len = names.len();

    match len {
        1 => println!("One element"),
        2 | 3 => println!("Two or three elements"),
        _ => println!("Probably more elements"),
    }

}
```

*Results:* `Probably more elements`

---

#### Indexing

> Collections are 0-index based

Indexing into a non-existing element will cause `panic`.

```rust
fn main() {
    let names: Vec<&str> = vec!["John", "Susanne", "Judy", "Reginald"];

    let fist = &names[9];
    println!("First element is: {}", fist);

}
```

*Results:*
```
thread 'main' panicked at 'index out of bounds: the len is 4 but the index is 9', /tmp/mdeval//READMEmd_78_86.rs:4:17
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

---

#### Accessing with Option monad

Using the `get` function will return `Option` monad that needs to be matched.

```rust
fn main() {
    let names: Vec<&str> = vec!["John", "Susanne", "Judy", "Reginald"];

    let first = names.get(9);
    match first {
        Some(name) => println!("Name found: {}", name),
        None => println!("Name not found"),
    }

}
```

*Results:* `Name not found`

---

### Errors

Simple example with returning an error arm of the result type based on the passed in option.
Here the caller needs to handle the result type.

```rust
use std::io::{Error, ErrorKind};

pub fn file_handler(inp: Option<&str>) -> Result<String, Error> {
    let custom_error = Error::new(ErrorKind::Other, "och no");

    match inp {
        Some(c) => Ok(c.to_string()),
        None => Err(custom_error),
    }
}

fn main(){
        let costam = None;
        let res = file_handler(costam);
        println!("Result: {:?}", res);

        let costam = Some("hi there");
        let res = file_handler(costam);
        println!("Result: {:?}", res);
}
```

*Results:*
```
Result: Err(Custom { kind: Other, error: "och no" })
Result: Ok("hi there")
```

---

### Traits

Traits are very similar to `C# interfaces` but much more versatile and powerful. For example, it's possible to use `dependency injection` techniques from OOP.

Here is an example based on 2 different logging mechanisms.

> One restriction to note is that we can implement a trait on a type only if at least one of the trait, or the type is local to our crate [[1]](#1)
 
```rust
trait LogParse {
    fn parse_me(&self) -> String;
}
struct DbLogger {
    log: String,
    stamp: String,
    db: String,
}
struct StdoutLogger {
    log: String,
    stamp: String,
}
impl LogParse for DbLogger {
    fn parse_me(&self) -> String {
        format!("Log {}, logged at {}, with db engine {}", self.log, self.stamp, self.db)
    }
}
impl LogParse for StdoutLogger {
    fn parse_me(&self) -> String {
        format!("Log {}, logged at {}", self.log, self.stamp)
    }
}

fn main() {
    let dber = DbLogger {
        log: String::from("aaa"),
        stamp: String::from("2022-10-28"),
        db: String::from("mysql"),
    };

    let structer = StdoutLogger {
        log: String::from("aaa"),
        stamp: String::from("2022-10-28"),
    };

   println!("Log parsed {}", structer.parse_me());
   println!("Log parsed {}", dber.parse_me());
}
```

*Results:*
```
Log parsed Log aaa, logged at 2022-10-28
Log parsed Log aaa, logged at 2022-10-28, with db engine mysql
```

---

#### Default implementation

This is very similar to `C# abstract classes`

```rust
pub trait Summary {
    // This does not have a default implementation
    // it will need to be provided
    fn summarize_author(&self) -> String;

    fn summarize(&self) -> String {
        format!(
            "This is a summary of the writing of {}",
            self.summarize_author()
        )
    }
}

struct Article {
    text: String,
    length: u32,
    author: String,
}

impl Summary for Article {
    fn summarize_author(&self) -> String {
        format!("@{}", self.author)
    }
    fn summarize(&self) -> String {
        format!("New implementation {}", self.text)
    }
}

fn main() {
    let blog = Article {
        text: String::from("sample text"),
        author: String::from("John Cleese"),
        length: 234,
    };

    println!("Summary: {}", blog.summarize());
}
```

*Results:* `Summary: New implementation sample text`

---

### ? Operator

[?](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html?highlight=Error%20propagation#where-the--operator-can-be-used) Will return the error value from the current function for the caller to handle.

---

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

---

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
/tmp/mdeval//READMEmd_176_184.sh: line 7: warning: here-document at line 1 delimited by end-of-file (wanted `EOF')
    Finished dev [unoptimized + debuginfo] target(s) in 0.24s
     Running `target/debug/rust-exercises arg1 arg2`
Hello World!
```

---

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


---

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

---

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

---

## Links & Resources

- [Rust in examples](https://doc.rust-lang.org/rust-by-example/index.html)

---

## Reference

<a id="1" href="https://doc.rust-lang.org/book/ch10-02-traits.html#:~:text=one%20restriction%20to%20note%20is%20that%20we%20can%20implement%20a%20trait%20on%20a%20type%20only%20if%20at%20least%20one%20of%20the%20trait%20or%20the%20type%20is%20local%20to%20our%20crate">[1]</a> : Trait restrictions to crate
