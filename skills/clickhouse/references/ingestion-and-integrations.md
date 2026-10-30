# 数据摄取与集成参考

## 目录

- 摄取原则
- 文件与对象存储
- JSON、CSV、Parquet 等格式
- Kafka 与流式摄取
- ClickPipes
- CDC：Postgres、MySQL、MongoDB
- 客户端、BI 与开发集成
- 摄取排障

## 摄取原则

ClickHouse 写入路径适合大批量 block。每次写入会生成 part，后台再 merge；小批量高频写入会让 part 数过多并拖慢查询。

默认建议：

- 客户端批量写入，避免单行插入。
- 写入前按目标表排序键预排序，可减少服务端排序成本。
- 格式优先 Parquet、Native、Arrow 等列式/二进制格式；JSONEachRow 适合易用性和中低量写入，但解析成本更高。
- 服务端拉取对象存储数据时，优先 `insert into ... select from s3(...)`、ClickPipes 或云端集成。
- 需要持续摄取与运维简化时，Cloud 优先 ClickPipes；自建可用 Kafka engine + 物化视图。

客户端直写时：

- Native 协议性能和类型保真更好。
- HTTP 接口兼容性强，支持多种格式，适合工具链和简单脚本。
- 同步插入对 MergeTree 默认具备 block 去重能力，但只在相同 block 重试时有效。

## 文件与对象存储

对象存储是 ClickHouse 常见入口。可直接查询或插入：

```sql
create table events
engine = MergeTree
order by (tenant_id, toDate(event_time), event_type)
as
select *
from s3('https://bucket/path/*.parquet');
```

或者先建空表：

```sql
create table events
engine = MergeTree
order by (tenant_id, toDate(event_time), event_type)
empty as
select *
from s3('https://bucket/path/*.parquet');
```

对象存储建议：

- 使用具体 glob，减少无关文件枚举。
- 大批量导入时把文件大小控制在利于并行的范围，避免大量极小文件。
- 对持续落盘文件，命名要可排序且单调递增；若顺序不可控，使用事件队列驱动的托管摄取。
- 可用虚拟列记录文件名、路径或对象元数据，便于审计和补数。

## JSON、CSV、Parquet 等格式

常见格式选择：

- Parquet：批量导入、数据湖、S3/GCS/Azure 对象存储首选。列式压缩好，可推断 schema。
- JSONEachRow/JSONLines/NDJSON：最通用的事件格式，适合日志和 API 输出。
- CSV/TSV：简单但类型和转义容易出错；明确 header、分隔符和空值策略。
- Native/RowBinary/Arrow/Avro/ORC：用于高性能或生态互通场景。

JSON 导入时的核心判断：

- 输入格式是 JSON 不等于表中要用 JSON 类型。
- 静态字段应建强类型列。
- 动态字段才用 JSON 类型，并给热字段类型提示。
- 多余字段默认可能被忽略或报错，按需求设置未知字段处理策略。

常用探索：

```sql
describe table s3('https://bucket/path/*.parquet')
settings describe_compact_output = 1;

select *
from s3('https://bucket/path/*.json', 'JSONEachRow')
limit 10;
```

## Kafka 与流式摄取

自建或需要细控时，可用 Kafka table engine 作为流式入口，通常搭配目标 MergeTree 表和物化视图：

```sql
create table events_queue
(
    event_time DateTime,
    tenant_id UInt32,
    event_type LowCardinality(String),
    payload String
)
engine = Kafka('broker:9092', 'events_topic', 'clickhouse_events', 'JSONEachRow')
settings
    kafka_num_consumers = 4,
    kafka_thread_per_consumer = 1;

create table events
(
    event_time DateTime,
    tenant_id UInt32,
    event_type LowCardinality(String),
    payload String
)
engine = MergeTree
partition by toYYYYMM(event_time)
order by (tenant_id, event_type, toDate(event_time));

create materialized view events_mv to events as
select *
from events_queue;
```

语义与限制：

- Kafka engine 通常是 at-least-once，重复可能发生。
- offset 提交和 ClickHouse 写入之间失败会造成重放；目标表可用 ReplicatedMergeTree block 去重或业务去重键缓解。
- 直接 select Kafka 表会推进 offset，排查时要谨慎。
- 可持久化 `_topic`、`_partition`、`_offset` 等虚拟列用于审计。
- 停止消费可 `detach table events_queue`，恢复用 `attach table`。

