---
title: Chapter 5 Bit Manipulation
weight: 5
type: docs
---

## Bit Manipulation By Hand

- Addition and multiplication can sometimes be seen as bit shifts (powers of 2).

## Bit Facts and Tricks

- `x ^ 0s = x`
- `x ^ 1s = ~x`
- `x ^ x = 0`
- `x ^ ~x = 1s`

## Two's Complement and Negative Numbers

- The two's complement of an N-bit number (with N-1 bits used for the number) is **the complement of the number with respect to `2^(N-1)`**. The binary representation of `-K` as a N-bit value is `concat(1,2^(N-1) - K)`.
- Another way: invert the bits in the positive representation and then add 1.

## Arithmetic vs. Logical Right Shift

- The arithmetic right shift essentially divides by two (fill in with the sign bit). The logical right shift does what we would visually see as shifting the bits (fill in zeros).
- Logical right shift: `>>>`.

## Common Bit Tasks: Getting and Setting

- Get bit: `num & (1 << i) != 0`.
- Set bit: `num | (1 << i)`.
- Clear bit: `num & ~(1 << i)`.
- **Clear the most significant bits**: `num & ((1 << i) - 1)`.
- **Clear the least significant bits**: `num & (-1 << i)`.

## Exercises

1.  Insertion: insert `M` into `N` such that `M` starts at bit `j` and ends at bit `i`. Both are 32-bit numbers.

- How to clear the designated bits in `N`?

2.  Binary to String: given a double between 0 and 1, print its binary representation. If it cannot be represented accurately in binary with at most 32 characters, print "ERROR".

- How to manually get every bit in a floating-point number?

3.  Flip Bit to Win: you can flip exactly one bit from a 0 to a 1 in an integer. Find the length of the longest sequence of 1s you could create.

- Convert the integer into an array that reflects the lengths of alternating sequences of 0s and 1s?
- The runtime cannot be optimized (B.C.R.), but what about memory usage? Only neighboring sequences are used?

4.  **Next Number: given a positive integer, print the next smallest and the next largest number that have the same number of 1 bits in their binary representation.**

- **If we flip a zero to a one, we must also flip a one to a zero. How would that change the value?** The number will be bigger if and only if the zero-to-one bit was to the left of the one-to-zero bit. Therefore, we need to flip the rightmost zero which has ones on the right of it (the rightmost non-trailing zero).
- **After the zero at position `p` is flipped, how to derive the next smallest number?** If there are `c1` 1s and `c0` 0s to the right of `p`, we could clear the bits to the right of `p` first, and then insert `c1-1` 1s on the right.
- **What about the next largest number?** Find the rightmost non-trailing one, flip it to a zero, clear bits, and insert 1s immediately to the right.
- **An arithmetic approach?** The next smallest: `n + (2^c0-1) + 1 + (2^(c1-1)-1)`. The next largest: `n - (2^c1-1) - 1 - (2^(c0-1)-1)`.

5.  **Debugger: what the following code does: `n & (n-1) == 0`?**

- What does it indicate about 1s?

6.  **Conversion: determine the number of bits you would need to flip to convert integer A to integer B.**

- `XOR` operation?
- **Continuously clear only the least significant bit?**  `c = c & (c-1)`.

7.  Pairwise Swap: swap odd and even bits in an integer with as few instructions as possible.

- Operate on respective digits and then put them together?
- Logical or arithmetic right shift?

8.  Draw Line: a monochrome screen is store as a single array of bytes, allowing eight consecutive pixels to be stored in one byte. The screen has width `w` divisible by 8. Draw a horizontal line from (x1, y) to (x2, y).

- If x1 and x2 are in the same byte?
- Full bytes to be set between them?

