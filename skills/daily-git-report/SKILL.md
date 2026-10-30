---
name: daily-git-report
description: 根据 Git commit、git diff 和工作区状态生成简洁日报。用于用户要求查看今天的 commit、总结代码变更、写日报、给老板汇报、把 Git 记录整理成编号短句时；默认面向略懂技术的管理者，少写实现细节，只写实际完成的事项。
---

# Git 日报

## 目标

把 Git 活动整理成适合汇报的短日报。只写实际完成的事项，不写开发流水账或效果评价。

## 默认作者邮箱

用户没有特别说明时，只统计以下邮箱对应的 commit：

- `hsr2026@qq.com`
- `forgetfulcode@gmail.com`
- `bestoftribe@foxmail.com`

如果用户明确要求“所有 commit”或“全仓库变更”，再取消作者筛选。

## 工作流程

1. 确认日期。
   - 用户给出日期时，使用该日期。
   - 用户说“今天”时，使用当前本地日期。

2. 收集 Git 事实。
   - 读取指定日期内、默认作者邮箱下的 commit、提交说明和统计信息。
   - 查看当前工作区状态。
   - 查看 staged 和 unstaged 的 tracked diff 统计。
   - 用户要求“所有代码变更”时，查看当天首个 commit 的父提交到最后一个 commit 的累计 diff，不只看 commit message。

3. 区分已提交和未提交内容。
   - tracked diff 存在且相关时，可以纳入“未提交变更”。
   - untracked 文件只在明显相关时提及。
   - 不要把未提交内容说成已完成，除非用户明确要完整工作记录。

4. 合并噪声。
   - 多个版本号提交合并成一条。
   - 多个 UI 体验优化提交合并成一到两条。
   - 测试只写“补充自动测试”或“增加测试覆盖”。
   - 默认不写 commit hash、文件路径、行数、函数名。

5. 面向汇报对象改写。
   - 给老板看时，优先保留与安全性、稳定性、体验、交付、排障效率相关的完成事项。
   - 给技术同事看时，可以多保留一点实现背景。
   - 中文请求用简体中文回复。

6. 输出格式。
   - 默认使用编号短句。
   - 每条是一个简洁陈述句，以“实现”“修复”“优化”“调整”等动作开头，只说做了什么。
   - 日报正文不使用“了”“——”“：”。
   - 通常 5 到 8 条，实际事项较少时据实减少。
   - 不加长前言，不附原始 Git 输出。

## 常用命令

按当前 shell 调整引号和转义。默认先筛选用户常用邮箱：

```bash
git log --since="YYYY-MM-DD 00:00:00" --until="YYYY-MM-DD 23:59:59" --author="hsr2026@qq.com\\|forgetfulcode@gmail.com\\|bestoftribe@foxmail.com" --date=iso --pretty=format:"%H%x09%ad%x09%ae%x09%s" --stat
git status --short
git diff --stat
git diff --cached --stat
```

如果 `--author` 正则在当前 Git 或 shell 中没有匹配结果，分别按三个邮箱查询后合并：

```bash
git log --since="YYYY-MM-DD 00:00:00" --until="YYYY-MM-DD 23:59:59" --author="hsr2026@qq.com" --date=iso --pretty=format:"%H%x09%ad%x09%ae%x09%s" --stat
git log --since="YYYY-MM-DD 00:00:00" --until="YYYY-MM-DD 23:59:59" --author="forgetfulcode@gmail.com" --date=iso --pretty=format:"%H%x09%ad%x09%ae%x09%s" --stat
git log --since="YYYY-MM-DD 00:00:00" --until="YYYY-MM-DD 23:59:59" --author="bestoftribe@foxmail.com" --date=iso --pretty=format:"%H%x09%ad%x09%ae%x09%s" --stat
```

累计 diff 使用当天纳入统计的第一个 commit 的父提交到最后一个 commit。

## 输出口径

推荐：

```text
1. 发布 3 个小版本。
2. 修复配置保存时密钥被误覆盖的问题。
3. 优化配置、测试和应用状态的操作流程。
```

避免：

```text
1. 修改 frontend/src/App.tsx 397 行，新增 modelDirtyRef 和 platformDirtyRef。
2. 提交 e076055 修复 keepExistingIfMaskedSecret。
```

## 安全规则

不要输出 secret、token、API key、私有地址或 `.env` 中的敏感值。如果 diff 中出现疑似密钥，只报告“发现敏感信息风险”，不要引用原文。
