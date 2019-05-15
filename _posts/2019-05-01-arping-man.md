---
layout: post
title: "arping manpage"
data: 2019-05-01
tags: [security, penetration test, kali linux]
excerpt: "arping"
comments: false
share: false
---

## arping

arping 主要的作用是发送`ARP`或者`ICMP`请求包到指定的主机，并显示返回的结果

主要的命令模式：

`arping [options] <host | -B>`

在这里，`host`，可以是主机名，也可以是IP地址，可以是MAC地址

请求包按照1秒1个的频率进行发送

当在ping一个IP地址的时候，一个`who-has`的`ARP`请求包会先发送。当在ping一个MAC地址的时候，一个`Echo`的`ICMP`请求包会以广播的形式发送。更多的技术原因要看README文档

关于时间，由于ARP包通常回复得很快，系统任务调度器（task scheduler）不能一直获得确切的时间。在一个空闲的系统，往返时间（roundtrip time - rtt）将会更加准确，但是随着负载的增加，时间会变得越来越不准确，在非空闲的系统，要通过`nice`校准时间，参数设定为`-15`或者其他。这不是arping独有的问题，其他的ping工具也有同样的问题。

```bash
nice -n -15 arping foobar
```

- 参数

`-0`：ping这个IP地址`0.0.0.0`，这个命令可以在还没有配置网卡的时候执行

`-a`：Audible ping

`-A`：只计算，和请求地址相同的地址（这个方法只适用于ping很多一次ping很多主机，如`arping-scan-net.sh`）

`-b`:和`-0`类似，不过是ping`255.255.255.255`.这个命令有可能得不到回复，因为它不是一般主机的操作

`-B`：使用主机去ping`255.255.255.255`

`-c <count>`:只发送`<count>`这么多的请求

`-C <count>`:只等待`<count>`这么多的回复

`-d`：寻找重复的响应。如果有两个不容MAC地址发送的，相同的响应，则退出，返回1（发现两个不同IP地址，但是MAC地址不同 - arp欺骗）

`-D`：显示的时候，将响应显示为`感叹号`，将丢失的包显示为`句号（英文）`

`-e`：像`-a`一样，但是会响，如果没有得到响应

`-F`：不尝试智能地显示网卡名称。即使没有这个参数，`-i`也可以禁止这个显示

`-g <group>`：给`<group>`进行setgid()

`-i <interface>`：不用猜测，使用指定的网卡

`-m <type>`:给将要收到的包添加时间戳的类型,`-vv`可以显示使用的类型

`-p`：打开网卡的混杂模式（promiscious mode），当你不’拥有‘你正在使用的MAC地址的时候，使用这个命令

`-P`：发送ARP响应而不是请求

`-q`：除了错误信息，不显示其他信息

`-Q <priority>`:设置`802.1p`的属性，应该要和`820.1Q`的标签(-V)一起使用。默认是0

`-r`：raw output，只有MAC/IP地址会在响应里显示

`-R`：和`-r`类似，但是显示的是`the other one`，可以和`-r`一起使用

`-s <MAC>`:可以设定源MAC地址，要和`-p`一起使用

`-S <IP>`:和`-b`，`-0`类似，不过是设置源地址。如果目标没有路由到这个IP地址，可能会得不到回复。如果使用的IP不是自己拥有的，可能要打开混杂模式`-p`。通过这个改变，可以找到主机拥有什么IP地址，而不用使用自己的IP地址

`-t <MAC>`:指定目标的MAC地址

`-T <IP>`:如果ping目标的MAC的时候，不对广播做出响应但是有可能对定向广播做出响应，可以用这个命令指向目标的地址

比如说，要找到MAC-A的地址，已知MAC-B和IP-B

```bash
arpping -S IP-B -s MAC-B -p MAC-A
```

`-u`:在ping NAC地址的时候显示index=received/sent，而不只是index=received

`-U`：发送自发的ARP包。这设置了目标的MAC地址子ARP帧去广播这个地址。自发的ARP包用来更新`neighbours's ARP caches`

