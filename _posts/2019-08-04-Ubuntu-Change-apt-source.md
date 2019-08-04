---
layout: post
title: Ubuntu18.04下更改apt源为阿里云源
subtitle: 'Ubuntu更换apt源'
date:       2019-08-04 12:00:00
author:     "Archer"
header-img: ""
categories: Ubuntu
tags:
    - Ubuntu
    - apt
---

### Ubuntu18.04下更改apt源为阿里云源

#### 一、备份apt源文件

修改的文件是sources.list，它在目录/etc/apt/下，sources.list是包管理工具apt所用的记录软件包仓库位置的配置文件，同样类型的还有位于 同目录
下sources.list.d文件下的各种.list后缀的各文件。

```text
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak.20190804
```

#### 二、查看系统版本信息

```text
lsb_release -c
```
```text
结果：
Codename:	bionic
```

```text
Ubuntu 12.04 (LTS)代号为precise。
Ubuntu 14.04 (LTS)代号为trusty。
Ubuntu 15.04 代号为vivid。
Ubuntu 15.10 代号为wily。
Ubuntu 16.04 (LTS)代号为xenial。
Ubuntu 18.04 (LTS)代号为bionic。
```

#### 三、修改apt源文件

```text
# 进入目录
cd /etc/apt/
# 清空source.list文件
sudo > sources.list
sudo vim sources.list
```
```text
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse

deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse

deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse

deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse

deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse

deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
```

sources.list文件的条目格式如下：
```text

deb http://site.example.com/debian distribution component1 component2 component3
deb-src http://site.example.com/debian distribution component1 component2 component3
```
后面几个参数是对软件包的分类（Ubuntu下是main， restricted，universe ，multiverse这四个），所以也可以编辑如下：
```text
deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted 
deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed universe multiverse
```

#### 四、更新软件列表和软件包
```text
sudo apt-get update 
sudo apt-get upgrade
```

#### 五、常见问题
1. sudo apt-get update 报错如下：
```text
Original exception was:
Traceback (most recent call last):
  File "/usr/lib/cnf-update-db", line 8, in <module>
    from CommandNotFound.db.creator import DbCreator
  File "/usr/lib/python3/dist-packages/CommandNotFound/db/creator.py", line 11, in <module>
    import apt_pkg
ModuleNotFoundError: No module named 'apt_pkg'
```

解决办法：
```text
sudo apt-get remove python3-apt
sudo apt-get install python3-apt
```

