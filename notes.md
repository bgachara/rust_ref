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

let finished = print_through(letters); /* letters have been moved into finished */
```
## References

- This is a way of pointing to a particular piece of data within memory.
- By referencing existing data, we can re-use that data without needing to allocate additional memory.
- References are found everywhere in Rust since memory is forcefully managed manually.


```mermaid
graph TD;
    A-->B;
    A-->C;
    B-->D;
    C-->D;
```
### & 

- Every time we declare a value with *let*, we are creating data that is stored in memory.
- We can then create a reference to that data by prefixing our expression with *&*.

```rust

let pi = 3.14159265359;
let funny_number = &pi;

println!("{funny_number}")

```
- We can also create references to references

```rust

let lightspeed = 299792458;

let fast = &lightspeed;
let still_fast = &&lightspeed;

let speed_of_light = &still_fast; /*this is equivalent &&&lightspeed*/

```

## Dereference

- When we need to access underlying data a reference points to directly, we can dereference with the * prefix.

```rust

let mut year = 3020;
let y = &mut year;

*y + 10;

println!("The year is {year}");

```
## Automatic Dereferencing

- Rust compiler will automatically dereference, specifically when the *.* operator is used.
- to_uppercase() automatically does this on earth below

```rust

let planet = "Earth";
let earth = &&&&planet;

assert_eq!("EARTH", earth.to_uppercase());

```

## Ref

- Permits access of an complex data structure's inner value by reference.

```rust

let starship: Option<String> = Some("Omaha".to_string());

match starship {
    Some(ref name) => println!("{}", name),
    None => {}
}

/*without use of ref, nest line would not compile */

println!("{:?}", starship);

```
- Ref can also be used with mutable data with Ref mut

```rust

let mut planet: Option<String> = Some("Waleco_8".to_string());

match planet {
    Some(ref mut name) => {
        name.push('8');
    }
    None => {}
}

```
- Ref technically accomplishes the same thing as & but is placed on the other side of the assignment.
- THought of as reciprocals of each other.

```rust

let val = "reciprocal";

let ref r1 = val;
let r2 = &val;

assert_eq!(r1, r2);

```
## Slices

- a reference to a range of elements from a collection is called a slice
- We use indexed expressions on a referenced collection to get a slice.

```rust

let s = String::from("Hello World");

let hello = &s[0..5];
let world = &s[6..11];

println!("{hello}{world}");

```

## Variables

- Is an identifier that points to a location in memory, which can either be data or function.

### Variable Declarations

- we use the *let* keyword with the *=* operator taking the form

```rust

let variable = "this is a &str";

```

- we can assign variables to any expression.

```rust

/* Closure */
let double = |d| d * 2;

/* This is the outcome of calling the closure */
let var = double(10);

/* Re-assign the value of the var */
let doubled_var = var;

```

## Inferred Types

- Rust compiler is good at inferring types based on context.

```rust

fn double(num: u128) -> u128 {
    num * 2
}

let stars = 10; /*type of u128*/

```
- When primitive types have no context , rust will fall back to *i32* for untyped integer literals and *f64* for untypes floating point literals.

```rust

let integer = 20; /* type i32 */
let float  = 2.1; /* type f64 */

```

## Type Signature

- We can manually annotate types by providing a type signature, which are declared with *: Type* placed after var name and before =: operator.

```rust

let small_integer: u16 = 28;

fn double(num: u128) -> u128 {
    num * 2
}

let unsigned_int: u8 = 28;

```
## Shadowing

- We can assign a new value to the same variable within the same scope without altering the original statement, this is called *Shadowing*

```rust

let favorite = "orange";
println!("{favourite}");

let favorite = "cerulan";
println!("{favourite}");

let favorite = "yellow";
println!("{favourite}");

```
- Shadowing a variable will always allocate memory for the new variable.

## Pattern Binding

- let statements accept a pattern on the lhs of the = operator.

```rust

let (a, b) = (10, "pie");

```
- We can also create an array when declaring values for each member.

```rust 

let [noun, verb, adjective ] = [ "arrays", "are", "homogenous" ]

```
## Unused variables

- Rust compiler will give us warnings when we have unused variables.
- Prefix var names with *_* to escape check

```rust

fn no_warnings(){
    let _used = "primal";
    let _unused = "human"

    println!("{_used}")
}

```

## Mutability

- Ability of a variable's value to be altered in memory.
- In Rust, all variables are immutable by default.
- Design is extremely useful in practice and helps avoid unitended behaviour.

### Mut keyword

- Once a variable is declared with let, its value cannot change.
- We must declare its immutability with *mut* keyword.

```rust

