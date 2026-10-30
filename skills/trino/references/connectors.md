# Trino 481 连接器与对象存储提炼

## Connector 工作模型

Connector 把 Trino 的标准 SPI 映射到底层数据源。每个 catalog 通过属性文件选择一个 connector，并设置连接、认证、元数据、类型映射、pushdown、写入、统计信息等行为。

回答连接器问题时先问清：

- 目标 connector 和 catalog 名。
- 数据源版本、认证方式、网络路径和 TLS 要求。
- 操作类型：只读、写入、DDL、权限、事务、表过程、表函数、统计信息、pushdown。
- 是否涉及对象存储、metastore、file format、schema evolution 或 time travel。
- Trino 版本是否是 481；如果不是，先核验差异。

## Connector 列表

Trino 481 文档覆盖的 connector 包括：

BigQuery、Black Hole、Cassandra、ClickHouse、Delta Lake、Druid、DuckDB、Elasticsearch、Exasol、Faker、Google Sheets、Hive、Hudi、Iceberg、Ignite、JMX、Kafka、Lakehouse、Loki、MariaDB、Memory、MongoDB、MySQL、OpenSearch、Oracle、Pinot、PostgreSQL、Prometheus、Redis、Redshift、SingleStore、Snowflake、SQL Server、System、Thrift、TPC-DS、TPC-H。

常见分组：

- 湖仓/文件：Hive、Iceberg、Delta Lake、Hudi、Lakehouse。
- JDBC 系关系库：MySQL、PostgreSQL、Oracle、SQL Server、MariaDB、ClickHouse、Snowflake、Redshift、SingleStore、Exasol、DuckDB。
- 搜索/日志/流：Elasticsearch、OpenSearch、Kafka、Loki、Prometheus、Pinot。
- NoSQL/其他：Cassandra、MongoDB、Redis、BigQuery、Google Sheets、Ignite、Druid。
- 内置/工具：System、JMX、Memory、Black Hole、Faker、TPC-H、TPC-DS、Thrift。

## 对象存储

Hive、Iceberg、Delta Lake、Hudi 等 connector 直接读写对象存储或远程文件系统中的数据文件。Trino 481 的关键变化：

- 每个 catalog 默认不启用文件系统支持，必须显式选择一个。
- Native 支持包括 `fs.azure.enabled`、`fs.gcs.enabled`、`fs.s3.enabled`、`fs.hadoop.enabled`。
- 每个 catalog 只能选择一种文件系统支持。
- `fs.hadoop.enabled` 用于 HDFS。通过 `fs.hadoop.enabled` 和旧 `hive.*` 属性访问 Azure/GCS/S3 的 legacy object storage 支持在 Trino 481 不可用。
- Azure、GCS、S3 应使用 native file system 实现。
- 额外能力包括 file system cache、metastore 配置、file format 配置。

## Lakehouse connector 共同点

Hive、Iceberg、Delta Lake 常见能力维度：

- General configuration 与 file system access configuration 分离。
- 支持表属性、列属性、metadata tables/columns、统计信息、动态过滤或 projection pushdown。
- 支持 `ALTER TABLE EXECUTE` 调用 connector 表过程，例如 `optimize`。
- 常有 `system.flush_metadata_cache()`。
- 对 DML/DDL/schema evolution/materialized view/time travel 支持差异很大，不能泛化。
- 对象存储写入需关注 file size、writer partition 数、compression、cache、metadata 并发和 metastore 压力。

## Hive

Hive connector 访问 Hive metastore 管理的表和文件：

