---
layout: post
title: "被动信息收集 - 搜索引擎"
data: 2019-05-01
tags: [security, penetration test, kali linux]
excerpt: "被动信息收集 - 搜索引擎"
comments: false
share: false
---

## 搜索引擎

通过搜索引擎，可以（可能）搜集目标企业的：

- 公司新闻动态
- 重要雇员信息
- 机密文档、网络拓扑
- 用户名密码
- 目标系统软硬件技术架构
- 目标系统历史性数据（搜索引擎的缓存）

搜索引擎，实际上就是一个全天候的爬虫机器+数据库，使用提供的筛选命令组合，可以从中找到目标的信息

## SHODAN

shodan是一个搜索引擎，只爬取服务器信息，不爬取页面信息

原理是，不断使用爬虫，获取服务器的banner信息

类似的搜索引擎还有钟馗之眼

- 常用过滤命令

    `net:<ip address>`:查询指定ip地址的服务器信息

    `net:<ip address>/<mask>`:查看一个网段的服务器信息

    `country:<country code>`:只显示输入指定国家的服务器信息

    `city:<city name>`:只显示位于指定城市的服务器信息

    `port:<port num>`:只显示指定端口的设备

    `os:"<os name>"`:筛选操作系统

    也可以直接搜索banner信息，如`hostname:`

- 一些搜索命令

    `200 OK cisco country:JP`,搜索日本的cisco设备

    `user:admin pass:password`

    `linux upnp avtech`

在首页的`explore`选项有提供一些搜索命令

- API使用

    在账户页面提供了api的key

- firefox的shodan插件

    搜索shodan的add-ons，直接安装

    每当流浪一个网站的时候，插件会自动从shodan查询这个页面，点击就可以看到详细信息

## google搜索引擎过滤语法

- 搜索结果包含/不包含指定关键字

    `+ <arg>`:包含arg

    `- <arg>`:不包含arg

- 完整语义搜索

    `"this is a query statement"`:结果只会显示和双引号内完全匹配的结果

- 搜索页面标题包含指定字符

    `intitle:<arg>`:在html中是`<title>`标签里面的内容

- 搜索页面内容包含指定字符

    `intext:<arg>`

- 搜索指定网站

    `site:<domain>`

    site也可以指定国家范围

- 搜索url中含有指定字符：

    `inurl:<arg>`

- 搜索指定文件类型：

    `filetype:<file_type>`

- inurl的使用实例

    `inurl:"/level/15/exec/-/show/"`

    `intitile:"netbotz application" "ok"`:搜索netbotz 的网络摄像头

    `inurl:/admin/login.php`:搜索后台登录页面

    `inurl:qq.txt`:那些盗用qq号的服务器遗留的信息

    `filetype:xls "username | password"`

    `inurl:Service.pwd`:搜索FrontPage的漏洞

    `https://www.exploit-db.com/google-hacking-database`：google的搜索命令

## 其他搜索引擎

- 世界第四大搜索引擎 - 俄罗斯

    www.yandex.com

## 命令行搜索

主要目的是完成大量并发的任务，好用

- 通过搜索引擎，查询指定的邮箱地址、主机信息

    `theharvester`

    部分参数：

    `-d`:domain

    `-b`:data source指定搜索引擎，具体看-h

    `-l`：指定最大并发线程，默认是50，如果数目太大，搜索引擎有可能会阻止访问

    `-f`：将结果输出到指定文件

    对于不同的域，使用不同的搜索引擎可以得到不同的搜索结果

- 命令行使用google对指定域名的服务器进行文件下载

    `metagoofil`

    部分参数：

    `-d`：domian

    `-t`：下载的文件类型

    `-l`：搜索结果数量限制（默认200）

    `-n`：下载文件数量限制

    `-o`：指定存储下载文件的地址

    `-f`：将结果输入到指定文件

## Maltego

一个综合性的图形界面信息收集软件

可以找到前面所说的信息，IP，ns等

## 其他信息搜索途径

社交网络、工商注册、新闻组、论坛、招聘网站

查询网站的历史版本：

`www.archive.org/web/web.php`，这个网站可以搜索到指定网站的历史的快照

## 生成个人专属的密码字典

`CUPP - Common User Password Profiler`

获取：

`git clone https://github.com/Mebus/cupp.git`

`python cup.py -i`

## Metadata信息收集

- Exif图片信息

    使用手机或者相机拍摄的时候，可以设置是否开启信息收集功能

    这个信息可以给照片添加当前的经纬度，以及当前设备信息

- 命令行收集图片的exif信息

    `exiftool <image>`

    里面通过筛选`GPS`字段，可以看到相关信息，设备信息可以通过`Make`字段查看，里面还可以看出一些设备相关的信息
