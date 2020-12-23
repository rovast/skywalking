# Observability Analysis Language（OAL）

OAL(Observability Analysis Language) —— 使用流模式分析数据。

OAL 聚焦服务、服务实例以及端点的指标。基于此，这种语言是容易学习和使用的。

从 6.3 开始，OAL 引擎被内置在后端分析平台（OAP）的运行时（即：`oal-rt` ,OAL 运行时）。
OAL 脚本存放在 `/config` 目录下，你可以对它进行修改，并且重启服务器来使其生效。
但是始终要记住，OAL 脚本是个编译语言，OAL 运行时会动态生成 java 字节码。

你可以在系统环境变量里设置 `SW_OAL_ENGINE_DEBUG=Y` 来查看生成了哪些 class。

## 语法

脚本文件命名格式：`*.oal`
```
// 声明 Metrics
METRICS_NAME = from(SCOPE.(* | [FIELD][,FIELD ...]))
[.filter(FIELD OP [INT | STRING])]
.FUNCTION([PARAM][, PARAM ...])

// 关闭通过硬编码定义的处理流
disable(METRICS_NAME);
```

## Scope

一级 **SCOPE** 有 `All`, `Service`, `ServiceInstance`, `Endpoint`, `ServiceRelation`, `ServiceInstanceRelation`, `EndpointRelation`。

同时也有一些二级 scopes，他们属于一级 scope。                        

阅读 [Scope 定义](scope-definitions.md)，你可以找到所有现有的 Scopes 和 Fields。


## Filter

使用 filter 来构建查询条件，进而筛选指定的字段值。你可以根据字段 name 和表达式来筛选。

表达式关系支持： `and`, `or` and `(...)`。

操作符支持： `==`, `!=`, `>`, `<`, `>=`, `<=`, `in [...]` ,`like %...`, `like ...%` , `like %...%` , `contain` 和 `not contain`，
根据字段类型（field type）来进行类型检测。 如果出现不兼容情况会触发编译错误或者代码生成错误。
## Aggregation Function

默认的函数由 SkyWalking OAP 核心来提供，而且还可以拓展。

提供的函数如下

- `longAvg`。每个 scope 实体所有 input 的平均值，其中 input 字段必须是 long 类型。
> 举例：instance_jvm_memory_max = from(ServiceInstanceJVMMemory.max).longAvg();

在这个例子中，input 是每个 ServiceInstanceJVMMemory scope 的请求，avg 基于字段 `max`。

- `doubleAvg`。每个 scope 实体所有 input 的平均值，其中 input 字段必须是 double 类型。
> instance_jvm_cpu = from(ServiceInstanceJVMCPU.usePercent).doubleAvg();

在这个例子中，input 是每个 ServiceInstanceJVMCPU scope 的请求，avg 基于字段 `usePercent`。

- `percent`。符合条件的百分比，保留小数点后两位。
> endpoint_percent = from(Endpoint.*).percent(status == true);

在这个例子中，所有的 input 是每个 endpoint 的请求，条件是 `endpoint.status == true`。

- `rate`。符合条件的比例，保留小数点后两位。
> browser_app_error_rate = from(BrowserAppTraffic.*).rate(trafficCategory == BrowserAppTrafficCategory.FIRST_ERROR, trafficCategory == BrowserAppTrafficCategory.NORMAL);

在这个例子中，input 是每个浏览器 app 的流量，`numerator` 条件时 `trafficCategory == BrowserAppTrafficCategory.FIRST_ERROR`， `denominator` 条件时 `trafficCategory == BrowserAppTrafficCategory.NORMAL`。
第一个参数是 `numerator` 条件。
第二个参数是 `denominator` 条件。

- `sum`。每个 scope 实体的请求量求和。
> service_calls_sum = from(Service.*).sum();

在这个例子里，就是每个服务的调用量。

