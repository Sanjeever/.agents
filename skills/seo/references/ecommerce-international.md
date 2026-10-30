# 电商 SEO 与国际化网站参考

覆盖电子商务网站 SEO、多语言/多区域网站、hreflang、露骨内容处理。

## 第一部分：电子商务 SEO

### 1. 网站结构

#### 导航设计原则

- Google 通过页面间的链接关系推断网站结构
- 确保从菜单→类别→子类别→**每个商品详情页**的链接路径完整
- 搜索框**不能替代**导航链接（Googlebot 通常不提交搜索表单）
- 所有希望被索引的商品必须通过 `<a href>` 链接可达
- 热门商品可从首页或博文直接链接（增加页面重要性信号）
- 无法通过链接覆盖的，用站点地图或 Merchant Center Feed 补充

#### 分页与加载

| 模式 | 做法 | SEO 要求 |
|------|------|---------|
| **分页** | 每页提供指向下一页的 `<a href>` | 每页唯一 URL（如 `?page=n`） |
| **"加载更多"** | 同一页面追加内容 | 用站点地图/Feed 补充 |
| **无限滚动** | 滚动加载 | Googlebot 不触发滚动，**必须**提供分页后备 |

分页最佳实践：
- 不要将第一页设为所有分页的 canonical
- Google 不再使用 `rel="next"`/`rel="prev"`（但其他搜索引擎可能使用）
- 不重要的过滤/排序变体用 noindex 或 robots.txt 排除

### 2. 商品 URL 结构

#### 最佳实践

| 推荐 | 避免 |
|------|------|
| `/product/black-t-shirt` | `/product/3243` |
| 每个变体独立 URL | 所有颜色共用同一 URL |
| 自引用 canonical | canonical 指向不同 URL |
| `<a href>` 导航 | JavaScript 事件导航 |
| 大小写统一 | 混用 |

#### 查询参数

- 使用标准格式：`?key=value&key2=value2`
- 不包含会话 ID、追踪代码、时间戳、用户位置等临时参数
- 排除重复的参数名

#### 变体/规格 URL

- 每个变体独立 URL：路径 `/t-shirt/green` 或参数 `?color=green`
- 不带查询参数的版本作为规范 URL

### 3. 电商结构化数据优先级

| 优先级 | 类型 | 说明 |
|--------|------|------|
| 🔴 必需 | Product + Offer | name、price、priceCurrency、availability |
| 🔴 必需 | BreadcrumbList | 面包屑导航路径 |
| 🟡 推荐 | Organization | 徽标、联系信息 |
| 🟡 推荐 | Review / AggregateRating | 评价和评分 |
| 🟡 推荐 | ProductGroup | 多款式/规格管理 |
| 🟢 可选 | MerchantReturnPolicy | 退货政策 |
| 🟢 可选 | ShippingService | 配送政策 |
| 🟢 可选 | VideoObject | 商品演示视频 |
| 🟢 可选 | LocalBusiness | 实体店信息 |

### 4. 商品数据分享

#### Merchant Center

- **"Google 购物"标签页**展示的必需条件
- 结构化数据与 Feed 数据必须一致
- 小型网站：自动 Feed + 结构化数据提高准确度
- 大型/频繁变化网站：定期 Feed 上传 + Content API 即时更新
- 商品售罄后：确保 Merchant Center 根据网站数据自动更新

#### 数据延迟处理

- 网站即时标记售罄，但 Merchant Center 可能延迟更新
- 解决方案：开启"根据网站内容自动更新"功能

### 5. 新电商网站发布

#### 四种策略

| 策略 | 做法 | 优点 | 缺点 |
|------|------|------|------|
| 全面发布 | 密码保护全站，择时开放 | 内容保密，配合营销 | 收录慢 |
| 首页发布 | 仅公开首页 + 占位符 | 提前建立域名知名度 | 商品不展示 |
| 发布不销售 | 全站上线，标记缺货 | 提前发现问题 | 客户无法下单 |
| 软发布 | 全功能上线，低调 | 可做真实测试 | 用户提前传播 |

**注意**：不要停用"加入购物车"按钮（Google 验证价格时需要）。

#### 发布后步骤

1. 验证网站所有权
2. 请求编入索引（少量用网址检查工具，大量用站点地图）
3. 跟踪索引进度（"网页索引编制"报告）
4. 有实体店 → 建立 Google 商家详情
5. 注册 Merchant Center

### 6. 商品评价撰写指南

优质评价特征：
- 从用户视角评估
- 展示专业知识和对主题的深入了解
- 提供一手证据（影像、音频、量化数据）
- 说明与竞品的对比和取舍
- 讨论优缺点，基于原创研究
- 聚焦最重要的决策因素
- 推荐时附上理由和一手证明
- 排名列表：每个项目独立且详实的说明

### 7. 内容展示平台汇总

| 平台 | 条件 |
|------|------|
| Google 搜索（商品摘要） | SEO + Product 结构化数据 |
| Google 图片 | 图片 SEO 最佳实践 |
| Google 购物标签页 | **必须注册 Merchant Center** |
| Google 商家资料 | 声明所有权 + 关联 Merchant Center |
| Google 地图 | 上传含位置数据的商品到 Merchant Center |
| Google 智能镜头 | 上传商品到 Merchant Center + 启用 + 图片最佳实践 |

---

## 第二部分：国际化与多语言网站

### 1. 网址结构选择

