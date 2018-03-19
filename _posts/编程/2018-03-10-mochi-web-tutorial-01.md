---
layout: blog
istop: true
background-image: /images/2018-03-10-mochiweb-tutorial-1.png
title:  初探mochi-web(1)
date: 2018-03-11
category: 编程
tags:
- erlang
- web develop
- mochiweb
---

## 简介
mochiweb 是 Erlang 的 web 开发框架，同时也可以视作是 Erlang 的一个可编程的 http 服务层。mochiweb　本身负责面向外部的 http 连接。 程序员只要写应用逻辑就行了。Mochiweb 对于 request 和　response 做了非常棒的抽象。和 Java EE 不一样，它其实做得不多, 主要还是对 http 协议的封装。

## 使用

### 查看文档
mochiweb 是完全开源的,它的所有东西都放在 github 仓库里了。 所以直接
``` bash
$ git clone https://github.com/mochi/mochiweb.git
```
进入 mochi web 目录
```bash
$ make edoc
==> mochiweb (prepare-deps)
==> mochiweb (get-deps)
==> mochiweb (compile)
==> mochiweb (doc)
```

这样就生成 mochiweb 的文档了。 打开网页就能直接看到
![](/images/2018-03-10-mochiweb-tutorial-1.png)


### 构建新的 mochiweb 应用

```bash
$ make app PROJECT=helloworld
==> mochiweb (create)
Writing ../helloworld/src/helloworld.app.src
Writing ../helloworld/src/helloworld.erl
Writing ../helloworld/src/helloworld_app.erl
Writing ../helloworld/src/helloworld_deps.erl
Writing ../helloworld/src/helloworld_sup.erl
Writing ../helloworld/src/helloworld_web.erl
Writing ../helloworld/start-dev.sh
Writing ../helloworld/bench.sh
Writing ../helloworld/priv/www/index.html
Writing ../helloworld/.gitignore
Writing ../helloworld/Makefile
Writing ../helloworld/rebar.config
Writing ../helloworld/rebar
```
进入 新建立的 helloworld 目录。不要忘记 make 一下。

make　会从 git 仓库拉取项目的依赖并编译源码。
```bash
$ make
./start-dev.sh
```

此时可以打开浏览器输入　`localhost:8080` 看到

![](/images/2018-03-10-mochiweb-tutorial-2.png)

这样就构建完成了最基本的 mochiweb 应用。

这是 mochiweb 项目的基本目录结构:
src 里存放　erlang 代码用来处理 http 请求,
priv/www/ 里放前端资源文件。 当然开发者也可以自己在 `helloworld_sup.erl`　文件里设定更多具体的参数细节。

![](/images/2018-03-10-mochiweb-tutorial-3.png)

mochiweb 的启动程式码:
```erlang
-module(helloworld).

-author("Mochi Media <dev@mochimedia.com>").
-export([start/0, stop/0]).

ensure_started(App) ->
    case application:start(App) of
        ok ->
            ok;
        {error, {already_started, App}} ->
            ok
    end.


%% @spec start() -> ok
%% @doc Start the helloworld server.
start() ->
    helloworld_deps:ensure(),
    ensure_started(crypto),
    application:start(helloworld).


%% @spec stop() -> ok
%% @doc Stop the helloworld server.
stop() ->
    application:stop(helloworld).
```


 在 `helloworld_web.erl` 的 loop　函数这里处理 http 请求。
```erlang
loop(Req, DocRoot) ->
    "/" ++ Path = Req:get(path),
    try
        case Req:get(method) of
            Method when Method =:= 'GET'; Method =:= 'HEAD' ->
                case Path of
                  "hello_world" ->
                    Req:respond({200, [{"Content-Type", "text/plain"}],
                    "Hello world!\n"});
                    _ ->
                        Req:serve_file(Path, DocRoot)
                end;
            'POST' ->
                case Path of
                    _ ->
                        Req:not_found()
                end;
            _ ->
                Req:respond({501, [], []})
        end
    catch
        Type:What ->
            Report = ["web request failed",
                      {path, Path},
                      {type, Type}, {what, What},
                      {trace, erlang:get_stacktrace()}],
            error_logger:error_report(Report),
            Req:respond({500, [{"Content-Type", "text/plain"}],
                         "request failed, sorry\n"})
    end.

```

## 总结

 mochiweb 的使用非常简单,　同时它也是完全开源的，意味着开发者可以凭自己喜好来定制它，erlang 代码简洁易懂，我认为它不仅是一个好的上手工具，也是非常好的学习资料。
