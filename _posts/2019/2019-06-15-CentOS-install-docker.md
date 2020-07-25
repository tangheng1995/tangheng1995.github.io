---
layout: post
title: "Linux 安装 Docker"
date: "2019-06-15 12:00:00"
category: docker
tags: docker
author: Archer
---
* content
{:toc}

## 介绍

Linux 安装 Docker。




## CentOS安装docker

### 安装

```shell
# yum 包更新到最新
sudo yum update

# 卸载旧版本(如果安装过旧版本的话)
sudo yum remove docker  docker-common docker-selinux docker-engine

# 安装需要的软件包， yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖
sudo yum install -y yum-utils device-mapper-persistent-data lvm2

# 设置yum源
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# 设置换国内源
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 安装docker
sudo yum install docker-ce

# 设置开机自启
systemctl start docker
systemctl enable docker

# 查看版本
docker version
```

### 更换镜像源，修改或创建vim /etc/docker/daemon.json文件

- 方法一:

```json
{
  "registry-mirrors": [
    "https://dockerhub.azk8s.cn",
    "https://reg-mirror.qiniu.com",
    "https://docker.mirrors.ustc.edu.cn"
  ]
}
```

- 方法二:

修改或创建vim /etc/sysconfig/docker

```shell
OPTIONS='--selinux-enabled \
--log-driver=journald \
--signature-verification=false \
--registry-mirror=https://kfwkfulq.mirror.aliyuncs.com'
if [ -z "${DOCKER_CERT_PATH}" ]; then
    DOCKER_CERT_PATH=/etc/docker
fi
```

```shell
sudo systemctl restart docker
```
