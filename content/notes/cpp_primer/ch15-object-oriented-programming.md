---
title: Chapter 15 Object-Oriented Programming
weight: 15
type: docs
---

## Defining Base and Derived Classes

- `virtual` member functions:
	- Virtual member functions are expected to be overridden by derived classes. **The call will be dynamically bound at run time depending on the type of the object.**
	- Virtual member functions are implicitly virtual in derived classes as well.
	- Member functions not declared as `virtual` are resolved at run time.
- Class derivation list: a comma-separated list of base class names each preceded by an optional access specifier (default to public for derived `struct` or private for derived `class`).
- A derived object contains multiple parts: a subobject containing the (non-`static`) members defined in the derived class itself, plus subobjects corresponding to each base class from which the derived class inherits. **This makes derived-to-base conversion possible (some limitations apply): pointers or references of a derived type can be supplied to those of the base type** (dynamic type and static type).
	- Base pointers/references cannot be implicitly converted to derived ones. `dynamic_cast` (request to check members at run time) or `static_cast` (assume the conversion is safe) can be used.
	- Objects cannot be converted automatically. However, it is often possible to convert with the copy constructor of the base class.
- Derived-class constructors use a base-class constructor in its initializer list. Otherwise, the base part will be default initialized.
- If a base class defines a `static` member, there is only one such member defined for the entire hierarchy.
- The declaration of a derived class does not contain its derivation list.
- **The base class must be defined before using it as a base class.**
- **Following the class name directly with `final` will prevent inheritance.**

## Virtual Functions

- We must define every virtual function regardless of whether it is used.
- Run-time resolution and derived-to-base conversion reflect the idea of polymorphism.
- **If `D` is derived from `B`, then a base class virtual can return a `B*` and the version in the derived can return a `D*` (if D to B derived-to-base conversion is accessible).** This is the only exception the types of virtual member functions do not match.
- `override` can be added to explicitly override a virtual. `final` can be added to prevent a function from being overridden. They appear after `const` and the trailing return.
- **The default arguments for virtual functions will be those defined in the pointer/reference type.**
- We can use a scope operator to prevent dynamic binding: `p->Base::func()`. It is often used in derived-class function definitions to call the base-class version.

## Abstract Base Classes

- A pure virtual function makes the class an abstract base class.
	- `= 0` may appear only on the declaration.
	- **A definition outside the class can be provided for a pure virtual function.** Derived classes may call this function in their implementations of the virtual function.


## Access Control and Inheritance

- `protected` members are accessible to friends of classes derived from this class.
- **A derived class member or friend may access the protected members of the base class only through a derived object, not through a base-class object.**
- The purpose of the derivation access specifier is to control the access to base members from the users of the derived class, not from the derived class itself.
- **For any given point in your code, if a public member of the base class would be accessible, then the derived-to-base conversion is also accessible, and not otherwise.**
- Friendship is also not inherited. However, friends of the base class have access to the embedded base object in derived objects.
- **`using Base::m` can be used to change the access level of a specific inherited name.** It can be only used for names the derived class is permitted to access.

## Class Scope under Inheritance

- The scope of a derived class is nested inside the scope of its base classes.
- The static type of an object/reference/pointer determines which members are visible. **So, name lookup is possible and will happen at compile time.** Even if a base class pointer points to a derived object, a member function specific to the derived class cannot be called.
- A derived-class member (data or function) with the same name as a member of the base class hides direct use of the base-class member. This will not cause an error. In this way, the hidden base-class member can be accessed with the scope operator.
- Like ordinary functions, functions defined in a derived class do not overload functions from the base class, even if they have different parameter lists. **This is because name lookup happens before type checking.**
- **If a derived class wants to make all the overloaded versions of the base class available through its type, it must override all of them or none of them. In this situation, `using` declaration (only the name, no parameter list) can be used to add all the overloaded instances of that function.**

## Constructors and Copy Control

- **A base class almost always needs a virtual destructor. However, the compiler will not synthesize move operations for the class in this way.**
- All that matters for synthesized copy-control member functions is that the base-class member is accessible and not deleted. **The synthesized construction/destruction process will automatically go up to the root of the hierarchy.**
- Derived copy-control members may be deleted if:
    - The corresponding member in the base class is deleted or inaccessible.
    - If the base class has an inaccessible destructor, the default and copy constructors in derived classes are deleted.
    - Move operations in the derived will be deleted if we use `= default` but it is impossible to synthesize one (the base class part cannot be moved).
- **In order to support copy/move operations in derived classes, base classes usually define all copy-control members explicitly (destructor -> move -> copy).**

```cpp
class Quote {
public:
    Quote() = default;              // memberwise default initialize
    Quote(const Quote&) = default;  // memberwise copy
    Quote(Quote&&) = default;       // memberwise copy
    Quote& operator=(const Quote&) = default;  // copy assign
    Quote& operator=(Quote&&) = default;       // move assign
    virtual ~Quote() = default;
    // other members as before
};
```

- Suggested definition of derived copy-control members:
    - Copy/Move constructor: the parameter can be directly used as the argument for the base class constructor in the initializer list (dynamic type).
    - Assignment operator: call the base operator with scope: `Base::operator=(rhs)`, and then assign the derived members.
    - Destructor: only need to destroy derived resources.
- **While an object is being constructed it is treated as if it has the same class as the constructor; calls to virtual functions will be bound as if the object has the same type as the constructor itself.** Similarly, for destructors. This is because the object is incomplete at that time, without derived members. Executing virtual member functions defined in derived classes may lead to errors.
- **For user-defined constructors, the derived class may inherit them from its direct base: `using Base::Base`. When applied to a constructor, `using` causes the compiler to generate code of a derived constructor for each constructor in the base.**
    - Derived members will be default initialized.
    - The access level of inherited constructors are the same as in the base class.
    - `explicit` or `constexpr` cannot be specified (remains the same as in the base class).
    - If a base-class constructor has default arguments, they are not inherited. **The derived class gets multiple inherited constructors in which each parameter with a default argument is successively omitted.**
    - The derived class can define some constructors with the same parameters as constructors in the base to hide them.

## Containers and Inheritance

- When a container with objects related by inheritance is needed, we typically define it to hold (smart) pointers to the base class. However, comparison operation is probably necessary to support storing pointers in ordered containers.
- Smart pointers can also perform derived-to-base conversions.
- **In order to allow users to operate the container with an object rather than a created pointer, two versions are usually defined: copying and moving the given object.** However, the class needs to allocate differently for different types. **In this situation, the base and derived functions need a virtual member `clone` that allocates a copy of itself.**

```cpp
void add_item(const Quote& sale);  // copy the given object
{ items.insert(std::shared_ptr<Quote>(sale.clone())); }
void add_item(Quote&& sale);       // move the given object
{ items.insert(std::shared_ptr<Quote>(std::move(sale).clone())); }
```

