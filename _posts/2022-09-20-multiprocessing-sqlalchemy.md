---
layout: post
title: 多线程调用sqlalchemy内存占用高
category: python
catalog: true
published: true
tags:
  - python
time: '2022.09.18 09:07:00'
---
# 一、问题背景
服务组件中有多线程对数据库进行查询操作，当数据表数据规模超过10K+以上，会发现多线程通过`sqlalchemy`进行查询结束后的服务进程内存占用一直较高无法对使用内存进行回收。

# 二、问题分析及定位
## 2.1 问题触发代码和执行
能触发多线程调用sqlalchemy内存占用持续走高的代码如下所示，数据库表可以连接自己的数据库服务，为了更好观测内存占比走高的情况，表内数据建议超过10K+。
```python
import os
import time
from multiprocessing.dummy import Pool

from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import create_engine
from sqlalchemy import Column
from sqlalchemy import Integer
from sqlalchemy import String
from sqlalchemy import Text
from sqlalchemy.orm import sessionmaker


# 数据库请填写自己的数据库地址
engine = create_engine('mysql+pymysql://name:pwd@ip:port/db')
Session = sessionmaker(engine)
my_pool = Pool(10)
Base = declarative_base()


# 数据库只是示例，可以选择任意自己的数据库表，出现内存快速上升现象的数据库表数据建议超10K+
class UserModel(Base):
    __tablename__ = 'users'
    id = Column(Integer, primary_key=True)
    name = Column(String(255), nullable=False)
    sex = Column(String(255), nullable=False)
    address = Column(Text(), nullable=False)
    age = Column(Integer())
    @property
    def json(self):
        ret = {c.name: getattr(self, c.name) for c in self.__table__.columns}
        return ret


def host_list():
    session = Session()
    print('listing host')
    result = session.query(UserModel).all()
    print(f'result len:{len(result)}')
    session.close()
    return result


def handle():
   my_pool.apply_async(host_list)


for _ in range(10):
    handle()


while True:
    time.sleep(1)
```
python的三方依赖件清单，使用的python版本为3.10。
```
Package    Version
---------- -------
greenlet   1.1.3
pip        20.2.3
psycopg2   2.9.3
PyMySQL    1.0.2
setuptools 47.1.0
SQLAlchemy 1.3.24

```

持续观测到的进程内存消耗(`top -p 进程ID`)情况如下所示：
```
PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
6221 root      20   0 1400680 240112   5780 S 180.0  3.0   0:26.16 python

PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
6221 root      20   0 1884144 724680   5780 S 102.7  9.0   1:13.01 python

PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
6221 root      20   0 2251728 1.051g   5780 S 101.7 13.8   2:26.43 python

PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
6221 root      20   0 2246808 1.178g   5780 S 101.0 15.4   4:07.04 python

PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
6221 root      20   0 2245240 1.254g   5780 S 101.0 16.4   5:38.18 python

PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
6221 root      20   0 1328580 399960   5804 S   0.0  5.0   8:41.12 python
```

在控制台上的打印输出：
```
$ python leak.py
listing host
listing host
listing host
listing host
listing host
listing host
listing host
listing host
listing host
listing host
result len:100000
result len:100000
result len:100000
result len:100000
result len:100000
result len:100000
result len:100000
result len:100000
result len:100000
result len:100000
```

