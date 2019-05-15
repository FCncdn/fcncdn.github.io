---
layout: post
title: "获取一个域名下的所有子域名？"
data: 2019-04-30
tags: [security， network]
excerpt: "DNS笔记"
comments: false
share: false
---

## 解决方案

- DNS区域查询

    对于没有禁止这个选项的DNS服务器，是一个不错的选择
    （但是基本都是不行的。。）

- 使用wolframalpha

    [wolframalpha](https://www.wolframalpha.com/)

    直接输入要查找的域名，在返回结果中找到`subdomains`选项，点击后会增加显示一个显示卡`Subdomains`，在这显示卡点击`more`可以获取返回子域名

    对于一个域名，如果可以找到结果，不是完整的结果,出来的结果太少

    对于一个域名，不一定可以返回`subdomains`结果

- 子域名爆破

    使用字典，对当前层级的域名进行暴力破解

    这有点DOS的味道，除了kali自带的工具之外，在github可以搜到一大堆

- 搜索引擎查询
