---
layout: post
title: Ubuntu安装SSR客户端和polipo转换http代理
subtitle: 'Ubuntu+ssrcli+polipo'
date:       2019-07-28 12:00:00
author:     "Archer"
categories: ssr
tags:
    - ubuntu
    - ssr
---

### Ubuntu安装SSR客户端和polipo转换http代理

#### 一、配置SSR客户端

下载SSR客户端(本次下载为python版)：

```text
https://github.com/shadowsocks/shadowsocks/archive/2.9.1.zip
```

解压至目录：

```text
unzip shadowsocks-2.9.1.zip /home/brook/profile
```

进入目录安装：

```text
cd /home/brook/profile/shadowsocks-2.9.1 && python setup.py build && python setup.py install
```

创建SSR配置文件：

```text
sudo vim /etc/shadowsocks.json
```

```text
{
    "server":"185.171.120.112",
    "server_port":6666,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"www.freevpnnet.com",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}

```

启动SSR客户端：

```text
python /home/brook/profile/shadowsocks-2.9.1/shadowsocks/local.py -c /etc/shadowsocks.json 
```

启动后如下：

```text
INFO: loading config from /etc/shadowsocks.json
2019-07-29 21:40:00 INFO     loading libcrypto from libcrypto.so.1.1
2019-07-29 21:40:00 INFO     starting local at 127.0.0.1:1080
2019-07-29 22:01:02 INFO     connecting www.google.com:443 from 127.0.0.1:35646
```

#### 二、转换http

Shadowsocks默认是用Socks5协议的，对于Terminal的get,wget等走http协议的地方是无能为力的，所以需要转换成http代理，加强通用性，这里使用的转换方法是基于Polipo

安装polipo：

```text
sudo apt-get install polipo
```

修改配置文件：

```text
logSyslog = true
logFile = /var/log/polipo/polipo.log

socksParentProxy = "127.0.0.1:1080"
socksProxyType = socks5

chunkHighMark = 50331648
objectHighMark = 16384

serverMaxSlots = 64
serverSlots = 16
serverSlots1 = 32

proxyAddress = "0.0.0.0"
proxyPort = 8123

```

重启polipo服务：

```text
sudo /etc/init.d/polipo restart
```

验证代理是否正常工作：

```text
export http_proxy="http://127.0.0.1:8123"
curl www.google.com
```

又返回结果极为正常，在浏览器中输入`http://127.0.0.1:8123`可进入到Polipo的使用说明和配置界面

可在chorme插件SwitchyOmega配置http代理
代理协议：http
代理服务器：127.0.0.1
端口：8123

firefox可直接在设置里设置，如上
或者配置系统网络配置，如上

#### 三、设置监控进程

参考 [首页-简书](https://tangheng1995.github.io/centos/2019/07/25/CentOS-install-Supervisor/)
