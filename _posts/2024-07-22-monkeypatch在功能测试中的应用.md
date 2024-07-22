---
layout: post
title: monkeypatch在功能性测试中的应用
category: software engineering
catalog: true
published: false
tags:
  - software engineering
  - test
time: '2024.07.22 10:32:00'
---

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
