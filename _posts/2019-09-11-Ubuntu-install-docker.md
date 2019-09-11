---
layout: post
title: Ubuntu 安装 docker
subtitle: Ubuntu 安装 docker 并配置docker管理工具Portainer
date:       2019-09-11 12:00:00
author:     "Archer"
categories: docker
tags:
    - ubuntu
    - docker
    - protainer
---

### Ubuntu 安装 docker

#### 一、安装Docker
Ubuntu版本为18.04

更新包
```text
sudo apt update
```

安装一些必备包，让apt可以使用https
```text
sudo apt install apt-transport-https ca-certificates curl software-properties-common
```

添加GPG key 官方仓库公钥
```text
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

添加Docker仓库到apt源：
```text
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
```

从docker最新仓库上更新包：
```text
sudo apt update
```

确认安装Docker是从最新的docker仓库而不是默认Ubuntu仓库：
```text
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
```text
sudo apt install docker-ce
```

设置开机自启：
```text
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

#### 二、安装Portainer

创建卷：
```text
docker volume create portainer_data
```

运行Portainer容器：
```text
docker run -d -p 8000:8000 -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer
```

访问端口9000，进入Portainer配置页面

#### 三、扩展
Ubuntu18.04安装docker参考文档：[How To Install and Use Docker on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04)
Portainer安装参考文档：[How simple is it to deploy Portainer?](https://www.portainer.io/installation/