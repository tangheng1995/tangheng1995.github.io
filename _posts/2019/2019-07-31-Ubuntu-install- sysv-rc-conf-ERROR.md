---
layout: post
title: "Linux 安装 sysv-rc-conf 报错"
date: "2019-07-31 12:00:00"
category: linux
tags: linux
author: Archer
---
* content
{:toc}

## 介绍

Linux 安装 sysv-rc-conf 报错。




## Ubuntu安装sysv-rc-conf无法定位软件包

安装sysv-rc-conf时提示

```text
sudo apt-get install sysv-rc-conf
正在读取软件包列表... 完成
正在分析软件包的依赖关系树
正在读取状态信息... 完成
E: 无法定位软件包 sysv-rc-conf
```

解决方案，添加镜像源：

```shell
sudo vim /etc/apt/sources.list

deb http://archive.ubuntu.com/ubuntu/ trusty main universe restricted multiverse

sudo apt update

sudo apt install sysv-rc-conf
```
