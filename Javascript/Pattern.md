# 改善前端项目代码质量笔记-JavaScript设计模式

在开发过程中，你有没有过写的时候畅快淋漓，等到回读自己的代码却不忍直视的经历：理解难、复用难、维护难。自己写过的代码过一段时间自己也弄不清为什么要这样写，逻辑思路理不清；代码高度耦合，新需求有相似功能提取可复用代码困难；添加个小功能要完整阅读理解整段代码以防引起意想不到的问题。

## 引入设计模式的目的

衣服一股脑地扔进衣柜当然是最简单的收纳方法，但时间久了衣服堆放的多了，就会发现很难从里面找到自己当天要穿的衣服。所以这时我们更多的会分门别类的叠好衣服放到固定的位置，甚至增加一些收纳箱用于整理收纳不同的衣物。

设计模式遵循着一条原则，即“找出程序中变化的地方，并将变化封装起来“。当我们找出可变的部分（每天变动的分类衣服）并把这部分封装（分类固定到不同位置或收纳箱内）起来，那么剩下的就是稳定不变（固定的分类收纳位置）的部分。设计模式确实可能带来更多的代码量（收纳箱），或许也会给系统带来额外的复杂度（分类衣服），不恰当的使用设计模式（错误分类）更可能使事情变得更糟。但软件开发的成本并非全部在初始开发阶段，很多项目是需要长期的迭代维护，设计模式能起到让人们写出可复用和高维护性程序的作用。

## 理解什么是设计模式

设计模式是前辈遵循单一职责原则、开放封闭原则、最小知识原则等指导思想在大量实践项目验证总结出来的最佳实践，类似于武功秘籍的招式。在遇到合适的场景施展起对应的招式，则恰到好处招招制敌。然而招式运用在乎融会贯通，如果生搬硬套为了用而用，反而会徒增代码复杂度，增加协作成本。

JavaScript是一种基于原型编程、多范式的动态脚本语言，支持面向对象、命令式和函数式的编程风格。传统的设计模式是从面向对象设计的角度出发的，因此并不是所有设计模式都适合JavaScript的，我们重点了解一些比较常见的设计模式在JS中的运用。

## 设计模式的开发实践

### 策略模式
策略模式是指其定义一系列的算法，把它们一个个封装起来，并且使它们可以互相替换。封装的策略算法一般是独立的，根据输入的使用方式调用对应的算法，目的在于将算法的使用与算法的实现分离开来。

618电商活动有各类打折促销活动需要计算商品成交价，简单的实现如下：
```js
function discountPrice(type, price) {
    if (type === "minus300_50") { // 满300减50
        return price - Math.floor(price / 300) * 50;
    } else if (type === "minus200_30") { // 满200减30
        return price - Math.floor(price / 200) * 30;
    } else if (type === "percent80") { // 8折
        return price * 0.8;
    } else if (type === "discount5") { // 立减5
        return price > 5 ? price - 5 : 0;
    }
}
discountPrice("minus200_30", 385); // 输出：355
```

随着折扣活动类型增加，计算函数将变得臃肿；折扣活动的变更需要通过修改计算函数来实现；其他地方用到相同的折扣计算方式不能复用。按策略模式改造一下，折扣算法提取到一个对象，通过折扣类型索引调用具体的算法执行计算。

```js
const strategies = {
    "minus300_50": function(price) {
        return price - Math.floor(price / 300) * 50;
    },
    "minus200_30": function(price) {
        return price - Math.floor(price / 200) * 30;
    },
    "percent80": function(price) {
       return price * 0.8; 
    },
    "discount5": function(price) {
        return price > 5 ? price - 5 : 0;
    },
}

function discountPrice(type, price) { // 单次折扣
    return strategies[type](price);
}
discountPrice("minus200_30", 385); // 满200减30

function multipleDiscountPrice(types, price) { // 多次折扣
    let result = price;
    types.forEach(type => {
        result = strategies[type](result);
    });
    return result;
}
multipleDiscountPrice(["minus300_50", "percent80"], 385); // 满300减50后再打8折
```

