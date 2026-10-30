# Doris 核心概念

## 产品定位

Apache Doris 是基于 MPP 架构的实时分析型数据库，强调高性能、简单易用和统一分析体验。它通过兼容 MySQL 协议的 SQL 接口，在大规模数据上支持亚秒级分析查询、高并发点查和高吞吐复杂分析。

主要场景：

- 实时分析：看板、用户行为分析、A/B 测试、日志分析、业务监控。
- 湖仓分析：对湖仓数据做联邦查询、外部 Catalog 访问和外部数据加速。
- 混合检索与 AI 分析：结构化过滤、全文检索、向量检索、AI 函数、RAG、语义搜索和 AI 可观测性。

## 数据组织

Doris 使用三层层级结构：

- Catalog：逻辑命名空间。`internal` 保存 Doris 内表。External Catalog 映射 Hive、Iceberg、Hudi、Paimon、JDBC 数据源和 Elasticsearch 类系统等外部数据源。
- Database：Catalog 内的业务或项目级隔离单元。
- Table：关系型对象，包含 Schema、表模型、分区、分布、属性和索引。

Internal Catalog 支持完整的 Doris 表管理。External Catalog 的能力取决于数据源类型：有些只支持查询，Iceberg 和 Paimon 可支持更完整的 DDL/DML 类数据管理能力。

## 系统架构

### 存算一体

经典 FE + BE 架构：

- FE 接收客户端请求、解析 SQL、管理元数据、规划查询、跟踪节点，并协调导入和查询状态。
- BE 存储数据并执行查询。它持有 Tablet、保存副本、执行 Compaction、扫描数据，并运行向量化和 Pipeline 执行。
- FE 高可用使用 Master、Follower、Observer 角色。Follower 参与选举；Observer 扩展读能力。
- Tablet 数据通常有多副本；使用三副本时，生产环境通常至少规划三个 BE 节点。

当环境稳定、没有共享存储、低延迟重要或需要简化运维时，优先选择该模式。

### 存算分离

云原生架构，将元数据、计算和共享存储分离：

- FE 仍负责请求入口、SQL 规划、SQL 层元数据、认证和授权。
- Meta Service 管理数据层元数据、版本、Rowset、Tablet 元数据、事务和计算组资源信息。
- 计算层由计算组组成。BE 是无状态计算节点，并在本地缓存热点数据。
- 共享存储保存持久化 Segment 和索引文件，常见于对象存储、HDFS 或 Ceph 兼容存储。

弹性计算、云部署、Kubernetes、工作负载隔离、多计算组共享数据和降低存储成本场景适合该模式。代价是依赖更多组件，运维复杂度更高。

## 执行与优化

- MPP 将查询拆分为跨节点执行的 Fragment 和 Instance。Exchange 算子负责 Instance 间的数据交换。
- 向量化执行按列批量处理数据，提升 CPU 缓存和 SIMD 利用率。
- Pipeline 执行调度算子流水线，提高 CPU 利用率并减少过量线程。
- RBO 处理确定性的规则改写，如谓词下推和常量折叠。
- CBO 使用统计信息选择 Join 顺序、Join 策略、数据分布和成本更低的执行计划。
- Runtime Filter 在 Join 过程中动态生成过滤条件，并下推到扫描节点以减少 I/O。
- 通过 `ANALYZE` 收集的统计信息对 CBO 质量很重要。

## 表模型

表模型在建表时选择，是存储、更新、聚合和查询行为的核心。

| 模型 | 中文名称 | 核心行为 | 适用场景 | 避免场景 |
| --- | --- | --- | --- | --- |
| `DUPLICATE KEY` | 明细模型 | 保留所有原始行，Key 列主要用于排序 | 日志、事件、事实明细、Ad-hoc 分析、高性能查询 | 需要行级更新或预聚合 |
| `UNIQUE KEY` | 主键模型 | 每个 Key 保留最新一行，支持行级更新、删除和部分列更新 | CDC、订单状态、实时维表、用户画像 | 主要目标是 Rollup 类预聚合 |
| `AGGREGATE KEY` | 聚合模型 | 按 Key 使用预定义聚合函数合并 Value 列 | 固定报表指标和预聚合汇总 | 灵活 Ad-hoc 聚合或大量 `count(*)` 查询 |

