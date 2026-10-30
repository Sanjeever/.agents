---
name: conventional-commits
description: 按 Conventional Commits 1.0.0 生成、校验、改写 Git 提交信息。适用于用户要求根据 staged changes、git diff、PR 摘要、squash merge、变更说明生成 commit message，检查或修正已有 commit message，判断提交类型、scope、breaking change 标记或 SemVer 影响时。
---

# Conventional Commits

使用 Conventional Commits 1.0.0 为 Git 变更生成、校验或改写提交信息。默认输出英文 commit message，除非用户明确要求中文，或仓库历史明显使用中文。

## 工作流程

1. 优先查看真实变更：使用 `git diff --staged`、`git diff`、`git status`、相关文件内容或用户提供的摘要。
2. 选择最窄且准确的 type。不要为了“看起来重要”而把维护性变更写成 `feat` 或 `fix`。
3. 只有在 scope 能明确定位包、模块、功能、目录或文件组时才添加 scope。
4. description 使用简短祈使句或名词化短语，避免句号结尾，避免夸大影响。
5. 只有确认存在兼容性破坏、公共 API 行为变化、配置格式变更、迁移要求或废弃支持时，才使用 `!` 或 `BREAKING CHANGE:`。
6. 当一个变更同时包含多个独立意图时，优先建议拆分提交；如果用户只要一个提交信息，选择主导意图。

## 格式

单行格式：

```text
<type>[optional scope][optional !]: <description>
```

需要正文或 footer 时：

```text
<type>[optional scope][optional !]: <description>

<body>

<footer>
```

默认只输出一条 commit message。用户要求解释、候选项或校验报告时，再附加说明。

## Type 选择

- `feat`: 新增用户可见能力、接口、配置能力或业务行为。
- `fix`: 修复错误、回归、异常行为或不符合预期的结果。
- `docs`: 仅修改文档、注释文档、说明文本或示例文档。
- `test`: 仅新增或调整测试，不改变生产行为。
- `refactor`: 重构代码结构，不改变外部行为。
- `perf`: 明确改善性能、资源使用或响应速度。
- `build`: 构建系统、依赖、打包、发布配置或工具链变更。
- `ci`: CI/CD 配置、流水线、自动化检查或部署流程变更。
- `style`: 仅格式化、空白、缩进、代码风格，且无逻辑变化。
- `chore`: 维护性杂项，且不适合以上类型。
- `revert`: 回滚已有提交。需要时在 footer 中引用被回滚提交。

## Scope 规则

scope 使用小写短名，优先来自仓库已有约定、包名、目录名、模块名或功能名。

示例：

```text
fix(auth): reject expired refresh tokens
docs(api): clarify pagination parameters
build(deps): update vite
```

不要为了填充格式强行添加 scope。不确定时省略。

## Breaking Change

可用两种形式标记 breaking change：

```text
feat(api)!: remove legacy token endpoint
```

或：

```text
feat(api): remove legacy token endpoint

BREAKING CHANGE: clients must use the OAuth token endpoint instead.
```

如果变更会影响 SemVer，按以下顺序判断：

- 任意 type 带 `!` 或 `BREAKING CHANGE:`：MAJOR。
- `feat` 且无 breaking change：MINOR。
- `fix` 且无 breaking change：PATCH。
- 其他 type 默认不隐含 SemVer bump，除非项目工具另有约定。

## 校验和改写

校验已有提交信息时，检查：

- 是否包含 type、冒号和冒号后的空格。
- scope 是否放在 type 后的括号中。
- description 是否非空且简洁。
- `!` 是否紧贴在 type/scope 后、冒号前。
- `BREAKING CHANGE:` 是否大写，并位于 footer。
- footer token 是否使用 Git trailer 风格。

改写时保留用户表达的真实意图，不添加 diff 中看不出的影响。

## 参考

需要核对完整规范或边界问题时，读取 `references/spec.md`。
