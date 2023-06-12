---
layout: post
title: kubernetes test SIG组相关能力梳理
category: k8s
catalog: true
published: true
tags:
  - k8s
  - test
time: '2023.06.12 09:35:00'
---
k8s测试基础设施(test-infra)的主要项目清单：
![]({{site.baseurl}}/img/2023/Q2/20230612-k8s-test-infra.png)

从PR触发测试执行流程：
![]({{site.baseurl}}/img/2023/Q2/20230612-test-infra工作流.png)

[k8s test-infra代码仓](https://github.com/kubernetes/test-infra)

# 一、自动化
## [prow](https://github.com/kubernetes/test-infra/tree/master/prow)
**用途**：基于k8s的CI/CD系统。  
主要的组件清单及功能：
- **Crier**: 用于报告prowjobs的状态，现在已有的reporter：Gerrit reporter、Pubsub reporter、GitHub reporter、Slack reporter,当然你也可以扩展reporter；
- **Deck**: 在prow页面上展示正在运行或最近运行的jobs；
- **prow-controller-manager**：管理在k8s中运行的作业执行和生命周期；
- **Tide**：管理Github的PR，可以对PR进行合并；
- **Horologium**: TBD；
- **Sinker**: TBD；

![]({{site.baseurl}}/img/2023/Q2/20230612-prow-architecture.png)

[Prow Status](https://prow.k8s.io/)
**补充prow status图**

## [kubetest1/2](https://github.com/kubernetes-sigs/kubetest2)
**用途**: kubetest是一个部署k8s集群并能运行端到端测试，kubetest已经被kubetest2替代。[详细文档](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-testing/e2e-tests.md)  
k8s中的e2e测试建立在[Ginkgo](https://onsi.github.io/ginkgo)和[Gomega](https://onsi.github.io/gomega)之上，该行为驱动开发测试框架(BDD)提供了很多特性。
构建k8s、启动cluster、执行测试、清理所有资源，可以使用这些命令：
```shell
# 构建k8s
kubetest2 gce --build --legacy-mode
# 执行测试
kubetest2 gce --gcp-project <project> --up
kubetest2 gce --test ginkgo -- --focus-regex "\[Feature:Performance\]"
# 清理
kubetest2 gce --gcp-project <project> --down
```
一个[nodepool e2e测试用例](https://github.com/kubernetes/kubernetes/blob/master/test/e2e/cloud/gcp/gke_node_pools.go)，具体测试代码块，如下所示：
```go
var _ = SIGDescribe("GKE node pools [Feature:GKENodePool]", func() {

	f := framework.NewDefaultFramework("node-pools")
	f.NamespacePodSecurityEnforceLevel = admissionapi.LevelPrivileged

	ginkgo.BeforeEach(func() {
		e2eskipper.SkipUnlessProviderIs("gke")
	})

	ginkgo.It("should create a cluster with multiple node pools [Feature:GKENodePool]", func(ctx context.Context) {
		framework.Logf("Start create node pool test")
		testCreateDeleteNodePool(ctx, f, "test-pool")
	})
})
```
**个人遗留问题，TBD**：这里的实际用例执行为什么用的是命令行去触发？是用户接触的是命令行模式？还是其他什么原因？

## [boskos](https://github.com/kubernetes-sigs/boskos)
用途：资源管理服务，为测试提供干净的GCP项目或者k8s集群。
Boskos支持两种资源类型：静态资源、动态资源。
```shell
---
resources:
  # Static
  - type: "aws-account"
    state: free
    names:
    - "account1"
    - "account2"
  # Dynamic
  - type: "aws-cluster"
    state: dirty
    min-count: 1
    max-count: 2
    lifespan: 48h
    needs:
      aws-account: 1
    config:
      type: AWSClusterCreator
      content: "..."
```

## gopherage
**用途**：go覆盖率工具，并且可以转换为testgrid兼容的junit。

# 二、可视化
## testgrid
**用途**：用于在网格中查看测试结果。  
k8s的testgrid实例可以查看[此链接](https://testgrid.k8s.io)，现在主要的代码都归档于[GCP group](https://github.com/GoogleCloudPlatform/testgrid)中。
**补充testgrid图**

# 三、其他
## kind

# 四、参考文献
- [k8s testing sig](https://github.com/kubernetes/community/blob/master/sig-testing/README.md)
