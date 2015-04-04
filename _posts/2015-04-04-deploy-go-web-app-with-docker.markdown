---
layout: post
title: "用 Docker 部署 Go Web 应用"
date: 2015-04-04T07:20:57+08:00
comments: true
categories: ["Go"]

keywords: "Go Golang Web Application Docker Deploy Development web应用 部署 Go语言 软件编程"
description: "Deploy Go Web App with Docker"
---

### 几句废话
按照惯例，先废话几句，再切入正题。学习 Go 语言快一个月了，用 Go 语言实现了[常见的数据结构和算法(持续更新中)](https://github.com/puffsun/go_algorithms_data_structures)，也用 Go 和 AngularJS 创建了一个[迷你 Todo List 项目](https://github.com/puffsun/todos)，回头我会大致介绍一下这个 Todo List 项目的大致结构，现在先聊聊我到目前为止对 Go 语言的观感。因为水平问题，我不可能聊得深入，只是我的个人感受。Go 语言在我看来有如下的优势：

* Go 编译器把代码和依赖直接编译成机器码，所以部署极其简单，一个二进制文件就可以任意部署。
* 语言层面的并发支持，可以充分支持多核。多核已经成为我们这个时代的常态，Go 代码不需要额外的努力就可以做到充分利用多核。Go 语言的并发范式是[CSP(Communicating Sequential Processes)](http://en.wikipedia.org/wiki/Communicating_sequential_processes)，它的好处之一在于并发程序之间的解耦，从而也有利于并发代码的维护和扩展，在软件领域，解耦永远是一个褒义词。拿 Java 和 Ruby 来比较：

    * Java 的内置并发支持来自于 `Thread` 和 `Lock`，我们这里没有足够篇幅讨论 Java 内置库 `java.util.concurrent` 或者 第三方库 [AKKA](http://akka.io/)。Thread 是比较底层的解决方案，类似于给了你锤子和钉子，你可以用它来做足够多的事情，但是经常，你也会不小心把锤子砸到自己手掌上。
    * Ruby 的并发由于[MRI](http://en.wikipedia.org/wiki/Ruby_MRI) 虚拟机中[GIL](http://en.wikipedia.org/wiki/Global_Interpreter_Lock)的存在，变成了伪并发。而如果你把 MRI 上运行的程序直接放到 [JRuby](http://jruby.org/) 或者 [Rubinius](http://rubini.us/) 这些支持真正并发的虚拟机上，多半会有严重的并发问题。
* Go 的工具链非常强大，比如 `gofmt`, `godef`, `goimports`, `golint`, `godoc`, 等等，Go 核心开发者还特意为 Vim, Emacs 和常见 IDE 用户配备了 [Gocode](https://github.com/nsf/gocode)，在 Gocode 里面，日常编程所需的工具，应有尽有。
* 语法简单，代码简洁。这一条因人而异，对我来说，相对于我以前初学 [Scala](http://www.scala-lang.org/) 的时候，被 Scala 的类型推导搞得七荤八素。而在 Go 语言的学习过程中从没遇到难以理解的概念，一切都顺其自然。
* 函数是 Go 语言的一等公民，这意味着函数可以接收函数作为参数，可以返回函数，带来了无穷的想象力。这一点 Java 和 Ruby 都做不到。Java 刚刚在 Java8 里面支持了闭包；而 Ruby 只能够用代码块来模拟这一功能，而且 Ruby 代码块的限制是最多只能有一个代码块作为参数，即使有这样的限制，Ruby 代码块已经成为 Ruby 的标志性构件之一，那么你想想 Go 里面作为一等公民的函数带来的威力吧。

基于以上的理由，Go 语言已经在我的最喜爱编程语言排行榜上位居第三，前两个分别是 `Java` 和 `JavaScript`, 不服莫辩，个人喜好。

### 为什么用 [Docker](https://www.docker.com/) 来部署 Go 项目
Go 目前多应用于 Web 项目中，而 Web 项目一直以来的其中一个让人头疼的地方就是部署，也就是在开发和到产品上线这个跨度里，有很多的不确定因素，比如宿主语言版本、依赖库版本、操作系统版本及补丁、配置文件差异、以及服务器数量等等。而 Docker 从诞生的第一天起就在尝试解决这些问题，而且给出了很好的解决方案。如果你对 Docker 完全不懂，可以参考这篇扫盲文章：[Why Docker Is Going To Dominate Your 2015](http://readwrite.com/2014/12/23/docker-to-dominate-in-2015). 另外顺便提一下，如果你身处 Web 开发领域而你在2014年没有听说或者摆弄一下 Docker，那么你一定要反思一下到底发生了什么！

### 阅读本文的前置条件

* Docker - 你得知道什么是 Docker client/daemon, Docker Image, Docker Container, Docker file, 会运行基本的 Docker 命令。
* Go 语言 - 能读懂 Go 代码，知道如何获取 Go 语言依赖的第三方库，有 Web 开发的基本知识。
* 最好懂点 [AngularJS](https://angularjs.org/)，因为我们要部署的项目前端用 AngularJS 实现。
* 知道如何用 Go 语言开发 Web 项目。不过不知道也没关系，你只需要把现有的项目用`git clone` 拷贝到本地，运行 Docker 命令就可以了。至于这个 Todo List 项目，以后有时间我会写一到两篇博客分别介绍如何用 Go 语言第三方库开发 Web 项目，如何用 AngularJS 开发单页面 Web 应用（Single Page Web Application）。
* 熟悉命令行，另外有一台 Linux 或者 Mac 的机器，这里的所有操作都在命令行完成。

### 我们开始吧
首先从 Github 获取我们要部署的代码，这里我们要部署的代码是我用 Go 语言和 JavaScript MVC 框架 AngularJS 开发的一个 [Todo List](https://github.com/puffsun/todos) 项目。有人会问，我们为什么不部署一个 Hello World呢？我的理由是 Hello World 和实际项目相差太远，它既不需要第三方依赖库管理，也不需要前端和后端分离。我们这里部署的就是一个比较接近于实际产品的 Go Web 应用，如果你跟着做可以遇到更多挑战。
{%highlight bash%}
mkdir ~/workspace && cd $_
git clone https://github.com/puffsun/todos.git
cd todos
{%endhighlight%}

看一下项目目录结构：
{%highlight bash%}
$ ls
Dockerfile     README.md      main.go        run.sh
Godeps/        app/           public/        tmp/

$ tree app
app
├── add_new_todo_test.go
├── app_suite_test.go
├── handlers.go
├── logger.go
├── repo.go
├── router.go
├── routes.go
├── todo.go
└── todos.sqlite3

0 directories, 9 files

$ ls public
css/          favicon.ico   index.html    js/           node_modules/ package.json  readme.md     test/
{%endhighlight%}

这里有几个需要留意的：

1. Dockerfile，这就是我们接下来的主角，关于 Dockerfile 的详细知识，参见[Dockerfile Official Reference](https://docs.docker.com/reference/builder/)
2. app 目录，这里就是我们的后台 Go 代码
3. public 这里就是我们的前端代码，AngularJS 框架实现的 Single Page Web App. 我们这里采用 `npm` 来获取 AngularJS 和这里要用到的 AngularJS 插件 `angular-mock`, `angular-route`，但是为了方便运行这个项目，我把AngularJS 和插件的源码直接放进版本管理系统，省去了安装 NodeJS 的麻烦(其实也不麻烦)。
4. Godeps 目录，这是 Go 的第三方依赖管理工具 `godeps` 创建的，这个工具类似 Ruby bundler，详见[这里](https://github.com/tools/godep)。另外它也省去了我们要在 Dockerfile 里详细指定每个 Go 第三方依赖的麻烦，我们只需要在 Dockerfile 中运行 `godep restore` 就可以自动安装所有依赖，当然，前提是你要安装了 godep: `go get github.com/tools/godep`。
5. run.sh，启动脚本，内容如下：

{%highlight bash linenos%}
#!/usr/bin/env bash

[ -z "$GOPATH" ] && { echo "Need to set GOPATH"; exit 1; }

if [ -x ${GOPATH}/bin/fresh ]; then
    ${GOPATH}/bin/fresh
else
    go run ./main.go
fi
{%endhighlight%}
fresh 是 Go 第三方命令行工具，用于监视指定目录下文件，并根据文件变化自动重启正在开发的 Go 应用，省去每次改文件都要手工重启的麻烦，它可以通过 `go get github.com/pilu/fresh` 来安装。


### Dockerfile
在这个项目中我已经创建好 Dockerfile：

{%highlight bash linenos%}
# Start from a Debian image with the latest version of Go installed
# and a workspace (GOPATH) configured at /go.
FROM golang:latest

MAINTAINER George Sun <http://codethoughts.info>

# For convenience, set an env variable with the path of the code
ENV APP_DIR  $GOPATH/src/github.com/puffsun/todos

# Copy the local package files to the container's workspace.
ADD . $APP_DIR

WORKDIR $GOPATH/src/github.com/puffsun/todos

# Install dependencies
RUN go get github.com/tools/godep
RUN go get github.com/pilu/fresh
RUN $GOPATH/bin/godep restore

# Build the project inside the container.
# (You may fetch or manage dependencies here,
# either manually or with a tool like "godep".)
RUN go install github.com/puffsun/todos

# Run the web application by default when the container starts.
ENTRYPOINT $GOPATH/bin/todos

# Document that the service listens on port 8000.
EXPOSE 8000
{%endhighlight%}
关于如何创建并且优化 Dockerfile，可以参考 [How to Optimise Your Dockerfile](http://blog.tutum.co/2014/10/22/how-to-optimize-your-dockerfile/).

### 创建 Docker Image
运行 `docker build`命令：
{%highlight bash%}
 docker build -t todos .
{%endhighlight%}
接下来你可以看到这么一堆不知所云的输出：
{%highlight bash%}
$ docker build -t todos .
Sending build context to Docker daemon 18.62 MB
Sending build context to Docker daemon
Step 0 : FROM golang:latest
 ---> 121a93c90463
Step 1 : MAINTAINER George Sun <http://codethoughts.info>
 ---> Running in 4e30237b7525
 ---> 0a7f367d6b6e
Removing intermediate container 4e30237b7525
Step 2 : ENV APP_DIR $GOPATH/src/github.com/puffsun/todos
 ---> Running in 42520a221616
 ---> eef290758d42
Removing intermediate container 42520a221616
Step 3 : ADD . $APP_DIR
 ---> bfed922de2bd
Removing intermediate container 03e6230e53d1
Step 4 : WORKDIR $GOPATH/src/github.com/puffsun/todos
 ---> Running in 089533c75557
 ---> f546cb0fab81
Removing intermediate container 089533c75557
Step 5 : RUN go get github.com/tools/godep
 ---> Running in 1b40d8c6dbe1
 ---> a1862a87f057
Removing intermediate container 1b40d8c6dbe1
Step 6 : RUN go get github.com/pilu/fresh
 ---> Running in a9a5737c303e
 ---> 20d05bd603f4
Removing intermediate container a9a5737c303e
Step 7 : RUN $GOPATH/bin/godep restore
 ---> Running in 561760e02e0b
 ---> d501ad34d6fe
Removing intermediate container 561760e02e0b
Step 8 : RUN go install github.com/puffsun/todos
 ---> Running in 0a6b748c582c
 ---> 2319a916598e
Removing intermediate container 0a6b748c582c
Step 9 : ENTRYPOINT $GOPATH/bin/todos
 ---> Running in 06164fcdd94a
 ---> fb7b8e0bce57
Removing intermediate container 06164fcdd94a
Step 10 : EXPOSE 8000
 ---> Running in b18d74e3a627
 ---> ff944097e3f6
Removing intermediate container b18d74e3a627
Successfully built ff944097e3f6
{%endhighlight%}
大概的意思就是 docker 运行了 Dockerfile 里的命令，并且它会不断创建临时的 Image，并基于临时 Image 创建下一步需要的 Image，完成之后 docker 会删除上一步创建的 Image。最终，我们得到了一个名为 todos 的 Image：
{%highlight bash%}
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
todos               latest              ff944097e3f6        3 minutes ago       587 MB
golang              latest              121a93c90463        4 days ago          514.9 MB
{%endhighlight%}
如果期间有什么错误，请自行 Google，或者联系我。

### 运行 Docker container
{%highlight bash%}
$ docker run -p 8000:8000 -ti todos /bin/bash
{%endhighlight%}
`-p 8000：8000` 表示把 Docker container 的8000端口映射到宿主机的8000端口。
`-ti` 表示交互式运行，并且直接把 Docker container 标准输出重定向到宿主机的标准输出。

如果一切顺利，你应该看到如下输出：
{%highlight bash%}
$ docker run -p 8000:8000 -ti todos /bin/bash
{%endhighlight%}

这时候启动浏览器，并访问 `http://localhost:8000`，你就应该看到一个美轮美奂的 Todo List 应用，并且它是可以工作的。如果想停止这个 Docker container，只需要Ctrl + C 即可，因为我们是用交互方式运行Docker container 的，否则要用 `docker stop`。

### Docker Hub
为了方便更多人尝试 Docker 和 Go 语言，我把刚才构建的 Docker Image 放到了 Docker 官方提供的平台 - [Docker Hub](https://registry.hub.docker.com/)。如果你不想经过上面的步骤，可以通过 `docker pull puffsun/todos` 来直接获取已经构建好的 Docker Image, happy hacking。

### 资源

[todos 项目源代码](https://github.com/puffsun/todos)

[TodoMVC](http://todomvc.com/)

[Dockerfile Reference](https://docs.docker.com/reference/builder/)

[Getting Started with Golang on Docker](http://blog.tutum.co/2015/01/27/getting-started-with-golang-on-docker/)

[为什么要使用 Go 语言，Go 语言的优势在哪里？](http://www.zhihu.com/question/21409296)
