---
title: Chapter 3 Composite Types
weight: 3
type: docs
---

## Arrays - Too Rigid to Use Directly

- Array declaration with type `[size]type`.
  - The zero value is **an array with all elements set to zero value**.
  - Arrays can be initialized with **array literals**:
    - `[3]int{1, 2, 3}`
    - `[3]int{1, 2}` (remaining elements are set to zero value)
    - `[12]int{1, 5: 4, 6, 10: 100, 15}` **(sparse array)**: equivalent to `[12]int{1, 0, 0, 0, 0, 4, 6, 0, 0, 0, 100, 15}`
    - `[...]int{1, 2, 3}` (the size can be inferred)
- Arrays can be compared with `==` and `!=`.
- Go only has one-dimensional arrays - multi-dimensional arrays are arrays of arrays: `[2][3]int` is an array of 3-element arrays.
- **Reading or writing past the end of an array or using a negative index is not allowed**: it is a compile-time error (constant or literal index) or a run-time panic (variable index).
- `len()` function returns the length of an array.
- **Converting arrays of different sizes to arrays of identical element types is not allowed.** This means you cannot write a function that works with arrays of any size.
- Refer to Chapter 6 on how arrays work behind the scenes.


## Slices

> Arrays provide the backing store for slices.

- Slice declaration with type `[]type`.
  - The zero value is **`nil`** (discussed in Chapter 6). `nil` has no type (like a literal).
  - Slices can be initialized with **slice literals**:
    - `[]int{10, 20, 30}`
    - `[]int{1, 5: 4, 6, 10: 100, 15}` **(sparse slice)**
  - Slices can also be created with `make` (discussed later).
- **Slices cannot be compared with `==` and `!=`, except for comparison with `nil`.**
  - `reflect.DeepEqual()` can be used to compare slices, but it is primarily intended for testing.
- Multidimensional slices are slices of slices: `[][]int`.
- Reading or writing past the end of a slice or using a negative index is not allowed: it is a run-time panic.

### `len`

- **`len()` is a built-in function because they can do things that cannot be done by ordinary functions: it takes an argument of any type of array, slice, or channel.**
  - This also applies to `append()`, `cap()`, `make()`, and other functions discussed later.
- `len(slice)` returns the length of the slice.
  - `len()` of a `nil` slice is `0`.

### `append`

- `append(slice, values...)` returns a slice of the same type. More than one value can be appended at a time.
  - The source slice can be `nil`.
  - **One slice is appended to another by using `...`: `x = append(x, y...)`** (more on `...` in Chapter 5).
  - Forgetting to assign the returned slice is a compile-time error (value is not used).
  - **All values are appended at once after growing the slice's capacity if needed.**

### Capacity

- Every slice has a capacity that may be larger than the length. When trying to append beyond the capacity, the `append` function uses the Go runtime to allocate a new slice with a larger capacity, copy over the values, and garbage collect the old memory.
  - As of Go 1.14, the capacity is doubled when less than 1024, and increased by 25% afterwards.
- `cap(slice)` returns the capacity of the slice.
  - The capacity of a `nil` slice is `0`.
  - An array can also be passed to `cap()`, returning the length of the array.

### `make`

- `make` creates a **zero-initialized slice with specified initial capacity**.
  - `make([]int, 5)`: a slice of length 5 and capacity 5 (all initialized to zero value).
  - `make([]int, 5, 10)`: a slice of length 5 and capacity 10 (all initialized to zero value).
  - Specifying capacity less than length is a compile-time error (constant or literal as capacity) or run-time panic (variable as capacity).

### Emptying a Slice (Go 1.21)

- `clear(slice)` sets all of the slices elements **to their zero value**.


### Declaring Your Slice

- The primary goal is to **minimize the number of times the slice needs to grow**.
  - Declare as `nil` instead of `[]type{}`/`make([]type, 0)` in most cases. They are identical except for comparison with `nil` and **conversion to JSON** (refer to Chapter 11).
  - **Use `make` with a zero length and non-zero capacity** in most cases when the number of elements is known in advance. If you are sure about the exact size, you may specify the length.

### Slicing Slices

- A slice expression creates a slice from another. The starting offset is 0 by default, and the ending offset is the length of the slice by default.
- **Data is not copied when slicing. That is, the two slices share the same underlying memory.** Changes to an element in a slice affect all slices that share that element.
  - **The capacity is also shared**: the subslice's capacity is the capacity of the original slice minus the offset of the subslice within the original slice. **However, growing any subslice does not affect the original slice's capacity.**
  - This makes `append` behave oddly sometimes: it may overwrite data in other slices.

```go
x := make([]int, 0, 5)
x = append(x, 1, 2, 3, 4)
y := x[:2]
z := x[2:]
fmt.Println(cap(x), cap(y), cap(z)) // output: 5 5 3
y = append(y, 30, 40, 50) // [1, 2, 30, 40, 50]
x = append(x, 60)         // [1, 2, 30, 40, 60]
z = append(z, 70)         // [1, 2, 30, 40, 70]
fmt.Println(x) // output: [1 2 30 40 70]
fmt.Println(y) // output: [1 2 30 40 50]
fmt.Println(z) // output: [30 40 70]
```

- **Use a three-part slice expression `x[low:high:max]` to specify the capacity of the subslice, which helps avoid sharing capacity between slices.**
  - The length of the subslice is `high - low`.
  - The capacity of the subslice is `max - low`.


### Converting Arrays to Slices

- **Using the slice expression on an array creates a slice.**
  - `[:]` converts the array to a slice without copying.
  - **The underlying memory is also shared between the array and the slice.**

### `copy`

