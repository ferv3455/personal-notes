---
title: Chapter 2 Predeclared Types and Declarations
weight: 2
type: docs
---

## The Predeclared Types

### The Zero Value

- A default **zero value** is assigned to any variable that is declared but not assigned a value.

### Literals

- Five kinds of literals in Go:
  - **Integer literals** are sequences of numbers.
    - Different prefixes indicate bases: `0b` for binary, `0o`/`0` for octal, `0x` for hexadecimal (uppercase is also allowed).
    - **Underscores are allowed in the middle** - it allows you to separate groups of digits only for readability. They can't be next to each other or at the beginning or end of a number.
  - **Floating-point literals** have decimal points.
    - They can also have an exponent with `e`/`E` followed by a positive or negative integer.
    - Underscores are also allowed.
    - **They can be written in hexadecimal with the `0x` prefix and letter `p` for the exponent: `0xa.bp2`.**
  - **Rune literals** are characters surrounded by single quotes.
    - They can be written as single Unicode characters (`'a'`), 8-bit octal numbers (`'\141'`), 8-bit hexadecimal numbers (`'\x61'`), 16-bit hexadecimal numbers (`'\u0061'`), or 32-bit Unicode numbers (`'\U00000061'`).
  - **String literals** include interpreted string literal (double quotes) and raw string literal (back quotes).
    - Raw string literals can contain any character except a back quote.
  - **Complex literals** are used for complex numbers.
- **Literals are untyped in Go**: they can interact with **any variable that is compatible with the literal**. The values only get a type when the developer specifies one (such as by assignment).
  - **Literals do have default types**: a literal defaults to a type when nothing in the expression specifies a type.


### Booleans

- Variables of `bool` type can have one of two values: `true` or `false`.
- The zero value of a `bool` is `false`.

### Numeric Types

- 8 **integer types**: `int8`, `int16`, `int32`, `int64`, `uint8`, `uint16`, `uint32`, and `uint64`.
  - **`byte` is an alias for `uint8`.**
  - `int` and `uint` are aliases for either `int32`/`uint32` or `int64`/`uint64`, depending on the CPU (32-bit or 64-bit). Because `int` isn't consistent, **it is a compile-time error to assign, compare, or perform operations between `int` and `int32`/`int64` without explicit type conversion.**
  - **`rune` is an alias for `int32`.**
  - `uintptr` is another special name discussed in Chapter 14.
- Using integer types:
  - The zero value of all integer types is `0`.
  - Integer literals default to being of `int` type.
  - Use `int` in most cases. If you are writing a library function that should work with any integer type, write a pair of functions with `int64` and `uint64` parameters (type conversions on function calls).
  - Integers support usual arithmetic operators, bit-manipulation operators, comparison operators and assignment operators.
    - The result of an integer division is an integer (truncation towards zero).
    - Dividing by zero causes a panic (more on panic in Chapter 8).
- 2 **floating-point types**: `float32` and `float64`.
  - The zero value of both types is `0`.
  - Floating-point literals default to being of `float64` type.
  - Use integers when possible: floats are not exact. Otherwise, use `float64` in most cases. 
  - Floating-point types support usual arithmetic operators, comparison operators and assignment operators.
    - Dividing a nonzero by zero returns `+Inf` or `-Inf` depending on the sign of the numerator.
    - Dividing zero by zero returns `NaN` (not a number).
    - Do not use `==` or `!=` to compare floats. Introduce a small tolerance and check if the difference is within that tolerance.
- 2 **complex types**: `complex64` and `complex128`.
  - `complex64` uses `float32` for real and imaginary parts, while `complex128` uses `float64`.
  - The zero value of both types is `0`.
  - Built-in function `complex()` creates a complex number of an appropriate type, and `real()` and `imag()` extract the real and imaginary parts.
  - Go is not a popular language for numerical computing despite its support for complex numbers. `gonum` is a third-party package to use for such purposes.

### A Taste of Strings and Runes

- `string` is the built-in type for strings.
  - The zero value of a string is the empty string.
  - A string literal's default type is `string`.
  - Strings can be compared with `==`, `!=`, `<`, `<=`, `>`, and `>=` operators. Strings can be concatenated with the `+` operator.
  - **Strings in Go are immutable.**
