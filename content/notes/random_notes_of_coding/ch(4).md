---
title: Chapter 4 Strings
weight: 4
type: docs
---

## 基本题目

### 344. 反转字符串

`swap` 函数

```cpp
class Solution {
public:
    void reverseString(vector<char>& s) {
        int i = 0, j = s.size() - 1;
        while (i < j) swap(s[i++], s[j--]);
    }
};
```

### 541. 反转字符串 II

`swap` 函数（或使用 `reverse` 函数）

```cpp
class Solution {
public:
    void reverseSubstr(string &s, int i, int j) {
        while (i < --j) swap(s[i++], s[j]);
    }

    string reverseStr(string s, int k) {
        int n = s.size();
        for (int i = 0; i < n; i += 2 * k) {
            reverseSubstr(s, i, min(i + k, n));
            // Or:
            // reverse(s.begin() + i, s.begin() + min(i + k, n));
        }
        return s;
    }
};
```

### 151. 翻转字符串里的单词

双指针滑动窗口

```cpp
class Solution {
public:
    string reverseWords(string s) {
        char *result = new char[s.size() + 1];
        int i = s.size(), j, k = 0;

        bool status = false;
        while (i-- >= 0) {
            bool new_status = (i >= 0 && s[i] != ' ');
            if (!status && new_status) j = i;
            else if (status && !new_status) {
                for (int t = 0; t < j - i; t++) {
                    result[k++] = s[i + t + 1];
                }
                result[k++] = ' ';
            }
            status = new_status;
        }

        result[k - 1] = '\0';
        string str(result);
        delete[] result;
        return str;
    }
};
```

### 189. 轮转数组

**原地轮转**

```cpp
class Solution {
public:
    void rotate(vector<int>& nums, int k) {
        k %= nums.size();
        reverse(nums.begin(), nums.end());
        reverse(nums.begin(), nums.begin() + k);
        reverse(nums.begin() + k, nums.end());
    }
};
```

**环状轮换：单次遍历会访问到 `lcm(n,k)/k` 个元素，因此总共需要执行 `gcd(n,k)` 次**

```cpp
class Solution {
public:
    int gcd(int a, int b) {
        return b ? gcd(b, a % b) : a;
    }

    void rotate(vector<int>& nums, int k) {
        int n = nums.size();
        k %= n;
        int total = gcd(n, k);
        for (int i = 0; i < total; i++) {
            int tmp = nums[i], curr = i;
            do {
                curr = (curr + k) % n;
                swap(tmp, nums[curr]);
            } while (curr != i);
        }
    }
};
```

### 28. 实现 `strStr()`

如何更好地理解和掌握 **KMP 算法**? - 阮行止的回答 - 知乎
https://www.zhihu.com/question/21923021/answer/1032665486

```cpp
class Solution {
public:
    void buildNext(vector<int> &next, const string s) {
        for (int i = 1; i < s.size(); i++) {
            int tmp = i;
            do {
                tmp = next[tmp - 1];
            } while (tmp > 0 && s[tmp] != s[i]);

            if (s[tmp] != s[i]) next[i] = 0;
            else next[i] = tmp + 1;
        }
    }

    int kmpMatch(const string &source, const string &pattern) {
        vector<int> next(pattern.size(), 0);
        buildNext(next, pattern);

        int i = 0, j = 0;
        while (i < source.size() && j < pattern.size()) {
            if (source[i] == pattern[j]) {
                i++;
                j++;
            }
            else if (j > 0) {
                j = next[j - 1];
            }
            else {
                i++;
            }
        }

        if (j >= pattern.size()) return i - j;
        else return -1;
    }

    int strStr(string haystack, string needle) {
        return kmpMatch(haystack, needle);
    }
};
```

### 459. 重复的子字符串

枚举比较

```cpp
class Solution {
public:
    bool repeatedSubstringPattern(string s) {
        for (int l = 1; l <= s.size() / 2; l++) {
            if (s.size() % l) continue;
            string pattern = s.substr(0, l);
            bool valid = true;
            for (int i = l; i < s.size(); i += l) {
                if (s.substr(i, l) != pattern) {
                    valid = false;
                    break;
                }
            }
            if (valid) return true;
        }
        return false;
    }
};
```

**注意到，长度为 n 的字符串 s 是字符串 t=ss 的子串，并且 s 在 t 中的起始位置不为 0 或 n，当且仅当 s 满足题目的要求。** [正确性证明](https://leetcode.cn/problems/repeated-substring-pattern/solutions/386481/zhong-fu-de-zi-zi-fu-chuan-by-leetcode-solution/)
