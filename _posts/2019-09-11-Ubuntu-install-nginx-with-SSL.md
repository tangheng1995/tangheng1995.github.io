---
layout: post
title: Ubuntu 安装 nginx，配置SSL加密
subtitle: Ubuntu 安装 nginx，配置SSL加密
date:       2019-09-11 12:00:00
author:     "Archer"
categories: nginx
tags:
    - ubuntu
    - nginx
    - ssl
---

## Ubuntu 安装 nginx，配置SSL加密

### 安装nginx

```shell
sudo apt-get nginx
```

### 修改nginx配置文件

```conf
# 进入 `/etc/nginx/sites-enabled` 目录，可查看nginx配置文件 `default -> /etc/nginx/sites-available/default`
# 创建新配置文件 go.conf
server {
        listen       80;
        server_name  blog.heygolang.cn;
        location / {
                proxy_pass https://tangheng1995.github.io/;
        }
  location /admin {
                proxy_pass http://127.0.0.1:9000;
        }
        error_page 404 /404.html;
            location = /40x.html {
        }
    }
server {
        listen       80;
        server_name  admin.heygolang.cn;
  location / {
                proxy_pass http://127.0.0.1:9000;
        }
        error_page 404 /404.html;
            location = /40x.html {
        }
    }
```

### 配置SSL证书

> 本次选用的SSL证书是由 [Let’s Encrypt](https://letsencrypt.org/) 提供的，本次系统及环境为 Ubuntu 18.04 和 nginx
> 配置详情页为 [certbot instructions](https://certbot.eff.org/lets-encrypt/ubuntubionic-nginx)

#### 添加 Certbot PPA

```shell
sudo apt-get update
sudo apt-get install software-properties-common
sudo add-apt-repository universe
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
```

#### 安装Certbot

```shell
sudo apt-get install certbot python-certbot-nginx
```

#### 选择获取证书还是安装证书

```shell
# 安装证书
sudo certbot --nginx
# 获取证书
sudo certbot certonly --nginx
```

#### 验证自动更新证书

> Certbot 会在证书过期前自动更新证书，除非你修改了配置

```shell
# 验证自动更新证书
sudo certbot renew --dry-run
```

此命令会更新以下目录下安装的配置：

```shell
/etc/crontab/
/etc/cron.*/*
systemctl list-timers
```

#### 验证你的网站是否生效

```shell
https://yourwebsite.com/
```

### 查看被Certbot修改后的nginx配置文件

```conf
server {
        server_name  blog.heygolang.cn;
        location / {
                proxy_pass https://tangheng1995.github.io/;
        }
        error_page 404 /404.html;
            location = /40x.html {
        }
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/admin.heygolang.cn/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/admin.heygolang.cn/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}
server {
        server_name  admin.heygolang.cn;
        location / {
                proxy_pass http://127.0.0.1:9000;
        }
        error_page 404 /404.html;
            location = /40x.html {
        }
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/admin.heygolang.cn/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/admin.heygolang.cn/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}
server {
    if ($host = admin.heygolang.cn) {
        return 301 https://$host$request_uri;
    } # managed by Certbot
        listen       80;
        server_name  admin.heygolang.cn;
    return 404; # managed by Certbot
}server {
    if ($host = blog.heygolang.cn) {
        return 301 https://$host$request_uri;
    } # managed by Certbot
        listen       80;
        server_name  blog.heygolang.cn;
    return 404; # managed by Certbot
}
```

### 引用

- [1] [Caddy](https://caddyserver.com/), 可以自动给http加密成https
