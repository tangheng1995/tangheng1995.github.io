---
layout: post
title: "Docker buildx 构建多平台镜像"
date: "2020-10-28 12:00:00"
category: docker
tags: docker
author: Archer
---
* content
{:toc}

## 介绍

在 Docker 19.03+ 版本中可以使用 $ docker buildx build 命令使用 BuildKit 构建镜像。该命令支持 --platform 参数可以同时构建支持多种系统架构的 Docker 镜像，大大简化了构建步骤。




## 开启 Docker CLI 实验特性

编辑 ~/.docker/config.json 文件，新增如下条目

```json
{
  "experimental": "enabled"
}
```

或者通过设置环境变量的方式：

```shell
export DOCKER_CLI_EXPERIMENTAL=enabled
```

刷新字体

## 开启 Dockerd 的实验特性

编辑 /etc/docker/daemon.json，新增如下条目

```json
{
  "experimental": true
}
```

检查docker实验特性是否开启：

```shell
docker version -f '{{.Server.Experimental}}'
```

## 重新加载

```shell
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## 新建 builder 实例

Docker for Linux 不支持构建 arm 架构镜像，我们可以运行一个新的容器让其支持该特性，Docker 桌面版无需进行此项设置。

```shell
docker run --rm --privileged tonistiigi/binfmt:latest --install all
```

```shell
# 适用于国内环境
$ docker buildx create --use --name=mybuilder --driver docker-container --driver-opt image=dockerpracticesig/buildkit:master

# 适用于腾讯云环境(腾讯云主机、coding.net 持续集成)
$ docker buildx create --use --name=mybuilder --driver docker-container --driver-opt image=dockerpracticesig/buildkit:master-tencent

# $ docker buildx create --name mybuilder --driver docker-container

$ docker buildx use mybuilder
```

```shell
docker buildx inspect mybuilder --bootstrap
```

可观察到支持全平台：

```shell
Name:   mybuilder
Driver: docker-container

Nodes:
Name:      mybuilder0
Endpoint:  unix:///var/run/docker.sock
Status:    running
Platforms: linux/amd64, linux/arm64, linux/riscv64, linux/ppc64le, linux/s390x, linux/386, linux/arm/v7, linux/arm/v6
```

## 构建镜像

* 直接构建全平台镜像

```shell
$ docker buildx build --platform linux/arm,linux/arm64,linux/amd64 -t myusername/hello . --push

# 查看镜像信息
$ docker buildx imagetools inspect myusername/hello
```

* 构建某单一平台镜像

```shell
$ docker buildx build --platform linux/arm64 -t myusername/hello:arm64 . --push

# 查看镜像信息
$ docker buildx imagetools inspect myusername/hello:arm64
```

## 引用

* [1] [使用 buildx 构建多种系统架构支持的 Docker 镜像](https://yeasy.gitbook.io/docker_practice/buildx/multi-arch-images)
* [2] [开启实验特性](https://yeasy.gitbook.io/docker_practice/install/experimental)
* [3] [Docker Buildx](https://docs.docker.com/buildx/working-with-buildx/)
