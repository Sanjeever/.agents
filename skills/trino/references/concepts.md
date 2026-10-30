# Trino 481 概念提炼

## 定位

Trino 是分布式 SQL 查询引擎，适合 OLAP、数据仓库、数据湖、湖仓、跨源联邦查询、批量分析、报表和交互式分析。不要把它描述成通用关系数据库或 OLTP 系统；事务、索引、行级写入、锁和约束能力通常由 connector 与底层系统决定。

## 架构

- **Cluster**：一个 Trino 集群包含一个 coordinator 和零个或多个 worker。生产查询集群通常需要 worker；单进程兼任 coordinator/worker 适合开发或测试。
- **Coordinator**：接收客户端请求，解析 SQL，生成逻辑和分布式计划，调度 worker，管理查询生命周期。客户端只连接 coordinator。
- **Worker**：执行 task，读取 connector split，处理数据并与其他 worker 交换中间结果。
- **Node**：运行 Trino JVM 的实例。文档里 node 常指进程，也常借指承载该进程的机器。
- **REST/HTTP**：coordinator、worker 和客户端协议都基于 HTTP/HTTPS。

## 数据访问模型

- **Connector** 类似数据库驱动，是 Trino 与数据源交互的 SPI 实现。
- **Catalog** 是一组 connector 配置属性。catalog 文件名决定 catalog 名；每个 catalog 必须指定 `connector.name`。
- **Schema** 组织表、视图、物化视图等对象，具体含义随数据源变化。
- **Table** 是无序行集合，列带 Trino 类型。源类型到 Trino 类型、Trino 类型写回源类型都由 connector 决定。
- 完整对象名是 `catalog.schema.table` 或 `catalog.schema.function`。跨 catalog 查询是 Trino 的核心价值，但跨源 join 的性能取决于数据规模、pushdown 和网络传输。

## 执行模型

- **Statement** 是用户提交的 SQL 文本。
- **Query** 是 statement 被解析后产生的执行实例，包含计划、stage、task、split、connector 访问等运行时状态。
- **Stage** 是分布式计划的树状分段，由 coordinator 管理，不直接在 worker 上运行。
- **Task** 是 stage 在 worker 上的执行单元。
- **Split** 是 connector 提供的可并行读取数据片段。
- **Driver** 是 task 内部的并行执行流水线。
- **Operator** 消费、转换、产生数据，例如 scan、filter、join、aggregate。
- **Exchange** 在 stage/task 之间传输中间数据；容错执行会把 exchange 数据 spool 到外部存储。

## 数据类型

核心类型族：

- Boolean：`BOOLEAN`。
- Integer：`TINYINT`、`SMALLINT`、`INTEGER`/`INT`、`BIGINT`。
- Floating point：`REAL`、`DOUBLE`。
- Exact numeric：`DECIMAL(p, s)`，最高 38 位精度，18 位以内性能更好。
- High precision numeric：`NUMBER`，非定长高精度数字，支持 `Infinity`、`-Infinity`、`NaN`，除以零报错；`NaN` 不等于任何值，包括自身。
- String/binary：`VARCHAR`、`CHAR`、`VARBINARY`、`JSON`、`VARIANT`。
- Date/time：`DATE`、`TIME(p)`、`TIME WITH TIME ZONE`、`TIMESTAMP(p)`、`TIMESTAMP(p) WITH TIME ZONE`、`INTERVAL YEAR TO MONTH`、`INTERVAL DAY TO SECOND`。
- Structural：`ARRAY`、`MAP`、`ROW`。
- Specialized：`IPADDRESS`、`UUID`、`HyperLogLog`、`P4HyperLogLog`、`SetDigest`、`QDigest`、`TDigest`。

类型注意事项：

- 数字字面量允许用下划线提高可读性，但不能前置、后置、连续或贴着小数点。
- `TIMESTAMP` 是 `TIMESTAMP(3)` 的别名；`TIMESTAMP(p)` 支持最高 12 位小数秒精度。降低精度会四舍五入，不是截断。
- `TIMESTAMP WITH TIME ZONE` 表示具体 instant，可使用 UTC/GMT/UT、数值 offset 或 IANA 时区名。
- `VARIANT` 与 `JSON` 不同：`VARIANT` 保留底层值类型，适合半结构化数据；Trino 481 对 Iceberg v3 的 `VARIANT` 支持是实验性的。
- Connector 不要求支持全部 Trino 类型。任何读写、CTAS、predicate pushdown 或表达式下推都要考虑 connector 类型映射。

## SQL 支持边界

核心引擎全局可用的语句包括 `CALL`、`PREPARE`、`EXECUTE`、`EXECUTE IMMEDIATE`、`DESCRIBE INPUT`、`DESCRIBE OUTPUT`、`EXPLAIN`、`EXPLAIN ANALYZE`、`SET SESSION`、`RESET SESSION`、`SET TIME ZONE`、`SHOW FUNCTIONS`、`SHOW SESSION`、`USE`、`VALUES`。

读元数据和读数据语句通常由所有 connector 支持，例如 `SELECT`、`DESCRIBE`、`SHOW CATALOGS`、`SHOW SCHEMAS`、`SHOW TABLES`、`SHOW COLUMNS`、`SHOW STATS` 等。

写入、DDL、权限和事务语句高度依赖 connector：

- 数据写入：`INSERT`、`UPDATE`、`DELETE`、`TRUNCATE`、`MERGE`。
- 对象管理：`CREATE/DROP/ALTER TABLE`、`CREATE/DROP/ALTER SCHEMA`、视图、物化视图、UDF。
- 安全操作：`CREATE ROLE`、`GRANT`、`DENY`、`REVOKE`、`SET ROLE`。
- 事务：`START TRANSACTION`、`COMMIT`、`ROLLBACK`。

## 客户端

- CLI、JDBC、Web UI、Python/Go 等客户端都连接 coordinator。
- JDBC URL 支持 `jdbc:trino://host:port`、`jdbc:trino://host:port/catalog`、`jdbc:trino://host:port/catalog/schema`。
- JDBC driver 要求 Java 11+；Java 22+ 对解压性能更好。driver 版本建议与集群相同或更新。
- JDBC 用户需要访问 `system.jdbc` schema 中的表。
- JDBC 支持 URL 参数或 `Properties` 传参，但同一参数不要同时放两处。
- JDBC/CLI 可使用 spooling protocol 提升大结果传输吞吐；客户端 JVM 或进程必须能访问配置的对象存储。
- Trino 481 JDBC 可通过 `externalAuthenticationTokenCache=SYSTEM` 把外部认证 token 持久化到 `~/.trino/`，跨 JDBC/CLI 进程复用。
