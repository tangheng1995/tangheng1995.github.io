---
layout: post
title: "Python优雅编程-字符串的格式化"
date: "2019-07-28 12:00:00"
category: python
tags: python
author: Archer
---
* content
{:toc}

## 介绍

Python优雅编程之字符串的格式化。




## Python优雅编程-字符串的格式化

### f-字符串(f-string)

格式化的字符串字面值（简称为f-字符串），是在字符串的开始引号之前加上一个f或F。在这样的字符串中，我们可以在花括号{}中引用变量或Python表达式。

```python
a, b = 10, 20
print(f"the sum of {a} + {b} is {a + b}")
```

```text
Output: the sum of 10 + 20 is 30
```

花括号{}里面的表达式可以有一些格式说明符，它们用来更好的控制值的格式化方式。比如下面这个例子，将浮点数保留到小数点后三位：

```python
# 这个f-字符串中的表达式，在:后面跟了限制小数点位数的格式说明符0.3f
pi = 3.1415926
print(f"pi is {pi:0.3f}")
```

```text
Output: pi is 3.142
```

```python
# 限制最小字符宽度的，可以让输出保持列对齐
persons = {'Tom': 23, 'Jack': 29, 'William': 20}
for name, age in persons.items():
    print(f'{name:7} : {age:5}')
```

```text
Output:
Tom     :    23
Jack    :    29
William :    20
```

### str.format() 方法

- 字符串里面的花括号被format方法传入的参数替换，所以，花括号的数量应该和传递给format的参数的数量保持一致，严格来说，花括号的数量不能多于
format传递的参数的数量，否则会报错

```python
print("I love {} and {}".format("Python", "Golang"))
```

```text
Output: I love Python and Golang
```

- 花括号中可以包含数字，用来表示传递给format()方法的对象的位置：

```python
print("I love {1} and {0}".format("Python", "Golang"))
```

```text
Output: I love Golang and Python
```

- 如果在format()方法中使用关键字参数，则使用参数的名称来引用它们的值：

```python
print("I love {p} and {g}".format(p = "Python", g = "Golang"))
```

```text
Output: I love Python and Golang
```

- 位置和关键字参数可以组合在一起使用

```python
print("I love {0} and {g}".format("Python", g = "Golang"))
```

```text
Output: I love Python and Golang
```

- 以给format传递一个字典和使用方括号[]来访问键来完成格式化：

```python
persons = {'Tom': 23, 'Jack': 29, 'William': 20}
print('Tom:{0[Tom]:d}, Jack:{0[Jack]}, William:{0[William]:d}'.format(persons))
```

```text
Output:Tom:23, Jack:29, William:20
```

花括号里面的0[Tom]:d的意思是，0代表传给format的第一个对象，即persons；[Tom]就是通过键来引用第一个对象中Tom对应的值，即23；:d是整数格式化
说明符，如果Tom得到值是字符串就会报错：Unknown format code 'd' for object of type 'str'

这里也可以使用**符号将字典作为关键字参数传递：

```python
print('Tom:{Tom:d}, Jack:{Jack}, William:{William:d}'.format(**persons))         
```

### 旧的字符串格式：%格式化方法

在Python 2中，使用百分号%进行格式化。但在Python3中，更推荐使用str.format()方法或f-字符串格式化。

```python
print("I love %s and %s".format("Python", "Golang"))
```

```text
Output: I love Python and Golang
```
