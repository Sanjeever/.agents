---
name: clickhouse
description: ClickHouse 文档知识技能，用于回答、起草、审查或实现 ClickHouse 相关方案。适用于 OLAP/实时分析、SQL 写法、表设计、MergeTree、主键/分区/TTL、物化视图、投影、字典、JSON/Parquet/S3/Kafka 摄取、ClickPipes、ClickHouse Cloud、备份恢复、Keeper/分片复制、性能调优、可观测性、chDB、迁移和集成问题。
---

# ClickHouse

使用本技能时，把 ClickHouse 当作面向大规模分析的列式数据库，而不是 OLTP 数据库。优先围绕查询模式、数据生命周期和摄取方式设计表结构；不要套用传统关系型数据库的主键、事务和频繁行级更新思路。

## 工作流程

1. 明确运行环境：ClickHouse Cloud、自建 ClickHouse、clickhouse-local、chDB，或 ClickPipes/托管集成。Cloud 与自建在复制、存储、备份、网络、安全和运维动作上差异很大。
2. 明确工作负载：实时仪表盘、日志/可观测性、时间序列、CDC、数据湖查询、批量导入、向量/AI、嵌入式分析等。
3. 先确定查询模式，再确定 schema：过滤列、聚合维度、时间范围、保留周期、更新/删除频率、Join 形态、摄取吞吐和延迟目标。
4. 用 MergeTree 家族作为默认表引擎。Cloud 中普通 MergeTree DDL 通常会映射到 SharedMergeTree 家族；自建高可用场景使用 ReplicatedMergeTree 与 Keeper。
5. 先给出可运行的最小方案，再说明性能和运维取舍。涉及生产环境时，补充监控点和失败模式。

## 引用资料

按任务读取对应引用文件，不要一次性加载全部：

- 表设计、SQL、物化视图、投影、字典、JSON、Join：读 `references/modeling-and-sql.md`。
- 数据导入、格式、S3/GCS/Azure、Kafka、ClickPipes、语言客户端和 BI 集成：读 `references/ingestion-and-integrations.md`。
- 性能调优、parts/merges、分区、TTL、备份恢复、Keeper、分片复制、故障排查：读 `references/performance-and-operations.md`。
- ClickHouse Cloud、SharedMergeTree、Cloud 备份/网络/安全、chDB、典型用例：读 `references/cloud-chdb-use-cases.md`。

## 默认立场

- 对分析查询，优化目标通常是减少读取列、跳过 granule/part、降低解压和聚合成本、让后台 merge 承担可延迟工作。
- 对写入，优先批量插入，避免小批量高频插入造成 too many parts。客户端直写时选择 native 或 HTTP；Cloud 托管摄取优先考虑 ClickPipes。
- 对 schema，优先严格类型、低精度数值/时间类型、必要时 LowCardinality/Enum；避免无必要的 Nullable 和泛化 String。
- 对查询加速，先选好 ORDER BY，再考虑物化视图、投影、跳数索引、字典、预聚合和数据重排。
- 对更新删除，默认认为代价高。能用插入式修正、ReplacingMergeTree、轻量删除、TTL 或重建分区解决时，不要先设计大量 mutation。
- 对 Join，默认建议减少 Join；小维表用字典或 direct join，复杂多表分析优先反规范化、物化视图或刷新视图。

## SQL 风格

写示例 SQL 时使用小写下划线别名，例如 `event_count`、`avg_latency_ms`。避免使用驼峰别名。生产示例要包含关键 `ENGINE`、`ORDER BY`、必要的 `PARTITION BY` 和生命周期策略。
