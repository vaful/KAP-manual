## 查询引擎

**查询路由是如何工作**

KAP支持三种查询引擎：Cube引擎，表索引（Table Index）引擎，下压（Pushdown）引擎

Cube引擎是被广泛使用的，为聚合类查询所设计的查询引擎，用于OLAP分析场景。

表索引引擎是列式存储引擎，为明细查询场景设计。在分析场景中，用户可以通过钻取聚合数据到最底层的明细数据。

下压引擎是其他SQL on Hadoop引擎，包括Hive，SparkSQL，Impala等。当某个查询不适合预计算引擎时，查询会被下压到其他下游查询引擎。这种情况下，查询延迟通常会延长到分钟级。

那么这三种引擎如何一起工作呢？查询的优先级如何定义？查询路由是如何工作的呢？

有两种类型的查询，聚合查询和明细查询，带有“group by“的是聚合查询，其他查询是明细查询。

对于聚合查询，KAP按照顺序：Cube引擎 > 表索引引擎 > 下压引擎

对于明细查询，KAP按照顺序：表索引引擎 > Cube引擎 > 下压引擎

每个查询引擎都可以独立地启动和关闭查询能力。如果表索引在Cube设计阶段并没有配置，则表索引引擎会忽略所有的查询。在某些场景，如果下压引擎没有通过配置文件启动，则查询也会被下压引擎忽略。Cube也可以通过配置参数`kylin.query.disable-cube-noagg-sql`为true，关闭Cube对明细查询的处理。

每个引擎都有自己的查询能力（通过维度、指标、列定义），如果查询与当前引擎不匹配，则查询会被路由到下一个引擎。