## 2.2 问题定位
### 2.2.1 多线程是否存在内存泄露或者应用管理不当？
#### 2.2.2.1 线程池Threadpool
将测试代码中的`host_list()`函数内容直接改成`paas`后在继续执行，进程的内存占用始终保持在一个数值，因此初步可以认为多线程和内存升高无直接关联。
```
PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
8691 root      20   0 1188260  28632   5672 S   0.0  0.4   0:00.33 python
```
`Threadpool`继承自`Pool`，和`Pool`有区别的是`Process()`静态函数被覆写。
```
@staticmethod
def Process(ctx, *args, **kwds):
    from .dummy import Process
    # Process实际是dummy.DummyProcess，是一个线程子类
    return Process(*args, **kwds)
```
我们继续看一下进程内对象的引用情况，修改上述问题代码的`while True`循环为如下代码：
```
while True:
    time.sleep(60)
    print('total refcount:')
    print(sys.gettotalrefcount())
```
对象的引用变化输出如下所示。
```
total refcount:
14121337
total refcount:
25266164
total refcount:
31223932
total refcount:
32043789
total refcount:
29213107
total refcount:
27321546
total refcount:
20589612
total refcount:
453416
```
将`while True`循环上方补充资源池的释放后再观察引用变量和内存占用的情况。
```
my_pool.close()

while True:
    time.sleep(60)
    print('total refcount:')
    print(sys.gettotalrefcount())
```
实际最终的进程内对象引用计数总量比不释放资源池（引用总数共453416）低1026。
```
total refcount:
452390
```
将`host_list()`函数的返回体改为`paas`后的进程内对象引用计数和资源池释放后的对象引用计数（引用计数总共452390）相比低6580。
```
total refcount:
445810
```
进程的内存最终占用情况比初始值比略高，但低于资源池未释放的进程最终内存占用（初始内存开销：160536KB，释放资源池后最终内存开销：328144KB，不释放资源池最终内存开销：399960KB）
```
PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
30213 root      20   0 1293916 328144   5804 S   0.0  4.1   8:30.66 python
```
从这个比对测试可以得出一个观测结论：**进程池资源释放能降低引用计数和内存占用，但最终的内存开销比初始进程内存占用大。**  
在进行一个主线程做数据库查询(`host_list()`)的内存占用观测和线程池规模为一的数据库查询(`host_list()`)做比对分析，发现**实际进程占用的`RES`内存规模和使用线程池时的内存开销关联不大**。
```
# 主线程内做host_list查询(不使用pymalloc)
PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
708 root      20   0  407960 214716   5644 S   0.0  2.7   0:13.27 python
# 主线程内做host_list查询，UserModel删除address和age字段(不使用pymalloc)
PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
6245 root      20   0  371508 178260   5640 S   0.0  2.2   0:11.11 python
# 线程池规模为1的host_list查询(不使用pymalloc)
PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
3406 root      20   0  656032 215800   5780 S   0.0  2.7   0:13.15 python
```

