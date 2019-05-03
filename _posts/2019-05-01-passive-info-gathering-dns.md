---
layout: post
title: "被动信息收集 - DNS"
data: 2019-05-01
tags: [security, penetration test, kali linux]
excerpt: "被动信息收集 - DNS"
comments: false
share: false
---

Passive information gathering.

关于DNS，参看笔记[DNS]

## 信息收集阶段

1. 开源智能(Open Source Intelligence)

    被动信息收集也称为*开源智能*，

    被动信息收集是基于公开渠道进行的信息收集，不对目标系统进行直接的访问、扫面等操作，收集的手段是基于搜索引擎、媒体等。

    被动信息的标准是，不对目标系统进行大规模的探测，尽量不留下痕迹

    美国军方以及北大西洋组织对开源智能都有说明

2. 信息收集阶段

    - 被动信息收集阶段

    - 正常交互的信息收集

    - 主动信息收集（扫描探测等）

3. 信息收集内容

    - 目标系统的IP地址段
    - 域名信息
    - 邮件地址（可以定位到邮件服务器）
    - 在公网公开的文档和图片信息（有可能会泄露组织的信息）
    - 公司地址（实地无线网络渗透、实地物理渗透）
    - 公司组织架构（根据不同部门展开不同社会工程学的攻击）
    - 联系电话、传真号码
    - 人员姓名、人员职务
    - 目标系统使用的技术架构（基于什么平台开发，使用什么技术）
    - 公开的商业信息

4. 信息用途

    利用收集到的信息，重构目标公司的运行；发现目标公司使用的系统、主机、服务等；收集到的信息可以作为社会工程学攻击的基础；发现额外的物理缺口

## 被动信息收集 - DNS