/* Immutable */
let three = 3;

/* mutable */
let mut king = "dead"

```
- Once declared as mutable, redeclare with *=*

```rust

let mut number = 20;

/*explicit reassign */
number = 80;

```
- Although mutable use cases exist for certain problems, it is advisable to avoid mutability in Rust.
- To change a variable's value without mutation, we can choose *shadowing* a variable instead.

```rust

let number = 10;
let number = number + 10;

println!("{num}"); /*prints 20*/

```
- This allows us to avoid the complexity of mutability but at the cost of extra memory allocation.

## Mutable References

- It is possible to mutate values by reference with &mut.

```rust

fn question(s: &mut String) {
    s.pop();
    s.push('?');
}

let mut sentence = String::from("I am");
question(&mut sentence);

println!("{sentence}");

```
- We can only have one mutable reference to a piece of data at a time.
- This means we cannot immutably borrow a mutable reference outside the lifetime of the mutable reference.

## Interior Mutability

- For more complex data types, we can only declare mutability on the entire type, either mutable or immutable.

```rust

struct Coordinate{
    x: i32,
    y: i32,
}

let mut coord = Coordinate { x: 20, y: 20};

/*we cannot mutate the fields because coord is mutable*/
coord.x = 32;
coord.y = 40;

```
- Types exist to allow for interior mutability on the Rust std library, be warned though that these types can circumvent compile-time guarantees and generate run-time errors when not properly used.

## Constants

- Constats are immutable data or functions that are declared at compile time, preferred when piece of data is used in many places viaout the codebase and want to avoid code duplication and ease development.
- *const* keyword used and SCREAMING_SNAKE_CASE convention.

- Constants require a type declaration and only types with a known size at compile time can be declared a constant

```rust

const ANIMAL: &str = "penguin";

/*String does not have a known size, so not used as constant*/
/*Below code will not compile*/
/*const OOPS: String = String::from('sorry'); */

```
- Constant can be assigned to any expression as long as that expression can be computed at compile time.

```rust

const SECONDS_IN_A_DAY: usize = 60 * 60 * 24;
```
- Constants follow the same visibility requirements as any other expression.

## Const fn

- In Rust, function pointer are a primitive data type, means we can declare functions as constants.
- Making a function constant will enforce restrictions to validate that the function will provide the same result when evaluated both at compile-time and runtime.
- Same concept as a mathemtically pure function and helps prevent unintended side effects.
- const fn parameters are limited to datatypes with a known size at compile-time.

```rust
const fn days_to_seconds(days: usize) -> usize {
    days *60 * 60 * 24
}
 
// We can utilize constant functions within other constant declarations
const WEEK_IN_SECONDS: usize = days_to_seconds(7);
 
let february_in_seconds = days_to_seconds(28);
 
println!("{WEEK_IN_SECONDS}");
println!("{february_in_seconds}");

```

## Associated constants

- Constants declared within traits.

```rust

trait Golf {
    const BIRDIE: i32 = -1;
}
 
struct Caddy;
 
impl Golf for Caddy {}
 
println!("{}", Caddy::BIRDIE);

```

## Modules 

- Separation of codebase into distinct sections helps.
- Rust has a module system that provides user-defined namespacing within our codebase.

### mod

- We use the mod keyword to define a module.
- A module has its own distinct scope and visibility.

```rust

mod cake {
    pub fn is_favorite(name: &str) -> bool {
    name == "Coconut"
    }
}

```

- Once declared its content can be accessed by utilising path syntax
- A path is created by chaining any number of nested modules together with the :: operator.

```rust

let guess = cake::is_favorite("Marble");

```
- Utilising paths intentionally in our code increases readability.

## Nesting modules

- Modules can be nested indefinitely.

```rust
mod cake {
  pub mod flavors {
    pub const COCONUT: &str = "Coconut";
 
    pub mod toppings {
      pub const SPRINKLES: &str = "Sprinkles";
    }
  }
}
 
println!("{}", cake::flavors::COCONUT);
println!("{}", cake::flavors::toppings::SPRINKLES);

```

## Importing Items

- With the *use* keyword we can import any module or contained item into the current scope, followed by the path to the item we wish to import.
- Items to be imported must be desclared public *pub*, all items are private by default.

```rust

mod cake {
  pub mod flavors {
   pub const COCONUT: &str = "Coconut";
}
}
 
/* A module must be `pub` to access it by name.*/
use cake::flavors;
 
println!("{}", flavors::COCONUT);

