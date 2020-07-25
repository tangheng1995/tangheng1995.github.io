---
layout: post
title: "CentOS 安装 jekyll"
date: "2019-06-01 12:00:00"
category: linux
tags: linux,blog
author: Archer
---
* content
{:toc}

## 介绍

CentOS 安装 jekyll 博客。




## CentOS 安装 jekyll

> EPEL (Extra Packages for Enterprise Linux)是基于Fedora的一个项目，为“红帽系”的操作系统提供额外的软件包，适用于RHEL、CentOS和Scientific Linux.

```shell
yum install -y  epel-release
```

### 安装jekyll

```shell
# 安装依赖
sudo yum install nodejs npm ruby ruby-devel rubygems git

# 修改gem源
gem sources --add https://gems.ruby-china.com/ --remove https://rubygems.org/
gem sources -l

# jekyll需要ruby 2.1以上，先安装ram升级ruby
curl -sSL https://get.rvm.io | bash -s stable

# 应用ram
source /etc/profile.d/rvm.sh

# 查看ruby版本
rvm list known

# 安装ruby 2.5
rvm install 2.5

# 验证版本
ruby -v

# 安装jekyll
gem install jekyll
```

### 主题

[寻找自己喜欢的主题](http://jekyllthemes.org/)
