---
layout: post
title: "kali命令行连接wifi"
date: 2019-04-01
tags: [kali linux]
excerpt_separator: <!--more-->
comments: false
share: false
---

旧电脑最后一次使用kali，应该是硬件哪里不行了，只能进去recovery模式，想apt卸载、更新一些包，但是kali默认是不开启网络服务的，以下是当时在命令行连接网络的方法：
<!--more-->

检查网卡是否开启：

```bash
iwconfig
```

打开网卡

```bash
ifconfig wlan0 up
```

接下来要连接wifi

1. 搜索附近的wifi

    ``` bash
    iw wlan0 scan
    ```
    找到要连接的ssid，假设找到的是"Tenda"，密码是"password123"

2. 生成一段wpa验证用的配置信息

    即使找到了想要的ssid，也不能直接输入明文密码，要通过加密转换。通过`wpa_passphrase`工具来转换密码，并将转换后的信息输出到一个任意命名的配置文件，假设是`ritesh.conf`

    ```bash
    wpa_passphrase "Tenda" >> ritesh.conf password123
    ```
    ```bash
    cat ritesh.conf

    network={
        ssid="Tenda"
        #psk="password123"
        psk=a59cafd6e4d197a51f683e137bcfa406dc29115a967e5f668bf96630eb712861
    }
    ```
3. 连接到指定ssid的wifi

    ```bash
    wap_supplicant -D wext -B -i wlan0 -c ritesh.conf
    ```

    `-D`表示要使用的驱动，`wext`表示linux无线扩展

    `-B`表示在后台执行这个命令

    `-i`指定端口，指定的是`wlan0`

    `-c`表示包含验证信息的配置文件


4. 检查wifi的连接

    ```bash
    iw wlan0 link
    ```
5. 配置IP

    这时候还不能上网，使用`ifconfig wlan0`可以查看，这时候还没有配置IP

    获取IP地址
    ```bash
    dhclient wlan0
    ```
    最后就可以连接，可以`ping`测试

但是最后还是进不去系统QAQ，而且因为使用的中文，命令行一大串的乱码