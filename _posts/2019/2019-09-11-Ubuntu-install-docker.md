---
layout: post
title: "Linux 安装 docker - Ubuntu"
date: "2019-09-11 12:00:00"
category: linux
tags: linux
author: Archer
---
* content
{:toc}

## 介绍

Ubuntu 安装 docker 并配置docker管理工具Portainer。




## Ubuntu 安装 docker

### 安装Docker

Ubuntu版本为18.04

更新包

```shell
sudo apt update
```

安装一些必备包，让apt可以使用https

```shell
sudo apt install apt-transport-https ca-certificates curl software-properties-common
```

添加GPG key 官方仓库公钥

```shell
# 国内源
curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add -

# 官方源
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

添加Docker仓库到apt源：

```shell
# 国内源
sudo add-apt-repository "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu $(lsb_release -cs) stable"

# 官方源
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
```

从docker最新仓库上更新包：

```shell
sudo apt update
```

确认安装Docker是从最新的docker仓库而不是默认Ubuntu仓库：

```shell
apt-cache policy docker-ce
```

可以看到终端输出结果：

```text
docker-ce:
  Installed: (none)
  Candidate: 18.03.1~ce~3-0~ubuntu
  Version table:
     18.03.1~ce~3-0~ubuntu 500
        500 https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
```

安装docker：

```shell
sudo apt install docker-ce
```

设置开机自启：

```shell
sudo systemctl status docker
```

可以看到终端输出为：

```text
Output
● docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2019-09-11 13:08:39 UTC; 2min 55s ago
     Docs: https://docs.docker.com
 Main PID: 10096 (dockerd)
    Tasks: 16
   CGroup: /system.slice/docker.service
           ├─10096 /usr/bin/dockerd -H fd://
           └─10113 docker-containerd --config /var/run/docker/containerd/containerd.toml
```

### 脚本自动安装

Docker 官方为了简化安装流程，提供了一套便捷的安装脚本，Ubuntu 系统上可以使用这套脚本安装：

```shell
curl -fsSL get.docker.com -o get-docker.sh

sudo sh get-docker.sh --mirror Aliyun
```

### 镜像加速

修改 /etc/docker/daemon.json 文件

```json
{
  "registry-mirrors": [
    "https://dockerhub.azk8s.cn",
    "https://reg-mirror.qiniu.com",
    "https://docker.mirrors.ustc.edu.cn"
  ]
}
```

```shell
sudo systemctl daemon-reload
sudo systemctl restart docker
```

检查镜像是否生效：

```shell
docker info
```

从结果中看到了如下内容，说明配置成功

```text
Registry Mirrors:
  https://dockerhub.azk8s.cn/
  https://reg-mirror.qiniu.com/
```

### 普通用户运行

```shell
sudo groupadd docker
sudo usermod -aG docker ${USER}
sudo systemctl restart docker
```

### 安装Portainer

创建卷：

```shell
docker volume create portainer_data
```

运行Portainer容器：

```shell
docker run -d -p 8000:8000 -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer
```

访问端口9000，进入Portainer配置页面

### 删除none标签镜像

```shell
docker rmi $(docker images | grep "none" | awk '{print $3}')

docker rmi $(docker images -f "dangling=true" -q)
```

### 引用

- [1] [How To Install and Use Docker on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04)
- [2] [How simple is it to deploy Portainer?](https://www.portainer.io/installation/)
