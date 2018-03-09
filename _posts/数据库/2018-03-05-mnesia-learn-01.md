---
layout: blog
istop: true
background-image: /images/2018-03-05-mnesia-learn-01.png
title: Mnesia学习笔记(1)
date: 2018-03-05-mnesia-learn-01
category: 数据库
tags:
- mnesia
- erlang
- 数据库
---

## Mnesia 介绍
Mnesia 是 Erlang OTP 平台的专用数据库，它是为软实时分布式高可用的计算工作而开发的，与主流关系型数据库不同，Mnesia 能非常灵活地在不同的服务器间迁移, 而且 Mnesia 不是使用SQL语言进行数据库操作，而是直接使用 Erlang 来操作管理，很多编程语言使用者都会鼓吹自己所用语言的 ORM 框架，然而 Erlang　并不需要套一层多余的框架就能直接操作自己的专属数据库。

### Mnesia 特性
- 数据库可以在运行的时候动态地重新配置
- 表可以声明诸如位置，副本，持久性这些属性
- 表可以在多个节点中建立副本，以此来增强系统的容错能力，
- 对于程序员来说，表的位置是透明的。程序指定表名，系统自动追踪表的位置
- 数据库的事务处理可以是分布式的，许多函数可以在一个事务里调用
- 多个事务可以并行运行
- 事务可以一次在多个节点中被执行

### Mnesia 附加应用
Mnesia 通常使用 QLC 来成特定函数，QLC 在 Erlang 标准库中，可以在操作数据库时引入该库。 QLC 增强了 Mnesia 的操作能力。它可以优化查询编译器，它的 “list comprehensions” 能力也非常强大，非常适合用来处理表。

## Mnesia  使用


我在网路上找了一圈，似乎 mnesia 没有什么好的 gui 配置工具，一切操作都需要在 erlang-shell 中完成。

### 启动 Mnesia 并创建数据库
首先设定 mnesia  目录
```
>erl -mnesia dir '"~/Documents/foobar"'
```
然后创建并启动数据库
```erlang-shell
> mnesia:create_schema([node()]).
> mnesia:start().
```
创建表
```erlang-shell
> mnesia:create_table(user,[]).
```
查看表信息
```
> mnesia:info().
```

mnesia 的基本操作大概如上。


### 用 Erlang 定义表并在数据库中

#### 定义表
这里定义一个为博客设计的表
``` erlang

-record(user, {
user_id,
nickname,
email,
name,
sex,
avatar
}).

-record(article, {id,
title,
author_name,
time,
tag,
content
}).
```

重新设置 Mnesia 数据库的位置
```
> erl -mnesia dir '“blogbase”'
```

```erlang-shell
> mnesia:create_schema([node()]).
ok
> mnesia:start().
ok
```

用 erlang 编写建表程式
```erlang
-module(blog).
-include_lib("stdlib/include/qlc.hrl").
-include("blog.hrl").
-export([init/0]).

init() ->
mnesia:create_table(user,[{type, bag},{attributes,record_info(fields,user) }]),
mnesia:create_table(article ,[{attributes ,record_info(fields,article)}]).
```

这里可以看到  user 表那里我给它设置了类型。根据 Erlang 的官方文档，Erlang 的表类型共有 4 个。分别是 `set` , `ordered_set` , `bag` , `duplicate_bag` , Mnesia 不支持 `duplicate_bag` ，其他都支持。顾名思义，这些都是用来表示集合的类型。一个博客用户通常会有多篇文章。所以在 User 表上我给它添加 `bag` 的类型声明。
 注：　1 对 1 用 `set` ， 1 对 多用 `bag` 。

进入 Erlang shell 执行刚刚写好的函数。

```erlang
> blog:init().
{atomic,ok}
> mnesia:info().
---> Processes holding locks <---
---> Processes waiting for locks <---
---> Participant transactions <---
---> Coordinator transactions <---
---> Uncertain transactions <---
---> Active tables <---
article        : with 0        records occupying 306      words of mem
user           : with 0        records occupying 306      words of mem
schema         : with 3        records occupying 669      words of mem
===> System info in version "4.15.3", debug level = none <===
opt_disc. Directory "/Users/gilbertwong/Documents/blog/blogbase" is used.
use fallback at restart = false
running db nodes   = [nonode@nohost]
stopped db nodes   = []
master node tables = []
remote             = []
ram_copies         = [article,user]
disc_copies        = [schema]
disc_only_copies   = []
[{nonode@nohost,disc_copies}] = [schema]
[{nonode@nohost,ram_copies}] = [user,article]
4 transactions committed, 0 aborted, 0 restarted, 4 logged to disc
0 held locks, 0 in queue; 0 local transactions, 0 remote
0 transactions waits for other nodes: []
ok
```

#### 操作表
 使用 erlang 编写函数进行数据插入操作
 insert_user(User, Article)
```erlang
insert_user(User, Article) ->
    Uname = User#user.name,
    Fun = fun() ->
                  mnesia:write(User),
                  mk_articles(Uname,Article)
          end,
    mnesia:transaction(Fun).

mk_articles(Uname, [Article|Tail]) ->
    mnesia:write(#article{author_name = Uname,
                          id = Article#article.id,
                          title = Article#article.title,
                          time = Article#article.time,
                          tag = Article#article.tag,
                          content = Article#article.content}),
    mk_articles(Uname,Tail);
mk_articles(_, []) ->
    ok.
```

 直接硬编码插入数据
 ```erlang
insert_user_example() ->
    User = #user{user_id = 12345,
                 nickname = "gilbert",
                 email = "gilbertwong96@outlook.com",
                 name = "Gilbert Wong",
                 sex = male
                },
    Article = #article{id = 10293,
                       title = "testTitle",
                       time = "2018-3-05",
                       tag = "testTag",
                       content = "testContent"},
    insert_user(User,Article).
 ```
 写入成功
 ```erlang-shell
 > blog:insert_user_example().
{atomic,ok}
 ```
 
查询数据:
```erlang
all_males() ->
    F = fun() ->
                Male = #user{sex = male, name = '$1', _ = '_'},
                mnesia:select(user, [{Male,[],['$1']}])
        end,
    mnesia:transaction(F).
``` 

结果:
```erlang-shell
> blog:all_males().
{atomic,["Gilbert Wong"]}
```

 直接使用 mnesia 提供的函数不够简洁明了，因此 Erlang OTP 还另外 QLC 库来操作数据库, 即通过 List Comprehensions 的语法糖来操作数据库。
 
示例:
```erlang
all_males_qlc()->
    F = fun() ->
                Q = qlc:q([E#user.name || E <-mnesia:table(user),E#user.sex == male]),
                qlc:e(Q)
        end,
    mnesia:transaction(F).
```
结果同上。

// 修改表数据
```
rename_males(New_name)->
    F = fun()->
                Q = qlc:q([E||E <- mnesia:table(user),
                              E#user.user_id == 12345]),
                Ms = qlc:e(Q),
                over_write(Ms,New_name)
        end,
    mnesia:transaction(F).


over_write([E|Tail],New_name) ->
    %% io:fwrite("~s~n",New_name),
    New = E#user{name = New_name},
    mnesia:write(New),
    1+over_write(Tail, New_name);
over_write([],_) ->
    0.
```

 结果为:
 ```erlang
 blog:rename_males("pwd").
{atomic,1}
 ```
