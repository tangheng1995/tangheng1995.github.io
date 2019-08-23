---
layout: post
title: Windows git 免密登录github
subtitle: git免密登录github
date:       2019-08-23 12:00:00
author:     "Archer"
categories: git
tags:
    - git
    - github
---

### Windows git 免密登录github

#### 一、git生成公钥
```text
# 其中邮箱为GitHub的邮箱，直接下一步，默认安装在Windows用户目录下：C:\Users\myuser\.ssh
ssh-keygen -t rsa -C "email@email.com" 
```
```text
# 目录下文件如下：
id_rsa
id_rsa.pub
```

#### 二、复制公钥内容至github
![]()

#### 三、验证ssh keys是否成功
```text
ssh -T git@github.com
```
![]()