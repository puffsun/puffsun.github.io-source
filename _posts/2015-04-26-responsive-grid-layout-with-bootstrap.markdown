---
layout: post
title: "Bootstrap 响应式栅格"
author: George Sun
date: 2015-04-26T18:02:12+08:00
comments: true
categories: ["Bootstrap"]
keywords: codethoughts.info codethoughts Front-end UI CSS LESS Bootstrap Twitter Web 页面布局
description: Responsive Guid Layout with Twitter Bootstrap
---

### 写在前面的废话
按照惯例，先废话几句。全栈工程师(Full Stack Developer)的概念现在越来越流行，在Web 开发领域，这一般指的是前端(Front-end)，后端(Back-end)都可以拿得起，放得下的工程师；如果水平更高一些，甚至可以做 Native App。一般来说公司为了降低招聘难度和人力资源成本，会分开招聘前端工程师(Front-end Developer)和后端(Back-end Developer)，究其原因，无外乎 Web 开发这个领域的技术栈非常庞大，对于一个新手程序员，如果一股脑把前端后端抛过去，那他一定会不知从何下手。比如 Web 应用的后端需要知道的知识有：关系数据库、SQL 语言、NoSQL、应用服务器、Web 服务器、框架，后端编程语言，MVC(Model-View-Controller)，ORM(Object-Relational Mapping)、网络协议，操作系统的常见开发、调试工具、IDE、等等，前端需要的知识有：HTML、JavaScript、CSS、CoffeeScript、CSS 预处理器、UI框架，CSS 框架，MVC 以及其变种，Web Components、NodeJS(现在前端工程师的必备工具很多依赖 NodeJS，比如 Yeoman，Grunt，Bower，等等)等等等等。别忘了，现在是前端技术爆棚的时代，现在市面上的很多技术和框架都在迅速演化。另一方面，上面所列的技术基本上只是统称，还需要详细分门别类，整个技术栈在新手看来就是浩瀚的大海，根本无从下手。在这种情况下，根据工业时代的传统来分工就自然而然了。

那为什么现在关于全栈工程师的讨论多了呢？我觉得跟 Facebook 的声称的他们只招聘全栈工程师的标准有关。那是不是说我把上面列出的或者遗漏的技术都学完了那就算是全栈工程师了呢？我认为不是。在我看来原因有两条：

1. 全栈工程师的本质是学习能力和开放思维，你无须把自己局限在前端工程师和后端工程师的思维陷阱里面。如果这个项目有前端需求，那就去把前端需要的某种技术快速搞清楚，后端也是如此，掌握多种技术只是全栈工程师的第一步，快速学习能力才是关键。
2. 把所有技术学完这个想法本身就很荒唐。


### 为什么要用 Bootstrap
如果你像我一样，开发了几年后端(Back-end)代码，然后又因为有成为全栈工程师的渴望，并在实践中学习前端技术，那么 CSS 和页面布局不出意外的话会成为你的绊脚石，起码对于我来说就是如此。我一直觉得 Web 页面的页面布局是个老大难问题，这个我个人感觉没有一年两年功底做不出美观的页面。另一方面，现在 Web 页面，响应式(Responsive), 移动优先(Mobile First)是趋势，这给页面布局带来了新的难题。不过没关系，有了 Bootstrap，只需要为数不多的 HTML 代码，你就可以轻松获得Responsive, Mobile First的页面布局。


### 写这篇文章的目的
主要是为了介绍如何用 Bootstrap 来写一个 Responsive, Mobile First 的栅格页面布局(Grid Layout)。目的是编写一个页面，可以在桌面系统，平板电脑和手机上自适应，根据屏幕分辨率自动调整页面的布局。如果你已经熟悉 Bootstrap，那么可以停止阅读了，而如果你对这个话题和 Bootstrap 都很感兴趣，那么请读下去。

