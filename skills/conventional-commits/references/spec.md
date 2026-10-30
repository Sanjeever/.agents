# Conventional Commits 1.0.0 参考摘要

来源：

- 官方规范：https://www.conventionalcommits.org/en/v1.0.0/
- 关于页面和工具列表：https://www.conventionalcommits.org/en/about/

## 核心结构

提交信息结构：

```text
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

实际生成时允许使用 breaking change 简写：

```text
<type>[optional scope]!: <description>
```

## 必要规则

- commit 必须以 type 开头，例如 `feat`、`fix`、`docs`。
- type 后可以带 scope，scope 放在括号中，例如 `fix(parser):`。
- type 或 scope 后可以带 `!`，表示 breaking change。
- 前缀必须以冒号和空格结束。
- description 必须紧跟冒号和空格，是变更的短摘要。
- body 可选，必须与 description 间隔一个空行。
- footer 可选，必须与 body 间隔一个空行。
- breaking change 必须通过 `!` 或 footer 中的 `BREAKING CHANGE:` 标记。
- `BREAKING CHANGE` 必须大写；`BREAKING-CHANGE` 作为 footer token 与其同义。

## SemVer 对应

- `fix` 对应 PATCH。
- `feat` 对应 MINOR。
- 任意 type 只要带 breaking change 标记，对应 MAJOR。
- 除 `feat`、`fix` 和 breaking change 外，其他 type 没有规范内置的 SemVer 含义。

## 常见 type

Conventional Commits 只强制定义 `feat` 和 `fix` 的含义，允许使用其他 type。日常可采用以下集合：

- `build`: 构建系统、依赖、打包、工具链。
- `chore`: 维护性杂项。
- `ci`: CI/CD 或自动化流水线。
- `docs`: 文档。
- `feat`: 新功能。
- `fix`: 修复 bug。
- `perf`: 性能优化。
- `refactor`: 不改变外部行为的重构。
- `style`: 格式、空白、代码风格。
- `test`: 测试。
- `revert`: 回滚。

## 示例

```text
docs: correct changelog spelling
```

```text
feat(lang): add Polish language
```

```text
fix: prevent duplicate requests

Track the latest request id and ignore stale responses.

Refs: #123
```

```text
feat(api)!: remove legacy token endpoint
```

```text
feat: remove legacy token endpoint

BREAKING CHANGE: clients must use the OAuth token endpoint instead.
```

## 决策建议

- 如果一个提交可以拆成多个独立意图，优先拆分。
- 初始开发阶段也按已有发布产品的标准写提交信息。
- type 大小写不是规范核心，但项目内应保持一致；默认使用小写。
- 如果 squash merge 由维护者统一写提交信息，普通贡献者不一定需要每个中间提交都符合规范。
- Conventional Commits 没有强制定义 revert 语义；需要时使用 `revert` type，并在 footer 引用提交 SHA。
