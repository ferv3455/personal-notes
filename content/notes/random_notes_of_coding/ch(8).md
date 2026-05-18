---
title: Chapter 8 Binary Trees (3)
weight: 8
type: docs
---

## 基本题目

### 700. 二叉搜索树中的搜索

二叉搜索树的定义

```cpp
class Solution {
public:
    TreeNode* searchBST(TreeNode* root, int val) {
        if (!root) return nullptr;
        else if (root->val > val) return searchBST(root->left, val);
        else if (root->val < val) return searchBST(root->right, val);
        else return root;
    }
};
```

### 98. 验证二叉搜索树

二叉搜索树的定义

```cpp
class Solution {
public:
    bool check(TreeNode *root, int &min, int &max) {
        if (!root) return false;
        min = root->val;
        max = root->val;

        int minT, maxT;
        if (root->left) {
            if (!check(root->left, minT, maxT)) return false;
            if (maxT >= root->val) return false;
            min = minT;
        }
        if (root->right) {
            if (!check(root->right, minT, maxT)) return false;
            if (minT <= root->val) return false;
            max = maxT;
        }
        return true;
    }

    bool isValidBST(TreeNode* root) {
        int min, max;
        return check(root, min, max);
    }
};
```

**递归过程的参数传递可以自底向上（引用/返回值），也可以自顶向下（参数）**

**二叉搜索树：仅需验证中序遍历结果的有序性即可**

### 530. 二叉搜索树的最小绝对差

中序遍历的有序性

```cpp
class Solution {
public:
    int getMinimumDifference(TreeNode* root) {
        stack<TreeNode *> s;
        int prev = -1, diff = INT_MAX;
        TreeNode *tmp = root;

        while (!s.empty() || tmp) {
            while (tmp) {
                s.push(tmp);
                tmp = tmp->left;
            }
            tmp = s.top();
            s.pop();

            if (prev >= 0 && tmp->val - prev < diff)
                diff = tmp->val - prev;
            prev = tmp->val;

            tmp = tmp->right;
        }

        return diff;
    }
};
```

### 501. 二叉搜索树中的众数

中序遍历的有序性

```cpp
class Solution {
public:
    vector<int> findMode(TreeNode* root) {
        vector<int> result;
        int occ = 0;
        stack<TreeNode *> s;
        TreeNode *tmp = root;
        int curr = INT_MIN, count = 0;
        bool flag = true;
        while (!s.empty() || tmp || flag) {
            while (tmp) {
                s.push(tmp);
                tmp = tmp->left;
            }

            if (s.empty()) {
                flag = false;
            }
            else {
                tmp = s.top();
                s.pop();
                if (tmp->val == curr) {
                    count++;
                    tmp = tmp->right;
                    continue;
                }
            }

            if (count > occ) {
                result.clear();
                result.push_back(curr);
                occ = count;
            }
            else if (count == occ) {
                result.push_back(curr);
            }

            if (tmp) {
                curr = tmp->val;
                count = 1;
                tmp = tmp->right;
            }
        }

        return result;
    }
};
```

### 236. 二叉树的最近公共祖先

记录父节点+链表相交

```cpp
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        unordered_map<TreeNode *, TreeNode *> parent;
        parent[root] = nullptr;
        queue<TreeNode *> buffer;
        buffer.push(root);
        TreeNode *tmp, *tmp2;
        while (!buffer.empty()) {
            tmp = buffer.front();
            buffer.pop();
            if (tmp->left) {
                parent[tmp->left] = tmp;
                buffer.push(tmp->left);
            }
            if (tmp->right) {
                parent[tmp->right] = tmp;
                buffer.push(tmp->right);
            }
        }

        tmp = p;
        tmp2 = q;
        while (tmp != tmp2) {
            tmp = parent[tmp];
            tmp2 = parent[tmp2];
            if (!tmp) tmp = q;
            if (!tmp2) tmp2 = p;
        }
        return tmp;
    }
};
```

**可以使用递归的方式求解：**

- 符合要求的节点中，左右子树都包含 p 或 q，或其本身等于 p 或 q 且一个子树包含 p 或 q
- **利用返回值将两个节点向上传递，直到某个节点相遇（即 left, root, right 三者中包含 p 与 q），该节点即为所求**

