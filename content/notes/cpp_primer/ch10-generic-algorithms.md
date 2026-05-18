---
title: Chapter 10 Generic Algorithms
weight: 10
type: docs
---

## Overview

- Headers: `algorithm`, `numeric`.
- In general, algorithms operate by traversing a range of element bounded by two iterators. **But since pointers act like iterators, passing in and returning pointers is also supported.**
- Algorithms depend on element-type operations (we can supply our own operation - predicate for most algorithms).

## A First Look at the Algorithms

- The list of algorithms: [Algorithms library - cppreference.com](https://en.cppreference.com/w/cpp/algorithm)
- Read-only algorithms: `find`, `find_if`, `count`, `accumulate`, `equal`, `for_each`.
    - `accumulate`: all elements should be convertible to the type of the third argument.
    - `equal`: the second sequence should not be shorter than the first one.
    - `for_each` takes a callable object and calls that object on each element in the input range.
- Algorithms that write: `fill`, `fill_n`, `copy`, `replace`, `replace_copy`, `transform`, `remove`, `remove_if`, `remove_copy_if`.
    - `fill_n` has no boundary checks. **It is wiser to use `back_inserter`, which takes a reference to a container and returns an safe insert iterator. If it goes beyond the boundary, new elements will be inserted at the end.**
    - `copy` returns the incremented destination iterator.
    - `transform` replaces each value with a new value calculated from the given predicate.
- Algorithms that reorder elements: `sort`, `stable_sort`, `unique`.
    - `sort` uses the element type's `<` operator.
    - `unique` should be used after `sort`. It places slots for duplicate elements at the end of the container and returns an iterator to the first of them (the elements at the end are not duplicate elements, just slots with unknown values).

## Customizing Operations

- We can pass any kind of callable object (with a call operator`()`) to an algorithm.
- **Lambda expressions are a kind of callables: `[capture list] (parameter list) -> return type {function body}`.**
    - **Capture list is a list of used local variables defined in the enclosing function.** These variables can be used inside the lambda. **It is used for local non-`static` variables only; local `static`s and global variables can be used directly.**
    - The parameter list (no parameters) and the return type (inferred from the code) can be omitted.
    - The parameter list doesn't support default arguments.
- When we pass a lambda to a function, we are defining both a new type and an object of that type. The captured variables are contained in the lambda object as data members.
- Possible lambda capture lists:
    - **The value of a captured variable is copied when the lambda is created, not when it is called.**
    - Variables can be captured by reference: `&v`. They must exist when the lambda is executed (especially when using a lambda as the return value).
    - **`&` or `=` direct the compiler to infer the capture list from the lambda body (by reference or by value). They can be mixed with explicit captures (put implicit ones first).**
    - Variables captured by value can also be changed if the parameter list (cannot be omitted) is followed by `mutable`. The variables are still evaluated by their value at definition.

<img src="../attachments/Pasted image 20240925150958.png">

```cpp
size_t v1 = 42;  // local variable
// f can change the value of the variables it captures
auto f = [v1] () mutable { return ++v1; };
v1 = 0;
auto j = f();    // j is 43
```

- **The return type needs to be specified if the body contains more than a return statement.**
- **`bind` (defined in `functional`) is a general-purpose function adaptor that generates a new callable with the given parameter list: `auto newCallable = bind(callable, arg_list)`.**
    - The old callable will be called with the given `arg_list` (arguments in its order).
    - `arg_list` can contain parameters from the new callable. Placeholders `_n` (from the namespace `std::placeholder`, starting from 1) need to be added in `arg_list` to indicate which parameter of the new callable will be used as this argument.
    - `bind` copies the non-placeholder parameters into the generated object by default (for placeholder parameters, they will be passed to the original callable directly). To use references, `ref()`/`cref()` (defined in `functional`) can be used.

## Revisiting Iterators

- Insert iterators (`back_inserter(c)`, `front_inserter(c)`, `inserter(c, iter)`) always insert elements at a fixed position (always at the back/end/ahead of an element).

```cpp
list<int> 1st = {1,2,3,4};
list<int> lst2, lst3; // empty lists
// after copy completes, 1st2 contains 4 3 2 1
copy(1st.cbegin(), lst.cend(), front_inserter(lst2));
// after copy completes, 1st3 contains 1 2 3 4
copy(1st.cbegin(), lst.cend(), inserter(lst3, lst3.begin()));
```

- `istream` and `ostream` iterators:
	- They can be used together with algorithms efficiently.
	- **The default initializer for `istream_iterator` is EOF.**
	- `istream_iterator` may use lazy evaluation: read may be delayed until we use it.
	- **`*` and `++` operators do nothing on an `ostream_iterator`.**

```cpp
istream_iterator<int> in_iter(cin), eof;  // read ints from cin
vector<int> vec(in_iter, eof);            // construct vec from an iterator range
// OR
istream_iterator<int> in_iter(cin), eof;
while (in_iter != eof)
	vec.push_back(*in_iter++);
```

```cpp
for (auto e : vec)
	out_iter = e;   // the assignment writes this element to cout
cout << endl;
// OR
copy(vec.begin(), vec.end(), out_iter);
cout << endl;
```

- `sort(vec.rbegin(), vec.rend())` sort the vector in reverse order.
- **The `base` member of a `reverse_iterator` returns the corresponding ordinary iterator:**

<img src="../attachments/Pasted image 20240925171502.png">

## Structure of Generic Algorithms

- The five iterator categories defines different sets of operations provided. This standard specifies the minimum category for each iterator parameter of algorithms.

<img src="../attachments/Pasted image 20240925204415.png">

- Algorithm parameter patterns:
	- `alg(beg, end, other args)`.
	- `alg(beg, end, dest, other args)`. **`dest` is where to write its output, and it is always assumed to be safe (insert iterator or `ostream_iterator` should be given).**
	- `alg(beg, end, beg2, other args)`, `alg(beg, end, beg2, end2, other args)`. `beg2` and `end2` denote a second input range. **If `end2` is not given, the second sequence is assumed to be not shorter than the first one.**

## Container-Specific Algorithms

- `list` and `forward_list` define several algorithms as members (specific to data structure):

<img src="../attachments/Pasted image 20240925212938.png">

<img src="../attachments/Pasted image 20240925213105.png">