```
## Exporting Items

- When the crate is a library, making an item public with pub will expose that item to users of our library.
- Rust allows us to limit where an item is accessible from when we make it public.
- If we have a function we want to make public internally within the crate and not to users out of it we can use *pub(crate)*

```rust

pub(crate) fn print_lemon() -> {
    println!("Lemon");
}

```
- Other designations include the parent module, *pub(super)* or we can designate a specific module with the *in* keyword, *pub(in path::to::module)*

## Separate files

- When we add files to our src folder,we can treat those file's content as modules to be imported.
- i.e add mod filename to define the file as a module.
- When using separate files, we can nest our files within folders that have the same name as the module.
- mod.rs inside a folder can take the place of a named file.

### crate, super, self

- For access of module that are not direct children of current module
    - crate - access modules from root of our project.
    - super - access relative parent module.
    - self - access current module.

## External crates

- after adding as dependency to cargo.toml, import by name.

### Renaming Imports

- using the *as* keyword.
- avoid naming conflicts and make code readable.

## Macros

- Rust's macro system is a way of manipulating and generating source code 
- Allow for things not possible in the normal language structure or require large amount of code repetition.

### What are they

- Procedures that expand and generate raw source code before the *rustc* compiler begins its compilation step.
- We can spot macros in a Rust program in two different places

```rust
/* Attributes are macros */
#[derive(Debug)]
struct Wow;

let wow = Wow;

/* calls ending with ! are macros */
println!("{wow:?} that is convenient!")

```
- #[derive()] will generate all the source code necessary for Wow to be able to print debug out.
- println! macro allows us to format and print a string with a convenient interpolation syntax.
- Macros are a core part of Rust and are powerful for creating intuitive programmer interfaces for your library.

## Function like Macros

- They look like normal functions whose name ends with a !.
- Unlike functions, input of the body of a macro call is arbitrary.
- We can denote the body of a macro with (), [], {}.

### std Library Macros

- *format!()* - interpolate and format strings.
- *println!()* - internally call and format when printing to stdout.
- *assert!()* - assert a conditional evaluation or panic upon failure.
- *unreachable!()*, *unimplemented!()* - panicking macros.

## Attributes

- Macros that allow for special things such as set compilation options, conditionally compile pieces of code, ignore lints and denote tests and benchmarks.
- Can be declared inside the scope of item it is being applied to, *inner attribute*, or before item being applied *outer attribute*

### Inner attributes

- declared with *#![attribute]* placed as first item declared in its scope.
- can be used in external blocks, functions, implementationsand modules

```rust

/* Inner attributes must be declared before any other items */
/* below code foregos all the compiler warnings */
#![allow(warnings)]

fn main() {
    let unused_variables = "no compiler warnings here.";
}

```
- If applied at the top of a module file then it will only apply to that module and its children.

### Outer attributes

- declare by placing #[attribute] before the item we would like applied to.

```rust

/*defining a test */
#[test] 
    pub fn is_true() {
        assert!(true, "successful test")
    }

```
## Attribute syntax

- Common input patterns

```rust

/* named */
#[no_Std]

/* named with value */
#[must_use = "This function should be used"]

/* named with list of identifiers or paths */
#[forbid(unsafe, warnings)]

/*named with list of keys and values */
#[cfg_atr(target_os = "linux", path = "os/linux.rs")]
```

## Derive

- defined #[derive(Trait)]
- allows us to automatically implement a trait for a type.

```rust
#[Derive(Debug, Clone)]

struct Chair {
    legs: u32,
    wooden: bool,
}

let chair = Chair {
    legs: 4,
    wooden: true,
}

/*we can print the debug output of our Chair type*/
println!("{chair:#?}")
```
- Debug is extremely useful trait for development purposes.
- *PartialEq*, *Eq*, *Copy*, *Clone*

## Available attributes

- Remember since attributes are procedural macros, we are not limited to the ones provided by the language.
- We can use procedural macros created by othersand even make our own.


## Control Flow

## If/else
## else if

```rust

let is_daytime = true;

if is_daytime {
    println!("What a beautiful day")
} else {
    println!("Zzzzzzzzzz")
}

```
## Exhaustiveness

- conditional evaluations are fully exhaustive, so every possible outcome must be accounted for
- This means that when a block is an expression, every other block must return the same type.

## Conditional operators

- used for conditional evaluations.

### Equality 

- can be checked with *==* and non-equality with *!=*
- They will work on any operators that implement the *Eq* and *PartialEq* traits

### Ordering

- operators <, >, <=, >=
- Work on any type that implements the *Ord* and *PartialOrd*

## If/Let

- allows us to compare against data within a complex type by destructuring a type and access its inner value with a concise syntax.
- best exemplified and most commonly encountered when accessing the values of monadic types such as Option<T> and Result<T, E>

### Declaring variables

- since if/let is an expression, it can be used to conditionally declare a variable.

### Destructuring other types.

- *TO RE-READ AND MAKE NOTES*

## Match

- used to handle complex conditional matching in a concise and readable way.
- takes in a pattern and compares it against any number of provided match arms.
- if the body is a single statement or expression we must terminate with a ,.
- match patterns are also exhaustive, meaning all possible patterns must be accounted for.

```rust

