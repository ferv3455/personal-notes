---
title: Chapter 9 Errors
weight: 9
type: docs
---

## How to Handle Errors: The Basics

- In most cases, you should set the other return values to zero values when an error is returned. **Exception: sentinel errors.**

```go
type error interface {
    Error() string
}
```

> Why not use exceptions?
> - Exceptions add at least one new code path through the code. This may lead to unexpected crashes when exceptions are not properly handled.
> - Making errors returned values forces developers to either handle error conditions or explicitly ignore them with `_`.
> - Errors are often handled inside an `if` statement, making it distinguishable from business logic.


## Use Strings for Simple Errors

- A new error can be created with `errors.New()` or `fmt.Errorf()`. The error string is returned by the `Error()` method.
  - **Error messages should not be capitalized nor should they end with punctuation or a newline.**
  - These functions return a **pointer** to an error struct. Therefore, two errors created with the same message are not equal using `==`.


## Sentinel Errors

- **Sentinel errors** are error variables declared at the package level. They are meant to signal that processing cannot continue because of a problem with the current state.
  - They usually have names starting with `Err`.
  - They are tested using equality (`==`), e.g. `if err == zip.ErrFormat { ... }`.
- **Be sure you need a sentinel error before you define one. They are part of your public API, and they should be self-explanatory without additional information.**


## Errors Are Values

- When using custom errors, never define a variable to be of the type of your custom error. Either explicitly return `nil` or define the variable to be of type `error`.

```go
func GenerateError(flag bool) error {
    var err CustomError
    if flag {
        err = ...
    }
    return err  // a struct will always be returned
}

err := GenerateError(false)  // err will never be nil
```


## Wrapping Errors

- **`fmt.Errorf()` can be used to wrap errors with the special verb `%w`.** The convention is to write `: %w` at the end.
  - `%w` allows the error to be added to the error tree.
  - `%v` and `%s` can be used to include the error message without wrapping it.
- **The type of the returned error implements the `Unwrap() error` method to support error wrapping.** It returns the wrapped error.
  - Custom error types need to implement the `Unwrap()` method to support wrapping.
- `errors.Unwrap()` calls the `Unwrap()` method and returns the wrapped error if there is one.
  - In most cases, use `errors.Is()` or `errors.As()` instead of `errors.Unwrap()`.


## Wrapping Multiple Errors

- `fmt.Errorf()` can merge multiple errors by using multiple `%w` verbs.
- `errors.Join()` can also be used to merge multiple errors into one.
- **The type of the returned error implements the `Unwrap() []error` method to support multiple wrapped errors.**
- **`errors.Unwrap()` returns `nil` if the error implements the `[]error` variant of `Unwrap()`.** Therefore, you may need to use type switch to access the wrapped errors.
  - **Use `errors.Is()` and `errors.As()` instead to examine the error tree.**

```go
switch err := err.(type) {
case interface {Unwrap() error}:
    // handle single error
    innerErr := err.Unwrap()
case interface {Unwrap() []error}:
    // handle multiple errors
    innerErrs := err.Unwrap()
default:
    // no wrapped errors
}
```

## Is and As

- `errors.Is()` checks whether an error or any error it wraps match a specific sentinel error.
  - Implement the `Is(target error) bool` method to override the default behavior (compare with `==`) if the error is not comparable with `==`. **This method also allows comparisons against errors that aren't identical instances (custom behavior).**

```go
func (re ResourceErr) Is(target error) bool {
    if other, ok := target.(ResourceErr); ok {
        return re.Code == other.Code || re.Resource == other.Resource
    }
}

if errors.Is(err, ResourceErr{Resource: "Database"}) {
    // catches all database-related errors
}
```

- `errors.As()` checks whether an error or any error it wraps match a specific type. The second argument is a pointer to a variable of the target type or a pointer to an interface.
  - `As` method can be implemented to override the default behavior, but it is rarely needed.


## Wrapping Errors with `defer`

- Put code to wrap errors in a `defer` closure at the beginning of a function. This works well if you wrap every error with the same message.


## `panic` and `recover`

- As soon as a panic happens, the current function exits immediately, and **any `defer`s attached to the current function start running**. When those `defer`s complete, the `defer`s of the caller function run, and so on until `main` is reached. The program exits with a message and a stack trace.
  - If there is a panic in a goroutine, **the `defer` chain ends at the function used to launch the goroutine.**
- **`recover()` can be called within a `defer` to check whether a panic happened.** If so, it returns the value passed to `panic()` and continues normal execution.
  - How it reacts to `panic(nil)` depends on Go version: before Go 1.21, it returns `nil` and continues execution; since 1.21, `panic(nil)` creates a new `PanicNilError`.
  - **A program exits if any goroutine panics without being recovered.**

```go
func div60(i int) {
    defer func() {
        if v := recover(); v != nil {
            fmt.Println(v)
        }
    }()
    fmt.Println(60 / i)
}
```

> This looks like exception handling?
> - Reserve panics for fatal situations only. Use `recover` to gracefully handle these situations (e.g. logging and shutting down).
> - `recover` doesn't make clear what could fail. Idiomatic Go favors explicitly outlining possible failure conditions.

- **`recover` is recommended if you are creating a library for third parties - don't let panics escape the public API. Convert panics into errors at the API boundary.**


## Getting a Stack Trace from an Error

- Use `fmt.Printf` and `%+v` verb to print an error with its stack trace if the error supports it.


