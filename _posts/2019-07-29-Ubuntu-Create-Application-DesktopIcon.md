---
layout: post
title: Ubuntu创建应用桌面快捷图标
subtitle: 'Ubuntu创建Goland桌面快捷方式'
date:       2019-07-29 12:00:00
author:     "Archer"
categories: golang
tags:
    - golang
    - IDE
---

### Ubuntu创建应用桌面快捷图标

#### 一、配置环境变量

下载最新安装包：
```text
https://www.jetbrains.com/go/download/#section=linux
```

解压至目录：
```text
/home/brook/profile/dev-tool
```

GOland启动程序如下：
```text
/home/brook/profile/dev-tool/goland-2019.2/GoLand-2019.2/bin/golang.sh
```

添加golang.sh至环境变量中,即可全局启动golang.sh：
```text
sudo vim /etc/profile
```

添加内容如下：
```text
export PATH=$PATH:/home/brook/profile/dev-tool/pycharm-professional-2019.2/pycharm-2019.2/bin
```

#### 二、创建桌面图标

进入 /usr/share/applications 目录，Ubuntu安装的软件快捷方式都保存在/usr/share/applications目录下，该目录下所有文件名都类似*.desktop

创建golang.desktop文件：
```text
[Desktop Entry]
Name=GoLand
Comment=Coding Golang
Exec=/home/brook/profile/dev-tool/goland-2019.2/GoLand-2019.2/bin/goland.sh
Icon=/home/brook/profile/dev-tool/goland-2019.2/GoLand-2019.2/bin/goland.png
Terminal=false
Type=Application
Categories=Development;
StartupNotify=true
NoDisplay=true
```

[Desktop Entry]是组名称，在规范文档中说明了必须放在所有属性前，其前面只能有注释
Type表示Desktop Entry类型，有Application , Link 和Directory 三种，使用Application表示可执行文件
Name是图标下边显示的名称
Icon是图标文件的路径，推荐使用png
Exec是启动命令，一般为可执行文件的路径，可以带有参数
Terminal表示是否在终端中运行

.desktop文件不需要可执行权限来启动程序，推荐将权限设置为644
```text
sudo chmod 744 goland.desktop 
```

使用xdg-desktop-menu命令来将自己编写的Desktop entry文件添加到系统应用列表，使用xdg-desktop-icon来将自己的Desktop entry文件添加到桌面上
```text
# 安装到应用列表
# 我们一般不会填vendor属性，使用novender选项
xdg-desktop-menu install --novendor goland.desktop
# 卸载
xdg-desktop-menu uninstall goland.desktop

# 安装到桌面
xdg-desktop-icon install --novendor goland.desktop
# 卸载
xdg-desktop-icon uninstall goland.desktop
```

即可在桌面查看