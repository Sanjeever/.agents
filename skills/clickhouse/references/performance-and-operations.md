# 性能与运维参考

## 目录

- 性能分析顺序
- Parts、merges 与 too many parts
- 主键、跳数索引和查询裁剪
- 分区、TTL 与生命周期
- 物化视图和预聚合运维
- 避免 OPTIMIZE FINAL 和昂贵 mutation
- 备份恢复
- Keeper、复制、分片与 Distributed
- 系统表与排障查询

## 性能分析顺序

处理慢查询时按顺序看：

1. 查询是否读取了过多行、列、part 或 granule。
2. `WHERE` 是否命中排序键、分区键或跳数索引。
3. 是否存在高基数 `GROUP BY`、大 Join、无界排序、`FINAL`、复杂 JSON 解析。
4. 表是否有过多 active parts，后台 merge 是否落后。
5. 数据类型是否过宽、Nullable/String 过多、压缩不佳。
6. 是否需要物化视图、投影、字典或重建排序键。

基础诊断：

```sql
explain indexes = 1
select ...
from table_name
where ...;

select
    query_id,
    read_rows,
    read_bytes,
    memory_usage,
    query_duration_ms
from system.query_log
where type = 'QueryFinish'
order by event_time desc
limit 20;
```

## Parts、merges 与 too many parts

每次 insert 生成 part。后台 merge 合并 part，生成更大的 part，并重建压缩列文件、稀疏索引、跳数索引、校验和等。高吞吐插入必须让 part 生成速度低于 merge 消化能力。

检查 active parts：

```sql
select
    database,
    table,
    count() as active_parts,
    sum(rows) as rows,
    formatReadableSize(sum(bytes_on_disk)) as bytes_on_disk
from system.parts
where active
group by database, table
order by active_parts desc
limit 20;
```

检查每个分区：

```sql
select
    partition,
    count() as active_parts,
    sum(rows) as rows
from system.parts
where database = 'db' and table = 'events' and active
group by partition
order by active_parts desc;
```

解决 too many parts：

- 增大插入批次，减少每秒 insert 次数。
- 降低分区基数，避免按用户、请求 ID、秒级时间分区。
- 让客户端或摄取管道合并小文件/小消息。
- 检查后台 merge 线程、磁盘/对象存储吞吐、CPU 和内存。
- 对历史碎片，可在低峰用重建表或谨慎手动 optimize，但不要把 `OPTIMIZE FINAL` 当日常机制。

## 主键、跳数索引和查询裁剪

稀疏主键索引每个 granule 存一条排序键标记，默认 granule 是 8192 行。索引足够小，可常驻内存；查询用它跳过不可能匹配的 granule。

验证索引效果：

```sql
explain indexes = 1
select count()
from events
where tenant_id = 42
  and event_type = 'error'
  and event_time >= today();
```

若输出显示 PrimaryKey 只选中了少量 granules，排序键有效。若全扫，优先考虑：

- 修改查询过滤条件，使其匹配排序键前缀。
- 重建表选择更合适的 `ORDER BY`。
- 加投影提供第二排序布局。
- 对非主键但可跳过的列使用 data skipping index，但只在数据局部性好时有效。

跳数索引适合补充，不应替代排序键。布隆类索引适合数组、token 或稀疏匹配；minmax/set 适合值在 part/granule 内有局部性的数据。创建后要物化旧数据。

## 分区、TTL 与生命周期

分区用于生命周期、冷热分层、整段删除和对象管理。ClickHouse 不跨分区 merge，因此分区键要低基数。

TTL 可删除行/列、移动到卷、重压缩或 rollup。TTL 在 merge 时生效，不是到期即刻删除。默认 TTL merge 触发有间隔，必要时可调整相关 merge timeout。

表级删除 TTL：

```sql
create table logs
(
    event_time DateTime,
    service LowCardinality(String),
    message String
)
engine = MergeTree
partition by toYYYYMM(event_time)
order by (service, toDate(event_time))
ttl event_time + interval 90 day;
```

按条件保留：

```sql
ttl
    event_time + interval 30 day delete where level != 'error',
    event_time + interval 180 day delete where level = 'error'
```

TTL rollup 要保证 `GROUP BY` 是主键前缀，并准备存储聚合结果的列。

## 物化视图和预聚合运维

增量物化视图只处理创建后进入源表的新 block。历史数据需要显式 backfill。常见做法：

1. 创建目标表。
2. 创建物化视图，或先暂停写入后创建。
3. 用 `insert into target select ... from source` 回填历史。
4. 校验目标表与源查询结果一致。

复杂迁移中可使用未来时间过滤避免双写：

```sql
create materialized view mv to target as
select ...
from source
where event_time >= '2026-01-01';

insert into target
select ...
from source
where event_time < '2026-01-01';
```

运维注意：

