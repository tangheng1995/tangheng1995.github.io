---
layout: post
title: "Golang redis 实践 性能测试及优化"
date: "2020-06-06 12:00:00"
category: golang
tags: golang
author: Archer
---
* content
{:toc}

## 介绍

Golang redis 实践 性能测试及优化。

本文简单测试redis连接开启关闭连接的性能情况，以及连接池性能情况。




## Golang redis实践 性能测试及优化

本次实践测试，是由于公司项目里大量redis开启关闭写在service层，一是感觉代码重复较多，二是感觉反复开启关闭连接会造成性能损耗，就简单demo测试。

package:

```text
github.com/garyburd/redigo/redis
```

### redis测试

```go
package cache

import (
	"fmt"

	"github.com/garyburd/redigo/redis"
)

// RedisConn RedisConn
var RedisConn redis.Conn

// Init Init
func Init() {
	c, err := redis.Dial("tcp", "127.0.0.1:6379")
	if err != nil {
		fmt.Println("Connect to redis error", err)
		return
	}
	RedisConn = c
	//defer c.Close()
}

// SetTest SetTest
func SetTest(c redis.Conn) {
	_, err := c.Do("SET", "mykey", "superWang")
	if err != nil {
		fmt.Println("redis set failed:", err)
	}
}

// GetTest1 GetTest1
func GetTest1() {
	c, err := redis.Dial("tcp", "127.0.0.1:6379")
	if err != nil {
		fmt.Println("Connect to redis error", err)
		return
	}
	defer c.Close()
	username, err := redis.String(c.Do("GET", "mykey"))
	if err != nil {
		fmt.Println("redis get failed:", err)
	} else {
		fmt.Printf("Get mykey: %v \n", username)
	}
}

// GetTest2 GetTest2
func GetTest2() {
	username, err := redis.String(RedisConn.Do("GET", "mykey"))
	if err != nil {
		fmt.Println("redis get failed:", err)
	} else {
		fmt.Printf("Get mykey: %v \n", username)
	}
}
```

`GetTest1()` 每次调用开启关闭连接

`GetTest2()` 则使用全局创建的连接

测试Demo

```go
package test

import (
	"benchmark/cache"
	"testing"
)

func BenchmarkGetTest1(b *testing.B) {
	for i := 0; i < b.N; i++ { // b.N，测试循环次数
		cache.GetTest1()
	}
}

func BenchmarkGetTest2(b *testing.B) {
	cache.Init()
	for i := 0; i < b.N; i++ { // b.N，测试循环次数
		cache.GetTest2()
	}
}

```

执行：

```text
go test -v -bench=. -cpu=8 -benchtime="3s" -timeout="5s" -benchmem
```

- benchmem：输出内存分配统计
- benchtime：指定测试时间
- cpu：指定GOMAXPROCS
- timeout：超时限制

输出：

```text
# GetTest1()
# 反复开启关闭连接

BenchmarkGetTest1-8         6477            504956 ns/op            9428 B/op         28 allocs/op
PASS
ok      branchmark/test 3.336s
```

```text
# GetTest2()

BenchmarkGetTest2-8        24621            142355 ns/op              96 B/op          5 allocs/op
PASS
ok      branchmark/test 4.994s
```

- BenchmarkGetTest1-8 ：-cpu参数指定，-8表示8个CPU线程执行
- 6477：表示总共执行了6477次
- 504956 ns/op ：表示每次执行耗时504956纳秒
- 9428 B/op:表示每次执行分配的内存（字节）
- 28 allocs/op：表示每次执行分配了多少次对象

对比发现，反复开启关闭连接耗时增加了3.5倍，分配内存增加了98倍，分配对象增加了5.6倍。

结论： 建议不要反复开启关闭redis连接

### redis 连接池

以上，不反复开启关闭连接性能更好，但是在高并发情况下，并不适用，刚好查看文档，发现有 [连接池](https://pkg.go.dev/github.com/garyburd/redigo/redis?tab=doc#Pool)

```go
package cache

import (
	"time"

	"github.com/garyburd/redigo/redis"
)

// Pool redis conn pool
var Pool *redis.Pool

// InitPool 初始化redis conn pool
func InitPool() {
	pool = &redis.Pool{
		MaxActive:   500,
		MaxIdle:     3,
		IdleTimeout: 240 * time.Second,
		Dial:        func() (redis.Conn, error) { return redis.Dial("tcp", "127.0.0.1:6379") },
	}
	Pool = pool
}
```

调用：

```go
conn := cache.Pool.Get()
defer conn.Close()
```

此时 `defer conn.Close()` 则不是关闭连接，而是释放连接到连接池

测试：

```go
package cache

import (
	"fmt"
	"time"

	"github.com/garyburd/redigo/redis"
)

// Pool redis conn pool
var Pool *redis.Pool

// InitPool 初始化redis conn pool
func InitPool() {
	pool := &redis.Pool{
		MaxIdle:     3,
		IdleTimeout: 240 * time.Second,
		Dial:        func() (redis.Conn, error) { return redis.Dial("tcp", "127.0.0.1:6379") },
	}
	Pool = pool
}

// GetTest3 GetTest3
func GetTest3() {
	conn := Pool.Get()
	defer conn.Close()
	username, err := redis.String(conn.Do("GET", "mykey"))
	if err != nil {
		fmt.Println("redis get failed:", err)
	} else {
		fmt.Printf("Get mykey: %v \n", username)
	}
}

```

```go
package test

import (
	"benchmark/cache"
	"testing"
)

func BenchmarkGetTest3(b *testing.B) {
	cache.InitPool()
	for i := 0; i < b.N; i++ { // b.N，测试循环次数
		cache.GetTest3()
	}
}
```

输出：

```text
# Pool

BenchmarkGetTest3-8        24814            145063 ns/op             208 B/op          8 allocs/op
PASS
ok      benchmark/test  5.082s
```

对比发现，执行时间差不多，分配内存增加2倍，分配对象增加1.6倍，但是由单连接变成连接池，高并发场景下性能还是提升很大

### 引用

- [1] [redis连接池](https://pkg.go.dev/github.com/garyburd/redigo/redis?tab=doc#Pool)
