---
layout: post
title: python set用法解析
category: python
catalog: true
published: true
tags:
  - python
time: '2022.09.30 17:33:00'
---
# Set介绍
Set是一个没有重复元素的无序集合。

# 创建Set集合的若干方式

## 通过大括号创建Set
通过大括号可以创建出一个`set`实例，但要注意的是如果没有集合元素，则`my_set = {}`是一个空的字典实例。
```python
my_set = {'a', 'b', 'c'}
my_set = {c for c in 'abracadabra' if c not in 'abc'}
```

## 通过set类型创建set实例
```python
my_set = set()
```

# FAQ

## Q：my_set = set('abc')创建set实例后，my_set的值是{'abc'}吗？
my_set的最后内容是`{'c', 'a', 'b'}`，为什么不是'abc'呢，因为set类的初始化入参是一个可迭代对象，而字符串就是一个可迭代对象，迭代的对象就是字符串里面的字符。`my_list = list('abc')`也和这个set('abc')原理相同。

# 参考文献
1. [datastructures](https://docs.python.org/3/tutorial/datastructures.html#sets)
2. [stdtypes](https://docs.python.org/3/library/stdtypes.html#set)
