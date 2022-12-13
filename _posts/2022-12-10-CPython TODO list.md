---
layout: post
title: CPython TODO List
category: cpython
catalog: true
published: false
tags:
  - cpython
time: '2022.12.10 12:52:00'
---
# 一、TODO List

## 1.1 简单知识点
- python template渲染能力不如go templates，支持简单语法看是不是可以支持一下？
- 【done】help()函数的差异：help(b'123')没问题，但help('123')不行的原因是字符串可能是代码某一模块功能(参见serhiy-storchaka的MR)[https://github.com/python/cpython/commit/1c205518a35939ef555c74d0e2f8954a5e1828e1]。


## 1.2 解释器、编译器加速
- 微软的faster cpython计划：https://github.com/faster-cpython/ideas/issues/218
- Eric Snow的想法：(issue 40512)[https://bugs.python.org/issue40512 per-interpreter] (multi-core-python)[https://github.com/ericsnowcurrently/multi-core-python/issues/34] (issue 40514)[https://bugs.python.org/issue40514]
- mark shan的想法：(faster cpython)[https://github.com/markshannon/faster-cpython/blob/master/plan.md]
- sam cross对多线程优化的总结及POC：优化内存创建，refcount算法加强等方面入手 (nogil)[https://github.com/colesbury/nogil]
 (google docs)[https://docs.google.com/document/d/18CXhDb1ygxg-YXNBJNzfzZsDFosB5e6BfnXLlejd9l0/edit#]

## 1.3 GIL细粒度化

## 1.4 函数性能优化
- frozen.c：70-98行，有多个反复的函数申明，是否要优化？

## 1.5 API优化
- nb_add需要补充相关架构方案：https://www.python.org/dev/peps/pep-0573/
- PEP补充说明：https://github.com/encukou/abi3/issues/19

## 1.6 风格类
- 大量的xx_Finalize()太丑，看是不是需要有个完善的析构函数方式？
- Py_IS_TYPE()不能使用Py_TYPE()?这个可以作为一个easy问题给新人提交？

## 1.7 变量优化
- 静态变量转换为动态变量：https://bugs.python.org/issue40077 https://bugs.python.org/issue40601 https://bugs.python.org/issue46417

## 1.8 子编译器
subinterp victor更新了一个问题
subinterpreter相关工作
https://pyfound.blogspot.com/2021/05/the-2021-python-language-summit_16.html
https://vstinner.github.io/isolate-subinterpreters.html
https://bugs.python.org/issue40601
https://bugs.python.org/issue45476
https://bugs.python.org/issue45490
https://bugs.python.org/issue44133
https://bugs.python.org/issue39511 共享实例需要转换为子编译器化，这个需要写PEP的支持，先放一放；


# 二、博客待输出
https://bugs.python.org/issue45021 可以写一篇博客
这个case可以写一篇作用域的博客，这里的考察点是b在这里会提示UnboundLocalError错误；
b = 6
def test(a):
    print(a)
    print(b)

    b = 2

test(1)
-------------------------------------------------------
def fun():
    temp = [lambda x: i*x for i in range(4)]
    return temp

for everyLambda in fun():
    print(everyLambda(2))
-------------------------------------------------------
a = [1, 2, 3, 4]
for i in a:
    a.append(i)
    a.remove(i)
这个可以写个解析题
--------------------------------------------------------
float('inf')可以写个解析题float('inf') == float('inf') - 1
---------------------------------------------------------
这个从自己码角度可以写一个
def func():
    yield (x for x in range(3))
for x in func():
    print(type(x))
    print(next(x))
------------------------------------
x = 0
y = 0

def f():
    x = 1
    y = 1
   
    class C:
      print(x, y)
      x = 2

f()
--------------------------------------------------------------------------------------------
返回值怎么创建的
def func(x):
    return x

print(func([]) is [])
-----------------------------
21. 写一篇关于module内存ref cycle的博客