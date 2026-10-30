# Cloud、chDB 与典型用例参考

## 目录

- ClickHouse Cloud 心智模型
- SharedMergeTree
- Cloud 备份、扩缩容、网络与安全
- ClickPipes 与 Cloud 摄取
- chDB
- 典型用例建模
- 迁移建议

## ClickHouse Cloud 心智模型

ClickHouse Cloud 把许多自建运维问题托管化：服务创建、扩缩容、备份、对象存储、网络入口、安全能力、ClickPipes 和控制台体验。回答 Cloud 问题时，先确认用户是在 Basic、Scale、Enterprise 或 BYOC/私有连接场景中，因为可用功能、备份配置、网络和合规选项可能不同。

Cloud 默认关注点：

- 不需要用户手动管理 Keeper、复制拓扑、硬件故障和很多存储配置。
- 普通 `MergeTree` DDL 可自动使用 Cloud 原生 SharedMergeTree 家族。
- 计算和存储分离，扩缩容比传统本地复制更轻。
- 备份和托管摄取有独立计费与策略。
- 网络安全通常涉及 allowlist、PrivateLink/Private Service Connect、VPC endpoint、static egress IP 和对象存储 bucket policy。

## SharedMergeTree

SharedMergeTree 是 Cloud 中替代 ReplicatedMergeTree 的云原生 MergeTree 家族，基于共享对象存储和 Keeper 协调元数据。普通用户通常继续写：

```sql
create table events
(
    event_time DateTime,
    id UInt64,
    message String
)
engine = MergeTree
order by (toDate(event_time), id);
```

Cloud 会使用对应的 SharedMergeTree 实现。Replacing、Summing、Aggregating 等 MergeTree 变体也有 Shared 版本。

特性：

- 插入、后台 merge、mutation 和扩缩容吞吐更适合共享存储。
- 副本间不直接复制数据和元数据，而是通过共享存储和 Keeper 协调。
- 插入本质上写入共享存储并提交元数据，不需要像 ReplicatedMergeTree 那样调 `insert_quorum`。
- 多数情况不需要 `select_sequential_consistency`；跨副本读写且必须避免短暂陈旧读时，优先同会话/同节点读，再考虑 `system sync replica lightweight`，最后才用 sequential consistency 设置。

Cloud introspection：

- ReplicatedMergeTree 常见的 `system.replication_queue`、`system.replicated_fetches` 在 SharedMergeTree 中有替代。
- 用 `system.virtual_parts` 查看当前和未来 parts、merge、mutation、drop partition 相关状态。
- 用 `system.shared_merge_tree_fetches` 查看主键和 checksum 拉取状态。

## Cloud 备份、扩缩容、网络与安全

Cloud 备份通常由全量和增量组成 backup chain。默认至少每日备份并保留一定时间；更高级别服务可配置频率和保留期。可导出到自有云账号对象存储，但自有 bucket 的生命周期和成本由用户负责。

备份回答要区分：

- Cloud 托管备份：控制台恢复到新服务。
- Bring Your Own Bucket：导出到 AWS S3、GCS 或 Azure Blob，并可从该备份恢复。
- SQL 命令式备份：适合更细粒度控制。
- 受监管、TDE 或跨区域场景可能有额外限制和费用。

网络与安全：

- IP allowlist 只解决公网来源控制。
- 私有连接需要云厂商对应能力，例如 AWS PrivateLink、GCP Private Service Connect、Azure Private Link。
- 对象存储摄取要同时考虑 ClickPipes 服务访问路径和 ClickHouse 服务实际读取数据路径。
- S3 bucket policy 常需 `GetObject`、`ListBucket`；unordered 模式还需要 SQS 权限。
- RBAC 使用用户、角色、row policy、settings profile、quota。生产中避免把 default 用户用于应用。

扩缩容：

- Cloud 可纵向或横向扩缩容，SharedMergeTree 减少扩缩容时的元数据复制负担。
- 大批量导入前，先临时放大服务通常比导入后等待更有效。
- 某些托管摄取资源在创建时按当前服务规模确定，创建前放大更可靠。

## ClickPipes 与 Cloud 摄取

Cloud 中优先考虑 ClickPipes 来接入 Kafka、对象存储、Kinesis、Pub/Sub、Postgres、MySQL、MongoDB 等。对用户方案：

- 如果用户想少运维、低代码、持续摄取：ClickPipes。
- 如果需要高度自定义转换、特殊错误处理或自建环境：Kafka engine/外部 ETL/客户端批量写入。
- 对象存储连续摄取要确认文件顺序语义；无法保证字典序时，Amazon S3 可使用 SQS/EventBridge unordered 模式。

