# MAL —— 度量分析语言(Meter Analysis Language)

度量系统提供了一种可编程式的分析语言 —— MAL(Meter Analysis Language)。它允许用户在 OAP 流系统里分析和聚合度量数据。
表达式的结果，可以由 agent 分析器或 OC/Prometheus 分析器读取。

## 语言数据类型

在 MAL 中，一个表达式或子表达式对以下两种数据类型进行求值：
 - 采样族 -  一组包含一系列名称相同的度量的样本（度量）。
 - 标量 - 简单的数值，支持 integer/long, floating/double。

## 采样族

一组采样，是 MAL 的基本组成单元，比如：

```
instance_trace_count
```

上面的采样族可能包含以下简单采样，这些采样由外部模块（如，instance、agent 分析器）提供：

```
instance_trace_count{region="us-west",az="az-1"} 100
instance_trace_count{region="us-east",az="az-3"} 20
instance_trace_count{region="asia-north",az="az-1"} 33
```

### Tag 过滤器

在一个采样族中，MAL 支持四中类型操作来过滤采样

 - tagEqual: 根据提供的 string 来严格匹配 tag
 - tagNotEqual: tag 和提供的 string 不相等
 - tagMatch: 满足正则匹配的 string 和 tag
 - tagNotMatch: 不满足正则匹配的 string 和 tags

举个例子，筛选所有的 instance_trace_count 采样，他们的条件是：地区是 us-west 和 asia-north，并且 az 是 az-1

```
instance_trace_count.tagMatch("region", "us-west|asia-north").tagEqual("az", "az-1")
```

### 二元操作符

MAL 有以下二元操作：

 - \+ (加)
 - \- (减)
 - \* (乘)
 - / (除)

二元操作符定义在以下的一对值之间： scalar/scalar, sampleFamily/scalar 以及 sampleFamily/sampleFamily。


标量/标量：对各自标量的运算
```
1 + 2
```

采样族/标量：即对每个采样族的采样都进行对应的运算
```
instance_trace_count + 2
``` 

或
```
2 + instance_trace_count
``` 

等于如下的运算，即：每个采样都和标量进行运算
```
instance_trace_count{region="us-west",az="az-1"} 102 // 100 + 2
instance_trace_count{region="us-east",az="az-3"} 22 // 20 + 2
instance_trace_count{region="asia-north",az="az-1"} 35 // 33 + 2
```

Between two sample families, a binary operator is applied to each sample in the left-hand side sample family and 
its matching sample in the right-hand sample family. A new sample family with empty name will be generated.
Only the matched tags will be reserved. Samples for which no matching sample in the right-hand sample family are not in the result.

Another sample family `instance_trace_analysis_error_count` is 

```
instance_trace_analysis_error_count{region="us-west",az="az-1"} 20
instance_trace_analysis_error_count{region="asia-north",az="az-1"} 11 
```

Example expression:

```
instance_trace_analysis_error_count / instance_trace_count
```

This returns a result sample family containing the error rate of trace analysis. The samples with region us-west and az az-3 
have no match and will not show up in the result:

```
{region="us-west",az="az-1"} 0.8  // 20 / 100
{region="asia-north",az="az-1"} 0.3333  // 11 / 33
```

### Aggregation Operation

Sample family supports the following aggregation operations that can be used to aggregate the samples of a single sample family,
resulting in a new sample family of fewer samples(even single one) with aggregated values:

 - sum (calculate sum over dimensions)
 - min (select minimum over dimensions) (TODO)
 - max (select maximum over dimensions) (TODO)
 - avg (calculate the average over dimensions) (TODO)
 
These operations can be used to aggregate over all label dimensions or preserve distinct dimensions by inputting `by` parameter. 

```
<aggr-op>(by: <tag1, tag2, ...>)
```

Example expression:

```
instance_trace_count.sum(by: ['az'])
```

will output a result:

```
instance_trace_count{az="az-1"} 133 // 100 + 33
instance_trace_count{az="az-3"} 20
```

### Function

`Duraton` is a textual representation of a time range. The formats accepted are based on the ISO-8601 duration format {@code PnDTnHnMn.nS}
 with days considered to be exactly 24 hours.

Examples:
 - "PT20.345S" -- parses as "20.345 seconds"
 - "PT15M"     -- parses as "15 minutes" (where a minute is 60 seconds)
 - "PT10H"     -- parses as "10 hours" (where an hour is 3600 seconds)
 - "P2D"       -- parses as "2 days" (where a day is 24 hours or 86400 seconds)
 - "P2DT3H4M"  -- parses as "2 days, 3 hours and 4 minutes"
 - "P-6H3M"    -- parses as "-6 hours and +3 minutes"
 - "-P6H3M"    -- parses as "-6 hours and -3 minutes"
 - "-P-6H+3M"  -- parses as "+6 hours and -3 minutes"

#### increase
`increase(Duration)`. Calculates the increase in the time range.

#### rate
`rate(Duration)`. Calculates the per-second average rate of increase of the time range.

#### irate
`irate()`. Calculates the per-second instant rate of increase of the time range.

#### tag
`tag({allTags -> })`. Update tags of samples. User can add, drop, rename and update tags.

#### histogram
`histogram(le: '<the tag name of le>')`. Transforms less based histogram buckets to meter system histogram buckets. 
`le` parameter hints the tag name of a bucket. 

#### histogram_percentile
`histogram_percentile([<p scalar>])`. Hints meter-system to calculates the p-percentile (0 ≤ p ≤ 100) from the buckets. 

#### time
`time()`. returns the number of seconds since January 1, 1970 UTC.

## Down Sampling Operation
MAL should instruct meter-system how to do downsampling for metrics. It doesn't only refer to aggregate raw samples to 
`minute` level, but also hints data from `minute` to higher levels, for instance, `hour` and `day`. 

Down sampling operations are as global function in MAL:

 - avg
 - latest (TODO)
 - min (TODO)
 - max (TODO)
 - mean (TODO)
 - sum (TODO)
 - count (TODO)

The default one is `avg` if not specific an operation.

If user want get latest time from `last_server_state_sync_time_in_seconds`:

```
latest(last_server_state_sync_time_in_seconds.tagEqual('production', 'catalog'))

or

latest last_server_state_sync_time_in_seconds.tagEqual('production', 'catalog')
```

## Metric level function

Metric has three level, service, instance and endpoint. They extract level relevant labels from metric labels, then
 hints meter-system which level this metrics should be.

 - `servcie([svc_label1, svc_label2...])` extracts service level labels from the array argument.
 - `instance([svc_label1, svc_label2...], [ins_label1, ins_label2...])` extracts service level labels from the first array argument, 
                                                                        extracts instance level labels from the second array argument.
 - `endpoint([svc_label1, svc_label2...], [ep_label1, ep_label2...])` extracts service level labels from the first array argument, 
                                                                      extracts endpoint level labels from the second array argument.

## More Examples

Please refer to [OAP Self-Observability](../../../oap-server/server-bootstrap/src/main/resources/fetcher-prom-rules/self.yaml)
