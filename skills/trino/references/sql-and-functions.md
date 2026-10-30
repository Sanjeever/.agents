# Trino 481 SQL 与函数提炼

## 查询结构

`SELECT` 支持常见 SQL 分句与 Trino 扩展：

- `WITH SESSION`：为单条查询设置 session property；Trino 481 支持绑定参数。
- `WITH FUNCTION`：定义 inline UDF。
- `WITH` / `WITH RECURSIVE`：公共表表达式；递归能力有限，注意终止条件和深度。
- `GROUP BY`：支持 `GROUPING SETS`、`CUBE`、`ROLLUP` 和 `GROUPING()`。
- Set operations：`UNION`、`INTERSECT`、`EXCEPT`。
- Ordering and limiting：`ORDER BY`、`OFFSET`、`LIMIT`、`FETCH FIRST`。
- `TABLESAMPLE`、`UNNEST`、`LATERAL`、子查询、`EXISTS`、`IN`、scalar subquery。
- `JSON_TABLE`：把 JSON 转换成关系行列。
- `MATCH_RECOGNIZE` 和窗口内 row pattern recognition：用于序列模式识别。
- Joins：`CROSS JOIN`、普通 join、`LATERAL`，Trino 481 增加 `NEAREST` 近似匹配 join 子句。

写示例时优先使用最小完整 SQL。对 connector 相关 DML/DDL，说明“语法存在”不等于“所有 connector 支持”。

## DDL、DML 与管理语句

常见语句：

- Catalog：`CREATE CATALOG`、`DROP CATALOG`，用于 dynamic catalog 管理。
- Schema/table/view/materialized view：`CREATE`、`DROP`、`ALTER`、`COMMENT`、`SHOW CREATE`。
- Data：`INSERT`、`UPDATE`、`DELETE`、`MERGE`、`TRUNCATE`。
- Prepared statements：`PREPARE`、`EXECUTE`、`DEALLOCATE PREPARE`，也可用 `EXECUTE IMMEDIATE` 降低客户端交互开销。
- Introspection：`DESCRIBE`、`DESCRIBE INPUT`、`DESCRIBE OUTPUT`、`SHOW STATS`、`SHOW FUNCTIONS`。
- Security：`GRANT`、`DENY`、`REVOKE`、role 相关语句。
- Transaction：`START TRANSACTION`、`COMMIT`、`ROLLBACK`，多数 connector 不支持事务。
- Table procedures：`ALTER TABLE ... EXECUTE` 调用 connector 提供的表过程，例如 lakehouse 文件优化。

Trino 481 的 `DESCRIBE OUTPUT` 可直接接受 inline query，不再必须先 `PREPARE`。

## 函数族

函数按主题组织，回答时优先定位到族，再给具体函数：

- Aggregate：一般聚合、bitwise、map、approximate、statistical、lambda aggregate。
- Array/map/row：数组、map、结构类型处理，常与 lambda 配合。
- Binary/bitwise/string/regexp/url/uuid/ipaddress。
- Comparison、conditional、logical、conversion。
- Date and time：时间运算、截断、interval、duration parse、MySQL/Java 日期格式、extract。
- Decimal、math、random、trigonometric、statistical distribution。
- JSON：SQL/JSON path、`json_exists`、`json_query`、`json_value`、`json_table`、`json_array`、`json_object`、cast、legacy JSON helpers。
- VARIANT：相等语义、下标访问、cast to/from scalar 和 container。
- Geospatial：构造、关系判断、操作、访问器、聚合、Bing tile、encoded polyline。Trino 481 使用 JTS 后更严格遵循 OGC，部分旧 WKT 会失败。
- Sketch：HyperLogLog、SetDigest、QDigest、TDigest、DataSketches。
- Window：聚合窗口、ranking、value functions。
- Table functions：内置 `exclude_columns`、`sequence`，以及 connector 提供的 table functions。
- AI functions：通过 provider 配置调用 Anthropic/OpenAI/Ollama 等模型。使用前必须明确服务端 provider 和 model 配置。

## NULL 与条件

- `IS NULL`、`IS NOT NULL`、`IS DISTINCT FROM`、`IS NOT DISTINCT FROM` 是处理 `NULL` 的核心工具。
- `TRY` 可把某些运行时错误转换为 `NULL`，适合脏数据转换；不要用来隐藏业务错误。
- `COALESCE` 返回第一个非空值；Trino 481 PostgreSQL connector 可下推 `COALESCE` 表达式计算。
- 聚合函数对 `NULL` 的处理各异。给出聚合示例时要说明是否忽略 `NULL`。