let chosen_number = 3;

match chosen_number {
    1 => println!("we have fouund them"),
    2 => {
        println!("dont worry");
        println!("call for them we can");
    }
    3 => println!("This is it"),
    _ => println!("do sth else"),
}

```
- Match patterns are processed from top to bottom.

### Match patterns

- Or pattern **|**
- Ranges **...**exclusive **..=** inclusive
- Binding - we can bind our matched value to a variable to utilize it in a matched block with **@**
- Match guards - perform additional conditional evaluations on each match arm.

## Loops

- allows us to control flow of code execution by repeating a block of code
- behaviour can be boundless or bounded by a conditional evaluation

### loop patterns

- loop
- break, continue
- while
- while let
- for/in  

### Loop labels

- allows us to tag our loops with labels utilizing syntax `label: loop {}

## Functions

- data + instructions for manipulating it = computer program.

### Function declarations

- declare using *fn* keyword, name of fn, arguments and body

```rust

fn main() {
    println!("Howdy");
}

say_howdy();
```
- Function bodies have their own scope and cannot access variables from the local enviroments

```rust
let location = "Kangemi!";

fn print_de_ting(){
/* this will not compile */
    println!("{location}")

let new_location = "Kawangware";
    println!("{new_location}");

}

- Closures are a kind of lazy function that allow access variables from the local environment.

```
### Return values

- we can specify value with -> operator followed by returned type.

```rust

fn another_fun () -> i32 {
    27

/*assign returned value to a new variable*/

let integer = another_fun();

println!("{integer}")
}

```
### Parameters

- Functions take in data to operate on, input parameters and in Rust always require type signature.
- declared with parameter name, followed by a : and type 

```rust

fn multiply(number_1: u32, number_2: u32) -> u32 {
    number_1 * number_2
}

```
- Variadic functions available via macros.

### Functions as parameters

- can pass functions as parameters with the *fn* pointer primitive
- type signature takes the form fn(T) -> T

```rust

fn increment(number: u32) -> u32 {
    number + 1;
}

fn roudabout( top: fn(number: u32) -> u32, new: u32 ) -> u32 {
    top(new)
}

let inc = roundabout(increment, 9)
```
- Forgo the trailing () when passing it as argument

## Closures

- Anonymous functions that can capture the state of the environment.
- Entre' of functional programming in Rust.
- Closures are called lazily which can help provide significant performance benefits under some conditions.

### Closure syntax

- same as functions but input parameters placed between | |.

```rust
/* function */
fn square_fun(num: u32, num2: u32 ) -> u32 { num * num2 };

/* closure */
/* verbose */
fn square_clo | num: u32, num2: u32 | -> u32 { num * num2 };

```
- Rust has ability to infer parameter and return types, and also no need { }

```rust
/* single argument */
let square = |a| a*a;

/* multiple arguments */
let square = |a, b| a * b;

/* no arguments */
let nil = || 9 * 10;
```
- When we store a closure as a variable, we can call it the same we would a function.

### Capturing Enviroment

- Closures differ form functions in that they capture values from the scope they are defined in.

```rust
let house_number = 388;
let print_number = || println!("{house_number}");

print_number();
```
- when we take values from the enviroment we take them by reference.

### Ownership and move

- move keyword tells closure to take ownership of local data it utilises.

```rust
let answer = 98;

let print_ans = move |x| x === answer; 

println!("{}", print_ans(100));

```
### Closure as Ds

- like functions can also be used as fields in structs or tuples.

### Laziness 

- Closures are not computed until they are called.
- For function that takes long to be computed, better performance by rewriting as a closure.

## Function Iteration

- When functions take function as parameters, closures allow us to write very succint code in a functional style.

### Iteration

- any type that implements the Iterator trait gives us a plethora of methods that allow us to operate on collections without having to use a *for* loop.
- we can create an iterator with the iter() method and then proceed through the collection with next()

