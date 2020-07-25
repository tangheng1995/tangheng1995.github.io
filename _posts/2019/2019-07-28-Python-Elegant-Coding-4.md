---
layout: post
title: "Python优雅编程-优化循环"
date: "2019-07-28 12:00:00"
category: python
tags: python
author: Archer
---
* content
{:toc}

## 介绍

Python优雅编程之优化循环。




## Python优雅编程-优化循环

### for循环

减少循环内部的计算，循环嵌套提取不需要的循环逻辑

优化前：

```python
a = [1, 2, 3]
b = [4, 5, 6]

for i in b:
    sum_a = sum(a)
    i += i* sum_a
    print(i)
```

优化后：

```python
a = [1, 2, 3]
b = [4, 5, 6]

sum_a = sum(a)
for i in b:
    i += i*sum_a
    print(i)
```

在命名空间中优先搜索局部变量，因此局部变量的查询效率要快，再循环中尽量引用局部变量。当在循环中需要多次引用某个变量时，尽量将其转换为局部变量

优化前：

```python
b = [4, 5, 6]

def sum_list():
    a = [1, 2, 3]
    sum_a = sum(a)
    for i in b:
        i += i*sum_a
        print(i)
```

优化后：

```python
def sum_list():
    a = [1, 2, 3]
    b = [4, 5, 6]
    sum_a = sum(a)
    for i in b:
        i += i*sum_a
        print(i)
```

多层for嵌套循环中，遵从外小内大

优化前：

```python
A = [1, 2, 3]
B = [4, 5, 6, 7]

for b in B:
    for a in a:
        pass
```

优化后：

```python
A = [1, 2, 3]
B = [4, 5, 6, 7]

for a in A:
    for b in B:
        pass
```

异常处理写在循环外面

在for循环中字符串的拼接，使用join比"+"更节省内存

### 使用with自动关闭资源

在文件读写处理过程中可能出错，可使用with控制流语句在处理文件出错的情况下也能关闭文件，它简化了try/finally操作，使用with...as...
不需要手动关闭文件资源

```python
with open('demo.txt', 'r') as readfile:
    readfile.readlines()
```
