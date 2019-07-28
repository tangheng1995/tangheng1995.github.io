---
layout: post
title: Python优雅编程-推导式
subtitle: 'Python优雅编程'
date:       2019-07-28 12:00:00
author:     "Archer"
header-img: "img/post-bg-python.png"
categories: python
tags:
    - python
---

### Python优雅编程-推导式

#### 一、列表推导式

```python
age_list = [25,26]
age_filter = [age for age in age_list if age < 26]
print(age_filter)
```

```text
Output: [25]
```

#### 二、字典推导式

```python
name_age = {"Jack": 26, "Tim":25}
name_age_filter = {name:age for name,age in name_age.iteritems() if age<25}
print(name_age_filter)

```

```text
Output: {'Tim':25}
```

#### 三、遍历列表并输出元素索引

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

#### 四、同时遍历两个列表

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

#### 五、使用Counter进行分步统计

```python
from collections import Counter
age_list = [26, 25, 25]
distrubution = Counter(age_list)
print(distrubution)

```

```text
Output: Counter({25: 2, 26:1})
```
