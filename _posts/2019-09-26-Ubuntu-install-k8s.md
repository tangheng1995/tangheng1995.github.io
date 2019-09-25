---
layout: post
title: Ubuntu 安装 K8S
subtitle: Ubuntu 安装 kubectl minikute
date:       2019-09-26 12:00:00
author:     "Archer"
categories: 
tags:
    - k8s
    - ubuntu
---

### Ubuntu 安装 K8S

#### 一、安装kubectl
```text
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
```
```text
chmod +x ./kubectl
```
```text
sudo mv ./kubectl /usr/local/bin/
```
```text
kubectl version
```

#### 二、安装 minikube
```text
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
  && chmod +x minikube
```
```text
sudo mv ./minikube /usr/local/bin/
```
- minikube 启动方法一，运行在虚拟机，这要求当前电脑上要先安装VirtualBox或者KVM：
```text
minikube start
```

- minikube 启动方法二，运行在宿主机，这种方式不需要在当前电脑安装ViirtualBox或者KVM：
```text
minikube start --vm-driver=none
```