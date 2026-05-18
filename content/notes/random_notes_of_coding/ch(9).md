---
title: Chapter 9 Backtracking
weight: 9
type: docs
---

## 基本题目

### 77. 组合

当前路径记录+回溯+递归

```cpp
class Solution {
public:
    void combine(vector<vector<int>> &result, vector<int> &curr, int n, int k) {
        if (k == 0) {
            result.push_back(curr);
            return;
        }
        if (k > n) return;

        // n is not selected
        combine(result, curr, n - 1, k);

        // n is selected
        curr.push_back(n);
        combine(result, curr, n - 1, k - 1);
        curr.pop_back();
    }

    vector<vector<int>> combine(int n, int k) {
        vector<vector<int>> result;
        vector<int> curr;
        combine(result, curr, n, k);
        return result;
    }
};
```

**可以将问题等价转换为二进制位的置位问题：n 位二进制数位中选 k 个置为 1**

- 可以使用一个整数来表示（每次使用运算的方式得到下一个数）
- 也可以朴素地使用一个数组来表示， [使用索引值的改变来实现目标](https://leetcode.cn/problems/combinations/solutions/405094/zu-he-by-leetcode-solution/)

### 216. 组合总和 III

回溯+递归+剪枝

```cpp
class Solution {
public:
    void combinationSum3(vector<vector<int>> &result, vector<int> curr, int k, int n, int i) {
        if (k == 0 && n == 0) {
            result.push_back(curr);
            return;
        }
        if (i == 0 || k > i) return;
        if (n < (k + 1) * k / 2 || n > (2 * i + 1 - k) * k / 2) return;

        combinationSum3(result, curr, k, n, i - 1);
        curr.push_back(i);
        combinationSum3(result, curr, k - 1, n - i, i - 1);
        curr.pop_back();
    }

    vector<vector<int>> combinationSum3(int k, int n) {
        vector<vector<int>> result;
        vector<int> curr;
        combinationSum3(result, curr, k, n, 9);
        return result;
    }
};
```

### 17. 电话号码的字母组合

回溯+递归

```cpp
class Solution {
private:
    unordered_map<char, string> dict = {
        {'2', "abc"},
        {'3', "def"},
        {'4', "ghi"},
        {'5', "jkl"},
        {'6', "mno"},
        {'7', "pqrs"},
        {'8', "tuv"},
        {'9', "wxyz"}
    };

public:
    void combine(vector<string> &result, string curr, const string &digits, int i) {
        if (i == digits.size()) {
            result.push_back(curr);
            return;
        }
        for (const char c : dict[digits[i]]) {
            combine(result, curr + c, digits, i + 1);
        }
    }

    vector<string> letterCombinations(string digits) {
        if (!digits.size()) return {};
        vector<string> result;
        combine(result, "", digits, 0);
        return result;
    }
};
```

### 39. 组合总和

回溯+递归+剪枝

```cpp
class Solution {
public:
    void combine(vector<vector<int>> &result, vector<int>& hist, vector<int>& candidates, int i, int target) {
        if (i >= candidates.size()) return;
        if (target == 0) {
            result.push_back(hist);
            return;
        }
        if (target < candidates[i]) return;

        combine(result, hist, candidates, i + 1, target);
        hist.push_back(candidates[i]);
        combine(result, hist, candidates, i, target - candidates[i]);
        hist.pop_back();
    }

    vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
        sort(candidates.begin(), candidates.end());
        vector<vector<int>> result;
        vector<int> hist;
        combine(result, hist, candidates, 0, target);
        return result;
    }
};
```

### 40. 组合总和 II

回溯+递归+剪枝+去重

```cpp
class Solution {
public:
    void combine(vector<vector<int>> &result, vector<int> &hist, vector<int> &candidates, int i, int target) {
        if (target == 0) {
            result.push_back(hist);
            return;
        }
        if (i >= candidates.size()) return;
        if (target < candidates[i]) return;

        hist.push_back(candidates[i]);
        combine(result, hist, candidates, i + 1, target - candidates[i]);
        hist.pop_back();

        int prev = candidates[i];
        while (i < candidates.size() && candidates[i] == prev) i++;
        combine(result, hist, candidates, i, target);
    }

    vector<vector<int>> combinationSum2(vector<int>& candidates, int target) {
        sort(candidates.begin(), candidates.end());
        vector<vector<int>> result;
        vector<int> hist;
        combine(result, hist, candidates, 0, target);
        return result;
    }
};
```
### 131. 分割回文串

回溯+递归

```cpp
class Solution {
public:
    bool isPalindrome(const string &s, int i, int j) {
        while (i < --j) if (s[i++] != s[j]) return false;
        return true;
    }

    void partition(vector<vector<string>> &result, vector<string> &hist, const string &s, int i) {
        if (i >= s.size()) {
            result.push_back(hist);
            return;
        }

        for (int j = i + 1; j <= s.size(); j++) {
            if (isPalindrome(s, i, j)) {
                hist.push_back(s.substr(i, j - i));
                partition(result, hist, s, j);
                hist.pop_back();
            }
        }
    }

    vector<vector<string>> partition(string s) {
        vector<vector<string>> result;
        vector<string> hist;
        partition(result, hist, s, 0);
        return result;
    }
};
```

**可以使用动态规划/memoization 来避免回文串的重复判断**

```cpp
class Solution {
public:
    vector<vector<bool>> palindrome;

    void initPalindrome(const string &s) {
        palindrome = vector<vector<bool>>(s.size(), vector<bool>(s.size()));
        for (int i = 0; i < s.size(); i++) palindrome[i][i] = true;
        for (int j = 1; j < s.size(); j++) {
            for (int i = 0; i < j; i++) {
                palindrome[i][j] = s[i] == s[j] && (j == i + 1 || palindrome[i + 1][j - 1]);
            }
        }
    }

    void partition(vector<vector<string>> &result, vector<string> &hist, const string &s, int i) {
        if (i >= s.size()) {
            result.push_back(hist);
            return;
        }

        for (int j = i; j < s.size(); j++) {
            if (palindrome[i][j]) {
                hist.push_back(s.substr(i, j - i + 1));
                partition(result, hist, s, j + 1);
                hist.pop_back();
            }
        }
    }

    vector<vector<string>> partition(string s) {
        initPalindrome(s);
        vector<vector<string>> result;
        vector<string> hist;
        partition(result, hist, s, 0);
        return result;
    }
};
```

### 93. 复原 IP 地址

回溯+递归+剪枝

```cpp
class Solution {
public:
    void restore(vector<string> &result, string curr, const string &s, int i, int splits) {
        if (splits == 4 && i >= s.size()) {
            result.push_back(curr);
            return;
        }
        if (splits >= 4 || i >= s.size()) return;
        if (s.size() - i > (4 - splits) * 3 || s.size() - i < 4 - splits) return;
        
        if (curr.size() > 0) curr += '.';

        if (s[i] == '0') {
            restore(result, curr + '0', s, i + 1, splits + 1);
            return;
        }
        for (int j = i + 1; j < i + 4 && j <= s.size(); j++) {
            string ss = s.substr(i, j - i);
            if (stoi(ss) <= 255) restore(result, curr + ss, s, j, splits + 1);
        }
    }

    vector<string> restoreIpAddresses(string s) {
        vector<string> result;
        restore(result, "", s, 0, 0);
        return result;
    }
};
```

### 78. 子集

回溯+递归

```cpp
class Solution {
public:
    void subsets(vector<vector<int>> &result, vector<int> &curr, const vector<int> &nums, int i) {
        if (i >= nums.size()) {
            result.push_back(curr);
            return;
        }

        subsets(result, curr, nums, i + 1);
        curr.push_back(nums[i]);
        subsets(result, curr, nums, i + 1);
        curr.pop_back();
    }

    vector<vector<int>> subsets(vector<int>& nums) {
        vector<vector<int>> result;
        vector<int> curr;
        subsets(result, curr, nums, 0);
        return result;
    }
}
```

二进制数位表示

```cpp
class Solution {
public:
    vector<vector<int>> subsets(vector<int>& nums) {
        vector<vector<int>> result;
        int n = nums.size();
        for (int m = 0; m < (1 << n); m++) {
            vector<int> curr;
            int tmp = m;
            for (int i = 0; i < n; i++) {
                if (tmp & 1) curr.push_back(nums[i]);
                tmp >>= 1;
            }
            result.push_back(curr);
        }
        return result;
    }
};
```

### 90. 子集 II

回溯+递归+去重

```cpp
class Solution {
public:
    void subsets(vector<vector<int>> &result, vector<int> &curr, const vector<int> &nums, int i) {
        if (i >= nums.size()) {
            result.push_back(curr);
            return;
        }

        curr.push_back(nums[i]);
        subsets(result, curr, nums, i + 1);
        curr.pop_back();

        int prev = nums[i];
        while (i < nums.size() && nums[i] == prev) i++;
        subsets(result, curr, nums, i);
    }

    vector<vector<int>> subsetsWithDup(vector<int>& nums) {
        sort(nums.begin(), nums.end());
        vector<vector<int>> result;
        vector<int> curr;
        subsets(result, curr, nums, 0);
        return result;
    }
};
```

**去重的两种方法：**

- **主动查找**：如果不选元素 `nums[i]`，那么会在递归时跳过之后与它相等的元素；
- **被动判断**：如果选择了 `nums[i-1]`，且 `nums[i-1] == nums[i]`，那么必须选择 `nums[i]`（连续相同元素取靠后的）。

### 491. 递增子序列

回溯+递归，使用集合来去重（类似于主动查找）

```cpp
class Solution {
public:
    void find(vector<vector<int>> &result, const vector<int> &nums, vector<int> &path, int i) {
        // Ends here
        int n = nums.size();
        if (path.size() > 1) result.push_back(path);

        // Exceed limits
        if (++i >= nums.size()) return;

        // Find another non-decreasing element
        int prev = path.size() ? path.back() : INT_MIN;
        unordered_set<int> found;
        while (i < n) {
            if (nums[i] >= prev && !found.count(nums[i])) {
                found.insert(nums[i]);
                path.push_back(nums[i]);
                find(result, nums, path, i);
                path.pop_back();
            }
            i++;
        }
    }

    vector<vector<int>> findSubsequences(vector<int>& nums) {
        vector<vector<int>> result;
        vector<int> path;
        find(result, nums, path, -1);
        return result;
    }
};
```

**回溯的两种思路**：

- 每次考察结果序列中的下一元素，判断要如何选择。选择即立刻输出结果；
- 每次考察原始序列中的下一元素，判断要不要添加到结果中。当遍历完原始序列后输出结果；

由于 path 中的路径是有序的，因此相等的元素一定连续出现。对于上述的第二种思路，如果遇到与上一元素相等，这表示中间的元素都被跳过，因此如果这两个元素只需要选择一个，可以规定一定会选择后面的元素，也即：相等元素的选择由起始位置决定，起始位置之后的都会被选择。**这也是第二种去重思路：如果选择了首个相等元素，后面的相等元素也必须被选择。**

```cpp
class Solution {
public:
    void find(vector<vector<int>> &result, const vector<int> &nums, vector<int> &path, int i) {
        if (i >= nums.size()) {
            if (path.size() >= 2) {
                result.push_back(path);
            }
            return;
        }

        int prev = path.size() ? path.back() : INT_MIN;
        if (nums[i] >= prev) {
            // Selected
            path.push_back(nums[i]);
            find(result, nums, path, i + 1);
            path.pop_back();
        }

        if (nums[i] != prev) {
            // Skipped
            find(result, nums, path, i + 1);
        }
    }

    vector<vector<int>> findSubsequences(vector<int>& nums) {
        vector<vector<int>> result;
        vector<int> path;
        find(result, nums, path, 0);
        return result;
    }
};
```
### 46. 全排列

回溯+递归

```cpp
class Solution {
public:
    void permute(vector<vector<int>> &result, vector<int> &path, vector<int> &nums, vector<bool> &selected) {
        bool flag = false;
        for (int i = 0; i < nums.size(); i++) {
            if (!selected[i]) {
                flag = true;
                path.push_back(nums[i]);
                selected[i] = true;
                permute(result, path, nums, selected);
                selected[i] = false;
                path.pop_back();
            }
        }

        if (!flag) {
            result.push_back(path);
        }
    }

    vector<vector<int>> permute(vector<int>& nums) {
        vector<vector<int>> result;
        vector<bool> selected(nums.size());
        vector<int> path;
        permute(result, path, nums, selected);
        return result;
    }
};
```

可以使用原地交换元素的方式来避免使用记录数组：每次选择元素时将它交换到前面

### 47. 全排列 II

回溯+递归+排序去重

```cpp
class Solution {
public:
    void permute(vector<vector<int>> &result, vector<int> &path, vector<int> &nums, vector<bool> &selected) {
        int prev = INT_MIN;
        for (int i = 0; i < nums.size(); i++) {
            if (!selected[i] && nums[i] != prev) {
                prev = nums[i];
                path.push_back(nums[i]);
                selected[i] = true;
                permute(result, path, nums, selected);
                selected[i] = false;
                path.pop_back();
            }
        }

        if (prev == INT_MIN) {
            result.push_back(path);
        }
    }

    vector<vector<int>> permuteUnique(vector<int>& nums) {
        vector<vector<int>> result;
        vector<bool> selected(nums.size());
        vector<int> path;
        sort(nums.begin(), nums.end());
        permute(result, path, nums, selected);
        return result;
    }
};
```

### 332. 重新安排行程

回溯+递归+排序去重

```cpp
class Solution {
public:
    bool find(vector<string> &path, unordered_map<string, vector<pair<string, bool>>> &tickets, int total) {
        if (!total) return true;

        vector<pair<string, bool>> &v = tickets[path.back()];
        string prev = "";
        for (int i = 0; i < v.size(); i++) {
            if (!v[i].second && v[i].first != prev) {
                prev = v[i].first;
                v[i].second = true;
                path.push_back(v[i].first);
                if (find(path, tickets, total - 1)) return true;
                path.pop_back();
                v[i].second = false;
            }
        }
        return false;
    }

    vector<string> findItinerary(vector<vector<string>>& tickets) {
        unordered_map<string, vector<pair<string, bool>>> map;
        for (const vector<string> &t : tickets) {
            map[t[0]].push_back({t[1], false});
        }
        for (auto &[k, v] : map) {
            sort(v.begin(), v.end());
        }
        
        vector<string> path = {"JFK"};
        find(path, map, tickets.size());
        return path;
    }
};
```

出现大量重复元素时，可以在记录数组中使用 int 来替代 bool，以此来更方便地去重。不过这也会使得排序等过程复杂化，需要使用有序 map

**本题是在半欧拉图中寻找欧拉通路，这可以借助 [Hierholzer 算法](https://leetcode.cn/problems/reconstruct-itinerary/solutions/389885/zhong-xin-an-pai-xing-cheng-by-leetcode-solution/) 实现**

### 51. N 皇后

回溯+递归+剪枝

```cpp
class Solution {
private:
    vector<bool> columns;
    vector<bool> diag1, diag2;

public:
    string encodeRow(int i, int n) {
        char *s = new char[n + 1];
        for (int k = 0; k < i; k++) s[k] = '.';
        s[i] = 'Q';
        for (int k = i + 1; k < n; k++) s[k] = '.';
        s[n] = '\0';
        return string(s);
    }

    void solve(vector<vector<string>> &result, vector<string> &path, int i, int n) {
        if (i < 0) {
            result.push_back(path);
            return;
        }

        for (int j = 0; j < n; j++) {
            if (!columns[j] && !diag1[i + j] && !diag2[i - j + n - 1]) {
                columns[j] = true;
                diag1[i + j] = true;
                diag2[i - j + n - 1] = true;
                path.push_back(encodeRow(j, n));
                solve(result, path, i - 1, n);
                path.pop_back();
                columns[j] = false;
                diag1[i + j] = false;
                diag2[i - j + n - 1] = false;
            }
        }
    }

    vector<vector<string>> solveNQueens(int n) {
        columns.resize(n);
        diag1.resize(2 * n - 1);
        diag2.resize(2 * n - 1);

        vector<vector<string>> result;
        vector<string> path;
        solve(result, path, n - 1, n);
        return result;
    }
};
```

**可以使用整数的二进制位来替代上面的记录数组：使用整数 `columns`, `diag1`, `diag2` 表示，每次移动到下一行时，将 `diag1` 和 `diag2` 分别左移、右移一位，这样就可以把各位置对齐。对三个数进行与/或操作即可得到可以放置的位置。**

```cpp
class Solution {
public:
    string encodeRow(int i, int n) {
        char *s = new char[n + 1];
        for (int k = 0; k < i; k++) s[k] = '.';
        s[i] = 'Q';
        for (int k = i + 1; k < n; k++) s[k] = '.';
        s[n] = '\0';
        return string(s);
    }

    void solve(vector<vector<string>> &result, vector<string> &path, int i, int n, int columns, int diag1, int diag2) {
        if (i < 0) {
            result.push_back(path);
            return;
        }

        int mask = 1;
        int available = ~(columns | diag1 | diag2);
        for (int j = 0; j < n; j++) {
            if (mask & available) {
                path.push_back(encodeRow(j, n));
                solve(result, path, i - 1, n, columns | mask, (diag1 | mask) << 1, (diag2 | mask) >> 1);
                path.pop_back();
            }
            mask <<= 1;
        }
    }

    vector<vector<string>> solveNQueens(int n) {
        vector<vector<string>> result;
        vector<string> path;
        solve(result, path, n - 1, n, 0, 0, 0);
        return result;
    }
};
```

### 37. 解数独

回溯+递归+剪枝

```cpp
class Solution {
private:
    vector<vector<bool>> rowCount, colCount, boxCount;

public:
    Solution() : rowCount(9, vector<bool>(9)), colCount(9, vector<bool>(9)), boxCount(9, vector<bool>(9)) {}
    inline int box(int i, int j) { return i / 3 * 3 + j / 3; }

    bool solve(vector<vector<char>>& board, int i, int j) {
        // Find the next empty spot
        while (i < 9) {
            if (j >= 9) {
                i++;
                j = 0;
            }
            else if (board[i][j] != '.') j++;
            else break;
        }

        // End
        if (i >= 9) return true;

        // Try to fill in a number
        for (int v = 0; v < 9; v++) {
            if (!rowCount[i][v] && !colCount[j][v] && !boxCount[box(i, j)][v]) {
                rowCount[i][v] = true;
                colCount[j][v] = true;
                boxCount[box(i, j)][v] = true;
                board[i][j] = v + '1';
                if (solve(board, i, j + 1)) return true;
                board[i][j] = '.';
                rowCount[i][v] = false;
                colCount[j][v] = false;
                boxCount[box(i, j)][v] = false;
            }
        }
        return false;
    }

    void solveSudoku(vector<vector<char>>& board) {
        // Load input
        int value;
        for (int i = 0; i < 9; i++) {
            for (int j = 0; j < 9; j++) {
                if (board[i][j] != '.') {
                    value = board[i][j] - '1';
                    rowCount[i][value] = true;
                    colCount[j][value] = true;
                    boxCount[box(i, j)][value] = true;
                }
            }
        }

        // Solve recursively
        solve(board, 0, 0);
    }
};
```
