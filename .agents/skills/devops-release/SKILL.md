---
name: devops-release
description: 量潮 DevOps 包发布流程。当用户说"发布"、"发版"、"切版本"、"publish"、"release"，或 ROADMAP 已实施完成需要正式发布时使用。
---

# devops-release — 量潮 DevOps 包发布流程

## 流程

### 前置检查

```bash
qtcloud-devops status
```

统一 status 按生命周期顺序聚合输出：contract → doctor → plan → code → build → test → release。确认以下全部通过：

- 构建成功
- 测试全部通过
- 版本一致（配置文件版本与最新 tag 一致）
- 工作区干净

### 决定版本号

1. **确定 scope** — 检查最近提交（上次 tag 到 HEAD）涉及的文件路径。如有多个 scope 改动，询问用户如何处理。无匹配时使用 `(root)`（即无前缀）。
2. **是否发版** — 有用户可见的逻辑改动（feat / fix / refactor / test）则发版；仅 typo、注释、CI 配置、README、chore 等非逻辑改动则**不发**，攒到下次。
3. **确定增量与预发布** — 参考[版本号规则](#版本号规则)推断 minor/patch、直发正式还是走预发布。

### 发布

```bash
qtcloud-devops release publish -v <version> -y
```

`release publish` 内部自动完成：更新配置文件版本 → 生成 CHANGELOG → 创建 tag → 推送 → 创建 GitHub Release。

**所有步骤都是幂等的。** 超时或失败后直接重跑相同的 `publish` 命令，不会产生重复 tag 或 Release。

#### tag 或 Release 已存在（重新发布）

修复代码后，用 `--force` 强制重新发布：

```bash
qtcloud-devops release publish -v <version> -y -f
```

`--force` / `-f` 自动完成：

- 删除已存在的 GitHub Release
- 删除已存在的远端 tag
- 删除本地 tag
- 重新创建 tag → 推送 → 创建 Release

等效于手动执行以下步骤，但一次完成：

```bash
git push --delete origin cli/v0.8.3       # 删除远端 tag
gh release delete cli/v0.8.3 --yes        # 删除 GitHub Release
qtcloud-devops release publish -v cli/v0.8.3 -y
```

#### CI 未触发

监听 `release: [published]` 事件。如果 CI 未触发，用 `--force` 重新发布以触发：

```bash
qtcloud-devops release publish -v <version> -y -f
```

### 发布后检查

```bash
qtcloud-devops status
```

或只看发布状态：

```bash
qtcloud-devops release status
```

## 版本号规则

### 格式

- `vX.Y.Z` — 单包仓库版
- `scope/vX.Y.Z` — monorepo 正式版，如 `cli/v0.8.3`
- `scope/vX.Y.Z-prerelease` — 预发布版，如 `cli/v0.8.3-rc.1`

### minor 还是 patch

- **重大变更**（新增命令/模块、重构架构、破坏性改动）→ **minor**（Y+1）
- **常规变更**（小 feat、fix、refactor 等）→ **patch**（Z+1）
- 当前版本为预发布系列 → 同阶段递增序号

### 直发正式还是走预发布

| 场景 | 发版方式 | 理由 |
|------|---------|------|
| patch 级别的简单修复 | **直发正式版** | 风险低，已有信心 |
| minor 级别的新功能 | **走预发布**（-rc.1） | 需要验证再上正式 |
| 大版本周期开发（多个 minor 积攒）| **走预发布**（alpha→beta→rc） | 分阶段验证，逐步稳定 |
| 正式版发布后发现紧急 bug | **直发 patch-rc.1** → 验证后转正式 | 快速修复，验证后发正式 |
| 当前已是预发布系列 | **同阶段递增序号** | 继续当前验证周期 |

**简单原则**：patch 直发正式，minor 先预发布。不确定时问用户。

### 预发布阶段

| 阶段 | 标签 | 起始时机 |
|------|------|---------|
| Alpha | `-alpha.N` | 大版本早期，功能未完成，API 不稳定 |
| Beta | `-beta.N` | 功能基本完成，API 冻结，供外部验证 |
| Release Candidate | `-rc.N` | 功能冻结，只修 bug，准备正式发版 |
| 正式版 | 无后缀 | 稳定发布 |

**晋级规则：**

- `alpha.k` → 功能够用时 → `beta.1`
- `beta.k` → 评审通过 → `rc.1`
- `rc.k` → 验证通过 → 去掉后缀发正式版
- 阶段切换时序号重置（`alpha.2` → `beta.1`）
- **正式发版（去掉后缀）由人类决定**，AI 不做

### major

**AI 不做 major bump。** 涉及 breaking change 或大版本升级，由人类指定版本号。

### 原则

**AI 做判断，人类做决定。** AI 负责常规判断（scope、发不发、minor/patch、预发布序号递增），重大拐点由人类控制（major 版本、正式发版去掉后缀）。不确定时问用户。
