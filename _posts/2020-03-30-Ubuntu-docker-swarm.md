---
layout: post
title: Ubuntu下使用docker swarm 创建集群
subtitle: Ubuntu下使用docker swarm 创建集群
date:       2020-03-30 12:00:00
author:     "Archer"
categories: 
tags:
    - ubuntu
    - docker
---

## Ubuntu下使用docker swarm 创建集群

- `docker-machine` 可在单节点上虚拟多个节点创建集群(测试使用，不建议生产)
- `docker swarm init --advertise-addr <MANAGER-IP>` 创建集群

本次测试使用多节点创建docker swarm 集群

### 创建集群

```shell
sudo docker swarm init --advertise-addr <MANAGER-IP>
```

MANAGER-IP 为我作为manager节点的IP，其他worker节点必须通过这个IP访问到该节点

```shell
$ sudo docker swarm init --advertise-addr 192.168.17.135
[sudo] password for brook:
Swarm initialized: current node (ijiiozx9qim84b7ky1k33et30) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-19wv30d5bzg9a5k60n4afvtcrhxbzek3c3zv2241s3pdbuvpzd-9ycrg2o42n3p8l5z8ga1hiz7t 192.168.17.135:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

```shell
# worker节点加入集群凭证
sudo docker swarm join --token SWMTKN-1-19wv30d5bzg9a5k60n4afvtcrhxbzek3c3zv2241s3pdbuvpzd-9ycrg2o42n3p8l5z8ga1hiz7t 192.168.17.135:2377
```

token凭证为新节点加入集群的凭证

查看token：

```shell
docker swarm join-token worker
```

```shell
# docker info 查看swarm状态
$ sudo docker info

Swarm: active
  NodeID: ijiiozx9qim84b7ky1k33et30
  Is Manager: true
  ClusterID: 04329fak6fjmxqxfmpcc0o0se
  Managers: 1
  Nodes: 1
```

```shell
# docker node ls 查看节点信息

$ sudo docker node ls

ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
ijiiozx9qim84b7ky1k33et30 *   swarm               Ready               Active              Leader              19.03.8
```

`*` 表示当前链接的节点，即我们执行操作命令的节点。

### 添加节点

主节点：192.168.17.135

子节点：

- 192.168.17.136
- 192.168.17.137

worker节点加入swarm集群

```shell
sudo docker swarm join --token SWMTKN-1-19wv30d5bzg9a5k60n4afvtcrhxbzek3c3zv2241s3pdbuvpzd-9ycrg2o42n3p8l5z8ga1hiz7t 192.168.17.135:2377
This node joined a swarm as a worker.
```

### 创建服务

在manager上创建服务

```shell
sudo docker service create --replicas 1 --name helloworld alpine ping docker.com

s05l117gvfosnct3s7us29l2i
```

- `docker service create`命令创建了一个service
- `--name` 将service命名为 `helloworld`
- `--replicas` 指定运行1个实例
- `alpine ping docker.com` 定义了service将以`Alpine Linux`为镜像的container运行，并在container内部执行命令`ping docker.com`

- `--publish`参数来暴露端口。target用来指定container内部的端口号

```shell
# docker service ls 查看服务
$ sudo docker service ls
ID            NAME        SCALE  IMAGE   COMMAND
s05l117gvfos  helloworld  1/1    alpine  ping docker.com
```

### 查看服务状态

在manager节点上执行 `docker service inspect --pretty <SERVICE-ID>` 查看service的详细信息,`docker service inspect <SERVICE-ID>` 查看json格式

```shell
$ sudo docker service inspect --pretty helloworld

ID:             s05l117gvfosnct3s7us29l2i
Name:           helloworld
Service Mode:   Replicated
 Replicas:      1
Placement:
UpdateConfig:
 Parallelism:   1
 On failure:    pause
 Monitoring Period: 5s
 Max failure ratio: 0
 Update order:      stop-first
