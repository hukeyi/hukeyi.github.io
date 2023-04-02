---
title: 【设计模式】详解工厂模式（工厂方法模式+抽象工厂模式）
author: hukeyi
date: 2022-12-08 19:38:00 +0800
categories: [设计模式, 理论]
tags: [设计模式, javascript]
math: true
toc: true
---

工厂模式有三个版本：

- ~~simple factory~~（不能被称为模式）
- factory method
- abstract factory

> 本文的披萨店例子参考《Head First 设计模式》的 Java 示例

## 问题场景

假设我们现在有一个披萨店，菜单上有三种披萨，根据顾客的实时订单选择创建相应的披萨实例：

```js
// 披萨的超类
class Pizza {
    constructor(){};
    prepare(){};
    bake(){};
    box(){};
}
// 菜单：
class BeefPizza extends Pizza {} // 牛肉披萨
class PorkPizza extends Pizza {} // 猪肉披萨
class ChickenPizza extends Pizza {} // 鸡肉披萨

// 披萨店的超类
class PizzaStore {
    constructor() {}
    // 顾客点菜，点的披萨种类为 type
    orderPizza(type) {
        let pizza;
        // 根据传入的 type 来实例化创建 pizza
        if (type === 'BeefPizza') {
            pizza = new BeefPizza();
        } else if (type === 'PorkPizza') {
            pizza = new PorkPizza();
        } else if (type === 'ChickenPizza') {
            pizza = new ChickenPizza();
        }
        
        pizza.prepare(); // 准备料
        pizza.bake(); // 烘烤
        pizza.box(); // 装盒
        return pizza; // 上菜
    }
}
```

在顾客点单前，我们无法获知应当创建哪个披萨的实例。从披萨这个例子抽象出来，就是「有些时候，究竟要实例化哪个类，要根据运行时实时传入的某些参数和条件决定」。

对于饭店来说，菜单会经常发生变化。一旦发生变化，我们就需要在 `PizzaStore` 中修改 `orderPizza` 函数，这会给代码的维护和更新制造难度。

以上问题有三种解决方案：

## 方案一：简单工厂

直接把那段创建对象代码抽出来，转移到一个单独的类中：

```js
class SimplePizzaFactory {
    constructor() {}
    createPizza(type) {
        let pizza;
        // 根据传入的 type 来实例化创建 pizza
        if (type === 'BeefPizza') {
            pizza = new BeefPizza();
        } else if (type === 'PorkPizza') {
            pizza = new PorkPizza();
        } else if (type === 'ChickenPizza') {
            pizza = new ChickenPizza();
        }
        return pizza;
    }
}
```

然后重写披萨店类：

```js
class PizzaStore {
    constructor() {
        // change：
        this.factory = new SimplePizzaFactory();
    }
    // 顾客点菜，点的披萨种类为 type
    orderPizza(type) {
        // change：
        let pizza = this.factory.createPizza(type);

        pizza.prepare(); // 准备料
        pizza.bake(); // 烘烤
        pizza.box(); // 装盒
        return pizza; // serve
    }
}
```

> 注意：**简单工厂并不是一种设计模式**，它只是一种编程习惯。毕竟，它只是把一段代码剪切粘贴到另一个地方。
{: .prompt-warning }

## 方案二：工厂方法模式

> factory method pattern

当披萨店要在不同地区开分店时，应该怎么做？

直接多次实例化 `PizzaStore` 吗？

假设每个地区的分店会根据不同地区人的口味差异微调菜单，此时又该如何修改代码呢？还是直接用 `SimplePizzaFactory` 类来创建披萨实例吗？不，这方法行不通了。因为不同地区的菜单种类、同样名称的披萨做法可能不同（比如四川麦当劳的麦辣鸡翅比广东的辣）。

这样的话，就为不同地区的分店都分别创建一个 `SimplePizzaFactory` ？四川的叫 `SimplePizzaSiChuanFactory`，广东的叫 `SimplePizzaGuangDongFactory`？假设生意做得特好，开了 100 个分店，那就需要写 100 个 `SimplePizzaFactory` 类，可以但麻烦。

