---
layout: post
title: "kippo && cowrie"
data: 2019-05-20
tags: [security]
excerpt: "蜜罐 - 阅读笔记 - 信息收集"
comments: false
share: false
---

## KIPPO

### 解决什么问题？

有这么一种场景：一般使用SSH协议来进行远程登录，为了进行远程登录，用户要向ssh服务器进行身份验证。攻击者会在网络中进行端口扫描，找到指定的，可以提供远程登录服务的端口，他们使用扫描器，一旦找到了，就会进行字典攻击、暴力破解。这是有可能成功的，因为这些服务器的管理员有可能使用弱口令，而一旦攻击者获得了密码，他就可以通过wget命令下载恶意软件，利用这些下载的恶意软件以及一些基本的工具，可以实现提权等操作。

KIPPO是面向这种场景的蜜罐，它可以部署在一个攻击者感兴趣的网段里面，模拟ssh登录，它包含了一个假的文件系统，攻击者一旦破解了ssh的登录密码，获得一个SSH会话之后，会进入到这个假的文件系统，他可以任意地查看，创建，删除里面的文件，而不会影响到真实的数据，同时KIPPO会保存攻击者输入的所有命令以及他下载的所有恶意软件，用于后续的分析

所以，如果只是一个单独的KIPPO，这最多只能算是一个研究型蜜罐。它的目的是诱使攻击者进入到KIPPO的虚假文件系统，然后对攻击者进行监视记录，最后安全研究员通过记录知道攻击者的目的是什么，他想获取什么数据，他使用的是什么软件。

### KIPPO提供的一些功能

- 虚假文件系统：可以增加，删除文件，模拟了Debian5.0的文件系统
- 可以给虚假的文件添加内容，这样可以让用户通过cat等命令输出文件内容
- 可以保存会话记录（Session logs）
- 可以保存下载的恶意文件
- 可以保存攻击者使用的shell命令
- 可以模拟ssh会话结束

### KIPPO的文件结构

```bash
honeydrive@honeydrive:/honeydrive/kippo$ ls
data  fs.pickle    kippo           kippo.tac    public.key  stop.sh
dl     .gitignore  kippo.cfg       log          README.md   txtcmds
doc   honeyfs      kippo.cfg.dist  private.key  start.sh    utils
```

- Data：保存各种数据文件，比如密码数据库文件
- fs.pickle：Pyhton的pickle格式文件，保存了一个虚拟文件系统
- dl:默认的文件，保存攻击者下载的恶意文件
- kippo.cfg：KIPPO的配置文件
- honeyfs：包含了要显示给攻击者的文件
- txtcmds：用于创建简单的可以输出文本的命令
- log：默认文件夹，用于记录攻击者使用的shell命令
- start.sh：启动KIPPO
- utils：包含了各种Kippo工具，如，`playlog.py`可以回显攻击者的shell会话

### KIPPO的文件系统

KIPPO将它的文件系统保存在一个pickle文件里面。

pickle是一个对象序列化的标准技术。它使用了简单的基于栈的虚拟机，这个虚拟机记录了用于重构对象的命令。一个pickle文件是一种类似于部分内存转存（partial memory dump），即允许用户可以将对象以二进制的形式写入硬盘，又允许用户可以从内存中读取对象

每当有一个新的用户要连接进入KIPPO，KIPPO都会将一个新的文件系统载入内存，攻击者任意地使用这个文件系统，而，对于下一个新的用户，KIPPO会将另一个新的文件系统载入内容，也就是说，每次用户连接到KIPPO，都会使用一个新的文件系统。

由于这个原因，如果攻击者有防范蜜罐的意识，KIPPO是容易被识别的

### 使用

KIPPO在github上的项目已经好久都没有更新了，按照上面的wiki进行配置，发现ssh客户端无法链接，而issue的回答则是推荐使用另一个正在维护的蜜罐 - cowrie

cowrie是基于KIPPO开发的，也是官方推荐的一个分支，在原来基础上增加了很多新的功能

## Cowrie

相关的安装配置，参考[官方文档](https://cowrie.readthedocs.io/en/latest/INSTALL.html)

### 文件

项目的文件结构和原来的有点变化

- etc/cowrie.cfg:加载的配置文件
- share/cowrie/fs.pickle:虚拟文件系统
- etc/userdb.txt:登录蜜罐的用户名和密码
- honeyfs/:虚拟文件系统包含的文件内容
- var/log/cowrie/cowrie.json:json格式的输出内容
- var/lib/cowrie/tty:session log，可以使用bin/playlog进行回放
- var/lib/cowrie/downloads：存储攻击者下载的内容
- share/cowrie/txtcmds:存储简单的虚假命令
- bin/createfs:创建一个虚假的文件系统
- bin/playlog:用于回放session log

