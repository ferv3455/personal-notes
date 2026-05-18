---
title: Chapter 13 Copy Control
weight: 13
type: docs
---

## Copy, Assign, and Destroy

- Copy constructor: the first parameter is a (`const`) reference type.
	- A copy constructor is synthesized even if we define other (non-copy) constructors.
	- A synthesized copy constructor copies each non-`static` member. Members are copied by the copy constructors for their classes. **Members of array type are copied by copying each element.**
	- The compiler is permitted to skip the copy/move constructor during copy initialization.
- Copy initialization ordinarily uses the copy constructor, unless there is a move constructor.
	- Copy initialization also happens when an object is passed/returned as a nonreference type, or when we brace initialize the elements in an array or an aggregate class.
- `operator=` must be defined as member functions. Copy-assignment operators usually return a reference to their left-hand operand.
	- A synthesized copy-assignment operator assigns each non-`static` member with the copy-assignment operator.
	- **The copy-assignment operator is called at assignment, while the copy constructor is called at initialization.**
- **Destructors destroy members in reverse order from the order they were initialized.**
	- In a destructor, the function body is executed first, and the members are destroyed.
	- A synthesized destructor has an empty function body.
- Rule of thumb:
	- Decide first whether the class needs a destructor. If so, it surely needs a copy constructor and copy-assignment operator as well.
	- If a class needs a copy constructor, it almost surely needs a copy-assignment operator.
- `= default` can be used to generate synthesized versions of copy-control members. The function will be inline if `= default` is added inside the class definition, otherwise not inline.
- **`= delete` can be used to define copy constructor and copy-assignment operators as deleted functions, which prevents copies.** It must appear on the first declaration of a deleted function.
	- `= delete` can be specified on any function.
	- The destructor should not be deleted member.
	- **In essence, the copy-control members are synthesized as deleted when it is impossible to copy, assign, or destroy a member of the class.** To be specific, if the destructor of a member is deleted, all control members (including the default constructor) are defined as deleted.
- It is legal to declare but not define a member function. We can declaring them as `private` to prevent using these functions.

## Copy Control and Resource Management

- **Guard against self-assignment at any time**: copy first, and then free the old object.
- Manage member resources of pointers: use a reference count.

## Swap

- `swap` is important for classes that we plan to use with reordering algorithms (sort, etc.).
- We can override the default behavior of `swap` by defining a friend function `swap(T &lhs, T &rhs)` that operates on our class.
- **`swap` functions should call `swap` instead of `std::swap`.** `std::swap` are used only as a backup for classes without `swap`. Proper way to implement this:

```cpp
void swap(Foo &lhs, Foo &rhs)
{
    using std::swap;
    swap(lhs.h, rhs.h); // uses the HasPtr version of swap
    // swap other members of type Foo
}
```

- Classes that define `swap` often use it to define `operator=`. The parameter is a copy, so it is safe to swap. It is automatically exception safe.

## Classes That Manage Dynamic Memory

- **Move constructors defined in several library classes make possible copying objects without needing to allocate dynamic memory every time.** For strings, the pointer to the array of characters in the memory is copied. To users, this copy operation transfers the ownership of the string to the new object, which works similarly as "move".
	- This is useful when copies are immediately destroyed after it is copied.
	- Also, for classes with resources that cannot be shared, move should be allowed.
- **`std::move` (in `utility`) should be used to signal that the move constructor is needed.** Use it as an argument in object initialization will call the move constructor.

## Moving Objects

- **An rvalue reference is declared with `&&`.** They may be bound only to an object that is about to be destroyed, which works well for moving.
- Lvalues persist, while rvalues are ephemeral:
	- **Expressions that yield lvalues**: functions that return lvalue references, along with the assignment, subscript, dereference, and prefix increment/decrement operators.
	- **Expressions that yield rvalues**: functions that return a nonreference type, along with the arithmetic, relational, bitwise, and postfix increment/decrement operators.
- **Binding references to values:**
	- **lvalue: we can only bind an lvalue reference (or to `const`) to it**
	- **rvalue: we can bind an lvalue reference to `const` or an rvalue reference to it**

