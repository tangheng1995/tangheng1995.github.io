---
layout: post
title: "Linux SSH远程登陆免密"
date: "2019-07-31 12:00:00"
category: linux
tags: linux
author: Archer
---
* content
{:toc}

## 介绍

Linux SSH远程登陆免密。




## Linux ssh远程登陆免密

目的：
要求节点A免密登陆到节点B

方案：
就是A想要连接B免密登录B，需要把A的id_rsa.pub文件内容写进B的authorized_keys里面

### 生成公钥密钥(此处我默认生成在/home/brook/.ssh/目录)

```text
ssh-keygen -t rsa -P ""
```

```text
Generating public/private rsa key pair.
Enter file in which to save the key (/home/brook/.ssh/id_rsa):
Your identification has been saved in /home/brook/.ssh/id_rsa.
Your public key has been saved in /home/brook/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:lGuZ63vwuXyI0wf9J06AtOlbPA7Np3hWzBfjJisU+Hc brook@brook-X75VC
The key's randomart image is:
+---[RSA 2048]----+
|                 |
|         .       |
|        o o      |
|       . * =   o |
|        S =.oo. o|
|       ..o.*.++E.|
|        .=++BoB. |
|       .o.*B=*o .|
|        o+=*+..o |
+----[SHA256]-----+
```

### 拷贝A的公钥至B的 ~/.ssh/ 目录下

- 方法一

```shell
# 此处 -p port 指的是B节点ssh端口
ssh-copy-id -i ~/.ssh/id_rsa.pub user@host -p port
# 输入密码即可，下次磕免密登陆
```

- 方法二

```shell
# 此处 -P port 指的是B节点ssh端口
scp -P port /home/brook/.ssh/id_rsa.pub user@host:~/.ssh/id_rsa.pub1

# 再登陆B节点/.ssh/ 目录，将id_rsa.pub1内容写至authorized_key文件
cat id_rsa.pub1 > authorized_key
```