- `histogram`。阅读 [Heatmap 的维基百科](https://en.wikipedia.org/wiki/Heat_map)
> all_heatmap = from(All.latency).histogram(100, 20);

在这个例子里，就是所有请求的热力图。
参数 (1) 是延迟计算的精度，比如上面的例子， 113ms 和 193ms 被当成 101-200ms group。
参数 (2) 是 group 的数量。在上面的例子里，21(param value + 1) groups 就是 0-100ms, 101-200ms, ... 1901-2000ms, 2000+ms 

- `apdex`。阅读 [Apdex 的维基百科](https://en.wikipedia.org/wiki/Apdex)
> service_apdex = from(Service.latency).apdex(name, status);

在这种情况下，dpdex 就是每个服务的得分。
参数 (1) 是服务名称，会影响到 Apdex 阈值。从 `config` 目录的 `service-apdex-threshold.yml` 加载。
参数 (2) 是请求的状态。状态(success/failure) 影响 Apdex 的计算。

- `p99`, `p95`, `p90`, `p75`, `p50`。[percentile 的维基百科](https://en.wikipedia.org/wiki/Percentile)
> all_percentile = from(All.latency).percentile(10);

**percentile** 是第一个多 metrics 值，在 7.0.0 引入。 
他有多个值，可以通过 `getMultipleLinearIntValues` GraphQL 查询获取。
在这种情况下，计算请求的 `p99`, `p95`, `p90`, `p75`, `p50`。
这个参数是 p99 延迟计算的精度，比如上面的例子，120ms 和 124 是同样的处理。
再 7.0.0 之前，使用 `p99`, `p95`, `p90`, `p75`, `p50` 函数来单独计算每个指标。
在 7.x 里依然支持，但是不推荐这么使用了。并且在官方的 OAL 脚本里已经没有了。

> all_p99 = from(All.latency).p99(10);

在这种情况下，计算请求的 p99 值。这个参数是 p99 计算的精度，和上面一样，120ms 和 124 是当成一样的。

## Metrics name

Metrics name 用于存储管理、告警和查询模块。他的类型由核心系统来自动推断。

## Group

所有的指标数据会根据 Scope.ID 和 分钟级别的时间来分组。

- 在 `Endpoint` scope里，Scope.ID = Endpoint id (这是基于 service 和服务的 Endpoint 的唯一 id)

## Disable

`Disable` 是 OAL 里的高级语句，只用在特定的场景下。
有些聚合和指标是硬编码的，使用 `disable` 可以使这些硬编码指标不生效。
比如 `segment`, `top_n_database_statement`。
默认情况下，没有硬编码被 disable。

## 示例
```
// 计算 Endpoint1 和 Endpoint2 的 p99
endpoint_p99 = from(Endpoint.latency).filter(name in ("Endpoint1", "Endpoint2")).summary(0.99)

// 计算以 `serv` 打头的 Endpoint 名称的 p99 信息
serv_Endpoint_p99 = from(Endpoint.latency).filter(name like "serv%").summary(0.99)

// 计算每个 Endpoint 的平均响应时间
endpoint_avg = from(Endpoint.latency).avg()

// 以 50ms 为间隔，计算每个 Endpoint 的 p50, p75, p90, p95 and p99 信息
endpoint_percentile = from(Endpoint.latency).percentile(10)

// 计算每个服务请求的成功率
endpoint_success = from(Endpoint.*).filter(status == true).percent()

// 计算每个服务的状态码是 404 或 500 或 503 的总和
endpoint_abnormal = from(Endpoint.*).filter(responseCode in [404, 500, 503]).sum()

// 计算服务的请求类型是 RequestType.PRC 或 RequestType.gRPC 的总和
endpoint_rpc_calls_sum = from(Endpoint.*).filter(type in [RequestType.PRC, RequestType.gRPC]).sum()

// Calculate the sum of endpoint name in ["/v1", "/v2"], for each service.
endpoint_url_sum = from(Endpoint.*).filter(endpointName in ["/v1", "/v2"]).sum()

// Calculate the sum of calls for each service.
endpoint_calls = from(Endpoint.*).sum()

// Calculate the CPM with the GET method for each service.The value is made up with `tagKey:tagValue`.
service_cpm_http_get = from(Service.*).filter(tags contain "http.method:GET").cpm()

// Calculate the CPM with the HTTP method except for the GET method for each service.The value is made up with `tagKey:tagValue`.
service_cpm_http_other = from(Service.*).filter(tags not contain "http.method:GET").cpm()

disable(segment);
disable(endpoint_relation_server_side);
disable(top_n_database_statement);
```
