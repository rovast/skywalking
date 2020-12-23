# 数据分析平台（OAP）
OAP(Observability Analysis Platform) 是一个新概念，自 SkyWalking 6.x 提出。
OAP 替换了以前的 SkyWalking 后端。 OAP 提供的能力如下：
## OAP 能力

OAP 从更多的数据源来接收数据，主要分为两大类：**Tracing** 和 **Metrics**。

- **Tracing**。包括 SkyWalking 数据格式、Zipkin v1,v2 数据格式以及 Jaeger 数据格式。
- **Metrics**。SkyWalking 融合了 Service Mesh platforms 平台（如 Istio, Envoy, Linkerd）来从数据平面或控制面来提供可观测性。
同时， SkyWalking 的探针也可以在 metrics 模式下高性能运行。

在提供了集成方案（如 SkyWalking 日志插件或工具集）的同时，SkyWalking 结合 trace id 和 span id 提供了 tracing 和 logging 的可视化聚合。

一般情况下，服务使用 gRPC 或者 HTTP 协议来提供，使不支持的生态融合起来更加简单。

## Tracing in OAP

OAP 里有两种 Tracing 处理
1. SkyWalking 5 系列的传统处理方式。在 SkyWalking 的 trace 段里格式化 tracing 数据，以及格式化 span 数据，甚至格式化 Zipkin 的数据格式。
OAP 分析这些数据段来拿到 metrics 数据，并且把 metrics 数据发送到流式聚合系统里。
2. 把 tracing 信息当成是日志的一种，并且只为 trace 提供保存和可视化能力。

SkyWalking 也可以接收其他项目的数据格式，比如 Zipkin, Jaeger, OpenCensus。这些数据格式也能以上述的两种方式来处理。

## Metrics in OAP


Metrics in OAP 是 SkyWalking 6 系列的一个新特性。
基于相连节点的 metrics 数据，我们可以为分布式系统构建可观测能力。不需要再依赖于 tracing 数据。

Metrics 数据在 OAP 集群里是以流模式来聚合的，具体可以阅读 [Observability Analysis Language](oal.md)，
这种方式提供了一种更方便的、以脚本形式的聚合分析指标数据的形式。
