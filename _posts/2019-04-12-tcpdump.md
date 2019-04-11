---
layout: post
title: "TCPDUMP基本使用"
data: 2019-04-12
tags: [security， web_security]
excerpt: "一个字符界面的抓包软件，是几乎所有Linux、Unix系统默认安装的软件"
comments: false
share: false
---

## TCPDUMP

1. 什么事TCPDUMP？

    一个字符界面的抓包软件，是几乎所有Linux、Unix系统默认安装的软件

2. 常用参数

    ```bash
    root@kali:~# tcpdump -h
    tcpdump version 4.9.2
    libpcap version 1.8.1
    OpenSSL 1.1.1b  26 Feb 2019
    Usage: tcpdump [-aAbdDefhHIJKlLnNOpqStuUvxX#] [ -B size ] [ -c count ]
            [ -C file_size ] [ -E algo:secret ] [ -F file ] [ -G seconds ]
            [ -i interface ] [ -j tstamptype ] [ -M secret ] [ --number ]
            [ -Q in|out|inout ]
            [ -r file ] [ -s snaplen ] [ --time-stamp-precision precision ]
            [ --immediate-mode ] [ -T type ] [ --version ] [ -V file ]
            [ -w file ] [ -W filecount ] [ -y datalinktype ] [ -z postrotate-command ]
            [ -Z user ] [ expression ]
    ```

    `-i` interface，指定监听端口

    `-s` snapshot-length 快照长度，也就是抓取一个包的长度，下面是来自man page的关于`-s`的说明

        每个数据包的Snarf快照长度字节数，而不是默认值262144字节。

        由于快照有限而被截断的数据包在输出中用``[| proto]''表示，其中proto是发生截断的协议级别的名称。

        请注意，拍摄较大的快照会增加处理数据包所需的时间，并且有效地减少数据包缓冲量。

        这可能会导致数据包丢失。

        您应该将snaplen限制为捕获您感兴趣的协议信息的最小数字。

        将snaplen设置为0会将其设置为默认值262144，以便与最新版本的tcpdump向后兼容。

    也就是说，如果不指定这个值，在这个版本的tcpdump默认是抓取262144字节/包，而赋值0则表示使用默认值

    `-n` 不将数据包的地址做名称解析，不显示域名，读取文件的时候，使用这个参数会快很多

    `-w <filename>` 将抓取的数据包保存到指定文件，如果指定了这个参数，在shell里面是没有输出的。另外输出的文件，tcpdump不会自动添加后缀，根据manpage的说明，最好使用`.pcap`的后缀

    `-r <filename>` 读取一个指定的数据包，根据manpage的说明，tcpdump在读取这个文件的时候，是不去检查后缀的，也就是说，只要这个文件包含了符合数据包形式的内容，不管后缀是`.pcap`, `.dmp`, `.cap`都可以读取

        ```txt
        The MIME type application/vnd.tcpdump.pcap has been registered with IANA for pcap files. The filename extension .pcap appears to be the most commonly used along with .cap  and  .dmp.  Tcpdump itself  doesn't  check the extension when reading capture files and doesn't add an extension when writing them (it uses magic numbers in the file header instead). However, many operating sys‐tems and applications will use the extension if it is present and adding one (e.g. .pcap) is recommended.
        ```

    `-A` 以ASCII形式解析每个数据包

    `-X`以十六进制的形式解析每个数据包