- 物化视图失败会影响插入链路；目标表 schema 与视图输出要稳定。
- 增量视图中的 Join 只由最左侧源表插入触发，右表变化不会自动重算。
- `parallel_view_processing` 可并行处理多个视图，但要评估资源。
- 对 `AggregatingMergeTree` 目标表，查询必须使用 `Merge` 后缀合并状态。

## 避免 OPTIMIZE FINAL 和昂贵 mutation

`OPTIMIZE TABLE ... FINAL` 会强制把 part 合并成更大 part，绕开正常 merge 选择逻辑，容易造成高 CPU、内存、磁盘或对象存储压力。只在明确维护窗口、必要场景和受控表上使用。

避免频繁：

```sql
alter table events update ...
alter table events delete ...
optimize table events final;
select ... from events final;
```

替代：

- 插入修正行和版本，用 `ReplacingMergeTree`。
- 用 TTL 或 `drop partition` 管理历史数据。
- 用新表重算后交换。
- 查询端用聚合或窗口取最新，而不是全表 `FINAL`。

## 备份恢复

复制不等于备份。复制能抗硬件故障，但不能防误删、错误 mutation、错误脚本或软件 bug，因为错误会复制到所有副本。

备份策略要提前演练恢复。没恢复演练过的备份不能视为可靠。

ClickHouse 支持：

- 全量和增量备份。
- 同步和 `ASYNC` 异步备份。
- 并发或禁止并发备份/恢复。
- 压缩与密码。
- 本地 disk、配置的 S3 disk、S3 endpoint、Azure Blob。
- database、table、partition、dictionary、view、access entities。

示例：

```sql
backup table db.events
to Disk('backups', 'events.zip')
async;

select *
from system.backups
where id = '<backup_id>'
format Vertical;
```

S3 示例：

```sql
backup database analytics
to S3('https://bucket/backups/analytics.zip', '<access_key>', '<secret_key>')
settings compression_method = 'zstd';
```

恢复要考虑：

- 增量备份恢复依赖完整备份链。
- access management 只有 SQL 管理的 users/roles/policies/settings profiles/quotas 可通过备份恢复；配置文件里的用户不在其中。
- 大集群恢复前验证目标集群版本、存储策略、磁盘配置、权限和容量。
- 生产策略应包含 RPO、RTO、加密、跨区域、保留期和定期恢复演练。

## Keeper、复制、分片与 Distributed

自建高可用使用 ClickHouse Keeper 协调 ReplicatedMergeTree。Keeper 管理复制元数据、leader 选举和分布式 DDL 协调，不存实际表数据。

概念：

- shard：数据水平分片，不同 shard 保存不同数据子集。
- replica：同一 shard 的副本，保存相同数据。
- Distributed 表：跨 shard 查询/写入入口，本身不存数据。
- ReplicatedMergeTree：每个 shard 内副本复制。
- Keeper quorum：通常需要奇数节点，避免单点。

自建 DDL 模式：

```sql
create database db on cluster 'cluster_1s_2r';

create table db.events on cluster 'cluster_1s_2r'
(
    event_time DateTime,
    id UInt64,
    message String
)
engine = ReplicatedMergeTree('/clickhouse/tables/{shard}/db/{uuid}', '{replica}')
order by (toDate(event_time), id);

create table db.events_all on cluster 'cluster_1s_2r'
as db.events
engine = Distributed('cluster_1s_2r', 'db', 'events', rand());
```

建议：

- 使用 Atomic database 和 `{uuid}` 避免频繁建删表时 Keeper 路径复用问题。
- 每个节点配置唯一 `shard`、`replica` 宏。
- 增删 Keeper 节点要按 quorum 规则逐步操作。
- 监控复制队列、延迟、broken parts、Keeper 会话和磁盘空间。

## 系统表与排障查询

常用系统表：

- `system.query_log`：慢查询、读行数、内存、异常。
- `system.parts`：part 数量、分区、大小、磁盘。
- `system.merges`：正在 merge 的任务。
- `system.mutations`：mutation 状态和失败。
- `system.disks`、`system.storage_policies`：磁盘和存储策略。
- `system.backups`、`system.backup_log`：备份恢复进度和错误。
- `system.tables`、`system.columns`：表和列元数据。
- 自建复制：`system.replication_queue`、`system.replicas`、`system.replicated_fetches`。
- Cloud SharedMergeTree：用 `system.virtual_parts`、`system.shared_merge_tree_fetches` 替代部分 ReplicatedMergeTree 复制队列表。

慢查询：

```sql
select
    query_id,
    user,
    query_duration_ms,
    read_rows,
    formatReadableSize(read_bytes) as read_bytes,
    memory_usage,
    query
from system.query_log
where type = 'QueryFinish'
order by query_duration_ms desc
limit 10;
```

mutation：

```sql
select
    database,
    table,
    mutation_id,
    command,
    is_done,
    latest_fail_reason
from system.mutations
where not is_done;
```

merge：

```sql
select
    database,
    table,
    elapsed,
    progress,
    num_parts,
    formatReadableSize(total_size_bytes_compressed) as compressed_size
from system.merges
order by elapsed desc;
```
