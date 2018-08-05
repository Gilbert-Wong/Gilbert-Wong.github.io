---
title: mqtt 学习笔记(2)
date: 2018-08-05
category: 
- Program Language
tags:
- erlang
- scala
- akka
---

Erlang 是一种基于虚拟机的解释型函数式编程语言，它从一开始就是为并行，高容错的通
信系统设计的，Erlang 的进程非常轻量，且都有独立的堆栈，Erlang 虚拟机会分别为每个
进程单独做垃圾回收处理，通常每个进程的堆很小，  垃圾回收的压力可以分摊到每个进程
，避免了其它语言（如 Java、Scala、Go） 那样因为垃圾回收而导致整个程序卡顿，Erlang
虚拟机的垃圾回收过程非常平稳，对于延迟敏感型的应用而言，是非常重要的特性。而且，
Erlang 的整个不可变类型系统以及基于消息的进程并发方案使数据关系复杂性大大降低，这
些优势为Erlang实现“软实时”奠定了非常良好的基础， Erlang 的 Actor模型（即通过邮箱
来收发消息）使得Erlang 开发者可以非常轻松地使用 Erlang 编写出可以处理大量网络连接
的服务器程序。 Erlang 之父 Joe Armstrong 在谈到 Erlang 的设计要求时就曾提到软件
维护需要在不停止系统的情况下进行，Erlang 完美地做到了这点，Erlang 虚拟机会为每个
模块保存 2 份代码，分别是“当前版本”代码以及“旧版本”代码，当模块第一次被加载时，
代码就是“当前版本“，当有新的代码被加载时，“当前版本”代码就会变为“旧版本”代码，而
新的代码会成为“当前版本”代码，Erlang 通过两个版本代码共存的方式来保证任何时候都
有一个版本的代码的可用的，以此来保证对外服务不会停止。通过使用 Erlang 的代码“热
交换”的能力可以大大提高 Erlang 程序的可用性，因此 Erlang 是一个非常适合用于编写
处理大规模 TCP 并发连接以及大规模消息系统的编程语言。

Akka 是一个为 Scala 提供并行计算能力的库，大大降低了 Scala 并发编程的难度，也是
Scala 社区编写高并发程序的首选方案，Scala Akka 库可以说是充分借鉴了 Erlang 的 
Actor 模型的思想，每个 Actor 是独立的一个容器，包含了状态，行为，以及属于它的一
个邮箱，child Actor 和监管策略。每个 Actor 互相通过消息传递的方式来通信，这里
可以看出 Akka 的 actor 和 Erlang 的 process 非常像，但实际上还是区别很大的， 由
于 Scala 是基于 JVM 上的语言，所以它不可能创建出像 Erlang process 那样轻量级的
进程，折中的方法就是多个 actor 共享一个线程，一个线程一组 actors ，这样才勉强
在 JVM 上实现像 Erlang 那样的 actor 模型。不过 Akka 最致命的缺点在于实现它的 Scala
是一门基于 JVM 的语言。众所周知 JVM 最初是为 Java 语言而设计的，Java 在设计之初
根本就没有充分考虑到并行，高容错，低延时，分布式这些现在看来非常重要的特性，所以
JVM 的 GC 处理是全局的，而不是像 Erlang 那样单个进程单个 GC，所以在数据并发量非
常大的情况下，一旦 GC 卡顿，很有可能所有 actor （线程）停止工作，Erlang VM 底层
提供了非常完善且久经历史考验的抢占式进程调度算法， 避免了死锁和饥饿的出现，Akka 
的调度器需要做非常复杂的配置，即使如此也比不上 Erlang 的进程调度，Erlang 支持代
码热重载， Scala 也支持，但是会因为 JVM 的类加载而丢失灵活性。Akka on Scala 或许
因为 JVM 的 JIT 特性显得速度更快一些，但是 Erlang 也有 BEAMJIT ---- 一个基于 
LLVM 的 Just-in-time 编译器，而且如同 java 通过 JNI 来调用 C语言编写的程序一样，
Erlang 也有 NIF 技术来提高性能。


下面这张表列出了 Erlang 有而 Akka on Scala 没有以及 Akka on Scala 有而 Erlang 没
有的特性：


|                      | **Erlang** | **Akka on Scala** |
|----------------------|------------|-------------------|
| message copy on send | ✓	        | ✕                 |
| per-process GC       | ✓          | ✕                 |
| java ecosystem       | ✕          | ✓                 |
| process scheduling   | ✓          | ✓                 |
| code-reload          | ✓          | ✓                 |
