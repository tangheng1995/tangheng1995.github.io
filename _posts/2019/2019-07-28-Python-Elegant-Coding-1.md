---
layout: post
title: "Python优雅编程-推导式"
date: "2019-07-28 12:00:00"
category: python
tags: python
author: Archer
---
* content
{:toc}

## 介绍

Python优雅编程之推导式。




## Python优雅编程-推导式

### 列表推导式

```python
age_list = [25,26]
age_filter = [age for age in age_list if age < 26]
print(age_filter)
```

```text
Output: [25]
```

### 字典推导式

```python
name_age = {"Jack": 26, "Tim":25}
name_age_filter = {name:age for name,age in name_age.iteritems() if age<25}
print(name_age_filter)
```

```text
Output: {'Tim':25}
```

### 遍历列表并输出元素索引

```python
name_list = ["Jack", "Tim"]
for index, name in enumerate(name_list):
    print(index, name)
```

```text
Output:
    0 Jack
    1 Tim
```

### 同时遍历两个列表

```python
name_list = ["Jack", "Tim"]
age_list = [26, 25]
for name, age in zip(name_list, age_list):
    print("name:{0} age:{1}".format(name, age))
```

```text
Output:
    name: Jack age:26
    name: Tim age:25
```

### 使用Counter进行分步统计

```python
from collections import Counter
age_list = [26, 25, 25]
distrubution = Counter(age_list)
print(distrubution)
```

```text
Output: Counter({25: 2, 26:1})
```
