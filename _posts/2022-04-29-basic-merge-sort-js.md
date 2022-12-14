---
title: 【基础算法01】排序算法之归并排序
author: hukeyi
date: 2022-04-29 19:24:00 +0800
categories: [数据结构与算法, 基础]
tags: [javascript]
math: true
mermaid: true
toc: true
---

# 排序算法之归并排序 

将整个数组对半分，直到每个子数组只剩1个元素；然后回溯往上合并，合并时按升序将合并前的子数组元素排列，最终得到整个数组的升序。

对半分操作会进行logn次，一次对半分操作对应一次合并操作，每次合并都需要遍历两个子数组的所有元素，因此时间复杂度为o(n * logn)

## 空间浪费的归并排序

以下为空间较浪费的非原地修改归并排序算法：

```javascript
function mergeSort(nums, left, right) {
    if (left === right) return [nums[left]]; // 递归终止条件，对半分到子数组只剩1个元素

    let mid = Math.floor((left + right) / 2); // 对半分的边界点
    let arr1 = mergeSort(nums, left, mid); // 左子数组继续merge
    let arr2 = mergeSort(nums, mid + 1, right); // 右子数组继续merge
    let merge = [], // 等会儿合并的新数组【这个每轮递归都会开辟的新数组，可以优化掉】
        i = 0, // 遍历左子数组的指针
        j = 0; // 遍历右子数组的指针
    while (i < arr1.length && j < arr2.length) {
        merge.push(arr1[i] <= arr2[j] ? arr1[i++] : arr2[j++]);
    }
    // 若有没遍历到终点的，剩下的数组元素直接接到merge后面
    if (i < arr1.length) merge = merge.concat(arr1.slice(i));
    else if (j < arr2.length) merge = merge.concat(arr2.slice(j));
    return merge;
}
```

- 时间复杂度：o(nlogn)
- 空间复杂度：o(n)。会占用递归栈logn和每层合并时会开辟新数组放合并元素，所有合并操作一共会开辟n的空间（因为新空间必定等于输入数组`nums`的空间）。
- 稳定

## 原地修改的归并排序（空间优化）

合并左右子树时开辟的辅助数组`merge`，可以提前在函数作用域外部开辟，每次递归函数合并时都重复使用同一个内存空间合并数组，避免递归时反复开辟空间造成的消耗。

```javascript
/**
* 原地排序的归并排序，优化空间
* 在sort函数外部开辟辅助空间，避免递归时频繁开辟内存空间造成的消耗
* @param {*} nums
* @returns
*/
function mergeSort(nums) {
    const temp = new Array(nums.length); // 辅助数组，保存合并前的两个子数组
    const sort = function (nums, left, right) {
        // 原地排序
        if (left === right) return;

        let mid = left + ((right - left) >> 1);
        sort(nums, left, mid);
        sort(nums, mid + 1, right);
        merge(nums, left, mid, right); // 合并
    };
    // 把两个有序数组nums[left:mid]和nums[mid+1:right]按序合并
    const merge = function (nums, left, mid, right) {
        // 把待合并的数组元素复制到辅助数组temp中，保证合并结果直接保存在nums中
        for (let i = left; i <= right; i++) temp[i] = nums[i];

        // i是左子数组的指针，j是右子数组的，p是nums的
        let i = left,
            j = mid + 1,
            p = left;
        while (p <= right) {
            if (i > mid) break; // 小优化：左子数组合并完后，nums剩下的部分nums[p:right]与右子数组剩下部分temp[j:right]完全一致，无需再nums[p++]=temp[j++]了，直接break
            else if (j > right) nums[p++] = temp[i++];
            else nums[p++] = temp[i] < temp[j] ? temp[i++] : temp[j++];
        }
    };


    sort(nums, 0, nums.length - 1);
    return nums; // 用于测试，实际的原地修改代码无需return
}
```
