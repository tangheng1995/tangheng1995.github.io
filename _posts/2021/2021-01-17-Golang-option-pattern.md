---
layout: post
title: "Golang Option 编程模式"
date: "2021-01-17 12:00:00"
category: golang
tags: golang,pattern
author: Archer
---
* content
{:toc}

## 介绍

Functional Options 这个编程模式。这是一个函数式编程的应用案例，编程技巧也很好，是目前 Go 语言中最流行的一种编程模式




## 配置选项问题

我们经常需要对一个对象（或是业务实体）进行相关的配置。比如下面这个业务实体（注意，这只是一个示例）：

```go
type Server struct {
    Addr     string
    Port     int
    Protocol string
    Timeout  time.Duration
    MaxConns int
    TLS      *tls.Config
}
```

* Addr,Port 必填，不为空

* Protocol 、 Timeout 和MaxConns 字段，这几个字段是不能为空的，但是有默认值的，比如，协议是 TCP，超时30秒 和 最大链接数1024个。

* TLS ，这个是安全链接，需要配置相关的证书和私钥。这个是可以为空的。

## Functional Options

```go
package main

import (
	"crypto/tls"
	"fmt"
	"time"
)

// Server Server信息
type Server struct {
	Addr     string
	Port     int
	Protocol string
	Timeout  time.Duration
	MaxConns int
	TLS      *tls.Config
}

// Option Option
type Option func(*Server)

// Protocol 配置协议
func Protocol(p string) Option {
	return func(s *Server) {
		s.Protocol = p
	}
}

// Timeout 配置超时时间
func Timeout(timeout time.Duration) Option {
	return func(s *Server) {
		s.Timeout = timeout
	}
}

// MaxConns 配置最大连接数
func MaxConns(maxconns int) Option {
	return func(s *Server) {
		s.MaxConns = maxconns
	}
}

// TLS 配置TLS加密
func TLS(tls *tls.Config) Option {
	return func(s *Server) {
		s.TLS = tls
	}
}

// NewServer 创建Server实例
func NewServer(addr string, port int, options ...func(*Server)) (*Server, error) {
	srv := Server{
		Addr:     addr,
		Port:     port,
		Protocol: "tcp",
		Timeout:  30 * time.Second,
		MaxConns: 1000,
		TLS:      nil,
	}
	for _, option := range options {
		option(&srv)
	}
	return &srv, nil
}

func main() {
	s, _ := NewServer("0.0.0.0", 8080, Timeout(300*time.Second), MaxConns(1000))
	fmt.Println(s)
}
```

Functional Options 这种方式，这种方式至少带来了 6 个好处：

1. 直觉式的编程；
2. 高度的可配置化；
3. 很容易维护和扩展；
4. 自文档；
5. 新来的人很容易上手；
6. 没有什么令人困惑的事（是 nil 还是空）。

## grpc Functional Options

```go
package main

import (
	"fmt"
	"time"
)

// Server 服务端
type Server struct {
	host string
	port int
	opts Options
}

// Options 服务端配置项
type Options struct {
	timeout time.Duration
	ssl     bool
	maxConn int
}

// Option 配置接口
type Option interface {
	apply(*Options)
}

type funcOption struct {
	f func(*Options)
}

func (fo *funcOption) apply(o *Options) {
	fo.f(o)
}

func newFuncOption(f func(*Options)) *funcOption {
	return &funcOption{
		f:f,
	}
}

// WithTimeout 配置服务端超时时间
func WithTimeout(t time.Duration) Option {
	return newFuncOption(func(o *Options){
		o.timeout = t
	})
}

// WithSSL 配置服务端SSL开关
func WithSSL(f bool) Option {
	return newFuncOption(func(o *Options){
		o.ssl = f
	})
}

// WithMaxConn 配置服务端最大连接数
func WithMaxConn(m int) Option {
	return newFuncOption(func(o *Options){
		o.maxConn = m
	})
}

// defaultOptions 服务端默认配置
func defaultOptions() Options {
	return Options{
		timeout: 30 * time.Second,
		ssl:     false,
		maxConn: 200,
	}
}

// NewServer 创建服务端实例
func NewServer(host string, port int, opts ...Option) (server *Server,err error){
	s := &Server{
		host: host,
		port: port,
		opts: defaultOptions(),
	}

	for _, opt := range opts {
		opt.apply(&s.opts)
	}

	return s,nil
}

func main() {
	s,_ := NewServer("localhost", 8000,WithTimeout(10),WithSSL(true),WithMaxConn(100))
	fmt.Println(s)
}
```

## 引用

* [1] [Go 编程模式：Functional Options](https://time.geekbang.org/column/article/330212)
