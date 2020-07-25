---
layout: post
title: "Ubuntu apt-get update与sudo apt-get upgrade区别"
date: "2019-08-04 12:00:00"
category: linux
tags: linux
author: Archer
---
* content
{:toc}

## 介绍

Ubuntu apt-get update与sudo apt-get upgrade区别。




## Ubuntu里apt-get update与sudo apt-get upgrade区别

### apt-get update

update是更新软件列表

UBUNTU下存在源列表，源列表里面都是一些网址信息，这每一条网址就是一个源，这个地址指向的数据标识着这台源服务器上有哪些软件可以安装使用。

```shell
sudo vim /etc/apt/sources.list
```

这个文件里加入或者注释（加#）掉一些源后，保存。这时候，我们的源列表里指向的软件就会增加或减少一部分。

获得最近的软件包的列表:(列表中包含一些包的信息，比如这个包是否更新过)

```shell
sudo apt-get update
```

这个命令，会访问源列表里的每个网址，并读取软件列表，然后保存在本地电脑。软件包管理器里看到的软件列表，都是通过update命令更新的。

### apt-get upgrade

upgrade是更新软件

```shell
sudo apt-get upgrade
```

这个命令，会把本地已安装的软件，与刚下载的软件列表里对应软件进行对比，如果发现已安装的软件版本太低，就会提示你更新。如果你的软件都是最新版本，会提示：

```text
升级了 0 个软件包，新安装了 0 个软件包，要卸载 0 个软件包，有 0 个软件包未被升级。
```

### 结论

apt-get update 指令会同步使用者端和APT 伺服器的RPM 索引清单（package list），APT 伺服器的RPM 索引清单置于base 资料夹内，使用者端电脑
取得base 资料夹内的bz2 RPM 索引清单压缩档后，会将其解压置放于/var/state/apt/lists/，而使用者使用apt-get install 
或apt-get dist-upgrade 指令的时候，就会将这个资料夹内的资料和使用者端电脑内的RPM 资料库比对，如此一来就可以知道那些RPM 已安装、未安装、或是
可以升级的。