### 什么是 Bootstrap
Bootstrap 是一个帮助开发者快速开发 Web 应用的前端框架，在我看来，它几乎是像是从后端开发转向全栈工程师的救星。有了 Bootstrap，你只需要寥寥几行 HTML 代码，就可以开发出美观的，响应式的，移动设备优先的 Web 布局，Bootstrap 在背后默默的完成那些复杂的，有很多浏览器兼容性大坑的工作。 Bootstrap 由 Twitter 工程师开发，最新的版本是3.3.4，也是我们要用的版本，请注意，Bootstrap 3 是 Mobile First的设计，并且和 Bootstrap 2 并不后向兼容，[这里是它的官方网站](http://getbootstrap.com/)。

Bootstrap 主要的竞争者是：

* [Foundation framework](http://foundation.zurb.com/)
* [Semantic UI](http://semantic-ui.com/)
* [Gumby framework](http://gumbyframework.com/)
* [Pure](http://purecss.io/)

### 什么是栅格(Grid)
栅格系统(Grid System)把一个页面分隔成多个行和列，从而可以根据 Grid 划分情况产生不同的布局，HTML 的内容可以分布在不同的栅格里。Bootstrap 把一个 Web 页面分隔成12列，每一列的宽度根据所在页面的屏幕分辨率和窗口大小而有所差别，也就是说 Bootstrap 的栅格系统是响应式的，并会根据窗口的大小动态调整。

### 设计原型图(Wireframe)
假定我们要开发一个博客，页面原型图我用[Moqups](https://moqups.com/)大致画出来了，为了保证博客的加载速度，我没有截屏，而是链接了原型图的链接。[Moqups](https://moqups.com/)是很赞的在线原型图(Online Wireframe)工具，免费用户可以创建两个公开或私密项目，并可以拥有5M 存储空间。

这是面向桌面浏览器的原型图：
[https://moqups.com/sunwinner3@gmail.com/autujKdy](https://moqups.com/sunwinner3@gmail.com/autujKdy)

这是面向平板电脑浏览器的原型图: 
[https://moqups.com/sunwinner3@gmail.com/autujKdy/p:ad5cd25bf](https://moqups.com/sunwinner3@gmail.com/autujKdy/p:ad5cd25bf)

这是面向手机浏览器的原型图：
[https://moqups.com/sunwinner3@gmail.com/autujKdy/p:add5d9731](https://moqups.com/sunwinner3@gmail.com/autujKdy/p:add5d9731)

### 用 Bootstrap 来实现

#### 桌面浏览器适配
我们知道，桌面一般指的是台式机(Desktop)或者笔记本(Laptop)，它们的分辨率一般比较大，所以我们会选择`col-md`来布局。这里讲解一下，在 Bootstrap Grid System 里，列分4种：

1. col-xs 用在很小的屏幕上，小于768px 一般是手机；
2. col-sm 用在较小的屏幕上，大于768px，一般是平板电脑；3. col-md 用在中等屏幕上，大于992px，一般是桌面系统；4. col-lg 用在特别大的屏幕上，大于1200px，一般是大显示器或者电视机。
我们来写代码实现桌面原型图，首先创建一个文件，叫做`blog.html`:
{%highlight bash%}
$ cd ~/projects && mkdir bootstrap-demo && cd $_
$ vim blog.html
{%endhighlight%}

填入以下 HTML 代码：
{%highlight html lineno%}
<!DOCTYPE html>    <html lang="en">      <head>        <meta charset="utf-8">        <meta http-equiv="X-UA-Compatible" content="IE=edge">        <meta name="viewport" content="width=device-width,initial-scale=1">        <title>My First Bootstrap Website</title><link rel="stylesheet" type="text/css" href="css/bootstrap.css">        <!--[if lt IE 9]>          <script src="https://oss.maxcdn.com/libs/html5shiv/3.7.0/html5shiv.js"></script><script src="https://oss.maxcdn.com/libs/respond.js/1.4.2/respond.min.js"></script> <![endif]-->      </head>      <body>            <!-- Body content goes here --><script src="//code.jquery.com/jquery-1.11.1.min.js"></script>
<!-- Latest compiled and minified JavaScript -->
<script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.4/js/bootstrap.min.js"></script>      </body></html>
{%endhighlight%}

这里我们用 CDN 引入了 [jQuery](http://jquery.com), [Bootstrap](http://getbootstrap.com/), [html5shiv](https://github.com/aFarkas/html5shiv) 和 [respond.js](http://responsejs.com/) 依赖，后两个库是用来来解决低版本 IE 对 HTML5支持不足的问题；对 Bootstrap 的安装，请参考[这个页面](http://getbootstrap.com/getting-started/)。

接下来，我们要完成页面的开发，在上面的标记`Body content goes here`位置填入以下代码：
{%highlight html lineno%}
<div class="container">    <div class="row">        <div class="col-md-12 text-center">            <h1>My First Bootstrap Blog</h1>        </div>    </div>    <hr>    <div class="row">        <div class="col-md-4">            <h3>Post Title 1</h3>            <p>Lorem ipsum dolor sit amet ... </p>        </div>
        <div class="col-md-4">            <h3>Post Title 2</h3>            <p>Lorem ipsum dolor sit amet ... </p>        </div>        <div class="col-md-4">            <h3>Post Title 3</h3>            <p>Lorem ipsum dolor sit amet ... </p>        </div>        <div class="col-md-4">            <h3>Post Title 4</h3>            <p>Lorem ipsum dolor sit amet ... </p>        </div>        <div class="col-md-4">            <h3>Post Title 5</h3>            <p>Lorem ipsum dolor sit amet ... </p>        </div>        <div class="col-md-4">            <h3>Post Title 6</h3>            <p>Lorem ipsum dolor sit amet ... </p>        </div>    </div>
    </div>
{%endhighlight%}
我把代码放在[jsbin.com](http://www.jsbin.com)上面了，可以通过[这个链接](http://jsbin.com/vejixoceje/2)来预览效果。jsbin 是很赞的前端快速分享、协作工具，并且支持 GitHub 集成登录。

这里我们可以看到，Bootstrap 允许一行被分隔成12列，我们设定每列占宽为4，多余的列则被挤到下一行显示。

#### 平板电脑浏览器适配
平板电脑需要显示为2列，这样我们就需要用`col-sm-6`来设计这个页面，修改以上代码：
{%highlight html lineno%}
<div class="container">    <div class="row">        <div class="col-md-12 text-center">            <h1>My First Bootstrap Blog</h1>        </div>    </div><hr>    <div class="row">        <div class="col-md-4 col-sm-6">            <h3>Post Title 1</h3>            <p>Lorem ipsum dolor sit amet ... </p>        </div>        <div class="col-md-4 col-sm-6">            <h3>Post Title 2</h3>            <p>Lorem ipsum dolor sit amet ... </p>        </div>        <div class="col-md-4 col-sm-6">            <h3>Post Title 3</h3>            <p>Lorem ipsum dolor sit amet ... </p>        </div>        <div class="col-md-4 col-sm-6">            <h3>Post Title 4</h3>            <p>Lorem ipsum dolor sit amet ... </p>            </div>        <div class="col-md-4 col-sm-6">            <h3>Post Title 5</h3>            <p>Lorem ipsum dolor sit amet ... </p>        </div>        <div class="col-md-4 col-sm-6">            <h3>Post Title 6</h3>            <p>Lorem ipsum dolor sit amet ... </p>        </div>    </div></div>
{%endhighlight%}
页面效果可以从[这里](http://jsbin.com/qawoji/1)预览，如果你拖动浏览器边框来改变窗口大小，可以看到页面布局会在三列和两列之间变化。

#### 手机浏览器适配
手机和平板电脑一样，可以横放(Landscape Mode)和竖放(Portrait Mode)。对玉分辨率较高的手机，横放会进入平板电脑的布局，我们就不再考虑这种情况。而竖放的手机，我们认为它的宽度小于768px，所以屏幕上只显示一列内容，占满了所有宽度，在这里我们用`col-xs-12`来布局。

修改上面平板电脑适配的代码：
{%highlight html lineno%}
<div class="container">    <div class="row">        <div class="col-md-12 text-center">            <h1>My First Bootstrap Blog</h1>        </div>    </div>    <hr>    <div class="row">        <div class="col-md-4 col-sm-6 col-xs-12">            <h3>Post Title 1</h3>            <p>Lorem ipsum dolor sit amet ... </p>        </div>        <div class="col-md-4 col-sm-6 col-xs-12">            <h3>Post Title 2</h3>            <p>Lorem ipsum dolor sit amet ... </p>        </div>        <div class="col-md-4 col-sm-6 col-xs-12">            <h3>Post Title 3</h3>            <p>Lorem ipsum dolor sit amet ... </p>        </div>        <div class="col-md-4 col-sm-6 col-xs-12">            <h3>Post Title 4</h3>            <p>Lorem ipsum dolor sit amet ... </p>        </div>        <div class="col-md-4 col-sm-6 col-xs-12">            <h3>Post Title 5</h3>            <p>Lorem ipsum dolor sit amet ... </p>        </div>        <div class="col-md-4 col-sm-6 col-xs-12">            <h3>Post Title 6</h3>            <p>Lorem ipsum dolor sit amet ... </p>        </div>    </div></div>
{%endhighlight%}

效果可以通过[这个链接](http://jsbin.com/haciwo/1)来预览，感谢 [jsbin.com](http://jsbin.com)。

### 结论
到这里我们就可以看到用 Bootstrap 来开发一个 Responsive, Mobile First的网站有多么容易，也大大减小了像我这样从后端向全栈转型的工程师的阻力。Bootstrap 除了 Grid System 以外，还提供了很多内容，包括导航(Navigation)，面包屑(Breadcramp)，Form 以及其他组件，并提供了Less/SASS扩展支持，详细情况请参考[这个页面](http://getbootstrap.com/components/)。

### 资源

[Bootstrap 官网](http://getbootstrap.com/)

[Moqups - 原型图设计](http://moqups.com)

[JSBin](http://jsbin.com)

[Site Point, 在线学习](http://www.sitepoint.com/)
