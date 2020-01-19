---
layout: post
title: Golang堆栈
subtitle: Golang堆栈
date:       2019-10-04 12:00:00
author:     "Archer"
categories: 
tags:
    - golang
    - heap
    - stack
---

### Golang堆栈

#### 什么是堆栈

在计算机中堆栈的概念分为：数据结构的堆栈和内存分配中堆栈。

##### 数据结构的堆栈

堆：堆可以被看成是一棵树，如：堆排序。在队列中，调度程序反复提取队列中第一个作业并运行，因为实际情况中某些时间较短的任务将等待很长时间才能结束，或者某些不短小，但具有重要性的作业，同样应当具有优先权。堆即为解决此类问题设计的一种数据结构。

栈：一种先进后出的数据结构。

这里着重讲的是内存分配中的堆和栈。

##### 内存分配中的堆和栈

栈（操作系统）：由操作系统自动分配释放 ，存放函数的参数值，局部变量的值等。其操作方式类似于数据结构中的栈。

堆（操作系统）： 一般由程序员分配释放， 若程序员不释放，程序结束时可能由OS回收，分配方式倒是类似于链表。

#### 堆栈缓存方式

栈使用的是一级缓存， 他们通常都是被调用时处于存储空间中，调用完毕立即释放。

堆则是存放在二级缓存中，生命周期由虚拟机的垃圾回收算法来决定（并不是一旦成为孤儿对象就能被回收）。所以调用这些对象的速度要相对来得低一些。

#### 堆栈跟踪

下面讨论堆栈跟踪信息以及如何在堆栈中识别函数所传递的参数。

以下测试案例的版本是Go 1.11

示例：

```text

package main

import "runtime/debug"

func main() {
   slice := make([]string, 2, 4)
   Example(slice, "hello", 10)
}
func Example(slice []string, str string, i int) {
   debug.PrintStack()
}
```

列表1是一个简单的程序， main函数在第5行调用Example函数。Example函数在第9行声明，它有三个参数，一个字符串slice,一个字符串和一个整数。它的方法体也很简单，只有一行，debug.PrintStack（），这会立即产生一个堆栈跟踪信息:

```text

goroutine 1 [running]:
runtime/debug.Stack(0x1, 0x0, 0x0)
    C:/Go/src/runtime/debug/stack.go:24 +0xae
runtime/debug.PrintStack()
    C:/Go/src/runtime/debug/stack.go:16 +0x29
main.Example(0xc000077f48, 0x2, 0x4, 0x4abd9e, 0x5, 0xa)
    D:/gopath/src/example/example/main.go:10 +0x27
main.main()
    D:/gopath/src/example/example/main.go:7 +0x79
```

堆栈跟踪信息：

第一行显示运行的goroutine是id为 1的goroutine。

第二行 debug.Stack()被调用

第四行 debug.PrintStack() 被调用

第六行 调用debug.PrintStack()的代码位置，位于main package下的Example函数。它也显示了代码所在的文件和路径，以及debug.PrintStack()发生的行数(第10行)。

第八行 也调用Example的函数的名字，它是main package的main函数。它也显示了文件名和路径，以及调用Example函数的行数。

下面主要分析 传递Example函数传参信息

```text
// Declaration
main.Example(slice []string, str string, i int)
// Call to Example by main.
slice := make([]string, 2, 4)
Example(slice, "hello", 10)
// Stack trace
main.Example(0x2080c3f50, 0x2, 0x4, 0x425c0, 0x5, 0xa)
```

上面列举了Example函数的声明，调用以及传递给它的值的信息。当你比较函数的声明以及传递的值时，发现它们并不一致。函数声明只接收三个参数，而堆栈中却显示6个16进制表示的值。理解这一点的关键是要知道每个参数类型的实现机制。

让我们看第一个[]string类型的参数。slice是引用类型，这意味着那个值是一个指针的头信息(header value)，它指向一个字符串。对于slice,它的头是三个word数，指向一个数组。因此前三个值代表这个slice。

```text
// Slice parameter value
slice := make([]string, 2, 4)
// Slice header values
Pointer:  0xc00006df48
Length:   0x2
Capacity: 0x4
// Declaration
main.Example(slice []string, str string, i int)
// Stack trace
main.Example(0xc00006df48, 0x2, 0x4, 0x4abd9e, 0x5, 0xa)
```

显示了0xc00006df48代表第一个参数[]string的指针，0x2代表slice长度，0x4代表容量。这三个值代表第一个参数。