## JSON

Trino 同时支持 `JSON` 类型、SQL/JSON path 与一组 JSON 构造/查询函数：

- SQL/JSON path 有 strict/lax 模式。lax 模式会自动展开/包装数组，更宽松但也更容易产生非预期匹配。
- `json_exists` 判断路径是否存在。
- `json_query` 返回 JSON 片段，可控制 quote wrapper、empty/error 行为。
- `json_value` 返回标量 SQL 值，可配置 `ON EMPTY` 和 `ON ERROR`。
- `json_table` 把 JSON 映射成表；Trino 481 对 `ON EMPTY` 和 `ON ERROR` 采用惰性求值以改善性能。
- 481 修复了含超过 16 位小数有效数字的 JSON 数字解析正确性问题。

## NUMBER 与 VARIANT

`NUMBER`：

- 表示至少 50 位十进制精度的非定长数字。
- 支持 `Infinity`、`-Infinity`、`NaN`，排序约定为 `-Infinity < finite < Infinity < NaN`。
- Trino 481 支持 `boolean` 到 `number`、`number` 与 `json` 的 cast，并支持 Python UDF 中的 `number`。

`VARIANT`：

- 可表示 object、array、string、number、boolean、null、date/time 等半结构化值。
- 不同于 `JSON`，`VARIANT` 保留更完整的底层类型语义。
- 支持对象/数组下标访问和 cast to/from scalar/container。
- Iceberg v3 对 `VARIANT` 支持仍应视为实验能力；客户端需用 481 或更新版本才能更好展示。

## UDF

UDF 是返回单个标量值的用户自定义函数。可声明为 inline UDF 或 catalog UDF。

Inline UDF：

- 在 `WITH FUNCTION` 中声明，只在当前 query 上下文有效。
- 名称不能包含 `.`。
- 多个 inline UDF 用逗号分隔，必须先声明后调用。
- 可遮蔽内置函数名，示例和审查时要避免误导。

Catalog UDF：

- 存储在支持 UDF 存储的 catalog 中。Trino 481 文档列出的支持者包括 Hive 和 Memory。
- 使用 `CREATE FUNCTION`、`DROP FUNCTION`、`SHOW FUNCTIONS` 管理。
- 调用通常使用完整名 `catalog.schema.function`，也可通过 `sql.default-function-catalog`、`sql.default-function-schema` 和 `sql.path` 设置默认位置。

SQL UDF：

- 使用 SQL routine 语句：`BEGIN`、`CASE`、`DECLARE`、`IF`、`ITERATE`、`LEAVE`、`LOOP`、`REPEAT`、`RETURN`、`SET`、`WHILE`。
- 支持 named arguments，位置参数必须在 named arguments 之前。
- 支持 labels，用于 `ITERATE` 和 `LEAVE` 控制嵌套块。
- 不支持递归、互递归，不允许在 UDF 内执行查询；UDF 只能处理输入值并通过 `RETURN` 输出。
- 默认是 `CALLED ON NULL INPUT`；多数 UDF 应显式使用 `RETURNS NULL ON NULL INPUT`，除非有特殊 null 处理。

Python UDF：

- 使用 `LANGUAGE PYTHON`、`WITH (handler = '...')` 和 `$$` 包裹 Python 代码。
- Python 运行在 Java JVM 内的 WASM sandbox，语言版本为 Python 3.13.0。
- 不支持网络、文件系统、包管理器、外部库；部分标准库被移除，另有少量库被加入。
- Trino 到 Python 类型映射包括 `ROW`->`tuple`、`ARRAY`->`list`、`MAP`->`dict`、`DECIMAL`->`decimal.Decimal`、`JSON`->`str`、`UUID`->`uuid.UUID`、`IPADDRESS`->`ipaddress` 类型。
- Python `datetime`/`time` 仅支持微秒精度；更高精度会四舍五入。带时区 timestamp 只支持 fixed offset。

UDF 性能建议：

- 避免在循环中反复创建数组或拼接字符串；优先使用 lambda 表达式或内置函数。
- UDF 可能消耗较多 CPU/内存；复杂逻辑应评估是否改为 Java plugin function。
