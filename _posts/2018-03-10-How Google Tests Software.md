---
published: true
layout: post
title: How Google Tests Software
category: test
tags:
  - test
time: '2018.03.10 16:24:00'
excerpt: 阅读《How Google Tests Software》，好的观点理念随手记录一下
---
《How Google Tests Software》

<!--more-->

## Points
1. 测试方法不当会扼杀一个本来有机会成功的商品或公司，至少拖慢这个产品的速度;
2. 工程生产力（Engineering Productivity）团队：更加关注生产力，而不是测试和质量。测试和质量是开发过程里每个人都要承担的工作。这意味着开发人员负责测试，开发人员负责质量。生产力团队负责帮助开发团队搞定这两项任务（横跨开发和测试工具链）；
3. coder需要肩负服务质量责任；
4. 每个developer都需要自己进行测试。tester（这里的tester应该是工程生产力团队的成员）的职责是确保其拥有自动化基础设施和支持自我依赖的保障流程。tester有让developer进行测试的能力；
5. Quality != Test. 质量可以通过将开发和测试放到搅拌机中搅拌到彼此分不清才能得到；
6. 质量更多的是一个预防行为而不是一个探测行为；

## Roles
1. software engineer(SWE):传统的开发角色，SWE将编写的功能函数传递给最终用户；
2. software engineer in test(SET):也是一个开发的角色，但更加注重可测性及一般测试基础设施建设。SETs检视设计并审视代码质量和风险，SETs重构代码使得代码更容易被测试并且编写单元测试框架和自动化；
3. software engineer（SE）:从最终用户视角进行测试并组织整体质量实践，分析测试结果，驱动测试执行并构建端到端测试自动化；

## Types of Tests
1. 为了避免让代码、集成和系统测试混淆在一起，谷歌用（small、medium及large）来强调测试范围而非形式；
 - Small tests:在一个伪造的环境中覆盖单一代码单元；
 - Medium tests:在一个伪造或真实环境中覆盖多个和交互的代码单元；
 - Large tests:在真实的生产环境中覆盖所有的代码单元；
作者举了个场景：如果一个自动化测试失败了，系统会探测最后的代码变更更可能是罪魁祸首，并发送一个邮件给作者并且自动的报告一个bug；

## REFS
[How google Tests Software](https://testing.googleblog.com/2011/01/how-google-tests-software.html)
[what test engineers do at google](https://testing.googleblog.com/2016/11/what-test-engineers-do-at-google.html)
[testing at speed and scale of google](https://testing.googleblog.com/2011/06/testing-at-speed-and-scale-of-google.html)
