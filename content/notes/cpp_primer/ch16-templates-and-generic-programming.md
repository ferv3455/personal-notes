---
title: Chapter 16 Templates and Generic Programming
weight: 16
type: docs
---

## Defining a Template

- Function/Class template: `template` followed by a template parameter list.
    - Each type parameter must be preceded by the keyword `class` or `typename`.
- When we call a function template, the compiler uses the arguments of the call to deduce the template arguments. **A specific version of the function will only be instantiated with its definition during compile time.** As a result, headers for templates typically include definitions as well as declarations.
    - Type-related errors can be found during instantiation.
- **Templates may take nontype parameters that represent values rather than types.** The values can be supplied by the user or deduced by the compiler. They must be constant expressions.

```cpp
template<unsigned N, unsigned M>
int compare(const char (&p1)[N], const char (&p2)[M])
{
    return strcmp(p1, p2);
}
```

- `inline` or `constexpr` specifier follows the parameter list and precedes the return value.
- Template programs should try to minimize the number of requirements placed on the argument types (e.g. operators).
- Instantiating a class template requires a list of explicit template arguments. The name of a class template is not the name of a type.
- For class templates, members defined inside the class body are implicitly inline as well. Member functions defined outside the body starts with `template` followed by the template parameter list: `ret-type Blob<T>::member-name(param-list)`.
- **A member function of a class template is instantiated only if the program uses the member function.** That is, it is okay if some of its unused operations are invalid for the template arguments.
- **When we are inside the scope of a class template, the compiler treats references to the template itself as if we had supplied template arguments matching the template's own parameters.** The return type of a member function defined out of the body is outside the scope of the class.

```cpp
template <typename T>
BlobPtr<T> BlobPtr<T>::operator++(int)
{
    BlobPtr ret = *this; // save the current value
    ++*this; // advance one element; prefix ++ checks the increment
    return ret; // return the saved state
}
```

- Friendship:
    - Specific (an instance of the template is friend to an instance of `C`/`func`): `friend class C<T>`, `friend ret-type func<T> (...)`. **Forward declaration is necessary to befriend a specific instantiation: `template <typename T> class C`.**
    - General (all instances of the template are friends to class `CC`): `template <typename T> friend class CC`. No forward declaration is required (`CC` is a class not a template).
    - Befriending the type parameter: `friend T`.
- Type alias for a class template: `template<typename T> using twin = pair<T, int>` - `twin<T>`.
- Static members: for any given type `X`, there is one `Foo<X>::count` member shared by all objects. `template<...>` is required if defined outside the class template body.
- Template parameters have the scope until the end of the template. Declarations from the outer scope is hidden. **A name used as a template parameter may not be reused within the template.**
- Template parameter names may not be consistent across all declarations and definition.
- Type class members of a template are not visible before instantiation. **When we want to inform the compiler that a name represents a type, we must use the keyword `typename`: `typename T::value_type`.**
- We can supply default template arguments.
    - When we use a class template, we must always follow the template's name with brackets even if its empty.

```cpp
template <typename T, typename F = less<T>>
int compare(const T &v1, const T &v2, F f = F())  // F() is constexpr
{
    if (f(v1, v2)) return -1;
    if (f(v2, v1)) return 1;
    return 0;
}
```

- Members can be templates as well:
    - They are instantiated at use as well.
    - If they are defined outside the template class body, we need to provide both template parameter lists: class template followed by the member template.
- **The same instantiation may appear in multiple object files. Explicit instantiation can be used to avoid the overhead.** (This works just like `extern` variables.)
    - Declaring an instantiation as `extern` promises that there will be a non`extern` use of that instantiation elsewhere.
    - The extern declaration must appear before any code that uses that instantiation.
    - **Instantiation definitions instantiate all its members** (the complier doesn't know which members will be used).

```cpp
// instantiation declaration (multiple) and definition (only one)
extern template class Blob<string>;            // declaration
template int compare(const int&, const int&);  // definition
```

- Difference in deleter for `shared_ptr` and `unique_ptr`:
	- `shared_ptr` may support separate deleters for different pointers (the address can be changed dynamically), and they can be of any callable type. As they will be loaded dynamically anyway, they are maintained in a callable field, which doesn't need the deleter type at compile time.
	- `unique_ptr` only needs a single deleter. By designating the deleter time at run time, the deleter function can be loaded at compile time, which improves performance.

## Template Argument Deduction

- Conversions in type deduction:
	- Top-level `const`s in the parameters and arguments are ignored.
	- Reference parameters to `const` (low-level) can be passed references to non`const` objects.
	- **If the function parameter is not a reference type, then the normal pointer conversion will be applied to arguments of array or function type.** An array argument will be converted to a pointer to its first element. Similarly, a function argument will be converted to a pointer to the function’s type.
	- **Functions with reference parameters can accept arrays/functions as arguments. However, the same type cannot be used as return type.** Also, the size of the array is also part of the array type.
	- Other conversions (integer, arithmetic, etc.) are not performed.

```cpp
template <typename T> T fobj(T, T); // arguments are copied
template <typename T> T fref(const T&, const T&); // references

int a[10], b[42];
fobj(a, b); // calls f(int*, int*)
fref(a, b); // error: array types don’t match
```

- Function-template arguments can be explicitly given to **resolve non-deducible types (like return type)** or control template instantiation.
	- Explicit template arguments are matched from left to right.
	- Normal conversions apply for explicitly specified arguments.
- Sometimes we need to see the parameter list before deciding the return type. **A trailing return type should be used.**
	- In order to deduce more types, the **type transformation library** (defined in `type_traits` can be used in template metaprogramming.

```cpp
template <typename It>
auto fcn(It beg, It end) ->
	typename remove_reference<decltype(*beg)>::type
{
	// process the range
	return *beg; // return a reference to an element from the range
}
```

<img src="../attachments/Pasted image 20241120174331.png" >

- Initialize/assign a function pointer from a function template: use the pointer type to instantiate the template.
	- If a function takes a function pointer as an argument, and there are overloaded versions over the pointer type, ambiguity may arise. In this scenario, explicit arguments are needed. 

