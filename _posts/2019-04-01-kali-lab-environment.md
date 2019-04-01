---
layout: post
title: "kali实验环境搭建"
data: 2019-04-01
tags: [security, penetratino test, kali linux]
excerpt_separator: <!--more-->
comments: false
share: false

---

准备实验环境作为渗透测试的练习，实验环境可能和实际环境有区别，但是，如果没有得到目标系统的授权，这就不是一次渗透测试。  
<!--more-->
自己搭建实验环境的一个好处是，可以看清楚整个攻击的过程，否则，可能会不清出攻击的原理，不清楚系统的结构，这次成功，下次可能就不成功，不清楚漏洞原理。

--- 

### 安装虚拟机

准备各种平台的虚拟机，方便后续针对指定平台的测试

- 下载虚拟机

    ```bash
    apt-get install virtualbox
    ```

    或者去官网下载对应的deb包   
    以上两种方法，在我这里都出现了这样的问题：  

    `kernel driver not installed (rc=-1908)`    

    后来不使用`apt-get`，先卸载了virtualbox，再用`aptitude`重新下载，重启一下，问题解决


- 虚拟网卡  

    在防火墙部分需要三个网卡，一个接互联网，一个接内网，一个接DMC，需要三个虚拟的网卡   

    三个网卡这样配置：  
    接互联网的网卡用桥接的方式，使用笔记本的wlan0   
    接内网的网卡，使用host-only的方式   
    接DMC的网卡，也使用host-only的方式

    VB的添加方法是，在全局设置，找到网络选项，直接添加即可，在5.2.0版本之后，这个host-only的添加被单独拿了出来，放在了‘工具’按钮的旁边



