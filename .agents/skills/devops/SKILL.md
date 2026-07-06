---
name: devops
description: 量潮 DevOps 顶层入口。当用户说"devops"、"发布"、"规划"、"测试"、"评审"、"CI/CD"、"流水线"时使用，分发到对应子 skill。
---

# devops — 量潮 DevOps 顶层入口

> DevOps 全生命周期入口。根据用户需求分发给对应子 skill。

## 生命周期

```
计划(plan) → 编码(code) → 构建(build) → 测试(test) → 发布(release) → 部署(deploy) → 运维(operate)
```

## 子 skill

| Skill | 命令 | 触发词 | 覆盖范围 |
|-------|------|--------|---------|
| `devops-plan` | `plan status/clean/doctor` | 规划、进度、ROADMAP | ROADMAP 查看/清理/修复/写入 |
| `devops-test` | `test status`, `doctor status` | 测试、覆盖率、CI 失败 | 测试运行、覆盖率检查、环境诊断 |
| `devops-release` | `release publish` | 发布、发版、bump、tag | 版本号审议、publish、CHANGELOG、GitHub Release |
| `devops-review` | — | 评审、review、代码质量 | 代码评审 checklist、质量评估 |

## 评审标准（各 skill 通用）

源自实际评审经验，适用于所有代码变更。

### 指标基线

| 指标 | 合格 | 优秀 |
|------|------|------|
| 测试通过率 | 100% | 100% |
| 行覆盖率 | ≥ 80% | ≥ 95% |
| 编译 warning | 0 | 0 |
| clippy warnings | pre-existing 可接受 | 0 新增 |

### 必查项

- **gix-first**：是否有新增的 `std::process::Command::new("git")`？读操作必须用 gix。
- **重复检测**：新函数是否与 toolkit 已有 API 重复？（如 `normalize_version`、`detect_language`）
- **依赖审查**：是否有不必要的新依赖？是否可用已有依赖替代？
- **path/registry**：本地 path 依赖是否已切回 crates.io registry？
- **错误处理**：使用自定义错误类型而非 `String`？实现了 `Display` + `Error` + `From`？
- **测试分层**：纯逻辑→单元测试，I/O 边界→集成测试（`tempfile`），异常路径全覆盖。
- **CHANGELOG**：breaking change 必须标注并写明迁移路径。

### 问题分级

| 级别 | 含义 | 处理方式 |
|------|------|---------|
| P0 | 不修不能发 | 正确性 bug、安全漏洞 |
| P1 | 建议发前修 | 架构背离、重复实现、不合理依赖 |
| P2 | 可发后修 | 代码风格、可维护性 |
| P3 | 值得记录 | 常规建议，不阻塞 |

## CLI 架构原则

- **读操作**：gix 优先（tag 读取、子模块扫描、日志收集）
- **写操作**：git2 兜底（tag 创建/删除、push gix 不支持的操作）
- **零新进程**：不新增 `Command::new("git")` 调用
- **委托优先**：CLI 层逻辑优先委托 toolkit 库，不手写重复实现

## 使用方式

用户说"xxx"时：

1. 识别属于哪个子 skill 的覆盖范围
2. 取对应的 skill 指令执行
3. 跨 skill 时按生命周期顺序推进（plan → code → build → test → release）

```bash
# 典型完整流程
qtcloud-devops plan status        # 规划进度
qtcloud-devops test status        # 测试状态
qtcloud-devops release publish --dry-run  # 预演发布
qtcloud-devops release publish -y         # 正式发布
```
