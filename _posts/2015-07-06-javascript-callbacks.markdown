---
layout: post
title: "JavaScript 回调函数"
date: 2015-07-06T21:10:08+08:00
author: George Sun
comments: true
categories: ["javascript"]
keywords: callback callbacks async codethoughts.info codethoughts javascript 编程语言 JS asynchronization 回调函数
description: JavaScript  Callbacks
---

## 为什么要异步

JavaScript 引擎是单线程运行的，它一次只能完成一个任务，如果有同时存在更多的任务必须排队等待前一个任务完成以后才可以分配到 CPU 时间。如果前一个任务占用了太多的 CPU 时间那么程序会出现无响应的状态，并阻塞后面的任务正常执行。针对这个问题，JavaScript 的解决方案是异步编程，而异步回调（async callback）则是 JavaScript 异步编程目前最为常用的模式。其他的异步编程模式有：事件监听，发布/订阅模式，Promise 等。本文不涉及异步回调之外的方式，以后的文章会相应的展开叙述，这篇文章会围绕 JavaScript 异步回调的特点，优点以及缺点展开。

## 事件循环（Event Loop）

如上所述，JavaScript 引擎是单线程运行的，它的任务很简单，在接收到执行命令的请求以后，每次运行一段 JavaScript 代码，请求执行JavaScript 代码的对象就是事件循环。

那么什么是事件循环呢？可以用如下伪码来帮助我们理解这个概念：

{%highlight js lineno%}
// 事件循环是一个先进先出的队列，我们这里用数组来模拟它
var eventLoop = [ ];
var event;

// 无限循环
while (true) {
    if (eventLoop.length > 0) {
        // 取出下一个事件
        event = eventLoop.shift();

        // 执行下一个事件
        try {
            event();
        }
        catch (err) {
            reportError(err);
        }
    }
}
{%endhighlight%}

上面的代码仅仅用来表示事件循环的概念，并不是真正的实现。我们可以看到，每次循环结束，JavaScript 引擎都会检查事件队列中是否还有未执行的事件，如果有那么从队列中取出并执行，这些事件事实上就是 JavaScript 的函数回调。

在 JavaScript 中有个很重要的细节需要注意，那就是 setTimeout() 函数。setTimeout() 函数并不会立即把函数回调加入到事件队列中，在运行到 setTimeout() 的时候，它会设置一个定时器，当定时器结束以后，JavaScript 宿主环境会把函数回调加入事件队列，随后JavaScript 引擎才有机会来调度 setTimeout() 的回调函数。

那加入此时有多个事件在队列中等待调度呢？此时，除了正在被调度的那个事件回调，其他的都必须等待，因为 JavaScript 是单线程的。这也是为什么 JavaScript 的 setTimeout 所设置的时间并不精确，总会根据当前事件队列的事件数量或多或少有延时。可以确定的一点是它绝对不会在定时器超时以前调用回调函数。

## 函数回调的执行

我们来通过一个例子解释一下函数回调的代码的执行顺序，仍然是 setTimeout()，如果换成 AJAX 请求也是一样的：

{%highlight js lineno%}
// A
setTimeout( function(){
    // C
}, 1000 );
// B
{%endhighlight%}

上面这段代码的执行顺序，用自然语言可以这样来描述：先执行 A，随后设置一个1000毫秒的定时器，然后执行 B，1000毫秒定时器超时以后，执行 C。请注意这里的细节，很多人在这里犯错。

另外，我们在这里可以看到 JavaScript 事件回调的一个缺点，它和我们人脑的思维方式相悖。当我们读代码的时候，是从上往下顺序捕捉代码，而事件回调则是先执行回调之后的代码，一段时间以后再去执行回调函数中的代码。

## 回调嵌套

我们来看一段在 JavaScript 中很常见的嵌套回调，也可以被称为回调金字塔：

{%highlight js lineno%}
listen( "click", function handler(evt){
    setTimeout( function request(){
        ajax( "http://some.url.1", function response(text){
            if (text == "hello") {
                handler();
            }
            else if (text == "world") {
                request();
            }
        } );
    }, 500) ;
} );
{%endhighlight%}

这段代码首先设置了点击事件回调，在这个回调中设置了定时器回调，随后在定时器回调函数中发出了一个 AJAX 请求，并为 AJAX 请求设置了相应回调函数。理解这段代码不算难，但是你可以想象一下如果有更多的回调代码会很难理解。

