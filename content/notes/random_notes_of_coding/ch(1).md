---
title: Chapter 1 Arrays
weight: 2
type: docs
---

## C++ 接口：vector

- 初始化：
    - 初始化列表 `vector<T> v = {...}`；
    - 默认初始化空数组；
    - 指定长度 `vector<T> v(n)`；
    - 指定长度，并初始化数值 `vector<T> v(n, m)`；
    - 复制其他迭代器 `vector<T> v(it_first, it_last)`。
- 元素访问：
    - `at(pos), operator[](pos)`；
    - `front()`；
    - `back()`。
- 迭代器：
    - `begin()`、`end()`；
    - `rbegin()`、`rend()`。
- 容量信息：
    - `empty()`；
    - `size()`；
    - `capacity()`。
- 修改元素：
    - `clear()`；
    - `insert(pos, v)`；
    - `erase(pos)`；
    - `push_back(v)`；
    - `pop_back()`；
    - `resize(count, v)`。
- 迭代方式 `const auto& v : vec`。

## 基本题目
### 704. 二分查找

二分查找

```cpp
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int lo = 0, hi = nums.size();
        int mi;
        while (lo < hi) {
            mi = (lo + hi) / 2;
            if (nums[mi] < target) lo = mi + 1;
            else if (nums[mi] > target) hi = mi;
            else return mi;
        }
        return -1;
    }
};
```

### 27. 移除元素

双指针（头尾）

```cpp
class Solution {
public:
    int removeElement(vector<int>& nums, int val) {
        int i = 0, j = nums.size();
        while (i < j) {
            if (nums[i] == val) nums[i] = nums[--j];
            else ++i;
        }
        return i;
    }
};
```

### 977. 有序数组的平方

双指针（中间开始往两边）

```cpp
class Solution {
public:
    vector<int> sortedSquares(vector<int>& nums) {
        vector<int> result(nums.size());
        int k = 0;
        int j = lower_bound(nums.begin(), nums.end(), 0) - nums.begin();
        int i = j - 1;
        while (i >= 0 && j < nums.size()) {
            if (nums[i] + nums[j] > 0) {
                result[k++] = nums[i] * nums[i];
                i--;
            }
            else {
                result[k++] = nums[j] * nums[j];
                j++;
            }
        }
        while (i >= 0) {
            result[k++] = nums[i] * nums[i];
            i--;
        }
        while (j < nums.size()) {
            result[k++] = nums[j] * nums[j];
            j++;
        }
        return result;
    }
};
```

**注意：可以采用逆序的方式填充，这样就不需要先求分界线，且不需要考虑边界。**

双指针（两边开始往中间：不需要判断边界）

```cpp
class Solution {
public:
    vector<int> sortedSquares(vector<int>& nums) {
        vector<int> result(nums.size());
        int k = result.size() - 1;
        int i = 0, j = nums.size() - 1;
        while (i <= j) {
            if (nums[i] + nums[j] > 0) {
                result[k--] = nums[j] * nums[j];
                j--;
            }
            else {
                result[k--] = nums[i] * nums[i];
                i++;
            }
        }
        return result;
    }
};
```

### 209. 长度最小的子数组

滑动窗口

```cpp
class Solution {
public:
    int minSubArrayLen(int target, vector<int>& nums) {
        int minLen = INT_MAX;
        int i = 0, j = -1;
        int sum = 0;

        while (++j < nums.size()) {
            sum += nums[j];
            while (sum - nums[i] >= target) {
                sum -= nums[i];
                i++;
            }
            if (sum >= target && j - i + 1 < minLen) {
                minLen = j - i + 1;
            }
        }

        if (minLen == INT_MAX) return 0;
        return minLen;
    }
};
```

**注意：子数组和也可以视为前缀和之差。**

### 59. 螺旋矩阵 II

模拟过程

```cpp
class Solution {
public:
    vector<vector<int>> generateMatrix(int n) {
        vector<vector<int>> result(n, vector<int>(n));
        int count = 1;
        for (int l = 0; l < (n + 1) / 2; l++) {
            for (int j = l; j < n - l; j++)
                result[l][j] = count++;
            for (int i = l + 1; i < n - l - 1; i++)
                result[i][n - l - 1] = count++;
            for (int j = n - l - 1; j > l; j--)
                result[n - l - 1][j] = count++;
            for (int i = n - l - 1; i > l; i--)
                result[i][l] = count++;
        }
        return result;
    }
};
```

## 拓展练习

### 35. 搜索插入位置

二分查找的返回值

