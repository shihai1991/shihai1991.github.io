---
layout: post
title: 多进程调用sqlalchemy内存占用高
category: cpython
catalog: true
published: true
tags:
  - cpython
time: '2022.09.18 09:07:00'
---
# 一、问题背景
服务组件中有多进程对数据库进行数据拉取和消费活动，但是当数据表超过10K+以上，会发现多进程通过`sqlalchemy`连接时存在内存占用持续走高并导致服务组件异常。

# 二、问题分析及定位
## 2.1 问题触发代码和执行
能触发多进程调用sqlalchemy内存占用持续走高的代码如下所示，数据库表可以连接自己的数据库服务，为了更好观测内存占比走高的情况，表内数据建议超过10K+。
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

持续观测到的进程内存消耗情况如下所示：
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
### 2.2.1 多进程是否存在内存泄露或者应用管理不当？
将测试代码中的`host_list()`函数内容直接改成`paas`后在继续执行，进程的内存占用始终保持在一个量级，因此初步可以考虑多进程和内存升高无关联。
```
PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
8691 root      20   0 1188260  28632   5672 S   0.0  0.4   0:00.33 python
```
在继续看一下多进程资源池的[实现逻辑](https://github.com/shihai1991/cpython/blob/9a34d853d7ad2e2f52dcd5d7fef5773a1dc98868/Lib/multiprocessing/pool.py)，实际`Pool`对象中对任务进行处理的主要是三个函数：
- _handle_workers：根据资源池规模创建多进程并启动进程，从`_inqueue`队列获取task，执行完task后放入`_outqueue`队列；
- _handle_tasks：对`taskqueue`队列中的task进行管理（暂时没看出有啥实际作用）；
- _handle_results：从`_outqueue`队列中获取任务并刷新`ApplyResult`结果并删除自身在pool缓存中的记录信息（`pool._cache[job]`）。
从上面对cpython进程pool的分析看pool本身不太可能出现内存泄露，那只能再看看sqlalchemy对内存的开销管理情况。
### 2.2.2 sqlalchemy对内存的开销管理有问题？
查看了stackoverflow的一些FAQ，发现sqlalchemy作者做了[比较准确的解释](https://stackoverflow.com/questions/7389759/memory-efficient-built-in-sqlalchemy-iterator-generator)：`before the SQLAlchemy ORM even gets a hold of one result, the whole result set is in memory`。
这个结合上面多进程控制台的输出就能解释这个情况了。在多进程查询过程中，实际在前期所有进程都一直在和数据库建立连接和查询数据，所以数据集都存在内存中，当查询陆续完成后，内存的开销也就不断下降。
# 三、参考文献
