---
layout: post
title: Sonar 进行Golang代码质量管理
subtitle: Sonar 进行Golang代码质量管理
date:       2020-06-30 12:00:00
author:     "Archer"
categories: 
tags:
    - golang
---

## Sonar 进行Golang代码质量管理

### Sonar 概述

Sonar 是一个用于代码质量管理的开放平台。通过插件机制，Sonar 可以集成不同的测试工具，代码分析工具，以及持续集成工具。

与持续集成工具（例如 Hudson/Jenkins 等）不同，Sonar 并不是简单地把不同的代码检查工具结果（例如 FindBugs，PMD 等）直接显示在 Web 页面上，而是通过不同的插件对这些结果进行再加工处理，通过量化的方式度量代码质量的变化，从而可以方便地对不同规模和种类的工程进行代码质量管理。

本次测试使用golang接入sonar，可展示可靠性、安全性、可维护性、覆盖率、重复等多项指标。

### 环境

- sonarqube-7.9.1-community
- [sonar-scanner-cli-4.2.0.1873](https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.2.0.1873-linux.zip)
- postgres-10

### 关键字

指标 | 简介
---- | ---
Bugs | bug个数及评分
Vulnerabilities |  安全漏洞个数及评分
Debt |  债务(代码问题)持续时间
Code Smells |  轻微问题：代码风格等等
Coverage |  单元测试覆盖率
Duplications |   代码重复率
Duplicated Blocks |  代码重复块数

### 安装服务端

本次测试选择SonarQube-7.9.1社区版docker镜像测试

创建挂在卷：

```text
$> docker volume create --name sonarqube_data
$> docker volume create --name sonarqube_extensions
$> docker volume create --name sonarqube_logs
```

启动postgres

```text
docker run -d -p 5432:5432 -e POSTGRES_PASSWORD=1 --name postgres postgres:10
```

启动sonarqube，因为我安装汉化插件，所以扩展目录我挂在 `/home/tangheng/profile/sonar/extensions`

```text
docker run -d --name sonarqube \
    -p 9000:9000 \
    -e SONAR_JDBC_URL=jdbc:postgresql://172.18.0.1:5432/sonar \
    -e SONAR_JDBC_USERNAME=postgres \
    -e SONAR_JDBC_PASSWORD=1 \
    -v sonarqube_data:/opt/sonarqube/data \
    -v /home/tangheng/profile/sonar/extensions:/opt/sonarqube/extensions \
    -v sonarqube_logs:/opt/sonarqube/logs \
    sonarqube:7.9.1-community
```

安装汉化插件：

SonarQube - 中文插件安装
下载地址：<https://github.com/SonarQubeCommunity/sonar-l10n-zh/releases/tag/sonar-l10n-zh-plugin-1.16>

将sonar-l10n-zh-plugin-1.16.jar放置 `/home/tangheng/profile/sonar/extensions/plugins` 目录

安装成功后，访问 <http://localhost:9000/> ，默认帐号密码： admin/admin

### 安装客户端

安装 sonar-scanner 至 `/usr/local/sonar-scanner` 目录：

```text
wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.2.0.1873-linux.zip
unzip sonar-scanner-cli-4.2.0.1873-linux.zip
mv sonar-scanner-4.2.0.1873-linux /usr/local/sonar-scanner
```

修改配置文件 `/usr/local/sonar-scanner/conf/sonar-scanner.properties`：

```text
#Configure here general information about the environment, such as SonarQube server connection details for example
#No information about specific project should appear here

#----- Default SonarQube server
sonar.host.url=http://localhost:9000

#----- Default source code encoding
sonar.sourceEncoding=UTF-8

sonar.jdbc.username=postgres
sonar.jdbc.password=1

#项目使用的语言
sonar.language=go  
#项目的独特关键字,maven 项目是 <groupId>:<artiactId>，go 项目自己定义就可以
sonar.projectKey=projectKey  
#将在web界面上显示的名字
sonar.projectName=cms 
#项目版本
sonar.projectVersion=1.0  
#需要分析的源码目录的路径
sonar.sources=.  
sonar.exclusions=**/*_test.go,**/vendor/**  
sonar.tests=.  
sonar.test.inclusions=**/*_test.go  
sonar.test.exclusions=**/vendor/**  
#golangci-lint 报告路径
sonar.go.golangci-lint.reportPaths=report.xml  
#单测覆盖率报告地址
sonar.go.coverage.reportPaths=coverage.out

sonar.jdbc.url=jdbc:mysql://172.18.0.1:5432/sonar?useUnicode=true&characterEncoding=utf8
```

配置环境变量：

```text
export PATH=$PATH:/usr/local/sonar-scanner/bin
```

验证：

```text
sonar-scanner -v

# 输出
INFO: Scanner configuration file: /usr/local/sonar-scanner/conf/sonar-scanner.properties
INFO: Project root configuration file: NONE
INFO: SonarQube Scanner 4.2.0.1873
INFO: Java 11.0.3 AdoptOpenJDK (64-bit)
INFO: Linux 4.19.0-6-amd64 amd64
```

### 检测代码

以cms项目为例。目前已经做了单元测试及覆盖率，不过多介绍：

makefile：

```text
test:
	go test $(CURDIR)/test -v -coverprofile=coverage.out -covermode=count -coverpkg=./...

coverage: test
	find . -name "coverage.out.*" -exec tail +2 {} >> coverage.out \;
	go tool cover -html coverage.out -o coverage.html
	go tool cover -func=coverage.out

scanner: coverage
	sonar-scanner
```

执行 `sonar-scanner` 命令，可将生成的报告上传到服务器。

### 总结

Sonar 为代码的质量管理提供了一个平台，可以说是目前最强大的代码质量管理工具之一。对项目增加CI测试流程同时，上传测试报告到统一服务器，既方便整体了解项目性能及漏洞，也能为后续迭代重构时提供有效参考。

### 参考

- [sonarqube服务端docker安装](https://docs.sonarqube.org/latest/setup/install-server/)
- [golang 单元测试和代码覆盖率统计](https://docsin.uniontech.com/?p=6984)
- [sonar汉化插件](https://github.com/SonarQubeCommunity/sonar-l10n-zh/releases/tag/sonar-l10n-zh-plugin-1.16)
