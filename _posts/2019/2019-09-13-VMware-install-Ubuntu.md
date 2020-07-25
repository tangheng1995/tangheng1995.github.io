---
layout: post
title: "VMware 安装 Ubuntu"
date: "2019-09-13 12:00:00"
category: linux
tags: linux
author: Archer
---
* content
{:toc}

## 介绍

VMware安装Ubuntu，多台虚拟机网络互通。




## VMware 安装 Ubuntu

### 下载ubuntu server

下载地址 [Ubuntu server](https://ubuntu.com/download/server)

### 安装ubuntu虚拟机

配置虚拟机网络时选择 `VMnet8(NAT模式)`
![VMnet8](/assets/images/2019/2019-09-13-vm-net.png)

配置虚拟机mirror连接时设置阿里云：

```text
http://mirrors.aliyun.com/ubuntu/
```

![aliyun](/assets/images/2019/2019-09-13-ubuntu-mirror.png)

mirror服务器列表为：

```shell
可将 http://cn.archive.ubuntu.com/ubuntu/ 替换为下列任意服务器：

# Ubuntu 官方（欧洲，国内较慢，无同步延迟）
http://archive.ubuntu.com/ubuntu/

# Ubuntu 官方中国（目前是阿里云）
http://cn.archive.ubuntu.com/ubuntu/

# 网易（广东广州电信/联通千兆双线接入）
http://mirrors.163.com/ubuntu/

# 搜狐（山东联通千兆接入）
http://mirrors.sohu.com/ubuntu/

# 阿里云（北京万网/浙江杭州阿里云服务器双线接入）
http://mirrors.aliyun.com/ubuntu/

# 中国开源软件中心
http://mirrors.oss.org.cn/ubuntu/

# 首都在线科技
http://mirrors.yun-idc.com/ubuntu/
```

安装OpenSSH

![OpenSSH](/assets/images/2019/2019-09-13-install-openssh.png)

### 验证ssh

查看多台虚拟机内网ip，ssh命令，登录正常，OK，搭建集群的前置条件完成
