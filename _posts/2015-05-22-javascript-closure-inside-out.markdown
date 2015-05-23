---
layout: post
title: "JavaScript 闭包详解"
date: 2015-05-22T23:07:39+08:00
comments: true
categories: ["javascript"]
keywords: codethoughts.info codethoughts javascript closure programming language scope 闭包 作用域
description: JavaScript closure inside out
---

### 为什么写这篇文章
在实际项目中用 JavaScript 写了一些代码之后，我一直想着能用一种方式来把脑子里对 JavaScript 编程语言一些比较凌乱的知识点整理一下。所以接下来的几篇博客，我会用几篇文章分别解释 JavaScript 语言中我认为最为重要的几个概念，目前我想写的几个知识点分别是闭包(Closure),作用域(Scope), 原型继承(Prototype Inhertance), this 以及函数(Function)，如果以后我觉得有重要的知识还会继续写下去。这篇文章里我想先聊聊 JavaScript 的闭包，下面我们就直入主题。 

### 为什么要学习闭包
闭包是 JavaScript 中一个很重要的概念，简单讲，理解了闭包，你可以写出更好的 JavaScript 代码。另外，不管你是否意识到，只要你写过一些JavaScript 代码，你就一定写过闭包，至少读过别人写的闭包代码。很多重要的JavaScript 构建块(building blocks)会通过闭包来实现，比如代码模块化，单例模式，封装私有变量和函数，以及避免全局变量污染等等，下面我们会看到一些实例来说明这些应用场景。

### 闭包的定义
我们先来看一个关于闭包的定义：

> 当一个函数即便在离开了它的词法作用域(Lexical Scope)的情况下，仍然存储并可以存取它的词法作用域(Lexical Scope)，这个函数就构成了闭包。

很抽象？我也这么认为。至少从这个定义，你可以看出JavaScript 的闭包和作用域有紧密的联系。

