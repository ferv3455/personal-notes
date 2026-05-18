---
title: Chapter 6 Pointers
weight: 6
type: docs
---

## A Quick Pointer Primer

- A pointer type is written as `*T`.
  - The zero value for any pointer type is `nil`.
    - **`nil` is not another name for 0**: it is an untyped identifier representing the lack of a value.
    - `nil` is defined in the universe block.
- **Pointer arithmetic is not allowed in Go.**
  - The `unsafe` package provides low-level operations on data structures (Chapter 16), but it is exceedingly rare to need it.
- `&` (address operator) takes the address of a variable.
- `*` (indirection operator) dereferences a pointer to access the value it points to.
  - Dereferencing a `nil` pointer causes a runtime panic.
- Creating pointers to values:
  - **The built-in function `new(T)` creates a pointer that points to a zero-value instance of type `T`** (rarely used in practice).
  - Use an `&` before a struct literal: `&T{...}`.
  - Declare a local variable and take its address.
  - For constants, use a **generic helper function** (refer to Chapter 8) that creates a pointer to the constant value.

```go
func makePointer[T any](v T) *T {
    return &v
}
```

## Don't Fear the Pointers

- In languages like Java and Python, **function parameters are also passed by value**. The reason why objects can be modified inside functions is that **instances are implemented and passed as pointers**.
- **Go gives you the choice to use pointers or values for both primitives and structs.**

## Pointers Indicate Mutable Parameters

- Go developers use pointers to indicate that a parameter is mutable.

## Pointers Are a Last Resort

- Pointers make it harder to understand data flow and may create extra work for the garbage collector. Rather than populating a struct via pointer parameters, consider returning a new struct instance.
- **The only time to consider pointer parameters is when the function expects an interface.** An exception is working with JSON (refer to Chapter 13), which needs the pointer to know the value types and allows for memory usage optimization.
- **Use a pointer return type only if there are states within the data type that needs to be modified.** An exception is I/O (refer to Chapter 13).


## Pointer Passing Performance

- For the vast majority of cases (less than megabytes), the difference between using a pointer and a value won't affect your program's performance.

## The Zero Value Versus No Value

- Pointer parameters/return values can be used to distinguish between a zero-value variable and an uninitialized variable.
  - The better approach is the comma ok idiom: a value-boolean pair.
  - An exception is JSON unmarshaling: a field of a pointer type is nullable.

## The Difference Between Maps and Slices

- **A map is implemented as a pointer to a struct.** It is always mutable, which prevents your API from being self-documenting.
  - Use a struct rather than passing a map around.
- **A slice is implemented as a struct with three fields: length, capacity, and a pointer to the block of memory.** When it is copied, the new slice has independent length and capacity, but it points to the same underlying array.
  - By default, you should assume that a slice is not modified by a function. Document it if it does (e.g. buffers).
- **Arrays are different from slices - they are not implemented as pointers. The entire array is copied when passed to a function.**

```go
func modify(arr [3]int) {
	arr[0] = 100
	fmt.Println(arr) // [100 2 3]
}

func main() {
	var a = [3]int{1, 2, 3}
	fmt.Println(a) // [1 2 3]
	modify(a)
	fmt.Println(a) // [1 2 3]
}
```


## Slices as Buffers

- Writing idiomatic Go means avoiding unneeded allocations (to reduce the garbage collector's workload).

## Reducing the Garbage Collector's Workload

- **Go is unusual in that it can increase the size of a stack while the program is running. This is because each goroutine has its own stack, and the Go runtime can dynamically adjust the size of these stacks as needed without interference from the operating system.**
- In order for Go to allocate the data on the stack, it must be a local variable of known size at compile time. **The pointer cannot be returned from the function** (otherwise it escapes the stack). Otherwise, it goes to the heap.
  - The escape analysis is not perfect (some data that could be stored on the stack escapes to the heap): it has to be conservative.
- The heap is the memory managed by the garbage collector. Once **no more variables on the stack point to data on the heap (directly or indirectly)**, the data becomes garbage and will be cleared out by the garbage collector.
  - **Keeping track of available chunks on the heap is non-trivial.** There are garbage-collection algorithms to optimize performance in different ways: higher throughput (find more garbage) or lower latency (finish quickly, used in Go).
  - **A slice of pointers has low spatial locality**, which makes it far slower to read and process (mechanical sympathy).
  - The two factors mentioned above are reasons why pointers should be used sparingly, and why Go performs better than Java and Python, where all objects are allocated on the heap.

## Tuning the Garbage Collector

- **The garbage collector allows the garbage to pile up for a bit before cleaning, which helps to improve efficiency.**
- The Go runtime provides settings to tune the garbage collector:
  - `GOGC` controls the heap size needed to **trigger the next garbage-collection cycle** (default to 100: double the current heap size).
    - Setting `GOGC=off` disables the garbage collector. This sometimes improves performance for programs that require dynamic memory allocation.
  - `GOMEMLIMIT` sets a *soft* limit to the **total amount of memory allowed to use** (default to `math.MaxInt64`).
    - The default unit is bytes, but you can use suffixes like `KiB`, `MiB`, `GiB`, and `TiB`. **They are power-of-two analogues of power-of-ten suffixes like `KB`, `MB`, `GB`, and `TB`.**
    - Limiting memory usage may avoid swapping, which can drastically reduce performance.
    - It can be exceeded when thrashing happens: the garbage collection cycles are rapidly being triggered because a program is repeatedly hitting the limit.


## Exercises

- Compiling the code with `-gcflags="-m"` shows when values escape to the heap.
- Objects passed into `fmt.Println` escape to the heap whether they are pointers or values - `fmt.Println` takes `...any` parameters, and **values passed via an interface parameter always escape to the heap** (refer to Chapter 7).


