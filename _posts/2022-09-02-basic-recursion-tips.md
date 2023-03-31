---
title: 【基础算法】如何思考递归之递归函数的返回值
author: hukeyi
date: 2022-09-02 22:51:00 +0800
categories: [数据结构与算法, 基础]
tags: [javascript]
math: true
toc: true
---

搞清递归的关键：清楚递归函数的定义 + 递归函数返回值的含义。

**递归函数返回值不一定是题目要求的目标结果**。

## 例题 1：最长同值路径

> [687. 最长同值路径](https://leetcode.cn/problems/longest-univalue-path/)。

题目要求：`root` 中的最长同值路径，此路径无需穿过 `root`。

```javascript
var longestUnivaluePath = function(root) {
    let ans = 0; // 结果辅助变量
    const dfs = function(root, preVal){
        if (!root) return 0;

        let leftmax = dfs(root.left, root.val);
        let rightmax = dfs(root.right, root.val);
        // 【更新结果，结果变量与 dfs 返回值无关】
        ans = Math.max(ans, leftmax + rightmax);
        if (root.val !== preVal) return 0;
        // 【返回穿过 root 的最长同值路径，而非 ans】
        return 1 + Math.max(leftmax, rightmax);
    }
    dfs(root, -1001);
    return ans;
};
```

注意，这道题的递归函数 `dfs(root, preVal)` 的返回值是**root 整棵树中，值为 preVal 且经过 root 的最长路径长度**。

如果在思考解法时，错把递归函数的返回值想成 `root` 树中的最长同值路径，就很容易卡壳。

## 例题 2：二叉树的直径

> [543. 二叉树的直径](https://leetcode.cn/problems/diameter-of-binary-tree/)

题目要求：`root` 的最长直径。

```javascript
var diameterOfBinaryTree = function(root) {
    let ans = 0; // 结果的辅助变量
    const dfs = function(root){
        if (!root) return 0;

        let leftMax = dfs(root.left);
        let rightMax = dfs(root.right);
        // 【更新结果，结果变量与 dfs 返回值无关】
        ans = Math.max(ans, leftMax + rightMax);
        // 【返回 root 的高度（含root）】
        return 1 + Math.max(leftMax, rightMax);
    }
    dfs(root);
    return ans;
}
```

这道题的递归函数 `dfs(root)` 返回值是 **root 到其叶子结点的最长路径/ root 的高度（含 root）**，返回值显然不是树的直径。

那么需要的结果在哪里更新呢？均是在合适的位置（比如这两道题都在后序位置）更新，利用递归函数外的辅助变量 `ans` ，加上子函数的返回值更新结果，最终不返回此结果。

## 例题 3：二叉树中的最大路径和

> [124. 二叉树中的最大路径和](https://leetcode.cn/problems/binary-tree-maximum-path-sum/)

```js
var maxPathSum = function(root) {
    let ans = -Infinity;
    // dfs(root) 返回的是 root 的某一条路径
    const dfs = function(root){
        if (!root) return 0;

        let leftMax = dfs(root.left);
        let rightMax = dfs(root.right);
        // max 指整棵子树内部路径的最大值
        let max = root.val + leftMax + rightMax;
        ans = Math.max(ans, max); // 更新结果
        // 注意返回值不是 max！
        // 返回值只能选择单独 root 或 左子树到根 或 右子树到根
        return Math.max(0, root.val + Math.max(0, leftMax, rightMax));
    }
    dfs(root);
    return ans;
};
```

本题 `dfs(root)` 的返回值为 `root` 这棵树中从 `root` 开始的最大路径和；但是结果 `ans` 需要的是无所谓起点的不重复经过结点的树的最大路径和。