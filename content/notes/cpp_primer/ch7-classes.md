---
title: Chapter 7 Classes
weight: 7
type: docs
---

> The ideas behind classes **(abstract data types)**: **data abstraction** (defining interface and implementation) and **encapsulation** (separating interface and implementation).

## 7.1 Defining Abstract Data Types

### Designing the `Sales_data` Class

- **Some operations should not be defined as member functions, but some should.** Refer to section 14.1 for details.

### Defining the Revised `Sales_data` Class

- Member functions must be declared inside the class. They may be defined inside the class itself or outside the class body.
  - Functions defined in the class are implicitly `inline`.
  - `void foo(); void foo() {}` is not allowed in the class body: **class member cannot be redeclared** (unlike ordinary functions).
- With the exception of `static` member functions (section 7.6), when we call a member function, **an extra implicit parameter `this` is initialized with the address of the object**. That is to say, calling a member function is equivalent to calling an ordinary function defined in the class scope with the argument `this`.
  - Any direct use of a member in the class is assumed to be an implicit reference through `this`.
  - **`this` is a `const` pointer.**
- **Adding `const` after the parameter list indicates that `this` is a pointer to `const`: these member functions are `const` member functions.** These functions can be called on `const` objects/references or pointers to `const` objects (only pointers to `const` can be bound to a `const` object).
  - If defined outside the class body, the `const` qualifier must be repeated.
- `*this` can be used to return the object itself (as a lvalue reference).

### Defining Nonmember Class-Related Functions

- Ordinarily, nonmember functions that are part of the interface of a class should be declared in the same header as the class itself.

### Constructors

- A constructor is run whenever an object of a class type is created. A class may have multiple overloaded constructors. They have the same name as the class and no return type.
- **Constructors may not be declared as `const`: the object assumes its `const`ness after construction. Therefore, constructors can write to `const` objects during construction.**
- A **default constructor** is one that takes no arguments. It is used to default-initialize an object (without an initializer).
- A **synthesized default constructor** is defined by the compiler if the class does not **explicitly define any constructors**. It uses the in-class initializers if any, otherwise default-initializes each member.
  - In other words, if a class requires control to initialize an object in one case, it is likely to require control in all cases.
  - The synthesized default constructor may default-initialize members built-in/compound types to undefined values. They should have in-class initializers or an explicit default constructor.
  - **If a class has a member of a class type without a default constructor, the compiler will not generate a synthesized default constructor for that class**.
  - *Refer to section 13.1.6 for other cases that prevent the compiler from generating a synthesized default constructor.*
- Under C++11, `= default` explicitly asks the compiler to **generate the synthesized default constructor**. It can appear inside or outside the class body.
- The **constructor initializer list** specifies initial values for data members. It appears after the parameter list and before the function body (starting with a colon).
  - **The initial value is wrapped in parentheses or curly braces (same semantics as value initialization).**
  - When a member is omitted from the constructor initializer list, it is implicitly initialized using the same process as the synthesized default constructor.
  - The members are initialized before the constructor body is executed (whether or not there is a constructor initializer list).
- *More to be discussed in sections 7.5 (constructors revisited), 15.7 (derived class constructors), 18.1.3 (exception handling), and Chapter 13 (copy control).*

### Copy, Assignment, and Destruction

- Ordinarily, if not explicitly defined, the compiler generates synthesized operations for copying, assignment, and destruction: **copying/assigning/destroying each member of the object**.
  - They are not reliable similarly to constructors (e.g. dynamic memory, refer to section 13.1.4).
- *More to be discussed in Chapter 13 (copy control).*



## 7.2 Access Control and Encapsulation

- Access specifiers enforce encapsulation:
  - `public`: members accessible to all parts of the program.
  - `private`: members accessible to member functions.
- If we use `struct`, members are `public` by default. If we use `class`, members are `private` by default. **Always use `class` if we intend to have `private` members.**

### Friends

- A class allows another class or function to access its non-`public` members by making it a friend: **include its declaration preceded by `friend`.**
- Friends are not members and are not affected by access control.
- A friend declaration only specifies access. **It is not a declaration of the function: in order to be called, it must be declared separately.**
  - Sometimes this is not enforced, but it is good practice to do so.
  - We usually declare the friend function in the same header as the class.


## 7.3 Additional Class Features

### Class Members Revisited

