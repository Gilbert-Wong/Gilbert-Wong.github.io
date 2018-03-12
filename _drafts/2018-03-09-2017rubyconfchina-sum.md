---
layout: blog
istop: true
background-image: /images/2018-03-05-mnesia-learn-01.png
title: 2017 RubyConf China Bhuztez Erlang
date: 2018-03-09-2017rubyconfchina-sum
category: 数据库
tags:
- erlang
- web develop
- rubyconf
- bhuztez
---

 今天看了去年 bhuztez 在ruby china conference 上提到的如何用 Erlang 快速开发　web framework。　作为一个 Erlang beginner ,在这里简单对他的温做个归纳总结。

## 为什么选择 Erlang  做 web 开发
### Erlang的性能

 Erlang 作为一个动态编程语言，顺序执行效率是一定比不上 java, C++ 这样的编译器运行时高度优化过的 静态类型的语言。但是由于 Erlang 语言和 vm 的特殊性，Erlang 的并发性能非常高， 同时由于 Erlang  的函数式语言的特点，因此它非常适合 web 这样的应用场景。

### Erlang 的开发效率
 Erlang 是一个非常紧凑的语言，  关键特性主要有模式匹配，消息机制，热更新。
 模式匹配没什么好说的，都是 Erlang 的基础。
 
 这里我不了解的主要就是 b大提到的 parse_transform() 。我下载的 Erlang 的官方文档里似乎也没提到。

```erlang
-module(example).

-compile({parse_transform,razor_id_trans}).
```

上面就是 parse_transform　的简单例子。
compile 参数元组里第一个原子是 parse_transform, 第二个是 parse_transform 的模块名。

```erlang
-module(razor_id_trans).
-export([parse_transform/2]).

parse_transform(Forms, _Options) ->

Form.
```
 
 
 
 

 
 
