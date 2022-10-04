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
```

## 通过set类型创建实例
```python
my_set = set()
```
## 

# 参考文献
1. [datastructures](https://docs.python.org/3/tutorial/datastructures.html#sets)
2. [stdtypes](https://docs.python.org/3/library/stdtypes.html#set)