---
layout: post
title: Ubuntu 安装 K8S
subtitle: Ubuntu 安装 kubectl minikute
date:       2019-09-28 12:00:00
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

![minikube](https://github.com/tangheng1995/tangheng1995.github.io/blob/master/img/in-post/post-js-version/2019-09-28-minilube-start.png?raw=true)

#### 三、启动镜像

```text
sudo kubectl run kubia --image=registry.cn-hongkong.aliyuncs.com/k8s_in_action/kubia:v0.0.1 --port=8080
```

因为是minikube创建的集群，所有不能通过LoadBalancer类型暴露外部访问连接

```text
sudo kubectl expose deployment kubia --type=NodePort
```

查看外部访问连接

```text
minikube service --url kubia
```

#### 四、deployment，pod，服务如何组合运行的

查看pod

```text
sudo kubectl get po
```

如下：

```text
NAME                     READY   STATUS    RESTARTS   AGE
kubia-84654c6d97-zxnb8   1/1     Running   0          12h
```

查看详情：

```text
sudo kubectl describe po kubia-84654c6d97-zxnb8
```

如下：

```text
Name:         kubia-84654c6d97-zxnb8
Namespace:    default
Priority:     0
Node:         minikube/192.168.17.131
Start Time:   Fri, 27 Sep 2019 19:02:20 +0000
Labels:       pod-template-hash=84654c6d97
              run=kubia
Annotations:  <none>
Status:       Running
IP:           172.17.0.2
IPs:
  IP:           172.17.0.2
Controlled By:  ReplicaSet/kubia-84654c6d97
Containers:
  kubia:
    Container ID:   docker://f42b859c86c85a93baca41b865c94e475973497c9abf6327f6a335b8d4886398
    Image:          registry.cn-hongkong.aliyuncs.com/k8s_in_action/kubia:v0.0.1
    Image ID:       docker-pullable://registry.cn-hongkong.aliyuncs.com/k8s_in_action/kubia@sha256:9497e7b118a92d1d3c9e0823a10c9b1cc94259c04689df49f8ef355b76eac052
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Fri, 27 Sep 2019 19:02:22 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-s6gm6 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-s6gm6:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-s6gm6
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:          <none>
```

查看服务：

```text
sudo kubectl get svc
```

如下：

```text
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP          39h
kubia        NodePort    10.106.149.92   <none>        8080:31922/TCP   12h
```

查看详情：

```text
sudo kubectl describe svc kubia
```

如下：

```text
Name:                     kubia
Namespace:                default
Labels:                   run=kubia
Annotations:              <none>
Selector:                 run=kubia
Type:                     NodePort
IP:                       10.106.149.92
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  31922/TCP
Endpoints:                172.17.0.2:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

![kubia-run](https://github.com/tangheng1995/tangheng1995.github.io/blob/master/img/in-post/post-js-version/2019-09-28-kubia-run.png?raw=true)
