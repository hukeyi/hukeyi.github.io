---
title: 【设计模式】详解单例模式
author: hukeyi
date: 2022-12-29 21:28:00 +0800
categories: [设计模式, 理论]
tags: [设计模式, javascript]
math: true
toc: true
---

单例模式是一个奇怪的设计模式：一个类**只有一个实例**，并提供全局接口来访问这个唯一实例。

> 部分人（[油管 up 主](https://www.youtube.com/watch?v=hUE_j6q0LTQ&list=PLrhzvIcii6GNjpARdnO4ueTUAVR9eMBpc&index=6)）的观点：永远别用单例模式。

## 问题场景

问：如何实现单例模式？如何保证类只有一个实例？

答：让类的构造函数私有。永远杜绝 `new ClassName()` 获得实例的方式。

问：既然类的构造函数是私有的，那么第一个且唯一一个实例又由谁来实例化呢？

答：由类的 static 成员函数。

问：什么是 [static 成员函数](https://www.geeksforgeeks.org/static-method-in-java-with-examples/)？

答：一个被所有类的实例共享，可以直接用类来调用，并且无论类实例化与否都存在的一个成员函数。

> The static keyword is used to construct methods that will exist regardless of whether or not any instances of the class are generated. Any method that uses the static keyword is referred to as a static method.
 
javascript 也有 [static](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Classes/static) 成员函数。

## 伪码实现

普通类直接 `new` 实例：

```js
// 普通的类获取实例的方式：
class Normal{
    constructor(){}
}
let normalInstance = new Normal();
```

单例模式通过奇怪的方式获得实例：

```js
class Singleton{
    private constructor(){
        this.theOneAndOnlyInstance; // static and private
    }
    public static getInstance(){
        // 第一次调用唯一实例尚未创建，则创建
        if (this.theOneAndOnlyInstance == undefined){
            this.theOneAndOnlyInstance = new Singleton();
        }
        // 返回唯一实例
        return this.theOneAndOnlyInstance;
    }
}
// 单例模式获取唯一实例的方式：
let singletonInstance = Singleton.getInstance();
```

## 优缺点

优点：

-  保证只有一个实例。由于单例模式只允许创建一个对象实例，可以避免在多次创建对象时产生额外的开销，节约了系统资源；
-  全局访问。由于单例模式只有一个实例，可以通过全局访问点轻松地访问该实例；
-  避免冲突。由于单例模式只有一个实例，避免了不同实例之间的冲突。

缺点：

-  可能引起全局变量的滥用。单例模式的实例是全局唯一的，如果不加限制地滥用全局变量，可能会带来一些问题，比如在不同的地方修改了该实例的状态，会影响到其他使用该实例的代码；
-  降低了代码的灵活性和可测试性。由于单例模式只允许创建一个实例，这意味着无法在运行时动态地替换实例，这可能会影响代码的灵活性和可测试性。同时，由于单例模式在全局范围内使用，它也可能会影响代码的可测试性。

## 现实应用

-  **需要严格控制实例数量的场景**。在某些情况下，由于系统资源有限或者其他一些原因，需要确保某个类只有一个实例。例如，数据库连接池、线程池等资源池都可以使用单例模式来管理资源的访问和分配。
-  **全局状态的共享场景**。在某些场景中，需要让多个对象共享同一个状态，这时可以使用单例模式来实现。例如，一个系统的配置信息、日志对象、用户信息等，都可以使用单例模式来实现全局的状态共享（e.g. Vue 2.x 源码中的 `Dep.target`）。
-  **避免创建多个相同对象的场景**。有些对象的创建和销毁的代价很高，比如文件系统、网络连接等，如果多次创建相同的对象会浪费系统资源。这时可以使用单例模式来避免创建多个相同的对象，从而提高系统的性能和效率。

## 参考资料

- [Youtube: singleton pattern](https://www.youtube.com/watch?v=hUE_j6q0LTQ&list=PLrhzvIcii6GNjpARdnO4ueTUAVR9eMBpc&index=6)
- 《Head First 设计模式》第五章
- [geeksforgeeks: Static Method in Java With Examples](https://www.geeksforgeeks.org/static-method-in-java-with-examples/)
- [MDN: static](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Classes/static)
