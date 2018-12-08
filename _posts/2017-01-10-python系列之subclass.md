---
published: true
layout: post
title: python-系列之subclass
category: python
tags:
  - python
time: '2017.01.10 22:09:00'
excerpt: >-
  在学习openstack性能工具rally时，有一个核心思想--pluinIn：任何组件都已pluginIn的方式合入。看似如此高大上的思想，来源仅仅是python-object中的__subclass__(),乘此机会简单介绍一下subclass以及plugin思想。
---
&emsp;&emsp;在学习openstack性能工具rally时，有一个核心思想--pluinIn：任何组件都已pluginIn的方式合入。看似如此高大上的思想，来源仅仅是python-object中的_subclasses_(),乘此机会简单介绍一下subclasses以及plugin思想。

<!--more-->

## subclasses()
&emsp;&emsp;在众多面向对象语言中，`继承`、`多态`、`重载`已然成为了标配。python作为一门支持多种`编程范式`的语言，显然也支持这些‘标配’，而且还多了个‘高配’--subclasses()。
```
class Base(object):
    """It is a core class

     Every subclasses's base class is this class.
    """
    pass

class Son(Base):
    """Base class's son class.

    """
    pass


class GrandSon(Son):
    """Base class's son class.

    """
    pass


if __name__ == "__main__":
    print Son.__subclasses__()
    print Base.__subclasses__()
```
&emsp;&emsp;通过调用类方法subclasses()我们就轻而易举的得到所有的儿子类。上面的demo输出结果如下所示：
```
[<class '__main__.GrandSon'>]
[<class '__main__.Son'>]
```
&emsp;&emsp;但通过该方法去得到所有的子孙类，会有一个问题：只能得到脚本内的子类，但是无法得到所有跨脚本、跨模板的子类。该问题可以参考：[有关python-subclass的一个问题贴](http://stackoverflow.com/questions/3862310/how-can-i-find-all-subclasses-of-a-class-given-its-name).
## import module
&emsp;&emsp;在第一节中，我们简单的介绍了python class中的subclasses()方法，通过该方法我们可以得到一个脚本内的所有子类函数，但是无法实现跨模块or脚本加载子类，这个就需要模块加载功能。
我们将上面那个demo代码改为：
```
plugin
  |
  |------common
  |      |------__init__.py
  |      |------base.py
  |------extend
  |      |------__init__.py
  |      |------extend.py
```
&emsp;&emsp;我们将Base类添加到base.py并将Son和GrandSon类移动到extend.py时，我们通过执行
```
print Base.__subclass__（）
```
无法获得Base类的子类。
这时我们可以通过加载模块的方式添加plugin模块，这样我们就可以得到Base类在plugin模块下的子类了。
添加模块代码如下所示：
```
def import_modules_from_package(package):
    """Import modules from package and append into sys.modules

    :param package: Full package name. For example: plugin.extend
    """
    path = [os.path.dirname(plugin.__file__), ".."] + package.split(".")
    path = os.path.join(*path)
    for root, dirs, files in os.walk(path):
        for filename in files:
            if filename.startswith("__") or not filename.endswith(".py"):
                continue
            new_package = ".".join(root.split(os.sep)).split("....")[1]
            module_name = "%s.%s" % (new_package, filename[:-3])
            print module_name
            if module_name not in sys.modules:
                __import__(module_name)
                sys.modules[module_name]
```
在base.py中做一个简单示范：
```
if __name__ == '__main__':
    //将plugin.extend模块引入
    import_modules_from_package('plugin.extend')
    print Base.__subclass__()
```
即可将Base在plugin.extend模块下的子类都输出出来。若在软件架构中，我们也可以调用`import_modules_from_package(xxx)`来实现插件式的代码管理。
## 一个简单的demo
&emsp;&emsp;笔者已将代码demo归档到[点这里，点这里](https://github.com/shihai1991/python-plugin)。大家如想运行demo的话，请执行/python-plugin/main/main.py,并且笔者已将加载模块功能封装到/python-plugin/plugin/common/discovery.py中。
