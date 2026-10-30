# Doris 运维、集成与湖仓参考

## 部署选型

按架构和运行环境选择部署方式：

| 场景 | 推荐方向 |
| --- | --- |
| 首次试用或小规模固定环境 | 存算一体 + 手动部署 |
| 本地机房且没有共享存储 | 存算一体 |
| 云上或弹性工作负载 | 存算分离 |
| 需要自动化的大规模生产环境 | Kubernetes + Doris Operator |
| 多团队共享数据和工作负载隔离 | 存算分离 + 计算组 |

撰写部署文档时，应包含前置条件、节点规划、端口、Java 版本、网络连通性、磁盘布局、启动/注册顺序和验证命令。

## 集群规划要点

重要组件：

- FE：请求入口、SQL 规划、元数据、节点管理。生产高可用通常使用奇数个可参与选举的 Follower FE；Observer 可扩展读能力。
- BE：在存算一体模式下负责数据存储和查询执行；在存算分离模式下作为无状态计算和缓存节点。
- Meta Service：存算分离架构中的元数据服务，管理数据版本、Rowset、Tablet、事务和计算组资源信息。

常见端口：

| 组件 | 端口名 | 默认值 | 主要通信方向 |
| --- | --- | --- | --- |
| FE | `http_port` | 8030 | FE/客户端 HTTP |
| FE | `rpc_port` | 9020 | BE 到 FE、FE 到 FE |
| FE | `query_port` | 9030 | MySQL 协议客户端 |
| FE | `edit_log_port` | 9010 | FE 元数据复制 |
| BE | `be_port` | 9060 | FE 到 BE Thrift |
| BE | `webserver_port` | 8040 | BE HTTP |
| BE | `heartbeat_service_port` | 9050 | FE 心跳 |
| BE | `brpc_port` | 8060 | FE/BE 和 BE/BE BRPC |

运维注意：

- `lower_case_table_names` 必须在集群初始化时确定，之后不应修改。
- 多网卡或复杂网络环境下，`priority_networks` 必须匹配节点网络 CIDR。
- FE 元数据应使用可靠存储，生产环境通常使用独立 SSD。
- BE 存储路径可以跨磁盘配置，并可包含介质提示。

## 手动部署存算一体流程

高层流程：

1. 准备操作系统、Java 17+、网络、磁盘和端口。
2. 部署第一个 FE 作为 Master。
3. 可选：通过 SQL 注册 FE Follower 或 Observer，再在首次启动时使用 helper 信息启动。
4. 启动 BE 前，先在 FE 中注册 BE 节点。
5. 启动 BE 进程。
6. 使用 `SHOW FRONTENDS`、`SHOW BACKENDS`、测试库表和示例插入/查询验证。
7. 初始化后设置非空 root 密码。

故障排查重点：

- FE 启动失败：Java、端口、元数据权限、日志。
- BE 注册失败：FE 可用性、网络、心跳端口、`priority_networks`、BE 日志。
- Alive 但没有 Tablet：数据均衡可能仍在进行。

## 计算组与存算分离运维

计算组在共享数据的同时隔离计算资源。文档应覆盖：

- 查看计算组。
- 授予和撤销计算组访问权限。
- 选择和设置默认计算组。
- 为会话切换计算组。
- 扩缩容计算组，以及预期的数据重均衡或缓存影响。

说明操作影响当前用户、其他用户还是整个集群。

## 导入、导出、删除与更新

### 导入

Doris 导入文档通常区分：

- 实时写入：`INSERT`、Stream Load、Group Commit。
- 流式导入：Routine Load、Kafka Connector、Flink Doris Connector、Streaming Job。
- 批量文件：Broker Load、本地文件 Stream Load、`INSERT INTO SELECT`。
- 外部数据源集成：Catalog 查询加写入。

每种导入方式都应说明：

- 数据源和支持格式。
- 同步或异步行为。
- Label/Job 状态以及如何检查。
- 错误容忍和拒绝行。
- 事务和可见性行为。
- 认证和网络要求。
- CSV、JSON、Parquet、ORC 和复杂类型等格式相关处理。

### 更新与删除

清晰说明表模型支持：

