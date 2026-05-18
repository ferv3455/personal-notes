---
title: Chapter 7 Types, Methods, and Interfaces
weight: 7
type: docs
---

> Go is designed to avoid inheritance while encouraging composition.

## Types in Go

- Go allows you to declare a type at any block level. 
- Type definition with `type`: declaring a user-defined type to have a concrete underlying type.
  - Abstract type: what a type should do.
  - Concrete type: what is the underlying type and how it does it.


## Methods

- **A method can only be defined at the package block level with a receiver specification.**
  - By convention, the receiver name is a short (one or two letter) abbreviation of the type's name.
  - Just like functions, methods cannot be overloaded.
  - Methods can only be defined in the same package as the type declaration.

### Pointer Receivers and Value Receivers

- **Receivers can be either pointer receivers or value receivers (not both).**
  - If the method modifies the receiver, use a pointer receiver.
  - **If the method needs to handle `nil` instances, use a pointer receiver.**
  - If a type has any pointer receiver methods, a common practice is to **use pointer receivers for all methods for consistency**.
- **Always call the method directly with `.`** whether the receiver is a pointer or a value, and whether the instance is a pointer or a value. The Go compiler automatically handles the conversion (taking the address or dereferencing).
  - **The method set of a pointer instance contains both pointer receiver methods and value receiver methods, while the method set of a value instance contains only value receiver methods.** This will affect interface implementation.
  - Calling a value receiver method with a `nil` pointer will cause a runtime panic.
  - **Calling a pointer receiver method with a non-addressable value will cause a compile-time error.**

```go
type Counter struct {
    total       int
    lastUpdated time.Time
}

func (c *Counter) Increment() {
    c.total++
    c.lastUpdated = time.Now()
}

func (c Counter) String() string {
    return fmt.Sprintf("Total: %d, Last Updated: %s", c.total, c.lastUpdated.Format(time.RFC3339))
}

func main() {
    counter := Counter{} // value instance
    counter.Increment()  // ok: automatically takes the address
    fmt.Println(counter) // ok

    counter2 := &Counter{} // pointer instance
    counter2.Increment()   // ok
    fmt.Println(counter2)  // ok: automatically dereferences

    Counter{}.String()    // ok: value receiver
    Counter{}.Increment() // error: cannot call pointer method on value
}
```

- **Do not write getter and setter methods unless you need them to meet an interface.** The exception is updating multiple fields at once.

### Code Your Methods for `nil` Instances

- Calling a method on a `nil` receiver is allowed - it can be useful in some situations (e.g. checking for base cases).

### Methods Are Functions Too

- A **method value** is the function obtained by accessing a method on a specific instance. **It has the signature of the method without the receiver.** It can access values in the fields of the instance.
- A **method expression** is the function obtained by accessing a method through its type. **It has the signature of the method with the receiver as the first parameter.**

```go
func (a Adder) AddTo(val int) int {
    return a.value + val
}

f1 := adder.AddTo  // method value: func(int) int
result1 := f1(10)  // calls adder.AddTo(10)

f2 := Adder.AddTo         // method expression: func(Adder, int) int
result2 := f2(adder, 10)  // calls adder.AddTo(10)
```

### Functions Versus Methods

- If it depends on values configured at startup or changed overtime, it should be a method of a struct type that holds those values.
- If your logic depends only on the input parameters, it should be a function.

### Type Declarations Aren't Inheritance

- A type declared based on another has the same underlying type, but it is a distinct type: **they cannot be used interchangeably without explicit conversion, and methods are not inherited**.
- **Since literals and constants have no types, they can be assigned to/used in operations with user-defined types with compatible underlying types.**
- **Numeric operations are allowed on user-defined types with numeric underlying types (`MyInt` based on `int`), but not between different types (`MyInt` and `int`).**

### Types Are Executable Documentation

- When you have the same underlying data, but different sets of operations to perform, make two types (one based on another) for clearer documentation.


## `iota` Is For Enumerations - Sometimes

- **Go doesn't have an enumeration type. `iota` lets you assign an increasing value to a set of constants.**
  - Put the invalid value first to catch uninitialized variables.

```go
type MailCategory int
const (
    Invalid MailCategory = iota
    Personal
    Spam
    Social
    Advertisement
)
```

- **In a constant block, if a line has neither the type nor a value, then the type and the assignment from the previous line is repeated.**
  - **The value of `iota` increments for each constant defined in the same block, starting with 0. It doesn't matter whether `iota` is used to define the value.**

```go
const (
    Field1 = 0         // 0
    Field2 = 1 + iota  // 1+1 = 2 (iota increments to 1)
    Field3 = 20        // 20 (iota increments to 2)
    Field4             // 20 (assignment same as the previous line; iota increments to 3)
    Field5 = iota      // 4 (iota increments to 4)
)
```

- **`iota` can also be used in a literal expression.**

```go
type BitField int
const (
    Flag1 BitField = 1 << iota  // 1 << 0 = 1
    Flag2                       // 1 << 1 = 2
    Flag3                       // 1 << 2 = 4
    Flag4                       // 1 << 3 = 8
)
```