### JavaScript 的作用域
作用域一般有两种模型，一种叫做词法作用域(Lexical Scope)，这就是 JavaScript 所采用的作用域模型；另外一种叫做动态作用域(Dynamic Scope)，比如 Perl，Bash Shell 等采用了这种模型，因为动态作用域和 JavaScript 无关，我们不做进一步解释，感兴趣的可以自己找[相关资料](http://en.wikipedia.org/wiki/Scope_(computer_science))。

词法作用域是静态的作用域模型，它在编写源代码的过程中已经确定了，和程序的调用者和运行上下文无关。具体到 JavaScript 来说，JavaScript 中有全局变量和局部变量，全局变量是在函数之外定义的变量或者函数内部没有用 var 关键字定义的变量，而局部变量则是用 var 关键字定义在函数内部的变量。在这里我想多说两句，在函数内部如果没有指定 var 关键字那么默认会创建全局变量；另一个问题是全局变量可以通过 window 对象来存取，很多人认为这是 window 对象上存储了一个全局变量的拷贝，其实不然，通过 window 存取的全局变量和不通过它访问的是同一个变量，指向同一块内存，就像是是一个硬币的正反面。

全局变量在任何地方都可以访问，而局部变量则不然，比如：
{%highlight js lineno%}
var global_var = "I'm global";
function A() {
    var local_var = "I'm local";
    console.log("global: " + global_var)
}
console.log("local: " + local_var); // 引用错误，类型是 ReferenceError: local_var is not defined
{%endhighlight%}
上面的引用错误说明在函数外部无法访问这里定义的`lcoal_var`。

### 嵌套的函数定义
我们只能通过变通的办法来访问函数的局部变量，一般来说，这个变通的办法就是在函数内部再定义一个函数，因为一个函数不仅可以访问全局变量，还可以访问它的外部函数(Outer function)定义的局部变量，比如下面的代码：
{%highlight js lineno%}
var global_var = "I'm global";
function outer() {
    var local_var = "I'm local";
    return function inner() {
        console.log("local: " + local_var);
    }; 
}
outer()(); // 输出：local: I'm local
{%endhighlight%}
函数 inner 不仅有自己的内部作用域，还可以访问全局变量，也可以访问它外部的 outer 函数定义的所有局部变量。我们知道函数是 JavaScript 的一等公民，可以作为其他函数的参数或者作为函数的返回值，这里我们把 inner 函数返回，这样我们就通过变通的方法在 outer 函数外部访问了 outer 函数内部定义的局部变量。那这里的内部函数 inner 就构成了闭包。在 JavaScript 语言中，闭包的定义可以简化为嵌套定义在函数内部的函数。

### 闭包和循环
在很多地方，比如 JavaScript 教程，论坛，书籍中，你会看到下面的一段代码：
{%highlight js lineno%}
for (var i = 1; i <= 5; i++) {
    setTimeout( function timer(){        console.log( i );    }, i*1000 );}
{%endhighlight%}
很明显，这个的 timer 函数是一个闭包。那么，这段代码的输出将会是什么呢？直觉上，一般人会认为这段代码会每隔一秒打印一个数字，数字分别是1, 2, 3, 4, 5. 但我可以负责任的告诉你，这个结论是错误的。如果你去运行这段代码，你会发现这段代码会每隔一秒打印一个数字6，一共5次.那为什么会这样呢？

首先来看，为什么是数字6？这段代码中循环的结束条件是`i <= 5`，另一方面，因为 JavaScript 是单线程执行，只有循环结束以后，setTimeout 中指定的函数才会被异步执行，也就是说在循环结束之前，setTimeout 没有被分配任何的 CPU 时间。

其次，为什么全都是数字6？直觉会认为，每个闭包都会有一份自己的变量拷贝，就像对此时运行的上下文的一个镜像一样。但事实上，JavaScript 闭包和作用域不是这样工作的。在上面的代码中，5次循环将会有5个函数被异步执行，这5个函数，每一个都指向同一个 `i` 变量，所以每次输出的 i 都是相同的值。

### 问题清楚了，如何解决呢
理解了问题所在，解决起来也就不难了。既然这5个异步函数指向同样的变量，那么我们只要能让他们都指向自己的那一份拷贝就可以了。如下代码所示：
{%highlight js lineno%}
for (var i = 1; i <= 5; i++) {
    (function() {
        var j = i;
        setTimeout( function timer(){                console.log( j );        }, j*1000 );    })();}
{%endhighlight%}
这里我们用一个立即执行函数表达式(Immediately-Invoked Function Expression (IIFE))，并在 IIFE 中声明了自己的本地变量`j`来获取并保存当时的 `i` 值。更优雅一些的解决方案是：

{%highlight js lineno%}
for (var i=1; i<=5; i++) {
    (function(j){        setTimeout( function timer(){            console.log( j );        }, j*1000 );    })(i);}
{%endhighlight%}
这里我们通过函数参数的形式来保存这个变量`i`的本地拷贝，也达到了同样的目的。

### 闭包的应用
在这篇文章里，我想列举两种闭包在 JavaScript 中的应用场景。

* 单例模式
* 代码模块化

首先来看单例模式。

#### 单例模式
但凡写过代码的人应该对[单例模式](http://en.wikipedia.org/wiki/Singleton_pattern)都不陌生，这里我也就不在多阐述了，直接上代码：
{%highlight js lineno%}
var Singleton = (function () {
    var instance;

    function createInstance() {
        return new Object("I am the instance");
    }
 
    return {
        getInstance: function () {
            if (!instance) {
                instance = createInstance();
            }
            return instance;
        }
    };
})();
 
function run() {
 
    var instance1 = Singleton.getInstance();
    var instance2 = Singleton.getInstance();
 
    console.log("Same instance? " + (instance1 === instance2));  
}
{%endhighlight%}

再来看模块化：

#### 代码模块化
{%highlight js lineno%}
function CoolModule() {
	var something = "cool";
	var another = [1, 2, 3];
	function doSomething() {
		console.log( something );
	}

	function doAnother() {
		console.log( another.join( " ! " ) );
	}

	return {
		doSomething: doSomething,
		doAnother: doAnother
	};
}

var foo = CoolModule();
foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
{%endhighlight%}

这就是 JavaScript 中模块化最基本的实现形式，上面展示的代码一般被称为暴露模块(Revealing Module)。代码模块化在大规模的 JavaScript 应用中很常见，也是必不可少的模式，如果没有模块化来解除代码之间的耦合，到了一定规模以后，代码会变得不可维护。关于 JavaScript 模块化模式的讨论，如果有兴趣了解的话可以参考[JavaScript Module Pattern: In-Depth](http://www.adequatelygood.com/JavaScript-Module-Pattern-In-Depth.html)。

### 资源
[Mozilla MDN - Closure](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Closures)

[学习Javascript闭包（Closure）](http://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html)

[You Don't Know JS: Scope & Closures](https://github.com/getify/You-Dont-Know-JS/blob/master/scope%20&%20closures/ch5.md)

[JavaScript Module Pattern: In-Depth](http://www.adequatelygood.com/JavaScript-Module-Pattern-In-Depth.html)