```rust

let numbers = [1, 2, 3];
let mut numbers = numbers.iter();

if let Some(first) = numbers.next() {
    println!("{first}");
}

if let Some(second) = numbers.next() {
    println!("{second}");
}
```
- Rust also provides us with the into_iter() method for taking ownership of the items of a collection and iter_mut() for accessing a mutable reference to each item.
- Check the other methods that the Iterator trait provides.

### Collect 

- transform an iterator back into a collection.
- Type annotations required for the returned type if they cannot be infered.

```rust 

let mut numbers = [10, 20, 30].iter();
numbers.next();
number.next();

let remaining_number: Vec<&u32> = numbers.collect(); /* [30] */
/* iterators are consuming the above remaining_numbers is a Vec with a single value 30*/
```

### Map

- Takes in a closure that will operate on each value of the collection.

```rust

let numbers = [10, 20, 30];

let nums: Vec<i32> = numbers.iter()
    .map(|a| a * 2)
    .collect();
println!("{nums:?}"); /* [20, 40, 60 ]*/

```

### Filter

- Return values that satisfy a provided boolean conditional

### Enumerate

- Allows us to access the indexes of a collection while iterating.

### Laziness

- Iterators in rust are lazy, evaluated until they are needed.
- When we want an operation that only has a side effect, use a for loop to ensure it is run.

```rust

let numbers = [10, 20, 30];

/* This will compile but not print the numbers */
numbers.iter()
    .map(|r| println!("map: {r}"));

/* This will print all the values of our collection */
for n in numbers {
    println!("for: {n}")
}
```
## Primitives

- Everything in Rust has a type.
- Primitives are types baked into the language itself and not in the std library.

### Boolean

### Integers

- Rust has multiple signed and unsigned integer primitives.
- They are designated by their memory size in bits and whether or not they allow negative numbers.
- *i* signed integers and allow negative numbers, while unsigned integers begin with *u*.
- i.e *u8* is an integer that represents form 0 to 255.
- *isize* = *i8*,*i16*,*i32*,*i64*, *i128* / *usize* = *u8*
- When no type is provided, i32 is infered.

### Floating points

- Two main types with differing precision. f32, f64.
- Denoted by the bit size in memory.

```rust

let integer = 102;
let negative = -42;
let float = 1.23;

/* annotate types */

let byte: u8 = 255;
let large: i64 = -98765677889;
let annotate_float: f32 = 2.3;

/* also postfix the type to the number*/

let new_postfix = 89u16;
let large = -9787654i64
```
## Chars

- This is unicode scalar value.
- Defined by specifying the character within single quote characters.

## Arrays and Vec

- used to create collections of data of the same type.
- Array: used when the collection has fixed length.
- Vec: used when the collection needs to grow and shrink in size.
- For data that is of different types use a struct or tuple.

- Due to their fixed size, arrays are very efficient at runtime.
- We can initializethe values of an array from an expression rather than manually defining each value,
```rust 

let integers = [10, 20, 30];

/* define a large array of e*/

let many_e = ['e'; 20];
```

## Accessing values

- We can access values via the index expression syntax, can be a single value or a range by index.
- single value: collection[2]
- we can also utilize any expression that evaluate to type usize.

```rust
fn one() -> usize { 1 }

let letter_b = array[one()];
```
- Ranges: supply a beginning and ending index separated by ...
- Range syntax is inclusive at the beginning and exclusive at the end.

```rust

let names = ["pluto", "jupiter", "earth", "neptune"];
let sub_names = &names[1...3]; /* [ jupiter, earth*/
println!("{name:?}");

```
## Looping /Iteration

- operate on any collection with for loops and iterators.

```rust
let array_of_chars = [1, 2, 3];

/* looping */
for c in array_of_chars {
    printl!("{c}");
}

/* function iteration */
array_of_chars.iter().map(|c| println!("{c}"));
```

## Vec

- Dynamically sized collection. Vec<T>
- Stores data in heap which allows it to grow or shrink in size.
- we can create new Vec using new() and from() but there also exists a vec![] macro

```rust
/* initialize a new, empty vec */
let new_vec: Vec<char> = vec![];

/* initilaize with values*/
let new_vec: Vec<char> = vec![1, 2, 3];

/* equivalent with keyword */
let mut new_vec = Vec::new();

```
- Like arrays, we also use index syntax expressions to access values.
- Unlike an array, access of out of bounds will compile and panic at runtime.
- use methods get() and first().

## &str and String

- &str is an immutable reference to data in memory, while a String is a collection that allows composition and mutation.
- &str data is stored on the stack, making them computationally efficient.

```rust

let immutable: String = "This is the way";

/* immutable */
let value: &str = "And that is it";

/* annotated string with an explicit lifetime */
let explicit: &'static str = "I am not even close";

```
- Since &str is immutable, we cannot do anything else other than validate them and access their data.
- When we don't know the size of string or plan on manipulating the data, use String.

