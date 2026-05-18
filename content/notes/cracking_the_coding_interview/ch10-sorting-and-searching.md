---
title: Chapter 10 Sorting and Searching
weight: 10
type: docs
---

## Common Sorting Algorithms

- Bubble Sort: Runtime `O(n^2)` average and worst case. Memory `O(1)`.
- Selection Sort: Runtime `O(n^2)` average and worst case. Memory `O(1)`.
- **Merge Sort**: Runtime `O(nlog(n))` average and worst case. Memory depends on the implementation, could be `O(n)` (auxiliary array).
- **Quick Sort**: Runtime `O(nlog(n))` average, `O(n^2)` worst case. Memory `O(log(n))`.
- Radix Sort: Runtime `O(kn)`. It takes advantage of the fact that integers have a finite number of bits. Each pass could be implemented with **counting sort** (`O(n+k)`) or **bucket sort**.

## Searching Algorithms

- Think beyond binary search.

## Exercises

1.  Sorted Merge: given two sorted arrays A and B, merge B into A in sorted order.

- In-place sorting: merge from the front or the back of A?

2.  Group Anagrams: sort an array of strings so that all the anagrams are next to each other.

- How to check if two words are anagrams? Sort each word? Or count the occurrences of distinct characters?
- How to group anagrams? Fully sort the array? Bucket sort?

3.  **Search in Rotated Array: given a sorted array of n integers that has been rotated an unknown number of times, find an element in the array.**

- **One half of the array must be ordered normally?** Find the normally ordered half by comparing `a[0]` and  `a[mid]`, and use its max and min values to determine where to search next.
- **What if there are identical elements?** If the left and the middle are identical, we should check if the rightmost element is different. If it is (it can only be smaller), then we can search just the right side. Otherwise, we have to search both halves.
- Does it matter if the one to find is identical to head/tail elements?

4.  **Sorted Search, No Size: you are given an array-like data structure `Listy` without a size method. `elementAt(i)` will return `-1` if index is out of range. Given a `Listy` which contained sorted, positive integers, search for element `x`.**

- **How to go through the list? What's the worst case time?** Go through the list exponentially to find the approximate location, which costs `O(log(n))` time. Then apply the binary search, which costs `O(log(n))` as well.

5.  Sparse Search: given a sorted array of strings that is interspersed with empty strings, find the location of a given string.

- How to address the situation when the middle is an empty string? Resort to neighboring strings?
- Search for an empty string?

6.  Sort Big File: Imagine you have a 20 GB file with one string per line. How would you sort the file?

- External merge sort?

7.  **Missing Int: given an input file with four billion non-negative integers, generate an integer that is not contained in the file. You have 1 GB memory available for this task. What if you have only 10 MB of memory? Assume all the values are distinct and we now have no more than one billion non-negative integers.**

- Bit vector?
- **How to deal with insufficient space for all distinct integers?** Count the numbers that fall in different ranges of integers, eg. 0-999,1000-1999, etc. If only 999 values fall in a particular range, we know a missing int must be in that range. Another pass on that range will do the job.
- **How to find the optimum range size?** The first pass: the number of ranges $2^{31}/S \le 2^{21}$, in that $2^{21}$ integers take up 10 MB. The second pass: the size of the bit vector $S/8 \le 2^{23}$.
- **What if even less memory?** Repetitive steps of shrinking the range.

8.  Find Duplicates: you have an array with all the numbers from 1 to N, where N is at most 32,000. The array may have duplicate entries and you do not know N. With only 4 KB memory available, print all duplicate elements in the array.

- Bit vector?

9.  **Sorted Matrix Search: given an MxN matrix in which each row and each column is sorted in ascending order, find an element.**

 - Naïve method: process row by row?
 - **How to eliminate non-necessary rows and columns?** If the start of a column is greater than `x`, then `x` is to the left of the column. If the end of a column is less than `x`, then `x` is to the right of the column. Similar logics apply to rows. We can start at either corner of the matrix, eliminate rows and columns, and the target element will end up in a corner of the submatrix.
 - **View the problem from a divide-and-conquer perspective. Find the location of the target element along the diagonal so that it is still sorted. What can we infer from this location?** Divide the matrix with a vertical and a horizontal line from there. The elements in the upper left and bottom right quadrants are either less or greater than `x`. We only need to recursively search the lower left quadrant and the upper right quadrant.

10. **Rank from Stream: imagine you are reading in a stream of integers. You wish to be able to look up the rank of number `x`(the number of values no greater than `x`) periodically. Implement the data structures and algorithms to support these operations.**

- Hold all elements in an array in sorted order?
- **Which data structure can dynamically maintain order information?** BST.
- How to save information of ranks in tree nodes?
- What if `x` is not found in the tree?

11. **Peaks and Valleys: given an array of integers, sort the array into an alternating sequence of peaks and valleys.**

- **Suboptimal solution: sort it first, and adjust the order?** Iterate through the elements with step 2, and swap with the previous one at each element.
- **The complete sorting is unnecessary. How to keep elements in a locally orderly way?** We can fix the sequence by swapping the center element with the largest adjacent element. Its correctness could be easily proved.

