---
layout: post
title: "Python优雅编程-类型校验"
date: "2019-07-28 12:00:00"
category: python
tags: python
author: Archer
---
* content
{:toc}

## 介绍

Python优雅编程之类型校验。




## Python优雅编程-类型校验

### isinstance校验类型

因为基于内建类型扩展的用户自定义类型，type()不能准确返回结果，因此在校验对象类型的时候，建议使用isinstance()函数，两者异同如下：

- type()用于获取一个未知对象的数据类型，isinstance()用于判断一个对象是否是已知类型
- type()不认为子类的实例化对象属于父类类型，isinstance()认为子类的实例化对象属于父类类型，即子类对象也属于父类对象

两者都可以检查某个变量是否属于某一数据类型

```python
num = 20
print(isinstance(num, type(num)))
```

```text
Output: True
```

两则都可以检查实例化对象是否属于某个类

```python
class Demo():
    num = 20

d = Demo()
print(isinstance(d, type(d)))
```

```text
Output: True

```

isinstance()能够判断出子类的实例化对象属于父类，但type()不能

```python
class Father():
    n = 10

class Son(Father):
    m = 20

f = Father()
s = Son()

print(isinstance(s, Father), type(s) == Father)
```

```text
Output: True False
```

### 使用literal_eval 替代 eval

eval()函数字符串str当成有效的表达式来求值并返回计算结果，其中包含如下三个参数：

- source : 必填，一个python表达式的字符串或者compile()返回的代码对象
- globals : 非必填，必须是dict
- locals : 非必填，任何映射对象，默许与globals参数一致

eval()函数将任何字符串当做表达式来处理，如果globals，locals参数都不指定，表达式将在eval调用的环境中执行，即默认为globals()和
locals()函数中包含的所有模块和函数。这就使得eval()存在一定的风险

```python
print(eval("__import__('os').system ('dir')"))
```

```text
Output:

2019/07/27  21:18    <DIR>          .
2019/07/27  21:18    <DIR>          ..
2019/07/26  19:32                66 .gitignore
2019/07/28  15:11    <DIR>          .idea
2019/07/26  19:32               700 404.html
2019/07/27  21:18             6,809 about.html
2019/07/26  19:32    <DIR>          css
2019/07/26  19:32             1,322 feed.xml
2019/07/26  19:32    <DIR>          fonts
2019/07/26  19:32             2,204 Gruntfile.js
2019/07/28  09:50    <DIR>          img
2019/07/26  20:22             1,240 index.html
2019/07/26  19:32    <DIR>          js
2019/07/26  19:32    <DIR>          less
2019/07/26  19:32            11,538 LICENSE
2019/07/26  19:32               644 package.json
2019/07/26  19:47                26 README.md
2019/07/26  19:32             2,178 tags.html
2019/07/26  20:57             2,721 _config.yml
2019/07/26  21:24    <DIR>          _includes
2019/07/26  21:57    <DIR>          _layouts
2019/07/28  15:10    <DIR>          _posts
```

ast.literal_eval()方法会判断需要计算的内容计算后是否是安全、合法的python类型，如果是则进行运算，否则会直接抛出异常

```python
import ast

print(ast.literal_eval()("__import__('os').system ('dir')"))
```

```text
Output:

Traceback (most recent call last):
  File "<input>", line 1, in <module>
TypeError: literal_eval() missing 1 required positional argument: 'node_or_string'
```

因此推荐使用ast.literal_eval()替换eval()

### str()和repr()的区别

str()面向用户，返回可读性强的字符串类型
repr()面向解释器或开发人员，返回python解释器内部含义

解释器中输入string默认调用repe()，而print("xxx")默认调用str()

repr()返回值一般可用eval(还原对象)：
obj= eval(repr(obj))

```python
string = '[1, 2, 3, 4]'
print(isinstance(string, str), string, repr(string))
```

```text
Output: True [1, 2, 3, 4] '[1, 2, 3, 4]'
```

### 判断对象是否为空

python中，None，空列表[]，空字典{}，空元组()，0 等一系列代表空和无的对象会被转换成False，除此之外的其他对象会被转化成True

```python
if [] or {} or () or None or 0 :
    print(False)
else:
    print(True)
```

```text
Output: True
```
