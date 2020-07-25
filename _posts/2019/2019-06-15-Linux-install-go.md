---
layout: post
title: "Linux 安装 Golang"
date: "2019-06-15 12:00:00"
category: golang
tags: golang
author: Archer
---
* content
{:toc}

## 介绍

Linux 安装 Golang。




## Linux安装go

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

### 开启Go Module

```shell
# -w 设置env变量，-u还原
go env -w GO111MODULE=on

# 设置代理
go env -w GOPROXY=https://goproxy.io,direct
```

deme测试：

```text
├─gingo
│  │  main.go
```

```shell
go mod init com.example/gingo
```

运行完生成 `go.mod` 文件，再根据项目需要下载包，`go.mod`会统一管理包版本
