---
title: 【基础算法】排序算法之计数排序
author: hukeyi
date: 2022-05-14 23:16:00 +0800
categories: [数据结构与算法, 基础]
tags: [javascript]
math: true
mermaid: true
toc: true
---

# 排序算法之计数排序 

> 参考：[CSDN：图解计数排序](https://blog.csdn.net/weixin_44820625/article/details/106808623?spm=1001.2101.3001.6650.3&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-3-106808623-blog-107865766.pc_relevant_paycolumn_v3&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-3-106808623-blog-107865766.pc_relevant_paycolumn_v3&utm_relevant_index=5)

一种空间换时间的**非比较**方法。

- 开辟一个长度为数组数值跨度（最大值-最小值+1）的辅助数组（桶）
- 遍历数组元素，将元素放置到各自应去的桶中
- 全部元素放置完后，遍历桶内的元素，重新整合得到的新数组就是排好序的

```javascript
function countSort(nums) {
    const len = nums.length, 
          max = Math.max(...nums),
          min = Math.min(...nums),
          size = max - min + 1; // count数组的size
    const count = new Array(size).fill(0); // count数组
    // 计数
    for (let n of nums) count[n - min]++;
    // 解决不稳定问题：
    // 原本经过上一行代码的count数组，count[i]表示【等于min+i】的元素个数
    // 经过下一行代码的重新整合后，count[i]表示【小于等于min+i】的元素个数
    // 重新整合count数组 count[i] = count[i] + count[i-1]
    for (let i = 1; i < size; i++) count[i] += count[i - 1];
    // 【反向遍历】原数组nums，整合新数组
    const sortNums = new Array(len);
    for (let i = len - 1; i >= 0; i--) {
        let idx = count[nums[i] - min] - 1;
        sortNums[idx] = nums[i];
        count[nums[i] - min]--;
    }
    return sortNums;
}
```

- 时间复杂度：o(n+k)。k为数组的数值跨度（`max - min + 1`）
- 空间复杂度：o(n+k)。当数组的数值集中在某个数值区间时，计数排序的效率最高；但当数值跨度比较大，比如`[0, 100000]`，计数排序就不适用了，非常浪费内存空间
- 稳定性：**稳定**

> 备注：原本的简单计数排序是不稳定的，经过两个操作变为稳定：
> 
> 1. 优化`count`数组，`count[i]`从**等于`min+i`**的元素个数变为**小于等于`min+i`**的元素个数
> 2. **反向遍历**原数组`nums`（不是计数数组`count`）。保证相同大小的元素在排序前后相对位置不变。这个脑内想象一下排序过程就很好理解，但是不好言语描述
{: .prompt-info }
