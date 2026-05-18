---
title: Chapter 6 Math and Logic Puzzles
weight: 6
type: docs
---

## Prime Numbers

- `gcd * lcm = x * y`.
- Naïve way of **checking for primality**: simply iterate from `2` through `n-1`(improvement: the square root of `n`), and check for divisibility on each iteration.
- **Generating a list of primes: the Sieve of Eratosthenes** - all non-prime numbers are divisible by a prime number. Cross off all numbers divisible by 2, 3, 5, 7, and so on.

## Probability

- Bayers' Theorem: $P(A|B) = P(B|A)P(A)/P(B)$.
- $P(A \vee B) = P(A) + P(B) - P(A\wedge B)$.
- Independent: $P(A\wedge B) = P(A)P(B)$.

## Develop Rules and Patterns

- You have two ropes, and each takes exactly one hour to burn. How would you use them to time exactly 15 minutes?

## Worst Case Shifting

- You have nine balls. Eight are of the same weight, and one is heavier. Find the heavy ball in just two uses of the scale.

## Exercises

1. The Heavy Pill: you have 20 bottles of pills. 19 bottles have 1.0 gram pills, but one has pills of weight 1.1 grams. Given a scale that provides an exact measurement, how would you find the heavy bottle? You can only use the scale once.

2. Basketball: you can play one of two games: get one shot to make the hoop, or, get three shots and you have to make two of three shots. If `p` is the probability of making a particular shot, which game should you pick?

3. Dominos: there is an 8x8 chessboard in which two diagonally opposite corners have been cut off. Can you use the 31 dominos to cover the entire board?

4. Ants on a Triangle: there are three ants on different vertices of a triangle. What is the probability of a collision if they start walking on the sides of the triangle? Each ant randomly picks a direction and walks at the same speed.

5. Jugs of Water: you have a five-quart jug, a three-quart jug, and an unlimited supply of water. How to come up with exactly four quarts of water?

- How many different volumes of water can possibly be acquired from the process?

6. **Blue-Eyed Island: all blue-eyed people must leave the island as soon as possible. Each person can see everyone else's eye color, but they do not know their own. How many days will it take the blue-eyed people to leave?**

- **The Base Case and Build approach? What if there are 1, 2, or more blue-eyed people?** If there are `c` blue-eyed people, they will leave together on the `c`th night.

7. The Apocalypse: what will the gender ratio be if all families should have babies until they have exactly one girl?

8. The Egg Drop Problem: there is a building of 100 floors. If an egg drops from the `n`th floor or above, it will break. You're given two eggs. Find `n` while minimizing the number of drops for the worst case.

- Worst case balancing?

10. 100 Lockers: there are 100 closed lockers. A man begins by opening all 100 lockers. Next, he closes every second locker (starting from locker 2). Then, he toggles every third locker. After 100 passes, how many lockers are open?

- When would a door be left open? Perfect squares?

11. **Poison: you have 1000 bottles of soda, and exactly one is poisoned. You have 10 test strips which can be used to detect poison. A single drop of poison will turn the strip positive permanently. You can put any number of drops on a test strip, but you can only run tests once a day, and it takes sever days to return a result. How to find the poisoned bottle in as few days as possible? Write code to simulate the approach.**

- Naïve approach: use up all test strips at once and wait 7 days for results?
- **Optimized approach: take advantage of the time delay? What about dripping on the same strip the next day?** Divide all bottles according to the first, second and third digit, and run tests on different days (day 0, 1, 2). The results shown on day 7, 8, 9 will reveal each digit of the poisoned bottle.
- **How to distinguish between bottle #383 and #388?** Run an additional test on day 3 on the third digit as well. Shift the final digit so that it winds up in a different place.
- **Optimal approach: what each strip really means?** A binary indicator. Map 1000 keys to different combinations of 10 binary values with their binary number representations.

