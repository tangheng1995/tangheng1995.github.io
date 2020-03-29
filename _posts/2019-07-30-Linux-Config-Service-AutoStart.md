---
layout: post
title: Ubuntu配置服务自启
subtitle: 'Ubuntu 配置服务自启'
date:       2019-07-30 12:00:00
author:     "Archer"
categories: lixnu
tags:
    - linux
---

## Ubuntu配置服务自启

### /etc/rc.local配置服务启动

#### 修改rc.local

systemd 默认读取 /etc/systemd/system 下的文件，该目录下的文件会链接/lib/systemd/system/下的文件。

执行 ls /lib/systemd/system 你可以看到有很多启动脚本，其中就有我们需要的 rc.local.service

修改rc.local.service

```shell
sudo vim /lib/systemd/system/rc.local.service
```

修改后如下：

```text
[Unit]
Description=/etc/rc.local Compatibility
Documentation=man:systemd-rc-local-generator(8)
ConditionFileIsExecutable=/etc/rc.local
After=network.target

[Service]
Type=forking
ExecStart=/etc/rc.local start
TimeoutSec=0
RemainAfterExit=yes
GuessMainPID=no

[Install]
WantedBy=multi-user.target
Alias=rc-local.service
```

[Unit] 段: 启动顺序与依赖关系； [Service] 段: 启动行为,如何启动，启动类型； [Install] 段: 定义如何安装这个配置文件，即怎样做到开机启动；

可以看出，/etc/rc.local 的启动顺序是在网络后面，但是显然它少了 Install 段，也就没有定义如何做到开机启动，所以显然这样配置是无效的。 因此我们就需要在后面帮他加上 [Install] 段。

#### 创建rc.local脚本

Ubuntu 18.04 默认没有/etc/rc.local这个文件，需要自己创建

```shell
sudo vim /etc/rc.local
```

创建内容如下：

```shell
#!/bin/bash
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.
# 添加服务启动命令
echo "看到这行字，说明添加自启动脚本成功。" > /usr/local/test.log
exit 0

nohup python /home/brook/PycharmProjects/flask-hello/hello.py &

```

重载units单元

```shell
systemctl daemon-reload
```

加权限：

```shell
# 加权限
sudo chmod +x /etc/rc.local
```

前面我们说 systemd 默认读取 /etc/systemd/system 下的配置文件, 所以还需要在 /etc/systemd/system 目录下创建软链接

将服务告知系统自启的命令。

```shell
# 开启服务
systemctl enable rc-local
# 等同于
sudo ln -s '/usr/lib/systemd/system/rc-local' '/etc/systemd/system/multi-user.target.wants/rc-local'
```

启动服务：

```shell
service rc.local start
# 或者
sudo systemctl start rc.local
```

启动结果如下：

```text
● rc-local.service - /etc/rc.local Compatibility
   Loaded: loaded (/lib/systemd/system/rc-local.service; enabled; vendor preset: enabled)
  Drop-In: /lib/systemd/system/rc-local.service.d
           └─debian.conf
   Active: active (exited) since Tue 2019-07-30 10:57:51 CST; 7s ago
     Docs: man:systemd-rc-local-generator(8)
  Process: 11026 ExecStart=/etc/rc.local start (code=exited, status=0/SUCCESS)

7月 30 10:57:51 brook-X75VC systemd[1]: Starting /etc/rc.local Compatibility...
7月 30 10:57:51 brook-X75VC systemd[1]: Started /etc/rc.local Compatibility.

```

检查启动服务的日志,有打印内容即可：

```shell
cat /usr/local/test.log
```

检查python脚本是否启动(巨坑，启动脚本使用root启动了，我普通愣是没检查到，一上午反反复复在检查)：

```shell
# flask最小应用，默认端口5000
sudo lsof -i:5000
```

### /etc/init.d/配置

在/etc/init.d/目录下新建启动脚本flask-hello.sh，按照LSB tags规范改写脚本如下(否则update-rc.d的时候会报警告
insserv: warning: script 'flask-hello.sh' missing LSB tags and overrides)：

```shell
#!/bin/bash

### BEGIN INIT INFO
# Provides:          brook
# Required-Start:    $local_fs $network
# Required-Stop:     $local_fs
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: flask-hello service
# Description:       flask-hello service daemon
### END INIT INFO

nohup python /home/brook/PycharmProjects/flask-hello/hello.py &
```

