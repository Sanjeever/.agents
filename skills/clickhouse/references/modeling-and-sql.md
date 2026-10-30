# 建模与 SQL 参考

## 目录

- 核心心智模型
- 表引擎与数据布局
- Schema 设计
- 排序键、主键与分区
- SQL 查询写法
- Join、字典与反规范化
- 物化视图、刷新视图与投影
- JSON 与半结构化数据
- 更新、删除与去重

## 核心心智模型

ClickHouse 适合 OLAP：大表、宽表、少量列读取、过滤/聚合/排序、批量写入、可接受最终一致的后台整理。不要把它当作单行事务系统。查询速度来自列式存储、压缩、向量化执行、稀疏主键索引、多核并行、数据跳过和后台 merge。

MergeTree 表由多个 part 组成。每次插入会产生新 part；后台 merge 将小 part 合成大 part，并在过程中重建索引、校验和、压缩文件及可选的跳数索引。读查询在开始时看到一个一致的 part 集合，插入与查询互相隔离。

## 表引擎与数据布局

默认使用 MergeTree 家族：

- `MergeTree`：通用分析表。
- `ReplacingMergeTree`：按排序键保留较新版本，适合插入式更正或 CDC 最终态。读最新态常需要 `FINAL` 或用版本/删除标记设计查询。
- `SummingMergeTree`：后台 merge 时按排序键累加数值列，适合简单计数/求和 rollup。
- `AggregatingMergeTree`：存储聚合状态，配合 `sumState`、`avgState`、`quantileState` 写入和 `sumMerge`、`avgMerge`、`quantileMerge` 查询。
- `Distributed`：自建集群上跨分片查询入口，本身不存数据。
- `Null`：常用于只触发物化视图而不保存原始数据的摄取管道。

典型表：

```sql
create table events
(
    event_time DateTime,
    tenant_id UInt32,
    service LowCardinality(String),
    event_type LowCardinality(String),
    user_id UInt64,
    duration_ms UInt32,
    payload String
)
engine = MergeTree
partition by toYYYYMM(event_time)
order by (tenant_id, event_type, toDate(event_time), user_id);
```

## Schema 设计

先从主查询表开始建模，再补充维表、字典、物化视图和导入管道。

数据类型规则：

- 用严格类型，不要把数值、日期、枚举长期存成 String。
- 使用能覆盖范围的最小整数类型；非负值用 UInt 系列。
- 日期只需要日期时用 `Date`/`Date32`；不需要亚秒时用 `DateTime`，需要更高精度才用 `DateTime64`。
- 低基数字符串或枚举列考虑 `LowCardinality(String)`；稳定有限枚举可用 `Enum8`/`Enum16`。
- 只在业务必须区分 null 与默认值时使用 `Nullable`。Nullable 会增加额外空值标记列，影响存储和查询。
- 固定长度字符串只有在值严格定长时才用 `FixedString`。
- 大文本、JSON 原文、稀疏属性可保留为 String 或 JSON，但热查询字段应抽为强类型列。

用数据抽样评估范围和基数：

```sql
select
    min(value) as min_value,
    max(value) as max_value,
    uniq(value) as unique_values
from source_table;
```

## 排序键、主键与分区

ClickHouse 的 `ORDER BY` 决定磁盘排序和稀疏主键索引。它不是 OLTP 中的唯一约束。默认把“主键”理解为排序键，除非明确讨论 `PRIMARY KEY` 与 `ORDER BY` 分离。

排序键选择：

- 优先放常用于 `WHERE` 且能过滤大量行的列。
- 从低基数到高基数排列通常更利于跳过和压缩，但要服从查询模式。
- 时间列常放在租户、类型等过滤维度之后；若常按时间范围查询，可用 `toDate(event_time)` 等降低索引大小。
- 4 到 5 个关键列通常足够，过长排序键会增加写入、索引和维护成本。
- 排序键创建后不能直接添加；需要重建表或用投影补充另一个物理布局。

分区是数据生命周期工具，不是首选查询优化工具。用低基数分区键，常见是按月/日分区。高基数分区会阻止跨分区 merge，导致 part 爆炸和 `too many parts`。

```sql
partition by toStartOfMonth(event_time)
order by (tenant_id, service, toDate(event_time), trace_id)
```

经验：

- TTL 为天/周级时可按日分区；TTL 为月/年级时按月分区。
- 如果查询经常跨所有分区，分区可能拖慢查询。
- 对删除整段历史数据，`drop partition` 或 TTL 与分区对齐通常优于行级删除。

## SQL 查询写法

优先让过滤条件命中排序键、分区键或可跳过索引。用 `explain indexes = 1` 检查是否跳过 parts/granules。

```sql
explain indexes = 1
select count()
from events
where tenant_id = 42
  and event_type = 'error'
  and event_time >= now() - interval 1 day;
```

常用技巧：

- 只选择需要的列，避免 `select *`。
- 对聚合维度和排序列尽量使用低基数字段或预处理后的字段。
- `prewhere` 可让 ClickHouse 尽早读取过滤列；多数场景优化器会自动下推。
- `group by all` 在示例和探索查询中方便，但生产 SQL 应明确维度更清晰。
- 用 `countIf`、`sumIf`、`avgIf`、`uniqIf` 等组合函数减少多次扫描。
- 近似聚合可用 `uniq`、`quantile` 系列换取速度；需要精确时用精确函数并说明成本。