```cpp
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        // Both p, q in root, left, right: return root
        // Either p or q: return p/q
        // Nonzero value other than p or q: return left/right
        // All nullptr: return nullptr

        if (!root || root == p || root == q) return root;
        TreeNode *left = lowestCommonAncestor(root->left, p, q);
        TreeNode *right = lowestCommonAncestor(root->right, p, q);
        if (left && right) return root;
        return left ? left : right;
    }
};
```

### 235. 二叉搜索树的最近公共祖先

利用二叉搜索树的特性：公共祖先的值一定位于两节点值之间

```cpp
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if (p->val > q->val) swap(p, q);
        while (true) {
            if (root->val > q->val) root = root->left;
            else if (root->val < p->val) root = root->right;
            else break;
        }
        return root;
    }
};
```

### 701. 二叉搜索树中的插入操作

二叉搜索树的定义

```cpp
class Solution {
public:
    TreeNode* insertIntoBST(TreeNode* root, int val) {
        if (!root) return new TreeNode(val);
        TreeNode *curr = root;
        while (true) {
            if (val < curr->val) {
                if (!curr->left) {
                    curr->left = new TreeNode(val);
                    return root;
                }
                else {
                    curr = curr->left;
                }
            }
            else {
                if (!curr->right) {
                    curr->right = new TreeNode(val);
                    return root;
                }
                else {
                    curr = curr->right;
                }
            }
        }
        return root;
    }
};
```

### 450. 删除二叉搜索树中的节点

分类讨论删除节点的位置

```cpp
class Solution {
public:
    TreeNode* deleteNode(TreeNode* root, int key) {
        TreeNode *prev = nullptr, *curr = root;
        while (curr && curr->val != key) {
            prev = curr;
            if (curr->val > key) curr = curr->left;
            else curr = curr->right;
        }

        if (!curr) return root;
        if (!curr->left || !curr->right) {
            // Leaf node / one child
            if (prev) {
                if (curr->val > prev->val) prev->right = curr->left ? curr->left : curr->right;
                else prev->left = curr->left ? curr->left : curr->right;
                return root;
            }
            else {
                return curr->left ? curr->left : curr->right;
            }
        }

        // Two children: delete the greatest key in left subtree
        TreeNode *tmp = curr->left;
        while (tmp->right) tmp = tmp->right;
        curr->val = tmp->val;
        curr->left = deleteNode(curr->left, tmp->val);
        return root;
    }
};
```

### 669. 修剪二叉搜索树

递归。树结构的修改可以通过使用返回值赋值来实现

```cpp
class Solution {
public:
    TreeNode *trimBST(TreeNode *node, int low, int high) {
        if (!node) return nullptr;
        if (node->val < low) return trimBST(node->right, low, high);
        else if (node->val > high) return trimBST(node->left, low, high);
        node->left = trimBST(node->left, low, high);
        node->right = trimBST(node->right, low, high);
        return node;
    }
};
```

### 108. 将有序数组转换为二叉搜索树

二分递归

```cpp
class Solution {
public:
    TreeNode* sortedArrayToBST(vector<int>& nums, int lo, int hi) {
        if (lo >= hi) return nullptr;
        int mi = (lo + hi) / 2;
        return new TreeNode(nums[mi], sortedArrayToBST(nums, lo, mi), sortedArrayToBST(nums, mi + 1, hi));
    }

    TreeNode* sortedArrayToBST(vector<int>& nums) {
        return sortedArrayToBST(nums, 0, nums.size());
    }
};
```

### 538. 把二叉搜索树转换为累加树

中序遍历（反向）

```cpp
class Solution {
public:
    TreeNode* convertBST(TreeNode* root) {
        stack<TreeNode *> s;
        TreeNode *tmp = root;
        int sum = 0;
        while (!s.empty() || tmp) {
            while (tmp) {
                s.push(tmp);
                tmp = tmp->right;
            }
            tmp = s.top();
            s.pop();
            sum += tmp->val;
            tmp->val = sum;
            tmp = tmp->left;
        }
        return root;
    }
};
```
