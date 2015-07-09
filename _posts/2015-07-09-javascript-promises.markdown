---
layout: post
title: JavaScript Promises
date: 2015-07-09T10:38:12+08:00
author: George Sun
comments: true
categories: ["javascript"]
keywords: Promise Promises async codethoughts.info codethoughts javascript 编程语言 JS asynchronization 
description: JavaScript Promises

---

在前一篇博客[JavaScript 回调函数](http://codethoughts.info/javascript/2015/07/06/javascript-callbacks/)，我们提出了回调函数带来的潜在问题，在这里再次列举一次：

* 过早调用了回调函数；
* 过晚或者压根没有调用回调函数；
* 过多或者过少的调用了回调函数；
* 没有传入回调函数所必须的参数；
* 吞掉了回调函数中的异常或者错误。

这篇文章我们主要讲述 JavaScript 中如何用 Promise 来解决上面的问题。首先来看看 Promise 究竟是什么。

## 什么是 Promise

Promise 对象表示一个目前尚不可用，但将来某个时间点可以被获得的值。一个浅显易懂的比方是，比如你到一个店了买一杯咖啡，收银员收钱以后给了你一张收费凭据，那么虽然你没有马上得到咖啡，但在将来你可以凭借这个收费凭据得到它。

在你等待咖啡制作的过程中，你可以发微信给朋友，告诉她：我点了咖啡，你过来和我一起喝一杯吧。现在可能会出现两种结果：

1. 几分钟之后，你的咖啡冲好了，你可以用收费凭证换取咖啡，然后你就和朋友愉快的享用咖啡了；
2. 几分钟之后，服务生告诉你，咖啡卖完了，你和朋友失望的回去了。

上述的场景形象得描述了 JavaScript Promise。从中我们也可以看到，Promise 不仅仅可以表示成功获取到值的情况，也可以表示获取值失败的情况。

Promise 在被 ES2015 正式引入到 JavaScript 之前就已经流行起来了，比如[jQuery.Deferred()](https://api.jquery.com/jquery.deferred/), [Q.js](https://github.com/kriskowal/q), [BlueBird](https://github.com/petkaantonov/bluebird) 等第三方库都支持 Promise，他们和 ES2015 规范的 Promise 略有差异，我们这里不再一一列举它们。本文的 Promise 遵循  ES2015 的语法，鉴于 ES2015 还没有被浏览器大规模采纳，实际场景中可以考虑 [ES2015 Promise Polyfill](https://github.com/jakearchibald/es6-promise) 或者采用第三方库。另外要注意的是：ES2015 的 Promise 遵循[Promises/A+](https://github.com/promises-aplus/promises-spec)标准，而[jQuery.Deferred()](https://api.jquery.com/jquery.deferred/)和[Promises/A+](https://github.com/promises-aplus/promises-spec)标准有差异，语法和语义都有不同。如果你想进一步了解[jQuery.Deferred()](https://api.jquery.com/jquery.deferred/)，可以参考这一篇博文[Write Better JavaScript with Promises](http://davidwalsh.name/write-javascript-promises)。

先来看一个 Promise 的实例，请注意，这里是 [Promises/A+](https://github.com/promises-aplus/promises-spec) 的语法：

{%highlight js lineno%}
var promise = new Promise(function(resolve, reject) {
  // 这里通常会异步执行一些代码
  
  var result = ...

  if (result) {
    resolve("It worked!");
  }
  else {
    reject(Error("It broken"));
  }
});

promise.then(function(result) {
  console.log(result); // "It worked!"
}, function(err) {
  console.log(err); // Error: "It broke"
});
{%endhighlight%}

这里可以看到，我们可以通过Promise 构造函数来创建一个 Promise 实例，它接受一个带两个参数的函数作为回调。回调函数的两个参数中，第一个表示成功的回调，调用以后表示 Promise 成功取得期望的值；第二个表示失败的回调，调用以后，表示 Promise 没有取得想要的值。

得到了 `promise` 对象以后，我们通过它暴露的 `then` 接口来使用它。`then` 接受两个参数，第一个是成功的回调函数（被 `resolve` 后该回调函数将会被自动加入到 JavaScript 引擎的事件队列中），第二个是失败的回调函数（被 `reject` 后该回调函数会被自动加入 JavaScript 引擎的事件队列中）。关于事件队列，可以参考我的上一篇博客[JavaScript 回调函数](http://codethoughts.info/javascript/2015/07/06/javascript-callbacks/)，需要注意的细节是加入事件队列的回调函数都是异步调用。

## 解决回调函数的信任问题

知道了 Promise 是什么以后，我们来看看它怎么解决我们在本文开篇提出的信任问题。前一篇文章我们了解到，JavaScript 回调函数本质上是控制反转（Inversion of Control），那么我们的思路就是如何通过 Promise 把被反转的控制权再次反转回来，从而再次获得对代码的控制权。

### 回调被过早调用
对于这个问题，Promise 具有天生的免疫力。因为，即便是立即成功返回（被 resolve）的 Promise 也是被异步处理的：

{%highlight js lineno%}
new Promise(function(resolve){ 
    resolve(42);
}).then(function(v) {
    console.log(v);
});
{%endhighlight%}

上面的代码可以看到，即便 `Promise` 被构建之后立即被 `resolve`，我们也必须通过 `then` 接口来获取返回值。由于 JavaScript 的异步特性，传入 `then` 接口的回调函数总是被加入到事件循环队列中，并最早在下一个时间片中进行处理，所以不存在过早调用问题。

### 回调被过晚调用

通过 `then` 注册的回调函数在 `resolve` 或者 `reject` 之后，会自动被加入到事件循环队列中，并在下一个可能的时间片被异步调用。如果通过多个 `then` 接口注册了多个回调，那么这些回调函数被调用的顺序和它们被 `resolve` 的顺序相同。有因为通过 `then` 注册的回调函数式异步调用，所以一个回调函数不可能会阻塞另外一个回调。比如：

{%highlight js lineno%}
p.then( function(){
    p.then( function(){
        console.log( "C" );
    } );
    console.log( "A" );
} );
p.then( function(){
    console.log( "B" );
} );
// A B C
{%endhighlight%}

这里，根据 Promise 的特点和 JavaScript 的异步特性，C 不能发生在 B 之前。

### 回调没有被调用

我们首先需要知道的一点是，只要 `Promise` 被 `resolve` 或者 `reject` 了，那么通过 `then` 注册的两个回调函数之一一定会被调用。但是如果 `Promise` 从来没有被 `resolve` 或者 `reject` 呢？ES2015 标准给的答案是 `Promise.race(...)`:

{%highlight js lineno%}
// 一个用来使 Promise 超时的工具函数
function timeoutPromise(delay) {
    return new Promise( function(resolve,reject){
        setTimeout( function(){
            reject( "Timeout!" );
        }, delay );
    } );
}

// 为 `foo()` 设置超时
Promise.race( [
    foo(),                  
    timeoutPromise( 3000 )  // 3秒超时
] )
.then(
    function(){
        // `foo(..)` 及时 resolve 了
    },
    function(err){
        // `foo()` 可能被 reject 了，也可能是超时了
        // 需要查看异常信息来确定
    }
);
{%endhighlight%}

`race` 接受一个 Promise 实例组成的数组作为参数，其中一个 Promise 被 `resolve` 了，那么它也就随之被 `resolve` 了。上面的代码设置了3秒超时，如果在三秒内 `foo` 没有被 `resolve`，那么它会被强制超时，从而解除了 Promise 的 `pending` 状态。这里需要提一下的是 Promise 有三个状态，分别是 `pending`, `fullfill`, 和 `reject`，分别对应 Promise 被新建，被 `resolve`和被 `reject` 之后的三个状态。

### 回调被过多或者过少调用

过少一般就是没有被调用，这我们在上一节已经解释过了，这里我们只看看被过多调用的情况。Promise 只能被 `resolve` 或者 `reject` 一次，你可以多次 `resolve` 或者 `reject` 它，或者同时调用二者，但只有第一次调用会生效，后面的调用都会被忽略，这也就解决了多次调用的问题。

### 没有传入必须的参数

ES2015规范规定，Promise 最多只能接受一个参数，不管是 `resolve` 还是 `reject`。如果在 `resolve` 或者 `reject` 的时候没有传入参数，那么默认会给通过 `then` 注册的回调函数传入 `undefined`；如果 `resolve` 或者 `resolve` 的时候传入了多个参数，那么除了第一个以外的参数会被忽略。如果你的确想传入多个参数，那么要把多个参数封装为数组或者对象。来看一个例子：

{%highlight js lineno%}
function getY(x) {
    return new Promise( function(resolve,reject){
        setTimeout( function(){
            resolve( (3 * x) - 1 );
        }, 100 );
    } );
}

function foo(bar,baz) {
    var x = bar * baz;

    return getY( x )
    .then( function(y){
        // 通过数组封装需要传入的多个参数
        return [x,y];
    } );
}

foo( 10, 20 )
.then( function(msgs){
    // 把封装的参数取出来
    var x = msgs[0];
    var y = msgs[1];

    console.log( x, y );    // 200 599
} );
{%endhighlight%}

这样我们可以突破 Promise 只能传入一个参数的限制。

### 吞掉了异常或错误

如果你对一个 `Promise` 调用了 `reject`，并传入拒绝的理由，那么通过 `then` 注册的 `rejection callback handler` 会被调用，所传入的拒绝的理由也会被再次传入 `rejection callback`。

但如果在 Promise 实例创建过程中，或者在 `resolve` 之前，代码中有异常抛出，那么这个异常会被 Promise 捕捉到，并被传入 `rejection callback`，来看一个例子：

{%highlight js lineno%}
var p = new Promise( function(resolve,reject){
    foo.bar();  // `foo` 没有定义，抛出 TypeError!
    resolve( 42 );  // 这一行永远不会被调用
} );

p.then(
    function fulfilled(){
        // 这一行也不会走到
    },
    function rejected(err){
        // 这里的 `err` 将会使 `TypeError`
    }
);
{%endhighlight%}

Promise 成功的捕捉并 `reject` 了 Promise。另一个细节是，假如上面的代码中 `fulfilled` 回调函数抛出异常会怎么样？

{%highlight js lineno%}
var p = new Promise( function(resolve,reject){
    resolve( 42 );
} );

p.then(
    function fulfilled(msg){
        foo.bar();
        console.log( msg ); // 这一行不会被调用
    },
    function rejected(err){
        // 这一行不会走到
    }
);
{%endhighlight%}

这里是不是异常就会被默默的吞掉了呢？答案是不会如此。`Promise.then()` 仍然会返回一个 `Promise`，上面的代码会导致下一个 `Promise` 被 `reject`，我们可以通过下面的代码来捕捉这个异常：

{%highlight js lineno%}

var p = new Promise( function(resolve,reject){
    resolve( 42 );
} );

p.then(
    function fulfilled(msg){
        foo.bar();
        console.log( msg ); // 这一行不会被调用
    },
    function rejected(err){
        // 这一行不会走到
    }
)
.catch( handleErrors );
{%endhighlight%}

这就相当于 try{} catch() {} finally {} 写过 Java 的同学都很熟悉。那有的人会问，如果 `Promise.catch()` 里面抛出异常会怎么样，答案是 `Promise.catch()` 返回的 `Promise` 会被 `reject`，这也是我们在使用 Promise 的时候需要注意的地方之一。

## 进一步阅读

我们这篇文章并没有牵涉到 JavaScript Promise 的方方面面，仅仅是围绕着如何用 ES2015 引入的 Promise 来解决函数回调所遗留的问题。如果你希望了解 Promise 的方方面面，比如 API接口，如何流式调用 Promise，异常处理等，可以参考：[You Don't Know JS - Promises](https://github.com/getify/You-Dont-Know-JS/blob/master/async%20%26%20performance/ch3.md), [JavaScript Promises](http://www.html5rocks.com/en/tutorials/es6/promises/) 和 [MDN Promise 文档](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)。

## 资源

[You Don't Know JS - Promises](https://github.com/getify/You-Dont-Know-JS/blob/master/async%20%26%20performance/ch3.md)

[JavaScript 回调函数](http://codethoughts.info/javascript/2015/07/06/javascript-callbacks/)

[JavaScript Promises](http://www.html5rocks.com/en/tutorials/es6/promises/)

[jQuery.Deferred() 文档](https://api.jquery.com/jquery.deferred/)

[Write Better JavaScript with Promises](http://davidwalsh.name/write-javascript-promises)



