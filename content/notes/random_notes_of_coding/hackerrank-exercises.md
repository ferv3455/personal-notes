---
title: (Extra) HackerRank Exercises
weight: 99
type: docs
---

## 基本题目

### 3-2 Tower Breakers

> 共有 n 座 m 高度的塔，每次可以将一座塔的高度缩减为其因数。两人轮流行动，直至所有塔的高度为 1，无法行动者失败。求谁将获胜。

Divide and Conquer

先求单座塔的问题 `f(1,m)`（先手取胜时为  `true`）：先手取胜当且仅当存在一个 `m` 的因数 `m'`，使得 `f(1,m')` 为 `false` （对手无法取胜）。然后推广到 `n` 座塔：

- 如果 `f(1,m)` 为 `false`，那么后手方可以在每座塔内击败先手方，先手败；
- 如果 `n` 为偶数，则后手方可以镜像先手方的操作，先手败；
- 如果 `n` 为奇数，且假设先手方先操作塔 1 。若后手方操作塔 2-n，则先手方可以镜像后手方的操作；若后手方操作塔 1，则由于先手方有单塔获胜策略，故先手胜。

### 4-3 New Year Chaos

> 初始序列的顺序为 1 至 n。每个数字至多可以向前移动两位。给定目标序列，判断要实现目标序列所需的移动次数。

保序的排列方式：基于目标序列，将目标位置的对应数字提前，并更新其他元素。这样一来可以实现最少次数的移动（每次从最靠前的元素开始）。

```go
func minimumBribes(q []int32) {
    var sum int = 0
    var n int = len(q)
    line := make([]int32, n) // the current line
    pos := make([]int, n)    // i + 1 is at position pos[i]
    for i := 0; i < n; i++ {
        line[i] = int32(i + 1)
        pos[i] = i
    }
    
    for i, id := range q {
        switch diff := pos[id - 1] - i; {
        case diff > 2:
            fmt.Println("Too chaotic")
            return
        case diff > 0:
            sum += diff
            for j := int(diff); j > 0; j-- {
                line[i + j] = line[i + j - 1]
                pos[line[i + j] - 1] = i + j
            }
            // pos[id - 1] = i
        }
    }
    fmt.Println(sum)
}
```

## 测试题目

### 3 Palindrome Index

> 给定一个字符串，从中去掉一个字符，可以将它变为回文串。找到这个位置。

- 基本解答：枚举位置+测试合法性，平方复杂度；
- 直接从两端开始比较，如果比较到 i 与 j 位时字符不同，则只可能有两种情况符合要求：去除 i 与去除 j。

### 4 Truck Tour

> 给定环路上各加油站的油量和到下一站的距离。求可开完环路的起点。

利用子序列的关系来避免各个位置从头求和：如果从 a 到 b 的元素和为负，则说明以其中任意一点 c 为起点，到 b 的元素和均为负（否则 a 到 c 的元素和为负）。

```go
func truckTour(petrolpumps [][]int32) int32 {
    // Write your code here
    var i, start, n int = 0, 0, len(petrolpumps)
    var sum int32 = 0
    for i - start < n {
        sum += petrolpumps[i % n][0] - petrolpumps[i % n][1]
        if sum < 0 {
            start = i + 1
            sum = 0
        }
        i++
    }
    return int32(start)
}
```
