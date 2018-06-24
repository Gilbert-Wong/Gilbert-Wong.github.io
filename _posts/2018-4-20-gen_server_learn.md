---
title: gen_server 学习
date: 2018-04-20 22:50:19
category:
- Erlang
tags:
- gen_server
- erlang
- Design Principles
---

 前些日子面试的时候， 面试官问了我关于 erlang 上的 gen_server 的问题，我当时没答上来，现在来学习一下什么是 gen_server。
 
 要了解什么是 gen_server，首先要知道什么是 erlang 的 behaviour。
 
## 什么是 behaviour
  首先要知道 erlang otp 上有一个基本概念叫 supervisor tree （监督树）。监督树是将工作进程和监督者进程联系起来的一个数据结构模型。通常在一个监督树下有很多进程，这些进程都有相似的结构，遵循相似的模式，为了让代码更具可读性，更具鲁棒性，更具可维护性， 就要将这些进程的代码抽象成两种范式的两个部分，一种是 behaviour 模块，另一种是 callback (回调)模块.
 
 erlang 大概有四种常用的 behaviour，分别是 `gen_server`, `gen_statem`, `gen_event` 和 `supervisor` 。

 `gen_server` 通常是为用来实现 C/S 结构程序的服务端而设计的。
 `gen_statem` 则是用来实现状态机。
 `gen_server` 是用来实现事件处理功能
 `gen_server` 是用来实现监督树的监督进程的。
 
## C/S 设计原则
 C/S 模型通常是被定义为一个中心服务器加上任意数量的客户端。在多个不同客户端想要共享资源的时候， 通常会用 C/S 模型来进行资源管理操作。
 ![](/images/2018-04-20-gen_server_learn_01.png)
 
## gen_server 使用示例 

~~~ erlang
-module(gen_server_example).
-behaviour(gen_server).

-export([start_link/0]).
-export([alloc/0, free/1]).
-export([init/1, handle_call/3, handle_cast/2]).

start_link() ->
    gen_server:start_link({local, gen_server_example}, gen_server_example, [],[]).

alloc() ->
    gen_server:call(gen_server_example, alloc).

free(Ch) ->
    gen_server:cast(gen_server_example, {free, Ch}).

init(_Args) ->
    {ok, channels()}.

handle_call(alloc, _From, Chs) ->
    {Ch, Chs2} = alloc(Chs),
    {reply, Ch, Chs2}.

handle_cast({free, Ch}, Chs) ->
    Chs2 = free(Ch, Chs),
    {noreply, Ch, Chs2}.

alloc({Allocated, [H|T] = _Free}) ->
    {H, {[H|Allocated], T}}.

channels() ->
    {_Allocated = [], _Free = lists:seq(1, 100)}.

free(Ch, {Alloc, Free} = Channels) ->
    case lists:member(Ch, Alloc) of
        true ->
            {lists:delete(Ch, Alloc), [Ch|Free]};
        false ->
            Channels
    end.
~~~

## gen_server 代码解释
 start_link/0 会调用 gen_server:start_link/4 该函数可以生成并连接到一个新的 gen_server 进程。
 start_link/4 中的第一个参数 {local, gen_server_example} 指定了进程的名字， 意为注册一个名为 gen_server_example 的 gen_server 进程。如果没有指定名字的话， gen_server 进程不会被创建。 gen_server 还可以以 {global, Name} 的方式创建，此时 gen_server 进程会由 global:register_name/2 来注册。
 第二个参数 gen_server_example 是回调模块的名称，回调模块即回调函数所在的模块。
 第三个参数是用来传递给回调函数的参数列表。
 第四个参数是用来选项列表。
 
 如果名字注册成功了，新的 gen_server 进程会调用 gen_server_example 中的 init 函数。
 
 gen_server:start_link/4 是同步函数，直到 gen_server 进程创建完成且准备好接收请求的时候它才会返回。
 如果 gen_server 进程是监督树的一部分，那么就必须使用 gen_server:start_link/4 来创建，如果只是创建一个单独的 gen_server 进程，那么应该使用 gen_server:start 去创建。
 同步请求 alloc 函数将会由 gen_server:call/2 来实现。alloc 也是一个请求，会以消息传递的形式发送给 gen_server。 当请求收到的时候，gen_server 会调用 handle_call(Request, From, State) 函数，该函数会返回一个元组 {reply, Reply, State1}, Reply  就是 gen_server 返回给客户端的内容，State1 是 gen_server 状态的值。
 异步请求 free 函数是由 gen_server:cast/2 实现的。 free 请求也会以消息传递的形式发送给genserver:cast, 然后返回 ok。当请求接收到的时候，gen_server 调用 handle_cast(Request, State)函数。该函数会随之返回一个元组{noreply, State1}。
 如果 gen_server 进程是监督树的一部分, 那就没有必要使用停止函数，gen_server 进程会被监督者进程自动终结。如果需要在进程终结前进行清理的话，gen_server 进程会在接收到关闭指令时调用回调函数 terminate(shutdown, State)。 
 如果 gen_server 进程是一个独立进程，那么就需要专门写一个 stop 函数去终止它。
 
 如果 gen_server 进程只是接收消息而不是接收请求，就必须去实现回调函数 handle_info(Info, State) 去处理。如果 gen_server  进程连接到其他进程（不是监督者进程）并且捕获出口信号，则需要去实现 code_change 函数。
