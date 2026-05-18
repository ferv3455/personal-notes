---
title: Chapter 11 Associative Containers
weight: 11
type: docs
---

## Using an Associative Container

- An overview of associative containers:
    - Range `for` yields `pair`s of keys and corresponding values.

<img src="../attachments/Pasted image 20240926173218.png">

## Overview of the Associative Containers

- Associative containers support general container operations except for position-specific operations (`push_front`, `back`, etc.).
- Initializing a map or a set:

```cpp
map<string, size_t> word_count;
map<string, string> authors = { {"Joyce", "James"}, {"Austen", "Jane"} };
set<string> exclude = {"the", "but", "and", "or", "an", "a", "A"};
set<string> iset(ivec.cbegin(), iven.cend());
multiset<string> miset(ivec.cbegin(), iven.cend());
```

- **Key type requirements: comparable by `<` (ordered containers) or hashable and comparable by `==` (unordered containers).**
    - Strict weak ordering: if neither key is less than the other, then they are equivalent.
    - **To designate the comparison function, use a function pointer as the type in the type definition, and pass the function (or a pointer) as the constructor argument: `muliset<T, decltype(cmp)*> s(cmp)`.**
- `pair`:
    - A `pair` can be initialized in these ways: `p` (value initialized), `p(v1, v2)`, `p{v1, v2}`, `p = {v1, v2}`, `make_pair(v1, v2)` (types are inferred).
    - The data members of `pair` (defined in `utility`) are `public`: `first` and `second` can be accessed.
    - `pair`s can be compared (relational operators).

## Operations on Associative Containers

- `key_type`, `mapped_type`, `value_type` are defined in each container. `value_type` is `key_type` (`set`) or `pair` (`map`).
- Dereferencing iterators will yield a `value_type` reference. The key is `const`. So for `set` types, the iterators are `const`.
- **When an iterator is used to traverse an ordered container, the keys are in ascending order.**
- `inserter` can be used to make the associative container the destination of an algorithm.
- Insertion operations (**for ordered associative containers, the location to insert will be searched to maintain the ascending order**):
    - For containers with unique keys, a `pair` will be returned, containing an iterator to the element and a `bool` indicating whether the element is inserted or duplicate.
    - For containers that allow multiple keys, an iterator will be returned.

<img src="../attachments/Pasted image 20240926173302.png">

- `erase` takes arguments of a key, an iterator, or an iterator range. It returns the number of elements removed.
- `map` and `unordered_map` provide the subscript operator and the `at` function that return an lvalue. **If `m[key]` does not exist, a new element is created and value initialized, so the subscript operator should not be called on a `const` map. However, `m.at(key)` will not create a new one and will throw an `out_of_range` exception, so it can used on `const` maps.**
- Ways of accessing/counting elements:
    - **Elements of the same key will be adjacent to each other in a `multimap`/`multiset`.** Thus, we can use `find` and `count`, or `lower_bound` and `upper_bound`, or `equal_range` (a pair of iterators) to locate the range of elements.

<img src="../attachments/Pasted image 20240926173347.png">

## The Unordered Containers

- Unordered container management operations (not used normally):

<img src="../attachments/Pasted image 20240926173412.png">

- **A `hash<key_type>` object will be used to generate the hash code. The hash function for ordinary types of keys have been implemented.** It needs to be supplied if we want to use it on customized class types. A simple strategy is to wrap an implemented hash function.
    - The constructor accepts arguments of `(bucket_size, hasher, eqOp)`.
    - If the `==` operator is defined in class, only the hash function needs to be overridden.

```cpp
size_t hasher(const Sales_data &sd) {
    return hash<string>()(sd.isbn());
}
bool eqOp(const Sales_data &lhs, const Sales_data &rhs) {
    return lhs.isbn() == rhs.isbn();
}

using SD_multiset = unordered_multiset<Sales_data, decltype(hasher)*, decltype(eqOp)*>;
SD_multiset bookstore(42, hasher, eqOp);
```

