---
title: 常问问题
weight: 5
toc: full-width
---

## 一般

### 什么是 Prometheus？

Prometheus is an open-source systems monitoring and alerting toolkit
with an active ecosystem. See the [overview](/docs/introduction/overview/).

### 如何比较 Prometheus 与其他监控系统？

See the [comparison](/docs/introduction/comparison/) page.

### 哪些依赖它 Prometheus 呢？

The main Prometheus server runs standalone and has no external dependencies.

### Prometheus 可制成具有高可用性？

Yes, run identical Prometheus servers on two or more separate machines.
Identical alerts will be deduplicated by the [Alertmanager](https://github.com/prometheus/alertmanager).

For [high availability of the Alertmanager](https://github.com/prometheus/alertmanager#high-availability),
you can run multiple instances in a
[Mesh cluster](https://github.com/weaveworks/mesh) and configure the Prometheus
servers to send notifications to each of them.

### 有人告诉我，Prometheus“没有规模”。

There are in fact various ways to scale and federate
Prometheus. Read [Scaling and Federating Prometheus](https://www.robustperception.io/scaling-and-federating-prometheus/)
on the Robust Perception blog to get started.

### Prometheus 是写在什么语言？

大多数 Prometheus 成分都用 Go 写.
也有的用 Java，Python 和 Ruby 编写的。

### 如何稳定是 Prometheus 的功能，存储格式和 API？

All repositories in the Prometheus GitHub organization that have reached
version 1.0.0 broadly follow
[semantic versioning](http://semver.org/). Breaking changes are indicated by
increments of the major version. Exceptions are possible for experimental
components, which are clearly marked as such in announcements.

Even repositories that have not yet reached version 1.0.0 are, in general, quite
stable. We aim for a proper release process and an eventual 1.0.0 release for
each repository. In any case, breaking changes will be pointed out in release
notes (marked by `[CHANGE]`) or communicated clearly for components that do not
have formal releases yet.

### 你为什么拉而不是推？

Pulling over HTTP offers a number of advantages:

- You can run your monitoring on your laptop when developing changes.
- You can more easily tell if a target is down.
- You can manually go to a target and inspect its health with a web browser.

Overall, we believe that pulling is slightly better than pushing, but it should
not be considered a major point when considering a monitoring system.

For cases where you must push, we offer the [Pushgateway](/docs/instrumenting/pushing/).

### 如何给登录到 Prometheus？

Short answer: Don't! Use something like the [ELK stack](https://www.elastic.co/products) instead.

Longer answer: Prometheus is a system to collect and process metrics, not an
event logging system. The Raintank blog post
[Logs and Metrics and Graphs, Oh My!](https://blog.raintank.io/logs-and-metrics-and-graphs-oh-my/)
provides more details about the differences between logs and metrics.

If you want to extract Prometheus metrics from application logs, Google's
[mtail](https://github.com/google/mtail) might be helpful.

### 是谁写的 Prometheus？

Prometheus was initially started privately by
[Matt T. Proud](http://www.matttproud.com) and
[Julius Volz](http://juliusv.com). The majority of its
initial development was sponsored by [SoundCloud](https://soundcloud.com).

It's now maintained and extended by a wide range of companies and individuals.

### 什么许可证 Prometheus 下发布？

Prometheus is released under the
[Apache 2.0](https://github.com/prometheus/prometheus/blob/master/LICENSE) license.

### 什么是 Prometheus 的复数形式呢？

After [extensive research](https://youtu.be/B_CDeYrqxjQ), it has been determined
that the correct plural of 'Prometheus' is 'Prometheis'.

### 我可以重新加载 Prometheus 的配置？

Yes, sending `SIGHUP` to the Prometheus process or an HTTP POST request to the
`/-/reload` endpoint will reload and apply the configuration file. The
various components attempt to handle failing changes gracefully.

### 我可以发送警报？

Yes, with the [Alertmanager](https://github.com/prometheus/alertmanager).

Currently, the following external systems are supported:

- Email
- Generic Webhooks
- [HipChat](https://www.hipchat.com/)
- [OpsGenie](https://www.opsgenie.com/)
- [PagerDuty](http://www.pagerduty.com/)
- [Pushover](https://pushover.net/)
- [Slack](https://slack.com/)

### 我可以创建仪表板？

Yes, we recommend [Grafana](/docs/visualization/grafana/) for production
usage. There are also [Console templates](/docs/visualization/consoles/).

### 我可以更改时区？为什么一切都在 UTC？

To avoid any kind of timezone confusion, especially when the so-called
daylight saving time is involved, we decided to exclusively use Unix
time internally and UTC for display purposes in all components of
Prometheus. A carefully done timezone selection could be introduced
into the UI. Contributions are welcome. See
[issue #500](https://github.com/prometheus/prometheus/issues/500)
for the current state of this effort.

## 仪表

### 这语言有仪器库？

There are a number of client libraries for instrumenting your services with
Prometheus metrics. See the [client libraries](/docs/instrumenting/clientlibs/)
documentation for details.

If you are interested in contributing a client library for a new language, see
the [exposition formats](/docs/instrumenting/exposition_formats/).

### 我可以监视机？

Yes, the [Node Exporter](https://github.com/prometheus/node_exporter) exposes
an extensive set of machine-level metrics on Linux and other Unix systems such
as CPU usage, memory, disk utilization, filesystem fullness, and network
bandwidth.

### 我可以监视网络设备？

Yes, the [SNMP Exporter](https://github.com/prometheus/snmp_exporter) allows
monitoring of devices that support SNMP.

### 我可以监视批处理作业？

Yes, using the [Pushgateway](/docs/instrumenting/pushing/). See also the
[best practices](/docs/practices/instrumentation/#batch-jobs) for monitoring batch
jobs.

### 哪些应用程序可以 Prometheus 监控开箱？

See [the list of exporters and integrations](/docs/instrumenting/exporters/).

### 我可以监视通过 JMX JVM 的应用程序？

Yes, for applications that you cannot instrument directly with the Java client, you can use the [JMX Exporter](https://github.com/prometheus/jmx_exporter)
either standalone or as a Java Agent.

### 什么是仪器的性能有何影响？

Performance across client libraries and languages may vary. For Java,
[benchmarks](https://github.com/prometheus/client_java/blob/master/benchmark/README.md)
indicate that incrementing a counter/gauge with the Java client will take
12-17ns, depending on contention. This is negligible for all but the most
latency-critical code.

## 故障排除

### 我的 Prometheus1.x 的服务器需要很长的时间来启动或垃圾邮件日志提供有关崩溃恢复丰富的信息。

You are suffering from an unclean shutdown. Prometheus has to shut down cleanly
after a `SIGTERM`, which might take a while for heavily used servers. If the
server crashes or is killed hard (e.g. OOM kill by the kernel or your runlevel
system got impatient while waiting for Prometheus to shutdown), a crash
recovery has to be performed, which should take less than a minute under normal
circumstances, but can take quite long under certain circumstances. See
[crash recovery](/docs/prometheus/1.8/storage/#crash-recovery) for details.

### 我的 Prometheus1.x 的服务器运行的内存不足。

See [the section about memory usage](/docs/prometheus/1.8/storage/#memory-usage)
to configure Prometheus for the amount of memory you have available.

### 我的 Prometheus1.x 的服务器报告是在“冲模式”或“存储需求节流”。

Your storage is under heavy load. Read
[the section about configuring the local storage](/docs/prometheus/1.8/storage/)
to find out how you can tweak settings for better performance.

## 履行

### 为什么所有的样本值 64 位浮点？我想整数。

We restrained ourselves to 64-bit floats to simplify the design. The
[IEEE 754 double-precision binary floating-point
format](http://en.wikipedia.org/wiki/Double-precision_floating-point_format)
supports integer precision for values up to 2<sup>53</sup>. Supporting
native 64 bit integers would (only) help if you need integer precision
above 2<sup>53</sup> but below 2<sup>63</sup>. In principle, support
for different sample value types (including some kind of big integer,
supporting even more than 64 bit) could be implemented, but it is not
a priority right now. A counter, even if incremented one million times per
second, will only run into precision issues after over 285 years.

### 为什么不 Prometheus 服务器组件支持 TLS 或认证？我可以添加那些？

Note: The Prometheus team has changed their stance on this during its development summit on
August 11, 2018, and support for TLS and authentication in serving endpoints is now on the
[project's roadmap](/docs/introduction/roadmap/#tls-and-authentication-in-http-serving-endpoints).
This document will be updated once code changes have been made.

While TLS and authentication are frequently requested features, we have
intentionally not implemented them in any of Prometheus's server-side
components. There are so many different options and parameters for both (10+
options for TLS alone) that we have decided to focus on building the best
monitoring system possible rather than supporting fully generic TLS and
authentication solutions in every server component.

If you need TLS or authentication, we recommend putting a reverse proxy in
front of Prometheus. See, for example [Adding Basic Auth to Prometheus with
Nginx](https://www.robustperception.io/adding-basic-auth-to-prometheus-with-nginx/).

This applies only to inbound connections. Prometheus does support
[scraping TLS- and auth-enabled targets](/docs/operating/configuration/#%3Cscrape_config%3E), and other
Prometheus components that create outbound connections have similar support.
