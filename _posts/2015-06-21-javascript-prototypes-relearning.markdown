---
layout: post
title: "重新认识 JavaScript Prototypes"
date: 2015-06-21T09:13:17+08:00
author: George Sun
comments: true
categories: ["javascript"]
keywords: Getter Setter ES5 codethoughts.info codethoughts javascript 编程语言 JS
description: JavaScript  Getters and Setters

---

## 为什么要写这篇文章

前面的[一篇文章](http://codethoughts.info/javascript/2015/05/22/javascript-closure-inside-out/)，我讲过希望写JavaScript 语言的几个方面，其中一个知识点就包括 Prototype。另外，在这里我列一下我已经写过或者翻译过的 JavaScript 的文章便于检索。

* [关于 Angular.js 你需要知道的知识](http://codethoughts.info/angular.js/2015/05/15/things-i-wish-i-were-told-about-angular-js/)
* [JavaScript 闭包详解](http://codethoughts.info/javascript/2015/05/22/javascript-closure-inside-out/)
* [深入 JavaScript 模块化模式](http://codethoughts.info/javascript/2015/06/01/javascript-module-pattern-in-depth/)
* [寻找 JavaScript 的 this](http://codethoughts.info/javascript/2015/06/04/javascript-find-the-lost-this/)
* [JavaScript 属性描述符](http://codethoughts.info/javascript/2015/06/16/javascript-property-descriptors/)
* [JavaScript Getters 和 Setters](http://codethoughts.info/javascript/2015/06/18/javascript-getters-and-setters/)

Prototype 是 JavaScript 中很重要的一个知识点，是学习 JavaScript 要清楚理解的一个概念。比如 [Wikipedia 的 JavaScript 条目](https://en.wikipedia.org/wiki/JavaScript) 是这样来解释 JavaScript 语言的：

> JavaScript is classified as a [prototype-based](https://en.wikipedia.org/wiki/Prototype-based_programming) [scripting language](https://en.wikipedia.org/wiki/Scripting_language) with [dynamic typing](https://en.wikipedia.org/wiki/Dynamic_language) and [first-class functions](https://en.wikipedia.org/wiki/First-class_functions). This mix of features makes it a [multi-paradigm language](https://en.wikipedia.org/wiki/Multi-paradigm), supporting [object-oriented](https://en.wikipedia.org/wiki/Object-oriented_programming), [imperative](https://en.wikipedia.org/wiki/Imperative_programming), and [functional](https://en.wikipedia.org/wiki/Functional_programming) programming styles.

翻译一下：JavaScript 是基于 Prototype 的脚本语言，它是动态类型的，并且在 JavaScript 里函数是一等公民。这些混合的特性使得 JavaScript 成为了支持面向对象、命令式和函数式的多范型语言。

短短两句话，包含了很多内容。我们这里对多范型、动态类型等暂且按下不表，以后的文章可能会陆续涉及，因为它们都是比较大的主题。在这篇文章里我先跟大家大致的聊聊 Prototype，以及为什么 JavaScript 被称为是基于 `Prototype` 的语言，另外看看目前对于 Prototype 的使用一个比较大误区，也就是很多人和很多现存的类库利用 Prototype 来为 JavaScript 语言构建面向对象的继承和多态特性，当然了，这也可能是本文比较有争议的地方。

## Prototype 是什么
所有的 JavaScript 对象都有一个隐含的属性来指向另外一个对象，我们称这个隐含的属性为 `Prototype`。当然了这个属性有可能是 `null`，比如用这种方式创建的对象：

{%highlight js lineno%}
var myObject = Object.create(null);
{%endhighlight%}

但无论如何，除了特别指定的情况之外，JavaScript 的对象都有一个非空的 `Prototype` 来指向另外一个对象。关于 `Object.create(...)`，可以参考 [Mozilla MDN](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/create) 的解释，你也可以简单理解为它创建了一个新的 JavaScript 对象，并为新创建的对象和原对象之间建立了 `Prototype` 连接。接下来让我们看看常规的，也就是 Prototype 非空的情况：

{%highlight js lineno%}
var anotherObject = {
    a: 2
};

// create an object linked to `anotherObject`
var myObject = Object.create( anotherObject );

myObject.a; // 2

for (var k in myObject) {
    console.log("found: " + k);
}
// found: a

("a" in myObject); // true
myObject.hasOwnProperty("a"); // false
{%endhighlight%}

这段代码有几点需要注意的地方：

1. 在 [JavaScript Getters 和 Setters](http://codethoughts.info/javascript/2015/06/18/javascript-getters-and-setters/) 这篇文章中，我提到 JavaScript 的[[Get]]和[[Put]]操作。`myObject.a;` 就触发了[[Get]]操作，它首先在对象 `myObject` 上搜索，如果没有发现需要的属性，就会沿着 `Prototype` 链继续向上搜索，并在 `anotherObject` 上发现了需要的属性；
2. `for...in` 和 `in` 都会检查对象的 `Prototype` 链，而 `Object.hasOwnProperty(...)` 不会；
3. 对 `Prototype` 链的搜索到 `Object.prototype` 终止，在这个对象上定义了很多工具方法，比如 `toString()`, `valueOf()`, 上面提到的 `hasOwnProperty()` 等等。

## 为什么说 JavaScript 是基于 `Prototype` 的语言

根据 [Wikipedia 的解释](https://en.wikipedia.org/wiki/Prototype-based_programming)，基于 `Prototype` 的语言是面向对象编程得一种风格。和传统的面向对象语言不同，基于 `Prototype` 的语言借助于 `Prototype` 来进行代码复用。这里我们暂时抛开抽象的概念，先通过一段代码来描述 JavaScript 如何进行面向对象编程：

{%highlight js lineno%}
function Foo(name) {
    this.name = name;
}

Foo.prototype.myName = function() {
    return this.name;
};

var fooA = new Foo( "a" );
var fooB = new Foo( "b" );

fooA.myName(); // "a"
fooB.myName(); // "b"

function Bar(name,label) {
    Foo.call( this, name );
    this.label = label;
}

// here, we make a new `Bar.prototype`
// linked to `Foo.prototype`
Bar.prototype = Object.create( Foo.prototype );

// Beware! Now `Bar.prototype.constructor` is gone,
// and might need to be manually "fixed" if you're
// in the habit of relying on such properties!

Bar.prototype.myLabel = function() {
    return this.label;
};

var b = new Bar( "a", "obj a" );

b.myName(); // "a"
b.myLabel(); // "obj a"
{%endhighlight%}

这段代码模拟了传统的面向对象编程，在这里 `Foo` 作为父类，而 `Bar` 作为子类，`Foo` 和 `Bar`之间通过 `Prototype` 链来建立了继承关系。在这段代码里我们需要注意如下的几点：

1. `Bar.prototype = Object.create( Foo.prototype );` 创建了 `Bar` 和 `Foo` 的父子关系，在这行代码运行之前，`Bar.prototype` 指向 `Object.prototype`，这之后则指向了 `Foo.prototype`；
2. JavaScript ES6 引入了设置对象 `Prototype` 链的标准做法，它通过修改现有的 `Prototype` 来实现，而不是通过像 `Object.prototype`一样通过新创建一个对象来替换原有的对象来实现，同时它也保证了 `constructor` 的一致性，虽然在实际中 `constructor` 基本上没有什么用处；
3. 在每个 `Prototype` 链上都有一个 `constructor` 属性。这里我们不想花很多篇幅来讲述 `constructor`，如果你有兴趣可以参考一下 [Mozilla MDN 的一篇文档](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/constructor)，以后我也可能会用一片博客来讲述。这里我想说的是，`Bar.prototype = Object.create( Foo.prototype );` 会改变 `Bar.prototype` 的 `constructor`，如果你想保持一致性，可以用 `Bar.prototype.constructor = Bar` 来修正它；
4. 传统的面向对象语言，继承隐含的意义是拷贝，而 JavaScript 的继承隐含的意义是引用关系。比如 `var fooA = new Foo( "a" );`，如果在传统的面向对象则意味着类 `Foo` 中定义的方法和属性会被拷贝到对象 `fooA` 上，而在 JavaScript 中仅仅是把 `fooA` 的 `Prototype` 链指向了 `Foo`，没有任何拷贝发生，也就是说有部分函数(不是方法)是这个继承链上所有的实例共享的，一旦被修改了，会影响到所有的现有实例；
5. 在上面的 `Foo` 函数和 `Bar` 函数里出现了 `this`，如果你不理解，可以参考我之前的一篇文章：[寻找 JavaScript 的 this](http://codethoughts.info/javascript/2015/06/04/javascript-find-the-lost-this/)；

## JavaScript 不是“传统”的面向对象语言

这一节我们我们来讨论一下用 JavaScript 来模拟传统面向对象编程的弊端在哪里。

首先，JavaScript 没有真正意义上“类”的概念，即使看起来像，比如上面的代码片段里的 `function Foo(...)`，它仅仅是看起来像类，而事实上只不过是 JavaScript 中的一个对象而已，和传统的面向对象语言中的类完成的功能大相径庭。在传统的面向对象语言中，类定义了实例的模板和继承关系，在创建实例的时候把属性深度克隆到对象实例上；而 JavaScript 中的“类”仅仅是作为 `Prototype` 链的一环，并被该链上的“下游类”共享，它仅仅是一个对象，并不是真真意义上的类。比如：

{%highlight js lineno%}
function Foo() {
    // ...
}

var a = new Foo();

Object.getPrototypeOf( a ) === Foo.prototype; // true
{%endhighlight%}

这里，`new Foo()` 会创建一个新的对象叫做 `a`, `a` 被链接到 `Foo.prototype` 上。这段代码里，没有实例化任何类，也没有任何拷贝行为发生，仅仅是两个对象通过 `Prototype` 链被连接到了一起。事实上，我们通过 `Object.create(...)` 可以得到更直观的结果，关于 `Object.create()`，请参见 [Mozilla MDN 文档](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create)。

在 JavaScript 程序中很多误解的根源在于 `基于 Prototype 的继承` 这一术语。由于面向对象编程在现代程序设计中的巨大影响力，很多人一见到这一术语，就自然而然的把 `类`、`构造函数`、`实例`、`多态`等术语一个萝卜一个坑的往 JavaScript 代码里靠，这通常会得到令人沮丧的结果，也是很多人对于 JavaScript 这门语言怨声载道的根源。正如我们上面提到的，继承隐含的意义是拷贝，而 JavaScript 的 `基于原型的继承` 没有任何拷贝发生，它实际上上对象通过 `Prototype` 来作为另外一个对象的代理(Delegation)，这也是我们下一节要讲述的内容。

说到这里，可能有人会问，那我通过自己实现拷贝，不就可以解决这一问题了吗？比如很多 JavaScript 类库会提供的 `mixin` 方法，会手工的把父类的属性和方法拷贝到子类上：

{%highlight js lineno%}
// vastly simplified `mixin(..)` example:
function mixin( sourceObj, targetObj ) {
    for (var key in sourceObj) {
        // only copy if not already present
        if (!(key in targetObj)) {
            targetObj[key] = sourceObj[key];
        }
    }

    return targetObj;
}

var Vehicle = {
    engines: 1,

    ignition: function() {
        console.log( "Turning on my engine." );
    },

    drive: function() {
        this.ignition();
        console.log( "Steering and moving forward!" );
    }
};

var Car = mixin( Vehicle, {
    wheels: 4,

    drive: function() {
        Vehicle.drive.call( this );
        console.log( "Rolling on all " + this.wheels + " wheels!" );
    }
} );
{%endhighlight%}

上面的代码解决了部分问题，但是仍然存在和传统的面向对象集成较大的差异：

1. JavaScript 的函数没有真真意义上的拷贝，虽然在“子类”和“父类”上都存在一个叫做 `drive` 的方法，它不是真正的深度克隆，仅仅是两个变量同时引用到了同一个函数，也就是说一旦通过一个实例修改了该函数，其他现有的实例都收影响；
2. `Vehicle.drive.call( this );` 这行代码是非常丑陋的多态；
3. 通过 `Prototype` 实现的多态会导致`变量隐藏 (Variable Shadowing)`，它会导致很多难以发现的问题，需要尽量避免。关于变量隐藏，可以参考我的上一篇博客的最后一节：[JavaScript Getters 和 Setters](http://codethoughts.info/javascript/2015/06/18/javascript-getters-and-setters/)。

## 应该使用代理，而不是继承

很多人，在学习和使用 JavaScript 语言的时候，都会觉得它非常棘手。比如，在传统的面向对象语言中，几行代码就可以实现的继承、封装和多态，为什么到了 JavaScript 代码里需要这么多复杂的机制，为什么会有这么多“坑”？于是很多人在接触 JavaScript 语言后，迅速转向了某个 JavaScript 框架，期望利用框架这个黑盒子能够解决用 JavaScript 来进行自己所熟悉的面向对象编程的问题。但事实上，框架面临一样的问题，它只不过是把问题掩盖了，日后还会在某个地方浮现出来。

事实上，导致这些混淆的根源是 JavaScript 不应该被当做一门传统的面向对象编程语言。在 JavaScript 中，对象之间通过 `Prototype` 链来代理，其实在脑海中想象的 JavaScript 运行时内存应该是数不清的对象通过多个 `Prototype` 链被相互链接在一起。

下面我们通过两段代码来展示 JavaScript 中模拟传统的面向对象和对象代理之间的区别，这样你可以更好的体会对象代理的优势所在。

首先来看一段模拟传统面向对象的代码，我们成为 OO(Object Oriented)：

{%highlight js lineno%}
function Foo(who) {
    this.me = who;
}
Foo.prototype.identify = function() {
    return "I am " + this.me;
};

function Bar(who) {
    Foo.call( this, who );
}
Bar.prototype = Object.create( Foo.prototype );

Bar.prototype.speak = function() {
    alert( "Hello, " + this.identify() + "." );
};

var b1 = new Bar( "b1" );
var b2 = new Bar( "b2" );

b1.speak();
b2.speak();
{%endhighlight%}

这段代码应该不需要多余的解释了，下面是通过对象之间的代理实现同样的功能，我们称之为OLOO (Object Linked to Other Objects)：

{%highlight js lineno%}
Foo = {
    init: function(who) {
        this.me = who;
    },
    identify: function() {
        return "I am " + this.me;
    }
};

Bar = Object.create( Foo );

Bar.speak = function() {
    alert( "Hello, " + this.identify() + "." );
};

var b1 = Object.create( Bar );
b1.init( "b1" );
var b2 = Object.create( Bar );
b2.init( "b2" );

b1.speak();
b2.speak();
{%endhighlight%}

我们这里没有 `New`，没有类似 `类` 的结构，也没有类似 `继承` 的结构，仅仅是三个对象通过 `Prototype` 链通过*代理 (Delegation)*来实现类似的功能。相比上面一段代码在概念上要易于理解得多，并且，需要维护的代码也变少了，虽然在这里体现不明显。关于 `OLOO` 设计模式我可能会在以后的博客中详述，这里限于篇幅就不多谈了。

## 总结

面向对象仅仅是一种设计模式，虽然目前它仍然是主流，但是并不是编程语言与生俱来的东西；更重要的是，面向对象设计(OOP)和 JavaScript 语言存在很大的阻抗，并不是一种适合应用在 JavaScript 程序设计中的模式，在 JavaScript 程序中，运用OLOO (Object Linked to Other Objects) 模式可以设计出更简单，也更易于维护的代码，并且应该成为主流，关于 OLOO，可以参考 [You Don't Know JS 系列的这一章](https://github.com/getify/You-Dont-Know-JS/blob/master/this%20%26%20object%20prototypes/ch6.md)。

## 资源

[You Don't Know JS: this & Object Prototypes](https://github.com/getify/You-Dont-Know-JS/blob/master/this%20%26%20object%20prototypes)

[Mozilla MDN - Object.create()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create)

[Wikipedia - Prototype-based programming](https://en.wikipedia.org/wiki/Prototype-based_programming)

[Wikipedia - JavaScript](https://en.wikipedia.org/wiki/JavaScript)
