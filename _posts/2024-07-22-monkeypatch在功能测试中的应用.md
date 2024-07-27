---
layout: post
title: Monkey Patch在功能性测试中的应用
category: software engineering
catalog: true
published: false
tags:
  - software engineering
  - test
time: '2024.07.22 10:32:00'
---

# 背景
Monkey Patch可以让我们在运行时对类或模块进行修改。在测试阶段，Monkey Patch是一种接管部分被测代码运行环境的简单方法，并将输入依赖和输出依赖替换为更方便进行测试的对象或函数。
通过[Wiki描述]((https://en.wikipedia.org/wiki/Monkey_patch)，Monkey Patch这个词可能有两种来源：
- 来源于Guerilla Patch(杂牌军Patch)，它指的是偷偷地修改代码。因为Guerilla发音很像Gorilla(猩猩)，后面改成Monkey(猴子)可能是因为让他听起来不是那么吓人；
- 指的是Monkeying about(捣鼓)；

对系统进行测试隔离的对象类别：
- Mock对象：通过JMock等三方工具创建而来，并能设置预期(Expect)值；
- Fake对象：需要自己手动创建和实现，有完整的实现逻辑，但不适用于现网运行；
- Stub（桩）对象：和Fake对象类似，需要自己手动创建和实现，实现可能会很简单，一个函数可能就只有一个`return`语句；
- Dummy对象：用来作为被测系统的数据填充；

# 参考文档
- [Martin Fowler: Mocks Aren't Stubs](https://martinfowler.com/articles/mocksArentStubs.html)
- [Python Docs: unittest.mock.patch](https://docs.python.org/3/library/unittest.mock.html#unittest.mock.patch)
- [Pytest Mock](https://github.com/pytest-dev/pytest-mock)
- [TestContainer](https://testcontainers.com/guides/introducing-testcontainers/)
- [TestContainer python](https://testcontainers-python.readthedocs.io/en/latest/)
- [Testing Like a Pro: A Step-by-Step Guide to Python’s Mock Library](https://www.kdnuggets.com/testing-like-a-pro-a-step-by-step-guide-to-pythons-mock-library)
- [Using Mocks to Test External Dependencies or Reduce Duplication](https://www.obeythetestinggoat.com/book/chapter_mocking.html)
- [Pytest: Using monkeypatch](http://f.javier.io/rep/books/python-testing-with-pytest.pdf)
- [Monkey Patch](https://en.wikipedia.org/wiki/Monkey_patch)
- [Testing in Python: Dependency Injection vs. Mocking](https://betterprogramming.pub/testing-in-python-dependency-injection-vs-mocking-5e542783cb20)
- [Testing with Mocks, Fakes, and Monkeypatching in Go](https://shanehowearth.com/testing-with-mocks-fakes-and-monkeypatching-in-go/)
- [Advanced Testing Techniques in Python: A Deep Dive into Monkeypatching, Mocking, and Pytest](https://blog.stackademic.com/advanced-testing-techniques-in-python-a-deep-dive-into-monkeypatching-mocking-and-pytest-ebd9264a69eb)
- [Testing Conferences
](https://github.com/TestingConferences/testingconferences.github.io/blob/master/_data/current.yml?spm=a2c6h.12873639.article-detail.21.adea3195btld4x&file=current.yml)
- [evelent monkey_patch()](https://github.com/eventlet/eventlet/blob/8bac9b2bb5ba02d42305446327a117ff51af177b/eventlet/patcher.py#L226)
- [软件测试面临的挑战与发展趋势](http://ckjs.ijournals.cn/uploadfile/news_images/ckjs/2020-02-28/%E5%A4%A7%E5%AE%B6%E8%AE%BA%E5%9D%9B%EF%BC%88%E6%9C%B1%E5%B0%91%E6%B0%9120200119%EF%BC%89.pdf)
- [Monkey Patching in Java](https://www.baeldung.com/java-monkey-patching)
- [What's the difference between faking, mocking, and stubbing?](https://stackoverflow.com/questions/346372/whats-the-difference-between-faking-mocking-and-stubbing)