## String

- This is stored on the heap and hence allow us to mutate value at will.
- While heap is not as fast as the stack,it allows rust to automatically resize the allocated memory at when needed at runtime.

```rust

let empty_string = String::new();
let value_string = String::from("Not Sure");
```
- String is implemented as a Vec<u8>, allowing us to utilize iterators and other methods accessible on Vec<T>.
- Dynamically sized data types don't implement the Copy trait because data is stored on a heap. Better to use Clone 

### Conversion 

- &str to String - use the String::from() or to_string() 
- String to &str - we have to only reference the value.

```rust
/* reference the value */
let privy = String::From('Papaya');

permit(&privy)

```
## Tuples

- To create compound types with differing contained types.
- Declare by placing data within parenthesis ()

```rust

let number = ('p', 38, "this is a trial");
/*access using dot notation*/

let char = number.0;
let number = number.1;
let string = number.2;

```
- Access fields of a tuple using dot notation. 
- We can also destructure a tuple anywhere the Rust syntax allows it.

### Tuple struct

- if we want to reuse a tuple across the codebase, we can define it as it's own custom type.
- This is referred to as a Named Tuple or a Tuple Struct using the struct keyword.

```rust 
/* tuple struct declarations must end with */
pub struct imdb = (&'static str, bool, f64);

/* type signatures are much shorter */
fn get_cat() -> Cat {
    println!()
}
```
- Advantage of naming our tuple is that we can create methods specific to this type utilizing an impl block.

## () Unit Type

- A tuple that does not contain any field() is its own primitive type in Rust.
- we can think of it as a piece of data without any actual data, its value is its own existence.
- Provide a safe way to handle certain situations while avoiding the pitfalls of a "null" type.

- A function which does not return a value actually returns a ().
- Unit Struct: we can also give names to our own custom unit types.

## Structs

- Much like tuples, used to group items of different types, main difference is that here you can give names for each field.
- declare using the struct keyword...with fields provided within its declaration block.
- names should be snake_case and type annotated

```rust

struct Persona {
    name: String,
    age: u8,
    location: String,
}

```

### Instantiating 

- All field options must have values otherwise code will not compile.
- We can also instantiate from variables where if names are the same we can use the shorthand notation.

### Accessing values

- Use the same dot notation with the desired field name.
- Having the fields accessed via names provides more clarity within the codebase.
- Multiple values can also be accessed using the ...var operator.

```rust
let guest_1 = Guest {
    name: "Wanjohi",
    age: 90,
    location: "Nakuru",
};

let guest_2 = Guest {
    name: "Grace",
    ...guest_1
};

println!("{} == {}", guest1.rsvp, guest2.rsvp);

```
- We can also use a function to populate the remaining fields as long as the funtions resolves to the same type for each remaining field.
- This is common on structs that implement the Default trait.
- We can also match on structs like we do on other datatypes.
- We may not want to match on all fields, only a subset hence the use of  _ for the value of a single field and ... for all remaining fields.

### impl Blocks

- they are powerful tools in Rust because when we create a struct we are defining our own custom data type.
- declaring a type allows us to make an impl block to create specialised functions that are specific to that type.
- Functions declared within an impl block are called methods.

## Enums

- ability to have a datatype whose value can only be one of a particular set of variants.
- declared using the enum keyword.
- follow the PascalCase naming convention.

```rust

enum InnerPlanets {
    Mercury,
    Venus,
    Earth,
    Mars,
}

/* use :: to create a value*/
let home = InnerPlanets::Earth;
```
- Since enums can only be of a particular value, matching on an enum is a very common and useful pattern.
- When matching on an enum all variants must be handled otherwise code will not compile.
- One can use the _ operator to handle the remaining unspecified variants.

```rust

let vacation_location = InnerPlanet::Mercury;
 
match vacation_location {
  InnerPlanet::Mercury => println!("Bring sun protection."),
  InnerPlanet::Venus => println!("Quite blue."),
  _ => {
    println!("Brrr...");
    println!("Bring a coat!");
  }
}
```

## Variant values

- Enum variants can also contain values.
- This is possible because enum variants are actually structs in Rust.

```rust

enum Meal {
    Pasta,
    StriFry(Vec<String>),
    Burrito {
        beans: bool,
        rice: bool,
    },
}
```
- We can access a variant's inner data via destructuring.

```rust
let dinner = Meal::Burrito {
    beans: true,
    rice: false,
}

```