![图一](https://github.com/tangheng1995/tangheng1995.github.io/blob/master/img/in-post/post-js-version/2019-10-04-1.png?raw=true)

```text
// String parameter value
“hello”
// String header values
Pointer: 0x4abd9e
Length:  0x5
// Declaration
main.Example(slice []string, str string, i int)
// Stack trace
main.Example(0xc00006df48, 0x2, 0x4, 0x4abd9e, 0x5, 0xa)
```

显示堆栈跟踪信息中的第4个和第5个参数代表字符串的参数。0x4abd9e是指向这个字符串底层数组的指针，0x5是"hello"字符串的长度，他们俩作为第二个参数。

![图二](https://github.com/tangheng1995/tangheng1995.github.io/blob/master/img/in-post/post-js-version/2019-10-04-2.png?raw=true)

第三个参数是一个整数，它是一个简单的word值

```text
// Integer parameter value
10
// Integer value
Base 16: 0xa
// Declaration
main.Example(slice []string, str string, i int)
// Stack trace
main.Example(0xc00006df48, 0x2, 0x4, 0x4abd9e, 0x5, 0xa) 
```

显示堆栈中的最后一个参数就是Example声明中的第三个参数，它的值是0xa，也就是整数10。

![图三](https://github.com/tangheng1995/tangheng1995.github.io/blob/master/img/in-post/post-js-version/2019-10-04-3.png?raw=true)

##### Methods

接下来让我们稍微改动一下程序，让Example变成方法。

```text
package main

import (
   "fmt"
   "runtime/debug"
)

type trace struct{}

func main() {
   slice := make([]string, 2, 4)
   var t trace
   t.Example(slice, "hello", 10)
}
func (t *trace) Example(slice []string, str string, i int) {
   fmt.Printf("Receiver Address: %p\n", t)
   debug.PrintStack()
}
```

上例在第8行新增加了一个类型trace，在第15将example改变为trace的pointer receiver的一个方法。第12行声明t的类型为trace，第13行调用它的方法。

因为这个方法声明为pointer receiver的方法，Go使用t的指针来支持receiver type，即使代码中使用值来调用这个方法。当程序运行时，堆栈跟踪信息如下：

```text
Receiver Address: 0x5781c8
goroutine 1 [running]:
runtime/debug.Stack(0x15, 0xc000071ef0, 0x1)
    C:/Go/src/runtime/debug/stack.go:24 +0xae
runtime/debug.PrintStack()
    C:/Go/src/runtime/debug/stack.go:16 +0x29
main.(*trace).Example(0x5781c8, 0xc000071f48, 0x2, 0x4, 0x4c04bb, 0x5, 0xa)
    D:/gopath/src/example/example/main.go:17 +0x7c
main.main()
    D:/gopath/src/example/example/main.go:13 +0x9a
```

第7行清晰的表明方法的receiver为pointer type。方法名和报包名中间有(*trace)。第二个值得注意的是堆栈信息中方法的第一个参数为receiver的值。方法调用总是转换成函数调用，并将receiver的值作为函数的第一个参数。我们可以总堆栈信息中看到实现的细节。

##### Packing

```text
import (
   "runtime/debug"
)

func main() {
   Example(true, false, true, 25)
}
func Example(b1, b2, b3 bool, i uint8) {

   debug.PrintStack()
}
```

再次改变Example的方法，让它接收4个参数。前三个参数是布尔类型的，第四个参数是8bit无符号整数。布尔类型也是8bit表示的，所以这四个参数可以被打包成一个word，包括32位架构和64位架构。当程序运行的时候，会产生有趣的堆栈：

```text
goroutine 1 [running]:
runtime/debug.Stack(0x4, 0xc00007a010, 0xc000077f88)
    C:/Go/src/runtime/debug/stack.go:24 +0xae
runtime/debug.PrintStack()
    C:/Go/src/runtime/debug/stack.go:16 +0x29
main.Example(0xc019010001)
    D:/gopath/src/example/example/main.go:12 +0x27
main.main()
    D:/gopath/src/example/example/main.go:8 +0x30
```

可以看到四个值被打包成一个单一的值了0xc019010001

```text
// Parameter values
true, false, true, 25

// Word value
Bits    Binary      Hex   Value
00-07   0000 0001   01    true
08-15   0000 0000   00    false
16-23   0000 0001   01    true
24-31   0001 1001   19    25

// Declaration
main.Example(b1, b2, b3 bool, i uint8)

// Stack trace
main.Example(0x19010001)
```

显示了堆栈的值如何和参数进行匹配的。true用1表示，占8bit, false用0表示，占8bit,uint8值25的16进制为x19,用8bit表示。我们课哟看到它们是如何表示成一个word值的。

Go运行时提供了详细的信息来帮助我们调试程序。通过堆栈跟踪信息stack trace，解码传递个堆栈中的方法的参数有助于我们快速定位BUG。

#### 变量是堆（heap）还是堆栈（stack）

Go中的每个变量都存在，只要有对它的引用即可。实现选择的存储位置与语言的语义无关。

存储位置确实会影响编写高效的程序。如果可能，Go编译器将为该函数的堆栈帧中的函数分配本地变量。但是，如果编译器在函数返回后无法证明变量未被引用，则编译器必须在垃圾收集堆上分配变量以避免悬空指针错误。此外，如果局部变量非常大，将它存储在堆而不是堆栈上可能更有意义。

在当前的编译器中，如果变量具有其地址，则该变量是堆上分配的候选变量。但是，基础的逃逸分析可以将那些生存不超过函数返回值的变量识别出来，并且因此可以分配在栈上。

Go的编译器会决定在哪(堆or栈)分配内存，保证程序的正确性。

下面通过反汇编查看具体内存分配情况：

新建 main.go

```text
package main

import "fmt"

func main() {
   var a [1]int
   c := a[:]
   fmt.Println(c)
}
```

查看汇编代码

```text
go tool compile -S main.go
```

输出：

```text
[root@localhost example]# go tool compile -S main.go 
"".main STEXT size=183 args=0x0 locals=0x60
    0x0000 00000 (main.go:5)    TEXT    "".main(SB), $96-0
    0x0000 00000 (main.go:5)    MOVQ    (TLS), CX
    0x0009 00009 (main.go:5)    CMPQ    SP, 16(CX)
    0x000d 00013 (main.go:5)    JLS    173
    0x0013 00019 (main.go:5)    SUBQ    $96, SP
    0x0017 00023 (main.go:5)    MOVQ    BP, 88(SP)
    0x001c 00028 (main.go:5)    LEAQ    88(SP), BP
    0x0021 00033 (main.go:5)    FUNCDATA    $0, gclocals·f6bd6b3389b872033d462029172c8612(SB)
    0x0021 00033 (main.go:5)    FUNCDATA    $1, gclocals·3ea58e42e2dc6c51a9f33c0d03361a27(SB)
    0x0021 00033 (main.go:5)    FUNCDATA    $3, gclocals·9fb7f0986f647f17cb53dda1484e0f7a(SB)
    0x0021 00033 (main.go:6)    PCDATA    $2, $1
    0x0021 00033 (main.go:6)    PCDATA    $0, $0
    0x0021 00033 (main.go:6)    LEAQ    type.[1]int(SB), AX
    0x0028 00040 (main.go:6)    PCDATA    $2, $0
    0x0028 00040 (main.go:6)    MOVQ    AX, (SP)
    0x002c 00044 (main.go:6)    CALL    runtime.newobject(SB)
    0x0031 00049 (main.go:6)    PCDATA    $2, $1
    0x0031 00049 (main.go:6)    MOVQ    8(SP), AX
    0x0036 00054 (main.go:8)    PCDATA    $2, $0
    0x0036 00054 (main.go:8)    PCDATA    $0, $1
    0x0036 00054 (main.go:8)    MOVQ    AX, ""..autotmp_4+64(SP)
```

注意到有调用newobject！其中main.go:6说明变量a的内存是在堆上分配的!

修改main.go

```text
package main

func main() {
   var a [1]int
   c := a[:]
   println(c)
}
```

再查看汇编代码

```text
[root@localhost example]# go tool compile -S main.go 
\"".main STEXT size=102 args=0x0 locals=0x28
    0x0000 00000 (main.go:3)    TEXT    "".main(SB), $40-0
    0x0000 00000 (main.go:3)    MOVQ    (TLS), CX
    0x0009 00009 (main.go:3)    CMPQ    SP, 16(CX)
    0x000d 00013 (main.go:3)    JLS    95
    0x000f 00015 (main.go:3)    SUBQ    $40, SP
    0x0013 00019 (main.go:3)    MOVQ    BP, 32(SP)
    0x0018 00024 (main.go:3)    LEAQ    32(SP), BP
    0x001d 00029 (main.go:3)    FUNCDATA    $0, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
    0x001d 00029 (main.go:3)    FUNCDATA    $1, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
    0x001d 00029 (main.go:3)    FUNCDATA    $3, gclocals·9fb7f0986f647f17cb53dda1484e0f7a(SB)
    0x001d 00029 (main.go:4)    PCDATA    $2, $0
    0x001d 00029 (main.go:4)    PCDATA    $0, $0
    0x001d 00029 (main.go:4)    MOVQ    $0, "".a+24(SP)
    0x0026 00038 (main.go:6)    CALL    runtime.printlock(SB)
    0x002b 00043 (main.go:6)    PCDATA    $2, $1
    0x002b 00043 (main.go:6)    LEAQ    "".a+24(SP), AX
    0x0030 00048 (main.go:6)    PCDATA    $2, $0
    0x0030 00048 (main.go:6)    MOVQ    AX, (SP)
    0x0034 00052 (main.go:6)    MOVQ    $1, 8(SP)
    0x003d 00061 (main.go:6)    MOVQ    $1, 16(SP)
    0x0046 00070 (main.go:6)    CALL    runtime.printslice(SB)
    0x004b 00075 (main.go:6)    CALL    runtime.printnl(SB)
    0x0050 00080 (main.go:6)    CALL    runtime.printunlock(SB)
```

没有发现调用newobject，这段代码a是在堆栈上分配的。

结论：

Go 编译器自行决定变量分配在堆栈或堆上，以保证程序的正确性。

#### 引用

- [1] [Stack Traces In Go](https://www.ardanlabs.com/blog/2015/01/stack-traces-in-go.html)
- [2] [Go堆栈的理解](https://segmentfault.com/a/1190000017498101)
