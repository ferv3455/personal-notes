---
title: Chapter 8 Recursion and Dynamic Programming
weight: 8
type: docs
---

## How to Approach

- Bottom-up: build the solution for one case off the previous case (or multiple previous cases).
- Top-down: divide the problem into subproblems.
- Half-and-half: divide the dataset in half, eg. binary search, merge sort.

## Recursive vs. Iterative Solutions

- Recursive algorithms use at least `O(n)` memory.
- **All recursive algorithms can be implemented iteratively and more efficiently.** Pay attention to the tradeoffs, though.

## Dynamic Programming & Memoization

- Nature: find overlapping subproblems and cache those results for future recursive calls/iterations.

## Exercises

1.  Triple Step: a child is running up a staircase with `n` steps and can hop either 1 step, 2 steps, or 3 steps at a time. How many possible ways the child can run up the stairs?

- Recursion? Memoization?

2.  Robot in a Grid: imagine a robot sitting on the upper left corner of a grid with `r` rows and `c` columns. It can only move right or down. There are certain cells that are "off limits". Find the path to the bottom right.

- Start recursion from upper left or bottom right?
- When to add locations into the path?
- Dynamic programming to save unreachable locations?

3.  **Magic Index: a magic index in an array `A[1...n-1]` is defined to be an index such that `A[i] = i`. Given a sorted array of distinct integers, find a magic index. What if the values are not distinct?**

- **What does it mean that the increasing numbers are distinct?** The difference between neighboring numbers is at least 1, and the indices increases at step 1. If there is a magic index, all numbers to its left are smaller than their indices, and all numbers to its right are greater (if not equal).
- **If the elements are not distinct, and we see that `A[mid] < mid`, could the magic index be anywhere on the left?** Notice that, for a magic index `i` on the left, `i = A[i] <= A[mid]`. Similar conclusions could be reached on the right. This determines the search range: `[low, min(mid - 1, A[mid])]` on the left, and `[max(mid + 1, A[mid]), hi]` on the right.

4.  Power Set: return all subsets of a set.

- How can we use `P(n)` to create `P(n + 1)`?
- What is the relationship between a number in `[0,2^n)` and a specific subset?

5.  **Recursive Multiply: multiply two positive integers without using `*` or `/`. You can use addition, subtraction, and bit shifting.**

- **Half-and-half? Doubling?** Assume `a < 0`. `a * b = (a/2) * b * 2`. If `a` is odd, `a * b = (a/2) * b * 2 + b`. Base case: `a = 0 or 1`.
- Is memoization necessary?

6.  Towers of Hanoi: move the disks from the first tower to the last using stacks.

- Recursive?

7.  **Permutations without Dups: compute all permutations of a string of unique characters.**

- How to generate permutations of the first `n` characters from the permutations of the first `n-1` characters?
- How to generate permutations of all `n` characters from the permutations of all `n-1` substrings?
- **What about passing the prefix down the recursion instead of passing up permutations?** Only a single list is necessary, and base cases will directly save results in it.

8.  **Permutations with Duplicates: compute all permutations of a string whose characters are not necessarily unique.**

- **Count the numbers of occurrences of each character at first?** Pass a `HashMap` to record remaining characters. Pay attention to backtracking.

9.  Parens: print all valid combinations of `n` pairs of parentheses.

- When can we use a left paren, and when can we use a right paren?

10. Paint Fill: given a screen, a point, and a new color, fill in the surrounding area until the color changes from the original color.

- DFS and BFS on a graph?

11. **Coins: given an infinite number of quarters (25 cents), dimes (10 cents), nickels (5 cents), and pennies (1 cent), calculate the number of ways of representing `n` cents.**

- **How many quarters can be used to represent 100 cents?** The total count is equal to the sum of five cases: using 0 quarters to represent 0,25,50,75 and 100 cents.
- Memoization of each pair `(amount, index)`?

12. Eight Queens: print all ways of arranging eight queens on a 8x8 chess board so that none of them share the same row, column, or diagonal.

- Start from a queen on the first/last row?
- Recursion/backtracking?

13. **Stack of Boxes: the boxes can only be stacked on top of one another if each box is strictly larger than the box above it. Compute the height of the tallest possible stack (sum of the heights of each box).**

- Sort beforehand to avoid looking backwards?
- How to calculate the maximum height of all stacks with a given bottom box?
- At each step, find the next bottom box, or decide whether to put the next box in the stack?

14. **Boolean Evaluation: given a boolean expression consisting of `0,1,&,|,^` and a desired boolean result value. Count the number of ways of parenthesizing the expression such that it evaluates to the result.**

- **Construct the expression tree top-down (last operator) or bottom-up (first operator)?** Split the expression into two parts, and discuss the possible value combinations in order to get the result.
- **The same subtrees are repeatedly calculated. How to optimize?** Store results of `countEval(expression, result)` with memoization.
- **What's the total number of ways of parenthesizing an expression?** Catalan numbers.

