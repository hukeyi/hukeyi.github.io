---
title: 【设计模式】详解组合模式
author: hukeyi
date: 2023-03-21 10:27:00 +0800
categories: [设计模式, 理论]
tags: [设计模式, javascript]
math: true
toc: true
---

组合模式通过树形结构，统一了对【对象】和【对象集合】的操作。用户无需理会面对的是一个对象还是一群对象，它们的操作手段是统一的。

> 需要与前文 [详解迭代器模式](https://hukeyi.github.io/posts/design-patterns-iterator-pattern/) 结合阅读。
{: .prompt-info }

## 问题场景

依然用 [详解迭代器模式](https://hukeyi.github.io/posts/design-patterns-iterator-pattern/) 的菜单例子。

迭代器模式中，我们需要合并早餐店和咖啡店的菜单，两者分别都只有一份菜单。

假如现在出现了新的需求，咖啡店的菜单中需要加上一份**子菜单**——甜品菜单。应当如何修改程序满足新需求呢？

直接看《HF》上的示意图，将菜单原本的数据结构重构为**树**：

![结点树示意图](/assets/img/2023/design-pattern-hf-ch09-02-tree.png)
_截取自《HF》_

> 其实如果使用 JavaScript 或 Python，直接用一个嵌套数组就好了嘛。迭代菜单时增加一个 if 条件，递归深度遍历数组。
> 
> 《HF》中以 Java 为例。Java 的数组定死了数组中所有元素的类型，无法像 JavaScript 和 Python 那样灵活。所以需要引入新的数据结构：树。（JavaScript 的递归深度遍历就是遍历树，也就是说解决思想是相同的）

【对象】对应叶子结点，【对象集合】指向非叶子结点。

咖啡店的菜单结构：

```javascript
const CafeMenu = [cafe1, cafe2, ..., sweetMenu];
const sweetMenu = [sweet1, sweet2, ...];
// 合并写成嵌套数组的样子：
CafeMenu = [
    cafe1, 
    cafe2, 
    ..., 
    [sweet1, sweet2, ...]
];
```

## 代码实现

组合模式的类图模版：

![组合模式类图](/assets/img/2023/design-pattern-hf-ch09-02-class-graph.png)
_截取自《HF》_

菜单例子的类图：

![菜单示例类图](/assets/img/2023/design-pattern-hf-ch09-02-class-graph-02.png)
_截取自《HF》_

```javascript
// 结点抽象类
class MenuComponent {
    /**
     * 一些结点操作
     */
    add(menuComponent) {}
    remove(menuComponent) {}
    getChild() {}

    /**
     * 一些菜单操作
     */
    getName() {}
    getPrice() {}

    /**
     * 打印菜单
     */
    print() {}
}

class MenuItem extends MenuComponent {
    constructor(name, price) {
        super();
        this.name = name;
        this.price = price;
    }
    // 菜单项类只实现菜单操作
    getName() {
        return this.name;
    }
    getPrice() {
        return this.price;
    }
    print() {
        console.log('----------');
        console.log(`名字：${this.getName()}\n价格：${this.getPrice()}`);
        console.log('----------');
    }
}

/**
 * 迭代器接口
 * hf 直接调用了 Java ArrayList 的 iterator()
 * JavaScript 无，只能自己写一个了
 * 借用迭代器模式的代码
 */
class Iterator {
    next() {}
    hasNext() {}
}
class MenuIterator extends Iterator {
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

class Menu extends MenuComponent {
    constructor(name, list) {
        super();
        this.name = name;
        this.list = list;
    }
    /**
     * 实现结点操作
     */
    add(menuComponent) {
        this.list.push(menuComponent);
    }
    remove(menuComponent) {
        let idx = -1;
        for (let i = 0; i < this.list.length; i++) {
            if ((this.list[i].name = menuComponent.name)) {
                idx = i;
                break;
            }
        }
        if (idx >= 0) {
            this.list.splice(idx, 1);
        }
    }
    /**
     * 实现菜单操作
     */
    getName() {
        return this.name;
    }
    /**
     * hf 直接调用了 Java ArrayList 的 iterator()
     * JavaScript 无，只能自己写一个了
     */
    createIterator() {
        return new MenuIterator(this.list);
    }
    // getPrice 对菜单无意义，不实现
    /**
     * 重点！！！
     * 递归操作，遍历树
     */
    print() {
        console.log('================');
        console.log(`菜单名：${this.getName()}`);
        console.log('================');
        let iterator = this.createIterator(); // hf 上的例子直接调用 ArrayList 的自带 iterator
        while (iterator.hasNext()) {
            let item = iterator.next();
            item.print();
        }
    }
}

/**
 * User code
 */
let allMenu = new Menu('Cafe Menu', []);
allMenu.add(new MenuItem('美式', '¥15'));
allMenu.add(new MenuItem('拿铁', '¥18'));
allMenu.add(new MenuItem('浓缩', '¥18'));

let sweetMenu = new Menu('Sweet Menu', []);
sweetMenu.add(new MenuItem('提拉米苏', '¥20'));
sweetMenu.add(new MenuItem('杯子蛋糕', '¥8'));

allMenu.add(sweetMenu);

allMenu.print();
```

输出：

```shell
================
菜单名：Cafe Menu
================
----------
名字：美式
价格：¥15
----------
----------
名字：拿铁
价格：¥18
----------
----------
名字：浓缩
价格：¥18
----------
================
菜单名：Sweet Menu
================
----------
名字：提拉米苏
价格：¥20
----------
----------
名字：杯子蛋糕
价格：¥8
----------
```

## 优缺点

优点：

- 用单一责任原则换取了「透明性」transparency，即菜单项还是菜单组合对用户透明（用户不知道自己操作的是菜单还是菜单项）。

缺点：

- 组合模式违反了「单一责任原则」。示例中的 `Component` 类兼顾了两项责任：执行操作和管理数据结构。

## 参考资料

- 《Head First 设计模式》第九章