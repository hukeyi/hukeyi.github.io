---
title: 【基础算法01】排序算法之选择排序（直接选择排序 + 堆排序）
author: hukeyi
date: 2022-04-20 20:59:00 +0800
categories: [数据结构与算法, 基础]
tags: [javascript]
math: true
mermaid: true
toc: true
---

下文中会用到的工具函数`swap(arr, i, j)`

```javascript
// 交换 arr 中下标为 i 和 j 的元素值
function swap(arr, i, j){
    [arr[i], arr[j]] = [arr[j], arr[i]];
}
```

# 排序算法之选择排序

狭义的“选择排序”指直接选择排序。本文默认升序。
> 参考：[CSDN：排序算法总结](https://blog.csdn.net/yushiyi6453/article/details/76407640)；测试排序算法：[leetcode：912. 排序数组](https://leetcode.cn/problems/sort-an-array/)
{: .prompt-info}

## 直接选择排序

假设有n个数字，一共进行n轮，每轮遍历数组的未排序部分：第1轮选择最小的数字，放到数组第1位；第2轮选择第二小的数字，放到数组第2位......直接选相应的牌放到相应位置。

两个for循环，外循环对应一轮确定的坑位，内循环找该坑位对应的下标。

```javascript
function selectionSort(arr) {
    const len = arr.length;
    for (let i = 0; i < len; i++) { // 第一个循环定位本轮要填的牌位
        let minIndex = i; // 搜索区间最小值下标
        for (let j = i; j < len; j++) { // 第二个循环找符合牌位条件的牌的下标
            if (arr[j] < arr[minIndex]) minIndex = j;
        }
        swap(arr, i, minIndex); // 交换牌位原元素和符合条件的元素
    }
}
```

- 时间复杂度：o(n^2)

- 空间复杂度：o(1)

- 稳定性：稳定

> 备注：排序算法“不稳定”是指，原序列中相等的元素，假设原顺序为a1 a2 a3，排序后此顺序**可能**会发生改变（比如，由 a1 a2 a3 -> a2 a1 a3），即**相等元素的原相对位置改变**，就叫排序算法不稳定；反之，排序后相对位置**一定**不变，则排序算法稳定。
{: .prompt-info }

- 与插入排序的区别？

**选择排序是坑位视角**，【“对着坑选牌”：站在坑位面前，面对一堆牌，“选择”某张牌放到面前的坑位】当前坑位为第i大的元素的位置，因此目标就是找第i大的元素；

**插入排序是牌视角**，【“拿着牌选坑”：拿着一张牌，面对已占好坑位的一群牌，然后伺机把当前牌插入其中】当前元素为第i大元素，因此根据元素大小一位一位往前比较坑位，挪到第i大的位置。

## 堆排序

堆排序的重点其实是[堆的实现](https://hukeyi.github.io/posts/basic-heap-js/)。如果语言内置了堆的实现，那就很简单了。

堆排序的逻辑与直接选择类似，都是对着坑选牌。首先选出最大/小的，然后选出次大/小的，以此类推。不同在堆排序选择牌的方法更高效。堆排序选一次牌只花 o(logn)，而直接选择排序选一次牌要花 o(n)。

### 写法一：空间复杂度 o(n)

```javascript
function heapSort(arr) {
    let minheap = new Heap((x, y) => x - y, arr);
    
    const ans = [], len = arr.length;
    // 逐渐remove堆顶元素到结果数组ans
    for (let i = 0; i < len; i++) {
        ans.push(minheap.removeTop());
    }
    return ans;
}
```

- 时间复杂度：o(nlogn)。`removeTop()`中包含一次`heapifyDown()`，时间复杂度为o(logn)。一共进行 n 次 `removeTop()`，因此堆排序为o(nlogn)。
- 空间复杂度：o(n)，结果数组 `ans` 额外开辟空间。

### 写法二：空间复杂度 o(1)

利用原数组空间，把堆顶元素直接放到原数组末尾，无需开辟新数组空间。相当于把原数组划分为两个部分：`arr[0:i]` 和`arr[i+1:n]`，前半部分是堆的剩余元素，后半部分是已排好顺序的数组。

本文实现升序，需要**大顶堆**将目标数组堆化，每次`removeTop()`后都将堆顶元素放到堆的最末尾处，这样就能形成升序。

```javascript
function heapSort(arr) {
    // arr与maxheap共用同一处空间
    // 将maxheap的this.size想象为一个始终指向堆末尾元素的指针
    // 随着while循环，堆逐渐remove top，this.size逐渐前移，而this.size后方的子数组即为当前排好顺序的数组arr
    let maxheap = new Heap((x, y) => y - x, arr);

    while (maxheap.size) { // 当堆不为空
        let top = maxheap.removeTop(); // 删除堆顶元素：堆内最大值
        arr[maxheap.size] = top; // 放到arr中
    }
    return arr;
}
```

- 为什么是常数空间复杂度？

看下面的例子：（“｜”前是堆内元素数组，后面是已排好序的`arr`数组）

```
【最初 maxheap 内】
[95, 34, 10, 6, 21, 2, 9, 2, 3, 7, 3, 1｜]
【第一轮】
[34, 21, 10, 6, 7, 2, 9, 2, 3, 1, 3｜95]
【第二轮】
[21, 7, 10, 6, 3, 2, 9, 2, 3, 1｜34, 95]
【第三轮】
[10, 7, 9, 6, 3, 2, 1, 2, 3｜21, 34, 95]
```

