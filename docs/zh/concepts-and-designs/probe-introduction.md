# 探针介绍

在 SkyWalking 中，探针可以是 agent 或者 sdk，用来承担采集遥测数据（包括：链路数据和指标数据）的职责。
你可以根据目标系统的技术栈，来采用不同的探针实现。
但不管咋样，他们做的都是同样的事情：收集、格式化数据，并把数据发送给后端。

从宏观层面看，所有的 SkyWalking 探针被分为三类。
- **语言类探针**。
这种类型的探针运行在目标服务的用户控件，并且作为用户代码的一部分。
比如 SkyWalking 的 Java 探针，使用 `-javaagent` 命令来在运行时注入代码。
另外一种探针是使用 hook 或者拦截机制来实现。所以说，这一类的探针基本都是基于特定语言或者库来实现。

- **Service Mesh 探针**。
Service Mesh 的探针从 sidecar、控制面或者代理来采集数据。
以前，代理一般都是用集群的 ingress，不过现在基于 Service Mesh 和 sidecar 我们也可以进行观测了。

- **第三方类库实现**。
SkyWalking 也接收其他主流库的数据格式。他可以分析这些数据并且转换为符合 SkyWalking 的数据格式。
我们先支持了 Zipkin 的 span 数据。查看
[Receiver for other tracers](../setup/backend/backend-receivers.md) 了解更多。

你不需要同时使用 **语言探针** 和 **Service Mesh 探针**，因为他们两者都可以采集数据， 
如果你非要这么干，那你的系统就会承受两次请求，并且分析的数据啥的，也会重复。

以下是使用探针的几种建议方式：
1. 仅使用**语言探针**。
2. 仅使用**第三方实现类库**。比如 Zipkin 生态。
3. 仅使用 **Service Mesh 探针**。
4. 结合 **语言探针** 和 **第三方类库** 来实现 **Service Mesh 探针**，进而 **tracing status** (高阶用法)。

最后，我们来举个例子说明什么是 **in tracing status**？

一般情况下，**语言探针** 和 **第三方库** 都能够发送 Trace 数据到后端，然后后端分析和聚合这些 Trace 数据。
**In tracing status** 就是说，后端把这些 Trace 信息看成是 Logs，只是保存他们，并且在 traces 和 metrics 之间建立联系就像 `这个链路属于哪一个服务或端点？`.

## 接下来
- 了解 SkyWalking 支持的探针：[服务探针](service-agent.md), [手动 SDK 探针](manual-sdk.md),
[Service Mesh 探针](service-mesh-probe.md) 和 [Zipkin receiver](trace-receiver.md).
- 了解完探针之后，阅读 [后端概述](backend-overview.md) 来了解数据的分析和持久化。
