---
title: Chapter 5 Stacks and Queues
weight: 5
type: docs
---

## C++ 接口：stack

- 初始化空栈；
- 元素访问：
    - `top()`。
- 容量信息：
    - `empty()`；
    - `size()`。
- 修改元素：
    - `push(v)`；
    - `pop()`。

## C++ 接口：queue

- 初始化空队列；
- 元素访问：
    - `front()`；
    - `back()`。
- 容量信息：
    - `empty()`；
    - `size()`。
- 修改元素：
    - `push(v)`；
    - `pop()`。

## C++ 接口：deque

- 初始化空队列；
- 元素访问：
    - `front()`；
    - `back()`。
- 容量信息：
    - `empty()`；
    - `size()`。
- 修改元素：
    - `clear()`；
    - `push_back(v)`；
    - `pop_back()`。
    - `push_front(v)`；
    - `pop_front()`。

## C++ 接口：priority_queue

- 初始化空优先队列：
    - 默认大顶堆
    - `priority_queue<int, vector<int>, greater<int>> q` 小顶堆
- 元素访问：
    - `top()`。
- 容量信息：
    - `empty()`；
    - `size()`。
- 修改元素：
    - `push(v)`；
    - `pop()`。

## 基本题目

### 232. 用栈实现队列

基本操作

```cpp
class MyQueue {
public:
    stack<int> st1, st2;
    
    void push(int x) {
        st1.push(x);
    }

    void dump() {
        while (!st1.empty()) {
            st2.push(st1.top());
            st1.pop();
        }
    }
    
    int pop() {
        if (st2.empty()) dump();
        int result = st2.top();
        st2.pop();
        return result;
    }
    
    int peek() {
        if (st2.empty()) dump();
        return st2.top();
    }
    
    bool empty() {
        return st1.empty() && st2.empty();
    }
};
```

### 225. 用队列实现栈

基本操作

```cpp
class MyStack {
public:
    queue<int> q;

    void push(int x) {
        int size = q.size();
        q.push(x);
        for (int i = 0; i < size; i++) {
            q.push(q.front());
            q.pop();
        }
    }
    
    int pop() {
        int result = q.front();
        q.pop();
        return result;
    }
    
    int top() {
        return q.front();
    }
    
    bool empty() {
        return q.empty();
    }
};
```

### 20. 有效的括号

表达式与操作符栈

```cpp
class Solution {
public:
    bool isValid(string s) {
        stack<char> st;
        for (const char c : s) {
            if (c == '(' || c == '[' || c == '{') st.push(c);
            else if (!st.empty()) {
                char last = st.top();
                st.pop();
                if ((last == '(' && c != ')') || (last == '[' && c != ']') || (last == '{' && c != '}')) return false;
            }
            else return false;
        }
        return st.empty();
    }
};
```

### 1047. 删除字符串中的所有相邻重复项

栈与递归操作之间的关系

```cpp
class Solution {
public:
    string removeDuplicates(string s) {
        char *buf = new char[s.size() + 1];
        int i = 0;
        for (const char c : s) {
            if (i > 0 && buf[i - 1] == c) i--;
            else buf[i++] = c;
        }
        buf[i] = '\0';

        string result(buf);
        delete[] buf;
        return result;
    }
};
```

### 150. 逆波兰表达式求值

表达式求值的操作数栈

```cpp
class Solution {
public:
    bool isOperator(const string &token) {...}
    int calculate(int op1, int op2, char token) {...}

    int evalRPN(vector<string>& tokens) {
        stack<int> operands;
        for (const string &token : tokens) {
            if (isOperator(token)) {
                int op1, op2;
                op2 = operands.top();
                operands.pop();
                op1 = operands.top();
                operands.pop();
                operands.push(calculate(op1, op2, token[0]));
            }
            else {
                operands.push(stoi(token));
            }
        }
        return operands.top();
    }
};
```

### 239. 滑动窗口最大值

单调栈（双端队列）

```cpp
class Solution {
public:
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        deque<int> dq;
        vector<int> result(nums.size() - k + 1);
        int n = nums.size();
        for (int i = 0; i < n; i++) {
            // Push nums[i]
            while (!dq.empty() && dq.back() < nums[i]) dq.pop_back();
            dq.push_back(nums[i]);

            // Pop nums[i - k]
            if (i >= k && dq.front() == nums[i - k]) dq.pop_front();

            // Add maximum
            if (i >= k - 1)
            result[i - k + 1] = dq.front();
        }
        return result;
    }
};
```
``
### 347. 前 K 个高频元素

找到数组中最大的 k 个元素：

- **规模为 k 的小顶堆**
- **快速切分**

```cpp
class Solution {
public:
    int partition(vector<pair<int, int>> &v, int lo, int hi) {
        int i = lo + 1, j = lo + 1;
        int pivot = v[lo].second;
        while (j < hi) {
            if (v[j].second > pivot) swap(v[i++], v[j++]);
            else j++;
        }
        swap(v[lo], v[i - 1]);
        return i - 1;
    }

    vector<int> topKFrequent(vector<int>& nums, int k) {
        // Preprocessing: frequency count
        unordered_map<int, int> counter;
        for (const int n : nums) counter[n]++;
        vector<pair<int, int>> frequency;
        for (const auto &[k, v] : counter) frequency.push_back({k, v});

        // Finding the largest k elements
        int lo = 0, hi = frequency.size(), mi;
        while (lo < hi) {
            mi = partition(frequency, lo, hi);
            if (mi > k) hi = mi;
            else if (mi < k - 1) lo = mi + 1;
            else break;
        }

        // Output
        vector<int> result;
        for (int i = 0; i < k; i++) {
            result.push_back(frequency[i].first);
        }
        return result;
    }
};
```
