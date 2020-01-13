---
layout: post
title: Go函数参数传递
subtitle: Go函数传参
date:       2020-01-13 12:00:00
author:     "Archer"
categories: 
tags:
    - golang
---

### Go函数传参

参数传递是指在程序的传递过程中，实际参数就会将参数值传递给相应的形式参数，然后在函数中实现对数据处理和返回的过程。比较常见的参数传递有：值传递，按地址传递参数或者按数组传递参数。

#### 1、常规传递

使用普通变量作为函数参数的时候，在传递参数时只是对变量值得拷贝，即将实参的值复制给变参，当函数对变参进行处理时，并不会影响原来实参的值。

例如：

```text
package main
import (
    "fmt"
)
func swap(a int, b int) {
    var temp int
    temp = a
    a = b
    b = temp
}
func main() {
    x := 5
    y := 10
    swap(x, y)
    fmt.Print(x, y)
}
```

输出结果：5 10

传递给swap的是x，y的值得拷贝，函数对拷贝的值做了交换，但却没有改变x,y的值。

#### 2、指针传递

  函数的变量不仅可以使用普通变量，还可以使用指针变量，使用指针变量作为函数的参数时，在进行参数传递时将是一个地址看呗，即将实参的内存地址复制给变参，这时对变参的修改也将会影响到实参的值。

我们还是用上面的的例子，稍作修改如下：

```text

package main
import (
    "fmt"
)
func swap(a *int, b *int) {
    var temp int
    temp = *a
    *a = *b
    *b = temp
}
func main() {
    x := 5
    y := 10
    swap(&x, &y)
    fmt.Print(x, y)
}
```

输出结果：10 5

#### 3、数组元素作为函数参数

使用数组元素作为函数参数时，其使用方法和普通变量相同，即是一个“值拷贝”。

例：

```text

package main
import (
    "fmt"
)
func function(a int) {
    a += 100
}
func main() {
    var s = [5]int{1, 2, 3, 4, 5}
    function(s[2])
    fmt.Print(s[2])
}
```

输出结果：3

可以看到将数组元素s[2]的值作为函数的实参，不管对形参做什么操作，实参都没有改变。

#### 4、数组名作为函数参数

和其他语言不同的是，go语言在将数组名作为函数参数的时候，参数传递即是对数组的复制。在形参中对数组元素的修改都不会影响到数组元素原来的值。这个和上面的类似，就不贴代码了，有兴趣的自行编写代码测试下吧。

#### 5、slice作为函数参数

在使用slice作为函数参数时，进行参数传递将是一个地址拷贝，即将底层数组的内存地址复制给参数slice。这时，对slice元素的操作就是对底层数组元素的操作。例如：

```text

package main
import (
    "fmt"
)
func function(s1 []int) {
    s1[0] += 100
}
func main() {
    var a = [5]int{1, 2, 3, 4, 5}
    var s []int = a[:]
    function(s)
    fmt.Println(s[0])
}
```

运行结果：101

#### 6、函数作为参数

在go语言中，函数也作为一种数据类型，所以函数也可以作为函数的参数来使用。例如：

```text

package main
import (
    "fmt"
)
func function(a, b int, sum func(int, int) int) {
    fmt.Println(sum(a, b))
}
func sum(a, b int) int {
    return a + b
}
func main() {
    var a, b int = 5, 6
    f := sum
    function(a, b, f)
}
```

运行结果：11

函数sum作为函数function的形参，而变量f是一个函数类型，作为function（）调用时的实参。
