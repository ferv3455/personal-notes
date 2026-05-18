---
title: Chapter 3 Hash Tables
weight: 3
type: docs
---

## C++ 接口：unordered_map

- 初始化：
    - 初始化列表 `unordered_map<Key, T> map = { {k1, v1}, {k2, v2}}`；
    - 默认初始化空映射；
    - 复制其他迭代器 `unordered_map<Key, T> map(it_first, it_last)`（支持 pair 类型的数组）。
- 元素查找：
    - `at(key), operator[](key)`；
    - `count(key)`；
    - `find(key)` 返回迭代器。
- 迭代器：
    - `begin()`、`end()`。
- 容量信息：
    - `empty()`；
    - `size()`。
- 修改元素：
    - `clear()`；
    - `insert(key, v)`；
    - `erase(key)`。
- 迭代方式：
    - `const auto& [key, value] : u`；
    - `const auto& n : u` 以 pair 类型处理。

## C++ 接口：unordered_set

- 初始化：
    - 初始化列表 `unordered_set<T> set = {...}`；
    - 默认初始化空集合；
    - 复制其他迭代器 `unordered_set<T> set(it_first, it_last)`。
- 元素查找：
    - `count(v)`；
    - `find(v)` 返回迭代器。
- 迭代器：
    - `begin()`、`end()`。
- 容量信息：
    - `empty()`；
    - `size()`。
- 修改元素：
    - `clear()`；
    - `insert(v)`；
    - `erase(v)`。
- 迭代方式：`const auto& v : set`。

## 基本题目

### 242. 有效的字母异位词

计数器

```cpp
class Solution {
public:
    bool isAnagram(string s, string t) {
        unordered_map<char, int> counter;
        for (char c : s) counter[c]++;
        for (char c : t) counter[c]--;
        for (const auto &[k, v] : counter)
            if (v != 0) return false;
        return true;
    }
};
```

**数组就是空间有限的、散列函数简单的哈希表**

```cpp
class Solution {
public:
    bool isAnagram(string s, string t) {
        vector<int> counter(26, 0);
        for (char c : s) counter[c - 'a']++;
        for (char c : t) counter[c - 'a']--;
        for (const int v : counter)
            if (v != 0) return false;
        return true;
    }
};
```

### 349. 两个数组的交集

简单计数器——集合

```cpp
class Solution {
public:
    vector<int> intersection(vector<int>& nums1, vector<int>& nums2) {
        unordered_set<int> exist;
        for (int v : nums1) exist.insert(v);

        vector<int> result;
        for (int v : nums2) {
            if (exist.count(v)) {
                result.push_back(v);
                exist.erase(v);
            }
        }

        return result;
    }
};
```

### 202. 快乐数

用集合来判断重复

```cpp
class Solution {
public:
    int calculateDigitSquared(int n) {
        int result = 0;
        while (n) {
            result += (n % 10) * (n % 10);
            n /= 10;
        }
        return result;
    }

    bool isHappy(int n) {
        unordered_set<int> history;
        while (history.count(n) == 0) {
            history.insert(n);
            n = calculateDigitSquared(n);
        }
        return n == 1;
    }
};
```

**这种使用前一个元素计算得到后一个元素的方式等价于链表结构，因此这也就等价于寻找链表中是否存在环（1 为链表尾）**

### 1. 两数之和

记录出现位置

```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_map<int, int> location;
        int lookup_idx;
        for (int i = 0; i < nums.size(); i++) {
            if (location.count(target - nums[i]) && (lookup_idx = location[target - nums[i]]) != i) {
                return {i, lookup_idx};
            }
            location[nums[i]] = i;
        }
        return {-1, -1};
    }
};
```

### 454. 四数相加 II

计数器的合并

```cpp
class Solution {
public:
    void countSums(unordered_map<int, int> &count, const vector<int> &nums1, const vector<int> &nums2) {
        for (const int n1 : nums1)
            for (const int n2 : nums2)
                count[n1 + n2]++;
    }

    int fourSumCount(vector<int>& nums1, vector<int>& nums2, vector<int>& nums3, vector<int>& nums4) {
        unordered_map<int, int> count1, count2;
        countSums(count1, nums1, nums2);
        countSums(count2, nums3, nums4);

        int result = 0;
        for (const auto &[sum, c] : count1)
            result += count2[-sum] * c;
        return result;
    }
};
```

### 15. 三数之和

**排序+迭代+双指针。避免重复：在迭代中与上一元素比较**

