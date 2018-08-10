---
title: 树莓派 Erlang 环境配置
date: 2018-08-05
category: 
- Management information system
tags:
- Raspberry
- erlang
- zsh
---

最近在尝试用树莓派搭建集群，这里来写一篇博客来记录一下我折腾的过程。

##  安装
首先是安装 Raspberry 到 sd 卡，这里没什么好说的，先去[官网](https://www.raspberrypi.org/downloads/)下载镜像，
然后执行`diskutil` 命令去查看所有存储设备:

```bash
diskutil list
```

找到你的 SD 卡设备后先通过下面这条命令卸载设备：

```bash
diskutil unmount /dev/disk3s1
```

然后通过 `dd` 将下载且解压出来的后缀名 `img` 的镜像文件写入设备(即 SD 卡):

```bash
sudo dd if=./raspberry.img of=/dev/disk3s1
```
 写入完毕后，开启树莓派电源，接上显示器，屏幕上如果显示树莓派的图像，就表示安装成功
 
## 基本配置

树莓派默认的帐号是 pi，密码是 raspiberry。登录进去以后我们需要做一些基本的配置。

我使用的树莓派 3B 是带 wifi 模块的，所以没有必要都通过极其不方便的以太网 RJ-45 接口去连接本地路由器，树莓派已经有
非常方便的 raspi-config 命令去修改一些诸如时区，locale，wifi 连接的配置等等。下图是执行 raspi-config 后的操作界面。　
![](/images/2018-08-09-RaspiberryConfig-01.png)

这里注意一下，`raspi-config` 的 locale 设置并不足够完善，所以我们还需要通过 vi 来编辑一下 `/etc/default/locale`，
改成这样：
```bash
LANG=en_US.UTF-8
LC_ALL=en_US.UTF-8
LANGUAGE=en_US.UTF-8
```
然后重启。

## 软件源配置
由于国内网络问题，raspiberry 通过软件源去下载更新软件的时候速度不是很理想，这里将默认软件源换成阿里云的镜像，除此之外
为了之后科学上网提高 github clone 速度，还需要添加 Debian backports 的源去下载 shadowsocks-libev。（树莓派默认的 
Raspbian 发行以是基于 Debian 的制作而成的）。我用的是 raspbian 是 stretch 版，因此需要添加 stretch 版的源。

修改 /etc/apt/sources.list 文件

```bash
deb http://mirrors.aliyun.com/raspbian/raspbian/ stretch main contrib non-free rpi
deb http://ftp.debian.org/debian stretch-backports main
```
添加完 Debian backports 的源后还需要导入 GPG 公钥，先安装 dirmngr
```bash
apt-get install dirmngr
gpg --keyserver pgpkeys.mit.edu --recv-key  8B48AD6246925553    
gpg -a --export 8B48AD6246925553 | sudo apt-key add -    

gpg --keyserver pgpkeys.mit.edu --recv-key  7638D0442B90D010    
gpg -a --export 7638D0442B90D010 | sudo apt-key add -    
```
导入完毕后立即`apt update & apt upgrade -y`


##  配置科学上网工具
树莓派自带的源中当然也是带有 shadowsocks-libev 的，但是之所以我上面还是添加 debian backports 源就是因为默认自带的软件源中
的 shadowsocks-libev 非常旧，甚至不支持 AEAD 加密。在配置完源之后只需要简单地执行：

```bash
apt-get install shadowsocks-libev -t stretch-backports
```

接下来是配置 `shadowsocks-libev` 的配置文件
```bash
vim /etc/shadowsocks-libev/ss-local.json
```

配置文件基本和服务器端的配置文件一致，格式如下：

```json
{
	"server":"your server ip",
	"server_port": your server port,
	"method": "crypto method",
	"password": "your password",
	"local_address": "127.0.0.1",
	"local_port": 1080
}
```
写好配置以后执行：
```bash
ss-local -c /etc/shadowsocks-libev/ss-local.json ＆
```

shadowsocks 本质上还是代理软件，而非 VPN，所以在使用命令行的时候没法直接通过 shadowsocks 开启的通道去连接被 GFW 严重限速
过的，这里还需要下载另外一个工具`proxychains`

执行命令：

```bash
apt install proxychains -y
```

和 shadowsocks-libev 一样，proxychains 也需要简单地配置，编辑：

```bash
vim /etc/proxychains.conf
```

这个在最后几行将 \[ProxyList\] 中的 socks 改为 shadowsocks 在本地圆环地址开启的 socks5 端口，如下：

```bash
[ProxyList]
socks5 127.0.0.1 1080
```
用法很简单: proxychains + 你的命令

## 增强 shell
 树莓派自带 shell 是 bash，非常垃圾，也不好扩展，我这里改用 zsh。
 
 一行命令安装 zsh：

```bash
apt install zsh -y
```

一行命令切换默认 shell:
```bash
chsh -s /usr/bin/zsh
```

一行命令安装 oh-my-zh:
```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
``` 

切换主题到 `ys`
安装几个未来会用到的 zsh 插件： `git`, `asdf`, `zsh-autosuggestions`, `autojump`, `zsh-reload`, `dotenv`, `zsh-completions`.

~/.zshrc 配置文件大概如下：

```bash
# Set name of the theme to load. Optionally, if you set this to "random"
# it'll load a random theme each time that oh-my-zsh is loaded.
# See https://github.com/robbyrussell/oh-my-zsh/wiki/Themes
ZSH_THEME="ys"
plugins=(
  git
  asdf
  zsh-autosuggestions
  zsh-completions
  zsh-reload
  dotenv
  autojump
)
```

还需要安装相应的工具去给 zsh 使用:

```
apt install autojump
proxychains git clone git://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
```

## 安装 asdf
可以看到我刚刚在 zsh 安装了一个名为 `asdf` 这个插件，`asdf` 是一个非常方便的语言运行时和数据库的版本管理软件工具，由于我刚刚已经在 zshrc
 文件文件中配了 asdf 插件，所以我这里只需要

```bash
git clone https://github.com/asdf-vm/asdf.git ~/.asdf --branch v0.5.1
source .zshrc
```
输入 asdf
```bash
asdf
version: v0.5.1

MANAGE PLUGINS
  asdf plugin-add <name> [<git-url>]   Add a plugin from the plugin repo OR, add a Git repo
                                       as a plugin by specifying the name and repo url
  asdf plugin-list                     List installed plugins
  asdf plugin-list --urls              List installed plugins with repository URLs
  asdf plugin-list-all                 List plugins registered on asdf-plugins repository with URLs
  asdf plugin-remove <name>            Remove plugin and package versions
  asdf plugin-update <name>            Update plugin
  asdf plugin-update --all             Update all plugins


MANAGE PACKAGES
  asdf install <name> <version>        Install a specific version of a package or,
                                       with no arguments, install all the package
                                       versions listed in the .tool-versions file
  asdf uninstall <name> <version>      Remove a specific version of a package
  asdf current                         Display current version set or being used for all packages
  asdf current <name>                  Display current version set or being used for package
  asdf where <name> <version>          Display install path for an installed version
  asdf which <name>                    Display install path for current version
  asdf local <name> <version>          Set the package local version
  asdf global <name> <version>         Set the package global version
  asdf list <name>                     List installed versions of a package
  asdf list-all <name>                 List all versions of a package


UTILS
  asdf reshim <name> <version>         Recreate shims for version of a package
  asdf update                          Update asdf to the latest stable release
  asdf update --head                   Update asdf to the latest on the master branch


"Late but latest"
-- Rajinikanth
```

安装成功！

## 安装 Erlang 运行时

终于到了最后安装 Erlang 运行时的时候了，先

```bash
proxychains asdf plugin-add erlang
proxychains asdf list-all erlang
```

然后安装我们需要的最新版本:

```bash
proxychains asdf install erlang 21.0.4
```

当然在此之前，还需要将 build Erlang/OTP 所需要的依赖安装好：

```bash
# Install Java Development Kit
apt install openjdk-9-jdk -y

# Install the build tools (dpkg-dev g++ gcc libc6-dev make)
apt-get -y install build-essential

# automatic configure script builder (debianutils m4 perl)
apt-get -y install autoconf

# Needed for HiPE (native code) support, but already installed by autoconf
# apt-get -y install m4

# Needed for terminal handling (libc-dev libncurses5 libtinfo-dev libtinfo5 ncurses-bin)
apt-get -y install libncurses5-dev

# For building with wxWidgets (observer needs this)
apt-get -y install libwxgtk2.8-dev libgl1-mesa-dev libglu1-mesa-dev libpng-dev erlang-wx

# For building ssl (libssh-4 libssl-dev zlib1g-dev)
apt-get -y install libssh-dev libssl-dev

# ODBC support (libltdl3-dev odbcinst1debian2 unixodbc)
apt-get -y install unixodbc unixodbc-dev

# Install tools for erlang Documetation Generate
apt -y install xsltproc fop libxml2-utils
```

如果是树莓派 3B 的话，安装 Erlang 21.0.4 大概需要一个小时 20 分钟。

![](/images/2018-08-09-RaspiberryConfig-02.png)

安装成功。

最后执行

```bash
asdf global erlang 21.0.4
```
环境变量设定成功！

现在可以在上面跑我们的 Erlang 应用了。

## Privoxy 安装配置

这里为什么我们要在安装 `ProxyChains` 后还要去加一层 http 代理？因为 `proxychains` 构建 Erlang 应用的通用工具 `rebar3`
无法使用 proxychains，至于为什么无法使用？ 我不知道，但是可以参考这条 stackoverflow 上的
[回答](https://stackoverflow.com/a/51202404/8490474)，用 Privoxy 做 http 代理或许可以解决问题。

首先输入命令安装：
```bash
apt install privoxy -y
```
然后编辑 `/etc/profile` 编写转发转发配置：

```bash
> vim /etc/profile
export http_proxy=http://127.0.0.1:8118
export https_proxy=http://127.0.0.1:8118
> source ~/.zshrc
```

 也可以先在当前 session 中执行
```bash
export http_proxy=http://127.0.0.1:8118
export https_proxy=http://127.0.0.1:8118
```

然后修改 privoxy 的配置文件:
```bash
> vim /etc/privoxy/config
```
将 `#forward-socks5t   /   127.0.0.1:9050` 改为 `forward-socks5t  / 127.0.0.1:1080`。

测试翻墙
```
curl www.google.com
```

如果没有成功，请尝试重启 shadowsocks-libev 和 privoxy 再测试。


如果全部安装配置好，就可以开始正常地构建 Erlang 应用了。
