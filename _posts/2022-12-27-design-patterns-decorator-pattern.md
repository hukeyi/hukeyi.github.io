---
title: 【设计模式】详解装饰者模式
author: hukeyi
date: 2022-12-27 11:37:00 +0800
categories: [设计模式, 理论]
tags: [设计模式, javascript]
math: true
toc: true
---

装饰者模式可以在不修改代码和不创建新的子类的情况下，动态地为对象扩展功能。

## 问题场景

假设我们开了一家咖啡店。

假设这家店是星巴克，它拥有一个饮料超类，名为 `Beverage`。所有咖啡子类都继承自这个饮料超类。饮料超类主要有两个成员函数：

```js
class Beverage{ // 抽象类
    constructor(){
        this.description = "default";
    }
    getDescription(){
        return this.description;
    }
    cost(){ /* 由子类实现 */}
}
```

我们举一个咖啡子类 `Mocha` 作为例子：

```js
class Mocha extends Beverage{
    constructor(){
        this.description = "Mocha";
    }
    cost(){
        return 1.99;
    }
}
```

### 问题1：类数量爆炸

星巴克并不只有一种摩卡咖啡，它还有双倍牛奶浓缩咖啡、抹茶星冰乐、蜂蜜红茶等等等等。

所有这些不同种类的咖啡，都需要继承自 `Beverage`。这样的话，最终的类图就会变得非常庞大。况且，假设顾客只想点一杯加三倍蜂蜜的摩卡咖啡，我们该如何实现这个需求呢？继续继承 `Beverage`，创建一个新的“三倍蜂蜜摩卡”子类吗？

### 问题2：解决问题 1 的同时引出问题 2

