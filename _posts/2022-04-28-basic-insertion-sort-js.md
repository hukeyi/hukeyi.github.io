---
title: 【基础算法】排序算法之插入排序（直接插入排序 + 希尔排序）
author: hukeyi
date: 2022-04-28 21:01:00 +0800
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

# 排序算法之插入排序

狭义的“插入排序”指直接插入排序。

## 直接插入排序

类似于打扑克牌开局码牌的过程，拿起一张牌，逐个比较它与手上牌的大小，把它放到相应位置。

一位一位往前挪，与**已经排好序的数字们**比较（与直接选择排序相比，选择排序每轮目标就是找到符合坑位条件的位置，然后直接填坑位）

两个for循环，外循环对应拿起的牌，内循环对应此牌与前面牌比较的过程。

### 交换法

```javascript
function insertionSort(arr) {
    const len = arr.length;
    for (let i = 0; i < len; i++) {
        let curr = arr[i]; // 拿起的牌
        for (let j = i - 1; j >= 0 && arr[j] > curr; j--) { // 前面已经拿起排过位置的牌
            swap(arr, j, j + 1); // 如果拿起的牌比它前一位的牌小，则把它往前挪
        }
    }
}
```

### 移动法

```javascript
// 移动法
function insertionSort2(arr) {
    const len = arr.length;
    for (let i = 0; i < len; i++) {
        // 以当前拿起的牌（curr）的值为准，往【前】查找curr应该放置的位置
        // e.g. 【0，2，4】（1）...
        // j从4开始往前/左遍历，只要当前arr[j]大于curr，那么arr[j]就应该往后/右挪
        // 直到arr[0]=0<1，for循环终止，说明curr应放在arr[0]后一位
        // 1）需记住j的最终位置，因此j的初始化必须在for条件句外；
        // 2）for循环终止后，需把curr放在它应该在的位置arr[j+1]
        let curr = arr[i],
            j; // 1）
        for (j = i - 1; j >= 0 && arr[j] > curr; j--) {
            arr[j + 1] = arr[j];
        }
        arr[j + 1] = curr; // 2）
    }
    return arr;
}
```

- 时间复杂度：o(n^2)
- 空间复杂度：o(1)
- 稳定性：稳定

## 希尔排序

> 参考：[图解希尔排序](https://www.cnblogs.com/chengxiao/p/6104371.html)。有图解，很清晰
{: .prompt-info } 

对**直接插入排序**的优化。

将原数组划分为n个增量序列（**下标间隔相同的元素们**集合形成的序列），在各个序列内部进行直接插入排序；逐渐减小增量的值；最终减小到增量为1，即对原数组进行直接插入排序，此时原数组经过之前一系列的增量插入排序，已经基本有序，而**直接插入排序对基本有序的序列的排序效率很高**。

增量序列（又称“步长序列”）是什么？

e.g. [0,1,2,3,4,5]，假设增量为2，则 [0,2,4] 为一个增量序列，[1,3,5] 为另一个，两者的相邻元素之间的下标差值均为2。

### 交换法

```javascript
function shellSort(nums) {
    const len = nums.length;
    // 外循环，确定增量gap，从len/2开始，逐渐减半
    // 增量gap表示，被分在同一组的元素的下标差值，比如gap=3，则0,3,6,...为一组
    // 每组内分别做直接插入排序
    for (let gap = len >> 1; gap > 0; gap >>= 1) {
        // 每次增量序列内的插入排序比较，都是从每组的第2个元素开始的（因为第1个元素是一个单元素数组，自然有序
        // 因此，i的for循环从gap开始，因为gap是第1个增量序列的第2个元素下标
        for (let i = gap; i < len; i++) {
            let j = i;
            // watch out! j-gap>=0 not j>=0
            while (j - gap >= 0 && nums[j] < nums[j - gap]) {
                swap(nums, j, j - gap);
                j -= gap; // 往前/左
            }
        }
    }
    return nums;
}
```

### 移动法

```javascript
function shellSort2(nums) {
    const len = nums.length;
    for (let gap = len >> 1; gap > 0; gap >>= 1) {
        for (let i = gap; i < len; i++) {
            let j = i - gap,
                curr = nums[i];
            // curr为当前我们目标插入的元素
            while (j >= 0 && nums[j] > curr) {
                nums[j + gap] = nums[j];
                j -= gap;
            }
            // 经过while循环，找到curr的rightful position，在j之后一个增量位
            nums[j + gap] = curr;
        }
    }
    return nums;
}
```

- 时间复杂度：跟随增量序列的选择改变，最好为o(nlogn) or o(n^1.5)，最差o(n^2)
- 空间复杂度：o(1)
- **稳定性：不稳定**。
  - 简单理解：当相同元素被分在不同的增量序列中时，它们之间的相对位置就可能改变。