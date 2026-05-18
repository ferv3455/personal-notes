---
title: Chapter 3 Stacks and Queues
weight: 3
type: docs
---

## Implementing a Stack

- A stack uses LIFO (last-in first-out) ordering.
- Constant-time adds and removes, but no access to the `i`th item.
- A stack offers an intuitive way to push temporary data and remove them to backtrack in **recursive algorithms**.

## Implementing a Queue

- A queue uses FIFO (first-in first-out) ordering.
- Double check **updating the first and last nodes in a queue**.
- One place where queues are often used is in **breadth-first search** or in **implementing a cache**.

## Exercises

1.  Three in One: use a single array to implement three stacks.

- Fixed division: equal/assigned limited space for each individual stack?
- Flexible division: how to maintain the locations of each stack if it can be circular (wrapped around to the beginning)?

2. Stack Min: design a stack with a function `min` to return the minimum element in `O(1)` time.

- Search every time when the minimum value is popped?
- What about keeping track of the minimum at each state?
- Is it possible to save the space of same minimum values?

3. Stack of Plates: implement `SetOfStacks` composed of several stacks, which creates a new stack once the previous one exceeds capacity. Implement a function `popAt(int index)` which performs a pop operation on a specific sub-stack.

- If an element is popped from a middle stack, is a "rollover" necessary?

4. Queue via Stacks: implement a queue with two stacks.

- Use the second stack for order reversing?
- A "lazy" approach?

5. **Sort Stack: sort a stack such that the smallest items are on the top. An additional temporary stack is available.**

- **Maintain a sorted stack, and insert elements one by one?** Let `s2` be the sorted stack. Pop one element from `s1` each step, and compare it to the top of `s2`. Keep moving the top of `s2` over to `s1` until it's no greater than the object element and push it into `s2`.
- **What about merge sort and quicksort? How many additional stacks are required?** Two extra stacks. 

6. Animal Shelter: an animal shelter, which holds only dogs and cats, operates on a strictly FIFO basis. People must select one of the following options: the oldest of all, the oldest cat or the oldest dog. Create the data structures to maintain this system.

- How many queues are allowed?