- 支持多 Hive 集群，通过多个 catalog 配置。
- 文件格式、ORC/Parquet 列名匹配、压缩、分区、bucket、view、materialized view、partition projection 等都有独立属性。
- 481 使用 native object storage；HDFS 仍通过 Hadoop 文件系统。
- 支持 Hive 到 Trino、Trino 到 Hive 类型映射；`UNIONTYPE` 等复杂类型存在限制。
- 安全模式包括 `allow-all`、`read-only`、`file`、`sql-standard`。
- 支持 `system.flush_metadata_cache()`，可按 schema/table/partition 粒度刷新。
- `optimize` 默认禁用，可通过 catalog session property 启用；非事务表优化有并发风险。
- Metadata tables 包括 `$properties`、`$partitions`，metadata columns 包括文件路径、bucket 等隐藏信息。
- 支持 table statistics，CBO 可利用统计信息；支持 dynamic filtering 和延迟执行以减少扫描。
- 常见性能属性关注 split 数量、split 大小、每秒 split、file status cache、query partition filter required。

## Iceberg

Iceberg connector 适合 Apache Iceberg 表：

- `iceberg.catalog.type` 选择 catalog 实现，例如 Hive、Glue、REST、JDBC、Nessie、Snowflake 等。
- 支持 v1/v2/v3 表能力，Trino 481 对 Iceberg v3 `VARIANT` 和 `timestamp(9)` 读写有新增支持。
- 支持 register/unregister table、migrate table、add files、`optimize`、`optimize_manifests`、`expire_snapshots`、`remove_orphan_files`、`drop_extended_stats` 等过程。
- 支持 row-level delete、schema evolution、partition transforms、sorted tables、branches/tags、snapshots、rollback、time travel。
- Metadata tables 包括 `$properties`、`$history`、`$metadata_log_entries`、`$snapshots`、`$manifests`、`$all_manifests`、`$partitions`、`$files`、`$entries`、`$all_entries`、`$refs`。
- 481 的 `$files` 表包含更多列，包括 `added_snapshot_id`。
- 支持 materialized views，刷新时可能涉及快照过期。
- 性能关注 table statistics、metadata caching、file system cache、projection pushdown、REST catalog vended credentials。

## Delta Lake

Delta Lake connector 访问 Delta transaction log：

- 支持 Hive metastore 相关配置和 native object storage。
- 481 移除 legacy object storage；并增加对 Delta checksum metadata/protocol 信息读取来降低 planning time，可用 `delta.load-metadata-from-checksum-file` 或 session property 禁用。
- 支持 Delta 到 Trino 与 Trino 到 Delta 类型映射，481 包含 `VARIANT` 读取映射。
- 支持 time travel by version/timestamp。
- 常见过程：`system.register_table`、`system.unregister_table`、`system.flush_metadata_cache`、`system.vacuum`。
- `delta.vacuum.min-retention` 是 vacuum 安全阈值。
- 支持 schema evolution、replace tables、shallow clone、metadata tables `$history`、`$partitions`、`$properties`、metadata columns。
- `system.table_changes` table function 用于 change data feed。
- 481 修复了 Delta + Parquet column indexes 下 `DELETE` 可能删错行的问题。
- 性能关注 extended statistics、active data files 数量、split 控制、file system cache。

## Hudi 与 Lakehouse

Hudi connector 提供 Hudi 表读写和元数据访问，覆盖 schema/table management、table properties、`$timeline` metadata table。Lakehouse connector 是聚合式湖仓入口，依赖 file system access 配置，481 同样使用 native object storage。

## Kafka

Kafka connector 用 catalog 把 topic 映射为表：

- 核心属性包括 `kafka.nodes`、`kafka.default-schema`、`kafka.table-names`、`kafka.table-description-dir`、安全协议和 SSL keystore/truststore。
- 表定义可来自本地 JSON 文件或 Confluent Schema Registry。
- Internal columns 可暴露 topic、partition、offset、key、message 等 Kafka 元数据。
- 支持 key/message 的 raw、CSV、JSON、Avro、Protobuf 编码/解码。
- Confluent schema registry 支持 subject 到 table 映射、Protobuf `oneof`、schema evolution 等。
- Kafka inserts 需要定义 encoder，且写入语义与传统数据库不同。
- SQL 支持主要围绕流式 topic 表查询/插入；不要假设支持事务、复杂 DDL 或删除。

