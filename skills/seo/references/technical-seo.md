# 技术 SEO 参考

覆盖 robots.txt、站点地图、规范化、URL 结构、meta 标签、重定向、JavaScript SEO。

## 1. 可索引性基础检查

### 1.1 robots.txt

**核心原则：robots.txt 阻止*抓取*，不阻止*索引*。** 被 robots.txt 屏蔽的页面如果被其他页面链接，仍可能出现在搜索结果中（但无摘要描述）。

正确用法：
- 阻止抓取低优先级资源（重复内容、过滤参数 URL）
- 阻止抓取会改变状态的 URL（购物车、登录、评论提交）
- 声明站点地图位置

错误用法：
- 用 robots.txt 阻止索引 → 应使用 noindex
- 用 robots.txt 隐藏敏感内容 → 应使用密码保护或 noindex
- 定期增删规则以"调整抓取预算" → 无效
- 对包含 noindex 的页面同时用 robots.txt 屏蔽 → Googlebot 看不到 noindex

配置模式：
```
User-agent: *
Disallow: /admin/
Disallow: /cart
Disallow: /*.pdf$
Allow: /

Sitemap: https://example.com/sitemap.xml
```

验证方法：Search Console → 设置 → robots.txt 报告；网址检查工具。

### 1.2 HTTP 状态码

| 状态码 | Google 处理方式 | 使用场景 |
|--------|---------------|---------|
| 200 | 正常索引 | 正常页面 |
| 301/308 | 永久重定向，目标 URL 成为规范 | 网站迁移、URL 变更 |
| 302/303/307 | 临时重定向，源 URL 仍在索引中 | A/B 测试、短期维护 |
| 404 | 页面不存在，从索引中移除 | 已删除的内容 |
| 410 | 已永久删除（比 404 处理更快） | 明确永久删除的页面 |
| 503 | 临时不可用（配 Retry-After 标头） | 1-2 天紧急停用 |
| soft 404 | 返回 200 但内容是错误页 → 需修复 | 应返回真实 404/410 或补充内容 |

**关键规则**：
- 用 503 停用不能超过 2 天，否则 URL 可能从索引中移除
- 不将多个不相关的旧 URL 全部重定向到首页（视为 soft 404）
- 重定向链不超过 5 跳（Googlebot 最多跟随 10 跳）

### 1.3 noindex

阻止页面出现在搜索结果中的**唯一正确方式**。实施前提：页面必须能被 Googlebot 抓取。

实施方式：
- HTML meta 标签：`<meta name="robots" content="noindex">`
- HTTP 标头（非 HTML 文件）：`X-Robots-Tag: noindex`
- 仅针对 Google：`<meta name="googlebot" content="noindex">`

常见错误：
- robots.txt 屏蔽 + noindex 同时使用 → noindex 不生效
- 在开发/过渡期间添加的 noindex 上线时忘记移除
- JavaScript 动态添加 noindex → 需要渲染后才能生效

## 2. 站点地图

### 2.1 何时需要站点地图

需要的情况：
- 网站很大（>500 个页面）
- 新网站，外部链接少
- 大量富媒体内容（视频、图片）或新闻
- 内容经常更新

不需要的情况：
- 网站较小（≤500 页）且内部链接完善
- 媒体文件/新闻内容不多

### 2.2 格式和限制

| 项目 | 限制 |
|------|------|
| 单个文件 | ≤50MB（未压缩），≤50,000 URL |
| Search Console 提交 | 最多 500 个站点地图索引文件 |
| 编码 | UTF-8 |
| URL 格式 | 绝对路径（完全限定 URL） |

### 2.3 最佳实践

- 只包含规范 URL（不含非规范变体）
- `<lastmod>` 必须准确（反映实质内容更新）
- `<priority>` 和 `<changefreq>` — Google 忽略
- 图片站点地图可帮助发现 JS 加载的图片
- 新闻站点地图只包含过去 2 天内的文章
- 视频站点地图可与 mRSS Feed 配合使用
- 可通过 robots.txt、Search Console 或 API 提交

### 2.4 站点地图扩展

| 扩展 | 用途 | 关键要求 |
|------|------|---------|
| 图片 | 发现 JS 加载的图片、CDN 图片 | 每个 `<url>` 最多 1000 个 `<image:image>` |
| 视频 | 提供时长、评分、年龄限制等 | 必须提供 content_loc 或 player_loc |
| 新闻 | 加速新闻发现 | 仅过去 2 天的文章，发布商名称须一致 |
| 组合 | 同时使用多个扩展 | 声明多个命名空间即可 |

## 3. 规范化 (Canonical)

### 3.1 规范化信号（按强度排序）

1. **重定向（301/308）** — 最强信号
2. **rel="canonical" link 元素或 HTTP 标头** — 强信号
3. **站点地图包含** — 弱信号
4. **其他自动信号** — HTTPS 优先、hreflang 集群

### 3.2 rel="canonical" 最佳实践

- 使用绝对路径 URL
- 放在 `<head>` 中
- 每个页面指向不同的规范 URL（不自引用则易出错）
- 不同方法（HTML、HTTP 标头、站点地图）指向同一 URL
- 不用 robots.txt 或网址移除工具做规范化
- 不同时使用 noindex 和 canonical（会在不同时间被分别处理）
- 转载内容 → 让转载方用 noindex，而非用 canonical 指向原创

### 3.3 常见规范化问题

