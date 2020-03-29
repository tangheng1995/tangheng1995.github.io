---
layout: post
title: Python优雅编程-字典
subtitle: 'Python优雅编程'
date:       2019-07-28 12:00:00
author:     "Archer"
header-img: ""
categories: python
tags:
    - python
---

## Python优雅编程-字典

### 字典取键值

如果目标key值不存在字典中，以下代码返回None或default

```python
name_age = {"Jack":26, "Tim":25}
jack_age = name_age.get("Jack")
print(jack_age)
```

```text
Output: 26
```

### 根据字典的值进行排序

```python
name_age = {"Jack":26, "Tim":25, "Tom":27}
sort_name_age = sorted(name_age.items(), key=lambda x:x[1], reverse=True)
print(sort_name_age)
```

```text
Output: [("Tom", 27), ("Jack", 26), ("Tim", 25)]
```

### 初始化字典值为列表

```python
from collections import defaultdict
name_age_dict = defaultdict(list)
name_age_dict["name"].append("Jack")
print(name_age_dict)
```

```text
Output: defaultdict(<type 'list'>, {'name': ['Jack']})
```

### 将list中的所有元素转为单个字符串

```python
name_list = ["Jack", "Tim"]
name_str = ",".join(name_list)
print(name_str)
```

```text
Output: Jack,Tim
```

### key-value对构建字典

```python
name_list = ["Jack", "Tim"]
age_lsit = [26, 25]
name_age_dict = dict(zip(name_list, age_lsit))
print(name_age_dict)
```

```text
Output: {"Jack": 26, "Tim": 25}
```

### 静态方法staticmethod

Python中用staticmethod修饰的函数属于类级别的静态方法，可以不用实例化直接调用，主要定义辅助函数、公用函数

```python
from collections import defaultdict

class Demo():
    @staticmethod
    def init_dict():
        name_age_dict = defaultdict(list)
        return name_age_dict


print(Demo.init_dict())
```

```text
Output: defaultdict(<type 'list'>, {})
```
