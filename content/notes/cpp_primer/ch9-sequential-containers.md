---
title: Chapter 9 Sequential Containers
weight: 9
type: docs
---

## Overview of the Sequential Containers

- Sequential container types to choose:

<img src="../attachments/Pasted image 20240924163043.png">

## Container Library Overview

- Container operations overview:

<img src="../attachments/Pasted image 20240924163154.png">
<img src="../attachments/Pasted image 20240924163113.png">

- Container constructors overview:

<img src="../attachments/Pasted image 20240924163211.png">

- A reverse iterator goes backward and inverts the meaning of iterator operations.
- Containers provide type aliases for elements. **`value_type`, `reference`, and `const_reference` are all correspondent to the real type, and are useful in generic programs.**
- `begin()` and `end()` are overloaded with a `const` (equivalent to `cbegin()`) and a non-`const` version. Using which version depends on whether the object is `const`.
- **Direct copying a container for initialization requires identical container types.** Otherwise, use a pair of range iterators (the element types don't need to be identical either, as long as they can convert).
- Arrays are declared to be of a fixed size: `array<int, 10>`. **In its default constructor, elements are default initialized. If it's list initialized, trailing elements will be value initialized (the same as built-in arrays).**
- Container assignment operations modify the entire container:

<img src="../attachments/Pasted image 20240924163246.png">

- Unlike built-in arrays, arrays can be copied or assigned (but not with a braced list of values).
- Assignment with `=` requires that operands have the same type. Otherwise, use `assign()` to support conversion.
- `swap()` does not copy elements (except for `array`) but swap internal data structures and pointers to the memory. It takes constant time. **As a result, iterators, references and pointer are not invalidated and still point to the previous data field.**
- Containers support `==/!=` and `>/>=/</<=` (except for unordered associative containers) between same types of containers. **They work similarly to `string` relationals and use the elements' relational operator `==` and `<`.**

## Sequential Container Operations

- Insertion operations for sequential containers:
    - When we call a `push` or `insert` member, we pass objects of the element type and those objects are copied into the container. **When we call an `emplace` member, we pass arguments to a constructor for the element type. The elements are constructed directly in place (avoid creating temporary objects).**

<img src="../attachments/Pasted image 20240924163320.png">

- Access operations for sequential containers:
    - **Access operations return references.** If the container is `const`, then return is a reference to `const`.
    - `at` ensures the index is valid.

<img src="../attachments/Pasted image 20240924163349.png">

- Remove operations for sequential containers:

<img src="../attachments/Pasted image 20240924163405.png">

- `forward_list` implements a singly linked list, so it has a different set of insert/remove operations:

<img src="../attachments/Pasted image 20240924163417.png">

- `resize(n)` or `resize(n, t)` resize containers by initializing additional elements (value initialization if no value is given) or discarding excess elements.
- **Operations that add or remove elements from a container can invalidate pointers, references or iterators.** They behave intuitively and match the implementation of data structures.
- `end()` is always invalidated when we add or remove elements in a `vector`, `string` or `deque` (remove any but the first element). Thus, `end()` should be called every time the container is resized.
- **Note: `deque` is actually implemented as a map of chunks of fixed size vector.**

<img src="../attachments/Pasted image 20240924163430.png">

## How a `vector` Grows

- `shrink_to_fit()`, `capacity()`, `reserve(n)` (at least `n`) can be used to manage and access the capacity of containers (`vector` or `string`, `shrink_to_fit()` also applies to `deque`).
    - `shrink_to_fit` is only a request; there is no guarantee.

## Additional `string` Operations

- Three additional ways to construct `string`s with slicing:

<img src="../attachments/Pasted image 20240924163506.png">

- `s.substr(pos = 0, n = npos)` returns a substring.
- Various operations to modify `string`s:
    - `append` is a shorthand way of calling `insert`, and `replace` is a shorthand way of calling `erase` and `insert`.
    - **Common interface:**
        - Specify the range of characters to remove: position and length, iterator range.
        - Specify the insertion point: index, iterator.
        - Specify the characters to add: another `string`, `char` pointer, brace-enclosed list of characters, character and count. The former two ways also support substrings (index, length).

<img src="../attachments/Pasted image 20240924163529.png">

- Six search functions with four overloaded versions each:
    - Returns `npos` if not found (`-1` in signed).

<img src="../attachments/Pasted image 20240924163603.png">

- `compare()` member function behaves similarly as `strcmp`. It accepts arguments in six formats:

<img src="../attachments/Pasted image 20240924163627.png">

- Conversions between `string`s and numbers:
    - **The first non-whitespace character in the `string` must be a character that can appear in a number. The function will read the `string` until it finds a character not in a number.**

<img src="../attachments/Pasted image 20240924163642.png">

## Container Adaptors

- `stack`, `queue`, and `priority_queue` are sequential container adaptors that make containers act like different types.
- **By default, `stack` and `queue` are implemented on a `deque`, and `priority_queue` is implemented on a `vector`.** It can be overridden with  the second type argument at creation.
- **`emplace()` also applies to these adaptors and can be used to insert objects constructed from an initializer list quickly.**

