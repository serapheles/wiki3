# Rust

## Data Types
Rust is a statically typed language, with types being checked at compile time. The compiler is quite good at determining the correct type, but explicit typing can be declared at assignment time. This is particularly important when an expression can be interpreted as multiple types, such as different numerical types. Primitive types in Rust are referred to as *scalar* types (alternatively, Rust defines a set of types as scalar that are considered primitive in other languages). The scalar types are integers, floating-points, booleans, and characters.

Integers come in signed and unsigned, each with five levels of size (8 bits to 128 bits). Additionally, there is a system dependent size (again, either signed or unsigned) depending on if the running architecture is 32 bit or 64 bit (which is the size used). Integer literals are interpreted as decimal by defaults, but can also be written in hex, octal, or binary. The underscore character "_" can be used as a visual break in the value, similar to a comma in written US English, with no effect on the value itself. Unsigned 8-bit integers specifically can also be written as byte, which is effectively the character for that value.

Floating-point values come in two sizes, 32 bits and 64 bits, and are always signed.

Boolean values are one byte.

Characters in Rust are denoted by ```char```, which is the smallest alphabetic or graphic element supported by Rust. Characters in Rust are four bytes and represent a Unicode scalar value, meaning a single character can be not only a letter like 'a', but can be an emoji included in Unicode. However, what is considered a single character in Unicode may not match one's expectation of what a single character should be, so care must be taken when dealing with character level parsing.

Strings generally come in two forms, ```str``` and ```String```. The former is a string slice, which refers to a sequence of UTF-8 encoded data stored elsewhere (such as hardcoded into the source code). String slices are built into the Rust language itself. The latter type, ```String```, is part of the Rust standard library and has much of the behavior and associated methods one may expect from strings in other languages. ```String``` is built on top of ```str```, which is relatively inflexible itself. A lot of the character array style behavior one might find in a language like C/C++ or even Java can be found in ```String```, but there are some notable differences. The most important difference involves limitations on direct indexing/referencing.

## Expressions
Operation expressions in Rust are mostly evaluated left to right, with the exception of assignment and updating operations (```|=``` and  ```+=```, for example) which are right to left, and comparison expressions (```!=```, et cetera) which require parentheses. Other expressions (that is, those without operators) generally have equal precedence, with left to right evaluation as written in the source code.

## Assignment Statements
Values are assigned to a variable by using the keyword ```let``` followed by the variable name, then an equal sign and the value to assign to the variable.
```let x = 0;```
By default, variables in Rust are immutable. To make a variable mutable, the keyword ```mut``` is added after the keyword ```let```, such as ```let mut y = 1```. Variables are explicitly typed by declaring it after the variable name but before the "=", such as ```let z: u128 = 2```. An assignment statement can also be conditional, such as ```let a = if boolean_value {value1} else {value2}```.

In addition to immutable variables, Rust also has constants, which are similar but must not hold a value that is determined at runtime. ```const MINUTES_IN_A_YEAR: u32 = 365 * 60 * 60```

## Subprograms
Functions are declared by the keyword ```fn``` followed by the function name, an optional type declaration for generics, the list of parameters as the name followed by the type, and finally an optional return value. A full function declaration: ```fn make_phone_vec<R>(mut reader: Reader<R>) -> Vec<Cell> where R: std::io::Read{}```. Rust uses the ```main``` function as the entry point of the program. By default, functions are not public and must be marked as such with the ```pub``` keyword. As a particularly odd quirk, explicitly including a semicolon marks an expression as a statement, so return values do not include semicolons.

Methods are declared inside blocks of code labeled with ```impl``` and the name of the associated struct, but are otherwise the same as regular functions.

## Object Oriented Programming
The book *Concepts of Programming Languages* says a language that is object oriented must support three language features: abstract data types, inheritance, and dynamic binding of method calls to methods. Whether or not Rust fulfills this definition depends on a combination of interpretation and how strictly one adheres to definitions.

At a high level view, Rust has structs, which groups related data into a self contained instance. These structures can then be given methods to define behavior. Since methods (as with any function in Rust) are private by default, there is natural encapsulation. So far, fairly object-oriented.

