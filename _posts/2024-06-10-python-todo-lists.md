---
layout: post
title: Python语言Todo List
category: python
catalog: true
tags:
  - programming language
  - python
time: '2024.06.10 22:57:00'
published: false
---
# Python语法
- 这个计算过程是否存在问题
```python
>>> 10312.2-115.4-18.4
10178.400000000001
```

# 编译器
## 解释器、编译器加速
* [微软的faster cpython计划](https://github.com/faster-cpython/ideas/issues/218)
* Eric Snow的想法：(issue 40512)\[https://bugs.python.org/issue40512] (multi-core-python)\[https://github.com/ericsnowcurrently/multi-core-python/issues/34] (issue 40514)\[https://bugs.python.org/issue40514]
* mark shan的想法：(faster cpython)\[https://github.com/markshannon/faster-cpython/blob/master/plan.md]
* sam cross对多线程优化的总结及POC：优化内存创建，refcount算法加强等方面入手 (nogil)\[https://github.com/colesbury/nogil] (google docs)\[https://docs.google.com/document/d/18CXhDb1ygxg-YXNBJNzfzZsDFosB5e6BfnXLlejd9l0/edit#]

## 子解释器
* https://pyfound.blogspot.com/2021/05/the-2021-python-language-summit\_16.html
* https://vstinner.github.io/isolate-subinterpreters.html
* https://bugs.python.org/issue40601
* 【待实现】[在Limit CAPI中隐藏静态类型，需要写PEP以及demo开发](https://github.com/python/cpython/issues/84781)
* https://bugs.python.org/issue45476
* https://bugs.python.org/issue45490
* https://bugs.python.org/issue44133
* https://bugs.python.org/issue39511 共享实例需要转换为子编译器化，这个需要写PEP的支持，先放一放；
* [PEP 684: A Per-Interpreter GIL](https://discuss.python.org/t/pep-684-a-per-interpreter-gil/19583)

# 求值循环
### Eval求值过程
ceval.c:PyEval\_EvalFrameDefault()->case(MAKE\_FUNCTION)->PyFunction\_NewWithQualName()->op->vectorcall = \_PyFunction\_Vectorcall;->\_PyFunction\_Vectorcall() \_PyThread\_CurrentFrames(): interpreter有tstate\_head（有一系列tstate），每一个tstate都可能指向一个frame 往interpreter中挂tstate可以通过new\_threadstate()来添加

### PyEval\_RestoreThread这个函数是干什么用的？

# CAPI
### C API优化
* 【PEP 620挂起】[Limited C API使用](https://github.com/python/cpython/issues/85283)
* nb\_add需要补充相关架构方案：https://www.python.org/dev/peps/pep-0573/
* PEP补充说明：https://github.com/encukou/abi3/issues/19
* 【Python Capi兼容性工具】[pythoncapi-compat](https://github.com/python/pythoncapi-compat)

# 函数机制
### 函数性能优化
* frozen.c：70-98行，有多个反复的函数申明，是否要优化？

###
* 【待确认】实例的函数用类可以直接调用，是否是设计问题

# 内存管理
### 引用计算的释放顺序
释放内存先释放外层，再释放内层，避免悬挂指针

### 测试引用泄露
./python -m test -R 3:3 test\_importlib -vv

### `__traceback__` 中存在循环引用


# 模块管理
* 大量的xx\_Finalize()太丑，看是不是需要有个完善的析构函数方式？
* Py\_IS\_TYPE()不能使用Py\_TYPE()?这个可以作为一个easy问题给新人提交？

### 多模块加载
跟踪issue: https://github.com/python/cpython/issues/78878

### 模块优化

【有争议，前向兼容问题】[创建\_pydatetime模块](https://github.com/python/cpython/issues/84976) -【ing】[多模块管理，有些属性相互依赖](https://github.com/encukou/abi3/issues/19) 在本地abi3\_19分支中实现，[PEP 630](https://peps.python.org/pep-0630/#open-issues)中的第二个问题，主要思路：在`_testcapi/heaptype.c`中写一个测试用的`Type_Spec`，并扩写`nb_add`逻辑，在通过spec来判断是否能做类别判断？

* 【done】[CTypes中的CFields不应该对外暴露](https://github.com/python/cpython/issues/78878)
* 【思考中】[PEP 630还有若干开放问题待解决，解决后可以创建一个PEP](https://peps.python.org/pep-0630/#type-checking)
* 【问题待确认】\_testmultiphase\_bad\_slot\_negative.c中的PyInit\_\_testmultiphase\_bad\_slot\_negative等用例是否还有效
* module内存ref cycle循环引用分析

### 模块导入问题

xxx\_toplevel()函数申明定义在frozen.c，实际自动生成是由freeze\_modules.py来处理，deepfreeze.c是有 import \_\_hello\_\_或者import **phello\_alias**，>> \_\_hello\_\_会发现模块输出的信息是会备注（frozen）信息 调用链可以用gdb -args ./python -c "import **hello**"然后在\_Py\_get\_\_\_hello\_\_\_toplevel打断点就进来了 deepfreeze.py生成deepfreeze.c

### builtin模块的编译和导入

builtin module的编译在setup.py，初始化导入在import.c中的`create_builtin()`函数中，要导入的模块集合放到PyImport\_Inittab数组中,此数组定义在Modules/config.c中

### [PEP 630](https://peps.python.org/pep-0630)

#### [metaclass获取问题](https://peps.python.org/pep-0630/#metaclasses)

[问题的触发代码](https://github.com/CharlieZhao95/cpython/blob/42b20d517e5ca6d3f7a23fa43389eff7d972dae2/Modules/\_datetimemodule.c#L3104)。当调用`nb_add`时，当模块切换到heap type时就可能无法检索到原来历史的module块信息，具体的原因是`nb_add`会传入两个object对象，我们无法确认那个heap type是此module创建出来的。\
用上面的分支代码构建后执行`./python -m test test_datetime -v`时，会发现用例出错。\
具体的报错信息为：

```shell
test_computations (test.datetimetester.TestSubclassDateTime_Fast) ... python: Objects/typeobject.c:3757: _PyType_GetModuleByDef: Assertion `_PyType_HasFeature((PyTypeObject *)type, (1UL << 9))' failed.
Fatal Python error: Aborted
```

实际执行的用例为：

```python
def test_computations(self):
   eq = self.assertEqual
   td = timedelta

   a = td(7) # One week
   b = td(0, 60) # One minute
   c = td(0, 0, 1000) # One millisecond
   eq(a+b+c, td(7, 60, 1000))
```

实际执行出错的代码在`_datemodule`模块的`delta_add()`函数。

```
static PyObject *
delta_add(PyObject *left, PyObject *right)
{
    PyObject *result = Py_NotImplemented;
    // 这行出错了，两个入参left，right无法确定哪个是datemodule创建出来的heap type。
    _datetimemodule_state *state = find_datetimemodule_state_by_type(Py_TYPE(left));

    if (PyDelta_Check(left, state) && PyDelta_Check(right, state)) {
        /* delta + delta */
        /* The C-level additions can't overflow because of the
         * invariant bounds.
         */
        int days = GET_TD_DAYS(left) + GET_TD_DAYS(right);
        int seconds = GET_TD_SECONDS(left) + GET_TD_SECONDS(right);
        int microseconds = GET_TD_MICROSECONDS(left) +
                           GET_TD_MICROSECONDS(right);
        _datetimemodule_state *state = find_datetimemodule_state_by_type(Py_TYPE(left));
        result = new_delta(days, seconds, microseconds, 1, state);
    }

    if (result == Py_NotImplemented)
        Py_INCREF(result);
    return result;
}
```
# 并行和并发
[concurrent.futures race condition](https://bugs.python.org/issue45021) 可以分析一下实现

#### 只有CPython的多线程才有GIL
Each Python process gets its own Python interpreter and memory space so the GIL won’t be a problem.

#### subprocess中的import是否和主进程import\_modules隔离
是的，这是两个独立的内存空间和解释器

#### subprocess中import module会影响编译器级别的module管理？
发现不影响外部的进程

#### 如何根据CPU配置进程并发数量？

# python基础知识
* 文档内没有小整数缓冲池的介绍，需要补充上，因为这个外部用户已经感知到。
* python template渲染能力不如go templates，支持简单语法看是不是可以支持一下？
* 【待分析】jdk和jre是区分运行时的，python是否需要区分，比如生产环境就不能提供pdb模块？
* 【done】help()函数的差异：help(b'123')没问题，但help('123')不行的原因是字符串可能是代码某一模块功能(参见serhiy-storchaka的MR)\[https://github.com/python/cpython/commit/1c205518a35939ef555c74d0e2f8954a5e1828e1]。
* 【】mysql中length()函数用于实际存储字节长度，char\_len()用于字符串长度，在python中，length()用于字符长度，哪个用于实际存储长度？可以用`len(str.encode('utf-8'))`进行计算。但加一个char\_len()更加合适？
* 【done】[PyCField\_New()函数不应该对外部用户暴露](https://github.com/python/cpython/pull/14837)
* 【待分析】在`_testcapi/heaptype.c`中Type\_Slots中有用{0}和{0, 0}的区别是什么？
* 【待分析】函数是function类，和object是否有关系？issubclass(funcA, object)会出错合理吗？
* 【待分析】[issue 82166](https://github.com/python/cpython/issues/82166),Serhiy的建议是重构，否则会持续恶化
* 【待分析】logging模块加filter只能创建一个实例然后调用`addfilter()`函数进行添加

#### re.match和re.search区别

match是从第一个字符开始匹配

#### 子类覆写掉外层调用方法，内层调用还是调用父类的原因是为啥？

#### python中的舍入算法

银行家舍入：4舍，5看奇偶：奇进偶舍

#### 内置函数

* vars == **dict**: 当前类的属性字典
* dir(): 实例+基类属性
* locals(): 返回局部变量

#### function和method的区别

类调用function，实例调用method

#### 装饰器装饰函数

装饰器初始化只会初始化一次

#### staticmethod和classmethod的区别

需要传入cls的是classmethod，不传入的是staticmethod

#### if语句判真逻辑

if \[] 或者 if ''判断逻辑是不会判真的。

#### **init**()的使用

子类有则不会默认调用父类，子类没有则会默认调用父类

#### 作用域的调试有bug

```bash
> /home/shihai/tests/test_py/test_scope5.py(10)C()
-> print(x, y)
(Pdb) l
  5         x = 1
  6         y = 1
  7
  8         class C:
  9           import pdb;pdb.set_trace()
 10  ->       print(x, y)
 11           x = 2
 12
 13     f()
[EOF]
(Pdb) p x
0
(Pdb) p y
0
(Pdb) c
0 1
```

#### 通过type()或者type.**new**()来创建类

X = type('X', (object,), dict(a=1))

#### \_\_future\_\_的基本使用

from **future** import absolute\_import的原因是避免有调用模块和系统库同名冲突

#### \_和\_\_的使用

\_内部属性 \_\_说明内部函数，子类无法继承 **xx**，表示魔术方法，类或者对象对于某些事件会自动触发

#### 不可变对象

str、bytes、tuple均是不可变对象，list、dict、set均是可变对象，创建后无法被修改。bytes和str的`replace()`实际是创建了一个新的copy副本。看是不是不可变序列，看对象是否能修改，以及修改后的执行`id()`显示的内存地址是否发生改变，内存地址发生改变就是不可变对象。

#### id(**eq**)的执行逻辑

下面的代码输出`a`和`b`的id(**eq**)有点诡异，待确认。

```
>>> class A:
...     pass
...
>>> def b():
...     pass
...
>>> id(A.__eq__)
139834352387184
>>> id(b.__eq__)
139834332534160
>>> a = A()
>>> id(a.__eq__)
139834332529040
>>> id(a.__eq__)
139834332532240
>>> id(a.__eq__)
139834332529040
>>> id(a.__eq__)
139834332532240
>>> id(a.__eq__)
139834332529040
>>> id(b.__eq__)
139834332532240
>>> id(b.__eq__)
139834332529040
```

### 这个可以写个解析题
```
float('inf')可以写个解析题float('inf') == float('inf') - 1
```

# 测试套
### 1.2 测试用例优化

* nb\_add这个slot在`Modules/_testcapi/heaptype.c`模块中没有相关测试用例，需要补充？

# 工具
### nm的使用

用nm可以看object files里面的symbols，用strip工具可以去除符号表
