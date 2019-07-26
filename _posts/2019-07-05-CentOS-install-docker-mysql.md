---
layout: post
title: 'Hello docker mysql'
subtitle: '安装docker mysql'
date: 2019-07-05
author: Archer
categories: 数据库
cover: ''
tags: mysql centos docker
---

### CentOS安装Mysql docker


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
docker pull mysql:5.7
```

4. 创建并启动容器

```text
    -p: 映射本地端口3306

    --restart-always: docker服务启动时，自动启动容器，并且当容器停止时，尝试重启容器。

    --restart具体参数值详细信息：
    no -  容器退出时，不重启容器；
    on-failure - 只有在非0状态退出时才从新启动容器；
    always - 无论退出状态是如何，都重启容器；

    MYSQL_ROOT_PASSWORD：设置root密码为root

    设置默认数据库编码为utf8mb4,默认排序规则为utf8mb4_unicode_ci

    -v : 挂载本地卷
```

```text
docker run --name mysql -p 3306:3306 --restart=always -e MYSQL_ROOT_PASSWORD=qaqa123456 -v "$PWD/data":/var/lib/mysql -d mysql:5.7 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```

5. 进入docker的mysql容器

```text
docker exec -it mysql /bin/bash
```

6. 登录数据库（此处的密码为参数-e MYSQL_ROOT_PASSWORD=root对应的值，此处密码为root）

```text
mysql -u root -p my_password
```

7. 修改远端连接mysql数据库

```text
use mysql;

alter user 'root'@'localhost' identified with mysql_native_password by 'my_password';

flush privileges;
```

8. 查看挂载卷位置：获取容器/镜像的元数据

```text
docker inspect container_id
```
能查看带如下：

```text
"Mounts": [
            {
                "Type": "bind",
                "Source": "/home/mysql-docker/data",
                "Destination": "/var/lib/mysql",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            }
        ]
```

即linux本地宿主机目录 /home/mysql-docker/data 映射容器目录 /var/lib/mysql
