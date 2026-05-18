---
title: Chapter 16 Moderate
weight: 16
type: docs
---

## Selected Exercises

1.  **Number Swapper: swap a number in place (without temporary variables).**

- Calculate the sum or the difference of two numbers first? Overflow?
- **Substitute addition and subtraction with bit manipulation?** Both operation can be replaced with XOR operation.
- What does it mean to XOR a bit with 0/1?

```cpp
class Solution {
public:
    vector<int> swapNumbers(vector<int>& numbers) {
        numbers[1] ^= numbers[0];
        numbers[0] ^= numbers[1];
        numbers[1] ^= numbers[0];
        return numbers;
    }
};
```

5.  Factorial Zeros: compute the number of trailing zeros in n factorial.

- Count the number of multiples of 5?

6.  Smallest Difference: given two arrays of integers, compute the pair of values (one value in each array) with the smallest (non-negative) difference.

- Sort the arrays first?

7.  **Number Max: find the maximum of two numbers. You should not use if-else or any other comparison operator.**

- Check the sign of `a - b`?
- **How to deal with `a - b` overflow?** If the numbers have different signs, we choose the positive one. Otherwise, `a - b` will not overflow.

```c
int sign(long long x) {
    return (x >> 63) & 1;
}

int maximum(int a, int b) {
    int sa = sign(a), sb = sign(b), sc = sign((long long)a - b);
    int diff_sign = sa ^ sb;

    int k = diff_sign * (1 ^ sa) + (1 ^ diff_sign) * (1 ^ sc);
    int q = 1 ^ k;
    return a * k + b * q;
}
```

10. **Living People: given a list of people with their birth and death years, compute the year with the most number of people alive. If there are more than one years that have the most number of people alive, return the smallest one.**

- Brute force: count the number of people alive each year?
- **Does it really matter whether years of birth and death are matched up?** No. So we can sort both arrays and walk through them to track the number of people. Update the number of people if there's someone born.
- **Remove the process of sorting?** The range of years is limited, so we can take advantage of the concept of bucket sort. Some issues should be taken care of, eg. recording death dates in the following year.

16. **Sub Sort: given an array of integers, find indices `m` and `n` such that if you sorted elements `m` through `n`, the entire array would be sorted. Minimize `n - m`.**

- **Intuitively, what should the start and end of the array be like?** The start and end of the array is already in order. We might find the longest increasing subsequence at the beginning and at the end.
- **How to meet the condition of `left < middle < right`?** Shrink the left and right subsequences. For the left subsequence, shorten it until the end is less than `min(middle, right)`. For the right subsequence, shorten it until the start is greater than `max(left, middle)`. Each step considers the remaining part of the array as a whole, so they can be independently executed.

```cpp
class Solution {
public:
    vector<int> subSort(vector<int>& array) {
        if (array.size() == 0) return {-1, -1};

        // find left and right sub-sequence
        int lend = 1;
        for (; lend < array.size(); lend++) {
            if (array[lend] < array[lend - 1]) {
                break;
            }
        }
        if (lend == array.size()) return {-1, -1};
        int rstart = array.size() - 1;
        for (; rstart >= 0; rstart--) {
            if (array[rstart] < array[rstart - 1]) {
                break;
            }
        }

        // min and max in the middle sequence
        int min_mid = INT_MAX, max_mid = INT_MIN;
        for (int i = lend; i < rstart; i++) {
            if (array[i] < min_mid) min_mid = array[i];
            if (array[i] > max_mid) max_mid = array[i];
        }
        
        int lend_target = (min_mid > array[rstart] ? array[rstart] : min_mid);
        int rstart_target = (max_mid < array[lend - 1] ? array[lend - 1] : max_mid);
        while (lend > 0 && array[lend - 1] > lend_target) lend--;
        while (rstart < array.size() && array[rstart] < rstart_target) rstart++;

        return {lend, rstart - 1};
    }
};
```

17. Contiguous Sequence: given an array of integers, find the contiguous sequence with the largest sum. Return the sum.

- Common solution - dynamic programming?
- Think about the array as being a sequence of alternating negative and positive numbers. Is it equivalent to the original problem?

18. **Pattern Matching: you are given two strings, `pattern` and `value`. The `pattern` string consists of just the letters a and b, describing a pattern within a string. For example, the string `catcatgocatgo` matches the pattern `aabab`(where `cat` is `a` and `go` is `b`). It also matches patterns like `a`, `ab`, and `b`. Determine if `value` matches `pattern`.**

- Brute force: look for all substrings of the `value` string as patterns of `a` and `b`?
- **Searching through all values for the alternate string (second pattern) is slow. How to find it quickly?** If the main string (first pattern) is determined, the locations for the alternate string can be automatically calculated.
- **Can we compare with patterns without building them?** Compare with the first instance of pattern `a/b` without constructing the whole string.

19. Pond Sizes: you have an integer matrix representing a plot of land, where the value at that loca­tion represents the height above sea level. A value of zero indicates water. A pond is a region of water connected vertically, horizontally, or diagonally. The size of the pond is the total number of connected water cells. Write a method to compute the sizes of all ponds in the matrix.

- DFS/BFS?
- Save space by using `land` matrix as `detected` as well?

```cpp
class Solution {
public:
    vector<int> pondSizes(vector<vector<int>>& land) {
        vector<int> pondSizes;
        for (int i = 0; i < land.size(); i++)
            for (int j = 0; j < land[0].size(); j++)
                if (!land[i][j])
                    pondSizes.push_back(detectPond(land, i, j));

        sort(pondSizes.begin(), pondSizes.end());
        return pondSizes;
    }

    int detectPond(vector<vector<int>>& land, int i, int j) {
        if (i < 0 || i >= land.size() || j < 0 || j >= land[0].size() || land[i][j] != 0) return 0;
        
        int result = 1;
        land[i][j] = -1;  // -1 means detected water cell
        for (int di = -1; di <= 1; di++)
            for (int dj = -1; dj <= 1; dj++)
                result += detectPond(land, i + di, j + dj);
        
        return result;
    }
};
```

20. **T9: on old cell phones, users typed on a numeric keypad and the phone would provide a list of words that matched these numbers. Each digit mapped to a set of 0-4 letters. Find which words can be represented by a given sequence of digits among a list of words.**

- How to list all words that can be represented by the digits?
- **Pruning? How to check if there are words in the dictionary that start with a prefix?** Tries (prefix trees). Construct a trie with the given dictionary. As we go down a recursion path and a trie path, stop early if the path terminates in the trie.
- **Instead of checking matches on the fly, can we do something in advance?** Use a hash table that maps characters to digits to convert all words in the dictionary to digits. Save each word in another hash table, which maps digit sequences to word lists. During word look-up, only one key-value pair needs to be looked up.

21. Sum Swap: given two arrays of integers, find a pair of values (one value from each array) that you can swap to give the two arrays the same sum.

- How to search for the position of a particular number in an array?
- What if the arrays are sorted?

22. **Langton's Ant: simulate the first K moves that the ant makes and print the final board as a grid.**

- **If arrays are used to represent the grid, how to handle negative expansions?** Relabel all the indices.
- **Do we actually need a grid?** All we need is a list of the white squares and the ant. We can use a HashSet of the white squares.
- **How to print the board with the HashSet?** Update the most top-left position and the most bottom-right position if necessary.

23. **Rand7 from Rand5: implement a method `rand7()` given `rand5()`.**

- **How to generate a range of values where each value is equally likely, and the range has at least seven elements?** `5 * rand5() + rand5()`.