- Unique Key 支持行级更新/删除和部分列更新。
- Aggregate Key 支持有限的部分列更新场景。
- Duplicate Key 适合追加明细场景，更新/删除能力有限。

说明操作是 SQL DML、导入时更新、Merge 还是 Delete Predicate。包含 Merge-on-Write、部分列更新和正确性敏感表属性的注意事项。

### 导出

导出文档应覆盖目标存储、路径、格式、凭证、异步 Job 状态、文件命名、权限和失败处理。

## 湖仓与 Catalog

Catalog 文档应分开说明：

- Catalog 连接什么数据源。
- 必需属性。
- 可选属性和默认值。
- 类型映射。
- 元数据缓存行为。
- 查询和写入支持。
- 谓词/函数下推支持。
- 安全和凭证处理。
- 故障排查。

常见 Catalog/数据源类型包括 Hive、Iceberg、Hudi、Paimon、Delta Lake、JDBC 数据库、Elasticsearch、Kudu、MaxCompute 和对象/文件存储。

### JDBC Catalog

JDBC Catalog 适用于数据集成、小型外部表和有限的跨源 Join。不要把它描述成大规模外部工作负载的查询加速机制。

要点：

- 必需属性包括 user、password、JDBC URL、driver URL 和 driver class。
- Driver URL 可以是配置驱动目录中的驱动文件名、本地绝对路径或 HTTP URL。
- 驱动包安全可能需要允许路径配置和 checksum。
- 连接池大小和超时设置影响并发和延迟。
- 谓词/函数下推依赖数据源，语义不一致时可能因正确性原因禁用。
- SQL 透传会把源端原生 SQL 直接发送给外部系统；Doris 不校验源端 SQL 语义。
- 透传查询和语句执行的权限不同。
- 凭证变更可能与连接池相互影响；要谨慎规划刷新或重启步骤。

## 查询加速运维

编写调优文档时，应从诊断开始：

- Schema：表模型、分桶键、Key 列、数据类型。
- 数据分布：倾斜、分区裁剪、Bucket 均衡。
- 统计信息：最近是否执行过 `ANALYZE`。
- 查询计划：`EXPLAIN`、Runtime Profile、Join 策略、扫描行数、Runtime Filter。
- 加速功能：同步或异步物化视图、索引、Colocate Join、Bucket Shuffle、Hint。

异步物化视图要点：

- 存储预计算结果，并通过任务刷新。
- 是最终一致。
- 全量刷新会重新计算全部数据；分区增量刷新只重新计算受影响分区。
- 透明改写取决于查询形态和支持的算子。
- 外表物化视图行为取决于元数据缓存和 Catalog 类型。

## 可观测性

可观测性文档通常覆盖：

- 日志、指标、Profile、审计/查询记录，以及 Prometheus/Grafana 或日志采集器等外部系统。
- 资源评估和 FE/BE 配置。
- 日志分析表设计。
- 采集链路、查询示例和故障排查。

撰写可观测性内容时，应包含数据量假设、Schema、保留策略、导入路径、看板/查询和故障诊断。

## 管理与故障排查领域

管理文档包括认证/授权、配置、Workload Group 和 Policy、Job、集群管理、系统表、Open API、日志、监控和数据修复。

常见故障排查主题包括：

- Compaction 原理和积压。
- Tablet 修复、Tablet 元数据工具、本地 Tablet 调试。
- FE 锁或元数据操作。
- 内存分析：全局内存、查询内存、导入内存、元数据内存、缓存、Heap Profile、jemalloc、OOM、进程内存超限、查询内存超限。
- 磁盘使用率和副本健康。

好的故障排查页面应包含现象、可能原因、诊断命令、结果解读、修复方式和升级处理条件。

## 连接器与生态文档

连接器页面应说明：

- 支持的版本范围。
- 数据流向：外部源到 Doris、Doris 到外部系统、联邦查询或双向。
- 必需的 Doris 表结构和外部系统准备。
- 配置属性和默认值。
- 投递语义、重试、幂等和失败恢复。
- 类型映射和不支持的类型。
- 吞吐调优参数。
- 验证步骤和常见错误。
