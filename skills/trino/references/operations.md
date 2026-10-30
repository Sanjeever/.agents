# Trino 481 运维、部署与优化提炼

## 部署基本要求

- 服务端运行要求 64-bit Java 25，最低 25.0.1，推荐最新 patch。Java 8/11/17/21/24 不适用于 Trino 481 服务端。
- 推荐 Eclipse Temurin OpenJDK。官方 Docker image 使用同一发行版。
- Linux 需要 64-bit，生产节点要配置足够的 `nofile` 和 `nproc`。
- 推荐一个节点只运行一个 Trino 进程。
- 数据目录应独立于安装目录，便于升级保留日志和运行数据。

## 配置文件

核心配置：

- `node.properties`：每节点环境配置。关键项包括 `node.environment`、`node.id`、`node.data-dir`。
- `jvm.config`：JVM 参数。`-Xmx` 通常设为节点内存 70%-85%，不能耗尽 OS/off-heap 所需内存；不支持 swap。
- `config.properties`：coordinator/worker、HTTP、discovery、查询管理、资源、安全等服务端配置。
- catalog properties：每个 catalog 的 connector 配置。

Coordinator 最小思路：

- `coordinator=true`
- 大集群通常设置 `node-scheduler.include-coordinator=false`
- `http-server.http.port=8080`
- `discovery.uri=http://coordinator:8080`

Worker 最小思路：

- `coordinator=false`
- 同一 `discovery.uri`

单机测试可让 coordinator 参与 worker 调度。生产大集群应避免 coordinator 承担数据处理，以保护调度和监控资源。

JVM 临时目录不能挂载 `noexec`，否则 Trino 加载 native code 时可能启动失败。必要时用 `-Djava.io.tmpdir=...` 指向可执行临时目录。

## 运行与日志

使用 launcher：

- `run` 前台启动。
- `start` 后台启动。
- `stop` 正常终止。
- `restart` 重启。
- `kill` 强制终止。
- `status` 查看状态。

常见日志：

- `launcher.log`：launcher 和早期 JVM 输出。
- `server.log`：主服务日志，排查启动失败首看。
- `http-request.log`：HTTP 请求日志。

日志可配置输出格式、路径、滚动、压缩、JSON/TCP channel。排障时先确认配置加载路径、node id、discovery、HTTP/HTTPS、catalog 初始化错误。

## 查询优化

Trino 优化依赖统计信息、connector 能力和 cost-based optimizer：

- Table statistics 包括行数、数据大小、null 数、NDV、min/max 等；connector 支持差异大。
- `EXPLAIN` 可查看逻辑/分布式/IO/验证计划；`EXPLAIN ANALYZE` 会执行查询并给运行时信息。
- Cost in `EXPLAIN` 可能显示未知，尤其缺少统计信息时。
- Join enumeration 和 join distribution selection 依赖统计信息。
- Join distribution 主要在 partitioned 与 replicated/broadcast 间选择；大表 broadcast 会导致内存风险。
- 缺少统计信息或列统计未知时，join ordering 可能较差。Trino 481 改善了部分 unknown statistics 场景的内存过量问题。
- Adaptive plan optimizations 在容错执行模式下可基于运行时统计调整计划，例如 partitioned join 重新排序。

## Dynamic filtering

Dynamic filtering 利用 join build side 值减少 probe side 扫描，常用于 star schema 或维表过滤事实表：

- 可通过 `EXPLAIN` 和查询信息确认 dynamic filters 是否收集/应用。
- 有 distinct values、size、range rows 等阈值。
- Hive connector 明确支持 dynamic filtering，并可延迟 split 调度等待动态过滤器。
- 如果维表过大、过滤选择性低或 connector 不支持，收益会降低。

## Resource groups

Resource groups 控制查询并发、排队、内存和 CPU 配额。一个 query 只属于一个 resource group。资源耗尽通常让新查询排队，而不是杀死已运行查询；队列满才拒绝新查询。

关键规则：

- 一个 resource group 可以有 sub-groups 或直接接收 queries，不能两者同时做。
- 配置管理器可为 file 或 db。
- File manager 读取 JSON；只需 coordinator 可访问。
- DB manager 支持 MySQL、PostgreSQL、Oracle，使用 `resource_groups_global_properties`、`resource_groups`、`selectors` 表。
- Selector 按顺序处理，第一条匹配生效；单个 selector 中条件是 AND。

常见 group 属性：

