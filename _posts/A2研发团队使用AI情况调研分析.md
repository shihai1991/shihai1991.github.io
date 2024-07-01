---
layout: post
title: A2研发团队使用AI情况调研分析
category: software engineering
catalog: true
published: false
tags:
  - software engineering
time: '2024.07.01 11:27:00'
---

# A1
## 设计
### Claude3 Sonnet
Claude3 Sonnet 通过模型可以辅助研发人员达成分析阶段的需求分析、绘制各类UML图、领域模型设计，并生成相关代码。

[生成式 AI 技术辅助软件系统设计开发](https://aws.amazon.com/cn/blogs/china/generative-ai-technology-assists-software-system-design-and-development/)

## 开发
### [Code Guru](https://aws.amazon.com/codeguru/)
是一款静态应用程序安全测试 (SAST) 工具，结合了ML和自动推理来识别代码中的漏洞，并提供有关如何修复这些漏洞的建议，并跟踪其状态直至关闭。CodeGuru Security检测到[开放式全球应用程序安全项目 (OWASP)发现的十大问题](https://owasp.org/www-project-top-ten/) [常见弱点列举 (CWE)](https://cwe.mitre.org/top25/archive/2022/2022_cwe_top25.html)发现的前25个问题 、日志注入、机密以及AWS API和SDK的不安全使用。CodeGuru Security还借鉴了AWS安全最佳实践，并在亚马逊上接受了数百万行代码的培训。

- [使用 Amazon CodeGuru Reviewer 检测日志记录中的安全问题](https://aws.amazon.com/cn/blogs/devops/detecting-security-issues-in-logging-with-amazon-codeguru-reviewer/)
- [用于安全的 AI/ML](https://docs.aws.amazon.com/prescriptive-guidance/latest/security-reference-architecture/ai-ml.html)

## 运维
### [Macie](https://aws.amazon.com/macie/)
是一项数据安全服务，它使用 ML 和模式匹配来发现并帮助保护您的敏感数据。Macie 会自动检测大量且不断增长的敏感数据类型，包括个人身份信息 (PII)，例如姓名、地址和财务信息，例如信用卡号。它还让您可以持续查看存储在 Amazon Simple Storage Service (Amazon S3) 中的数据。

### [Devops Guru](https://aws.amazon.com/devops-guru/)
Amazon DevOps Guru是一种采用ML的服务，可轻松提高应用程序的操作性能和可用性。DevOps Guru检测偏离正常操作模式的行为，这样您就可以在操作问题影响您的客户之前及早地识别出它们。

[利用 Amazon DevOps Guru 提供的主动洞察节省成本并提高 Lambda 应用程序性能](https://aws.amazon.com/cn/blogs/devops/save-cost-and-improve-lambda-application-performance-with-proactive-insights-from-amazon-devops-guru/)

### [Lookout for Metrics](https://aws.amazon.com/lookout-for-metrics/)
可自动对AWS工作负载和第三方云应用程序进行异常检测和性能监控。

### [GuardDuty](https://aws.amazon.com/guardduty/)
是一种威胁检测服务，它使用 ML、异常检测和集成威胁情报来持续监控恶意活动和未经授权的行为，以帮助保护您的 AWS 账户、实例、无服务器和容器工作负载、用户、数据库和存储。

### AIOps
IT 运维人工智能（AIOps）是指使用人工智能（AI）技术维护 IT 基础设施的过程。您可以自动执行关键运维任务，例如性能监控、工作负载调度和数据备份。AIOps 技术使用现代机器学习（ML）、自然语言处理（NLP）和其他高级 AI 方法来提高 IT 运营效率。该技术可以收集和分析许多不同来源的数据，为 IT 运维提供主动、个性化和实时的见解。
以下是一些为满足 AIOps 要求而构建的 AWS 产品：
- [Amazon DevOps Guru](https://aws.amazon.com/devops-guru/)是一项机器学习支持型服务，可帮助软件团队自动检测云中的异常操作；
- [Amazon CodeGuru Security](https://aws.amazon.com/codeguru/)是软件测试工具，可使用机器学习算法自动扫描和识别代码漏洞；
- [Amazon Lookout for Metrics](https://aws.amazon.com/lookout-for-metrics/)可自动对 AWS 工作负载和第三方云应用程序进行异常检测和性能监控；

[AIOps](https://aws.amazon.com/cn/what-is/aiops/)

# A2