```cpp
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        int n = nums.size();
        sort(nums.begin(), nums.end());
        vector<vector<int>> result;
        for (int i = 0; i < n; i++) {
            if (i > 0 && nums[i] == nums[i - 1]) continue;
            int j = i + 1, k = n - 1;
            while (j < k) {
                if (j > i + 1 && nums[j] == nums[j - 1]) {
                    j++;
                    continue;
                }
                if (k < n - 1 && nums[k] == nums[k + 1]) {
                    k--;
                    continue;
                }
                if (nums[i] + nums[j] + nums[k] > 0) k--;
                else if (nums[i] + nums[j] + nums[k] < 0) j++;
                else {
                    result.push_back({nums[i], nums[j], nums[k]});
                    j++;
                    k--;
                }
            }
        }
        return result;
    }
};
```

### 18. 四数之和

排序+迭代+双指针+避免重复

```cpp
class Solution {
public:
    vector<vector<int>> fourSum(vector<int>& nums, int target) {
int n = nums.size();
        sort(nums.begin(), nums.end());
        vector<vector<int>> result;
        for (int i = 0; i < n; i++) {
            if (i > 0 && nums[i] == nums[i - 1]) continue;
            for (int l = n - 1; l > i; l--) {
                if (l < n - 1 && nums[l] == nums[l + 1]) continue;
                int j = i + 1, k = l - 1;
                while (j < k) {
                    if (j > i + 1 && nums[j] == nums[j - 1]) {
                        j++;
                        continue;
                    }
                    if (k < l - 1 && nums[k] == nums[k + 1]) {
                        k--;
                        continue;
                    }
                    if ((long long)nums[i] + nums[j] + nums[k] + nums[l] > target) k--;
                    else if ((long long)nums[i] + nums[j] + nums[k] + nums[l] < target) j++;
                    else {
                        result.push_back({nums[i], nums[j], nums[k], nums[l]});
                        j++;
                        k--;
                    }
                }
            }
        }
        return result;
    }
};
```

## 拓展练习

### 383. 赎金信

计数器

```cpp
class Solution {
public:
    bool canConstruct(string ransomNote, string magazine) {
        int counter[26] = {0};
        for (const char c : magazine) counter[c - 'a']++;
        for (const char c : ransomNote) {
            if (!counter[c - 'a']) return false;
            counter[c - 'a']--;
        }
        return true;
    }
};
```

### 49. 字母异位词分组

对字符串进行编码：桶排序

```cpp
class Solution {
public:
    string encode(const string &str) {
        int counter[26] = {0};
        for (const char c : str)
            counter[c - 'a']++;
        
        stringstream result;
        for (int i = 0; i < 26; i++) {
            if (counter[i])
                result << (char)(i + 'a') << to_string(counter[i]);
        }
        return result.str();
    }

    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        unordered_map<string, vector<string>> groups;
        for (const string &str : strs)
            groups[encode(str)].push_back(str);
        
        vector<vector<string>> result;
        for (const auto &[k, v] : groups)
            result.push_back(v);
        return result;
    }
};
```

**在 C++ 语言中，可以对数组定义新的散列函数，从而将数组作为映射的键**

```cpp
auto arrayHash = [fn = hash<int>{}] (const array<int, 26>& arr) -> size_t {
    return accumulate(arr.begin(), arr.end(), 0u, [&](size_t acc, int num) {
        return (acc << 1) ^ fn(num);
    });
};

unordered_map<array<int, 26>, vector<string>, decltype(arrayHash)> mp(0, arrayHash);
```

### 438. 找到字符串中所有字母异位词

滑动窗口+计数器

```cpp
class Solution {
public:
    bool checkValid(int *counter) {
        for (int j = 0; j < 26; j++) {
            if (counter[j] != 0) return false;
        }
        return true;
    }

    vector<int> findAnagrams(string s, string p) {
        int counter[26] = {0};
        for (const char c : p) {
            counter[c - 'a']--;
        }

        int ns = s.size(), np = p.size();
        for (int i = 0; i < np && i < ns; i++) {
            counter[s[i] - 'a']++;
        }

        vector<int> result;
        for (int i = np; i < ns; i++) {
            if (checkValid(counter)) result.push_back(i - np);
            counter[s[i] - 'a']++;
            counter[s[i - np] - 'a']--;
        }
        if (checkValid(counter)) result.push_back(ns - np);
        return result;
    }
};
```

**为了避免每一次滑动窗口都重新比较正确性，可以追踪数量匹配的字母数，并在滑动窗口时更新这个数值。**

### 350. 两个数组的交集 II

计数器

```cpp
class Solution {
public:
    vector<int> intersect(vector<int>& nums1, vector<int>& nums2) {
        if (nums1.size() > nums2.size())
            return intersect(nums2, nums1);

        unordered_map<int, int> counter;
        for (const int n : nums1) counter[n]++;

        vector<int> result;
        for (const int n : nums2) {
            if (counter[n] > 0) {
                result.push_back(n);
                counter[n]--;
            }
        }

        return result;
    }
};
```
