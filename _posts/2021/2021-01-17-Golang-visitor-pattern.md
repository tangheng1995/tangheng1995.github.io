---
layout: post
title: "Golang Visitor 编程模式"
date: "2021-01-17 12:00:00"
category: golang
tags: golang,pattern
author: Archer
---
* content
{:toc}

## 介绍

Visitor 是面向对象设计模式中一个很重要的设计模式,这个模式是将算法与操作对象的结构分离的一种方法。这种分离的实际结果是能够在不修改结构的情况下向现有对象结构添加新操作，是遵循开放 / 封闭原则的一种方法。




## 简单实例

* 代码中有一个Visitor的函数定义，还有一个Shape接口，这需要使用 Visitor函数作为参数。

* 实例的对象 Circle和 Rectangle实现了 Shape 接口的 accept() 方法，这个方法就是等外面给我们传递一个 Visitor。

```go
package main

import (
    "encoding/json"
    "encoding/xml"
    "fmt"
)

type Visitor func(shape Shape)

type Shape interface {
    accept(Visitor)
}

type Circle struct {
    Radius int
}

func (c Circle) accept(v Visitor) {
    v(c)
}

type Rectangle struct {
    Width, Heigh int
}

func (r Rectangle) accept(v Visitor) {
    v(r)
}

func JsonVisitor(shape Shape) {
    bytes, err := json.Marshal(shape)
    if err != nil {
        panic(err)
    }
    fmt.Println(string(bytes))
}

func XmlVisitor(shape Shape) {
    bytes, err := xml.Marshal(shape)
    if err != nil {
        panic(err)
    }
    fmt.Println(string(bytes))
}

func main() {
  c := Circle{10}
  r :=  Rectangle{100, 200}
  shapes := []Shape{c, r}

  for _, s := range shapes {
    s.accept(JsonVisitor)
    s.accept(XmlVisitor)
  }

}
```

实现两个 Visitor：一个是用来做 JSON 序列化的；另一个是用来做 XML 序列化的。这段代码的目的就是想解耦数据结构和算法。虽然使用 Strategy 模式也是可以完成的，而且会比较干净，但是在有些情况下，多个 Visitor 是来访问一个数据结构的不同部分，这种情况下，数据结构有点像一个数据库，而各个 Visitor 会成为一个个的小应用。 kubectl就是这种情况。

## kubectl Visitor实现

```go
package main

import "fmt"

// Visitor 对外暴露的接口
type Visitor interface {
	Visit(VisitorFunc) error
}

// Visit 对外暴露的接口的实现方法
func (info *Info) Visit(fn VisitorFunc) error {
	return fn(info, nil)
}

// VisitorFunc 真实装载进接口方法里的实现
type VisitorFunc func(*Info, error) error

// Info 需要被操作的对象
type Info struct {
	Namespace   string
	Name        string
	OtherThings string
}

// LogVisitor 多态实现Visitor接口
type LogVisitor struct {
	visitor Visitor
}

// Visit 多态实现isitor接口方法
func (v LogVisitor) Visit(fn VisitorFunc) error {
	return v.visitor.Visit(func(info *Info, err error) error {
		fmt.Println("LogVisitor() before call function")
		err = fn(info, err)
		fmt.Println("LogVisitor() after call function")
		return err
	})
}

func main() {
    info := Info{}
    
    // 将声明的Info对象抽象赋值给Visitor
    var v Visitor = &info
    
    // 再将Visitor赋值给具体处理逻辑的LogVisitor对象
	v = LogVisitor{v}

    // 待装载给Visit方法的真实实现，类型为VisitorFunc类型
	loadFile := func(info *Info, err error) error {
		info.Name = "TestLog"
		info.Namespace = "LogNamespace"
		info.OtherThings = "log..."
		return err
	}

    // 具体实现loadFile装载进Visit执行
	v.Visit(loadFile)
	fmt.Println(info)
}

```

* 有一个 VisitorFunc 的函数类型的定义；

* 一个 Visitor 的接口，其中需要 Visit(VisitorFunc) error 的方法（这就像是我们上面那个例子的 Shape ）；

* 为Info 实现 Visitor 接口中的 Visit() 方法，实现就是直接调用传进来的方法（与前面的例子相仿）。

* 声明了一个 LogVisitor 的结构体，这个结构体里有一个 Visitor 接口成员，这里意味着多态；

* 在实现 Visit() 方法时，调用了自己结构体内的那个 Visitor的 Visitor() 方法，这其实是一种修饰器的模式，用另一个 Visitor 修饰了自己

## 引用

* [1] [Go 编程模式：Kubernetes Visitor模式](https://time.geekbang.org/column/article/330467)
