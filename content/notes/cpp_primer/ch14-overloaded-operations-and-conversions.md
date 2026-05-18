---
title: Chapter 14 Overloaded Operations and Conversions
weight: 14
type: docs
---

## Basic Concepts

- Except for the overloaded `operator()`, an overloaded operator may not have default arguments.
- A member operator function has one less explicit parameter than the operator has operands (the first operand is bound to `this`). **The left-hand operand must be an object of the class (no conversions).**
- **An operator function must either be a member of a class or have at least one parameter of class type.** This means built-in types cannot have overloaded operators.
- `+`, `-`, `*`, `&` can serve as both unary and binary operators. Both can be overloaded.
- Operators that can or cannot be overloaded:

<img src="../attachments/Pasted image 20241013144426.png" >

- An overloaded operator function can be called directly: `operator+(data1, data2)`, `data1.operator+=(data2)`.
- The overloaded versions do not preserve operand-evaluation guarantees: **no short-circuit evaluation properties in `&&` and `||`**. These operands with built-in meanings should not be overloaded.
- Whether to define an operator a member or an ordinary nonmember function:
	- **`=`, `[]`, `()`, `->` must be members.**
	- Compound assignment operators and operators that change the state of the object ought to be members.
	- Symmetric operators (even with mixed types, e.g. adding `int` and `double`) should be defined as ordinary nonmember functions.

## Input and Output Operators

- Must be nonmember functions.
- Output operator: `ostream &operator<<(ostream&, const T&)`.
	- Minimal formatting: allow users to control the details.
	- Usually declared as friends.
- Input operator: `istream &operator>>(istream&, T&)`.
	- Rather than checking each read, we check once after reading all the data at once and before using those data: directly use the `istream` object as the condition.
	- Ensure the object should be in a usable state, not in an intermediate state.
	- **An input operator may set `failbit` to indicate errors.**

## Arithmetic and Relational Operators

- Operands should be references to `const`.
- The generated value should be distinct from either operand.
- **Classes that define both an arithmetic operator and the related compound assignment ordinarily ought to implement the arithmetic operator by using the compound assignment (not the other way around).**
- Definitions of `==` and `<` operators should be consistent if both exist: **if two objects are `!=`, then one object should be `<` the other.**

## Assignment Operators

- Assignment operators are not limited to copy- and move-assignment. For example, **`initializer_list` can be used to assign a vector**.

## Subscript Operator

- The subscript operator accepts a parameter of `std::size_t` and returns a reference to the element.
- Usually both `const` and non `const` versions are defined.

## Increment and Decrement Operators

- **Classes that define increment or decrement operators should define both the prefix and postfix versions.** These operators usually should be defined as members.
- Declaration and definition:
	- **Prefix operators: `T &operator++()`**. A reference to `*this` is returned.
	- **Postfix operators: `T operator++(int)`**. Save the current state in an object and return.
- They can be call explicitly: to call the postfix version, a value must be passed.

## Member Access Operators

- The dereference (`*`) and arrow (`->`) operators are often used in classes that represent iterators and in smart pointer classes.
	- Dereference: `T &operator*() const`.
	- Arrow: `T *operator->() const`.
- The arrow operator never loses its fundamental meaning of member access. **When we overload arrow, we change the object from which arrow fetches the specified member.** We cannot change the fact that arrow fetches a member. `ptr->mem` is equivalent to:
	- **Case 1: `ptr` is a built-in pointer: `(*ptr).mem`.**
	- **Case 2: `ptr` is a class object: `ptr.operator->()->mem`.** Specifically, the result of `point.operator->()` is used to fetch `mem`. **If it returns a pointer (address), then it falls back to case 1. Otherwise it must be an object (recursively apply case 2).**

## Function-Call Operator

- **Objects of classes that define the call operator are referred to as function objects.** Such objects “act like functions” because we can call them.
	- Function objects are most often used as arguments to generic algorithms.
- **Lambdas: the compiler translates lambda expressions into an unnamed function object** (the generated function-call operator is a `const` member function unless `mutable`).
	- Classes generated from a lambda expression have a deleted default constructor, deleted assignment operators, and a default destructor. Whether the class has a defaulted or deleted copy/move constructor depends in the usual ways on the types of the captured data members (same as in ordinary classes).
