---
title: apache kafka 搭建教程
date: 2018-06-26
category: 
- Management information system
tags:
- apache kafka
- linux
- mac os x
---

这几天忙着折腾 apache kafka，这里写一个笔记，记录整理一下 apache kafka 集群的搭建。

# 目标机器
## Zookeeper
公网 IP: 115.159.26.125
内网 IP: 172.17.16.7
服务器: cent os 7
先解释一下为什么这里要用到 zookeeper。
首先几乎任何一个 kafka 集群都需要用到 Zookeeper，而且一个集群只能有一个 zookeeper。kafka 用到 zookeeper 的目的是：

1. 在 brokers 中选举一个控制器，在 kafka 集群中控制器是为了对所有分区维护一个领导者和跟随者的关系，当某一个结点关机了，是控制者来通知其他备份机去替代该结点上的分区领导者。Zookeeper　选举出来的控制器只能有一个，当这个控制器崩溃的时候，会重新选举出来一个新的。
2. zookeeper 会用来管理集群成员。
3. 配置 Topic：管理每个分区所包含的 Topic 数量，去管理谁来当 leader，谁来当 replica。
4. 配额管理：管理每台客户端允许读写的数据量
5. ACL: 管理谁来可以读写，读写哪个主题。

这里注一下，其实在某些版本可以不使用 zookeeper，不过这个时候创建消费者就不能再使用到 zookeeper 了，需要使用新的 API，bootstrap-server 去创建消费者。

## kafka1
公网 IP: 115.159.104.145
内网 IP: 172.17.16.17 
服务器:  cent os 7

## kafka2
公网 IP: 115.159.202.176
内网 IP: 172.17.16.3 
服务器:  cent os 7

# 软件安装
kafka 是使用 scala 编写的发布与订阅消息系统，需要依赖 JDK 提供的jvm 运行时环境，kafka 官方是建议运行环境安装 java 8，这里 `yum install java-1.8.0` 一条命令就能搞定。

kafka 的安装包可以从官网上下载。这里把 kafka 的安装包放到 `/opt` 目录下，放在 opt，linux/unix 推荐把一些可选且不依赖系统的软件包放到 `/opt` 目录中。

在三台机器上都执行以下命令来安装 kafka。
```bash
cd /opt && wget http://mirrors.hust.edu.cn/apache/kafka/1.0.0/kafka_2.12-1.0.0.tgz && tar xvf kafka_2.12-1.0.0.tgz
```

# 软件部署　
在解包完 kafka 安装包后，需要创建 zookeeper 的数据目录，三台服务器的目录是。同样的，三台机器都执行以下命令：

```bash
cd /opt && mkdir -p zookeeper/zkdata zookeeper/zkdatalog
```

kafka 对内存占用也比较高，所以还需要在配置文件中修改一下 JVM 的参数:
```bash
cd /opt && vim kafka_2.12-1.0.0/bin/kafka-server-start.sh
export KAFKA_HEAP_OPTS="-Xmx2G -Xms2G"
```
如果不把 JVM 内存改大点，在启动 kafka 的时候可能会出现一些奇奇怪怪的问题。

由于前面创建了 zookeeper 的数据目录，这里在 zookeeper 配置中还需要再修改一些东西。
```bash
cd /opt && vim kafka_2.12-1.0.0/config/zookeeper.properties
```
```bash
maxClientCnxns=0
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/opt/zookeeper/zkdata
dataLogDir=/opt/zookeeper/zkdatalog
clientPort=2181
server.1=172.17.16.7:2888:3888
server.2=172.17.16.17:2888:3888
server.3=172.17.16.3:2888:3888
```
在 Zookeeper 服务器(172.17.16.7)执行:
```bash
echo "1" > /opt/zookeeper/zkdata/myid
```
在 kafka1 服务器(172.17.16.17)执行:
```bash
echo "2" > /opt/zookeeper/zkdata/myid
```
在 kafka2 服务器(172.17.16.3)执行:
```bash
echo "3" > /opt/zookeeper/zkdata/myid
```
上述三条命令是用来指定服务器标识符的。

