---
layout: post
title: Golang无缓冲channel和有缓冲channel
subtitle: Golang无缓冲channel和有缓冲channel
date:       2020-01-16 12:00:00
author:     "Archer"
categories: 
tags:
    - golang
    - channel
---

### Golang无缓冲channel和有缓冲channel

#### channel的定义

channel是一个对应make创建的底层数据结构的引用。
当我们复制一个channel或用于函数参数传递时，我们只是拷贝了一个channel引用，因此调用者和被调用者将引用同一个channel对象。和其它的引用类型一样，channel的零值也是nil。
定义一个channel时，也需要定义发送到channel的值的类型。channel可以使用内置的make()函数来创建：
我们先来看一段代码

##### 无缓冲channel

```text
package main

import "fmt"

func main(){
    ch:=make(chan int) //这里就是创建了一个channel，这是无缓冲管道注意
    go func(){                  //创建子go程
        for i:=0;i<6;i++{
            ch<-i     //循环写入管道
            fmt.Println("写入",i)
        }
    }()

    for i:=0;i<6;i++{      //主go程
        num:=<-ch         //循环读出管道
        fmt.Println("读出",num)
    }
}

```

输出：

```text
写入 0
读出 0
读出 1
写入 1
写入 2
读出 2
读出 3
写入 3
写入 4
读出 4
读出 5

```

先创建了一个匿名函数的子go程，和main的主go程一起争夺cpu，但是我们在里面创建了一个管道，无缓冲管道有一个规则那就是必须读写同时操作才会有效果，所以上述输出是成对出现，如果只进行读或者只进行写那么会被阻塞，被暂时停顿等待另外一方的操作，在这里我们定义了一个容量为0的通道，这就是无缓冲通道，如下图

![无缓冲channel]()

无缓冲通道就是这样，一次只能传输一个数据
总结一下就是无缓冲特性：
同一时刻，同时有 读、写两端把持 channel。
如果只有读端，没有写端，那么 “读端”阻塞。
如果只有写端，没有读端，那么 “写端”阻塞。
读channel： <- channel
写channel： channel <- 数据

##### 有缓冲channel

```text
package main

import "fmt"

func main(){
    ch:=make(chan int,1) //这里就是创建了一个channel，这是有缓冲管道注意
    go func(){                  //创建子go程
        for i:=0;i<6;i++{
            ch<-i     //循环写入管道
            fmt.Println("写入",i)
        }
    }()

    for i:=0;i<6;i++{      //主go程
        num:=<-ch         //循环读出管道
        fmt.Println("读出",num)
    }
}

```

输出：

```text
写入 0
写入 1
读出 0
读出 1
读出 2
写入 2
写入 3
写入 4
读出 3
读出 4
读出 5

```

![有缓冲channel]()

在第 1 步，右侧的 goroutine 正在从通道接收一个值。
在第 2 步，右侧的这个 goroutine独立完成了接收值的动作，而左侧的 goroutine 正在发送一个新值到通道里。
在第 3 步，左侧的goroutine 还在向通道发送新值，而右侧的 goroutine 正在从通道接收另外一个值。这个步骤里的两个操作既不是同步的，也不会互相阻塞。
最后，在第 4 步，所有的发送和接收都完成，而通道里还有几个值，也有一些空间可以存更多的值。
有缓冲通道就是图中所示，一方可以写入很多数据，不用等对方的操作，而另外一方也可以直接拿出数据，不需要等对方写，但是注意一点（如果写入的一方把channel写满了，那么如果要继续写就要等对方取数据后才能继续写入，这也是一种阻塞，读出数据也是一样，如果里面没有数据则不能取，就要等对方写入），而有缓冲channel定义也比较简单

```text
ch:=make(chan int,10)//chan int 只能写入读入int类型的数据，10代表容量为10.
```

总结起来就是：
channel 中自带缓冲区。创建时可以指定缓冲区的大小。
w：直到缓冲区被填满后，写端才会阻塞。
r：缓冲区被读空，读端才会阻塞。
len：代表缓冲区中，剩余元素个数，
cap：代表缓冲区的容量
