---
layout: post
title: Golang默认包管理工具
subtitle: Golang默认包管理工具go mod
date:       2020-05-27 12:00:00
author:     "Archer"
categories: 
tags:
    - golang
---

## Golang默认包管理工具

go mod是go语言内置的包管理工具，集成在go tool中，安装好go就可以使用。

### 配置go env

```shell
GO111MODULE="on"
GOARCH="amd64"
GOBIN=""
GOCACHE="/home/tangheng/.cache/go-build"
GOENV="/home/tangheng/.config/go/env"
GOEXE=""
GOFLAGS=""
GOHOSTARCH="amd64"
GOHOSTOS="linux"
GOINSECURE=""
GONOPROXY=""
GONOSUMDB=""
GOOS="linux"
GOPATH="/home/tangheng/go"
GOPRIVATE=""
GOPROXY="https://goproxy.cn,direct"
GOROOT="/usr/local/go"
GOSUMDB="sum.golang.org"
GOTMPDIR=""
GOTOOLDIR="/usr/local/go/pkg/tool/linux_amd64"
GCCGO="gccgo"
AR="ar"
CC="gcc"
CXX="g++"
CGO_ENABLED="1"
GOMOD="/dev/null"
CGO_CFLAGS="-g -O2"
CGO_CPPFLAGS=""
CGO_CXXFLAGS="-g -O2"
CGO_FFLAGS="-g -O2"
CGO_LDFLAGS="-g -O2"
PKG_CONFIG="pkg-config"
GOGCCFLAGS="-fPIC -m64 -pthread -fmessage-length=0 -fdebug-prefix-map=/tmp/go-build303757599=/tmp/go-build -gno-record-gcc-switches"
```

```shell
# 修改go env参数
go env -w GO111MODULE="on"

# 重置go env参数
go env -w GO111MODULE=""
```

### go mod init

初始化

```shell
go mod init <module-name>
```

### go mod download

下载依赖，也可直接运行main函数自动下载需要的依赖

### go mod tidy

同步依赖包，添加需要的，移除多余的

### go mod vendor

将依赖包放入项目vendor目录，方便项目部署反复下载依赖

### go get

go mod不再下载源码进$GOPATH/src

go mod的下载目录在$GOPATH/pkg/mod，并且是文件权限是只读

```shell
# 升级依赖包
go get -u 
```

### vendor 模式

go mod是不推荐使用vendor目录的，而是直接使用source或cache中的包。

module mode下默认忽略vendor目录。通过flag-mod=vendor设置vendor模式，依赖只从顶层的vendor中查找。可以通过环境变量GOFLAGS=-mod=vendor来设置flag。

```shell
go env -w GOFLAGS="-mod=vendor"
```

### replace

让原本依赖的 github.com/repo/pkg 包，实际使用 github.com/your-fork/pkg@v。

```shell
go mod edit -replace old[@v]=new[@v]

# 如果不是replace本地包，必须带上版本号
go mod edit -replace golang.org/x/crypto=github.com/golang/crypto@v0.0.0-20190621222207-cc06ce4a13d4
```

```shell
# go.mod
replace golang.org/x/crypto => github.com/golang/crypto v0.0.0-20190621222207-cc06ce4a13d4

replace golang.org/x/crypto v0.0.0-20190621222207-cc06ce4a13d4 => github.com/golang/crypto v0.0.0-20190621222207-cc06ce4a13d4
```

### 清缓存

```shell
go clean -modcache
```

### go.mod & go.sum

go.mod：依赖列表和版本约束。

go.sum：记录module文件hash值，用于安全校验。

### 引用

- [1] [golang内置包管理工具go mod简明教程](https://segmentfault.com/a/1190000019314903)
- [2] [官方文档](https://tip.golang.org/cmd/go/#hdr-Download_modules_to_local_cache)
- [3] [How do I use vendoring with modules? Is vendoring going away?](https://github.com/golang/go/wiki/Modules#how-do-i-use-vendoring-with-modules-is-vendoring-going-away)