---
layout: post
title: "Git 常用配置"
date: "2020-04-01 12:00:00"
category: git
tags: git
author: Archer
---
* content
{:toc}

## 介绍

Git 常用配置。

- 配置多用户
- 配置代理




## Git 配置多用户

```text
# 用户A
name: A
email: a-mail@mail.com

# 用户B
name: B
email: b-mail@mail.com
```

### 生成ssh-key

```shell
# 用户A
ssh-keygen -t rsa -C "a-mail@mail.com"

# 用户B
ssh-keygen -t rsa -C "b-mail@mail.com"
```

分别生成公钥私钥：

```shell
# 用户A
id_rsa_a 和 id_rsa_a.pub

# 用户B
id_rsa_b 和 id_rsa_b.pub
```

将公钥内容粘贴到自己的github或者其他第三方SSH公钥上。

### 创建或修改config文件

创建或修改config文件 (默认.ssh下)

```text
Host github.com
    HostName github.com
    IdentityFile ~/.ssh/id_rsa_a
    User A

Host gitee.com
    HostName gitee.com
    IdentityFile ~/.ssh/id_rsa_b
    User B
```

### 测试

```text
ssh -vT git@github.com
ssh -vT git@gitee.com
```

进各自项目设置用户邮箱：

```text
git config user.name "yourname"
git config user.email "youremail"
```

## 配置代理

配置github代理，加速下载：

```text
git config --global http.https://github.com.proxy socks5://127.0.0.1:1080

#取消代理
git config --global --unset http.https://github.com.proxy
```

查看：

```text
git config --global -e
```
