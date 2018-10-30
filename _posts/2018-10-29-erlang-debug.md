---
title: Erlang 调试方法小结
date: 2018-10-30
category:
- Work
tags:
- erlang
---

使用 Erlang 工作也快半年了，想在这里总结一下我用 Erlang 陆陆续续学会的一些 debug 技巧。

首先 Erlang 不像 C++, java, python 等常用语言，它们有完善的图形界面调试工而 Erlang 则不同，虽然 Erlang 程序员可以在 Erlang shell 中调用 debugger:start() 去开启一个 gui 的 debugger，但是实际用起来还是捉襟见肘，在某些非常知名的 Erlang 开源项目中，比如 EMQX, 它的默认运行时是不会集成 debugger 模块的，并且在服务器上对 Erlang 程序进行调试也是不方便去调用 debugger:start()去开启 gui 调试工具来调试，开发者是很难给自己写出来的 Erlang Code debug 的。这里介绍几种我比较常用的调试方法。

# dbg

 Erlang 和其它常见的动态语言( 如: scheme, python, ruby...)一样，都自带了一个语言的 shell 去直接调用函数， 在 Erlang 自带的库中， dbg 就是一个非常好用的可以用于调试 Erlang 程序的 module，
 基本用法如下：

 ``` erlang
dbg:start().
dbg:tracer().
dbg:p(all,c).
dbg:tpl(emqx_retainer, on_session_subscribed, x).
dbg:ctp(emqx_retainer, on_session_subscribed).
```
这里最关键函数，其中 tpl 指的是对指定模块的指定函数去做 trace，在 Erlang 应用程序执行的时候， 终端会打印指定函数的调用信息，如函数返回值，函数的参数信息。

# print

在 Erlang 中除了 dbg 调试法，我觉得还有一个比较重要，也是比较通用的就是打印调试法，据说大神 Linux Torvalds 在调试 Linux 时多数时候也会使用打印法去 debug linux kernel 。erlang 的打印函数是 io:format(). 很多时候只要写成这样 `io:format("~p", [Variable])` 就可以打印任意你想打印的 Erlang 数据结构。


# 其他

与上述两种调试法不同，有些调试方法并不是用来 trace 单个 Erlang 进程的，它更多的是用来查看整个 Erlang 应用程序内存占用，比如:

```erlang
erlang:system_info(process_count).
```

这条命令是用来查看 Erlang 进程数量是否正常，是否超过 Erlang 虚拟机设置好的最大进程数。

然后是通过
```erlang
erlang:memory().
```
去查看节点的内存瓶颈。看看是否是 Erlang 虚拟机的哪一部分内存占用比较大。

如果是进程的内存占用比较大，那么就使用以下命令去查看占用内存最高的进程状态：

```erlang
spawn(fun()-> etop:start([{output, text}, {interval, 1}, {lines, 20}, {sort, memory}]) end).
```

最后是查看占用内存最高的进程状态：
```erlang
erlang:process_info(pid(0,0,0)).
```

最后就是到占用内存最高的进程所在的代码去 review ，看看是否还有优化的空间。

