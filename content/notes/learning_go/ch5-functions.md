---
title: Chapter 5 Functions
weight: 5
type: docs
---

## Declaring and Calling Functions

- Function definition: `func div(num int, denom int) int { return num / denom }`
  - When multiple input parameters are of the same type, they can be grouped: `(num, denom int)`

### Simulating Named and Optional Parameters

- Go does not have named and optional input parameters.
- Define a struct that has fields that match the desired parameters to emulate this behavior.

### Variadic Input Parameters and Slices

- The variadic parameter is defined using **`...` before the type**: `values ...int`.
  - It must be the last parameter in the parameter list.
  - The variable is a slice of the specified type.
- **A slice may be supplied to a variadic parameter by appending `...` to the slice variable**: `values...`.

### Multiple Return Values

- The types of the return values are listed in parentheses, separated by commas: `(int, int, error)`.
  - By convention, the `error` is always the last value returned.
- **Do not put parentheses around the returned values - that's a compile-time error**.

### Multiple Return Values Are Multiple Values

- The values returned must be assigned to multiple variables at once with `:=` or `=`. **Assigning to a single variable (as in Python) is a compile-time error.**

### Ignoring Return Values

- Ignore one or more return values: use `_` to assign the unused values.
- **Implicitly ignoring all of the return values is allowed.**

### Named Return Values

- Supplying names to return values **pre-declares variables to use within the function to hold the return values**.
  - You must surround names return values with parentheses even if there is only one.
  - **If only some of the return values need names, use `_` to name the remaining return values**.
- The named return parameters only gives a way to **declare an intent** to use variables to hold return values, but **don't require them to be used**.
- Be careful of shadowing named return value variables.

### Blank Returns - Never Use These!

- With named return values, you can use a blank `return` to return the current values of these variables.
  - A `return` is still needed at the end of the function even if it's blank.
- Avoid using blank returns - they make it harder to understand data flow.




## Functions Are Values

- The type of a function is determined by its **signature** - `func`, types of parameters, types of return values.
  - Example: `func(int, int) (int, error)`.
- Functions are values and can be assigned to variables or stored in data structures.
- **The zero value for a function is `nil`.**
- **Functions cannot be overloaded** - each function name stores only one function value.

> Error handling is what separates the professionals from the amateurs.

### Function Type Declarations

- `type` can be used to define a function type.
  - Functions with the same signature can be assigned to variables of that function type.

### Anonymous Functions

- Only anonymous functions can be declared inside other functions. They can be called immediately (by adding `()` after the function body) or assigned to variables.
- **Anonymous functions are useful for `defer` statements and goroutines**.



## Closures

- Functions declared inside functions are **closures**. They are able to access and modify variables declared in the outer function.
- Closures allow you to limit a function's scope.
- **Closures are useful when passed to other functions or returned from a function. They allow you to use local variables outside of the function.**

### Passing Functions as Parameters

- `sort.Slice` is example of taking a function as a parameter. Passed in as an argument, a closure can **capture** variables from the surrounding function.

```go
sort.Slice(items, func(i, j int) bool {
    return items[i].Value < items[j].Value
})
```

### Returning Functions from Functions

- This is useful for customizing behavior of a function (an example is building middleware for a web server - refer to Chapter 13)


> [!WARNING]
> As in other languages, Go closures **capture** the variable itself (by reference), rather than copying its value at the time the closure is created. This means that if you modify the variable in the outer scope, or if the loop advances, the closure will see the updated value.
> For example, the following code will lead to infinite recursion:
> ```go
> f = func() (any, error) {
>     result, err := f()
>     return result, err
> }
> ```



## `defer`

- The cleanup code is attached to the function with the `defer` keyword.
  - **It will be executed on panic, but not on `os.Exit(...)`.**
  - Multiple functions can be deferred in a function. They are executed in **LIFO (Last In, First Out)** order.
  - The code within `defer` closures runs **after** the return statement. **Any variables passed into a deferred closure aren't evaluated until the closure runs.**
  - The deferred function may return values, but they are ignored.

```go
// Ensure file is closed when function exits
f, err := os.Open("file.txt")
if err != nil {
    log.Fatal(err)
}
defer f.Close()
```

- Deferred functions can be used to **examine or modify named return values**. It allows us to take actions based on an error.

```go
// Rollback transaction if an error occurs; otherwise commit it
defer func() {
    if err == nil {
        err = tx.Commit()
    } else {
        tx.Rollback()
    }
}()
```

- **A common pattern in Go is for a function that allocates a resource to also return a closure that cleans up the resource (called a closer).**



## Go Is Call By Value

- Call-by-value: the arguments are always copied into the function's parameters.
- **Maps and slices behave differently: they are implemented with pointers.** Refer to Chapter 6 for details.


