---
layout: post
title: 'Hello docker'
subtitle: '安装docker'
date: 2019-06-15
author: Archer
categories: 容器
cover: ''
tags: docker centos
---

### CentOS安装docker

1. 使用 `root` 权限登录 Centos。确保 yum 包更新到最新

   `sudo yum update`

2. 卸载旧版本(如果安装过旧版本的话)

   `sudo yum remove docker  docker-common docker-selinux docker-engine`

3. 安装需要的软件包， yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖

   `sudo yum install -y yum-utils device-mapper-persistent-data lvm2`

4. 设置yum源

   `sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo`

5. 设置换国内源

   `sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo`

6. 更换镜像源，修改或创建vim /etc/docker/daemon.json文件：vim /etc/docker/daemon.json
方法一:

```text
{
    "registry-mirrors": [
        "https://kfwkfulq.mirror.aliyuncs.com",
        "https://2lqq34jg.mirror.aliyuncs.com",
        "https://pee6w651.mirror.aliyuncs.com",
        "https://registry.docker-cn.com",
        "http://hub-mirror.c.163.com"
    ],
    "dns": ["8.8.8.8","8.8.4.4"]
}
```

方法二:
修改或创建vim /etc/sysconfig/docker

```text
OPTIONS='--selinux-enabled \
--log-driver=journald \
--signature-verification=false \
--registry-mirror=https://kfwkfulq.mirror.aliyuncs.com'
if [ -z "${DOCKER_CERT_PATH}" ]; then
    DOCKER_CERT_PATH=/etc/docker
fi
```

```text
sudo systemctl restart docker
```

7. 安装docker

   ```sudo yum install docker-ce```

8. 启动并加入开机启动

   ```systemctl start docker

        systemctl enable docker```

9. 验证安装是否成功(有client和service两部分表示docker安装启动都成功了)

   ```docker version```
