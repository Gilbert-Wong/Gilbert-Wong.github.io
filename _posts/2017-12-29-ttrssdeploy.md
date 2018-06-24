---
title: tinyrss 在 ubuntu 16.04 上的部署
date: 2017-12-30
category: 
- Management information system
tags:
- ubuntu
- tinyrss
- reeder
- 运维
---

## 目的
手头有一个闲置的vps，希望它能够物尽其用，因此我打算在上面搭建一个tiny tiny rss 阅读服务。(feedly 免费版有太多限制，付费版成本远远高于一般的廉价服务器租用费用)。

## 安装tt-rss
服务器系统是ubuntu 16.04，安装过程已经高度自动化了。只要ssh远程登录主机后执行两条去安装并根据安装提示一步一步地去填写mysql的root账户的密码以及tt-rss账户的密码即可。


``` bash
$ sudo apt install -y mysql-server php-mbstring
$ sudo apt install -y tt-rss
```

包管理器自动安装的网站路径在```/usr/share/tt-rss/```下。schema目录存放的是sql脚本，www目录存放的是tt-rss站点的网站资源。

## 扩展tt-rss

很多时候我们并不是在pc上开着浏览器阅读rss讯息的，而是使用Reeder这类的移动客户端来同步自己订阅的rss文章，这个时候就需要用到[fever](https://github.com/dasmurphy/tinytinyrss-fever-plugin)插件。

此时切换到```/usr/share/tt-rss/www/plugins```目录，执行命令:
``` bash
$ git clone https://github.com/dasmurphy/tinytinyrss-fever-plugin
```
之后在插件栏中启用fever插件。

![](/images/2017-12-29-ttrssdeploy-1.png)

然后在偏好设置中启用API访问
![](/images/2017-12-29-ttrssdeploy-2.png)

进入Fever Emulation 页面设置Fever密码
![](/images/2017-12-29-ttrssdeploy-3.png)

接下来在移动客户端上设定Fever的帐号密码，地址就可以同步自己订阅的所有rss文章了。

## 关于Rss全文输出

很多站点提供的Rss阅读源通常只会给出节选的文章段落，想要阅读全部内容需要打开文章链接才可以，非常影响Rss的阅读体验。可以采取两种方式来解决这个问题，一种是给tt-rss服务安装插件，另一种是直接使用第三方的服务，将无法显示全文的rss链接转化为全文rss链接。我这里推荐的是feedocean(https://feedocean.com)服务。可以不用注册帐号直接把自己需要的信息源的Rss链接复制到首页的输入框就可以生成rss全文输出的链接了。非常方便。当然也可以另外安装tt-rss插件的方法来实现全文输出，只是我个人觉得这种方法增加了tt-rss的管理成本，而且可能会出现插件版本于tt-rss版本不兼容的情况，所以直接使用了免费的 feedocean。