- 微软虚拟机

    微软产品[下载地址](https://www.microsoft.com/en-us/evalcenter/)     
    微软虚拟机的[下载地址](https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/)

    这里提供的虚拟机，难以作为系统漏洞发现的平台，没有已知漏洞，但是可以安装一些ｗｉｎ平台上软件，去测试这些软件的漏洞 。如果要对系统漏洞进行测试，要额外安装一个没有打安全补丁的系统版本，然后自己去检验测试漏洞。

    安装虚拟机的时候最好准备的是英文版的，使用msf的时候，里面有可以渗透攻击大量的已知的win平台的漏洞代码，而这些攻击代码是针对英文平台，部分攻击代码没有办法针对中文平台


- Linux虚拟机

    大量Linux虚拟机[下载地址](https://www.turnkeylinux.org/)（turnkeylinux），这里都是针对特定软件现成的虚拟机  
    也可以自己安装一个linux发行版，在上面安装各种服务，测试这些服务的漏洞


- LAMP搭建

    在turnkeylinux[下载](https://www.turnkeylinux.org/lamp)一个已经搭好的LAMP，载入镜像即可


- Metasploitables2


    [下载地址](https://metasploit.help.rapid7.com/docs/metasploitable-2)    
    这是一个装有很多测试用的软件的镜像系统，是很好的练习环境，里面包括了TWiki, phpMyAdmin, Mutillidae, DVWA, WebDAV

    默认用户：msfadmin 默认密码：msfadmin

    在使用Mutillidae进行渗透练习的时候，会遇到这样问题：所有的测试都不成功。这是因为mutillidae的配置有错，导致所有的测试不成功

    解决办法：修改Mutillidae的配置文件

    ```bash
    sudo nano /var/www/mutillidae/config.inc
    ```

    一开始的内容是这样的：

    ```php
    <?php
        $dbhost = 'localhost';
        $dbuser = 'root';
        $dbpass = '';
        $dbname = 'metasploit';
    ?>
    ```
    将`$dbname = 'metasploit'`改为`$dbname = 'owasp10'`

---

### 模拟真实网络

最简单的测试类型是，将攻击方和被攻击方放在同一个网络里面，这是理想的状态    
但是真实的网络环境会有防火墙、WAF、入侵检测等的情况，攻击方和被攻击方基本不可能在同一个网络里面     
当我们在理想环境攻击成功，可以转换到拥有防护机制的网络环境，看看攻击方法是否还有效

- 典型中小型企业网络拓扑    
    防火墙有三种口，假设为A，B，C   
    防火墙的端口A（一个或者多个） 接互联网  
    防火墙的端口B接内网交换机，交换机划分多个vlan和不同的网段到不同的部门   
    防火墙的端口C接DMC区的交换机，交换机接对外服务的服务器等    

从这个结构可以看出，其实最主要的是防火墙的配置，下面配置一个防火墙系统，模拟一个防火墙接三种网络的情况

- 模拟防火墙 - m0n0wall

    使用m0n0wall防火墙系统来模拟真实环境的防火墙
    [M0N0wall](https://sourceforge.net/projects/m0n0wall/)，一款体积很小的防火墙系统，甚至可以安装到树莓派。这个系统可以在企业使用，但是需要更大的单板电脑。

    设置网卡：
    1. 按照上述的网络拓扑，一共要设置三块网卡   
        对于连接互联网的网卡（Wan），使用桥接的方式，把笔记本的网卡当做连接互联网的网卡     
        对于连接内网的网卡（Lan），虚拟一个网卡，使用host-only的连接方式，ip地址为：10.1.1.1，关闭dhcp，让防火墙分配IP  
        对于连接DMC的网卡，也虚拟一个网卡，使用host-only的连接方式，ip地址为：10.1.2.1


    配置LAN口：
    
    1. 安装     
        在设置完网卡之后，直接导入虚拟机安装。安装的时候，第一次先安装到硬盘，安转完之后，退出光驱的iso文件 
    2. 设置端口，指定网卡对应的接口（Lan、Wan、Optional）   
        然后再次进入系统，选择网卡配置选项1(Interfaces: assign network ports)，依次进行vlan的分配以及lan端口、wan端口和optional 端口的设置，结束之后，系统会重启

        网卡和端口名是这样对应的：  
        em0 网卡1 桥接                  WLAN    
        em1 网卡2 host-only     LAN     
        em2 网卡3 host-only     OPT     

    3. 设置局域网网卡的IP地址（第二项 - set up LAN IP address）

        给接LAN网卡的端口设置IP地址为10.1.1.10，掩码24位，启动dhcp，dhcp的起始地址为10.1.1.20，dhcp的结束地址为10.1.1.100
    
    4. web登录密码的重设（选项三）

        在上一步设置完网卡的IP地址之后，就可以通过浏览器访问，这一步设置访问的web登录密码，默认的用户是admin，密码是mono

    配置结束，重启系统

    
    结束之后，测试配置

    再开启一个虚拟机，将这个虚拟机连接防火墙的lan口相同的网卡连接（当做办公网的一台主机），开启的是之前的ubuntu，通过`ifconfig`发现，分配得到的IP地址是`10.1.1.100`。登录em1的网卡IP地址`10.1.1.10`，验证防火墙的WEB管理功能。可以登上，输入用户名`admin`以及密码`mono`，成功进入防火墙的管理页面。

    至此，完成了局域网LAN的配置，我们可以自定局域网的网段、网卡IP地址以及dhcp分配范围

    - 配置WAN
        由于WAN口使用的是桥接笔记本的网卡，直接使用dhcp分配就可以

    - 设置防火墙规则

        选择菜单里面的 Firewall-> Reuls,选择LAN选项卡，可以发现，已经有几条写好的规则，这几条的规则表示允许LAN的数据包以IPv4和IPv6协议的形式去到任意的目的地

        新增一条规则：

            Action: pass
            Interface: WAN
            protocol: any
            Source.Type: LAN subnet
            Destination.Type: any

        这条规则表示的意思是，允许LAN数据包以任意协议发送到任意目的地   
        然后点击 ApplyChanges，规则生效。

- 双层异构防火墙

    有些网络环境比较大的公司会设置两道防火墙，一层使用硬件防火墙，做快速简单的IP转发和过滤，另一层防火墙是过滤防火墙，可以对应用层进行过滤以及防病毒的过滤、反垃圾邮件的过滤、WEB content的过滤。

    结构是这样的：  
    第一个防火墙部署m0n0wall，第二个防火墙部署[pfsense](pfsense.org)    
    第一道防火墙有两个端口，第一个端口连接互联网，第二个端口连接DMS的交换机     
    第二道防火墙有两个端口，第一个端口连接CMS的交换机，第二个端口连接LAN局域网（内网），当然这一层也可以提前    

    上面所说的ubuntu、metaspolitable2、LAPM等可以部署在DMC区，win系统可以部署在LAN内网里面，由此构成了一个较为复杂的网络环境

- pfsense部署

    pfsense是基于m0n0wall开发出来的防火墙，在它的基础上增加了一些功能，集成了大量的插件如WAF、IPS、防病毒等     
    [下载地址](https://www.pfsense.org/download/)

    1. 安装
        
        没有特别的问题

    2. 设置端口(选项1 - Assign INterfaces)

        同样是分配Vlan，指定WAN端口对应的网卡，指定LAN对应的网卡，指定OPT对应的网卡，这里的设置和m0n0wall的一样

        em0 网卡1 桥接                  WLAN    
        em1 网卡2 host-only     LAN     
        em2 网卡3 host-only     OPT 

    3. 设置端口IP地址（选项2 - Set interfaces IP address）

        同样先设置LAN口的IP地址     
        IP地址： 10.1.1.10  
        掩码位：24  
        dhcp起始地址： 10.1.1.20    
        dhcp结束地址： 10.1.1.100

    4. web登录密码重设（选项3 - Reset webConfigurator password）

        默认密码设置为 pfsense

        配置完，重启一下系统，打开另一个ubuntu的虚拟机进行检测  
        打开终端，`ifconfig`发现配置的IP是`10.1.1.100`  
        打开浏览器访问网卡IP地址，出现登录页面，使用默认的用户名`admin`，和默认密码`pfsense`登录成功。（啊，这个界面比m0n0wall设计地好看）登录成功之后友谊setup引导，跳过，去设置防火墙规则。

        防火墙这里和m0n0wall没有太大的区别，基本一样    
        而，WAN口使用的是桥接，默认使用dhcp分配就行

    至此，pfsense的基本部署完成


---

### 补充

1. 什么是Vlan
    
    ref： [http://network.51cto.com/art/201409/450885_all.htm](http://network.51cto.com/art/201409/450885_all.htm)



