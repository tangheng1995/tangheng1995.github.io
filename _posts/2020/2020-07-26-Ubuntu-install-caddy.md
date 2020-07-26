---
layout: post
title: "Linux 安装 Caddy 及使用"
date: "2020-07-26 16:33"
category: linux
tags: linux
author: Archer
---
* content
{:toc}

Caddy 是一款使用 Go 语言开发的 Web 服务器。其配置更为简洁，并可以自动申请及配置 SSL 证书。




## 安装

添加 caddy 官方的软件包源到 Ubuntu 的包管理器中，并通过管理器直接安装 caddy v2，安装完成会自动运行 caddy 服务。

```text
#添加caddy官方的软件源，更新软件包并安装caddy v2
#
$ echo "deb [trusted=yes] https://apt.fury.io/caddy/ /" \
    | sudo tee -a /etc/apt/sources.list.d/caddy-fury.list
$ sudo apt update
$ sudo apt install caddy
```

安装顺利出现：

```text
...
Unpacking caddy (2.0.0) ...###.................................]

Setting up caddy (2.0.0) ...###############....................]
...
```

查看caddy版本信息，以验证是否安装成功

```text
$ caddy version

# v2.0.0 h1:pQSaIJGFluFvu8KDGDODV8u4/QRED/OPyIR+MWYYse8=
```

caddy 的服务管理

```text
systemctl enable caddy.service   # 开机启动
systemctl start caddy.service    # 启动
systemctl stop caddy.service     # 停止
systemctl restart caddy.service  # 重启
systemctl status caddy.service   # 查看状态
systemctl daemon-reload          # 重载配置

```

## 配置

安装成功后，默认生成的 caddy 配置文件Caddyfile中的部分信息，如下：

```text
# domain name.
:80

# Set this path to your site's directory.
root * /usr/share/caddy

# Enable the static file server.
file_server

# Another common task is to set up a reverse proxy:
# reverse_proxy localhost:8080
```

自动生成证书的位置:$HOME/.caddy或者使用caddy -log stdout查看日志输出信息

配置多个站点：

```text
# 站点1
# 在配置的虚拟主机域名后使用{}，该主机的所有配置信息包含在{}中。
localhost {
 respond "Hello, world!"
}

# 站点2
# 在配置的虚拟主机域名后使用{}，该主机的所有配置信息包含在{}中。
localhost:2016 {
 respond "Goodbye, world!"
}
```

Caddyfile 配置文件示例

```text
example.com {
  root /var/www/example.com  # 如果不需要在本地放置站点内容，可以不需要。
  gzip
  reverse_proxy /ray localhost:10000 {
    header_up -Origin
  }
}
```

说明：

* root /var/www/example.com：example.com 域名指向的网站根目录；

* proxy /ray localhost:10000 {…}：将特定的路径请求（这里指发送到 / ray 的请求），转发到主机本地端口 10000。

修改了配置文件后，需要重新就加载 caddy 服务：`systemctl restart caddy.service`

## 引用

* [1] [caddy doc](https://caddyserver.com/docs/download)