RollbackConfig:
 Parallelism:   1
 On failure:    pause
 Monitoring Period: 5s
 Max failure ratio: 0
 Rollback order:    stop-first
ContainerSpec:
 Image:         alpine:latest@sha256:b276d875eeed9c7d3f1cfa7edb06b22ed22b14219a7d67c52c56612330348239
 Args:          ping docker.com
 Init:          false
Resources:
Endpoint Mode:  vip
```

`docker service ps helloworld` 查看service在哪些节点上运行

```shell
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
som7bkgohcv1        helloworld.1        alpine:latest       swarm               Running             Running 10 minutes ago
```

### 服务中任务副本数

集群分布式部署服务可指定服务副本数，在manager节点上执行 `docker service scale <SERVICE-ID>=<NUMBER-OF-TASKS>`

```shell
$ sudo docker service scale helloworld=5

helloworld scaled to 5
overall progress: 5 out of 5 tasks
1/5: running
2/5: running
3/5: running
4/5: running
5/5: running
```

查看服务状态：

```shell
$ sudo docker service ps helloworld

ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE                ERROR               PORTS
som7bkgohcv1        helloworld.1        alpine:latest       swarm               Running             Running 17 minutes ago
lbznumtbgtx2        helloworld.2        alpine:latest       swarm-worker1       Running             Running about a minute ago
xcu7by8nmb2j        helloworld.3        alpine:latest       swarm-worker2       Running             Running about a minute ago
px3urph3p30s        helloworld.4        alpine:latest       swarm-worker2       Running             Running about a minute ago
924k8gdu234m        helloworld.5        alpine:latest       swarm               Running             Running about a minute ago
```

### 删除服务

在manager节点上 `docker service rm <SERVICE-ID>`

```shell
$ sudo docker service rm helloworld

helloworld
```

查看服务是否被删除：

```shell
$ sudo docker service inspect helloworld
[]
Status: Error: no such service: helloworld, Code: 1
```

### 滚动更新

```shell
 $ docker service create \
   --replicas 3 \
   --name redis \
   --update-delay 10s \
   redis:alpine
```

在部署时，我们需要对升级操作的策略进行设置。

--update-delay指定每一个task升级的延时时间。例如，我们要求每个task较之前一个task开始升级的时间延时T秒，则把参数设置成Ts，s表示单位秒，m表示单位分钟，h表示单位小时。所以10m30s，表示将延时10分30秒。

默认情况下一次更新1个task，可以通过参数--update-parallelism来设置同时开始更新的最大task个数。

默认情况下当一个执行更新任务的task的状态成为RUNNING时，才会继续开始下一个task的更新，知道所有的task都成为RUNNING状态为止。一旦更新操作过程中，有一个task返回FAILED状态，则更新会立即停止。我们可以在docker service create或者docker service update命令中使用--update-failure-action参数，来控制更新失败后的操作。

manager更新任务：

```shell
$ docker service update --image redis:alpine3.11   redis

redis
```

调度更新的步骤如下：

- 停止第一个task。
- 更新第一个停止的task。
- 更新完成后将task启动起啦。
- 如果task返回RUNNING状态，在等待task更新延时时间到达后，开始更新下一个task。
- 如果task返回FAILED 状态，则停止更新操作。

### DRAIN状态的节点

DRAIN状态将阻止节点接收manager分配的任务。同时也意味着，manager将停止该节点上的task，并将task转移到其他ACTIVE状态的节点上运行。

`docker node update --availability drain <NODE-ID>` 将节点状态设置为DRAIN

```shell
$ sudo docker node update --availability drain worker1

worker1
```

`docker node update --availability active <NODE-ID>`使节点状态变回到ACTIVE

### 引用

- [1] [Swarm模式关键概念](https://www.bookstack.cn/read/docker-swarm-guides/swarmmo-shi-guan-jian-gai-nian.md)
- [2] [Swarm部署服务](https://yeasy.gitbooks.io/docker_practice/swarm_mode/deploy.html)