如果你不喜欢上面代码层层缩进，那么可以把它改写如下：

{%highlight js lineno%}
listen( "click", handler );

function handler() {
    setTimeout( request, 500 );
}

function request(){
    ajax( "http://some.url.1", response );
}

function response(text){
    if (text == "hello") {
        handler();
    }
    else if (text == "world") {
        request();
    }
}
{%endhighlight%}

现在缩进问题解决了，但上面的代码也仅仅解决了缩进问题。事实上，导致回调金字塔难以理解最大的原因并不是犹如千层饼似的一层一层的缩进，而是所设置的回调违背的人脑的直觉。上面的代码可以简化为如下模型：

{%highlight js lineno%}
doA( function(){
    doB();

    doC( function(){
        doD();
    } )

    doE();
} );

doF();
{%endhighlight%}

它的正确执行顺序是： doA() -> doF() -> doB() -> doC() -> doE() -> doD()，我想，很多人需要非常仔细才能正确的理解上面代码的执行顺序，代码的层层缩进只是回调金字塔的的现象，它本质上的问题是与人脑的直觉相悖。

## 回调的信任问题

除了上面的问题以外，回调还有更严重的问题需要解决，那就是信任问题。我们再来看一段代码：

{%highlight js lineno%}
// A
ajax( "..", function(..){
    // C
} );
// B
{%endhighlight%}

这里，A 和 B 都会立即执行，而 C 则会在 AJAX 响应返回以后执行。在这里我们把执行回调函数的主动权交给第三方，也就是 AJAX，我们失去了对它的控制权，这也会导致一系列的问题，事实上这也是 JavaScript 中基于回调驱动设计（Callback-Driven Design）的主要问题。

我们可以把上面的代码称作“控制反转（Inversion of Control）”，也就是我们把代码控制权交由第三方，基于此，我们和第三方代码之间达成了一个协定，我们期望代码可以根据这个协定来执行，但是由于我们失去了对代码的控制权，很多情况下代码并不完全根据这个既定的协定来执行。

我们通过另一个示例来进一步阐述，假设我们正在开发一个电子商务系统的结账功能模块，最终的付款功能我们一般是交由第三方系统来完成，比如支付宝，Aria 或者 Cybersource 等，通过它们，我们可以实现收费和订单追踪功能。假定他们提供了支付的回调接口，那么代码可能像下面这样：

{%highlight js lineno%}
analytics.trackPurchase( purchaseData, function(){
    chargeCreditCard();
    displayThankyouPage();
} );
{%endhighlight%}
我们把回调代码交由第三方的接口来完成，系统经过测试，看起来一切都很完美。但是实现支付的第三方代码在将来随时可能被改动，因为他们是第三方代码，而且控制被反转了，我们对它失去了控制权。假如由于第三方的代码在将来的代码中引入了 bug，他们多次调用了我们设置的回调函数，那么用户可能会被多次收费，也会导致出现完全相同的多个订单，这可是个大问题。那么你可能会把上面的代码稍微改动一下，变成这样：

{%highlight js lineno%}
var tracked = false;

analytics.trackPurchase( purchaseData, function(){
    if (!tracked) {
        tracked = true;
        chargeCreditCard();
        displayThankyouPage();
    }
} );
{%endhighlight%}

看起来解决了多次调用问题。新的问题来了，加入第三方代码因为新的 bug，从来没有调用我们的回调函数，我们该怎么应付？事实上，异步回调函数带来潜在的问题可能有很多：

* 过早调用了回调，在订单没有被追踪之前；
* 过晚或者压根没有调用回调函数；
* 过多或者过少的调用了回调函数；
* 没有传入回调函数所必须的参数；
* 吞掉了回调函数中的异常或者错误
* ...

到这里，你对控制被反转的回调函数带来的潜在问题可能有了新的认识，想解决这个问题，我们需要引入新的元素。下一篇文章，我们讨论一下我们如何用 JavaScript ES6（正式的名称是 ES2015）引入的 Promise 解决我们所面临的问题。

## 资源

[You Don't Know JS](https://github.com/getify/You-Dont-Know-JS/blob/master/async%20%26%20performance/)

[详解JavaScript异步编程的模式](http://www.hello-code.com/blog/javascript/201409/4015.html)















