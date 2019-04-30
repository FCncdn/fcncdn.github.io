---
layout: post
title: "使用BIND搭建一个DNS服务器"
data: 2019-04-30
tags: [security， network]
excerpt: "DNS笔记"
comments: false
share: false
---

## 搭建dns服务器

根据对dns服务器类型的理解，为了模式真实的企业DNS服务器搭建，要搭建的应该是权威域名服务器，里面存储了目标企业的所有（部分）域名信息

过程应该是：使用本地的DNS迭代解析器，解析一个域名（存在与构造的dns服务器里面），并将dns服务器地址指定为构造的dns服务器

这里使用BIND

## 什么是BIND？

[来自wiki](https://zh.wikipedia.org/wiki/BIND)

BIND - Berkeley Internet Name Domain

现在互联网上最常用的DNS软件，使用BIND作为服务器软件的DNS服务器约占所有DNS服务器的九成，BIND现在由互联网系统协会（Internet Systems Consortiem）负责开发和

## 如何获取DNS服务器的BIND版本

```bash
# 获取DNS服务器
dig stu.edu.cn ns

# 查询其中一个DNS服务器的BIND版本信息
dig +noall +answer txt chaos VERSION.BIND @dns1.stu.edu.cn.
```

返回结果：

```bash
VERSION.BIND.   0   CH  TXT "STU BIND 9.10"
```

猜测，校园网使用的是`BIND 9.10`

## 利用BIND部署一个DNS服务器

[ref1](https://linuxtechlab.com/configuring-dns-server-using-bind/)

[ref2](http://www.servermom.org/how-to-install-and-setup-bind9-on-ubuntu-server/)

[ref3](https://wiki.ubuntu.org.cn/Bind9%E5%AE%89%E8%A3%85%E8%AE%BE%E7%BD%AE%E6%8C%87%E5%8D%97#Master_Server.EF.BC.88.E4.B8.BB.E6.9C.8D.E5.8A.A1.E5.99.A8.EF.BC.89)

[ref4](https://blog.csdn.net/lyhDream/article/details/77620932)

方案：设定一个DNS服务器A，使用ubuntu，安装的是`bind 9.11.3-lubuntu1.7-Ubuntu`

```text
DNS服务器
    IP:192.168.43.23
```

DNS服务器A存储了3条记录：

`aaa.fcncdn.com IN  A   192.168.43.21`

`bbb.fcncdn.com IN  A   192.168.43.22`

`ccc.fcncdn.com IN  CNAME bbb`

1. 修改conf配置文件

    bind的主配置文件是`named.conf`，里面include了三个文件

    在`named.conf.local`添加一条记录:

    ```bash
    zone "fcncdn.com"{
        type master;
        file "/etc/bind/db.fcncdn.com"
    }
    ```

    在`named.conf.options`添加如下记录：

    ```bash
    listen-on port 53 {127.0.0.1;192.168.43.23;};
    allow-transfer {none;};
    ```

2. 添加区数据文件（file 指定的文件）

    ```bash
    $TTL 604800
    @   IN  SOA fcncdn.com. root.localhost. (
         2019043122     ;Serial
         604800         ; Refresh
         86400          ; Retry
         2419200        ; Expire 604800 ) ; Negative Cache TTL
    ;
    @ IN NS localhost.
    fcncdn.com IN NS 192.168.43.23

    aaa IN A 192.168.43.21
    bbb IN A 192.168.43.22
    ccc IN CNAME bbb
    ```

    每次修改这个文件，都要更新`Serial`，这个字段一般用这种格式表示`yymmddss`

3. 重启bind9服务

    `service bind9 restart`

4. 测试

    在同一网段的另一台主机

    ```bash
    dig @192.168.43.23 fcncdn.com
    ```

    可以看到对应的信息，如果`fcncdn.com`对应的地址部署了主机，需要修改`/etc/resolve.conf`，添加192.168.43.23才能访问