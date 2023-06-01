## Promql 示例

### topk

```bash
# topk... sum... by...
topk(5, sum(CpuPercent{__DATABASE__="db1", aggr="avg"}) by (ip))
```

### 均值

```bash
# 所有实例的平均协程数
avg(go_goroutines)
# 按时间聚合
avg(avg_over_time(go_goroutines[1m]))
```

### 求和

```bash
# 维度聚合
sum(increase(prometheus_http_requests_total[5m]))
sum(increase(events_total{__DATABASE__='db1'}[5m])) by (object_kind)

# sum_over_time(range-vector): 按时间段求和
(sum(sum_over_time(metric1{__DATABASE__="db1",aggr="avg"}[1d]))*60)/3*20/256
(sum(sum_over_time(metric1{__DATABASE__="db1",aggr="avg"}[1d]))*60)/3*(20/256)

```

### 递增

```bash
# 过去一小时内每5分钟有多少次请求可以这么写。这里使用更常用的increase函数和range query
increase(metric1{__DATABASE__='db1'}[5m])
``` 

### 分组

```bash
# sum... by...
sum(increase(events_total{__DATABASE__='db1'}[5m])) by (group_field)

# 以IP分组
sum(CpuPercent{__DATABASE__="db1", aggr="avg"}) by (ip)
```

### 线性预测

```bash
# 利用过去30分钟的数据预测1小时后的数据
predict_linear(disk_used_bytes[30m], 3600)
```

### 正则


```bash
# =：选择与提供的字符串完全相等的标签。
# !=：选择不等于提供的字符串的标签。
# =~：选择与提供的字符串进行正则表达式匹配的标签。
# !~：选择与提供的字符串不匹配的标签。
http_requests_total{environment=~"staging|testing|development",method!="GET"}

# 正则表达式匹配完全锚定。env=~"foo"被视为env=~"^foo$"。
http_requests_total{env=~"foo",method!="GET"}
# ip范围
MemUsedPercent{__DATABASE__="d1",ip=~"(11\\.22\\.2(4|5|6|7)\\..*)"}[1m:1m]
```

### dalta

```bash
# 计算 CPU 温度在两小时内的差异
dalta(cpu_temp_celsius{host="zeus"}[2h])
```


### 参考

- [查询示例 - prometheus.io ](https://prometheus.io/docs/prometheus/latest/querying/examples/) 
- [基础 - prometheus.io](https://prometheus.io/docs/prometheus/latest/querying/basics/)