## JDBC 系连接器

JDBC 系 connector 通常具备共同形态：

- `connection-url`、`connection-user`、`connection-password` 和 `connector.name` 是基础。
- 多个数据库/实例通常通过多个 catalog 配置。
- 连接安全常由 JDBC URL 参数、证书和连接属性控制。
- SQL support、type mapping、table functions、procedures、pushdown、table statistics 因数据库而异。
- 多数支持 `query(varchar) -> table` table function，用底层 SQL 直接查询远端系统；这是绕过 Trino SQL 解析和部分安全/优化语义的高级能力，回答时要提示风险。
- 一些 connector 支持 `system.execute('query')` 执行远端语句；需特别关注权限和副作用。

典型差异：

- MySQL：支持 JSON、primary key table property、timestamp 类型处理、统计信息、pushdown。481 修复 JSON 小数精度读取问题。
- PostgreSQL：支持 JSON/JSONB、ARRAY、UUID、GEOMETRY、VECTOR、HSTORE 等映射；481 增强 `COALESCE` 下推并修复 JSON 小数精度读取问题。
- Oracle：`NUMBER` 映射需关注默认 scale 与 rounding mode；datetime/character 映射复杂；predicate pushdown 对部分类型有限制。
- SQL Server：481 增加 JSON 类型支持；有 bulk copy 写入属性；支持 `procedure(varchar) -> table` table function。
- ClickHouse：481 增加读取所有 ClickHouse `DECIMAL` 列支持；支持多服务器 catalog、table properties 和 pushdown。
- Snowflake/Redshift/SingleStore/Exasol/MariaDB：关注远端类型、pushdown、表函数和连接安全差异。

## Pushdown

Pushdown 把部分处理交给数据源，可减少 Trino 扫描、网络传输和远端/本地成本。支持取决于 connector 和底层系统。

类型：

- Predicate pushdown：`WHERE` 行过滤。成功时对应 filter 不再表现为 Trino 侧 `ScanFilterProject`。
- Projection pushdown：只读取所需列。`EXPLAIN` 的 `TableScan` layout 只包含相关列。
- Dereference pushdown：只读取 `ROW` 顶层或嵌套字段。
- Aggregation pushdown：成功时 `EXPLAIN` 中没有 Trino 侧 `Aggregate` operator，聚合表现为 connector `TableScan` 的一部分。
- Join pushdown：通常要求 join predicates 可下推且表来自同一 catalog；成功时 `EXPLAIN` 中不出现 Trino 侧 `Join`。
- Limit pushdown：下推未排序 `LIMIT`/`FETCH FIRST`。
- Top-N pushdown：下推 `ORDER BY ... LIMIT`。

Aggregation pushdown 常见限制：复杂 grouping、聚合参数表达式、coercion、带 ordering 的聚合、带 filter 的聚合。

Pushdown 不总是有益。Join pushdown 可能放大行数或给远端系统造成压力；需要结合数据量、选择性、远端索引/统计和资源隔离判断。

## 表函数与过程

常见 connector table functions：

- JDBC 系：`query(varchar) -> table`。
- Elasticsearch/OpenSearch：`raw_query(varchar) -> table`。
- BigQuery：`query(varchar) -> table`。
- Delta/Iceberg：`table_changes`。
- Google Sheets：`sheet(id, range) -> table`。
- Loki：`query_range(varchar, timestamp, timestamp) -> table`。
- MongoDB：`query(database, collection, filter) -> table`。

常见 procedures：

- Metadata cache：`system.flush_metadata_cache()`。
- Register/unregister：Hive/Iceberg/Delta 相关表注册/注销。
- Iceberg maintenance：`add_files`、`add_files_from_table`、`expire_snapshots`、`remove_orphan_files`、`optimize`、`optimize_manifests`。
- Delta maintenance：`vacuum`、`optimize`。

回答 procedure 问题时必须提示：权限、connector 支持、表格式、并发写入、保留期和对象存储可见性都会影响正确性。
