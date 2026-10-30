# Trino 481 开发与扩展提炼

## 插件与 SPI

Trino 通过 plugin architecture 扩展 connector、函数、event listener、认证、访问控制、group provider 等能力。插件通过 Java SPI 与 Trino 集成，并部署到 plugin 目录。

开发建议：

- 先确认扩展点：connector、function、table function、event listener、password/header/certificate authenticator、system access control、group provider。
- 插件元数据和 service loader 配置必须完整。
- 打包时避免依赖冲突，必要时 shade。
- 部署后每个节点都需要安装相同插件，并重启。

## Connector 开发

核心接口和职责：

- `ConnectorFactory`：创建 connector，读取配置并注册服务。
- `ConnectorMetadata`：schema/table/view/materialized view、列、统计、DDL/DML、权限相关元数据。
- `ConnectorSplitManager`：枚举 splits，是读取并行度的入口。
- `ConnectorRecordSetProvider` 或 `ConnectorPageSourceProvider`：读取数据。
- `ConnectorPageSinkProvider`：写入数据。
- Type mapping：把远端类型映射为 Trino 类型，并处理 Trino 写回远端系统。

Pushdown 实现维度：

- Limit 和 Top-N pushdown。
- Predicate pushdown。
- Projection 和 dereference pushdown。
- Aggregation pushdown。
- Join pushdown。

实现 pushdown 时要保证正确性优先于性能。不能下推的表达式必须留给 Trino 执行，避免远端 SQL 语义差异导致错误结果。

## 写入与 MERGE 支持

`INSERT`、`CREATE TABLE AS`、`MERGE`、`UPDATE`、`DELETE` 支持需要 connector 明确实现：

- 写入路径通常涉及 page sink、transaction/commit/rollback、writer scaling、file/table handle。
- `MERGE` 需要处理 standard SQL 语义、重复匹配检测、row change paradigm、redistribution、connector merge handle、merge sink 和 metadata API。
- 非事务 connector 的 `INSERT`/`MERGE` 可能有部分成功风险，应在文档和错误处理上明确。

## 函数插件

Java 函数插件可提供 scalar function、parametric scalar function、aggregation function，也可声明 deprecated function。

实现建议：

- 使用 Trino 类型系统，不要把 Java 类型假设等同于 SQL 类型。
- 对 null convention、determinism、hidden/deprecated 标记保持明确。
- Parametric function 要约束类型变量，避免运行时类型错误。
- Aggregation function 要清晰定义 state、input、combine、output。

## Table functions

Table function declaration 包括名称、参数和返回 schema 分析逻辑。参数类型包括：

- Scalar arguments。
- Descriptor arguments。
- Table arguments，可指定 set/row semantics、prune/keep when empty、pass-through columns。

`analyze()` 决定返回类型和执行计划信息。执行阶段要尊重 access control 和表参数语义。

## Event listeners

Event listener 可接收 query created/completed、split completed 等事件。Trino 481 文档覆盖 HTTP、Kafka、MySQL、OpenLineage 等内置 listener 配置，也支持自定义 event listener 插件。

实现时注意：

- listener 不能拖慢 query critical path。
- 失败策略、重试、批量、异步发送、匿名化和字段过滤要明确。
- OpenLineage 用于血缘和 query metadata，不应替代权限审计。

## 认证与访问控制扩展

可实现：

- Certificate authenticator。
- Password authenticator。
- Header authenticator。
- Group provider。
- System access control。

安全扩展必须清楚区分 authenticated user、session user、original user、groups、principal，并且在 impersonation、role、catalog/schema/table/function/procedure/session property/query 操作上做一致决策。

## Client protocol

Trino client REST API 使用 HTTP：

- 客户端提交 SQL，服务端返回 `QueryResults`。
- 客户端根据 `nextUri` 拉取后续结果。
- 请求/响应头承载 user、source、catalog、schema、path、session properties、prepared statements、roles、extra credentials、client tags、transaction id 等。
- Spooling protocol 会返回结果段位置，客户端再取回。

开发客户端时不要伪造或覆盖 Trino 协议保留头。大结果优先支持 spooling 和压缩编码。

## 测试指南

Trino 测试指南强调高价值、低脆弱性：

- 避免组合爆炸式参数化测试。
- 避免为了测试而创建复杂抽象。
- 避免 stateful test class 和测试继承。
- 避免 flaky tests、禁用测试、用 `Assumptions.abort()` 隐藏问题。
- 保持 setup/teardown 简单，资源生命周期明确。
- 对插件和 connector 功能，测试重点应覆盖 connector 行为、SQL 语义、错误路径和与 Trino SPI 的集成。

实现变更时优先写能证明行为的最小测试，不要用大量重复断言制造维护成本。
