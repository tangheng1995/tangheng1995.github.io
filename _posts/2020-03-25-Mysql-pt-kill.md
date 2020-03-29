---
layout: post
title: Mysql 数据库拥塞处理工具pt-kill
subtitle: Percona-Toolkit系列之pt-kill杀会话利器
date:       2020-03-25 12:00:00
author:     "Archer"
categories: 
tags:
    - mysql
---

## Mysql 数据库拥塞处理工具pt-kill

### pt-kill介绍

生产环境中我们时常遇到这样的情况，数据库性能恶劣，需要马上杀掉全部会话，不然数据库就挂起来。我们可以先找show processlist的输出来杀会话，但是比较麻烦。pt-kill为我们解决了杀会话问题。

### 常用杀会话场景

```shell
# -- 1、每10秒检查一次，发现有 Query 的进程就给干掉

# 只打印-每10秒检查一次，发现有 Query 的进程就给干掉
pt-kill --host=192.168.65.128 --port=3306 --user=root --password=rootpwd  --match-db='db222'  --match-command="Query" --busy-time 30 --victims all --interval 10 --daemonize  --print --log=/tmp/1.log

# 执行杀操作
pt-kill --host=192.168.65.128 --port=3306 --user=root --password=rootpwd  --match-db='db222'  --match-command="Query" --busy-time 30 --victims all --interval 10 --daemonize --kill --log=/tmp/kill.log
```

```shell
# -- 2、查杀select大于30s的会话

# 只打印-查杀select大于30s的会话
pt-kill --host=192.168.65.128 --port=3306 --user=root --password=rootpwd  --match-db='db222'  --match-info "select|SELECT" --busy-time 30 --victims all --interval 10 --daemonize  --print --log=/tmp/pt_select.log

# 执行杀操作-查杀select大于30s的会话
pt-kill --host=192.168.65.128 --port=3306 --user=root --password=rootpwd  --match-db='db222'  --match-info "select|SELECT" --busy-time 30 --victims all --interval 10 --daemonize --kill --log=/tmp/pt_select_kill.log
```

```shell
# -- 3、查杀某IP来源的会话

# 只打印-查杀某IP来源的会话
pt-kill --host=192.168.65.128 --port=3306 --user=root --password=rootpwd  --match-db='db222'  --match-host "192.168.65.129" --busy-time 30 --victims all --interval 10 --daemonize  --print --log=/tmp/pt_select.log

# 执行杀操作-查杀某IP来源的会话
pt-kill --host=192.168.65.128 --port=3306 --user=root --password=rootpwd  --match-db='db222'  --match-host "192.168.65.129" --busy-time 30 --victims all --interval 10 --daemonize --kill --log=/tmp/pt_select_kill.log
```

```shell
# -- 4、查杀访问某用户的会话

# 只打印-查杀访问某用户的会话
pt-kill --host=192.168.65.128 --port=3306 --user=root --password=rootpwd  --match-db='db222'  --match-user "u2" --busy-time 30 --victims all --interval 10 --daemonize  --print --log=/tmp/pt_select.log

# 执行杀操作-查杀访问某用户的会话
pt-kill --host=192.168.65.128 --port=3306 --user=root --password=rootpwd  --match-db='db222'  --match-user "u2" --busy-time 30 --victims all --interval 10 --daemonize --kill --log=/tmp/pt_select_kill.log
```

```shell
# -- 5、杀掉正在进行filesort的sql

# 只打印-杀掉正在进行filesort的sql
pt-kill --host=192.168.65.128 --port=3306 --user=root --password=rootpwd  --match-db='db222'  --match-command Query --match-state "Sorting result"  --run-time 1 --busy-time 30 --victims all --interval 10 --daemonize  --print --log=/tmp/pt_select.log

# 执行杀操作-杀掉正在进行filesort的sql
pt-kill --host=192.168.65.128 --port=3306 --user=root --password=rootpwd  --match-db='db222'  --match-command Query --match-state "Sorting result"  --run-time 1 --busy-time 30 --victims all --interval 10 --daemonize --kill --log=/tmp/pt_select_kill.log


# 只打印
pt-kill --host=192.168.65.128 --port=3306 --user=root --password=rootpwd  --match-db='db222'  --match-command Query --match-state "Creating sort index"  --run-time 1 --busy-time 30 --victims all --interval 10 --daemonize  --print --log=/tmp/pt_select.log

# 执行杀操作
pt-kill --host=192.168.65.128 --port=3306 --user=root --password=rootpwd  --match-db='db222'  --match-command Query --match-state "Creating sort index"  --run-time 1 --busy-time 30 --victims all --interval 10 --daemonize --kill --log=/tmp/pt_select_kill.log
```

### 引用

- [1] [Percona-Toolkit系列之pt-kill杀会话利器](http://www.fordba.com/percona-toolkit-pt-kill.html)
- [2] [pt-kill用法记录](https://www.cnblogs.com/bjx2020/p/9076953.html)
- [3] [pt-kill官网](https://www.percona.com/doc/percona-toolkit/LATEST/pt-kill.html)
