---
title: Chapter 2 Linked Lists
weight: 2
type: docs
---

## 基本题目

### 203. 移除链表元素

辅助头节点

```cpp
class Solution {
public:
    ListNode* removeElements(ListNode* head, int val) {
        ListNode dummy(0, head);
        ListNode *ptr = &dummy, *tmp;
        while (ptr && ptr->next) {
            if (ptr->next->val == val) {
                tmp = ptr->next;
                ptr->next = tmp->next;
                delete tmp;
            }
            else {
                ptr = ptr->next;
            }
        }
        return dummy.next;
    }
};
```

### 707. 设计链表

链表的基本操作

```cpp
class MyLinkedList {
public:
    struct Node {
        int val;
        Node *next;
        Node(int v) : val(v), next(nullptr) {}
    };

    Node *head, *tail;

    MyLinkedList() {
        head = new Node(-10);
        tail = head;
    }
    
    int get(int index) {
        Node *tmp = head;
        for (int i = 0; i <= index; i++) {
            tmp = tmp->next;
            if (!tmp) return -1;
        }
        return tmp->val;
    }
    
    void addAtHead(int val) {
        Node *newNode = new Node(val);
        newNode->next = head->next;
        head->next = newNode;
        if (tail == head) tail = newNode;
    }
    
    void addAtTail(int val) {
        tail->next = new Node(val);
        tail = tail->next;
    }
    
    void addAtIndex(int index, int val) {
        Node *tmp = head;
        for (int i = 0; i < index; i++) {
            tmp = tmp->next;
            if (!tmp) return;
        }

        Node *newNode = new Node(val);
        newNode->next = tmp->next;
        tmp->next = newNode;
        if (tmp == tail) tail = newNode;
    }
    
    void deleteAtIndex(int index) {
        Node *tmp = head;
        for (int i = 0; i < index; i++) {
            tmp = tmp->next;
            if (!tmp) return;
        }
        if (!tmp->next) return;
        if (tmp->next == tail) tail = tmp;
        tmp->next = tmp->next->next;
    }
};
```

### 206. 反转链表

链表指针+递归/迭代

```cpp
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        ListNode *newHead = nullptr, *curr = head, *tmp;
        while (curr) {
            tmp = curr;
            curr = curr->next;
            tmp->next = newHead;
            newHead = tmp;
        }
        return newHead;
    }
};
```

**也可以用递归的方式解决：`n->next->next = n` 处理交界区域**

### 24. 两两交换链表节点

链表指针+递归/迭代

```cpp
class Solution {
public:
    ListNode* swapPairs(ListNode* head) {
        ListNode *newHead = new ListNode(0, head);
        ListNode *tmp = newHead, *n1, *n2;
        while ((n1 = tmp->next) && (n2 =tmp->next->next)) {
            tmp->next = n2;
            n1->next = n2->next;
            n2->next = n1;
            tmp = n1;
        }
        return newHead->next;
    }
};
```
### 19. 删除链表的倒数第 N 个节点

快慢指针

```cpp
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        ListNode *newHead = new ListNode(0, head);
        ListNode *slow = newHead, *fast = slow;
        for (int i = 0; i <= n; i++) fast = fast->next;
        while (fast) {
            slow = slow->next;
            fast = fast->next;
        }
        slow->next = slow->next->next;
        return newHead->next;
    }
};
```
### 面试题 02.07. 链表相交

双指针分别追踪两个链表，位置的距离关系

```cpp
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        if (!headA || !headB) return nullptr;
        ListNode *tmp1 = headA, *tmp2 = headB;
        while (tmp1 || tmp2) {
            if (!tmp1) tmp1 = headB;
            if (!tmp2) tmp2 = headA;
            if (tmp1 == tmp2) return tmp1;
            tmp1 = tmp1->next;
            tmp2 = tmp2->next;
        }
        return nullptr;
    }
};
```

也可以使用额外的数据结构来判断共同节点

### 142. 环形链表 II

快慢指针，位置的距离关系

```cpp
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        if (!head) return nullptr;
        ListNode *slow = head, *fast = head;
        do {
            if (!fast->next || !fast->next->next) return nullptr;
            fast = fast->next->next;
            slow = slow->next;
        } while (fast != slow);
        fast = head;
        while (fast != slow) {
            fast = fast->next;
            slow = slow->next;
        }
        return slow;
    }
};
```
