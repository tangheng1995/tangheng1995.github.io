---
layout: post
title: 'Hello docker redis'
subtitle: '安装docker redis'
date: 2019-07-05
author: Archer
categories: 数据库
cover: ''
tags: redis centos docker
---

### CentOS安装Redis docker

1. 切换root用户

```text
su root
```

2. 启动docker

```text
systemctl start docker
```

3. 拉取mysql 5.7镜像

```text
docker pull redis
```

4. 创建并启动容器

```text
    -p: 映射本地端口6379

    --restart-always: docker服务启动时，自动启动容器，并且当容器停止时，尝试重启容器。

    redis-server：redis服务器

    --appendonly yes：开启持久化

    -v : 挂载本地卷
```

```text
docker run --name redis -p 6379:6379 --restart=always -v "$PWD/data":/data -d redis redis-server --appendonly yes
```
