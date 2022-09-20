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
服务组件中有多进程对数据库进行数据拉取和消费活动，但是当数据表超过10K+以上，会发现多进程通过`sqlalchemy`连接时存在内存占用持续走高的情况。

# 二、问题分析
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


```
# 参考文献
