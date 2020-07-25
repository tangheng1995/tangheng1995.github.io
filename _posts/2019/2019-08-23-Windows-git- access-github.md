---
layout: post
title: "Git 免密登录 Github"
date: "2019-08-23 12:00:00"
category: git
tags: git
author: Archer
---
* content
{:toc}

## 介绍

Windows Git 免密登录 Github。




## Windows git 免密登录github

### git生成公钥

```shell
# 其中邮箱为GitHub的邮箱，直接下一步，默认安装在Windows用户目录下：C:\Users\myuser\.ssh
ssh-keygen -t rsa -C "email@email.com"
```

```shell
# 目录下文件如下：
id_rsa
id_rsa.pub
```

### 复制公钥内容至github

![pub](/assets/images/2019/2019-08-23-git-access-github-02.png?raw=true)

### 验证ssh keys是否成功

```shell
ssh -T git@github.com
```

![ssh keys](/assets/images/2019/2019-08-23-git-access-github.png)