| 问题 | 原因 | 解决 |
|------|------|------|
| 同一内容多个 URL | www/非 www、HTTP/HTTPS、参数排序/过滤 | 301 重定向 + canonical |
| 设备版本重复 | 桌面版和移动版不同 URL | 移动版 canonical 指向桌面版 |
| 区域版本混淆 | en-US 和 en-GB 被视为重复 | 使用 hreflang + 各自 canonical |
| CMS 自动生成错误 canonical | 插件或模板错误 | 检查并修复 CMS 配置 |
| 跨域 canonical 攻击 | 网站被入侵 | 修复安全漏洞 |

## 4. URL 结构

### 最佳实践

| 推荐 | 避免 |
|------|------|
| 描述性文字路径：`/wiki/Aviation` | 无意义参数：`/index.php?topic=42` |
| 连字符分隔：`/summer-clothing/filter?color=dark-grey` | 下划线：`/summer_clothing` |
| 统一小写 | 大小写混用（`/APPLE` vs `/apple` 视为不同 URL） |
| 参数用标准格式：`?key=value&key2=value2` | 非标准参数格式 |
| 减少不改变内容的参数 | 包含会话 ID、追踪代码、时间戳 |

### URL 问题处理

| 问题 | 解决方案 |
|------|---------|
| 累加过滤产生无限 URL | robots.txt 阻止或 nofollow 过滤链接 |
| 无限日历页（未来日期） | nofollow 动态未来链接 |
| 损坏的相对链接 | 使用根相对 URL |
| 会话 ID 参数 | 改用 Cookie |

## 5. 链接

### 可抓取的链接

Google 只能跟随 `<a>` 元素的 `href` 属性。以下方式抓取工具**无法**发现：
- `<span onclick="...">` 或 `<div onclick="...">`
- `<a onclick="..."`（无 href）
- `<a href="javascript:...">`
- 通过 JavaScript 事件处理程序导航
- 需要用户交互（点击、滑动、输入）才能加载的链接

### 锚文本最佳实践

- 描述性：`鬼椒种植指南` > `点击此处`
- 简洁有意义
- 自然融入上下文
- 图片链接：使用 alt 属性作为锚文本

### rel 属性速查

| rel 值 | 用途 |
|--------|------|
| `sponsored` | 广告/付费链接 |
| `ugc` | 用户生成内容（评论、论坛） |
| `nofollow` | 不希望传递排名信号 |
| 组合使用 | `rel="ugc nofollow"` |

## 6. Meta 标签

### Google 支持的 meta 标签

| 标签 | 作用 |
|------|------|
| `description` | 可能用于搜索结果摘要（非排名因素） |
| `robots` | 控制抓取/索引行为 |
| `googlebot` | 仅对 Google 生效的控制规则 |
| `notranslate` | 禁止 Google 提供翻译 |
| `viewport` | 移动设备呈现方式 |
| `rating` | 标记成人内容 |

### robots meta 常用规则

| 规则 | 作用 |
|------|------|
| `noindex` | 不显示在搜索结果中 |
| `nofollow` | 不追踪页面上的链接 |
| `nosnippet` | 不显示文本摘要或视频预览 |
| `max-snippet:[n]` | 限制摘要最多 n 个字符 |
| `max-image-preview:large` | 允许大图预览（Google 探索需要） |
| `notranslate` | 不提供翻译 |
| `noimageindex` | 不索引页面上的图片 |
| `unavailable_after:[date]` | 指定日期后移除 |

### 冲突处理

当 `robots` 和 `googlebot` 规则冲突时，应用**限制更严**的规则。

### data-nosnippet

在 `span`、`div`、`section` 上使用，排除特定部分不出现在摘要中。不要通过 JS 动态添加或移除。

### X-Robots-Tag

用于非 HTML 文件（PDF、图片、视频），语法与 robots meta 相同，支持正则匹配文件类型。

## 7. 重定向

### 类型对比

| 类型 | 实现方式 | Google 处理 |
|------|---------|-----------|
| 永久 | HTTP 301/308 | 目标 URL 成为规范 |
| 永久 | meta refresh（0 秒） | 视为永久重定向 |
| 临时 | HTTP 302/303/307 | 源 URL 保留在索引中 |
| 临时 | meta refresh（>0 秒） | 视为临时重定向 |

### 实施优先级

服务器端重定向 > meta refresh > JavaScript 重定向

### 最佳实践

- 避免重定向链 >3 跳
- 不要将所有旧 URL 重定向到首页
- 网站迁移后至少保留重定向 1 年
- 迁移前降低 DNS TTL 值

## 8. JavaScript SEO

### Google 处理 JS 三阶段

抓取（HTTP 请求解析 HTML）→ 渲染（无头 Chromium 执行 JS）→ 索引（使用渲染后 DOM）

### 关键约束

- Googlebot 不会点击、滑动、输入来触发内容加载
- 需要用户权限的 API（Camera、WebSocket）不可用
- localStorage、sessionStorage、Cookie 在渲染时被清除
- HTML 页面内容有 2MB 限制

### 常见问题和解决方案

| 问题 | 解决方案 |
|------|---------|
| JS 加载主要内容 | SSR、静态渲染或动态渲染 |
| 延迟加载内容 Googlebot 看不到 | 使用 Intersection Observer + 资源已加载的回退 |
| soft 404（SPA 非存在页面返回 200） | 重定向到 404 页面或动态注入 noindex |
| JS 修改 canonical | 不要在 JS 中覆盖 HTML 中已有的 canonical |
| 基于片段的导航（#） | 使用 History API |
| 缓存问题（Googlebot 使用旧版 JS） | 使用内容指纹（如 `main.2bb85551.js`） |

### 动态渲染

仅作为**临时解决**方案。长期应使用 SSR、静态渲染或 hydration。只要内容相似，不视为伪装。