- The standard library defines a set of operator classes for creating function objects.
	- They can be used to override the default operator in algorithms.
	- The library function objects work for pointer comparation (**normally, two pointers cannot be compared**).

<img src="../attachments/Pasted image 20241013200919.png" >

- A callable object has a type specific to the class. However, two callable objects with different types may share the same **call signature: return type and argument types. A call signature corresponds to a function type**.
- **`funtion` types (defined in `functional`) can be used to store any callable object with a common call signature.** The template accepts the corresponding function type to create the function class type.
	- The `function` type overloads the call operator: it takes its own arguments and passes them to the stored callable object.
	- Assigning an overloaded function to a `function` object will cause ambiguity. We can use an intermediate function pointer or a lambda function to resolve it.

<img src="../attachments/Pasted image 20241013231613.png" >

## Overloading, Conversions and Operators

- Converting constructors and conversion operators define **class-type conversions**. Such conversions are also referred to as **user-defined conversions**.
- **Conversion operator: `operator T() const` converts the object to type `T`.** `T` can be of any function return type other than `void`.
	- In the declaration, `operator T` is the function name, and there is no return type.
	- **An implicit user-defined conversion can be preceded or followed by a standard (built-in) conversion.** For example, `si + 3.14` implicitly converts `si` to `int`, and it is converted to `double` using the built-in conversion.
	- Conversion operators can yield surprising results: `cin << n` performs left-shift.
- Declaring the conversion operator as `explicit` will suppress implicit conversions. **Only `static_cast` can be used to do the conversion with one exception: used as a condition (if/loop/condition expression, logical operands).**
	- Conversion to `bool` is usually intended for use in conditions. As a result, `operator bool` ordinarily should be defined as `explicit`.
- Ambiguity: multiple conversion paths may occur:
	- Mutual conversions (constructor in A and conversion in B). Only explicit function calls are allowed: `b.operator A()`, `A(b)`.
	- Built-in arithmetic type conversion is involved. **When two user-defined conversions are used, the rank of the standard conversion, if any, preceding or following the conversion function is used to select the best match.** If they have the same rank, ambiguity occurs.
	- Overloaded functions take parameters that differ by class types that define the same converting constructors. If two (or more) user-defined conversions provide a viable match, the conversions are considered equally good. **The rank of any standard conversions that might or might not be required is not considered.**

```cpp
// Example of built-in type conversion.
struct A {
	A(int = 0);               // usually a bad idea to have two
	A(double);                // conversions from arithmetic types
	operator int() const;     // usually a bad idea to have two
	operator double() const;  // conversions to arithmetic types
	// other members
};

void f2(long double);
A a;
f2(a);     // error ambiguous: f(A::operator int())
		   //               or f(A::operator double())
long lg;
A a2(lg);  // error ambiguous: A::A(int) or A::A(double)
short s = 42;
A a3(s);   // uses A::A(int)
           // promoting short to int is better than converting short to double
```

```cpp
// Example of overloaded functions.
struct C {
	C(int);
	// other members
};
struct D {
	D(int);
	// other members
};
struct E {
	E(double);
	// other members
};

void manip(const C&);
void manip(const D&);
manip(10); // error ambiguous: manip(C(10)) or manip(D(10))

void manip2(const C&);
void manip2(const E&);
manip2(10); // error ambiguous: two different user-defined conversions could be used
```

- All overloaded operators will be considered when we use the operator **(no order, no overloading)**:
	- Ordinary nonmember versions of that operator.
	- Overloaded versions of the operator defined by that class.
	- Built-in versions of the operator.

```cpp
class SmallInt {
	friend SmallInt operator+(const SmallInt&, const SmallInt&);
public:
	SmallInt(int = 0); // conversion from int
	operator int() const { return val; } // conversion to int
private:
	std::size_t val;
};

SmallInt s1, s2;
SmallInt s3 = s1 + s2; // uses overloaded operator+
int i = s3 + 0;        // error: ambiguous
```

