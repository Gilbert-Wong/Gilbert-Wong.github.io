---
layout: blog
istop: true
background-image: /images/2018-06-13-emqttd-work-summary(1)-01.png
title: emqttd 工作总结（1）
date: 2018-06-13
category: 编程
tags:
- erlang
- mqtt
---

# 问题描述
之前在 github 上被 assigned 了一个 issue，是和 emqttd 相关的一个 issue。

这是 issue 地址: <>

开本 issue 的用户在使用 emqttd 的时候，发现了一个消息发送行为与其它 mqtt broker（比如mosquitto, hivemq）不同，使用场景是这样：connect 控制包中的 clean_session 设为 false，publish 包的 qos 设为 1，retain flag 设为 true，此时有两个 Client 连上了这个 broker， Client1 在订阅了 qos 为 1 的 test/777 的 topic 后断开，Client2 此时发布三条消息，消息 payload 分别是 1,2,3，retain flag 设为 treu, qos 等级设为 1，此时 clien1 连接上 broker 后，会收到 client2 发布的三条信息，再订阅上 test/777 的 topic，此时会收到 retain = true , payload 为“3” 的 message， mosquitto 和 hivemq 都是正常的，而我们的 emqtt 会丢掉最后 retain 为 true 的消息。所以要修复这个问题。

# 解决思路
retain 消息发送是和 emq_retainer 的插件有关，进入 emq_retainer.erl 模块。

~~~ erlang
load(Env) ->
emqttd:hook('session.subscribed', fun ?MODULE:on_session_subscribed/4, [Env]),
emqttd:hook('message.publish', fun ?MODULE:on_message_publish/2, [Env]).
~~~

会发现 emqttd 是通过 hook 的方式去调用 emq_retainer, 在 session.subscribed 的时候，会执行 on_session_subscribed 函数，然后去读取离线状态下其他客户端发的 retaine为 true 的最后一条消息。所以问题应该就出在 on_session_subscribed 或 on_message_publish 这两个函数中。

# 解决方案
在启动 emqttd 之后，使用 dbg 去跟踪 on_session_subscribed 函数。dbg 是 Erlang 语言非常方便的一个跟踪调试工具，可以在终端中使用 `erl -man dbg` 命令去文档中查看它的具体用法。调试命令如下：
``` erlang
dbg:start().
dbg:tracer().
dbg:p(all,c).
dbg:tpl(emqt_retainer, on_session_subscribed, x).
dbg:ctp(emqt_retainer, on_session_subscribed).
```
上面调试命令中有一个 x, 这里简单介绍一下 dbg。dbg 内置有三种 trace patterns ，分别是 x, c, cx 。x 代表 exception_trace,  c 代表 caller_trace，cx 代表 caller_exception_trace。exception_trace 用来打印函数名，函数参数，返回值，以及从函数中拋出去的异常，call_trace 则是用来打印函数名，参数以及函数调用信息的,caller_exception_trace 则是将前两者结合起来打印。

不断地对模块函数进行高度跟踪，最后会发现在重新订阅 test/777 主题的时候，没有去读取 sor_retained 的 MSG，所以问题就好解决了，去 emqttd_session 模块的 handle_cast那里去查看一下它是怎么处理 Duplicated Message。

~~~ erlang
case maps:find(Topic, SubMap) of
    {ok, NewQos} ->
        ?LOG(warning, "Duplicated subscribe: ~s, qos = ~w", [Topic, NewQos], State),
        SubMap;
~~~

这里它有一个逻辑没写对，在将新的 Qos 去旧的 Qos 匹配的时候，忘记调用 emqttd_hooks:run 函数了，导致 Client1 在第二次订阅后，没有正常收到 retained message。

因此只要添加上一句 hook 调用就行了。
~~~ erlang
case maps:find(Topic, SubMap) of
    {ok, NewQos} ->
        emqttd_hooks:run('session.subscribed', [ClientId, Username], {Topic, Opts}),
        ?LOG(warning, "Duplicated subscribe: ~s, qos = ~w", [Topic, NewQos], State),
        SubMap;
~~~

问题解决!

# 致谢
多谢公司前辈刘新宇在解决本问题时给我的指导和帮助，让我学到了很多东西。　
