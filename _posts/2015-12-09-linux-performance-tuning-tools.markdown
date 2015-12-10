---

layout: post
title: Linux 常见性能调优工具
date: 2015-12-09T19:05:38+08:00
author: George Sun
comments: true
categories: ["Linux"]
keywords: codethoughts.info codethoughts linux performance tuning tools 性能调优 工具
description: "Linux Performance Tuning Tools"

---

我在工作中和业余时间开发的项目中，绝大部分项目的部署环境是 Linux，其中以 Ubuntu 占绝大多数。因此掌握在 Linux 环境下用来性能调优的工具也是一个程序员安身立命的技能之一。在这篇文章里，我主要是流水账式的提到一些常见的性能调优工具，以及它们的基本用法。如果希望系统的学习操作系统和应用程序的性能调优，请参考：[Systems Performance: Enterprise and the Cloud by Brendan Gregg (Prentice Hall, 2013)](http://www.amazon.com/Systems-Performance-Enterprise-Brendan-Gregg/dp/0133390098/)。当然了，这本书也有很高的学习曲线，可以把它作为一本性能调优的参考书来按需查询。

## 关于性能调优
性能调优是一个比较大的话题，也是考察一个程序员水平的重要参考因素。限于篇幅，在这里我只作简略的介绍，希望进一步了解的同学请移步 [酷壳网](http://coolshell.cn/) 参考陈浩大神的两篇文章：[性能调优攻略](http://coolshell.cn/articles/7490.html) 和 [由12306.cn谈谈网站性能技术](http://coolshell.cn/articles/6470.html)，或者去 Google 相关文章。

粗略的讲，系统性能需要考虑两个参数：

* 吞吐量 Throughput
* 延时 Latency

吞吐量是系统在指定时间内可以处理的请求数，延时是系统处理一个请求需要消耗的时间。在进行性能调优的时候，需要对这两个参数进行一些折衷，因为在某个稳定的状态下，如果系统再次提升的吞吐量，与此同时会增加系统的延时系统的延时，因为系统的 CPU，内存和带宽资源是一定的，反之也成立。

## 性能调优的方式
鉴于现代操作系统和应用程序的复杂性，性能调优的方式也是相当的丰富多彩。譬如 Linux 操作系统自身提供了各种各样的工具来分析和监控系统资源的分配和使用情况，这也是我们这篇文章要介绍的主题。另外，很多服务器，数据库，框架，虚拟机和浏览器等等也分别提供了检测和分析性能问题的工具和接口。我们无法在一篇文章里涵盖，甚至无法在一本书里涵盖。

另外，比较流行的方式是使用 Profiler 来观测应用程序的资源分配情况，比如 Java 社区很流行的 [JProfiler](https://www.ej-technologies.com/products/jprofiler/overview.html)，Java Mission Control 和 VisualVM 都是性能调优的利器，通过他们可以观察到应用程序是否有线程死锁，是否消耗了过多的 CPU，是否有内存溢出以及网络使用情况等等。在 Java 社区以外的平台，比如 Node.js 也有类似的工具。

对于前端应用，也有很多选择，比如 Google Speed Tracer，Yahoo YSlow 以及 Chrome Developer Tools，如果有需求，可以参阅各自的文档，将来我可能会写一些类似的文章。

## Linux 下性能分析工具
这里才是本文的正题，在这里我们来看看一些常见的性能分析工具，并简略介绍它们的用法。

### 用 ps 查看进程
Ubuntu 下的 ps 工具支持多种风格的语法：

* Unix 风格 - 一个短横线分隔，比如 ps -ef
* BSD 风格 - 没有短横线分隔，比如 ps aux
* GNU 风格 - 由两个短横线分隔，比如 ps --version

各种风格可以按自己喜好选择，下面来看一些它的常见用法：

查询指定进程的所有子进程 `ps u --ppid 2390`

查询所有进程
{%highlight bash%}
ps aux # BSD 风格
ps -ef # Unix 风格
{%endhighlight%}
3. 查看线程 `ps m`
更多用法请参考 `man ps`

### 用 top 或 htop 监控进程及系统资源使用情况

top 可以用来实时监控最活跃（最消耗 CPU）或者最消耗内存的进程，它也是最为人熟知的 Unix 系统工具之一。下面是它的一些常见命令：

* M (shift + m) 按内存使用量来对进程排序
* T (shift + t) 按总 CPU 使用量来对进程排序
* P (shift + p) 按当前 CPU 使用率来对进程排序
* u 只显示当前用户的进程
* f 选择需要显示的统计数据
* ? 显示使用帮助

如果条件允许，一定要使用 htop 工具来替代 top，它提供了更人性化的界面和更多的功能，比如可以显示各个 CPU 的实时使用率以及查找功能等等，可以自己去体验一下。

### 用 lsof 查找打开的文件和打开该文件的进程
如我们所知，在 Unix 下，一切皆资源，无论是文件，文件夹，管道，外设等等，通过 lsof 它们都无所遁形。默认情况下，lsof 输出大量的信息，大家可以在 Linux 命令行输入 `lsof` 尝试一下。一般情况下我们会把输出重定向到文件中，并用编辑器打开详细检查，或者对输出用管道过滤。

默认情况下，lsof 输出下列信息：

* COMMAND 启动进程，并占用资源的命令
* PID 当前进程 ID
* USER 运行该进程的用户
* FD 文件描述符
* TYPE 文件类型（普通文件，目录，网络连接等等）
* DEVICE 打开文件的设备
* SIZE
* NODE 文件的 inode
* NAME 文件名

下面来看一下 lsof 的常见用法:

查找打开指定目录下文件的进程: `lsof ~`；
查找指定进程打开的资源： `lsof -p <pid>`；
查看帮助: `lsof -h`。

### 用 strace 或 ltrace 跟踪程序运行路径
如果你的应用程序异常退出，那么我们上面介绍的工具根本无从下手，因为他们监测的都是活动的进程，这个时候就轮到 strace 和 ltrace 闪亮登场了。这两个命令都可以打印出程序在运行过程中的运行轨迹，区别在于 ltrace 只会打印应用程序对共享库的使用情况，而 strace 则包括系统调用。我们这里着重介绍一下 strace。

strace 的使用方法很简单，strace 后接需要运行的命令即可，拿 cat 为例：

{%highlight bash%}
$ strace cat /dev/null
###  Outputs as below
execve("/bin/cat", ["cat", "/dev/null"], [/* 74 vars */]) = 0
brk(0)                                  = 0x1eff000
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f5fc9db5000
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
open("/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=85346, ...}) = 0
mmap(NULL, 85346, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f5fc9da0000
close(3)                                = 0
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
open("/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\320\37\2\0\0\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0755, st_size=1840928, ...}) = 0
mmap(NULL, 3949248, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7f5fc97d0000
mprotect(0x7f5fc998b000, 2093056, PROT_NONE) = 0
mmap(0x7f5fc9b8a000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1ba000) = 0x7f5fc9b8a000
mmap(0x7f5fc9b90000, 17088, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7f5fc9b90000
close(3)                                = 0
mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f5fc9d9f000
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f5fc9d9d000
arch_prctl(ARCH_SET_FS, 0x7f5fc9d9d740) = 0
mprotect(0x7f5fc9b8a000, 16384, PROT_READ) = 0
mprotect(0x60a000, 4096, PROT_READ)     = 0
mprotect(0x7f5fc9db7000, 4096, PROT_READ) = 0
munmap(0x7f5fc9da0000, 85346)           = 0
brk(0)                                  = 0x1eff000
brk(0x1f20000)                          = 0x1f20000
open("/usr/lib/locale/locale-archive", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=7224592, ...}) = 0
mmap(NULL, 7224592, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f5fc90ec000
close(3)                                = 0
fstat(1, {st_mode=S_IFCHR|0620, st_rdev=makedev(136, 8), ...}) = 0
open("/dev/null", O_RDONLY)             = 3
fstat(3, {st_mode=S_IFCHR|0666, st_rdev=makedev(1, 3), ...}) = 0
fadvise64(3, 0, 0, POSIX_FADV_SEQUENTIAL) = 0
read(3, "", 65536)                      = 0
close(3)                                = 0
close(1)                                = 0
close(2)                                = 0
exit_group(0)                           = ?
+++ exited with 0 +++
{%endhighlight%}

上面可以看到，strace 给出了非常详尽的输出，从输出中我们可以看到系统调用，共享库函数调用以及返回值等等，对我们调试程序以及检查程序性能大有帮助。

### 用 time 检查程序 CPU 占用时间
这里需要注意的是，Ubuntu 下以及其他常见的 Linux 发行版的 Shell 自带了一个 time 命令，它有别于我们这里要介绍的 time。我们这里想介绍的 time 工具位于 /usr/bin/time。

{%highlight bash%}
$ /usr/bin/time ls -l
0.00user 0.00system 0:00.00elapsed 33%CPU (0avgtext+0avgdata 1376maxresident)k
8inputs+0outputs (1major+429minor)pagefaults 0swaps
{%endhighlight%}

从输出可以看到，CPU 时间分为应用占用时间和内核调用占用时间，另外该命令还输出了应用运行的总时间，在此期间，CPU 可能会被分配去做优先级更高的任务，这也会在总时间总体现出来。随后的信息反应了运行 `ls -l` 命令的内存使用状况。

### 用 uptime 查看系统负载
这个命令很直观：
{%highlight bash%}
$ uptime
 21:33:27 up  2:19,  2 users,  load average: 0.00, 0.05, 0.11
{%endhighlight%}
需要解释的是后面的三个数据，他们分别表示了系统在最近1分钟，5分钟和15分钟内的负载状况。如果负载达到1，那么系统是处在满载的状况，这时候需要我们进一步调查为什么系统会有如此高的负载。另外，负载也有可能高于1的情况，比如双核CPU，最高的系统负载是2。

### 用 vmstat 监测 CPU 和内存使用情况
vmstat 有着悠久的历史，在检测的同时也不会给操作系统带来很多额外的开销，所以是系统性能监测的拿手兵器之一。我们来看一下 vmstat 的输出是什么样，这是理解 vmstat 的关键：

{%highlight bash%}
$ vmstat 2
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 0  0    216 341616 122896 501616    0    0    38    35   52  191  2  1 97  0  0
 0  0    216 341676 122896 501648    0    0     0     0  208 1456  7  1 92  0  0
 0  0    216 341612 122904 501648    0    0     0    34  163 1193  5  1 94  0  0
{%endhighlight%}
上面的命令，vmstat 会每间隔两秒打印一行。procs 表示进程，memory 表示内存，swap 表示页面在内存中的交换情况， io 表示硬盘使用状况，system 表示内黑的系统调用情况，cpu 表示 CPU 时间的占用情况。具体到每一项可以参考 man `vmstat`，其中有详细的解释。

### 用 iostat 监测输入输出
事实上，我们可以用 `vmstat -d` 来查看详细的输入输出信息，不过这个命令的输出过于详细，也不太好观察。所以需要查看输入输出数据的时候，我们首先会考虑 iostat。

{%highlight bash%}
$ iostat
Linux 3.13.0-44-generic (sung2-vm) 	12/09/2015 _x86_64_ (3 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           2.02    0.17    0.55    0.11    0.00   97.16

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
sda              12.26       107.60        98.81     995651     914324
{%endhighlight%}

在最下面，我们可以看到每个硬盘的读写情况。我们还可以指定 iostat 调用的时间间隔 `iostat 2`，这样 iostat 每隔两秒会输出一次，便于我们实时观测。如果需要更详尽的输出，可以指定 `iostat -p ALL`，其他用法请参考 `man iostat`。

### 用 iotop 实时观测硬盘输入输出
类似 top 命令，这个命令实时输出每个进程对硬盘的读写状况，这里不再赘述，可以参考 `man iotop`。

### 用 pidstat 实时监测进程
通过以上命令，如果定位到某个进行资源使用情况有异常，可以用 pidstat 来监测指定进程的资源状况，该命令的输出格式和 vmstat 类似。请参考 `man pidstat` 查阅输出的每一列所表示的含义以及它的用法。

## 结语
这里我们只是覆盖了一些常见的系统性能调优工具，正如我们前面所述，Linux 操作系统自身以及社区提供了大量的工具用来监测系统的性能，如下的一篇文章里就有一些非常有趣并有用的工具，我们没有覆盖到，有兴趣的同学可以参考：[20 Command Line Tools to Monitor Linux Performance](http://www.tecmint.com/command-line-tools-to-monitor-linux-performance/)。如果希望系统的学习系统性能调优，请参考：[Systems Performance: Enterprise and the Cloud by Brendan Gregg (Prentice Hall, 2013)](http://www.amazon.com/Systems-Performance-Enterprise-Brendan-Gregg/dp/0133390098/)。

## 资源
[Systems Performance: Enterprise and the Cloud by Brendan Gregg (Prentice Hall, 2013)](http://www.amazon.com/Systems-Performance-Enterprise-Brendan-Gregg/dp/0133390098/)

[20 Command Line Tools to Monitor Linux Performance](http://www.tecmint.com/command-line-tools-to-monitor-linux-performance/)

[How Linux Worls 2nd edition](http://www.amazon.com/How-Linux-Works-Superuser-Should/dp/1593275676)

 
