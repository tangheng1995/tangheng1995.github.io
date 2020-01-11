---
layout: post
title: 'CentOS 安装 jekyll'
subtitle: 'CentOS 安装 jekyll'
date: 2019-06-01
author: Archer
categories: centos
cover: ''
tags: centos jekyll blog github
---

1. 安装依赖

    ```text
    sudo yum install nodejs npm ruby ruby-devel rubygems git
    ```

2. 修改gem源

    ```text
    gem sources --add https://gems.ruby-china.com/ --remove https://rubygems.org/

    gem sources -l
    ```

3. jekyll需要ruby 2.1以上，先安装ram升级ruby

    ```text
    curl -sSL https://get.rvm.io | bash -s stable
    ### 应用ram
   source /etc/profile.d/rvm.sh
    ```

4. 查看ruby版本

    ```text
    rvm list known
    ```

5. 安装ruby 2.5

    ```text
    rvm install 2.5
    ### 验证版本
    ruby -v
    ```
  
6. 安装jekyll

    ```text
    gem install jekyll
    ```

7. 寻找自己喜欢的主题

    ```text
    http://jekyllthemes.org/
    ```
