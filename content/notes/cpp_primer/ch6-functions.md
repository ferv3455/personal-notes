---
title: Chapter 6 Functions
weight: 6
type: docs
---

## 6.1 Function Basics

- Arguments are initializers for the function's parameters. All parameters are always initialized. **We have no guarantees about the order in which arguments are evaluated (section 4.1).**
  - Implicit type conversion is allowed just as in any initialization.
- A function's parameter list can be empty but not omitted. For compatibility with C, an empty parameter list can be specified as `void`.
  - Local variables at the outermost scope of the function body may not use the same name as any parameter - they behave like local variables.
  - **Parameter names are optional.** Unused parameters can be omitted in a declaration.
- **The return type of a function may not be an array type or a function type, but a pointer to either of them is acceptable (section 6.3 and 6.7).**

### Local Objects

- **Automatic objects** are objects that exist only while a block is executing. Parameters and local variables are automatic objects, which hide declarations of the same name made in an outer scope.
  - Storage for parameters/local variables is allocated when the function begins and released when the function ends.
- **Local `static` objects are initialized at the first time execution reaches their definition inside the function.** Their lifetime continues across function calls.
  - Local `static` objects are value initialized without an explicit initializer.

### Function Declarations

- Function declarations are also known as **function prototypes**.
- Like any other name, a function may be declared multiple times but defined only once. Functions should be declared in header files and defined in source files.
- In a declaration, a semicolon replaces the function body. Parameter names are often omitted in a declaration, but they can be used to improve readability.

### Separate Compilation

```bash
clang++ main.cpp hello.cpp -o main         # compile and link in one step

clang++ -c main.cpp -o main.o              # separate compilation to object code
clang++ -c hello.cpp -o hello.o
clang++ main.o hello.o -o main             # link object files
```


## 6.2 Argument Passing

> Parameter initialization works the same way as variable initialization.

### Passing Arguments by Value

- When we initialize a nonreference type parameter from an argument, the value of the initializer (argument) is copied. The function does not affect the argument.
  - The same applies to pointers: the address is copied.

### Passing Arguments by Reference

- When we initialize a reference type parameter from an argument, it is bound to the object from which it is initialized. Copies are avoided.
- Reference parameters let us return multiple results from a function.

### `const` Parameters and Arguments

- Just as in any other initialization, top-level `const`s (of both sides) are ignored. Because of this, **two functions with parameters differing only in top-level `const`s cannot be overloaded**.

```cpp
// We can pass either a const or a non-const int to the following functions
void fcn(const int i);
void fcn(int i);           // error: redefinition of fcn(int)
```

- Low-level `const`s are not ignored and thus be matched exactly.
- **Using a reference instead of a reference to `const` unduly limits the type of arguments that can be used with the function.** Use references to `const` whenever possible.
  - References to `const` can be bound to rvalues (e.g. arbitrary expressions, literals, objects of convertible types).
  - References to `const` can be bound to `const` lvalues (e.g. `const` objects, low-level `const` references).


### Array Parameters

- Since we cannot copy an array, and an array is usually converted to a pointer (section 3.5), we are actually passing a pointer (without the size) when we pass an array to a function.

```cpp
// These three declarations are equivalent
void print(const int*);
void print(const int[]);
void print(const int[10]);  // dimension is only for documentation purposes

int i = 0, j[2] = {0, 1};
print(&i);    // ok: int*
print(j);     // ok: int*, which can be used to initialize const int*
```

- Three common techniques to manage pointer parameters (sizes):
  - Add an end marker in the array (e.g. null character in C-style strings).
  - Pass pointers to the first and one past the last element (STL convention).
  - Pass a size parameter.
- **Use pointers to `const` whenever possible.**
- We may define a parameter that is a reference to an array - the size is part of the type. However, this also limits the usefulness of the function.
- **A multi-dimensional array is passed as a pointer to its first element, which is an array. Therefore, the sizes of the second and subsequent dimensions must be specified.**


### `main`: Handling Command-Line Options

