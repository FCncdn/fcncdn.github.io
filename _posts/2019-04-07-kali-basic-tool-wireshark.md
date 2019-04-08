---
layout: post
title: "kali基本工具 - wireshark "
data: 2019-04-07
tags: [security, penetration test, kali linux]
excerpt: "wireshark 的基本使用"
comments: false
share: false
---

## Wireshark的基本使用

wireshark主要工作是数据包协议分析。抓包的工作是Libpcap9（linux平台），wireshark将libpcap9抓到的包进行解码，得到一个完整的包。

衡量一个抓包分析工具的重要指标是它的解码能力。

注意，wireshark默认是通过端口来识别协议的，如HTTP是80，有一种情况是，HTTP使用其他端口，这种情况要手动解包，选择对应的协议在网络层进行解包

wireshark的基本使用包括：

- 启动
- 选择抓包网卡
- 混杂模式
- 实时抓包
- 保存和分析捕获文件
- 首选项

1. 混杂模式

    如果不开启混杂模式，网卡只会抓取源或者目的地是本机的数据包，以及网络里面广播的包，而经过这个网卡的但是目的地不是这个网卡的数据包则不会抓取

2. 抓包筛选器

    作用是，通过命令的形式过滤数据包，可以只抓取某种协议的包，也可以不抓取某种协议的包，可以抓取某种协议某个端口的包等

    关键字：`and`,`or`,`only`，`host`，`ip`

    抓到的包可以保存，保存的格式最好是`.pcap`，这是很多抓包工具可以识别的格式

3. 首选项

    更改布局格式，以及添加显示的列

## 常见协议类型分析

- wireshark数据包分层

    这里的分层指的是2号窗口的分层

    第一层`Frame`是wireshark提供的对这个数据包数据分析，从第二层开始才是包的内容

    第二层`Ethernet`，以太网层，里面有三层内容：  
  - 第一层`Destination`目的地址
  - 第二层`Source`源地址
  - 第三层`Type`上层使用协议类型

- 第三层就是网络层的各种协议

    1. 第三层协议 - arp

        arp包在wireshark里面显示有9层内容

        `Hardware type` 硬件地址类型，1 表示以太网  

        `Protocol type` 协议类型，这里显示的是要转换的协议，这些协议都会被转换为硬件地址

        `Hardware size` 硬件地址长度  

        `Protocol size` 协议长度  

        `Opcode` 操作代码，1 表示请求  

        `Sender Mac address` 发送方的mac地址  

        `Sender IP address` 发送方的ip地址  

        `Target Mac address` 目标的mac地址  

        `Target IP address` 目标的ip地址  

    2. 第三层协议 - IP

        `Version` 版本号  

        `Header length` 头长度  

        `Differentiated Serviecs Field` dsf  

        `Total Length` 总长度（IP包头到数据字段结尾的长度）  

        `Indentification` IP的ID  

        `Flags`包的类型  

        `Time to live` TTL生存时间（不同操作系统不同）  ，每过一个路由，ttl的值就减一，如果减到0还没有到达目的地，这个包就会被丢弃。不同操作系统的TTL是不同的：window：128、linux：64  

        `Protocol` 上层使用的协议，这个字段占用了一个字节，也就是说，一共有255个协议可以写在这里（0没有效果），其中TCP（6），UDP（17），ICMP（1），IGMP（2）  

        `Header checksum` IP头校验值，每两个值做异或运算，这里可以看到IP包是否又被其他人修改  

        `Source` 源IP地址  

        `Destination` 目的IP地址  

        `Option`，不是每一个数据包都有这个字段，通常的`TCP`和`UDP`是没有这个字段的  


- 第四层

    1. TCP

        `Source Port` 源端口

        `Destination Port` 目的端口

        `[Stream index]` 流索引

        `[TCP Segment Len]` 

        `Sequence number` 序列号

        `Acknowledgment number` 指定获取下一个TCP包的`Sequence number`

        `Header Length` 头长度（TCP包头的头长度）

        `Flags` 表示TCP包的类型

        `Window size vlaue` 窗口大小

        `Checksum` 校验值

        `Urgent pointer`

        `Options`

        TCP的三次握手，TCP面向连接的协议，每次传输数据包，目标都要返回一个确认值，如果没有返回，发送方会从新发送这个包。TCP三次握手是双方建立连接的过程。三次握手的包都会以发送方发送syn开始，接收方接收并放回一个syn ack包，最后发送方再次发送一个ack包结束。三次握手之后，就开始传输数据，每发送一个数据，对方都要返回一个ack的确认包

- 应用层

    1. DNS

        DNS是基于UDP的一个协议

        `Transaction ID` 传输ID

        `Flags` 包的类型

        `Question` 如果是1表示是提问的

        `Answer RRS`

        `Authority RRS` 如果是1表示是一个返回包，是一个权威的查询

        `Additional RRs`

        `Queries` 数据字段，包括了：`name`域名，`Type`域名类型，`Class`IN表示internet

        `Answers`

        `Authoritative nameservers`

    2. HTTP协议

        HTTP是基于TCP协议的

        请求包
        `Request Method` 请求方法

        `Request URI` 请求地址

        `Request Version` 请求版本

        `Host` 主机名

        `User-Agent` 客户端浏览器的信息

        `Accept` */*

        `Accept-langue`

        `Accept-Encoding`

        `Referer`

        `Connection` 连接

        `Cookie` 

        空行

        HTTP数据

（这里的协议说明，需要计算机网络更详细的补充）

## 数据流

数据流，是完成传输一个页面的一组数据包

## 企业级抓包部署方案

sniffer

Cace / riverbed /  cascad pilot

1. wireshark的不足

在抓取大流量的情况，wireshark的表现不足

cace是基于wireshark二次开发的软件，对于大文件的打开、分析速度比wireshark快

企业一般要求全流量抓包，一般的做法是，在内网核心的三层路由器上，做镜像端口，将vlan与vlan之间的流量镜像到一个口上，将路由连接防火墙的端口也镜像到那个指定的端口

镜像的意思是，将所有端口的流量全部复制到另一个端口，在这个端口连接一个开启抓包软件的主机，这个端口只能收包，这个端口的主机一个网卡用来抓包，一个网卡用以网络通信，允许管理员进行远程分析查看

如果遇到流量很大的情况，当流量超过了端口的带宽会出现丢包，可以通过多块网卡的捆绑增加带宽