接下来可以开始启动 zookeeper 服务器了。三台机器都要执行以下命令来启动 zookeeper:
```bash
cd /opt && ./kafka_2.12-1.0.0/bin/zookeeper-server-start.sh kafka_2.12-1.0.0/config/zookeeper.properties &
```

和 zookeeper 类似。kafka 这里也要创建 log 存储目录，在三台机器下都执行以下命令:
```bash
cd /opt && mkdir -p kafka/kafkalog
```
然后是修改 kafka 配置，三台机都要改。
```bash
broker.id=$myid  #这里的 broker id 分别是上面指定的 myid，三台机分别是 1,2,3。
host.name=192.168.100.152
num.network.threads=3
num.io.threads=8
log.dirs=/opt/kafka/kafkalogs/
zookeeper.connect=172.17.16.7:2181,172.17.16.17:2181,172.17.16.3:2181
```
然后在三台机器上去启动 kafka 服务：
```bash
cd /opt && ./kafka_2.12-1.0.0/bin/kafka-server-start.sh kafka_2.12-1.0.0/config/server.properties &
```

到了这一步，kafka 集群的部署可以算基本完成了。
接下来可以开始测试了，可以在任意两个结点启动一个 kafka 消费者，用以下命令去启动:

```bash
cd /opt && ./kafka_2.12-1.0.0/bin/kafka-console-consumer.sh --zookeeper 172.17.16.7:2181，172.17.16.17:2181,172.17.16.3:2181 --topic message_publish
```
在另外一个结点启动生产者:
```bash
cd /opt && ./kafka_2.12-1.0.0/bin/kafka-console-producer.sh --broker-list 172.17.16.7:9002，172.17.16.17:9002,172.17.16.3:9002 --topic message_publish
```
如果通过终端在生产者服务器中任意填写消息发送出去，可以在另外两个消费者上看到收到的消息，那么 kafka 集群的搭建基本算是成功了。



# 本地搭建 kafka 单机
对于开发者而言，在编写基于 kafka 的程序的时候，使用集群调试是非常耗资源且没有必要的事，这个时候需要在开发机本机上搭建一个简单的单机的 kafka 服务器。通常我们开发机都是 mac，我这里就简单介绍一下如何快捷方便地在 mac 本机上搭建 kafka。

## 依赖安装
这里要稍稍注意一下，在安装 java 的时候，由于现在 java 的版本已经更新到 java10 了，而 kafka 构建在 java8 上，所以这里不能直接`brew install java`，改用`brew install java8`去安装 java。

安装 kafka 只需要执行 ```brew install kafka```　就能把最新版的 kafka 以及相应的依赖全部安装到本机中。

注：为什么这里我们不把 kafka 安装包放在/opt 目录？ 因为每次在 opt 目录操作都需要临时提权 root，这是有安全风险的，而且苹果官方也不推荐这么做，在更新 kafka 软件包的时候也没有使用`homebrew`包管理器来的方便。

如果已经把 homebrew 安装的程序目录添加到自己的 shell path 中的话，那就可以在任意目录底下执行 zookeeper 和 kakfa 的命令，由于开发机上的 kafka 的单机单结点，如果没有特殊需要甚至可以不做任何配置就拿来用，homebrew 安装的 kafka 的所有命令可以在/usr/local/bin 目录下找到，它们都是链接到/usr/local/Cellar/kafka/1.1.0/bin 目录下可执行程序的软链接。配置文件在/usr/local/etc/kafka 目录下。

启动 kafka 程序的时候只需在终端下执行下面两条命令就可以了:
```bash
zkServer start
kafka-server-start /usr/local/etc/kafka/server.properties&
```
接下来就可以用 `kafka-*` 命令去随便怎么操作 kafka 了。
关闭 kafka 服务器的时候先后执行以下两条命令:
```bash
kafka-server-stop
zkServer stop
```
就可以正常关闭了。