- The second parameter of `main` (`argv`) is an array of pointers to C-style strings (null-terminated) (`char *argv[]`). The array can be alternatively defined as a pointer (`char **argv`).
- **The element just past the last in `argv` is guaranteed to be 0 (`nullptr`).**


### Functions with Varying Parameters

- Three primary ways to write a function that takes a varying number of arguments: `initializer_list` (values of the same type), variadic templates (section 16.4), ellipsis (for compatibility with C).
- **`initializer_list` from `initializer_list` header works like a `vector` of `const` values.**

```cpp
// Initialization
initializer_list<T> lst;           // default initialization: empty list
initializer_list<T> lst{a, b, c};  // list initialization (may use =)
initializer_list<T> lst2(lst);     // copy initialization (may use =)

// Limited operations
lst.size();
lst.begin();
lst.end();
```

- **We enclose a sequence of values in curly braces to pass them to a function that takes an `initializer_list` parameter** - they must be of the same type as elements in the `initializer_list` (or convertible to that type without precision loss).
- Note: `auto` can be used to deduce the type (`initializer_list<T>`) of an initializer list, and the same idea applies to using range `for` loop on an initializer list. **However, mixed types are not allowed - no implicit conversions are done.**

```cpp
auto lst = {0, 1, 2};         // lst is an initializer_list<int>
auto lst2 = {0.1, 0.2, 0.3};  // lst2 is an initializer_list<double>
auto lst3 = {0, 1, 2.2};      // error: mixed types

for (auto &i : {0, 1, 2}) { ... }    // ok: i is an int
for (auto &i : {0, 1, 2.2}) { ... }  // error: mixed types
```

- **Ellipsis parameters are used to be compatible with C, in which `varargs` is used to extract the arguments.**
  - An ellipsis parameter must appear as the last element in a parameter list.
  - The comma before the ellipsis is optional if there are other parameters.

```cpp
void foo(int, int, ...);
void foo(int, int...);
void foo(...);
```


## 6.3 Return Types and the `return` Statement

### Functions with No Return Value

- **A function with a `void` return type may use `return expression` only to return the result of calling another `void` function** - though it actually returns nothing.

```cpp
void g() {}
void f() {
    return g();  // ok because g() has type void
}
```

### Functions That Return a Value

- **The return value is used to initialize a temporary at the call site (possible copy/move construction), and that temporary is the result of the function call. In our code, that temporary is usually used to initialize another named object (possible copy/move construction).**
  - Values are returned in the same way as variable initialization: the value returned must have the same type as the return type, or it must have a type that can be implicitly converted (section 4.11) to that type.
  - Under C++11, functions can return a braced list of values: they are used for list initialization of the temporary that represents the return value.

```cpp
T func() {
    T obj;
    return obj;   // T tmp = obj;
}
T obj = func();   // T obj = tmp;
```

- Do not return a reference or pointer to a local object. It will be destroyed when the function ends.
  - **Be careful about implicit type conversions!**

```cpp
const string &manip()
{
    string ret;
    if (!ret.empty())
        return ret;      // WRONG: returning a reference to a local object
    else
        return "Empty";  // WRONG: "Empty" is converted to a temporary std::string
}
```

- **Calls to functions that return references are lvalues; other return types yield rvalues.**
- The `main` function is allowed to terminate without a return. 0 is implicitly returned to indicate success. `cstdlib` defines two preprocessor variables for program status return values: `EXIT_SUCCESS` and `EXIT_FAILURE`.
- The `main` function may not call itself recursively.


### Returning a Pointer to an Array

- **We can only return a pointer to an array, not an array (arrays cannot be copied).**
- Defining functions that return pointers or references to arrays can be intimidating. We may use type aliases, trailing return types, or `decltype` to simplify the syntax.

```cpp
// Original way
int (*func(int i))[10];  // returns a pointer to an array of size 10

// Type aliases
typedef int arrT[10];
using arrT = int[10];
arrT *func(int i);

// Trailing return type
auto func(int i) -> int(*)[10];

// decltype
int arr[10];
decltype(arr) *func(int i);
```

