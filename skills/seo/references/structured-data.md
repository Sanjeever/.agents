# 结构化数据参考

覆盖结构化数据政策、格式、主要类型和验证方法。

## 1. 通用政策

### 技术政策

| 规则 | 说明 |
|------|------|
| **格式** | JSON-LD（推荐）、微数据、RDFa |
| **访问** | 不使用 robots.txt 或 noindex 阻止 Googlebot 访问 |
| **内容一致性** | 标记的内容必须是用户可见的内容 |
| **完整性** | 填写所有必需属性，尽可能填写建议属性 |
| **位置** | 将结构化数据放在要描述的网页上，而非汇总页 |
| **不含无关内容** | 不标记网页未呈现的内容 |

### 质量指南

- 标记内容必须真实反映页面内容
- 不要以虚假或误导方式标记内容
- 标记正确也不能保证展示富媒体结果
- data-vocabulary.org 已不再支持

## 2. 丰富搜索结果（Enriched Search）

要求更严格的功能：招聘信息、食谱、活动
- 所有必需属性完整
- 标记信息与页面一致
- **叶级内容**：仅适用于具体内容页，不适用于列表/类别页

## 3. 主要结构化数据类型

### 3.1 通用类型

| 类型 | 用途 | 关键属性 | 备注 |
|------|------|---------|------|
| **Organization** | 组织信息、徽标 | name、url、logo | 放首页 |
| **WebSite** | 网站名称 | name、url、alternateName | 仅首页，域名级 |
| **BreadcrumbList** | 面包屑导航 | itemListElement（≥2 个 ListItem）含 position、name、item | 几乎所有页面适用 |

### 3.2 内容类型

| 类型 | 用途 | 关键属性 |
|------|------|---------|
| **Article** | 文章 | headline、author、datePublished、dateModified、image |
| **NewsArticle** | 新闻报道 | 含 Article 属性 + 新闻特定属性 |
| **BlogPosting** | 博客帖子 | 含 Article 属性 |
| **Course** | 课程列表 | name、description、provider |
| **Dataset** | 数据集 | name、description、license |
| **Event** | 活动 | name、startDate、location（含 address） |
| **Book** | 图书 | name、author、isbn |
| **Movie** | 影片 | name、image、dateCreated、director |

### 3.3 电商类型

| 类型 | 用途 | 关键必需属性 |
|------|------|------------|
| **Product + Offer**（商品摘要） | 商品信息展示（非直接购买） | name + 至少一个（review / aggregateRating / offers） |
| **Product + Offer**（商家信息） | 商品信息（可直接购买） | name、offers（含 price、priceCurrency、availability） |
| **ProductGroup** | 多款式/规格 | variesBy、productGroupID、hasVariant（Product 数组） |
| **Review** | 评价 | author、itemReviewed、reviewRating |
| **AggregateRating** | 总体评分 | ratingValue、ratingCount 或 reviewCount |
| **MerchantReturnPolicy** | 退货政策 | returnPolicyCategory、merchantReturnDays |
| **ShippingService** | 配送政策 | shippingRate、deliveryTime |
| **MemberProgram** | 会员回馈 | name、programMembership |

### 3.4 特殊功能类型

| 类型 | 用途 | 关键属性 | 资格限制 |
|------|------|---------|---------|
| **FAQPage** | FAQ 页面 | mainEntity（Question 数组） | **仅限**政府/健康权威机构网站 |
| **QAPage** | 问答页面 | mainEntity（含已接受的 Answer） | 需要已接受的答案 |
| **HowTo** | 操作指南 | name、step（HowToStep 数组） | 文字步骤，不用图片替代 |
| **Recipe** | 食谱 | name、image、recipeIngredient、recipeInstructions | |
| **VideoObject** | 视频 | name、thumbnailUrl、uploadDate、contentUrl 或 embedUrl | |
| **JobPosting** | 招聘信息 | title、description、datePosted、hiringOrganization、jobLocation | |
| **LocalBusiness** | 本地商家 | name、address、telephone | 配合 Google 商家资料 |
| **ClaimReview** | 事实核查 | claimReviewed、author、reviewRating | |
| **Speakable** | 可朗读内容 | speakable（cssSelector） | Beta 版 |
| **Subscription/PaywalledContent** | 付费内容 | isAccessibleForFree、hasPart | |
| **DiscussionForumPosting** | 论坛帖子 | author、datePublished、text | |
| **ProfilePage** | 个人资料 | mainEntity（Person） | |
| **SoftwareApp** | 软件应用 | name、applicationCategory、operatingSystem | |
| **MathSolver** | 数学求解器 | mathExpression | |
| **VacationRental** | 民宿 | name、description、address | |
| **Quiz** | 知识问答 | 含 Question 和 Answer | 教育用途 |

### 3.5 图片相关

| 标记方式 | 用途 |
|---------|------|
| ImageObject（图片元数据） | 许可和归属信息（creator、creditText、copyrightNotice、license） |
| primaryImageOfPage | 指定页面首选图片 |
| og:image | Open Graph 协议 |

## 4. JSON-LD 最佳实践

### 推荐模板结构

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "[类型]",
  "name": "[名称]",
  ...
}
</script>
```

### 实施原则

- 放在 `<head>` 或 `<body>` 中，推荐 `<head>`
- 使用 schema.org 类型和属性名
- 值为 URL 时使用完全限定 URL
- 日期使用 ISO 8601 格式
- 价格使用数字格式，币种用 ISO 4217
- 可通过 Google 跟踪代码管理器动态注入
- 使用 GTM 变量从页面提取数据，避免在 GTM 中硬编码导致不一致

## 5. 常见错误排查

| 错误 | 后果 | 解决 |
|------|------|------|
| 缺少必需属性 | 无法显示富媒体结果 | 补全必需属性 |
| 标记用户看不到的内容 | 违反政策，可能受人工处置 | 只标记可见内容 |
| data-vocabulary.org 使用 | 已失效 | 迁移到 schema.org |
| robots.txt 屏蔽资源 | JS/CSS 未加载影响数据提取 | 允许抓取关键资源 |
| 列表页标记单个实体 | 违反叶级内容原则 | 只在详情页标记具体实体 |
| 动态注入与 HTML 冲突 | 数据不一致 | 统一数据源 |
| Product 标记无价格/库存 | 无法显示商家信息 | 补充 offers（price + availability） |

## 6. 验证方法

1. **富媒体搜索结果测试工具** — 针对单个 URL 测试
2. **Schema Markup Validator** — 验证 schema.org 语法
3. **Search Console 富媒体搜索结果报告** — 监控全网状态（错误/警告/有效项目趋势）
4. **网址检查工具** — 查看 Googlebot 看到的已渲染结构化数据

## 7. 结构化数据与 JavaScript

- 可通过 JavaScript 动态生成结构化数据
- 注意动态生成的 Product 标记可能导致购物抓取频率降低
- 使用 GTM 时优先通过变量提取页面数据，而非在 GTM 中复制数据
- 验证：用网址检查工具查看 Googlebot 渲染后的结构化数据

## 8. 关键提醒

- 即使所有标记正确、验证通过，Google 也**不保证**一定会显示富媒体结果
- 结构化数据是资格条件，不是显示保证
- 与页面实质内容质量共同决定是否展示
- 违反政策可能导致整个网站的富媒体结果资格被取消
