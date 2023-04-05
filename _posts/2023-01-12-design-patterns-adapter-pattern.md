---
title: 【设计模式】详解适配器模式
author: hukeyi
date: 2023-01-12 11:02:00 +0800
categories: [设计模式, 理论]
tags: [设计模式, javascript]
math: true
toc: true
---

适配器沟通了两种本不兼容的接口。

> adapter 电源适配器，就是充电头。别名 wrapper。e.g. 让中式插头能够用日式插座充电。

## 问题场景

假设我想在日本的咖啡店里使符合中国标准的笔记本电脑，则需要使用一个电源适配器：

> 电脑线的中式插头 => 适配器 => 咖啡店的日式插座

电源适配器解决电器/电线与插座的适配问题，我们不需要更换电脑充电线甚至更换电脑，也不需要换一个提供中式插座的咖啡店，只需要使用一个适配器就可以达到目的（不换插头不换电脑不换店实现充电）。

针对出国旅行，还有一种万能转换插头。无论是去欧洲国家，还是美洲国家，还是亚洲国家，万能转换插头上都有对应标准的插头。只要保证我们想要使用的电器电线插头能够插入这个适配器，就无需考虑面对的插座长什么样的问题了。

这个例子中，target（目标）就是电器电线（中式插头），adapter（适配器）就是万能转换插头，而 adaptee（被适配者）就是不同国家的插座口。

说回代码。代码中的适配器解决类与接口的适配问题。当某一个接口（中式插头）想要使用另一个接口（日式插座），但两者无法匹配上时，适配器模式（万能转换插头）就派上用场了：

> 用户想要使用的接口 => 适配器 => 被适配的接口

![适配器模式的类图](/assets/img/2023/design-pattern-ch07-01-00.png)
_截取自《Head First 设计模式》_

## 代码实现

```js
// 中式插头 target
class ChinesePlugTarget {
    constructor() {}
    charge() {}
}

// 日式插座 adaptee
class JapaneseOutletAdaptee {
    constructor() {}
    chargeWithJapanStandard() {
        console.log('Out with Japan standard. ');
    }
}

// 万能转换插头 adapter
// 这个继承关系表明，这是一个输入端已经插入了中式插头的适配器
class Adapter extends ChinesePlugTarget {
    constructor(adaptee) {
        super();
        this.adaptee = adaptee; // 被适配者（日式插座）
    }
    charge() {
        this.adaptee.chargeWithJapanStandard();
        console.log('In with China standard. ');
    }
}

// User Code
// 进入日本某个咖啡店，找到日式插座
let outlet = new JapaneseOutletAdaptee();
// 万能转换插头是中式插头的子类，继承关系表示中式插头已经插入万能转换插头
// 拿出（中式插头->）万能转换插头，并插入日式插座
let chinesePlugWithAdapter = new Adapter(outlet);
// 开始充电
chinesePlugWithAdapter.charge();
```

执行结果：

```shell
Out with Japan standard. 
In with China standard. 
```

### 对象适配器

上面的代码实现方式在《HF》中被称为「对象适配器」。意思是说，这是用「组合」而非「继承」的方式实现的适配器模式。

为什么是「组合」？

看上面的代码。`Adapter` 与 `Adaptee` 之间的关系是 has-a 关系，`adaptee` 是 `Adapter` 的一个数据成员：

```js
this.adaptee = adaptee;
```

### 类适配器

那么，是否存在另一种用**继承**方式实现的适配器模式呢？

Yes，这种模式叫类适配器。

什么叫用「继承」方式实现的适配器模式？

简单说，就是 `Adapter` 与 `Adaptee` 之间是继承关系，前者继承后者。Wait，`Adapter` 不是已经继承了 `Target` 吗？这就是所谓的「**多重继承**」。

可惜，JAVA 中没有多重继承，javascript 也没有。只看一眼它的类图：

![类适配器模式的类图](/assets/img/2023/design-pattern-ch07-01-01.png)
_截取自《Head First 设计模式》_

## 现实应用

- **接口适配器**。比如某软件中涉及多个接口，接口之间存在不兼容的情况，可以使用接口适配器将不兼容的接口转换为兼容的接口，以实现软件的正常运行。
- **类适配器**。比如某软件需要调用某类库中的一些类，但是这些类没有实现相应的接口，此时可以使用类适配器，将这些类适配成接口，以实现软件的正常运行。 
- **对象适配器**。比如某软件需要调用某库中的一些对象，但是这些对象没有实现相应的接口，此时可以使用对象适配器，将这些对象适配成接口，以实现软件的正常运行。

## 容易混淆的结构型设计模式

四个容易混淆的设计模式（比较四者区别的笔记待上传）：

- [适配器模式](https://hukeyi.github.io/posts/design-patterns-adapter-pattern/)
- 外观模式（待上传）
- 代理者模式（待上传）
- [装饰者模式](https://hukeyi.github.io/posts/design-patterns-decorator-pattern/)

## 参考资料

- [Youtube: adapter pattern](https://www.youtube.com/watch?v=2PKQtcJjYvc&list=PLrhzvIcii6GNjpARdnO4ueTUAVR9eMBpc&index=8) 
- 《Head First 设计模式》第七章