- `maxQueued`、`hardConcurrencyLimit` 必需。
- `softConcurrencyLimit` 控制软并发。
- `softMemoryLimit` 可用绝对值或集群内存百分比。
- `softCpuLimit`/`hardCpuLimit` 需与 `cpuQuotaPeriod` 配合。
- `hardPhysicalDataScanLimit` 控制物理扫描量。
- `schedulingPolicy` 可为 `fair`、`weighted_fair`、`weighted`、`query_priority`。
- `schedulingWeight` 用于 weighted 策略。
- `jmxExport` 可暴露 JMX 指标。

Selector 可按 `user`、`originalUser`、`authenticatedUser`、`userGroup`、`source`、`queryText`、`queryType`、`clientTags` 路由。`queryType` 包括 `SELECT`、`EXPLAIN`、`DESCRIBE`、`INSERT`、`UPDATE`、`MERGE`、`DELETE`、`ANALYZE`、`DATA_DEFINITION`、`ALTER_TABLE_EXECUTE`。

## Fault-tolerant execution

默认情况下 worker 资源不足或故障会导致 query 失败。Fault-tolerant execution 通过重试 query 或 task 缓解失败，并将中间 exchange 数据 spool 到可复用存储。

启用：

- `retry-policy=QUERY`：失败时重试整个 query，适合大量小查询。
- `retry-policy=TASK`：失败时重试 task，适合大批量查询；必须配置 exchange manager。
- `retry-policy` 默认 `NONE`。

注意：

- 不会重试语法错误、用户错误或本身错误的查询。
- 不支持 fault-tolerant execution 的 connector 会报类似“不支持 query retries”的错误。
- Trino 481 文档列出的 FTE connector 包括 BigQuery、Delta Lake、Hive、Iceberg、MariaDB、MongoDB、MySQL、Oracle、PostgreSQL、Redshift、SQL Server。
- `QUERY` 默认只对结果集不超过 `exchange.deduplication-buffer-size` 的查询提供内存级容错；大结果建议配置 exchange manager。
- `TASK` retry 更适合独立批处理集群，短查询高并发可能增加延迟。
- Spooling 数据默认加密，关闭不推荐。

Exchange manager：

- 配置文件通常为 `exchange-manager.properties`。
- `exchange-manager.name` 可为 `filesystem` 或 `hdfs`。
- `exchange.base-directories` 指定 spool 目录，可是 S3/S3-compatible、Azure Blob、GCS、HDFS、本地文件系统等。
- 本地文件系统只适合单机非生产，分布式必须所有节点共享可访问。
- S3 I/O 压力大时可配置多个 bucket/目录分散负载，并设置生命周期规则清理遗留对象。

## Client protocol 与 spooling

Trino client protocol 支持 direct protocol 和 spooling protocol。Spooling protocol 将大结果分段写入共享文件系统/对象存储，客户端再取回，可提升吞吐并降低 coordinator 压力。

服务端相关属性以 `protocol.spooling.*`、`fs.segment.*`、`fs.azure.enabled`、`fs.s3.enabled`、`fs.gcs.enabled`、`fs.location` 等为主。客户端必须能访问 spooling 存储。

## 监控与可观测性

- JMX：用于 JVM、cluster、query、task、connector 等指标。
- OpenMetrics：可供 Prometheus/Grafana 等采集。
- OpenTelemetry：配置 tracing 和 exporter endpoint，可用于分布式追踪。
- Event listeners：HTTP、Kafka、MySQL、OpenLineage 等，用于 query created/completed、审计、血缘和指标外发。
- Web UI：用于查看 query、stage、task、worker 和资源状态；Preview Web UI 需显式启用。

## 常见排障入口

- 启动失败：检查 Java 25、JVM 参数、`node.properties`、`config.properties`、data dir 权限、临时目录 `noexec`、端口冲突。
- Worker 不加入：检查 `node.environment` 一致、`discovery.uri`、coordinator 网络、HTTP/HTTPS scheme。
- Catalog 不加载：检查 `connector.name`、插件是否安装、连接 URL、凭证、TLS、对象存储文件系统配置。
- 查询慢：看 `EXPLAIN`、统计信息、join distribution/order、pushdown 是否发生、动态过滤、split 数、扫描文件数、远端系统负载。
- 内存问题：看 query memory、task memory、spill/FTE、join broadcast、大 aggregation、resource groups。
- 大查询容错：确认 retry policy、connector FTE 支持、exchange manager、spooling 存储吞吐和生命周期。