```cpp
int i = 42;

int &r = i;               // ok: r refers to i
int &r2 = i * 42;         // error: i * 42 is an rvalue
const int &r3 = i * 42;   // ok: we can bind a reference to const to an rvalue

int &&rr = i;             // error: cannot bind an rvalue reference to an lvalue
int &&rr2 = i * 42;       // ok: bind rr2 to the result of the multiplication
```

- `std::move` returns a rvalue reference to its given object. **After the call, we cannot use the value of the moved-from object.**
- **Move constructors has an rvalue reference as its parameter.** After moving, the original object must no longer point to moved resources. The rvalue reference can be edited.
- `noexcept` can be used to promise that a function does not throw any exceptions. It appears between the parameter list and `:`. It should be added in both declaration and definition.
- **Move constructors should be indicated as `noexcept`**: library functions like `push_back` can only use a move constructor if it won't throw an exception (or it will not be able to recover). The same applies to move-assignment operators.
- **Moved-from objects must be left destructible.**
- Synthesizing move constructors and move-assignment operators:
    - The compiler synthesizes the move constructor and move assignment only if a class does not define any of its own copy-control members and only if all the non-`static` data members can be moved constructed and move assigned, respectively.
    - The corresponding copy operation will be used in place of move if it's not defined.
- A move operation is never implicitly deleted. **If we ask the compiler to generate a `default` move operation and the compiler is unable to do so, it will be deleted.**
- **Classes that define a move constructor or move-assignment operator must also define their own copy operations.** Otherwise, those members are deleted by default.
- Rvalues are moved, and lvalues are copied (ordinary function matching). However, if there is no move constructor, rvalues will be copied **(`T&&` can be converted to a `const T&`)**.

```cpp
StrVec v1, v2;
v1 = v2;                   // v2 is an lvalue; copy assignment
StrVec getVec(istream &);  // getVec returns an rvalue
v2 = getVec(cin);          // getVec(cin) is an rvalue; move assignment
```

- If the parameter of the assignment operator is defined as the ordinary type `T`, then it is both the move- and copy-assignment operator:
    - Copy-assignment: the argument itself is a copy, and it will be swapped with the original object.
    - Move-assignment: the moved-from object will be moved to the parameter object, which is further swapped.
- Ordinarily, dereferencing an iterator returns a lvalue reference. **`make_move_iterator` transforms it to a move iterator, which yields rvalue references. They can be passed into copy algorithms to use move operations instead.**
- Move-enabled member functions typically use the same parameter pattern as others: `const T&` and `T&&`.

```cpp
void StrVec::push_back(const string& s)
{
    chk_n_alloc();
    alloc.construct(first_free++, s);
}
void StrVec::push_back(string &&s)
{
    chk_n_alloc();
    alloc.construct(first_free++, std::move(s));
}

StrVec vec;             // empty StrVec
string s = "some string or another";
vec.push_back(s);       // calls push_back(const string&)
vec.push_back("done");  // calls push_back(string&&)
```

- **The lvalue/rvalue property of `this` is indicated similarly as `const`: place a reference qualifier `&` (lvalue only) or `&&` (rvalue only) after the parameter list (after `const`).** It must appear in both the declaration and the definition.
    - Member functions can be overloaded based on its reference qualifier. **None or all of overloaded functions must be provided with a reference qualifier.**

## An Example of Constructors and Assignment Operators

```cpp
class A {
public:
	A(int i) : m(i) {}
	A(const A &a) : m(a.m) {}
	A(A &&a) noexcept : m(std::move(a.m)) {}
	A &operator=(const A &a) {}
	A &operator=(A &&a) noexcept {}
	int m;
}

A func() {
    return A(1);
}

int main() {
    std::cout << 1 << std::endl;
    A a = func();
    std::cout << 2 << std::endl;
    A a1(a);
    std::cout << 3 << std::endl;
    a1 = a;
    std::cout << 4 << std::endl;
    a1 = func();
    return 0;
}
```

Output:

```text
1
Constructor
Move constructor
Move constructor
2
Copy constructor
3
Copy assignment
4
Constructor
Move constructor
Move assignment
```

- The code above should be run with a `-fno-elide-constructors` flag.
- `return` involves a move/copy operation. To return an object, it is copied/moved to a temporary object.
- If the move operations are removed, copy constructors and operations will be used instead.
- Defining move constructors/operations will implicitly delete the copy versions.

