---
published: true
layout: post
title: smokeping代码走读
category: ops
tags:
  - ops
time: '2017.12.21 20:24:00'
excerpt: smokeping代码走读
---

smokeping架构设计(第一次小结)

<!--more-->

## 背景介绍

## 基础安装

## 核心代码
&emsp;&emsp;base类和basefork类是smokeping的基类，所有探针（probe）扩展都需要从这两个继承这两个类,其中fping、echoping等系统工具在smokeping中一个独立的探针（probe）,如下图probe所示。basefork与base类相比的差异点在于basefork支持多进程的并发性。在当前的smokeping版本中，Fping是base类的衍生类，其他的探针都从basefork衍生。为什么Fping不从basefork类继承呢？这是因为fping程序并不支持对多主机的并发测试。

&emsp;&emsp;base有三个核心属性：step、offset以及pings。step指的是两次探针使用的间隔，上图中的every 60s就是两次探针的间隔时间。当多探针并发执行时，会对网络带来较大的负载影响，而offset可以调整任意一个探针的执行时间点。在一个完整的间隔中到底向目标机器发送多少个探测报文这个由pings来决定。


```perl
package Smokeping::probes::base;

sub step {
	my $self = shift;
	my $rv = $self->{cfg}{Database}{step};
	unless (defined $self->{cfg}{General}{concurrentprobes}
	    and $self->{cfg}{General}{concurrentprobes} eq 'no') {
		$rv = $self->{properties}{step} if defined $self->{properties}{step};
	}
	return $rv;
}

sub offset {
	my $self = shift;
	my $rv = $self->{cfg}{General}{offset};
	unless (defined $self->{cfg}{General}{concurrentprobes}
	    and $self->{cfg}{General}{concurrentprobes} eq 'no') {
		$rv = $self->{properties}{offset} if defined $self->{properties}{offset};
	}
	return $rv;
}

sub pings {
	my $self = shift;
	my $target = shift;
	# $target is not used; basefork.pm overrides this method to provide a target-specific parameter
	my $rv = $self->{cfg}{Database}{pings};
	$rv = $self->{properties}{pings} if defined $self->{properties}{pings};
	return $rv;
}
```

smokeping的核心架构类图如下所示：

![]({{site.baseurl}}/img/2017122503smokeping_struct.png)

如果用户要对smokeping进行探针扩展，只需要继承base或者basefork即可（覆写ping或者pingone函数）。
&emsp;&emsp;smokeping的探测数据存放于RRDtool中。RRDtool (轮替型数据库工具, round-robin database tool) 旨在处理时间序列资料，例如网络带宽、温度或CPU负载。 资料储存在环形缓冲区为基底的数据库中，因此系统储存占用量随时间保持恒定。

![]({{site.baseurl}}/img/2017122601Circular_buffer.svg.png)