## Join、字典与反规范化

ClickHouse 支持 Join，但高性能工作负载应减少 Join 数量。目标是让大事实表查询尽量单表完成。

优先级：

1. 能在摄取时反规范化的维度，直接写入事实表。
2. 小型、键值式、最新值维表，用 Dictionary 和 `dictGet`，或让 direct join 生效。
3. 需要多阶段转换或预聚合，用物化视图。
4. 需要常规 Join 时，过滤尽量下推到 Join 两侧，右表尽量小。

Join 建议：

- 避免每个查询超过 3 到 4 个 Join。
- 只需要任意一个匹配时用 `ANY JOIN`。
- 大表 Join 前先用子查询过滤。
- 排序键相同的表可考虑 sort-merge join。
- 默认 `join_algorithm = 'auto'` 可从 hash join 回退到可落盘算法；手动选算法前要基准测试。
- 字典不保留重复键，不适合一对多或多对多完整关系。

字典适合查询时或写入时 enrichment：

```sql
create dictionary users_dict
(
    id UInt64,
    country LowCardinality(String)
)
primary key id
source(clickhouse(query 'select id, country from users'))
lifetime(min 600 max 900)
layout(hashed());

select
    dictGet('users_dict', 'country', user_id) as country,
    count() as event_count
from events
group by country
order by event_count desc;
```

## 物化视图、刷新视图与投影

增量物化视图是插入触发器：源表每插入一个 block，就执行定义的 SELECT，并把结果写入目标表。它把计算成本从查询时移到写入时。

简单 rollup：

```sql
create table events_per_day
(
    day Date,
    tenant_id UInt32,
    event_count UInt64
)
engine = SummingMergeTree
order by (tenant_id, day);

create materialized view events_per_day_mv
to events_per_day as
select
    toDate(event_time) as day,
    tenant_id,
    count() as event_count
from events
group by tenant_id, day;
```

目标表使用 `SummingMergeTree` 或 `AggregatingMergeTree` 时，物化视图的 `GROUP BY` 应与目标表 `ORDER BY` 对齐。聚合状态示例：

```sql
create table latency_per_day
(
    day Date,
    service LowCardinality(String),
    p99_state AggregateFunction(quantile(0.99), UInt32),
    avg_state AggregateFunction(avg, UInt32)
)
engine = AggregatingMergeTree
order by (service, day);

create materialized view latency_per_day_mv
to latency_per_day as
select
    toDate(event_time) as day,
    service,
    quantileState(0.99)(duration_ms) as p99_state,
    avgState(duration_ms) as avg_state
from events
group by service, day;

select
    day,
    service,
    quantileMerge(0.99)(p99_state) as p99_duration_ms,
    avgMerge(avg_state) as avg_duration_ms
from latency_per_day
group by day, service;
```

刷新物化视图适合周期性重算、复杂 Join、快照式报表和允许延迟的 denormalization。增量视图适合实时 rollup、过滤、转换和多阶段摄取。

投影是同一张表上的隐藏物理布局或预聚合，查询可由优化器自动选择。适合单表不同过滤顺序或聚合布局，不适合 Join、复杂 ETL、显式过滤物化、链式处理、依赖 `FINAL` 的查询或非 MergeTree 表。

选择规则：

- 需要显式目标表、跨表转换、过滤、链式管道：物化视图。
- 想让查询继续打原表、只优化单表访问路径：投影。
- 查询模式稳定且聚合结果很小：物化视图目标表。
- 主键已经不能改，但需要第二种排序：投影或重建表。

## JSON 与半结构化数据

区分 JSON 数据格式与 JSON 列类型。能用强类型列表达的字段，不要因为输入是 JSON 就使用 JSON 类型。

建模规则：

- 静态结构：递归展开为普通列、`Tuple`、`Array(Tuple(...))`、`Nested`。
- 动态结构：使用 `JSON` 类型，并为热路径字段提供类型提示。
- 已知但稀疏的字段：普通列加默认值通常比 Nullable 更快。
- 不需要的路径用 `SKIP` 或 `SKIP REGEXP`，避免浪费存储和推断成本。
- 控制 `max_dynamic_paths`，不要无界放大。
- JSON 子列可参与排序键，但必须给出明确类型，避免歧义。

示例：

```sql
create table app_events
(
    event_time DateTime,
    tenant_id UInt32,
    user_id UInt64,
    attrs JSON(user_agent String, country String)
)
engine = MergeTree
order by (tenant_id, attrs.country, toDate(event_time));
```

## 更新、删除与去重

ClickHouse 的 mutation 会重写数据 part，代价可能很高。默认不要把频繁更新设计成 `alter table ... update/delete`。

替代方案：

- 插入新版本，使用 `ReplacingMergeTree(version)` 或查询时取最新版本。
- 插入删除标记，查询过滤 `is_deleted = 0`，后台或离线清理。
- 对时间保留用 TTL。
- 对整段数据用 `drop partition`。
- 对大批修正用重建新表、交换表名或分区级重灌。

`FINAL` 可在查询时合并最终态，但会增加开销。生产查询应尽量通过排序键、版本字段、预聚合或离线合并降低对 `FINAL` 的依赖。