```cpp
class Solution {
public:
    int searchInsert(vector<int>& nums, int target) {
        int lo = 0, hi = nums.size();
        int mi;
        while (lo < hi) {
            mi = (lo + hi) / 2;
            if (nums[mi] > target) hi = mi;
            else if (nums[mi] < target) lo = mi + 1;
            else return mi;
        }
        return lo;
    }
};
```

### 34. 在排序数组中查找元素的第一个和最后一个位置

使用二分查找搜寻第一个大于目标值/最后一个小于目标值的元素

```cpp
class Solution {
public:
    vector<int> searchRange(vector<int>& nums, int target) {
        int lo = 0, hi = nums.size();
        int mi;
        while (lo < hi) {
            mi = (lo + hi) / 2;
            if (nums[mi] > target) hi = mi;
            else if (nums[mi] < target) lo = mi + 1;
            else break;
        }

        if (lo >= hi) return {-1, -1};

        int l = lo, h = mi + 1, m;
        while (l < h) {
            m = (l + h) / 2;
            if (nums[m] >= target) h = m;
            else l = m + 1;
        }
        lo = h;

        l = mi;
        h = hi;
        while (l < h) {
            m = (l + h) / 2;
            if (nums[m] > target) h = m;
            else l = m + 1;
        }
        hi = l - 1;

        return {lo, hi};
    }
};
```

**可以将问题简化为查找第一个大于等于目标值的元素、第一个大于目标值的元素**

### 69. x 的平方根

广义二分查找

```cpp
class Solution {
public:
    int mySqrt(int x) {
        int prev = -1, curr = 1;
        while ((long long)curr * curr < x) {
            prev = curr;
            curr *= 2;
        }
        
        // Search from prev + 1 to curr
        int lo = prev + 1, hi = curr + 1, mi;
        long long val;
        while (lo < hi) {
            mi = (lo + hi) / 2;
            val = (long long)mi * mi;
            if (val > x) hi = mi;
            else if (val < x) lo = mi + 1;
            else return mi;
        }

        return lo - 1;
    }
};
```

**这里将原问题复杂化了：二分查找存在明确的上下限：0 与 x**

**牛顿迭代法：求解 $f(x)=x^2-C$ 的零点：选任意点为起点，构造切线，求切线与坐标轴的交点，使用新的横坐标得到新的点。以此方式逼近交点**

```cpp
class Solution {
public:
    int mySqrt(int x) {
        if (x <= 1) return x;

        double x0 = x, x1 = x / 2;
        while (fabs(x0 - x1) > 1e-7) {
            x0 = x1;
            x1 = (x0 + x / x0) / 2;
        }
        return (int)x1;
    }
};
```

### 367. 有效的完全平方数

牛顿迭代法

```cpp
class Solution {
public:
    int mySqrt(int x) {
        if (x <= 1) return x;

        double x0 = x, x1 = x / 2;
        while (fabs(x0 - x1) > 1e-7) {
            x0 = x1;
            x1 = (x0 + x / x0) / 2;
        }
        return (int)x1;
    }

    bool isPerfectSquare(int num) {
        int x = mySqrt(num);
        return x * x == num;
    }
};
```

### 26. 删除排序数组中的重复项

快慢指针

```cpp
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        int i = 0;
        int prev = nums[i] - 1;
        for (int j = 0; j < nums.size(); j++) {
            if (nums[j] != prev) {
                prev = nums[j];
                nums[i++] = prev;
            }
        }
        return i;
    }
};
```

### 283. 移动零

快慢指针，查找的值是固定的，可以不交换元素而在最后统一赋值

```cpp
class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        int i = 0;
        for (int j = 0; j < nums.size(); j++) {
            if (nums[j] != 0) {
                nums[i++] = nums[j];
            }
        }
        while (i < nums.size()) {
            nums[i++] = 0;
        }
    }
};
```

### 844. 比较含退格的字符串

双指针（两字符串各一个），逆向移动，逐字符比较

比较栈的元素全部相同：从操作序列的末尾开始反向比较

```cpp
class Solution {
public:
    char nextChar(const string &s, int &i) {
        // The last character of s[0..i] (inclusive .. exclusive)
        int bsCount = 0;
        while (--i >= 0) {
            if (s[i] == '#') bsCount++;
            else if (bsCount > 0) bsCount--;
            else return s[i];
        }
        return '\0';
    }

    bool backspaceCompare(string s, string t) {
        int i = s.size(), j = t.size();
        char cs, ct;
        while (i >= 0 && j >= 0) {
            cs = nextChar(s, i);
            ct = nextChar(t, j);
            if (cs != ct) return false;
        }
        return i < 0 && j < 0;
    }
};
```

### 904. 水果成篮

滑动窗口，每次更新需要去除上一个出现的水果种类，因此只需要维护上一个水果的最近一次出现位置