## Use Embedding for Composition

- Any type can be embedded within a struct. **This promotes the methods on the embedded type to the containing struct.**
- If there are name conflicts, the embedded field's type is needed to refer to the obscured fields or methods.

```go
type Employee struct {
    Name string
    ID   int
}

func (e Employee) Description() string {
    return fmt.Sprintf("Employee Name: %s, ID: %d", e.Name, e.ID)
}

type Manager struct {
    Employee  // embedded field
    Level int
    Name  string // name conflict
}

// Example
m := Manager{
    Employee: Employee{Name: "Alice", ID: 123},
    Level: 2,
    Name: "Bob",
}
fmt.Println(m.Description())  // calls Employee.Description()
fmt.Println(m.ID)             // access Employee.ID
fmt.Println(m.Name)           // access Manager.Name
fmt.Println(m.Employee.Name)  // access Employee.Name
```



## Embedding Is Not Inheritance

- You cannot assign a variable of the containing struct to an embedded field's type. **You must explicitly access the embedded field.**

```go 
var e Employee = m           // error
var e Employee = m.Employee  // ok
```

- **There is no dynamic dispatch**: if a method on an embedded field calls another method on the embedded field, and the containing struct has a method with the same name, the embedded field's method is invoked.

```go
type Inner struct {}
func (i Inner) Foo() {}
func (i Inner) Bar() {
    i.Foo()
}

type Outer struct {
    Inner
}
func (o Outer) Foo() {}

// Example
o := Outer{
    Inner: Inner{},
}
o.Bar() // Inner.Bar() -> Inner.Foo()
```

- **The methods on an embedded field count towards the method set of the containing struct. They can make the containing struct implement an interface.**



## A Quick Lesson on Interfaces

- Interfaces are defined with `type` and `interface` keywords.
  - Interfaces are usually named with "er" endings.
  - The methods defined by an interface (the method set) must be implemented by a concrete type (included in the method set) to meet the interface.
  - As mentioned in the "Methods" section, **the method set of a pointer instance contains both pointer receiver methods and value receiver methods, while the method set of a value instance contains only value receiver methods.**

```go
// Counter type
type Counter struct {
    total int
}
func (c *Counter) Increment() {
    c.total++
}
func (c Counter) String() string {
    return fmt.Sprintf("Total: %d", c.total)
}

// Incrementer interface
type Incrementer interface {
    Increment()
}

// Two instances of Counter
pointerCounter := &Counter{}
valueCounter := Counter{}

// Assigning to interface variables
var s1 fmt.Stringer = pointerCounter // ok
var s2 fmt.Stringer = valueCounter   // ok
var i1 Incrementer = pointerCounter  // ok
var i2 Incrementer = valueCounter    // error
```


## Interfaces Are Type-Safe Duck Typing

- **Interfaces are implemented implicitly.** Only the caller needs to know about the interface (what the caller needs). Nothing is declared on the concrete type side.
  - It enables both type safety (ensure the methods are implemented) and decoupling (no need to declare with something like `implements`), bridging the functionality in both static and dynamic languages.
- **Using standard interfaces encourages the decorator pattern: factory functions that take in an instance of an interface and return another type that implements the same interface.**

```go
r, err := os.Open(fileName) // r is an *os.File, which implements io.Reader
gz, err := gzip.NewReader(r) // gz is a *gzip.Reader, which also implements io.Reader

func process (r io.Reader) error {} // applies to both *os.File and *gzip.Reader
```


## Embedding and Interfaces

- An interface can be embedded within another interface.
- **An interface can also be embedded in a struct.** In this way, the struct implements the embedded interface with all its methods. However, **the methods are not implemented if not overridden** (they are `nil`) - calling them will cause a runtime panic.


## Accept Interfaces, Return Structs

- Accept interfaces: make code more flexible, and explicitly declare the used functionality.
  - **Both pointer and value instances can be passed in as long as they implement the interface (they may have different method sets though).**
- Return structs: make it easier to gradually update the return values in new versions of the code (updating an interface may break existing structs).
  - An exception is errors - returning the `error` interface is common practice.
- Potential drawback: **invoking a function with parameters of interface types involves heap allocations for each interface parameter.**
  - This is related to how interfaces are implemented in Go (refer to the next section). **If the interface needs to store a copy of something that cannot fit in the data pointer directly, Go allocates the value on the heap and puts a pointer to it into the interface.**



## Interfaces and `nil`

- Interfaces are implemented as a struct with two fields: one for the value and one for the type.
  - **An interface is `nil` if and only if both the value and the type pointers are `nil`.**
  - As long as the type field is non-`nil`, the interface is non-`nil` - even if the value field is `nil`. **You must use reflection/type assertion to check whether the value is `nil`.**
- `nil` for an interface variable means whether you can invoke methods on it.

```go
var p *int = nil
var i any = p         // i is non-nil: type is *int, value is nil
fmt.Println(i == nil) // false

i = nil               // i is nil: type is nil, value is nil
fmt.Println(i == nil) // true
```


