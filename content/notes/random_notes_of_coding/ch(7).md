---
title: Chapter 7 Binary Trees (2)
weight: 7
type: docs
---

## 基本题目

### 226. 翻转二叉树

递归

```cpp
class Solution {
public:
    TreeNode* invertTree(TreeNode* root) {
        if (!root) return root;
        TreeNode *tmp = invertTree(root->left);
        root->left = invertTree(root->right);
        root->right = tmp;
        return root;
    }
};
```

### 101. 对称二叉树

递归

```cpp
class Solution {
public:
    bool isSymmetric(TreeNode *root1, TreeNode *root2) {
        if (!root1 && !root2) return true;
        if (!root1 || !root2 || root1->val != root2->val) return false;
        return isSymmetric(root1->left, root2->right) && isSymmetric(root1->right, root2->left);
    }

    bool isSymmetric(TreeNode* root) {
        if (!root) return true;
        return isSymmetric(root->left, root->right);
    }
};
```

**迭代：可以使用队列来实现逐层遍历，并通过控制入队顺序来实现功能**

```cpp
class Solution {
public:
    bool isSymmetric(TreeNode* root) {
        if (!root) return true;
        queue<TreeNode *> q;
        q.push(root->left);
        q.push(root->right);

        TreeNode *tmp1, *tmp2;
        while (q.size() >= 2) {
            tmp1 = q.front();
            q.pop();
            tmp2 = q.front();
            q.pop();
            if (!tmp1 && !tmp2) continue;
            else if (!tmp1 || !tmp2 || tmp1->val != tmp2->val) return false;
            q.push(tmp1->left);
            q.push(tmp2->right);
            q.push(tmp1->right);
            q.push(tmp2->left);
        }

        return q.empty();
    }
};
```
### 222. 完全二叉树的节点个数

递归

```cpp
class Solution {
public:
    int countNodes(TreeNode* root) {
        if (!root) return 0;
        return countNodes(root->left) + countNodes(root->right) + 1;
    }
};
```

**二分查找+节点总数的二进制表示与路径的关系**

```cpp
class Solution {
public:
    bool exists(TreeNode *root, int height, int size) {
        int mask = 1 << (height - 2);
        while (mask > 0) {
            root = (size & mask) ? root->right : root->left;
            mask >>= 1;
        }
        return root;
    }

    int countNodes(TreeNode* root) {
        if (!root) return 0;
        int height = 0;
        TreeNode *tmp = root;
        while (tmp) {
            height++;
            tmp = tmp->left;
        }

        int lo = 1 << (height - 1), hi = (1 << height) - 1, mi;
        while (lo < hi) {
            mi = (lo + hi + 1) / 2;
            if (exists(root, height, mi)) lo = mi;
            else hi = mi - 1;
        }
        return lo;
    }
};
```

### 110. 平衡二叉树

递归+传递高度信息

```cpp
class Solution {
public:
    int balancedHeight(TreeNode *root) {
        if (!root) return 0;
        int h1, h2;
        if ((h1 = balancedHeight(root->left)) < 0 || (h2 = balancedHeight(root->right)) < 0) return -1;
        if (abs(h1 - h2) > 1) return -1;
        return max(h1, h2) + 1;
    }

    bool isBalanced(TreeNode* root) {
        return balancedHeight(root) >= 0;
    }
};
```

### 257. 二叉树的所有路径

DFS 递归/栈

```cpp
class Solution {
public:
    void getPaths(vector<string> &result, string path, TreeNode *node) {
        path += to_string(node->val);
        if (!node->left && !node->right) result.push_back(path);
        else {
            if (node->left) getPaths(result, path + "->", node->left);
            if (node->right) getPaths(result, path + "->", node->right);
        }
    }

    vector<string> binaryTreePaths(TreeNode* root) {
        vector<string> result;
        getPaths(result, "", root);
        return result;
    }
};
```

```cpp
class Solution {
public:
    vector<string> binaryTreePaths(TreeNode* root) {
        vector<string> result;
        stack<string> paths;
        stack<TreeNode *> st;
        paths.push("");
        st.push(root);

        TreeNode *tmp;
        string prev;
        while (!st.empty()) {
            tmp = st.top();
            st.pop();
            prev = paths.top();
            paths.pop();

            prev += ("->" + to_string(tmp->val));
            if (tmp->right) {
                st.push(tmp->right);
                paths.push(prev);
            }
            if (tmp->left) {
                st.push(tmp->left);
                paths.push(prev);
            }

            if (!tmp->left && !tmp->right) {
                result.push_back(prev.substr(2));
            }
        }
        return result;
    }
};
```

