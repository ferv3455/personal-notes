---
title: Chapter 2 Linked Lists
weight: 2
type: docs
---

## Creating a Linked List

- If linked lists are implemented through a single `Node` class, we should focus on the management of the head node, for some objects might still be pointing to the old head after changes.
- A `LinkedList` class may be used to wrap the `Node` class.

## Deleting a Node from a Singly Linked List

- Find the `prev` and set `prev.next` to `n.next`.
- Doubly linked lists: update `n.next.prev` to `n.prev`.
- Important things to remember: memory deallocation, **null pointer**, **head or tail pointer**.

## The "Runner" Technique

- The "runner" (**second pointer**) is used in many linked list problems.
- Example: to weave the first and second half of a linked list together, two pointers can be used to locate the split location, one moving twice as quick as the other.

## Recursive Problems

- Recursive algorithms take at least `O(n)` space, where `n` is the depth of the recursive call.
- All recursive algorithms **can be implemented iteratively**, although they may be much more complex.

## Exercises

1. Remove Dups: remove duplicates from an unsorted linked list. What if a temporary buffer is not allowed?

- How to track duplicate elements?

2. **Return Kth to Last: find the kth to last element of a singly linked list.**

- If the linked list size is known: a straightforward solution?
- **Recursively: how to know how many nodes are left after the current one?** Use the return value as a counter of nodes after this. 
- Iteratively: two pointers?

3. **Delete Middle Node: delete a node in the middle of a singly linked list, given only access to that node (without head).** 

- **How to avoid change the `next` pointer in the previous node?** Copy the data from the next node over to the current one, so that the `next` pointer in the previous node still points to this one, but it represents the next one to be deleted.
- What if the node is the last node in the linked list?

4. Partition: partition a linked list around a value `x`, such that all nodes less than `x` come before all nodes greater than or equal to `x`.

- Quick sort?
- Instead of in-place element shifts, what about creating two different linked lists?
- What about a single linked list growing both forward and backward?

5. Sum Lists: you have two numbers represented by a linked list, where each node contains a single digit. The digits are stored in reverse order. Add the two numbers and return the sum as a linked list. What about stored in forward order?

- How addition works exactly?
- What if a linked list is shorter than another, especially when stored in forward order?
- Recursive solutions to linked lists stored in forward order?

6. **Palindrome: check if a linked list is a palindrome.**

- What exactly is a palindrome? What's it like in reverse order?
- **Iteratively: how to save and read the reverse of the first half? If the size is unknown, how to determine where the first half stops?** Push the first half of the elements onto a stack using the fast runner/slow runner technique. 
- If we have acquired the linked list size beforehand, how to implement recursively?

7. Intersection: determine if two linked lists intersect, and return the intersecting node. 

- Intuitively, intersecting linked lists have the same last node. How to derive the intersection from that?
- Can we turn differently-sized linked lists into ones of equal lengths?
- Consider both linked lists with shapes of AC and BC. Is it possible to find common parts in ACBC and BCAC, which are of equal lengths?

8. **Loop Detection: given a circular linked list (a corrupt linked list with a loop), return the node at the beginning of the loop.**

- Use the fast runner/slow runner technique. Will two cars running on a circuit at difference speeds eventually meet? Pay attention to non-circular cases.
- **How far is the fast runner behind when the slow runner entered the loop? Where will they meet eventually?** If it takes the slow runner `k` steps to enter the loop, the fast runner should be `LOOP_SIZE - k % LOOP_SIZE` steps behind. They will meet after `LOOP_SIZE - k % LOOP_SIZE` steps when they are `k % LOOP_SIZE` steps before the head of the loop.
- **How to find the head of the loop?** Their meeting point should be `k` steps away from the loop head. A runner starting at the list head would also be `k` steps away.

