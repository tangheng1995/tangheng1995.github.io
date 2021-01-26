---
layout: post
title: "Linux kali 包管理工具 snap 使用"
date: "2021-01-26 12:00:00"
category: linux
tags: linux,kali
author: Archer
---
* content
{:toc}

## 介绍

Kali 基于 Debian10，snap 包管理工具




## 更新kali源

修改 /etc/apt/sources.list , 将相关 url 改成阿里云的源。

```text
#deb https://mirrors.aliyun.com/kali kali-rolling main non-free contrib
#deb-src https://mirrors.aliyun.com/kali kali-rolling main non-free contrib
```

> [kali阿里源](https://developer.aliyun.com/mirror/kali?spm=a2c6h.13651102.0.0.3e221b11t9JCcq)

## 安装snap

```shell
$ sudo apt update
$ sudo apt install snapd
$ sudo snap install core
```

若snap安装异常，可卸载重新安装：

```shell
$ sudo apt autoremove --purge snapd

$ sudo apt install snapd
```

查看服务是否启动：

```shell
➜  ~ systemctl status snapd.service
● snapd.service - Snap Daemon
     Loaded: loaded (/lib/systemd/system/snapd.service; disabled; vendor preset: disabled)
     Active: inactive (dead)
TriggeredBy: ● snapd.socket
```

```shell
systemctl restart snapd.service
```

```shell
➜  ~ systemctl status snapd.service  
● snapd.service - Snap Daemon
     Loaded: loaded (/lib/systemd/system/snapd.service; disabled; vendor preset: disabled)
     Active: active (running) since Tue 2021-01-26 19:08:41 CST; 4s ago
TriggeredBy: ● snapd.socket
   Main PID: 3598 (snapd)
      Tasks: 10 (limit: 9449)
     Memory: 23.7M
     CGroup: /system.slice/snapd.service
             └─3598 /usr/lib/snapd/snapd
```

查看snap版本：

```shell
➜  ~ snap version                  
snap    2.45.2-1
snapd   2.45.2-1
series  16
kali    2020.3
kernel  5.7.0-kali1-amd64
```

安装postman测试：

```text
➜  ~ snap install postman
2021-01-26T19:10:42+08:00 INFO Waiting for automatic snapd restart...
Warning: /snap/bin was not found in your $PATH. If you've not restarted your session since you
         installed snapd, try doing that. Please see https://forum.snapcraft.io/t/9469 for more
         details.

postman 7.36.1 from Postman, Inc. (postman-inc✓) installed
```

## 快捷启动

将snap安装的postman添加启动器快捷启动，snap应用桌面图标目录 `/var/lib/snapd/desktop/applications`

```shell
# 启动器
xdg-desktop-menu install --novendor postman_postman.desktop
```

## 更多命令

列出计算机上所有snap安装情况：

```shell
sudo snap list
```

在应用商店中查找snap：

```shell
sudo snap find <软件包名>
```

安装Snap软件：

```shell
sudo snap install <snap软件包名>
```

更新Snap软件：

```shell
sudo snap refresh <snap软件包名>
```

更新所有的snap软件包：

```shell
sudo snap refresh all
```

要将Snap还原到以前安装的版本：

```shell
sudo snap revert <snap软件包名>
```

卸载snap软件：

```shell
sudo snap remove <snap软件包名>
```

## 引用

* [1] [Post http://localhost/v2/snaps/hello-world: 102 dial unix /run/snapd.socket: connect: connection refused](https://forum.snapcraft.io/t/snap-d-error-cannot-communicate-with-server-connection-refused/6093)
