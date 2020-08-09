---
title: 使用 Prometheus 第一步
linkTitle: 第一步
weight: 3
---

欢迎来到 Prometheus!
Prometheus 是一个监控平台 收集度量 从监控目标 通过刮度量 HTTP 端点 这些目标.
本指南将告诉您如何安装, 配置和监控我们的第一 Prometheus 资源.
您可以下载，安装和运行 Prometheus.
您还可以下载和安装 exporter, 工具 暴露的主机和服务的时间序列数据.
我们的第一个 exporter 将会 Prometheus 本身, 这提供了多种关于存储器使用，垃圾回收，和多个主机级度量。

## 下载 Prometheus

针对您的平台[下载 Prometheus 最新版本](/download), 然后解压:

```language-bash
tar xvfz prometheus-*.tar.gz
cd prometheus-*
```

Prometheus 服务器称为`prometheus`一个二进制 (要么 `prometheus.exe` 上 Microsoft Windows).
我们可以运行的二进制程序，通过使`--help`标志看到它的选项的帮助 .

```language-bash
./prometheus --help
usage: prometheus [<flags>]

The Prometheus monitoring server

. . .
```

在开始 Prometheus 之前, 让我们来配置它.

## 配置 Prometheus

Prometheus 配置是 [YAML](http://www.yaml.org/start.html).
Prometheus 下载带有一个在`prometheus.yml`文件里示例配置 这是一个很好的地方开始.

我们已经去掉了大部分的示例文件的注释，使之更加简洁 (注释以`＃`前缀的行).

```language-yaml
global:
  scrape_interval:     15s
  evaluation_interval: 15s

rule_files:
  # - "first.rules"
  # - "second.rules"

scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ['localhost:9090']
```

在示例配置文件中有配置的三个区块 : `global`, `rule_files`, 和 `scrape_configs`.

`global`块控制 Prometheus 服务器的全局配置.
我们目前有两个选择.
首先, `scrape_interval`, 控制 Prometheus 多久会刮目标.
您可以覆盖这个单个目标.
在这种情况下，全局设置是每 15 秒刮.
`evaluation_interval` 选项控制 Prometheus 多久会评估规则.
Prometheus 使用规则来创建新的时间序列，并生成警报.

`rule_files`块指定我们希望 Prometheus 服务器负载的任何规则的位置.
现在，我们已经有了无规则.

最后一个块，`scrape_configs`，控制哪些资源 Prometheus 显示器。
由于 Prometheus 还公开有关数据本身 因为 HTTP 端点可以刮去并监视其自己的健康.
在默认配置中有一个单一的作业, 叫 `prometheus`,其中刮除由 Prometheus 服务器暴露的时间序列数据.
该作业包含一个单一的，静态配置，目标端口`localhost` `9090`.
Prometheus 预计指标可用的目标 在目录 `/metrics`.
所以这个默认的工作是通过 URL 刮: http://localhost:9090/metrics.

时间序列数据返回将详细介绍 Prometheus 服务器的状态和性能.

有关配置选项的完整规范, 参见[配置文档](/docs/operating/configuration).

## 开始 Prometheus

与我们的新创建的配置文件启动 Prometheus, 转到包含 Prometheus 二进制和运行目录:

```language-bash
./prometheus --config.file=prometheus.yml
```

Prometheus 应该启动.
你也应该能够浏览到状态页面本身有关 在 http://localhost:9090.
给它 30 秒左右从自己的 HTTP 指标端点收集有关数据本身.

您也可以验证 Prometheus 被导航到它自己的指标端点服务有关自身的度量: http://localhost:9090/metrics.

## 使用浏览器的表达

让我们试着寻找一些数据 Prometheus 已收集有关其自身.
要使用 Prometheus 的内置浏览器表达, 导航 http://localhost:9090/graph 并选择“Graph”选项卡中的“Console”视图.

你可以从 http://localhost:9090/metrics 收集 , 一个度量 Prometheus 出口量约本身被称为 `promhttp_metric_handler_requests_total` (`/ metrics`的总数 请求 Prometheus 服务器一直担任).
来吧，进入控制台表达这:

```
promhttp_metric_handler_requests_total
```

这应返回许多不同的时间序列 (随着记录每个最新值), 所有的指标名称 `promhttp_metric_handler_requests_total`, 但不同的标签.
这些标签指定不同的请求状态.

如果我们只关心请求 这导致了 HTTP 代码`200`, 我们可以利用这个查询检索信息:

```
promhttp_metric_handler_requests_total{code="200"}
```

要计算返回时间序列的数量，你可以写:

```
count(promhttp_metric_handler_requests_total)
```

欲了解更多有关表达式语言, 参见[表达式语言文档](/docs/querying/basics/).

## 使用绘图接口

为了图表情, 导航 http://localhost:9090/graph 并使用 "Graph" 标签.

例如, 输入以下表达式曲线图的每秒 HTTP 请求速率 返回状态码 200 发生在自刮 Prometheus:

```
rate(promhttp_metric_handler_requests_total{code="200"}[1m])
```

Y 你可以用图形范围的参数和其他设置实验.

## 监测的其他目标

从单独 Prometheus 收集度量不是 Prometheus 能力很大的代表性.
为了得到一个什么样的 Prometheus 可以做的更好的感觉, 我们建议探索对其他出口文件.
[使用节点出口监测 Linux 或 MACOS 主机度量](/docs/guides/node-exporter)指南是一个良好的开端.

## 摘要

在本指南中，您安装 Prometheus，配置了 Prometheus 实例来监控资源，并学会在 Prometheus 表达浏览器的时间序列数据工作的一些基础知识。
要继续学习 Prometheus，检查出的[概述](/docs/introduction/overview) 对于下一步该怎么去探索一些想法.
