# 概述

SkyWalking 是一个开源的可观测性平台，用来从服务和云原生基础设施收集、分析、聚合以及可视化数据。
SkyWalking 给分布式系统提供了提供了一目了然的视图。
他是一个现代的 APM 系统，尤其是当下的云原生架构、基于分布式容器的架构。

## 为什么使用 SkyWalking？

SkyWalking 给分布式系统提供了观测和监控方案，并且适用于众多场景。
首先，就像传统的处理方式一样，SkyWalking 为服务提供了自动探针，比如：ava, C#, Node.js, Go, PHP 和 Nginx LUA(欢迎贡献 Python 和 C++ 语言探针)。
在多语言、持续部署的大环境下，云原生基础建设变得越来越牛批，同时也变得越来越复杂。
SkyWalking 的 service mesh 接收器可以从 mesh 体系里接收遥测数据，比如说：Istio/Envoy 和 Linkerd。这样可以方便使用者来了解整个分布式系统。

SkyWalking 为 **service**(s), **service instance**(s), **endpoint**(s) 提供了观测能力。术语 Service，Instance 和 Endpoint 现在已经很常见了，我们来看下在 SkyWalking 里这三个属于的具体含义：

- **Service（服务）**。用来处理请求的一系列负载，他们有相同的表现行为。 你可以在 agent 或者 SDK 里定义 Service 名称。SkyWalking 也可以使用你在平台中定义的名称，比如 Istio。
- **Service Instance（服务实例）**。之前讲到的服务里的每一个独立的工作负载就是服务实例。就像 Kubernetes 里的 `pods` 一样，它不一定是一个独立的操作系统进程。不过，如果你使用的是 instrument agents，服务实例一般就是一个 OS 进程。
- **Endpoint（端点）**。一个服务请求的路径，比如 HTTP 的 URI 路径，或者 gPRC 请求的 service class + method signature。

SkyWalking 给用户提供了服务和端点的拓扑关系，可以查看每一个 服务/服务 实例/端点 的指标和一系列告警规则。

除此之外，你还可以在其他系统中集成 SkyWalking：
1. 集合其他的分布式追踪系统使用 SkyWalking 的 agent 和 SDK，比如 Zipkin, Jaeger and OpenCensus。
2. 其他的指标系统：比如 Prometheus, Sleuth(Micrometer)。

## 架构
SkyWalking 在逻辑上分为四大部分：探针、平台后端、存储和 UI。

<img src="http://skywalking.apache.org/assets/frame-v8.jpg?u=20200423"/>

- **探针** 根据 SkyWalking 的需求来采集和格式化数据（不同的探针支持不同的数据源）。
- **平台后端** 支持数据的聚合和分析，作为 UI 和探针的中间层。分析工作包括 SkyWalking 本身的追踪和指标；以及第三方组件，如 Istio 和 Envoy 的遥测数据，Zipkin 的追踪格式化等。你甚至可以自定义聚合和分析数据，参考：[Observability Analysis Language for native metrics](oal.md) 和 [Meter System for extension metrics](meter.md)。
- **存储** 通过 开放/可插拔 的接口来支撑 SkyWalking 数据。你也可以使用其他现有的存储方案，比如：ElasticSearch, H2 或者 Sharding-Sphere 提供的 MySQL 集群。你也可以自己实现，当然我们也欢迎你实现新的存储。
- **UI** 一个高度自定义的 web 平台，可以用来管理 SkyWalking 数据和可视化。

## 接下来
- SkyWalking [项目目标](project-goals.md)
- FAQ, [为什么不在 SkyWalking 架构中引入 MQ](../FAQ/why_mq_not_involved.md)