- Like structs, enums are our own custom data type.
- We can provide methods for our enums through an impl block.

## Monads and Option

- They circumvent the decades old problem of Null type.

### Option

- is a two variant enum with generic types on at least one of the variants
- The Option<T> is an enum with two variants Some(T) and None

```rust

pub enum Option<T> {
    Some<T>,
    None,
}

let some_str = Some("has a value");
let no_str = None;

```
- Since it only has two variants and we can pass along its Some(T) variant, it somehow acts as a boolean expression that is capable of passing a value with it.
- Useful for situations where the result is unknown, can either be there or missing i.e database request.
- use the unwrap() method to access data of a monadic result. Data for Some() and panic for None.

## Result monad

- Result<T, E> is also another monadic type.
- clever way of handling the propagation of errors in a sensible way using characteristics of monads.
- It is an enum of two variants: Ok<T> denotes success and Err<E> denotes error.

```rust
 enum Result<T, E> {
    Ok<T>,
    Err<E>
 }

 let succes = Ok("good job");
 let error = Err("what has hassed");


fn crib(number:i32) -> Result<bool, String>
{ if number < 0 {
    Err("This is an error from result")
} else if number > 5 {
    Ok(true)
} else {
    Ok(false)
}
}

```
- Thanks to the capacity of monadsto pass generic data on both variant arms, we can both pass a successful boolean result and treat the overall successful operation  of the function as its own boolean result. At the same time able to provide context to our errors.

### Error propagation

- as with Option<>, we use unwrap() to access values
- we can utilize if with is_ok() and is_err() methods.
- unwrap discouraged because of program panic, use if let for safer erro patterns.

### ? operator

- Being able to propagate errors without a lot of syntax is crucial when keeping code clean. more time to focus on the good side of code.
- ? operator used to achieve this exactly.
- When we have a function that returns a Result, any expression within its body that also returns a Result can be appended with ? to force any Err result to be returned immediately.

### Custom Erro Types

- While we can utilize any type for defining error context, enums are a great canditate for errors.
- To allow other errors types to be carried into their own ? operator, we must implement From<T> for the desired type.

## Lifetimes

- Introduced to help handle how long particular references lives before it is dropped from memory.

### Borrow checker

- rustc has a built-in system that looks at scopes of references and checks to make sure that no references are dropped before we try and use them.

```rust
/* we declare an empty variable */
let a;
{
    let b = true; /* we create a var b with value true.*/
    a = &b; /* assign a value to be reference b*/
} /* value of b is dropped because it is out of scope */

/* we can no longer access a because the value it is referencing no longer exists */
/* println!("{}", a)*/
```
- Rust compiler uses `lifetime ellision rules` to determine lifetimes of variables hence no need to always explicitly declare them.

### Annotating lifetimes

- append the & with a *'label* to annotate a lifetime on a reference.
- Lifetimes are generic in nature and usually denoted with simple names such as a' and b'.

```rust

let live: &'a str = "This will annotate the lifetime of '`a'.";

```
- When we create a data structure with lifetimes, we must annotate the lifetime on the field as well as on our new type directly.
- LA for custom types occur between <> placed after the name of our data structure.
- Functions also annotated the same way.

```rust
struct Outfielder<'a> {
    name: &'a str,
}

/* we can declare multiple lifetimes by separating them with commas */

struct Batter<'a, 'b> {
    name: &'a str,
    stance: &'b str,
}

fn pass_to<'a>(name: &'a str) -> String {
    format!("passing the ball to {name}")
}
```
- When we declare an impl block for a type that utilises lifetimes, we must also annotate it directly but they are inherently passed to all contained methods.
- *`static* annotation defines a lifetime as being capable of living for the duration of our program. I.e declaring constants.

```rust

let NEW_CONST: &'static str = "This gotta stick";
```
- can solve alot of lifetime issues by declaring them as 'static
- Generics and lifetimes annotaions are located in the same place, when we have both generics and la for the same item we declare lifetimes first.

## Type Aliasing

- Well-named code is sometimes the best form of documentation.
- In the cases where we feel the need for a custom type but find an already existing type that can fulfill our requirements we can type alias it.
- use keyword type.

```rust

/* Name resolves to String */

type Name = String;

/* use it */
struct Person(Name);

fn print_name(person: Name) {
    let name = person.0;
    println!("{}", name);
}
```

- Importing aliasing: use the `as` keyword to provide alternate names for imports.

## Traits

- Used to define shared behaviour between different types.
- Defines all the methods that a trait must implement to be considered a member of that trait.

### Define shared behavior

- 

