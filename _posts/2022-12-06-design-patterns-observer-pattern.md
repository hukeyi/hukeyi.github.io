---
title: 【设计模式】详解观察者模式
author: hukeyi
date: 2022-12-06 14:18:00 +0800
categories: [设计模式, 理论]
tags: [设计模式, javascript]
math: true
toc: true
---

观察者模式允许对象（订阅者）被通知订阅对象（发布者）的变化，并且根据订阅对象（发布者）的变化更新自身状态。

## 问题场景

假设有一个发布者 A，A 的状态经常改变；另有一个订阅者 B，B 想要实时获知 A 的当前状态。A 可能有多个订阅者，即 A <-> B1, A <-> B2, A <-> B3, ......, A <-> Bn，形成一对多关系。

我们应该用什么方法来实现这两者间的交流（A 状态变化 -> B 获知 A 的状态变化）？

## 方案比较

### 方案 1：轮询（Polling）

轮询，即“查询”。观察者对象 B 定期查询被观察对象 A 的状态是否发生变化。

轮询的缺点在于：

1.  当 A 的某两次状态变化时间间隔小于 B 的查询时间间隔时，B 会漏掉其中一次状态变化；
2.  当 A 很长时间都没有状态变化时，B 持续不断地查询操作就太过冗余。在实际应用中，A 可能拥有多个观察者 b1, b2, b3, ..., bn，如果每一个观察者都用轮询的方式频繁向 A 查询状态，A 的负荷就太大了。

### 方案 2：推送（Push）

推送，即“通知”。当 A 的状态变化时，A 主动通知订阅者 B：”我的状态改变了。“

推送解决了轮询的两个缺陷：

1.  A 一有状态变化，就通知 B，不会漏掉某次状态更新；
2.  A 只在状态变化时通知自己的订阅者们，订阅者们也只在获得通知时查询（或者 A 在通知的同时发送新数据）A 的当前状态，大大减少 A 的负荷。

推送即是观察者模式采用的解决方案。

## 常见问题

1.  观察者模式中，B 只能观察唯一一个 A 吗？No，B 可以同时观察多个发布者。
2.  观察者模式 = 发布者-订阅者模式吗？Yes，观察者模式也被称为发布者-订阅者模式。
3.  Vue 的响应式系统是观察者模式吗？Yes，Vue 的响应式系统就是一个典型的观察者模式实现。

## 代码实现

```js
class Subject {
    // 被观察者（发布者）
    constructor() {
        this.observers = []; // 观察 Subject 的观察者的列表
    }
    addObserver(observer) { // 添加观察者
        this.observers.push(observer);
    }
    removeObserver(observer) { // 删除观察者
        this.observers = this.observers.filter((obs) => obs !== observer);
    }
    notify() { // 通知更新
        this.observers.forEach((observer) => observer.update());
    }
}
class Observer {
    // 观察者（订阅者）
    constructor(name) {
        this.name = name;
        this.subject = null; // 通常会观察多个对象，一般设置为数组，这里简化处理
    }
    update() { // 响应更新
        console.log(`Observer ${this.name} notified of a change!`);
    }
    subscribe(subject) { // 关注/订阅
        this.subject = subject;
        this.subject.addObserver(this);
    }
    unsubscribe() { // 取消关注/取消订阅
        if (this.subject) this.subject.removeObserver(this);
        this.subject = null;
    }
}

// User code
const subject = new Subject();
const observer1 = new Observer('a');
observer1.subscribe(subject);
const observer2 = new Observer('b');
observer2.subscribe(subject);

subject.notify();
// Observer a notified of a change!
// Observer b notified of a change!
observer1.unsubscribe();
subject.notify();
// Observer b notified of a change!
```

## 优缺点

优点：

-  观察者模式实现了发布者与订阅者的解耦；
-  订阅者的增加和删除不会影响其他订阅者和它们订阅的发布者；
-  观察者模式实现了一对多关系，一个发布者可以通知多个订阅者。

缺点：

- 当一个发布者的订阅者过多时，可能会引起性能问题。因为一旦发布者变化，就需要通知其所有订阅者更新；
- 在实际应用中，订阅者通常可以订阅多个发布者，这会导致难以理清订阅者的更新时机，因为不是所有发布者更新，订阅者就必须跟着更新（Vue 使用全局变量 `Dep.target` 解决这个问题）；
- 订阅者的更新顺序难以理清（Vue 通过给各个 `watcher` 增加 id 解决这个问题。id 的大小按照 `watcher` 的初始化顺序排列，更新时先排序，再按 id 顺序更新）。

## 现实应用

- **Vue 的响应式系统**：Vue 中的数据对象被转换成可观察对象（subject），当数据发生变化时，所有订阅该对象的观察者（observer，在 Vue 中为 `Watcher` 类）都会收到通知并执行相应的操作
- **社交媒体**：微博等社交媒体软件使用观察者模式来实现用户订阅关系的实时更新。用户关注/订阅其他用户或话题，当被关注/订阅的用户或话题有新动态时，订阅者会收到通知并更新其时间线。
- **用户交互界面设计**：比如按钮点击、窗口拖拽、键盘按键等。GUI 应用程序（订阅者）通常需要响应用户的操作（发布者）。
- **数据监控和日志系统**：比如网站的访问日志，当有用户访问网站时，服务器（发布者）会将相关信息写入到日志文件（订阅者）中。

## 参考资料

- [Youtube: Observer Pattern](https://www.youtube.com/watch?v=_BpmfnqjgzQ&list=PLrhzvIcii6GNjpARdnO4ueTUAVR9eMBpc&index=2&ab_channel=ChristopherOkhravi)
- 《Head First 设计模式》第二章