#### 2.2.2.2 进程池Pool
在继续看一下多进程资源池的[实现逻辑](https://github.com/shihai1991/cpython/blob/9a34d853d7ad2e2f52dcd5d7fef5773a1dc98868/Lib/multiprocessing/pool.py)，实际`Pool`对象中对任务进行处理的主要是三个函数：
- _handle_workers：根据资源池规模创建多进程/线程并启动进程/线程，从`_inqueue`队列获取task，执行完task后放入`_outqueue`队列；
- _handle_tasks：对`taskqueue`队列中的task进行管理（暂时没看出有啥实际作用）；
- _handle_results：从`_outqueue`队列中获取任务并刷新`ApplyResult`结果并删除自身在pool缓存中的记录信息（`pool._cache[job]`）。  
从上面对cpython线程/进程pool的分析看pool本身不太可能出现内存泄露，那只能再看看sqlalchemy对内存的开销管理情况。

### 2.2.2 sqlalchemy对内存的开销管理有问题？
查看了stackoverflow的一些FAQ，发现sqlalchemy作者做了[比较准确的解释](https://stackoverflow.com/questions/7389759/memory-efficient-built-in-sqlalchemy-iterator-generator)：`Most DBAPI implementations fully buffer rows as they are fetched - so usually, before the SQLAlchemy ORM even gets a hold of one result, the whole result set is in memory`。
这个结合上面多线程/进程的执行输出就能解释这个情况了。在多进程查询过程中，实际在前期所有线程/进程都一直在和数据库建立连接和查询数据，所以数据集都存在内存中，当查询陆续完成后，内存的开销也就不断下降。
![sqlalchemy framework](https://www.researchgate.net/profile/Taban-Habibu-2/publication/337018547/figure/fig7/AS:821734068473857@1572928077238/SQLAlchemy-Dependencies-Layers.ppm)

# 三、解决办法
## 3.1 数据库查询优化
### 3.1.1 [yield_per()](https://docs.sqlalchemy.org/en/14/orm/query.html?highlight=yield_per#sqlalchemy.orm.Query.yield_per)
当查询结果较大时，可以通过调用`yield_per()`函数来批量查询结果，这样就避免python解释器开辟较大的内存区。

### 3.2.2 [rangeQuery](https://github.com/sqlalchemy/sqlalchemy/wiki/RangeQuery-and-WindowedRangeQuery)

### 3.2.3 sqlalchemy内存泄漏（待确定）
一个10K+查询操作，进程内存占用从30MB提升到180MB+，不太可能全都是因为python内存管理机制导致，[sqlalchemy是否有内存泄漏，继续移步至社区开展讨论](https://github.com/sqlalchemy/sqlalchemy/discussions/8542)。  
从测试结果看，单线程或者多线程循环查询后内存不会无限提升，因此sqlalchemy本身不太可能存在内存泄露。为什么sqlalchemy多线程并发查询后缓存不会全都释放的原因，参看sqlalchemy的maintainer回复结果：
```
That's unrelated to threading. If you open multiple connection some of them will be kept open in the pool, since a queue pool is used when connecting to most databases by sqlalchemy.
```

## 3.2 内存管理机制探究
上面执行进程从最初的240112KB到最后的399960KB，实际从测试结果看应该由三部分机制构成。

### 3.2.1 Python GC机制
资源池未关闭导致相关对象引用计数持续存在，gc未能所有内存开销。
建议：**如果内存对象引用计数一时半会分析不清楚，可以使用多进程进行查询，因为多进程内python资源是独立的，进程完成处理后所有资源都会被销毁**。

### 3.2.2 python内存管理机制
如果所有内存都需要python和OS进行内存申请和释放，这个过程是比较耗时的。因此python对内存管理做了优化，调用`[pymalloc](https://docs.python.org/3/c-api/memory.html#pymalloc)`时就会调用arena区和pool池对小于512bytes的小对象内存使用进行了优化，确保能从arena区中申请小对象内存（到了本人知识盲区地带，没太正儿八经对python解释器对内存的管理，大家可以先看参考文献1，有时间我再补充刷新）。
将`pymalloc`内存分配器关闭后（通过环境变量来控制 `export PYTHONMALLOC=malloc`）在执行初始测试代码发现最终实际进程内存占用量高了5倍+（399960KB->2.150g）。
```
PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
16342 root      20   0 3088992 2.150g   5776 S   0.0 28.2   7:33.13 python
```

### 3.2.3 线程/进程的创建及管理
线程和进程的创建本身调用了系统的接口，线程/进程本身有自己持有的内存开销，相关线程/进程的[相关占用内存就会被释放](https://docs.python.org/3/library/multiprocessing.html#multiprocessing.pool.ThreadPool)。

# 五、参考文献
1.[Memory Management in Python](https://www.honeybadger.io/blog/memory-management-in-python/)  
2.[Releasing memory in Python](https://stackoverflow.com/questions/15455048/releasing-memory-in-python)  
3.[Multiprocessing -- Thread Pool Memory Leak?](https://stackoverflow.com/questions/51272951/multiprocessing-thread-pool-memory-leak)  
4.[pmap high memory usage by python script](https://stackoverflow.com/questions/29348245/pmap-high-memory-usage-by-python-script)
5.[https://github.com/sqlalchemy/sqlalchemy/issues/4616](https://github.com/sqlalchemy/sqlalchemy/issues/4616)
