---
layout: post
title: "Play! Framework 大致观感"
date: 2015-04-25T10:02:59+08:00
comments: true
categories: ["Java"]
keywords: codethoughts.info codethoughts Play! play framework overview 框架 Java Web HTTP
description: Overview of play framework, Java web development framework for Agile
---

### 为什么写这篇文章
前段时间花时间比较了几个 Web 框架，考量的对象来自于 Golang 社区和 Java 社区：[Beego](http://beego.me/), [Martini](http://martini.codegangsta.io/), [Play! 2](https://www.playframework.com/), [Spring MVC](http://projects.spring.io/spring-framework/)。在选择的过程中并没有多少客观的统计数据，全凭我的个人理解和不同社区的开发者在选型时候的分析，在这篇文章里面我想聊聊为什么会最终选择 [Play! 2 Framework](https://www.playframework.com/)，以及对 Play! 2 Framework 的大致观感，最后就是用最少的步骤搭建一个基于 Play! 的项目。

### Web 框架选型的依据
通常来说，你需要考量以下方面：

* 数据库/NoSQL 支持
* 缓存支持
* 日志管理
* 开发工具和社区活跃程度
* Session 管理
* 宿主语言
* 性能
* 扩展性

### 为什么没有 Ruby on Rails
Rails 框架完全符合上面的依据，同时也是非常成熟和流行的框架，但是我现在对于动态语言有些不太感冒，更倾向于静态，编译型语言。Ruby 代码的风格过于奔放，读优秀程序员写出的 Ruby 代码固然优美，维护新手程序员写出的 Ruby 代码又恨不得让你把眼珠子都挖出来；另一方面 Ruby 这样的动态语言代码极度依赖于测试，包括但也不限于单元测试和集成测试，没有测试覆盖的 Ruby 代码就是狗屎。而在国内的IT公司，不写或者尽量少写测试代码似乎成了默契，起码我待过的数家公司都是如此，这些地方技术选型上都不应该采用动态语言。

### 大致观感
Play! 框架可以说是我用过的最顺手，开发效率最高的Web 开发框架，提供了开发、测试、发布一揽子的工具支持，甚至有插件支持直接打包为各种操作系统的安装包。

### Play! framework 的特点
简单讲，Play! 框架是个类Ruby on Rails 风格的全栈(full-stack)开发框架，在设计上内置 RESTful 支持，从 `Play! 2` 开始, Play! 全面拥抱了 Scala，采用了 Scala 作为模板语言，事实上，从2.0版本开始，Play! 框架可以说原生支持 Scala。

####抛弃 JavaEE
传统的 Java Web 开发中，程序员一定避不开 Servlet，以往的 Java Web 框架都是基于 Servlet 来构建的，最终开发完成的应用也一定要在Servlet 容器里运行。Java 程序员应该都知道，Servlet API 提供了一套底层的 API 来处理 HTTP 请求和响应，Play! framework 在这里的做法非常大胆，全面抛弃 Servlet另起炉灶，这也带来了新的优势。Play! 采用 Apache Mina 作为底层 HTTP Server，并做了自己的封装，从而实现代码的热替换(hot swapping)，这里的优势是修改了代码，只要刷新浏览器新修改的代码就可以立即生效，这对开发生产力的提升大有好处，具体的代码分析可以参见[这篇文章](http://androider.iteye.com/blog/710533)。

#### 抛弃 XML 配置文件
写过 Java Web 项目的程序员都知道 XML 噩梦是怎么回事，现在又向 元注释(Annotation)噩梦前进了。Play! 给出的回应是它只有一个全局配置文件，绝大部分的配置是自动设置，或者是默认设置，这一点深得 Ruby on Rails CoC(Code on Convension) 的精髓。

####无状态(Stateless)框架
这意味着 Play! 不会把状态保存在 Server 端。Play! 认为数据库可以存储状态，Play! 框架在每次 HTTP Request 之间不会在 Server 端存储状态，所需的状态都需要在 HTTP Request 之间传递。当然在 Play! 里也有 session 的概念，也有一个方法 `session()`可以存取 session，但是这里只不过是浏览器端的 cookie，你调用 `session()`方法也只能存取 String 类型的值。

另外，无状态也对应用程序的扩展性很有利，你只需要新增一个节点，重新部署一个 Play! 应用就可以立即增加系统的负载能力。

####强类型模板(template)
Play! 2 对Play! 框架进行了完全的重写，完全拥抱了 Scala，并用 Scala 替代 Groovy 作为模板语言。事实上，Play! 2的模板就是可编译的 Scala 代码，如果编译通不过，你可以立即从浏览器或者控制台(console)看到错误信息，而不必等到重新部署，调用了相应页面才可以发现错误，这也可以大大提高开发生产力。

####工具支持
Play! 提供了类似 Rails 的命令行工具来生成样板代码(boilerplate code)，而在最新的版本`2.3`又和`activator`平台集成，提供了浏览器中开发的工具支持。

####社区以及市场占有率
Play! 的市场占有率大约在8%, 虽然不算高，但是考虑到它的推出时间，我觉得已经很不错了，在社区方面做得也还算不错，在 StackOverflow 上提出的问题很多有高质量的解答。


### Play! framework 缺点
不是所有的框架都是十全十美的，Play! 也不例外。我认为 Play! 最主要的问题在于 Scala 语言的复杂性和它的后向兼容性。

####模板引擎

Scala 语言本身的学习曲线非常高，我在刚接触 Scala 语言的时候被它的类型推导折腾得七荤八素，我觉得其他人也不例外，相比较学习 Go 语言这样核心精简的语言简直就是手到擒来，拿 Scala 作为模板引擎大大提高了模板的复杂性。

####后向兼容性

Play! 2 是对 Play! 1 的完全重写，Play! 2 是和 Scala 无缝集成的，如果想完全发挥 Play! 2 的威力，最好采用 Scala 语言。同时它也不对 Play! 1 后向兼容，如果希望从 Play! 1 升级到 2，那么只有完全重写，这样会导致它失去一批老用户。

### 示例

#### 安装 Play!

首先到 Play! 框架[下载页面](https://www.playframework.com/download)下载安装包并解压到本地目录，最好下载 [Offline Distribution](http://downloads.typesafe.com/typesafe-activator/1.3.2/typesafe-activator-1.3.2.zip) 包，否则所有依赖仍然要远程下载，这在我国网络环境下是个巨大的挑战。当然你也可以修改为国内的源来增加下载速度，遗憾的是我目前还没有发现一个稳定并且完整的国内源。可以参考[这个页面](https://www.playframework.com/documentation/2.3.x/Installing)来安装 Play!。

#### 新建项目
首先新建一个目录，并查看一下 Play! 提供的命令行工具：

{%highlight bash%}
$ cd ~/prog/play && mkdir demo && cd $_
$ activator -h

Usage: activator <command> [options]

  Command:
  ui                 Start the Activator UI
  new [name] [template-id]  Create a new project with [name] using template [template-id]
  list-templates     Print all available template names
  -h | -help         Print this message

  Options:
  -v | -verbose      Make this runner chattier
  -d | -debug        Set sbt log level to debug
  -mem <integer>     Set memory options (default: , which is -Xms1024m -Xmx1024m -XX:PermSize=64m -XX:MaxPermSize=256m)
  -jvm-debug <port>  Turn on JVM debugging, open at the given port.

  # java version (default: java from PATH, currently java version "1.7.0_45")
  -java-home <path>  Alternate JAVA_HOME

  # jvm options and output control
  -Dkey=val          Pass -Dkey=val directly to the java runtime
  -J-X               Pass option -X directly to the java runtime
                     (-J is stripped)

  # environment variables (read from context)
  JAVA_OPTS          Environment variable, if unset uses ""
  SBT_OPTS           Environment variable, if unset uses ""
  ACTIVATOR_OPTS     Environment variable, if unset uses ""

In the case of duplicated or conflicting options, the order above
shows precedence: environment variables lowest, command line options highest.  
{%endhighlight%}

这里可以看到，Play! 提供了很多选项，其中 `activator ui` 提供了在浏览器中新建、编译、运行、编写 Play! 代码的功能；`activator --jvm-debug <port>` 则提供了调试 Play! 项目代码的快捷方式； `activator new` 可以新建项目，也就是我们下一步要做的。

{%highlight bash%}

$ activator new sample

Fetching the latest list of templates...

Browse the list of templates: http://typesafe.com/activator/templates
Choose from these featured templates or enter a template name:
  1) minimal-akka-java-seed
  2) minimal-akka-scala-seed
  3) minimal-java
  4) minimal-scala
  5) play-java
  6) play-scala