```rust

trait Harmonize {
    /* if we omit the body we must define on implementation */
    fn sound(&self) -> String;
    /* if we provide one, acts as a defualt that can be overwritten*/
    fn listen(&mut self) {
        std::thread::sleep_ms(27700);
    }
}

```
### Implementation Traits

- syntax: impl Trait for Type {}
- trait signatures for trait methods must match the trait's definition.

```rust
struct Human(String);

impl Harmonize for Human {
    fn sound(&self) -> String{
        self.0.clone()
    }

    /* already implemented listen*/
}

```
- Traits are most useful when applied to multiple types
- Trait methods are always public
- We cannot implement a trait  from an external crate on a type from an external crate, must make an intermediary type to connect them.

### Generics

- Since traits are methods that must be fulfilled, we can use traits  with generics to create meaningful type signatures.

### Deriving Traits

- provides a way to implement certain types without having to declare an impl block.
- This is called deriving a trait and is accomplished by placing the #[derive(Trait)] before our data structure.

```rust 
#[derive(Debug)]
struct Passerine {
    freq: Vec<u64>,
}

let bird = Passerine {
    freq: vec![827, 23, 12, 189],
};

/* Now it is possible to print the debug out for Passerine type */
println!("{bird:?}")
```
### Scope

- In order to utilize the methods of a trait, that trait must be in scope.

## Generics

- Rust being a strog-types language, we must provide single type signatures for data structures and function parameters.
- This helps the compiler figure out how to manage its memory safely, can however be limiting to a programmer.
- Generic types can work across multiple types.

### When to use

- Imagine working on the velocity of an object, might be measured either in integer or float point numbers.
- Building separate data structures would mean code duplication.
- Generics help avoid this code repetition by combining them into a single datatype using the generic type T.

```rust

struct Velocity<T>(T);

let velocity_int = Velocity(5);
let velocity_float = Velocity(3.8);

```
- data types such as Option<T> and Vec<T>.

## Declaring Generics

- we can utilize generics as fields on custom data types and as function parameters.
- Since generics are situational, any item utilizing  generics must provide a signature declaring the generic types it uses.
- We annotate genrics the same as lifetimes.
- Signature is declared within <> following the item we are annotating.
- Generic types are conventionally named with single uppercase letters such as T.

```rust
/* Here the 'Wrapper' struct utilizes a single generic type T */

struct Wrapper<T> {
    data: T,
}

/* if multiple separate them with commas */
enum Present<T, U> {
    Food(T),
    Card(U),
}

/* in functions, generics are declared after the name and before the parameters */
fn wrap_data<T>(data: T) -> Wrapper<T> {
    Wrapper {
        data,
    }
}
```
- When declaring generics on impl blocks, the generic type is made available to the entire block, means we forgo declaring generics on contained methods.

```rust

impl<T> Wrapper<T> {
    fn get_data(self) -> T {
        self.data
    }
}
```
### Trait bounds

- Trait requirements on generic types are called Trait Bounds.
- We can require multiple constraints on the same generic type.

### impl Trait

- used for function parameters and return values, allows us to utilize generics without having to manually declare them.

### where 

- useful when applying many constraints to generics 

### Turbofish generics
<!-- REVISIT FROM HERE -->

## Declarative Macros

- Most straightforward way of creating own macros is with macro_rules!
- Macros with this special macro are known as declarative macros and are a quick way for us to start manipulating our code.

### macro_rules!

- D-macros allow us to take syntactic frags of Rust language as input and then return raw source code.
- They allow arbitrary repetition of code.

```rust

macro_rules! make_it {
    () => {
        /* everything here will be generated in source code when the macro is called */
    }
}

/* parameters are referred to as metavariables and respective types are fragment-specifiers */
/* declare them the same manner as function parameters, must start with $ and no spaces between the metavariable, its fragment specifier and : */
/* fragment specifiers include expr, ident, stmt, ty, literals, */
```
### exporting macros

- #[macro_export] attribute used.
- visible to other crates when imported as a dependency.
- alternatively use #[macro_use] to export all macros within a module.

## Procedural Macros

- More feature-complete approach to creating macros than a d-macros
- Appear in more places in the language and generate lots of code with minimal input.

### Kinds of P-macros

- Function-like macros: function_like!()
- Attribute macros: #[attribute]
- Custom derive macros: #[derive()]

- P-macros are required to be in their own crate and utilize a different approach to parsing input than declarative macros.


## No Garbage Collection / Runtimes

- No garbage collection pauses.
- No memory overhead( except what you add ).
- Can issue system calls( fork/exec ).
- Can run on systems without an OS.
- Free FFI calls to other languages.

### Built-in dependency management.