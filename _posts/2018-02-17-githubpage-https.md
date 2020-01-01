---
title: github page给自定义域名加上小绿锁
date: 2018-02-17
category: 
- Tooltips
tags:
- githubpage
- https
- cloudflare
---

## 目的
在浏览器中访问绑定域名的 github page 页面，默认显示的不是 https 状态的链接。我希望访问的时候能够显示 https 状态。

## 替换域名伺服器
找把梯子注册登录 Cloudflare, 根据提示填写自己的域名。进入DNS管理面板，找到 Cloudflare 给出的域名服务器地址。
![](/images/2018-02-17-githubpage-https-1.png)

然后进入自己申请域名的网站，这里以 Goddady 为例。修改域名服务器为Cloudflare给出的地址。
![](/images/2018-02-17-githubpage-https-2.png)

## 设置ssl证书加密
进入 crypto 管理面板，这里CloudFlare会提供一个有效期 15 年的ssl证书。选择Full加密或者Flexible皆可。
![](/images/2018-02-17-githubpage-https-3.png)

最后只要设置Page Rules 即可。利用CloudFlare的Page Rules 功能可以将我的 [blog.gilbertwong.co](https://blog.gilbertwong.co)的所有文件都启用https加密。

## 注意
如果你和我一样，在 github 上都是用 jekyll 构建静态博客的话，请将 _config.yml 配置文件中的
```
url:              https://gilbert-wong.github.io # 域名
```

改为

```
url:              https://blog.gilbertwong.co # 域名
```

否則可能會出现 css 文件 无法正确在浏览器中加载的问题。
