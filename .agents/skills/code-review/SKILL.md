---
name: code-review
description: 代码审查。使用 qtcloud-code review 命令对指定目录进行静态代码分析，检测长函数、长参数列表、unsafe 块、未用变量、缺少测试等问题。
---

# code-review — 代码审查

> 使用 `qtcloud-code review` 对源码目录执行静态分析检测。
>
> 与 `qtcloud-devops code audit` 的分工：devops 做文本级门禁（红/绿），code-review 做 AST 级诊断（精确到行号）。

## 命令

### 审查目录

```bash
qtcloud-code review <path>
```

| 选项 | 用途 |
|------|------|
| `--format json` | JSON 格式输出 |
| `--rules <rules>` | 仅运行指定规则（逗号分隔，如 `long-function,missing-tests`） |
| `--status` | 写入 STATUS.md（供 `qtcloud-devops code audit` 聚合展示） |

### 列出可用规则

```bash
qtcloud-code list-rules
qtcloud-code list-rules --json
```

## 使用场景

| 场景 | 命令 |
|------|------|
| 提交前自查 | `qtcloud-code review src/` |
| CI 门禁 | `qtcloud-code review . --status`（写入 STATUS.md，失败退出码非 0） |
| 仅检测特定规则 | `qtcloud-code review src/ --rules long-function,missing-tests` |
| JSON 输出对接工具 | `qtcloud-code review . --format json` |
