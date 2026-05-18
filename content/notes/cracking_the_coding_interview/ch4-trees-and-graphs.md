---
title: Chapter 4 Trees and Graphs
weight: 4
type: docs
---

## Types of Trees

- Binary tree: up to two children for each node.
- Binary search tree: all left descendents `<= n <` all right descendents for each node.
- Balanced tree: **balanced enough to ensure `O(log n)` times for `insert` and `find`**, eg. **red-black trees and AVL trees**.
- Complete binary tree: every level except the last is filled.
- Full binary trees: zero/two children for each node.
- Perfect binary trees: both full and complete, and the last level has the maximum number of nodes.

## Binary Tree Traversal

- You should be comfortable implementing in-order, pre-order, and post-order traversal.

## Binary Heaps (Min-Heaps and Max-Heaps)

- A min-heap is a **complete binary tree**.
- Insertion: start by inserting at the bottom, and then fix the tree by swapping it with its parent (bubble up).
- Min extraction: swap it with the last element in the heap, and then fix the tree by swapping it with one of its children (bubble down).

## Tries (Prefix Trees)

- Characters are stored at each node. **Each path down the tree may represent a word.**
- The \* nodes (null nodes) are often used to indicate complete words.
- A trie can check if a string is a valid prefix in `O(K)` time, the same runtime as a hash table will take.
- Many problems involving **lists of valid words** leverage a trie as an optimization.

## Graphs

- If there is a path between every pair of vertices, it is called a connected graph.
- A acyclic graph is one without cycles.
- Two ways to represent a graph: adjacency list (most common), adjacency matrices.

## Graph Search

- DFS: start at the root and explore each branch completely before moving on to the next branch. Preferred if **every node is to be visited**.
- BFS: start at the root and explore each neighbor before going on to any of their children. Preferred if **the shortest path is to be found**.
- **Any form of tree traversal is a form of DFS.**
- **Bidirectional Search**: used to find the shortest path between a source and destination node. It operates by running two simultaneous breath-first searches from each node.

## Exercises

1.  Route Between Nodes: given a directed graph, find out whether there is a route between two specific nodes.

- How to maintain states for each node? `Unvisited, Visited, Visiting`?
- Tradeoffs between BFS and DFS?

2.  Minimal Tree: given a sorted array with unique integers, create a binary search tree with minimal height.

- Recursively?

3.  List of Depths: given a binary tree, create a linked list of all the nodes at each depth.

- Is level-by-level traversal actually necessary (a modified version of BFS)?
- Simply pass depth values in recursion?
- Which solution is more efficient in terms of time and space?

4.  Check Balanced: check if a binary tree is balanced. The heights of two subtrees of any node in a balanced tree never differ by more than one.

- How can we efficiently calculate the height of each subtree?
- How to pass information of both balance and height in the return value in recursion?

5.  **Validate BST: check if a binary tree is a binary search tree.**

- Convert the tree into a sorted array? What if there are duplicate values?
- **Is an array necessary? Only neighboring elements in the array are compared.** Track only the last element we saw and compare it with the current one in recursion.
- **Leverage the definition of BST?** Pass down a range (min, max) for each subtree according to the root value, and check whether all values fall into the range.
- Base cases and null cases?

6.  Successor: find the "next" node (in-order successor) of a given node in a BST. Each node has a link to its parent.

- If there is a right subtree?
- What if no right subtree? Traverse upwards?
- What if already on the far right of the tree?

7.  **Build Order: given a list of projects and a list of dependencies (pairs of projects, where the second project is dependent on the first). Find a proper build order.**

- (Topological Sort) Where to start? What should we do with these nodes? What is the time and space efficiency of this algorithm?
- **What about DFS? Where should we place the projects in leaf nodes?** When we return from children nodes, we may add the root node to the front of the list, since every children must have already placed after it in the sequence.
- The time and space efficiency of DFS?
- **What if there are cycles? How to prevent from infinite loops?** Add `Partial` state to nodes in the process of DFS.

8.  **First Common Ancestor: find the first common ancestor of two nodes in a binary tree. Avoid storing additional nodes in a data structure.**

- **What if nodes have links to their parents?** It is equivalent of finding the intersection of two linked lists.
- **What about tracing `p`'s path upwards and check if any of the nodes cover `q`?** Only siblings on the way up need to be checked. This method has a better worse-case runtime of `O(n)`.
- **If nodes don't have links to parents, how to determine that the first common ancestor is found in recursion?** If both nodes are on different sides of a node, then it's the first common ancestor. Otherwise, process the subtree with both nodes.
- **Is it possible to check node locations and do the recursion simultaneously? Is returning `p`/`q`/`null`/the common ancestor enough?** The following two cases cannot be distinguished, so a flag indicating whether it's the result is necessary:
    - `p` is a child of `q`, vice versa.
    - `p` is in the tree and `q` is not, vice versa.

9.  **BST Sequences: a binary search tree was created by traversing through an array from left to right and inserting each element. Print all possible arrays that could have led to a given BST with distinct elements.**

- **Recursively: what does it mean to be an array that could lead to a BST?** Weave each array derived from left child with each array derived from right child, and then prepend each array with the parent node.
- **Recursively: how the weaving works?** Select either the first element of the first array or that of the second array as the new first element. Then weave the remaining arrays.

10. Check Subtree: T1 and T2 are two very large binary trees. Determine if T2 is a subtree of T1.

- **A simple approach: what could be inferred from the string representations of traversals of each tree? In which order?** The pre-order traversal of a tree is unique if the NULL nodes are included (eg. prefix notation of expressions). The pre-order traversal of a subtree is a substring of that of the parent.
- **An alternative approach: search through T1, and match the subtree if the node has the same value as the root of T2. Is the time complexity actually `O(nm)`?** A tighter bound should be `O(n+km)`, where `k` is the number of occurrences of T2's root in T1.
- Which solution is better, in terms of memory and (worst-case/average) runtime? What's the expected number of node checks for the alternative approach?

11. **Random Node: add the method `getRandomNode()` to BST. All nodes should be equally likely to be chosen.**

- What's the disadvantage of choosing randomly from an array with the same elements?
- How can we keep the probabilities evenly distributed across the options in traversals?
- **How to save expensive random number calls?** Subtract `LEFT_SIZE + 1` from the previous number instead of picking another.

12. **Paths with Sum: given a binary tree in which each node contains an integer value. Count the number of paths that sum to a given value. The path does not need to start or end at the root or a leaf, but it must go downwards.**

- **Brute force? What is the time complexity?** `O(N log N)`.
- **How to reuse some path-tracing in the brute force approach?** In a given path, for each position `y`, we are trying to find the start position `x`, so that the values in between sum to the target. If the running sum to `y` is tracked (sum of all elements on the path to `y`), we just need to find the number of `x`'s that the running sum to `x` equals to `runningSum(y) - target`.
- **How to save the running sums?** The number of occurrences of running sums on the current path can be stored in a hash table, in the form of `(runningSum, counts)`. Pay attention to values changes in the hash table in recursion and traceback.