However, Rust does not have inheritance, as it is understood in a language like Java or C++. Instead, Rust deals with the issues inheritance is used to solve through traits. Traits are a way to ensure behavior, like a more granular version of Java's interfaces, and can be used in a couple different ways. After a trait is defined, declaring a struct implements it ensures it has that behavior (much like a child can override a parent's method, the details of how it is implemented is not guaranteed). Of much more interest, however, is that generics in rust can require a particular trait (or traits). This ensures that a generic type can be used that will be able to fulfill a particular function. By combining enums (which are a bit expanded compared to what may be found in some other languages; they are like a specialized struct), structs, and traits, many polymorphic problems and other places of code reuse can be fulfilled by Rust, in a slightly less traditional way.

## Exception Handling
Rust does not have a concept explicitly called exception, instead Rust conceptually breaks errors into two major categories: recoverable and unrecoverable. These are fairly self explanatory; unrecoverable errors are mostly bugs and recoverable errors are things like missing files or whatever nonsense a user tries to do. There is some potential for overlap in dealing with these, but in general unrecoverable errors use the ```panic!``` macro, while recoverable errors (should) use ```Result<T, E>```.

```panic!``` is called in one of two ways. Either the runtime code calls it on an unrecoverable error, or it is explicitly called via the macro. The main benefit to calling panic is its use in debugging. Its default formatting is actually fairly expansive, much less particular about what is passed to it, or how, than a standard print statement. However, since panic is a symptom of bugs (poorly written code is surely a bug, too), it should generally be avoided in production/released code.

Recoverable errors should use ```Result``` instead. ```Result``` is actually an enum, with two possibilities: ```Ok``` and ```Err```, each of which wraps a generic type. There are quite a few ways to deal with a result, but two notable methods are ```unwrap_or_else``` which either passes the value if it's Ok or executes a closure if it's an Err value, and ```match```, which can have anywhere from a single response on any value to an extensive nested response based on the type of value, error, et cetera. When a function returns a Result, there is a shorthand operator ```?``` to deal with the Result in a way that eliminates a lot of common code.

# Sample Program
```
use std::fs::File;
use std::io::{BufRead, BufReader, ErrorKind};
use std::thread;

//Reads a file line by line, ignoring lines that start with '#', putting each into a vector, which is then returned.
fn read_vec(file_name: &str) -> Vec<String> {
    BufReader::new(File::open(file_name).unwrap_or_else(|error| {
        if error.kind() == ErrorKind::NotFound {
            panic!("File not found: {}", file_name);
        } else {
            panic!("Error reading file {}: {:?}", file_name, error);
        }
    })).lines()
        .map(|x| x.unwrap())
        .filter(|x| !x.starts_with("#"))
        .collect::<Vec<String>>()
}

//Just calls a sort method on a passed vector. Used to show how easy threading is in Rust, compared
//to the abomination that is Java.
fn pass_sort(mut vec: Vec<String>) -> Vec<String> {
    vec.sort();
    vec
}

//Reads the wiki file by line into a vector, creates two vectors from each half of the original,
//then passes each to a sub-thread to be sorted. After they are sorted, the vectors are combined in
//a new vector (which has sorted halves, but is not (likely) sorted overall).
//This was just to show some interesting things about the language, in particular how easy threading
//is.
fn main() {
    let mut vec = read_vec("wiki.md");
    let size = &vec.len() / 2;
    let mut left = Vec::with_capacity(size);
    let mut right = Vec::with_capacity(size);
    left.extend_from_slice(&vec[..size]);
    right.extend_from_slice(&vec[size..]);

    let left_join_handle = thread::spawn(move || {
        pass_sort(left.to_vec())
    });
    let right_join_handle = thread::spawn(move || {
        pass_sort(right.clone().to_vec())
    });

    let mut partial_sorted = Vec::from(
        left_join_handle.join().
            unwrap_or_else(|error| {
                panic!("Problem attempting to sort left: {:?}", error);
            }));
    ;
    partial_sorted.append(&mut right_join_handle.join().unwrap_or_else(|error| {
        panic!("Problem attempting to sort right: {:?}", error);
    }));
}
```
The official Rust book (https://doc.rust-lang.org/book/title-page.html) was used as an extensive reference, though this only captured a small portion of what was said about any given topic.
