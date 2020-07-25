---
layout: post
title: "Python优雅编程-列表元组常见用法"
date: "2019-07-28 12:00:00"
category: python
tags: python
author: Archer
---
* content
{:toc}

## 介绍

Python优雅编程之列表元组常见用法。




## Python优雅编程-列表元组常见用法

### 列表/元组拆解

```python
# 常见用法
L = ["Tim", "Boy", "27"]
name = L[0]
sex = L[1]
age = L[2]
print(name, sex, age)
```

```python
# 优雅用法
name, sex, age = ["Tim", "Boy", "27"]
print(name, sex, age)
```

```text
Output: Tim Boy 27
```

### 使用操作符in

```python
# 常见用法
name, sex, age = ["Tim", "Boy", "27"]
if name == "Tim" or name == "Tom":
    print(name)
```

```python
# 优雅用法
name, sex, age = ["Tim", "Boy", "27"]
if name in ["Tim", "Tom"]:
    print(name)
```

```text
Output: Tim
```

### 字符串连接join

```python
# 常见用法
L = ["Tim", "Boy", "27"]
S = ""
for s in L:
    S += s
print(S)
```

```python
# 优雅用法
L = ["Tim", "Boy", "27"]
S = "".join(L)
print(S)
```

```text
Output: TimBoy27
```

### 字典键值判断

```python
# 常见用法
D = {"name": "Tim", "sex": "Boy", "age": "27"}
if D.has_key("age"):
    print(D["age"])
```

```python
# 优雅用法
D = {"name": "Tim", "sex": "Boy", "age": "27"}
if "age" in D:
    print(D["age"])
```

```text
Output: 27
```

### 真假判断

```python
# 常见用法
l = ["Tim", "Boy", "27"]
if len(l) != 0:
    print(l)
if l != []:
    print(l)
```

```python
# 优雅用法
l, n = ["Tim", "Boy", "27"], []
if l:
    print(l)
if not n:
    print(n)
```

```text
Output: 
    ["Tim", "Boy", "27"]
    []
```

### 遍历列表及索引

```python
# 常见用法
l = ["Tim", "Boy", "27"]
for i in range(len(l)):
    print(i, l[i])
```

```python
# 优雅用法
l = ["Tim", "Boy", "27"]
for index, value in enumerate(l):
    print(index, value)
```

```text
Output:
    0 Tim
    1 Boy
    2 27
```

### 嵌套列表推导

```python
# 常见用法
l = ["Tim", "Boy", ["A", "B"]]
for i in l:
    if isinstance(i, list):
        for item in i:
            print(item)
```

```python
# 优雅用法
l = ["Tim", "Boy", ["A", "B"]]
n = [item for i in l for item in i if isinstance(i, list)]
for i in n:
    print(i)
```

```text
Output: A B
```

### 循环嵌套

```python
# 常见用法
m1 = ["Tim", "Boy"]
n1 = ["Tom", "Boy"]
for m in m1:
    for n in n1:
        print(m, n)
```

```python
# 优雅用法
from itertools import product

m1 = ["Tim", "Boy"]
n1 = ["Tom", "Boy"]
for m,n in product(m1, n1):
    print(m, n)
```

```text
Output:
    Tim Tom
    Tim Boy
    Boy Tom
    Boy Boy
```

### 条件的列表推导

```python
num = [15, 65, 27, 40, 90]
print(n for n in num if n > 20 and n % 2 == 0)
```

```text
Output: [40, 90]
```

```python
num = [15, 65, 27, 40, 90]
print([n if n >= 20 else n*2 for n in num])
```

```text
Output: [30, 68, 27, 40, 90]
```
