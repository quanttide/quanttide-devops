---
name: devops
description: 量潮 DevOps 顶层入口。当用户说"devops"、"发布"、"规划"、"测试"、"评审"、"CI/CD"、"流水线"时使用，分发到对应子 skill。
---

# devops — 量潮 DevOps 顶层入口

> DevOps 全生命周期入口。本 skill 只做 `status`（看数据）和 `audit`（做评价），不做写入操作。写入由各子 skill 处理。

## 流程

```
status（看数据）→ audit（对照标准做评价）→ 分发到子 skill（执行）
```

## status — 看数据

> status 是 CLI 命令，聚合展示当下的原始数据，不做评价。

| 命令 | 用途 |
|------|------|
| `qtcloud-devops status` | 聚合概览：contract → doctor → plan → code → build → test → release |
| `qtcloud-devops doctor status` | 系统诊断：检查 git/gh/cargo/python 等工具链 |
| `qtcloud-devops code status` | 查看组件同步状态 |
| `qtcloud-devops build status` | 查看构建状态 |
| `qtcloud-devops plan status [scope]` | 查看 ROADMAP 规划进度 |
| `qtcloud-devops test status` | 查看测试状态和覆盖率 |
| `qtcloud-devops release status` | 查看发布状态（tag、CHANGELOG、GitHub Release） |

## audit — 做评价

> CLI 暂无 `audit` 命令。audit 是 AI agent 拿到 status 数据后，对照以下标准做人工评审。
> status 和 audit 都在演化中，数据维度和 checklist 会随评审经验积累持续补充。

### 指标基线

| 指标 | 合格 | 优秀 |
|------|------|------|
| 测试通过率 | 100% | 100% |
| 行覆盖率 | ≥ 80% | ≥ 95% |
| 编译 warning | 0 | 0 |
| clippy warnings | pre-existing 可接受 | 0 新增 |

### 必查项

- **gix-first**：是否有新增的 `std::process::Command::new("git")`？读操作必须用 gix
- **重复检测**：新函数是否与 toolkit 已有 API 重复？（如 `normalize_version`、`detect_language`）
- **依赖审查**：是否有不必要的新依赖？是否可用已有依赖替代？
- **path/registry**：本地 path 依赖是否已切回 crates.io registry？
- **错误处理**：使用自定义错误类型而非 `String`？实现了 `Display` + `Error` + `From`？
- **测试分层**：纯逻辑→单元测试，I/O 边界→集成测试（`tempfile`），异常路径全覆盖
- **CHANGELOG**：breaking change 必须标注并写明迁移路径

### 问题分级

| 级别 | 含义 | 处理方式 |
|------|------|---------|
| P0 | 不修不能发 | 正确性 bug、安全漏洞 |
| P1 | 建议发前修 | 架构背离、重复实现、不合理依赖 |
| P2 | 可发后修 | 代码风格、可维护性 |
| P3 | 值得记录 | 常规建议，不阻塞 |

## 子 skill

| Skill | 触发词 | 覆盖范围 |
|-------|--------|---------|
| `devops-plan` | 规划、进度、ROADMAP | ROADMAP 查看/清理/修复/写入 |
| `devops-test` | 测试、覆盖率、CI 失败 | 测试运行、覆盖率检查、环境诊断 |
| `devops-release` | 发布、发版、bump、tag | 版本号审议、publish、CHANGELOG、GitHub Release |
| `devops-code` | 组件、子模块、同步 | 组件同步状态查看和同步操作 |
| `devops-build` | 构建、build、编译 | 构建状态查看 |
| `devops-plan` | 规划、进度、ROADMAP | ROADMAP 查看/清理/修复/写入 |

## 路由规则

1. 匹配触发词 → 取对应子 skill 指令执行
2. 不明确时 → 先跑 `status` 看数据，然后 `audit` 对照标准评价，再按生命周期顺序推荐下一步
3. 跨 skill 任务 → 依次执行相关子 skill
