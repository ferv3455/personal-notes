---
title: Chapter 6 Binary Trees (1)
weight: 6
type: docs
---

## 基本题目

### 144. 二叉树的前序遍历

```cpp
class Solution {
public:
    void traverse(vector<int> &result, TreeNode *root) {
        if (!root) return;
        result.push_back(root->val);
        traverse(result, root->left);
        traverse(result, root->right);
    }
};
```

**前序遍历的其他实现方式：迭代法**

```cpp
class Solution {
public:
    vector<int> preorderTraversal(TreeNode* root) {
        vector<int> result;
        stack<TreeNode *> s;
        s.push(root);
        
        while (!s.empty()) {
            TreeNode *tmp = s.top();
            s.pop();
            if (!tmp) continue;

            result.push_back(tmp->val);
            s.push(tmp->right);
            s.push(tmp->left);
        }
        return result;
    }
};
```

### 145. 二叉树的后序遍历

```cpp
class Solution {
public:
    void traverse(vector<int> &result, TreeNode *root) {
        if (!root) return;
        traverse(result, root->left);
        traverse(result, root->right);
        result.push_back(root->val);
    }
};
```

**后序遍历的迭代实现：将前序遍历的结果颠倒**

```cpp
class Solution {
public:
    vector<int> postorderTraversal(TreeNode* root) {
        vector<int> result;
        stack<TreeNode *> s;
        s.push(root);
        
        while (!s.empty()) {
            TreeNode *tmp = s.top();
            s.pop();
            if (!tmp) continue;

            result.push_back(tmp->val);
            s.push(tmp->left);
            s.push(tmp->right);
        }
        reverse(result.begin(), result.end());
        return result;
    }
};
```

### 94. 二叉树的中序遍历

```cpp
class Solution {
public:
    void traverse(vector<int> &result, TreeNode *root) {
        if (!root) return;
        traverse(result, root->left);
        result.push_back(root->val);
        traverse(result, root->right);
    }
};
```

**中序遍历的迭代实现：**

```cpp
class Solution {
public:
    vector<int> inorderTraversal(TreeNode* root) {
        vector<int> result;
        stack<TreeNode *> s;
        TreeNode *tmp = root;

        while (tmp) {
            // Go down along the leftmost branch
            while (tmp) {
                s.push(tmp);
                tmp = tmp->left;
            }

            // Go up until a right subtree is found
            while (!s.empty()) {
                tmp = s.top();
                s.pop();
                result.push_back(tmp->val);
                tmp = tmp->right;
                if (tmp) break;
            }
        }
        // or:
        while (!s.empty() || tmp) {
            // Go down along the leftmost branch
            while (tmp) {
                s.push(tmp);
                tmp = tmp->left;
            }
            
            // Process
            tmp = s.top();
            s.pop();
            result.push_back(tmp->val);
            
            // Try to step right
            tmp = tmp->right;
        }

        return result;
    }
};
```

**三种遍历的通用迭代方式：标记栈中元素（父节点）**

```cpp
class Solution {
public:
    vector<int> inorderTraversal(TreeNode* root) {
        vector<int> result;
        stack<TreeNode *> s;
        if (root) s.push(root);

        while (!s.empty()) {
            TreeNode *tmp = s.top();
            s.pop();
            if (tmp) {
                // Normal node: go down
                if (tmp->right) s.push(tmp->right);
                s.push(tmp);
                s.push(nullptr);
                if (tmp->left) s.push(tmp->left);
            }
            else {
                // Special node: output
                tmp = s.top();
                s.pop();
                result.push_back(tmp->val);
            }
        }

        return result;
    }
};
```

**Morris 遍历**

<img src="../attachments/Pasted image 20240306103733.png" >

```cpp
class Solution {
public:
    vector<int> inorderTraversal(TreeNode* root) {
        vector<int> result;
        TreeNode *x = root, *pred;
        while (x) {
            if (!x->left) {
                result.push_back(x->val);
                x = x->right;
            }
            else {
                pred = x->left;
                while (pred->right && pred->right != x) pred = pred->right;
                if (!pred->right) {
                    pred->right = x;
                    x = x->left;
                }
                else {
                    pred->right = nullptr;
                    result.push_back(x->val);
                    x = x->right;
                }
            }
        }
        return result;
    }
};
```

### 102. 二叉树的层序遍历

DFS，使用递归函数的参数来传递层数信息

```cpp
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        vector<vector<int>> result;
        traverse(result, root, 0);
        return result;
    }

    void traverse(vector<vector<int>> &result, TreeNode *root, int depth) {
        if (!root) return;
        if (result.size() <= depth) result.push_back({});
        result[depth].push_back(root->val);
        traverse(result, root->left, depth + 1);
        traverse(result, root->right, depth + 1);
    }
};
```

**BFS，将层次遍历拆解为两层循环，显式表示各层**

```cpp
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        if (!root) return {};
        vector<vector<int>> result;
        queue<TreeNode *> q;
        int layerSize = 1;
        q.push(root);

        while (layerSize) {
            result.push_back({});
            for (int i = 0; i < layerSize; i++) {
                TreeNode *tmp = q.front();
                q.pop();
                result.back().push_back(tmp->val);
                if (tmp->left) q.push(tmp->left);
                if (tmp->right) q.push(tmp->right);
            }
            layerSize = q.size();
        }

        return result;
    }
};
```

