---
name: trino
description: Trino 481 文档知识技能，用于回答、起草、审查或实现 Trino 相关方案。适用于分布式 SQL 查询、SQL 语法与函数、数据类型、连接器与 catalog、Hive/Iceberg/Delta/Kafka/JDBC 数据源、对象存储、部署配置、安全认证授权、查询优化、容错执行、资源组、客户端、UDF、SPI 插件开发和 Trino 文档写作审查问题。
---

# Trino

使用本技能时，将事实口径限定为 Trino 481。内置资料是 Trino 481 文档的提炼，不是源文档摘录；回答、审查或写作时不要直接引用或链接原始文档文件。

如果用户明确询问其他 Trino 版本、升级差异或“最新”行为，先核对对应版本文档；不要把 Trino 481 的行为直接当作其他版本事实。

## 工作流程

1. 先判断任务类型和用户角色：SQL 用户、连接器使用者、集群管理员、安全管理员、插件开发者、文档作者或审查者。
2. 明确目标版本是否为 Trino 481。用户未说明版本时，按 Trino 481 回答并说明口径。
3. 按主题只读取必要参考资料，不要一次加载全部：
   - Trino 定位、架构、catalog/schema/table、执行模型、数据类型、客户端：读 `references/concepts.md`。
   - SQL 语句、查询结构、函数族、UDF、`NUMBER`、`VARIANT`、JSON、窗口和模式识别：读 `references/sql-and-functions.md`。
   - 连接器、catalog 配置、Hive/Iceberg/Delta/Kafka/JDBC 系连接器、对象存储和 pushdown：读 `references/connectors.md`。
   - 部署、配置属性、资源组、容错执行、查询优化、监控、日志、Web UI、客户端协议：读 `references/operations.md`。
   - TLS、认证、授权、访问控制、用户/组映射、内部通信、secret、数据源安全：读 `references/security.md`。
   - SPI、连接器开发、函数插件、事件监听器、认证/访问控制扩展、测试指南：读 `references/development.md`。
   - 文档写作、审查、示例、术语和版本一致性：读 `references/writing-review.md`。
4. 先给可执行的最小答案，再补充连接器差异、版本限制和运维风险。
5. 涉及属性默认值、完整类型映射、完整函数列表、连接器支持矩阵或安全规则 JSON 时，如果参考资料没有覆盖，应核对 Trino 481 文档或用户给定材料。

## 默认立场

- Trino 是面向大规模分析的分布式 SQL 查询引擎，不是 OLTP 数据库，也不是 MySQL/PostgreSQL/Oracle 的替代品。
- coordinator 负责解析、计划、调度和客户端连接；worker 执行 task、读取 connector split、交换中间数据。
- catalog 是 connector 的配置实例；同一种 connector 可以配置多个 catalog。完整对象名按 `catalog.schema.object` 定位。
- SQL 能否读写、执行事务、管理权限、支持表过程或 pushdown，首先取决于 connector 和底层数据源。
- 生产环境优先关注 TLS/HTTPS、认证、授权、内部通信 secret、资源组、查询内存和容错执行；不要只给单机测试配置。
- 示例 SQL 的字段名或别名使用小写下划线。配置属性、函数名、类型名、connector 名和 Java 类名保持英文原样。

## Trino 481 重点

- 运行 Trino 481 服务端需要 64-bit Java 25，最低 25.0.1；Java 8/11/17/21/24 不适用。
- `NUMBER` 支持更高精度的非定长数字语义；Trino 481 增强了 `boolean`、`json`、Python UDF 与 `number` 的转换和使用。
- `VARIANT` 是半结构化值类型，遵循 Iceberg Variant 规范；CLI/JDBC 在 481 支持 `variant`，Iceberg v3 表有实验性支持。
- `DESCRIBE OUTPUT` 支持 inline query，例如直接描述 `SELECT` 的输出。
- `SELECT` 的 join 支持 `NEAREST` 近似匹配子句。
- `WITH SESSION`、`SET SESSION`、`CALL` 支持绑定参数。
- Hive、Iceberg、Delta Lake、Lakehouse 在 481 移除 legacy object storage 支持；Azure/GCS/S3 使用 native file system 支持，`fs.hadoop.enabled` 只用于 HDFS。
