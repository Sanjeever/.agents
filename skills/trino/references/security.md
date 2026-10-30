# Trino 481 安全提炼

## 安全默认值与配置顺序

Trino 默认安装不启用安全功能。新集群建议按顺序逐步启用，不要混在一起调试：

1. 启用 TLS/HTTPS。优先使用负载均衡或反向代理终止 HTTPS，并使用受信任证书。
2. 配置内部通信 shared secret。
3. 启用认证。先用 password file 验证链路，再切换 LDAP、OAuth2、JWT、certificate、Kerberos 等。
4. 启用授权和访问控制。先用 file-based access control，再迁移到 OPA、Ranger 或自定义实现。

每一步变更后重启并验证，再进入下一步。

## TLS/HTTPS

- Trino 使用 HTTPS/TLS 保护客户端到 coordinator 通信。
- 可由负载均衡器终止 TLS，也可由 Trino coordinator 直接提供 HTTPS。
- 直接在 Trino 上启用 HTTPS 时，常用属性包括 `http-server.https.enabled`、`http-server.https.port`、`http-server.https.keystore.path`、`http-server.https.keystore.key`。
- 自签证书适合测试，不适合作为生产默认建议。
- 证书错误排查关注 SAN、hostname verification、truststore、keystore 格式和密码。

## 认证

Trino 认证在 coordinator 入口完成。支持：

- Password file：快速启动和测试认证链路。使用 password authenticator 配置和 password file。
- LDAP：常见企业认证方式，涉及 bind DN/password、search base/filter、TLS、AD/OpenLDAP 差异。
- OAuth 2.0/OIDC：浏览器外部认证和 token flow；Trino 481 JDBC 支持 access token 自动 refresh 场景，并支持系统级 token cache。
- JWT：通过 key file/JWKS、issuer、audience、principal field 验证 bearer token。
- Certificate：使用客户端证书和 user mapping。
- Kerberos：配置复杂，涉及 keytab、principal、service name、krb5 config、TLS 和 canonical hostname；能避免时不首选。
- Salesforce：通过 Salesforce 组织和 token 验证。

`http-server.authentication.type` 可配置多种认证类型。多 password authenticator 或 header authenticator 可通过对应 `*.config-files` 配置。

## 用户与组映射

- User mapping 将认证系统中的复杂用户名映射为 Trino 用户名，例如把邮箱映射为短用户名。
- Group mapping 将用户映射到组，供访问控制和资源组使用。
- Group provider 可为 file 或 LDAP。
- 映射规则常用 Java regex；要注意大小写、捕获组和不匹配时行为。

## 授权与访问控制

Trino 默认内置 system access control 允许所有已认证用户执行所有操作。生产环境应配置至少一种访问控制：

- `allow-all`：默认允许，适合测试。
- `read-only`：只读。
- File-based system access control：JSON 规则，支持 catalog/schema/table/column/function/procedure/table procedure/session property/query/impersonation/principal/system information 等规则。
- OPA access control：把决策交给 Open Policy Agent，支持 row filtering、column masking、batch column masking。
- Ranger access control：与 Apache Ranger 集成，可做细粒度权限、动态 row filter、column masking 和 audit。
- 自定义 system access control：通过 SPI 实现。

列权限行为：

- 默认 `SELECT *` 遇到无权限列会报错。
- 可用 `hide-inaccessible-columns=true` 静默隐藏不可访问列，但这会改变用户感知，应谨慎配置。

## File-based access control

规则可分为 system-level 和 catalog-level：

- System-level 管理 catalog 可见性、schema/table/column、function、procedure、table procedure、session property、query、impersonation、principal、system information、authorization 等。
- Catalog-level 可只作用于单个 catalog 的 schema/table/column/function/session property。
- 支持刷新周期，适合小中规模静态规则。
- 规则顺序、默认 deny/allow、正则匹配和 JSON 结构容易出错；审查时重点检查覆盖范围和最后兜底规则。

## 内部通信

- coordinator 与 worker 内部通信应使用 shared secret，属性为 `internal-communication.shared-secret`。
- 可要求内部 HTTPS：`internal-communication.https.required`。
- 可启用 HTTP/2：`internal-communication.http2.enabled`。
- 内部 TLS 会增加 CPU 成本，必要时调优 JVM secure random、TLS 参数和证书。

## Secrets

Catalog 和配置文件中的密码等 secret 可通过 secrets management 机制处理，避免明文落盘。写部署方案时不要把示例密码扩展成生产建议。

## 数据源安全

Trino 集群访问数据源的身份和权限由各 catalog/connector 配置决定。不要把 Trino 用户权限自动等同于底层数据源权限。常见模式：

- 使用共享服务账号连接数据源，由 Trino 访问控制做授权。
- 使用 Kerberos、OAuth、extra credentials 或 connector 支持的 impersonation/delegation。
- 对象存储权限、metastore 权限和表格式权限需要同时考虑。

## 客户端安全

- CLI/JDBC 支持 TLS、password、external authentication、certificate、JWT、Kerberos 等。
- JDBC `SSLVerification=FULL` 是默认；`NONE` 只适合临时诊断。
- JDBC `extraHeaders` 不能覆盖协议头，例如 `X-Trino-User`。
- `sessionUser` 和 impersonation 需要访问控制明确允许。
- `source`、`clientTags` 会影响 resource group selection，不应作为安全边界。
