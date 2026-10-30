# 抓取与索引参考

覆盖 Googlebot、移动端优先索引、网站迁移、A/B 测试、内容移除、AMP。

## 1. Googlebot

### 关键参数

- **两种抓取工具**：智能手机版（主要，用于移动端索引）和桌面版
- **用户代理**：通过 HTTP User-Agent 标头区分
- **robots.txt**：两者共享同一产品令牌（`Googlebot`），无法分别控制
- **渲染**：使用最新版 Chrome（无头 Chromium）
- **HTML 限制**：前 2MB 内容（PDF 为 64MB）
- **抓取频率**：大多数网站平均每几秒一次

### 验证 Googlebot 真实性

对请求来源 IP 地址做 DNS 反向查找，或与 Google 公布的 IP 范围比对。**不要仅依赖 User-Agent 字符串**。

### 抓取三阶段

1. **抓取**：检查 robots.txt → 发送 HTTP 请求 → 解析 HTML 中的 href 链接
2. **渲染**：HTTP 200 的页面进入渲染队列 → 无头 Chromium 执行 JS → 重新解析渲染后链接
3. **索引**：使用渲染后的 DOM 内容编入索引

## 2. 移动端优先索引

Google 默认使用**智能手机版 Googlebot** 抓取和索引。这是绝大多数网站的默认模式。

### 三种移动配置

| 配置 | 说明 | 推荐度 |
|------|------|--------|
| **自适应设计** | 同一 URL + 同一 HTML + CSS 媒体查询 | ⭐ 最推荐 |
| **动态提供** | 同一 URL，根据 User-Agent 返回不同 HTML | 可接受 |
| **单独 URL** | 不同 URL（m.example.com vs www.example.com） | 复杂 |

### 核心检查清单

- [ ] 移动版和桌面版使用相同的 robots meta 标签
- [ ] 移动版内容 ≥ 桌面版内容（不得少）
- [ ] 不依赖用户交互（滑动、点击、输入）加载主要内容
- [ ] Googlebot 可以抓取移动版资源（CSS、JS、图片）
- [ ] 桌面版和移动版使用相同的结构化数据
- [ ] 桌面版和移动版使用等效的 title 和 meta description

### 单独 URL 配置

桌面版：
```html
<link rel="canonical" href="https://example.com/page">
<link rel="alternate" media="only screen and (max-width: 640px)" href="https://m.example.com/page">
```

移动版：
```html
<link rel="canonical" href="https://example.com/page">
```

### 常见错误

| 错误 | 原因 | 解决 |
|------|------|------|
| 移动版缺少结构化数据 | 数据仅放在桌面版 | 两个版本都添加 |
| 移动版有 noindex | 两边 robots meta 不一致 | 统一规则 |
| 移动版图片缺失 | 不同步 | 确保内容对等 |
| 多个桌面版重定向到同一移动版 URL | 配置错误 | 1:1 映射 |
| 移动 URL 含片段（#） | 使用了片段标识符 | 移除片段 |

## 3. 网站迁移

### 迁移类型

1. **更改 URL**：域名变更、HTTP→HTTPS、路径重构
2. **不更改 URL**：仅更换托管商或 CDN

### 迁移前检查清单

- [ ] 新旧网站均已验证 Search Console 所有权（含所有变体）
- [ ] 新网站 robots.txt 不包含开发时的 noindex/Disallow
- [ ] 从最重要 URL 开始建立旧→新 URL 映射
- [ ] DNS TTL 降低至数小时（提前至少一周）
- [ ] 新网站服务器容量足够（迁移后 Google 会暂时增加抓取频率）

### 迁移中执行

- [ ] 实施所有 301/308 永久重定向
- [ ] 更新内部链接指向新 URL
- [ ] 新网站所有页面添加自引用 canonical
- [ ] 更新 hreflang 注解
- [ ] 提交新网站站点地图
- [ ] 在 Search Console 提交地址更改请求

### 迁移后监控

- [ ] 新旧网站流量趋势（旧站索引数归零 + 新站上升 = 完成）
- [ ] 监控 404 错误报告
- [ ] 至少保留重定向 1 年（推荐无限期）
- [ ] 预期排名波动：中型网站数周，大型网站更久
- [ ] 不要同时更改多项：域名 + CMS + 布局应分步进行

### 常见迁移错误