```cpp
class Solution {
public:
    int totalFruit(vector<int>& fruits) {
        int prev = -1, prevIdx = -1;
        int curr = -1;
        int count = 0;
        int maxCount = 0;

        for (int i = 0; i < fruits.size(); i++) {
            if (fruits[i] == curr) {
                count++;
            }
            else if (fruits[i] == prev) {
                prev = curr;
                prevIdx = i - 1;
                curr = fruits[i];
                count++;
            }
            else {
                count = i - prevIdx;
                prev = curr;
                prevIdx = i - 1;
                curr = fruits[i];
            }

            if (count > maxCount) {
                maxCount = count;
            }
        }

        return maxCount;
    }
};
```

### 76. 最小覆盖子串

滑动窗口的拆解：滑动窗口的本质就是队列（每次读入一个元素，然后扔掉一些没用的元素）。要追溯多个元素的位置，因此对每个字符维护一个队列，表示窗口范围内的出现位置。读取下一字符时移除队列头部的位置

```cpp
class Solution {
public:
    string minWindow(string s, string t) {
        map<char, pair<int, queue<int>>> tracker;
        for (const char ct : t) {
            tracker[ct].first++;
        }

        bool foundAll = false;
        int shortestLen = INT_MAX;
        int shortestId = -1;
        for (int i = 0; i < s.size(); i++) {
            char cs = s[i];
            if (tracker.find(cs) != tracker.end()) {
                tracker[cs].second.push(i);
                if (tracker[cs].first == 0)
                    tracker[cs].second.pop();
                else
                    tracker[cs].first--;

                // Check validity
                if (!foundAll) {
                    foundAll = true;
                    for (auto const &[k, v] : tracker) {
                        if (v.first != 0) {
                            foundAll = false;
                            break;
                        }
                    }
                }

                if (foundAll) {
                    // Find begin index
                    int beginId = i;
                    int id;
                    for (auto const &[k, v] : tracker) {
                        if ((id = v.second.front()) < beginId) {
                            beginId = id;
                        }
                    }

                    // Compare shortest
                    if (i - beginId + 1 < shortestLen) {
                        shortestId = beginId;
                        shortestLen = i - beginId + 1;
                    }
                }
            }
        }

        if (shortestId < 0) return "";
        else return s.substr(shortestId, shortestLen);
    }
};
```

**事实上，上述方法复杂化了该问题。只需要使用一个滑动窗口即可实现上述的过程。在循环过程中维护一个统计窗口内字符出现次数的计数器，先向右扩展到符合要求，在收缩左侧的区间，得到局部最优解**

```cpp
class Solution {
public:
    bool checkValid(const map<char, int> &counter) {
        for (const auto &[k, v] : counter)
            if (v > 0) return false;
        return true;
    }

    string minWindow(string s, string t) {
        map<char, int> counter;
        for (const char ct : t) {
            counter[ct]++;
        }

        int shortestLen = INT_MAX;
        int shortestId = -1;
        int i = 0, j = 0;     // Window Interval [i, j]
        while (j < s.size()) {
            char cs = s[j];
            if (counter.find(cs) != counter.end()) {
                counter[cs]--;
                if (checkValid(counter)) {
                    do {
                        if (counter.find(s[i]) != counter.end())
                            counter[s[i]]++;
                        i++;
                    } while (checkValid(counter) && i <= j);
                    if (shortestLen > j - i + 2) {
                        shortestId = i - 1;
                        shortestLen = j - i + 2;
                    }
                }
            }
            j++;
        }

        if (shortestId < 0) return "";
        else return s.substr(shortestId, shortestLen);
    }
};
```

**可以使用滑动窗口解决的问题性质：不存在两个局部最优解，其中一个被另一个真包含**

### 54. 螺旋矩阵

模拟过程，注意边界情况（1\*1,1\*n, m\*1）

```cpp
class Solution {
public:
    vector<int> spiralOrder(vector<vector<int>>& matrix) {
        int m = matrix.size(), n = matrix[0].size(), k = 0;
        vector<int> result(m * n);
        for (int l = 0; l < min((m + 1) / 2, (n + 1) / 2); l++) {
            for (int j = l; j < n - l; j++)
                result[k++] = matrix[l][j];
            for (int i = l + 1; i < m - l; i++)
                result[k++] = matrix[i][n - l - 1];
            if (m > 2 * l + 1)
                for (int j = n - l - 2; j > l; j--)
                    result[k++] = matrix[m - l - 1][j];
            if (n > 2 * l + 1)
                for (int i = m - l - 1; i > l; i--)
                    result[k++] = matrix[i][l];
        }
        return result;
    }
};
```
