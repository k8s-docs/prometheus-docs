---
title: 概述
weight: 1
---

## 什么是 Prometheus?

[Prometheus](https://github.com/prometheus) 是一个开放源码的系统监控和报警工具包原建于[SoundCloud](http://soundcloud.com).
公司自成立以来 2012, 很多公司和组织都采用`Prometheus`, 与该项目有一个非常活跃的开发者和用户[社区](/community).
它现在是一个独立的开源项目和任何一家公司的保持独立 .
为了强调这一点, 并澄清该项目的治理结构, Prometheus 在 2016 年加盟 [云本地计算基础](https://cncf.io/) 成为[Kubernetes](http://kubernetes.io/)后第二个托管项目.

对于 Prometheus 的更详尽的概述, 查看来自[媒体](/docs/introduction/media/)部分的链接资源.

### 特征

Prometheus 的主要特点是:

- 多维[数据模型](/docs/concepts/data_model/) 通过指标名称和键/值对确定时间序列数据
- PromQL,[灵活的查询语言](/docs/prometheus/latest/querying/basics/)利用这一维度
- 不依赖 分布式存储; 单个服务器节点是自治
- 时间序列收集情况 通过通过 HTTP 拉模式
- [推时间序列](/docs/instrumenting/pushing/) 支持 通过中间网关
- 目标是通过服务发现或静态配置发现
- 作图和仪表板支持的多种模式

### 组件

Prometheus 生态系统 由多个组件, 其中许多都是可选:

- 主要的[Prometheus 服务器](https://github.com/prometheus/prometheus) 其中擦伤和存储时间序列数据
- [客户端库](/docs/instrumenting/clientlibs/) 插装应用程序代码
- [推送网关](https://github.com/prometheus/pushgateway) 支持短命工作
- 特殊目的 [exporters](/docs/instrumenting/exporters/) 像服务 HAProxy, StatsD, Graphite, 等等
- [alertmanager](https://github.com/prometheus/alertmanager) 处理警报
- 各种支持工具

大多数 Prometheus 成分都写在[Go](https://golang.org/), 使它们易于构建和部署为静态二进制文件.

### 架构

此图说明 Prometheus 的架构和一些生态系统的组成部分:

![Prometheus 架构](/assets/architecture.png)

从仪表工作 Prometheus 擦伤指标, 直接或通过对短命工作的中介推送网关.
它在本地存储所有的样品刮掉 and 运行规则对这些数据要么汇总 and 从现有的数据记录新的时间序列或生成警报.
[Grafana](https://grafana.com/) 或其他 API 消费者 可以被用于可视化所采集的数据.

## 它什么时候合适？

Prometheus 非常适用于记录任何纯粹的数值时间序列.
它适合 同时 机为中心的监控 以及 监测高度动态的面向服务的体系结构.
在微服务的世界, 它的多维数据采集和查询的支持是一个特别的力量.

Prometheus 被设计用于可靠性, 为系统 您在停电时去让您快速诊断问题.
每 Prometheus 服务器是独立的, 不依赖于网络存储或其他远程服务.
你可以依靠它，当你的基础设施的其他部分被破坏, 而你并不需要设置大量的基础设施使用它.

## 它什么时候不适合？

Prometheus 值的可靠性.
您可以随时查看 什么统计资料有关系统, 即使在故障情况.
如果你需要 100％的准确度, 诸如用于每个请求的计费, Prometheus 不是一个好的选择因为收集到的数据可能不会被详细，完整的足够.
在这种情况下 你最好是去使用一些其他系统 收集和分析计费数据 , 和 Prometheus 为您监视其余.
