---
title: Chapter 1 Arrays and Strings
weight: 1
type: docs
---

## Hash Tables

- A simple implementation: an **array of linked lists** and a **hash code function**:
    - Map the hash code `hash(key)` to array index (mod).
    - Store/Search in the linked list (collision).
- Worst case running time `O(N)`. Optimum lookup time `O(1)` if collisions are kept to minimum.
- Alternatively, the hash table can be implemented with a **balanced BST**, giving the lookup time of `O(log N)`.

## ArrayList & Resizable Arrays

- Resizes itself as needed while still providing `O(1)` access.
- Each doubling takes `O(N)` time, but the **amortized insertion runtime** is `O(1)`.

## StringBuilder & String Concatenation

- **Concatenating `n` strings with length `x`**:
    - `O(xn^2)` for ordinary methods: **creating a copy for each concatenation**.
    - StringBuilder: maintaining a resizable array of all strings, and copying back to string only when necessary.

## Exercises

1.  Is Unique: Determine if a string has all unique characters. What if without additional data structures?

- An ASCII string or a Unicode string?
- Shortcut for some straightforward cases (eg. long strings)?
- Minimizing space usage by substituting an integer for an boolean array?
- Without data structures: brute force or sorting?

2.  Check permutation: decide if one string is a permutation of the other.

- Case sensitive?
- Sorting or character counting?

3.  URLify: replace all spaces in a string with `'%20'` in place.

- Start from the end and work backwards (extra buffer)?
- Implementation: immutable or mutable data types?

4.  **Palindrome Permutation: check if a string is a permutation of a palindrome.**

- Defining features of a palindrome?
- Shortcuts in terms of string length?
- Character counts?
- Check odd number counts in the end or on the fly?
- **More elegant way of recording character counts with bit vectors? How to check no bits/exactly one bit of 1?** No overlap between `x` and `x-1`.

5.  **One Away: check if two strings are one edit (or zero edits) away.**

- Dynamic programming?
- The meanings of a single operation? What differences should the strings have to be one edit (replacement, insertion/removal) away?
- **Is it possible to merge all operations into one method?** The pointer of the longer string will always be moved. The movement of the shorter one is determined by character matching and string lengths.

6.  String Compression: basic string compression with the counts of repeated characters. For example, the string `aabcccccaaa` would become `a2b1c5a3`.

- Is the ordinary method efficient enough? `O(p+k^2)` (k is the number of sequences)?
- StringBuilder?
- What if the result is longer than the original string? Check afterwards or beforehand?

7.  Rotate Matrix: rotate an image by 90 degrees, which is represented by a NxN matrix of 4-byte pixels.

- In place solution? Operating by layers?

8.  **Zero Matrix: if an element in an MxN matrix is 0, its entire row and column are set to zero.**

- Turning the entire matrix into zeros?
- Keep a second matrix of zero locations? Or as simple as two arrays/bit vectors?
- **How to reduce the space to O(1)?** Use the first row and the first column as a replacement for these arrays. Keep records of original zeros in these areas beforehand.

9.  String Rotation: check if a string is a rotation of another with only one call to `isString`.

- How can s1 be found in s2? What about a repetition of s2?
