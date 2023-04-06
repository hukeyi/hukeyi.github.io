---
title: 【设计模式】详解外观模式
author: hukeyi
date: 2023-01-17 14:27:00 +0800
categories: [设计模式, 理论]
tags: [设计模式, javascript]
math: true
toc: true
---

外观模式简化了系统与用户的交互流程。提供统一接口访问系统和其下的子系统的所有接口。用户只需要与外观接口交互就能使用整个系统。

## 问题场景

高解耦系统（所有类的设计遵循「单一责任原则」，即每一个类只负责最小的功能）通常拥有相互组合但解耦的数量众多的类。初始化某个系统可能涉及到较繁杂的实例化流程（因为类 A 可能需要类 B 的实例用于初始化，而类 B 又需要类 C 的实例初始化）。

> 复杂的实例化流程不代表高解耦系统是不好的，这只是高解耦特性必然会带来的结果。
{: .prompt-info }

外观模式是什么？我们创造出一个虚拟的壳（外观），把系统的所有类全部包裹其中。使用系统的用户只需要与这个壳交互。

用现实的例子来比喻，汽车的一键启动按钮和全自动洗衣机的智能启动按钮。

驾驶者不需要了解汽车的内部构造，只需要学会点火、操纵方向盘和操纵杆以及控制油门和刹车，就能正常驾驶了。点火按钮、方向盘、操纵杆、油门和刹车都是汽车系统提供给驾驶者的接口。

全自动洗衣机只需要你 1）放入衣物；2）放入洗涤剂；3）按下启动按钮。它自动包揽设置水量、加水、洗涤、漂洗和脱水等洗衣程序。智能启动按钮就是洗衣机提供给操作者的接口。

## 代码实现

```js
class Computer { // 电脑内部系统
    getElectricShock() {
        console.log('Ouch!');
    }
    makeSound() {
        console.log('Beep beep!');
    }
    showLoadingScreen() {
        console.log('Loading...');
    }
    closeEverything() {
        console.log('Closing everything...');
    }
    sooth() {
        console.log('Zzzzz');
    }
    pullCurrent() {
        console.log('Haaah!');
    }
}

class ComputerFacade { // 电脑外观，提供开关
    constructor(computer) {
        this.computer = computer;
    }

    turnOn() {
        this.computer.getElectricShock();
        this.computer.makeSound();
        this.computer.showLoadingScreen();
        this.computer.pullCurrent();
        console.log('Computer is on!');
    }

    turnOff() {
        this.computer.sooth();
        this.computer.closeEverything();
        this.computer.pullCurrent();
        console.log('Computer is off!');
    }
}

const computer = new Computer();
const computerFacade = new ComputerFacade(computer);

computerFacade.turnOn();  
computerFacade.turnOff(); 
```

输出：

```shell
Ouch!
Beep beep!
Loading...
Haaah!
Computer is on!

Zzzzz
Closing everything...
Haaah!
Computer is off!
```

## 得墨忒耳定律

A module should not know about the inner workings of the objects it manipulates.

只和你的密友谈话。

> [wiki: 得墨忒耳定律](https://zh.wikipedia.org/zh-tw/%E5%BE%97%E5%A2%A8%E5%BF%92%E8%80%B3%E5%AE%9A%E5%BE%8B)，又叫 The law of Demeter / Principle of least knowledge。
> 
> 1.  每個單元對於其他的單元只能擁有有限的知識：只是與當前單元緊密聯繫的單元；
> 2.  每個單元只能和它的朋友交談：不能和陌生單元交談；
> 3.  只和自己直接的朋友交談。
{: .prompt-info }

代码中表现为，减少连续调用链的长度：

```js
a.b().c().d(); // No!

a.d(); // Yes! d() 中调用 b() 和 c()
```

对于任何对象，在该对象方法内都只调用以下范围内的方法：

- 对象本身
- 对象的任何组件
- 此方法所创建或实例化的任何对象
- 方法的输入参数中传入的任何对象

总而言之就是要减少耦合。

## 参考资料

- [Youtube: facade pattern]() 
- 《Head First 设计模式》第七章