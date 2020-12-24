# 度量系统(Meter System)

度量系统是另一种流计算模型，针对 metrics 数据设计。
在 [OAL](oal.md)中，有明确的 [Scope 定义](scope-definitions.md)，还有自带的一些 objects。
度量系统聚焦数据类型，并且提供了更遍历的方式，让使用者来定义 scope 实体。

在后端实现里，度量系统可以对接不同的 receivers 和 fetchers，查阅 [后端设置文档](../setup/backend/backend-setup.md) 了解更多。

每个在度量系统中声明的 metrics 都需要包含以下的属性：
1. **Metrics Name**。这是一个全局的唯一 name，并且不要和 OAL 的变量名重复。
2. **Function Name**。这个函数用户 metrics 的分布式聚合、值的计算、以及采样率计算。并且根据函数的不同，需要不同的数据结构，比如 avg 函数，他的参数需要的是 `Long` 类型。
3. **Scope Type**。不像 OAL 那样有大量的逻辑 scope 定义，在度量系统里，只有 type。Type 的值包括：service, instance, 和 endpoint。

scope entity 的 name 值，比如像 service name，在 metrics data value 生成 metrics data 时是必须的。

注意，metrics 必须在项目启动阶段就声明好，在运行时是不能修改的。

度量系统支持以下的绑定函数：
- **avg**。计算同一个 metrics 下的所有 entity 的平均值。
- **histogram**。对可配置的 buckets 数量进行聚合，buckets 是可配置的，但是必须在声明阶段指定。
- **percentile**. 阅读 [percentile 维基百科](https://en.wikipedia.org/wiki/Percentile)。不像 OAL，我们在度量系统里函数里默认提供了 50/75/90/95/99， percentile 接受 0 -  100 的排名。