- A class can have local names for types via `typedef` or `using`. They are subject to the same access controls as any other member. **They must appear before usage.**
- Member functions defined inside the class are automatically `inline` (implicit). We can also explicitly declare a member function as `inline` as part of its declaration inside the class body (explicit) or **its definition outside the class body** (`inline` on definition).
  - It is legal to have `inline` on both declaration and definition, but **specifying it only on the definition outside the class body** is preferred.
  - Similar to ordinary `inline` functions, **`inline` member functions should be defined in the same header as the class definition**.
- **A `mutable` data member is never `const`**, even when it is a member of a `const` object or accessed in a `const` member function (via a pointer to `const`). A `const` member function may change a `mutable` member.


### Functions That Return `*this`

- Returning `*this` (a lvalue reference) allows chained calls to member functions.
- **A `const` member function that returns `*this` should have a return type that is a reference to `const`**: `*this` is a `const` object.
- **Overloading can be based on `const`**: it virtually declares the implicit `this` parameter to be a pointer to `const` or not.
  - **The overloaded versions may call the same `const` private utility function to reuse common code**: the non-`const` `this` pointer can be converted to a pointer to `const`. This is useful in maintaining well-designed programs.


### Class Types

- Two classes are different even if they have exactly the same member list.
- We can refer to a class type directly with its name or use the name following the keyword `class` or `struct`: `Sales_data item`, `class Sales_data item`, `struct Sales_data item` (**even if the class is defined with keyword `class`**).
- We can declare a class without defining it (**forward declaration**). After the declaration, the type is an **incomplete type**.
  - **We can define pointers or references to such types, and we can declare but not define functions that use an incomplete type as a parameter or return type**. A class must be defined before we create objects of that type (determine storage size) or access a member (determine its layout).
  - A class cannot have data members of its own type, but it can have member that are pointers or references to its own type: **the class is considered declared after its class name has been seen**.
  - Data members can be specified to be of a class type only if the class has been defined with one exception: static class members (section 7.6).

```cpp
class T;

class A {
  T t;          // error: T is an incomplete type
  T *pt;        // ok: pointer to an incomplete type
  static T st;  // ok: static member of an incomplete type
};
```

### Friendship Revisited

- By designating another class as a friend, the member functions of that class can access all the members of the class granting friendship.
  - **Classes and nonmember functions need not have been declared before they are used in a friend declaration. The name is implicitly assumed to be part of the surrounding scope**.
  - **We can define the friend function inside the class body, but we must still provide a declaration outside the class itself to make that function visible - even to the class where it is defined**.

```cpp
struct X {
  // This definition does not declare f
  friend void f() { /* friend function can be defined here */ }

  X() { f(); }            // error: no declaration for f
  void g();
  void h();
};

void X::g() { return f(); } // error: f hasn't been declared
void f();                   // declares the function defined inside X
void X::h() { return f(); } // ok: declaration for f is now in scope
```

- We may declare a single member function of another class as a friend.
  - This requires careful structuring of the program to accommodate interdependencies (ordering of declarations and definitions): **declaration of that class must appear before the class granting friendship**.
  - Overloaded functions are still different functions - friendship applies to only the specified function.
- **Friendship is not transitive, reciprocal, or inherited.** Each class controls its own friends.




## 7.4 Class Scope

- Every class defines its own new scope. This is why we must use the scope operator `::` to define member functions outside the class.
  - **Once the compiler sees the class name in out-of-class function definitions, the rest of the code (parameter list and function body) is in the scope of the class.**
  - Since the return type appears before the class name, the class must be specified. After that, the class must also be specified for the function name.

```cpp
Window_mgr::ScreenIndex Window_mgr::add_Screen(const Screen &s) {
  screens.push_back(s);
  return screens.size() - 1;
}
```

### Name Lookup and Class Scope

- The compiler processes classes in two steps - **compiling member declarations** first, and **processing member function bodies** next (after the entire class has been seen). Therefore, member functions may use other members **in their bodies** regardless of where they appear.
- Normal **name lookup** process:
  - Look for a declaration in the block where the name is used. Only names declared before the use are considered.
  - Look for a declaration in the enclosing scope before the block.
  - Proceed to other enclosing scopes.
  - The program is in error.
- **Block-scope name lookup inside member definitions**:
  - Look for a declaration inside the member function. Only names declared before the use are considered.
  - Look for a declaration inside the class. All members are considered.
  - Look for a declaration in the enclosing scope **before the member function definition**.
  - Proceed to other enclosing scopes.
  - The program is in error.