- `rune` is an alias for `int32` (used for wide characters).
  - A rune literal's default type is `rune`.
- **There are no character types in Go. To print a single character represented by a rune/byte, use the `%c` format specifier with `fmt.Printf` instead of `fmt.Print`/`fmt.Println`.**

### Explicit Type Conversion

- Go doesn't allow automatic type promotion: **explicit type conversion is required when types don't match**.

```go
var x int = 10
var y float64 = 30.2
var z float64 = float64(x) + y
var d int = x + int(y)
```

- **Other types cannot be converted to booleans, whether implicitly or explicitly**, which may be used in control flow statements. One of the comparison operators must be used to produce a boolean value.
  - Idiomatic Go values comprehensibility over conciseness.

### Literals Are Untyped

- Literals are untyped - integer literals can be added to floating-point literals without explicit conversion. They can be used with any variable as long as the type is compatible.
  - **When a literal is operated with typed variables, the literal takes the type of the other variable.** If the literal's value is not compatible with the type, it is a compile-time error.


## `var` Versus `:=`

- The verbose way is to use the `var` keyword:
  - `var x int = 10`. 
  - `var x = 10`. The type can be left off if the type on the right side is the expected type.
  - `var x int`. The right side can be left off to use the zero value.
  - Declare multiple variables of the same type at once: `var x, y int = 10, 20`, `var x, y int`.
  - Declare multiple variables of different types at once: `var x, y = 10, "hello"` (via type inference).
  - **Declaration list**:

```go
var (
    x    int
    y        = 20
    z    int = 30
    d, e     = 40, "hello"
    f, g string
)
```

- `:=` can be used to replace **a `var` declaration that uses type inference within a function (not at package level)**.
  - Multiple variables of different types can be declared at once as with `var`: `x, y = 10, "hello"`.
  - `:=` allows assigning values to existing variables **as long as there is at least one new variable on the left side**.
- Avoid using `:=` when:
  - Initializing a variable to its zero value.
  - Assigning an untyped constant or literal and the default type is not what you want for the variable.
  - `:=` sometimes creates unexpected new variables (**shadowing variables**).
- Only declare multiple variables on the same line when assigning values returned from **a function or the comma ok idiom**.
- **Mutable package-level variables should be avoided.**



## Using `const`

- `const` declares a constant with similar syntax to `var`.
- **Constants in Go are a way to give names to literals - they can only hold values that the compiler can figure out at compile time.** Variables cannot be declared as `const`.
  - They can only be assigned literals (numeric, rune, string, complex), `iota` (covered in Chapter 7), or expressions made up of them, built-in functions (`complex`, `real`, `imag`, `len`, `cap`) and operators.
  - There are no immutable arrays, slices, maps, or structs.


## Typed and Untyped Constants

- **An untyped constant works exactly like a literal: it has no type of its own, but does have a default type.** A typed constant can only be assigned to a variable of the same type.

```go
const x = 10
var a int = x      // ok
var b float64 = x  // ok
var c byte = x     // ok

const y int = 10
var d int = y      // ok
var e float64 = y  // error
var f byte = y     // error
```


## Unused Variables

- **Every declared local variable must be read. Otherwise, it is a compile-time error.**
- The compiler's unused variable check is not exhaustive: it won't complain as long as a variable is read once in its lifetime.
  - `golangci-lint` will catch such errors: read it, assign to it, but not use it later.
- Constants are not variables: Go allows unread constants.


## Naming Variables and Constants

- Go requires identifier names to start with a letter or underscore, and the name can contain numbers, underscores, and letters.
  - **Here "letter" and "number" allow any Unicode character that is considered a letter or number.** However, using non-ASCII characters is discouraged.
- **Idiomatic Go doesn't use snake case and uses camel case instead.**
- `_` is a special identifier name in Go (refer to Chapter 5).
- **Do not use all-uppercase letters and underscores for constants like in other languages: Go uses the case of the first letter to determine visibility (refer to Chapter 9).**
- The smaller the scope for a variable, the shorter the name that's used for it.
  - Single-letter variable names are common: `k` (key), `v` (value), `i`, `j` (indexes), `f` (float), `b` (byte), `s` (string), etc.
- Use descriptive names for package-level variables and constants.
