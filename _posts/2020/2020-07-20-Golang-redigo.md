---
layout: post
title: "Golang redis 使用"
date: "2020-07-20 12:00:00"
category: golang
tags: golang
author: Archer
---
* content
{:toc}

## 介绍

Golang redis 使用 及阿里redis开发规范。

- github.com/gomodule/redigo/redis




## Golang redis使用

使用 gomodule/redigo/redis 库处理redis数据库相关操作。

### 安装

```text
go get github.com/gomodule/redigo/redis
```

### 使用

```go
package cache

import (
	"time"

	"github.com/gomodule/redigo/redis"
)

// Pool redis conn pool
var Pool *redis.Pool

// Redis 结构体
type Redis struct {
	Conn redis.Conn
}

// InitPool 初始化redis conn pool
func InitPool() {
	pool = &redis.Pool{
		MaxActive:   500,
		MaxIdle:     3,
		IdleTimeout: 240 * time.Second,
		Dial:        func() (redis.Conn, error) { return redis.Dial("tcp", "127.0.0.1:6379") },
	}
    Pool = pool
    RedisConn = pool.Get()
}

// GetRedisConn get redis conn
func GetRedisConn() redis.Conn{ 
	conn = Pool.Get()
	return &Redis{Conn: conn}
}

// AddStudentCourse 添加学生选修的课程
func (r *Redis)AddStudentCourse(stdID int, cIDs []int) (err error){
	defer r.Conn.Close() 
	for _, cID := range cIDs {
		r.Conn.Send("SADD", fmt.Sprintf(StudentCourseFormat, stdID),cID)
		r.Conn.Send("SADD", fmt.Sprintf(CourseStudentFormat, cID),stdID)
	}
	return r.Conn.Flush()
}

// GetStudentCourseCount 学生选修的课程数
func (r *Redis)GetStudentCourseCount(stdID int) (courseCount int){
	defer r.Conn.Close() 
	return redis.Int(r.Conn.Do("SCARD", fmt.Sprintf(StudentCourseFormat, stdID)))
}
```

### 开发规范

#### 键值设计

##### key名设计

- 可读性和可管理性

以业务名(或数据库名)为前缀(防止key冲突)，用冒号分隔，比如业务名:表名:id

```text
ugc:video:1
```

- 简洁性

保证语义的前提下，控制key的长度，当key较多时，内存占用也不容忽视，例如：

```text
user:{uid}:friends:messages:{mid}

# 简化
u:{uid}:fr:m:{mid}
```

- 不要包含特殊字符

反例：包含空格、换行、单双引号以及其他转义字符

##### value设计

- 拒绝bigkey(防止网卡流量、慢查询)

string类型控制在10KB以内，hash、list、set、zset元素个数不要超过5000。

反例：一个包含200万个元素的list。

非字符串的bigkey，不要使用del删除，使用hscan、sscan、zscan方式渐进式删除，同时要注意防止bigkey过期时间自动删除问题(例如一个200万的zset设置1小时过期，会触发del操作，造成阻塞，而且该操作不会不出现在慢查询中(latency可查))

- 选择适合的数据类型。

例如：实体类型(要合理控制和使用数据结构内存编码优化配置,例如ziplist，但也要注意节省内存和性能之间的平衡)

反例：

```text
set user:1:name tom
set user:1:age 19
set user:1:favor football
```

正例:

```text
hmset user:1 name tom age 19 favor football
```

##### 控制key的生命周期，redis不是垃圾桶

建议使用expire设置过期时间(条件允许可以打散过期时间，防止集中过期)，不过期的数据重点关注idletime。

#### 命令使用

##### O(N)命令关注N的数量

例如hgetall、lrange、smembers、zrange、sinter等并非不能使用，但是需要明确N的值。有遍历的需求可以使用hscan、sscan、zscan代替。

##### 禁用命令

禁止线上使用keys、flushall、flushdb等，通过redis的rename机制禁掉命令，或者使用scan的方式渐进式处理。

##### 合理使用select

redis的多数据库较弱，使用数字进行区分，很多客户端支持较差，同时多业务用多数据库实际还是单线程处理，会有干扰。

##### 使用批量操作提高效率

```text
原生命令：例如mget、mset。
非原生命令：可以使用pipeline提高效率。
```

但要注意控制一次批量操作的元素个数(例如500以内，实际也和元素字节数有关)。

注意两者不同：

```text
1. 原生是原子操作，pipeline是非原子操作。
2. pipeline可以打包不同的命令，原生做不到
3. pipeline需要客户端和服务端同时支持。
```

##### Redis事务功能较弱，不建议过多使用

Redis的事务功能较弱(不支持回滚)，而且集群版本(自研和官方)要求一次事务操作的key必须在一个slot上(可以使用hashtag功能解决)

##### Redis集群版本在使用Lua上有特殊要求

- 1.所有key都应该由 KEYS 数组来传递，redis.call/pcall 里面调用的redis命令，key的位置，必须是KEYS array, 否则直接返回error，"-ERR bad lua script for redis cluster, all the keys that the script uses should be passed using the KEYS array"

- 2.所有key，必须在1个slot上，否则直接返回error, "-ERR eval/evalsha command keys must in same slot"

##### 必要情况下使用monitor命令时，要注意不要长时间使用

### 示例

```go
package main

import (
	"fmt"
	"time"

	"github.com/gomodule/redigo/redis"
)

const (
	// StudentCourseFormat 学生选修的课程数
	StudentCourseFormat = "student:%d:course"
	// CourseStudentFormat 课程被选修的学生数
	CourseStudentFormat = "course:%d:student"
)

// Pool redis conn pool
var Pool *redis.Pool

// Redis 结构体
type Redis struct {
	Conn redis.Conn
}

// InitRedis init redis
func InitRedis() {
	pool = &redis.Pool{
		MaxActive:   500,
		MaxIdle:     3,
		IdleTimeout: 240 * time.Second,
		Dial:        func() (redis.Conn, error) { return redis.Dial("tcp", "127.0.0.1:10100") },
	}
	Pool = pool
}

// GetRedisConn get redis conn
func GetRedisConn() redis.Conn{ 
	conn = Pool.Get()
	return &Redis{Conn: conn}
}

// AddStudentCourse 添加学生选修的课程
func (r *Redis)AddStudentCourse(stdID int, cIDs []int) (err error){
	defer r.Conn.Close() 
	for _, cID := range cIDs {
		r.Conn.Send("SADD", fmt.Sprintf(StudentCourseFormat, stdID),cID)
		r.Conn.Send("SADD", fmt.Sprintf(CourseStudentFormat, cID),stdID)
	}
	return r.Conn.Flush()
}

// GetStudentCourseCount 学生选修的课程数
func (r *Redis)GetStudentCourseCount(stdID int) (courseCount int){
	defer r.Conn.Close() 
	return redis.Int(r.Conn.Do("SCARD", fmt.Sprintf(StudentCourseFormat, stdID)))
}

// GetCourseStudentCount 获取选修的课程学生数
func (r *Redis)GetCourseStudentCount(cID int) (studentCount int){
	defer r.Conn.Close() 
	return redis.Int(r.Conn.Do("SCARD", fmt.Sprintf(CourseStudentFormat, cID)))
}

func main(){
	r := GetRedisConn()
	r.AddStudentCourse(1, [1,2,3,4])
	fmt.Println(r.GetStudentCourseCount(1))
}

```

### 参考

- [1] [阿里云redis开发规范](https://developer.aliyun.com/article/531067)