调优方向：

- 增大 `kafka_max_block_size`，让每次插入 block 更大。
- `kafka_num_consumers` 不应超过 topic 分区数；集群内优先横向分摊消费者。
- `kafka_thread_per_consumer = 1` 可让 flush 并行。
- 延迟可接受时增大 flush interval 以换吞吐。
- 监控 consumer lag、目标表 part 数、后台 merge、错误日志。

## ClickPipes

ClickPipes 是 ClickHouse Cloud 的托管摄取平台。适合希望减少自建 pipeline、连接外部数据源或持续摄取的场景。

支持范围包括：

- 流式：Kafka、Confluent Cloud、Redpanda、AWS MSK、Azure Event Hubs、WarpStream、Kinesis、GCP Pub/Sub。
- 对象存储：Amazon S3、GCS、Azure Blob、S3 兼容对象存储。
- DBMS/CDC：Postgres、MySQL、MongoDB 等。

对象存储 ClickPipes：

- 支持一次性和持续摄取。
- S3 持续摄取默认依赖文件名字典序；新文件名必须大于已处理文件。
- 需要任意顺序摄取时，Amazon S3 可接 SQS/EventBridge；S3 兼容服务通常不支持 unordered 模式。
- 支持 JSON、CSV、TSV、Parquet、Avro。
- 通过 staging table 实现对象级 exactly-once 语义。
- 可用 `_file` 虚拟列记录来源文件。
- 单文件大小存在上限，超大文件需要切分。

ClickPipes 调优项包括最大插入字节、最大文件数、处理线程、插入线程、下载线程、轮询间隔、是否使用 cluster function、是否并行处理物化视图。大导入任务应先放大 ClickHouse Cloud 服务再创建 pipe，因为 pipe 规模通常在创建时确定。

## CDC：Postgres、MySQL、MongoDB

CDC 迁移或复制时：

- 先确认源库版本、逻辑复制/binlog 配置、网络连通、用户权限和初始快照方式。
- 初始加载与增量复制要分开考虑吞吐、锁、延迟和一致性。
- ClickHouse 端通常需要重新建模，不应机械复制 OLTP schema。
- 对更新频繁的表，考虑 ReplacingMergeTree、版本列、删除标记和最终态查询策略。
- 对宽 JSON 或动态列，按热字段抽列，其余保留 JSON。
- 表级 resync、pause/resume、schema change 要纳入运维流程。

Postgres 常见前置条件是逻辑复制、publication、replication slot 和有权限的复制用户。MySQL 常见前置条件是 binlog、server_id、row 格式、复制用户和网络访问。

## 客户端、BI 与开发集成

语言客户端：

- Python 常用 `clickhouse-connect`，支持查询、插入、参数、DataFrame 等。
- Java、JavaScript、Rust、Go 等均有客户端，注意协议、TLS、压缩、批量插入接口。
- SQLAlchemy 可用于 Python 生态，但复杂分析语句和性能关键路径应直接写 SQL。

BI/SQL 工具：

- Grafana、Tableau、Power BI、Superset、Metabase、DBeaver、DataGrip 等都可接入。
- 仪表盘查询要限制时间范围、避免高基数无界 group by，必要时用物化视图或投影。
- 用户权限建议最小化：只读角色、限定数据库/表、必要的 settings profile。

开发工具：

- `clickhouse-local` 适合本地查询文件、转换格式、验证 SQL。
- chDB 适合嵌入 Python/Go/Rust/Node 等进程内分析。

## 摄取排障

常见问题与检查：

- `too many parts`：批次太小、分区基数太高、merge 跟不上。增大批量，降低分区粒度，检查后台 merge。
- JSON 解析失败：字段类型不一致、未知字段、null 与非 Nullable、转义问题。先抽样 `select * from file/s3 limit`，再固定 schema。
- Kafka lag 增长：消费者不足、block 太小、目标表写入慢、物化视图复杂、后台线程不足。
- 重复数据：at-least-once 重试、CDC 重放、对象存储重复事件。设计幂等键、版本列或去重表。
- Cloud ClickPipes 无法连接源：检查 IP allowlist、PrivateLink/VPC endpoint、IAM/角色策略、bucket/list/get 权限、区域匹配。
- 导入慢：文件太小、格式解析重、目标表排序键复杂、物化视图过多、服务规格过小。
