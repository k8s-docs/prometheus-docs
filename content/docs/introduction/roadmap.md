---
title: 路线图
weight: 6
---

The following is only a selection of some of the major features we plan to
implement in the near future. To get a more complete overview of planned
features and current work, see the issue trackers for the various repositories,
for example, the [Prometheus
server](https://github.com/prometheus/prometheus/issues).

### 服务器端的度量元数据支持

At this time, metric types and other metadata are only used in the
client libraries and in the exposition format, but not persisted or
utilized in the Prometheus server. We plan on making use of this
metadata in the future. The first step is to aggregate this data in-memory
in Prometheus and provide it via an experimental API endpoint.

### 采用 OpenMetrics

The OpenMetrics working group is developing a new standard for metric exposition.
We plan to support this format in our client libraries and Prometheus itself.

### 回填时间序列

Backfilling will permit bulk loads of data in the past. This will allow for
retroactive rule evaluations, and transferring old data from other monitoring
systems.

### TLS 和 HTTP 服务端点认证

The HTTP serving endpoints in Prometheus, Alertmanager, and the official exporters
do not have built-in support for TLS and authentication yet. Adding this support
will make it easier for people to deploy Prometheus components securely without
requiring a reverse proxy to add those features externally.

### 支持生态系统

Prometheus has a range of client libraries and exporters. There are always more
languages that could be supported, or systems that would be useful to export
metrics from. We will support the ecosystem in creating and expanding these.
