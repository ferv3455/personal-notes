---
title: Algorithms to Remember
weight: 91
type: docs
---

## 递归主定理

$T(n) = a T(n/b) + f(n)$, $c = \log_b a$

- $f(n) = O(n^{c-\epsilon})$: $T(n) = \Theta(n^c)$
- $f(n) = \Theta(n^{c}\log^k n)$: $T(n) = \Theta(n^c \log^{k+1}n)$
- $f(n) = \Omega(n^{c+\epsilon})$, $af(n/b)\le kf(n)$ (正则条件): $T(n) = \Theta(f(n))$

## AVL 树（平衡二叉搜索树）

- 旋转：左旋/先右旋后左旋/右旋/先左旋后右旋（根据失衡节点与子节点的平衡因子决定）
- 插入、删除：自底向上解决失衡节点

## 红黑树

WIP

## 图的遍历

DFS 和 BFS 均有 $O(m+n)$ 时间复杂度和 $O(n)$ 空间复杂度

## 搜索算法

- 暴力搜索：线性搜索、BFS、DFS
- 自适应搜索：二分查找、哈希表、二叉搜索树、AVL 树

<img src="../attachments/Pasted image 20241111201001.png" >

## 排序算法

计数排序：计算计数数组的前缀和，从后往前填充

<img src="../attachments/Pasted image 20241111202256.png" width="100%">

## 图论算法

- 并查集：第一次查询操作$O(\log n)$（包括`find`和`join`），后续趋近于$O(1)$
- DFS、BFS：$O(n+m)$（使用栈/队列实现，注意标记和检测`visited`数组的时间）
- 最小生成树：Prim $O(n^2)$（距离数组实现）/$O(m\log m)$（优先队列实现）（这个关系与Dijkstra算法一致），Kruskal $O(m\log m)$（优先队列、并查集实现）
- 拓扑排序：$O(n+m)$（本质上与BFS相同）
- 通用搜索算法总结：$O(n+m)$（栈/队列）、$O(m\log m)$（优先队列，如果可以修正队列元素则为$O((n+m)\log n)$）、$O(n^2)$（距离数组）
    - DFS、BFS：使用栈与队列，指定起点（优先级/距离均为1）
    - 拓扑排序：使用队列，起点为入度为0的节点（优先级/距离均为1）
    - Dijkstra：使用节点到起点的距离作为距离数组元素/边的优先级
    - Prim：使用节点到生成树到距离作为距离数组元素/边的优先级
- 最短路径：
    - Dijkstra：单源、权值非负，$O(n^2)$（稠密图）/$O(m\log m)$（稀疏图）
        - A\*可用于权值修正，更快找到首个有效解
        - 要添加边数限制，则不适用于Dijkstra，只能采用二维距离数组（动态规划思想，同时考虑不同边数的路径），$O(km)$
    - Bellman-Ford：单源、负权边、检测负权回路，$O(mn)$（可使用队列优化稀疏图平均情况、最优情况的时间复杂度）
    - Floyd-Warshall：多源、负权边、检测负权回路（算法结束后检测`dist[i][i]`），$O(n^3)$（矩阵可压缩为二维）
