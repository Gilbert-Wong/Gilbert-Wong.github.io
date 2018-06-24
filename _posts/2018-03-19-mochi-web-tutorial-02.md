---
title:  初探mochi-web(2) - 参数化模块
date: 2018-03-19
category: 
- Erlang
tags:
- erlang
- web develop
- mochiweb
- parameterized module
---


在阅读 mochiweb 的示例源码的时候，我们会见到类似下面这样的写法。
```erlang
loop(Req,_DocRoot)->
    "/" ++ Path = Req:get_path(),
    Body = Req:recv_body(),
    Method = Req:get(method),
    ...,
    ....,
    Response = Req:ok({"text.html;charset=utf-8",[],chunked}),
    Response:write_chunk("Some text here....."),
    ...
```
mochiweb  库里的模块都是以 `mochiweb_`开头的, 然而上面的示例中却出现了 `Req` 这样模块名，究竟是怎么一回事呢。这里就涉及到了 erlang 的一个非官方的一个隐藏了的语法特性: `parameterized module`。

 如果我们把 `Req` 以 io:format 的方式在终端打印出来会发现 
 
 ![](/images/2018-03-19-mochiweb-tutorial-2-01.png)
 
  即 `element(1,Req) == mochiweb_request` , 

 也就是说 `Req` 其实就是 `mochiweb_request`。
 
  这个特性就有点类似于面向对象里的“类实例化对象”这样的过程。当然 Erlang 仍然是一个函数式的编程语言。这个特性其实在 Erlang 中是备受争议的，因此它是 Erlang 的隐藏的一个实验性质的特性。
 
 一个参数化的或者说抽象的模块是一个带有自由变量的模块,  它非常像 lambda 表达式。你可以实例化任意一个抽象模块。就像调用模块函数一样方便。
 
例子:

File: main.erl
```erlang
-module(main).
-export([start/0]).
start() ->
  M1 = print:new("Humpty"),
  M2 = print:new("Dumpty"),
  M1:message("Hello!"),
  M2:message("What's up"),
  ok.
```

File: print.erl
```erlang
-module(print,[Name]).
-export([message/1]).
message(Text) ->
io:fwrite("~s: '~s'", [Name, Text]),
   ok.
```
 
  这里可以看到 `print` 模块是一个带有参数的模块，在 `main` 模块里通过 `new` 的方法创建了两个模块，分别是 M1, M2 。这两个模块能够直接调用 `print` 的函数。
  
回过来看 mochiweb。这个 `Req` 究竟是在哪里实例化的呢？　`mochiweb_request` 模块具有 5 个参数。使用 emacs 的 helm-ag 搜索很快就能找到。`mochiwe_request:new/5` 这个函数已经封装在 `mochiweb`  模块的 `new_request`　函数里了，`new_request`　函数里将参数包在几层用来模式匹配，来增强函数的适应能力。 最后在 mochiweb 的　`mochiweb_http` 模块里可以看到 `Req` 在 `headers`  函数里通过 `mochiweb:new_request/1` 的来实例化。

 参数实例化虽然能够增强语言的表达能力，能够使 Erlang 的模块调用更灵活，但是 Erlang 官方显然不建议这么做，毕竟在 Erlang 的官方文档里，几乎看不到 Erlang 的 `parameterized module`  的痕迹，甚至我的 emacs flycheck for erlang  会对这种语法报错，但是我们在阅读其他大牛的源码的时候，会经常看到他们的这种写法，至少还是要了解一下参数化模块的概念，防止自己都看不懂别人写的代码。
