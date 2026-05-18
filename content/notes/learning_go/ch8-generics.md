---
title: Chapter 8 Generics
weight: 8
type: docs
---

## Generics Reduce Repetitive Code and Increase Type Safety

- Things impossible without generics:
  - Check if multiple calls to a function accepting an `any` interface use the same type (e.g. `binTree.Insert()`).
  - Create an instance of a variable specified by an interface.
  - Check if two variables of type `any` are of the same type.
  - Process a slice of `any` type without `reflect` - **functions that operate on slices would be repeatedly implemented for each type**.

## Introducing Generics in Go

- [Type Parameters Proposal](https://go.googlesource.com/proposal/+/refs/heads/master/design/43651-type-parameters.md)

```go
type Stack[T any] struct {
    vals []T
}

func (s *Stack[T]) Push(v T) {
    s.vals = append(s.vals, v)
}

func (s *Stack[T]) Pop() (T, bool) {
    if len(s.vals) == 0 {
        var zero T  // get a zero value for a generic type
        return zero, false
    }
    v := s.vals[len(s.vals)-1]
    s.vals = s.vals[:len(s.vals)-1]
    return v, true
}

// Usage
func main() {
    var intStack Stack[int]
    intStack.Push(1)
    intStack.Pop()
}
```

- Type parameter information is placed within brackets. There is a **type constraint**, which is an interface to specify valid types.
  - A new built-in interface `comparable` is defined in the universe block to support comparison operators (`==`, `!=`).


## Generic Functions Abstract Algorithms

- Generic functions are also supported.

```go
func Map[T1, T2 any](s []T1, f func(T1) T2) []T2 {
    result := make([]T2, len(s))
    for i, v := range s {
        result[i] = f(v)
    }
    return result
}
```

## Generics and Interfaces

- You can also create interfaces that have type parameters.

```go
type Differ[T any] interface {
    fmt.Stringer
    Diff(T) float64
}

func FindCloser[T Differ[T]](pair1, pair2 Pair[T]) Pair[T] {
    ...
}
```

## Use Type Terms to Specify Operators

- **Type elements** in an interface specify which types can be assigned to a type parameter and which operators are supported. They list concrete types separated by `|`. **The allowed operators are the ones valid for all listed types.**
  - Put a `~` before a type term to allow **any type whose underlying type is that type**.
  - The interface may have method elements as well.

```go
// All integer types support the modulus operator
type Integer interface {
    ~int | ~int8 | ~int16 | ~int32 | ~int64 |
    ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64 | ~uintptr
}

// All types that support comparison operators
// The cmp package is introduced in Go 1.21 with this interface
type Ordered interface {
    ~int | ~int8 | ~int16 | ~int32 | ~int64 |
    ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64 | ~uintptr |
    ~float32 | ~float64 |
    ~string
}
```


## Type Inference and Generics

- When type inference isn't possible, all type arguments must be specified.

```go
var a int = 10
b1 := ConvertToInt64(a)
b2 := Convert[int, int64](a)
```

## Type Elements Limit Constants

- **The constants need to be valid for all type terms in the type element if used in numerical operations.**

```go
// Invalid: 1000 is not valid for uint8
func Plus1000[T Integer](a T) T {
    return a + 1000
}

// Valid
func Plus100[T Integer](a T) T {
    return a + 100
}
```


## Combining Generic Functions with Generic Data Structures

## More on comparable

- As mentioned before, interfaces are comparable, but if the underlying value types are not comparable, the code will panic at runtime.

```go
func Comparer[T comparable](a, b T) bool {
    return a == b
}

type MyInt int
type MyIntSlice []int

var a1 MyInt = 10
var b1 MyInt = 10
Comparer(a1, b1)  // OK

var a2 MyIntSlice = []int{1,2,3}
var b2 MyIntSlice = []int{1,2,3}
Comparer(a2, b2)  // compile error: MyIntSlice does not satisfy comparable

var a3 any = a1
var b3 any = b1
Comparer(a3, b3)  // OK

var a4 any = a2
var b4 any = b2
Comparer(a4, b4)  // panic: runtime error: comparing uncomparable type main.MyIntSlice
```


## Things That Are Left Out

- No operator overloading: avoid readability issues.
- No additional type parameters on methods (methods cannot have additional type parameters beyond those of the receiver type): [Type Parameters Proposal](https://go.googlesource.com/proposal/+/refs/heads/master/design/43651-type-parameters.md).
- No variadic type parameters - all variadic variables must match a single declared type.
- Others: specialization (function overloaded with type-specific versions), currying (partially instantiated generic functions), and metaprogramming (produce code at compile time).


## Idiomatic Go and Generics

- Unlike C++ (which uses generics to turn runtime operation into compile-time operation), generics in Go don't have better performance, if not worse due to additional runtime lookups.


## Adding Generics to the Standard Library

- Starting from Go 1.21, the standard library includes functions that use generics to implement common algorithms. Prefer using these functions.


## Future Features Unlocked

- Possibility: sum types (a bounded set of types).