| 错误 | 解决 |
|------|------|
| 重定向指向不存在的 URL | 检查并修复 404 |
| 开发时的 noindex 未移除 | 上线前逐页检查 |
| 未更新站点地图 | 提交包含新 URL 的站点地图 |
| 服务器容量不足 | 提前扩容 |
| 使用 JavaScript 重定向而非服务器端重定向 | 改用 301/308 |

## 4. A/B 测试

### 原则

- 不伪装：Googlebot 和用户看到**相同内容**
- 变体 URL 用 `rel="canonical"` 指向原始 URL
- 使用 **302 临时重定向**（不用 301）
- 测试完成后立即移除测试元素（长期运行可能被视为欺骗）

## 5. 暂时关闭网站

### 推荐：限制功能而非停用

- 停用购物车 — 不影响搜索
- 显示横幅通知 — 用 data-nosnippet 防止内容进入摘要
- 更新结构化数据（库存状态、营业时间）
- 通过 Search Console 或站点地图触发重新抓取

### 不推荐：完全停用

即使只关闭几周也会对索引产生**重大负面影响**，无加速恢复机制。

临时紧急停用（1-2 天）可返回 503 + Retry-After 标头：
- 超过 2 天 → URL 从索引移除
- 不对 robots.txt 返回 503
- 不返回 403/404/410 或使用 noindex 来"暂停"
- 不在 Search Console 使用临时移除工具

长期停用应返回 200 并提供占位符首页。

## 6. 内容移除

### 方法对比

| 方法 | 适用 | 速度 | 持久性 |
|------|------|------|--------|
| 删除文件 | 所有类型 | 需等待重新抓取 | 永久 |
| 密码保护 | 所有类型 | 即时 | 永久 |
| noindex | 所有类型 | 需等待重新抓取 | 永久（保留规则） |
| robots.txt | 仅图片/视频 | 较慢 | 持久 |
| 移除工具 | 紧急情况 | ~1 天 | 约 6 个月 |
| 4xx/5xx | 临时 | 取决于状态码 | 临时 |

### 图片移除

**robots.txt（仅对 Googlebot-Image）：**
```
User-agent: Googlebot-Image
Disallow: /images/private/
```

**X-Robots-Tag（推荐，需允许抓取）：**
```
X-Robots-Tag: noindex
```

注意：`noimageindex` robots meta 只阻止**当前页面上的图片**被索引。同一图片在别的页面仍可能被索引。

### 隐去信息保护

- 不依赖超小字号、同色文字或图片遮盖 → 不是真正的隐去
- 在嵌入文档前先裁剪/修改图片
- 将文档转为不保留编辑历史的格式（PNG/WEBP）
- 文件名中的敏感信息用哈希替换

## 7. 请求重新抓取

| 场景 | 方法 |
|------|------|
| 少量 URL | Search Console 网址检查工具（有配额限制） |
| 大量 URL | 提交更新的站点地图 |
| 需等待 | 几天到几周不等，多次请求不缩短等待时间 |

## 8. AMP

### 核心准则

- 必须遵循 AMP HTML 规范
- 内容和功能尽可能与规范页相同
- URL 应易于理解（`example.com/amp/page` 而非 `test.com/page`）
- 必须有效（用 AMP 测试工具验证）
- 规范页必须链接到 AMP 页（`rel="amphtml"`）

### 移除 AMP

**仅移除 AMP 版本：**
1. 从规范页移除 `rel="amphtml"`
2. 对 AMP URL 设置 301/302 重定向到规范页
3. 最终可返回 404 或 301

**完全移除（所有版本）：**
1. 删除 AMP 和非 AMP 版本
2. 使用移除工具

## 9. Google 可索引的文件类型

### 文本类型
HTML、CSV、KML/KMZ、GPX、SVG、TeX/LaTeX、TXT、源代码（C/C++/C#/Java/Perl/Python）、WML、XML

### 编码文档
PDF、PostScript、EPUB、HWP、Excel、PowerPoint、Word、OpenOffice、RTF

### 媒体
- 图片：BMP、GIF、JPEG、PNG、WebP、SVG、AVIF
- 视频：3GP、AVI、MOV、MP4、MPEG、WebM、WMV 等

**探测方式**：优先看 Content-Type 响应标头；缺失或不正确时使用文件扩展名或解析内容判断。
