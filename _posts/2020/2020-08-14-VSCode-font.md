---
layout: post
title: "VSCode font 字体配置"
date: "2020-08-14 12:00:00"
category: tool
tags: tool
author: Archer
---
* content
{:toc}

## 介绍

Kali 2020.2版安装VSCode字体显示异常，editor、terminal字体异常。




## 下载安装字体

```shell
$cd /usr/share/fonts/truetype/
$sudo git clone https://github.com/abertsch/Menlo-for-Powerline.git
```

刷新字体

```shell
$sudo fc-cache -f -v
```

## 设置字体

`editor.fontFamily` 第一项设置 `'Menlo for Powerline'` 即可恢复editor字体显示。

`terminal.integrated.fontFamily` 设置 `Menlo for Powerline` 即可恢复terminal字体显示