### 404. 左叶子之和

DFS 递归

```cpp
class Solution {
public:
    int sumOfLeftLeaves(TreeNode *root, bool leftChild) {
        if (!root) return 0;
        if (!root->left && !root->right) return leftChild ? root->val : 0;
        return sumOfLeftLeaves(root->left, true) + sumOfLeftLeaves(root->right, false);
    }

    int sumOfLeftLeaves(TreeNode* root) {
        return sumOfLeftLeaves(root, false);
    }
};
```

### 513. 找树左下角的值

DFS 递归/BFS 迭代

```cpp
class Solution {
public:
    int findBottomLeftValue(TreeNode* root) {
        int result;
        queue<TreeNode *> q;
        q.push(root);
        TreeNode *tmp;

        while (!q.empty()) {
            result = q.front()->val;
            int n = q.size();
            for (int i = 0; i < n; i++) {
                tmp = q.front();
                q.pop();
                if (tmp->left) q.push(tmp->left);
                if (tmp->right) q.push(tmp->right);
            }
        }

        return result;
    }
};
```

**只需要找一个树节点，这个节点应当是 BFS 遍历（从右到左）的最后一个节点。无需逐层**

```cpp
class Solution {
public:
    int findBottomLeftValue(TreeNode* root) {
        int result;
        queue<TreeNode *> q;
        q.push(root);
        TreeNode *tmp;

        while (!q.empty()) {
            tmp = q.front();
            q.pop();
            if (!tmp) continue;
            q.push(tmp->right);
            q.push(tmp->left);
            result = tmp->val;
        }

        return result;
    }
};
```

### 112. 路径总和

DFS 递归/BFS 迭代

```cpp
class Solution {
public:
    bool hasPathSum(TreeNode* root, int targetSum) {
        if (!root) return false;
        if (!root->left && !root->right) return targetSum == root->val;
        return hasPathSum(root->left, targetSum - root->val) || hasPathSum(root->right, targetSum - root->val);
    }
};
```

### 106. 从中序与后序遍历序列构造二叉树

递归+数组元素位置的反向映射

```cpp
class Solution {
private:
    unordered_map<int, int> inIndex;

public:
    TreeNode *buildTree(vector<int>& inorder, vector<int>& postorder, int i, int j, int size) {
        if (!size) return nullptr;
        int val = postorder[j + size - 1];
        int index = inIndex[val];
        return new TreeNode(val, buildTree(inorder, postorder, i, j, index - i), 
                                 buildTree(inorder, postorder, index + 1, j + index - i, i + size - index - 1));
    }

    TreeNode* buildTree(vector<int>& inorder, vector<int>& postorder) {
        for (int i = 0; i < inorder.size(); i++) inIndex[inorder[i]] = i;
        return buildTree(inorder, postorder, 0, 0, inorder.size());
    }
};
```