## 拓展练习

### 107. 二叉树的层次遍历 II

将 102 题目中的结果反转即可

### 199. 二叉树的右视图

只需存储每一层的最右边元素

```cpp
class Solution {
public:
    vector<int> rightSideView(TreeNode* root) {
        if (!root) return {};
        vector<int> result;
        queue<TreeNode *> q;
        int layerSize = 1;
        q.push(root);

        while (layerSize) {
            TreeNode *tmp;
            for (int i = layerSize; i > 0; i--) {
                tmp = q.front();
                q.pop();
                if (tmp->left) q.push(tmp->left);
                if (tmp->right) q.push(tmp->right);
            }
            result.push_back(tmp->val);
            layerSize = q.size();
        }

        return result;
    }
};
```

### 637. 二叉树的层平均值

只需存储每一层的和与节点数

```cpp
class Solution {
public:
    vector<double> averageOfLevels(TreeNode* root) {
        if (!root) return {};
        vector<double> result;
        queue<TreeNode *> q;
        q.push(root);

        while (!q.empty()) {
            TreeNode *tmp;
            int count = 0;
            long long total = 0;
            for (int i = q.size(); i > 0; i--) {
                tmp = q.front();
                q.pop();
                total += tmp->val;
                count++;
                if (tmp->left) q.push(tmp->left);
                if (tmp->right) q.push(tmp->right);
            }
            result.push_back(total * 1.0 / count);
        }

        return result;
    }
};
```

### 429. N 叉树的层序遍历

```cpp
class Solution {
public:
    vector<vector<int>> levelOrder(Node* root) {
        if (!root) return {};
        vector<vector<int>> result;
        queue<Node *> q;
        q.push(root);

        while (!q.empty()) {
            Node *tmp;
            result.push_back({});
            for (int i = q.size(); i > 0; i--) {
                tmp = q.front();
                q.pop();
                result.back().push_back(tmp->val);
                for (Node *n : tmp->children) {
                    q.push(n);
                }
            }
        }
                      
        return result;
    }
};
```

### 515. 在每个树行中找最大值

```cpp
class Solution {
public:
    vector<int> largestValues(TreeNode* root) {
        if (!root) return {};
        vector<int> result;
        queue<TreeNode *> q;
        q.push(root);

        while (!q.empty()) {
            TreeNode *tmp;
            int max = INT_MIN;
            for (int i = q.size(); i > 0; i--) {
                tmp = q.front();
                q.pop();
                if (tmp->val > max) max = tmp->val;
                if (tmp->left) q.push(tmp->left);
                if (tmp->right) q.push(tmp->right);
            }
            result.push_back(max);
        }

        return result;
    }
};
```

### 116. 填充每个节点的下一个右侧节点指针

```cpp
class Solution {
public:
    Node* connect(Node* root) {
        if (!root) return {};
        queue<Node *> q;
        q.push(root);

        while (q.front()) {
            Node *tmp, *prev = nullptr;
            for (int i = q.size(); i > 0; i--) {
                tmp = q.front();
                q.pop();
                if (prev) prev->next = tmp;
                q.push(tmp->left);
                q.push(tmp->right);
                prev = tmp;
            }
            prev->next = nullptr;
        }

        return root;
    }
};
```

### 117. 填充每个节点的下一个右侧节点指针 II

```cpp
class Solution {
public:
    Node* connect(Node* root) {
        if (!root) return {};
        queue<Node *> q;
        q.push(root);

        while (!q.empty()) {
            Node *tmp, *prev = nullptr;
            for (int i = q.size(); i > 0; i--) {
                tmp = q.front();
                q.pop();
                if (prev) prev->next = tmp;
                if (tmp->left) q.push(tmp->left);
                if (tmp->right) q.push(tmp->right);
                prev = tmp;
            }
            prev->next = nullptr;
        }

        return root;
    }
};
```

### 104. 二叉树的最大深度

```cpp
class Solution {
public:
    int maxDepth(TreeNode* root) {
        if (!root) return {};
        queue<TreeNode *> q;
        q.push(root);
        int depth = 0;

        while (!q.empty()) {
            TreeNode *tmp;
            for (int i = q.size(); i > 0; i--) {
                tmp = q.front();
                q.pop();
                if (tmp->left) q.push(tmp->left);
                if (tmp->right) q.push(tmp->right);
            }
            depth++;
        }

        return depth;
    }
};
```

### 111. 二叉树的最小深度

```cpp
class Solution {
public:
    int minDepth(TreeNode* root) {
        if (!root) return {};
        queue<TreeNode *> q;
        q.push(root);
        int depth = 1;

        while (!q.empty()) {
            TreeNode *tmp;
            for (int i = q.size(); i > 0; i--) {
                tmp = q.front();
                q.pop();
                if (!tmp->left && !tmp->right) return depth;
                if (tmp->left) q.push(tmp->left);
                if (tmp->right) q.push(tmp->right);
            }
            depth++;
        }

        return 0;
    }
};
```

### 559. n 叉树的最大深度

```cpp
class Solution {
public:
    int maxDepth(Node* root) {
        if (!root) return 0;
        int max = 0, tmp;
        for (Node *node : root->children) {
            if ((tmp = maxDepth(node)) > max) {
                max = tmp;
            }
        }
        return max + 1;
    }
};
```