通过策略模式改造的代码，消除了原来函数里大片的条件分支语句；计算价格的算法分别封装在策略对象上，与执行函数分离开来，新增修改算法变得十分简单；如代码所示，算法在多次折扣函数中也得到复用。

### 职责链模式

职责链模式是使多个对象都有机会处理请求，将这些对象连成一条链，并沿着这条链传递请求，直到有一个对象处理它为止，从而避免请求发送者和接受者之间耦合。

还是上述电商活动，在此基础上优惠折扣有了额外的定金预定等限制和购买优先级：
type: 表示订单类型，1为minus300_50定金用户，2为minus200_30定金用户，3为discount5普通用户
pay: 表示用户是否支付定金，未支付定金回退为普通用户处理
stock：表示库存数量，定金用户不受此限制
```js
function order(type, pay, vip, stock) {  
    if (type === 1) { // 满300减50订单类型
        if (pay) { // 已支付定金
            const price = discountPrice("minus300_50", 385);
            console.log("需要支付" + price + "元");
        } else { // 未支付定金
            if (stock > 0) {
                const price = discountPrice("discount5", 385);
                console.log("需要支付" + price + "元");
            } else {
                console.log("库存不够");
            }
        }
    } else if (type === 2) { // 满200减30订单类型
        if (pay) {
            const price = discountPrice("minus200_30", 385);
            console.log("需要支付" + price + "元");
        } else {
            if (stock > 0) {
                const price = discountPrice("discount5", 385);
                console.log("需要支付" + price + "元");
            } else {
                console.log("库存不够");
            }
        }
    } else if (type === 3) { // 普通用户
        if (stock > 0) {
            const price = discountPrice("discount5", 385);
            console.log("需要支付" + price + "元");
        } else {
            console.log("库存不够");
        }
    }
}

order(1, true, false, 36); // 输出：需要支付335元
```

这段代码虽然能运行正常，但远远谈不上是一段好代码。order函数巨大难以阅读，大量条件语句耦合在一起修改维护困难。我们采用职责链模式重构这段代码，把满300减50、满200减30和普通用户分成3个函数链一起，请求在链条中传递直到能处理的节点。

```js
function order300_50(type, pay, stock) { // 满300减50订单函数
    if (type === 1 && pay === true) {
        const price = discountPrice("minus300_50", 385);
        console.log("需要支付" + price + "元");
    }  else {
        return "next"; // 约定该节点函数不能处理请求，返回字符串next通知传递给下一个节点
    }
}

function order200_30(type, pay, stock) { // 满200减30订单函数
    if (type === 2 && pay === true) {
        const price = discountPrice("minus200_30", 385);
        console.log("需要支付" + price + "元");
    } else {
        return "next";
    }
}

function orderNormal(type, pay, stock) { // 普通订单函数
    if (stock > 0) {
        const price = discountPrice("discount5", 385);
        console.log("需要支付" + price + "元");
    } else {
        console.log("库存不够");
    }
}

Function.prototype.after = function(fn) { // 用AOP实现职责链
    const self = this;
    return function() {
        const ret = self.apply(this, arguments);
        if (ret === "next") {
            return fn.apply(this, arguments);
        }
        return ret;
    }
}

const order = order300_50.after(order200_30).after(orderNormal);

order(1, ture, 36); // 输出：需要支付335元
order(2, ture, 36); // 输出：需要支付355元
order(3, false, 36); // 输出：需要支付380元
```

通过改进代码结构清晰很多，去掉了许多嵌套条件语句，每种订单类型的函数独立互不影响；链中的节点可以灵活的拆分重组，或增加或删除节点，或者改变节点的位置都是容易的。

## 未完待续部分

有用的设计模式还有很多等待学习整理：单例模式、中介者模式、发布-订阅模式、代理模式、命令模式等。除设计模式上，改善代码质量还可以从项目架构规范和一些常见而容易忽略的细节上入手。