设置权限：

```shell
sudo chmod +x flask-hello.sh
```

将启动脚本添加到系统启动脚本里,此处99代表优先级，数字越大表示执行越晚：

```shell
cd /etc/init.d/ && sudo update-rc.d flask-hello.sh defaults 99
```

### 使用启动项管理工具sysv-rc-conf

```shell
# 安装
sudo apt-get install sysv-rc-conf

# 运行
sudo ysv-rc-conf

# 参数命令：
Ctrl+N:下一页
Ctrl+P：上一页
Space：选中或取消
```

### 扩展

**update-rc.d**
Ubuntu或者Debian系统中update-rc.d命令，是用来更新系统启动项的脚本。这些脚本的链接位于/etc/rcN.d/目录，对应脚本位于/etc/init.d/目录。在了解update-rc.d命令之前，你需要知道的是有关Linux系统主要启动步骤，以及Ubuntu中运行级别的知识

**Linux 系统主要启动步骤**
读取 MBR 的信息，启动 Boot Manager
加载系统内核，启动 init 进程， init 进程是 Linux 的根进程，所有的系统进程都是它的子进程
init 进程读取 /etc/inittab 文件中的信息，并进入预设的运行级别。通常情况下/etc/rcS.d/ 目录下的启动脚本首先被执行，然后是/etc/rcN.d/ 目录
根据 /etc/rcS.d/ 文件夹中对应的脚本启动 X-window 服务器 xorg，X-window 为 Linux 下的图形用户界面系统。
启动登录管理器，等待用户登录。

**Linux运行级别**
Linux系统有7个运行级别(runlevel)

- 运行级别0：系统停机状态，系统默认运行级别不能设为0，否则不能正常启动
- 运行级别1：单用户工作状态，root权限，用于系统维护，禁止远程登陆
- 运行级别2：多用户状态(没有NFS)
- 运行级别3：完全的多用户状态(有NFS)，登陆后进入控制台命令行模式
- 运行级别4：系统未使用，保留
- 运行级别5：X11控制台，登陆后进入图形GUI模式
- 运行级别6：系统正常关闭并重启，默认运行级别不能设为6，否则不能正常启动

**update-rc.d的参数使用**
update-rc.d的参数使用

- -n：不做任何事情，只显示将要做的(预览、做测试)
- -f：强制移除符号连接，即使/etc/init.d/script-name仍然存在 [NN | SS KK]：NN表示执行序号（0-99），SS表示启动时的执行序号，KK表示关机时的执行序号，SS、KK主要用于在脚本直接的执行顺序上有依赖关系的情况下

**举例**
举例

```shell
 #1.添加一个启动脚本，执行序号是99。
 $ update-rc.d startBlog defaults 99
 #2.A启动后B才能启动，B关闭后A才关闭（A的启动顺序比B的小，结束顺序比B的大）
 $ update-rc.d script_for_A defaults 80 20
 $ update-rc.d script_for_B defaults 90 10
 #3.添加一个不被其他任何服务需要的服务
 $ update-rc.d script_name defaults 98 02
 #4.添加一个需要 开始/结束 序号在20的服务的服务
 $ update-rc.d script_depends_on_service_20 defaults 20
 #5.移除一个脚本，假定/etc/init.d/目录下的脚本文件已先被删除
 $ update-rc.d script_name remove
 #6.移除一个脚本，不管/etc/init.d/目录下的脚本文件是否已删除
 $ update-rc.d -f script_name remove
 #7.按指定顺序、在指定运行级别中启动或关闭
 $ update-rc.d ＜basename＞ start|stop ＜order＞ ＜runlevels＞
 #####实例：
 $ update-rc.d apachectl start 20 2 3 4 5 . stop 20 0 1 6 .
 #####解析：表示在2、3、4、5这五个运行级别中，由小到大，第20个开始运行apachectl；
      #在 0 1 6 这3个运行级别中，第20个关闭apachectl。
      #这是合并起来的写法，注意它有2个点号，效果等于下面方法：
 $ update-rc.d apachectl defaults
```

**开机启动/禁止启动某个服务**
开机启动/禁止启动某个服务

```text
update-rc.d [-n] name enable|disable [ S|2|3|4|5 ]
update-rc.d cron enable
删除开机启动项

update-rc.d [-n] [-f] name remove
update-rc.d -n -f supervisor remove
```
