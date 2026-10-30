# Trino 481 文档写作与审查提炼

## 总体原则

- 按 Trino 481 事实口径写作，不要把其他版本行为混入。
- 不直接引用或链接原始文档文件；用提炼后的概念、规则和示例表达。
- 先解释功能是什么、适用场景、前置条件，再写配置、SQL、示例、限制和排障。
- 技术名词、配置属性、SQL 关键字、函数名、connector 名、Java 类名保持英文原样。
- 简体中文说明中避免把专有名词硬翻译到不可检索。
- 示例字段名、别名和 SQL 生成内容使用小写下划线。

## 文档结构建议

功能文档：

- 概述。
- 适用场景与限制。
- 前置条件。
- 配置步骤。
- 示例。
- 验证方式。
- 排障。
- 相关安全或性能注意事项。

SQL 语句文档：

- 标题为语句名。
- Synopsis。
- Description。
- Examples。
- Limitations 或 See also，按需要添加。
- 明确 connector 支持差异，特别是 DML、DDL、权限和事务。

Connector 文档：

- Requirements。
- Configuration。
- Multiple instances/catalogs。
- Security/connection security。
- Type mapping。
- SQL support。
- Procedures/table functions。
- Performance/pushdown/statistics。
- Limitations。

运维文档：

- 说明影响范围：coordinator、worker、所有节点、catalog、client 或外部服务。
- 标明配置文件位置类别，不依赖用户知道部署布局。
- 给出最小可运行配置，再说明生产配置差异。
- 写清 restart/reload 是否需要，以及如何验证。

## 审查清单

审查文档或变更时优先找具体问题：

- 事实是否限定为 Trino 481？是否误把 connector-specific 行为写成全局行为？
- 是否把 Trino 描述成 OLTP 数据库或传统 RDBMS 替代品？
- 是否说明 coordinator/worker、catalog/connector/schema/table 的边界？
- SQL 示例是否可运行？对象名、catalog/schema、别名、字符串引号是否一致？
- DML/DDL/transaction/security 语句是否说明 connector 支持差异？
- 配置示例是否区分 `node.properties`、`jvm.config`、`config.properties`、catalog properties、exchange manager、resource groups？
- 是否遗漏 Java 25 要求、TLS/认证/授权、内部 shared secret 等生产关键点？
- 对 Hive/Iceberg/Delta/Lakehouse，是否错误使用 legacy object storage 或旧 `hive.*` S3/Azure/GCS 属性？
- 对 pushdown、statistics、dynamic filtering 是否过度承诺性能收益？
- 对 table functions/procedures 是否说明权限、副作用和底层系统语义？
- 对 UDF 是否说明 inline/catalog 差异、SQL UDF 限制、Python UDF sandbox 限制？
- 对安全规则 JSON 是否说明规则顺序、默认行为、regex 和刷新？

## 示例写法

推荐：

```sql
SELECT order_status, count(*) AS order_count
FROM example.sales.orders
WHERE order_date >= DATE '2026-01-01'
GROUP BY order_status;
```

避免：

- 使用 `SELECT *` 展示权限或列级访问场景，除非就是要说明其风险。
- 在示例中混用大小写别名，如 `orderCount`。
- 给出生产密码、真实 token、真实 endpoint。
- 用 `SSLVerification=NONE` 作为默认连接建议。
- 说“所有 connector 支持 `MERGE`/事务/权限管理”。

## 版本敏感点

Trino 481 特别要注意：

- Java 25 服务端要求。
- `NUMBER`、`VARIANT`、Python UDF `number` 支持。
- `DESCRIBE OUTPUT` inline query。
- `NEAREST` join。
- `WITH SESSION`、`SET SESSION`、`CALL` 参数绑定。
- JDBC external authentication token cache 到 `~/.trino/`。
- Hive/Iceberg/Delta/Lakehouse 移除 legacy object storage 支持。
- JTS 替换 Esri geometry library 后，WKT 和 `ST_Union` 行为有 breaking changes。

## 回答风格

- 用户要方案时，先给最小配置或 SQL，再解释必要条件和限制。
- 用户要排障时，按“现象 -> 检查点 -> 命令/SQL -> 可能原因 -> 修复动作”组织。
- 用户要比较 connector 时，用表格列出能力差异，避免长段文字。
- 用户要审查时，先列 bug/风险/遗漏测试，再给摘要。
- 不确定精确默认值或完整支持矩阵时，明确说需要核对 Trino 481 文档或用户提供的材料。