ClickPipes 和目标表上的物化视图兼容，但要避免下游物化视图通过普通 view 间接读取目标表，否则 staging 机制可能无法覆盖依赖关系。

## chDB

chDB 是嵌入进程内的 ClickHouse OLAP 引擎，适合在 Python、Go、Rust、Node.js、Bun、C/C++ 等语言中使用 ClickHouse 能力，而不部署 ClickHouse Server。

适合：

- 本地或应用内分析。
- 查询 Parquet、CSV、JSON、Arrow、ORC、S3 文件。
- Python 数据科学工作流。
- Pandas/Arrow 数据快速 SQL 分析。
- 轻量 ETL、测试 SQL、嵌入式报表。

不适合：

- 多用户服务端数据库替代。
- 需要长期集群存储、复制、权限、多租户隔离的生产 OLAP。
- 高并发写入服务。

Python SQL API：

```python
import chdb

result = chdb.query(
    """
    select is_error, count() as event_count
    from file('events.jsonl', 'JSONEachRow')
    group by is_error
    """,
    output_format="DataFrame",
)
```

持久 session：

```python
from chdb import session as chs

sess = chs.Session("analytics.chdb")
sess.query("create database if not exists app")
sess.query("""
    create table app.events
    order by event_time
    as select *
    from file('events.parquet', 'Parquet')
""")
```

DataStore API 提供类 Pandas 体验，惰性执行并下推为 SQL。适合从 Pandas 迁移：常见操作可通过替换 import 开始，但要提醒用户语义差异、懒执行和 SQL 类型规则。

## 典型用例建模

### 时间序列

按时间保留和范围查询设计：

- 分区：按月或日，取决于保留周期和写入量。
- 排序键：租户/指标名/标签低基数字段 + 时间桶 + 实体 ID。
- Rollup：用物化视图把原始秒级/毫秒级数据聚合到分钟、小时、天。
- TTL：原始数据短保留，rollup 长保留。

```sql
create table metrics_raw
(
    ts DateTime64(3),
    tenant_id UInt32,
    metric LowCardinality(String),
    host LowCardinality(String),
    value Float64
)
engine = MergeTree
partition by toYYYYMM(ts)
order by (tenant_id, metric, host, toStartOfHour(ts), ts)
ttl ts + interval 30 day;
```

### 可观测性和日志

日志/trace/event 数据特点是高写入、时间范围查询、标签过滤、JSON payload、保留周期明确。

建议：

- 热字段抽为强类型列：时间、服务、环境、level、trace_id、span_id、tenant、status。
- 原始 message/payload 可保留 String 或 JSON。
- 分区与 TTL 对齐。
- 常用仪表盘用物化视图预聚合。
- 避免按 trace_id 高基数字段分区；可放在排序键后部或使用跳数索引。

### 数据湖分析

ClickHouse 可以直接读 Parquet/ORC/CSV/JSON，也可通过 catalog 访问湖仓。策略：

- 探索阶段直接 `s3`/catalog 查询。
- 高频查询或低延迟需求，把热数据导入 MergeTree。
- 大对象存储导入优先 Parquet，按文件并行。
- Schema inference 只作起点，生产表要显式优化类型、排序键和分区。

### AI/ML 与向量搜索

向量场景通常是结构化过滤 + 向量相似度：

- 把租户、时间、类别、状态等过滤字段放入排序键或可跳过字段。
- 向量列与 embedding 版本、模型名一起管理。
- 对大规模向量检索，先用结构化条件减少候选集，再做相似度。
- 存储文本、metadata 和 embedding 时，动态 metadata 可用 JSON，但热过滤字段要抽列。

### Key-value 和实时查找

ClickHouse 可做高吞吐分析查询中的点查辅助，但不是低延迟 OLTP KV。若用于 KV 样式：

- 尽量批量读写。
- 排序键以 key 开头。
- 需要最新值时考虑 ReplacingMergeTree(version)。
- 对维表点查，Dictionary 更合适。

## 迁移建议

从 OLTP、数据仓库或日志系统迁移到 ClickHouse 时：

1. 不要逐表照搬。先列出最重要查询和仪表盘。
2. 识别主事实表，围绕过滤条件和聚合维度设计排序键。
3. 决定哪些维度反规范化，哪些保留为字典或 Join。
4. 从源格式推断 schema 后，手动优化类型。
5. 设计初始加载、增量同步、回填、校验和回滚。
6. 给生产表加 TTL、备份、监控和权限。
7. 用真实数据基准测试，不用小样本判断排序键和分区。

迁移后的验收指标：

- 关键查询读行数和读字节明显下降。
- 写入批次不会持续堆积 parts。
- 仪表盘查询在目标延迟内，且峰值内存可控。
- 回填和 CDC 重放可以重复执行且不破坏一致性。
- 备份恢复演练成功。
