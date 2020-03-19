---
layout: post
title: Mysql-8.0压缩包安装(Win)
subtitle: mysql-8.0.19-winx64
date:       2020-03-20 12:00:00
author:     "Archer"
categories: 
tags:
    - mysql
---

### Mysql-8.0压缩包安装(Win)

#### 1. 下载zip安装包

![(mysql-8.0.19-winx64.zip)](https://dev.mysql.com/downloads/file/?id=492455)

#### 2. 安装

##### 2.1 解压

我解压目录为 D:\devsoft\Mysql\mysql-8.0.19-winx64

> 解压后的文件目录，缺失data文件夹和my.ini配置文件

添加路径至环境变量PATH：

```text
D:\devsoft\Mysql\mysql-8.0.19-winx64
D:\devsoft\Mysql\mysql-8.0.19-winx64\bin
```

##### 2.2 配置初始化my.ini文件

在根目录 D:\devsoft\Mysql\mysql-8.0.19-winx64 添加 my.ini文件并创建data目录:

```text
[mysqld]
# 设置3306端口
port=3306

# 设置mysql的安装目录
basedir=D:\devsoft\Mysql\mysql-8.0.19-winx64   # 根据自己的地址更改

# 设置mysql数据库的数据的存放目录
datadir=D:\devsoft\Mysql\mysql-8.0.19-winx64\data   # 根据自己的地址更改

<!-- # 允许最大连接数 -->
max_connections=200

# 允许连接失败的次数。这是为了防止有人从该主机试图攻击数据库系统
max_connect_errors=10

# 服务端使用的字符集默认为UTF8
character-set-server=utf8mb4

# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB

# 默认使用“mysql_native_password”插件认证
default_authentication_plugin=mysql_native_password

[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8mb4

[client]
# 设置mysql客户端连接服务端时默认使用的端口
port=3306
default-character-set=utf8mb4
```

##### 2.3 安装mysql

在安装时，必须以管理员身份运行cmd，否则在安装时会报错，会导致安装失败的情况

遇到在 C:\Windows\system32> 命令无法切换到需要安装mysql的目录时，执行：

```text
cd /d D:\devsoft\Mysql\mysql-8.0.19-winx64\bin
```

在MySQL安装目录的 bin 目录路径下执行命令：

```text
mysqld --initialize --console
```

执行完成后，会打印 root 用户的初始默认密码,记录密码

##### 2.4 启动mysql服务

```text
# 启动
net start mysql

# 停止
net stop mysql

# 删除服务
sc delete mysql
```

##### 2.5 连接mysql更改密码

```text
# 登录
mysql -u root -p

# 遇到报错，win + r 输入 services.msc 启动 mysql 服务
ERROR 2003 (HY000): Can't connect to MySQL server on 'localhost' (10061)
```

修改密码：

```text
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '新密码';  
```