咖啡店中的新种类咖啡，其实都是在饮料基础上加上各种调料，例如蜂蜜、牛奶等等。我们可以将这些调料单独抽象出来，只留下几个以饮料基底命名的基础类，然后在基础类之上改变各种调料变量，组合成不同的饮料，就能解决类数量爆炸的问题（这个思想有点类似于[策略模式](https://hukeyi.github.io/posts/design-patterns-strategy-pattern/)）。那么，这个该如何在代码中实现呢？

很容易想到，在超类 `Beverage` 中添加 setter 函数（e.g. `setMilk()`）。当加入调料后，价格也应当相应地变化。我们再添加 hasOrNot 函数（e.g. `hasMilk()`），用于判断饮料中是否加入了某种调料：

```js
class Beverage{
    constructor(){
        // ...
        this.hasMilk = false;
        this.hasHoney = false;
        // ...
    }
    // ...
    setMilk(){}
    hasMilk(){}
    setHoney(){}
    hasHoney(){}
    // ...
    cost(){
        let x = 1.99;
        if (this.hasMilk()) x += 0.5;
        else if (this.hasHoney()) x += 0.6;
        // else if ...
        return x;
    }
}
```

看上去似乎没问题？

但是，如果出现了新需求，比如现在我们需要调整调料价格、增加新的调料或删除旧的调料，就需要对超类代码进行大量修改，包括更改cost函数、更改构造函数、添加或删除setter和has函数等。

另外，如果星巴克新增一种酒精饮料，该饮料不能加入蜂蜜或牛奶，那么将出现类似于[策略模式的问题](https://hukeyi.github.io/posts/design-patterns-strategy-pattern/#%E9%97%AE%E9%A2%98-1-%E5%AD%90%E7%B1%BB%E4%B8%8E%E7%88%B6%E7%B1%BB%E6%88%90%E5%91%98%E5%87%BD%E6%95%B0%E4%B8%8D%E4%B8%80%E8%87%B4)，因为子类必然继承超类的成员函数。

> [参考视频](https://www.youtube.com/watch?v=GCraGHx6gso&list=PLrhzvIcii6GNjpARdnO4ueTUAVR9eMBpc&index=3&ab_channel=ChristopherOkhravi) 中的讲解者认为，HF 教材的星巴克例子很适合讲解装饰者模式，但却不是装饰者模式的 best use case。就是说，解决以上咖啡店例子问题的最佳设计模式其实不是装饰者模式。
> 
> 我大概能 get 到他的意思。对我来说诡异的点是：饮料调料的类「继承」自饮料超类。调料又不是一种饮料对吧？但它却继承自饮料超类，well…… 符合直觉的做法是额外创造一个调料超类，然后「组合」调料超类的各种子类+饮料超类的各种子类来搭配出新的饮料。

## 代码实现

让我们先来看一下装饰者模式的类图（截取自[参考视频](https://www.youtube.com/watch?v=GCraGHx6gso&list=PLrhzvIcii6GNjpARdnO4ueTUAVR9eMBpc&index=3&ab_channel=ChristopherOkhravi)）：

![装饰者模式星巴克例子的类图](/assets/img/2022/design-pattern-hf-03-00.png){: width="60%" height="60%"}

**装饰者模式的核心：装饰者类既是（is-A） Beverage 超类的子类，又拥有（has-A）一个 Beverage 超类的另外子类的实例。**

什么叫既是 is-a 又是 has-a？例如，在以下代码中，我们定义了一个调料装饰者抽象类，它继承自 Beverage 超类，同时拥有一个 Beverage 类型的实例变量 beverage：

```js
// 饮料超类
class Beverage{/* ... */}

// 调料装饰者类
class AddonDecorator extends Beverage{ // is-a
    constructor(){
        super();
        this.beverage = null; // has-a
    }
}
```

现在，让我们来看一下如何将调料（装饰者）添加到饮料（被装饰者）中。

首先是以上类图的完整实现，定义我们的“菜单”：

```js
class Beverage {
    // 抽象类
    constructor() {
        this.description = 'default';
    }
    getDescription() {
        return this.description;
    }
    cost() {/* 由子类实现 */}
}

class Decaf extends Beverage {
    constructor() {
        super();
        this.description = 'Decaf';
    }
    cost() {
        return 0.99;
    }
}

class Espresso extends Beverage {
    constructor() {
        super();
        this.description = 'Espresso';
    }
    cost() {
        return 1.99;
    }
}

// 调料装饰者抽象类
class AddonDecorator extends Beverage {
    constructor() {
        super();
        this.beverage = null;
        this.description = '';
    }
    getDescription() {
        // watch out: here starts wrap
        return this.beverage.getDescription() + ', ' + this.description;
    }
    cost() {}
}

class Soy extends AddonDecorator {
    constructor(beverage) {
        super();
        this.beverage = beverage;
        this.description = 'soy';
    }
    cost() {
        return this.beverage.cost() + 0.5;
    }
}

class Caramel extends AddonDecorator {
    constructor(beverage) {
        super();
        this.beverage = beverage;
        this.description = 'caramel';
    }
    cost() {
        return this.beverage.cost() + 0.3;
    }
}
```

接下来，让我们来看一下如何将调料（装饰者）添加到饮料（被装饰者）中。我们如何获得「双倍 Soy 的 Espresso」饮料呢？

![装饰者模式：双倍 Soy Espresso 示例图](/assets/img/2022/design-pattern-hf-03-01.png){: width="40%" height="40%" }

```js
// 首先，创建一个饮料基底 espresso
let ourCoffee = new Espresso();

// 接下来，添加一份 soy 调料
ourCoffee = new Soy(ourCoffee); // 包装
// 再添加一份 soy 调料
ourCoffee = new Soy(ourCoffee); // 再次包装

// 现在，检查一下我们的饮料构成：
ourCoffee.getDescription(); // Espresso, soy, soy
// 最后，计算价格：
ourCoffee.cost(); // 2.99
```

以上是整个系统的代码实现。现在，我们可以根据需要添加不同的饮料和调料，从而创建出无数种不同的饮料组合。

## 优缺点

优点：

-  满足开闭原则。不需修改已有代码，灵活地扩展功能；
-  将功能分解成更小、更具体的类，使得类的职责更加单一，更容易维护和扩展；
-  在运行时能灵活地增减功能，通过组合不同的装饰者，可以创造出大量不同行为的组合。

缺点：

-  由于装饰者模式功能过度分解、分解得过于细致，可能导致代码过于复杂化；
-  同样由于较多的细小的类，会导致方法调用和对象创建较为复杂，降低系统的运行效率。

## 现实应用

- **用户图形交互界面**。用装饰者模式为各种 UI 添加新特性，比如改变边界样式或颜色。
- **输入/输出流**。Java 的输入输出流使用装饰者模式控制。为基础的输入输出加入缓冲池（一个 bit 一个 bit 地读写 -> 缓冲池满后再统一读写，减少读写次数）或数据压缩功能。
- **日志记录**。在日志基础信息之上，添加时间戳或者额外的日志信息。
- **加密系统**。在已有加密系统上多增加一层加密层。
- **网络服务器**。 Express 应用里创建 http server 和利用 Socket.IO 初始化 WebSocket 协议：

```js
// 首先, 获得基础服务器 http server
let { httpServer } = require('http');

// 然后，加上 Websocket 协议
const { Server } = require('socket-io');
let socket = new Server(httpServer, options);

// 最后，开启 http server 监听
httpServer.listen(3000);
```

## 参考资料

- [Youtube: decorator pattern](https://www.youtube.com/watch?v=GCraGHx6gso&list=PLrhzvIcii6GNjpARdnO4ueTUAVR9eMBpc&index=3&ab_channel=ChristopherOkhravi)
- 《Head First 设计模式》第三章