### Returning a Pointer to a Function

*Refer to section 6.7.*

## 6.4 Overloaded Functions

- Functions that have the same name but different parameter lists and that appear in the same scope are overloaded.
  - The `main` function may not be overloaded.
- Overloaded functions must differ in the number or the types of their parameters. The following do not count as differences:
  - Return types.
  - Parameter names.
  - **Type aliases (e.g. `typedef`, `using`).**
  - **Top-level `const` qualifiers of parameters.**
- We can overload based on whether a parameter is a reference or pointer to `const` (low-level `const`). The compiler will prefer the non-`const` versions when we pass a non-`const` object or pointer to non-`const`.
  - `const_cast` is useful in overloaded functions - one version can call the other and cast away `const`.
- For any given call to an overloaded function, there are three possible outcomes: best match, no match, ambiguous call.

### Overloading and Scope

> It is a bad idea to declare a function locally.

- Names do not overload across scopes - local variables hide all declarations of the same name in an outer scope.
  - **In C++, name lookup happens before type checking.**


## 6.5 Features for Specialized Uses

### Default Arguments

- Only trailing parameters can have default arguments.
  - We should order the parameters so that those most likely to have default values appear last.
- Arguments in the call are resolved by position. The default arguments are used for the trailing arguments of a call.
- **Each parameter can have its default specified only once in a given scope.** Subsequent declarations can only add a default for a parameter without a default.

```cpp
void func(int, int, char = ' ');
void func(int, int, char = '*');  // error: redeclaration
void func(int, int = 0, char);    // ok: adds a default argument
```

- A default argument can be any expression that has a type convertible to the type of the parameter **except for local variables**.

```cpp
// Global variables
int wd = 80;
char def = ' ';
int ht();

void func(int = ht(), int = wd, char = def);
```

- **Names used as default arguments are resolved in the scope of the function declaration. The value that those names represent is evaluated at the time of the call**.

```cpp
void f2()
{
	def = '*';          // changes the global: affects default argument
	int wd = 100;       // hides the global: does not affect default argument
	func();             // calls func(ht(), 80, '*')
}
```


### Inline and `constexpr` Functions

- Calling a function is apt to be slower than evaluating the equivalent expression: saving/restoring registers, copying arguments, branching to the function, etc.
- A function specified as `inline` may be expanded at each call. **It is only a request: the compiler may choose to ignore it if it is complex.**
- A `constexpr` function can be used in a constant expression. It must meet certain restrictions:
  - The `return` type and the parameter types must be literal types (section 2.4).
  - The function body must contain exactly one `return` statement **except for statements that generate no actions (e.g. null statements, type aliases, `using` declarations)**.
- **A `constexpr` function is permitted to accept non-`constexpr` arguments and return non-`constexpr` values. In this case, the function is evaluated at compile time.** The call will be replaced (inline) by a resulting value iff all arguments are constant expressions (the value is evaluated during compilation). Otherwise, the function is called and executed at runtime.

```cpp
constexpr int new_sz() { return 42; }
constexpr size_t scale(size_t cnt) { return new_sz() * cnt; }

constexpr int foo = new_sz();      // ok
int arr[scale(2)];                 // ok: equivalent to arr[84]
int i = 2;
int arr2[scale(i)];                // error: scale(i) is not a constant expression
```

- **Unlike other functions, `inline` and `constexpr` may be defined multiple times in the program, but all of them must match exactly.** Therefore, it is safe to define them in header files.


### Aids for Debugging

- **Processor variables (e.g. `NDEBUG`) and macro names (e.g. `assert`) must be unique within the program (unless redefined identically).**
- `assert` is a **preprocessor macro** defined in `cassert`, which acts like an inline function. `assert(expr);` evaluates `expr` and if the expression is false, `assert` writes a message and terminates the program.
  - **`assert` does nothing if `NDEBUG` is defined.** Therefore, it should be used only to verify things that truly should never happen (for debugging), not for runtime checks.
