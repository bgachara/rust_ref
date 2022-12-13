# Notes of Rust PL


## General Features

- Inherent memory safety of the language.
- Manual management of memory.
- On par performance with c and cpp.
- General purpose language that can fall into any domain.
- 178 different built-in architecture compilation targets.

## Rust Toolchain

- **rustc** - is the rust compiler.
- **rustup** - manages rust version.
- **cargo** - main tool during development.

- With Cargo, projects are called packages and can consist of one or more **crates**.
- *cargo new project_name*
- *cargo.toml* serves as the config file for our crate.
- *--lib* flag used when creating a library.

## Naming conventions

- **UpperCamelCase** - reserved for traits and types.
- **snake_case** - reserved for attributes, variables, functions and macros.
- **SCREAMING_SNAKE_CASE** - reserved for constants.

`Familiarise the Cargo commands`

## Cargo Manifest

- helps manage dependencies, compilation options and package metadata and crucial for uploading project to crates.io.

```toml
[package]

name = "mybinary"
version = "3.03.2"
edition = "2021"

description = "describe our crate for crates.io"
keywords = "keywords for search on crates.io"
homepage = "/url"


[dependecies]
#we specify external dependencies here.

```
- versioning has a multitude of ways to deal with complex dependency resolution.

## Statement 

- This is a segment of code that does not return any value.
- End with a semicolon to denote nothing returned

```rust

/* This is a statement, but we can not access its value */

"property";

/* We can access the data created by this statement with the variable amswer */
/* this is a let statement, holds the data but does it not return the value itself */

let answer = "property"

/* a function that does not return any value is also a statement*/

fn say_answer() {
    let answer = "purple";
    println!("{answer}") 
}

```

## Expressions

- This is a code segment that returns a value.
- Rust being a "expression-oriented" language, all code blocks will implicitly return their value unless we utilize a semicolon to terminate the expression.

```rust

/* This will return the &str "green" */

"green"

/* This will return the &str "green" */

fn give_answer() -> String {
    let answer = "green".to_string()
    answer
}

println!("{}", give_answer());    
```

## Patterns

- special syntax rules that are useful in certain situations.
- help make language more readable and allow us to do things otherwise not easily accomplished.

```rust
/* pattern to declare more than one variable with let */

let (x, y) = (5, 10);
```

## Scope and Ownership

- Strict scoping rules allow the compiler to know when memory can be safely accessed.

### Blocks

- block of code is a collection of statements and an optional expression contained within {}

```rust

/* Statement */
{
    let number_1 = 11;
    let number_2 = 31;
    let sum = number_1 + number_2;
    println!("{sum}")
}

/* Expression block */

{
    let number_1 = 11;
    let number_2 = 31;
    number_1 + number_2
}


```
- Blocks can be treated as the single statement or expression they evaluate to. This means we can assign variabels to a block of code.

```rust

let sum = {
    let number_1 = 11;
    let number_2 = 31;
    number_1 + number_2
};

println!("{sum}");

```
- If we look closely we can see functions are actually just callable, named blocks.

```rust

fn sum() -> u32 {
    let number_1 = 11;
    let number_2 = 31;
    number_1 + number_2
}

```

## Scope

- concept of whether or not a particular item exists in memory and is accessible at a certain location in our codebase
- In Rust, scope of any particular item is limited to the block it is contained in.
- When a block is closed, all of its values are released from memory and are then considered *out-of-scope*, else *in-scope*

```rust

let number = 10;

{
    println!("{number}");

    let number = 22;
    println!("{number}")
} /*second declaration of `number` is dropped from memory here, now out-of-scope*/

println!("{number}")

```
## Visibility

- We can make an item accessible outside of its normal scope by denoting it as public with the *pub* keyword.
- All items in rust are private by default, pi only accessed within their declared module and any children modules.

```rust

mod numbers {
    pub const ZERO: i32 = 0;
}

mod another_scope {
    use super::numbers::ZERO;

    fn print_zero() {
        println!("{ZERO}");
    }
}

```
- fields of complex datatypes have their own visibility qualifiers.

```rust

pub struct Number {
    pub value: i32,
}

let mut number = Number { value: 0 }

/*we can only access value directly because it is public*/
number.value += 1;
println!("{}", number.value);

```
- when crate is a library, all items denoted as public will be accessible to anyone who imports our library.

## Ownership

- Scoping rules are very strict and with good reason.
- Managing *lifetimes* and *mutability* in a memory-safe way is much easier when we disallow accessing items from parent blocks.

### Rules

- each value in Rust has a variable that's called it's owner.
- there can only be one owner at a time.
- when the owner goes out of scope, the value will be dropped.

- rules have different implications depending on whether our data is stored on the stack or heap.

### Stack vs Heap

- If we assign a variable to an existing variable with a stack-based type such as *i32*,it will make a computationally inexpensive copy of that value.

````rust

let stack_1 = 32;
let stack_2 = stack_1; /*the value of `stack_1` is copied into `stack_2`*/

/* we now have two values we can with */
println!("{stack_1}")
println!("{stack_2}")

````
- when working with datatypes that utilize the heap, such as *String*, we cannot copy values from one variable to another since heap-based types don't implement the *Copy* trait.
- instead of copying, Rust will instead move the value out of the original variable and into the new one.

```rust
let heap_1 = String::from("Only you can!");
let heap_2 = heap_1; /*the value of heap_1 is moved to heap_2*/

/*we can't print heap_1 because it is now owned by heap_2*/
println!("{heap_2}");

```
- We can choose to clone our data, which is equivalent to copying on the heap, unlike implicit cloning this time it must be implicitly stated.
- We can clone any type that implements the *Clone* trait.

```rust
let heap_1 = String::from("Only you can!");
let heap_2 = heap_1.Clone(); /*we have now cloned the value from heap_1 */

/*we now have two values we can work with*/
println!("{heap_1}");
println!("{heap_2}");

```
- Cloning is only necessary when we need another copy of the data.
- When we are not in need of a separate copy, we can instead *reference* the data.

## Functions

- Ownership with functions work much the same way.

```rust
/* when we have a function that return a value, the ownership of that value is passed to the caller */

fn abc() -> String {
    "abc".to_String()
}

let letters = abc(); /* The value created in abc() is now owned by letters */

/* when we have a function that passes a value through it, it can be thought of as temporarily taking ownership of that value until the function call has completed */
fn print_through(s: String) -> String {
    s
}

let finished = print_through(letters); /* letters have been moved into finished*/
```