(hit tab to see a list of all templates)

{%endhighlight%}
Play! 提供了很多模板，我们可以基于这些模板快速创建项目。目前从命令行可以存取的所有模板众多，这里就不一一列举了，可以用`Tab`把它们都调出来。这里假定我们希望基于`play-java`模板来创建我们的新项目：
{%highlight bash%}
$ > play-java
OK, application "sample" is being created using the "play-java" template.

To run "sample" from the command line, "cd sample" then:
/Users/gsun/prog/play/demo/sample/activator run

To run the test for "sample" from the command line, "cd sample" then:
/Users/gsun/prog/play/demo/sample/activator test

To run the Activator UI for "sample" from the command line, "cd sample" then:
/Users/gsun/prog/play/demo/sample/activator ui
{%endhighlight%}

按照 Play!的指示，我们进入项目目录，并且再次运行 `activator run`命令：
{%highlight bash%}

$ activator run
[info] Loading global plugins from /Users/gsun/.sbt/0.13/plugins
[info] downloading file:/Users/gsun/dev/activator-1.3.2/repository/com.typesafe.play/play-java-ws_2.11/2.3.8/jars/play-java-ws_2.11.jar ...
[info]  [SUCCESSFUL ] com.typesafe.play#play-java-ws_2.11;2.3.8!play-java-ws_2.11.jar (7ms)
[info] Done updating.