**可以用巧妙的迭代+栈方法解答：** [力扣解答](https://leetcode.cn/problems/construct-binary-tree-from-inorder-and-postorder-traversal/solutions/426738/cong-zhong-xu-yu-hou-xu-bian-li-xu-lie-gou-zao-14/)

### 654. 最大二叉树

递归

```cpp
class Solution {
public:
    TreeNode* constructMaximumBinaryTree(vector<int>& nums, int lo, int hi) {
        if (hi <= lo) return nullptr;
        int maximum = INT_MIN, maxId = -1;
        for (int i = lo; i < hi; i++) {
            if (nums[i] > maximum) {
                maxId = i;
                maximum = nums[i];
            }
        }
        return new TreeNode(nums[maxId], constructMaximumBinaryTree(nums, lo, maxId), constructMaximumBinaryTree(nums, maxId + 1, hi));
    }

    TreeNode* constructMaximumBinaryTree(vector<int>& nums) {
        return constructMaximumBinaryTree(nums, 0, nums.size());
    }
};
```

**可以使用单调栈来避免多次查找最大元素：维护一个递减栈，每次出现比栈顶更大的元素，则将那些更小的元素弹出，作为新节点的左子树**

```cpp
class Solution {
public:
    TreeNode* constructMaximumBinaryTree(vector<int>& nums) {
        stack<TreeNode *> s;
        TreeNode *tmp;

        for (const int n : nums) {
            tmp = nullptr;
            while (!s.empty() && s.top()->val < n) {
                s.top()->right = tmp;
                tmp = s.top();
                s.pop();
            }
            tmp = new TreeNode(n, tmp, nullptr);
            s.push(tmp);
        }

        tmp = nullptr;
        while (!s.empty()) {
            s.top()->right = tmp;
            tmp = s.top();
            s.pop();
        }
        return tmp;
    }
};
```

### 617. 合并二叉树

DFS 递归/BFS 迭代

```cpp
class Solution {
public:
    TreeNode* mergeTrees(TreeNode* root1, TreeNode* root2) {
        if (!root1) return root2;
        if (!root2) return root1;
        root1->val += root2->val;
        root1->left = mergeTrees(root1->left, root2->left);
        root1->right = mergeTrees(root1->right, root2->right);
        return root1;
    }
};
```

## 拓展练习

### 100. 相同的树

递归

```cpp
class Solution {
public:
    bool isSameTree(TreeNode* p, TreeNode* q) {
        if (!p && !q) return true;
        else if (!p || !q || p->val != q->val) return false;
        return isSameTree(p->left, q->left) && isSameTree(p->right, q->right);
    }
};
```

### 572. 另一个树的子树

递归

```cpp
class Solution {
public:
    bool isEqual(TreeNode *node1, TreeNode *node2) {
        if (!node1 && !node2) return true;
        else if (!node1 || !node2 || node1->val != node2->val) return false;
        return isEqual(node1->left, node2->left) && isEqual(node1->right, node2->right);
    }

    bool isSubtree(TreeNode* root, TreeNode* subRoot) {
        if (!root) return false;
        if (isEqual(root, subRoot)) return true;
        return isSubtree(root->left, subRoot) || isSubtree(root->right, subRoot);
    }
};
```

**如果在树遍历的过程中加入 NULL 节点，那么前序遍历的结果与树结构一一对应。后序遍历的结果也满足这个性质，中序遍历不满足**

```cpp
class Solution {
public:
    void preorderTraverse(vector<int> &v, TreeNode *node) {
        if (!node) {
            v.push_back(INT_MIN);
            return;
        }
        v.push_back(node->val);
        preorderTraverse(v, node->left);
        preorderTraverse(v, node->right);
    }

    void constructNext(vector<int> &next, const vector<int> &pattern) {
        int m = pattern.size();
        for (int i = 1; i < m; i++) {
            int tmp = next[i - 1];
            while (tmp > 0 && pattern[i] != pattern[tmp]) tmp = next[tmp - 1];
            if (pattern[i] == pattern[tmp]) next[i] = tmp + 1;
            else next[i] = 0;
        }
    }

    bool kmpMatch(const vector<int> &source, const vector<int> &pattern) {
        int n = source.size(), m = pattern.size();
        vector<int> next(m, 0);
        constructNext(next, pattern);
        int i = 0, j = 0;
        while (i < n && j < m) {
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
        return j == m;
    }

    bool isSubtree(TreeNode* root, TreeNode* subRoot) {
        vector<int> preorder1, preorder2;
        preorderTraverse(preorder1, root);
        preorderTraverse(preorder2, subRoot);
        return kmpMatch(preorder1, preorder2);
    }
};
```

### 113. 路径总和 II

DFS 递归+路径回溯

```cpp
class Solution {
public:
    vector<vector<int>> pathSum(TreeNode* root, int targetSum) {
        vector<vector<int>> result;
        vector<int> path;
        pathSumFinder(result, path, root, targetSum);
        return result;
    }

    void pathSumFinder(vector<vector<int>> &result, vector<int> &path, TreeNode *root, int target) {
        if (!root) return;
        path.push_back(root->val);
        target -= root->val;

        if (!root->left && !root->right) {
            if (target == 0) result.push_back(path);
            path.pop_back();
            return;
        }

        pathSumFinder(result, path, root->left, target);
        pathSumFinder(result, path, root->right, target);
        path.pop_back();
    }
};
```

**除了路径回溯，还可以使用哈希表的方式存储逆序路径，用于路径输出**

### 105. 从前序与中序遍历序列构造二叉树

递归+数组元素位置的反向映射。与 106 类似，可以用特殊的迭代方式完成

```cpp
class Solution {
private:
    unordered_map<int, int> inIndex;

public:
    TreeNode *buildTree(vector<int>& preorder, vector<int>& inorder, int i, int j, int size) {
        if (!size) return nullptr;
        int val = preorder[i];
        int index = inIndex[val];
        return new TreeNode(val, buildTree(preorder, inorder, i + 1, j, index - j), 
                                 buildTree(preorder, inorder, i + index + 1 - j, index + 1, j + size - index - 1));
    }

    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        for (int j = 0; j < inorder.size(); j++) inIndex[inorder[j]] = j;
        return buildTree(preorder, inorder, 0, 0, inorder.size());
    }
};
```