- If necessary, we can use the scope operator `::` to specify the scope where a name is defined: **`C::m` indicates the member variable, and `::m` indicates the global variable**.

```cpp
int height;

class Screen {
public:
  typedef std::string::size_type pos;
  void f(pos height) {
    cursor = width * height;         // height is the parameter
    cursor = width * this->height;   // height is the member
    cursor = width * Screen::height; // height is the member
    cursor = width * ::height;       // height is the global variable
  }
  void set(pos);
private:
  pos cursor = 0;
  pos height = 0, width = 0;
};

Screen::pos verify(Screen::pos);
void Screen::set(pos ht) {
  height = verify(ht);  // ok: verify() appears before it is used
}
```

```cpp
class Account {
  Money f2() { return 0; }     // error: Money not defined yet
  void f3() { Money m = 0; }   // ok: Money used in definition
  typedef double Money;
  Money f1() { return 0; }     // ok: Money defined before use
};
```

- **Type names are special: if a member uses a type name from an outer scope, then the class may not subsequently redefine that name (even with the same definition).**
  - The compiler may quietly accept such code even though the program is in error.
  - The best practice is to put definitions of type names at the beginning of a class.

```cpp
typedef double Money;
class Account {
  Money f() { return 0; } // Money from the outer scope
  typedef double Money;   // error: redefinition of Money
};
```



## 7.5 Constructors Revisited

### Constructor Initializer List

- **Members are initialized in the order in which they appear in the class definition.** The order of initialization or whether they appear in the constructor initializer list does not matter.
  - In other words, the compiler initializes members in the order of their declarations: use the value from the constructor initializer list if specified; otherwise, use the in-class initializer if specified; otherwise, default-initialize.
  - **This may lead to problems if one member depends on another for initialization. The compiler may or may not issue a warning.** Therefore, it is a good idea to write constructor initializers in the same order as the member declarations, and to avoid using members to initialize other members.
- **Some members must be initialized and not assigned in the constructor body**: `const`, reference, or of a class type without a default constructor.
- A constructor that supplies default arguments for all its parameters also defines the default constructor.

### Delegating Constructors

- A **delegating constructor** uses another constructor from the class to perform initialization. The constructor initializer list must have **a single entry**: the name of the class itself, and its arguments match another constructor.
- The constructor initializer list and the function body of the delegated-to constructor are both executed **before returning to the function body of the delegating constructor**.

### The Role of the Default Constructor

- **The default constructor is used both for default initialization and value initialization.**
- ***Default initialization happens:***
  - When we define **non-`static` variables or arrays at block scope** without initializers, they are default-initialized.
  - When we define a class with class-type members, and the **synthesized default constructor** is used, then each member is default-initialized (if no in-class initializer is specified).
  - When we define a class with class-type members, and we define a default constructor, then each member **not in the constructor initializer list** is default-initialized (if no in-class initializer is specified).
- ***Value initialization happens:***
  - When we provide **fewer initializers than the size of the array**, the remaining elements are value-initialized.
  - When we define a **local `static` object** without an initializer, it is value-initialized.
  - When we **explicitly request value initialization**: `T()`, `T obj{}`, `T{}`.
  - `std::allocator<T>::construct` also uses value-initialization (refer to section 12.2 for details), which is widely used in the standard library (`std::vector`, etc.).
- **`T obj()` defines a function instead of an object.** Use `T obj` instead for default initialization.


### Implicit Class-Type Conversions

- **Every constructor that can be called with a single argument defines an implicit conversion to the class type. They are called converting constructors.**
  - Implicit conversions work well with function calls with reference-to-`const` parameters: conversion creates a temporary object, which can bind to a reference to `const`.
  - `std::string` has a converting constructor that takes a `const char *` argument - this is why they can be used interchangeably in many situations.
- **Only one class-type conversion is allowed.**

```cpp
std::string null_book = "9-999-99999-9"; // implicit conversion: const char * -> string
item.combine(null_book);            // implicit conversion: string -> Sales_data
item.combine("9-999-99999-9");      // error: two conversions required
item.combine(string("9-999-99999-9"));     // ok: explicit conversion: string -> Sales_data
item.combine(Sales_data("9-999-99999-9")); // ok: explicit conversion: const char * -> string
```

- Class-type conversions may be undesirable. **We can prevent the use of a constructor for implicit conversions by declaring it as `explicit` (not added on definition).**
  - `explicit` is meaningful only for constructors that can be called with a single argument.
