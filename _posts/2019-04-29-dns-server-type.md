---
layout: post
title: "DNS服务器类型 - 笔记"
data: 2019-04-29
tags: [security， network]
excerpt: "DNS笔记"
comments: false
share: false
---
## DNS服务器类型

[ref](https://www.cloudflare.com/learning/dns/dns-server-types/)

所有的DNS服务器都是以下四种类型之一：

- recursive resolvers 迭代解析器
- root nameservers 根域名服务器
- TLD nameservers TLD 域名服务器
- authoritative nameservers 权威域名服务器

1. recursive resolver 迭代解析器

    迭代解析器aka DNS解析器，是DNS查询里面第一个要经过的地方，迭代解析器就像客户机和DNS域名服务器的中间人

    当收到网络客户端的域名查询请求之后，迭代解析器要么返回缓存的数据，或者将请求发送到根域名服务器，等接收到回应之后，将请求发送到TLD域名服务器，最后，将请求发送到权威域名服务器，权威域名服务器将目标IP地址发送到迭代解析器，迭代解析器将结果作为响应返回给客户端

    在这个过程，迭代解析器会缓存权威服务器返回的结果

    大部分的互联网用户使用他们的ISP提供的迭代解析器

2. root nameserver 根域名服务器

    每一个迭代解析器，都知道13个DNS根域名服务器，而且他们是迭代解析器查询的第一个位置

    根域名服务器接收迭代解析器的域名查询，然后根域名服务器返回响应，这个响应将迭代解析器指向TLD域名服务器，具体取决与域名的拓展（.com,.net,.org）

    根域名服务器有一个非盈利组织监督 Internet Corporaton for Assigned Names and Number(ICANN)

    值得一提的是，虽然有13个根域名服务器，但是这不是说，在根域名系统里面只有13个机器，虽然这13种根域名服务器，但是他们每一个在世界各地都有复制，这些分布的服务器使用任播（anycast routing）的方式提供快速的响应（一共大概有600+个服务器）

3. TLD nmeserver 顶级域名服务器

    TLD名称服务器维护共享公共域扩展名的所有域名的信息，例如.com，.net或网址中最后一个点之后的任何域名。比如拓展名为`.com`，TLD域名服务器包含了所有以`.com`结尾的网站信息

    比如，用户搜索`google.com`，主机在收到来自根域名服务器的响应之后，迭代解析器会发送请求到`.com`的TLD域名服务器，得到的响应将指向权威域名服务器

    TLD域名服务器由`Assigned Numbers Authority - IANA` 维护，这个机构是ICANN的一个分支。

    IANA将TLD域名服务器分为两组：

    - Generic top-level domains 通用顶级域名，这些域名不属于任意一个国家的，一些有名的通用TLD如：`.com .org .net .edu .gov`

    - Country code top-level domains 国家/地区代码顶级域名，这些域名是特定于某个国家或者地区的，如`.uk .us .ru .jp`

    还有第三类的针对于（国家或者机构）基础设施的域名，但是现在基本都不使用了，这个分类是针对`.arpa`域名划分的，这个域名是创建现代DNS的过渡域名，现在也只剩下了历史意义

4. authoritative nameserver 权威域名服务器

    当迭代解析器接收到来自TLD的回应之后，这回应将迭代解析器指向一个权威域名服务器。权威域名服务器，通常是寻找IP地址这个过程中的最后一步。

    权威域名服务器，包含了他所服务的域名的信息（比如google.com）,它可以提供递归解析程序，其中包含DNS A记录中找到的该服务器的IP地址，如果域名是CNAME记录（别名），他会提供一个迭代解析器，里面包含了CNAME记录，这个迭代解析器将会执行一个新的DNS查询，从权威域名服务器获取记录
