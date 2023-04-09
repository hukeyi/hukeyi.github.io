---
title: 【设计模式】详解迭代器模式
author: hukeyi
date: 2023-03-18 10:45:00 +0800
categories: [设计模式, 理论]
tags: [javascript]
math: true
toc: true
---

迭代器模式在对象集合内部实现未知的情况下，为**类型不同**的对象集合提供**统一的遍历迭代方法**。

## 问题场景

《HF》的例子：早餐店和咖啡店合并，需要统一菜单。

早餐店的菜单使用 `ArrayList`，咖啡店菜单使用数组。统一菜单时，需要用两个 `for` 循环调用不同的 API 实现。

有什么设计方法，能够在统一菜单时，无需知道菜单各自内部的数据结构，直接统一遍历合并菜单？提供一个统一的「迭代器」接口。

让两种菜单都实现统一的迭代器接口。需要创建菜单的用户只需要调用迭代器 api，合并菜单就 ok 了。

![](/assets/img/2023/design-pattern-iterator-pattern-class-graph.png)

## 代码实现

只实现了一个用数组存菜单项的早餐菜单：

```js
/**
 * 迭代器接口
 */
class Iterator {
    // get next item in collection
    next() {}
    // is there next item in collection
    hasNext() {}
}

class BreakfastMenuIterator extends Iterator {
    constructor(list) {
        super();
        this.list = list;
        this.position = 0; // current iterate position in the list
    }
    next() {
        if (this.hasNext()) return this.list[this.position++];
    }
    hasNext() {
        return this.position < this.list.length;
    }
}

/**
 * 菜单接口
 */
class Menu {
    addMenuItem(name, price) {}
}

/**
 * 只实现一个菜单意思一下
 */
class MenuItem {
    constructor(name, price) {
        // 忽略 get 函数，简化代码
        // 直接用 this.[prop] 拿值
        this.name = name;
        this.price = price;
    }
}
class BreakfastMenu extends Menu {
    constructor(list) {
        super();
        this.list = list || new Array();
    }
    createIterator() {
        return new BreakfastMenuIterator(this.list);
    }
    addMenuItem(name, price) {
        this.list.push(new MenuItem(name, price));
    }
}

/**
 * User code
 */

// create menu
let breakfastMenu = new BreakfastMenu();
breakfastMenu.addMenuItem('肉', '¥10');
breakfastMenu.addMenuItem('菜', '¥8');

// print menu
let iterator = breakfastMenu.createIterator();
console.log('----------');
while (iterator.hasNext()) {
    let item = iterator.next();
    console.log(`菜名：${item.name}\n价格：${item.price}`);
    console.log('----------');
}
```

输出结果：

```shell
----------
菜名：肉
价格：¥10
----------
菜名：菜
价格：¥8
----------
```

## 现实应用

- 各种编程语言代码库的提供的数据结构的迭代器对象。比如 C++，`<vector>` 的迭代器对象由其成员函数 `begin()` 和 `end()` 返回。

## 参考资料

- [Youtube: iterator pattern](https://www.youtube.com/watch?v=uNTNEfwYXhI&list=PLrhzvIcii6GNjpARdnO4ueTUAVR9eMBpc&index=16&ab_channel=ChristopherOkhravi) 
- 《Head First 设计模式》第九章