---
title: 入门
weight: 1
---

本指南是一个 "Hello World"-样式 教程 它展示了如何安装, 配置, 和在一个简单的例子设置中使用 Prometheus.
你会下载并在本地运行 Prometheus, 配置它来刮本身和一个示例应用程序,
再与查询，规则和图形工作 要利用收集到的时间序列数据.

## 下载并运行 Prometheus

[下载最新版本](https://prometheus.io/download) Prometheus 为您的平台, 然后解压并运行它:

```bash
tar xvfz prometheus-*.tar.gz
cd prometheus-*
```

Prometheus 开始之前，让我们来配置它.

## 配置 Prometheus 监控本身

Prometheus 收集指标 从监控目标 通过刮度量 HTTP 端点 这些目标.
由于 Prometheus 还公开数据 在大约本身相同的方式, 它也能刮和监测自身健康.

而一个 Prometheus 服务器收集大约只有自身数据不是在实践中非常有用, 这是一个很好的例子开始.
保存以下基本配置 Prometheus 作为一档名为 `prometheus.yml`:

```yaml
global:
  scrape_interval: 15s # 默认, 每15秒刮目标 .

  # 当通信时这些附加标签的任何时间序列或警报
  # 外部系统 (federation, remote storage, Alertmanager).
  external_labels:
    monitor: "codelab-monitor"

# 一刮配置正好含有一个端点来刮:
# 在这里它的本身Prometheus.
scrape_configs:
  # 作业名称添加为标签 `job=<job_name>` 要刮掉来自这个配置任何时间序列 .
  - job_name: "prometheus"

    # 覆盖全局默认 和每5秒从这份作业刮目标  .
    scrape_interval: 5s

    static_configs:
      - targets: ["localhost:9090"]
```

有关配置选项的完整规范, 参见[配置文档](configuration/configuration.md).

## 启动 Prometheus

要使用新创建的配置文件启动 Prometheus, 改成含有 Prometheus 二进制文件的目录并运行:

```bash
# Start Prometheus.
# By default, Prometheus stores its database in ./data (flag --storage.tsdb.path).
./prometheus --config.file=prometheus.yml
```

Prometheus 应该启动。
你也应该能够浏览到状态页面大约在自身[localhost:9090](http://localhost:9090).
给它几秒钟收集有关数据本身 从自己的 HTTP 指标端点.

您也可以验证 Prometheus 是服务本身有关的指标导航到它的终点指标:[localhost:9090/metrics](http://localhost:9090/metrics)

## 使用浏览器的表达

让我们试着看一些数据 Prometheus 已收集有关其自身.
要使用 Prometheus 的内置浏览器表达, 导航http://localhost:9090/graph 并选择"Graph"内标签"Console" 视图 .

你可以从[localhost:9090/metrics](http://localhost:9090/metrics)收集,
一个度量 Prometheus 出口量约本身叫做 `prometheus_target_interval_length_seconds` (目标擦之间的时间的实际额度).
前进和 进入此 表达控制台 然后单击 "Execute":

```
prometheus_target_interval_length_seconds
```

这应返回许多不同的时间序列 (随着记录每个最新值), 所有的指标名称`prometheus_target_interval_length_seconds`, 但不同的标签.
这些标签指定不同的等待时间和百分目标组的间隔.

如果我们只关心第 99 百分位延迟, 我们可以利用这个查询检索信息:

```
prometheus_target_interval_length_seconds{quantile="0.99"}
```

要计算返回时间序列的数量，你可以写:

```
count(prometheus_target_interval_length_seconds)
```

欲了解更多有关表达式语言, 参见[表达式语言文档】(querying/basics.md).

## 使用绘图接口

为了图表情, 导航 http://localhost:9090/graph 并使用 "Graph" 标签.

例如, 输入以下表达式绘制图形的每秒帧率 的 在自刮 Prometheus 创建块:

```
rate(prometheus_tsdb_head_chunks_created_total[1m])
```

与图表范围的参数和其他设置实验.

## 启动了一些取样目标

让我们把这个更有趣，并开始了一些例如目标 Prometheus 刮.

节点导出器被用作一个例子目标, 有关使用它的更多信息[查看这些说明](https://prometheus.io/docs/guides/node-exporter/)

```bash
tar -xzvf node_exporter-*.*.tar.gz
cd node_exporter-*.*

# Start 3 example targets in separate terminals:
./node_exporter --web.listen-address 127.0.0.1:8080
./node_exporter --web.listen-address 127.0.0.1:8081
./node_exporter --web.listen-address 127.0.0.1:8082
```

你现在应该有例如目标 监听 http://localhost:8080/metrics,http://localhost:8081/metrics, 和 http://localhost:8082/metrics.

## 配置 Prometheus 监控的取样目标

现在，我们将配置 Prometheus 刮这些新的目标。
让我们组的所有三个端点成一个作业 叫 `node`.
然而, 设想前两个端点的生产目标, 而第三个表示金丝雀实例.
要 Prometheus 这个模型, 我们可以添加端点的几组到一个单一的作业, 增加额外的标签 至 每个目标组.
在这个例子中, 我们将添加 `group="production"` 标签 至 目标第一组, 同时加入 `group="canary"` 到第二.

为达到这个, 下面的工作定义添加到 `scrape_configs` 部分 在你的 `prometheus.yml`并重新启动您的 Prometheus 实例:

```yaml
scrape_configs:
  - job_name: "node"

    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s

    static_configs:
      - targets: ["localhost:8080", "localhost:8081"]
        labels:
          group: "production"

      - targets: ["localhost:8082"]
        labels:
          group: "canary"
```

去表达浏览器 并验证 Prometheus 目前拥有大约时间序列信息 这些例子端点暴露, 如 `node_cpu_seconds_total`.

## 对于刮数据聚合成新的时间序列配置规则

虽然在我们的例子不是问题, 那聚集在成千上万的时间序列查询可以得到缓慢当计算出的 ad-hoc.
为了使这更有效, Prometheus 让您预录表达式为全新的持久性的时间序列通过配置记录规则.
比方说，我们感兴趣的是记录 CPU 时间的每秒帧率 (`node_cpu_seconds_total`) 平均超过每个实例的所有 CPU (但保留 `job`, `instance` 和 `mode` 尺寸) 如在 5 分钟的窗口测量. 我们可以这样写:

```
avg by (job, instance, mode) (rate(node_cpu_seconds_total[5m]))
```

尝试绘图这个表达式。

从这个表达式产生的时间序列记录到一个新指标叫 `job_instance_mode:node_cpu_seconds:avg_rate5m`, 创建一个文件
用下面的记录规则并将其保存为 `prometheus.rules.yml`:

```
groups:
- name: cpu-node
  rules:
  - record: job_instance_mode:node_cpu_seconds:avg_rate5m
    expr: avg by (job, instance, mode) (rate(node_cpu_seconds_total[5m]))
```

为了使 Prometheus 捡起这个新规则, 添加 `rule_files` 声明 在你的 `prometheus.yml`.
这个配置现在看起来应该是这样:

```yaml
global:
  scrape_interval: 15s # By default, scrape targets every 15 seconds.
  evaluation_interval: 15s # Evaluate rules every 15 seconds.

  # Attach these extra labels to all timeseries collected by this Prometheus instance.
  external_labels:
    monitor: "codelab-monitor"

rule_files:
  - "prometheus.rules.yml"

scrape_configs:
  - job_name: "prometheus"

    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s

    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "node"

    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s

    static_configs:
      - targets: ["localhost:8080", "localhost:8081"]
        labels:
          group: "production"

      - targets: ["localhost:8082"]
        labels:
          group: "canary"
```

重新启动 Prometheus 以新配置并验证一个新的时间序列 与度量名称 `job_instance_mode:node_cpu_seconds:avg_rate5m`
现在通过查询其提供 通过表达浏览器或绘图它.