重要点：

- Key 列必须位于 Value 列之前。
- Key 列定义存储排序；在 Unique 和 Aggregate 模型中还参与唯一性或合并语义。
- 在支持的 Unique 模型场景下，`ORDER BY` 可以将局部排序列与 Key 列分开定义。
- 表模型创建后不能修改；变更模型通常需要新建表并迁移数据。

## 排序键、分区与分桶

排序键有助于裁剪、压缩和等值/范围查询性能。优先选择高频过滤的等值或范围列。不要设置过多 Key 列，因为会增加写入和排序成本。

分区：

- 按值范围或列表切分数据，常见于时间分区。
- 有助于生命周期管理和分区裁剪。
- 表是否分区在创建时确定。
- 分区列创建后不能直接修改。

分桶：

- Hash 或 Random 分布决定 Tablet 位置和并行度。
- 选择高基数、分布均匀的列。
- 避免使用大量 NULL 或固定值的列。
- 分桶选择不当会导致数据倾斜和并行度不足。
- 分桶/分布列创建后不能直接修改；重新设计通常需要迁移数据。

## 索引

Doris 使用多种裁剪和检索机制：

- Sorted Compound Key/前缀索引：适合前导 Key 列和常见过滤。
- Min/Max：帮助数值或日期类数据做范围裁剪。
- Bloom Filter：适合高基数等值过滤。
- 倒排索引：支持在选定列上做快速检索和全文类过滤。
- NGram BloomFilter：适合子串类场景。
- 向量索引：支持向量工作负载的近似最近邻检索。

推荐索引时，要把索引与查询谓词和写入成本关联起来。索引可以提升查询速度，但可能增加导入、Compaction、存储或重建成本。

## 数据导入选型

根据数据源、频率和延迟选择导入方式：

| 场景 | 优先选择 |
| --- | --- |
| 应用通过 JDBC 写入极少量数据 | `INSERT` |
| 高并发或频繁小批量写入 | Group Commit 配合 `INSERT` 或 Stream Load |
| HTTP 高吞吐写入或本地文件 | Stream Load |
| 对象存储或 HDFS 批量文件 | Broker Load 或 `INSERT INTO SELECT` |
| Doris 主动拉取 Kafka 流 | Routine Load |
| Kafka 推送式集成 | Doris Kafka Connector |
| Flink 流式或 CDC 管道 | Flink Doris Connector / Flink CDC |
| 内置从 MySQL/PostgreSQL/S3 持续导入 | Streaming Job |
| 查询外部湖仓/数据源后写入 | Catalog 加 `INSERT INTO SELECT` |

相关时应说明事务行为和可见性。面向用户的示例应包含表 DDL、导入命令、状态检查和数据验证。

## 查询加速

常见技术：

- Schema 设计：选择合适的表模型、分桶列、Key 列和高效数据类型。
- 物化视图：
  - 同步物化视图与内表维护绑定，适用范围有限。
  - 异步物化视图是物理化的预计算结果，通过任务刷新，并支持对合适查询形态做透明改写。
  - 异步物化视图是最终一致，不是实时一致。
  - SPJG 类查询是透明改写的主要目标。
  - 窗口函数、`UNION ALL`、`LIMIT`、`ORDER BY`、`CROSS JOIN` 等算子可能限制透明改写。
- Runtime Filter、CBO 统计信息、Join 策略、Colocate/Bucket Shuffle、分区裁剪和外部元数据缓存调优都是常见查询调优主题。

## AI 与混合检索

Doris 的 AI 能力围绕统一 SQL 分析展开，组合结构化数据、向量检索、全文检索、AI 函数和可观测性。撰写 AI 功能文档时，要明确数据流：模型或资源配置、SQL 调用、检索/搜索逻辑和运维限制。
