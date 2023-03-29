---
title: 【设计模式】策略模式的概念和代码讲解
author: hukeyi
date: 2022-12-04 16:37:00 +0800
categories: [设计模式, 理论]
tags: [设计模式]
math: true
mermaid: true
toc: true
---

## 简介

策略模式将不同的算法封装到各自的策略类中，使得它们可以互相替换，以达到动态改变对象的行为的目的。

## 问题场景

> 例子引自参考视频中的博主。

策略模式是为了解决以下类型的问题而提出的：

假设我们有一个超类鸭子，鸭子有两种行为：叫和飞。

```js
// 超类鸭子
class Duck {
	quack() { // 叫
		console.log('呱呱叫');
	}
	fly() { // 飞
		console.log('飞');
	}
}
```

我们可以继承超类鸭子，得到其子类番鸭、野鸭。每一种鸭子都需要实现叫和飞两种行为，每一种鸭子的叫声和飞行方式可能不同：

```js
// 番鸭，继承超类鸭子
class MallardDuck extends Duck {
	// 重写呱呱叫方法
	quack() {
		console.log('嘎嘎叫');
	}
	// 重写飞方法
	fly() {
		console.log('飞');
	}
}

// 野鸭，继承超类鸭子
class WildDuck extends Duck {
	// 重写呱呱叫方法
	quack() {
		console.log('嘎嘎叫');
	}
	// 重写飞方法
	fly() {
		console.log('飞');
	}
}
```

以下方法有两个问题需要解决：

### 问题 1: 子类与父类成员函数不一致

假设现在有一个类，名为玩具鸭，它只有鸭子的外形，但它只能叫不能飞。然而，由于它继承了超类鸭子，因此必须实现飞的成员函数，这该怎么办？

一种想法是，在飞行的成员函数内部实现不能飞行的逻辑。这种想法可行，但并不理想。

```js
// 宠物鸭，继承超类鸭子
class PetDuck extends Duck {
	// 重写叫
	quack() {
		console.log('哇哇叫');
	}
	// 重写飞
	fly() {
		console.log('不能飞');
	}
}
```

### 问题 2: 子类间存在重复的函数代码

上面的代码可以看出，番鸭和野鸭的叫声完全一致，但又与超类鸭子默认的叫声不同，两个子类都需要重载父类的呱呱叫函数，且重载后两类的函数代码完全一致。

你可能会说，不就是复制粘贴一下，问题不大。但当这个问题发生在现实的项目中，你面对的不只是两个重复的函数，而是若干个重复的函数。也就是说会出现若干段完全相同的冗余代码。

## 解决方案

既然每一种鸭子都有多种不同的实现方式来实现“叫”和“飞行”这两种行为，有时这些实现方式还会彼此重叠。那么，我们应该将这部分变化的代码抽象出来进行额外的处理。

简单来说，我们需要抽象出接口，每个接口都代表一种行为。我们可以完全放弃继承超类的子类，只需要一个超类和 n 个抽象接口类。超类代表的是一种“有 n 类行为”的概念，每一类行为有不同的策略。

我们还是以鸭子为例子。对于超类鸭子，我们可以抽象出两个接口类：“叫”和“飞”：

```js
// 定义鸭子叫的策略
class QuackStrategy {
	quack() {}
}

// 定义鸭子飞的策略
class FlyStrategy {
	fly() {}
}
```

### 解决问题 1

如何通过抽象接口类解决问题1呢？

以飞行为例。我们已经抽象出一个接口类叫“飞行”，其中可以包含“普通飞行函数”、“随便飞飞函数”等多种飞行方式。对于不能飞行的鸭子来说，直接命名一个“不能飞行函数”。

```js
// 定义普通飞行策略
class NormalFlyStrategy extends FlyStrategy {
	fly() {
		console.log('普通地飞');
	}
}

// 定义随便飞策略
class WildFlyStrategy extends FlyStrategy {
	fly() {
		console.log('随便地飞');
	}
}
// 定义不能飞策略
class NoFlyStrategy extends FlyStrategy {
	fly() {
		console.log('不能飞');
	}
}
```

为什么将“不能飞行”定义为一个函数是更好的选择，而不是重载超类的飞行函数并在其中实现“不能飞行”的逻辑呢？从函数语义化来看，父类的飞行函数隐含了鸭子会飞的含义，而在实现“不能飞行”的逻辑中却暗示了鸭子可以飞行，这是矛盾的。

用抽象接口类处理此问题在逻辑上更顺畅。因为“飞行”的抽象接口类的含义是“与飞行相关的行为/策略”，“不能飞行”就是一种与飞行相关的行为。

### 解决问题 2

问题 2 的解决是显而易见的。抽象接口类中把各种行为定义好，不同种类的鸭子需要哪种行为策略直接调用对应的成员函数就可以了，无需复制粘贴代码：

```js
const gua = new GuaQuackStrategy();
const fly = new NormalFlyStrategy();

const MallardDuck = new Duck(gua, fly); // 直接调用对应行为策略
const WildDuck = new Duck(gua, fly);
```

## 设计思想

策略模式的设计思想是将代码中变化的部分（行为策略）与不变的部分（超类定义）分离。

具体实现方式是，将父类中的每种行为抽象出来，由各个子类单独实现。而子类不直接继承父类，而是通过将各种行为的策略进行组合，等价于创建了一个新的子类。

实现策略模式需要以下两个步骤：

1.  实现不同的行为策略，即在父类中定义抽象方法，并由各个子类单独实现。
2.  将不同的行为策略组合成不同的类，这些组合后的类等价于一个新的子类。