- `copy(dest, src)` copies **as many values it can from source slice to destination slice**, limited by whichever slice is smaller, and returns the number of values copied (can be discarded).
  - **The two slices may cover overlapping sections of an underlying slice. Copying is done for all values at once.**

```go
x := []int{1, 2, 3, 4}
num := copy(x[:3], x[1:])
fmt.Println(x, num) // output: [2 3 4 4] 3
```

- Arrays cannot be the source or destination of `copy`, but you may convert them to slices with `[:]`.

### Converting Slices to Arrays

- **Slices can be converted to arrays with type conversion: `([size]type)(slice)`.**
  - **The data in the slice is copied to the new memory.**
  - **The size can be smaller than the slice length - an array will be created from a subset of the slice. However, it cannot be bigger.**
- **Slices can be converted to pointers to arrays with type conversion: `(*[size]type)(slice)`.**
  - **No data is copied. The underlying memory is shared between the array and the slice.**
  - **The size must be less than or equal to the slice length.**


## Strings and Runes and Bytes

- Strings support indexing and slicing, which return a byte and a string, respectively.
  - **Indexes are in bytes, not characters/runes.** Therefore, indexing or slicing may produce invalid UTF-8 sequences. Use them with caution.
- `len(str)` returns the number of bytes in the string.
- Type conversions:
  - A rune or byte can be converted to a string.
  - **Converting an `int` to a `string` using type conversion produces unexpected results: a one-character string.** `strconv.Itoa(i)` should be used instead.
  - A string can be converted back and forth to a slice of bytes or runes.
- **Use functions in `strings` or `unicode/utf8` packages to extract substrings and code points instead of slicing and indexing.**

## Maps

- Map declaration with type `map[keyType]valueType`.
  - The zero value is **`nil`** (discussed in Chapter 6). A `nil` map has a length of 0. **Reading a `nil` map always returns the zero value for the map's value type. Writing to a `nil` map causes a runtime panic.**
  - Maps can be initialized with **map literals**:
    - `map[string]int{}`
    - `map[string]int{"Alice": 23, "Bob": 30}`: if the content spans multiple lines, a comma is required after each line **(including the last line)**.
  - Maps can also be created with `make`, which specifies the initial capacity: `make(map[string]int, 10)`.
- Maps cannot be compared with `==` and `!=`, except for comparison with `nil`.
- **The key for a map can be any comparable type: not a slice or a map.**

### Reading and Writing a Map

- **Reading the value assigned to a map key that was never set returns the zero value for the map's value type.** Incrementing its value works as expected even if the key was never set.
- Always use `=` to write to a map key: `:=` is not allowed.

```go
m := map[string]int{}
fmt.Println(m["Alice"]) // output: 0
m["Alice"]++
fmt.Println(m["Alice"]) // output: 1
```

### The comma ok Idiom

- The comma ok idiom is used to check if a key exists in the map: `v, ok := m[...]`. `ok` is `true` if the key exists.

### Deleting from Maps

- `delete(m, key)` removes the key-value pair from the map.
  - **If the key isn't present or if the map is `nil`, nothing happens.** Therefore, there is no need to check before deleting.

### Emptying a Map (Go 1.21)

- `clear(m)` removes all key-value pairs from the map.
  - It works on `nil` maps as well.


### Comparing Maps (Go 1.21)

- The `maps` package contains helper functions for working with maps: `Equal`, `EqualFunc` can be used to compare two maps for equality.


### Using Maps as Sets

- Using the key of the map for the type you want to store and using a `bool` for the value is a common way to implement sets.
  - The value can also be of type `struct{}`. It uses zero bytes while a `bool` uses one byte. However, the comma ok idiom is needed to check for membership if `struct{}` is used.
- Third-party libraries provide functionalities for set operations like union, intersection, and difference.



## Structs

> Go doesn't have classes and inheritance. It does things a little differently (refer to Chapter 7).

- Type definition of a struct: `struct { ... }`.

```go
struct {
    field1 type1
    field2 type2
    ...
}
```

- Defining a **named struct type**: `type person struct { ... }`. This binds the struct type to the name `person`.
  - `type` seems like `typedef` in C/C++ to define an alias, but **it actually defines a new type** (refer to Chapter 7).
  - Structs can be defined at package level or inside functions.
- Variable declaration and definition:
  - **The zero value has every field set to its zero value (not `nil`).**
  - Structs can be initialized with **struct literals**:
    - `person{"Julia", 40, "cat"}` (comma-separated list of values): **a value for every field must be specified**.
    - `person{age: 30, name: "Beth"}` (map-like syntax): you may leave out keys and specify fields in any order. Any field not specified is set to its zero value.


### Anonymous Structs

- The struct definition (`struct { ... }`) is equivalent to a struct name when used in variable declarations.

```go
var p1 struct {
    ... // fields
}
p1.name = "Alice"

p2 := struct {
    ... // fields
}{
    name: "Bob",
}
```

- Anonymous struct types can only be associated with a single instance. This is helpful in two situations:
  - **Unmarshaling and marshaling**: translating from/to external data formats (JSON or protocol buffers, refer to Chapter 11).
  - Writing tests (refer to Chapter 13).


### Comparing and Converting Structs

- **Structs entirely composed of comparable types are comparable with `==` and `!=`.** Those with slice, map, function, or channel fields are not comparable.
- Comparisons between variables that represent structs of different types are not allowed.
- **Go allows conversion between two struct types if they have the same names, order, and types**.
- If two struct types are being compared, **at least one of them has a type of an anonymous struct**, and both structs have the same names, order, and types, then **comparison is allowed without a type conversion**.

```go
type person struct {
    name string
}
p1 := person{}
var p2 struct {
    name string
}
fmt.Println(p1 == p2) // allowed
```