| 结构 | 示例 | 优点 | 缺点 |
|------|------|------|------|
| **ccTLD（推荐）** | `example.de` | 地理定位明确，与服务器位置无关 | 成本高，基础设施多 |
| **子域名** | `de.example.com` | 易设置，可不同服务器 | 用户可能无法从 URL 判断 |
| **子目录** | `example.com/de/` | 易设置，维护成本低 | 单一服务器位置 |
| **URL 参数** | `example.com?loc=de` | 无 | **不推荐** |
| **Cookie/浏览器设置** | 动态调整 | 无 | **不推荐** |

**不要使用**：IP 位置分析（不可靠）、Accept-Language 标头（Googlebot 不设置）。

### 2. Google 确定目标语言/区域的依据（优先级排序）

1. ccTLD（国家/地区代码顶级域名）
2. hreflang 注解
3. 服务器 IP 位置（不精确，CDN 环境下尤其）
4. 其他线索：本地地址/电话、本地语言和货币、本地链接

### 3. hreflang 实施

#### 三种方式（等价）

1. **HTML 标记**（最常用）：
```html
<link rel="alternate" hreflang="en" href="https://example.com/en/page" />
<link rel="alternate" hreflang="de" href="https://example.com/de/page" />
<link rel="alternate" hreflang="x-default" href="https://example.com/" />
```

2. **HTTP 标头**（非 HTML 文件）：
```
Link: <https://example.com/en/file.pdf>; rel="alternate"; hreflang="en"
```

3. **站点地图**：
```xml
<url>
  <loc>https://example.com/en/page</loc>
  <xhtml:link rel="alternate" hreflang="de" href="https://example.com/de/page"/>
  <xhtml:link rel="alternate" hreflang="en" href="https://example.com/en/page"/>
</url>
```

#### 核心规则

- 每个语言版本必须列出**自身以及所有其他版本**
- 必须有**返回链接**（A→B 则 B→A），否则标记被忽略
- 备用 URL 必须是**完全限定 URL**（含协议）
- 备用 URL 可位于不同网域
- 提供 `x-default` 处理不匹配的语言/区域

#### 语言/区域代码格式

| 代码 | 含义 |
|------|------|
| `de` | 德语，不限区域 |
| `en-GB` | 面向英国的英语 |
| `zh-Hans` | 中文（简体） |
| `zh-Hant` | 中文（繁体） |
| `zh-Hans-US` | 面向美国用户的中文（简体） |
| `x-default` | 不匹配时使用 |

语言：ISO 639-1；区域：ISO 3166-1 Alpha 2；文字：ISO 15924。

#### 常见错误

| 错误 | 说明 |
|------|------|
| 缺少返回链接 | A→B 但 B 没有→A |
| 语言代码格式错误 | `EN`（应为 `en`）、只写区域不写语言 |
| 区域代码无效 | `UK`（应为 `GB`）、`EU`（不是国家代码） |
| 使用相对 URL | 必须是完全限定 URL |
| 语言区域自适应 | 根据 IP 切换内容，Googlebot 默认来自美国 |

### 4. 语言区域自适应网页的特殊问题

如果网站根据访问者 IP/浏览器设置动态调整内容：
- Googlebot 的大部分抓取 IP 在美国
- Googlebot **不设置** Accept-Language 标头
- 其他语言版本可能不被抓取、索引或排名

**建议**：使用单独 URL 配合 hreflang 注解代替自适应内容。

### 5. 多语言内容管理

- 不同语言版本使用不同 URL（强烈建议）
- 各页面内容和导航使用同一种语言
- 避免原文和译文并行显示
- 提供指向其他语言版本的明显超链接（不要自动重定向）
- Google 通过**可见内容**确定语言，不依赖代码级 `lang` 属性

### 6. 处理重复内容

同一网域内同一语言的不同区域变体（如 en-US 和 en-GB 内容相似）：
- 设定首选版本
- 结合使用 `rel="canonical"` 和 `hreflang`

---

## 第三部分：露骨内容网站

### 安全搜索过滤范围

- 露骨色情内容
- 裸露内容
- 逼真的情趣玩具
- 以性为目的的约会/陪侍服务
- 暴力或血腥内容
- 指向以上内容的链接

有教育、纪实、科普或艺术价值的内容**可以**显示。

### 最佳实践

1. 防止用户生成有害内容（审核流程、哈希匹配检 CSAM）
2. 允许 Google 提取视频内容文件（不允许可能导致降位）
3. 允许 Googlebot 不触发年龄限制抓取（阻止可能导致全站降位）
4. 将露骨内容放在单独网域/子网域（`explicit.example.com`）
5. 使用元数据标记：`<meta name="rating" content="adult">`
6. 视频站点地图：`<video:family_friendly>no</video:family_friendly>`

### 被错误标记的排查

1. 分别测试安全搜索开启/关闭时网页是否显示
2. 用 `site:` 运算符检查全站状态
3. 检查是否错误添加了 `rating=adult`
4. 检查视频站点地图是否错误标记 `family_friendly:no`
5. 检查用户生成内容审核机制
6. 修正后等待 2-3 个月再提交审核（重新分类需要时间）

### 注意事项

- 分类器最多需要 2-3 个月处理
- 大量 CSAM 移除要求 → 全站降位
- 大量未经同意露骨图像移除要求 → 全站降位
- 模糊处理图片不一定能规避分类
