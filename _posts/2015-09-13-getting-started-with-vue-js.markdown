---
layout: post
title: Vue.js 初体验
date: 2015-09-13T10:16:40+08:00
author: George Sun
comments: true
categories: ["javascript"]
keywords: 框架 codethoughts.info codethoughts javascript 编程语言 JS Vue.js AngularJS React React.js Angular
description: Getting Started with Vue.js

---

最近有机会在产品代码中写了一些 AngularJS 代码，所以也写了一些关于 AngularJS 的博客，比如这一篇：[AngularJS 单元测试](http://codethoughts.info/javascript/2015/09/06/angularjs-apps-unit-testing/)。当然了，正如你在标题看到的，AngularJS 不是本文的主题。我在学习和使用 AngularJS 的时候看到了一个受它启发的框架 [Vue.js](http://vuejs.org/)。准确的说，Vue.js 受到了很多流行的框架影响，比如 AngularJS、React.js、KnockoutJS等，可以说是博采众家之长，所以它也有一些有趣的设计理念值得我们学习和参考。在这篇文章里面我们先来看一个 Vue.js 入门实例，最后讨论 Vue.js，AngularJS，React.js 主要的不同点。

如果在阅读本文之前你已经熟悉 AngularJS 和 React.js 那是最好不过了，如果不熟悉也没关系，鉴于本人的水平，我也不会做很深入的探讨，主要集中在框架用法方面，对于框架设计也会蜻蜓点水的带过，对大部分熟悉 Web 开发的人来说应该不会有很高的学习曲线。

## Vue.js 简介

Vue.js 的设计是遵循 [MVVM（Model-View-ViewModel）模式](https://en.wikipedia.org/wiki/Model_View_ViewModel)的，或者你也可以把这个模式称作 Model-View-Whatever 或者 Model-View-WhatTheFuck 模式。这个模式是 JavaScript 框架中流行的架构模式，其中包括 AngularJS、React.js 等很多主流的框架。Vue.js 和 AngularJS 有很多相似之处，比如双向数据绑定、Directive、Filter，Template 等等。另外，它又把 Routing、HTTP 请求和处理等功能作为插件来处理，可以参见[Vue.js 插件列表](https://github.com/vuejs)。

学过 AngularJS 的开发者都知道，AngularJS 有大量的概念需要理解，比如 Controller、Directive、数据绑定、Template、Service/Factory/Provider、Module、依赖注入、$scope 等等，这并不是一个完整的列表。Vue.js 可以说是 AngularJS 的子集，并加入了自己的创新（或者说是对其他框架的借鉴）。比如 Vue.js 借鉴了 React.js 的 View Component 概念来进行组件化的开发，并在其上应用了双向数据绑定，也利用 Templating 和 Directive 来规避 React.js 的 JSX 技术。

上面的介绍对于不熟悉 AngularJS 和 React.js 的开发者有点难以理解，我们到此打住，看完一个实例以后我们接着介绍。

## Vue.js 实战

### 引入 Vue.js 库

我们可以通过 CDN 来引入它：

{%highlight html%}
<script src="https://cdnjs.cloudflare.com/ajax/libs/vue/0.12.7/vue.min.js"></script>
{%endhighlight%}

### 创建 Model、View 和 View-Model

我们可以通过 `Vue` 这个类来创建 Vue.js 的 View-Model。在 MVVM 架构中，View-Model 是 View 和 Model 联系的纽带，在 View-Model 中封装了状态和 View 的显示逻辑，View-Model 通过和 View 和 Model 打交道来起到纽带的作用，这和常规的 MVC 架构中 Controller 的地位有所区别。在常规的 MVC 架构中，Controller 是所有请求的入口，负责 Routing，处理所有请求，读取/更新 Model，更新 View 等一系列功能。显然，View-Model 和 Controller 相比非常很轻量级的。

假定我们有了 Model：

{%highlight js linenos%}
var myModel = {
   name: "George Sun",
   age: 30
};
{%endhighlight%}

也有了 View：

{%highlight html linenos%}
<div id="my_view">
{% raw %}{{ name }} is {{ age }} years old.{% endraw %}
</div>
{%endhighlight%}

那现在我们可以通过创建 View-Model 来为 View 和 Model 建立联系：

{%highlight js linenos%}
var myViewModel = new Vue({
   el: '#my_view',
   data: myModel
});
{%endhighlight%}

上面的代码中，`el` 通过 CSS Selector 指向 View，`data` 指向了 Model。另外，我们可以看到，Vue.js 的模板的变量绑定语法和 AngularJS 是一致的，都是 Mustache 风格的。因为 Vue.js 支持双向数据绑定，Model 中的数据更新会立即在 View 层表现出来。我们可以通过一个实例来体验一下。

### 双向数据绑定

上面的示例中，我们在 View 层对数据是只读的，所以只是单向数据绑定。如果希望支持双向数据绑定，我们需要引入 `v-model` 这个 Directive（在 AngularJS 中对等的组件叫做 `ng-model`）。

我们来对 View 层的代码稍作修改：

{%highlight html linenos%}
<div id="my_view">
  <label for="name">Enter name:</label>
  <input type="text" v-model="name" id="name" name="name" /> 
  {% raw %}{{ name }} is {{ age }} years old.</p>{% endraw %}
</div>
{%endhighlight%}

注意上面代码中的 `v-model`，正是通过它我们做到了简单的双向数据绑定，当然背后的机制不是如此简单，Vue.js 是通过 JavaScript 事件机制来做到的。说到这里，我们来看看在 Vue.js 中如何处理事件。

### 处理事件

在 Vue.js 中，如果你希望为 View 层增加事件处理回调，那么你需要把这些回调函数集中放在 View-Model 层的 `methods` 里。为了方便我们在事件回调函数中存取 View 层的数据，Vue.js 在事件对象上增加了一个叫做 `targetVM` 的属性，如下面的示例：

{%highlight js linenos%}
var myViewModel = new Vue({
   el: '#my_view',
   data: myModel,

   // A click handler inside methods
   methods: {
      myClickHandler: function(e) {
         // Use targetVM to access the name
         alert("Hello " + e.targetVM.name);
      }
   }
});
{%endhighlight%}

在 View 代码中，我们可以通过 `v-on` 来为 View 层的元素增加事件回调函数，比如：

{%highlight html linenos%}
<div id="my_view">
   Name: <input type="text" v-model="name"> 
   <button v-on="click: myClickHandler">Say Hello</button>
</div>
{%endhighlight%}

### 其他
其他一些组件，比如 `v-repeat`，`filter` 都和 AngularJS 完全一致，我们就不一一列举了。另外，我们可以在 View-Model 中通过 `ready` 添加应用程序初始化代码，比如下面的实例：

{%highlight js linenos%}
var myViewModel = new Vue({
   el: '#my_view',
   data: myModel,

   // Anything within the ready function will run when the application loads
   ready: function() {
      // When the application loads, we want to call the method that initializes
      // some data
       this.fetchEvents(); 
   },
   methods: {
       fetchEvents: function() {
         // do something...
       }
   }
});
{%endhighlight%}

对于任何 Web 应用，HTTP 请求和处理都是必不可少的组成部分，你可能还在好奇，为什么到现在都没有介绍 Vue.js 是如何处理 HTTP 的。事实上，Vue.js 通过插件机制，把对 HTTP 的处理放在一个叫做 `vue-resource` 的插件里了。当然了，它的用法和 AngularJS 的 HTTP 也没有太大的区别，这里不再赘述，[这里](https://github.com/vuejs/vue-resource) 是 vue-resource 的主页。

## 关于 Vue.js 设计的讨论

### 和 AngularJS 的异同

Vue.js 受到了来自 AngularJS 巨大的影响，关于这一点，我们在开篇有了一些讨论。当然了，这些都是很好的影响，虽然 AngularJS 的学习曲线很高，也有很多概念需要理解，但是 AngularJS 的很多设计决策是非常实用的，比如依赖注入，双向绑定、Directive 等等。

Vue.js 和 AngularJS 最大的不同来自于 Vue.js的双向绑定机制。 Vue.js 在检查 Model 的变化的时候，它通过把 Model 的属性转换成 ES5 的 Getters/Setters，并通过事件机制来通知 Model 的变化。关于 ES5 Getters/Setters，可以参见我之前的一篇文章：[JavaScript Getters 和 Setters](http://codethoughts.info/javascript/2015/06/18/javascript-getters-and-setters/)。而 AngularJS 是通过 `脏数据检测（Dirty Checking）` 机制来做到这一点的。AngularJS 框架中有一个概念叫做 `digest cycle`，可以把它想象成一个循环。AngularJS借助这个循环来检查在所有 `$scope` 中是否有被监测的数据被修改。AngularJS Dirty Checking 机制被诟病的地方通常有两个：一是难以理解，有可能经过几轮或者更多轮的的脏数据检测才可以处理所有 Model 的更新；二是效率不高，因为如果 `$watch` 过多，性能通常会下降的很厉害。

另外，Vue.js 相比较于 AngularJS 最大的特点是 `简单`。它没有依赖注入，没有 `$digest/$apply`，没有 Service/Factory/Provider 等，另外，不像 AngularJS 可以帮助你组织应用程序的结构，如果使用 Vue.js，你要自己为代码组织结构负责，这也是为了简单和灵活性而付出的代价。

### 和 React.js 的异同

Vue.js 和 React.js 的相同点在于都提供了 View Component 的概念，这样可以通过构建独立的 View Component 来组合成更大的应用，带来了更好的代码和功能模块化。然而他们最大的不同点在于实现 View Component 的框架内部机制完全不同。React 有个 Virtual DOM 的概念，这是一个存在于内存中的，对浏览器中真正 DOM 的代理。React.js 对于 DOM 的操作是由对 Virtual DOM 进行 `diff` 操作来比较二者之间不同来达到提高性能的目的。和 Vue 的双向绑定和可修改的 DOM 不一样，React.js 只支持单向数据绑定，DOM 通常是只读的。另外，Vue.js 中，DOM 通常通过触发事件回调来完成，所以他们是完全不同的实现机制。

另外，React.js 的另一块招牌是 JSX，它 看起来很像 XML，是 JavaScript 语法扩展，可以通过 [Babel](https://babeljs.io/) 来翻译成 JavaScript，可以参考我的另一篇文章来了解 React 和 JSX: [React.js 初体验](http://codethoughts.info/javascript/2015/08/23/react-js-hands-on/)。JSX 的初衷是为了简化 React.js 组件的编写，避免冗长的 JavaScript 代码。可 Vue.js 的作者在设计支持完全摒弃了 JSX，原因是他认为 JSX 把 HTML 和 JavaScript 混在一起的做法让他觉得不爽。值得一提的是，Vue.js 的作者既是一个 Web 设计师，又是前端开发者，所以在这个设计决策上，他有着切身体验。

可以参见 [Vue.js Common FAQs](http://vuejs.org/guide/faq.html) 来了解更多关于 Vue.js 的背景和知识。

## 资源

[Vue.js 主页](http://vuejs.org/)

[Vue.js Common FAQs](http://vuejs.org/guide/faq.html)

[Build an App with Vue.js](https://scotch.io/tutorials/build-an-app-with-vue-js-a-lightweight-alternative-to-angularjs#installing-the-dependencies)

[Getting Started With Vue.js](http://www.sitepoint.com/getting-started-with-vue-js/)

[React.js 初体验](http://codethoughts.info/javascript/2015/08/23/react-js-hands-on/)