通过策略模式，可以在不影响代码的整体结构的前提下，灵活地添加、删除、修改各种行为策略，从而使代码更加灵活和易于维护。

## 代码实现

首先是上文的鸭子例子，使用策略模式的完整代码：

```js
// 定义鸭子叫的策略
class QuackStrategy {
	quack() {}
}

// 定义鸭子飞的策略
class FlyStrategy {
	fly() {}
}

// 定义呱呱叫策略
class GuaQuackStrategy extends QuackStrategy {
	quack() {
		console.log('呱呱叫');
	}
}
// 定义嘎嘎叫策略
class GaQuackStrategy extends QuackStrategy {
	quack() {
		console.log('嘎嘎叫');
	}
}
// 定义哇哇叫策略
class WaQuackStrategy extends QuackStrategy {
	quack() {
		console.log('哇哇叫');
	}
}

// 定义普通飞行策略
class NormalFlyStrategy extends FlyStrategy {
	fly() {
		console.log('普通地飞');
	}
}

// 定义随便飞策略
class WildFlyStrategy extends FlyStrategy {
	fly() {
		console.log('随便地飞');
	}
}
// 定义不能飞策略
class NoFlyStrategy extends FlyStrategy {
	fly() {
		console.log('不能飞');
	}
}

// 定义鸭子类，使用策略模式
class Duck {
	constructor(quackStrategy, flyStrategy) {
		this.quackStrategy = quackStrategy;
		this.flyStrategy = flyStrategy;
	}

	// 呱呱叫
	quack() {
		this.quackStrategy.quack();
	}

	// 飞
	fly() {
		this.flyStrategy.fly();
	}
}

// User Code
const gua = new GuaQuackStrategy();
const wa = new WaQuackStrategy();
const fly = new NormalFlyStrategy();
const noFly = new NoFlyStrategy();

const MallardDuck = new Duck(gua, fly);
const WildDuck = new Duck(gua, fly);
const PetDuck = new Duck(wa, noFly);

MallardDuck.quack(); // 呱呱叫
MallardDuck.fly(); // 普通地飞

WildDuck.quack(); // 呱呱叫
WildDuck.fly(); // 普通地飞

PetDuck.quack(); // 哇哇叫
PetDuck.fly(); // 不能飞
```

另外，还可用于设计购物折扣策略：

```js
class ShoppingCart {
	constructor(discountStrategy) {
		this.discountStrategy = discountStrategy;
		this.items = [];
	}
	addItem(item) {
		this.items.push(item);
	}
	removeItem(item) {
		/* ... */
	}
	getTotal() {
		let total = 0;
		this.items.forEach((item) => {
			total += item.price;
		});
		return total;
	}
	getTotalAfterDiscount() {
		const discount = this.discountStrategy.calculateDiscount(this);
		return this.getTotal() - discount;
	}
}

// 折扣策略
class NoDiscount {
	// 无折扣
	calculateDiscount() {
		return 0;
	}
}

class FixedDiscount {
	// 固定折扣，比如减 50 元
	constructor(discountAmount) {
		this.discountAmount = discountAmount;
	}
	calculateDiscount() {
		return this.discountAmount;
	}
}

class PercentageDiscount {
	// 百分比折扣，比如打 5 折
	constructor(discountPercentage) {
		this.discountPercentage = discountPercentage;
	}
	calculateDiscount(context) {
		return context.getTotal() * (this.discountPercentage / 100);
	}
}

// User Code
const cart = new ShoppingCart(new NoDiscount()); // 设置无折扣策略
cart.addItem({ name: 'Item 1', price: 100 });
cart.addItem({ name: 'Item 2', price: 200 });

console.log(cart.getTotal()); // 300
console.log(cart.getTotalAfterDiscount()); // 300

cart.discountStrategy = new FixedDiscount(50); // 设置固定折扣减 50 元
console.log(cart.getTotalAfterDiscount()); // 250

cart.discountStrategy = new PercentageDiscount(10); // 设置百分比折扣打九折
console.log(cart.getTotalAfterDiscount()); // 270
```

## 优缺点

优点：

-   提高了代码的可扩展性和可维护性，策略类可以单独扩展或修改，而不影响其他部分的代码；
-   符合开放封闭原则，可以在不修改原有代码的情况下，增加新的策略实现；
-   提高了代码复用性，不同的上下文可以共享同一套策略，从而避免代码重复。

缺点：

-   策略模式需要用户自行选择合适的策略，如果策略类过多，可能会导致用户难以选择；
-   当策略类过多时，可能会占用过多的内存。

## 现实应用

策略模式可以应用在很多现实场景中：

-   **排序算法**：把各个排序算法用策略模式整合起来，用户根据自己对时空复杂度的需求，选择不同的排序算法。
-   **支付系统**：支付系统通常拥有不同的支付方式，比如银行卡、支付宝和微信支付等。用策略模式封装不同的支付方式，供用户在支付时选择。
-   **游戏系统**：游戏的人机对战模式，电脑水平的难易程度选择可以用策略模式封装，供用户选择。
-   **用户交互界面设计**：对于同一app，用户使用的屏幕尺寸、像素等会随着设备和用户的缩放变化，此时可以把不同尺寸、像素对应的布局方式用策略模式封装。在使用过程中，根据设备的变化动态变化。

## 参考资料

- [Youtube: Strategy Pattern](https://www.youtube.com/watch?v=v9ejT8FO-7I&list=PLrhzvIcii6GNjpARdnO4ueTUAVR9eMBpc&index=1)
- 《Head First 设计模式》第一章