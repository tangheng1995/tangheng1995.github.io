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

## CentOS安装go

### wget程序，go get需要依赖git，因此先安装git

```shell
yum -y install wget git
```

### 下载二进制文件

```shell
wget -c https://studygolang.com/dl/golang/go1.10.3.linux-amd64.tar.gz
```

### 解压到指定目录

```shell
tar -C /usr/local -xzf go1.10.3.linux-amd64.tar.gz
```

### 配置环境变量

```shell
vim /etc/profile
export GOROOT=/usr/local/go
export PATH=$PATH:$GOROOT/bin
```

### 设置 GOPATH 编译目录，需要编译的项目都应该放到该目录

```shell
mkdir -p /home/gopath
vim /etc/profile
export GOPATH=/home/gopath
```

### 加载环境变量

```shell
source /etc/profile
```

### 验证go版本

```shell
go version
```