1. 域名和FQDN(Fully qualified domain name)的区别

    如`www.baidu.com`

    其中，`baidu.com`是域名，而`www.baidu.com`是FQDN([完整限定域名](https://zh.wikipedia.org/wiki/%E5%AE%8C%E6%95%B4%E7%B6%B2%E5%9F%9F%E5%90%8D%E7%A8%B1))

    一个域名可能包含大量的域名记录，一条域名记录就是FQDN，完全限定域名

2. 什么是FQDN？

    Fully qualified domain name，完全限定域名，可以指定在域名系统树状图下一个确切的位置，没有模糊空间。简单来说，他是用来指定一个特定的服务器的。

    完全限定域名由主机名称和母网域名组成，如主机名称`myshot`，母网域名`example.com`，那么完全限定域名就是`myhost.example.com`

## DNS收集工具 - nslookup

nslookup有两种模式，交互式，非交互式

交互式允许用户查询各种主机的或者域名的域名服务器信息，或者打印一个域名包含的所有主机信息，进入交互式有两种方法：

- 不添加参数
- 第一个参数是`-`，第二个参数是域名服务器的主机名或者是域名服务器的IP地址

```bash
nslookup
```

非交互式，只能打印主机或者域名的名字和请求信息，使用非交互式的方法

- 第一个参数是要查找的IP地址或者主机名，第二个可选参数可以是要指定的域名服务器的主机名或者是IP地址

```bash
nslookup www.baidu.com 8.8.8.8
```

1. nsloopup交互式命令

    `host`[server]：使用默认的域名服务器查找`host`的信息，可以使用指定的域名服务器`[server]`，如果`host`是一个IP地址并且查找的类型是`A`或者`PTR`，那么会返回主机的名称

    `server`：如果不指定参数，返回当前使用的DNS服务器信息，如果指定了参数则把参数作为域名服务器

    `set keyword[=value]`这里的`keword`有多种

    - all

        打印目前设置的各种参数

    - class=value

        将当前的查询class改为：

        `IN`：Internet class

        `CH`：Chaos class

        `HS`：Hesiod class

        `ANY`：widlcard

    - [no]debug

        如果是`debug`，可以查看回应包的所有内容，以及在查找中发现的中间回应包的所有内容。默认是`nodebug`。（可以查看CName的转换过程）

    - [no]d2

        真正的debug模式，可以详细地看到nslookup在干什么(输出了很多信息，包括引用函数的顺序，以及各种在做什么的信息)。默认是`nod2`

    - port=value

        改变TCP/UDP域名服务器的端口值，默认是53

    - type=value

        改变要查找的DNS记录类型，默认是`a`，也就是A记录

    - [no]recurse

        `recurse`，告诉域名服务器，如果找不到信息则从找另一个服务器查询，默认是`recurse`

    - ndots=number

        限制域名里面的`.`的数量，如果超过这个数值就停止查询。

    - timeout=number

        改变默认设置的超时时间

2. 使用的文件

    ```bash
    /etc/resolv.conf
    ```

## DNS收集工具 - dig

1. dig 参数

    dig，除了命令行的使用方式，还拥有过批处理功能，这种方式允许从一个文件读取查询请求。

    在查询的时候，除非指定了一个域名服务器，否则dig会从`/etc/resolve.conf`依次尝试每一个域名服务器，如果找不到一个可用的域名服务器，dig会将请求发送到本地主机

    在命令行操作时，如果不指定参数，dig会执行根域名`.`的`NS`查询

    可以填写一些命令到`${HOME}/.digrc`，这个文件会被读取并且，里面的命令会在dig命令行指定的参数之前调用

    一个典型的dig命令是这样子的：

    ```bash
    dig @server name type
    ```

    `server`:要查找的IP地址或者域名，可以是点分十进制的IPv4地址，也可以是分号分割的IPv6地址，如果给定的是主机名，dig会先接解析这个主机名然后才搜索那个域名服务器。如果没有指定这个参数，dig会从`/etc/resolv.conf`拿取DNS服务器地址。如果没有可用的地址，dig会向本地主机发起搜索请求。

    `name`：被搜索的资源记录

    `type`：这个参数可以确定需要哪种类型的请求，对应了DNS的查询类型 - any,a,mx,sig,等，默认使用a记录

    其余参数

    `-4`：只使用IPv4

    `-6`：只使用IPv6

    `-b address[#port]`:设置发起搜索的IP的源地址，这个IP地址必须是本地主机的有效网卡之一，或者是`0.0.0.0`，`::`，后面可以接一个接口

    `-c class`：设置域名类，默认是IN

    `-f file`：批处理模式，dig从文件中读取查询的请求，文件中每一行都必须是单独的dig命令

    `-k` keyfile：使用TSIG对请求进行签名

    `-p port`：设定端口默认使用53

    `-q name`：指定要搜索的域名

    `-x addr`：反向查询，如果设定了这个选项，则不需要再指定`name,class,type`

2. 查询操作

    dig提供了很多查询操作，和谐操作可以影响结果的显示，有些操作是在请求头设置或者重设标志位，有些决定响应内容里面哪些部分会被打印，而其他则决定了超时和重试的策略

    `+[no]additional`：显示或者不显示响应内容中的额外部分

    `+[no]adfalg`:设置或者不设置请求头里面的AD（authentic data）位。这个标志位，可以用于判断是否所有的响应内容以及验证部分已经通过服务器的验证策略，如果AD=1，则表明所有记录已经通过验证以及响应内容不在OPT-OUT范围。AD=0则表明，响应内容中有些是不安全的或者是未验证的。默认是开启设置的。

    `+[no]all`：设置或者清楚所有的显示标记（显示内容）

    `+[no]answer`：显示或者不显示响应内容，默认是显示

    `+[no]authority`：显示或者不显示验证部分，默认是显示的

    `+[no]badcookie`：重新查找新的服务器的cookie，如果收到了`BADCOOKIE`的响应

    `+[no]besteffort`：显示消息里面不完整的内容，默认不显示

    `+[no]trace`:显示从根域名服务器到目标域名服务器的详细内容（而不是从本地设置的dns服务器开始查询）

    等

    结合参数达到目的如，只显示响应内容

    ```bash
    dig +noall +answer <name>
    ```

3. 其他操作

    如果将linux作为dns服务器，提供dns的服务一般都是`BIND`，可以通过dig来查找dns服务器的`BIND`版本信息

    - 如何查找到一个域名下所有的主机记录（FQDN）？

        前面所说的命令都只能针对与输入的域名进行查询，而不能对这个域名的主机进行查询

        思路是：通过查询dns服务器的`BIND`版本，如果是一个旧版本，一个有漏洞的版本，就可以利用这个漏洞，从DNS服务器中下载所有的记录信息

        `BIND`版本的信息是以`txt`的类型存在，而且是`chaos`类（CH）

        先查找目标域名的ns记录，找到域名服务器，再利用下面命令查找域名服务器的`BIND`版本

        ```bash
        dig +answer +noall txt chaos VERSION.BIND @<dns_server_name>
        ```

        实际上这个信息是可以屏蔽的

    - 如何鉴别DNS劫持？

        使用`trace`命令，从根域开始迭代查询，如果有劫持，其中一个dns服务器名称对应的IP地址是错误的

## DNS收集工具 - host

`host [option] {name} [server]`

host是一个简单的DNS查询工具。`name`是要查询的域名名字，可以是点分十进制的IPv4地址，也可以是冒号分割的IPv6地址，如果是这两种情况，默认执行DNS逆向查询。`server`是一个可选选项，既可以是dns服务器的名字，也可以IP地址，如果不指定，`host`默认使用`/etc/resolve.conf`里面的服务器

- 部分选项

    `-4`：只使用IPv4进行dns查询

    `-6`：只使用IPv6进行查询

    `-l`：List zone，对`name`执行区DNS域查询，他会返回ns，ptr以及a记录

    `-t type`：指定查询类型，如果是`IXFR`，可以在后面跟上`serial number`：`-t IXFR=2019043122`

    `-T,-U`:使用TCP或者UDP进行传输，默认使用UDP，如果查询类型是`AXRF`则默认使用TCP，如果查询类型`type`指定为`any`，则默认使用TCP

## DNS区域传输

一个比较吸引人的问题：如何获取一个域名的所有子域名？

如果一些DNS服务器没有配置好，利用DNS区域传输是手段之一

- 什么是DNS区域传输？[ref](https://en.wikipedia.org/wiki/DNS_zone_transfer)

    区域传输，DNS的AXFR查询类型，是一种DNS transaction。这是其中一种管理员会使用的机制，目的是将一个服务器的DNS数据库复制到这一组DNS服务器

    使用TCP并以CS形式进行传输，请求区域传输的可能是从服务器，也可能是备用服务器。被请求的服务器通常称为主服务器

    区域传输（zone transger），由一个前言（preamble）以及后面接着的实际要传递的数据组成。前言包括了SOA（Start of Authority）的资源记录，SOA的作用是用于查找一个区域的顶点（zone apex），也就是说，SOA就是区域的最顶部的节点。SOA的域里面有一个`serial number`字段，这个字段决定了下面实际要传输的数据是否需要进行传输：客户端拿它拥有的最新的那份`serial number`和SOA的资源记录进行比较，如果SOA的`serial number`比备份的大，客户端就认为zone里面的数据已经改变了，并且会请求实际要传输的数据。如果比较结果是相同，则客户端继续使用原来那个备份的数据（如果它有的话）

    实际数据传输，由客户端以一个特殊的查询类型`AXFR（value 252）`，使用TCP发送查询（opcode 0）开始。服务器返回一组响应消息（根据下面的抓包测试，这些记录都在一个dns响应包里面），包括了区域里所有的资源记录。第一个消息响应包包括了SOA记录，其他的数据依次发送。最后，服务器重复发送SOA记录，表示数据发送完毕。这里，dns的查询包和响应包，使用的是TCP协议，而不再是UDP协议。

- 常见的DNS区域传输命令

    ```bash
    dig @ns1.example.com exampl.com axfr
    ```

    ```bash
    host -T -l example.com nameserver
    ```

- dig命令测试

    参考`使用bind搭建dns服务器`的部署，修改`name.conf.options`文件，将`allow tranfer {none;}`字段注释(如果不注释得不到结果)

    然后在另一个主机上使用`dig @192.168.43.23 fcncdn.com axfr`,返回结果：

    ```bash
    ; <<>> DiG 9.11.5-P4-3-Debian <<>> @192.168.43.23 fcncdn.com axfr
    ; (1 server found)
    ;; global options: +cmd
    fcncdn.com.		604800	IN	SOA	fcncdn.com. root.localhost. 2019050122 604800 86400 2419200 86400
    fcncdn.com.		604800	IN	NS	localhost.
    aaa.fcncdn.com.	604800	IN	A	192.168.43.21
    bbb.fcncdn.com.	604800	IN	A	192.168.43.22
    ccc.fcncdn.com.	604800	IN	CNAME	bbb.fcncdn.com.
    fcncdn.com.fcncdn.com. 604800 IN	NS	192.168.43.23.fcncdn.com.
    fcncdn.com.		604800	IN	SOA	fcncdn.com. root.localhost. 2019050122 604800 86400 2419200 86400
    ;; Query time: 1 msec
    ;; SERVER: 192.168.43.23#53(192.168.43.23)
    ;; WHEN: 三 5月 01 00:59:57 CST 2019
    ;; XFR size: 7 records (messages 1, bytes 268)
    ```

- dig命令抓包分析

    对`dig @192.168.43.23 fcncdn.com axfr`进行抓包分析，一共抓到11个包

    1~3：本机和DNS服务器（192.168.43.23）的TCP三次握手
    4：本机向DNS服务器发送的DNS查询，查询类型为AXRF
    5：本机向DNS服务器发送的ACK包
    6：DNS服务器向本机返回的查询结果，在`Answer`字段包括了区域文件的所有记录（最后一条是SOA记录，表示数据传输结束）
    7：DNS服务器向本机发送的ACK包
    8~11：TCP的4次握手，结束连接

    上述第4个包和第6个包，dns协议的上层协议使用的是TCP协议，而不再是UDP协议

    同时，注意到，TCP使用的是53端口，也就是说，DNS协议同时占用了UDP和TCP的53端口，当需要进行查询的时候使用UDP，当需要区域传输的时候使用TCP

- host命令测试

    使用命令`host -l fcncdn.com 192.168.43.23`

    通过抓吧，返回结果是一样的

## DNS字典爆破

另一种获取子域名的方法是使用DNS字典爆破

kali 自带的工具:`fierce`,`dnsdict6`(kali2.0之后不是系统自带工具),`dnsenum`,`dnsmap`,`dnsrecon`

- fierce（好慢啊）

    ```bash
    fierce -dnsserver <dns server> -dns <domain name> -wordlist /usr/share/fierce/host.txt
    ```

    `/usr/share/fierce/host.txt`是fierce自带的字典

- dnsenum

    一般的命令格式：`dnsenum [options] <domain>`

    ```bash
    dnsenum -f /usr/share/dnsenum/dns.txt -dnsserver 8.8.8.8 sina.com -o sina.xml
    ```

    dnsenum的字典：`/usr/share/dnsenum/dns.txt`

    `-f <file>`：指定字典文件

    `-dnsserver`：指定dns服务器

    `-o <file>`：将结果输出为xml格式的文件

    输出格式比fierce好看

- dnsmap

    ```bash
    dns <domain> -w <dict>
    ```

- dnsrecon

    ```bash
    dnsrecon -d sina.com --lifetime 10 -t brt -D /usr/share/dnsrecon/namelist.txt
    ```

    部分选项：

    `-d`：域名

    `--lifetime`:超时时间

    `-t`：枚举类型 brt指的是爆破，std指的是标准的方式，较慢

    `-D`：指定字典文件

    这个可以指定超时时间，比较快

## DNS注册信息

注册DNS的时候，有可能会留下注册人的信息，这些信息可以用于后续社会工程学和物理攻击

Whois服务器信息：

```text
afrnic http://www.afrinic.net
apnic   http://www.apnic.net
arin    http://ws.arin.net
iana    http://www.iana.com
icann   http://www.icann.org
lacnic  http://www.lacnic.net
nro http://www.nro.net
ripe    http://www.ripe.net
internic    http://www.internic.net
```

命令行工具：whois

后面即可以跟域名，也可以跟IP地址

## 其他

1. 智能DNS

    当使用工具改变DNS服务器的时候，发现最后解析的IP地址不同，原因是，DNS服务器会将域名解析为最接近的服务器地址

2. 反垃圾邮件

    对来源IP地址进行反向域名解析（spf记录），如果查询结果不一致则说明，发送的邮件是伪造域名

## 操作

- 查询指定域名的域名服务器
- 查询指定域名的邮件服务器
- 查询指定域名的地址、主机信息
- 使用其他DNS服务器进行查询
- 反向查询域名信息
