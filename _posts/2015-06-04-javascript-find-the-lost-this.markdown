---
layout: post
title: "寻找 JavaScript 的 this"
author: George Sun
date: 2015-06-04T19:23:10+08:00
comments: true
categories: ["javascript"]
keywords: codethoughts.info codethoughts javascript this programming language object rule 编程语言 JS
description: find the lost this in JavaScript
---

## 为什么写这篇文章
最近在看 [You Don't Know JS](https://github.com/getify/You-Dont-Know-JS) 系列一共6本书，说是我读过最好的 JavaScript 书籍不为过，[Amazon.com](http://www.amazon.com/s/ref=nb_sb_noss?url=search-alias%3Daps&field-keywords=you+don%27t+know+js) 上对6本几乎全五星的评价让所有的赞美都黯然失色，最关键的是，这一套书居然有免费、开源的版本！

这套书的第三本主要讲述了 JavaScript 中的 [this 和 Prototype](https://github.com/getify/You-Dont-Know-JS/tree/master/this%20%26%20object%20prototypes)，也是在这本书中，我第一次看到了如何用4个规则来断定 JavaScript 代码中当前的 `this` 对象到底指向谁。在这篇博客中，我想对原文进行一些归纳总结，既作为学习笔记，又可以帮助对 JavaScript 有兴趣的同行。

## 为什么要讨论 JavaScript 的 `this`
JavaScript 中的 `this` 是让新手甚至是使用 JavaScript 多年的老手经常感到困惑的地方。有的时候 `this` 指向的是全局对象，有的时候是当前的调用对象，在很多 JavaScript 类库中，`this` 经常被绑定到当前所操作的 DOM 对象上，比如 jQuery 对的事件绑定方法，这让 JavaScript 中的 `this` 显得很神秘，这篇博客的目的就是解开 `this` 那神秘的面纱。后面我们会一一讲述以上这几种情况，在讲解之前，我们先理解 JavaScript 代码的调用点(Call-site)和调用栈(Call-stack)。

## JavaScript 函数的调用点(Call-site)
顾名思义，函数的调用点就是函数被调用的地方，请注意，并不是被函数被声明的地方。来看这一段代码（来自[You Don't Know JS](https://github.com/getify/You-Dont-Know-JS/blob/master/this%20%26%20object%20prototypes/ch2.md)）：

{%highlight js lineno%}
function baz() {
    // 调用栈是 `baz`
    // 调用点是全局作用域
    console.log( "baz" );
    bar(); // <--bar 的调用点
}

function bar() {
    // 调用栈是: `baz` -> `bar`
    // 调用点是在 `baz` 内
    console.log( "bar" );
    foo(); // <-- `foo` 的调用点
}

function foo() {
    // 调用栈是 `baz` -> `bar` -> `foo`
    // 调用点在 `bar` 内
    console.log( "foo" );
}

baz(); // <-- `baz` 的调用点

{%endhighlight%}
上面的代码注释分别标出了三个函数的调用点和调用栈，如果你在函数 foo 里面打了断点，那么程序在断点暂停的时候可以清楚的看到调用栈。在理解 `this` 之前，先搞清楚 JavaScript 函数的调用栈很重要。

## 关于 `this` 的4个游戏规则
在 JavaScript 函数执行期间，`this` 指向的对象可以通过函数的调用点来确定。另外需要注意的是，这4个规则的优先级各不相同。接下来我们先看看这4个规则分别是什么，再确定它们的优先级。

### 规则1：默认绑定(Default Binding)
如果其他三个规则都不合适，那么就一定是默认绑定这种情况，在 JavaScript 代码中，单独的函数调用一定是这种情况。比如来看下面的代码：
{%highlight js lineno%}
function foo() {
    console.log( this.a );
}

var a = 2;
foo(); // 2
{%endhighlight%}
在上面这段代码中，函数的调用点是全局作用域，那么默认绑定会生效，`this` 指向全局对象。如果函数运行在[strict mode](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Strict_mode)下，情况会有不同吗？我们来看代码：
{%highlight js lineno%}
function foo() {
    "use strict";

    console.log( this.a );
}

var a = 2;
foo(); // TypeError: `this` is `undefined`
{%endhighlight%}
可以看到，在`strict mode`下，依然是默认绑定生效了，可是这时候 `this` 指向的不再是全局对象了，而是 `undefined`。

### 规则2：隐式绑定(Implicit Binding)
这个规则需要检查的是，当前函数的调用点是不是有上下文对象，或者被包含在某个对象之内，来看这段代码：
{%highlight js lineno%}
function foo() {
    console.log( this.a );
}

var obj = {
    a: 2,
    foo: foo
};

obj.foo(); // 2
{%endhighlight%}

这里我们可以看到，函数 `foo` 是在对象外部声明，然后被对象内的属性引用，其实这和在对象内部声明函数一样，不会对函数的调用点产生影响。因为函数 `foo` 被包含在对象 `obj` 内部，那么通过 `obj` 对象调用的时候，`foo` 函数的上下文对象就是 `obj`，在函数内部，`this` 也就被绑定到 `obj` 对象。

### 规则3：显式绑定(Explicit Binding)
在 JavaScript 中，所有的函数都可以调用 `call` 和 `apply` 来显式的绑定 `this`，比如：
{%highlight js lineno%}
function foo() {
    console.log( this.a );
}

var obj = {
    a: 2
};

foo.call( obj ); // 2
{%endhighlight%}
上面这段代码显式的把 `foo` 函数的 `this` 绑定到 `obj` 对象上。但是在实际场景中，仍然有一种情况，显式的绑定 `this` 仍然不能解决问题，比如看这段代码：

{%highlight js lineno%}
function foo() {
    console.log( this.a );
}

function doFoo(fn) {
    // `fn` 仅仅是对 `foo` 的另外一个引用

    fn(); // <-- 调用点!
}

var obj = {
    a: 2,
    foo: foo
};

var a = "oops, global"; // `a` 存在于全局范围

doFoo( obj.foo ); // "oops, global"
{%endhighlight%}

虽然这里我们`隐式`的把 `this` 绑定到 `obj` 对象上，但是很不幸，在函数作为参数传递的过程中，`this` 丢失了。在上面这段代码里，`obj.foo` 仅仅是对函数 `foo` 的一个引用，在 `obj.foo` 作为参数传递的时候，默认绑定会生效，`this` 也指向了 `window` 全局变量。为了解决这个问题，我们需要对显式绑定做一点改进。

{%highlight js lineno%}
function foo(something) {
    console.log( this.a, something );
    return this.a + something;
}

var obj = {
    a: 2
};

var bar = function() {
    return foo.apply( obj, arguments );
};

var b = bar( 3 ); // 2 3
console.log( b ); // 5
{%endhighlight%}

事实上，你也经常会在实际代码中见到这样的一个工具方法：

{%highlight js lineno%}
function foo(something) {
    console.log( this.a, something );
    return this.a + something;
}

// simple `bind` helper
function bind(fn, obj) {
    return function() {
        return fn.apply( obj, arguments );
    };
}

var obj = {
    a: 2
};

var bar = bind( foo, obj );

var b = bar( 3 ); // 2 3
console.log( b ); // 5
{%endhighlight%}
这个模式很常见，所以 JavaScript 自己也提供了同样的方法，可以通过 `Function.prototype.bind` 来调用：

{%highlight js lineno%}
function foo(something) {
    console.log( this.a, something );
    return this.a + something;
}

var obj = {
    a: 2
};

var bar = foo.bind( obj );

var b = bar( 3 ); // 2 3
console.log( b ); // 5
{%endhighlight%}

`bind()`返回了一个新的函数，这个函数的 `this` 被强制绑定到我们指定的任意对象上，这里是 `obj`。


### 规则4：通过 `new` 关键字绑定
JavaScript 中的任意函数，如果在调用的时候，前面 `new` 关键字，那么如下的几件事情会依次发生：

1. 新对象被创建；
2. 新创建的对象被加入到 `prototype` 链中；
3. 新创建的对象被设为该函数调用的 `this`；
4. 除非该函数指定返回其他对象，否则新创建的对象将会成为返回值

来看下面这段代码：

{%highlight js lineno%}
function foo(a) {
    this.a = a;
}

var bar = new foo( 2 );
console.log( bar.a ); // 2
{%endhighlight%}

`new foo(...)` 把新创建的对象绑定为 `this`，这也就是 `new` 绑定。

## 确定 `this`
在这里，对于如何给确定 `this` 的指向，我直接给出结论，感兴趣的可以到[这里](https://github.com/getify/You-Dont-Know-JS/blob/master/this%20%26%20object%20prototypes/ch2.md)查看详细的分析。

首先找出函数的调用点，据此分析上述四个规则哪一种被首先应用了，如果同时存在两种以上的规则，那么会应用优先级高的规则。

* 函数被 `new` 关键字调用了吗？如果是，那么 `this` 就是新创建的对象。
    {%highlight js lineno%}
    var bar = new foo()
    {%endhighlight%}
    
* 函数通过 `apply` 或 `call` 或 `bind` 显式绑定了吗？如果是，`this` 指向显式指定的对象。
    {%highlight js lineno%}
    var bar = foo.call( obj2 )
    {%endhighlight%}
    
* 函数通过隐式绑定调用了吗？如果是那么就是隐式调用的对象。
    {%highlight js lineno%}
    var bar = obj1.foo()
    {%endhighlight%}
    
* 如果以上都不是，那么就是默认绑定，`this` 指向全局变量或者 `undefined` (strict mode 下)。
    {%highlight js lineno%}
    var bar = foo()
    {%endhighlight%}
    
`this` 被应用的优先级按照这里对4条规则的检查顺序由高到低排列。
    
## 例外
在上述的四个规则之外，还有几种情况可以作为例外来处理，比如在显式绑定的时候，传入 `null` 或者 `undefined`，这时候 `this` 不会指向 `null` 或者 `undefined`，而默认绑定会生效。其他的少数情况可以参见[这里](https://github.com/getify/You-Dont-Know-JS/blob/master/this%20%26%20object%20prototypes/ch2.md)，限于篇幅，我就不一一列举了。

## 资源
[You Don't Know JS: this & Object Prototypes](https://github.com/getify/You-Dont-Know-JS/blob/master/this%20%26%20object%20prototypes)