- Implicit conversions happen when we perform copy initialization `T t = a`. Direct initialization `T t(a)` directly calls the constructor, so **`explicit` constructors still can be used for direct initialization**.
- **We can use `static_cast` to explicitly force a conversion even if the constructor is `explicit`**: `static_cast<T>(a)` is equivalent to `T(a)` (cast to a temporary object).



### Aggregate Classes

- A class is an **aggregate class** if:
  - All of its data members are `public`.
  - It has no user-defined constructors.
  - It has no in-class initializers.
  - It has no base classes or virtual functions.
- Data members of an aggregate class can be initialized with a braced list of member initializers. **Trailing members are value-initialized if fewer initializers are provided than members.**
- Inherited from C, the initializers can also be in the format of `.{member} = value`. This allows initializing members in a different order.

```cpp
struct Data {
  int i;
  std::string s;
};

Data d1 = { 0, "Anna" };
Data d2 = { .s = "Anna", .i = 0 }; 
Data d3 = { 0 };  // s is value-initialized to ""
```


### Literal Classes

- **Only literal classes may have `constexpr` member functions. They are implicitly `const` (no longer true in C++14).**
- An aggregate class is a literal class if its data members are all of literal type.
- A non-aggregate class is a literal class if:
  - The data members are all of literal type.
  - **It has at least one `constexpr` constructor.**
  - If a data member has an in-class initializer, it must be a constant expression for built-in types or use a `constexpr` constructor for class types.
  - **The class has the default destructor.**
- **A `constexpr` constructor can either be declared as `= default` or have an empty body.** It must initialize every data member with a constant expression or using a `constexpr` constructor.

```cpp
class A { // A is a literal class
public:
  // constructors: default or with an empty body
  constexpr A() = default;
  constexpr A(int w, int h) : width(w), height(h) {}

  // destructor: default (synthesized)

  // literal classes may have constexpr member functions
  constexpr int area() const { return width * height; }
private:
  // literal-type members: constant expression initializers
  int width = 1, height = 1; 
};

constexpr A a;
constexpr int area = a.area();
```


## 7.6 `static` Class Members

- **`static` member functions are not bound to any object; they do not have a `this` pointer.** Therefore, they cannot be declared as `const`. Also, explicit and implicit uses of `this` are not allowed, including using a non-`static` member.
- `static` members can be accessed in the following ways:
  - Directly through the scope operator: `T::m`.
  - Through an object, reference, or pointer: `t.m`, `pt->m`.
  - Inside member functions, use directly by name.
- `static` member functions can be defined inside or outside the class body: **`static` is not repeated on definition outside the class body**.
- **Non-`const` `static` data members should be defined and initialized outside the class body**.
  - This is different from ordinary data members: although in the same form (`T m = value`), ordinary data members need in-class initializers used when constructing an object, while `static` data members need to be initialized only once. If initialized inside the class body, they may be included in multiple files, leading to multiple definitions.
  - `static` member functions may be used to initialize `static` data members. No qualifier is needed on the function name - it is in the scope of the class.
  - The best practice is to define and initialize `static` data members in the same file as ordinary non-inline member functions.
- **`const` integral `static` data members may have in-class initializers, and `constexpr` `static` data members must have in-class initializers.** The initializers must be constant expressions.
  - They can be used immediately after definition - even inside the class itself.
  - Even if a `const`/`constexpr` `static` data member is initialized in the class body, **it should still be defined outside the class body if it is used in a context where the value cannot be substituted with the constant expression**. The out-of-class definition does not specify an initial value.

```cpp
// Example: in-class initializers
class A {
  static const int a;         // ok
  static const int b = 0;     // ok: const integral may have an initializer
  static constexpr int c;     // error: constexpr must have an initializer
  static constexpr int d = 0; // ok
};

const int A::a = 0;
```

```cpp
// Example 1: no out-of-class definition needed
class Account {
  static constexpr int period = 30;
  double daily_tbl[period];  // the only use of period: no extra definition needed
};

// Example 2: out-of-class definition needed
class Account2 {
  static constexpr int period = 30;
};
constexpr int Account2::period; // definition outside the class body
void func(const int &);
func(Account2::period);  // period cannot be substituted: definition is needed
```

- **A `static` data member can have incomplete type.** In particular, it can be an instance of the class itself.
- **A `static` data member can be used as a default argument.** Using a non-`static` member as a default argument provides no object from which to obtain the value and so is an error.

```cpp
class Screen {
public:
  Screen &clear(char = bg);
private:
  static const char bg;
};
```
