---
layout: post
title: "在firefox浏览器启动vlc插件"
data: 2019-05-01
tags: [security, penetration test, kali linux]
excerpt: "vlc浏览器插件"
comments: false
share: false
---

使用的是ubuntu系统

[vlc官网](https://www.videolan.org/vlc/download-debian.html)说明，要安装两个软件：

```bash
vlc browser-plugin-vlc
```

第一个是vlc本省，第二个是vlc的浏览器插件

安装完之后，firefox插件选项里面没有发现对应的vlc插件，原因是高版本firefox（52）已经弃用了NPAPI 插件，而vlc浏览器插件依赖这个NPAPI 插件，所以不能使用。

解决方法是使用低版本的firefox，具体下载地址和安装，参考环境配置，这里安装的是`52.9.0esr`,打开之后，在插件选项里面可以看到vlc插件