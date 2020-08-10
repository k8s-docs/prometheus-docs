---
title: Federation
weight: 6
---

Federation 允许 Prometheus 服务器从另一个 Prometheus 服务器刮选定的时间序列 .

## 用例

有不同的使用情况进行联盟。
常用, 它是用来实现任一可伸缩的 Prometheus 监控设置或拉动相关指标从一个服务的 Prometheus 到另一个.

### 层次联邦

层次联邦允许普罗米修斯规模化环境 几十数据中心和几百万个节点.
在这种情况下使用, 联邦拓扑结构类似于树, 与更高级别的普罗米修斯服务器 收集聚集的时间序列数据 从次级服务器更大数量的.

例如, 一个设置可能由许多每个数据中心普罗米修斯服务器 在高细节数据收集 (实例级向下钻取),
和一组全球普罗米修斯服务器 它收集和存储仅汇总数据 (作业级向下钻取) 从这些本地服务器.
这提供了全局总图和详细的地方意见.

### 跨服联盟

In cross-service federation, a Prometheus server of one service is configured
to scrape selected data from another service's Prometheus server to enable
alerting and queries against both datasets within a single server.

For example, a cluster scheduler running multiple services might expose
resource usage information (like memory and CPU usage) about service instances
running on the cluster. On the other hand, a service running on that cluster
will only expose application-specific service metrics. Often, these two sets of
metrics are scraped by separate Prometheus servers. Using federation, the
Prometheus server containing service-level metrics may pull in the cluster
resource usage metrics about its specific service from the cluster Prometheus,
so that both sets of metrics can be used within that server.

## 配置联盟

On any given Prometheus server, the `/federate` endpoint allows retrieving the
current value for a selected set of time series in that server. At least one
`match[]` URL parameter must be specified to select the series to expose. Each
`match[]` argument needs to specify an
[instant vector selector](querying/basics.md#instant-vector-selectors) like
`up` or `{job="api-server"}`. If multiple `match[]` parameters are provided,
the union of all matched series is selected.

To federate metrics from one server to another, configure your destination
Prometheus server to scrape from the `/federate` endpoint of a source server,
while also enabling the `honor_labels` scrape option (to not overwrite any
labels exposed by the source server) and passing in the desired `match[]`
parameters. For example, the following `scrape_configs` federates any series
with the label `job="prometheus"` or a metric name starting with `job:` from
the Prometheus servers at `source-prometheus-{1,2,3}:9090` into the scraping
Prometheus:

```yaml
scrape_configs:
  - job_name: "federate"
    scrape_interval: 15s

    honor_labels: true
    metrics_path: "/federate"

    params:
      "match[]":
        - '{job="prometheus"}'
        - '{__name__=~"job:.*"}'

    static_configs:
      - targets:
          - "source-prometheus-1:9090"
          - "source-prometheus-2:9090"
          - "source-prometheus-3:9090"
```
