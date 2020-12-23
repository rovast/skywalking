# Service Mesh 探针

Service Mesh 探针使用了可拓展机制，比如 Istio。

## 什么是 Service Mesh？

Istio 的文档是这么解释的
> service mesh 这个属于一般是用来描述微服务网络，这个网络用来把各个服务紧密联系在一起。（
> The term service mesh is often used to describe the network of microservices that make up such applications and the interactions between them.）

随着服务网格的的体积和复杂度上升，理解和管理的难度会越来越大。
他的需求包括：服务发现、复杂均衡、错误恢复、指标、监控，还有其他复杂的运维需求，比如：A/B 测试、金丝雀发布、限流、访问控制，端到端认证等。

## 探针从哪里采集数据？

Istio 是一个经典的 Service Mesh 设计和实现。他定义了 **控制面** 和 **数据面**，并得到广泛的应用。下面是 Istio 的架构：

![Istio Architecture](https://istio.io/latest/docs/ops/deployment/architecture/arch.svg)

Service  Mesh 的探针可以选择从 **数据平面** 来采集数据。在 Istio 中，就是从数据面 Envoy sidecar 采集遥测数据。
在单个请求中，探针采集两个遥测实体：client 端和 server 端。

## Service Mesh 是如何使 backend 工作的？

从上面的探针来看，这种类型的探针没有 trace 相关的数据，那 SkyWalking 是如何工作的？

Service Mesh 探针从每个请求里采集遥测数据，并且他知道源、目标、端点、延迟和状态。
基于这些，后端可以知晓整个链路图，通过把这些数据组成一条条数据，以及即将到来请求的每个节点的指标信息。
后端在解析 trace 数据时一样也是需要这些指标数据。
所以，正确的描述是：**Service Mesh 指标就是 trace 需要聚合的指标，他们是一样的**

## 接下来
- 如果你想使用 Service Mesh 探针，阅读 [在 Service Mesh 上配置 SkyWalking](../setup/README.md#on-service-mesh) document.
