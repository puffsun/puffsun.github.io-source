---
layout: post
title: "关于 Angular.js 你需要知道的知识"
date: 2015-05-15T19:50:20+08:00
author: George Sun
comments: true
categories: ["Angular.js"]
keywords: codethoughts.info codethoughts Front-end UI CSS LESS Angular AngularJS Angular.js Framework
description: Things I Wish I Were Told About Angular.js
---

### 译者写在前面的话
最近时间一直比较紧张，没有足够的时间深入学习任何东西，但是博客更新频率还是要保证的。这里挑了一篇 Angular.js 的好文来翻译，不仅可以温故而知新，也能帮助希望深入学习 Angular.js 框架的同行。想看原文请移步[Things I Wish I Were Told About Angular.js](http://ruoyusun.com/2013/05/25/things-i-wish-i-were-told-about-angular-js.html)。如需转载译文，请标明出处。

翻译说明：原文中关于 Angular.js 的术语，比如 Controller, Module, Directive 保持英文不翻译，我认为这样便于理解。

### 译文
最近我参与了一个使用了 Angular.js 的项目。到我写这篇文章的时候，这个项目达到了中等规模（大约10个 Module，20个 Controller, 5个 Service, 10个 Directive 左右），单元测试的覆盖率也非常不错。当我回过头来看，发现跟以前相比，我学习了多得多的关于 Angular.js 的知识。整个过程并不是顺风顺水，在整个过程我重构并重写了很多代码，而且有很多事情我希望在我开始使用 Angular.js 之前就有人告诉我了。

提醒：这篇文章基于 Angujar.js 1.0.x 稳定分支，如果你目前正在使用更新的版本，其中的部分内容可能并不适用。

### 关于学习曲线
Angular.js 的学习曲线和 Backbone.js 非常不同。Backbone.js 在开始学习的时候有很高的学习曲线，在写一个简单的应用之前，你需要知道几乎所有的关于 Backbone.js 的知识。如果你搞定了这些，那么关于 Backbo.js 你需要了解的也就剩下一些常见的构建块(Building Blocks)和最佳实践了。

然而，Angular.js 非常不一样，刚起步的时候几乎没有障碍，浏览一下 Angular.js 主页上的一些示例，你就可以起步了。这个时候，你不需要理解它的核心概念，比如：Module, Service, Dependency Injection, Scope. 有了 Controller 和一些 Directive 你就可以开始了，而且它的文档对于快速上手 Angular.js 也足够好。

问题是当你深入 Angular.js，打算开发一些正式的应用，它的学习曲线会陡增，在这个时候，你会发现 Angular.js 的文档要么不完整，要么过于繁琐。要想跨越这个阶段，并且真正从所学的知识在项目中获益需要不少时间。这也是我希望一开始就有人告诉我的第一件事，至少我在遇到那么多问题的时候不会倍感挫折。

### 在一开始就理解 Module
Angular.js 不会强制你使用它的 Module 系统，所以在一开始，我只写了一个巨大的 CoffeeScript 文件，里面包含了一堆 Controller。很快，这个巨大的源文件失控了，你几乎没有办法浏览整个源文件；另一方面，即使只用到了其中一个 Controller，我也必须把这个大文件加载到每一个页面。

在这种情况下，我停止写新代码，并着手重构这些代码。第一步，根据不同的页面，把每个 Controller 拆分到不同的 Module 中，然后抽取了一些公用的 Filter 和 Service。随后，我确保所有的模块都正确的设置了依赖，并且所有的 Components 都成功的注入了。当然，在这个阶段，你必须理解依赖注入（Dependency Injection）：一个在 Java 和 C# 社区里很流行的概念，可在动态语言的社区中却不是如此。幸运的是（有些人可能是恰好相反），我写过不少 Java 和 C#代码，理解 Angular.js 的依赖注入没有花我太多的时间 - 即便是第一次在 JavaScript 代码中使用依赖注入显得不太自然。一旦你理解了依赖注入，你一定会爱上它，它会使你的代码低耦合，容易维护。

如果你不想大规模的重构你的 Angular.js 代码，那么你应该在一开始学习并且计划你的代码模块。

### 当你的 Controller 过大的时候，学习 Scope
我确信你在一开始使用 Angular.js 就会使用`$scope`, 但是你可能并没有完全理解它，其实你不必完全理解它，你可以把它当做一个魔法。然而，这里有些你可能会遇到的常见问题：

* 为什么我的 View 没有被更新？

{%highlight js lineno%}
function SomeCtrl($scope) {
	setTimeout(function() {
		$scope.message = "I am here!";
	}, 1000);
}
{%endhighlight%}

* 为什么`ng-model="touched"` 不工作？

{%highlight js lineno%}
function SomeCtrl($scope) {
	$scope.items = [{ value: 2 }];
	$scope.touched = false;
}
{%endhighlight%}

{%highlight html lineno%}
<ul>
	<li ng-repeat="item in items">
		<input type="text" ng-model="item.value">
		<input type="checkbox" ng-model="touched">
	</li>	
</ul>
{%endhighlight%}
这些都和`$scope`有关。更重要的是，当你的 Controller 变得越来越大的时候，你应该把它分割成子 Controller，Controller 之间的继承关系也和`$scope`紧密相关。那么你需要理解`$scope`是如何工作的：

1. 那些神奇的数据绑定(Data Binding)是如何和`$scope.$watch()` 和 `$scope.$apply()`扯上关系的；
2. `$scope`是如何在父子 Controller之间共享的；
3. 什么时候 Angular.js 会偷偷的创建一个新的`$scope`；
4. 在`$scope`之间，这些事件（`$scope.$on`, `$scope.$emit`, `$scope.$broadcast`）是如何在`$scope`之间通信的；

一开始的两个问题答案分别是：

* 你需要`$scope.$apply()`
* Angular.js 在 `ng-repeat`中偷偷的创建了一个新的 Scope

### 当你尝试在 Controller 中操作 DOM 的时候，写一个 Directive 吧
Angular.js 提倡分离 Controller 和 View(DOM)，但有些时候，你可能需要在 Controller 中存取 DOM，一个典型的例子是 jQuery 插件。虽然 jQuery 插件看起来有点“过时”了，但是它的生态系统并不会很快消失。在 Controller 中使用 jQuery 插件绝对是个坏主意，这样做会使得这个 Controller 难以重用，难以测试。

这时候，最佳做法是写一个 Directive。我非常确信你用过 Directive，但是你可能不会想到你可以自己写一个。事实上，Directive 是 Angu.js 里最强大的概念，你可以把它想象成可重用的组件，不仅对于 jQuery 插件，而且是对于任何在多个地方使用的子 Controller而言，看起来有点像[Shadow DOM](http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom/)。最神奇的地方在于，你的 Controller 不需要知道 Directive 的存在，通过 Scope 共享和`$scope`事件可以达到这个目的。

不幸的是，Directive 的文档可以说是 Angular.js 文档中最差的一部分，其中满是术语和抽象的、没有上下文的解释，只有很少的实例，所以如果你不能完全理解 Directive 也别被吓到了。我过了这一关的办法是开始写自己的 Directive，即使是在没有完全理解文档的情况下。当我处于不知道如何是好的阶段的时候，我就回到文档或者去 Google 一番。在我成功的把 jQuery 插件封装到自己的 Directive 之后，我几乎掌握了关于 Directive 的大部分概念，通过实践来学习应该是在没有好文档的情况下最有效的学习方式了。

如果你们想找到一字箴言，那么我的建议是：把控制逻辑放入 Directive Controller，DOM 逻辑放在链接过来的函数(function)中，二者通过 Scope 共享来结合。

### Angular.js 的路由(Router)可能跟你期待的不一样
这是一个我花了好些时间才爬出来的坑。Angular.js 的路由有特定的用途，它不像 Backbone.js 的路由那样，监听`location.hash`，并在路由匹配的时候调用相应的函数。Angular.js 的路由就像服务端的路由一样，用来和`ng-view`一起工作。当路由被触发的时候，一个预定义的模板(template)被加载并被注入到`ng-view`中，与此同时，一些 Controller 会被实例化。如果你不喜欢这个解决方案，那么你只能开发自定义的 Service (把已有的类库封装到 Angular.js 的 Service 中去)。

这并不是一个通用的办法，仅仅是一个特定的解决方案。然而，我在这里花了大量的时间，希望这个提示可以节约你一些时间。

### 关于测试
Angular.js 提倡分离 View 和 Controller，所以给 Controller 写单元测试就是一件非常容易的事情，因为你不需要和 DOM 打交道。写单元测试的意图并不是要证明你的应用可以无误的运行 - 这是集成测试(Integration Test)的目的。单元测试只是用来在集成测试失败的时候快速定位有 bug 的代码，唯一需要和 DOM 打交道的单元测试是在 Directive 中。

因为有依赖注入，在给 Controller 写单元测试的时候，你可以很容易 Mock 任何依赖，这也使得测试的断言简单很多。一个典型的场景是在实例化 Controller 的时候使用 Mock，另一方面，Angular.js 的依赖注入是`以最后一个为准(last win rule)`，你可以通过在测试中注册一个假的实现重新绑定（覆盖）其依赖。

Angular.js 还有 E2E 测试，虽然设置和运行起来稍显麻烦，特别是你的页面在进入 JavaScript 控制之前先被一些服务端语言渲染。在花了好些时间来设置 E2E 测试之后，我又重新回到了把 Selenium 测试作为高阶的集成测试。（译者注：时至今日，Angular.js 的 E2E 测试已经成为主流，可以参考[Protractor](http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom/)）。

虽然我最近大量使用了 Angular.js，我现在还不是Angular.js 的专家。事实上，我写这篇博客的原因之一也是抱着学习的目的，如果本文有误，我也很欢迎你能纠正文中的错误或者我理解有误的地方。如果你有任何评论，请到 [HackerNews](https://news.ycombinator.com/item?id=5770733) 讨论，也欢迎你到[Twitter](https://twitter.com/insraq)联系原作者。

### 资源
[Things I Wish I Were Told About Angular.js](http://ruoyusun.com/2013/05/25/things-i-wish-i-were-told-about-angular-js.html)

[Shadow DOM](http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom/)

[Protractor](http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom/)





