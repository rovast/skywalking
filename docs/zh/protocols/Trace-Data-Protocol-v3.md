# Trace Data Protocol v3

Trace 数据协议约定了 SkyWalking 和后端分析系统的数据格式。

## 概述

Trace 数据格式可以用两种协议来进行通信
- [gRPC format](https://github.com/apache/skywalking-data-collect-protocol)
- [HTTP 1.1](HTTP-API-Protocol.md)

### 上报 service instance 的状态

1. Service Instance 属性

Service instance 除了 name 之外，还有更多的信息，如果探针想要上报此类信息，使用 `ManagementService#reportInstanceProperties` 服务，
提供 string-key/string-value 对作为参数。 `language` of target instance is expected at least.

2. Service Ping

Service instance 需要和后端系统保持连接。探针每分钟调用 `ManagementService#keepAlive` 来保持连接。

### 上报 trace 和 metrics 信息

有了 service id 和 service instance id 之后，我们就可以发送 traces 和 metrics 数据了，目前我们有
1. `TraceSegmentReportService#collect`，用于 skywalking 的 trace 格式
2. `JVMMetricReportService#collect`，用于 skywalking 的 jvm 格式

> SegmentObject 定义 https://github.com/apache/skywalking-data-collect-protocol/blob/master/language-agent/Tracing.proto

关于 trace 的格式，有以下几点需要注意：

1. Segment 是 SkyWalking 里的概念，应该包含单个 OS 进程中每个请求的所有 span。通常是基于语言的单个线程。
2. Span 有三组不同信息

   * EntrySpan，代表一个服务提供端或者服务端入口。作为一个 APM 系统，我们的目标是应用服务器。基本上所有的服务，MQ 消费者都会 EntrySpan。

   * LocalSpan，表示一个普通的 Java 方法，它不是远程 servce，也不是 MQ 的生产者/消费者，也不是服务（如 http 服务）的提供者/消费者。

   * ExitSpan，表示服务的客户端，或者 MQ 的生产者（SkyWalking 早期称之为 `LeafSpan`。比如：DB 组件 JDBC，reading Redis/Memcached are cataloged an ExitSpan. 

3. Span 跨进程或者线程的父级信息称为 Reference，Reference 携带了 trace id, 
segment id, span id, service name, service instance name, endpoint name 以及 target address。这些本次请求的上游客户端信息。

阅读 [Cross Process Propagation Headers Protocol v3](Skywalking-Cross-Process-Propagation-Headers-Protocol-v3.md) 了解更多。

4. `Span#skipAnalysis` 可以设置为 TRUE，这样的话 span 就需要进行后端分析。