## Interfaces Are Comparable

- Two instances of an interface type are equal only if their types are equal and their values are equal. **Whether to compare the actual values or pointers depends on the interface value types.**
  - Here the instance type includes whether it is a pointer or a value.
  - **If the underlying value types are not comparable, comparing the interface instances will trigger a runtime panic.**
  - If you have to compare two interface instances (e.g., in a function), **you may use the `Comparable` method on `reflect.Value` to inspect the interface before using it with `==` or `!=`**.

```go
var di1 DI = 10
var di2 DI = 10
DCompare(di1, di2)   // true
DCompare(&di1, &di2) // false: different values (pointers)
DCompare(&di1, di2)  // false: different types
```


## The Empty Interface Says Nothing

- An **empty interface** `interface{}` can be used to declare variables that can hold values of any type.
  - The syntax is not special case - it is similar to defining variables with anonymous structs.
  - `any` is added as a type alias for `interface{}` in Go 1.18.
- A common use is as a placeholder for data of uncertain schema (e.g., JSON). Now that generics are available, consider using them instead when possible.
  - **Avoid using `any`** - Go is designed as a strongly typed language, and attempts to work around this are unidiomatic.
- Accessing the value of an empty interface:
  - Reads: use type assertions or type switches.
  - Writes: use reflection (refer to Chapter 16).



## Type Assertions and Type Switches

- Type assertion: names a concrete type that implemented the interface or another interface that is **implemented by the concrete type whose value is stored in the interface**.
  - A type assertion must match the type exactly - it fails even if two types share the same underlying type (`int` and `MyInt`). Assertion failure causes a runtime panic.
  - The comma-ok idiom can be used to check if the assertion succeeded. **Always use this - you don't know how others will reuse your code.**

> Type assertion vs type conversion:
> - Type assertions can be applied only to interface types.
> - Type assertions are checked at runtime, while type conversions are checked at compile time. Exception: **conversions between slices and array pointers may fail at runtime**.

- Type switch: specify a variable of an interface type and follow it with `.(type)`. **Usually it is assigned to a new variable valid only within each case block: `switch i := i.(type)` - shadowing is idiomatic here.**
  - `nil` can be used for one case to see if the interface has no associated type.
  - If you list more than one type on a case, **the variable is of type `any`**.
  - Always have a `default` case to catch unexpected types.
- Reflection is used when you don't know the underlying type at all (refer to Chapter 16).


## Use Type Assertions and Type Switches Sparingly

- For the most part, treat a parameter or return value as the type supplied and not what else it could be.
- Some use cases of type assertions/switches:
  - Check if the concrete type also **implements another interface** so that some work can be skipped (e.g. checking for `io.WriterTo` before copying data from `io.Reader`).
    - Drawback: wrapped implementations may not be detected (e.g. errors; `*bufio.Reader` returned by `bufio.NewReader()` hides interface implementations of the passed-in `io.Reader`).
  - APIs may evolve over time. Type assertions may be used to check if the passed-in concrete type **implements new functionality**, and provide fallbacks if not.


## Function Types Are a Bridge to Interfaces

- Go allows methods on any user-defined type. **This allows functions to implement interfaces.**
  - Benefit: **no need to define a struct type just to implement an interface with a single method**. If the method needs to hold states or depends on other functions, a struct type can be still be used.

```go
// In http package, services are registered with Handlers
type Handler interface {
    ServeHTTP(w http.ResponseWriter, r *http.Request)
}

func (mux *ServeMux) register(pattern string, handler Handler) {
    ...
}

// Apart from defining struct types that implement Handler,
// we can define function types that implement Handler.
// This is what http.HandleFunc does internally:

// Define a function type that matches the method signature
type HandlerFunc func(http.ResponseWriter, *http.Request)
// Similar types may also be called **adapters**

// Then define the method on the function type
func (f HandlerFunc) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    f(w, r)
}

// Now any variable of type HandlerFunc implements the Handler interface.
// Functions with the same signature can be converted to HandlerFunc and then used as Handler.
// As is done in http.HandleFunc:

func HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
    DefaultServeMux.register(pattern, HandlerFunc(handler))
}
```


## Implicit Interfaces Make Dependency Injection Easier

- **Dependency injection**: code should explicitly specify the functionality it needs to perform its task. Interfaces make it easier to achieve this.
  - Example code structure: [ch07/sample_code/dependency_injection/main.go](https://github.com/learning-go-book-2e/ch07/blob/main/sample_code/dependency_injection/main.go) **The `main` function is the only part that knows what all the concrete types actually are.**
  - Dependency injection **makes unit testing easier**: mock implementations of interfaces can be provided to test specific parts of the code in isolation. Refer to Chapter 15.


## Wire

- [Wire](https://github.com/google/wire), a dependency injection helper, uses code generation to create concrete type declarations that you wrote in `main`.


## Go Isn't Particularly Object-Oriented (and That's Greate)

- The best word to use to label Go's style is practical.


