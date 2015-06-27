---
layout: post
title: "漫谈 JavaScript 函数"
date: 2015-06-22T17:41:48+08:00
author: George Sun
comments: true
categories: ["javascript"]
keywords: Functions Function 函数 ES5 codethoughts.info codethoughts javascript 编程语言 JS 高阶函数、Hoisting、原子性 函数作用域 模块化
description: JavaScript Functions

---

函数（Function）在 JavaScript 语言中处于核心地位，基本上你写的任何 JavaScript 代码都由一个或多个函数组成，所以理解函数是用好 JavaScript 编程语言的关键之一。这篇博客里我们来聊聊 JavaScript 函数的几个特性：高阶函数、Hoisting、原子性和函数作用域特性。

## 函数是 JavaScript 的一等公民

[函数式编程（Functional Programming）](https://en.wikipedia.org/wiki/Functional_programming) 正在成为主流，一些原来不支持函数式编程的语言最近也开始支持这种风格，比如说 Java8 引入的[闭包（Closure）](https://en.wikipedia.org/wiki/Closure_(computer_programming))支持使得 Java 语言在某种程度上可以支持函数式风格编程。这篇文章不会详细的讲述函数式编程，如果对使用 JavaScript 来进行函数式编程，可以参考[Functional JavaScript](http://shop.oreilly.com/product/0636920028857.do)。我想说的是函数式编程的一个重要特性就是函数作为一等公民，简单的说，函数可以被当做另外的函数参数（高阶函数），也可以作为返回值。JavaScript 从诞生的第一天起就支持高阶函数，也支持闭包。
JavaScript 中最常见的高阶函数是方法的回调，比如我们用 jQuery 来为一个 HTML 元素增加点击事件回调：

{%highlight js lineno%}
$("#btn_1").click(function() {
  alert("Btn 1 Clicked");
});

{%endhighlight%}

而函数作为返回值的经典示例就是闭包了：

{%highlight js lineno%}
function multiplier(factor) {
  return function(number) {
    return number * factor;
  };
}

var twice = multiplier(2);
console.log(twice(5)); // 10
{%endhighlight%}

闭包是 JavaScript 的一个重要特性，我之前写过一篇关于它的文章，可以参考：[JavaScript 闭包详解](http://codethoughts.info/javascript/2015/05/22/javascript-closure-inside-out/)。


## 函数声明提升（Hoisting）

JavaScript 的变量声明会被提升（Hoisting），JavaScript 引擎在执行的时候，会把变量的声明都提升到当前作用域的最前面，这个特性其实是由于 JavaScript 引擎在运行代码的时候，会先扫描一遍代码，实例化变量并赋值为初始值，第二遍才会真正去执行代码。关于细节，可以参考[You Don't Know JS: Scope & Closures - Hoisting](https://github.com/getify/You-Dont-Know-JS/blob/master/scope%20%26%20closures/ch4.md)，这也是一个需要很多篇幅的主题，本文不涉及详细内容，以后我可能会写相关的内容。

我们只是看看和函数相关的作用域提升，先看两段代码：

{%highlight js lineno%}
foo();

function foo() {
    console.log( 2 ); // undefined
}
{%endhighlight%}
这里我们可以看到函数声明被提升了，那么函数表达式呢？

{%highlight js lineno%}
foo(); // 抛出 TypeError 异常！

var foo = function bar() {
    // ...
};
{%endhighlight%}

这里可以看到两个现象，一是变量 `foo` 被提升了，二是抛出了 `TypeError`异常，说明 `foo` 变量已经存在了，但是它的值是 `undefined`。如果变量 `foo` 根本不存在，那么应该抛出 `ReferenceError`。 我们来验证一下：
{%highlight js lineno%}
var foo;

foo(); // TypeError
bar(); // ReferenceError

foo = function() {
    var bar = ...self...
    // ...
}
{%endhighlight%}

这段程序验证了函数表达式的变量被提升了，而且值会被初始化为 `undefined`，和普通变量的提升行为一致。

## JavaScript 函数是原子的（Atomic）

JavaScript 运行时是单线程执行的，它带来了一个有利的副作用是函数一旦开始执行，它的所有代码就一定会完整的运行，直至函数结束。在此期间，它不会被其他函数中断，所以 JavaScript 里也没有并发问题，也不需要类似 Java 那样复杂得内存模型（Memory Model）来定义一些列的前置和后置条件，因为一个语句执行的时候，不会有其他语句来修改内存中变量的值。

但是凡事都有例外，JavaScript 是单线程的这个表述也不例外。HTML5引入了Web Worker，用于处理一些耗时的后台任务，Web Worker 的实现一般是操作系统级别的独立线程。我们先来看一个例子：

{%highlight js lineno%}
var myWorker = new Worker("my_task.js");

myWorker.onmessage = function (oEvent) {
  console.log("Worker said : " + oEvent.data);
};

myWorker.postMessage("ali");
{%endhighlight%}

`my_task.js` 里的代码将会独立运行在一个线程里，并通过发送消息和启动 Web Worker 的代码通讯。`my_task.js` 代码如下：

{%highlight js lineno%}
postMessage("I\'m working before postMessage(\'ali\').");

onmessage = function (oEvent) {
  postMessage("Hi " + oEvent.data);
};
{%endhighlight%}

如果你写过诸如 Java 的多线程代码，你一定会有疑问 Web Worker 会不会引入多线程问题？答案是不会。因为 Web Worker 仅仅能通过消息来和宿主环境相互通信。它没有任何途径访问 DOM 或者非线程安全的变量。并且发送的消息中的对象会被序列化来得到一份拷贝，所以它并不会导致一般多线程的竞争问题。关于 Web Worker 可以参考 [Mozilla MDN 文档](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers)。

另一个例外是 ES6 引入的 Generator。一般认为，Generator 可以解决 JavaScript 代码的[回调金字塔问题](http://callbackhell.com/)，比如下面这样的Node.js代码：

{%highlight js lineno%}
fs.readdir(source, function(err, files) {
  if (err) {
    console.log('Error finding files: ' + err)
  } else {
    files.forEach(function(filename, fileIndex) {
      console.log(filename)
      gm(source + filename).size(function(err, values) {
        if (err) {
          console.log('Error identifying file size: ' + err)
        } else {
          console.log(filename + ' : ' + values)
          aspect = (values.width / values.height)
          widths.forEach(function(width, widthIndex) {
            height = Math.round(width / aspect)
            console.log('resizing ' + filename + 'to ' + height + 'x' + height)
            this.resize(width, height).write(destination + 'w' + width + '_' + filename, function(err) {
              if (err) console.log('Error writing file: ' + err)
            })
          }.bind(this))
        }
      })
    })
  }
})
{%endhighlight%}

代码的逻辑被层层覆盖，非常晦涩难懂。Generator 引入了 `yield` 关键字，可以在调用 `yield` 的地方暂停函数执行，并在 `yield` 的代码执行完成后重新回到原始的函数调用，另外原始的函数还可以通过 `yield` 来交换数据，来看一个例子：

{%highlight js lineno%}
function* idMaker(){
  var index = 0;
  while(index < 3)
    yield index++;
}

var gen = idMaker();

console.log(gen.next().value); // 0
console.log(gen.next().value); // 1
console.log(gen.next().value); // 2
console.log(gen.next().value); // undefined
{%endhighlight%}

可以看到，Generator 通过 `function*` 来定义。这里我们不对 Generator 过多的解释，它也是一个很大的话题，如果有兴趣的话可以参考[You Don't Know JS: Async & Performance - Chapter 4: Generators](https://github.com/getify/You-Dont-Know-JS/blob/master/async%20&%20performance/ch4.md)。这里我想说的是，Generator 会导致 JavaScript 函数的原子性受到影响，这是在实际代码中需要注意的地方。

## JavaScript 函数是变量作用域的边界

JavaScript 是没有块作用域的，只有函数才可以引入新的作用域，我们先来看一段代码：

{%highlight js lineno%}
for (var i=0; i<10; i++) {
    console.log( i );
}
console.log("i = " + i); // i = 10
{%endhighlight%}

我们在 `for` 循环里面定义的变量 `i` 在循环外部仍然是可见的，也就是说，在 JavaScript 里，代码块 `{}` 不是作用域的边界，这一点困惑了很多 JavScript 新手。如果你希望引入新的作用域，那么必须求助于函数：

{%highlight js lineno%}
var a = 2;

function foo() { // <-- insert this

    var a = 3;
    console.log( a ); // 3

} // <-- and this
foo(); // <-- and this

console.log( a ); // 2
{%endhighlight%}

这里可以看到函数 `foo` 内部的变量 `a` 和外部定义的同名变量是完全不同的两个变量。上面代码的问题在于，它解决了作用域问题的同时，也引入了一个多余的新函数，并且函数需要一次额外的调用。解决这个问题需要借助于 `自调用匿名函数（Immediately Invoked Function Expression）`，也可以简称为 IIFE。上面的代码被改写如下：

{%highlight js lineno%}
var a = 2;

(function foo(){

    var a = 3;
    console.log( a ); // 3

})();

console.log( a ); // 2
{%endhighlight%}

IIFE 在 JavaScript 代码中有很多非常有用的应用，比如说 jQuery 为了避免命名冲突，通常会这么写：

{%highlight js lineno%}
jQuery.noConflict();
 
(function( $ ) {
    // 可以安全的使用 $ 了
})( jQuery );
{%endhighlight%}

通过 IIFE，也可以轻松在 JavaScript中引入代码模块管理：

{%highlight js lineno%}
var counter = (function(){
	var i = 0;
	return {
		get: function(){
			return i;
		},

		set: function( val ){
			i = val;
		},

		increment: function() {
			return ++i;
		}
	};
}());
{%endhighlight%}
这里也应用了闭包，关于闭包可以参考我的一篇博客：[JavaScript 闭包详解](http://codethoughts.info/javascript/2015/05/22/javascript-closure-inside-out/)；关于 JavaScript 模块化，可以参考我的另一篇博客：[深入 JavaScript 模块化模式](http://codethoughts.info/javascript/2015/06/01/javascript-module-pattern-in-depth/)。

## 资源

[You Don't Know JS: Scope & Closures](https://github.com/getify/You-Dont-Know-JS/blob/master/scope%20&%20closures)

[You Don't Know JS: Async & Performance - Chapter 4: Generators](https://github.com/getify/You-Dont-Know-JS/blob/master/async%20&%20performance/ch4.md)

[Callback Hell](http://callbackhell.com/)

[Mozilla MDN - function\*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function\*)

[Mozilla MDN - Using Web Workers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers)

[Wikipedia - Functional Programming](https://en.wikipedia.org/wiki/Functional_programming)

[Eloquent JavaScript - Functions](http://eloquentjavascript.net/03_functions.html)

[JavaScript 闭包详解](http://codethoughts.info/javascript/2015/05/22/javascript-closure-inside-out/)
