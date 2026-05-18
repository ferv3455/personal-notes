---
title: Chapter 4 Blocks, Shadows, and Control Structures
weight: 4
type: docs
---

## Blocks (From Outer to Inner)

- **Universe block**: all pre-declared identifiers (`int`, `true`, etc. - **they are not keywords**) live in the universe block.
- **Package block**: variables, constants, types, and functions declared outside of any functions are placed in the package block.
- **File block** (one per file within the package): `import` statements define names from other packages for use in the file block.
- **Function block**: all variables defined at the top level of a function (including parameters) are in the function block.
- Within a function, every set of braces `{}` defines another block, including `if`, `for`, and `switch` statements.

## Shadowing Variables

- A **shadowing variable** has the same name as a variable in a containing block. For as long as the shadowing variable exists, you cannot access a shadowed variable.
  - It is very easy to accidentally shadow a variable with `:=`: **it only reuses variables declared in the current block**.
- Identifiers from the universe block and package block can be shadowed - be careful not to do so.
- `go vet` does not report shadowing as a likely error. Refer to Chapter 11 for third-party tools to detect it.


## `if`

- Go supports declaring variables **scoped to the condition and to both the `if` and `else` blocks** - this is a special block.

```go
if n := rand.Intn(10); n > 5 {
    fmt.Println(n - 5)
} else {
    fmt.Println(n)
}
```

## `for`, Four Ways

### The Complete `for` Statement (C-style)

- `for init; condition; post { ... }`
  - **You must use `:=` to initialize new variables: `var` is not legal here.**
  - One or more of the three parts can be left out.

### The Condition-Only `for` Statement (`while`)

- `for condition { ... }`

### The Infinite `for` Statement (`while true`)

- `for { ... }`

### `break` and `continue`

- `break` exits the loop immediately.
  - An infinite loop and a `if`-`break` can be used to simulate `do/while`.
- `continue` proceeds to the next iteration.
  - Go encourages short `if` statement bodies with `continue` or `break` instead of nested code.

### The `for-range` Statement (Iterators)

- **`for-range` loops iterate over built-in data structures** (strings, arrays, slices, maps, and channels).
  - `for i/k, v := range vals { ... }`: `i/k` is the index/key, `v` is the value.
  - `for _, v := range vals { ... }`: ignore the index/key.
  - `for i/k := range vals { ... }`: only get the index/key.
- **Iterating over maps does not guarantee a specific order.**
  - Fixed map ordering has the following problems:
    - Writing code that depends on a specific order of maps.
    - Hash DoS attacks: sending keys that all hash to the same value to degrade performance.
  - The following modifications were made:
    - The hash algorithm for maps includes a random number generated on creation.
    - The order of `for-range` over a map varies a bit on each execution.
  - **Exception: the formatting functions (`fmt.Print`, etc.) always output maps with keys in ascending order.**
- **`for-range` iterates over runes instead of bytes in strings.** The offset is incremented by the number of bytes in the rune, and **the offset (the number of bytes from the beginning) and the rune value** are returned.
- **The `for-range` value is a copy.** Modifying it does not modify the underlying data structure.

### `for` Loop Scoping Changes

- Prior to Go 1.22, the loop variables are per-loop scope: they are created once and reused on each iteration. [This may lead to unexpected behavior with goroutines or functions](https://go.dev/blog/loopvar-preview).
- [Go 1.22 fixes this behavior](https://tip.golang.org/doc/go1.22#language) by creating new variables on each iteration:
  - For C-style `for` loops, a new variable is created and initialized with the value in the previous iteration.
  - For `for-range` loops, new variables are created and initialized with the next value from the underlying data structure.
- This is a backward-breaking change.

```go
for _, v := range x {
    fmt.Printf("%p\n", &v) // different address on each iteration
}
```


### Labeling Your `for` Statements

- `break` and `continue` can exit or skip over an iterator of an outer loop with **labels**.

```go
outer:
    for _, outerVal := range outerVals {
        for _, innerVal := range outerVal {
            // process innerVal
            if invalidSituation(innerVal) {
                continue outer
            }
        }
    }
```

### Choosing the Right `for` Statement

- Use a `for-range` loop for most cases.
- Use a C-style `for` loop when not iterating over the entire data structure.
- The infinite `for` loop may be used to implement the iterator pattern.



## Expression `switch`

- You may declare a variable scoped to all branches of the `switch` statement.
- Unlike C, **each `case` is a separate block**.
- **Cases in `switch` statements don't fall through.** For multiple values triggering the same logic:
  - Separate multiple matches with commas: `case 1, 2, 3:`.
  - Use `fallthrough` to explicitly continue to the next case. Shouldn't be commonly used.
- **`break` can be explicitly used to exit early from a case.** Shouldn't be commonly used.
- `switch` can be used on any type comparable with `==`.


### Blank Switches

- **A blank `switch` allows using any boolean comparison for each `case`.**
  - Specially scoped variables can be declared as well: `switch x := v; { ... }`.

```go
switch {
case x < 0:
    fmt.Println("x is negative")
case x == 0:
    fmt.Println("x is zero")
default:
    fmt.Println("x is positive")
}
```

### Choosing Between `if` and `switch`

- A `switch` statement indicates that a relationship exists between the values or comparisons in each case (similar comparisons, etc.).


## `goto`

- Go forbids jumps that skip over **variable declarations** and jumps that go into **an inner or parallel block**.
- It makes the code more readable in rare situations, such as error handling.
- A real-world example of `goto`: `floatBits` in `strconv`.


