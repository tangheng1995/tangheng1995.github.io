---
layout: post
title: 'Hello go'
subtitle: '安装go'
date: 2019-06-15
author: Archer
categories: 语言
cover: ''
tags: go centos
---

### CentOS安装go

1. wget程序，go get需要依赖git，因此先安装git

   `yum -y install wget git`

2. 下载二进制文件

   `wget -c https://studygolang.com/dl/golang/go1.10.3.linux-amd64.tar.gz`

3. 解压到指定目录

   `tar -C /usr/local -xzf go1.10.3.linux-amd64.tar.gz`

4. 配置环境变量

   `vim /etc/profile`

   `export GOROOT=/usr/local/go`

   `export PATH=$PATH:$GOROOT/bin`

5. 设置 GOPATH 编译目录，需要编译的项目都应该放到该目录

   `mkdir -p /home/gopath`

   `vim /etc/profile`

   `export GOPATH=/home/gopath`

6. 加载环境变量

   `source /etc/profile`

7. 验证go版本

   `go version`