3. 抓包过滤器

    tcpdump的抓包过滤机制参考le`pcap-filter(7)`，可以通过`man pcap-filter`查看具体内容，只有当过滤器表达式的值为真的时候，数据包才会被抓取

    部分翻译：

    在`pacp-filter`，过滤器表达式由一个或者多个元素（primitive）组成，元素通常由一个`id`（名字或者数字）以及一个或者多个前缀组成。

    有三种类型的前缀：

    - type

        `type`表示`id`（名字或者数字）表示的是什么东西，`type`可以是：`host`, `net`, `port`, `portrange`

        如果没有指定`type`的值，默认使用`host`

    - dir

        `dir`， direction，表示`id`特定的传输方向，`dir`可以是：`src`,`dst`,`src or dst`, `src and dst`, `ra`, `ta`, `addr1`, `addr2`, `addr3`, `addr4`等。

        其中,`ra`, `ta`, `addr1`, `addr2`, `addr3`, `addr4`只适用于IEEE 802.11 无线局域网

        如果没有指定`dir`，默认值是`src and dst`

    - proto

        `proto`, protocol, 指定一个特定的协议，`proto`可以是:`ehter`,`fddi`,`tr`,`wlan`,`ip`,`ip6`,`arp`,`rarp`,`decnet`,`tcp`,`udp`等

        如果`proto`没有指定值，默认情况下要根据`type`来确定默认值，如

        ```bash
        # src foo
        (ip or arp orrarp) src foo

        # net foo
        (ip or arp orrarp) src foo

        # port 53
        (tcp and udp) port 53
        ```

    除了上述三种类型的元素，还有一些特殊的元素：`gateway`,`broadcast`,`less`,`greater`以及算术表达式

    元素之间可以通过`and`,`or`,`not`来进行连接组成更复杂的过滤器表达式

    更多的参数说明要参考`man pcap-filter`

4. 显示筛选

    - 利用系统命令进行筛选

        思路是，先读取抓包文件的内容，再利用管道符传到其他命令做筛选

        如，获取抓包文件中所有的源地址：

        ```bash
        tcpdump -r package -n | awk 'BEGIN{print "======src address======"}{print $3}' | sort -u
        ```

    - 利用过滤器

        同样可以使用`pcap-filter`和抓包过滤器的使用没有不同，区别是不用`-i`参数，而是使用`-r`参数

5. 对TCP包的筛选

    在`man tcpdump`里面，有一小节讲到如何按TCP包的字段进行筛选

    对于TCP包头，一共有20个字节

    ```bash
    0                            15                              31
    -----------------------------------------------------------------
    |          source port          |       destination port        |
    -----------------------------------------------------------------
    |                        sequence number                        |
    -----------------------------------------------------------------
    |                     acknowledgment number                     |
    -----------------------------------------------------------------
    |  HL   | rsvd  |C|E|U|A|P|R|S|F|        window size            |
    -----------------------------------------------------------------
    |         TCP checksum          |       urgent pointer          |
    -----------------------------------------------------------------
    ```

    现在假设我们向对`tcpflags`进行筛选，如，只显示`syn`包

    `tcpflags`是第13个字节（从零开始数），在过滤器表达式里面可以这样表示：`tcp[13]`或者`tcp[tcpflags]`（同理，其他tcp字段也可以这样写）

    ```bash
    |               |
    |---------------|
    |C|E|U|A|P|R|S|F|
    |---------------|
    |7   5   3     0|
    ```

    这个字节，从右到左，是0到7，所以目标`syn`是第2位，现在只想要`syn`的tcp包，那么这个字节就是下面的形式：

    ```bash
    |C|E|U|A|P|R|S|F|
    |---------------|
    |0 0 0 0 0 0 1 0|
    |---------------|
    |7 6 5 4 3 2 1 0|
    ```

    最后算出来十进制值是2，所以在过滤器表达式里面就可以这样写`tcp[13]==2`或者`tcp[13]==tcp-syn`（同理，其他的falg值也可以这样写）

    还有一种情况就是，只想要`syn`的包，而无论是否还有其他的flag值

    这种情况可以通过逻辑与进行处理，通过将这个字段的值和`tcp-syn`的值进行逻辑与，如果最后得到的值仍然是`tcp-syn`，则抓取这个包

    ```bash
         00010010 SYN-ACK              00000010 SYN
    AND  00000010 (we want SYN)   AND  00000010 (we want SYN)
        --------                      --------
    =    00000010                 =    00000010

    ```

    ```bash
    tcpdump -i wlan0 'tcp[13] & tcp-syn == tcp-syn'
    ```

    注意，过滤器表达式要使用引号或者反斜杠去隐藏shell里面的特殊字符`&`