更好的方法是下放决策权——超类 `PizzaStore` 不再决定如何制造实例（`PizzaStore` 不再负责实现 `createPizza(type)` ），而是把决定权交给披萨店分店，让它们自己决定菜单和菜谱（让 `PizzaStore` 的子类各自负责实现此函数）。这就是工厂方法模式：

```js
class PizzaStore {
    constructor() {}
    // 抽象方法，继承于 PizzaStore 的子类都必须实现此方法
    // 【createPizza 就是工厂函数】
    createPizza(type) {}
    orderPizza(type) {
        let pizza = this.createPizza(type);
        pizza.prepare();
        pizza.bake();
        pizza.box();
        return pizza;
    }
}

class SpicyPizza extends Pizza{} // 辣披萨
class OtherPizza extends Pizza{} // 其他披萨

class SiChuanPizzaStore extends PizzaStore { // 四川披萨店
    constructor() {
        super();
    }
    // 实现超类 PizzaStore 的抽象方法
    createPizza(type) {
        if (type === 'spicy') {
            return new SpicyPizza();
        } else {
            return new OtherPizza();
        }
    }
}
```

这几个类的使用方法：

```js
// 顾客走进一家四川披萨店
let store = new SiChuanPizzaStore();
// 顾客点单“辣披萨”，获得辣披萨
let pizza = store.orderPizza('spicy');
```

### 一句话定义

工厂方法是什么？

超类把实例化的决定下放给子类。超类定义一个创建对象的接口，让子类来决定要实例化哪个类。

## 方案三：抽象工厂模式

> abstract factory pattern

抽象工厂模式其实就是**拥有多个相关的创建实例函数**的工厂方法模式。

工厂方法的「工厂」是一个类中的成员函数（e.g. `createPizza()`），而抽象工厂中的「工厂」是一整个类（e.g. `class ModeFactory`）。

抽象工厂模式在什么时候能派上用场？油管 up 举的例子很好理解：app 的 light mode 和 dark mode。app 的 UI 会根据显示模式的转换，来转换主题颜色，这个应用场景非常适合使用抽象工厂模式。

假设 app 里有一个对话框，包含弹出窗口和按钮：

```js
// Products
class Alert { // 弹出窗口
    render() {}
}
class AlertLight extends Alert {
    render() {
        console.log('Light Alert. ');
    }
}
class AlertDark extends Alert {
    render() {
        console.log('Dark Alert. ');
    }
}

class Button { // 按钮
    render() {}
}
class ButtonLight extends Button {
    render() {
        console.log('Light Button. ');
    }
}
class ButtonDark extends Button {
    render() {
        console.log('Dark Button. ');
    }
}

// Factorys 工厂的抽象类
class ModeFactory {
    constructor() {}

    createAlert() {} // createProductA()
    createButton() {} // createProductB()
}
// concrete factorys
class LightModeFactory extends ModeFactory {
    constructor() {
        super();
    }

    createAlert() {
        return new AlertLight();
    }
    createButton() {
        return new ButtonLight();
    }
}
class DarkModeFactory extends ModeFactory {
    constructor() {
        super();
    }

    createAlert() {
        return new AlertDark();
    }
    createButton() {
        return new ButtonDark();
    }
}

// 工厂的客户，不一定非要是个类
class App {
    constructor(factory) {
        this.factory = factory;
    }
    createDialogBox() {
        let alert = this.factory.createAlert();
        let btn = this.factory.createButton();

        alert.render();
        btn.render();
    }
}

// 使用方法：建立一个暗黑模式的 app
let factory = new DarkModeFactory();
let app = new App(factory);

app.createDialogBox();
// Dark Alert. 
// Dark Button. 
```

控制台输出：

```shell
Dark Alert. 
Dark Button. 
```

## 参考资料

- [Youtube: Factory Pattern](https://www.youtube.com/watch?v=EcFVTgRHJLM&list=PLrhzvIcii6GNjpARdnO4ueTUAVR9eMBpc&index=4&ab_channel=ChristopherOkhravi)
- 《Head First 设计模式》第四章
- [维基百科：抽象工厂模式示例](https://zh.m.wikipedia.org/zh-hans/%E6%8A%BD%E8%B1%A1%E5%B7%A5%E5%8E%82)