--- (Running the application, auto-reloading is enabled) ---

[info] play - Listening for HTTP on /0:0:0:0:0:0:0:0:9000

(Server started, use Ctrl+D to stop and go back to the console...)

[info] Compiling 4 Scala sources and 2 Java sources to /Users/gsun/prog/play/demo/sample/target/scala-2.11/classes...
[info] 'compiler-interface' not yet compiled for Scala 2.11.1. Compiling...
[info]   Compilation completed in 20.839 s
[info] play - Application started (Dev)
{%endhighlight%}
这时候打开浏览器，并访问`http://localhost:9000`，你会发现一个完整的应用已经运行，你只需要根据需要增加或者修改功能就可以了，可以说搭建一个 Play! 框架的项目需要的时间非常短，效率非常高。

另外 `activator ~run` 可以实现代码的热替换(hot swapping)，修改代码之后，程序员只需要刷新浏览器就可以立即看到修改结果。这里是在项目目录内部运行`activator help` 的结果：
{%highlight bash%}
[sample] $ help

  help                                    Displays this help message or prints detailed help on requested commands (run 'help <command>').
  completions                             Displays a list of completions for the given argument string (run 'completions <string>').
  about                                   Displays basic information about sbt and the build.
  tasks                                   Lists the tasks defined for the current project.
  settings                                Lists the settings defined for the current project.
  reload                                  reload

        (Re)loads the project in the current directory.

reload plugins

        (Re)loads the plugins project (under project directory).

reload return

        (Re)loads the root project (and leaves the plugins project).
  projects                                Lists the names of available projects or temporarily adds/removes extra builds to the session.
  project                                 Displays the current project or changes to the provided `project`.
  set [every] <setting>                   Evaluates a Setting and applies it to the current project.
  session                                 Manipulates session settings.  For details, run 'help session'.
  inspect [uses|tree|definitions] <key>   Prints the value for 'key', the defining scope, delegates, related definitions, and dependencies.
  <log-level>                             Sets the logging level to 'log-level'.  Valid levels: debug, info, warn, error
  plugins                                 Lists currently available plugins.
  ; <command> (; <command>)*              Runs the provided semicolon-separated commands.
  ~ <command>                             Executes the specified command whenever source files change.
  last                                    Displays output from a previous command or the output from a specific task.
  last-grep                               Shows lines from the last output for 'key' that match 'pattern'.
  export <tasks>+                         Executes tasks and displays the equivalent command lines.
  exit                                    Terminates the build.
  --<command>                             Schedules a command to run before other commands on startup.
  show <key>                              Displays the result of evaluating the setting or task associated with 'key'.
  all <task>+                             Executes all of the specified tasks concurrently.

More command help available using 'help <command>' for:
  !, +, ++, <, alias, append, apply, eval, iflast, onFailure, reboot, shell

[sample] $
{%endhighlight%}
这里我就不再一一讲解了，如果你有兴趣，那么可以自己去探索。

### 结语
如果你喜欢 Ruby on Rails 开发的高效，又喜欢 Java/Scala 静态语言的编译期代码检查以及稳定性，那么 Play! 框架你应该认真考虑一下，它是我至今用过的开发最高效的 Java Web 框架。

### 资源
[Play! framework 官网](https://www.playframework.com/)

[Play! framework hotswap及源码分析](http://androider.iteye.com/blog/710533)

[Play! framework 官方文档](https://www.playframework.com/documentation/2.3.x/Home)

[Scala 编程语言官网](http://www.scala-lang.org/)