- `NDEBUG` can be defined either by `#define NDEBUG` or by using the command-line option `-D NDEBUG` when compiling the program.
- **Variables and processor variables defined by the compiler/preprocessor:**
  - `__func__`: the name of the enclosing function (e.g. `main`).
  - `__FILE__`: the name of the current source file (e.g. `wdebug.cc`).
  - `__LINE__`: the current line number in the source file (e.g. `27`).
  - `__TIME__`: the time the file was compiled (e.g. `20:50:03`).
  - `__DATE__`: the date the file was compiled (e.g. `Jul 11 2012`).



## 6.6 Function Matching

- Function matching involves the following steps:
  - Identify the set of **candidate functions** that have the same name and whose declarations are visible at the point of the call.
  - Select **viable functions** that can be called with the given arguments (same number of parameters, matching/convertible types).
  - Find the **best match**, if any (refer to section 6.6.1). There is an overall best match if there is one and only one function for which (otherwise the call is ambiguous - error):
    - **The match for each argument is no worse than the match required by any other viable function.**
    - At least one argument is a better match than any other viable function.

> Casts should not be needed to call an overloaded function in well-designed systems.


### Argument Type Conversions

- Type conversions are ranked as follows:
  - **An exact match**: identical types, array/function-to-pointer conversion, top-level `const` added/discarded.
  - **`const` conversion** (section 4.11.2): low-level `const` added.
  - **Integral promotion** (section 4.11.1): small integral types converted to a larger type (at least `int`).
  - **Arithmetic conversion** (section 4.11.1) or **pointer conversion** (section 4.11.2): unsigned/signed conversions, floating-point/integral conversions, `nullptr`/`0`/`void *` conversions.
  - **Class-type conversion** (section 14.9).
- Notes:
  - The small integral types always promotes to `int` or a larger integral type when type conversion is needed. Therefore, given two functions that take an `int` and a `short`, respectively, **the `short` version will be called only on values of type `short`**.
  - **All arithmetic conversions (unsigned int/int, int/long, int/double) are equivalent.**

```cpp
void ff(int);
void ff(short);
ff('a');      // calls ff(int): char promoted to int

void manip(long);
void manip(float);
manip(3);     // ambiguous: 3 is an int
manip(3L);    // calls manip(long): exact match
manip(3.14);  // ambiguous: 3.14 is a double
manip(3.14f); // calls manip(float): exact match
```



## 6.7 Pointers to Functions

- Declaring of a function pointer: `ret-type (*name)(parameter-list)`.
- A function's type is determined by its return type and the types of its parameters.
- When we use the name of a function as a value, it is automatically converted to a pointer (just like an array): `pf = func` is equivalent to `pf = &func`.
- **We can use a pointer to a function to call the function** (no need to dereference): `pf(args)` is equivalent to `(*pf)(args)`.
- There is no conversion between pointers to one function type and pointers to another function type, even if the return types and parameter types are convertible.
  - When we declare a pointer to an overloaded function, **the type must match one of the overloaded functions exactly**.
- As with arrays, we can write a function-type parameter, but it will be treated as a pointer.
- **Type alias declarations for functions and function pointers:**

```cpp
typedef bool Func(int, int*);
typedef decltype(f) Func2;
using Func3 = bool(int, int*);

typedef bool(*FuncP)(int, int*);
typedef decltype(f) *FuncP2;
using FuncP3 = bool(*)(int, int*);
```

- As with arrays (section 6.3.3), **we can only return a pointer to a function, not a function**. We may use type aliases, trailing return types, or `decltype` to simplify the syntax.

```cpp
// Original way
int (*func(int i))(int*, int);  // returns a pointer to a function

// Type aliases
typedef int funcT(int*, int);
using funcT = int(int*, int);
funcT *func(int i);

// Trailing return type
auto func(int i) -> int (*)(int*, int);

// decltype
int f(int*, int);
decltype(f) *func(int i);
```
